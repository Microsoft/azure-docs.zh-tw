---
title: Azure 服務匯流排訊息延遲 | Microsoft Docs
description: 服務匯流排訊息的延遲傳遞
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
ms.date: 01/26/2018
ms.author: spelluru
ms.openlocfilehash: f9dbd663ce3b15e6a825f0ef73f3dd5d1f5df76b
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697538"
---
# <a name="message-deferral"></a>訊息延遲

當佇列或訂用帳戶用戶端接收到其願意處理的訊息，但處理目前因為應用程式內部的特殊情況而無法進行時，它可以選擇「延遲」到稍後某個時點才擷取訊息。 訊息會保留於佇列或訂用帳戶中，但會將它擱置於一旁。

延遲是專為工作流程處理案例而建立的功能。 工作流程架構可能需要以特定順序處理某些作業，並且可能必須延後處理某些已接收的訊息，直到其他訊息通知指定的先前工作已完成為止。

這裡提供的簡單說明範例是訂單處理順序，其中來自外部付款提供者的付款通知會在將相符的採購單從存放區前端傳播到履行系統之前出現在系統中。 在此情況下，履行系統可能會延遲處理付款通知，直到沒有與它相關聯的訂單為止。 在集結的案例 (來自不同來源的訊息會衍生工作流程轉送) 中，即時執行順序可能確實是正確的，但反映結果的訊息可能不會依序送達。

歸根結底，延遲有助於將訊息從送達的順序重新排序為可處理它們的順序，同時還能讓那些訊息安全地保留於需要延後處理的訊息存放區中。

## <a name="message-deferral-apis"></a>訊息延遲 API

API 是 .NET Framework 用戶端中的 [BrokeredMessage.Defer](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.defer?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_Defer) 或 [BrokeredMessage.DeferAsync](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deferasync?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_DeferAsync)、.NET 標準用戶端中的 [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync)，以及 Java 用戶端中的 **mesageReceiver.defer** 或 **messageReceiver.deferSync**。 

延遲的訊息會和所有其他作用中的訊息一起保留於主要佇列中 (與存留於子佇列中屬無效信件的訊息不同)，但無法再使用一般的 Receive/ReceiveAsync 函式來接收它們。 如果應用程式無法追蹤延遲的訊息，則可透過[訊息瀏覽](message-browsing.md)來探索它們。

若要擷取延遲的訊息，其擁有者會負責記住 [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber)，因為它會延遲該訊息。 任何知道延遲訊息序號的接收者稍後都可使用 `Receive(sequenceNumber)` 明確地接收訊息。

如果訊息因為用於處理該訊息的特殊資源暫時無法使用而無法處理，但不應立即暫止訊息處理，則用來將該訊息放在一邊數分鐘的方法是，記住要在數分鐘內發佈之[已排程的訊息](message-sequencing.md)中的 **SequenceNumber**，並在已排程的訊息送達時，重新擷取延遲的訊息。 請注意，如果訊息處理常式取決於適用於所有作業的資料庫，而該資料庫暫時無法使用，則它不應使用延遲，而是在資料庫再次可供使用之前，完全暫停接收訊息。

延遲訊息不會影響訊息到期，這表示延遲的訊息仍然會在一開始排定的時間到期，然後移到無效信件佇列中 (如果是如此設定)。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)