---
title: 模擬 Azure 微服務中的失敗 | Microsoft Docs
description: 本文說明關於在 Microsoft Azure Service Fabric 中找到的 Testability 動作。
services: service-fabric
documentationcenter: .net
author: motanv
manager: timlt
editor: heeldin
ms.assetid: ed53ca5c-4d5e-4b48-93c9-e386f32d8b7a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/07/2017
ms.author: motanv
ms.openlocfilehash: 27c6671c170f4c03c63270772651051830d8e4ec
ms.sourcegitcommit: 4f9fa86166b50e86cf089f31d85e16155b60559f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34757616"
---
# <a name="testability-actions"></a>Testability 動作
為了模擬不可靠的基礎結構，Azure Service Fabric 會提供開發人員用於模擬各種真實失敗案例及狀態轉換的方法。 這些方法會以 Testability 動作的形式公開。 這些動作是低階 API，會導致特定錯誤插入、狀態轉換或驗證。 結合這些動作後，便可以為您的服務撰寫完整的測試案例。

Service Fabric 會提供由這些動作所組成的一些常見測試案例。 強烈建議您使用這些內建案例，因為它們是經過仔細挑選，可用來測試常見狀態轉換和失敗案例的案例。 不過，若您想要新增案例涵蓋範圍以補足內建案例尚未涵蓋，或並非針對您的應用程式量身打造的案例時，動作就可用來建立自訂測試案例。

這些動作的 C# 實作可在 System.Fabric.dll 組件中找到。 System Fabric PowerShell 模組可在 Microsoft.ServiceFabric.Powershell.dll 組件中找到。 ServiceFabric PowerShell 模組會在執行階段安裝過程中安裝，讓您方便使用。

## <a name="graceful-vs-ungraceful-fault-actions"></a>非失誤性與失誤性錯誤動作比較
Testability 動作分為兩個主要貯體：

* 失誤性錯誤：這些錯誤會模擬失敗案例 (例如機器重新啟動和處理程序當機)。 在這類失敗情況下，處理程序的執行內容會突然停止。 這表示在應用程式再次啟動前，您都不能執行任何狀態清除作業。
* 非失誤性錯誤：這些錯誤會模擬正常動作 (例如由負載平衡觸發的複本移動及卸除)。 在這類情況下，服務會在結束前取得關閉通知並且清除狀態。

為了提供更好的品質驗證，請在引發各種非失誤性及失誤性錯誤時，執行服務及商務工作負載。 失誤性錯誤會模擬服務處理程序在部分工作流程中突然結束的案例。 當服務複本由 Service Fabric 還原時，便會測試復原路徑。 這有助於測試資料的一致性，以及服務狀態在失敗之後是否已正確維護。 其他錯誤集 (非失誤性錯誤) 會測試由 Service Fabric 移動的複本是否正確反應。 這會測試 RunAsync 方法中的取消處理作業。 該服務必須檢查已設定的取消語彙基元、正確地儲存其狀態，並結束 RunAsync 方法。

## <a name="testability-actions-list"></a>Testability 動作清單
| 動作 | 說明 | 受控 API | PowerShell Cmdlet | 非失誤性/失誤性錯誤 |
| --- | --- | --- | --- | --- |
| CleanTestState |會移除叢集的所有測試狀態，以防止測試驅動程式不正確關閉。 |CleanTestStateAsync |Remove-ServiceFabricTestState |不適用 |
| InvokeDataLoss |會引發資料遺失至服務分割區中。 |InvokeDataLossAsync |Invoke-ServiceFabricPartitionDataLoss |非失誤性 |
| InvokeQuorumLoss |會放置指定狀態服務分割區至仲裁遺失中。 |InvokeQuorumLossAsync |Invoke-ServiceFabricQuorumLoss |非失誤性 |
| MovePrimary |將指定的狀態服務主要複本移動至指定的叢集節點。 |MovePrimaryAsync |Move-ServiceFabricPrimaryReplica |非失誤性 |
| MoveSecondary |將目前的服務狀態次要複本移動至不同的叢集節點。 |MoveSecondaryAsync |Move-ServiceFabricSecondaryReplica |非失誤性 |
| RemoveReplica |移除叢集中的複本以模擬複本失敗案例。 這將會關閉複本，並將其轉換為 'None' 角色，進而移除其在叢集中的所有狀態。 |RemoveReplicaAsync |Remove-ServiceFabricReplica |非失誤性 |
| RestartDeployedCodePackage |藉由重新啟動部署在叢集中之節點上的程式碼封裝，模擬程式碼封裝處理程序的失敗案例。 這會中止程式碼封裝處理程序 (會重新啟動所有由其代管的使用者服務複本)。 |RestartDeployedCodePackageAsync |Restart-ServiceFabricDeployedCodePackage |失誤性 |
| RestartNode |藉由重新啟動節點，模擬 Service Fabric 叢集節點的失敗案例。 |RestartNodeAsync |Restart-ServiceFabricNode |失誤性 |
| RestartPartition |藉由重新啟動部分或所有分割區複本，模擬資料中心停機或叢集停機的案例。 |RestartPartitionAsync |Restart-ServiceFabricPartition |非失誤性 |
| RestartReplica |藉由重新啟動保存在叢集中的複本、關閉該複本再重新開啟的方式，模擬複本失敗案例。 |RestartReplicaAsync |Restart-ServiceFabricReplica |非失誤性 |
| StartNode |會啟動叢集中已停止的節點。 |StartNodeAsync |Start-ServiceFabricNode |不適用 |
| StopNode |藉由停止叢集中的節點，模擬節點失敗案例。 在呼叫 StartNode 前，節點會維持關閉。 |StopNodeAsync |Stop-ServiceFabricNode |失誤性 |
| ValidateApplication |會驗證應用程式中所有 Service Fabric 服務的可用性和健康情況 (通常在引發部分錯誤至系統後)。 |ValidateApplicationAsync |Test-ServiceFabricApplication |不適用 |
| ValidateService |會驗證 Service Fabric 服務的可用性和健康情況 (通常在引發部分錯誤至系統後)。 |ValidateServiceAsync |Test-ServiceFabricService |不適用 |

