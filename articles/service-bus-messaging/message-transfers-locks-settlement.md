---
title: Azure 服務匯流排訊息傳輸、鎖定和安置 | Microsoft Docs
description: 服務匯流排訊息傳輸和安置作業概觀
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
ms.date: 09/25/2018
ms.author: spelluru
ms.openlocfilehash: a1835e26f67427f84abd8a3cf24ad196fec8e99d
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854417"
---
# <a name="message-transfers-locks-and-settlement"></a>訊息傳輸、鎖定和安置

訊息代理程式 (例如服務匯流排) 的主要功能是將訊息接受至佇列或主題，並加以保留以供日後擷取。 「傳送」一詞通常會用來將訊息傳輸至訊息代理程式。 「接收」一詞通常會用來將訊息傳輸至擷取用戶端。

當用戶端傳送訊息時，它通常想要知道訊息是否已正確傳輸到並由訊息代理程式所接受，或者是否發生了某種錯誤。 這個肯定或否定的認可讓使用戶端和訊息代理程式能夠了解訊息傳輸狀態的相關資訊，因此稱為「安置」。

同樣地，當訊息代理程式將訊息傳輸到用戶端時，訊息代理程式和用戶端會想要了解訊息是否已成功處理，因而可加以移除，或者訊息傳遞或處理是否失敗，因此可能必須再次傳遞訊息。

## <a name="settling-send-operations"></a>安置傳送作業

使用任何支援的服務匯流排 API 用戶端，一律可明確地安置傳送至服務匯流排的作業，這表示 API 作業會等候服務匯流排的接受結果送達，然後才會完成傳送作業。

如果服務匯流排拒絕了訊息，拒絕就會包含錯誤指標及文字，並於內部包含「追蹤識別碼」。 拒絕也會包含是否可在任何預期成功的情況下重試此作業的相關資訊。 在用戶端，會將此資訊轉換成例外狀況，並引發傳送作業的呼叫端。 如果已接受訊息，作業就會以無訊息方式完成。

使用 AMQP 通訊協定 (此為標準 .NET 用戶端和 Java 用戶端的獨佔通訊協定且[這是 .NET Framework 用戶端的選項](service-bus-amqp-dotnet.md)) 時，會以完全非同步方式將訊息傳輸與安置進行管線傳送，並建議您使用非同步程式設計模型 API 變化。

傳送者可以在線路上快速且連續地放上數個訊息，而不需等候每個訊息的確認，不然就是使用 SBMP 通訊協定或使用 HTTP 1.1 的情況。 那些非同步傳送作業會在接受並儲存個別訊息時完成 (在分割的實體上，或是在傳送到不同實體的作業重疊時)。 此外，可能也不會依原始傳送順序完成。

用於處理傳送作業結果的策略可能會對您的應用程式產生立即且顯著的效能影響。 本節中的範例是以 C# 撰寫的，同樣適用於 Java Futures。

如果應用程式會產生暴增的訊息 (這裡以單純迴圈來說明)，以及在傳送下一個訊息之前要在何處等候每個傳送作業完成 (相似的同步或非同步 API 圖形)，則傳送 10 個訊息只會在 10 個要安置的循序完整來回行程之後完成。

假設從內部部署網站到服務匯流排之間有 70 毫秒的 TCP 來回行程延遲距離，並假設服務匯流排只有 10 毫秒能夠接受並儲存每個訊息，則下列迴圈至少需要花費 8 秒，而且不會計算承載傳輸時間或潛在的路由壅塞效果：

```csharp
for (int i = 0; i < 100; i++)
{
  // creating the message omitted for brevity
  await client.SendAsync(…);
}
```

如果應用程式立即連續啟動 10 個非同步傳送作業，並個別等候其各自完成，則那 10 個傳送作業的往返行程時間會重疊。 10 個訊息都會立即連續傳輸，甚至可能共用 TCP 框架，而整體傳輸持續期間主要取決於它取得已傳輸至訊息代理程式之訊息所需的網路相關時間。

對先前迴圈做出相同的假設，下列迴圈的總重疊執行時間可能會保持在一秒內：

```csharp
var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  tasks.Add(client.SendAsync(…));
}
await Task.WhenAll(tasks);
```

