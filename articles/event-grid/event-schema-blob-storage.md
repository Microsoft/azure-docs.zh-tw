---
title: Azure Event Grid blob 儲存體事件結構描述
description: 描述 Azure Event Grid blob 儲存體事件的屬性
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: reference
ms.date: 08/17/2018
ms.author: tomfitz
ms.openlocfilehash: 11524f8868a0102e30b06f3385a26b1bd06aae6e
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141088"
---
# <a name="azure-event-grid-event-schema-for-blob-storage"></a>blob 儲存體的 Azure Event Grid 事件結構描述

本文提供 blob 儲存體事件的屬性與結構描述。 如需事件結構描述的簡介，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。

如需範例指令碼和教學課程的清單，請參閱[儲存體事件來源](event-sources.md#storage)。

## <a name="available-event-types"></a>可用的事件類型

blob 儲存體會發出下列事件類型：

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.Storage.BlobCreated | 建立 blob 時引發。 |
| Microsoft.Storage.BlobDeleted | 刪除 blob 時引發。 |

## <a name="example-event"></a>事件範例

下列範例顯示 blob 建立事件的結構描述： 

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/testfile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://example.blob.core.windows.net/testcontainer/testfile.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

blob 刪除事件的結構描述如下： 

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/testfile.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://example.blob.core.windows.net/testcontainer/testfile.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```
 
## <a name="event-properties"></a>事件屬性

事件具有下列的最高層級資料：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 主題 | 字串 | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| 主旨 | 字串 | 發行者定義事件主體的路徑。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | 字串 | 事件的唯一識別碼。 |
| data | 物件 | blob 儲存體帳戶。 |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| api | 字串 | 觸發事件的作業。 |
| clientRequestId | 字串 | 用戶端產生的不透明值，具有 1 KB 的字元限制。 當您啟用儲存體分析記錄時，它會記錄在分析記錄中。 |
| requestId | 字串 | 要求的唯一識別碼。 使用它針對要求進行疑難排解。 |
| etag | 字串 | 此值可讓您依條件執行作業。 |
| contentType | 字串 | 為 blob 指定內容類型。 |
| contentLength | integer | Blob 大小 (以位元組為單位)。 |
| blobType | 字串 | Blob 的類型。 有效值為 "BlockBlob" 或 "PageBlob"。 |
| url | 字串 | blob 的路徑。 |
| 排序器 | 字串 | 使用者控制的值，可用來追蹤要求。 |
| storageDiagnostics | 物件 | 儲存體診斷的相關資訊。 |
 
## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
* 如需有關使用 blob 儲存體事件的簡介，請參閱[路由 blob 儲存體事件 - Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)。 
