---
title: Azure 事件中樞的程式設計指南 | Microsoft Docs
description: 使用 Azure .NET SDK 撰寫 Azure 事件中樞的程式碼。
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
ms.service: event-hubs
ms.topic: article
ms.date: 08/12/2018
ms.author: shvija
ms.openlocfilehash: bfb2db8a4a0091e26cc2b893e615ba831da30ac7
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746319"
---
# <a name="event-hubs-programming-guide"></a>事件中樞程式設計指南

本文會討論一些使用 Azure 事件中樞來撰寫程式碼的常見案例。 它假設使用者對事件中樞已有初步了解。 如需事件中樞的概念概觀，請參閱 [事件中樞概觀](event-hubs-what-is-event-hubs.md)。

## <a name="event-publishers"></a>事件發佈者

您可以使用 HTTP POST 或透過 AMQP 1.0 連線，將事件傳送到事件中樞。 使用選擇取決於應用的特定案例。 AMQP 1.0 連線是以服務匯流排中的代理連線形式計量，其較適合經常出現大量訊息且需要低延遲的案例，因為它們可提供持續的傳訊通道。

在使用 .NET 受控 API 時，用於將資料發佈到事件中樞的主要建構是 [EventHubClient][] 和 [EventData][] 類別。 [EventHubClient][] 提供將事件傳送到事件中樞時所透過的 AMQP 通訊通道。 [EventData][] 類別代表事件，可用來將訊息發佈到事件中樞。 這個類別包含主體、一些中繼資料，以及有關事件的標頭資訊。 當 [EventData][] 物件通過事件中樞時，系統會為它新增其他屬性。

## <a name="get-started"></a>開始使用

[Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) NuGet 套件中會提供支援事件中樞的 .NET 類別。 您可以使用 Visual Studio 方案總管，或 Visual Studio 中的[套件管理員主控台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)進行安裝。 若要這樣做，請在 [Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console) 視窗中發出下列命令：

```shell
Install-Package Microsoft.Azure.EventHubs
```

## <a name="create-an-event-hub"></a>建立事件中樞

您可以使用 Azure 入口網站、Azure PowerShell 或 Azure CLI 來建立事件中樞。 如需詳細資訊，請參閱[使用 Azure 入口網站來建立事件中樞命名空間和事件中樞](event-hubs-create.md)。

## <a name="create-an-event-hubs-client"></a>建立事件中樞用戶端

與事件中樞互動的主要類別是 [Microsoft.Azure.EventHubs.EventHubClient][EventHubClient]。 您可以使用 [CreateFromConnectionString](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createfromconnectionstring) 方法來具現化這個類別，如以下範例所示：

```csharp
private const string EventHubConnectionString = "Event Hubs namespace connection string";
private const string EventHubName = "event hub name";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
{
    EntityPath = EventHubName

};
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

## <a name="send-events-to-an-event-hub"></a>將事件傳送到事件中樞

您可藉由建立 [EventHubClient][] 執行個體並透過 [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) 方法以方同步方式傳送它，將事件傳送到事件中樞。 這個方法會採用單一 [EventData][] 執行個體參數，並以同步方式將它傳送到事件中樞。

## <a name="event-serialization"></a>事件序列化

[EventData][] 類別具有[兩個多載建構函式](/dotnet/api/microsoft.azure.eventhubs.eventdata.-ctor)，它們會採用代表事件資料承載的各種參數 (位元組或位元組陣列)。 在搭配使用 JSON 和 [EventData][]時，您可以使用 **Encoding.UTF8.GetBytes()** 來擷取 JSON 編碼字串的位元組陣列。 例如︰

```csharp
for (var i = 0; i < numMessagesToSend; i++)
{
    var message = $"Message {i}";
    Console.WriteLine($"Sending message: {message}");
    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
}
```

## <a name="partition-key"></a>資料分割索引鍵

傳送事件資料時，您可以指定雜湊值，以產生分割區指派。 您可使用 [PartitionSender.PartitionID](/dotnet/api/microsoft.azure.eventhubs.partitionsender.partitionid) 屬性來指定分割區。 不過，使用分割區的決策暗示可用性與一致性之間的選擇。 

### <a name="availability-considerations"></a>可用性考量

您可以選擇使用資料分割索引鍵，至於是否要使用，請您慎重考慮。 在許多情況下，如果事件的順序很重要，您最好選擇使用資料分割索引鍵。 當您使用資料分割索引鍵時，這些資料分割必須在單一節點上可供使用，但經過一段時間後，節點有可能會發生中斷，例如在計算節點重新開機及修補時。 因此，如果您設定了資料分割識別碼，但該資料分割因為某些緣故變得無法使用，您就無法嘗試存取該資料分割中的資料。 如果您最重視的是高可用性，請勿指定資料分割索引鍵，因為在該情況下，事件會使用先前所述的循環配置資源模型傳送到資料分割。 在此案例中，您必須在可用性 (沒有資料分割識別碼) 和一致性 (將事件繫結至資料分割識別碼) 之間做出明確抉擇。

另一項考量是對處理事件的延遲進行處理。 有時候，捨棄資料並重試會比試著跟上處理腳步來得好，因為後者可能會導致下游處理延遲得更久。 例如，拿股票行情即時看板來說，等待完整的最新資料會比較好，但在即時聊天或 VOIP 的案例中，您會寧願先收到資料，即便資料並不完整也沒關係。

有鑑於上述可用性考量，您可能會在這些案例中選擇下列其中一個錯誤處理策略︰

- 停止 (停止讀取事件中樞，直到情況獲得修正)
- 捨棄 (訊息並不重要，將其卸除)
- 重試 (視情況重試訊息)

如需可用性和一致性之間利弊得失的詳細資訊與討論，請參閱[事件中樞的可用性和一致性](event-hubs-availability-and-consistency.md)。 

## <a name="batch-event-send-operations"></a>批次事件傳送作業

分批傳送事件可以協助增加輸送量。 您可使用 [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) API 來建立批次，資料物件稍後可針對 [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) 呼叫新增至該批次。

單一批次不能超過每個事件 256 KB 的限制。 此外，批次中的每個訊息都會使用相同的身分識別。 確保批次未超過最大事件大小是傳送者的責任。 如果超過的話，系統會產生用戶端 **Send** 錯誤。 您可以使用協助程式方法 [EventHubClient.CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) 以確保批次不超過 256 KB。 您會從 [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) API 取得空 [EventDataBatch](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch)，然後使用 [TryAdd](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch.tryadd) 新增事件來建立批次。 

## <a name="send-asynchronously-and-send-at-scale"></a>以非同步方式傳送和大規模傳送

您可以用非同步方式將事件傳送到事件中樞。 以非同步方式傳送會增加用戶端傳送事件的速率。 [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) 會傳回 [Task](https://msdn.microsoft.com/library/system.threading.tasks.task.aspx) 物件。 您可以在用戶端上使用 [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) 類別來控制用戶端重試選項。

## <a name="event-consumers"></a>事件取用者

[EventProcessorHost][] 類別能處理來自事件中樞的資料。 在 .NET 平台上建置事件讀取器時，您應該使用這項實作。 [EventProcessorHost][] 能為事件處理器實作提供安全執行緒、多處理序、安全的執行階段環境，進而提供檢查點和資料分割租用管理。

若要使用 [EventProcessorHost][] 類別，您可以實作 [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor)。 這個介面包含四個方法：

* [OpenAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.openasync)
* [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.closeasync)
* [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync)
* [ProcessErrorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processerrorasync)

若要啟動事件處理，請將 [EventProcessorHost][] 具現化，其中需為事件中樞提供適當的參數。 例如︰

```csharp
var eventProcessorHost = new EventProcessorHost(
        EventHubName,
        PartitionReceiver.DefaultConsumerGroupName,
        EventHubConnectionString,
        StorageConnectionString,
        StorageContainerName);
