---
title: Azure Event Grid 服務匯流排事件結構描述
description: 描述 Azure Event Grid 中服務匯流排事件的屬性
services: event-grid
author: banisadr
manager: darosa
ms.service: event-grid
ms.topic: reference
ms.date: 08/17/2018
ms.author: babanisa
ms.openlocfilehash: afb85f20c49821ca98e078791730a3376198e9e5
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42140481"
---
# <a name="azure-event-grid-event-schema-for-service-bus"></a>服務匯流排的 Azure Event Grid 事件結構描述

本文提供服務匯流排事件的屬性與結構描述。 如需事件結構描述的簡介，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。

如需範例指令碼和教學課程的清單，請參閱[服務匯流排事件來源](event-sources.md#service-bus)。

## <a name="available-event-types"></a>可用的事件類型

服務匯流排會發出下列事件類型：

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners | 當佇列或訂用帳戶有作用中的訊息但沒有接聽的接收者時，就會引發。 |
| Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener | 當無效信件佇列中有作用中訊息但沒有作用中的接聽程式時，就會引發。 |

## <a name="example-event"></a>事件範例

下列範例會顯示不包含接聽程式事件的作用中訊息結構描述：

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

無效信件佇列事件的結構描述如下：

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
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
| namespaceName | 字串 | 資源所在的服務匯流排命名空間。 |
| requestUri | 字串 | 要發出此事件的特定佇列或訂用帳戶 URI。 |
| entityType | 字串 | 發出事件 (佇列或訂閱) 的服務匯流排實體類型。 |
| queueName | 字串 | 如果訂閱佇列則為作用中訊息佇列。 如果使用主題 / 訂用帳戶則為 null 值。 |
| topicName | 字串 | 包含作用中訊息所屬之服務匯流排訂用帳戶的主題。 如果使用佇列則為 null 值。 |
| subscriptionName | 字串 | 具有作用中訊息的服務匯流排訂用帳戶。 如果使用佇列則為 null 值。 |

## <a name="next-steps"></a>後續步驟

* 如需 Azure Event Grid 的簡介，請參閱[什麼是 Event Grid？](overview.md)
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
* 如需使用 Azure Event Grid 搭配服務匯流排的詳細資訊，請參閱[服務匯流排與 Event Grid 的整合概觀](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md)。
* 嘗試[接收 Functions 或 Logic Apps 的服務匯流排事件](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json).
