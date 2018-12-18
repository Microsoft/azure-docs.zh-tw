---
title: 使用 Azure 通知中樞將當地語系化通知推送至 iOS 裝置| Microsoft Docs
description: 了解如何使用 Azure 通知中樞將當地語系化的通知推送至 iOS 裝置。
services: notification-hubs
documentationcenter: ios
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 484914b5-e081-4a05-a84a-798bbd89d428
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: d19fc4290f32359d3af66d96512f65abb17f5d34
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918618"
---
# <a name="tutorial-push-localized-notifications-to-ios-devices-using-azure-notification-hubs"></a>教學課程：了解如何使用 Azure 通知中樞將當地語系化通知推送至 iOS 裝置 

> [!div class="op_single_selector"]
> * [Windows 市集 C#](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

本教學課程說明如何使用 Azure 通知中樞的[範本](notification-hubs-templates-cross-platform-push-messages.md)功能，廣播已依語言及裝置進行當地語系化的即時新聞通知。 在本教學課程中，您必須以在[使用通知中樞傳送即時新聞]中所建立 iOS 應用程式為基礎。 完成之後，您將可以註冊自己感興趣的類別、指定要以哪種語言接收通知，並僅接收該語言之特定類別的推播通知。

此案例分成兩部分：

* iOS 應用程式允許用戶端裝置指定語言，以及訂閱不同的即時新聞類別；
* 後端會使用 Azure 通知中樞的**標記**和**範本**功能來廣播通知。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 更新應用程式使用者介面
> * 建置 iOS 應用程式
> * 從 .NET 主控台應用程式傳送當地語系化的範本通知
> * 從裝置傳送當地語系化的範本通知

## <a name="overview"></a>概觀

在[使用通知中樞傳送即時新聞]中，您已建置使用**標記**來訂閱不同即時新聞類別通知的應用程式。 但有許多應用程式是以多個市場為目標的，因此需要當地語系化。 這表示通知本身的內容必須進行當地語系化，並傳遞至正確的裝置集。 本教學課程會說明如何使用通知中樞的**範本**功能，輕鬆地傳遞已當地語系化的即時新聞通知。

> [!NOTE]
> 傳送當地語系化通知的其中一個方法，是為每個標記建立多個版本。 例如，若要支援英文、法文和中文，您必須為世界新聞建立三個不同的標記："world_en"、"world_fr" 和 "world_ch"。 接著，您必須將世界新聞的當地語系化版本分別傳送至這三個標記。 在本主題中，您會使用範本來避免使用過多的標籤和傳送過多訊息。

以較高的層級而言，範本可用來指定特定裝置接收通知的方式。 範本可參照您的應用程式後端所傳送的訊息中包含的屬性，藉以指定確切的裝載格式。 在您的案例中，您會傳送地區設定無從驗證且包含所有支援語言的訊息：

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

接著，您會確保裝置會註冊至能參照正確屬性的範本。 例如，想要註冊法文新聞的 iOS 應用程式會使用下列語法進行註冊：

```json
{
    aps:{
        alert: "$(News_French)"
    }
}
```

如需範本的詳細資訊，請參閱[範本](notification-hubs-templates-cross-platform-push-messages.md)一文。

## <a name="prerequisites"></a>必要條件

- 完成[將通知推送至特定的 iOS 裝置](notification-hubs-ios-xplat-segmented-apns-push-notification.md)教學課程並具備該教學課程中的程式碼，因為本教學課程是以該程式碼為基礎。
- Visual Studio 2017 是選擇性的。

## <a name="update-the-app-user-interface"></a>更新應用程式使用者介面

在本節中，您會修改在[使用通知中樞傳送即時新聞]主題中所建立的即時新聞應用程式，以使用範本來傳送當地語系化的即時新聞。

在您的 **MainStoryboard_iPhone.storyboard** 中，新增具有三種語言的分段控制：英文、法文與中文。

![建立 iOS UI 分鏡腳本][13]

接著，請確實在您的 ViewController.h 中新增 IBOutlet，如下圖所示：

![建立參數的輸出][14]

## <a name="build-the-ios-app"></a>建置 iOS 應用程式

1. 在您的 Notification.h 中新增 *retrieveLocale* 方法，然後修改儲存和訂閱方法，如下列程式碼所示：

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;

    - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;

    - (NSSet*) retrieveCategories;

    - (int) retrieveLocale;
    ```
    在您的 Notification.m 中新增地區設定參數，並將其儲存在使用者預設值中，以修改 *storeCategoriesAndSubscribe* 方法：

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
        [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
        [defaults synchronize];

        [self subscribeWithLocale: locale categories:categories completion:completion];
    }
    ````

    然後修改 *subscribe* 方法，以加入地區設定：

    ```objc
    - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
        SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];

        NSString* localeString;
        switch (locale) {
            case 0:
                localeString = @"English";
                break;
            case 1:
                localeString = @"French";
                break;
            case 2:
                localeString = @"Mandarin";
                break;
        }

        NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"localizednewsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    您使用的是 registerTemplateWithDeviceToken 方法，而非 registerNativeWithDeviceToken。 您在註冊範本時必須提供 json 範本，以及範本的名稱 (因為應用程式可能會註冊不同的範本)。 請確實將您的類別註冊為標籤，因為要確保能夠收到這些新聞的通知。

    加入方法，從使用者預設設定擷取地區設定：

    ```objc
    - (int) retrieveLocale {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        int locale = [defaults integerForKey:@"BreakingNewsLocale"];

        return locale < 0?0:locale;
    }
    ```

