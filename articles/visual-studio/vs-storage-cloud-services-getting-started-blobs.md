---
title: 開始使用 Blob 儲存體和 Visual Studio 已連接服務 (雲端服務) | Microsoft Docs
description: 在使用 Visual Studio 已連接服務連接到儲存體帳戶之後，如何於 Visual Studio 雲端服務專案中開始使用 Azure Blob 儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: 1144a958-f75a-4466-bb21-320b7ae8f304
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ms.openlocfilehash: 076dc58876afbe309e75373a78f84a79951ec2b9
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42146178"
---
# <a name="get-started-with-azure-blob-storage-and-visual-studio-connected-services-cloud-services-projects"></a>開始使用 Azure Blob 儲存體和 Visual Studio 已連接服務 (雲端服務專案)
[!INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

## <a name="overview"></a>概觀
本文描述如何在 Visual Studio 雲端服務專案中使用 [ **加入已連接服務** ] 對話方塊建立或參考 Azure 儲存體帳戶後，開始搭配使用 Azure Blob 儲存體。 我們會說明如何存取及建立 Blob 容器，以及如何執行上傳、列出和下載 Blob 等一般工作。 這些範例均以 C\# 撰寫，並使用 [Microsoft Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx)。

Azure 二進位大型物件 (Microsoft Azure Blob) 儲存是一項儲存大量非結構化資料的服務，全球任何地方都可透過 HTTP 或 HTTPS 來存取這些資料。 單一 Blob 可以是任何大小。 Blob 可以是影像、音訊和視訊檔、原始資料及文件檔案。

就像檔案在資料夾中一樣，儲存體 Blob 位於容器中。 在您已建立儲存體之後，您會在儲存體中建立一個或多個容器。 例如，在稱為 "Scrapbook" 的儲存體中，您可以在儲存體中建立稱為 "images" 的容器來儲存圖片，並建立稱為 "audio" 的容器來儲存音訊檔。 建立容器之後，就可以將個別的 Blob 檔案上傳至這些容器。

* 如需以程式設計方式處理 Blob 的詳細資訊，請參閱 [以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。
* 如需 Azure 儲存體的一般資訊，請參閱 [儲存體文件](https://azure.microsoft.com/documentation/services/storage/)。
* 如需 Azure 雲端服務的一般資訊，請參閱 [雲端服務文件](https://azure.microsoft.com/documentation/services/cloud-services/)。
* 如需 ASP.NET 應用程式設計的詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。

## <a name="access-blob-containers-in-code"></a>在程式碼中存取 blob 容器
若要以程式設計方式存取雲端服務中的 blob，您需要加入下列項目 (如果尚未存在)。

1. 將下列程式碼命名空間宣告，新增至您想要在其中以程式設計方式存取 Azure 儲存體之任何 C# 檔案內的頂端。
   
        using Microsoft.Framework.Configuration;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;
        using System.Threading.Tasks;
        using LogLevel = Microsoft.Framework.Logging.LogLevel;
2. 取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼，從 Azure 服務組態取得您的儲存體連接字串和儲存體帳戶資訊。
   
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("<storage account name>_AzureStorageConnectionString"));
3. 取得 **CloudBlobClient** 物件，來參考儲存體帳戶中的現有容器。
   
        // Create a blob client.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
4. 取得 **CloudBlobContainer** 物件，以參考特定的 Blob 容器。
   
        // Get a reference to a container named "mycontainer."
        CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

> [!NOTE]
> 請將前一個程序中顯示的所有程式碼使用於後續章節中顯示的程式碼之前。
> 
> 

## <a name="create-a-container-in-code"></a>在程式碼中建立容器
> [!NOTE]
> 有些對外向 ASP.NET 中 Azure 儲存體執行呼叫的 API 是非同步的。 如需詳細資訊，請參閱 [使用 Async 和 Await 進行非同步程式設計](http://msdn.microsoft.com/library/hh191443.aspx) 。 以下範例中的程式碼假設使用非同步程式設計方法。
> 
> 

若要在儲存體帳戶中建立容器，您只需要新增對 **CreateIfNotExistsAsync** 的呼叫，如下列程式碼所示：

    // If "mycontainer" doesn't exist, create it.
    await container.CreateIfNotExistsAsync();


若要讓所有人都能使用容器中的檔案，您可以使用下列程式碼將容器設定為公用容器。

    await container.SetPermissionsAsync(new BlobContainerPermissions
    {
        PublicAccess = BlobContainerPublicAccessType.Blob
    });


網際網路上的任何人都可以看到公用容器中的 Blob，但要有適當的存取金鑰，才能修改或刪除這些 Blob。

## <a name="upload-a-blob-into-a-container"></a>將 Blob 上傳至容器
Azure 儲存體支援區塊 Blob 和頁面 Blob。 在大多數情況下，建議使用區塊 Blob 的類型。

若要將檔案上傳至區塊 Blob，請取得容器參照，並使用該參照來取得區塊 Blob 參照。 擁有 Blob 參照後，即可藉由呼叫 **UploadFromStream** 方法，將任何資料流上傳至其中。 此操作會建立 Blob (如果其並不存在) 或覆寫 Blob (如果其已存在)。 下列範例顯示如何將 Blob 上傳到容器，並假設已建立該容器。

    // Retrieve a reference to a blob named "myblob".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

    // Create or overwrite the "myblob" blob with contents from a local file.
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
        blockBlob.UploadFromStream(fileStream);
    }

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob
若要列出容器中的 Blob，請先取得容器參照。 然後您即可使用容器的 **ListBlobs** 方法來擷取 Blob 和 (或) 其中的目錄。 若要針對傳回的 **IListBlobItem** 存取一組豐富的屬性與方法，您必須先將它轉換至 **CloudBlockBlob**、**CloudPageBlob** 或 **CloudBlobDirectory** 物件。 如果不清楚類型，可使用類型檢查來決定要將其轉換至何種類型。 下列程式碼示範如何擷取和輸出 **photos** 容器中每個項目的 URI：

    // Loop over items within the container and output the length and URI.
    foreach (IListBlobItem item in container.ListBlobs(null, false))
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

如先前程式碼範例所示，Blob 服務也具備容器中之目錄的概念。 正因如此，您能夠以更像資料夾的結構組織 Blob。 例如，假設名為 **photos**的容器中有下列一組區塊 Blob：

    photo1.jpg
    2010/architecture/description.txt
    2010/architecture/photo3.jpg
    2010/architecture/photo4.jpg
    2011/architecture/photo5.jpg
    2011/architecture/photo6.jpg
    2011/architecture/description.txt
    2011/photo7.jpg

當您在容器上呼叫 **ListBlobs** (如上一個範例所示) 時，傳回的集合將包含 **CloudBlobDirectory** 和 **CloudBlockBlob** 物件，其分別代表最上層所包含的目錄和 Blob。 以下是輸出結果：

    Directory: https://<accountname>.blob.core.windows.net/photos/2010/
    Directory: https://<accountname>.blob.core.windows.net/photos/2011/
    Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg


此外，您可以選擇將 **ListBlobs** 方法的 **UseFlatBlobListing** 參數設定為 **true**。 如此會導致不論目錄為何，都將每個 Blob 當成 **CloudBlockBlob**傳回。 以下是對 **ListBlobs**的呼叫︰

    // Loop over items within the container and output the length and URI.
    foreach (IListBlobItem item in container.ListBlobs(null, true))
    {
       ...
    }

其結果如下：

    Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2010/architecture/description.txt
    Block blob of length 314618: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo3.jpg
    Block blob of length 522713: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo4.jpg
    Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2011/architecture/description.txt
    Block blob of length 419048: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo5.jpg
    Block blob of length 506388: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo6.jpg
    Block blob of length 399751: https://<accountname>.blob.core.windows.net/photos/2011/photo7.jpg
    Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg

如需詳細資訊，請參閱 [CloudBlobContainer.ListBlobs](https://msdn.microsoft.com/library/azure/dd135734.aspx)。

## <a name="download-blobs"></a>下載 Blob
若要下載 Blob，請先擷取 Blob 參照，然後呼叫 **DownloadToStream** 方法。 下列範例使用 **DownloadToStream** 方法將 Blob 內容傳送給資料流物件，您接著可將該物件永久儲存成本機檔案。

    // Get a reference to a blob named "photo1.jpg".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

    // Save blob contents to a file.
    using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
    {
        blockBlob.DownloadToStream(fileStream);
    }

您也可以使用 **DownloadToStream** 方法，將 Blob 的內容當成文字字串下載。

    // Get a reference to a blob named "myblob.txt"
    CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

    string text;
    using (var memoryStream = new MemoryStream())
    {
        blockBlob2.DownloadToStream(memoryStream);
        text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
    }

## <a name="delete-blobs"></a>刪除 Blob
若要刪除 Blob，請先取得 Blob 參考，然後呼叫 **Delete** 方法。

    // Get a reference to a blob named "myblob.txt".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

    // Delete the blob.
    blockBlob.Delete();


## <a name="list-blobs-in-pages-asynchronously"></a>以非同步方式分頁列出 Blob
如果您要列出大量的 Blob，或是要控制在單一列出作業中傳回的結果數，您可以在結果頁面中列出 Blob。 此範例說明如何以非同步方式分頁傳回結果，使執行不會因為等待大型結果集傳回而中斷。

此範例說明一般 Blob 列出方式，但您也可以執行階層式清單，方法是將 **ListBlobsSegmentedAsync** 方法的 **useFlatBlobListing** 參數設為 **false**。

範例方法會呼叫非同步方法，因此前面必須加上 **async** 關鍵字，且必須傳回 **Task** 物件。 為 **ListBlobsSegmentedAsync** 方法指定的 await 關鍵字會擱置範例方法的執行，直到列出工作完成為止。

    async public static Task ListBlobsSegmentedInFlatListing(CloudBlobContainer container)
    {
        // List blobs to the console window, with paging.
        Console.WriteLine("List blobs in pages:");

        int i = 0;
        BlobContinuationToken continuationToken = null;
        BlobResultSegment resultSegment = null;

        // Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
        // When the continuation token is null, the last page has been returned and execution can exit the loop.
        do
        {
            // This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
            // or by calling a different overload.
            resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);
            if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
            foreach (var blobItem in resultSegment.Results)
            {
                Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
            }
            Console.WriteLine();

            //Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;
        }
        while (continuationToken != null);
    }

## <a name="next-steps"></a>後續步驟
[!INCLUDE [vs-storage-dotnet-blobs-next-steps](../../includes/vs-storage-dotnet-blobs-next-steps.md)]

