---
title: 建立用來準備作業的工作並對計算節點完成作業 - Azure Batch | Microsoft Docs
description: 使用作業層級準備工作以減少傳輸到 Azure Batch 計算節點的資料，並在作業完成時解除工作以清理節點。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 63d9d4f1-8521-4bbb-b95a-c4cad73692d3
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 02/27/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: da69cc22fbb071ce3fa4b2c53aaf0b1ec4ba5e46
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096305"
---
# <a name="run-job-preparation-and-job-release-tasks-on-batch-compute-nodes"></a>在 Batch 計算節點上執行作業準備和作業解除工作

 Azure Batch 作業在執行其工作之前，通常需要經過某種形式的設定，並需要在其工作完成時進行後置作業維護。 您可能需要將常見的工作輸入資料下載到計算節點，或在作業完成後，將工作輸出資料上傳至 Azure 儲存體。 您可以使用**作業準備**和**作業解除**工作來執行這些操作。

## <a name="what-are-job-preparation-and-release-tasks"></a>什麼是作業準備和作業解除工作？
在作業的工作執行之前，會在排定執行至少一個工作的所有計算節點上執行作業準備工作。 作業一旦完成，作業解除工作會在集區中的每個節點上執行，集區至少會執行一個工作。 如同一般的 Batch 工作，您可以指定要在作業準備或作業解除工作執行時叫用的命令列。

作業準備和作業解除工作會提供熟悉的 Batch 工作功能，例如檔案下載 ([資源檔][net_job_prep_resourcefiles])、提升權限的執行、自訂環境變數、最大執行持續時間、重試計數和檔案保留時間。

在接下來幾節中，您將了解如何使用在 [Batch .NET][api_net] 程式庫中找到的 [JobPreparationTask][net_job_prep] 和 [JobReleaseTask][net_job_release] 類別。

> [!TIP]
> 作業準備和作業解除工作在「共用集區」環境中特別有用；在這種環境中，計算節點的集區會在作業執行之間保存，並由許多作業使用。
> 
> 

## <a name="when-to-use-job-preparation-and-release-tasks"></a>使用作業準備和作業解除工作的時機
作業準備和作業解除工作適用於下列情況︰

**下載常見的工作資料**

Batch 作業通常需要一組常用的資料做為作業工作的輸入。 例如，在每日風險分析計算中，市場資料是作業專屬的，也是作業中所有工作通用的。 這個市場資料 (通常是幾個 GB 的大小) 應該只下載到各個計算節點一次，讓在節點上執行的任何工作可以使用它。 在作業的其他工作執行之前使用 **作業準備工作** 將此資料下載到每個節點。

**刪除作業和工作的輸出**

在「共用集區」環境中，作業之間的集區計算節點並未解除委任，因此您可能需要刪除執行之間的作業資料。 您可能需要保留節點上的磁碟空間，或符合組織的安全性原則。 使用 **作業解除工作** 可刪除作業準備工作所下載的資料或在工作執行期間產生的資料。

**記錄保留**

您可能想要保留一份工作產生的記錄檔，或失敗應用程式所產生的損毀傾印檔案。 在這類情況下使用**作業解除工作**，將此資料壓縮並上傳至 [Azure 儲存體][azure_storage]帳戶。

> [!TIP]
> 另一個可保存記錄以及其他作業和工作輸出資料的方法是使用 [Azure Batch 檔案慣例](batch-task-output.md) 庫。
> 
> 

## <a name="job-preparation-task"></a>作業準備工作
執行作業的工作之前，Batch 會在排定執行工作的每個計算節點上執行作業準備工作。 依預設，Batch 服務會等作業準備工作完成，才執行節點上排定的工作。 不過，您可以設定服務不要等待。 如果節點重新啟動，作業準備工作會再次執行，但您也可以停用此行為。

作業準備工作只會在排定執行工作的節點上執行。 這可避免未指派工作的節點執行不必要的準備工作。 這種情況會發生在當作業的工作數目小於集區中的節點數目時。 此外，也適用於已啟用 [並行工作執行](batch-parallel-node-tasks.md) 時，而如果作業計數小於可能的並行工作總數，則會讓一些節點閒置。 透過不在閒置的節點上執行作業準備工作，您在資料傳輸費用上可以花費更少金錢。

> [!NOTE]
> [JobPreparationTask][net_job_prep_cloudjob] 與 [CloudPool.StartTask][pool_starttask] 的不同之處在於，JobPreparationTask 在每個作業開始時執行，而 StartTask 只在計算節點第一次加入集區或重新啟動時執行。
> 
> 

## <a name="job-release-task"></a>作業解除工作
作業一旦標示為完成，作業解除工作會在集區中的每個節點上執行，集區至少會執行一個工作。 透過發出終止要求將工作標示為完成。 然後 Batch 服務會將作業狀態設定為「正在終止」，終止與作業相關聯的任何作用中或執行中工作，並執行作業解除工作。 作業接著會進入「已完成」狀態。

> [!NOTE]
> 工作刪除也會執行工作解除任務。 不過，如果先前已終止作業，當後來刪除該作業時，解除工作不會執行第二次。
> 
> 

## <a name="job-prep-and-release-tasks-with-batch-net"></a>使用 Batch .NET 進行作業準備和作業解除工作
若要使用作業準備工作，請將 [JobPreparationTask][net_job_prep] 物件指派給作業的 [CloudJob.JobPreparationTask][net_job_prep_cloudjob] 屬性。 同樣地，初始化 [JobReleaseTask][net_job_release] 並將它指派給作業的 [CloudJob.JobReleaseTask][net_job_prep_cloudjob] 屬性即可設定作業的解除任務。