2. 在已修改 Notifications 類別之後，您必須確保 ViewController 會使用新的 UISegmentControl。 請在 *viewDidLoad* 方法中新增下列程式碼行，以確實顯示目前選取的地區設定：

    ```objc
    self.Locale.selectedSegmentIndex = [notifications retrieveLocale];
    ```

    接著，在 *subscribe* 方法中，將您對 *storeCategoriesAndSubscribe* 的呼叫變更為下列程式碼：

    ```objc
    [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                    @"Subscribed!" delegate:nil cancelButtonTitle:
                                    @"OK" otherButtonTitles:nil, nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

3. 最後，您必須在 AppDelegate.m 中更新 *didRegisterForRemoteNotificationsWithDeviceToken* 方法，以便能在應用程式啟動時正確重新整理您的註冊。 使用下列程式碼，針對通知的 *subscribe* 方法，變更您的方法呼叫：

    ```obj-c
    NSSet* categories = [self.notifications retrieveCategories];
    int locale = [self.notifications retrieveLocale];
    [self.notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

## <a name="optional-send-localized-template-notifications-from-net-console-app"></a>(選擇性) 從.NET 主控台應用程式傳送當地語系化的範本通知

[!INCLUDE [notification-hubs-localized-back-end](../../includes/notification-hubs-localized-back-end.md)]

## <a name="optional-send-localized-template-notifications-from-the-device"></a>(選擇性) 從裝置傳送當地語系化的範本通知

如果您無法存取 Visual Studio，或只想要測試直接從裝置上的應用程式，請傳送當地語系化的範本通知。 您可以對在先前教學課程中所定義的 `SendNotificationRESTAPI` 方法，加入當地語系化的範本參數。

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
    json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\","
            \"News_English\":\"Breaking %@ News in English : %@\","
            \"News_French\":\"Breaking %@ News in French : %@\","
            \"News_Mandarin\":\"Breaking %@ News in Mandarin : %@\","
            categoryTag, self.notificationMessage.text,
            categoryTag, self.notificationMessage.text,  // insert English localized news here
            categoryTag, self.notificationMessage.text,  // insert French localized news here
            categoryTag, self.notificationMessage.text]; // insert Mandarin localized news here

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

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將當地語系化通知傳送至 iOS 裝置。 若要了解如何將通知推送至 iOS 應用程式的特定使用者，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[將通知推送給特定使用者](notification-hubs-aspnet-backend-ios-apple-apns-notification.md)

<!-- Images. -->

[13]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
[14]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png






<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: http://msdn.microsoft.com/library/jj927168.aspx
[使用通知中樞傳送即時新聞]: notification-hubs-ios-xplat-segmented-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-ios
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-ios
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-users-ios
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[JavaScript and HTML]: ../get-started-with-push-js.md

[Windows Developer Preview registration steps for Mobile Services]: ../mobile-services-windows-developer-preview-registration.md
[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for iOS]: http://msdn.microsoft.com/library/jj927168.aspx