## <a name="running-a-testability-action-using-powershell"></a>使用 PowerShell 執行 Testability 動作
本教學課程會示範如何使用 PowerShell 執行 Testability 動作。 您將學習如何針對本機 (一整體) 叢集或 Azure 叢集，執行 Testability 動作。 當您安裝 Microsoft Service Fabric MSI 時，Microsoft.Fabric.Powershell.dll (Service Fabric PowerShell 模組) 就會自動安裝。 當您開啟 PowerShell 提示字元時，此模組就會自動載入。

教學課程區段：

* [針對一整體叢集執行動作](#run-an-action-against-a-one-box-cluster)
* [針對 Azure 叢集執行動作](#run-an-action-against-an-azure-cluster)

### <a name="run-an-action-against-a-one-box-cluster"></a>針對一整體叢集執行動作
若要針對本機叢集執行 Testability 動作，請先連接到叢集並在系統管理員模式下開啟 PowerShell 提示字元。 讓我們來看看 **Restart-ServiceFabricNode** 動作。

```powershell
Restart-ServiceFabricNode -NodeName Node1 -CompletionMode DoNotVerify
```

這裡的 **Restart-ServiceFabricNode** 動作正在名為 "Node1" 的節點上執行。 完成模式會指定該動作不需確認 restart-node 的動作是否確實成功。 指定完成模式為 "Verify" 將會讓其確認重新啟動動作是否確實成功。 您可以透過分割區索引鍵及複本種類指定節點名稱，而不是直接指定其名稱，如下所示：

```powershell
Restart-ServiceFabricNode -ReplicaKindPrimary  -PartitionKindNamed -PartitionKey Partition3 -CompletionMode Verify
```


```powershell
$connection = "localhost:19000"
$nodeName = "Node1"

Connect-ServiceFabricCluster $connection
Restart-ServiceFabricNode -NodeName $nodeName -CompletionMode DoNotVerify
```

**Restart-ServiceFabricNode** 應該用來重新啟動叢集中的 Service Fabric 節點。 這會停止 Fabric.exe 處理程序，此程序會重新啟動該節點代管的所有系統服務和使用者服務複本。 使用此 API 測試您的服務有助於發現容錯復原路徑中的錯誤。 它可協助模擬叢集中節點的失敗案例。

下列螢幕擷取畫面顯示 **Restart-ServiceFabricNode** Testability 命令的實際操作。

![](media/service-fabric-testability-actions/Restart-ServiceFabricNode.png)

第一個 **Get-ServiceFabricNode** (Service Fabric PowerShell 模組中的 Cmdlet) 的輸出顯示，本機叢集有五個節點：Node.1 至 Node.5。 在名為 Node.4 的節點上執行 **Restart-ServiceFabricNode** Testability 動作 (Cmdlet) 之後，我們可以看到節點的運作時間已重設。

### <a name="run-an-action-against-an-azure-cluster"></a>針對 Azure 叢集執行動作
針對 Azure 叢集執行 Testability 動作 (使用 PowerShell)，與針對本機叢集執行該動作的方式類似。 唯一的差異在於：執行該動作前，您必須先連接至 Azure 叢集，而非連接至本機叢集。

## <a name="running-a-testability-action-using-c35"></a>使用 C&#35; 執行 Testability 動作
若要使用 C# 執行 Testability 動作，您必須先使用 FabricClient 連接至叢集。 接著，取得執行該動作所需的參數。 不同的參數可用來執行相同的動作。
查看 RestartServiceFabricNode 動作時，您會發現執行該動作的一種方式就是，使用叢集中的節點資訊 (節點名稱和節點執行個體識別碼)。

```csharp
RestartNodeAsync(nodeName, nodeInstanceId, completeMode, operationTimeout, CancellationToken.None)
```

參數說明：

* **CompleteMode** 會指定該模式不需確認重新啟動動作是否確實成功。 指定完成模式為 "Verify" 將會讓其確認重新啟動動作是否確實成功。  
* **OperationTimeout** 會在擲回 TimeoutException 例外狀況前，設定完成作業所需的時間。
* **CancellationToken** 可讓擱置中的呼叫取消。

您可以透過分割區索引鍵及複本種類指定節點名稱，而不是直接指定其名稱。

如需詳細資訊，請參閱 [PartitionSelector 和 ReplicaSelector](#partition_replica_selector)。

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Fabric.Testability;
using System.Fabric;
using System.Threading;
using System.Numerics;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");
        string nodeName = "N0040";
        BigInteger nodeInstanceId = 130743013389060139;

        Console.WriteLine("Starting RestartNode test");
        try
        {
            //Restart the node by using ReplicaSelector
            RestartNodeAsync(clusterConnection, serviceName).Wait();

            //Another way to restart node is by using nodeName and nodeInstanceId
            RestartNodeAsync(clusterConnection, nodeName, nodeInstanceId).Wait();
        }
        catch (AggregateException exAgg)
        {
            Console.WriteLine("RestartNode did not complete: ");
            foreach (Exception ex in exAgg.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("RestartNode completed.");
        return 0;
    }

    static async Task RestartNodeAsync(string clusterConnection, Uri serviceName)
    {
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);
        ReplicaSelector primaryofReplicaSelector = ReplicaSelector.PrimaryOf(randomPartitionSelector);

        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(primaryofReplicaSelector, CompletionMode.Verify);
    }

    static async Task RestartNodeAsync(string clusterConnection, string nodeName, BigInteger nodeInstanceId)
    {
        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(nodeName, nodeInstanceId, CompletionMode.Verify);
    }
}
```

## <a name="partitionselector-and-replicaselector"></a>PartitionSelector 和 ReplicaSelector
### <a name="partitionselector"></a>PartitionSelector
PartitionSelector 是 Testability 中公開的協助程式，可用來選取特定分割區 (可在其中執行任何 Testability 動作)。 如果已事先知道分割區識別碼，就能將其用於選取特定分割區。 或者，您可以提供分割區索引鍵，作業就會在內部解析分割區識別碼。 您也可以選擇選取隨機分割區。

若要使用此協助程式，請建立 PartitionSelector 物件並使用其中一種 Select* 方法來選取分割區。 接著將 PartitionSelector 物件傳遞至有需求的 API。 如果不選取任何選項，它會預設為隨機分割區。

```csharp
Uri serviceName = new Uri("fabric:/samples/InMemoryToDoListApp/InMemoryToDoListService");
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
string partitionName = "Partition1";
Int64 partitionKeyUniformInt64 = 1;

// Select a random partition
PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

// Select a partition based on ID
PartitionSelector partitionSelectorById = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);

// Select a partition based on name
PartitionSelector namedPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionName);

