---
title: 啟動和停止叢集節點以測試 Azure Service Fabric 應用程式 | Microsoft Docs
description: 了解如何使用錯誤插入，藉由啟動和停止叢集節點的方式，測試 Service Fabric 應用程式。
services: service-fabric
documentationcenter: .net
author: LMWF
manager: rsinha
editor: ''
ms.assetid: f4e70f6f-cad9-4a3e-9655-009b4db09c6d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 6/12/2017
ms.author: lemai
ms.openlocfilehash: 95c3726caeb19d6bbf7153533951bb18cd7d0e57
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44055398"
---
# <a name="replacing-the-start-node-and-stop-node-apis-with-the-node-transition-api"></a>以節點轉換 API 取代啟動節點和停止節點 API

## <a name="what-do-the-stop-node-and-start-node-apis-do"></a>啟動節點和停止節點 API 有什麼功用？

停止節點 API (受控︰[StopNodeAsync()][stopnode]、PowerShell：[Stop-ServiceFabricNode][stopnodeps]) 會停止 Service Fabric 節點。  Service Fabric 節點是處理序，不是 VM 或機器 – VM 或機器仍將繼續執行。  在文件的其餘部分，「節點」是指 Service Fabric 節點。  停止節點時是將其放入「停止」狀態，此時節點不是叢集的成員，無法裝載服務，因此就像是個「關閉」的節點。  在將錯誤插入系統以測試應用程式系統時，這非常有用。  啟動節點 API (受控︰[StartNodeAsync()][startnode]、PowerShell：[Start-ServiceFabricNode][startnodeps]) 會反轉停止節點 API，將節點帶回一般狀態。

## <a name="why-are-we-replacing-these"></a>為什麼要取代它們？

如先前所述，「停止」的 Service Fabric 節點是使用停止節點 API 刻意鎖定目標的節點。  「關閉」節點是基於其他因素 (例如 VM 或機器已關閉) 而關閉的節點。  使用停止節點 API 時，系統不會公開資訊，您無法區分「停止」節點和「關閉」節點。

此外，這些 API 傳回的某些錯誤的描述不明確。  例如，在已經「停止」的節點上叫用停止節點 API 會傳回 *InvalidAddress* 錯誤。  這種經驗可以改善。

此外，節點停止的持續時間是「無限期」，直到 Start Node API 被叫用。  我們發現這可能會造成問題，比較容易出錯。  例如，我們已經看到使用者在節點叫用停止節點 API 之後將它忘得一乾二淨的問題。  之後，將無從得知它是「關閉」或「停止」節點。


## <a name="introducing-the-node-transition-apis"></a>節點轉換 API 簡介

我們已經用一組新的 API 解決上述問題。  新的節點轉換 API (受控︰[StartNodeTransitionAsync()][snt]) 可用來將 Service Fabric 節點轉換為「停止」狀態，或將它從「停止」狀態轉換為一般狀態。  請注意，此 API 名稱中的 "Start" 不是啟動節點之意。  是指開始系統將執行、會將節點轉換為「停止」或啟動狀態的非同步作業。

**使用量**

如果節點轉換 API 被呼叫時沒有擲回例外狀況，則系統已接受非同步作業，並將執行它。  成功的呼叫並不表示作業完成。  若要取得作業的目前狀態資訊，請呼叫節點轉換進度 API (受控︰ [GetNodeTransitionProgressAsync()][gntp])，並搭配此作業叫用節點轉換 API 時使用的 guid。  節點轉換進度 API 會傳回 NodeTransitionProgress 物件。  此物件的 State 屬性會指定作業的目前狀態。  如果狀態是 "Running"，則作業正在執行。  如果是 "Completed"，則作業完成沒有錯誤。  如果是 "Faulted"，則表示執行作業發生問題。  Result 屬性的 Exception 屬性會指出問題為何。  請參閱 https://docs.microsoft.com/dotnet/api/system.fabric.testcommandprogressstate 了解 State 屬性的相關資訊，以及之後的「範例用法」一節中的程式碼範例。


