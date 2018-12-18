---
title: 使用 Azure 通知中樞將通知推送至特定使用者 | Microsoft Docs
description: 了解如何使用 Azure 通知中樞將推播通知傳送至特定的使用者。
documentationcenter: ios
author: dimazaid
manager: kpiteira
editor: spelluru
services: notification-hubs
ms.assetid: 1f7d1410-ef93-4c4b-813b-f075eed20082
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 04/13/2018
ms.author: dimazaid
ms.openlocfilehash: 270311af94d0c0551626fc2906cade84e0c60664
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918959"
---
# <a name="tutorial-push-notifications-to-specific-users-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至特定使用者

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

本教學課程將示範如何使用 Azure 通知中心，來將推播通知傳送到特定裝置上的特定應用程式使用者。 ASP.NET WebAPI 後端可用來驗證用戶端並產生通知，如指引主題[從您的應用程式後端註冊](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend)中所示。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立 WebAPI 專案
> * 對 WebAPI 後端驗證用戶端
> * 使用 WebAPI 後端來註冊通知
> * 從 WebAPI 後端傳送通知
> * 發佈新的 WebAPI 後端
> * 修改您的 iOS 應用程式
> * 測試應用程式

## <a name="prerequisites"></a>必要條件

本教學課程假設您已建立並設定通知中樞，如 [開始使用通知中樞 (iOS)](notification-hubs-ios-apple-push-notification-apns-get-started.md)中所述。 本教學課程還是 [安全推播 (iOS)](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md) 教學課程的必要條件。
如果您想要使用 Mobile Apps 作為您的後端服務，請參閱 [開始使用 Mobile Apps 推播](../app-service-mobile/app-service-mobile-ios-get-started-push.md)。

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="modify-your-ios-app"></a>修改您的 iOS 應用程式

1. 開啟您在 [開始使用通知中心 (iOS)](notification-hubs-ios-apple-push-notification-apns-get-started.md) 教學課程中建立的 [單頁] 檢視應用程式。

   > [!NOTE]
   > 本節假設您已使用空白組織名稱來設定您的專案。 否則，您需要在所有類別名稱的前方加上您的組織名稱。

2. 在 **Main.storyboard** 中，從物件程式庫新增螢幕擷取畫面中顯示的元件。

    ![在 Xcode 介面產生器中編輯腳本][1]

   * **使用者名稱**：含有預留位置文字 ( *輸入使用者名稱*) 的 UITextField，位於傳送結果標籤正下方且受到左右邊界限制並位於傳送結果標籤正下方。
   * **密碼**：含有預留位置文字 ( *輸入密碼*) 的 UITextField，位於使用者名稱文字欄位正下方且受到左右邊界限制並位於使用者文字欄位正下方。 勾選 [ **傳回金鑰** ] 底下屬性偵測器中的 [ *安全文字輸入*] 選項。
   * **登入**：密碼文字欄位正下方標記的 UIButton，並取消勾選 [控制項內容] 底下屬性偵測器中的 [啟用] 選項。
   * **WNS**：當中樞中已設定 Windows 通知服務時，用來啟用傳送通知至該服務的標籤與開關。 請參閱 [Windows 入門](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)教學課程。
   * **GCM**：當中樞中已設定 Google Cloud Messaging 時，用來啟用傳送通知至該服務的標籤與開關。 請參閱 [Android 入門](notification-hubs-android-push-notification-google-gcm-get-started.md) 教學課程。
   * **APNS**：啟用傳送通知給 Apple 平台通知服務之功能的標籤與開關。
   * **收件者使用者名稱：** 含有預留位置文字 (*收件者使用者名稱標記*) 的 UITextField，位於 GCM 標籤正下方，且受到左右邊界與 GCM 正下方的限制。

    在 [開始使用通知中心 (iOS)](notification-hubs-ios-apple-push-notification-apns-get-started.md) 教學課程中已經新增一些元件。

3. **Ctrl** 可拖曳檢視中的元件到 **ViewController.h**，並新增這些新的輸出。

    ```objc
    @property (weak, nonatomic) IBOutlet UITextField *UsernameField;
    @property (weak, nonatomic) IBOutlet UITextField *PasswordField;
    @property (weak, nonatomic) IBOutlet UITextField *RecipientField;
    @property (weak, nonatomic) IBOutlet UITextField *NotificationField;

    // Used to enable the buttons on the UI
    @property (weak, nonatomic) IBOutlet UIButton *LogInButton;
    @property (weak, nonatomic) IBOutlet UIButton *SendNotificationButton;

    // Used to enabled sending notifications across platforms
    @property (weak, nonatomic) IBOutlet UISwitch *WNSSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *GCMSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *APNSSwitch;

    - (IBAction)LogInAction:(id)sender;
    ```

