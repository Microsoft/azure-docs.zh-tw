---
title: 設計有效率的清單查詢 - Azure Batch | Microsoft Docs
description: 藉由在要求集區、作業、工作和計算節點等 Batch 資源的資訊時篩選查詢，以提高效能。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 031fefeb-248e-4d5a-9bc2-f07e46ddd30d
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 06/26/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6bc31e8541797930583e41fb6efbb6473cd4b894
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39004450"
---
# <a name="create-queries-to-list-batch-resources-efficiently"></a>建立查詢以便有效率地列出 Batch 資源

在此您將了解如何透過減少在使用 [Batch .NET][api_net] 程式庫查詢作業、工作、計算節點和其他資源時所傳回的資料量，增加 Azure Batch 應用程式的效能。

幾乎所有 Batch 應用程式都必須執行某種類型的監視或其他查詢 Batch 服務的作業，並且通常是定期執行。 例如，若要判斷作業中是否有任何剩餘的已排入佇列的工作，您必須取得作業內每項工作的資料。 若要判斷集區中節點的狀態，您必須取得集區中每個節點的資料。 這篇文章說明如何以最有效率的方式執行這類查詢。

> [!NOTE]
> Batch 服務會針對計算作業中的工作，以及計算 Batch 集區中的計算節點這類常見案例，提供特殊的 API 支援。 您可以呼叫[取得工作計數][rest_get_task_counts]和[列出集區節點計數][rest_get_node_counts] 作業，而非使用清單查詢。 這些作業比清單查詢更有效率，但會傳回更多限制的資訊。 請參閱[依照狀態計算工作和計算節點](batch-get-resource-counts.md)。 


## <a name="meet-the-detaillevel"></a>認識 DetailLevel
在生產用 Batch 應用程式中，作業、工作和計算節點等實體的數量可能有數千個。 當您要求這些資源的資訊時，可能會有大量資料必須從 Batch 服務「傳送」到每個查詢上的應用程式。 透過限制項目數量及查詢所傳回的資訊類型，您可以加速查詢，因而提高應用程式的效能。

此 [Batch .NET][api_net] API 程式碼片段會列出與作業相關聯的「每一項」工作，以及各工作的「所有」屬性：

```csharp
// Get a collection of all of the tasks and all of their properties for job-001
IPagedEnumerable<CloudTask> allTasks =
    batchClient.JobOperations.ListTasks("job-001");
```

不過，您可以藉由對查詢套用「詳細資料層級」，以進行更有效率的清單查詢。 在 [JobOperations.ListTasks][net_list_tasks] 方法中提供 [ODATADetailLevel][odata] 物件即可執行此動作。 此程式碼片段只是傳回已完成之工作的識別碼、命令列和計算節點資訊屬性：

```csharp
// Configure an ODATADetailLevel specifying a subset of tasks and
// their properties to return
ODATADetailLevel detailLevel = new ODATADetailLevel();
detailLevel.FilterClause = "state eq 'completed'";
detailLevel.SelectClause = "id,commandLine,nodeInfo";

// Supply the ODATADetailLevel to the ListTasks method
IPagedEnumerable<CloudTask> completedTasks =
    batchClient.JobOperations.ListTasks("job-001", detailLevel);
```

