---
title: 使用 Azure 服務匯流排提升效能的最佳做法 |Microsoft Docs
description: 描述如何使用服務匯流排來在交換代理訊息時將效能最佳化。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
ms.service: service-bus-messaging
ms.topic: article
ms.date: 09/14/2018
ms.author: spelluru
ms.openlocfilehash: cfce11546249310ce00e5f19ba81520cc9dd78cf
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392630"
---
# <a name="best-practices-for-performance-improvements-using-service-bus-messaging"></a>使用服務匯流排傳訊的效能改進最佳作法

本文描述如何使用 Azure 服務匯流排來在交換代理的訊息時將效能最佳化。 本文的第一個部分說明提供協助提高效能的不同機制。 第二個部分針對如何在特定案例中利用可提供最佳效能的方式來使用服務匯流排提供指引。

在本文中，「用戶端」一詞是指任何存取服務匯流排的實體。 用戶端可以擔任傳送者或接收者的角色。 「傳送者」一詞用於將訊息傳送至服務匯流排佇列或主題訂用帳戶的服務匯流排佇列或主題用戶端。 「接收者」一詞指的是從服務匯流排佇列或訂用帳戶接收訊息的服務匯流排佇列或訂用帳戶用戶端。

這些章節介紹幾個服務匯流排用來協助提升效能的概念。

## <a name="protocols"></a>通訊協定

服務匯流排可讓用戶端透過三種通訊協定的其中一種傳送和接收訊息：

1. 進階訊息佇列通訊協定 (AMQP)
2. 服務匯流排傳訊通訊協定 (SBMP)
3. HTTP

AMQP 和 SBMP 會更有效率，因為只要傳訊處理站存在，它們就會維護服務匯流排連線。 它也會實作批次處理和預先擷取作業。 除非明確提到，否則本文中的所有內容都假設為使用 AMQP 和 SBMP。

## <a name="reusing-factories-and-clients"></a>重複使用處理站和用戶端

服務匯流排用戶端物件，例如 [QueueClient][QueueClient] 或 [MessageSender][MessageSender]，會透過也提供內部連接管理的 [MessagingFactory][MessagingFactory] 物件來建立。 當您傳送一個訊息，並在傳送下一個訊息時重新建立傳訊處理站或佇列、主題及訂用帳戶用戶端之後，建議您不要將其關閉。 關閉傳訊處理站會刪除服務匯流排服務的連接，並在重新建立處理站時建立新的連接。 建立連接是成本高昂的作業，您可以藉由重新使用多項作業的相同處理站和用戶端物件來避免此作業。 您可以安全地使用 [QueueClient][QueueClient] 物件，從並行非同步作業和多個執行緒傳送訊息。 

## <a name="concurrent-operations"></a>並行作業

執行作業 (傳送、接收、刪除等等) 需要一點時間。 這個時間包括服務匯流排服務處理作業的時間加上要求和回覆的延遲時間。 若要增加每次的作業數目，就必須並行執行作業。 