4. 在 **ViewController.h** 中，在匯入陳述式後面新增以下的 `#define`。 將 *<輸入您的後端端點\>* 預留位置替換成上一節中用來部署應用程式後端的目的地 URL。 例如：*http://you_backend.azurewebsites.net*。

    ```objc
    #define BACKEND_ENDPOINT @"<Enter Your Backend Endpoint>"
    ```

5. 在您的專案中，建立一個名為 **RegisterClient** 的新 **Cocoa Touch class** 作為與您建立之 ASP.NET 後端互動的介面。 建立繼承自 `NSObject`的類別。 然後在 RegisterClient.h 中新增下列程式碼。

    ```objc
    @interface RegisterClient : NSObject

    @property (strong, nonatomic) NSString* authenticationHeader;

    -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
        andCompletion:(void(^)(NSError*))completion;

    -(instancetype) initWithEndpoint:(NSString*)Endpoint;

    @end
    ```

6. 在 `RegisterClient.m` 中，更新 `@interface` 區段：

    ```objc
    @interface RegisterClient ()

    @property (strong, nonatomic) NSURLSession* session;
    @property (strong, nonatomic) NSURLSession* endpoint;

    -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                andCompletion:(void(^)(NSError*))completion;
    -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                completion:(void(^)(NSString*, NSError*))completion;
    -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSString*)token
                tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion;

    @end
    ```

7. 使用以下程式碼取代 RegisterClient.m 中的 `@implementation` 區段：

    ```objc
    @implementation RegisterClient

    // Globals used by RegisterClient
    NSString *const RegistrationIdLocalStorageKey = @"RegistrationId";

    -(instancetype) initWithEndpoint:(NSString*)Endpoint
    {
        self = [super init];
        if (self) {
            NSURLSessionConfiguration* config = [NSURLSessionConfiguration defaultSessionConfiguration];
            _session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:nil];
            _endpoint = Endpoint;
        }
        return self;
    }

    -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
                andCompletion:(void(^)(NSError*))completion
    {
        [self tryToRegisterWithDeviceToken:token tags:tags retry:YES andCompletion:completion];
    }

    -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                andCompletion:(void(^)(NSError*))completion
    {
        NSSet* tagsSet = tags?tags:[[NSSet alloc] init];

        NSString *deviceTokenString = [[token description]
            stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]];
        deviceTokenString = [[deviceTokenString stringByReplacingOccurrencesOfString:@" " withString:@""]
                                uppercaseString];

        [self retrieveOrRequestRegistrationIdWithDeviceToken: deviceTokenString
            completion:^(NSString* registrationId, NSError *error) {
            NSLog(@"regId: %@", registrationId);
            if (error) {
                completion(error);
                return;
            }

            [self upsertRegistrationWithRegistrationId:registrationId deviceToken:deviceTokenString
                tags:tagsSet andCompletion:^(NSURLResponse * response, NSError *error) {
                if (error) {
                    completion(error);
                    return;
                }

                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*)response;
                if (httpResponse.statusCode == 200) {
                    completion(nil);
                } else if (httpResponse.statusCode == 410 && retry) {
                    [self tryToRegisterWithDeviceToken:token tags:tags retry:NO andCompletion:completion];
                } else {
                    NSLog(@"Registration error with response status: %ld", (long)httpResponse.statusCode);

                    completion([NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                userInfo:nil]);
                }

            }];
        }];
    }

    -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSData*)token
                tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion
    {
        NSDictionary* deviceRegistration = @{@"Platform" : @"apns", @"Handle": token,
                                                @"Tags": [tags allObjects]};
        NSData* jsonData = [NSJSONSerialization dataWithJSONObject:deviceRegistration
                            options:NSJSONWritingPrettyPrinted error:nil];

        NSLog(@"JSON registration: %@", [[NSString alloc] initWithData:jsonData
                                            encoding:NSUTF8StringEncoding]);

        NSString* endpoint = [NSString stringWithFormat:@"%@/api/register/%@", _endpoint,
                                registrationId];
        NSURL* requestURL = [NSURL URLWithString:endpoint];
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"PUT"];
        [request setHTTPBody:jsonData];
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                self.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];
        [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];

        NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            if (!error)
            {
                completion(response, error);
            }
            else
            {
                NSLog(@"Error request: %@", error);
                completion(nil, error);
            }
        }];
        [dataTask resume];
    }

    -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                completion:(void(^)(NSString*, NSError*))completion
    {
        NSString* registrationId = [[NSUserDefaults standardUserDefaults]
                                    objectForKey:RegistrationIdLocalStorageKey];

        if (registrationId)
        {
            completion(registrationId, nil);
            return;
        }

        // request new one & save
        NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/api/register?handle=%@",
                                _endpoint, token]];
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"POST"];
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                self.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

        NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (!error && httpResponse.statusCode == 200)
            {
                NSString* registrationId = [[NSString alloc] initWithData:data
                    encoding:NSUTF8StringEncoding];

                // remove quotes
                registrationId = [registrationId substringWithRange:NSMakeRange(1,
                                    [registrationId length]-2)];

                [[NSUserDefaults standardUserDefaults] setObject:registrationId
                    forKey:RegistrationIdLocalStorageKey];
                [[NSUserDefaults standardUserDefaults] synchronize];

                completion(registrationId, nil);
            }
            else
            {
                NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                if (error)
                    completion(nil, error);
                else {
                    completion(nil, [NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                userInfo:nil]);
                }
            }
        }];
        [dataTask resume];
    }

    @end
    ```

    此程式碼會實作在指引文章[從您的應用程式後端註冊](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend)中所說明的邏輯，方法是使用 NSURLSession 來對您的應用程式後端執行 REST 呼叫，然後使用 NSUserDefaults 來將自通知中樞傳回的 registrationId 儲存於本機。

    此類別需要設定其 **authorizationHeader** 屬性，才能正常運作。 此屬性是在登入後由 **ViewController** 類別所設定。

