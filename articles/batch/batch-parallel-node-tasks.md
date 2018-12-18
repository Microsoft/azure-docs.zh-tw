---
title: 平行執行工作以便有效率地使用計算資源 - Azure Batch | Microsoft Docs
description: 在 Azure Batch 集區中的每個節點上執行並行工作時，使用較少的運算節點以增加效率和降低成本
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 538a067c-1f6e-44eb-a92b-8d51c33d3e1a
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 05/22/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c52c9fc6b47b03b3ca6db96decb8b4777577d00e
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39174239"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>並行執行工作以充分使用 Batch 計算節點 

藉由在 Azure Batch 集區的每個計算模式同時執行一個以上的工作，您可以用較少的集中區節點將資源使用量最大化。 對於某些工作負載來說，這會產生縮短作業時間和降低成本的效益。

雖然某些案例受益於將節點的所有資源配置給單一工作，但是許多案例受益於允許多個工作共用這些資源：

* **最小化資料傳輸** 。 在此案例中，您可以將共用資料複製到較少數量的節點，並在每個節點上平行執行工作，以大幅降低資料傳輸費用。 此做法尤其適用於當要複製到每個節點的資料必須在地理區域之間傳輸時。
* **最大化記憶體使用量** ，但是只有在短時間之內，以及在執行期間的變數時間。 您可以運用具有更多記憶體的較少但較大的計算節點，有效率地處理這類高峰流量。 這些節點會有在每個節點上執行的平行工作，但是每個工作會在不同的時間利用節點的大量記憶體。
* **緩和節點數目限制** 。 目前，針對節點間通訊設定的集區限制為 50 個計算節點。 如果這類集區中的每個節點能夠以平行方式執行工作，則可以同時執行較多的工作。
* **複寫內部部署計算叢集**，例如當您第一次將計算環境移至 Azure 時。 如果目前的內部部署解決方案在每個計算節點執行多個工作，您可以增加節點工作的數目上限，以更密切地反映該組態。

## <a name="example-scenario"></a>範例案例
為了舉例說明平行工作執行的優點，讓我們假設您的工作應用程式 CPU 和記憶體需求的 [Standard\_D1](../cloud-services/cloud-services-sizes-specs.md) 節點大小是足夠的。 但是為了在要求的時間內完成作業，需要 1000 個這類節點。

如果不使用 Standard\_D1 節點 (具有 1 個 CPU 核心)，您可以採用具有 16 個核心的 [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) 節點，並啟用平行工作執行。 因此，不需使用 1000 個節點，而只需 63 個節點，用量「少了16 倍」  。 此外，由於資料只需複製到 63 個節點，如果每個節點都需要大型應用程式檔案或參考資料，那麼作業持續時間和效率都能再獲得改善。

## <a name="enable-parallel-task-execution"></a>啟用平行工作執行
您可以針對集區層級的平行工作執行，設定計算節點。 使用 Batch .NET 程式庫時，請在建立集區時設定 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 屬性。 如果您使用 Batch REST API，請在集區建立期間於要求本文中設定 [maxTasksPerNode][rest_addpool] 元素。

Azure Batch 允許您將每個節點的最大工作數目設定為多達節點核心數目的 4 倍 (4x)。 例如，如果集區設定的節點大小為 [大] \(四個核心)，則 `maxTasksPerNode` 可以設定為 16。 如需每個節點大小的核心數目的詳細資料，請參閱 [雲端服務的大小](../cloud-services/cloud-services-sizes-specs.md)。 如需服務限制的詳細資訊，請參閱 [Azure Batch 服務的配額和限制](batch-quota-limit.md)。

> [!TIP]
> 為您的集區建構[自動調整公式][enable_autoscaling]時，請務必考慮 `maxTasksPerNode` 值。 例如，評估 `$RunningTasks` 的公式可能大幅受到每個節點的工作增加的影響。 如需詳細資訊，請參閱 [自動調整 Azure Batch 集區中的運算節點](batch-automatic-scaling.md) 。
>
>

## <a name="distribution-of-tasks"></a>工作的分佈
在集區內的計算節點能夠同時執行工作時，請務必指定您希望在集區內進行跨節點分佈工作的方式。

