---
title: 使用 .NET Standard 程式庫從 Azure 事件中樞接收事件 | Microsoft Docs
description: 開始使用 .NET Standard 中的 EventProcessorHost 接收訊息
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: 5abb2447fa90ea5900afb86746cc17eff62c2d2e
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166271"
---
# <a name="get-started-receiving-messages-with-the-event-processor-host-in-net-standard"></a>開始使用 .NET Standard 中的事件處理器主機來接收訊息

> [!NOTE]
> 您可在 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver) 上取得此範例。

本教學課程說明如何撰寫 .NET Core 主控台應用程式，以使用**事件處理器主機**程式庫從事件中樞接收訊息。 您可以依現狀執行 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver) 解決方案，其中以您事件中樞和儲存體帳戶的值來取代字串。 或者，您可以遵循本教學課程中的步驟，來建立自己的解決方案。

## <a name="prerequisites"></a>必要條件

* [Microsoft Visual Studio 2015 或 2017](http://www.visualstudio.com)。 本教學課程中的範例使用 Visual Studio 2017，但也支援 Visual Studio 2015。
* [.NET Core Visual Studio 2015 或 2017 工具](http://www.microsoft.com/net/core)。
* Azure 訂用帳戶。
* Azure 事件中樞命名空間和事件中樞。
* 一個 Azure 儲存體帳戶。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>建立事件中樞命名空間和事件中樞  

第一個步驟是使用 [Azure 入口網站](https://portal.azure.com)來建立「事件中樞」類型的命名空間，然後取得您應用程式與事件中樞進行通訊所需的管理認證。 若要建立命名空間和事件中樞，請依照[這篇文章](event-hubs-create.md)中的程序操作，然後繼續進行本教學課程。  

## <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶  

1. 登入 [Azure 入口網站](https://portal.azure.com)。  
2. 在入口網站的左側導覽窗格中，依序選取 [建立資源]、類別中的 [儲存體] 以及 [儲存體帳戶 - Blob、檔案、資料表、佇列]。  
3. 完成 [建立儲存體帳戶] 視窗中的欄位，然後選取 [檢閱 + 建立]。 

    ![建立儲存體帳戶][1]

4. 在 [檢閱 + 建立] 頁面上檢閱欄位的值，然後選取 [建立]。 
5. 在看到「部署成功」訊息之後，選取新儲存體帳戶的名稱。 
6. 在 [基本資訊] 視窗中，選取 [Blob]。 
7. 選取位於頂端的 [+ 容器]。 指定容器的名稱。  
8. 選取左側視窗中的 [存取金鑰]，然後複製儲存體容器的名稱、儲存體帳戶及 **key1** 的值。 

    將這些值儲存到記事本或一些其他暫存位置。

## <a name="create-a-console-application"></a>建立主控台應用程式

啟動 Visual Studio。 從 [檔案] 功能表中，按一下 [新增]，再按 [專案]。 建立 .NET Core 主控台應用程式。

![新增專案][2]

## <a name="add-the-event-hubs-nuget-package"></a>新增事件中樞 NuGet 封裝

遵循下列步驟，在您的專案中新增 [**Microsoft.Azure.EventHubs**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) 和 [**Microsoft.Azure.EventHubs.Processor**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) .NET Standard 程式庫 NuGet 套件： 

1. 以滑鼠右鍵按一下新建立的專案，然後選取 [管理 NuGet 套件]。
2. 按一下 [瀏覽] 索引標籤，然後搜尋 **Microsoft.Azure.EventHubs** 並選取 [Microsoft.Azure.EventHubs] 套件。 按一下 [安裝]  完成安裝作業，然後關閉此對話方塊。
3. 重複步驟 1 和 2，並安裝 **Microsoft.Azure.EventHubs.Processor** 套件。

## <a name="implement-the-ieventprocessor-interface"></a>實作 IEventProcessor 介面

1. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，按一下 [新增]，然後按一下 [類別]。 將新類別命名為 **SimpleEventProcessor**。

2. 開啟 SimpleEventProcessor.cs 檔案，然後在檔案開頭新增下列 `using` 陳述式。

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

3. 實作 `IEventProcessor` 介面。 以下列程式碼取代 `SimpleEventProcessor` 類別的整個內容：

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

## <a name="write-a-main-console-method-that-uses-the-simpleeventprocessor-class-to-receive-messages"></a>撰寫使用 SimpleEventProcessor 類別來接收訊息的主要主控台方法

1. 在 Program.cs 檔案開頭處加入 `using` 陳述式。

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

2. 針對事件中樞連接字串、事件中樞名稱、儲存體帳戶容器名稱、儲存體帳戶名稱及儲存體帳戶金鑰，將常數新增到 `Program` 類別。 新增下列程式碼，其中將預留位置取代成其對應的值：

    ```csharp
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    private const string StorageContainerName = "{Storage account container name}";
    private const string StorageAccountName = "{Storage account name}";
    private const string StorageAccountKey = "{Storage account key}";

    private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);
    ```   

3. 將名為 `MainAsync` 的新方法新增到 `Program` 類別，如下所示：

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        Console.WriteLine("Registering EventProcessor...");

        var eventProcessorHost = new EventProcessorHost(
            EventHubName,
            PartitionReceiver.DefaultConsumerGroupName,
            EventHubConnectionString,
            StorageConnectionString,
            StorageContainerName);

        // Registers the Event Processor Host and starts receiving messages
        await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

        Console.WriteLine("Receiving. Press ENTER to stop worker.");
        Console.ReadLine();

        // Disposes of the Event Processor Host
        await eventProcessorHost.UnregisterEventProcessorAsync();
    }
    ```

3. 將下列程式碼行新增至 `Main` 方法：

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

    Program.cs 檔案看起來應該會像下面這樣：

    ```csharp
    namespace SampleEphReceiver
    {

        public class Program
        {
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";
            private const string StorageContainerName = "{Storage account container name}";
            private const string StorageAccountName = "{Storage account name}";
            private const string StorageAccountKey = "{Storage account key}";

            private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                Console.WriteLine("Registering EventProcessor...");

                var eventProcessorHost = new EventProcessorHost(
                    EventHubName,
                    PartitionReceiver.DefaultConsumerGroupName,
                    EventHubConnectionString,
                    StorageConnectionString,
                    StorageContainerName);

                // Registers the Event Processor Host and starts receiving messages
                await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

                Console.WriteLine("Receiving. Press ENTER to stop worker.");
                Console.ReadLine();

                // Disposes of the Event Processor Host
                await eventProcessorHost.UnregisterEventProcessorAsync();
            }
        }
    }
    ```

4. 執行程式，並確定沒有任何錯誤。

恭喜！ 您現在已使用「事件處理器主機」從事件中樞接收訊息。

## <a name="next-steps"></a>後續步驟
您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

[1]: ./media/event-hubs-dotnet-standard-getstarted-receive-eph/event-hubs-python1.png
[2]: ./media/event-hubs-dotnet-standard-getstarted-receive-eph/netcorercv.png
