---
title: 計算作業和節點狀態的數目 - Azure Batch | Microsoft Docs
description: 計算 Azure Batch 作業和計算節點狀態的數目，以利管理及監視 Batch 解決方案。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.topic: article
ms.date: 09/07/2018
ms.author: danlep
ms.openlocfilehash: e1d6f2d6181e70fde75907191664dcf6cd0b7252
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391754"
---
# <a name="monitor-batch-solutions-by-counting-tasks-and-nodes-by-state"></a>依狀態計算作業和節點數目以監視 Batch 解決方案

若要監視及管理大規模的 Azure Batch 解決方案，處於各種狀態的資源必須要有精準的計數。 Azure Batch 提供有效的作業，可取得 Batch *作業*和*計算節點*的這些計數。 請使用這些作業，而非可能很耗時的清單查詢 (會傳回大型工作或節點集合的相關詳細資訊)。

* [取得作業計數][rest_get_task_counts]可取得作業中的作用中、執行中和已完成工作的彙總計數，以及成功或失敗的工作計數。 

  藉由計算處於各個狀態的工作數目，您可以更輕鬆地向使用者顯示作業進度，或偵測可能會影響作業的非預期延遲或失敗。 「取得工作計數」適用於 Batch 服務 API 2017-06-01.5.1 起的版本，以及相關 SDK 和工具。

* [列出集區節點計數][rest_get_node_counts]可取得每個集區中處於各種狀態的專用和低優先順序計算節點的數目：建立中、閒置、離線、先佔、重新開機中、重新安裝映像中、啟動中和其他狀態。 

  藉由計算各個狀態的節點數目，您將可判斷是否有足夠的計算資源可執行作業，並找出集區的潛在問題。 「列出集區節點計數」適用於 Batch 服務 API 2018-03-01.6.1 起的版本，以及相關 SDK 和工具。

如果您使用不支援工作計數或節點計數作業的服務版本，請改用清單查詢來計算這些資源的數目。 此外，也請使用清單查詢來取得其他 Batch 資源 (例如應用程式、檔案和作業) 的相關資訊。 如需將篩選條件套用至清單查詢的詳細資訊，請參閱[建立查詢以便有效率地列出 Batch 資源](batch-efficient-list-queries.md)。

## <a name="task-state-counts"></a>工作狀態計數

「取得工作計數」作業會依下列狀態計算工作數目：

- **作用中** - 已排入佇列且能夠執行、但目前尚未指派給計算節點的工作。 如果工作[相依於尚未完成的父工作](batch-task-dependencies.md)，也會計為 `active`。 
- **執行中** - 已指派給計算節點、但尚未完成的工作。 處於 `preparing` 或 `running` 狀態的工作會計為 `running`，如[取得工作相關資訊][rest_get_task]作業所示。
- **已完成** - 因已順利完成，或未順利完成且已達到其重試限制，而不再符合執行條件的工作。 
- **成功** - 工作執行結果為 `success` 的工作。 Batch 會藉由檢查 [executionInfo][rest_get_exec_info] 屬性的 `TaskExecutionResult` 屬性來判斷工作是否成功。
- **失敗** - 工作執行結果為 `failure` 的工作。

下列 .NET 程式碼範例示範如何依狀態擷取工作計數： 

```csharp
var taskCounts = await batchClient.JobOperations.GetTaskCountsAsync("job-1");

Console.WriteLine("Task count in active state: {0}", taskCounts.Active);
Console.WriteLine("Task count in preparing or running state: {0}", taskCounts.Running);
Console.WriteLine("Task count in completed state: {0}", taskCounts.Completed);
Console.WriteLine("Succeeded task count: {0}", taskCounts.Succeeded);
Console.WriteLine("Failed task count: {0}", taskCounts.Failed);
```

您可以使用類似 REST 的模式和其他支援的語言來取得作業的工作計數。 

> [!NOTE]
> 2018-08-01.7.0 之前的 Batch 服務 API 版本也會在「取得工作計數」回應中傳回 `validationStatus` 屬性。 這個屬性會指出 Batch 是否已檢查狀態計數有沒有與「列出工作 API」所回報的狀態保持一致。 值為 `validated` 只表示 Batch 已至少對作業是否一致檢查過一次。 `validationStatus` 屬性的值不會指出「取得工作計數」所傳回的計數是否為最新的。
>

## <a name="node-state-counts"></a>節點狀態計數

「列出集區節點計數」作業會依據每個集區中的下列狀態為計算節點計數。 對於每個集區中的專用節點和低優先順序節點，會提供個別的彙總計數。

