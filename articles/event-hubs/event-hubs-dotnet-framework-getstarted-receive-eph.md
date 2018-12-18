---
title: 使用 .NET Framework 從 Azure 事件中樞接收事件 | Microsoft Docs
description: 請遵循本教學課程，以使用 .NET Framework 從 Azure 事件中樞接收事件。
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: c4974bd3-2a79-48a1-aa3b-8ee2d6655b28
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/02/2018
ms.author: shvija
ms.openlocfilehash: 15c2ec0aa8b795a826eae29026b7039491dbb64f
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42141307"
---
# <a name="receive-events-from-azure-event-hubs-using-the-net-framework"></a>使用 .NET Framework 從 Azure 事件中樞接收事件

## <a name="introduction"></a>簡介

Azure 事件中樞是一項服務，可處理來自連線裝置和應用程式的大量事件資料 (遙測資料)。 收集資料至「事件中樞」之後，可以使用存放裝置叢集來儲存資料，或使用即時分析提供者進行轉換。 此大規模事件收集和處理功能是新型應用程式架構 (包括物聯網 (IoT)) 的重要元件。

本教學課程說明如何撰寫一個使用**[事件處理器主機][Event Processor Host]** 從「事件中樞」接收訊息的 .NET Core 主控台應用程式。 若要使用 .NET Framework 傳送事件，請參閱[使用 .NET Framework 將事件傳送至 Azure 事件中樞](event-hubs-dotnet-framework-getstarted-send.md)一文，或按一下左側目錄中適當的傳送語言。

[事件處理器主機][EventProcessorHost]是一個 .NET 類別，透過管理持續檢查點以及來自事件中樞的平行接收，簡化來自事件中樞之事件的接收作業。 使用事件處理器主機，可讓您將事件分割到多個接收者，即使裝載於不同的節點時也是一樣。 此範例說明單一接收者如何使用事件處理器主機。 [相應放大事件處理][Scale out Event Processing with Event Hubs]範例說明如何搭配使用事件處理器主機與多個接收者。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* [Microsoft Visual Studio 2017 或更高版本](http://visualstudio.com)。
* 使用中的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/free/)。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>建立事件中樞命名空間和事件中樞

第一個步驟是使用 [Azure 入口網站](https://portal.azure.com)來建立「事件中樞」類型的命名空間，然後取得您應用程式與「事件中樞」進行通訊所需的管理認證。 若要建立命名空間和「事件中樞」，請依照[這篇文章](event-hubs-create.md)中的程序操作，然後繼續進行本教學課程中的下列步驟。

## <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶

若要使用[事件處理器主機][EventProcessorHost]，您必須擁有 [Azure 儲存體帳戶][Azure Storage account]：

1. 登入 [Azure 入口網站][Azure portal]，然後按一下畫面左上方的 [建立資源]。

2. 按一下 [儲存體]，然後按一下 [儲存體帳戶]。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage1.png)

3. 在 [建立儲存體帳戶] 窗格中，輸入儲存體帳戶名稱。 選擇要在其中建立資源的 Azure 訂用帳戶、資源群組和位置。 接著，按一下 [建立]。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage2.png)

4. 在儲存體帳戶清單中，按一下新建立的儲存體帳戶。

5. 在 [儲存體帳戶] 窗格中，按一下 [存取金鑰]。 複製 **key1** 的值以便稍後在此教學課程中使用。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage3.png)

## <a name="create-a-receiver-console-application"></a>建立接收者主控台應用程式

1. 在 Visual Studio 中，使用**主控台應用程式**專案範本建立新的「Visual C# 桌面應用程式」專案。 將專案命名為 **Receiver**。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-receiver-csharp1.png)
2. 在 [方案總管] 中，以滑鼠右鍵按一下 [收件者]專案，然後按一下 [管理方案的 NuGet 封裝]。
3. 按一下 [瀏覽] 索引標籤，然後搜尋 `Microsoft Azure Service Bus Event Hub - EventProcessorHost`。 按一下 [安裝] 並接受使用條款。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-eph-csharp1.png)
   
    Visual Studio 會下載、安裝並加入 [Azure 服務匯流排事件中樞 - EventProcessorHost NuGet 封裝](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost)的參考與其所有相依性。