8. 在 ViewController.h 中，為 RegisterClient.h 新增 `#import` 陳述式。 然後為裝置權杖新增宣告，並參照 `@interface` 區段中的 `RegisterClient` 執行個體：

    ```objc
    #import "RegisterClient.h"

    @property (strong, nonatomic) NSData* deviceToken;
    @property (strong, nonatomic) RegisterClient* registerClient;
    ```

9. 在 ViewController.m 中，於 `@interface` 區段中新增私用方法宣告：

    ```objc
    @interface ViewController () <UITextFieldDelegate, NSURLConnectionDataDelegate, NSXMLParserDelegate>

    // create the Authorization header to perform Basic authentication with your app back-end
    -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                    AndPassword:(NSString*)password;

    @end
    ```

    > [!NOTE]
    > 下列片段不是安全驗證結構描述，您應將 **createAndSetAuthenticationHeaderWithUsername:AndPassword:** 的實作替代成特定的驗證機制，使該機制產生註冊用戶端類別所利用的驗證權杖，例如 OAuth、Active Directory。

10. 然後在 `ViewController.m` 的 `@implementation` 區段中新增以下程式碼，這會新增實作以設定裝置權杖與驗證標頭。

    ```objc
    -(void) setDeviceToken: (NSData*) deviceToken
    {
        _deviceToken = deviceToken;
        self.LogInButton.enabled = YES;
    }

    -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                    AndPassword:(NSString*)password;
    {
        NSString* headerValue = [NSString stringWithFormat:@"%@:%@", username, password];

        NSData* encodedData = [[headerValue dataUsingEncoding:NSUTF8StringEncoding] base64EncodedDataWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];

        self.registerClient.authenticationHeader = [[NSString alloc] initWithData:encodedData
                                                    encoding:NSUTF8StringEncoding];
    }

    -(BOOL)textFieldShouldReturn:(UITextField *)textField
    {
        [textField resignFirstResponder];
        return YES;
    }
    ```

    請留意到設定裝置權杖會啟用登入按鈕。 這是因為作為登入動作的一部分，檢視控制器會向應用程式後端註冊推播通知。 因此，在裝置權杖已正確設定之前，不要讓登入動作可被存取。 只要登入是在推播註冊之前發生，您就可以將前者與後者分開。