請務必注意，所有的非同步程式設計模型都會使用某種形式之以記憶體為基礎的隱藏工作佇列來保留暫止作業。 當 [SendAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.sendasync#Microsoft_Azure_ServiceBus_QueueClient_SendAsync_Microsoft_Azure_ServiceBus_Message_) (C#) 或 **Send** (Java) 傳回時，會將傳送工作排入該工作佇列，但通訊協定軌跡只會在工作開始執行之後開始。 對於傾向推送暴增訊息且必須考量可靠性的程式碼而言，應該謹慎地不要同時「在途中」放置太多訊息，因為所有傳送訊息都會佔據記憶體，直到確實地將它們放到線路上為止。

旗號 (如下列以 C# 撰寫的程式碼片段所示) 是同步物件，會在需要時啟用這類應用程式層級的節流。 這種旗號用法允許同時最多傳送 10 個訊息。 這 10 個可用的旗號鎖定之一會在傳送之前取得，並在傳送完成之後釋放。 第 11 個則會經歷迴圈等待，直到先前的傳送中至少有一個完成為止，然後使其鎖定可供使用：

```csharp
var semaphore = new SemaphoreSlim(10);

var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  await semaphore.WaitAsync();

  tasks.Add(client.SendAsync(…).ContinueWith((t)=>semaphore.Release()));
}
await Task.WhenAll(tasks);
```

應用程式**絕對不應**以「射後不理」的方式來起始非同步傳送作業，而不擷取作業的結果。 這樣做可以載入內部和不可見的工作佇列，直到記憶體耗盡，並防止應用程式偵測到傳送錯誤：

```csharp
for (int i = 0; i < 100; i++)
{

  client.SendAsync(message); // DON’T DO THIS
}
```

透過低層級的 AMQP 用戶端，服務匯流排也可接受「預先安置」的傳輸。 預先安置的傳輸是一個射後不理作業，其結果 (不論哪種方式) 不會回報給用戶端，而且會在傳送時將訊息視為已安置。 缺少對用戶端的意見反應，也意味著沒有可採用來用於診斷的資料，這表示這個模式無法透過 Azure 支援服務來取得協助。

## <a name="settling-receive-operations"></a>安置接收作業

對於接收作業，服務匯流排 API 用戶端會啟用兩個不同的 Explicit 模式：「接收並刪除」和「查看鎖定」。

[接收並刪除](/dotnet/api/microsoft.servicebus.messaging.receivemode)模式會告知訊息代理程式，考慮它傳送給接收用戶端且在傳送時已安置的所有訊息。 那就表示訊息會被視為在訊息代理程式將其放在線路上之後立即取用。 如果訊息傳輸失敗，訊息就會遺失。

此模式的優點是接收者不需對訊息採取進一步動作，同時也不會因為等候安置結果而變慢。 如果個別訊息中包含的資料值很低和 (或) 資料在極短時間內才有意義，則此模式是合理的選擇。

[查看鎖定](/dotnet/api/microsoft.servicebus.messaging.receivemode)模式會告知訊息代理程式，接收用戶端想要明確地安置已接收的訊息。 訊息可讓接收者用來加以處理，同時保留於服務的獨佔鎖定下，如此，其他競爭的接收者就無法看見該訊息。 鎖定的持續期間一開始會定義於佇列或訂用帳戶層級，可由擁有鎖定的用戶端透過 [RenewLock](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_) 作業來延長。

當訊息鎖定時，從相同佇列或訂用帳戶接收的其他用戶端均可取得鎖定，並擷取下一個不是處於作用中鎖定的可用訊息。 明確釋放訊息上的鎖定或當鎖定到期時，訊息會在擷取順序的前端或附近重新出現以便重新傳遞。

當接收者重複釋放訊息，或者它們讓鎖定經歷定義的次數 ([maxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount#Microsoft_ServiceBus_Messaging_QueueDescription_MaxDeliveryCount)) 時，就會從佇列或訂用帳戶中自動移除訊息，並將它放入相關聯的無效信件佇列。

當它在 API 層級呼叫 [Complete](/dotnet/api/microsoft.servicebus.messaging.queueclient.complete#Microsoft_ServiceBus_Messaging_QueueClient_Complete_System_Guid_) 時，接收用戶端會利用肯定的認可來起始已接收訊息的安置。 這會向訊息代理程式表示，訊息已成功處理，並已從佇列或訂用帳戶中移除訊息。 訊息代理程式會利用一則表示是否可執行安置的回覆來回覆接收者的安置意圖。

當接收用戶端無法處理訊息，但想要重新傳遞訊息時，它可以呼叫 [Abandon](/dotnet/api/microsoft.servicebus.messaging.queueclient.abandon)，明確地要求釋放並立即解除鎖定訊息，或者它不需執行任何動作，並等待鎖定到期。

如果接收用戶端無法處理訊息，而且知道重新傳遞訊息和重試作業將不會有任何助益，它可以拒絕訊息，藉由呼叫 [DeadLetter](/dotnet/api/microsoft.servicebus.messaging.queueclient.deadletter) 來將它移至無效信件佇列，它也允許您設定一個自訂屬性，其中包括可從無效信件佇列中使用訊息擷取的原因代碼。

有一個特殊的安置案例是延遲，其他文章中另有討論。

**Complete** 或 **Deadletter** 作業以及 **RenewLock** 作業可能會因為網路問題、如果保留的鎖定過期，或者其他服務端發生防止安置的情況而失敗。 在後兩者的任一種情況下，服務會傳送否定的認可，以 API 用戶端中的例外狀況來呈現。 如果原因是網路連線中斷，則會卸除鎖定，因為服務匯流排不支援在不同連線上復原現有的 AMQP 連結。

如果 **Complete** 失敗 (通常發生在訊息處理的最尾端，以及在處理工作進行數分鐘之後的某些情況下)，接收應用程式可以決定是否要保存工作的狀態，並且在訊息第二次傳遞時忽略相同的訊息，或者是否要丟棄工作結果並在重新傳遞訊息時進行重試。

用來識別重複訊息傳遞的一般機制是檢查訊息識別碼，這可以且應該由傳送者設定為唯一值，可能會以來自原始處理序的識別碼為基準。 作業排程器可能會將訊息識別碼設為作業的識別碼，該作業是其嘗試使用指定的背景工作指派給某個背景工作的作業，而且，如果該作業已經完成，則該背景工作會忽略第二次的作業指派。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)
