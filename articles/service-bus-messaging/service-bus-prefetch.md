---
title: Azure 服務匯流排預先擷取訊息 | Microsoft Docs
description: 藉由預先擷取 Azure 服務匯流排訊息來提升效能。
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
ms.date: 08/30/2018
ms.author: spelluru
ms.openlocfilehash: 92a5a90424de56b5cef97b790c6f8b3685fa833f
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47410165"
---
# <a name="prefetch-azure-service-bus-messages"></a>預先擷取 Azure 服務匯流排訊息

在任何官方的服務匯流排用戶端上啟用「預先擷取」時，接收者就會以無訊息方式取得更多訊息 (最多可達 [PrefetchCount](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount#Microsoft_Azure_ServiceBus_QueueClient_PrefetchCount) 限制)，超出應用程式一開始所要求的範圍。

因此，單一起始的 [Receive](/dotnet/api/microsoft.servicebus.messaging.queueclient.receive) 或 [ReceiveAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receiveasync) 呼叫會取得訊息，以便在其成為可用而儘速傳回時立即取用。 接著，用戶端會在背景中取得後續訊息來填滿預先擷取緩衝區。

## <a name="enable-prefetch"></a>啟用預先擷取

透過 .NET，您可以藉由將 **MessageReceiver**、**QueueClient** 或 **SubscriptionClient** 的 [PrefetchCount](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount#Microsoft_Azure_ServiceBus_QueueClient_PrefetchCount) 屬性設為大於零的數字，來啟用預先擷取功能。 將值設定為零會關閉預先擷取。

您可以輕鬆地將此設定新增至 [QueuesGettingStarted](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/QueuesGettingStarted) \(英文\) 或 [ReceiveLoop](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/ReceiveLoop) \(英文\) 範例設定的接收端，即可在那些內容中看見效果。

當訊息在預先擷取緩衝區中可供使用時，就會從緩衝區中立即履行任何後續的 **Receive**/**ReceiveAsync** 呼叫，而緩衝區會在空間變成可用時於背景中加以補充。 如果沒有任何訊息可供傳遞，接收作業就會如預期般清空緩衝區，然後加以等候或封鎖。

預先擷取也會以 [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage) 和 [OnMessageAsync](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessageasync) API 中的相同方式來運作。

## <a name="if-it-is-faster-why-is-prefetch-not-the-default-option"></a>如果預先擷取速度更快，為什麼它不是預設選項？

預先擷取加速訊息流程的方式是，在應用程式要求某個訊息時和之前，先將訊息備妥以供本機擷取。 這個輸送量提升是應用程式作者必須明確進行取捨的結果：

利用 [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) 接收模式，所有擷取到預先擷取緩衝區的訊息便無法再於佇列中使用，而且只會位於記憶體中的預先擷取緩衝區，直到應用程式透過 **Receive**/**ReceiveAsync** 或 **OnMessage**/**OnMessageAsync** API 接收到它們為止。 如果應用程式在接收到訊息之前終止，則那些訊息將會永久遺失。

在 [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock) 接收模式中，會以鎖定狀態來將擷取到預先擷取緩衝區的訊息擷取到緩衝區，並且會有具鎖定期限的逾時時鐘。 如果預先擷取緩衝區夠大，而且會花這麼長的時間來處理位於預先擷取緩衝區的訊息鎖定到期，甚至是當應用程式正在處理訊息時的訊息鎖定到期，則可能會有一些複雜難懂的事件需要應用程式處理。

應用程式所取得的訊息可能含有過期或即將到期的鎖定。 若是如此，應用程式可能會處理該訊息，但接著為發現它因為鎖定到期而無法完成。 應用程式可以檢查 [LockedUntilUtc](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.lockeduntilutc) 屬性 (受限於訊息代理程式和本機電腦時鐘之間的時鐘誤差)。 如果訊息鎖定已過期，應用程式就必須忽略該訊息；不應對訊息進行任何 API 呼叫。 如果訊息尚未到期但即將到期，則可呼叫 [message.RenewLock()](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_) 來更新鎖定，並以另一個預設鎖定期間來延長鎖定。

如果預先擷取緩衝區中的鎖定會以無訊息方式到期，則該訊息會被視為已放棄，並可再次從佇列加以擷取。 那樣可能會導致它被擷取到預先擷取緩衝區；放置於結尾處。 假設在訊息到期期間通常無法處理預先擷取緩衝區，這會導致重複預先擷取訊息，但永遠不會以可用 (有效地鎖定) 狀態有效地進行傳遞，而且最終會在超過最大傳遞計數時，將其移至無效信件佇列中。

如果您需要進行高度可靠的訊息處理，而且處理需要相當多的工作和時間，則建議謹慎使用預先擷取功能，或者完全不使用。

如果您需要高輸送量且訊息處理通常很便宜，預先擷取就會產生顯著的輸送量優點。

預先擷取計數上限需要和佇列或訂用帳戶上設定的鎖定持續期間取得平衡，例如，鎖定逾時至少要超過針對預先擷取緩衝區的大小上限累積的預期訊息處理時間，再加上一個訊息。 同時，鎖定逾時不應這麼長，訊息會在不小心被卸除時超過其 [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) 上限，因此需要讓它們的鎖定到期，才能重新傳遞。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)
