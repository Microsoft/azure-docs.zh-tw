---
title: 開始使用 Azure Blob 儲存體和 Visual Studio 已連線服務 (ASP.NET) | Microsoft Docs
description: 在使用 Visual Studio 已連線服務連線到儲存體帳戶之後，如何於 Visual Studio 的 ASP.NET 專案中開始使用 Azure Blob 儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: b3497055-bef8-4c95-8567-181556b50d95
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/07/2017
ms.author: ghogen
ms.openlocfilehash: b8c0891d8528e0622682788e5b10d5a76ad92ec8
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141095"
---
# <a name="get-started-with-azure-blob-storage-and-visual-studio-connected-services-aspnet"></a>開始使用 Azure Blob 儲存體和 Visual Studio 已連線服務 (ASP.NET)

> [!div class="op_single_selector"]
> - [ASP.NET](./vs-storage-aspnet-getting-started-blobs.md)
> - [ASP.NET Core](./vs-storage-aspnet-core-getting-started-blobs.md)

Azure Blob 儲存體是可將非結構化資料儲存在雲端作為物件或 Blob 的服務。 Blob 儲存體可以儲存任何類型的文字或二進位資料，例如文件、媒體檔案或應用程式安裝程式。 Blob 儲存體也稱為物件儲存體。

本教學課程說明如何為一些使用 Blob 儲存體的常見案例撰寫 ASP.NET 程式碼。 案例包括建立 blob 容器，以及上傳、列出、下載和刪除 blob。

[!INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

## <a name="prerequisites"></a>必要條件

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)

[!INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]


[!INCLUDE [storage-development-environment-include](../../includes/vs-storage-aspnet-getting-started-setup-dev-env.md)]

## <a name="create-an-mvc-controller"></a>建立 MVC 控制器 

1. 在 [方案總管] 中，於 [控制器] 上按一下滑鼠右鍵。

2. 從快顯功能表中，選取 [新增] > [控制器]。

    ![方案總管的螢幕擷取畫面，並反白顯示 [新增] 與 [控制站]](./media/vs-storage-aspnet-getting-started-blobs/add-controller-menu.png)

1. 在 [新增 Scaffold] 對話方塊中，選取 [MVC 5 控制器 - 空白]，然後選取 [新增]。

    ![[新增 Scaffold] 對話方塊的螢幕擷取畫面](./media/vs-storage-aspnet-getting-started-blobs/add-controller.png)

1. 在 [新增控制器] 對話方塊中，將控制器命名為 BlobsController，然後選取 [新增]。

    ![[新增控制器] 對話方塊的螢幕擷取畫面](./media/vs-storage-aspnet-getting-started-blobs/add-controller-name.png)

1. 將下列 `using` 指示詞新增至 `BlobsController.cs` 檔案：

    ```csharp
    using Microsoft.Azure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```

## <a name="connect-to-a-storage-account-and-get-a-container-reference"></a>連接至儲存體帳戶並取得容器參考

Blob 容器是 blob 和資料夾的巢狀階層。 本文件中剩餘的步驟需要對 Blob 容器的參考，因此應該將程式碼放在其自己的方法中以供重複使用。

下列步驟會使用 **Web.config** 中的連接字串來建立方法，以連線至儲存體帳戶。這些步驟也會建立對容器的參考。  **Web.config** 中的連接字串會以 `<storageaccountname>_AzureStorageConnectionString` 格式命名。 

1. 開啟 `BlobsController.cs` 檔案。

1. 新增名為 **GetCloudBlobContainer** 且會傳回 **CloudBlobContainer** 的方法。  請務必以 **Web.config** 中金鑰的實際名稱取代 `<storageaccountname>_AzureStorageConnectionString`。
    
    ```csharp
    private CloudBlobContainer GetCloudBlobContainer()
    {
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
                CloudConfigurationManager.GetSetting("<storageaccountname>_AzureStorageConnectionString"));
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
        CloudBlobContainer container = blobClient.GetContainerReference("test-blob-container");
        return container;
    }
    ```

