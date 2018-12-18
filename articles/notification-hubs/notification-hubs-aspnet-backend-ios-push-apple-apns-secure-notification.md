---
title: Azure 通知中心安全推播
description: 了解如何從 Azure 將安全的推播通知傳送至 iOS 應用程式。 程式碼範例是以 Objective-C 及 C# 撰寫。
documentationcenter: ios
author: dimazaid
manager: kpiteira
editor: spelluru
services: notification-hubs
ms.assetid: 17d42b0a-2c80-4e35-a1ed-ed510d19f4b4
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 04/25/2018
ms.author: dimazaid
ms.openlocfilehash: d3ba967a164a35af5bf66f7e74d5f95b5dc2a37f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38308566"
---
# <a name="azure-notification-hubs-secure-push"></a>Azure 通知中心安全推播
> [!div class="op_single_selector"]
> * [Windows Universal](notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md)
> * [iOS](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md)
> * [Android](notification-hubs-aspnet-backend-android-secure-google-gcm-push-notification.md)
> 
> 

## <a name="overview"></a>概觀
Microsoft Azure 中的推播通知支援可讓您存取易於使用、多重平台的大規模推播基礎結構，因而可大幅簡化消費者和企業應用程式在行動平台上的推播通知實作。

基於法規或安全性限制，應用程式有時會想要在通知中加入無法透過標準推播通知基礎結構傳送的內容。 本教學課程說明如何透過用戶端裝置和應用程式後端之間的安全、已驗證連線來傳送敏感資訊，以達到相同體驗。

概括而言，流程如下所示：

1. 應用程式後端：
   * 在後端資料庫中儲存安全裝載。
   * 將此通知的識別碼傳送至裝置 (不會傳送安全資訊)。
2. 收到通知時，裝置上的應用程式會執行下列動作：
   * 裝置會連絡後端並要求安全裝載。
   * 應用程式會以通知的形式在裝置上顯示裝載。

請務必注意在上述流程 (與本教學課程) 中，我們假設使用者登入後，裝置會將驗證權杖儲存在本機儲存體中。 由於裝置可使用此權杖擷取通知的安全裝載，因此可保證順暢的體驗。 如果您的應用程式沒有將驗證權杖儲存在裝置上，或如果這些權杖可能會過期，裝置應用程式應在收到通知時顯示一般通知，以提示使用者啟動應用程式。 應用程式會接著驗證使用者，並顯示通知裝載。

本安全推播教學課程說明如何以安全的方式傳送推播通知。 本教學課程會以 [通知使用者](notification-hubs-aspnet-backend-ios-apple-apns-notification.md) 教學課程為基礎，因此您應先完成該教學課程中的步驟。

> [!NOTE]
> 本教學課程假設您已建立並設定通知中樞，如 [開始使用通知中樞 (iOS)](notification-hubs-ios-apple-push-notification-apns-get-started.md)中所述。
> 
> 

[!INCLUDE [notification-hubs-aspnet-backend-securepush](../../includes/notification-hubs-aspnet-backend-securepush.md)]

## <a name="modify-the-ios-project"></a>修改 iOS 專案
現在，您已修改應用程式後端將只傳送通知的*識別碼*，接著必須變更 iOS 應用程式來處理該通知，並回呼後端以擷取要顯示的安全訊息。

若要達到此目標，我們必須撰寫可從應用程式後端擷取安全內容的邏輯。

1. 在 **AppDelegate.m**中，請確定應用程式已註冊無訊息通知，以便處理從後端傳送出來的通知識別碼。 在 didFinishLaunchingWithOptions 中新增 **UIRemoteNotificationTypeNewsstandContentAvailability** 選項：
   
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeNewsstandContentAvailability];
2. 在 **AppDelegate.m** 的開頭處，新增包含下列宣告的實作區段：
   
        @interface AppDelegate ()
        - (void) retrieveSecurePayloadWithId:(int)payloadId completion: (void(^)(NSString*, NSError*)) completion;
        @end
