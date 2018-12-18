---
title: Azure 傳訊服務比較
description: 比較 Azure Event Grid、事件中樞，以及服務匯流排。 建議針對不同案例挑選服務。
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.topic: overview
ms.date: 07/05/2018
ms.author: tomfitz
ms.openlocfilehash: c54f5c5fba31933a6dd9f98d1ecfb6d03a3ad9be
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003742"
---
# <a name="choose-between-azure-services-that-deliver-messages"></a>在傳遞訊息的 Azure 服務之間進行選擇

Azure 提供三種服務，協助在整個解決方案中傳遞事件訊息。 這些服務為：

* [Event Grid](/azure/event-grid/)
* [事件中樞](/azure/event-hubs/)
* [服務匯流排](/azure/service-bus-messaging/)

雖然彼此有一些相似之處，但每個服務皆針對特殊情況所設計。 本文說明這些服務之間的差異，並協助您了解應當為您的應用程式選擇哪一個服務。 在許多情況下，傳訊服務是互補的，而且可以共同使用。

## <a name="event-vs-message-services"></a>事件與訊息服務

傳遞事件的服務和傳遞訊息的服務之間有一個重要的區別。

### <a name="event"></a>Event

事件是狀況或狀態變更的輕量級通知。 事件發行者不會預期事件的處理方式。 事件取用者會決定要如何處理通知。 事件可以是離散單位或系列的一部分。

離散事件會報告狀態變更並可採取動作。 若要採取下一個步驟，取用者只需要知道發生狀況。 事件資料包含所發生事件的資訊，但沒有觸發該事件的資料。 例如，事件會通知取用者檔案已建立。 通知中可能具有關於檔案的一般資訊，但不包含檔案本身。 離散事件非常適合需要調整的無伺服器解決方案。

系列事件會報告狀況且可進行分析。 事件會依時間排序且相互關聯。 取用者需要循序的事件系列，才能分析所發生的狀況。

### <a name="message"></a>訊息

訊息是由待取用或儲存在其他地方的服務所產生的未經處理資料。 此訊息包含觸發訊息管線的資料。 訊息的發行者會預期取用者處理訊息的方式。 兩端之間有合約存在。 例如，發行者會傳送包含未經處理資料的訊息，並預期取用者從該資料建立檔案並且在工作完成時傳送回應。

## <a name="comparison-of-services"></a>服務比較

| 服務 | 目的 | 類型 | 使用時機 |
| ------- | ------- | ---- | ----------- |
| Event Grid | 回應式程式設計 | 事件散發 (離散) | 回應狀態變更 |
| 事件中樞 | 巨量資料管線 | 事件串流 (系列) | 遙測和分散式資料流 |
| 服務匯流排 | 高價值的企業傳訊 | 訊息 | 訂單處理和財務交易 |

### <a name="event-grid"></a>Event Grid

Event Grid 是可供進行回應式事件導向程式設計的事件背板。 其使用的是發行 - 訂閱模型。 發行者發布事件，但不會預期要處理哪些事件。 由訂閱者決定要處理的事件。

Event Grid 不僅與 Azure 服務緊密整合，並可與第三方服務整合。 它可簡化事件取用，並且無須不斷輪詢來降低成本。 Event Grid 有效且可靠地從 Azure 和非 Azure 資源中路由事件。 其會將事件發佈至已註冊的訂閱者端點。 事件訊息中具有您必須對服務和應用程式中的變更做出回應的資訊。 事件方格不是資料管線，因此並不會傳遞已更新的實際物件。

其具有下列特性︰

* 可動態擴充
* 低成本
* 無伺服器
* 至少一次傳遞

### <a name="event-hubs"></a>事件中樞

Azure 事件中樞是巨量資料管線。 它有助於擷取、保留和重送遙測和事件串流資料。 資料可來自許多並行來源。 事件中樞可讓遙測和事件資料提供給各種資料流處理基礎結構和分析服務使用。 可供資料流或配套的事件批次使用。 此服務提供單一解決方案，可啟用快速資料擷取以供即時處理，也可重複重送所儲存的未經處理資料。 它可以將串流資料擷取到檔案中，以進行處理及分析。

其具有下列特性︰

* 低延遲
* 每秒可接收和處理數百萬個事件
* 至少一次傳遞

### <a name="service-bus"></a>服務匯流排

服務匯流排適用於傳統的企業應用程式。 這些企業應用程式需要交易、排序、重複偵測和瞬間一致性。 服務匯流排可讓雲端原生應用程式提供可靠的狀態轉換管理以供業務處理使用。 要處理不可遺失或重複的高價值訊息時，請使用 Azure 服務匯流排。 服務匯流排也可以協助混合式雲端解決方案之間高度安全的通訊，並可將現有的內部部署系統連接到雲端解決方案。

服務匯流排是代理傳訊系統。 它可將訊息儲存在「訊息代理程式」(例如，佇列) 中，直到取用方準備接收這些訊息為止。

其具有下列特性︰

* 需要輪詢之可靠的非同步訊息傳遞 (企業傳訊即服務)
* 進階傳訊功能，例如 FIFO、批次處理/工作階段、交易、無效信件處理、暫存控制項、路由和篩選，以及重複偵測
* 至少一次傳遞
* 選擇性依序傳遞

## <a name="use-the-services-together"></a>一起使用服務

在某些情況下，您可以同時使用這些服務以滿足不同角色。 例如，電子商務網站可以使用服務匯流排來處理訂單，使用事件中樞來擷取站台遙測，使用 Event Grid 以回應事件 (例如運送項目)。

在其他情況，也可將各種服務連結在一起以形成事件和資料管線。 可以使用 Event Grid 來回應其他服務中的事件。 如需使用 Event Grid 與事件中樞將資料移轉至資料倉儲的範例，請參閱[將巨量資料串流至資料倉儲](event-grid-event-hubs-integration.md)。 下圖顯示串流資料的工作流程。

![串流資料概觀](./media/compare-messaging-services/overview.png)

## <a name="next-steps"></a>後續步驟

* 如需有關 Azure 傳訊服務的詳細資訊，請參閱部落格文章[事件、資料點和訊息 - 為您的資料選擇正確的 Azure 傳訊服務](https://azure.microsoft.com/blog/events-data-points-and-messages-choosing-the-right-azure-messaging-service-for-your-data/)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。
* 若要開始使用事件中樞，請參閱[使用 Azure 入口網站建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。
* 若要開始使用服務匯流排，請參閱[使用 Azure 入口網站建立服務匯流排命名空間](../service-bus-messaging/service-bus-create-namespace-portal.md)。
