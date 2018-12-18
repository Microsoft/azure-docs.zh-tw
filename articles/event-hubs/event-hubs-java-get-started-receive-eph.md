---
title: 使用 Java 從 Azure 事件中樞接收事件 | Microsoft Docs
description: 開始使用 Java 從事件中樞接收事件
services: event-hubs
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.date: 08/26/2018
ms.author: shvija
ms.openlocfilehash: ee1339d02fb23282d3589a80385f982eae2865fe
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128161"
---
# <a name="receive-events-from-azure-event-hubs-using-java"></a>使用 Java 從 Azure 事件中樞接收事件

事件中樞是高度可擴充的擷取系統，每秒可擷取數百萬個事件，讓應用程式能處理並分析已連線裝置與應用程式產生的大量資料。 收集到事件中樞後，您可以使用任何即時分析提供者或儲存體叢集轉換和儲存資料。

如需詳細資訊，請參閱 [事件中樞概觀][Event Hubs overview]。

本教學課程也會示範如何使用以 Java 撰寫的主控台應用程式，將事件接收到事件中樞。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* Java 開發環境。 針對本教學課程，我們採用 [Eclipse](https://www.eclipse.org/)。
* 使用中的 Azure 帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶][]。

本教學課程中的程式碼是根據 [GitHub 上的 EventProcessorSample 程式碼](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/EventProcessorSample)，您可以檢查該程式碼以查看完整的運作中應用程式。

## <a name="receive-messages-with-eventprocessorhost-in-java"></a>在 Java 中使用 EventProcessorHost 接收訊息

**EventProcessorHost** 是一個 Java 類別，透過管理持續檢查點以及來自事件中樞的平行接收，簡化來自事件中樞之事件的接收作業。 使用 EventProcessorHost，您可以將事件分割到多個接收者，即使裝載於不同的節點時也是一樣。 這個範例顯示單一接收者如何使用 EventProcessorHost。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

若要使用 EventProcessorHost，您必須擁有 [Azure 儲存體帳戶][Azure Storage account]：

1. 登入 [Azure 入口網站][Azure portal]，然後按一下畫面左上方的 [+ 建立資源]。
2. 按一下 [儲存體]，然後按一下 [儲存體帳戶]。 在 [建立儲存體帳戶] 視窗中，輸入儲存體帳戶名稱。 完成其餘欄位，選取您想要的區域，然後按一下 [建立]。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage2.png)

3. 按一下新建立的儲存體帳戶，然後按一下 [存取金鑰]：
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage3.png)

    將 key1 值複製到暫存位置，以便稍後在此教學課程中使用。

### <a name="create-a-java-project-using-the-eventprocessor-host"></a>使用 EventProcessor 主機建立 Java 專案

適用於事件中樞的 Java 用戶端程式庫可以在來自 [Maven 中央儲存機制][Maven Package]的 Maven 專案中使用，而且可在您的 Maven 專案檔內使用下列相依性宣告來參考。 成品 azure-eventhubs-eph 目前的版本是 2.0.1，而成品 azure-eventhubs 目前的版本是 1.0.2：    

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>1.0.2</version>
</dependency>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs-eph</artifactId>
    <version>2.0.1</version>
