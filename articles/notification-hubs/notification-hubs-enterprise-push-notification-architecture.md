---
title: 通知中樞 - 企業推送架構
description: 在企業環境中使用 Azure 通知中樞的指引
services: notification-hubs
documentationcenter: ''
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 903023e9-9347-442a-924b-663af85e05c6
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 962bc996a86340bb10a28b90ef6340a98c5d9275
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430601"
---
# <a name="enterprise-push-architectural-guidance"></a>企業推送架構指引
當代的企業正逐漸朝著為使用者 (外部) 或員工 (內部) 建立行動應用程式的方向邁進。 他們擁有現成的後端系統 (無論是大型主機還是某些 LoB 應用程式)，而這些系統必須整合到行動應用程式架構中。 本指南會討論如何以最佳方式進行整合，並針對常見案例建議適用的可行方案。

常見的需求是當後端系統發生使用者感興趣的事件時，透過行動應用程式將推播通知傳送給使用者。 例如，在 iPhone 上安裝某家銀行之銀行應用程式的銀行客戶，想要在帳戶扣款金額超過一定數目時獲得通知，或是在 Windows Phone 上安裝預算核准應用程式的財務部門員工，想要在收到核准要求時獲得通知的內部網路案例。

銀行帳戶或核准處理很可能會在後端系統中完成，而這必須對使用者發出推送。 可能會有多個這樣的後端系統，而這些系統都必須建置相同的邏輯，以在事件觸發通知時進行推送。 其複雜度在於以單一推送系統整合數個後端系統，其中使用者可能會訂閱不同的通知，甚至擁有多個行動應用程式。 以內部網路行動應用程式為例，行動應用程式可能需要接收來自多個上述後端系統所傳送的通知。 由於後端系統不知道 (或不需要知道) 推送的語意/技術，因此常用的傳統方案是導入一個元件，該元件會輪詢後端系統是否有任何使用者感興趣的事件，並負責將推送訊息傳送給用戶端。

另一個較好的解決方案，是使用「Azure 服務匯流排 - 主題/訂用帳戶」模型，它可以降低複雜度，同時讓解決方案具調整性。

以下是方案的一般架構 (我們已利用多個行動應用程式將其一般化，不過該架構亦適用於只有一個行動應用程式的案例)。

## <a name="architecture"></a>架構
![][1]

本架構圖中的關鍵是提供主題/訂用帳戶程式撰寫模型的 Azure 服務匯流排 (如需詳細資料，請參閱[服務匯流排發佈/訂用帳戶程式撰寫])。 本案例中的接收器是行動後端 (通常是 [Azure 行動服務]，它會將推送發送給行動裝置)，它不會直接接收來自後端系統的訊息，反之，會透過由 Azure 服務匯流排所提供的中繼抽象層，讓行動後端得以接收來自一或多個後端系統的訊息。 您需要為每個後端系統建立服務匯流排主題 (例如帳戶、HR、財務)，它們基本上是使用者感興趣的「主題」，能讓訊息以推播通知的形式傳送。 後端系統會將訊息傳送到這些主題。 藉由建立服務匯流排訂閱，行動後端能訂閱一或多個這類型的主題。 它能讓行動後端接收來自對應後端系統的通知。 行動後端會持續接聽與其訂閱相關的訊息，待訊息抵達後，它會立即轉向並以通知形式將訊息傳送到通知匯流排。 通知中樞接著會將訊息傳遞給行動應用程式。 以下是重要元件的清單：

1. 後端系統 (LoB/舊版系統)
   * 建立服務匯流排主題
   * 傳送訊息
1. 行動後端
   * 建立服務訂閱
   * 接收訊息 (來自後端系統)
   * 將通知傳送給用戶端 (透過 Azure 通知中樞)
1. 行動應用程式
   * 接收及顯示通知

### <a name="benefits"></a>優點：
1. 接收器 (行動應用程式/透過通知中樞傳送的服務) 與傳送器 (後端系統) 的解離可讓您在變更少量架構的情況下整合額外的後端系統。
1. 它也能促成讓多個行動應用程式接收來自一或多個後端系統事件的案例。  

## <a name="sample"></a>範例：
### <a name="prerequisites"></a>必要條件
完成下列教學課程以熟悉概念，以及常用的建立和設定步驟：

1. [服務匯流排發佈/訂用帳戶程式撰寫]：此教學課程詳細說明如何使用服務匯流排主題/訂用帳戶、如何建立命名空間來包含主題/訂用帳戶、如何傳送和接收來自它們的訊息。
1. [通知中樞 - Windows Universal 教學課程]：此教學課程說明如何設定 Windows 市集應用程式，以及使用通知中樞來註冊和接收通知。

