---
title: Azure 服務匯流排的交易處理概觀 | Microsoft Docs
description: Azure 服務匯流排不可部分完成交易和傳送方式概觀
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 64449247-1026-44ba-b15a-9610f9385ed8
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/22/2018
ms.author: spelluru
ms.openlocfilehash: bc0e0bfb8d4dc8637c191785a98f5d15e086cbf5
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696426"
---
# <a name="overview-of-service-bus-transaction-processing"></a>服務匯流排交易處理概觀

本文討論 Microsoft Azure 服務匯流排的交易功能。 [使用服務匯流排的不可部分完成交易範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AtomicTransactions)會說明大部分的討論。 本文只包含交易處理概觀以及服務匯流排中的「傳送方式」功能，而「不可部分完成交易」範例的範圍更廣且更複雜。

## <a name="transactions-in-service-bus"></a>服務匯流排中的交易

[*交易*](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AtomicTransactions#what-are-transactions)會將兩個以上的作業一起分組到「執行範圍」。 本質上，這類交易必須確定屬於指定作業群組的所有作業都一起成功或失敗。 在這部分，所有交易都會當成一個單位，通常稱為「不可部分完成性」。 

服務匯流排是交易訊息代理人，並確保其訊息存放區之所有內部作業的交易完整性。 服務匯流排內的所有訊息傳輸 (例如，將訊息移至[寄不出的信件佇列](service-bus-dead-letter-queues.md)，或[自動轉寄](service-bus-auto-forwarding.md)實體之間的訊息) 為交易式。 這麼一來，如果服務匯流排接受訊息，表示訊息已經儲存並標上序號。 從那時開始，服務匯流排內的任何訊息傳輸都是跨實體的協調作業，並且不會導致訊息遺失 (來源成功，但目標失敗) 或重複 (來源失敗，但目標成功)。

服務匯流排支援交易範圍內單一傳訊實體 (佇列、主題、訂用帳戶) 的分組作業。 例如，您可以將數個訊息傳送至交易範圍內的一個佇列；而且，在交易順利完成時，只會將訊息認可到佇列的記錄檔。

## <a name="operations-within-a-transaction-scope"></a>交易範圍內的作業

可在交易範圍內執行的作業如下︰

* **[QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient)、[MessageSender](/dotnet/api/microsoft.azure.servicebus.core.messagesender)、[TopicClient](/dotnet/api/microsoft.azure.servicebus.topicclient)**：Send、SendAsync、SendBatch、SendBatchAsync 
* **[BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)**：Complete、CompleteAsync、Abandon、AbandonAsync、Deadletter、DeadletterAsync、Defer、DeferAsync、RenewLock、RenewLockAsync 

不包括接收作業，因為假設應用程式使用 [ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode) 模式、在一些接收迴圈內或使用 [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage) 回呼來取得訊息，然後只會開啟交易範圍來處理訊息。

訊息的處置 (完成、放棄、寄不出的信件、延遲) 接著發生於整體交易結果的範圍內，並與整體交易結果相依。

## <a name="transfers-and-send-via"></a>傳輸和「傳送方式」

若要啟用從佇列到處理器再到另一個佇列的交易式資料送交，服務匯流排支援「傳輸」。 在傳輸作業中，寄件者會先將訊息傳送至「傳輸佇列」，而傳輸佇列會使用自動轉寄功能所依賴的相同穩固傳輸實作，立即將訊息移至想要的目的地佇列。 訊息絕不會認可到傳輸佇列的記錄檔，因此，傳輸佇列的取用者可以看到它。

傳輸佇列本身是寄件者輸入訊息的來源時，就能知道這個交易式功能的能力。 換句話說，服務匯流排可以「透過」傳輸佇列將訊息傳輸到目的地佇列，同時對輸入訊息執行完整 (或延遲，或寄不出信件) 作業 (全部都在一個不可部分完成作業中)。 

### <a name="see-it-in-code"></a>透過程式碼查看

若要設定這類傳輸，您可以建立將目標設為透過傳輸佇列之目的地佇列的訊息寄件者。 您也要有從相同佇列提取訊息的收件者。 例如︰

```csharp
var sender = this.messagingFactory.CreateMessageSender(destinationQueue, myQueueName);
var receiver = this.messagingFactory.CreateMessageReceiver(myQueueName);
```

簡單交易接著會使用這些元素，如下列範例所示︰

```csharp
var msg = receiver.Receive();

using (scope = new TransactionScope())
{
    // Do whatever work is required 

    var newmsg = ... // package the result 

    msg.Complete(); // mark the message as done
    sender.Send(newmsg); // forward the result

    scope.Complete(); // declare the transaction done
} 
```

## <a name="next-steps"></a>後續步驟

如需服務匯流排佇列的詳細資訊，請參閱下列文章。

* [如何使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [使用自動轉寄鏈結服務匯流排實體](service-bus-auto-forwarding.md)
* [自動轉寄範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AutoForward)
* [使用服務匯流排的不可部分完成交易範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AtomicTransactions)
* [比較 Azure 佇列和服務匯流排佇列](service-bus-azure-and-service-bus-queues-compared-contrasted.md)


