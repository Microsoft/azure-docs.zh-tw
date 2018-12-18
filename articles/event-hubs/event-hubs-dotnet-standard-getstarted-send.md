---
title: 使用 .NET Standard 傳送事件至 Azure 事件中樞 | Microsoft Docs
description: 開始在 .NET Standard 中傳送事件至事件中樞
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
ms.openlocfilehash: 6f95d8dc291911ac7506e33b80c2d71c8f50dfdc
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405626"
---
# <a name="get-started-sending-messages-to-azure-event-hubs-in-net-standard"></a>開始在 .NET Standard 中傳送訊息至 Azure 事件中樞

> [!NOTE]
> 您可在 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender) 上取得此範例。

本教學課程說明如何撰寫一個 .NET Core 主控台應用程式，以將一組訊息傳送到事件中樞。 您可以依現狀執行 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender) 解決方案，其中以您事件中樞的值來取代 `EventHubConnectionString` 和 `EventHubName` 字串。 或者，您可以遵循本教學課程中的步驟，來建立自己的解決方案。

## <a name="prerequisites"></a>必要條件

* [Microsoft Visual Studio 2015 或 2017](http://www.visualstudio.com)。 本教學課程中的範例使用 Visual Studio 2017，但也支援 Visual Studio 2015。
* [.NET Core Visual Studio 2015 或 2017 工具](http://www.microsoft.com/net/core)。
* Azure 訂用帳戶。
* [事件中樞命名空間和事件中樞](event-hubs-quickstart-portal.md)。

為了將訊息傳送到事件中樞，本教學課程會使用 Visual Studio 來撰寫一個 C# 主控台應用程式。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>建立事件中樞命名空間和事件中樞

若要建立命名空間和事件中樞，請依照[這篇文章](event-hubs-quickstart-portal.md)中的程序操作，然後繼續進行本教學課程。

## <a name="create-a-console-application"></a>建立主控台應用程式

啟動 Visual Studio。 從 [檔案] 功能表中，按一下 [新增]，再按 [專案]。 建立 .NET Core 主控台應用程式。

![新增專案][1]

## <a name="add-the-event-hubs-nuget-package"></a>新增事件中樞 NuGet 封裝

遵循下列幾個步驟，將 [`Microsoft.Azure.EventHubs`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) .NET 標準程式庫 NuGet 套件新增至您的專案： 

1. 以滑鼠右鍵按一下新建立的專案，然後選取 [管理 NuGet 套件]。
2. 按一下 [瀏覽] 索引標籤，然後搜尋「Microsoft.Azure.EventHubs」並選取 [Microsoft.Azure.EventHubs] 套件。 按一下 [安裝]  完成安裝作業，然後關閉此對話方塊。

## <a name="write-some-code-to-send-messages-to-the-event-hub"></a>撰寫一些程式碼以將訊息傳送到事件中樞

1. 在 Program.cs 檔案開頭處加入 `using` 陳述式：

    ```csharp
    using Microsoft.Azure.EventHubs;
    using System.Text;
    using System.Threading.Tasks;
    ```

2. 針對「事件中樞」連接字串和實體路徑 (個別事件中樞名稱)，將常數新增到 `Program` 類別。 以建立事件中樞時所取得的適當值取代方括號中的預留位置。 請確定 `{Event Hubs connection string}` 是命名空間層級的連接字串，而不是事件中樞字串。 

    ```csharp
    private static EventHubClient eventHubClient;
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    ```

3. 將名為 `MainAsync` 的新方法新增到 `Program` 類別，如下所示：

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
        // Typically, the connection string should have the entity path in it, but this simple scenario
        // uses the connection string from the namespace.
        var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
        {
            EntityPath = EventHubName
        };

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

        await SendMessagesToEventHub(100);

        await eventHubClient.CloseAsync();

        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
    }
    ```

4. 將名為 `SendMessagesToEventHub` 的新方法新增到 `Program` 類別，如下所示：

    ```csharp
    // Creates an event hub client and sends 100 messages to the event hub.
    private static async Task SendMessagesToEventHub(int numMessagesToSend)
    {
        for (var i = 0; i < numMessagesToSend; i++)
        {
            try
            {
                var message = $"Message {i}";
                Console.WriteLine($"Sending message: {message}");
                await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
            }

            await Task.Delay(10);
        }

        Console.WriteLine($"{numMessagesToSend} messages sent.");
    }
    ```

5. 將下列程式碼行新增至 `Program` 類別中的 `Main` 方法：

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

   Program.cs 看起來應該會像下面這樣。

    ```csharp
    namespace SampleSender
    {
        using System;
        using System.Text;
        using System.Threading.Tasks;
        using Microsoft.Azure.EventHubs;

        public class Program
        {
            private static EventHubClient eventHubClient;
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
                // Typically, the connection string should have the entity path in it, but for the sake of this simple scenario
                // we are using the connection string from the namespace.
                var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
                {
                    EntityPath = EventHubName
                };

                eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

                await SendMessagesToEventHub(100);

                await eventHubClient.CloseAsync();

                Console.WriteLine("Press ENTER to exit.");
                Console.ReadLine();
            }

            // Creates an event hub client and sends 100 messages to the event hub.
            private static async Task SendMessagesToEventHub(int numMessagesToSend)
            {
                for (var i = 0; i < numMessagesToSend; i++)
                {
                    try
                    {
                        var message = $"Message {i}";
                        Console.WriteLine($"Sending message: {message}");
                        await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                    }
                    catch (Exception exception)
                    {
                        Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
                    }

                    await Task.Delay(10);
                }

                Console.WriteLine($"{numMessagesToSend} messages sent.");
            }
        }
    }
    ```

6. 執行程式，並確定沒有任何錯誤。

恭喜！ 您現在已將傳送訊息到事件中樞。

## <a name="next-steps"></a>後續步驟
您可以透過下列連結來深入了解事件中樞：

* [從事件中樞接收事件](event-hubs-dotnet-standard-getstarted-receive-eph.md)
* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

[1]: ./media/event-hubs-dotnet-standard-getstarted-send/netcoresnd.png
