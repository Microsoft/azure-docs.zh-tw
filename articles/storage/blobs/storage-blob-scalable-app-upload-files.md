---
title: 將大量隨機資料平行上傳至 Azure 儲存體 | Microsoft Docs
description: 了解如何使用 Azure SDK 將大量隨機資料平行上傳至 Azure 儲存體帳戶
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.custom: mvc
ms.component: blobs
ms.openlocfilehash: 557dd1d89fc05d82f1839a7b02356857f41164c6
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39399731"
---
# <a name="upload-large-amounts-of-random-data-in-parallel-to-azure-storage"></a>將大量隨機資料平行上傳至 Azure 儲存體

本教學課程是一個系列的第二部分。 本教學課程會示範如何部署應用程式，來將大量的隨機資料上傳至 Azure 儲存體帳戶。

在本系列的第二部分中，您將瞭解如何：

> [!div class="checklist"]
> * 設定連接字串
> * 建置應用程式
> * 執行應用程式
> * 驗證連線數目

Azure Blob 儲存體會提供可擴充的服務來儲存您的資料。 若要確保您的應用程式能發揮最好的效能，建議您了解 Blob 儲存體的運作方式。 關於 Azure Blob 限制的知識十分重要，若要深入了解這些限制，請造訪：[Blob 儲存體的延展性目標](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#azure-blob-storage-scale-targets)。

使用 Blob 設計高度執行的應用程式時，[分割區命名](../common/storage-performance-checklist.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#subheading47)是另一個重要的要素。 Azure 儲存體使用範圍型的資料分割配置來進行縮放和負載平衡。 此設定表示具有相似命名慣例或前置詞的檔案會進入相同分割區。 此邏輯也包含將對其上傳檔案的容器名稱。 在本教學課程中，您會使用以 GUID 命名的檔案和隨機產生的內容。 然後這些項目會上傳至五個具有隨機名稱的不同容器。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須先完成上一個儲存體教學課程：[為可擴充的應用程式建立虛擬機器和儲存體帳戶][previous-tutorial]。

## <a name="remote-into-your-virtual-machine"></a>遠端連線到您的虛擬機器

在本機電腦上使用下列命令，建立使用虛擬機器的遠端桌面工作階段。 以虛擬機器的公用 IP 位址取代 IP 位址。 出現提示時，請輸入您在建立虛擬機器時所使用的認證。

```
mstsc /v:<publicIpAddress>
```

## <a name="configure-the-connection-string"></a>設定連接字串

在 Azure 入口網站中，瀏覽至您的儲存體帳戶。 在儲存體帳戶中選取 [設定] 下的 [存取金鑰]。 從主要或次要金鑰複製**連接字串**。 登入先前教學課程中所建立的虛擬機器。 以系統管理員身分開啟 [命令提示字元]，並以 `/m` 執行 `setx` 命令，此命令會儲存機器的設定環境變數。 您須重新載入**命令提示字元**，才能使用環境變數。 取代下列範例中的 **\<storageConnectionString\>**：

```
setx storageconnectionstring "<storageConnectionString>" /m
```

完成後，請開啟另一個**命令提示字元**，並瀏覽至 `D:\git\storage-dotnet-perf-scale-app`，然後輸入 `dotnet build` 來建置應用程式。

## <a name="run-the-application"></a>執行應用程式

瀏覽至 `D:\git\storage-dotnet-perf-scale-app`。

輸入 `dotnet run` 執行應用程式。 第一次執行 `dotnet` 時，它會填入您的本機套件快取，以提升還原速度及啟用離線存取。 完成此命令需一分鐘的時間，且只會發生一次。

```
dotnet run
```

應用程式會建立五個隨機命名的容器，並開始將暫存目錄中的檔案上傳至儲存體帳戶。 應用程式會將最小執行緒設為 100，以及將 [DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit(v=vs.110).aspx) 設為 100，確保執行應用程式時能允許大量的並行連線。

除了設定執行緒和連線限制設定，[UploadFromStreamAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblockblob.uploadfromstreamasync?view=azure-dotnet) 的 [BlobRequestOptions](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions?view=azure-dotnet) 方法也會設定為使用平行處理原則，並停用 MD5 雜湊驗證。 檔案會以 100 MB 的區塊上傳，這項設定可提供更佳的效能，但如果使用效能差的網路則可能提高成本，因為整個 100 MB 的區塊可能會因為失敗而需要重試。

|屬性|值|說明|
|---|---|---|
|[ParallelOperationThreadCount](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.paralleloperationthreadcount?view=azure-dotnet)| 8| 上傳時，此設定會將 Blob 分成區塊。 為達到最高效能，此值應為核心數目的 8 倍。 |
|[DisableContentMD5Validation](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.disablecontentmd5validation?view=azure-dotnet)| true| 此屬性可停用檢查上傳內容的 MD5 雜湊。 停用 MD5 驗證可獲得較快的傳輸速度。 但不會確認所傳輸檔案的有效性和完整性。   |
|[StoreBlobContentMD5](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.storeblobcontentmd5?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_StoreBlobContentMD5)| false| 此屬性可判斷是否已計算及儲存檔案的 MD5 雜湊。   |
| [RetryPolicy](/dotnet/api/microsoft.windowsazure.storage.blob.blobrequestoptions.retrypolicy?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_RetryPolicy)| 具有 2 秒的輪詢，最多重試 10 次 |決定要求的重試原則。 連線失敗時進行重試，在此範例中，[ExponentialRetry](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.exponentialretry?view=azure-dotnet) 原則設定為具有 2 秒的輪詢，以及最多重試 10 次。 當您的應用程式快達到 [Blob 儲存體的延展性目標](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#azure-blob-storage-scale-targets)時，此設定就十分重要。  |

`UploadFilesAsync` 工作如下列範例所示：

```csharp
private static async Task UploadFilesAsync()
{
    // Create random 5 characters containers to upload files to.
    CloudBlobContainer[] containers = await GetRandomContainersAsync();
    var currentdir = System.IO.Directory.GetCurrentDirectory();

    // path to the directory to upload
    string uploadPath = currentdir + "\\upload";
    Stopwatch time = Stopwatch.StartNew();
    try
    {
        Console.WriteLine("Iterating in directory: {0}", uploadPath);
        int count = 0;
        int max_outstanding = 100;
        int completed_count = 0;

        // Define the BlobRequestionOptions on the upload.
        // This includes defining an exponential retry policy to ensure that failed connections are retried with a backoff policy. As multiple large files are being uploaded
        // large block sizes this can cause an issue if an exponential retry policy is not defined.  Additionally parallel operations are enabled with a thread count of 8
        // This could be should be multiple of the number of cores that the machine has. Lastly MD5 hash validation is disabled for this example, this improves the upload speed.
        BlobRequestOptions options = new BlobRequestOptions
        {
            ParallelOperationThreadCount = 8,
            DisableContentMD5Validation = true,
            StoreBlobContentMD5 = false
        };
        // Create a new instance of the SemaphoreSlim class to define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        List<Task> tasks = new List<Task>();
        Console.WriteLine("Found {0} file(s)", Directory.GetFiles(uploadPath).Count());

        // Iterate through the files
        foreach (string path in Directory.GetFiles(uploadPath))
        {
            // Create random file names and set the block size that is used for the upload.
            var container = containers[count % 5];
            string fileName = Path.GetFileName(path);
            Console.WriteLine("Uploading {0} to container {1}.", path, container.Name);
            CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

            // Set block size to 100MB.
            blockBlob.StreamWriteSizeInBytes = 100 * 1024 * 1024;
            await sem.WaitAsync();

            // Create tasks for each file that is uploaded. This is added to a collection that executes them all asyncronously.  
            tasks.Add(blockBlob.UploadFromFileAsync(path, null, options, null).ContinueWith((t) =>
            {
                sem.Release();
                Interlocked.Increment(ref completed_count);
            }));
            count++;
        }

        // Creates an asynchronous task that completes when all the uploads complete.
        await Task.WhenAll(tasks);

        time.Stop();

        Console.WriteLine("Upload has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());

        Console.ReadLine();
    }
    catch (DirectoryNotFoundException ex)
    {
        Console.WriteLine("Error parsing files in the directory: {0}", ex.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

下列範例是 Windows 系統上執行的應用程式輸出 (已截斷)。

```
Created container https://mystorageaccount.blob.core.windows.net/9efa7ecb-2b24-49ff-8e5b-1d25e5481076
Created container https://mystorageaccount.blob.core.windows.net/bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b
Created container https://mystorageaccount.blob.core.windows.net/9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f
Created container https://mystorageaccount.blob.core.windows.net/47646f1a-c498-40cd-9dae-840f46072180
Created container https://mystorageaccount.blob.core.windows.net/38b2cdab-45fa-4cf9-94e7-d533837365aa
Iterating in directory: D:\git\storage-dotnet-perf-scale-app\upload
Found 50 file(s)
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\1d596d16-f6de-4c4c-8058-50ebd8141e4d.txt to container 9efa7ecb-2b24-49ff-8e5b-1d25e5481076.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\242ff392-78be-41fb-b9d4-aee8152a6279.txt to container bbe5f0c8-be9e-4fc3-bcbd-2092433dbf6b.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\38d4d7e2-acb4-4efc-ba39-f9611d0d55ef.txt to container 9ac2f71c-6b44-40e7-b7be-8519d3ba4e8f.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\45930d63-b0d0-425f-a766-cda27ff00d32.txt to container 47646f1a-c498-40cd-9dae-840f46072180.
Starting upload of D:\git\storage-dotnet-perf-scale-app\upload\5129b385-5781-43be-8bac-e2fbb7d2bd82.txt to container 38b2cdab-45fa-4cf9-94e7-d533837365aa.
...
Upload has been completed in 142.0429536 seconds. Press any key to continue
```

### <a name="validate-the-connections"></a>驗證連線

當檔案正在上傳時，您可以確認與您儲存體帳戶建立的並行連線數目。 開啟**命令提示字元**，然後輸入 `netstat -a | find /c "blob:https"`。 此命令可顯示目前使用 `netstat` 開啟的連線數目。 下列範例顯示的輸出類似您自己執行教學課程時所見的輸出。 如範例中所見，將隨機檔案上傳至儲存體帳戶時時開啟了 800 個連線。 此值會在整個上傳作業期間變更。 藉由以區塊的形式來平行上傳，就可大幅減少傳送內容所需的時間。

```
C:\>netstat -a | find /c "blob:https"
800

C:\>
```

## <a name="next-steps"></a>後續步驟

在此系列的第二個部分中，您已學到如何將大量隨機資料平行上傳至儲存體帳戶，例如如何：

> [!div class="checklist"]
> * 設定連接字串
> * 建置應用程式
> * 執行應用程式
> * 驗證連線數目

接著請前往此系列的第三部分，從儲存體帳戶下載大量的資料。

> [!div class="nextstepaction"]
> [將大量大型檔案平行上傳至儲存體帳戶](storage-blob-scalable-app-download-files.md)

[previous-tutorial]: storage-blob-scalable-app-create-vm.md
