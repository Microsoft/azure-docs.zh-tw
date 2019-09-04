---
title: Azure Event Grid blob 儲存體事件結構描述
description: 描述 Azure Event Grid blob 儲存體事件的屬性
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: reference
ms.date: 01/17/2019
ms.author: spelluru
ms.openlocfilehash: bd85353aa37cf182a807d99cdc9fb63ead00edeb
ms.sourcegitcommit: 2aefdf92db8950ff02c94d8b0535bf4096021b11
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/03/2019
ms.locfileid: "70232428"
---
# <a name="azure-event-grid-event-schema-for-blob-storage"></a>blob 儲存體的 Azure Event Grid 事件結構描述

本文提供 blob 儲存體事件的屬性與結構描述。 如需事件結構描述的簡介，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。

如需範例指令碼和教學課程的清單，請參閱[儲存體事件來源](event-sources.md#storage)。

## <a name="list-of-events-for-blob-rest-apis"></a>Blob REST Api 的事件清單

當用戶端藉由呼叫 Blob REST Api 來建立、取代或刪除 blob 時, 就會觸發這些事件。

 |事件名稱 |描述|
 |----------|-----------|
 |**Microsoft.Storage.BlobCreated** |建立或取代 blob 時觸發。 <br>具體而言, 當用戶端使用`PutBlob`可在 Blob REST API 中取得的、 `PutBlockList`或`CopyBlob`作業時, 就會觸發此事件。   |
 |**Microsoft.Storage.BlobDeleted** |刪除 blob 時觸發。 <br>具體而言, 當用戶端呼叫`DeleteBlob` Blob REST API 中可用的作業時, 就會觸發此事件。 |

> [!NOTE]
> 如果您想要確保只有在完全認可區塊 Blob 時才會觸發**microsoft.storage.blobcreated**事件, 請篩選`CopyBlob`、 `PutBlob`和`PutBlockList` REST API 呼叫的事件。 只有在資料完全認可至區塊 Blob 之後, 這些 API 呼叫才會觸發**microsoft.storage.blobcreated**事件。 若要瞭解如何建立篩選準則, 請參閱[篩選事件方格的事件](https://docs.microsoft.com/azure/event-grid/how-to-filter-events)。

## <a name="list-of-the-events-for-azure-data-lake-storage-gen-2-rest-apis"></a>Azure Data Lake Storage Gen 2 REST Api 的事件清單

如果您在儲存體帳戶上啟用階層式命名空間, 而且用戶端呼叫 Azure Data Lake Storage Gen2 REST Api, 就會觸發這些事件。

> [!NOTE]
> 這些事件處於公開預覽狀態, 且僅適用于**美國西部 2**和**美國中西部**區域。

 |事件名稱|描述|
 |----------|-----------|
 |**Microsoft.Storage.BlobCreated** | 建立或取代 blob 時觸發。 <br>具體而言, 當用戶端使用 Azure Data Lake Storage Gen2 REST API 中`CreateFile`提供`FlushWithClose`的和作業時, 就會觸發此事件。 |
 |**Microsoft.Storage.BlobDeleted** |刪除 blob 時觸發。 <br>具體而言, 當用戶端呼叫`DeleteFile` Azure Data Lake Storage Gen2 REST API 中提供的作業時, 也會觸發此事件。 |
 |**Microsoft.Storage.BlobRenamed**|在重新命名 blob 時觸發。 <br>具體而言, 當用戶端使用 Azure Data Lake Storage Gen2 REST API 中`RenameFile`提供的作業時, 就會觸發此事件。|
 |**Microsoft.Storage.DirectoryCreated**|建立目錄時觸發。 <br>具體而言, 當用戶端使用 Azure Data Lake Storage Gen2 REST API 中`CreateDirectory`提供的作業時, 就會觸發此事件。|
 |**Microsoft.Storage.DirectoryRenamed**|在目錄重新命名時觸發。 <br>具體而言, 當用戶端使用 Azure Data Lake Storage Gen2 REST API 中`RenameDirectory`提供的作業時, 就會觸發此事件。|
 |**Microsoft.Storage.DirectoryDeleted**|刪除目錄時觸發。 <br>具體而言, 當用戶端使用 Azure Data Lake Storage Gen2 REST API 中`DeleteDirectory`提供的作業時, 就會觸發此事件。|

> [!NOTE]
> 如果您想要確保只有在完全認可區塊 Blob 時才會觸發**microsoft.storage.blobcreated**事件, 請篩選`FlushWithClose` REST API 呼叫的事件。 只有在資料完全認可至區塊 Blob 之後, 此 API 呼叫才會觸發**microsoft.storage.blobcreated**事件。 若要瞭解如何建立篩選準則, 請參閱[篩選事件方格的事件](https://docs.microsoft.com/azure/event-grid/how-to-filter-events)。

<a id="example-event" />

## <a name="the-contents-of-an-event-response"></a>事件回應的內容

觸發事件時, 事件方格服務會將該事件的相關資料傳送至訂閱端點。

本章節包含每個 blob 儲存體事件的資料外觀範例。

### <a name="microsoftstorageblobcreated-event"></a>Microsoft.storage.blobcreated 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/test-container/blobs/new-file.txt",
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
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/new-file.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobcreated-event-data-lake-storage-gen2"></a>Microsoft.storage.blobcreated 事件 (Data Lake Storage Gen2)

如果 blob 儲存體帳戶具有階層式命名空間, 資料看起來會類似先前的範例, 但這些變更除外:

* 此`dataVersion`索引鍵會設定為的`2`值。

* 此`data.api`索引鍵會設定為字串`CreateFile`或`FlushWithClose`。

* 此`contentOffset`索引鍵包含在資料集內。

> [!NOTE]
> 如果應用程式使用`PutBlockList`作業將新的 blob 上傳至帳戶, 資料就不會包含這些變更。

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "text/plain",
    "contentLength": 0,
    "contentOffset": 0,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/new-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event"></a>BlobDeleted 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/file-to-delete.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event-data-lake-storage-gen2"></a>BlobDeleted 事件 (Data Lake Storage Gen2)

如果 blob 儲存體帳戶具有階層式命名空間, 資料看起來會類似先前的範例, 但這些變更除外:

* 此`dataVersion`索引鍵會設定為的`2`值。

* 此`data.api`索引鍵會設定為字串`DeleteFile`。

* 金鑰包含路徑`dfs.core.windows.net`。 `url`

> [!NOTE]
> 如果應用程式使用`DeleteBlob`作業從帳戶中刪除 blob, 則資料不會包含這些變更。

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
    "api": "DeleteFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/file-to-delete.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobrenamed-event"></a>BlobRenamed 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-file.txt",
  "eventType": "Microsoft.Storage.BlobRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-file.txt",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorycreated-event"></a>DirectoryCreated 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-new-directory",
  "eventType": "Microsoft.Storage.DirectoryCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-new-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectoryrenamed-event"></a>DirectoryRenamed 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-directory",
  "eventType": "Microsoft.Storage.DirectoryRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-directory",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorydeleted-event"></a>DirectoryDeleted 事件

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/directory-to-delete",
  "eventType": "Microsoft.Storage.DirectoryDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "DeleteDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/directory-to-delete",
    "recursive": "true", 
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

## <a name="event-properties"></a>事件屬性

事件具有下列的最高層級資料：

| 屬性 | 類型 | 描述 |
| -------- | ---- | ----------- |
| topic | string | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| subject | string | 發行者定義事件主體的路徑。 |
| eventType | string | 此事件來源已註冊的事件類型之一。 |
| eventTime | string | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | string | 事件的唯一識別碼。 |
| data | object | blob 儲存體帳戶。 |
| dataVersion | string | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | string | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

資料物件具有下列屬性：

| 屬性 | 類型 | 描述 |
| -------- | ---- | ----------- |
| api | string | 觸發事件的作業。 |
| clientRequestId | string | 用於儲存體 API 作業的用戶端提供要求識別碼。 此識別碼可用來在記錄檔中使用「用戶端要求識別碼」欄位與 Azure 儲存體診斷記錄相互關聯, 並可在使用「x-ms-用戶端要求-識別碼」標頭的用戶端要求中提供。 請參閱[記錄格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。 |
| requestId | string | 儲存體 API 作業由服務產生的要求識別碼。 可用於利用記錄中的 "request-id-header" 欄位與 Azure 儲存體診斷記錄建立關聯，並從 'x-ms-request-id' 標頭中的 API 呼叫初始化傳回。 請參閱[記錄格式](https://docs.microsoft.com/rest/api/storageservices/storage-analytics-log-format)。 |
| eTag | string | 此值可讓您依條件執行作業。 |
| contentType | string | 為 blob 指定內容類型。 |
| contentLength | integer | Blob 大小 (以位元組為單位)。 |
| blobType | string | Blob 的類型。 有效值為 "BlockBlob" 或 "PageBlob"。 |
| contentOffset | number | 在事件觸發應用程式完成寫入檔案時, 所採取之寫入作業的位移 (以位元組為單位)。 <br>只有在具有階層命名空間的 blob 儲存體帳戶上觸發的事件才會出現。|
| destinationUrl |string | 在作業完成之後, 將會存在的檔案的 url。 例如, 如果檔案已重新命名, 則`destinationUrl`屬性會包含新檔案名的 url。 <br>只有在具有階層命名空間的 blob 儲存體帳戶上觸發的事件才會出現。|
| sourceUrl |string | 作業之前存在之檔案的 url。 例如, 如果檔案已重新命名, 則`sourceUrl`會包含重新命名作業之前原始檔案名稱的 url。 <br>只有在具有階層命名空間的 blob 儲存體帳戶上觸發的事件才會出現。 |
| url | string | blob 的路徑。 <br>如果用戶端使用 Blob REST API, 則 url 會有下列結構:  *\< \>blob.core.windows.net/\< /容器名稱檔案名\> \< \>* . <br>如果用戶端使用 Data Lake Storage REST API, 則 url 會有下列結構:  *\< / \> \> \<dfs.core.windows.net/\<檔----name file-name\>* . |
| recursive | string | `True`在所有子目錄上執行作業;否則`False`為。 <br>只有在具有階層命名空間的 blob 儲存體帳戶上觸發的事件才會出現。 |
| sequencer | string | 不透明的字串值表示任何特定 Blob 名稱之事件的邏輯順序。  使用者可使用標準字串比較，以了解 Blob 名稱相同之兩個事件的相對順序。 |
| storageDiagnostics | object | Azure 儲存體服務偶爾包含診斷資料。 出現時，事件消費者應該予以忽略。 |
|屬性|類型|描述|
|-------------------|------------------------|-----------------------------------------------------------------------|

## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
* 如需有關使用 blob 儲存體事件的簡介，請參閱[路由 blob 儲存體事件 - Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)。 