> [!NOTE]
> 即使 test-blob-container 尚不存在，此程式碼也會建立對它的參考。 如此一來，才能使用下一個步驟中所示的 `CreateIfNotExists` 方法來建立容器。

## <a name="create-a-blob-container"></a>建立 Blob 容器

下列步驟說明如何建立 blob 容器：

1. 新增名為 `CreateBlobContainer` 且會傳回 `ActionResult` 的方法。

    ```csharp
    public ActionResult CreateBlobContainer()
    {
        // The code in this section goes here.

        return View();
    }
    ```
 
1. 取得 `CloudBlobContainer` 物件，此物件代表對所需 Blob 容器名稱的參考。 
   
    ```csharp
    CloudBlobContainer container = GetCloudBlobContainer();
    ```

1. 呼叫 `CloudBlobContainer.CreateIfNotExists` 方法來建立容器 (如果尚不存在)。 如果容器不存在且已成功建立，則 `CloudBlobContainer.CreateIfNotExists` 方法會傳回 **true**。 否則，此方法會傳回 **false**。    

    ```csharp
    ViewBag.Success = container.CreateIfNotExists();
    ```

1. 使用 Blob 容器的名稱來更新 `ViewBag`。

    ```csharp
    ViewBag.BlobContainerName = container.Name;
    ```
    
    以下顯示已完成的 `CreateBlobContainer` 方法：

    ```csharp
    public ActionResult CreateBlobContainer()
    {
        CloudBlobContainer container = GetCloudBlobContainer();
        ViewBag.Success = container.CreateIfNotExists();
        ViewBag.BlobContainerName = container.Name;

        return View();
    }
    ```

1. 在 [方案總管] 中，於 [檢視] 資料夾上按一下滑鼠右鍵。

1. 如果沒有 **Blob** 資料夾，請建立一個。 從快顯功能表中，選取 [新增] > [新增資料夾]。 將新資料夾命名為 Blobs。 
 
1. 在 [方案總管] 中，展開 [檢視] 資料夾，然後在 [Blobs] 上按一下滑鼠右鍵。

1. 從快顯功能表中，選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 **CreateBlobContainer**，然後選取 [新增]。

1. 開啟 `CreateBlobContainer.cshtml` 並加以修改，以便其如下列程式碼片段所示：

    ```csharp
    @{
        ViewBag.Title = "Create Blob Container";
    }
    
    <h2>Create Blob Container results</h2>

    Creation of @ViewBag.BlobContainerName @(ViewBag.Success == true ? "succeeded" : "failed")
    ```

