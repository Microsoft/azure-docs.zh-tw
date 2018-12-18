---
title: 從 Azure 儲存體下載大量隨機資料 | Microsoft Docs
description: 了解如何使用 Azure SDK 從 Azure 儲存體帳戶下載大量隨機資料
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.custom: mvc
ms.component: blobs
ms.openlocfilehash: d71d09dde45897ad171109f6e091ae29c7cb91c2
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397274"
---
# <a name="download-large-amounts-of-random-data-from-azure-storage"></a>從 Azure 儲存體下載大量隨機資料

本教學課程是一個系列課程的第三部分。 本教學課程說明如何從 Azure 儲存體下載大量資料。

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 更新應用程式
> * 執行應用程式
> * 驗證連線數目

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須先完成上一個儲存體教學課程：[平行上傳大量隨機資料到 Azure 儲存體][previous-tutorial]。

## <a name="remote-into-your-virtual-machine"></a>遠端連線到您的虛擬機器

 若要與虛擬機器建立遠端桌面工作階段，請在您的本機電腦上使用下列命令。 以虛擬機器的公用 IP 位址取代 IP 位址。 出現提示時，請輸入您在建立虛擬機器時所使用的認證。

```
mstsc /v:<publicIpAddress>
```

## <a name="update-the-application"></a>更新應用程式

在上一個教學課程中，您只將檔案上傳到儲存體帳戶。 在文字編輯器中開啟 `D:\git\storage-dotnet-perf-scale-app\Program.cs`。 使用下列範例取代 `Main` 方法。 此範例已經將上傳工作變更為註解，並將下載工作及完成後刪除儲存體帳戶中內容的工作取消註解。

```csharp
public static void Main(string[] args)
{
    Console.WriteLine("Azure Blob storage performance and scalability sample");
    // Set threading and default connection limit to 100 to ensure multiple threads and connections can be opened.
    // This is in addition to parallelism with the storage client library that is defined in the functions below.
    ThreadPool.SetMinThreads(100, 4);
    ServicePointManager.DefaultConnectionLimit = 100; // (Or More)

    bool exception = false;
    try
    {
        // Call the UploadFilesAsync function.
        UploadFilesAsync().GetAwaiter().GetResult();

        // Uncomment the following line to enable downloading of files from the storage account.  This is commented out
        // initially to support the tutorial at https://docs.microsoft.com/azure/storage/blobs/storage-blob-scaleable-app-download-files.
        // DownloadFilesAsync().GetAwaiter().GetResult();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        exception = true;
    }
    finally
    {
        // The following function will delete the container and all files contained in them.  This is commented out initialy
        // As the tutorial at https://docs.microsoft.com/azure/storage/blobs/storage-blob-scaleable-app-download-files has you upload only for one tutorial and download for the other. 
        if (!exception)
        {
            // DeleteExistingContainersAsync().GetAwaiter().GetResult();
        }
        Console.WriteLine("Press any key to exit the application");
        Console.ReadKey();
    }
}
```

應用程式更新之後，您需要再次建置該應用程式。 開啟 `Command Prompt` 並瀏覽至 `D:\git\storage-dotnet-perf-scale-app`。 執行下列範例中的 `dotnet build`，以重建應用程式：

```
dotnet build
```

## <a name="run-the-application"></a>執行應用程式

應用程式重建完成之後，即能以更新過的程式碼執行應用程式。 請開啟 `Command Prompt` (若未開啟) 並瀏覽至 `D:\git\storage-dotnet-perf-scale-app`。

輸入 `dotnet run` 執行應用程式。

```
dotnet run
```

應用程式會讀取 **storageconnectionstring**中指定之儲存體帳戶內的容器。 它會使用容器中的 [ListBlobsSegmented](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.listblobssegmented?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobContainer_ListBlobsSegmented_System_String_System_Boolean_Microsoft_WindowsAzure_Storage_Blob_BlobListingDetails_System_Nullable_System_Int32__Microsoft_WindowsAzure_Storage_Blob_BlobContinuationToken_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_) 方法逐一查看 Blob 10，並使用 [DownloadToFileAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.downloadtofileasync?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlob_DownloadToFileAsync_System_String_System_IO_FileMode_Microsoft_WindowsAzure_Storage_AccessCondition_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_) 方法將它們下載到本機電腦。
下表顯示下載每個 Blob 時所定義的 [BlobRequestOptions](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions?view=azure-dotnet)。