在此程式碼片段中，`myBatchClient` 是 [BatchClient][net_batch_client] 的執行個體，而 `myPool` 是 Batch 帳戶內的現有集區。

```csharp
// Create the CloudJob for CloudPool "myPool"
CloudJob myJob =
    myBatchClient.JobOperations.CreateJob(
        "JobPrepReleaseSampleJob",
        new PoolInformation() { PoolId = "myPool" });

// Specify the command lines for the job preparation and release tasks
string jobPrepCmdLine =
    "cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
string jobReleaseCmdLine =
    "cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";

// Assign the job preparation task to the job
myJob.JobPreparationTask =
    new JobPreparationTask { CommandLine = jobPrepCmdLine };

// Assign the job release task to the job
myJob.JobReleaseTask =
    new JobReleaseTask { CommandLine = jobReleaseCmdLine };

await myJob.CommitAsync();
```

如先前所述，終止或刪除作業時會執行解除任務。 使用 [JobOperations.TerminateJobAsync][net_job_terminate] 終止作業。 使用 [JobOperations.DeleteJobAsync][net_job_delete] 刪除作業。 您通常會在作業的工作完成時或達到定義之逾時時終止或刪除作業。

```csharp
// Terminate the job to mark it as Completed; this will initiate the
// Job Release Task on any node that executed job tasks. Note that the
// Job Release Task is also executed when a job is deleted, thus you
// need not call Terminate if you typically delete jobs after task completion.
await myBatchClient.JobOperations.TerminateJobAsync("JobPrepReleaseSampleJob");
```

## <a name="code-sample-on-github"></a>GitHub 上的程式碼範例
若要了解作業準備和作業解除工作的運作，請查看 GitHub 上的 [JobPrepRelease][job_prep_release_sample] 範例專案。 此主控台應用程式會做這些事：

1. 建立包含兩個節點的集區。
2. 建立具有作業準備、解除和標準工作的作業。
3. 執行作業準備工作，將節點識別碼第一次寫入節點的「共用」目錄的文字檔中。
4. 在每個節點上執行工作，將其工作識別碼寫入同一文字檔案中。
5. 一旦完成所有工作 (或達到逾時)，會列印每個節點的文字檔案的內容到主控台。
6. 在作業完成時，執行作業解除工作會將該檔案從節點中刪除。
7. 列印執行作業準備和解除工作的每個節點上這些工作的結束代碼。
8. 暫停執行以允許確認刪除作業和/或集區。

範例應用程式的輸出類似這樣：

```
Attempting to create pool: JobPrepReleaseSamplePool
Created pool JobPrepReleaseSamplePool with 2 nodes
Checking for existing job JobPrepReleaseSampleJob...
Job JobPrepReleaseSampleJob not found, creating...
Submitting tasks and awaiting completion...
All tasks completed.

Contents of shared\job_prep_and_release.txt on tvm-2434664350_1-20160623t173951z:
-------------------------------------------
tvm-2434664350_1-20160623t173951z tasks:
  task001
  task004
  task005
  task006

Contents of shared\job_prep_and_release.txt on tvm-2434664350_2-20160623t173951z:
-------------------------------------------
tvm-2434664350_2-20160623t173951z tasks:
  task008
  task002
  task003
  task007

Waiting for job JobPrepReleaseSampleJob to reach state Completed
...

tvm-2434664350_1-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

tvm-2434664350_2-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

Delete job? [yes] no
yes
Delete pool? [yes] no
yes

Sample complete, hit ENTER to exit...
```

> [!NOTE]
> 因為新集區中各個節點的建立和啟動時間並不一樣 (某些節點比其他節點還早做好工作準備)，您可能會看到不同的輸出。 具體而言，因為工作會快速完成，集區的某個節點可能會執行作業的所有工作。 如果發生這種情況，您會發現未執行任何工作的節點不會有作業準備和作業解除工作存在。
> 
> 

### <a name="inspect-job-preparation-and-release-tasks-in-the-azure-portal"></a>在 Azure 入口網站中檢查作業準備和作業解除工作
當您執行範例應用程式時，您可以使用 [Azure 入口網站][portal]檢視作業和其工作的屬性，或甚至下載作業的工作修改過的共用文字檔案。

下面的螢幕擷取畫面顯示在執行範例應用程式之後，Azure 入口網站中所出現的 **準備工作刀鋒視窗** 。 在工作完成之後 (但在刪除作業與集區之前) 瀏覽至 JobPrepReleaseSampleJob 屬性，然後按一下 [準備工作] 或 [解除工作] 以檢視其屬性。

![Azure 入口網站中的作業準備屬性][1]

## <a name="next-steps"></a>後續步驟
### <a name="application-packages"></a>應用程式套件
除了作業準備工作外，您還可以使用 Batch 的 [應用程式套件](batch-application-packages.md)功能來為計算節點做好工作執行準備。 此功能特別適合用於部署不需要執行安裝程式的應用程式、包含許多 (100 個以上) 檔案的應用程式，或需要嚴格版本控制的應用程式。

### <a name="installing-applications-and-staging-data"></a>安裝應用程式和預備資料
這篇 MSDN 論壇文章概述幾個方法來讓您的節點做好執行工作的準備︰

[在 Batch 計算節點上安裝應用程式和預備資料][forum_post]

其作者是其中一位 Azure Batch 小組成員，內容在討論數種可供您部署應用程式和資料到計算節點的技巧。

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[azure_storage]: https://azure.microsoft.com/services/storage/
[portal]: https://portal.azure.com
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]:https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_job_prep_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_job_prep_resourcefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.resourcefiles.aspx
[net_job_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.deletejobasync.aspx
[net_job_terminate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_job_release]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobreleasetask.aspx
[net_job_release_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx

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

[1]: ./media/batch-job-prep-release/portal-jobprep-01.png