### <a name="sample-code"></a>範例程式碼
如需完整範例程式碼，請參閱 [通知中樞範例]。 其可劃分為三個元件：

1. **EnterprisePushBackendSystem**
   
    a. 本專案使用 *WindowsAzure.ServiceBus* NuGet 套件，並以[服務匯流排發佈/訂用帳戶程式撰寫]為基礎。
   
    b. 此應用程式為簡易的 C# 主控台應用程式，可用來模擬 LoB 系統，讓訊息得以傳遞到行動應用程式。
   
        static void Main(string[] args)
        {
            string connectionString =
                CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
   
            // Create the topic
            CreateTopic(connectionString);
   
            // Send message
            SendMessage(connectionString);
        }
   
    c. `CreateTopic` 是用來建立服務匯流排主題。
   
        public static void CreateTopic(string connectionString)
        {
            // Create the topic if it does not exist already
   
            var namespaceManager =
                NamespaceManager.CreateFromConnectionString(connectionString);
   
            if (!namespaceManager.TopicExists(sampleTopic))
            {
                namespaceManager.CreateTopic(sampleTopic);
            }
        }
   
    d. `SendMessage` 可用來將訊息傳送到此服務匯流排主題。 基於範例目的，此程式碼只會定期傳送一組隨機訊息給主題。 在一般情況下，會有一個後端系統在發生事件時傳送訊息。
   
        public static void SendMessage(string connectionString)
        {
            TopicClient client =
                TopicClient.CreateFromConnectionString(connectionString, sampleTopic);
   
            // Sends random messages every 10 seconds to the topic
            string[] messages =
            {
                "Employee Id '{0}' has joined.",
                "Employee Id '{0}' has left.",
                "Employee Id '{0}' has switched to a different team."
            };
   
            while (true)
            {
                Random rnd = new Random();
                string employeeId = rnd.Next(10000, 99999).ToString();
                string notification = String.Format(messages[rnd.Next(0,messages.Length)], employeeId);
   
                // Send Notification
                BrokeredMessage message = new BrokeredMessage(notification);
                client.Send(message);
   
                Console.WriteLine("{0} Message sent - '{1}'", DateTime.Now, notification);
   
                System.Threading.Thread.Sleep(new TimeSpan(0, 0, 10));
            }
        }
1. **ReceiveAndSendNotification**
   
    a. 本專案使用 *WindowsAzure.ServiceBus* 和 *Microsoft.Web.WebJobs.Publish* NuGet 套件，並以[服務匯流排發佈/訂用帳戶程式撰寫]為基礎。
   
    b. 下列主控台應用程式會以 [Azure WebJob] 的形式執行，因為它必須連續執行以接聽來自 LoB/後端系統的訊息。 此應用程式是您行動後端的一部分。
   
        static void Main(string[] args)
        {
            string connectionString =
                     CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
   
            // Create the subscription that receives messages
            CreateSubscription(connectionString);
   
            // Receive message
            ReceiveMessageAndSendNotification(connectionString);
        }
   
    c. `CreateSubscription` 是用來為後端系統傳送訊息的主題，建立服務匯流排訂用帳戶。 由於商務案例不盡相同，此元件會針對相對應的主題建立一或多個訂用帳戶 (例如，有些會接收來自 HR 系統的訊息、有些會接收來自財務系統的訊息，依此類推)。
   
        static void CreateSubscription(string connectionString)
        {
            // Create the subscription if it does not exist already
            var namespaceManager =
                NamespaceManager.CreateFromConnectionString(connectionString);
   
            if (!namespaceManager.SubscriptionExists(sampleTopic, sampleSubscription))
            {
                namespaceManager.CreateSubscription(sampleTopic, sampleSubscription);
            }
        }
   
    d. 使用 ReceiveMessageAndSendNotification，以使用訂用帳戶來讀取主題中的訊息，如果讀取成功，則會使用 Azure 通知中樞製作通知 (在範例案例中，即 Windows 原生快顯通知) 以傳送給行動應用程式。
   
        static void ReceiveMessageAndSendNotification(string connectionString)
        {
            // Initialize the Notification Hub
            string hubConnectionString = CloudConfigurationManager.GetSetting
                    ("Microsoft.NotificationHub.ConnectionString");
            hub = NotificationHubClient.CreateClientFromConnectionString
                    (hubConnectionString, "enterprisepushservicehub");
   
            SubscriptionClient Client =
                SubscriptionClient.CreateFromConnectionString
                        (connectionString, sampleTopic, sampleSubscription);
   
            Client.Receive();
   
            // Continuously process messages received from the subscription
            while (true)
            {
                BrokeredMessage message = Client.Receive();
                var toastMessage = @"<toast><visual><binding template=""ToastText01""><text id=""1"">{messagepayload}</text></binding></visual></toast>";
   
                if (message != null)
                {
                    try
                    {
                        Console.WriteLine(message.MessageId);
                        Console.WriteLine(message.SequenceNumber);
                        string messageBody = message.GetBody<string>();
                        Console.WriteLine("Body: " + messageBody + "\n");
   
                        toastMessage = toastMessage.Replace("{messagepayload}", messageBody);
                        SendNotificationAsync(toastMessage);
   
                        // Remove message from subscription
                        message.Complete();
                    }
                    catch (Exception)
                    {
                        // Indicate a problem, unlock message in subscription
                        message.Abandon();
                    }
                }
            }
        }
        static async void SendNotificationAsync(string message)
        {
            await hub.SendWindowsNativeNotificationAsync(message);
        }
   
    e. 若要將此應用程式發佈為 **WebJob**，請以滑鼠右鍵按一下 Visual Studio 中的解決方案，然後選取 [發佈為 WebJob]。
   
    ![][2]
   
    f. 選取發佈設定檔並建立要裝載此 WebJob 的新 Azure 網站 (如果它不存在的話)。備妥網站後，請予以 [發佈]。
   
    ![][3]
   
    g. 將作業設定為 [連續執行]，如此一來，當您登入 [Azure 入口網站]時，應能看見與以下範例相似的內容：
   
    ![][4]
