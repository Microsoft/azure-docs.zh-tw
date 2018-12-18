---
title: Azure 服務匯流排進階和標準傳訊定價層概觀 | Microsoft Docs
description: 服務匯流排進階和標準傳訊層級
services: service-bus-messaging
documentationcenter: .net
author: djrosanova
manager: timlt
editor: ''
ms.assetid: e211774d-821c-4d79-8563-57472d746c58
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/30/2018
ms.author: spelluru
ms.openlocfilehash: bde5629fdd500896e557f89ce6b819169366df97
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407678"
---
# <a name="service-bus-premium-and-standard-messaging-tiers"></a>服務匯流排進階和標準傳訊層級

服務匯流排傳訊 (包含佇列和主題等實體) 結合雲端級別的企業傳訊功能與豐富的發佈-訂閱語意。 服務匯流排傳訊作為許多縝密雲端解決方案的通訊骨幹。

服務匯流排傳訊的「進階」層級可滿足一般客戶對於關鍵任務應用程式的規模、效能和可用性要求。 針對生產案例，建議使用進階層。 雖然功能組幾乎完全相同，但這兩層級的服務匯流排傳訊設計用來服務不同的使用案例。

下表強調大致的一些差異。

| 進階 | 標準 |
| --- | --- |
| 高輸送量 |變動輸送量 |
| 可預測的效能 |變動延遲 |
| 固定價格 |隨用隨付變動價格 |
| 相應增加和相應減少工作負載的能力 |N/A |
| 訊息大小上限為 1 MB |訊息大小上限為 256 KB |

**服務匯流排進階傳訊**提供 CPU 和記憶體層級的資源隔離，讓每個客戶工作負載能隔離執行。 此資源容器稱為「傳訊單位」 。 每個進階命名空間都會被配置至少一個傳訊單位。 您可以為每個服務匯流排進階命名空間購買 1、2 或 4 個傳訊單位。 單一工作負載或實體可以跨越多個傳訊單位，而傳訊單位數目可以隨意變更，雖然計費是依 24 小時或每日費率收費。 結果是您的服務匯流排方案的效能可預測並可重複。

此效能不僅更可預測並可取得，而且還更快速。 服務匯流排進階傳訊是以 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)中引進的儲存引擎為建置基礎。 使用進階訊息，尖峰效能的速度比標準層級的速度快很多。

## <a name="premium-messaging-technical-differences"></a>進階傳訊技術差異

下列各節討論進階與標準傳訊層之間的一些差異。

### <a name="partitioned-queues-and-topics"></a>分割的佇列和主題

進階傳訊中不支援分割的佇列和主題。 如需分割的詳細資訊，請參閱[分割的佇列和主題](service-bus-partitioning.md)。

### <a name="express-entities"></a>快速實體

因為進階傳訊是在完全隔離的執行階段環境中執行，所以在進階命名空間中並不支援快速實體。 如需快速功能的詳細資訊，請參閱 [QueueDescription.EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress) 屬性。

如果您的程式碼是在標準傳訊下執行，而您想要將它移植到高階層，請確定 [EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress) 屬性設為 **false** (預設值)。

## <a name="get-started-with-premium-messaging"></a>開始使用進階傳訊

開始使用進階訊息非常簡單，其程序類似於標準傳訊。 首先在 [Azure 入口網站](https://portal.azure.com)中[建立命名空間](service-bus-create-namespace-portal.md)。 請務必選取 [定價層] 之下的 [進階]。 按一下 [檢視完整定價詳細資料] 以查看每一層的詳細資訊。

![create-premium-namespace][create-premium-namespace]

您也可以[使用 Azure Resource Manager 範本建立進階命名空間](https://azure.microsoft.com/resources/templates/101-servicebus-pn-ar/)。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列連結：

* [Azure 服務匯流排進階傳訊簡介 (部落格文章)](http://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
* [Azure 服務匯流排進階傳訊簡介 (Channel9)](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)
* [服務匯流排訊息概觀](service-bus-messaging-overview.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)

<!--Image references-->

[create-premium-namespace]: ./media/service-bus-premium-messaging/select-premium-tier.png
