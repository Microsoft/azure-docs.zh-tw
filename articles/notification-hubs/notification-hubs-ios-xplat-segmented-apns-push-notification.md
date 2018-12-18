---
title: 使用 Azure 通知中樞將通知推送至特定 iOS 裝置| Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 通知中樞將推播通知傳送至特定的 iOS 裝置。
services: notification-hubs
documentationcenter: ios
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 6ead4169-deff-4947-858c-8c6cf03cc3b2
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 18caf2b1b96052d93737c8a9815e2e6643a52a67
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918057"
---
# <a name="tutorial-push-notifications-to-specific-ios-devices-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至特定 iOS 裝置

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>概觀

本教學課程說明如何使用 Azure 通知中樞將即時新聞通知廣播至 iOS 應用程式。 完成時，您將能夠註冊自己所感興趣的即時新聞類別，並僅接收那些類別的推播通知。 此情況為適用於許多應用程式的常見模式，其中必須將通知傳送給先前宣告對該通知感興趣的使用者群組，例如 RSS 閱讀程式、供樂迷使用的應用程式等等。

在通知中樞內建立註冊時，您可以透過包含一或多個 *tags* 來啟用廣播案例。 當通知傳送至標記時，已註冊該標記的裝置都會收到該通知。 由於標籤只是簡單的字串而已，您無需預先佈建標籤。 如需標記的詳細資訊，請參閱[通知中樞路由與標記運算式](notification-hubs-tags-segment-push-message.md)。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 在應用程式中新增類別選項
> * 傳送加註標記的通知
> * 從裝置傳送通知
> * 執行應用程式並產生通知

## <a name="prerequisites"></a>必要條件

這個主題會以您在[教學課程：使用 Azure 通知中樞將通知推播至 iOS 應用程式][get-started]中所建立的應用程式為基礎。 在開始進行本教學課程之前，您必須先完成[教學課程：使用 Azure 通知中樞將通知推播至 iOS 應用程式][get-started]。

## <a name="add-category-selection-to-the-app"></a>在應用程式中新增類別選項

第一個步驟是在您現有的腳本上新增 UI 元素，以便使用者選取要註冊的類別。 使用者所選取的類別會儲存在裝置上。 啟動應用程式時，您的通知中心內會建立以所選取類別作為標籤的裝置註冊。

1. 在您的 **MainStoryboard_iPhone.storyboard** 中，從物件程式庫新增下列元件：

    * 具有「即時新聞」文字的標籤，
    * 具有「世界」、「政治」、「商業」、「技術」、「科學」、「體育」等類別文字的標籤，
    * 六個參數 (一個類別一個)，預設會將每個參數 [狀態] 設為 [關閉]。
    * 一個標示為「訂閱」的按鈕

    您的腳本應如下所示：

    ![Xcode 介面產生器][3]

2. 在輔助編輯器中，建立所有參數的出口，並將其命名為 "WorldSwitch"、"PoliticsSwitch"、"BusinessSwitch"、"TechnologySwitch"、"ScienceSwitch"、"SportsSwitch"
3. 為按鈕建立名為 **subscribe** \(訂閱\) 的動作。 您的 ViewController.h 應該包含下列程式碼：

    ```objc
    @property (weak, nonatomic) IBOutlet UISwitch *WorldSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *PoliticsSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *BusinessSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *TechnologySwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *ScienceSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *SportsSwitch;

    - (IBAction)subscribe:(id)sender;
    ```

4. 建立稱為 `Notifications` 的新 **Cocoa Touch 類別**。 在 Notifications.h 的介面區段中複製下列程式碼：

    ```objc
    @property NSData* deviceToken;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName;

    - (void)storeCategoriesAndSubscribeWithCategories:(NSArray*)categories
                completion:(void (^)(NSError* error))completion;

    - (NSSet*)retrieveCategories;

    - (void)subscribeWithCategories:(NSSet*)categories completion:(void (^)(NSError *))completion;
    ```

5. 在 Notifications.m 中新增下列 import 指示詞：

    ```objc
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    ```

