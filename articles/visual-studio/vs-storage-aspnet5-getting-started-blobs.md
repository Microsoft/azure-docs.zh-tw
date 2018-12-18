---
title: 開始使用 Blob 儲存體和 Visual Studio 已連接服務 (ASP.NET Core) | Microsoft Docs
description: 在使用 Visual Studio 已連接服務建立儲存體帳戶之後，如何在 Visual Studio ASP.NET Core 專案中開始使用 Azure Blob 儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: 094b596a-c92c-40c4-a0f5-86407ae79672
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 11/14/2017
ms.author: ghogen
ms.openlocfilehash: 566e2edc0157ccd02e0b44ae7df86c4b484858b0
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31793948"
---
# <a name="get-started-with-azure-blob-storage-and-visual-studio-connected-services-aspnet-core"></a>開始使用 Azure Blob 儲存體和 Visual Studio 已連接服務 (ASP.NET Core)

[!INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

本文說明如何在您使用 Visual Studio 的**已連線的服務**功能建立或參考 ASP.NET Core 專案中的 Azure 儲存體帳戶之後，開始在 Visual Studio 使用 Azure Blob 儲存體。 **已連線的服務**作業會安裝適當的 NuGet 套件，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案設定檔。 (如需 Azure 儲存體的一般資訊，請參閱[儲存體文件](https://azure.microsoft.com/documentation/services/storage/)。)

Azure 二進位大型物件 (Microsoft Azure Blob) 儲存是一項儲存大量非結構化資料的服務，全球任何地方都可透過 HTTP 或 HTTPS 來存取這些資料。 單一 Blob 可以是任何大小。 Blob 可以是影像、音訊和視訊檔、原始資料及文件檔案。 本文描述如何在您於 ASP.NET Core 專案中使用 Visual Studio 的**已連線的服務**建立 Azure 儲存體帳戶之後，開始使用 Blob 儲存體。

就像檔案在資料夾中一樣，儲存體 Blob 位於容器中。 在您已建立 Blob 之後，您會在 Blob 中建立一個或多個容器。 例如，在稱為 "Scrapbook" 的 Blob 中，您可以建立稱為 "images" 的容器來儲存圖片，並建立稱為 "audio" 的容器來儲存音訊檔。 建立容器之後，就可以將個別的檔案上傳至這些容器。 如需以程式設計方式處理 Blob 的詳細資訊，請參閱[快速入門：使用 .NET 上傳、下載及列出 Blob](../storage/blobs/storage-quickstart-blobs-dotnet.md)。

某些 Azure 儲存體 API 是非同步的，本文章中的程式碼假設我們正在使用非同步方法。 如需詳細資訊，請參閱[非同步程式設計](https://docs.microsoft.com/dotnet/csharp/async)。

## <a name="access-blob-containers-in-code"></a>在程式碼中存取 blob 容器

若要以程式設計方式存取 ASP.NET Core 專案中的 Blob，您需要加入下列程式碼 (若其不存在)：

1. 加入必要的 `using` 陳述式：

    ```cs
    using Microsoft.Extensions.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Extensions.Logging.LogLevel;
    ```

1. 取得代表儲存體帳戶資訊的 `CloudStorageAccount` 物件。 使用下列程式碼，從 Azure 服務組態取得您的儲存體連接字串和儲存體帳戶資訊：

    ```cs
     CloudStorageAccount storageAccount = new CloudStorageAccount(
        new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(
        "<storage-account-name>",
        "<access-key>"), true);
    ```

1. 使用 `CloudBlobClient` 物件來取得儲存體帳戶中現有容器的 `CloudBlobContainer` 參考：

    ```cs
    // Create a blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get a reference to a container named "mycontainer."
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    ```

## <a name="create-a-container-in-code"></a>在程式碼中建立容器

您也可以使用 `CloudBlobClient`，藉由呼叫 `CreateIfNotExistsAsync`在儲存體帳戶中建立容器：

```cs
// Create a blob client.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Get a reference to a container named "my-new-container."
CloudBlobContainer container = blobClient.GetContainerReference("my-new-container");

// If "mycontainer" doesn't exist, create it.
await container.CreateIfNotExistsAsync();
```

若要讓所有人都能使用容器中的檔案，請將容器設定為公用容器：

```cs
await container.SetPermissionsAsync(new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
});
```

## <a name="upload-a-blob-into-a-container"></a>將 Blob 上傳至容器

若要將 Blob 檔案上傳至容器，請取得容器參考，並使用該參考來取得 Blob 參考。 接著，呼叫 `UploadFromStreamAsync` 方法將所有資料串流上傳到該參考。 此作業會在 Blob 不存在的情況下建立 Blob，並覆寫現有的 Blob。 

```cs
// Get a reference to a blob named "myblob".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with the contents of a local file
// named "myfile".
using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

若要列出容器中的 Blob，請先取得容器參考，然後再呼叫 `ListBlobsSegmentedAsync` 方法來擷取其中的 Blob 和/或目錄。 若要針對傳回的 `IListBlobItem` 存取一組豐富屬性與方法，請將其轉換成 `CloudBlockBlob`、`CloudPageBlob` 或 `CloudBlobDirectory` 物件。 如果不知道 Blob 類型，可使用類型檢查來決定要將它轉換成何種類型。

```cs
BlobContinuationToken token = null;
do
{
    BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(token);
    token = resultSegment.ContinuationToken;

    foreach (IListBlobItem item in resultSegment.Results)
    {
        if (item.GetType() == typeof(CloudBlockBlob))
        {
            CloudBlockBlob blob = (CloudBlockBlob)item;
            Console.WriteLine("Block blob of length {0}: {1}", blob.Properties.Length, blob.Uri);
        }

        else if (item.GetType() == typeof(CloudPageBlob))
        {
            CloudPageBlob pageBlob = (CloudPageBlob)item;

            Console.WriteLine("Page blob of length {0}: {1}", pageBlob.Properties.Length, pageBlob.Uri);
        }

        else if (item.GetType() == typeof(CloudBlobDirectory))
        {
            CloudBlobDirectory directory = (CloudBlobDirectory)item;

            Console.WriteLine("Directory: {0}", directory.Uri);
        }
    }
} while (token != null);
```

如需列出 Blob 容器內容的其他方法，請參閱[快速入門：使用 .NET 上傳、下載及列出 Blob](../storage/blobs/storage-quickstart-blobs-dotnet.md#list-the-blobs-in-a-container)。

## <a name="download-a-blob"></a>下載 Blob

若要下載 Blob，請先取得 Blob 的參考，然後呼叫 `DownloadToStreamAsync` 方法。 下列範例會使用 `DownloadToStreamAsync` 方法，將 Blob 內容傳送給資料流物件，您接著可將該物件儲存為本機檔案。

```cs
// Get a reference to a blob named "photo1.jpg".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

// Save the blob contents to a file named "myfile".
using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
{
    await blockBlob.DownloadToStreamAsync(fileStream);
}
```

如需將 Blob 儲存為檔案的其他方法，請參閱[快速入門：使用 .NET 上傳、下載及列出 Blob](../storage/blobs/storage-quickstart-blobs-dotnet.md#download-blobs)。

## <a name="delete-a-blob"></a>刪除 Blob

若要刪除 Blob，請先取得 Blob 的參考，然後呼叫 `DeleteAsync` 方法：

```cs
// Get a reference to a blob named "myblob.txt".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

// Delete the blob.
await blockBlob.DeleteAsync();
```

## <a name="next-steps"></a>後續步驟

[!INCLUDE [vs-storage-dotnet-blobs-next-steps](../../includes/vs-storage-dotnet-blobs-next-steps.md)]