</dependency>
```

對於不同類型的組建環境，您可以明確從 [Maven 中央儲存機制][Maven Package]取得最新發佈的 JAR 檔案。  

1. 針對下列範例，在您最喜愛的 Java 開發環境中，先為主控台/殼層應用程式建立新的 Maven 專案。 類別稱為 `ErrorNotificationHandler`。     
   
    ```java
    import java.util.function.Consumer;
    import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   
    public class ErrorNotificationHandler implements Consumer<ExceptionReceivedEventArgs>
    {
        @Override
        public void accept(ExceptionReceivedEventArgs t)
        {
            System.out.println("SAMPLE: Host " + t.getHostname() + " received general error notification during " + t.getAction() + ": " + t.getException().toString());
        }
    }
    ```
2. 使用下列程式碼，建立名為 `EventProcessorSample`的新類別。 使用您建立事件中樞和儲存體帳戶時所用的值，取代預留位置：
   
   ```java
   package com.microsoft.azure.eventhubs.samples.eventprocessorsample;

   import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
   import com.microsoft.azure.eventhubs.EventData;
   import com.microsoft.azure.eventprocessorhost.CloseReason;
   import com.microsoft.azure.eventprocessorhost.EventProcessorHost;
   import com.microsoft.azure.eventprocessorhost.EventProcessorOptions;
   import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   import com.microsoft.azure.eventprocessorhost.IEventProcessor;
   import com.microsoft.azure.eventprocessorhost.PartitionContext;

   import java.util.concurrent.ExecutionException;
   import java.util.function.Consumer;

   public class EventProcessorSample
   {
       public static void main(String args[]) throws InterruptedException, ExecutionException
       {
           String consumerGroupName = "$Default";
           String namespaceName = "----NamespaceName----";
           String eventHubName = "----EventHubName----";
           String sasKeyName = "----SharedAccessSignatureKeyName----";
           String sasKey = "----SharedAccessSignatureKey----";
           String storageConnectionString = "----AzureStorageConnectionString----";
           String storageContainerName = "----StorageContainerName----";
           String hostNamePrefix = "----HostNamePrefix----";
        
           ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder()
                .setNamespaceName(namespaceName)
                .setEventHubName(eventHubName)
                .setSasKeyName(sasKeyName)
                .setSasKey(sasKey);
        
           EventProcessorHost host = new EventProcessorHost(
                EventProcessorHost.createHostName(hostNamePrefix),
                eventHubName,
                consumerGroupName,
                eventHubConnectionString.toString(),
                storageConnectionString,
                storageContainerName);
        
           System.out.println("Registering host named " + host.getHostName());
           EventProcessorOptions options = new EventProcessorOptions();
           options.setExceptionNotification(new ErrorNotificationHandler());

           host.registerEventProcessor(EventProcessor.class, options)
           .whenComplete((unused, e) ->
           {
               if (e != null)
               {
                   System.out.println("Failure while registering: " + e.toString());
                   if (e.getCause() != null)
                   {
                       System.out.println("Inner exception: " + e.getCause().toString());
                   }
               }
           })
           .thenAccept((unused) ->
           {
               System.out.println("Press enter to stop.");
               try 
               {
                   System.in.read();
               }
               catch (Exception e)
               {
                   System.out.println("Keyboard read failed: " + e.toString());
               }
           })
           .thenCompose((unused) ->
           {
               return host.unregisterEventProcessor();
           })
           .exceptionally((e) ->
           {
               System.out.println("Failure while unregistering: " + e.toString());
               if (e.getCause() != null)
               {
                   System.out.println("Inner exception: " + e.getCause().toString());
               }
               return null;
           })
           .get(); // Wait for everything to finish before exiting main!
        
           System.out.println("End of sample");
       }
    ```
3. 使用下列程式碼，再建立一個名為 `EventProcessor` 的類別：
   
    ```java
    public static class EventProcessor implements IEventProcessor
    {
        private int checkpointBatchingCount = 0;

        // OnOpen is called when a new event processor instance is created by the host. 
        @Override
        public void onOpen(PartitionContext context) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is opening");
        }

        // OnClose is called when an event processor instance is being shut down. 
        @Override
        public void onClose(PartitionContext context, CloseReason reason) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is closing for reason " + reason.toString());
        }
        
        // onError is called when an error occurs in EventProcessorHost code that is tied to this partition, such as a receiver failure.
        @Override
        public void onError(PartitionContext context, Throwable error)
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " onError: " + error.toString());
        }

        // onEvents is called when events are received on this partition of the Event Hub. 
        @Override
        public void onEvents(PartitionContext context, Iterable<EventData> events) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " got event batch");
            int eventCount = 0;
            for (EventData data : events)
            {
                try
                {
                    System.out.println("SAMPLE (" + context.getPartitionId() + "," + data.getSystemProperties().getOffset() + "," +
                            data.getSystemProperties().getSequenceNumber() + "): " + new String(data.getBytes(), "UTF8"));
                    eventCount++;
                    
                    // Checkpointing persists the current position in the event stream for this partition and means that the next
                    // time any host opens an event processor on this event hub+consumer group+partition combination, it will start
                    // receiving at the event after this one. 
                    this.checkpointBatchingCount++;
                    if ((checkpointBatchingCount % 5) == 0)
                    {
                        System.out.println("SAMPLE: Partition " + context.getPartitionId() + " checkpointing at " +
                            data.getSystemProperties().getOffset() + "," + data.getSystemProperties().getSequenceNumber());
                        // Checkpoints are created asynchronously. It is important to wait for the result of checkpointing
                        // before exiting onEvents or before creating the next checkpoint, to detect errors and to ensure proper ordering.
                        context.checkpoint(data).get();
                    }
                }
                catch (Exception e)
                {
                    System.out.println("Processing failed for an event: " + e.toString());
                }
            }
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " batch size was " + eventCount + " for host " + context.getOwner());
        }
    }
    ```

本教學課程使用單一 EventProcessorHost 執行個體。 若要增加輸送量，建議您執行多個 EventProcessorHost 執行個體 (最好能在個別的機器上)。  這麼做也會提供備援性。 在這些情況下，各種執行個體會自動彼此協調以對已接收的事件進行負載平衡。 如果您想要多個接收者都處理 *所有* 事件，則必須使用 **ConsumerGroup** 概念。 收到來自不同電腦的事件時，根據在其中執行 EventProcessorHost 執行個體的電腦 (或角色) 來指定名稱可能十分有用。

## <a name="publishing-messages-to-eventhub"></a>將訊息發佈至 EventHub

在取用者擷取訊息之前，發行者必須先將這些訊息發佈至分割區。 需注意的是，在 com.microsoft.azure.eventhubs.EventHubClient 物件上使用 sendSync() 方法將訊息同步發佈到事件中樞後，可以根據是否指定分割區索引鍵，將訊息傳送到特定分割區或以循環配置方式分散到所有可用的分割區。

指定代表分割索引鍵的字串後，將會雜湊處理此索引鍵來判斷事件要傳送到哪個分割區。

若未設定分割區索引鍵，則訊息會循環配置到所有可用的分割區

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

```

