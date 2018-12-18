---
title: 使用 Azure Batch 服務 API 將作業和工作輸出保存到 Azure 儲存體 | Microsoft Docs
description: 了解如何使用 Batch 服務 API 將工作和作業輸出保存到 Azure 儲存體。
services: batch
author: dlepow
manager: jeconnoc
editor: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 06/16/2017
ms.author: danlep
ms.openlocfilehash: f562a6647cadbde6c46eba87b180dfb4cbb3fb90
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126307"
---
# <a name="persist-task-data-to-azure-storage-with-the-batch-service-api"></a>使用 Batch 服務 API 將工作資料保存到 Azure 儲存體

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

從 2017-05-01 版開始，Batch 服務 API 針對使用虛擬機器設定在集區上執行的工作和作業管理員工作，支援將輸出資料保存到 Azure 儲存體。 當您新增工作時，可以指定 Azure 儲存體中的容器作為工作輸出的目的地。 當工作完成時，Batch 服務就會將任何輸出資料寫入該容器中。

使用 Batch 服務 API 來保存工作輸出的好處，是您不需要修改工作正在執行的應用程式。 相反地，您可以對用戶端應用程式進行幾個簡單的修改，從建立工作的程式碼內保存工作的輸出。   

## <a name="when-do-i-use-the-batch-service-api-to-persist-task-output"></a>何時使用 Batch 服務 API 來保存工作輸出？

Azure Batch 提供多個方法來保存工作輸出。 使用 Batch 服務 API 是一個便利方式，最適合下列情節：

- 您需要撰寫程式碼，以從用戶端應用程式內保存工作輸出，而不需要修改您工作正在執行的應用程式。
- 您需要保存的輸出，是來自使用虛擬機器設定在集區中建立的 Batch 工作和作業管理員工作。
- 您需要將輸出保存到具有任意名稱的 Azure 儲存體容器。
- 您需要將輸出保存到根據 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions)命名的 Azure 儲存體容器。 

如果您的情節與以上所列不同，可能需要考慮不同的方法。 例如，Batch 服務 API 目前不支援在工作執行時將輸出串流至 Azure 儲存體。 若要將輸出串流，請考慮使用適用於 .NET 的 Batch 檔案慣例程式庫。 針對其他語言，您必須實作自己的解決方案。 如需保存工作輸出之其他選項的詳細資訊，請參閱[將作業和工作輸出保存到 Azure 儲存體](batch-task-output.md)。 

## <a name="create-a-container-in-azure-storage"></a>在 Azure 儲存體中建立容器