11. 在 ViewController.m 中，使用以下程式碼片段為您的 [ **登入** ] 按鈕實作動作方法，以及實作一個方法來使用 ASP.NET 後端傳送通知訊息。

    ```objc
    - (IBAction)LogInAction:(id)sender {
        // create authentication header and set it in register client
        NSString* username = self.UsernameField.text;
        NSString* password = self.PasswordField.text;

        [self createAndSetAuthenticationHeaderWithUsername:username AndPassword:password];

        __weak ViewController* selfie = self;
        [self.registerClient registerWithDeviceToken:self.deviceToken tags:nil
            andCompletion:^(NSError* error) {
            if (!error) {
                dispatch_async(dispatch_get_main_queue(),
                ^{
                    selfie.SendNotificationButton.enabled = YES;
                    [self MessageBox:@"Success" message:@"Registered successfully!"];
                });
            }
        }];
    }

    - (void)SendNotificationASPNETBackend:(NSString*)pns UsernameTag:(NSString*)usernameTag
                Message:(NSString*)message
    {
        NSURLSession* session = [NSURLSession
            sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:nil
            delegateQueue:nil];

        // Pass the pns and username tag as parameters with the REST URL to the ASP.NET backend
        NSURL* requestURL = [NSURL URLWithString:[NSString
            stringWithFormat:@"%@/api/notifications?pns=%@&to_tag=%@", BACKEND_ENDPOINT, pns,
            usernameTag]];

        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"POST"];

        // Get the mock authenticationheader from the register client
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
            self.registerClient.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

        //Add the notification message body
        [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];
        [request setHTTPBody:[message dataUsingEncoding:NSUTF8StringEncoding]];

        // Execute the send notification REST API on the ASP.NET Backend
        NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (error || httpResponse.statusCode != 200)
            {
                NSString* status = [NSString stringWithFormat:@"Error Status for %@: %d\nError: %@\n",
                                    pns, httpResponse.statusCode, error];
                dispatch_async(dispatch_get_main_queue(),
                ^{
                    // Append text because all 3 PNS calls may also have information to view
                    [self.sendResults setText:[self.sendResults.text stringByAppendingString:status]];
                });
                NSLog(status);
            }

            if (data != NULL)
            {
                xmlParser = [[NSXMLParser alloc] initWithData:data];
                [xmlParser setDelegate:self];
                [xmlParser parse];
            }
        }];
        [dataTask resume];
    }
    ```

12. 更新 [ **傳送通知** ] 按鈕的動作來使用 ASP.NET 後端，並傳送至由任何開關啟用的任何 PNS。

    ```objc
    - (IBAction)SendNotificationMessage:(id)sender
    {
        //[self SendNotificationRESTAPI];
        [self SendToEnabledPlatforms];
    }

    -(void)SendToEnabledPlatforms
    {
        NSString* json = [NSString stringWithFormat:@"\"%@\"",self.notificationMessage.text];

        [self.sendResults setText:@""];

        if ([self.WNSSwitch isOn])
            [self SendNotificationASPNETBackend:@"wns" UsernameTag:self.RecipientField.text Message:json];

        if ([self.GCMSwitch isOn])
            [self SendNotificationASPNETBackend:@"gcm" UsernameTag:self.RecipientField.text Message:json];

        if ([self.APNSSwitch isOn])
            [self SendNotificationASPNETBackend:@"apns" UsernameTag:self.RecipientField.text Message:json];
    }
    ```

13. 在 **ViewDidLoad**函數中，新增下列內容以具現化 RegisterClient 執行個體，並設定文字欄位的委派。

    ```objc
    self.UsernameField.delegate = self;
    self.PasswordField.delegate = self;
    self.RecipientField.delegate = self;
    self.registerClient = [[RegisterClient alloc] initWithEndpoint:BACKEND_ENDPOINT];
    ```

14. 現在於 **AppDelegate.m** 中，移除 `application:didRegisterForPushNotificationWithDeviceToken:` 方法中的所有內容，並使用下列內容取代它，以確定檢視控制器包含從 APN 擷取的最新裝置權杖：

    ```objc
    // Add import to the top of the file
    #import "ViewController.h"

    - (void)application:(UIApplication *)application
                didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
        ViewController* rvc = (ViewController*) self.window.rootViewController;
        rvc.deviceToken = deviceToken;
    }
    ```

15. 最後，在 **AppDelegate.m**中，確定您有下列方法：

    ```objc
    - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
        NSLog(@"%@", userInfo);
        [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
    }
    ```

## <a name="test-the-application"></a>測試應用程式

1. 在 XCode 中，在實體 iOS 裝置上執行應用程式 (推播通知無法在模擬器中運作)。
2. 在 iOS 應用程式 UI 中，為使用者名稱和密碼輸入相同的值。 然後按一下 [ **登入**]。

    ![iOS 測試應用程式][2]

3. 您應該會看到註冊成功的快顯通知。 按一下 [確定]。

    ![顯示的 iOS 測試通知][3]

4. 在 **收件者使用者名稱標記* 文字欄位中，輸入從其他裝置註冊時搭配使用的使用者名稱標記。
5. 輸入通知訊息並按一下 [ **傳送通知**]。 只有已經使用收件者使用者名稱標記註冊的裝置才會收到通知訊息。 通知訊息只會傳送給那些使用者。

    ![iOS 測試已標記的通知][4]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已學會如何針對具有與其註冊相關聯標記的使用者，將通知推送至這些特定使用者。 若要了解如何推送以位置為基礎的通知，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[推送以位置為基礎的通知](notification-hubs-push-bing-spartial-data-geofencing-notification.md)


[1]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-interface.png
[2]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-user-pwd.png
[3]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-registered.png
[4]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-msg.png
