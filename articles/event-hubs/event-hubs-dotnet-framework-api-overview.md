---
title: Azure 事件中樞 .NET Framework API 概觀 | Microsoft Docs
description: 一些主要事件中樞 .NET Framework 用戶端 API 的摘要。
services: event-hubs
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: dotnet
ms.topic: article
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: 46b0599e6c4d5b4cc2dd9370a28616019b79b50d
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42745646"
---
# <a name="event-hubs-net-framework-api-overview"></a>事件中樞 .NET Framework API 概觀

本文將摘要列出一些主要 Azure 事件中樞 [.NET Framework 用戶端 API](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)。 分為兩種類別：管理和執行階段 API。 執行階段 API 是由傳送和接收訊息所需的所有作業組成。 管理作業可讓您管理事件中樞實體狀態，方法是建立、更新和刪除實體。

[監視案例](event-hubs-metrics-azure-monitor.md)跨越管理和執行階段。 如需 .NET API 的詳細參考文件，請參閱 [.NET Framework](/dotnet/api/microsoft.servicebus.messaging.eventhubclient)、[.NET Standard](/dotnet/api/microsoft.azure.eventhubs) 和 [EventProcessorHost API](/dotnet/api/microsoft.azure.eventhubs.processor) 參考。

## <a name="management-apis"></a>管理 API

若要執行下列管理作業，您必須擁有事件中樞命名空間的 **管理** 權限：

### <a name="create"></a>建立

```csharp
// Create the event hub
var ehd = new EventHubDescription(eventHubName);
ehd.PartitionCount = SampleManager.numPartitions;
await namespaceManager.CreateEventHubAsync(ehd);
```

### <a name="update"></a>更新

```csharp
var ehd = await namespaceManager.GetEventHubAsync(eventHubName);

// Create a customer SAS rule with Manage permissions
ehd.UserMetadata = "Some updated info";
var ruleName = "myeventhubmanagerule";
var ruleKey = SharedAccessAuthorizationRule.GenerateRandomKey();
ehd.Authorization.Add(new SharedAccessAuthorizationRule(ruleName, ruleKey, new AccessRights[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send} )); 
await namespaceManager.UpdateEventHubAsync(ehd);
```

### <a name="delete"></a>刪除

```csharp
await namespaceManager.DeleteEventHubAsync("event hub name");
```

## <a name="run-time-apis"></a>執行階段 API
### <a name="create-publisher"></a>建立發佈者

```csharp
// EventHubClient model (uses implicit factory instance, so all links on same connection)
var eventHubClient = EventHubClient.Create("event hub name");
```

### <a name="publish-message"></a>發佈訊息

```csharp
// Create the device/temperature metric
var info = new MetricEvent() { DeviceId = random.Next(SampleManager.NumDevices), Temperature = random.Next(100) };
var data = new EventData(new byte[10]); // Byte array
var data = new EventData(Stream); // Stream 
var data = new EventData(info, serializer) //Object and serializer 
{
    PartitionKey = info.DeviceId.ToString()
};

// Set user properties if needed
data.Properties.Add("Type", "Telemetry_" + DateTime.Now.ToLongTimeString());

// Send single message async
await client.SendAsync(data);
```

### <a name="create-consumer"></a>建立取用者

```csharp
// Create the Event Hubs client
var eventHubClient = EventHubClient.Create(EventHubName);

// Get the default consumer group
var defaultConsumerGroup = eventHubClient.GetDefaultConsumerGroup();

// All messages
var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index);

// From one day ago
var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index, startingDateTimeUtc:DateTime.Now.AddDays(-1));

// From specific offset, -1 means oldest
var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index,startingOffset:-1); 
```

### <a name="consume-message"></a>取用訊息

```csharp
var message = await consumer.ReceiveAsync();

// Provide a serializer
var info = message.GetBody<Type>(Serializer)

// Get a byte[]
var info = message.GetBytes(); 
msg = UnicodeEncoding.UTF8.GetString(info);
```

## <a name="event-processor-host-apis"></a>Event Processor Host API

這些 API 會透過在可用的背景工作之間散佈資料分割，提供恢復功能給可能會變成無法使用的背景工作角色處理序。

```csharp
// Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.
// Use the EventData.Offset value for checkpointing yourself, this value is unique per partition.

var eventHubConnectionString = System.Configuration.ConfigurationManager.AppSettings["Microsoft.ServiceBus.ConnectionString"];
var blobConnectionString = System.Configuration.ConfigurationManager.AppSettings["AzureStorageConnectionString"]; // Required for checkpoint/state

var eventHubDescription = new EventHubDescription(EventHubName);
var host = new EventProcessorHost(WorkerName, EventHubName, defaultConsumerGroup.GroupName, eventHubConnectionString, blobConnectionString);
await host.RegisterEventProcessorAsync<SimpleEventProcessor>();

// To close
await host.UnregisterEventProcessorAsync();
```

[IEventProcessor](/dotnet/api/microsoft.servicebus.messaging.ieventprocessor) 介面定義如下：

```csharp
public class SimpleEventProcessor : IEventProcessor
{
    IDictionary<string, string> map;
    PartitionContext partitionContext;

    public SimpleEventProcessor()
    {
        this.map = new Dictionary<string, string>();
    }

    public Task OpenAsync(PartitionContext context)
    {
        this.partitionContext = context;

        return Task.FromResult<object>(null);
    }

    public async Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
        foreach (EventData message in messages)
        {
            // Process messages here
        }

        // Checkpoint when appropriate
        await context.CheckpointAsync();

    }

    public async Task CloseAsync(PartitionContext context, CloseReason reason)
    {
        if (reason == CloseReason.Shutdown)
        {
            await context.CheckpointAsync();
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

若要深入了解事件中樞案例，請造訪下列連結：

* [Azure 事件中樞是什麼？](event-hubs-what-is-event-hubs.md)
* [事件中樞程式設計指南](event-hubs-programming-guide.md)

.NET API 參考如下：

* [Microsoft.ServiceBus.Messaging](/dotnet/api/microsoft.servicebus.messaging)
* [Microsoft.Azure.EventHubs.EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost)