## <a name="implementing-a-custom-checkpointmanager-for-eventprocessorhost-eph"></a>實作 EventProcessorHost (EPH) 的自訂 CheckpointManager

此 API 所提供的機制，可讓您在預設的實作方式與您的使用案例不相容時實作自訂檢查點管理程式。

預設的檢查點管理程式會使用 Blob 儲存體，但如果您以自訂實作方式覆寫 EPH 所使用的檢查點管理程式，您將可使用任何存放區來支援檢查點管理程式實作。

您必須建立會實作 com.microsoft.azure.eventprocessorhost.ICheckpointManager 介面的類別

請使用您自訂的檢查點管理程式實作 (com.microsoft.azure.eventprocessorhost.ICheckpointManager)

在您的實作中，您可以覆寫預設的檢查點機制，並根據您自己的資料存放區 (SQL Server、CosmosDB、Redis 快取等) 實作自己的檢查點。 我們建議，用來支援檢查點管理程式實作的存放區，應可供所有處理取用者群組事件的 EPH 執行個體存取。

您可以使用在環境中將可供使用的任何資料存放區。

com.microsoft.azure.eventprocessorhost.EventProcessorHost 類別可為您提供 2 個建構函式，讓您覆寫 EventProcessorHost 的檢查點管理程式。

## <a name="next-steps"></a>後續步驟

您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: event-hubs-what-is-event-hubs.md
[Azure Storage account]: ../storage/common/storage-create-storage-account.md
[Azure portal]: https://portal.azure.com
[Maven Package]: https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22

<!-- Images -->
[11]: ./media/service-bus-event-hubs-get-started-receive-ephjava/create-eph-csharp2.png
[12]: ./media/service-bus-event-hubs-get-started-receive-ephjava/create-eph-csharp3.png
[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