在此範例案例中，如果作業中有數千個工作，則第二次查詢傳回結果的速度，通常會比第一次快很多。 使用 Batch .NET API 列出項目時，使用 ODATADetailLevel 的詳細資訊如 [下](#efficient-querying-in-batch-net)所示。

> [!IMPORTANT]
> 我們強烈建議「一律」在 .NET API 清單呼叫中提供 ODATADetailLevel 物件，以確保應用程式發揮最高效率和效能。 透過指定詳細層級，您可以幫助縮短 Batch 服務回應時間、提高網路使用率，以及讓用戶端應用程式的記憶體使用量降到最低。
> 
> 

## <a name="filter-select-and-expand"></a>篩選、選取及展開
[Batch .NET][api_net] 和 [Batch REST][api_rest] API 可讓您減少清單中傳回的項目數和每個項目傳回的資訊量。 您可以藉由在執行清單查詢時指定**篩選**、**選取**和**展開字串**來執行此動作。

### <a name="filter"></a>Filter
篩選字串是可減少傳回的項目數的運算式。 例如，只列出工作正在執行的作業，或只列出可執行作業的計算節點。

* 篩選字串包含一個或多個運算式，而運算式由屬性名稱、運算子和值構成。 可指定的屬性及每個屬性支援的運算子，取決於您查詢的每個實體類型。
* 多個運算式可以透過邏輯運算子 `and` 和 `or` 結合。
* 此範例篩選字串只會列出執行中「轉譯」工作： `(state eq 'running') and startswith(id, 'renderTask')`。

### <a name="select"></a>選取
選取字串限制每個項目傳回的屬性值。 指定屬性名稱的清單，而且查詢結果中只有針對項目傳回的那些屬性值。

* 選取字串由屬性名稱的逗號分隔清單組成。 您可以針對查詢的實體類型指定任何屬性。
* 此範例選取字串會指定應該針對每個工作只傳回三個屬性： `id, state, stateTransitionTime`。

### <a name="expand"></a>展開
展開字串減少取得某些資訊所需的 API 呼叫次數。 當您使用展開字串時，可以透過單一 API 呼叫取得每個項目的相關詳細資訊。 不是首先取得實體的清單，然後在清單中要求每個項目的資訊，而是您可以使用展開字串在單一 API 呼叫中取得相同資訊。 較少的 API 呼叫表示較佳的效能。

* 與選取字串相似，展開字串可以控制清單查詢結果中是否包含特定資料。
* 只有在用於列出作業、作業排程、作業和集區時，才支援展開字串。 目前，它僅支援統計資訊。
* 當需要所有屬性但未指定選取字串時， *必須* 使用展開字串來取得統計資料資訊。 如果使用選取字串來取得屬性子集，則可以在選取字串中指定 `stats` ，不需要指定展開字串。
* 此範例展開字串指定應該在清單中傳回每個項目的統計資料資訊： `stats`。

> [!NOTE]
> 建構任何這三種查詢字串類型時 (篩選、選取和展開)，您必須確定屬性名稱和大小寫符合其對應的 Batch REST API 元素。 例如，使用 .NET [CloudTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask) 類別時，即使 .NET 屬性是 [CloudTask.State](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.state)，也必須指定 **state** 而不是 **State**。 關於 .NET 和 REST API 之間的屬性對應，請參閱下表。
> 
> 

### <a name="rules-for-filter-select-and-expand-strings"></a>篩選、選取和展開字串的規則
* 篩選、選取和展開字串中的屬性名稱，應該和 [Batch REST][api_rest] API 中的屬性名稱一致，就算是使用 [Batch .NET][api_net] 或其他 Batch SDK 的其中一個也是如此。
* 所有屬性名稱都會區分大小寫，但屬性值不會區分大小寫。
* 日期/時間字串有兩種格式，開頭必須加上 `DateTime`。
  
  * W3C-DTF 格式範例：`creationTime gt DateTime'2011-05-08T08:49:37Z'`
  * RFC 1123 格式範例：`creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`
* 布林值字串為 `true` 或 `false`。
* 如果指定無效的屬性或運算子，將會導致 `400 (Bad Request)` 錯誤。

## <a name="efficient-querying-in-batch-net"></a>在 Batch .NET 中有效率地查詢
在 [Batch .NET][api_net] API 內，[ODATADetailLevel][odata] 類別用來提供篩選、選取和展開字串給清單作業。 ODataDetailLevel 物件有三個公用字串屬性，可以在建構函式中指定或是直接在物件上設定。 然後您可以將 ODataDetailLevel 物件當做參數傳給各種清單作業，例如 [ListPools][net_list_pools]、[ListJobs][net_list_jobs] 和 [ListTasks][net_list_tasks]。

* [ODATADetailLevel][odata].[FilterClause][odata_filter]：限制傳回的項目數。
* [ODATADetailLevel][odata].[SelectClause][odata_select]：指定隨著每個項目一起傳回的屬性值。
* [ODATADetailLevel][odata].[ExpandClause][odata_expand]：在單一 API 呼叫中擷取所有項目的資料，而不是針對每個項目個別呼叫。

下列程式碼片段使用 Batch .NET API，有效率地向 Batch 服務查詢一組特定集區的統計資料。 在此案例中，Batch 使用者具有測試與生產的集區。 這些測試集區識別碼前面會加上 "test"，而生產集區識別碼則會加上 "prod"。 在程式碼片段中，「myBatchClient」  是適當初始化的 [BatchClient](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient) 類別的執行個體。

```csharp
// First we need an ODATADetailLevel instance on which to set the filter, select,
// and expand clause strings
ODATADetailLevel detailLevel = new ODATADetailLevel();

// We want to pull only the "test" pools, so we limit the number of items returned
// by using a FilterClause and specifying that the pool IDs must start with "test"
detailLevel.FilterClause = "startswith(id, 'test')";

// To further limit the data that crosses the wire, configure the SelectClause to
// limit the properties that are returned on each CloudPool object to only
// CloudPool.Id and CloudPool.Statistics
detailLevel.SelectClause = "id, stats";

// Specify the ExpandClause so that the .NET API pulls the statistics for the
// CloudPools in a single underlying REST API call. Note that we use the pool's
// REST API element name "stats" here as opposed to "Statistics" as it appears in
// the .NET API (CloudPool.Statistics)
detailLevel.ExpandClause = "stats";

// Now get our collection of pools, minimizing the amount of data that is returned
// by specifying the detail level that we configured above
List<CloudPool> testPools =
    await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();
```

> [!TIP]
> 使用「選取」和「展開」子句設定的 [ODATADetailLevel][odata] 執行個體，也可以傳給適當的 Get 方法，例如 [PoolOperations.GetPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getpool.aspx)，以限制傳回的資料量。
> 
> 

## <a name="batch-rest-to-net-api-mappings"></a>Batch REST 與 .NET API 的對應
篩選、選取和展開字串中的屬性名稱在名稱和大小寫方面， *必須* 反映其 REST API 對應項目。 下表提供 .NET 和 REST API 對應項目之間的對應。

### <a name="mappings-for-filter-strings"></a>篩選字串的對應
* **.NET 清單方法**：此欄的每個 .NET API 方法都接受 [ODATADetailLevel][odata] 物件作為參數。
* **REST 清單要求**：此資料行的每個 REST API 頁面都連結至一個資料表，其中指定「篩選」字串中允許的屬性和作業。 建構 [ODATADetailLevel.FilterClause][odata_filter] 字串時會使用這些屬性名稱和作業。

| .NET 清單方法 | REST 清單要求 |
| --- | --- |
| [CertificateOperations.ListCertificates][net_list_certs] |[列出帳戶中的憑證][rest_list_certs] |
| [CloudTask.ListNodeFiles][net_list_task_files] |[列出與作業相關聯的檔案][rest_list_task_files] |
| [JobOperations.ListJobPreparationAndReleaseTaskStatus][net_list_jobprep_status] |[列出作業準備和工作解除作業的狀態][rest_list_jobprep_status] |
| [JobOperations.ListJobs][net_list_jobs] |[列出帳戶中的作業][rest_list_jobs] |
| [JobOperations.ListNodeFiles][net_list_nodefiles] |[列出節點上的檔案][rest_list_nodefiles] |
| [JobOperations.ListTasks][net_list_tasks] |[列出與工作相關聯的作業][rest_list_tasks] |
| [JobScheduleOperations.ListJobSchedules][net_list_job_schedules] |[列出帳戶中的作業排程][rest_list_job_schedules] |
| [JobScheduleOperations.ListJobs][net_list_schedule_jobs] |[列出與作業排程相關聯的作業][rest_list_schedule_jobs] |
| [PoolOperations.ListComputeNodes][net_list_compute_nodes] |[列出集區中的運算節點][rest_list_compute_nodes] |
| [PoolOperations.ListPools][net_list_pools] |[列出帳戶中的集區][rest_list_pools] |

### <a name="mappings-for-select-strings"></a>選取字串的對應
* **Batch .NET types**：Batch .NET API 類型。
* **REST API entities**：此資料行中的每個頁面包含一個或多個資料表，其中列出類型的 REST API 屬性名稱。 建構「選取」  字串時會使用這些屬性名稱。 建構 [ODATADetailLevel.SelectClause][odata_select] 字串時會使用這些相同的屬性名稱。

| Batch .NET types | REST API entities |
| --- | --- |
| [憑證][net_cert] |[取得憑證的相關資訊][rest_get_cert] |
| [CloudJob][net_job] |[取得作業的相關資訊][rest_get_job] |
| [CloudJobSchedule][net_schedule] |[取得作業排程的相關資訊][rest_get_schedule] |
| [ComputeNode][net_node] |[取得節點的相關資訊][rest_get_node] |
| [CloudPool][net_pool] |[取得集區的相關資訊][rest_get_pool] |
| [CloudTask][net_task] |[取得作業的相關資訊][rest_get_task] |

## <a name="example-construct-a-filter-string"></a>範例：建構篩選字串
建構 [ODATADetailLevel.FilterClause][odata_filter] 的篩選字串時，請參閱「篩選字串的對應」下方的資料表，找出與您想要執行的清單作業相對應的 REST API 文件頁面。 在該頁面的第一個含有多資料列的資料表中，您可以找到可篩選的屬性及其支援的運算子。 假設您想要擷取結束碼不為零的所有作業，[與作業相關聯的清單作業][rest_list_tasks]中的這一列指定適用的屬性字串和允許的運算子：

| 屬性 | 允許的作業 | 類型 |
|:--- |:--- |:--- |
| `executionInfo/exitCode` |`eq, ge, gt, le , lt` |`Int` |

因此，為了列出具有非零結束碼的所有作業，篩選字串如下：

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

## <a name="example-construct-a-select-string"></a>範例：建構選取字串
如果要建構 [ODATADetailLevel.SelectClause][odata_select]，請參閱「選取字串的對應」下方的資料表，瀏覽至與您要列出的實體類型相對應的 REST API 頁面。 在該頁面的第一個含有多資料列的資料表中，您可以找到可選取的屬性及其支援的運算子。 假設您只想要擷取清單中每個作業的識別碼和命令列，您可以在[取得作業的相關資訊][rest_get_task]中的適當資料表找到這幾列：

| 屬性 | 類型 | 注意 |
|:--- |:--- |:--- |
| `id` |`String` |`The ID of the task.` |
| `commandLine` |`String` |`The command line of the task.` |

為了讓每個列出的作業只包含識別碼和命令列，選取字串如下列：

`id, commandLine`

## <a name="code-samples"></a>程式碼範例
### <a name="efficient-list-queries-code-sample"></a>有效率的清單查詢程式碼範例
請查閱 GitHub 上的 [EfficientListQueries][efficient_query_sample] 範例專案，以了解有效率的清單查詢可如何提升應用程式的效能。 這個 C# 主控台應用程式會建立並將大量工作加入至作業。 然後，它會對 [JobOperations.ListTasks][net_list_tasks] 方法進行多個呼叫，並且傳遞設定了不同屬性值的 [ODATADetailLevel][odata] 物件，來變更要傳回的資料量。 它會產生類似下列的輸出：

```
Adding 5000 tasks to job jobEffQuery...
5000 tasks added in 00:00:47.3467587, hit ENTER to query tasks...

4943 tasks retrieved in 00:00:04.3408081 (ExpandClause:  | FilterClause: state eq 'active' | SelectClause: id,state)
0 tasks retrieved in 00:00:00.2662920 (ExpandClause:  | FilterClause: state eq 'running' | SelectClause: id,state)
59 tasks retrieved in 00:00:00.3337760 (ExpandClause:  | FilterClause: state eq 'completed' | SelectClause: id,state)
5000 tasks retrieved in 00:00:04.1429881 (ExpandClause:  | FilterClause:  | SelectClause: id,state)
5000 tasks retrieved in 00:00:15.1016127 (ExpandClause:  | FilterClause:  | SelectClause: id,state,environmentSettings)
5000 tasks retrieved in 00:00:17.0548145 (ExpandClause: stats | FilterClause:  | SelectClause: )

Sample complete, hit ENTER to continue...
```

如同經過的時間所顯示的，您可以透過限制屬性和傳回的項目數目來大幅降低查詢回應時間。 您可以在 GitHub 上的 [azure-batch-samples][github_samples] 儲存機制，找到本範例專案和其他範例專案。

### <a name="batchmetrics-library-and-code-sample"></a>BatchMetrics 程式庫和程式碼範例
除了上述的 EfficientListQueries 程式碼範例，您還可以在 [azure-batch-samples][github_samples] GitHub 儲存機制中找到 [BatchMetrics][batch_metrics] 專案。 BatchMetrics 範例專案會示範如何使用 Batch API 有效率地監視 Azure Batch 作業進度。

[BatchMetrics][batch_metrics] 範例包含可併入自己專案的 .NET 類別庫專案，以及用來運作和示範如何使用程式庫的簡單命令列程式。

專案內的範例應用程式會示範下列作業：

1. 選取特定屬性以便只下載您需要的屬性
2. 篩選狀態轉換時間以便只下載上次查詢之後的變更

例如，下列方法會出現在 BatchMetrics 程式庫。 它會傳回 ODATADetailLevel，指出只應該取得所查詢實體的 `id` 和 `state` 屬性。 它也會指出只應傳回自指定的 `DateTime` 參數之後其狀態已變更的實體。

```csharp
internal static ODATADetailLevel OnlyChangedAfter(DateTime time)
{
    return new ODATADetailLevel(
        selectClause: "id, state",
        filterClause: string.Format("stateTransitionTime gt DateTime'{0:o}'", time)
    );
}
```

## <a name="next-steps"></a>後續步驟
### <a name="parallel-node-tasks"></a>平行節點工作
[使用並行節點工作最大化 Azure Batch 計算資源使用量](batch-parallel-node-tasks.md) 是另一篇與 Batch 應用程式效能有關的文章。 某些類型的工作負載可以受益於在規模較大但數量較少的計算節點上執行平行工作。 如需這類案例的詳細資訊，請查看 [範例案例](batch-parallel-node-tasks.md#example-scenario) 。


[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_metrics]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchMetrics
[efficient_query_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries
[github_samples]: https://github.com/Azure/azure-batch-samples
[odata]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.aspx
[odata_ctor]: https://msdn.microsoft.com/library/azure/dn866178.aspx
[odata_expand]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.expandclause.aspx
[odata_filter]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.filterclause.aspx
[odata_select]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.selectclause.aspx

[net_list_certs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx
[net_list_compute_nodes]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listcomputenodes.aspx
[net_list_job_schedules]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobschedules.aspx
[net_list_jobprep_status]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobpreparationandreleasetaskstatus.aspx
[net_list_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[net_list_nodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listnodefiles.aspx
[net_list_pools]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx
[net_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobs.aspx
[net_list_task_files]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[net_list_tasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx

[rest_list_certs]: https://msdn.microsoft.com/library/azure/dn820154.aspx
[rest_list_compute_nodes]: https://msdn.microsoft.com/library/azure/dn820159.aspx
[rest_list_job_schedules]: https://msdn.microsoft.com/library/azure/mt282174.aspx
[rest_list_jobprep_status]: https://msdn.microsoft.com/library/azure/mt282170.aspx
[rest_list_jobs]: https://msdn.microsoft.com/library/azure/dn820117.aspx
[rest_list_nodefiles]: https://msdn.microsoft.com/library/azure/dn820151.aspx
[rest_list_pools]: https://msdn.microsoft.com/library/azure/dn820101.aspx
[rest_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/mt282169.aspx
[rest_list_task_files]: https://msdn.microsoft.com/library/azure/dn820142.aspx
[rest_list_tasks]: https://msdn.microsoft.com/library/azure/dn820187.aspx

[rest_get_cert]: https://msdn.microsoft.com/library/azure/dn820176.aspx
[rest_get_job]: https://msdn.microsoft.com/library/azure/dn820106.aspx
[rest_get_node]: https://msdn.microsoft.com/library/azure/dn820168.aspx
[rest_get_pool]: https://msdn.microsoft.com/library/azure/dn820165.aspx
[rest_get_schedule]: https://msdn.microsoft.com/library/azure/mt282171.aspx
[rest_get_task]: https://msdn.microsoft.com/library/azure/dn820133.aspx

[net_cert]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificate.aspx
[net_job]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_schedule]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjobschedule.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx

[rest_get_task_counts]: /rest/api/batchservice/get-the-task-counts-for-a-job
[rest_get_node_counts]: /rest/api/batchservice/account/listpoolnodecounts