**區分停止節點和關閉節點** 如果是使用節點轉換 API *停止*節點，節點查詢的輸出 (受控：[GetNodeListAsync()][nodequery]、PowerShell：[Get-ServiceFabricNode][nodequeryps]) 將顯示此節點的 *IsStopped* 屬性值為 true。  請注意，這和 NodeStatus屬性的值 (Down) 不同。  如果 NodeStatus屬性的值為 Down，但 IsStopped 為 false，則節點並非使用節點轉換 API 停止，而是因其他原因而「關閉」。  如果 IsStopped屬性為 true，而NodeStatus 屬性為 Down，則是使用節點轉換 API 停止節點。

使用節點轉換 API 啟動「停止」節點會將它恢復運作，再次成為叢集的一般成員。  節點查詢 API 的輸出會顯示IsStopped 是 false，而 NodeStatus 是 Down 以外的值 (例如 Up)。


**有限的持續時間** 使用節點轉換 API停止節點時，其中一個必要參數 stopNodeDurationInSeconds 表示該節點要保持「停止」的時間，單位為秒。  這個值必須在允許範圍內，最低 600，最高 14400。  此時間過期之後，節點本身會自動重新啟動到 Up 狀態。  請參閱以下範例 1 的範例用法。

> [!WARNING]
> 避免混用節點轉換 API 和啟動節點、停止節點 API。  建議只使用節點轉換 API。  > 如果已經使用停止節點 API 停止節點，則應該先使用啟動節點 API 啟動它，再使用 > 節點轉換 API。

> [!WARNING]
> 在相同節點上無法平行呼叫多個節點轉換 API。  在這種情況下，節點轉換 API 將   > 擲回 FabricException，其 ErrorCode 屬性的值為 NodeTransitionInProgress。  一旦特定節點的節點轉換  > 已經開始，您應該等到作業到達終止狀態 (Completed、Faulted 或 ForceCancelled)，才開始  > 對同一節點進行新的轉換。  允許在不同節點上平行呼叫節點轉換。


#### <a name="sample-usage"></a>範例用法


**範例 1** - 下列範例會使用節點轉換 API 來停止節點。