```

接著，呼叫 [RegisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync) 以向執行階段註冊 [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) 實作：

```csharp
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
```

此時，主機會嘗試使用「窮盡」演算法來取得事件中樞內每個資料分割的租用。 這些租用會延續一段指定時間，然後必須更新。 本案例中，當新節點上線時，背景工作執行個體會保留租用，而每當取得更多租用的嘗試發生時，負載會隨著在節點之間轉移。

![事件處理器主機](./media/event-hubs-programming-guide/IC759863.png)

經過一段時間後，均衡的局面將會出現。 此動態功能可讓您將 CPU 架構自動調整套用至消費者，以便進行向上和向下調整。 由於事件中樞沒有直接的訊息計數概念，因此平均 CPU 使用率通常是測量後端或消費者規模最合適的機制。 如果發佈者發佈的事件數量開始超出消費者的處理能力，消費者上增加的 CPU 可用來引發背景工作執行個體計數自動調整。

[EventProcessorHost][] 類別還能實作以 Azure 儲存體為基礎的檢查點機制。 這項機制能儲存每個磁碟分割的位移，方便各個消費者判斷前一個消費者的最後一個檢查點。 由於資料分割會透過租用在節點之間轉換，因此這是能促進負載移位的同步處理機制。

## <a name="publisher-revocation"></a>發佈者撤銷

除了 [EventProcessorHost][]的進階執行階段功能之外，「事件中樞」還能讓您撤銷發佈者，以防止特定發佈者將事件傳送到到事件中樞。 當發行者權杖遭到洩露，或軟體更新造成發佈者出現不當行為時，這些功能很有用。 在這些情況下，您可以封鎖發佈者 SAS 權杖中的發佈者身分識別，避免它們發佈事件。

如需有關發佈者撤銷，以及如何以發佈者身分傳送到事件中樞的詳細資訊，請參閱[事件中樞大規模安全發佈](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab)範例。

## <a name="next-steps"></a>後續步驟

若要深入了解事件中樞案例，請造訪下列連結：

* [事件中樞 API 概觀](event-hubs-api-overview.md)
* [何謂事件中樞](event-hubs-what-is-event-hubs.md)
* [事件中樞的可用性和一致性](event-hubs-availability-and-consistency.md)
* [事件處理器主機 API 參考](/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost)

[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[EventHubClient]: /dotnet/api/microsoft.azure.eventhubs.eventhubclient
[EventData]: /dotnet/api/microsoft.azure.eventhubs.eventdata
[CreateEventHubIfNotExists]: /dotnet/api/microsoft.servicebus.namespacemanager.createeventhubifnotexists
[PartitionKey]: /dotnet/api/microsoft.servicebus.messaging.eventdata#Microsoft_ServiceBus_Messaging_EventData_PartitionKey
[EventProcessorHost]: /dotnet/api/microsoft.azure.eventhubs.processor
