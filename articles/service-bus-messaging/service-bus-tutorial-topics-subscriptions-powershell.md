---
title: 教學課程 - 透過 Azure PowerShell 使用發佈/訂閱通道和主題篩選來更新零售庫存商品 | Microsoft Docs
description: 在本教學課程中，您將了解如何從主題和訂用帳戶傳送及接收訊息，以及如何使用 Azure PowerShell 新增和使用篩選規則
services: service-bus-messaging
author: spelluru
manager: timlt
ms.author: spelluru
ms.date: 09/22/2018
ms.topic: tutorial
ms.service: service-bus-messaging
ms.custom: mvc
ms.openlocfilehash: cda2162873b612cb138487e3d120a3bce3fea1ae
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407003"
---
# <a name="tutorial-update-inventory-using-powershell-and-topicssubscriptions"></a>教學課程：使用 PowerShell 和主題/訂用帳戶來更新庫存

Microsoft Azure 服務匯流排是一項多租用戶雲端傳訊服務，可在應用程式和服務之間傳送資訊。 非同步作業可讓您進行靈活的代理傳訊，搭配結構化的先進先出 (FIFO) 傳訊及發佈/訂閱功能。 

本教學課程說明如何使用 PowerShell 建立傳訊命名空間和該命名空間內的佇列，以對服務匯流排佇列傳送和接收訊息，以及如何取得該命名空間上的授權認證。 程序接著會說明如何使用 [.NET Standard 程式庫](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus)從這個佇列傳送和接收訊息。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 Azure PowerShell 建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 PowerShell 新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

此案例的範例是多個零售商店的庫存商品更新。 在此案例中，每個 (或每組) 商店都會收到預定要傳送給他們的訊息，用以更新其商品。 本教學課程將說明如何使用訂用帳戶和篩選實作此案例。 首先，您會以 3 個訂用帳戶建立主題、新增的某些規則和篩選條件，然後從主題和訂用帳戶來傳送和接收訊息。

![主題](./media/service-bus-tutorial-topics-subscriptions-powershell/about-service-bus-topic.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶][]。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您已安裝︰

