---
title: 提交大量工作 - Azure Batch | Microsoft Docs
description: 如何在單一 Azure Batch 作業中有效率地提交非常大量的工作
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 08/24/2018
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: 3c683b24db2899ee680988c7bedc760d6bb8ec73
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052812"
---
# <a name="submit-a-large-number-of-tasks-to-a-batch-job"></a>將大量工作提交至 Batch 作業

當您執行大規模 Azure Batch 工作負載時，您也許是想要將成千上萬，甚至更多的工作提交至單一作業。 

本文提供指引和一些程式碼範例，用以將具有大幅增加輸送量的大量工作，提交至單一 Batch 作業。 提交工作之後，工作會進入 Batch 佇列，在您為作業指定的集區上進行處理。

## <a name="use-task-collections"></a>使用工作集合

Batch API 提供了有效率地以「集合」形式，將工作新增至作業的方法，不用一次新增一個工作。 新增大量工作時，您應該使用適當的方法或多載，將工作新增為集合。 一般而言，您會藉由在逐一查看一組作業的輸入檔或參數時定義工作，來建構工作集合。

您在單一呼叫中可以新增的工作集合大小上限，取決於您使用的 Batch API：

* 下列 Batch API 將集合限制為 **100 個工作**。 視工作的大小，限制也可以更小；例如，工作有大量資源檔或環境變數時。

    * [REST API](/rest/api/batchservice/task/addcollection)
    * [Python API](/python/api/azure-batch/azure.batch.operations.TaskOperations?view=azure-python#azure_batch_operations_TaskOperations_add_collection)
    * [Node.js API](/javascript/api/azure-batch/task?view=azure-node-latest#addcollection)

  使用這些 API 時，您需要提供邏輯來分割工作數目，以符合集合限制，以及在新增工作失敗時處理錯誤並重試。 如果工作集合太大而無法新增，要求會產生錯誤，應該以較少的工作重試。

* 下列 API 支援更大的工作集合 - 僅受限於提交用戶端上的 RAM 可用性。 這些 API 會以透明方式，針對較低層級的 API 將工作集合分割為「區塊」，並且在新增工作失敗時重試。

    * [.NET API](/dotnet/api/microsoft.azure.batch.cloudjob.addtaskasync?view=azure-dotnet)
    * [Java API](/java/api/com.microsoft.azure.batch.protocol._tasks.addcollectionasync?view=azure-java-stable)
    * [Azure Batch CLI 擴充功能](batch-cli-templates.md)與 Batch CLI 範本
    * [Python SDK 擴充功能](https://pypi.org/project/azure-batch-extensions/)

## <a name="increase-throughput-of-task-submission"></a>增加工作提交的輸送量

將大型工作集合新增至作業需要一些時間 - 例如，透過 .NET API 新增 20,000 個工作需要最多 1 分鐘的時間。 根據 Batch API 和您的工作負載而定，您可以藉由修改下列一或多個項目，以改善工作輸送量：

* **工作大小** - 新增大型工作所花費的時間，比新增較小工作的時間久。 若要減少集合中每個工作的大小，您可以簡化工作命令列、減少環境變數數目，或者更有效率地處理工作執行需求。 例如，並非使用大量資源檔，而是在集區上使用[啟動工作](batch-api-basics.md#start-task)，或使用[應用程式套件](batch-application-packages.md)或 [Docker 容器](batch-docker-container-workloads.md)，安裝工作相依性。

* **平行作業數目** - 依據 Batch API 而定，藉由增加 Batch 用戶端的並行作業數目上限來增加輸送量。 使用 .NET API 中的 [BatchClientParallelOptions.MaxDegreeOfParallelism](/dotnet/api/microsoft.azure.batch.batchclientparalleloptions.maxdegreeofparallelism) 屬性，或方法 (例如 Batch Python SDK 擴充功能中的 [TaskOperations.add_collection](/python/api/azure-batch/azure.batch.operations.TaskOperations?view=azure-python#add-collection)) 的 `threads` 參數，進行這項設定。 (這個屬性不適用於原生 Batch Python SDK。)根據預設，此屬性設為 1，但是設為較高的數值可改善作業的輸送量。 您會耗用網路頻寬和一些 CPU 效能，來換取增加的輸送量。 工作輸送量會增加到最多 100 乘以 `MaxDegreeOfParallelism` 或 `threads`。 在實務上，您應該將並行作業數目設為 100 以下。 
 
  Azure Batch CLI 擴充功能與 Batch 範本，會根據可用核心數目自動增加並行作業數目，但是這個屬性在 CLI 中無法設定。 

* **HTTP 連線限制** - 並行 HTTP 連線數目可以在 Batch 用戶端新增大量工作時，對用戶端的效能進行節流。 HTTP 連線數目受限於特定 API。 例如，使用 .NET API 進行開發時，[ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit) 屬性預設會設為 2。 我們建議您將此值增加為接近或大於平行作業數目的值。

## <a name="example-batch-net"></a>範例：Batch .NET

下列 C# 程式碼片段會顯示使用 Batch .NET API 新增大量工作時，所要進行的設定。

若要增加工作輸送量，請針對 [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient?view=azure-dotnet) 的 [MaxDegreeofParallelism](/dotnet/api/microsoft.azure.batch.batchclientparalleloptions.maxdegreeofparallelism) 屬性增加值。 例如︰

```csharp
BatchClientParallelOptions parallelOptions = new BatchClientParallelOptions()
  {
    MaxDegreeOfParallelism = 15
  };
...
```
使用 [AddTaskAsync](/dotnet/api/microsoft.azure.batch.cloudjob.addtaskasync?view=azure-dotnet) 或 [AddTask](/dotnet/api/microsoft.azure.batch.cloudjob.addtask?view=azure-dotnet
) 方法的適當多載，將工作集合新增至作業。 例如︰

```csharp
// Add a list of tasks as a collection
List<CloudTask> tasksToAdd = new List<CloudTask>(); // Populate with your tasks
...
await batchClient.JobOperations.AddTaskAsync(jobId, tasksToAdd, parallelOptions);
```


## <a name="example-batch-cli-extension"></a>範例：Batch CLI 擴充功能

使用 Azure Batch CLI 擴充功能與 [Batch CLI 範本](batch-cli-templates.md)來建立作業範本 JSON 檔案，其中包含[工作處理站](https://github.com/Azure/azure-batch-cli-extensions/blob/master/doc/taskFactories.md) (英文)。 工作處理站會根據單一工作定義，設定作業的相關工作集合。  

以下是具有大量工作 (在此案例中是 250,000 個) 的一維參數掃掠作業。 工作命令列是簡單的 `echo` 命令。

```json
{
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "myjob",
            "constraints": {
                "maxWallClockTime": "PT5H",
                "maxTaskRetryCount": 1
            },
            "poolInfo": {
                "poolId": "mypool"
            },
            "taskFactory": {
                "type": "parametricSweep",
                "parameterSets": [
                    {
                        "start": 1,
                        "end": 250000,
                        "step": 1
                    }
                ],
                "repeatTask": {
                    "commandLine": "/bin/bash -c 'echo Hello world from task {0}'",
                    "constraints": {
                        "retentionTime":"PT1H"
                    }
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
```
若要使用範本執行作業，請參閱[使用 Azure Batch CLI 範本和檔案傳輸](batch-cli-templates.md)。

## <a name="example-batch-python-sdk-extension"></a>範例：Batch Python SDK 擴充功能

若要使用 Azure Batch Python SDK 擴充功能，請先安裝 Python SDK 和擴充功能：

```
pip install azure-batch
pip install azure-batch-extensions
```

設定 `BatchExtensionsClient`，它會使用 SDK 擴充功能：

```python

client = batch.BatchExtensionsClient(base_url=BATCH_ACCOUNT_URL, resource_group=RESOURCE_GROUP_NAME, batch_account=BATCH_ACCOUNT_NAME)
...
```

建立要新增至作業的工作集合。 例如︰


```python
tasks=list()
# Populate the list with your tasks
...

```

使用 [task.add_collection](/python/api/azure-batch/azure.batch.operations.TaskOperations?view=azure-python#add-collection) 來新增工作集合。 設定 `threads` 參數來增加並行作業數目：

```python
try:
    client.task.add_collection(job_id, threads=100)
except Exception as e:
    raise e
```

Batch Python SDK 擴充功能也支援使用工作處理站的 JSON 規格，將工作參數新增至作業。 例如，為參數掃掠設定作業參數，類似於前述 [Batch CLI 範本](#example-batch-cli-template)範例中的項目：

```python
parameter_sweep = {
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "myjob",
            "poolInfo": {
                "poolId": "mypool"
            },
            "taskFactory": {
                "type": "parametricSweep",
                "parameterSets": [
                    {
                        "start": 1,
                        "end": 250000,
                        "step": 1
                    }
                ],
                "repeatTask": {
                    "commandLine": "/bin/bash -c 'echo Hello world from task {0}'",
                    "constraints": {
                        "retentionTime":"PT1H"
                    }
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
...
job_json = client.job.expand_template(parameter_sweep)
job_parameter = client.job.jobparameter_from_json(job_json)
```

將作業參數新增至作業。 設定 `threads` 參數來增加並行作業數目：

```python
try:
    client.job.add(job_parameter, threads=50)
except Exception as e:
    raise e
```

## <a name="next-steps"></a>後續步驟

* 深入了解如何使用 Azure Batch CLI 擴充功能，來搭配 [Batch CLI 範本](batch-cli-templates.md)。
* 深入了解 [Batch Python SDK 擴充功能](https://pypi.org/project/azure-batch-extensions/)。