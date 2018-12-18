---
title: 將 Azure Blob 儲存體事件路由至自訂 Web 端點 - PowerShell | Microsoft Docs
description: 使用 Azure Event Grid 以訂閱 Blob 儲存體事件。
services: storage,event-grid
author: david-stanford
ms.author: dastanfo
ms.date: 08/23/2018
ms.topic: article
ms.service: storage
ms.component: blobs
ms.openlocfilehash: 8482678a9c42fa2d960dee54c9810593cd820553
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45731979"
---
# <a name="route-blob-storage-events-to-a-custom-web-endpoint-with-powershell"></a>使用 PowerShell 將 Blob 儲存體事件路由至自訂的 Web 端點

Azure Event Grid 是一項雲端事件服務。 在本文中，您可以使用 Azure PowerShell 來訂閱 Blob 儲存體事件、觸發事件並檢視結果。 

通常，您會將事件傳送至可處理事件資料及採取行動的端點。 不過，若要簡化這篇文章，您可將事件傳送至可收集及顯示訊息的 Web 應用程式。

當您完成時，您會看到事件資料已傳送至 Web 應用程式。

![檢視結果](./media/storage-blob-event-quickstart-powershell/view-results.png)

## <a name="setup"></a>設定

本文需要您執行最新版本的 Azure PowerShell。 如果您需要安裝或升級，請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。

## <a name="log-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶，並遵循畫面上的指示以進行驗證。

```powershell
Connect-AzureRmAccount
```

這個範例會使用 **westus2**，並將選取項目儲存在變數中，以利隨時使用。

```powershell
$location = "westus2"
```

## <a name="create-a-resource-group"></a>建立資源群組

Event Grid 為 Azure 資源，必須放入 Azure 資源群組中。 資源群組是在其中部署與管理 Azure 資源的邏輯集合。

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立資源群組。

下列範例會在 westus2 位置建立名為 gridResourceGroup 的資源群組。  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzureRmResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

您可以在一般用途 v2 儲存體帳戶與 Blob 儲存體帳戶中找到 Blob 儲存體事件。 **一般用途 v2** 儲存體帳戶支援所有儲存體服務 (包括 Blob、檔案、佇列和表格) 的所有功能。 **Blob 儲存體帳戶**是特製化的儲存體帳戶，可將非結構化資料儲存為 Azure 儲存體中的 Blob (物件)。 Blob 儲存體帳戶類似於一般用途儲存體帳戶，可共用所有強大的持續性、可用性、延展性以及您現今使用的效能功能，包括區塊 Blob 和附加 Blob 的 100% API 一致性。 如需詳細資訊，請參閱 [Azure 儲存體帳戶概觀](../common/storage-account-overview.md)。

使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/New-AzureRmStorageAccount) 建立可進行 LRS 複寫的 Blob 儲存體帳戶，然後擷取儲存體帳戶內容，此內容定義要使用的儲存體帳戶。 使用儲存體帳戶時，會參考內容而非重複提供認證。 這個範例會建立名為 **gridstorage** 的儲存體帳戶，並含有本機備援儲存體 (LRS)。 

> [!NOTE]
> 儲存體帳戶名稱位於全域命名空間中，因此，您需要將某些隨機字元附加至此指令碼中提供的名稱。

```powershell
$storageName = "gridstorage"
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind BlobStorage `
  -AccessTier Hot

$ctx = $storageAccount.Context
```

## <a name="create-a-message-endpoint"></a>建立訊息端點

訂閱主題之前，讓我們建立事件訊息的端點。 通常，端點會根據事件資料採取動作。 若要簡化此快速入門，請部署[預先建置的 Web 應用程式](https://github.com/Azure-Samples/azure-event-grid-viewer)以顯示事件訊息。 已部署的解決方案包含 App Service 方案、App Service Web 應用程式，以及 GitHub 中的原始程式碼。

以 Web 應用程式的唯一名稱取代 `<your-site-name>`。 Web 應用程式名稱必須是唯一的，因為它是 DNS 項目的一部分。

```powershell
$sitename="<your-site-name>"

