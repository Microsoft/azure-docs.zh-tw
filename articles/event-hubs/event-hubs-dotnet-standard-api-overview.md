---
title: Azure 事件中樞 .NET Standard API 概觀 | Microsoft Docs
description: .NET Standard API 概觀
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.topic: article
ms.date: 08/13/2018
ms.author: shvija
ms.openlocfilehash: 9b952bd96828c4f2c140cb2d75cecb9379895a63
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746638"
---
# <a name="event-hubs-net-standard-api-overview"></a>事件中樞 .NET Standard API 概觀

本文將摘要列出一些主要 Azure 事件中樞 [.NET Standard 用戶端 API](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/)。 事件中樞目前有兩個 .NET Standard 用戶端程式庫︰

* [Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs)：提供所有基本執行階段作業。
* [Microsoft.Azure.EventHubs.Processor](/dotnet/api/microsoft.azure.eventhubs.processor)：新增可讓您記錄處理過之事件的額外功能，並且是從事件中樞讀取的最簡單方式。

## <a name="event-hubs-client"></a>事件中樞用戶端

[EventHubClient](/dotnet/api/microsoft.azure.eventhubs.eventhubclient) 是您用來傳送事件、建立接收者，以及取得執行階段資訊的主要物件。 此用戶端會連結到特定的事件中樞，並建立新的「事件中樞」端點連線。

### <a name="create-an-event-hubs-client"></a>建立事件中樞用戶端

[EventHubClient](/dotnet/api/microsoft.azure.eventhubs.eventhubclient) 物件是從連接字串建立。 具現化新用戶端的最簡單方式，如下列範例所示︰

```csharp
var eventHubClient = EventHubClient.CreateFromConnectionString("Event Hubs connection string");
```

若要以程式設計方式編輯連接字串，您可以使用 [EventHubsConnectionStringBuilder](/dotnet/api/microsoft.azure.eventhubs.eventhubsconnectionstringbuilder) 類別，並將連接字串做為參數傳遞到 [EventHubClient.CreateFromConnectionString](/dotnet/api/microsoft.azure.eventhubs.eventhubclient#Microsoft_Azure_EventHubs_EventHubClient_CreateFromConnectionString_System_String_)。

```csharp
var connectionStringBuilder = new EventHubsConnectionStringBuilder("Event Hubs connection string")
{
    EntityPath = EhEntityPath
};

var eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

### <a name="send-events"></a>傳送事件

若要將事件傳送到事件中樞，請使用 [EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata) 類別。 主體必須是 `byte` 陣列，或者 `byte` 陣列區段。

```csharp
// Create a new EventData object by encoding a string as a byte array
var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));
// Set user properties if needed
data.Properties.Add("Type", "Informational");
// Send single message async
await eventHubClient.SendAsync(data);
```

### <a name="receive-events"></a>接收事件

從事件中樞接收事件的建議方式是使用[事件處理器主機](#event-processor-host-apis)，它提供了自動追蹤事件中樞位移的功能，以及資料分割資訊。 不過，在某些情況中，您可能會想要使用核心事件中樞程式庫的彈性來接收事件。

#### <a name="create-a-receiver"></a>建立接收者

接收者會繫結至特定的資料分割，因此為了要接收事件中樞中的所有事件，您必須建立多個執行個體。 最好以程式設計的方式取得資料分割資訊，而不是以硬式編碼資料分割識別碼的方式。 若要這樣做，您可以使用 [GetRuntimeInformationAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient#Microsoft_Azure_EventHubs_EventHubClient_GetRuntimeInformationAsync) 方法。

```csharp
// Create a list to keep track of the receivers
var receivers = new List<PartitionReceiver>();
// Use the eventHubClient created above to get the runtime information
var runTimeInformation = await eventHubClient.GetRuntimeInformationAsync();
// Loop over the resulting partition IDs
foreach (var partitionId in runTimeInformation.PartitionIds)
{
    // Create the receiver
    var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, PartitionReceiver.EndOfStream);
    // Add the receiver to the list
    receivers.Add(receiver);
}
```

由於事件一律不會從事件中樞內移除 (只會過期)，因此您必須指定適當的起始點。 下列範例顯示可能的組合：

```csharp
// partitionId is assumed to come from GetRuntimeInformationAsync()

// Using the constant PartitionReceiver.EndOfStream only receives all messages from this point forward.
var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, PartitionReceiver.EndOfStream);

// All messages available
var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, "-1");

// From one day ago
var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, DateTime.Now.AddDays(-1));
```

#### <a name="consume-an-event"></a>使用事件

```csharp
// Receive a maximum of 100 messages in this call to ReceiveAsync
var ehEvents = await receiver.ReceiveAsync(100);
// ReceiveAsync can return null if there are no messages
if (ehEvents != null)
{
    // Since ReceiveAsync can return more than a single event you will need a loop to process
    foreach (var ehEvent in ehEvents)
    {
        // Decode the byte array segment
        var message = UnicodeEncoding.UTF8.GetString(ehEvent.Body.Array);
        // Load the custom property that we set in the send example
        var customType = ehEvent.Properties["Type"];
        // Implement processing logic here
    }
}       
```

## <a name="event-processor-host-apis"></a>Event Processor Host API

這些 API 會透過在可用的背景工作之間散佈資料分割，以便提供恢復功能給可能會變成無法使用的背景工作角色處理序：

```csharp
// Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.

// Read these connection strings from a secure location
var ehConnectionString = "{Event Hubs connection string}";
var ehEntityPath = "{event hub path/name}";
var storageConnectionString = "{Storage connection string}";
var storageContainerName = "{Storage account container name}";

var eventProcessorHost = new EventProcessorHost(
    ehEntityPath,
    PartitionReceiver.DefaultConsumerGroupName,
    ehConnectionString,
    storageConnectionString,
    storageContainerName);

// Start/register an EventProcessorHost
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

// Disposes the Event Processor Host
await eventProcessorHost.UnregisterEventProcessorAsync();
```

以下是 [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) 介面的範例實作：

```csharp
public class SimpleEventProcessor : IEventProcessor
{
    public Task CloseAsync(PartitionContext context, CloseReason reason)
    {
        Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
        return Task.CompletedTask;
    }

    public Task OpenAsync(PartitionContext context)
    {
        Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
        return Task.CompletedTask;
    }

    public Task ProcessErrorAsync(PartitionContext context, Exception error)
    {
        Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
        return Task.CompletedTask;
    }

    public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
        foreach (var eventData in messages)
        {
            var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
            Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
        }

        return context.CheckpointAsync();
    }
}
```

## <a name="next-steps"></a>後續步驟

若要深入了解事件中樞案例，請造訪下列連結：

* [Azure 事件中樞是什麼？](event-hubs-what-is-event-hubs.md)
* [可用的事件中樞 API](event-hubs-api-overview.md)

.NET API 參考如下：

* [Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs)
* [Microsoft.Azure.EventHubs.Processor](/dotnet/api/microsoft.azure.eventhubs.processor)