1. 在 [方案總管] 中，展開 [檢視] > [共用] 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Create blob container", "CreateBlobContainer", "Blobs")</li>
    ```

1. 執行應用程式，然後選取 [建立 Blob 容器]，以查看類似以下螢幕擷取畫面的結果︰
  
    ![[建立 Blob 容器] 的螢幕擷取畫面](./media/vs-storage-aspnet-getting-started-blobs/create-blob-container-results.png)

    如先前所述，只有在容器不存在且已建立時，`CloudBlobContainer.CreateIfNotExists` 方法才會傳回 **true**。 因此，如果在容器已存在時執行應用程式，則此方法會傳回 **false**。

## <a name="upload-a-blob-into-a-blob-container"></a>將 Blob 上傳至 blob 容器

在[建立 Blob 容器](#create-a-blob-container)之後，請將檔案上傳至該容器。 本節將逐步引導您將本機檔案上傳至 Blob 容器。 這些步驟會假設有一個名為 test-blob-container 的 Blob 容器。 

1. 開啟 `BlobsController.cs` 檔案。

1. 新增名為 `UploadBlob` 且會傳回字串的方法。

    ```csharp
    public string UploadBlob()
    {
        // The code in this section goes here.

        return "success!";
    }
    ```
 
1. 在 `UploadBlob` 方法內，取得 `CloudBlobContainer` 物件，此物件代表對所需 Blob 容器名稱的參考。 
   
    ```csharp
    CloudBlobContainer container = GetCloudBlobContainer();
    ```

1. Azure 儲存體支援不同的 Blob 類型。 本教學課程使用區塊 Blob。 若要擷取對區塊 Blob 的參考，請呼叫 `CloudBlobContainer.GetBlockBlobReference` 方法。

    ```csharp
    CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
    ```
    
    > [!NOTE]
    > Blob 名稱是用來擷取 Blob 之 URL 的一部分，且可以是任何字串，包括檔案的名稱。

1. 有了 Blob 參考之後，您便可以藉由呼叫 Blob 參考物件的 `UploadFromStream` 方法，將任何資料流上傳至其中。 如果 Blob 不存在，`UploadFromStream` 方法就會建立 Blob，若已存在，則會予以覆寫。 (請將 &lt;file-to-upload> 變更為所要上傳之檔案的完整路徑)。

    ```csharp
    using (var fileStream = System.IO.File.OpenRead(@"<file-to-upload>"))
    {
        blob.UploadFromStream(fileStream);
    }
    ```
    
    以下顯示已完成的 `UploadBlob` 方法 (含有所要上傳之檔案的完整路徑)：

    ```csharp
    public string UploadBlob()
    {
        CloudBlobContainer container = GetCloudBlobContainer();
        CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
        using (var fileStream = System.IO.File.OpenRead(@"c:\src\sample.txt"))
        {
            blob.UploadFromStream(fileStream);
        }
        return "success!";
    }
    ```

1. 在 [方案總管] 中，展開 [檢視] > [共用] 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Upload blob", "UploadBlob", "Blobs")</li>
    ```

1. 執行應用程式，並選取 [上傳 blob]。  success! 一字 應該會出現。
    
    ![成功驗證的螢幕擷取畫面](./media/vs-storage-aspnet-getting-started-blobs/upload-blob.png)
  
## <a name="list-the-blobs-in-a-blob-container"></a>列出 blob 容器中的 blob