New-AzureRmResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" `
  -siteName $sitename `
  -hostingPlanName viewerhost
```

部署需要幾分鐘的時間才能完成。 成功部署之後，檢視 Web 應用程式，確定它正在執行。 在網頁瀏覽器中，瀏覽至：`https://<your-site-name>.azurewebsites.net`

您應會看到網站目前未顯示任何訊息。

[!INCLUDE [event-grid-register-provider-powershell.md](../../../includes/event-grid-register-provider-powershell.md)]

## <a name="subscribe-to-your-storage-account"></a>訂閱您的儲存體帳戶

您可訂閱主題，告知 Event Grid 您想要追蹤的事件。下列範例會訂閱您所建立的儲存體帳戶，並從 Web 應用程式傳遞 URL 作為事件通知的端點。 Web 應用程式的端點必須包含的尾碼 `/api/updates/`。

```powershell
$storageId = (Get-AzureRmStorageAccount -ResourceGroupName $resourceGroup -AccountName $storageName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzureRmEventGridSubscription `
  -EventSubscriptionName gridBlobQuickStart `
  -Endpoint $endpoint `
  -ResourceId $storageId
```

再次檢視 Web 應用程式，並注意訂用帳戶的驗證事件已傳送給它。 選取眼睛圖示來展開事件資料。 Event Grid 會傳送驗證事件，以便端點確認它要接收事件資料。 Web 應用程式包含用來驗證訂用帳戶的程式碼。

![訂用訂用帳戶事件](./media/storage-blob-event-quickstart-powershell/view-subscription-event.png)

## <a name="trigger-an-event-from-blob-storage"></a>從 Blob 儲存體觸發事件

現在，讓我們觸發事件以了解 Event Grid 如何將訊息散發至您的端點。 首先，建立容器和物件。 然後，將物件上傳至容器。

```powershell
$containerName = "gridcontainer"
New-AzureStorageContainer -Name $containerName -Context $ctx

echo $null >> gridTestFile.txt

Set-AzureStorageBlobContent -File gridTestFile.txt -Container $containerName -Context $ctx -Blob gridTestFile.txt
```

您已觸發此事件，而 Event Grid 會將訊息傳送至您在訂閱時設定的端點。 檢視您的 Web 應用程式以查看剛傳送的事件。

```json
[{
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Storage/storageAccounts/myblobstorageaccount",
  "subject": "/blobServices/default/containers/gridcontainer/blobs/gridTestFile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T20:33:51.0595757Z",
  "id": "4d96b1d4-0001-00b3-58ce-16568c064fab",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "Azure-Storage-PowerShell-d65ca2e2-a168-4155-b7a4-2c925c18902f",
    "requestId": "4d96b1d4-0001-00b3-58ce-16568c000000",
    "eTag": "0x8D4E4E61AE038AD",
    "contentType": "application/octet-stream",
    "contentLength": 0,
    "blobType": "BlockBlob",
    "url": "https://myblobstorageaccount.blob.core.windows.net/gridcontainer/gridTestFile.txt",
    "sequencer": "00000000000000EB0000000000046199",
    "storageDiagnostics": {
      "batchId": "dffea416-b46e-4613-ac19-0371c0c5e352"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

## <a name="clean-up-resources"></a>清除資源
如果您打算繼續使用此儲存體帳戶和事件訂用帳戶，請勿清除在本文中建立的資源。 如果您不打算繼續，請使用下列命令來刪除您在本文建立的資源。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>後續步驟

您現在知道如何建立主題和事件訂用帳戶，請深入了解 Blob 儲存體事件以及 Event Grid 如何協助您：

- [回應 Blob 儲存體事件](storage-blob-event-overview.md)
- [關於 Event Grid](../../event-grid/overview.md)