|屬性|值|說明|
|---|---|---|
|[DisableContentMD5Validation](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.disablecontentmd5validation?view=azure-dotnet)| true| 此屬性可停用檢查上傳內容的 MD5 雜湊。 停用 MD5 驗證可獲得較快的傳輸速度。 但不會確認所傳輸檔案的有效性和完整性。 |
|[StorBlobContentMD5](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.storeblobcontentmd5?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_StoreBlobContentMD5)| false| 此屬性可判斷是否已計算及儲存 MD5 雜湊。   |

`DownloadFilesAsync` 工作如下列範例所示：

```csharp
private static async Task DownloadFilesAsync()
{
    CloudBlobClient blobClient = GetCloudBlobClient();

    // Define the BlobRequestionOptions on the download, including disabling MD5 hash validation for this example, this improves the download speed.
    BlobRequestOptions options = new BlobRequestOptions
    {
        DisableContentMD5Validation = true,
        StoreBlobContentMD5 = false
    };

    // Retrieve the list of containers in the storage account.  Create a directory and configure variables for use later.
    BlobContinuationToken continuationToken = null;
    List<CloudBlobContainer> containers = new List<CloudBlobContainer>();
    do
    {
        var listingResult = await blobClient.ListContainersSegmentedAsync(continuationToken);
        continuationToken = listingResult.ContinuationToken;
        containers.AddRange(listingResult.Results);
    }
    while (continuationToken != null);

    var directory = Directory.CreateDirectory("download");
    BlobResultSegment resultSegment = null;
    Stopwatch time = Stopwatch.StartNew();

    // Download the blobs
    try
    {
        List<Task> tasks = new List<Task>();
        int max_outstanding = 100;
        int completed_count = 0;

        // Create a new instance of the SemaphoreSlim class to define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        // Iterate through the containers
        foreach (CloudBlobContainer container in containers)
        {
            do
            {
                // Return the blobs from the container lazily 10 at a time.
                resultSegment = await container.ListBlobsSegmentedAsync(null, true, BlobListingDetails.All, 10, continuationToken, null, null);
                continuationToken = resultSegment.ContinuationToken;
                {
                    foreach (var blobItem in resultSegment.Results)
                    {

                        if (((CloudBlob)blobItem).Properties.BlobType == BlobType.BlockBlob)
                        {
                            // Get the blob and add a task to download the blob asynchronously from the storage account.
                            CloudBlockBlob blockBlob = container.GetBlockBlobReference(((CloudBlockBlob)blobItem).Name);
                            Console.WriteLine("Downloading {0} from container {1}", blockBlob.Name, container.Name);
                            await sem.WaitAsync();
                            tasks.Add(blockBlob.DownloadToFileAsync(directory.FullName + "\\" + blockBlob.Name, FileMode.Create, null, options, null).ContinueWith((t) =>
                            {
                                sem.Release();
                                Interlocked.Increment(ref completed_count);
                            }));

                        }
                    }
                }
            }
            while (continuationToken != null);
        }

        // Creates an asynchronous task that completes when all the downloads complete.
        await Task.WhenAll(tasks);
    }
    catch (Exception e)
    {
        Console.WriteLine("\nError encountered during transfer: {0}", e.Message);
    }

    time.Stop();
    Console.WriteLine("Download has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());
    Console.ReadLine();
}
```

### <a name="validate-the-connections"></a>驗證連線

當檔案正在下載時，您可以確認與您儲存體帳戶建立的並行連線的數目。 開啟 `Command Prompt` 並輸入 `netstat -a | find /c "blob:https"`。 此命令可顯示目前使用 `netstat` 開啟的連線數目。 下列範例顯示的輸出類似您自己執行教學課程時所見的輸出。 如範例中所見，從儲存體帳戶下載隨機檔案時開啟了超過 280 個連線。

```
C:\>netstat -a | find /c "blob:https"
289

C:\>
```

## <a name="next-steps"></a>後續步驟

在此系列的第三個部分中，您已學到從儲存體帳戶下載大量隨機資料，例如如何：

> [!div class="checklist"]
> * 執行應用程式
> * 驗證連線數目

前進到系列的第四部分，以在入口網站中驗證輸送量和延遲計量。

> [!div class="nextstepaction"]
> [在入口網站中驗證輸送量和延遲計量](storage-blob-scalable-app-verify-metrics.md)

[previous-tutorial]: storage-blob-scalable-app-upload-files.md