1. **EnterprisePushMobileApp**
   
    a. 此應用程式為 Windows 市集應用程式，它能接收行動後端中執行的 WebJob 所發出的快顯通知，並加以顯示。 此程式碼是以[通知中樞 - Windows Universal 教學課程]為基礎。  
   
    b. 確認應用程式可接收快顯通知。
   
    c. 確認應用程式啟動時會呼叫以下通知中樞註冊程式碼 (取代 *HubName* 和 *DefaultListenSharedAccessSignature* 之後)：
   
        private async void InitNotificationsAsync()
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
   
            var hub = new NotificationHub("[HubName]", "[DefaultListenSharedAccessSignature]");
            var result = await hub.RegisterNativeAsync(channel.Uri);
   
            // Displays the registration ID so you know it was successful
            if (result.RegistrationId != null)
            {
                var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }

### <a name="running-sample"></a>執行範例：
1. 確認 WebJob 已成功執行，並已排定連續執行。
1. 執行 **EnterprisePushMobileApp，這將會啟動 Windows 市集應用程式。
1. 執行 **EnterprisePushBackendSystem** 主控台應用程式來模擬 LoB 後端並開始傳送訊息，您應該會看見與以下影像類似的快顯通知：
   
    ![][5]
1. 這些訊息最初是傳送至服務匯流排主題，這些主題是受到您 WebJob 中服務匯流排訂用帳戶所監視。 待服務匯流排主題接收到訊息後，它會建立通知並傳送給行動應用程式。 在 [Azure 入口網站]中，當您前往 WebJob 的 [記錄] 連結時，可以瀏覽 WebJob 記錄來確認處理狀態：
   
    ![][6]

<!-- Images -->
[1]: ./media/notification-hubs-enterprise-push-architecture/architecture.png
[2]: ./media/notification-hubs-enterprise-push-architecture/WebJobsContextMenu.png
[3]: ./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png
[4]: ./media/notification-hubs-enterprise-push-architecture/WebJob.png
[5]: ./media/notification-hubs-enterprise-push-architecture/Notifications.png
[6]: ./media/notification-hubs-enterprise-push-architecture/WebJobsLog.png

<!-- Links -->
[通知中樞範例]: https://github.com/Azure/azure-notificationhubs-samples
[Azure 行動服務]: http://azure.microsoft.com/documentation/services/mobile-services/
[Azure Service Bus]: http://azure.microsoft.com/documentation/articles/fundamentals-service-bus-hybrid-solutions/
[服務匯流排發佈/訂用帳戶程式撰寫]: http://azure.microsoft.com/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/
[Azure WebJob]: ../app-service/web-sites-create-web-jobs.md
[通知中樞 - Windows Universal 教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[Azure 入口網站]: https://portal.azure.com/
