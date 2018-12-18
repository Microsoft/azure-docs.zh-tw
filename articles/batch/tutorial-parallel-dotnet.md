---
title: 執行平行工作負載 - Azure Batch .NET
description: 教學課程 - 使用 Batch .NET 用戶端程式庫透過 Azure Batch 中的 ffmpeg 平行進行媒體檔案轉碼
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 09/07/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 02b715ade9a9a537f6bd0e476ada299140bff4bb
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48815506"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-net-api"></a>教學課程：使用 .NET API 透過 Azure Batch 執行平行工作負載

使用 Azure Batch 在 Azure 中有效率地執行大規模的平行和高效能運算 (HPC) 批次作業。 本教學課程會逐步說明使用 Batch 執行平行工作負載的 C# 範例。 您會了解 Batch 應用程式的通用工作流程，以及如何以程式設計方式與 Batch 和儲存體資源進行互動。 您會了解如何：

> [!div class="checklist"]
> * 將應用程式套件新增至您的 Batch 帳戶
> * 驗證 Batch 和儲存體帳戶
> * 將輸入檔案上傳至儲存體
> * 建立計算節點的集區來執行應用程式
> * 建立作業和工作來處理輸入檔案
> * 監視工作執行
> * 擷取輸出檔案

在本教學課程中，您會使用 [ffmpeg](http://ffmpeg.org/) 開放原始碼工具將 MP4 媒體檔案平行轉換為 MP3 格式。 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

* 適用於 Linux、macOS 或 Windows 的 [Visual Studio 2017](https://www.visualstudio.com/vs) 或 [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1)。

* Batch 帳戶和連結的 Azure 儲存體帳戶。 若要建立這些帳戶，請參閱使用 [Azure 入口網站](quick-create-portal.md)或 [Azure CLI](quick-create-cli.md) 的 Batch 快速入門。

* [Windows 64 位元版本的 ffmpeg 3.4](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-3.4-win64-static.zip) (.zip)。 將 ZIP 檔案下載到本機電腦。 在本教學課程中，您只需要 ZIP 檔案。 您不需要將此檔案解壓縮或在本機進行安裝。 

## <a name="sign-in-to-azure"></a>登入 Azure

在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

## <a name="add-an-application-package"></a>新增應用程式套件

使用 Azure 入口網站，將 ffmpeg 新增至 Batch 帳戶作為[應用程式套件](batch-application-packages.md)。 應用程式套件可協助您管理工作應用程式並將其部署到集區中的計算節點。 

1. 在 Azure 入口網站中，按一下 [更多服務] > [Batch 帳戶]，然後按一下您的 Batch 帳戶名稱。
3. 按一下 [應用程式] > [新增]。
4. 針對 [應用程式識別碼]，輸入 ffmpeg，以及 3.4 套件版本。 選取您先前下載的 ffmpeg ZIP 檔案，然後按一下 [確定]。 ffmpeg 應用程式套件會新增至您的 Batch 帳戶。

![新增應用程式套件](./media/tutorial-parallel-dotnet/add-application.png)

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-and-run-the-sample"></a>下載及執行範例

### <a name="download-the-sample"></a>下載範例

從 GitHub [下載或複製範例應用程式](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial)。 若要使用 Git 用戶端複製範例應用程式存放庫，請使用下列命令：

```
git clone https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial.git
```

瀏覽至包含 Visual Studio 方案檔 `BatchDotNetFfmpegTutorial.sln` 的目錄。

在 Visual Studio 中開啟方案檔，然後使用您為帳戶取得的值來更新 `program.cs` 中的認證字串。 例如︰

```csharp
// Batch account credentials
private const string BatchAccountName = "mybatchaccount";
private const string BatchAccountKey  = "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==";
private const string BatchAccountUrl  = "https://mybatchaccount.mybatchregion.batch.azure.com";

// Storage account credentials
private const string StorageAccountName = "mystorageaccount";
private const string StorageAccountKey  = "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ==";
```

[!INCLUDE [batch-credentials-include](../../includes/batch-credentials-include.md)]

此外，請確定方案中的 ffmpeg 應用程式套件參考符合您上傳至 Batch 帳戶的 ffmpeg 套件識別碼和版本。

```csharp
const string appPackageId = "ffmpeg";
const string appPackageVersion = "3.4";
```

### <a name="build-and-run-the-sample-project"></a>建置及執行範例專案

在 Visual Studio 中建置及執行應用程式，或是在命令列使用 `dotnet build` 和 `dotnet run` 命令。 執行此應用程式之後，檢閱程式碼以了解應用程式的每部分用途。 例如，在 Visual Studio 中：

* 在 [方案總管] 中以滑鼠右鍵按一下方案，然後按一下 [建置方案]。 

* 出現提示時，請確認任何 NuGet 封裝的還原。 如果您需要下載遺漏的套件，請確保已安裝 [NuGet 套件管理員](https://docs.nuget.org/consume/installing-nuget)。

然後加以執行。 當您執行範例應用程式時，主控台輸出大致如下。 在執行期間，集區的計算節點啟動後，您會在 `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` 遇到暫停。 

```
Sample start: 12/12/2017 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [WinFFmpegPool]...
Creating job [WinFFmpegJob]...
Adding 5 tasks to job [WinFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]...

Sample end: 12/12/2017 3:29:36 PM
Elapsed time: 00:09:14.3418742
```


移至 Azure 入口網站中您的 Batch 帳戶，以監視集區、計算節點、作業和工作。 例如，若要查看集區中計算節點的熱度圖，請按一下 [集區] > [WinFFmpegPool]。

當工作正在執行時，熱度圖會如下所示：

![集區熱度圖](./media/tutorial-parallel-dotnet/pool.png)


以預設設定執行應用程式時，一般的執行時間**大約 10 分鐘**。 建立集區佔用大部分的時間。

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>檢閱程式碼

後面各節將範例應用程式細分為用來處理 Batch 服務中工作負載的數個步驟。 在閱讀本文的其餘部分時，請參考解決方案中的 `Program.cs`，因為本文並不會討論範例中的每一行程式碼。

### <a name="authenticate-blob-and-batch-clients"></a>驗證 Blob 和 Batch 用戶端

為了與連結的儲存體帳戶進行互動，應用程式會使用適用於 .NET 的 Azure 儲存體用戶端程式庫。 其使用 [CloudStorageAccount](/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount) 建立帳戶的參考，並使用共用金鑰驗證進行驗證。 然後會建立 [CloudBlobClient](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobclient)。

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                                StorageAccountName, StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

應用程式會建立 [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) 物件，以在 Batch 服務中建立和管理集區、作業和工作。 範例中的 Batch 用戶端會使用共用金鑰驗證。 Batch 也支援透過 [Azure Active Directory](batch-aad-auth.md) 進行驗證，以便驗證個別使用者或自動執行的應用程式。

```csharp
BatchSharedKeyCredentials sharedKeyCredentials = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(sharedKeyCredentials))
...
```

### <a name="upload-input-files"></a>上傳輸入檔案

應用程式會將 `blobClient` 物件傳遞至 `CreateContainerIfNotExistAsync` 方法，以建立輸入檔案 (MP4 格式) 的儲存體容器和工作輸出的容器。

```csharp
CreateContainerIfNotExistAsync(blobClient, inputContainerName;
CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

接著，檔案會從本機 `InputFiles` 資料夾上傳至輸入容器。 儲存體中的檔案會定義為 Batch [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) 物件，Batch 之後可將這類物件下載到計算節點。 

上傳檔案時需要 `Program.cs` 中的兩個方法：

* `UploadResourceFilesToContainerAsync`：傳回 ResourceFile 物件的集合，並在內部呼叫 `UploadResourceFileToContainerAsync` 以上傳在 `inputFilePaths` 參數中傳入的每個檔案。
* `UploadResourceFileToContainerAsync`：將每個檔案當作 blob 上傳至輸入容器。 上傳檔案之後，它會取得此 blob 的共用存取簽章 (SAS) 並傳回代表它的 ResourceFile 物件。 

```csharp
string inputPath = Path.Combine(Environment.CurrentDirectory, "InputFiles");

List<string> inputFilePaths = new List<string>(Directory.GetFileSystemEntries(inputPath, "*.mp4",
    SearchOption.TopDirectoryOnly));

List<ResourceFile> inputFiles = await UploadResourceFilesToContainerAsync(
  blobClient,
  inputContainerName,
  inputFilePaths);
```

如需使用 .NET 將檔案當作 Blob 上傳至儲存體帳戶的詳細資訊，請參閱[使用 .NET 上傳、下載及列出 Blob](../storage/blobs/storage-quickstart-blobs-dotnet.md)。

### <a name="create-a-pool-of-compute-nodes"></a>建立計算節點的集區

接著，範例會呼叫 `CreatePoolIfNotExistAsync` 以在 Batch 帳戶中建立計算節點集區。 這個已定義的方法會使用 [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) 方法來設定節點數目、VM 大小和集區設定。 在此，[VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) 物件會將 [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) 指定至 Azure Marketplace 中發佈的 Windows Server 映像。 Batch 支援 Azure Marketplace 中各式各樣的 VM 映像，以及自訂 VM 映像。

使用已定義的常數可設定節點數目和 VM 大小。 Batch 支援專用節點和[低優先順序節點](batch-low-pri-vms.md)，而您可以在集區中使用其中一種或同時使用兩種。 專用節點會保留給您的集區使用。 低優先順序節點則會以較低的價格從 Azure 中的剩餘容量提供。 如果 Azure 沒有足夠的容量，便無法使用低優先順序節點。 此範例預設建立的集區只包含 5 個大小為 Standard_A1_v2 的低優先順序節點。 

將 [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) 新增到集區設定，ffmpeg 應用程式便會部署至計算節點。 

[CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) 方法會將集區提交至 Batch 服務。

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2012-R2-Datacenter-smalldisk",
    version: "latest");

VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.windows amd64");

pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: DedicatedNodeCount,
    targetLowPriorityComputeNodes: LowPriorityNodeCount,
    virtualMachineSize: PoolVMSize,                                                
    virtualMachineConfiguration: virtualMachineConfiguration);

pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
    new ApplicationPackageReference {
    ApplicationId = appPackageId,
    Version = appPackageVersion}};

await pool.CommitAsync();  
```

### <a name="create-a-job"></a>建立工作

Batch 工作會指定要在其中執行工作的集區及選擇性設定，例如工作的優先順序和排程。 此範例會藉由呼叫 `CreateJobAsync` 來建立作業。 這個已定義的方法會使用 [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) 方法在您的集區上建立作業。 

[CommitAsync](/dotnet/api/microsoft.azure.batch.cloudjob.commitasync) 方法會將作業提交至 Batch 服務。 一開始作業沒有任何工作。

```csharp
CloudJob job = batchClient.JobOperations.CreateJob();
job.Id = JobId;
job.PoolInformation = new PoolInformation { PoolId = PoolId };

await job.CommitAsync();
```

### <a name="create-tasks"></a>建立工作

此範例會藉由呼叫 `AddTasksAsync` 方法在作業中建立工作，而這個方法會建立一份 [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) 物件清單。 每項 `CloudTask` 都會使用 [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline) 屬數來執行 ffmpeg，以處理輸入 `ResourceFile` 物件。 ffmpeg 已在先前建立集區時安裝於每個節點上。 在此，命令列會執行 ffmpeg，將每個輸入 MP4 (影片) 檔案轉換為 MP3 (音訊) 檔案。

此範例會在執行命令列之後，為 MP3 檔案建立 [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) 物件。 每項工作的輸出檔案 (在此例中只有一個輸出檔案) 都會使用工作的 [OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles) 屬性，上傳至所連結儲存體帳戶中的容器。

然後，範例會使用 [AddTaskAsync](/dotnet/api/microsoft.azure.batch.joboperations.addtaskasync) 方法將工作新增至作業，該方法會將工作排入佇列以在計算節點上執行。 

```csharp
for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);

    // Define task command line to convert each input file.
    string appPath = String.Format("%AZ_BATCH_APP_PACKAGE_{0}#{1}%", appPackageId, appPackageVersion);
    string inputMediaFile = inputFiles[i].FilePath;
    string outputMediaFile = String.Format("{0}{1}",
        System.IO.Path.GetFileNameWithoutExtension(inputMediaFile),
        ".mp3");
    string taskCommandLine = String.Format("cmd /c {0}\\ffmpeg-3.4-win64-static\\bin\\ffmpeg.exe -i {1} {2}", appPath, inputMediaFile, outputMediaFile);

    // Create a cloud task (with the task ID and command line) 
    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };

    // Task output file
    List<OutputFile> outputFileList = new List<OutputFile>();
    OutputFileBlobContainerDestination outputContainer = new OutputFileBlobContainerDestination(outputContainerSasUrl);
    OutputFile outputFile = new OutputFile(outputMediaFile,
       new OutputFileDestination(outputContainer),
       new OutputFileUploadOptions(OutputFileUploadCondition.TaskSuccess));
    outputFileList.Add(outputFile);
    task.OutputFiles = outputFileList;
    tasks.Add(task);
}

// Add tasks as a collection
await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
return tasks
```

### <a name="monitor-tasks"></a>監視工作

當 Batch 將工作新增至作業時，此服務會自動將工作排入佇列並進行排程，以便在相關聯集區中的計算節點上執行。 根據您指定的設定，Batch 會處理所有工作佇列、排程、重試和其他工作管理責任。 

監視工作執行的方法有許多種。 此範例會定義 `MonitorTasks` 方法，而這個方法只會報告完成和工作失敗或成功的狀態。 `MonitorTasks` 程式碼會指定 [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel)，進而有效率地只選取最少工作相關資訊。 然後，它會建立 [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor)，以提供用來監視工作狀態的協助程式公用程式。 在 `MonitorTasks` 中，此範例會等候所有工作在某個時限內達到 `TaskState.Completed`。 然後它會終止作業，並回報任何已完成但可能發生失敗 (例如結束代碼不為零) 的工作。

```csharp
TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
try
{
    await taskStateMonitor.WhenAll(addedTasks, TaskState.Completed, timeout);
}
catch (TimeoutException)
{
    batchClient.JobOperations.TerminateJob(jobId);
    Console.WriteLine(incompleteMessage);
    return false;
}
batchClient.JobOperations.TerminateJob(jobId);
 Console.WriteLine(completeMessage);
...

```

## <a name="clean-up-resources"></a>清除資源

應用程式在執行工作之後，會自動刪除它所建立的輸入儲存體容器，並且為您提供用於刪除 Batch 集區和工作的選項。 BatchClient 的 [JobOperations](/dotnet/api/microsoft.azure.batch.batchclient.joboperations) 和 [PoolOperations](/dotnet/api/microsoft.azure.batch.batchclient.pooloperations) 類別都有對應的刪除方法 (在您確認刪除時呼叫)。 雖然您不需支付作業和工作本身的費用，但您需支付計算節點的費用。 因此，我們建議您只在必要時配置集區。 當您刪除集區時，節點上的所有工作輸出也會跟著刪除。 不過，輸出檔案會保留在儲存體帳戶中。

若不再需要，可刪除資源群組、Batch 帳戶和儲存體帳戶。 若要在 Azure 入口網站中這麼做，請選取 Batch 帳戶的資源群組，然後按一下 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 將應用程式套件新增至您的 Batch 帳戶
> * 驗證 Batch 和儲存體帳戶
> * 將輸入檔案上傳至儲存體
> * 建立計算節點的集區來執行應用程式
> * 建立作業和工作來處理輸入檔案
> * 監視工作執行
> * 擷取輸出檔案

如需更多使用 .NET API 來排程和處理 Batch 工作負載的範例，請參閱 GitHub 上的範例。

> [!div class="nextstepaction"]
> [Batch C# 範例](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp)