1. [Visual Studio 2017 Update 3 (版本 15.3, 26730.01)](http://www.visualstudio.com/vs) 或更新版本。
2. [.NET Core SDK](https://www.microsoft.com/net/download/windows)，2.0 版或更新版本。

進行本教學課程時，您必須執行最新版本的 Azure PowerShell。 如果您需要安裝或升級，請參閱[安裝和設定 Azure PowerShell][]。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>登入 Azure

發出下列命令以登入 Azure。 如果您在 Cloud Shell 中執行 PowerShell 命令，則不需要執行這些步驟： 

1. 安裝服務匯流排 PowerShell 模組：

   ```azurepowershell-interactive
   Install-Module AzureRM.ServiceBus
   ```

2. 執行下列命令以登入 Azure：

   ```azurepowershell-interactive
   Login-AzureRmAccount
   ```

4. 設定目前的訂用帳戶內容，或查看目前作用中訂用帳戶：

   ```azurepowershell-interactive
   Select-AzureRmSubscription -SubscriptionName "MyAzureSubName" 
   Get-AzureRmContext
   ```

## <a name="provision-resources"></a>佈建資源

登入 Azure 後請發出下列命令，以佈建服務匯流排資源。 請務必將所有預留位置取代為適當的值。

```azurepowershell-interactive
# Create a resource group 
New-AzureRmResourceGroup –Name my-resourcegroup –Location westus2

# Create a Messaging namespace
New-AzureRmServiceBusNamespace -ResourceGroupName my-resourcegroup -NamespaceName namespace-name -Location westus2

# Create a queue 
New-AzureRmServiceBusQueue -ResourceGroupName my-resourcegroup -NamespaceName namespace-name -Name queue-name -EnablePartitioning $False

# Get primary connection string (required in next step)
Get-AzureRmServiceBusKey -ResourceGroupName my-resourcegroup -Namespace namespace-name -Name RootManageSharedAccessKey
```

在 `Get-AzureRmServiceBusKey` Cmdlet 執行之後，請將連接字串與您選取的佇列名稱複製並貼到暫存位置，例如 [記事本]。 您在下一個步驟將會用到這些資料。

## <a name="send-and-receive-messages"></a>傳送及接收訊息

建立命名空間和佇列，且您已擁有必要的認證之後，您即可傳送和接收訊息。 您可以在[此 GitHub 範例資料夾](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/BasicSendReceiveQuickStart)中查看程式碼。

若要執行程式碼，請執行下列動作：

1. 發出下列命令，以複製[服務匯流排 GitHub 存放庫](https://github.com/Azure/azure-service-bus/)︰

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. 開啟 PowerShell 提示字元。

3. 瀏覽至範例資料夾 `azure-service-bus\samples\DotNet\GettingStarted\BasicSendReceiveQuickStart\BasicSendReceiveQuickStart`。

4. 如果您尚未使用下列 PowerShell Cmdlet 取得連接字串，請執行此動作。 請務必將 `my-resourcegroup` 和 `namespace-name` 取代為您自己的值： 

   ```azurepowershell-interactive
   Get-AzureRmServiceBusKey -ResourceGroupName my-resourcegroup -Namespace namespace-name -Name RootManageSharedAccessKey
   ```
5.  在 PowerShell 提示字元中，輸入下列命令：

   ```shell
   dotnet build
   ```
6.  瀏覽到 `\bin\Debug\netcoreapp2.0` 資料夾。
7.  輸入下列命令以執行程式。 請務必將 `myConnectionString` 取代為您先前取得的值，並將 `myQueueName` 取代為您已建立的佇列名稱：

   ```shell
   dotnet BasicSendReceiveQuickStart.dll -ConnectionString "myConnectionString" -QueueName "myQueueName"
   ``` 
8. 觀察 10 個傳送至佇列，和後續接收自佇列的訊息：

   ![程式輸出](./media/service-bus-quickstart-powershell/dotnet.png)

## <a name="clean-up-resources"></a>清除資源

執行下列命令以移除資源群組、命名空間和所有相關資源：

```powershell-interactive
Remove-AzureRmResourceGroup -Name my-resourcegroup
```

## <a name="understand-the-sample-code"></a>了解範例程式碼

本節將詳細說明範例程式碼的功能。 

### <a name="get-connection-string-and-queue"></a>取得連接字串和佇列

連接字串和佇列名稱會傳至 `Main()` 方法作為命令列引數。 `Main()` 會宣告兩個字串變數來保存這些值：

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string QueueName = "";

    for (int i = 0; i < args.Length; i++)
    {
        var p = new Program();
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i+1]}");
            ServiceBusConnectionString = args[i + 1]; 
        }
        else if(args[i] == "-QueueName")
        {
            Console.WriteLine($"QueueName: {args[i+1]}");
            QueueName = args[i + 1];
        }                
    }

    if (ServiceBusConnectionString != "" && QueueName != "")
        MainAsync(ServiceBusConnectionString, QueueName).GetAwaiter().GetResult();
    else
    {
        Console.WriteLine("Specify -Connectionstring and -QueueName to execute the example.");
        Console.ReadKey();
    }                            
}
```
 
接著，`Main()` 方法會啟動非同步訊息迴圈 `MainAsync()`。

### <a name="message-loop"></a>訊息迴圈

`MainAsync()` 方法會使用命令列引數建立佇列用戶端、呼叫名為 `RegisterOnMessageHandlerAndReceiveMessages()` 的接收訊息處理常式，並傳送訊息組：

```csharp
static async Task MainAsync(string ServiceBusConnectionString, string QueueName)
{
    const int numberOfMessages = 10;
    queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

    Console.WriteLine("======================================================");
    Console.WriteLine("Press any key to exit after receiving all the messages.");
    Console.WriteLine("======================================================");

    // Register QueueClient's MessageHandler and receive messages in a loop
    RegisterOnMessageHandlerAndReceiveMessages();

    // Send Messages
    await SendMessagesAsync(numberOfMessages);

    Console.ReadKey();

    await queueClient.CloseAsync();
}
```

`RegisterOnMessageHandlerAndReceiveMessages()` 方法會設定幾個訊息處理常式選項，然後呼叫佇列用戶端的 `RegisterMessageHandler()` 方法，繼而開始接收：

```csharp
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
```

### <a name="send-messages"></a>傳送訊息

訊息建立和傳送作業會在 `SendMessagesAsync()` 方法中執行：

```csharp
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
```

### <a name="process-messages"></a>處理訊息

`ProcessMessagesAsync()` 方法會認可、處理以及完成訊息的接收：

```csharp
static async Task ProcessMessagesAsync(Message message, CancellationToken token)
{
    // Process the message
    Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

    // Complete the message so that it is not received again.
    await queueClient.CompleteAsync(message.SystemProperties.LockToken);
}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 Azure PowerShell 佈建資源，然後從服務匯流排主題及其訂用帳戶傳送和接收訊息。 您已了解如何︰

> [!div class="checklist"]
> * 使用 Azure 入口網站建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 .NET 程式碼新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

如需傳送和接收訊息的範例，請開始使用 [GitHub 上的服務匯流排範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted)。

繼續進行下一個教學課程，以深入了解如何使用服務匯流排的發佈/訂閱功能。

> [!div class="nextstepaction"]
> [使用 PowerShell 和主題/訂用帳戶來更新庫存](service-bus-tutorial-topics-subscriptions-cli.md)

[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[安裝和設定 Azure PowerShell]: /powershell/azure/install-azurerm-ps