---
title: 將 Azure 事件方格用於 CloudEvents 結構描述中的事件
description: 說明如何在 Azure 事件方格中設定事件的 CloudEvents 結構描述。
services: event-grid
author: banisadr
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 07/13/2018
ms.author: babanisa
ms.openlocfilehash: 4f1f0e95ae74ef41ed91be55f4c964671e8f723b
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39044544"
---
# <a name="use-cloudevents-schema-with-event-grid"></a>透過事件方格使用 CloudEvents 結構描述

除了[預設事件結構描述](event-schema.md)以外，Azure 事件方格原本也就支援 [CloudEvents JSON 結構描述](https://github.com/cloudevents/spec/blob/master/json-format.md)中的事件。 [CloudEvents](http://cloudevents.io/) 是用來以常見的方式說明事件資料的[開放式標準規格](https://github.com/cloudevents/spec/blob/master/spec.md)。

CloudEvents 提供用以發佈和取用雲端型事件的常見事件結構描述，可簡化互通性。 此結構描述可支援統一的工具、路由和處理事件的標準方式，以及將外部事件結構描述還原序列化的通用方式。 透過通用結構描述，您將可更輕鬆地跨平台整合工作。

目前有數個[共同作業者](https://github.com/cloudevents/spec/blob/master/community/contributors.md) (包括 Microsoft) 正透過 [Cloud Native Compute Foundation](https://www.cncf.io/) 建置 CloudEvents。 目前可用的版本為 0.1。

本文說明如何透過事件方格使用 CloudEvents 結構描述。

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="cloudevent-schema"></a>CloudEvent 結構描述

下列範例說明 CloudEvents 格式的 Azure Blob 儲存體事件：

``` JSON
{
    "cloudEventsVersion" : "0.1",
    "eventType" : "Microsoft.Storage.BlobCreated",
    "eventTypeVersion" : "",
    "source" : "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}#blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "eventID" : "173d9985-401e-0075-2497-de268c06ff25",
    "eventTime" : "2018-04-28T02:18:47.1281675Z",
    "data" : {
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
    }
}
```

CloudEvents v0.1 具有下列可用屬性：

| CloudEvents        | 類型     | 範例 JSON 值             | 說明                                                        | 事件方格對應
|--------------------|----------|--------------------------------|--------------------------------------------------------------------|-------------------------
| eventType          | 字串   | "com.example.someevent"          | 發生的事件類型                                   | eventType
| eventTypeVersion   | 字串   | "1.0"                            | eventType 的版本 (選用)                            | dataVersion
| cloudEventsVersion | 字串   | "0.1"                            | 事件使用之 CloudEvents 規格的版本        | *已傳遞*
| 來源             | URI      | "/mycontext"                     | 說明事件產生者                                       | topic#subject
| eventID            | 字串   | "1234-1234-1234"                 | 事件的識別碼                                                    | id
| eventTime          | Timestamp| "2018-04-05T17:31:00Z"           | 事件發生時的時間戳記 (選用)                    | eventTime
| schemaURL          | URI      | "https://myschema.com"           | 資料屬性所符合之結構描述的連結 (選用) | *未使用*
| contentType        | 字串   | "application/json"               | 說明資料編碼格式 (選用)                       | *未使用*
| 擴充功能         | 對應      | { "extA": "vA", "extB", "vB" }  | 任何其他中繼資料 (選用)                                 | *未使用*
| data               | Object   | { "objA": "vA", "objB", "vB" }  | 事件承載 (選用)                                       | data

如需詳細資訊，請參閱 [CloudEvents 規格](https://github.com/cloudevents/spec/blob/master/spec.md#context-attributes)。

在 CloudEvents 結構描述中傳遞的事件標頭值與事件方格結構描述的該值相同，不同之處在於 `content-type`。 針對 CloudEvents 結構描述，該標頭值是 `"content-type":"application/cloudevents+json; charset=utf-8"`。 針對事件方格結構描述，該標頭值是 `"content-type":"application/json; charset=utf-8"`。

## <a name="configure-event-grid-for-cloudevents"></a>設定 CloudEvents 的事件方格

您可以使用事件方格來輸入和輸出 CloudEvents 結構描述中的事件。 您可以將 CloudEvents 用於系統事件 (例如 Blob 儲存體事件和 IoT 中樞事件) 及自訂事件。 它也可以在線上來回轉換這些事件。


| 輸入結構描述       | 輸出結構描述
|--------------------|---------------------
| CloudEvents 格式 | CloudEvents 格式
| 事件方格格式  | CloudEvents 格式
| CloudEvents 格式 | 事件方格格式
| 事件方格格式  | 事件方格格式

對於所有的事件結構描述，事件方格在發佈至事件方格主題和建立事件訂閱時，都需要進行驗證。 如需詳細資訊，請參閱 [Event Grid 安全性和驗證](security-authentication.md)。

### <a name="input-schema"></a>輸入結構描述

若要將自訂主題的輸入結構描述設為 CloudEvents，請在建立自訂主題 `--input-schema cloudeventv01schema` 時，在 Azure CLI 中使用下列參數。 自訂主題現在會預期 CloudEvents v0.1 格式的傳入事件。

若要建立事件方格主題，請使用：

```azurecli
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid topic create \
  --name <topic_name> \
  -l westcentralus \
  -g gridResourceGroup \
  --input-schema cloudeventv01schema
```

CloudEvents 的目前版本不支援事件的批次處理。 若要透過 CloudEvent 結構描述將事件發佈至主題，請個別發佈每個事件。

### <a name="output-schema"></a>輸出結構描述

若要將事件訂閱的輸出結構描述設為 CloudEvents，請在建立事件訂閱 `--event-delivery-schema cloudeventv01schema` 時，在 Azure CLI 中使用下列參數。 此事件訂閱的事件現在會以 CloudEvents v0.1 格式傳遞。

若要建立事件訂閱，請使用：

```azurecli
az eventgrid event-subscription create \
  --name <event_subscription_name> \
  --topic-name <topic_name> \
  -g gridResourceGroup \
  --endpoint <endpoint_URL> \
  --event-delivery-schema cloudeventv01schema
```

CloudEvents 的目前版本不支援事件的批次處理。 為 CloudEvent 結構描述設定的事件訂閱會個別接收每個事件。 目前，當事件是在 CloudEvents 結構描述中傳遞時，您無法針對 Azure Functions 應用程式使用事件方格觸發程序。 您必須使用 HTTP 觸發程序。 如需實作 HTTP 觸發程序 (該觸發程序會接收 CloudEvents 結構描述中的事件) 的範例，請參閱[使用 HTTP 觸發程序作為事件方格觸發程序](../azure-functions/functions-bindings-event-grid.md#use-an-http-trigger-as-an-event-grid-trigger)。

## <a name="next-steps"></a>後續步驟

* 如需關於監視事件傳遞的資訊，請參閱[監視 Event Grid 訊息傳遞](monitor-event-delivery.md)。
* 建議您測試、評論和[參與](https://github.com/cloudevents/spec/blob/master/CONTRIBUTING.md) CloudEvents。
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