```csharp
        // Helper function to get information about a node
        static Node GetNodeInfo(FabricClient fc, string node)
        {
            NodeList n = null;
            while (n == null)
            {
                n = fc.QueryManager.GetNodeListAsync(node).GetAwaiter().GetResult();
                Task.Delay(TimeSpan.FromSeconds(1)).GetAwaiter();
            };

            return n.FirstOrDefault();
        }

        static async Task WaitForStateAsync(FabricClient fc, Guid operationId, TestCommandProgressState targetState)
        {
            NodeTransitionProgress progress = null;

            do
            {
                bool exceptionObserved = false;
                try
                {
                    progress = await fc.TestManager.GetNodeTransitionProgressAsync(operationId, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                    exceptionObserved = true;
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                    exceptionObserved = true;
                }

                if (!exceptionObserved)
                {
                    Console.WriteLine("Current state of operation '{0}': {1}", operationId, progress.State);

                    if (progress.State == TestCommandProgressState.Faulted)
                    {
                        // Inspect the progress object's Result.Exception.HResult to get the error code.
                        Console.WriteLine("'{0}' failed with: {1}, HResult: {2}", operationId, progress.Result.Exception, progress.Result.Exception.HResult);

                        // ...additional logic as required
                    }

                    if (progress.State == targetState)
                    {
                        Console.WriteLine("Target state '{0}' has been reached", targetState);
                        break;
                    }
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (true);
        }

        static async Task StopNodeAsync(FabricClient fc, string nodeName, int durationInSeconds)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            // Create a Guid
            Guid guid = Guid.NewGuid();

            // Create a NodeStopDescription object, which will be used as a parameter into StartNodeTransition
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, durationInSeconds);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStopDescription from above, which will stop the target node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    // This is retryable
                }
                catch (FabricTransientException fte)
                {
                    // This is retryable
                }

                // Backoff
                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until hte desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**範例 2** - 下列範例會啟動「停止」節點。  它會使用第一個範例中的一些協助程式方法。

```csharp
        static async Task StartNodeAsync(FabricClient fc, string nodeName)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = n.NodeInstanceId;

            // Create a NodeStartDescription object, which will be used as a parameter into StartNodeTransition
            NodeStartDescription description = new NodeStartDescription(guid, n.NodeName, nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until hte desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**範例 3** - 下列範例示範錯誤用法。  這種用法不正確，是因為它提供的 stopDurationInSeconds 大於允許的範圍。  由於 StartNodeTransitionAsync() 將會失敗並發生嚴重錯誤，作業將不被接受，應該不會呼叫進度 API。  這個範例會使用第一個範例中的一些協助程式方法。

```csharp
        static async Task StopNodeWithOutOfRangeDurationAsync(FabricClient fc, string nodeName)
        {
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();

            // Use an out of range value for stopDurationInSeconds to demonstrate error
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, 99999);

            try
            {
                await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
            }

            catch (FabricException e)
            {
                Console.WriteLine("Caught {0}", e);
                Console.WriteLine("ErrorCode {0}", e.ErrorCode);
                // Output:
                // Caught System.Fabric.FabricException: System.Runtime.InteropServices.COMException (-2147017629)
                // StopDurationInSeconds is out of range ---> System.Runtime.InteropServices.COMException: Exception from HRESULT: 0x80071C63
                // << Parts of exception omitted>>
                //
                // ErrorCode InvalidDuration
            }
        }
```

**範例 4** - 下列範例示範當節點轉換 API 所起始的作業被接受，但在稍後執行失敗時，節點轉換進度 API 傳回的錯誤資訊。  在此狀況中，失敗是因為節點轉換 API 嘗試啟動不存在的節點。  這個範例會使用第一個範例中的一些協助程式方法。

```csharp
        static async Task StartNodeWithNonexistentNodeAsync(FabricClient fc)
        {
            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = 12345;

            // Intentionally use a nonexistent node
            NodeStartDescription description = new NodeStartDescription(guid, "NonexistentNode", nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.  In this case, it will end up in the Faulted state since the node does not exist.
            // When StartNodeTransitionProgressAsync()'s returned progress object has a State if Faulted, inspect the progress object's Result.Exception.HResult to get the error code.
            // In this case, it will be NodeNotFound.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Faulted).ConfigureAwait(false);
        }
```

[stopnode]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.faultmanagementclient?redirectedfrom=MSDN#System_Fabric_FabricClient_FaultManagementClient_StopNodeAsync_System_String_System_Numerics_BigInteger_System_Fabric_CompletionMode_
[stopnodeps]: https://msdn.microsoft.com/library/mt125982.aspx
[startnode]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.faultmanagementclient?redirectedfrom=MSDN#System_Fabric_FabricClient_FaultManagementClient_StartNodeAsync_System_String_System_Numerics_BigInteger_System_String_System_Int32_System_Fabric_CompletionMode_System_Threading_CancellationToken_
[startnodeps]: https://msdn.microsoft.com/library/mt163520.aspx
[nodequery]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient#System_Fabric_FabricClient_QueryClient_GetNodeListAsync_System_String_
[nodequeryps]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricnode
[snt]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.testmanagementclient#System_Fabric_FabricClient_TestManagementClient_StartNodeTransitionAsync_System_Fabric_Description_NodeTransitionDescription_System_TimeSpan_System_Threading_CancellationToken_
[gntp]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.testmanagementclient#System_Fabric_FabricClient_TestManagementClient_GetNodeTransitionProgressAsync_System_Guid_System_TimeSpan_System_Threading_CancellationToken_
