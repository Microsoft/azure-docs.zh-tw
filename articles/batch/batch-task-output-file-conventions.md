---
title: 使用適用於 .NET 的檔案慣例程式庫將作業和工作輸出保存到 Azure 儲存體 - Azure Batch | Microsoft Docs
description: 了解如何使用適用於 .NET 的 Azure Batch 檔案慣例程式庫，將 Batch 工作和作業輸出保存到 Azure 儲存體，並在 Azure 入口網站檢視保存的輸出。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 16e12d0e-958c-46c2-a6b8-7843835d830e
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 06/16/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0b4ff1799f77581452859d1dbc0e6e9cc47062e4
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128044"
---
# <a name="persist-job-and-task-data-to-azure-storage-with-the-batch-file-conventions-library-for-net"></a>使用適用於 .NET 的 Batch 檔案慣例程式庫將作業和工作輸出保存到 Azure 儲存體 

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

保存工作資料的其中一個方法是使用[適用於 .NET 的 Azure Batch 檔案慣例程式庫][nuget_package]。 檔案慣例程式庫會簡化將工作輸出資料儲存至 Azure 儲存體並且擷取它的程序。 您可以將檔案慣例程式庫用於工作和用戶端程式碼 &mdash; 在工作程式碼中用來保存檔案，以及在用戶端程式碼中用來列出和擷取檔案。 您的工作程式碼也可以將程式庫用於擷取上游工作的輸出，例如[工作相依性](batch-task-dependencies.md)案例。 

若要使用檔案慣例程式庫擷取輸出檔案，您可以藉由依據識別碼和用途列出它們，找到指定作業或工作的檔案。 您不需要知道檔案的名稱或位置。 例如，您可以使用檔案慣例程式庫以列出指定工作的所有中繼檔案，或取得指定作業的預覽檔案。

> [!TIP]
> 從 2017-05-01 版開始，Batch 服務 API 針對使用虛擬機器設定建立，在集區上執行的工作和作業管理員工作，支援將輸出資料保存到 Azure 儲存體。 Batch 服務 API 提供簡單的方式來保存程式碼內的輸出，該程式碼會建立工作，並且作為檔案慣例程式庫的替代方案。 您可以修改 Batch 用戶端應用程式以保存輸出，而不需要更新您的工作正在執行的應用程式。 如需詳細資訊，請參閱[使用 Batch 服務 API 將工作資料保存到 Azure 儲存體](batch-task-output-files.md)。
> 
> 

## <a name="when-do-i-use-the-file-conventions-library-to-persist-task-output"></a>使用檔案慣例程式庫保存工作輸出的時機？

Azure Batch 提供多個方法來保存工作輸出。 檔案慣例最適合這些案例：

- 您可以輕鬆地修改您的工作正在執行的應用程式程式碼，使用檔案慣例程式庫來保存檔案。
- 您在工作還在執行時，想要將資料串流至 Azure 儲存體。
- 您想要保存的資料是在使用雲端服務設定或虛擬機器設定建立的集區中。
- 用戶端應用程式或作業中的其他工作必須依識別碼或依用途來尋找及下載工作輸出檔案。 
- 您想要在 Azure 入口網站中檢視工作輸出。

如果您的情節與以上所列不同，可能需要考慮不同的方法。 如需保存工作輸出之其他選項的詳細資訊，請參閱[將作業和工作輸出保存到 Azure 儲存體](batch-task-output.md)。 

## <a name="what-is-the-batch-file-conventions-standard"></a>Batch 檔案慣例標準是什麼？

[Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)提供目的地容器的命名配置，以及寫入輸出檔案的 blob 路徑。 保存到 Azure 儲存體 (符合檔案慣例標準) 的檔案會自動可在 Azure 入口網站中檢視。 入口網站知道命名慣例，因此可以顯示符合它的檔案。

適用於 .NET 的檔案慣例程式庫會根據檔案慣例標準，自動命名您的儲存體容器和工作輸出檔案。 檔案慣例程式庫也提供方法來根據作業識別碼、工作識別碼或目的，在 Azure 儲存體中查詢輸出檔案。   