使用 [CloudPool.TaskSchedulingPolicy][task_schedule] 屬性，您可以指定工作應該跨集區中的所有節點平均指派 (「散佈」)。 或者，您可以在工作指派到集區中其他節點之前，盡可能將最多工作指派給每個節點 (「封裝」)。

做為這項功能有何重要的範例，請考慮上述範例中 [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) 節點的集區，以 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 的值 16 進行設定。 如果 [CloudPool.TaskSchedulingPolicy][task_schedule] 以 [ComputeNodeFillType][fill_type] 為 *Pack* 進行設定，它會最大化每個節點全部 16 個核心的使用量，並且允許[自動調整集區](batch-automatic-scaling.md)從集區剪除未使用的節點 (未指派任何工作的節點)。 這可最小化資源使用量和節省金錢。

## <a name="batch-net-example"></a>Batch .NET 範例
這個 [Batch .NET][api_net] API 程式碼片段示範建立集區的要求，該集區包含四個節點，而每個節點最多有四項工作。 它會指定工作排程原則，該原則會先以工作填滿每個節點，再將工作指派給集區中的其他節點。 如需有關使用 Batch .NET API 新增集區的詳細資訊，請參閱 [BatchClient.PoolOperations.CreatePool][poolcreate_net]。

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.MaxTasksPerComputeNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

## <a name="batch-rest-example"></a>Batch REST 範例
這個 [Batch REST][api_rest] API 程式碼片段示範建立集區的要求，該集區包含兩個大型節點，而每個節點最多有四項工作。 如需有關如何使用 REST API 新增集區的詳細資訊，請參閱[將集區新增至帳戶][rest_addpool]。

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  }
  "targetDedicatedComputeNodes":2,
  "maxTasksPerNode":4,
  "enableInterNodeCommunication":true,
}
```

> [!NOTE]
> 您只能在建立集區時設定 `maxTasksPerNode` 元素和 [MaxTasksPerComputeNode][maxtasks_net] 屬性。 建立集區後無法加以修改。
>
>

## <a name="code-sample"></a>程式碼範例
GitHub 上的 [ParallelNodeTasks][parallel_tasks_sample] 專案提供使用 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 屬性的說明。

這個 C# 主控台應用程式使用 [Batch .NET][api_net] 程式庫來建立具有一或多個計算節點的集區。 它會在這些節點上執行可設定數目的工作，以模擬可變負載。 應用程式的輸出會指定哪些節點執行每個工作。 此應用程式也會提供作業參數和持續時間的摘要。 兩個不同的範例應用程式執行的輸出摘要部分會在下方顯示。

```
Nodes: 1
Node size: large
Max tasks per node: 1
Tasks: 32
Duration: 00:30:01.4638023
```

首次執行範例應用程式會顯示該部分與集區中的單一節點，以及每個節點一項作業的預設設定，作業持續時間超過 30 分鐘。

```
Nodes: 1
Node size: large
Max tasks per node: 4
Tasks: 32
Duration: 00:08:48.2423500
```

第二次執行範例會顯示作業持續時間大幅降低。 這是因為集區已設定為每個節點四項工作，可允許平行工作執行以在近一季的時間完成作業。

> [!NOTE]
> 在上述摘要中的作業持續時間不包括集區建立時間。 上述的每個作業已提交至先前建立的集區，其運算節點在提交時間處於 [閒置]  狀態。
>
>

## <a name="next-steps"></a>後續步驟
### <a name="batch-explorer-heat-map"></a>Batch 總管熱圖
[Batch Explorer][batch_labs] 是免費、功能豐富、獨立用戶端的工具，可以協助建立、偵錯及監視 Azure Batch 應用程式。 Batch Explorer 包含「熱圖」功能，可提供工作執行的視覺效果。 執行 [ParallelTasks][parallel_tasks_sample] 範例應用程式時，您可以使用熱圖功能輕易地視覺化每個節點上的平行工作執行。


[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[enable_autoscaling]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[fill_type]: https://msdn.microsoft.com/library/microsoft.azure.batch.common.computenodefilltype.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[rest_addpool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[task_schedule]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx

