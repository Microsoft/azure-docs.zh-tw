---
title: 開始使用 Azure 服務匯流排佇列 | Microsoft Docs
description: 撰寫使用服務匯流排傳訊佇列的 C# 主控台應用程式。
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 68a34c00-5600-43f6-bbcc-fea599d500da
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: hero-article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 09/07/2018
ms.author: spelluru
ms.openlocfilehash: 30ff970a1f6243bbba6aa42046cf6e08c808f4e5
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49068855"
---
# <a name="get-started-with-service-bus-queues"></a>開始使用服務匯流排佇列

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

本教學課程涵蓋下列步驟：

1. 使用 Azure 入口網站建立服務匯流排命名空間。
2. 使用 Azure 入口網站建立服務匯流排佇列。
3. 撰寫 .NET Core 主控台應用程式，以將一組訊息傳送到佇列。
4. 撰寫 .NET Core 主控台應用程式，以從佇列接收這些訊息。

## <a name="prerequisites"></a>必要條件

1. [Visual Studio 2017 Update 3 (版本 15.3, 26730.01)](http://www.visualstudio.com/vs) 或更新版本。
2. [.NET Core SDK](https://www.microsoft.com/net/download/windows)，2.0 版或更新版本。
2. Azure 訂用帳戶。

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="1-create-a-namespace-using-the-azure-portal"></a>1.使用 Azure 入口網站建立命名空間

> [!NOTE] 
> 您也可以使用 [PowerShell](/powershell/azure/get-started-azureps) 來建立服務匯流排命名空間和傳訊實體。 如需詳細資訊，請參閱[使用 PowerShell 來管理服務匯流排資源](service-bus-manage-with-ps.md)。

如果您已建立服務匯流排傳訊命名空間，請跳至[使用 Azure 入口網站建立佇列](#2-create-a-queue-using-the-azure-portal)一節。

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="2-create-a-queue-using-the-azure-portal"></a>2.使用 Azure 入口網站建立佇列

如果您已經建立服務匯流排佇列，請跳至[將訊息傳送至佇列](#3-send-messages-to-the-queue)一節。

[!INCLUDE [service-bus-create-queue-portal](../../includes/service-bus-create-queue-portal.md)]

## <a name="3-send-messages-to-the-queue"></a>3.將訊息傳送到佇列

為了將訊息傳送到佇列，請使用 Visual Studio 撰寫 C# 主控台應用程式。

### <a name="create-a-console-application"></a>建立主控台應用程式

啟動 Visual Studio 並建立新的**主控台應用程式 (.NET Core)** 專案。

### <a name="add-the-service-bus-nuget-package"></a>新增服務匯流排 NuGet 封裝

1. 以滑鼠右鍵按一下新建立的專案，然後選取 [管理 NuGet 套件]。
2. 按一下 [瀏覽] 索引標籤，搜尋 **[Microsoft.Azure.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus/)**，然後選取 [Microsoft.Azure.ServiceBus] 項目。 按一下 [安裝]  完成安裝作業，然後關閉此對話方塊。
   
    ![選取 NuGet 封裝][nuget-pkg]

### <a name="write-code-to-send-messages-to-the-queue"></a>撰寫程式碼以將訊息傳送到佇列

1. 在 Program.cs 中，在命名空間定義的頂端新增下列 `using` 陳述式 (在類別宣告的前面)：
   
    ```csharp
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;
    ```

2. 在 `Program` 類別中，宣告下列變數。 將 `ServiceBusConnectionString` 變數設定為建立命名空間時取得的連接字串，並將 `QueueName` 設定為建立佇列時使用的名稱：
   
    ```csharp
    const string ServiceBusConnectionString = "<your_connection_string>";
    const string QueueName = "<your_queue_name>";
    static IQueueClient queueClient;
    ``` 

3. 以下列這一行程式碼取代 `Main()` 的預設內容：

    ```csharp
    MainAsync().GetAwaiter().GetResult();
    ```

4. 直接在 `Main()` 後面，新增下列非同步 `MainAsync()` 方法，以呼叫傳送訊息方法：

    ```csharp
    static async Task MainAsync()
    {
        const int numberOfMessages = 10;
        queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

        Console.WriteLine("======================================================");
        Console.WriteLine("Press ENTER key to exit after sending all the messages.");
        Console.WriteLine("======================================================");

        // Send messages.
        await SendMessagesAsync(numberOfMessages);

        Console.ReadKey();

        await queueClient.CloseAsync();
    }
    ```

5. 直接在 `MainAsync()` 方法後面，新增下列 `SendMessagesAsync()` 方法，以執行傳送 `numberOfMessagesToSend` 所指定訊息數目 (目前設為 10) 的工作：

    ```csharp
    static async Task SendMessagesAsync(int numberOfMessagesToSend)
    {
        try
        {
            for (var i = 0; i < numberOfMessagesToSend; i++)
            {
                // Create a new message to send to the queue.
                string messageBody = $"Message {i}";
                var message = new Message(Encoding.UTF8.GetBytes(messageBody));

                // Write the body of the message to the console.
                Console.WriteLine($"Sending message: {messageBody}");

                // Send the message to the queue.
                await queueClient.SendAsync(message);
            }
        }
        catch (Exception exception)
        {
            Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
        }
    }
    ```
   
6. Program.cs 檔案看起來應該會像下面這樣。
   
    ```csharp
    namespace CoreSenderApp
    {
        using System;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Azure.ServiceBus;

        class Program
        {
            // Connection String for the namespace can be obtained from the Azure portal under the 
            // 'Shared Access policies' section.
            const string ServiceBusConnectionString = "<your_connection_string>";
            const string QueueName = "<your_queue_name>";
            static IQueueClient queueClient;

            static void Main(string[] args)
            {
                MainAsync().GetAwaiter().GetResult();
            }

            static async Task MainAsync()
            {
                const int numberOfMessages = 10;
                queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

                Console.WriteLine("======================================================");
                Console.WriteLine("Press ENTER key to exit after sending all the messages.");
                Console.WriteLine("======================================================");

                // Send Messages
                await SendMessagesAsync(numberOfMessages);
                        
                Console.ReadKey();

                await queueClient.CloseAsync();
            }

            static async Task SendMessagesAsync(int numberOfMessagesToSend)
            {
                try
                {
                    for (var i = 0; i < numberOfMessagesToSend; i++)
                    {
                        // Create a new message to send to the queue
                        string messageBody = $"Message {i}";
                        var message = new Message(Encoding.UTF8.GetBytes(messageBody));

                        // Write the body of the message to the console
                        Console.WriteLine($"Sending message: {messageBody}");

                        // Send the message to the queue
                        await queueClient.SendAsync(message);
                    }
                }
                catch (Exception exception)
                {
                    Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
                }
            }
        }
    }
    ```

7. 執行程式，並檢查 Azure 入口網站：按一下命名空間 [概觀] 視窗中的佇列名稱。 佇列的 [基本資訊] 畫面即會顯示。 請注意，佇列的 [使用中訊息計數] 值現在是 **10**。 每次執行傳送者應用程式而未擷取訊息 (如下一節所述)，這個值就會增加 10。 也請注意，每當應用程式將訊息新增到佇列時，佇列的目前大小就會讓 [基本資訊] 視窗上的 [目前] 值增加。
   
      ![訊息大小][queue-message]

## <a name="4-receive-messages-from-the-queue"></a>4.從佇列接收訊息

若要接收您剛傳送的訊息，請建立另一個 .NET Core 主控台應用程式，並安裝 **Microsoft.Azure.ServiceBus** NuGet 套件 (類似於先前的傳送者應用程式)。

### <a name="write-code-to-receive-messages-from-the-queue"></a>撰寫程式碼以從佇列接收訊息

1. 在 Program.cs 中，在命名空間定義的頂端新增下列 `using` 陳述式 (在類別宣告的前面)：
   
    ```csharp
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;
    ```

2. 在 `Program` 類別中，宣告下列變數。 將 `ServiceBusConnectionString` 變數設定為建立命名空間時取得的連接字串，並將 `QueueName` 設定為建立佇列時使用的名稱：
   
    ```csharp
    const string ServiceBusConnectionString = "<your_connection_string>";
    const string QueueName = "<your_queue_name>";
    static IQueueClient queueClient;
    ```

3. 以下列這一行程式碼取代 `Main()` 的預設內容：

    ```csharp
    MainAsync().GetAwaiter().GetResult();
    ```

4. 直接在 `Main()` 後面，新增下列非同步 `MainAsync()` 方法，以呼叫 `RegisterOnMessageHandlerAndReceiveMessages()` 方法：

    ```csharp
    static async Task MainAsync()
    {
        queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

        Console.WriteLine("======================================================");
        Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
        Console.WriteLine("======================================================");

        // Register the queue message handler and receive messages in a loop
        RegisterOnMessageHandlerAndReceiveMessages();

        Console.ReadKey();

        await queueClient.CloseAsync();
    }
    ```

5. 直接在 `MainAsync()` 方法後面，新增下列方法，以註冊訊息處理常式及接收傳送者應用程式所傳送的訊息：

    ```csharp
    static void RegisterOnMessageHandlerAndReceiveMessages()
    {
        // Configure the message handler options in terms of exception handling, number of concurrent messages to deliver, etc.
        var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
        {
            // Maximum number of concurrent calls to the callback ProcessMessagesAsync(), set to 1 for simplicity.
            // Set it according to how many messages the application wants to process in parallel.
            MaxConcurrentCalls = 1,

            // Indicates whether the message pump should automatically complete the messages after returning from user callback.
            // False below indicates the complete operation is handled by the user callback as in ProcessMessagesAsync().
            AutoComplete = false
        };

        // Register the function that processes messages.
        queueClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
    }
    ```

6. 直接在前一個方法後面，新增下列 `ProcessMessagesAsync()` 方法，以處理收到的訊息：
 
    ```csharp
    static async Task ProcessMessagesAsync(Message message, CancellationToken token)
    {
        // Process the message.
        Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

        // Complete the message so that it is not received again.
        // This can be done only if the queue Client is created in ReceiveMode.PeekLock mode (which is the default).
        await queueClient.CompleteAsync(message.SystemProperties.LockToken);

        // Note: Use the cancellationToken passed as necessary to determine if the queueClient has already been closed.
        // If queueClient has already been closed, you can choose to not call CompleteAsync() or AbandonAsync() etc.
        // to avoid unnecessary exceptions.
    }
    ```

7. 最後，新增下列方法以處理任何可能發生的例外狀況：
 
    ```csharp
    // Use this handler to examine the exceptions received on the message pump.
    static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
    {
        Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
        var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
        Console.WriteLine("Exception context for troubleshooting:");
        Console.WriteLine($"- Endpoint: {context.Endpoint}");
        Console.WriteLine($"- Entity Path: {context.EntityPath}");
        Console.WriteLine($"- Executing Action: {context.Action}");
        return Task.CompletedTask;
    }    
    ```    
   
8. Program.cs 檔案看起來應該會像下面這樣：
   
    ```csharp
    namespace CoreReceiverApp
    {
        using System;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Azure.ServiceBus;

        class Program
        {
            // Connection String for the namespace can be obtained from the Azure portal under the 
            // 'Shared Access policies' section.
            const string ServiceBusConnectionString = "<your_connection_string>";
            const string QueueName = "<your_queue_name>";
            static IQueueClient queueClient;

            static void Main(string[] args)
            {
                MainAsync().GetAwaiter().GetResult();
            }

            static async Task MainAsync()
            {
                queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

                Console.WriteLine("======================================================");
                Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
                Console.WriteLine("======================================================");

                // Register QueueClient's MessageHandler and receive messages in a loop
                RegisterOnMessageHandlerAndReceiveMessages();
                    
                Console.ReadKey();

                await queueClient.CloseAsync();
            }

            static void RegisterOnMessageHandlerAndReceiveMessages()
            {
                // Configure the MessageHandler Options in terms of exception handling, number of concurrent messages to deliver etc.
                var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
                {
                    // Maximum number of Concurrent calls to the callback `ProcessMessagesAsync`, set to 1 for simplicity.
                    // Set it according to how many messages the application wants to process in parallel.
                    MaxConcurrentCalls = 1,

                    // Indicates whether MessagePump should automatically complete the messages after returning from User Callback.
                    // False below indicates the Complete will be handled by the User Callback as in `ProcessMessagesAsync` below.
                    AutoComplete = false
                };

                // Register the function that will process messages
                queueClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
            }

            static async Task ProcessMessagesAsync(Message message, CancellationToken token)
            {
                // Process the message
                Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

                // Complete the message so that it is not received again.
                // This can be done only if the queueClient is created in ReceiveMode.PeekLock mode (which is default).
                await queueClient.CompleteAsync(message.SystemProperties.LockToken);

                // Note: Use the cancellationToken passed as necessary to determine if the queueClient has already been closed.
                // If queueClient has already been Closed, you may chose to not call CompleteAsync() or AbandonAsync() etc. calls 
               // to avoid unnecessary exceptions.
            }

            static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
            {
                Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
                var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
                Console.WriteLine("Exception context for troubleshooting:");
                Console.WriteLine($"- Endpoint: {context.Endpoint}");
                Console.WriteLine($"- Entity Path: {context.EntityPath}");
                Console.WriteLine($"- Executing Action: {context.Action}");
                return Task.CompletedTask;
            }
        }
    }
    ```
4. 執行程式，並再次檢查入口網站。 請注意，[使用中訊息計數] 和 [目前] 值現在是 **0**。
   
    ![佇列長度][queue-message-receive]

恭喜！ 您現已建立佇列、將一組訊息傳送至該佇列，以及從相同佇列接收這些訊息。

## <a name="next-steps"></a>後續步驟

查看 [GitHub 存放庫以及範例](https://github.com/Azure/azure-service-bus/tree/master/samples)，其中會展示一些更進階的服務匯流排傳訊功能。

<!--Image references-->

[nuget-pkg]: ./media/service-bus-dotnet-get-started-with-queues/nuget-package.png
[queue-message]: ./media/service-bus-dotnet-get-started-with-queues/queue-message.png
[queue-message-receive]: ./media/service-bus-dotnet-get-started-with-queues/queue-message-receive.png

