---
title: 管理 Azure 內容傳遞網路中 Azure Blob 儲存體的期限 | Microsoft Docs
description: 深入了解選項，以控制 Azure CDN 快取中的 Blob 存留時間。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ad4801e9-d09a-49bf-b35c-efdc4e6034e8
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 02/1/2018
ms.author: mazha
ms.openlocfilehash: a0f89a272fa300f6acced2de02ba5465ab282079
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33765631"
---
# <a name="manage-expiration-of-azure-blob-storage-in-azure-cdn"></a>在 Azure CDN 中管理 Azure Blob 儲存體的到期
> [!div class="op_single_selector"]
> * [Azure 網頁內容](cdn-manage-expiration-of-cloud-service-content.md)
> * [Azure Blob 儲存體](cdn-manage-expiration-of-blob-content.md)
> 
> 

Azure 儲存體中的 [Blob 儲存體服務](../storage/common/storage-introduction.md#blob-storage)是數個已與 Azure 內容傳遞網路 (CDN) 整合之 Azure 型來源的其中一個。 任何可公開存取的 Blob 內容均可在 Azure CDN 中加以快取，直到其存留時間 (TTL) 結束。 TTL 是由來自原始伺服器之 HTTP 回應中的 `Cache-Control` 標頭所決定。 本文章會說明幾種可設定位於 Azure 儲存體中，Blob 上的 `Cache-Control` 標頭的方法。

您也可以藉由設定 [CDN 快取規則](#setting-cache-control-headers-by-using-caching-rules)，從 Azure 入口網站控制快取設定。 如果您建立快取規則，並將其快取行為設定為 [覆寫] 或 [略過快取]，就會忽略本文所討論之原始提供的快取設定。 如需一般快取概念的相關資訊，請參閱[快取如何運作](cdn-how-caching-works.md)。

> [!TIP]
> 您可以選擇不替 blob 設定 TTL。 在此情況下，除非您已在 Azure 入口網站中設定快取規則，否則 Azure CDN 會自動套用七天的預設 TTL。 此預設 TTL 僅會套用至一般 Web 傳遞最佳化。 針對大型檔案的最佳化，預設 TTL 為一天；針對媒體串流最佳化，預設 TTL 則為一年。
> 
> 針對 Azure CDN 如何加快對 blob 和其他檔案的存取速度，如需詳細資訊請參閱 [Azure 內容傳遞網路概觀](cdn-overview.md)。
> 
> 若要深入了解 Azure Blob 儲存體，請參閱 [Blob 儲存體簡介](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)。
 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>使用 CDN 快取規則設定 Cache-Control 標頭
設定 blob `Cache-Control` 標頭的慣用方法為在 Azure 入口網站中使用快取規則。 如需 CDN 快取規則的詳細資訊，請參閱[使用快取規則控制 Azure CDN 快取行為](cdn-caching-rules.md)。

> [!NOTE] 
> 快取規則僅適用於「**來自 Verizon 的 Azure CDN 標準**」和「**來自 Akamai 的 Azure CDN 標準**」的設定檔。 針對「**來自 Verizon 的 Azure CDN 進階**」設定檔，您必須使用 [管理] 入口網站中的 [Azure CDN 規則引擎](cdn-rules-engine.md)來執行類似功能。

**瀏覽至 CDN 快取規則頁面**：

1. 在 Azure 入口網站中，選取 CDN 設定檔，然後選取 blob 的端點。

2. 在左窗格的 [設定] 下方，選取 [快取規則]。

   ![CDN 快取規則按鈕](./media/cdn-manage-expiration-of-blob-content/cdn-caching-rules-btn.png)

   [快取規則] 頁面隨即出現。

   ![CDN 快取頁面](./media/cdn-manage-expiration-of-blob-content/cdn-caching-page.png)


**使用全域快取規則設定 blob 儲存體服務的 Cache-Control 標頭：**

1. 在 [全域快取規則] 下方，將 [查詢字串快取行為] 設定為 [忽略查詢字串]，並將 [快取行為] 設定為 [覆寫]。
      
2. 在 [快取到期期間] 的 [秒鐘] 方塊中輸入 3600 或在 [小時] 方塊中輸入 1。 

   ![CDN 全域快取規則範例](./media/cdn-manage-expiration-of-blob-content/cdn-global-caching-rules-example.png)

   這個全域快取規則會設定一小時的快取期間，並影響針對端點的所有要求。 它會覆寫由端點指定之原始伺服器所傳送的任何 `Cache-Control` 或 `Expires` HTTP 標頭。   

3. 選取 [ **儲存**]。
 
**使用自訂快取規則設定 blob 檔案的 Cache-Control 標頭：**

1. 在 [自訂快取規則] 下，建立兩個比對條件：

     A. 在第一個比對條件，將 [比對條件] 設為 [路徑]，並在 [符合值] 輸入 `/blobcontainer1/*`。 將 [快取行為] 設定為 [覆寫]，並在 [時數] 方塊中輸入 4。

    B. 在第二個比對條件，將 [比對條件] 設為 [路徑]，並在 [符合值] 輸入 `/blobcontainer1/blob1.txt`。 將 [快取行為] 設定為 [覆寫]，並在 [時數] 方塊中輸入 2。

    ![CDN 自訂快取規則範例](./media/cdn-manage-expiration-of-blob-content/cdn-custom-caching-rules-example.png)

    第一個自訂快取規則會替您的端點指定之來源伺服器上 `/blobcontainer1` 資料夾中的所有 blob 檔案，設定四個小時的快取期間。 第二個規則只會針對 `blob1.txt` blob 檔案覆寫第一個規則，為其設定兩個小時的快取期間。

2. 選取 [ **儲存**]。


## <a name="setting-cache-control-headers-by-using-azure-powershell"></a>使用 Azure PowerShell 設定 Cache-Control 標頭
[Azure PowerShell](/powershell/azure/overview) 是其中一種最快速、最強大的 Azure 服務管理方式。 請使用 `Get-AzureStorageBlob` Cmdlet 來取得 Blob 的參考，然後設定 `.ICloudBlob.Properties.CacheControl` 屬性。 

例如︰

```powershell
# Create a storage context
$context = New-AzureStorageContext -StorageAccountName "<storage account name>" -StorageAccountKey "<storage account key>"

# Get a reference to the blob
$blob = Get-AzureStorageBlob -Context $context -Container "<container name>" -Blob "<blob name>"

# Set the CacheControl property to expire in 1 hour (3600 seconds)
$blob.ICloudBlob.Properties.CacheControl = "max-age=3600"

# Send the update to the cloud
$blob.ICloudBlob.SetProperties()
```

> [!TIP]
> 您也可以使用 PowerShell 來[管理 CDN 設定檔和端點](cdn-manage-powershell.md)。
> 
>

## <a name="setting-cache-control-headers-by-using-net"></a>使用 .NET 設定 Cache-Control 標頭
若要使用 .NET 程式碼指定 blob 的 `Cache-Control` 標頭，請使用[適用於 .NET 的 Azure 儲存體用戶端程式庫](../storage/blobs/storage-dotnet-how-to-use-blobs.md)，來設定 [CloudBlob.Properties.CacheControl](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.blobproperties.cachecontrol.aspx) 屬性。

例如︰

```csharp
class Program
{
    const string connectionString = "<storage connection string>";
    static void Main()
    {
        // Retrieve storage account information from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);

        // Create a blob client for interacting with the blob service.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create a reference to the container
        CloudBlobContainer <container name> = blobClient.GetContainerReference("<container name>");

        // Create a reference to the blob
        CloudBlob <blob name> = container.GetBlobReference("<blob name>");

        // Set the CacheControl property to expire in 1 hour (3600 seconds)
        blob.Properties.CacheControl = "max-age=3600";

        // Update the blob's properties in the cloud
        blob.SetProperties();
    }
}
```

> [!TIP]
> [適用於 .NET 的 Azure Blob 儲存體範例](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/)中另外還提供了更多 .NET 程式碼範例。
> 

## <a name="setting-cache-control-headers-by-using-other-methods"></a>使用其他方法設定 Cache-Control 標頭

### <a name="azure-storage-explorer"></a>Azure 儲存體總管
透過 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)，您可以檢視和編輯您的 Blob 儲存體資源，包括如 *CacheControl* 屬性的屬性。 

若要使用 Azure 儲存體總管更新 blob 的 CacheControl 屬性：
   1. 選取 blob，然後從捷徑功能表選取 [屬性]。 
   2. 向下捲動至 CacheControl 屬性。
   3. 輸入值，然後選取 [儲存]。


![Azure 儲存體總管屬性](./media/cdn-manage-expiration-of-blob-content/cdn-storage-explorer-properties.png)

### <a name="azure-command-line-interface"></a>Azure 命令列介面
透過 [Azure 命令列介面](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) (CLI)，您可以從命令列管理 Azure Blob 資源。 若要在使用 Azure CLI 上傳 Blob 時設定 Cache-Control 標頭，請使用 `-p` 參數來設定 *cacheControl* 屬性。 下列範例顯示如何將 TTL 設定為一小時 (3600 秒)：
  
```azurecli
azure storage blob upload -c <connectionstring> -p cacheControl="max-age=3600" .\<blob name> <container name> <blob name>
```

### <a name="azure-storage-services-rest-api"></a>Azure 儲存體服務 REST API
您可以使用 [Azure 儲存體服務 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx) \(英文\)，在要求上使用下列作業來明確設定 *x-ms-blob-cache-control* 屬性：
  
   - [Put Blob](https://msdn.microsoft.com/library/azure/dd179451.aspx) \(英文\)
   - [Put Block List](https://msdn.microsoft.com/library/azure/dd179467.aspx) \(英文\)
   - [Set Blob Properties](https://msdn.microsoft.com/library/azure/ee691966.aspx) \(英文\)

## <a name="testing-the-cache-control-header"></a>測試 Cache-Control 標頭
您可以輕鬆地驗證 Blob 的 TTL 設定。 使用瀏覽器的[開發人員工具](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/)，測試 blob 是否包含 `Cache-Control` 的回應標頭。 您也可以使用 [Wget](https://www.gnu.org/software/wget/) \(英文\)、[Postman](https://www.getpostman.com/) \(英文\) 或 [Fiddler](http://www.telerik.com/fiddler) \(英文\) 之類的工具來檢查回應標頭。

## <a name="next-steps"></a>後續步驟
* [了解如何在 Azure CDN 中管理雲端服務內容的到期](cdn-manage-expiration-of-cloud-service-content.md)
* [了解快取概念](cdn-how-caching-works.md)