// Select a partition based on partition key
PartitionSelector uniformIntPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionKeyUniformInt64);
```

### <a name="replicaselector"></a>ReplicaSelector
PartitionSelector 是 Testability 中公開的協助程式，可用來協助選取複本 (可在其中執行任何 Testability 動作)。 如果已事先知道複本識別碼，就能將其用於選取特定複本。 此外，您也可以選取主要複本或隨機次要複本。 ReplicaSelector 衍生自 PartitionSelector，因此您必須同時選取複本和分割區，以便在其中執行您想要的 Testability 作業。

若要使用此協助程式，請建立 ReplicatorSelector 物件，並設定您想選取複本和分割區的方式。 接著可以將其傳遞至有需求的 API。 如果不選取任何選項，它會預設為隨機複本和隨機分割區。

```csharp
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);
long replicaId = 130559876481875498;

// Select a random replica
ReplicaSelector randomReplicaSelector = ReplicaSelector.RandomOf(partitionSelector);

// Select the primary replica
ReplicaSelector primaryReplicaSelector = ReplicaSelector.PrimaryOf(partitionSelector);

// Select the replica by ID
ReplicaSelector replicaByIdSelector = ReplicaSelector.ReplicaIdOf(partitionSelector, replicaId);

// Select a random secondary replica
ReplicaSelector secondaryReplicaSelector = ReplicaSelector.RandomSecondaryOf(partitionSelector);
```

## <a name="next-steps"></a>後續步驟
* [Testability 案例](service-fabric-testability-scenarios.md)
* 如何測試您的服務
  * [模擬服務工作負載期間的失敗案例](service-fabric-testability-workload-tests.md)
  * [服務對服務間通訊的失敗案例](service-fabric-testability-scenarios-service-communication.md)