用戶端會透過執行非同步作業來排程並行作業。 下一個要求會在前一個要求完成之前啟動。 非同步傳送作業的程式碼片段範例如下：
  
 ```csharp
  Message m1 = new BrokeredMessage(body);
  Message m2 = new BrokeredMessage(body);
  
  Task send1 = queueClient.SendAsync(m1).ContinueWith((t) => 
    {
      Console.WriteLine("Sent message #1");
    });
  Task send2 = queueClient.SendAsync(m2).ContinueWith((t) => 
    {
      Console.WriteLine("Sent message #2");
    });
  Task.WaitAll(send1, send2);
  Console.WriteLine("All messages sent");
  ```
  
  非同步接收作業的程式碼範例如下。 請參閱[此處](https://github.com/Azure/azure-service-bus/blob/master/samples/DotNet/Microsoft.Azure.ServiceBus/SendersReceiversWithQueues)的完整程式：
  
  ```csharp
  var receiver = new MessageReceiver(connectionString, queueName, ReceiveMode.PeekLock);
  var doneReceiving = new TaskCompletionSource<bool>();

  receiver.RegisterMessageHandler(...);
  ```

## <a name="receive-mode"></a>接收模式

建立佇列或訂用帳戶用戶端時，您可以指定接收模式：「查看鎖定」或「接收與刪除」。 預設接收模式是 [PeekLock][PeekLock]。 以此模式操作時，用戶端會傳送要求，以接收來自服務匯流排的訊息。 用戶端收到訊息後，它會傳送要求以完成訊息。

將接收模式設為 [ReceiveAndDelete][ReceiveAndDelete] 時，兩個步驟會在單一要求中結合。 這些步驟可減少整體的作業數目並可改善整體訊息輸送量。 此效能改善的風險為遺失訊息。

服務匯流排不支援接收和刪除作業的交易。 此外，在用戶端想要延遲或讓訊息[寄不出去](service-bus-dead-letter-queues.md)的任何案例中都需要鎖定語意。

## <a name="client-side-batching"></a>用戶端批次處理

用戶端批次處理可讓佇列或主題用戶端將訊息的傳送延遲一段時間。 如果用戶端在此期間傳送其他訊息，它將以單一批次傳輸訊息。 用戶端批次處理也會導致佇列或訂用帳戶用戶端將多個**完成**要求整批放入單一要求中處理。 批次處理僅適用於非同步**傳送**及**完成**作業。 同步作業會立即傳送至服務匯流排服務。 查看或接收作業不會進行批次處理，也不會跨用戶端進行。

根據預設，用戶端使用的批次間隔為 20 毫秒。 您可以藉由在建立傳訊處理站之前設定 [BatchFlushInterval][BatchFlushInterval] 屬性來變更批次間隔。 這個設定會影響此處理站所建立的所有用戶端。

若要停用批次處理，請將 [BatchFlushInterval][BatchFlushInterval] 屬性設為 **TimeSpan.Zero**。 例如︰

```csharp
MessagingFactorySettings mfs = new MessagingFactorySettings();
mfs.TokenProvider = tokenProvider;
mfs.NetMessagingTransportSettings.BatchFlushInterval = TimeSpan.FromSeconds(0.05);
MessagingFactory messagingFactory = MessagingFactory.Create(namespaceUri, mfs);
```

批次處理並不會影響可計費的傳訊作業數目，而且僅適用於使用 [Microsoft.ServiceBus.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) 程式庫的服務匯流排用戶端通訊協定。 HTTP 通訊協定不支援批次處理。

## <a name="batching-store-access"></a>批次處理存放區存取

為了提高佇列、主題或訂用帳戶的輸送量，服務匯流排會在寫入至其內部存放區時批次處理多個訊息。 如果在佇列或主題上啟用，將會批次處理訊息寫入至存放區。 如果在佇列或訂用帳戶上啟用，將會批次處理從存放區刪除訊息。 如果某個實體啟用批次處理的存放區存取，服務匯流排會延遲與該實體有關的存放區寫入作業，最多 20 毫秒。 

> [!NOTE]
> 批次處理時沒有遺失訊息的風險，即使在 20ms 批次處理間隔結尾有服務匯流排失敗。 

在此間隔期間進行的其他存放區作業都會加入至批次。 批次處理的存放區存取只會影響**傳送**和**完成**作業；接收作業不會受到影響。 批次處理的存放區存取是實體上的屬性。 批次處理會在啟用批次處理存放區存取的所有實體進行。

建立新佇列、主題或訂用帳戶時，預設會啟用批次處理的存放區存取。 若要停用批次處理的存放區存取，請在建立實體之前將 [EnableBatchedOperations][EnableBatchedOperations] 屬性設為 **false**。 例如︰

```csharp
QueueDescription qd = new QueueDescription();
qd.EnableBatchedOperations = false;
Queue q = namespaceManager.CreateQueue(qd);
```

批次處理的存放區存取並不會影響可計費的傳訊作業數目，並且是佇列、主題或訂用帳戶的屬性。 它獨立於接收模式及用於用戶端和服務匯流排服務之間的通訊協定之外。

## <a name="prefetching"></a>預先擷取

[預先擷取](service-bus-prefetch.md)可讓佇列或訂用帳戶用戶端在執行接收作業時從服務載入額外的訊息。 用戶端會將這些訊息儲存在本機快取中。 快取的大小取決於 [QueueClient.PrefetchCount][QueueClient.PrefetchCount] 或 [SubscriptionClient.PrefetchCount][SubscriptionClient.PrefetchCount] 屬性。 啟用預先擷取的每個用戶端會維護自己的快取。 快取不會跨用戶端共用。 如果用戶端起始接收作業且其快取是空的，則服務會傳輸一批次的訊息。 批次的大小等於快取的大小或 256 KB，以較小者為準。 如果用戶端起始接收作業且快取包含一則訊息，訊息會從快取中擷取。

預先擷取訊息時，服務會鎖定預先擷取的訊息。 藉由鎖定，其他接收者就無法接收預先擷取的訊息。 如果接收者無法在鎖定到期之前完成訊息，訊息就會變成可供其他接收者接收。 訊息的預先擷取副本會保留在快取中。 當取用過其快取複本的接收者嘗試完成該訊息時，他會收到例外狀況。 根據預設，訊息鎖定會在 60 秒之後到期。 此值可延長為 5 分鐘。 若要避免過期訊息遭到取用，快取大小應該一律小於用戶端在鎖定逾時間隔內可取用的訊息數目。

使用 60 秒的預設鎖定到期時間時，[PrefetchCount][SubscriptionClient.PrefetchCount] 的良好值為處理站中所有接收者最高處理速率的 20 倍。 例如，處理站建立三個接收者，而每個接收者每秒可以處理最多 10 則訊息。 預先擷取計數不應該超過 20 X 3 X 10 = 600。 根據預設，[PrefetchCount][QueueClient.PrefetchCount] 設為 0，表示沒有其他訊息從服務擷取。

預先擷取訊息會增加佇列或訂用帳戶的整體輸送量，因為此舉可減少訊息作業的整體數目或來回次數。 然而，擷取第一個訊息需要較長的時間 (因為訊息大小增加)。 接收預先擷取的訊息比較快速，因為用戶端已經下載這些訊息。

伺服器會在它將訊息傳送至用戶端時，檢查訊息的存留時間 (TTL) 屬性。 用戶端不會在收到訊息時檢查訊息的 TTL 屬性。 相反地，即使訊息由用戶端快取時已超過其 TTL，仍然可以收到訊息。

預先擷取並不會影響可計費的傳訊作業數目，而且僅適用於服務匯流排用戶端通訊協定。 HTTP 通訊協定不支援預先擷取。 同步和非同步接收作業皆可使用預先擷取。

## <a name="express-queues-and-topics"></a>快速佇列和主題

快速實體可提高輸送量並縮短延遲，而只有在「標準」傳訊層才支援這些實體。 在[進階命名空間](service-bus-premium-messaging.md)中建立的實體不支援快速選項。 使用快速實體時，如果訊息傳送至佇列或主題，訊息不會立即儲存在訊息存放區。 相反地，它會快取在記憶體中。 如果訊息保留在佇列中超過幾秒鐘，它會自動寫入至穩定儲存體，藉此保護它免於因中斷而遺失。 將訊息寫入記憶體快取會增加輸送量並減少延遲，因為訊息傳送時沒有存取穩定儲存體。 在幾秒鐘內取用的訊息不會寫入至訊息存放區。 下列範例會建立快速主題。

```csharp
TopicDescription td = new TopicDescription(TopicName);
td.EnableExpress = true;
namespaceManager.CreateTopic(td);
```

如果將包含不能遺失之重要資訊的訊息傳送至快速實體，傳送者可以強制服務匯流排立即藉由將 [ForcePersistence][ForcePersistence] 屬性設為 **true** 來將訊息儲存到穩定儲存體。

> [!NOTE]
> 快速實體並不支援交易。

## <a name="partitioned-queues-or-topics"></a>分割的佇列或主題

服務匯流排會在內部使用相同的節點和訊息存放區來處理和儲存傳訊實體 (佇列或主題) 的所有訊息。 另一方面，[分割的佇列或主題](service-bus-partitioning.md)會在多個節點和訊息存放區中散佈。 分割的佇列和主題不僅會產生比一般佇列和主題更高的輸送量，也會展現較優異的可用性。 若要建立分割的實體，請將 [EnablePartitioning][EnablePartitioning] 屬性設為 **true**，如下列範例所示。 如需分割實體的詳細資訊，請參閱[分割的傳訊實體][Partitioned messaging entities]。

> [!NOTE]
> [Premium SKU](service-bus-premium-messaging.md) 不支援分割的實體。 

```csharp
// Create partitioned queue.
QueueDescription qd = new QueueDescription(QueueName);
qd.EnablePartitioning = true;
namespaceManager.CreateQueue(qd);
```

## <a name="multiple-queues"></a>多個佇列

如果您不能使用分割的佇列或主題，或無法由單一分割佇列或主題處理預期的負載，您必須使用多個傳訊實體。 使用多個實體時，請針對每個實體建立專屬用戶端，而不是讓所有實體使用相同的用戶端。

## <a name="development-and-testing-features"></a>開發與測試功能

服務匯流排有一項專門用於開發的功能，此功能**永遠不應該用在生產組態**：[TopicDescription.EnableFilteringMessagesBeforePublishing][]。

當新的規則或篩選器新增至主題時，您可以使用 [TopicDescription.EnableFilteringMessagesBeforePublishing][] 來確認新的篩選運算式如預期般運作。

## <a name="scenarios"></a>案例

下列各節描述典型傳訊的案例，並簡述慣用的服務匯流排設定。 輸送量速率會分類為小型 (小於 1 則訊息/秒)、中型 (1 則訊息/秒或更多但小於 100 則訊息/秒) 和高型 (100 則訊息/秒或更多)。 用戶端數目可分類為小型 (5 個或更少)、中型 (5 個以上但小於或等於 20 個) 和大型 (超過 20 個)。

### <a name="high-throughput-queue"></a>高輸送量佇列

目標：最大化單一佇列的輸送量。 傳送者和接收者的數目很少。

* 若要增加傳送到佇列的整體速率，請使用多個訊息處理站來建立傳送者。 對每個傳送者使用非同步作業或多個執行緒。
* 若要增加從佇列接收的整體速率，請使用多個訊息處理站來建立接收者。
* 使用非同步作業來利用用戶端批次處理。
* 將批次處理間隔設為 50 毫秒，以減少服務匯流排用戶端通訊協定傳輸數目。 如果使用多個傳送者，批次處理間隔會增加到 100 毫秒。
* 讓批次處理的存放區存取保持啟用。 此存取會增加訊息寫入至佇列的整體速率。
* 將預先擷取計數設為 20 乘以處理站所有接收者的最高處理速率。 此計數會減少服務匯流排用戶端通訊協定傳輸數目。
* 使用分割的佇列以改善效能和可用性。

### <a name="multiple-high-throughput-queues"></a>多個高輸送量佇列

目標：最大化多個佇列的整體輸送量。 個別佇列的輸送量為中或高。

若要取得跨多個佇列的最大輸送量，請使用簡述的設定以最大化單一佇列的輸送量。 此外，使用不同的處理站來建立傳送至或接收自不同佇列的用戶端。

### <a name="low-latency-queue"></a>低延遲性佇列

目標：最小化佇列或主題的端對端延遲。 傳送者和接收者的數目很少。 佇列的輸送量為小或中。

* 停用用戶端批次處理。 用戶端會立即傳送訊息。
* 停用批次處理的存放區存取。 服務會立即將訊息寫入至存放區中。
* 如果使用單一用戶端，請將預先擷取計數設為 20 乘以接收者的處理速率。 如果多個訊息同時抵達佇列，服務匯流排用戶端通訊協定會同時將它們全部傳輸。 當用戶端收到下一個訊息時，該訊息已經在本機快取中。 快取應該很小。
* 如果使用多個用戶端，請將預先擷取計數設為 0。 藉由設定該計數，第二個用戶端可以在第一個用戶端仍在處理第一個訊息時接收第二個訊息。
* 使用分割的佇列以改善效能和可用性。

### <a name="queue-with-a-large-number-of-senders"></a>具有大量傳送者的佇列

目標：最大化具有大量傳送者之佇列或主題的輸送量。 每個傳送者都會以中等速率傳送訊息。 接收者的數目很少。

服務匯流排可啟用多達 1000 個並行連線至訊息實體 (或使用 AMQP 可達 5000 個)。 在命名空間層級強制執行這項限制，且佇列/主題/訂用帳戶的上限為每個命名空間的並行連線限制。 對佇列而言，這個數目是在傳送者和接收者之間共用的。 如果 1000 個連線全都為傳送者所需，請以主題和單一訂用帳戶取代此佇列。 主題會接受多達 1000 個來自傳送者的並行連線，而訂用帳戶可接受來自接收者的額外 1000 個並行連線。 如果需要超過 1000 個並行傳送者，傳送者必須透過 HTTP 將訊息傳送至服務匯流排通訊協定。

若要最大化輸送量，請執行下列步驟：

* 如果每個傳送者都位於不同的處理序中，每個處理序僅使用單一處理站。
* 使用非同步作業來利用用戶端批次處理。
* 使用預設的批次處理間隔 20 毫秒來減少服務匯流排用戶端通訊協定傳輸數目。
* 讓批次處理的存放區存取保持啟用。 此存取會增加訊息寫入至佇列或主題的整體速率。
* 將預先擷取計數設為 20 乘以處理站所有接收者的最高處理速率。 此計數會減少服務匯流排用戶端通訊協定傳輸數目。
* 使用分割的佇列以改善效能和可用性。

### <a name="queue-with-a-large-number-of-receivers"></a>具有大量接收者的佇列

目標：最大化具有大量接收者之佇列或訂用帳戶的接收速率。 每個接收者皆以中等速率接收訊息。 傳送者的數目很少。

服務匯流排可啟用多達 1000 個並行連線至實體。 如果佇列需要超過 1000 個接收者，請以主題和多個訂用帳戶取代此佇列。 每個訂用帳戶可支援最多 1000 個並行連線。 此外，接收者可以透過 HTTP 通訊協定存取佇列。

若要最大化輸送量，請執行下列作業：

* 如果每個接收者都位於不同的處理序中，每個處理序僅使用單一處理站。
* 接收者可以使用同步或非同步作業。 若指定個別接收者的中等接收速率，完全要求的用戶端批次處理不會影響接收者輸送量。
* 讓批次處理的存放區存取保持啟用。 此存取會減少實體的整體負載。 它也會減少訊息寫入至佇列或主題的整體速率。
* 將預先擷取計數設為較小的值 (例如，PrefetchCount = 10)。 此計數可以避免接收者在其他接收者具有大量快取的訊息時閒置。
* 使用分割的佇列以改善效能和可用性。

### <a name="topic-with-a-small-number-of-subscriptions"></a>具有少量訂用帳戶的主題

目標：最大化具有少量訂用帳戶之主題的輸送量。 許多訂用帳戶收到一則訊息，表示所有訂用帳戶的合併接收速率高於傳送速率。 傳送者的數目很少。 每個訂用帳戶的接收者數目很少。

若要最大化輸送量，請執行下列作業：

* 若要增加傳送到主題的整體速率，請使用多個訊息處理站來建立傳送者。 對每個傳送者使用非同步作業或多個執行緒。
* 若要增加從訂用帳戶接收的整體速率，請使用多個訊息處理站來建立接收者。 對每個接收者使用非同步作業或多個執行緒。
* 使用非同步作業來利用用戶端批次處理。
* 使用預設的批次處理間隔 20 毫秒來減少服務匯流排用戶端通訊協定傳輸數目。
* 讓批次處理的存放區存取保持啟用。 此存取會增加訊息寫入至主題的整體速率。
* 將預先擷取計數設為 20 乘以處理站所有接收者的最高處理速率。 此計數會減少服務匯流排用戶端通訊協定傳輸數目。
* 使用分割的主題以改善效能和可用性。

### <a name="topic-with-a-large-number-of-subscriptions"></a>具有大量訂用帳戶的主題

目標：最大化具有大量訂用帳戶之主題的輸送量。 許多訂用帳戶收到一則訊息，表示所有訂用帳戶的合併接收速率遠高於傳送速率。 傳送者的數目很少。 每個訂用帳戶的接收者數目很少。

如果所有訊息都路由傳送至所有訂用帳戶，具有大量訂用帳戶的主題通常會顯露較低的整體輸送量。 這樣較低的輸送量是因為每個訊息都被接收了許多次，而且包含在主題與其所有訂用帳戶中的所有訊息都儲存在相同的存放區中。 它會假設每個訂用帳戶的傳送者數目和接收者數目都很少。 服務匯流排支援每個主題最多 2,000 個訂用帳戶。

若要最大化輸送量，請嘗試下列步驟：

* 使用非同步作業來利用用戶端批次處理。
* 使用預設的批次處理間隔 20 毫秒來減少服務匯流排用戶端通訊協定傳輸數目。
* 讓批次處理的存放區存取保持啟用。 此存取會增加訊息寫入至主題的整體速率。
* 將預先擷取設為 20 乘以預期的接收速率 (以秒為單位)。 此計數會減少服務匯流排用戶端通訊協定傳輸數目。
* 使用分割的主題以改善效能和可用性。

## <a name="next-steps"></a>後續步驟

若要深入了解如何最佳化服務匯流排效能，請參閱[分割的傳訊實體][Partitioned messaging entities]。

[QueueClient]: /dotnet/api/microsoft.azure.servicebus.queueclient
[MessageSender]: /dotnet/api/microsoft.azure.servicebus.core.messagesender
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[PeekLock]: /dotnet/api/microsoft.azure.servicebus.receivemode
[ReceiveAndDelete]: /dotnet/api/microsoft.azure.servicebus.receivemode
[BatchFlushInterval]: /dotnet/api/microsoft.servicebus.messaging.messagesender.batchflushinterval
[EnableBatchedOperations]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations
[QueueClient.PrefetchCount]: /dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount
[SubscriptionClient.PrefetchCount]: /dotnet/api/microsoft.azure.servicebus.subscriptionclient.prefetchcount
[ForcePersistence]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage.forcepersistence
[EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[Partitioned messaging entities]: service-bus-partitioning.md
[TopicDescription.EnableFilteringMessagesBeforePublishing]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablefilteringmessagesbeforepublishing
