---
title: Azure Event Grid 事件結構描述
description: 描述 Azure Event Grid 中事件提供的屬性
services: event-grid
author: banisadr
manager: timlt
ms.service: event-grid
ms.topic: reference
ms.date: 07/20/2018
ms.author: babanisa
ms.openlocfilehash: f7be7e5f5e51a47b95d39047af9bcf08e463ca34
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39186106"
---
# <a name="azure-event-grid-event-schema"></a>Azure Event Grid 事件結構描述

本文說明出現在所有事件中的屬性和結構描述。 事件包含一組五個必要字串屬性和一個必要資料物件。 這些屬性通用於任何發行者的所有事件。 資料物件含有各發行者特有的屬性。 系統主題下的屬性專屬於資源提供者，像是 Microsoft Azure 儲存體或 Azure 事件中樞。

事件來源會將事件以陣列型態傳送至 Azure 事件方格，陣列中可包含數個事件物件。 張貼事件到事件方格主題時，陣列總大小最大為 1 MB。 陣列中的每個事件會限制為 64 KB。 如果事件或陣列超過大小限制，您會收到 **413 承載太大**回應。

事件方格會將事件傳送給包含單一事件之陣列中的訂閱者。 此行為未來可能會變更。

您可以在[事件結構描述存放區](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/eventgrid/data-plane)中找到 Event Grid 事件的 JSON 結構描述和每個 Azure 發行者的資料承載。

## <a name="event-schema"></a>結構描述

下列範例顯示所有事件發行者會使用的屬性：

```json
[
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
]
```

例如，針對 Azure Blob 儲存體事件發行的結構描述為：

```json
[
  {
    "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
    "subject": "/blobServices/default/containers/oc2d2817345i200097container/blobs/oc2d2817345i20002296blob",
    "eventType": "Microsoft.Storage.BlobCreated",
    "eventTime": "2017-06-26T18:41:00.9584103Z",
    "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
      "api": "PutBlockList",
      "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
      "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
      "eTag": "0x8D4BCC2E4835CD0",
      "contentType": "application/octet-stream",
      "contentLength": 524288,
      "blobType": "BlockBlob",
      "url": "https://oc2d2817345i60006.blob.core.windows.net/oc2d2817345i200097container/oc2d2817345i20002296blob",
      "sequencer": "00000000000004420000000000028963",
      "storageDiagnostics": {
        "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
      }
    },
    "dataVersion": "",
    "metadataVersion": "1"
  }
]
```

## <a name="event-properties"></a>事件屬性

所有事件皆包含下列相同的最高層級資料：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 主題 | 字串 | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| 主旨 | 字串 | 發行者定義事件主體的路徑。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | 字串 | 事件的唯一識別碼。 |
| data | 物件 | 資源提供者特有的事件資料。 |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

若要了解資料物件中的屬性，請參閱事件來源：

* [Azure 訂用帳戶 (管理作業)](event-schema-subscriptions.md)
* [容器登錄](event-schema-container-registry.md)
* [Blob 儲存體](event-schema-blob-storage.md)
* [事件中樞](event-schema-event-hubs.md)
* [IoT 中心](event-schema-iot-hub.md)
* [媒體服務](../media-services/latest/media-services-event-schemas.md?toc=%2fazure%2fevent-grid%2ftoc.json)
* [資源群組 (管理作業)](event-schema-resource-groups.md)
* [服務匯流排](event-schema-service-bus.md)

針對自訂主題，事件發行者會決定資料物件。 最高層級的資料應包含與標準資源定義事件相同的欄位。

將事件發佈至自訂主題時，為您的事件建立主旨，以便訂閱者輕鬆地了解他們是否對該事件感興趣。 訂閱者使用主旨以篩選和路由事件。 考慮為事件發生的位置提供路徑，以便訂閱者可以根據該路徑的區段進行篩選。 該路徑可讓訂閱者縮小或放大篩選事件的範圍。 例如，如果您在主旨中提供三區段路徑，例如 `/A/B/C`，訂閱者可根據第一個區段 `/A` 進行篩選，以取得一組廣泛的事件。 這些訂閱者可使用 `/A/B/C` 或 `/A/D/E` 之類的主旨來取得事件。 其他訂閱者可以根據 `/A/B` 篩選以取得一組範圍較小的事件。

有時候您的主旨需要更多關於發生事情的細節。 例如，當檔案新增至容器時，**儲存體帳戶**發行者會提供主旨 `/blobServices/default/containers/<container-name>/blobs/<file>`。 訂閱者可根據路徑 `/blobServices/default/containers/testcontainer` 篩選，以取得儲存體帳戶中該容器的所有事件，但不會取得其他容器中的事件。 訂閱者也可以根據尾碼 `.txt` 進行篩選或路由，以僅與文字檔搭配使用。

## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
