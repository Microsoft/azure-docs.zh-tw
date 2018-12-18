---
title: 對 Azure Blob 儲存體中的容器與 Blob 啟用公用讀取權限 | Microsoft Docs
description: 了解如何讓容器與 Blob 可供匿名存取，以及如何以程式設計方式存取。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/26/2017
ms.author: tamram
ms.openlocfilehash: e27be86a7a14a38c5083949a1a7255574d2d0dc6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956076"
---
# <a name="manage-anonymous-read-access-to-containers-and-blobs"></a>管理對容器與 Blob 的匿名讀取權限。
您可以對 Azure Blob 儲存體中的容器及其 Blob 啟用匿名與公用讀取權限。 如此您就可以將這些資源的唯讀存取權限授與他人，而無須共用您的帳戶金鑰，也無須要求共用存取簽章 (SAS)。

公用讀取權限適用於您想要某些 Blob 永遠可供匿名讀取存取的狀況。 對於更深入的控管需求，您可以建立共用存取簽章。 共用存取簽章可讓您針對一段特定時間提供不同權限的限制存取。 如需建立共用存取簽章的詳細資訊，請參閱[在 Azure 儲存體中使用共用存取簽章 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

## <a name="grant-anonymous-users-permissions-to-containers-and-blobs"></a>授與容器和 Blob 的匿名使用者權限
根據預設，可能只有儲存體帳戶的擁有者能存取容器及其內部的任何 Blob。 若要為匿名使用者授與容器及其 Blob 的讀取權限，您可以設定容器權限以允許公用存取。 匿名使用者可以讀取可公開存取之容器內的 Blob，而不需驗證要求。

您可以為容器設定下列權限︰

* **無公用讀取權限︰** 只有儲存體帳戶擁有者可以存取容器和其 Blob。 這是所有新建容器的預設值。
* **僅對 Blob 有公用讀取權限：** 您可以透過匿名要求讀取容器內的 Blob，但您無法使用容器資料。 匿名用戶端無法列舉容器內的 Blob。
* **完整的公用讀取權限：** 可以透過匿名要求讀取所有容器和 Blob 資料。 用戶端可以透過匿名要求列舉容器內的 Blob，但無法列舉儲存體帳戶內的容器。

您可以使用下列方式設定容器權限：

* [Azure 入口網站](https://portal.azure.com)
* [Azure PowerShell](../common/storage-powershell-guide-full.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [Azure CLI](../common/storage-azure-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#create-and-manage-blobs)
* 使用其中一個儲存體用戶端程式庫或 REST API 以程式設計方式設定

### <a name="set-container-permissions-in-the-azure-portal"></a>在 Azure 入口網站中設定容器權限
若要在 [Azure 入口網站](https://portal.azure.com)中設定容器權限，請遵循下列步驟：

1. 在入口網站中開啟 [儲存體帳戶] 刀鋒視窗。 您可以在主要入口網站的功能表刀鋒視窗中選取 [儲存體帳戶]，來尋找您的儲存體帳戶。
1. 在功能表刀鋒視窗的 [BLOB 服務] 下方，選取 [Blob]。
1. 以滑鼠右鍵按一下容器資料列或選取省略符號來開啟容器的**操作功能表**。
1. 在操作功能表中選取 [存取原則]。
1. 從下拉式功能表中選取 [存取類型]。

    ![編輯容器中繼資料對話方塊](./media/storage-manage-access-to-resources/storage-manage-access-to-resources-0.png)

### <a name="set-container-permissions-with-net"></a>使用 .NET 設定容器權限
若要使用 .NET 的 C# 和儲存體用戶端程式庫來設定容器的權限，請先呼叫 **GetPermissions** 方法來擷取容器的現有權限。 接著為由 **GetPermissions** 方法傳回的 **BlobContainerPermissions** 物件設定 **PublicAccess** 屬性。 最後，使用更新的權限呼叫 **SetPermissions** 方法。

下列範例將容器的權限設為完整公用讀取權限。 若只要將 Blob 的權限設為公用讀取權限，請將 **PublicAccess** 屬性設為 **BlobContainerPublicAccessType.Blob**。 若要移除匿名使用者的所有權限，請將屬性設為 **BlobContainerPublicAccessType.Off**。

```csharp
public static void SetPublicContainerPermissions(CloudBlobContainer container)
{
    BlobContainerPermissions permissions = container.GetPermissions();
    permissions.PublicAccess = BlobContainerPublicAccessType.Container;
    container.SetPermissions(permissions);
}
```

## <a name="access-containers-and-blobs-anonymously"></a>匿名存取容器與 Blob
匿名存取容器與 Blob 的用戶端可使用不需要認證的建構函式。 以下範例顯示可匿名參考 Blob 服務資源的不同方法。

### <a name="create-an-anonymous-client-object"></a>建立匿名用戶端物件
您可以為帳戶提供 Blob 服務端點，來建立用於匿名存取的新服務用戶端物件。 不同，您也必須知道可供匿名存取的帳戶中的容器名稱。

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob service endpoint.
    CloudBlobClient blobClient = new CloudBlobClient(new Uri(@"https://storagesample.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

### <a name="reference-a-container-anonymously"></a>匿名參考容器
如果您有可供匿名使用之容器的 URL，您可以使用該 URL 直接參考容器。

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(new Uri(@"https://storagesample.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

### <a name="reference-a-blob-anonymously"></a>匿名參考 Blob
如果您有可供匿名存取之 Blob 的 URL，您可以使用該 URL 直接參考 Blob：

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(@"https://storagesample.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", System.IO.FileMode.Create);
}
```

## <a name="features-available-to-anonymous-users"></a>匿名使用者可使用的功能
下表顯示當容器的 ACL 設為允許公用存取時，匿名使用者可能會呼叫哪些作業。

| REST 作業 | 具有完整公開讀取權限 | 僅對 Blob 有公開讀取權限 |
| --- | --- | --- |
| 列出容器 |只有擁有者 |只有擁有者 |
| 建立容器 |只有擁有者 |只有擁有者 |
| 取得容器屬性 |全部 |只有擁有者 |
| 取得容器中繼資料 |全部 |只有擁有者 |
| 設定容器中繼資料 |只有擁有者 |只有擁有者 |
| 取得容器 ACL |只有擁有者 |只有擁有者 |
| 設定容器 ACL |只有擁有者 |只有擁有者 |
| 刪除容器 |只有擁有者 |只有擁有者 |
| 列出 Blob |全部 |只有擁有者 |
| 放置 Blob |只有擁有者 |只有擁有者 |
| 取得 Blob |全部 |全部 |
| 取得 Blob 屬性 |全部 |全部 |
| 設定 Blob 屬性 |只有擁有者 |只有擁有者 |
| 取得 Blob 中繼資料 |全部 |全部 |
| 設定 Blob 中繼資料 |只有擁有者 |只有擁有者 |
| 放置區塊 |只有擁有者 |只有擁有者 |
| 取得區塊清單 (僅限認可區塊) |全部 |全部 |
| 取得區塊清單 (僅限未認可的區塊或所有區塊) |只有擁有者 |只有擁有者 |
| 放置區塊清單 |只有擁有者 |只有擁有者 |
| 刪除 Blob |只有擁有者 |只有擁有者 |
| 複製 Blob |只有擁有者 |只有擁有者 |
| 快照 Blob |只有擁有者 |只有擁有者 |
| 租用 Blob |只有擁有者 |只有擁有者 |
| 放置頁面 |只有擁有者 |只有擁有者 |
| 取得頁面範圍 |全部 |全部 |
| 附加 Blob |只有擁有者 |只有擁有者 |

## <a name="next-steps"></a>後續步驟

* [Azure 儲存體服務的驗證](https://msdn.microsoft.com/library/azure/dd179428.aspx)
* [使用共用存取簽章 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [使用共用存取簽章來委派存取權](https://msdn.microsoft.com/library/azure/ee395415.aspx)