3. 然後在實作區段中新增下列程式碼，並以先前為後端取得的端點取代預留位置 `{back-end endpoint}` ：

```
        NSString *const GetNotificationEndpoint = @"{back-end endpoint}/api/notifications";

        - (void) retrieveSecurePayloadWithId:(int)payloadId completion: (void(^)(NSString*, NSError*)) completion;
        {
            // check if authenticated
            ANHViewController* rvc = (ANHViewController*) self.window.rootViewController;
            NSString* authenticationHeader = rvc.registerClient.authenticationHeader;
            if (!authenticationHeader) return;


            NSURLSession* session = [NSURLSession
                                     sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]
                                     delegate:nil
                                     delegateQueue:nil];


            NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/%d", GetNotificationEndpoint, payloadId]];
            NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
            [request setHTTPMethod:@"GET"];
            NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", authenticationHeader];
            [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

            NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (!error && httpResponse.statusCode == 200)
                {
                    NSLog(@"Received secure payload: %@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);

                    NSMutableDictionary *json = [NSJSONSerialization JSONObjectWithData:data options: NSJSONReadingMutableContainers error: &error];

                    completion([json objectForKey:@"Payload"], nil);
                }
                else
                {
                    NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                    if (error)
                        completion(nil, error);
                    else {
                        completion(nil, [NSError errorWithDomain:@"APICall" code:httpResponse.statusCode userInfo:nil]);
                    }
                }
            }];
            [dataTask resume];
        }
```

    This method calls your app back-end to retrieve the notification content using the credentials stored in the shared preferences.

1. 現在，我們必須處理內送通知，並使用上述方法擷取要顯示的內容。 首先，我們必須啟用您的 iOS 應用程式，可在接收推播通知時於背景中執行。 在 **XCode** 中，在左側面板中選取您的應用程式專案，然後在中央窗格的 [目標] 區段中，按一下您的主要應用程式目標。
2. 接著按一下中央窗格頂端的 [功能] 索引標籤，並核取 [遠端通知] 核取方塊。
   
    ![][IOS1]
3. 在 **AppDelegate.m** 中，新增下列可處理推播通知的方法：
   
        -(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
        {
            NSLog(@"%@", userInfo);
   
            [self retrieveSecurePayloadWithId:[[userInfo objectForKey:@"secureId"] intValue] completion:^(NSString * payload, NSError *error) {
                if (!error) {
                    // show local notification
                    UILocalNotification* localNotification = [[UILocalNotification alloc] init];
                    localNotification.fireDate = [NSDate dateWithTimeIntervalSinceNow:0];
                    localNotification.alertBody = payload;
                    localNotification.timeZone = [NSTimeZone defaultTimeZone];
                    [[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
   
                    completionHandler(UIBackgroundFetchResultNewData);
                } else {
                    completionHandler(UIBackgroundFetchResultFailed);
                }
            }];
   
        }
   
    請注意，比較理想的案例是處理遺失驗證標頭屬性或遭到後端拒絕的情況。 這些案例的特定處理絕大部分會依您的目標使用者經驗而定。 其中一個選項就是透過一般提示顯示通知，方便使用者進行驗證並擷取實際通知。

## <a name="run-the-application"></a>執行應用程式
若要執行應用程式，請執行下列動作：

1. 在 XCode 中，在實體 iOS 裝置上執行應用程式 (推播通知無法在模擬器中運作)。
2. 在 iOS 應用程式 UI 中，輸入使用者名稱和密碼。 這些可以是任何字串，但必須是相同值。
3. 在 iOS 應用程式 UI 中，按一下 [登入] 。 然後按一下 [傳送推播] 。 您應該會在您的通知中心內看見安全通知。

[IOS1]: ./media/notification-hubs-aspnet-backend-ios-secure-push/secure-push-ios-1.png