如果您要使用 .NET 以外的語言進行開發，可以在您的應用程式中自行實作檔案慣例標準。 如需詳細資訊，請參閱[關於 Batch 檔案慣例標準](batch-task-output.md#about-the-batch-file-conventions-standard)。

## <a name="link-an-azure-storage-account-to-your-batch-account"></a>將 Azure 儲存體帳戶連結到您的 Batch 帳戶

若要使用檔案慣例程式庫將輸出資料保存到 Azure 儲存體，您必須先將 Azure 儲存體帳戶連結到您的 Batch 帳戶。 如果您尚未這麼做，請使用 [Azure 入口網站](https://portal.azure.com)將儲存體帳戶連結到您的 Batch 帳戶：

1. 在 Azure 入口網站中瀏覽至您的 Batch 帳戶。 
2. 在 [設定] 下，選取 [儲存體帳戶]。
3. 如果您還沒有與您的 Batch 帳戶相關聯的儲存體帳戶，請按一下 [儲存體帳戶 (無)]。
4. 從訂用帳戶清單中選取儲存體帳戶。 為了達到最佳效能，使用位於與您執行工作所在之 Batch 帳戶相同區域的 Azure 儲存體帳戶。

## <a name="persist-output-data"></a>保存輸出資料

若要使用檔案慣例程式庫保存作業和工作輸出資料，請在 Azure 儲存體中建立容器，然後將輸出儲存至容器。 在您的工作程式碼中使用[適用於 .NET 的 Azure 儲存體用戶端程式庫](https://www.nuget.org/packages/WindowsAzure.Storage)，將工作輸出上傳到容器中。 

如需在 Azure 儲存體中使用容器和 Blob 的詳細資訊，請參閱[以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。

> [!WARNING]
> 使用檔案慣例程式庫保存的所有作業和工作輸出，都會儲存在相同容器中。 如果大量工作同時嘗試保存檔案，可能會強制執行[儲存體節流限制](../storage/common/storage-performance-checklist.md#blobs)。
> 
> 

### <a name="create-storage-container"></a>建立儲存體容器

若要將工作輸出保存到 Azure 儲存體，先建立一個名為 [CloudJob][net_cloudjob].[PrepareOutputStorageAsync][net_prepareoutputasync] 的容器。 這個擴充方法會採用 [CloudStorageAccount][net_cloudstorageaccount] 物件作為參數。 它會建立容器，容器名稱是根據檔案慣例標準，使其內容可以由 Azure 入口網站和本文稍後討論的擷取方法進行探索。

您通常會放置此程式碼以在用戶端應用程式中建立容器 &mdash; 也就是建立您的集區、作業及工作的應用程式。

```csharp
CloudJob job = batchClient.JobOperations.CreateJob(
    "myJob",
    new PoolInformation { PoolId = "myPool" });

// Create reference to the linked Azure Storage account
CloudStorageAccount linkedStorageAccount =
    new CloudStorageAccount(myCredentials, true);

// Create the blob storage container for the outputs
await job.PrepareOutputStorageAsync(linkedStorageAccount);
```

### <a name="store-task-outputs"></a>儲存工作輸出

當您在 Azure 儲存體中備妥容器之後，工作便可以使用檔案慣例程式庫中的 [TaskOutputStorage][net_taskoutputstorage] 類別來將輸出儲存到容器。

在您的工作程式碼中，先建立一個 [TaskOutputStorage][net_taskoutputstorage] 物件，然後在工作完成時呼叫 [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync] 方法，以將其輸出儲存到 Azure 儲存體。

```csharp
CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");

TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
    linkedStorageAccount, jobId, taskId);

/* Code to process data and produce output file(s) */

await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");
```

[TaskOutputStorage](https://msdn.microsoft.com/library/microsoft.azure.batch.conventions.files.taskoutputstorage.aspx).[SaveAsync](https://msdn.microsoft.com/library/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync.aspx) 方法的 `kind` 參數會分類保存的檔案。 有四個預先定義的 [TaskOutputKind][net_taskoutputkind] 類型：`TaskOutput`、`TaskPreview`、`TaskLog` 和 `TaskIntermediate.`。您也可以定義輸出的自訂類別。

這些輸出類型可供您在稍後針對特定工作的保存輸出查詢 Batch 時，指定要列出的輸出類型。 換句話說，當您列出某個工作的輸出時，可以根據其中一個輸出類型來篩選清單。 例如，「給我工作 *109* 的 *preview* 輸出」。 本文稍後的 [擷取輸出](#retrieve-output) 會提供列出和擷取輸出的詳細資訊。

> [!TIP]
> 輸出類型也會決定特定檔案出現在 Azure 入口網站的位置：TaskOutput 類別的檔案會出現在 [工作輸出檔案] 底下，而 TaskLog 檔案會出現在 [工作記錄] 底下。
> 
> 

### <a name="store-job-outputs"></a>儲存工作輸出

除了儲存工作輸出，您也可以儲存和整個作業相關聯的輸出。 例如，在影片轉譯作業的合併工作中，您可以將完整轉譯的影片以作業輸出的形式保存。 當您的作業完成時，用戶端應用程式即可列出並擷取該作業的輸出，而不需要查詢個別的工作。

呼叫 [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] 方法，並指定 [JobOutputKind][net_joboutputkind] 和檔案名稱以儲存作業輸出：

```csharp
CloudJob job = new JobOutputStorage(acct, jobId);
JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);

await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");
```

如同工作輸出的 **TaskOutputKind** 類型，您會使用 [JobOutputKind][net_joboutputkind] 類型來分類作業的保存檔案。 此參數可讓您在稍後查詢 (列出) 特定的輸出類型。 **JobOutputKind** 類型包含了輸出和預覽類別，且支援建立自訂類別。

### <a name="store-task-logs"></a>儲存工作記錄檔

除了在工作或作業完成時將檔案保存到永久性儲存體之外，您可能也會需要保存在工作執行期間更新的檔案 &mdash; 例如記錄檔或 `stdout.txt` 和 `stderr.txt`。 針對此目的，Azure Batch 檔案慣例庫會提供 [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync] 方法。 透過 [SaveTrackedAsync][net_savetrackedasync]，您可以追蹤節點上檔案的更新 (依照您指定的時間間隔)，並將這些更新保存到 Azure 儲存體。

在下列程式碼片段中，我們使用 [SaveTrackedAsync][net_savetrackedasync]，於工作執行期間每 15 秒更新一次 Azure 儲存體中的 `stdout.txt`：

```csharp
TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
string logFilePath = Path.Combine(
    Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

// The primary task logic is wrapped in a using statement that sends updates to
// the stdout.txt blob in Storage every 15 seconds while the task code runs.
using (ITrackedSaveOperation stdout =
        await taskStorage.SaveTrackedAsync(
        TaskOutputKind.TaskLog,
        logFilePath,
        "stdout.txt",
        TimeSpan.FromSeconds(15)))
{
    /* Code to process data and produce output file(s) */

    // We are tracking the disk file to save our standard output, but the
    // node agent may take up to 3 seconds to flush the stdout stream to
    // disk. So give the file a moment to catch up.
     await Task.Delay(stdoutFlushDelay);
}
```

加上註解的區段 `Code to process data and produce output file(s)` 是您的工作正常會執行的程式碼預留位置。 例如，您可能有程式碼會從 Azure 儲存體下載資料，並對這些資料執行轉換或計算。 此程式碼片段的重點，在於示範如何將這樣的程式碼用 `using` 區塊包裝，以透過 [SaveTrackedAsync][net_savetrackedasync] 定期更新檔案。

節點代理程式是一項程式，會在集區中的每個節點上執行，並在節點與 Batch 服務之間提供命令和控制介面。 `Task.Delay` 呼叫在此 `using` 區塊的結尾是必要的，以確保節點代理程式有時間清除節點上 stdout.txt 檔案的標準內容。 若沒有此延遲，就可能會遺漏輸出的最後幾秒。 此延遲可能並非所有檔案都需要。

> [!NOTE]
> 當您使用 **SaveTrackedAsync** 啟用檔案追蹤時，只有附加到追蹤檔案的資料會保存到 Azure 儲存體。 請只將此方法用於追蹤非輪替記錄檔，或其他使用附加作業寫入至結尾的檔案。
> 
> 

## <a name="retrieve-output-data"></a>擷取輸出資料

當您使用 Azure Batch 檔案慣例庫擷取保存的輸出時，您是以工作和作業為主的方式進行。 您可以要求指定工作或作業的輸出，而不需要知道它在 Azure 儲存體中的位置，甚至連檔案名稱也不需要。 相反地，您可以依據工作或作業識別碼要求輸出檔案。

下列程式碼片段會逐一查看作業的工作，並顯示該工作輸出檔案的相關資訊，然後從儲存體下載該工作的檔案。

```csharp
foreach (CloudTask task in myJob.ListTasks())
{
    foreach (OutputFileReference output in
        task.OutputStorage(storageAccount).ListOutputs(
            TaskOutputKind.TaskOutput))
    {
        Console.WriteLine($"output file: {output.FilePath}");

        output.DownloadToFileAsync(
            $"{jobId}-{output.FilePath}",
            System.IO.FileMode.Create).Wait();
    }
}
```

## <a name="view-output-files-in-the-azure-portal"></a>在 Azure 入口網站中檢視輸出檔案

Azure 入口網站會顯示使用 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)保存到已連結 Azure 儲存體帳戶的工作輸出檔案和記錄。 您可以使用自選的語言實作這些慣例，或是使用 .NET 應用程式中的檔案慣例程式庫。

若要讓您的輸出檔案顯示在入口網站中，您必須滿足下列需求：

1. [連結 Azure 儲存體帳戶](#requirement-linked-storage-account) 到您的 Batch 帳戶。
2. 保存輸出時，依照預先定義的儲存體容器命名與檔案命名慣例。 您可以在檔案慣例程式庫的[讀我檔案][github_file_conventions_readme]中找到這些慣例的定義。 如果您使用 [Azure Batch 檔案慣例][nuget_package]程式庫來保存您的輸出，您的檔案會根據檔案慣例標準進行保存。

若要在 Azure 入口網站中檢視工作輸出檔案和記錄，請瀏覽到您對其輸出有興趣的工作，然後按一下 [已儲存的輸出檔案] 或 [已儲存的記錄]。 此影像顯示識別碼為 "007" 之工作的 [已儲存的輸出檔案]  ：

![Azure 入口網站中的 [工作輸出] 刀鋒視窗][2]

## <a name="code-sample"></a>程式碼範例

[PersistOutputs][github_persistoutputs] 範例專案是 GitHub 上的其中一個 [Azure Batch 程式碼範例][github_samples]。 此 Visual Studio 解決方案示範如何使用 Azure Batch 檔案慣例庫，將工作輸出保存到永久性儲存體。 若要執行範例，請遵循下列步驟：

1. 在 **Visual Studio 2017** 中開啟專案。
2. 將您 Batch 和儲存體的**帳戶認證**新增到 Microsoft.Azure.Batch.Samples.Common 專案中的 **AccountSettings.settings**。
3.  (但不要執行) 該解決方案。 如果出現提示，請還原任何 NuGet 封裝。
4. 使用 Azure 入口網站來為 [PersistOutputsTask](batch-application-packages.md) 上傳 **應用程式封裝**。 將 `PersistOutputsTask.exe` 及其相依性組件包含在 .zip 封裝中，將應用程式識別碼和應用程式封裝版本分別設為 "PersistOutputsTask" 和 "1.0"。
5. **啟動** (執行) **PersistOutputs** 專案。
6. 當系統提示您選擇要用於執行範例的持續性技術時，輸入 **1** 可使用檔案慣例程式庫保存工作輸出來執行範例。 

## <a name="next-steps"></a>後續步驟

### <a name="get-the-batch-file-conventions-library-for-net"></a>取得適用於 .NET 的 Batch 檔案慣例程式庫

適用於 .NET 的 Batch 檔案慣例程式庫可以在 [NuGet][nuget_package] 取得。 程式庫會使用新方法擴充 [CloudJob][net_cloudjob] 和 [CloudTask][net_cloudtask] 類別。 另請參閱檔案慣例程式庫的[參考文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.conventions.files)。

檔案慣例程式庫的[原始程式碼][github_file_conventions]可以在 GitHub 上的 Microsoft Azure SDK for .NET 存放庫中取得。 

### <a name="explore-other-approaches-for-persisting-output-data"></a>探索保存輸出資料的其他方法

- 請參閱[將作業和工作輸出保存到 Azure 儲存體](batch-task-output.md)，以取得保存工作和作業資料的概觀。
- 請參閱[使用 Batch 服務 API 將工作資料保存到 Azure 儲存體](batch-task-output-files.md)，以深入了解如何使用 Batch 服務 API 來保存輸出資料。

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Batch/FileConventions
[github_file_conventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_cloudstorageaccount]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.aspx
[net_cloudtask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_fileconventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/AutoRest/src/Batch/FileConventions/README.md
[net_joboutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputkind.aspx
[net_joboutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.aspx
[net_joboutputstorage_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync.aspx
[net_msdn]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[net_prepareoutputasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync.aspx
[net_saveasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync.aspx
[net_savetrackedasync]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync.aspx
[net_taskoutputkind]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputkind.aspx
[net_taskoutputstorage]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.aspx
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: http://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "入口網站中的 [已儲存的輸出檔案] 和 [已儲存的記錄檔] 選取器"
[2]: ./media/batch-task-output/task-output-02.png "Azure 入口網站中的 [工作輸出] 刀鋒視窗"
