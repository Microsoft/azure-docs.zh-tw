---
title: 服務匯流排寄不出的信件佇列 |Microsoft Docs
description: Azure 服務匯流排寄不出的信件佇列的概觀
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 68b2aa38-dba7-491a-9c26-0289bc15d397
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/26/2018
ms.author: spelluru
ms.openlocfilehash: df69406bd0a6e09f66262ce66f8eeb7cada2422a
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393974"
---
# <a name="overview-of-service-bus-dead-letter-queues"></a>服務匯流排寄不出的信件佇列的概觀

Azure 服務匯流排佇列和主題訂用帳戶提供次要的子佇列，稱為「無效信件佇列」(DLQ)。 寄不出的信件佇列並不需要明確建立，而且無法刪除或以其他方式在主要實體之外管理。

本文說明服務匯流排中的無效信件佇列。 大部分的討論內容是以 GitHub 上的[無效信件佇列範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/DeadletterQueue)加以說明。
 
## <a name="the-dead-letter-queue"></a>無效信件佇列

無效信件佇列的目的，是保留無法傳遞至任何收件者的訊息，或是無法加以處理的訊息。 隨後可以從 DLQ 移除訊息並加以檢查。 透過運算子的協助，應用程式可能會更正問題並重新提交訊息、記錄發生錯誤及採取更正動作。 

從 API 和通訊協定的觀點而言，DLQ 非常類似任何其他佇列，不同之處在於訊息只會透過父實體的無效信件作業提交。 此外，存留時間並未遵守，而且您無法從 DLQ 讓訊息寄不出去。 寄不出的信件佇列完全支援鎖定傳遞和交易式作業。

請注意，DLQ 沒有自動清除。 訊息會保留在 DLQ 中，直到您明確地從 DLQ 擷取訊息並在寄不出的信件訊息上呼叫 [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync)。

## <a name="moving-messages-to-the-dlq"></a>將訊息移至 DLQ

服務匯流排中有幾個活動會導致訊息從傳訊引擎本身內部推送至 DLQ。 應用程式也可以明確地將訊息移至 DLQ。 

由於訊息代理程式會移動訊息，當訊息代理程式在下列訊息上呼叫其內部版本的 [DeadLetter](/dotnet/api/microsoft.azure.servicebus.queueclient.deadletterasync) 方法時，會新增兩個屬性至訊息︰`DeadLetterReason` 和 `DeadLetterErrorDescription`。

應用程式可以將自己的程式碼定義為 `DeadLetterReason` 屬性，但是系統會設定下列值。

| 條件 | DeadLetterReason | DeadLetterErrorDescription |
| --- | --- | --- |
| 一律 |HeaderSizeExceeded |已超過這個串流的大小配額。 |
| !TopicDescription.<br />EnableFilteringMessagesBeforePublishing 和 SubscriptionDescription.<br />EnableDeadLetteringOnFilterEvaluationExceptions |exception.GetType().Name |exception.Message |
| EnableDeadLetteringOnMessageExpiration |TTLExpiredException |訊息已過期，且已停止傳送。 |
| SubscriptionDescription.RequiresSession |工作階段識別碼為 null。 |啟用工作階段的實體不允許工作階段識別項為 null 的訊息。 |
| ！寄不出的信件佇列 |MaxTransferHopCountExceeded |Null |
| 應用程式明確停止傳送 |應用程式所指定 |應用程式所指定 |

## <a name="exceeding-maxdeliverycount"></a>超過 MaxDeliveryCount

佇列和訂用帳戶分別具有 [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) 和 [SubscriptionDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.maxdeliverycount) 屬性；預設值為 10。 每當在鎖定下傳遞訊息 ([ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode))，但已明確放棄或鎖定已過期時，訊息的 [BrokeredMessage.DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 就會遞增。 當 [DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 超過 [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) 時，訊息會移到 DLQ，指定 `MaxDeliveryCountExceeded` 原因代碼。

無法停用此行為，但是您可以將 [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) 設定為非常大的數字。

## <a name="exceeding-timetolive"></a>超過 TimeToLive

當 [QueueDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_EnableDeadLetteringOnMessageExpiration) 或 [SubscriptionDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription#Microsoft_ServiceBus_Messaging_SubscriptionDescription_EnableDeadLetteringOnMessageExpiration) 屬性設為 **true** (預設值是 **false**)，所有過期的訊息會移到 DLQ，指定 `TTLExpiredException` 原因代碼。

請注意，至少要有一個作用中接收者從主要佇列或訂用帳戶提取時，過期的訊息才會清除並移至 DLQ；這是刻意設計的行為。

## <a name="errors-while-processing-subscription-rules"></a>在處理訂用帳戶規則時發生錯誤

為訂用帳戶啟用 [SubscriptionDescription.EnableDeadLetteringOnFilterEvaluationExceptions](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription#Microsoft_ServiceBus_Messaging_SubscriptionDescription_EnableDeadLetteringOnFilterEvaluationExceptions) 屬性時，可以在 DLQ 中擷取到執行訂用帳戶的 SQL 篩選器規則時發生的任何錯誤，還有騷擾訊息。

## <a name="application-level-dead-lettering"></a>應用程式層級無效信件處理

除了系統提供的無效信件處理功能之外，應用程式可以使用 DLQ 明確拒絕無法接受的訊息。 這包括由於任何類型的系統問題而無法正確處理的訊息、保存格式不正確之承載的訊息，或在使用某些訊息層級安全性配置時無法進行驗證的訊息。

## <a name="dead-lettering-in-forwardto-or-sendvia-scenarios"></a>ForwardTo 或 SendVia 案例中寄不出的信件處理

系統會在下列情況下，將訊息傳送到轉送寄不出的信件佇列：

- 訊息會通過 3 個以上[鏈結在一起](service-bus-auto-forwarding.md)的佇列或主題。
- 目的地佇列或主題已停用或刪除。
- 目的地佇列或主題超過最大實體大小。

若要擷取這些寄不出的信件訊息，您可以使用 [FormatTransferDeadletterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formattransferdeadletterpath) 公用程式方法來建立接收者。

## <a name="example"></a>範例

下列程式碼片段會建立訊息接收者。 在主要佇列的接收迴圈中，程式碼會利用 [Receive(TimeSpan.Zero)](/dotnet/api/microsoft.servicebus.messaging.messagereceiver#Microsoft_ServiceBus_Messaging_MessageReceiver_Receive_System_TimeSpan_) 擷取訊息，它會要求訊息代理程式立即傳回已可供使用的任何訊息，或者不傳回任何結果。 如果程式碼接收到訊息，它會立即放棄它，這樣會增加 `DeliveryCount`。 在系統將訊息移至 DLQ 之後，當 [ReceiveAsync](/dotnet/api/microsoft.servicebus.messaging.messagereceiver#Microsoft_ServiceBus_Messaging_MessageReceiver_ReceiveAsync_System_TimeSpan_) 傳回 **null**，主要佇列會是空的，而且迴圈會結束。

```csharp
var receiver = await receiverFactory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);
while(true)
{
    var msg = await receiver.ReceiveAsync(TimeSpan.Zero);
    if (msg != null)
    {
        Console.WriteLine("Picked up message; DeliveryCount {0}", msg.DeliveryCount);
        await msg.AbandonAsync();
    }
    else
    {
        break;
    }
}
```

## <a name="next-steps"></a>後續步驟

如需服務匯流排佇列的詳細資訊，請參閱下列文章。

* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [比較 Azure 佇列和服務匯流排佇列](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