- **建立中** - 由 Azure 配置、尚未開始加入集區的 VM。
- **閒置** - 目前未執行工作的可用計算節點。
- **LeavingPool** - 因使用者明確加以移除，或是因集區調整大小或自動調整已關閉而退出集區的節點。
- **離線** - Batch 無法用來排程新工作的節點。
- **先佔** - 因 Azure 回收 VM 而已從集區中移除的低優先順序節點。 在可替換低優先順序 VM 容量時，可以重新初始化節點 `preempted`。
- **重新開機中** - 正在重新啟動的節點。
- **重新安裝映像中** - 正在重新安裝作業系統的節點。
- **執行中** - 正在執行一或多個工作 (非啟動工作) 的節點。
- **啟動中** - 正在啟動 Batch 服務的節點。 
- **StartTaskFailed** - [啟動工作][rest_start_task]失敗並且已耗盡所有重試次數，且其啟動工作設定了 `waitForSuccess` 的節點。 此類節點無法用來執行工作。
- **不明** - 失去與 Batch 服務的聯繫、且狀態不明的節點。
- **無法使用** - 因發生錯誤而無法用來執行工作的節點。
- **WaitingForStartTask** - 啟動工作已開始執行、但設定了 `waitForSuccess`且啟動工作未完成的節點。

下列 C# 程式碼片段說明如何列出目前帳戶中所有集區的節點計數：

```csharp
foreach (var nodeCounts in batchClient.PoolOperations.ListPoolNodeCounts())
{
    Console.WriteLine("Pool Id: {0}", nodeCounts.PoolId);

    Console.WriteLine("Total dedicated node count: {0}", nodeCounts.Dedicated.Total);

    // Get dedicated node counts in Idle and Offline states; you can get additional states.
    Console.WriteLine("Dedicated node count in Idle state: {0}", nodeCounts.Dedicated.Idle);
    Console.WriteLine("Dedicated node count in Offline state: {0}", nodeCounts.Dedicated.Offline);

    Console.WriteLine("Total low priority node count: {0}", nodeCounts.LowPriority.Total);

    // Get low-priority node counts in Running and Preempted states; you can get additional states.
    Console.WriteLine("Low-priority node count in Running state: {0}", nodeCounts.LowPriority.Running);
    Console.WriteLine("Low-priority node count in Preempted state: {0}", nodeCounts.LowPriority.Preempted);
}
```
下列 C# 程式碼片段說明如何列出目前帳戶中給定集區的節點計數。

```csharp
foreach (var nodeCounts in batchClient.PoolOperations.ListPoolNodeCounts(new ODATADetailLevel(filterClause: "poolId eq 'testpool'")))
{
    Console.WriteLine("Pool Id: {0}", nodeCounts.PoolId);

    Console.WriteLine("Total dedicated node count: {0}", nodeCounts.Dedicated.Total);

    // Get dedicated node counts in Idle and Offline states; you can get additional states.
    Console.WriteLine("Dedicated node count in Idle state: {0}", nodeCounts.Dedicated.Idle);
    Console.WriteLine("Dedicated node count in Offline state: {0}", nodeCounts.Dedicated.Offline);

    Console.WriteLine("Total low priority node count: {0}", nodeCounts.LowPriority.Total);

    // Get low-priority node counts in Running and Preempted states; you can get additional states.
    Console.WriteLine("Low-priority node count in Running state: {0}", nodeCounts.LowPriority.Running);
    Console.WriteLine("Low-priority node count in Preempted state: {0}", nodeCounts.LowPriority.Preempted);
}
```
您可以將類似的模式用於 REST 和其他支援的語言，以取得集區的節點計數。
 
## <a name="next-steps"></a>後續步驟

* 若要深入了解 Batch 服務概念和功能，請參閱 [Batch 功能概觀](batch-api-basics.md) 。 本文討論主要 Batch 資源 (例如集區、計算節點、作業和工作)，並提供服務功能的概觀。

* 如需對列出 Batch 資源的查詢套用篩選條件的相關資訊，請參閱[建立查詢以便有效率地列出 Batch 資源](batch-efficient-list-queries.md)。


[rest_get_task_counts]: /rest/api/batchservice/job/gettaskcounts
[rest_get_node_counts]: /rest/api/batchservice/account/listpoolnodecounts
[rest_get_task]: /rest/api/batchservice/task/get
[rest_list_tasks]: /rest/api/batchservice/task/list
[rest_get_exec_info]: /rest/api/batchservice/task/get#executionInfo
[rest_start_task]: /rest/api/batchservice/pool/add#starttask

