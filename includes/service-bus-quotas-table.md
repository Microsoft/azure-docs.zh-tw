---
title: 包含檔案
description: 包含檔案
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 08/29/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 481ae07ae9f8877ff93b2fee948849076c054906
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43187024"
---
下表列出服務匯流排訊息的特定配額資訊。 如需有關服務匯流排的價格及其他配額的詳細資訊，請參閱 [服務匯流排價格](https://azure.microsoft.com/pricing/details/service-bus/) 概觀。

| 配額名稱 | 影響範圍 | 注意 | 值 |
| --- | --- | --- | --- | --- |
| 每個 Azure 訂用帳戶的基本/標準命名空間數目上限 |命名空間 |入口網站會拒絕後續對於更多基本/標準命名空間的要求。 |100|
| 每個 Azure 訂用帳戶的進階命名空間數目上限 |命名空間 |入口網站會拒絕後續對於更多進階命名空間的要求。 |10 |
| 佇列/主題大小 |實體 |在建立佇列/主題時定義。 <br/><br/> 後續內送訊息會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 |1、2、3、4 GB 或 5 GB。<br /><br />在進階 SKU，以及啟用[分割](/azure/service-bus-messaging/service-bus-partitioning)的標準 SKU 中，佇列/主題大小上限為 80 GB。 |
| 命名空間上的並行連線數目 |命名空間 |後續對更多連線的要求將會遭到拒絕，而且呼叫端程式碼將會收到例外狀況。 REST 作業不會計入並行 TCP 連線內。 |NetMessaging: 1,000<br /><br />AMQP: 5,000 |
| 佇列/主題/訂用帳戶實體上的並行接收要求數目 |實體 |後續接收要求會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 這個配額套用至一個主題的所有訂用帳戶的並行接收作業數目合計。 |5,000 |
| 每個命名空間的主題/佇列數目 |命名空間 |後續在命名空間上建立新主題或佇列的要求都會遭到拒絕。 因此，如果透過 [Azure 入口網站][Azure portal]設定，則會產生錯誤訊息。 如果從管理 API 進行呼叫，則呼叫端程式碼會收到例外狀況。 |10,000 (基本/標準層)。 命名空間中主題和佇列的總數必須小於或等於 10,000。 <br/><br/>在進階中，則是每個傳訊單位 (MU) 1000 個。 上限為 4000。 |
| 每個命名空間的[分割主題/佇列](/azure/service-bus-messaging/service-bus-partitioning)數目 |命名空間 |後續要求在命名空間上建立新分割主題或佇列會遭到拒絕。 因此，如果透過 [Azure 入口網站][Azure portal]設定，則會產生錯誤訊息。 如果從管理 API 進行呼叫，則呼叫端程式碼會收到 **QuotaExceededException** 例外狀況。 |基本層和標準層 - 100<br/><br/>[Premium](../articles/service-bus-messaging/service-bus-premium-messaging.md) 層不支援分割的實體。<br/><br />每個分割佇列或主題的配額計數為每個命名空間 10,000 個實體。 |
| 任何傳訊實體路徑的大小上限︰佇列或主題 |實體 |- |260 個字元 |
| 任何傳訊實體名稱的大小上限︰命名空間、訂用帳戶或訂用帳戶規則 |實體 |- |50 個字元 |
| [SessionID](/dotnet/api/microsoft.azure.servicebus.message.sessionid) 訊息的大小上限 | 實體 |- | 128 |
| 佇列/主題/訂用帳戶實體的訊息大小 |實體 |超出這些配額的內送訊息將會遭到拒絕，而且呼叫端程式碼將會收到例外狀況。 |訊息大小上限︰256 KB ([標準層](../articles/service-bus-messaging/service-bus-premium-messaging.md)) / 1MB ([進階層](../articles/service-bus-messaging/service-bus-premium-messaging.md))。 <br /><br />由於系統額外負荷，所以此限制小於這些值。<br /><br />標頭大小上限︰64 KB<br /><br />屬性包中的標頭屬性數目上限︰**byte/int.MaxValue**<br /><br />屬性包中的屬性大小上限︰沒有明確限制。 受限於標頭大小上限。 |
| 佇列/主題/訂用帳戶實體的訊息屬性大小 |實體 |已產生 **SerializationException** 例外狀況。 |每個屬性的訊息屬性大小上限為 32 K。所有屬性的累積大小不能超過 64 K。此限制適用於 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 的整個標頭，其中同時包含使用者屬性和系統屬性 (例如 [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber)、[Label](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.label)、[MessageId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.messageid) 等等)。 |
| 每個主題的訂用帳戶數目 |實體 |後續要求建立主題的其他訂用帳戶會遭到拒絕。 因此，如果透過入口網站設定，則會顯示錯誤訊息。 如果從管理 API 進行呼叫，則呼叫端程式碼會收到例外狀況。 |2,000 |
| 每個主題的 SQL 篩選器數目 |實體 |後續要求在主題上建立其他篩選器都會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 |2,000 |
| 每個主題的相互關聯篩選器數目 |實體 |後續要求在主題上建立其他篩選器都會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 |100,000 |
| SQL 篩選器/動作的大小 |命名空間 |後續要求建立其他篩選器都會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 |篩選條件字串的長度上限︰1024 (1 K)。<br /><br />規則動作字串的長度上限︰1024 (1 K)。<br /><br />每個規則動作的運算式數目上限︰32。 |
| 每個命名空間、佇列或主題的 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 規則數目 |實體、命名空間 |後續要求建立其他規則都會遭到拒絕，而且呼叫端程式碼會收到例外狀況。 |規則數目上限︰12。 <br /><br /> 在服務匯流排命名空間上設定的規則可套用到該命名空間中的所有佇列和主題。 |
| 每個交易的訊息數目 | 交易 | 其他內送訊息都會遭到拒絕，而且呼叫端程式碼會收到例外狀況，指出「無法在單一交易中傳送超過 100 則訊息」。 | 100 <br /><br /> 適用於 **Send()** 和 **SendAsync()** 作業。 |

[Azure portal]: https://portal.azure.com