下一節將說明如何列出 blob 容器中的 blob。 範例程式碼參考在區段中建立的 *test-blob-container*，[建立 blob 容器](#create-a-blob-container)。

1. 開啟 `BlobsController.cs` 檔案。

1. 新增名為 `ListBlobs` 且會傳回 `ActionResult` 的方法。

    ```csharp
    public ActionResult ListBlobs()
    {
        // The code in this section goes here.

    }
    ```
 
1. 在 `ListBlobs` 方法內，取得 `CloudBlobContainer` 物件，此物件代表對 Blob 容器的參考。 
   
    ```csharp
    CloudBlobContainer container = GetCloudBlobContainer();
    ```
   
1. 若要列出 Blob 容器中的 Blob，請使用 `CloudBlobContainer.ListBlobs` 方法。 `CloudBlobContainer.ListBlobs` 方法會傳回 `IListBlobItem` 物件，可以轉換成 `CloudBlockBlob`、`CloudPageBlob`或 `CloudBlobDirectory` 物件。 下列程式碼片段列舉 blob 容器中的所有 blob。 每個 Blob 都會以其類型為基礎，轉換成適當的物件。 其名稱 (如果是 **CloudBlobDirectory**，則為 URI) 會新增至清單。

    ```csharp
    List<string> blobs = new List<string>();

    foreach (IListBlobItem item in container.ListBlobs())
    {
        if (item.GetType() == typeof(CloudBlockBlob))
        {
            CloudBlockBlob blob = (CloudBlockBlob)item;
            blobs.Add(blob.Name);
        }
        else if (item.GetType() == typeof(CloudPageBlob))
        {
            CloudPageBlob blob = (CloudPageBlob)item;
            blobs.Add(blob.Name);
        }
        else if (item.GetType() == typeof(CloudBlobDirectory))
        {
            CloudBlobDirectory dir = (CloudBlobDirectory)item;
            blobs.Add(dir.Uri.ToString());
        }
    }

    return View(blobs);
    ```

    除了 blob，blob 容器可以包含目錄。 假設有名為 test-blob-container 的 blob 容器與下列階層︰

        foo.png
        dir1/bar.png
        dir2/baz.png

    使用上述的程式碼範例，**blob** 字串清單包含類似下列的值︰

        foo.png
        <storage-account-url>/test-blob-container/dir1
        <storage-account-url>/test-blob-container/dir2

    如您所見，此清單只包含最上層實體，而不包含巢狀的 (bar.png 和 baz.png)。 若要列出 blob 容器內的所有實體，請變更程式碼，使 **CloudBlobContainer.ListBlobs** 方法針對 **useFlatBlobListing** 參數傳入 **true**。    

    ```csharp
    //...
    foreach (IListBlobItem item in container.ListBlobs(useFlatBlobListing:true))
    //...
    ```

    將 **useFlatBlobListing** 參數設為 **true** 會傳回 blob 容器中所有實體的簡單列表。 這會產生下列結果：

        foo.png
        dir1/bar.png
        dir2/baz.png
    
    以下顯示已完成的 **ListBlobs** 方法：

    ```csharp
    public ActionResult ListBlobs()
    {
        CloudBlobContainer container = GetCloudBlobContainer();
        List<string> blobs = new List<string>();
        foreach (IListBlobItem item in container.ListBlobs(useFlatBlobListing: true))
        {
            if (item.GetType() == typeof(CloudBlockBlob))
            {
                CloudBlockBlob blob = (CloudBlockBlob)item;
                blobs.Add(blob.Name);
            }
            else if (item.GetType() == typeof(CloudPageBlob))
            {
                CloudPageBlob blob = (CloudPageBlob)item;
                blobs.Add(blob.Name);
            }
            else if (item.GetType() == typeof(CloudBlobDirectory))
            {
                CloudBlobDirectory dir = (CloudBlobDirectory)item;
                blobs.Add(dir.Uri.ToString());
            }
        }

        return View(blobs);
    }
    ```

1. 在 [方案總管] 中，展開 [檢視] 資料夾，然後在 [Blobs] 上按一下滑鼠右鍵。

2. 從快顯功能表中，選取 [新增] > [檢視]。

1. 在 [新增檢視] 對話方塊中，針對檢視名稱輸入 `ListBlobs`，然後選取 [新增]。

1. 開啟 `ListBlobs.cshtml`，然後以下列程式碼取代內容：

    ```html
    @model List<string>
    @{
        ViewBag.Title = "List blobs";
    }
    
    <h2>List blobs</h2>
    
    <ul>
        @foreach (var item in Model)
        {
        <li>@item</li>
        }
    </ul>
    ```

1. 在 [方案總管] 中，展開 [檢視] > [共用] 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("List blobs", "ListBlobs", "Blobs")</li>
    ```

1. 執行應用程式，然後選取 [列出 Blob]，以查看類似以下螢幕擷取畫面的結果︰
  
    ![[列出 Blob] 的螢幕擷取畫面](./media/vs-storage-aspnet-getting-started-blobs/listblobs.png)

## <a name="download-blobs"></a>下載 Blob

本節說明如何下載 Blob。 您可以將它保存到本機儲存體，或是將內容讀取到字串中。 範例程式碼參考在區段中建立的 *test-blob-container*，[建立 blob 容器](#create-a-blob-container)。

1. 開啟 `BlobsController.cs` 檔案。

1. 新增名為 `DownloadBlob` 且會傳回字串的方法。

    ```csharp
    public string DownloadBlob()
    {
        // The code in this section goes here.

        return "success!";
    }
    ```
 
1. 在 `DownloadBlob` 方法內，取得 `CloudBlobContainer` 物件，此物件代表對 Blob 容器的參考。
   
    ```csharp
    CloudBlobContainer container = GetCloudBlobContainer();
    ```

1. 呼叫 `CloudBlobContainer.GetBlockBlobReference` 方法來取得 Blob 參考物件。 

    ```csharp
    CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
    ```

1. 若要下載 Blob，請使用 `CloudBlockBlob.DownloadToStream` 方法。 下列程式碼會將 Blob 的內容傳輸至串流物件。 系統會接著將該物件保存至本機檔案。 (請將 *&lt;local-file-name>* 變更為代表 Blob 下載位置的完整檔案名稱)。 

    ```csharp
    using (var fileStream = System.IO.File.OpenWrite(<local-file-name>))
    {
        blob.DownloadToStream(fileStream);
    }
    ```
    
    以下顯示已完成的 `ListBlobs` 方法 (含有所要建立之本機檔案的完整路徑)：
    
    ```csharp
    public string DownloadBlob()
    {
        CloudBlobContainer container = GetCloudBlobContainer();
        CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
        using (var fileStream = System.IO.File.OpenWrite(@"c:\src\downloadedBlob.txt"))
        {
            blob.DownloadToStream(fileStream);
        }
        return "success!";
    }
    ```

1. 在 [方案總管] 中，展開 [檢視] > [共用] 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Download blob", "DownloadBlob", "Blobs")</li>
    ```

1. 執行應用程式，並選取 [下載 blob] 下載 blob。 `CloudBlobContainer.GetBlockBlobReference` 方法呼叫中指定的 Blob 會下載至 `File.OpenWrite` 方法呼叫中指定的位置。  success! 文字 應該會出現在瀏覽器中。 

## <a name="delete-blobs"></a>刪除 Blob

下列步驟說明如何刪除 blob：

1. 開啟 `BlobsController.cs` 檔案。

1. 新增名為 `DeleteBlob` 且會傳回字串的方法。

    ```csharp
    public string DeleteBlob()
    {
        // The code in this section goes here.

        return "success!";
    }
    ```

1. 在 `DeleteBlob` 方法內，取得 `CloudBlobContainer` 物件，此物件代表對 Blob 容器的參考。
   
    ```csharp
    CloudBlobContainer container = GetCloudBlobContainer();
    ```

1. 呼叫 `CloudBlobContainer.GetBlockBlobReference` 方法來取得 Blob 參考物件。 

    ```csharp
    CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
    ```

1. 若要刪除 Blob，請使用 `Delete` 方法。

    ```csharp
    blob.Delete();
    ```
    
    已完成的 `DeleteBlob` 方法應如下所示：
    
    ```csharp
    public string DeleteBlob()
    {
        CloudBlobContainer container = GetCloudBlobContainer();
        CloudBlockBlob blob = container.GetBlockBlobReference("myBlob");
        blob.Delete();
        return "success!";
    }
    ```

1. 在 [方案總管] 中，展開 [檢視] > [共用] 資料夾，然後開啟 `_Layout.cshtml`。

1. 在最後一個 **Html.ActionLink** 之後，新增下列 **Html.ActionLink**：

    ```html
    <li>@Html.ActionLink("Delete blob", "DeleteBlob", "Blobs")</li>
    ```

1. 執行應用程式，然後選取 [刪除 Blob] 以刪除 `CloudBlobContainer.GetBlockBlobReference` 方法呼叫中指定的 Blob。 success! 文字 應該會出現在瀏覽器中。 選取瀏覽器的 [上一頁] 按鈕，然後選取 [列出 Blob] 以確認 Blob 已不存在於容器中。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 ASP.NET 在「Azure 儲存體」中儲存、列出及擷取 Blob。 如需了解 Azure 中的其他資料儲存選項，請檢視更多功能指南。

  * [開始使用 Azure 資料表儲存體和 Visual Studio 已連線的服務 (ASP.NET)](vs-storage-aspnet-getting-started-tables.md)
  * [開始使用 Azure 佇列儲存體和 Visual Studio 已連線的服務 (ASP.NET)](vs-storage-aspnet-getting-started-queues.md)