6. 複製 Notifications.m 檔案實作區段中的下列程式碼：

    ```objc
    SBNotificationHub* hub;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName{

        hub = [[SBNotificationHub alloc] initWithConnectionString:listenConnectionString
                                    notificationHubPath:hubName];

        return self;
    }

    - (void)storeCategoriesAndSubscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];

        [self subscribeWithCategories:categories completion:completion];
    }

    - (NSSet*)retrieveCategories {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        NSArray* categories = [defaults stringArrayForKey:@"BreakingNewsCategories"];

        if (!categories) return [[NSSet alloc] init];
        return [[NSSet alloc] initWithArray:categories];
    }

    - (void)subscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion
    {
        //[hub registerNativeWithDeviceToken:self.deviceToken tags:categories completion: completion];

        NSString* templateBodyAPNS = @"{\"aps\":{\"alert\":\"$(messageParam)\"}}";

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"simpleAPNSTemplate" 
            jsonBodyTemplate:templateBodyAPNS expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    此類別會使用本機儲存體來儲存和擷取此裝置接收到的新聞類別。 此外，它也包含使用 [範本](notification-hubs-templates-cross-platform-push-messages.md) 註冊來註冊這些類別的方法。

7. 在 AppDelegate.h 檔案中，新增 Notifications.h 的匯入陳述式，並新增 Notifications 類別執行個體的屬性：

    ```objc
    #import "Notifications.h"

    @property (nonatomic) Notifications* notifications;
    ```

8. 在 AppDelegate.m 的 **didFinishLaunchingWithOptions** 方法中，於方法的開頭處加入程式碼來初始化通知執行個體。  

    `HUBNAME` 與 `HUBLISTENACCESS` (定義於 hubinfo.h 中) 的 `<hub name>` 及 `<connection string with listen access>` 預留位置，應已用稍早取得之 *DefaultListenSharedAccessSignature* 的通知中樞名稱與連接字串所取代。

    ```objc
    self.notifications = [[Notifications alloc] initWithConnectionString:HUBLISTENACCESS HubName:HUBNAME];
    ```

    > [!NOTE]
    > 因為隨用戶端應用程式散佈的憑證通常不安全，您應只將接聽存取權的金鑰隨用戶端應用程式散佈。 您的應用程式可透過接聽存取權來註冊通知，但無法修改現有的註冊或無法傳送通知。 在安全的後端服務中，會使用完整存取金鑰來傳送通知和變更現有的註冊。

9. 在 AppDelegate.m 的 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中，使用下列程式碼來取代方法中的程式碼，以將裝置權杖傳遞給 notifications 類別。 Notifications 類別會為通知執行與類別之間的註冊。 如果使用者變更類別選取項目，則可以呼叫 `subscribeWithCategories` 方法以回應 [subscribe] \(訂閱\) 按鈕來更新它們。

    > [!NOTE]
    > 由於 Apple 推播通知服務 (APNS) 所指派的裝置權杖可能隨時會變更，因此您應經常註冊通知以避免通知失敗。 此範例會在應用程式每次啟動時註冊通知。 若是經常執行 (一天多次) 的應用程式，如果距離上次註冊的時間不到一天，則您可能可以略過註冊以保留頻寬。

    ```objc
    self.notifications.deviceToken = deviceToken;

    // Retrieves the categories from local storage and requests a registration for these categories
    // each time the app starts and performs a registration.

    NSSet* categories = [self.notifications retrieveCategories];
    [self.notifications subscribeWithCategories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

    此時，**didRegisterForRemoteNotificationsWithDeviceToken** 方法中不應有其他程式碼。

10. 完成[開始使用通知中樞][get-started]教學課程時，下列方法應該已出現在 AppDelegate.m 中。 否則，請予以新增。

    ```objc
    -(void)MessageBox:(NSString *)title message:(NSString *)messageText
    {

        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self
            cancelButtonTitle:@"OK" otherButtonTitles: nil];
        [alert show];
    }

    * (void)application:(UIApplication *)application didReceiveRemoteNotification:
       (NSDictionary *)userInfo {
       NSLog(@"%@", userInfo);
       [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
     }
    ```

    此方法會顯示簡易 **UIAlert**，以處理應用程式執行時接收到的通知。

11. 在 ViewController.m 中，新增 AppDelegate.h 的匯入陳述式，並將下列程式碼複製到 XCode 所產生的 **subscribe** 方法中。 此程式碼會更新通知註冊，以使用使用者在使用者介面中所選擇的新類別標記。

    ```objc
    #import "Notifications.h"

    NSMutableArray* categories = [[NSMutableArray alloc] init];

    if (self.WorldSwitch.isOn) [categories addObject:@"World"];
    if (self.PoliticsSwitch.isOn) [categories addObject:@"Politics"];
    if (self.BusinessSwitch.isOn) [categories addObject:@"Business"];
    if (self.TechnologySwitch.isOn) [categories addObject:@"Technology"];
    if (self.ScienceSwitch.isOn) [categories addObject:@"Science"];
    if (self.SportsSwitch.isOn) [categories addObject:@"Sports"];

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    [notifications storeCategoriesAndSubscribeWithCategories:categories completion: ^(NSError* error) {
        if (!error) {
            [(AppDelegate*)[[UIApplication sharedApplication]delegate] MessageBox:@"Notification" message:@"Subscribed!"];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

    此方法會建立類別的 **NSMutableArray**，並使用 **Notifications** 類別在本機儲存體中儲存清單，並在您的通知中樞註冊對應標籤。 變更類別時，系統會使用新類別重新建立註冊。

3. 在 ViewController.m 中，於 **viewDidLoad** 方法中新增下列程式碼，以根據先前儲存的類別來設定使用者介面。

    ```objc
    // This updates the UI on startup based on the status of previously saved categories.

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    NSSet* categories = [notifications retrieveCategories];

    if ([categories containsObject:@"World"]) self.WorldSwitch.on = true;
    if ([categories containsObject:@"Politics"]) self.PoliticsSwitch.on = true;
    if ([categories containsObject:@"Business"]) self.BusinessSwitch.on = true;
    if ([categories containsObject:@"Technology"]) self.TechnologySwitch.on = true;
    if ([categories containsObject:@"Science"]) self.ScienceSwitch.on = true;
    if ([categories containsObject:@"Sports"]) self.SportsSwitch.on = true;
    ```

應用程式現在可以在裝置本機儲存體中儲存一組類別，以用來在每次應用程式啟動時於通知中樞註冊。 使用者可以在執行階段變更選取的類別，然後按一下 [訂閱]  方法來更新裝置的註冊。 接下來，您會更新應用程式，以直接在應用程式本身傳送即時新聞通知。

## <a name="optional-send-tagged-notifications"></a>(選擇性) 傳送加註標記的通知

如果您無法存取 Visual Studio，可以跳到下一節，並從應用程式本身傳送通知。 您也可以使用通知中樞的 [偵錯] 索引標籤，從 [Azure 入口網站] 傳送正確的範本通知。 

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="optional-send-notifications-from-the-device"></a>(選擇性) 從裝置傳送通知

通知通常會由後端服務傳送，但您可直接從應用程式傳送即時新聞通知。 若要這樣做，您須更新您在[開始使用通知中樞][get-started]教學課程中所定義的 `SendNotificationRESTAPI` 方法。

1. 在 `ViewController.m` 中，以下列方式更新 `SendNotificationRESTAPI` 方法，使其接受參數作為類別標記，並會傳送正確的[範本](notification-hubs-templates-cross-platform-push-messages.md)通知。

    ```objc
    - (void)SendNotificationRESTAPI:(NSString*)categoryTag
    {
        NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
                                    defaultSessionConfiguration] delegate:nil delegateQueue:nil];

        NSString *json;

        // Construct the messages REST endpoint
        NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
                                            HUBNAME, API_VERSION]];

        // Generated the token to be used in the authorization header.
        NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

        //Create the request to add the template notification message to the hub
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        [request setHTTPMethod:@"POST"];

        // Add the category as a tag
        [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

        // Template notification
        json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\"}",
                categoryTag, self.notificationMessage.text];

        // Signify template notification format
        [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

        // JSON Content-Type
        [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

        //Authenticate the notification message POST request with the SaS token
        [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

        //Add the notification message body
        [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

        // Send the REST request
        NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                    completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
            {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (error || httpResponse.statusCode != 200)
                {
                    NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
                }
                if (data != NULL)
                {
                    //xmlParser = [[NSXMLParser alloc] initWithData:data];
                    //[xmlParser setDelegate:self];
                    //[xmlParser parse];
                }
            }];

        [dataTask resume];
    }
    ```

2. 在 `ViewController.m` 中，更新 [Send Notification] \(傳送通知\) 動作，如下列程式碼所示。 這會使它個別使用每個標記來傳送通知，並傳送至多個平台。

    ```objc
    - (IBAction)SendNotificationMessage:(id)sender
    {
        self.sendResults.text = @"";

        NSArray* categories = [NSArray arrayWithObjects: @"World", @"Politics", @"Business",
                                @"Technology", @"Science", @"Sports", nil];

        // Lets send the message as breaking news for each category to WNS, GCM, and APNS
        // using a template.
        for(NSString* category in categories)
        {
            [self SendNotificationRESTAPI:category];
        }
    }
    ```

3. 重新建置專案，並確定您沒有建置錯誤。

## <a name="run-the-app-and-generate-notifications"></a>執行應用程式並產生通知

1. 按 [執行] 按鈕，以建置專案並啟動應用程式。 選取要訂閱的一些即時新聞選項，然後按 [訂閱]  按鈕。 您應該會看到一個對話方塊，表示已訂閱通知。

    ![iOS 上的範例通知][1]

    當您選擇 [訂閱] 時，應用程式會將選取的類別轉換成標籤，並在通知中心內為選取的標籤要求新裝置註冊。

2. 輸入要以即時新聞形式傳送的訊息，然後按下 [傳送通知]  按鈕。 或者，執行.NET 主控台應用程式來產生通知。

    ![在 iOS 中變更通知喜好設定][2]

3. 每個訂閱即時新聞的裝置都會收到您剛剛傳送的即時新聞通知。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將廣播通知傳送至註冊相關類別的特定 iOS 裝置。 若要了解如何推送當地語系化的通知，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[推送當地語系化的通知](notification-hubs-ios-xplat-localized-apns-push-notification.md)


<!-- Images. -->
[1]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-subscribed.png
[2]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios1.png
[3]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios2.png








<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: http://msdn.microsoft.com/library/jj927168.aspx
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-ios-xplat-localized-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-ios-notify-users.md
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/dn530749.aspx
[Notification Hubs How-To for iOS]: http://msdn.microsoft.com/library/jj927168.aspx
[get-started]: notification-hubs-ios-apple-push-notification-apns-get-started.md
[Azure 入口網站]: https://portal.azure.com