4. 在 [接收者] 專案上按一下滑鼠右鍵，按一下 [新增]，然後按一下 [類別]。 將新類別命名為 **SimpleEventProcessor**，然後按一下 [新增] 以建立該類別。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-receiver-csharp2.png)
5. 在 SimpleEventProcessor.cs 檔案開頭處新增下列陳述式：
    
  ```csharp
  using Microsoft.ServiceBus.Messaging;
  using System.Diagnostics;
  ```
    
  接著，將該類別的主體取代為下列程式碼：
    
  ```csharp
  class SimpleEventProcessor : IEventProcessor
  {
    Stopwatch checkpointStopWatch;
    
    async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
    {
        Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
        if (reason == CloseReason.Shutdown)
        {
            await context.CheckpointAsync();
        }
    }
    
    Task IEventProcessor.OpenAsync(PartitionContext context)
    {
        Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
        this.checkpointStopWatch = new Stopwatch();
        this.checkpointStopWatch.Start();
        return Task.FromResult<object>(null);
    }
    
    async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
        foreach (EventData eventData in messages)
        {
            string data = Encoding.UTF8.GetString(eventData.GetBytes());
    
            Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
                context.Lease.PartitionId, data));
        }
    
        //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
        if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
        {
            await context.CheckpointAsync();
            this.checkpointStopWatch.Restart();
        }
    }
  }
  ```
    
  **EventProcessorHost** 會呼叫這個類別，來處理接收自事件中樞的事件。 `SimpleEventProcessor` 類別會使用碼錶定期在 **EventProcessorHost** 內容上呼叫檢查點方法。 這個流程可確保重新啟動接收者時，遺失的處理工作不超過五分鐘。
6. 在 **Program** 類別檔案的頂端，新增下列 `using` 陳述式：
    
  ```csharp
  using Microsoft.ServiceBus.Messaging;
  ```
    
  然後，以下列程式碼取代 `Program` 類別中的 `Main` 方法，並替代您先前儲存的事件中樞名稱和命名空間層級連接字串，以及您在先前各節中複製的儲存體帳戶和金鑰。 
    
  ```csharp
  static void Main(string[] args)
  {
    string eventHubConnectionString = "{Event Hubs namespace connection string}";
    string eventHubName = "{Event Hub name}";
    string storageAccountName = "{storage account name}";
    string storageAccountKey = "{storage account key}";
    string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);
    
    string eventProcessorHostName = Guid.NewGuid().ToString();
    EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
    Console.WriteLine("Registering EventProcessor...");
    var options = new EventProcessorOptions();
    options.ExceptionReceived += (sender, e) => { Console.WriteLine(e.Exception); };
    eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>(options).Wait();
    
    Console.WriteLine("Receiving. Press enter key to stop worker.");
    Console.ReadLine();
    eventProcessorHost.UnregisterEventProcessorAsync().Wait();
  }
  ```

7. 執行程式，並確定沒有任何錯誤。
  
恭喜！ 您現在已使用「事件處理器主機」從「事件中樞」收到訊息。


> [!NOTE]
> 本教學課程使用單一 [EventProcessorHost][EventProcessorHost] 執行個體。 若要增加輸送量，建議您執行多個 [EventProcessorHost][EventProcessorHost] 執行個體 (如[相應放大事件處理](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3)範例所示)。 在這些情況下，各種執行個體會自動彼此協調，以對已接收的事件進行負載平衡。 如果您想要多個接收者都處理 *所有* 事件，則必須使用 **ConsumerGroup** 概念。 收到來自不同電腦的事件時，根據在其中執行 [EventProcessorHost][EventProcessorHost] 執行個體的電腦 (或角色) 來指定名稱可能十分有用。 如需這些主題的詳細資訊，請參閱[事件中樞概觀][Event Hubs overview]和[事件中樞程式設計指南][Event Hubs Programming Guide]主題。
> 
> 

## <a name="next-steps"></a>後續步驟

您現在已建置一個可建立事件中樞及傳送和接收資料的工作應用程式，您可造訪下列連結來深入了解詳細資訊：

* [事件處理器主機概觀][Event Processor Host]
* [事件中樞概觀][Event Hubs overview]
* [事件中樞常見問題集](event-hubs-faq.md)

<!-- Images. -->
[19]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj1.png
[20]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj2.png
[21]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs2.png

<!-- Links -->
[EventProcessorHost]: /dotnet/api/microsoft.servicebus.messaging.eventprocessorhost
[Event Hubs overview]: event-hubs-about.md
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[Event Hubs Programming Guide]: event-hubs-programming-guide.md
[Azure Storage account]:../storage/common/storage-create-storage-account.md
[Event Processor Host]: event-hubs-event-processor-host.md
[Azure portal]: https://portal.azure.com
