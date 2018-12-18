---
title: 回應 Azure Blob 儲存體事件 | Microsoft Docs
description: 使用 Azure Event Grid 以訂閱 Blob 儲存體事件。
services: storage,event-grid
author: cbrooksmsft
ms.author: cbrooks
ms.date: 01/30/2018
ms.topic: article
ms.service: storage
ms.component: blobs
ms.openlocfilehash: d38ab71ed2d2ebff04004f02589cfccca4199318
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42144245"
---
# <a name="reacting-to-blob-storage-events"></a>回應 Blob 儲存體事件

Azure 儲存體事件可讓應用程式使用新式無伺服器架構來回應 Blob 的建立和刪除。 它不需要複雜的程式碼或昂貴且無效率的輪詢服務來執行此動作。  而是改為透過 [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) 將事件推送給訂閱者，例如 [Azure Functions](https://azure.microsoft.com/services/functions/)、[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)，甚至推送到您自己的自訂 http 接聽程式，且只需要支付有使用的項目的費用。 

常見的 Blob 儲存體事件案例包括映像或影片處理、搜尋索引，或任何檔案導向的工作流程。  非同步檔案上傳非常適合事件。  在變更不常見但情況需要立即回應的情況下，以事件為基礎的架構可能特別有效。

如需快速範例，請參閱[將 Blob 儲存體事件路由至自訂的 Web 端點 - CLI](storage-blob-event-quickstart.md) 或[將 Blob 儲存體事件路由至自訂的 Web 端點 - PowerShell](storage-blob-event-quickstart-powershell.md)。 

![Event Grid 模型](./media/storage-blob-event-overview/event-grid-functional-model.png)

## <a name="blob-storage-accounts"></a>Blob 儲存體帳戶
Blob 儲存體事件適用於 [Blob 儲存體帳戶](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#blob-storage-accounts)和[一般用途 v2 儲存體帳戶](../common/storage-account-options.md#general-purpose-v2-accounts)。 **一般用途 v2 (GPv2)** 儲存體帳戶支援所有儲存體服務 (包括 Blob、檔案、佇列和表格) 的所有功能。 **Blob 儲存體帳戶**是特製化的儲存體帳戶，可將非結構化資料儲存為 Azure 儲存體中的 Blob (物件)。 Blob 儲存體帳戶類似於一般用途儲存體帳戶，可共用所有強大的持續性、可用性、延展性以及您現今使用的效能功能，包括區塊 Blob 和附加 Blob 的 100% API 一致性。 對於只需要封鎖或附加 Blob 儲存體的應用程式，我們建議使用 Blob 儲存體帳戶。 

## <a name="available-blob-storage-events"></a>可用的 Blob 儲存體事件
Event Grid 使用[事件訂閱](../../event-grid/concepts.md#event-subscriptions)將事件訊息路由至訂閱者。  Blob 儲存體事件訂閱可以包含兩種類型的事件：  

> |活動名稱|說明|
> |----------|-----------|
> |`Microsoft.Storage.BlobCreated`|透過 `PutBlob`、`PutBlockList` 或 `CopyBlob` 作業建立或取代 Blob 時引發|
> |`Microsoft.Storage.BlobDeleted`|透過 `DeleteBlob` 作業刪除 Blob 時引發|

## <a name="event-schema"></a>事件結構描述
Blob 儲存體事件包含了回應資料變更時所需的所有資訊。  因為 eventType 屬性開頭為 "Microsoft.Storage"，可以藉此識別出 Blob 儲存體事件。 Event Grid 事件屬性之使用方式的其他資訊列於[Event Grid 事件結構描述](../../event-grid/event-schema.md)。  

> |屬性|類型|說明|
> |-------------------|------------------------|-----------------------------------------------------------------------|
> |主題|字串|發出事件之儲存體帳戶的完整 Azure Resource Manager 識別碼。|
> |主旨|字串|做為事件主體之物件的相對資源路徑，使用的 Azure Resource Manager 擴充格式與我們用於描述 Azure RBAC 之儲存體帳戶、服務以及容器的相同。  此格式包含保留大小寫的 Blob 名稱。|
> |eventTime|字串|產生事件的日期/時間，採用 ISO 8601 格式|
> |eventType|字串|"Microsoft.Storage.BlobCreated" 或 "Microsoft.Storage.BlobDeleted"|
> |id|字串|此事件的唯一識別碼|
> |dataVersion|字串|資料物件的結構描述版本。|
> |metadataVersion|字串|最上層屬性的結構描述版本。|
> |data|物件|Blob 儲存體專有事件資料集合|
> |data.contentType|字串|Blob 的內容類型，會以 Blob 的 Content-Type 標頭傳回|
> |data.contentLength|number|以整數表示的 Blob 大小代表位元組數目，會以 Blob 的 Content-Length 標頭傳回。  隨 BlobCreated 事件傳送，但不隨 BlobDeleted 傳送。|
> |data.url|字串|做為事件主體之物件的 url|
> |data.eTag|字串|引發此事件時的物件 etag。  無法用於 BlobDeleted 事件。|
> |data.api|字串|觸發此事件的 API 作業名稱。 對於 BlobCreated 事件，此值為 "PutBlob"、"PutBlockList" 或 "CopyBlob"。 對於 BlobDeleted 事件，此值為 "DeleteBlob"。 這些值是出現在 Azure 儲存體診斷記錄中的相同 api 名稱。 請參閱[記錄作業和狀態訊息](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)。|
> |data.sequencer|字串|不透明的字串值表示任何特定 Blob 名稱之事件的邏輯順序。  使用者可使用標準字串比較，以了解 Blob 名稱相同之兩個事件的相對順序。|
> |data.requestId|字串|儲存體 API 作業由服務產生的要求識別碼。 可用於利用記錄中的 "request-id-header" 欄位與 Azure 儲存體診斷記錄建立關聯，並從 'x-ms-request-id' 標頭中的 API 呼叫初始化傳回。 請參閱[記錄格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。|
> |data.clientRequestId|字串|儲存體 API 作業由用戶端產生的要求識別碼。 可用於利用記錄中的 "client-request-id" 欄位與 Azure 儲存體診斷記錄建立關聯，並且可使用 "x-ms-client-request-id" 標頭於用戶端要求中提供。 請參閱[記錄格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。 |
> |data.storageDiagnostics|物件|Azure 儲存體服務偶爾包含診斷資料。 出現時，事件消費者應該予以忽略。|
|data.blobType|字串|Blob 的類型。 有效值為 "BlockBlob" 或 "PageBlob"。| 

以下是 BlobCreated 事件的範例：
```json
[{
  "topic": "/subscriptions/319a9601-1ec0-0000-aebc-8fe82724c81e/resourceGroups/testrg/providers/Microsoft.Storage/storageAccounts/myaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file1.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T01:57:26.005121Z",
  "id": "602a88ef-0001-00e6-1233-1646070610ea",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "799304a4-bbc5-45b6-9849-ec2c66be800a",
    "requestId": "602a88ef-0001-00e6-1233-164607000000",
    "eTag": "0x8D4E44A24ABE7F1",
    "contentType": "text/plain",
    "contentLength": 447,
    "blobType": "BlockBlob",
    "url": "https://myaccount.blob.core.windows.net/testcontainer/file1.txt",
    "sequencer": "00000000000000EB000000000000C65A",
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

如需詳細資訊，請參閱 [Blob 儲存體事件結構描述](../../event-grid/event-schema-blob-storage.md)。

## <a name="filtering-events"></a>篩選事件
Blob 事件訂閱可以根據事件類型來篩選，也可以依據容器名稱和建立或刪除之物件的 Blob 名稱進行篩選。  篩選可以在事件訂用帳戶的[建立](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az_eventgrid_event_subscription_create)期間或[稍後](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az_eventgrid_event_subscription_update)套用至事件訂用帳戶。 事件格線中的主體篩選是根據 “begins with” 和 “ends with” 的相符項目來運作，以將具有相符主體的事件傳遞給訂閱者。 

Blob 儲存體事件的主體使用格式：

```
/blobServices/default/containers/<containername>/blobs/<blobname>
```

若要比對儲存體帳戶的所有事件，您可以將主體篩選條件保留空白。

若要比對建立於共用前置詞的一組容器之 Blob 事件，請使用 `subjectBeginsWith` 篩選條件，例如：

```
/blobServices/default/containers/containerprefix
```

若要比對建立於特定容器之 Blob 事件，請使用 `subjectBeginsWith` 篩選條件，例如：

```
/blobServices/default/containers/containername/
```

若要比對建立於共用 Blob 名稱前置詞的特定容器之 Blob 事件，請使用 `subjectBeginsWith` 篩選條件，例如：

```
/blobServices/default/containers/containername/blobs/blobprefix
```

若要比對建立於共用 Blob 名稱前置詞的特定容器之 Blob 事件，請使用 `subjectEndsWith` 篩選條件，例如 ".log" 或 ".jpg"。 如需詳細資訊，請參閱 [Event Grid 概念](../../event-grid/concepts.md#event-subscriptions)。

## <a name="practices-for-consuming-events"></a>消費事件做法
處理 Blob 儲存體事件的應用程式應該遵循幾個建議做法：
> [!div class="checklist"]
> * 由於可設定多個訂用帳戶以將事件路由至相同的事件處理常式，因此重要的是，不要假設事件來自於特定來源，而要檢查訊息主題以確定其來自預期的儲存體帳戶。
> * 同樣地，檢查 eventType 也是必須進行的步驟之一，而且不要假設您收到的所有事件都是您預期的類型。
> * 由於訊息可能會在延遲一段時間後以錯誤順序送達，請使用 [etag] 欄位以了解您的物件資訊是否仍是最新狀態。  使用 [排序器] 欄位以了解任何特定物件上的事件順序。
> * 請使用 [blobType] 欄位以了解 Blob 允許何種類型的作業，以及您應該使用何種類型的用戶端程式庫來存取 Blob。 有效值為 `BlockBlob` 或 `PageBlob`。 
> * 請使用帶有 `CloudBlockBlob` 和 `CloudAppendBlob` 建構函式的 [url] 欄位存取 Blob。
> * 請忽略您不了解的欄位。 此做法將有助於保持未來可能新增功能的彈性。


## <a name="next-steps"></a>後續步驟

深入了解 Event Grid 並嘗試 Blob 儲存體事件：

- [關於 Event Grid](../../event-grid/overview.md)
- [將 Blob 儲存體事件路由至自訂的 Web 端點](storage-blob-event-quickstart.md)
