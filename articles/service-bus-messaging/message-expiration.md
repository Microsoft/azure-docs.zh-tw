---
title: Azure 服務匯流排訊息到期 | Microsoft Docs
description: Azure 服務匯流排訊息的到期與存留時間
services: service-bus-messaging
documentationcenter: ''
author: clemensv
manager: timlt
editor: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2018
ms.author: spelluru
ms.openlocfilehash: c1c38370ae508dffc6888dd7e94514406b8da327
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405261"
---
# <a name="message-expiration-time-to-live"></a>訊息到期 (存留時間)

訊息內部的承載 (或是訊息傳遞給接收者的命令或查詢) 幾乎一律受限於某種形式之應用程式層級的到期期限。 在這類期限之後，系統便不再提供內容，或者不再執行要求的作業。

對於通常會在應用程式或應用程式組件的部分執行內容中使用佇列和主題的開發與測試環境，它也很適合用來將擱置的測試訊息自動進行記憶體回收，如此一來，就能全新開始下一個測試執行。

任何個別訊息的到期都能透過設定 [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) 系統屬性來控制，其會指定相對的持續期間。 將訊息加入實體的佇列時，到期就會變成絕對瞬間。 在那段時間，[ExpiresAtUtc](/dotnet/api/microsoft.azure.servicebus.message.expiresatutc) 屬性會採用值 [(**EnqueuedTimeUtc**](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc#Microsoft_ServiceBus_Messaging_BrokeredMessage_EnqueuedTimeUtc) + [**TimeToLive**)](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive)。

經過 **ExpiresAtUtc** 瞬間之後，訊息就會變成不適合用於擷取。 到期不會影響目前已鎖定來進行傳遞的訊息；仍會正常處理那些訊息。 如果鎖定到期或已放棄訊息，則到期將會立即生效。

儘管訊息處於鎖定狀態，但應用程式可能擁有已到期的訊息。 應用程式是否願意繼續進行處理，或選擇放棄訊息，都取決於實作者。

## <a name="entity-level-expiration"></a>實體層級的到期

傳送到佇列或主題的所有訊息都受限於預設的到期，其設定於具 [defaultMessageTimeToLive](/azure/templates/microsoft.servicebus/namespaces/queues) 屬性的實體層級，同時也可在建立期間於入口網站中加以設定且稍後進行調整。 預設的到期適用於所有傳送至實體且未明確設定 [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) 的訊息。 預設到期也可用來作為 **TimeToLive** 值的上限。 若訊息的 **TimeToLive** 到期比預設值還長，則會在加入佇列之前，以無訊息方式調整為 **defaultMessageTimeToLive** 值。

您可以藉由設定 [EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enabledeadletteringonmessageexpiration#Microsoft_ServiceBus_Messaging_QueueDescription_EnableDeadLetteringOnMessageExpiration) 屬性，或在入口網站中勾選相對的方塊，選擇性地將過期的訊息移到[無效信件佇列](service-bus-dead-letter-queues.md)。 如果將選項保留為已停用，即會卸除過期的訊息。 移至無效信件佇列的過期訊息可以藉由評估訊息代理程式儲存於使用者屬性區段的 [DeadletterReason](service-bus-dead-letter-queues.md#moving-messages-to-the-dlq) 屬性來區分；在此案例中，值為 [TTLExpiredException](service-bus-dead-letter-queues.md#moving-messages-to-the-dlq)。

在上述案例中，訊息會受到保護以免在鎖定下到期，而且如果在實體上設定旗標，則會在放棄鎖定或鎖定到期時，將訊息移到無效信件佇列。 不過，如果已成功安置訊息則不會加以移動，接著，儘管名義上已到期，還是假設應用程式已成功處理它。

關於期滿的 [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) 和自動執行 (和交易式) 無效信件組合是個非常重要的工具，讓您有信心在到達期限時，能夠擷取指定給某個處理常式或一組處理常式且未達期限的作業來處理。

例如，假設有個網站需要在有限範圍的後端上可靠地執行作業，而且偶爾會經歷流量尖峰，或者想要根據該後端的可用性事件來隔離。 在正常情況下，適用於已提交使用者資料的伺服器端處理常式會將資訊推入佇列，隨後會收到回覆，確認已成功將交易處理至回覆佇列中。 如果出現流量尖峰且後端處理常式無法即時處理其待處理項目，則會傳回無效信件佇列上的過期作業。 互動式使用者會收到通知，表示要求的作業需要比平常多一點的時間，接著可將要求放入處理路徑的不同佇列，以便透過電子郵件來將最終處理結果傳送給使用者。 

## <a name="temporary-entities"></a>暫時性實體

您可以將服務匯流排佇列、主題和訂用帳戶建立為暫時性實體，若經過一段指定的時間未使用它們，即會自動移除。
 
自動清除適用於開發和測試案例，在這類案例中會動態建立實體，並且不會在使用之後加以清除，因為測試或偵錯執行發生部分中斷。 當應用程式針對接收回到 Web 伺服器處理序的回應建立動態實體 (例如回覆佇列)，或者建立到另一個相對存留期較短的物件 (當物件執行個體消失時，就很難在其中可靠地清理那些實體) 時，也很適合使用。

使用 [autoDeleteOnIdle](/azure/templates/microsoft.servicebus/namespaces/queues) 屬性來啟用此功能，實體必須閒置 (未使用) 了此屬性所設定的持續期間之後才會自動刪除。 最小持續期間為 5 分鐘。
 
您必須透過 Azure Resource Manager 作業，或透過 .NET Framework 用戶端 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) API 來設定 **autoDeleteOnIdle** 屬性。 您無法透過入口網站來設定它。


## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)