若要將工作輸出保存到 Azure 儲存體，您必須建立容器來作為輸出檔案的目的地。 在執行工作之前，最好是在提交作業之前建立容器。 若要建立容器，請使用適當的 Azure 儲存體用戶端程式庫或 SDK。 如需有關「Azure 儲存體 API」的資訊，請參閱 [Azure 儲存體文件](https://docs.microsoft.com/azure/storage/)。

例如，如果您要以 C# 撰寫應用程式，請使用[適用於 .NET 的 Azure 儲存體用戶端程式庫](https://www.nuget.org/packages/WindowsAzure.Storage/)。 下列範例說明如何建立容器：

```csharp
CloudBlobContainer container = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
await conainer.CreateIfNotExists();
```

## <a name="get-a-shared-access-signature-for-the-container"></a>取得容器的共用存取簽章

建立容器之後，可使用寫入容器的存取權取得共用的存取簽章 (SAS)。 SAS 會提供委派存取權給容器。 SAS 會使用一組指定的權限，以及一段指定的時間間隔來授與存取權。 Batch 服務需要具有寫入權限的 SAS，才能將工作輸出寫入容器中。 如需關於 SAS 的詳細資訊，請參閱[在 Azure 儲存體中使用共用存取簽章 \(SAS\)](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

當您使用 Azure 儲存體 API 取得 SAS 時，API 會傳回 SAS 權杖字串。 這個權杖字串會包含 SAS 的所有參數，包括權限和 SAS 有效的時間間隔。 若要使用 SAS 存取 Azure 儲存體中的容器，您必須將 SAS 權杖字串附加至資源 URI。 資源的 URI 以及附加的 SAS 權杖會共同提供 Azure 儲存體的已驗證存取。

下列範例說明如何取得容器的唯寫 SAS 權杖字串，然後會將 SAS 附加至容器 URI：

```csharp
string containerSasToken = container.GetSharedAccessSignature(new SharedAccessBlobPolicy()
{
    SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddDays(1),
    Permissions = SharedAccessBlobPermissions.Write
});

string containerSasUrl = container.Uri.AbsoluteUri + containerSasToken; 
```

## <a name="specify-output-files-for-task-output"></a>指定工作輸出的輸出檔案

若要指定工作的輸出檔案，請建立 [OutputFile](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfile) 物件的集合，並當您建立這項工作時，將它指派給 [CloudTask.OutputFiles](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles#Microsoft_Azure_Batch_CloudTask_OutputFiles) 屬性。 

下列 .NET 程式碼範例所建立的工作，會將隨機數字寫入名為 `output.txt` 的檔案。 此範例會建立要寫入容器中的 `output.txt` 之輸出檔案。 此範例也會建立任何比對檔案模式之記錄檔的輸出檔案`std*.txt` (_例如_ 、`stdout.txt` 和 `stderr.txt`)。 容器 URL 需要先前針對容器建立的 SAS。 Batch 服務會使用 SAS 來驗證容器的存取權： 

```csharp
new CloudTask(taskId, "cmd /v:ON /c \"echo off && set && (FOR /L %i IN (1,1,100000) DO (ECHO !RANDOM!)) > output.txt\"")
{
    OutputFiles = new List<OutputFile>
    {
        new OutputFile(
            filePattern: @"..\std*.txt",
            destination: new OutputFileDestination(
         new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId)),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
        new OutputFile(
            filePattern: @"output.txt",
            destination: 
         new OutputFileDestination(new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId + @"\output.txt")),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
}
```

### <a name="specify-a-file-pattern-for-matching"></a>指定要進行比對的檔案模式

當您指定輸出檔案時，可以使用 [OutputFile.FilePattern](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfile.filepattern#Microsoft_Azure_Batch_OutputFile_FilePattern) 屬性來指定要進行比對的檔案模式。 檔案模式可能會比對零個檔案、單一檔案或工作所建立的一組檔案。

**FilePattern** 屬性會支援標準檔案系統萬用字元，例如 `*` (適用於非遞迴比對) 和 `**` (適用於遞迴比對)。 例如，上述程式碼範例會指定檔案模式以非遞迴的方式比對 `std*.txt`： 

`filePattern: @"..\std*.txt"`

若要上傳單一檔案，請指定不包含萬用字元的檔案模式。 例如，上述程式碼範例會指定檔案模式比對 `output.txt`：

`filePattern: @"output.txt"`

### <a name="specify-an-upload-condition"></a>指定上傳條件

[OutputFileUploadOptions.UploadCondition](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileuploadoptions.uploadcondition#Microsoft_Azure_Batch_OutputFileUploadOptions_UploadCondition) 屬性允許條件式上傳輸出檔案。 常見的情節是，如果工作成功就上傳一組檔案，如果失敗就上傳一組不同的檔案。 例如，只有在工作失敗且以非零結束代碼結束時，您才需要將詳細資訊記錄檔上傳。 同樣地，只有當工作成功時，您才需要將結果檔案上載，因為如果工作失敗，這些檔案可能就會遺失或不完整。

上述程式碼範例會將 **UploadCondition** 屬性設為 **TaskCompletion**。 此設定會指定在工作完成之後才將檔案上傳，無論結束代碼的值為何。 

`uploadCondition: OutputFileUploadCondition.TaskCompletion`

如需其他設定，請參閱 [OutputFileUploadCondition](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.common.outputfileuploadcondition) 列舉。

### <a name="disambiguate-files-with-the-same-name"></a>釐清具有相同名稱的檔案

作業中的工作可能會產生具有相同名稱的檔案。 例如，`stdout.txt` 和 `stderr.txt` 是針對作業中執行的每項工作所建立。 每項工作都會在自己的內容中執行，因此這些檔案在節點的檔案系統上不會產生衝突。 不過，當您從多個工作將檔案上傳至共用的容器時，必須將具有相同名稱的檔案加以釐清。

[OutputFileBlobContainerDestination.Path](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination.path#Microsoft_Azure_Batch_OutputFileBlobContainerDestination_Path) 屬性會指定目的地 blob 或輸出檔案的虛擬目錄。 您可以使用 **Path** 屬性來命名 blob 或虛擬目錄，方法是將具有相同名稱的輸出檔案在 Azure 儲存體中以唯一方式命名。 在路徑中使用工作識別碼，是確保唯一名稱並可輕鬆識別檔案的好方法。

如果 **FilePattern** 屬性設為萬用字元運算式，則比對模式的所有檔案都會上傳到 **Path** 屬性所指定的虛擬目錄中。 例如，如果容器是 `mycontainer`、工作識別碼是 `mytask`，以及檔案模式是 `..\std*.txt`，Azure 儲存體中輸出檔案的絕對 URI 就會類似於：

```
https://myaccount.blob.core.windows.net/mycontainer/mytask/stderr.txt
https://myaccount.blob.core.windows.net/mycontainer/mytask/stdout.txt
```

如果將 **FilePattern** 屬性設定為符合單一檔案名稱，表示它不包含任何萬用字元，**Path** 屬性的值就會指定完整的 blob 名稱。 如果您預期會與多個工作的單一檔案發生命名衝突，則可包含虛擬目錄的名稱作為檔案名稱的一部分，以釐清這些檔案。 例如，設定 **Path** 屬性，可包括工作識別碼、分隔符號字元 (通常是正斜線)，以及檔案名稱：

`path: taskId + @"/output.txt"`

一組工作之輸出檔案的絕對 URI 將類似於：

```
https://myaccount.blob.core.windows.net/mycontainer/task1/output.txt
https://myaccount.blob.core.windows.net/mycontainer/task2/output.txt
```

如需有關 Azure 儲存體中虛擬目錄的詳細資訊，請參閱[列出容器中的 blob](../storage/blobs/storage-quickstart-blobs-dotnet.md#list-the-blobs-in-a-container)。


## <a name="diagnose-file-upload-errors"></a>診斷檔案上傳錯誤

如果將輸出檔案上傳至 Azure 儲存體失敗，工作就會移至「**已完成**」狀態，且會設定 [TaskExecutionInformation.FailureInformation](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.taskexecutioninformation.failureinformation#Microsoft_Azure_Batch_TaskExecutionInformation_FailureInformation) 屬性。 檢查 **FailureInformation** 屬性來判斷所發生的錯誤。 例如，如果找不到容器，就會在檔案上傳時發生以下錯誤： 

```
Category: UserError
Code: FileUploadContainerNotFound
Message: One of the specified Azure container(s) was not found while attempting to upload an output file
```

在每次檔案上傳時，Batch 會將 `fileuploadout.txt` 和 `fileuploaderr.txt` 兩個記錄檔寫入計算節點中。 若要進一步了解特定錯誤，您可以檢查這些記錄檔。 在從未嘗試檔案上傳的案例中 (例如因為工作本身無法執行)，這些記錄檔就不會存在。

## <a name="diagnose-file-upload-performance"></a>診斷檔案上傳效能

`fileuploadout.txt` 檔案會記錄上傳進度。 您可以檢查這個檔案，進一步了解您檔案上傳所需的時間。 請注意，上傳效能有許多促成因素，包括節點大小、上傳時節點上的其他活動、目標容器是否在相同區域中作為 Batch 集區、同時上傳至儲存體帳戶的節點數等等。

## <a name="use-the-batch-service-api-with-the-batch-file-conventions-standard"></a>使用 Batch 服務 API 搭配 Batch 檔案慣例標準

當您使用 Batch 服務 API 來保存工作輸出時，可以隨您所好命名您的目的地容器和 blob。 您也可以選擇根據 [Batch 檔案慣例標準](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/Batch/Support/FileConventions#conventions) 將它們命名。 檔案慣例標準會以作業和工作名稱作為基礎，針對給定之輸出檔案，決定 Azure 儲存體中的目的地容器和 blob 的名稱。 如果您要針對命名輸出檔案使用檔案慣例標準，您的輸出檔案就可在 [Azure 入口網站](https://portal.azure.com)中檢視。

如果您要以 C# 進行開發，可以使用[適用於 .NET 的 Batch 檔案慣例程式庫](https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files)的內建方法。 此程式庫會為您建立正確命名的容器和 blob 路徑。 例如，您可以作業名稱作為基礎呼叫 API，來取得容器的正確名稱：

```csharp
string containerName = job.OutputStorageContainerName();
```

您可以使用 [CloudJobExtensions.GetOutputStorageContainerUrl](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.getoutputstoragecontainerurl) 方法，將用來寫入容器的共用存取簽章 (SAS) URL 傳回。 然後，您可以將此 SAS 傳遞到 [OutputFileBlobContainerDestination](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination) 建構函式。

如果您要以 C# 以外的語言進行開發，必須自行實作檔案慣例標準。

## <a name="code-sample"></a>程式碼範例

[PersistOutputs][github_persistoutputs] 範例專案是 GitHub 上的其中一個 [Azure Batch 程式碼範例][github_samples]。 此 Visual Studio 解決方案示範如何使用適用於 .NET 的 Batch 用戶端程式庫，將工作輸出保存到永久性儲存體。 若要執行範例，請遵循下列步驟：

1. 在 **Visual Studio 2017** 中開啟專案。
2. 將您 Batch 和儲存體的**帳戶認證**新增到 Microsoft.Azure.Batch.Samples.Common 專案中的 **AccountSettings.settings**。
3.  (但不要執行) 該解決方案。 如果出現提示，請還原任何 NuGet 封裝。
4. 使用 Azure 入口網站來為 [PersistOutputsTask](batch-application-packages.md) 上傳 **應用程式封裝**。 將 `PersistOutputsTask.exe` 及其相依性組件包含在 .zip 封裝中，將應用程式識別碼和應用程式封裝版本分別設為 "PersistOutputsTask" 和 "1.0"。
5. **啟動** (執行) **PersistOutputs** 專案。
6. 當系統提示您選擇要用於執行範例的持續性技術時，輸入 **2** 可使用 Batch 服務 API 保存工作輸出來執行範例。
7. 如需要，請再次執行範例，輸入 **3** 可使用 Batch 服務 API 保存輸出，並根據檔案慣例標準命名目的地容器和 blob 路徑。

## <a name="next-steps"></a>後續步驟

- 如需使用適用於 .NET 的檔案慣例程式庫來保存工作輸出的詳細資訊，請參閱[使用適用於 .NET 的 Batch 檔案慣例程式庫，將作業和工作資料保存到 Azure 儲存體](batch-task-output-file-conventions.md)。
- 如需在 Azure Batch 中保存輸出資料之其他方法的相關資訊，請參閱[將作業和工作輸出保存到 Azure 儲存體](batch-task-output.md)。

[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
