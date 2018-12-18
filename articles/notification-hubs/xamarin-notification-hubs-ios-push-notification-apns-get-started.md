---
title: 使用 Azure 通知中樞將通知推送至 Xamarin.iOS 應用程式 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 通知中樞將推播通知傳送至 Xamarin.iOS 應用程式。
services: notification-hubs
keywords: ios 推播通知,推播訊息,推播通知
documentationcenter: xamarin
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 4d4dfd42-c5a5-4360-9d70-7812f96924d2
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 08/23/2018
ms.author: dimazaid
ms.openlocfilehash: 4704d9bb04f6dc69c69df434562c03b868baf045
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42917698"
---
# <a name="tutorial-push-notifications-to-xamarinios-apps-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至 Xamarin.iOS 應用程式

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>概觀

本教學課程示範如何使用 Azure 通知中樞將推播通知傳送至 iOS 應用程式。 您會使用 [Apple Push Notification Service (APNS)](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html) 建立可接收推播通知的空白 Xamarin.iOS 應用程式。

完成之後，您將可使用通知中樞，將推播通知廣播到所有執行您的應用程式的裝置。 [NotificationHubs 應用程式][GitHub]範例中提供完成的程式碼。

在本教學課程中，您會建立/更新程式碼以執行下列工作：

> [!div class="checklist"]
> * 產生憑證簽署要求檔案
> * 針對推播通知註冊應用程式
> * 建立應用程式的佈建設定檔
> * 針對 iOS 推播通知設定您的通知中樞
> * 傳送測試推播通知

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
- 最新版的 [Xcode][Install Xcode]
- iOS 10 (或更新版本) 相容的裝置
- [Apple Developer Program](https://developer.apple.com/programs/) 成員資格。
- [Visual Studio for Mac]
  
  > [!NOTE]
  > 基於 iOS 推播通知的組態需求，您必須在實體 iOS 裝置 (iPhone 或 iPad) 上部署和測試範例應用程式，而不是在模擬器中。

完成本教學課程是 Xamarin.iOS 應用程式的所有其他通知中樞教學課程的先決條件。

[!INCLUDE [Notification Hubs Enable Apple Push Notifications](../../includes/notification-hubs-enable-apple-push-notifications.md)]

## <a name="configure-your-notification-hub-for-ios-push-notifications"></a>針對 iOS 推播通知設定您的通知中樞

本節將引導您進行相關步驟以建立新的通知中樞，並使用您先前建立的 **.p12** 推播憑證，設定以 APNS 進行驗證的機制。 如果您想要使用已經建立的通知中樞，可以跳至步驟 5。

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-ios-settings-for-the-notification-hub"></a>設定通知中樞的 iOS 設定

1. 在 [通知設定] 群組中選取 [Apple] \(APNS\)。
2. 選取 [憑證]、按一下 [檔案] 圖示，然後選取您先前匯出的 **.p12** 檔案。
3. 指定憑證的**密碼**。
4. 選取 [沙箱] 模式。 只有在您想傳送推播通知給從市集購買應用程式的使用者時，才可使用 [生產] 模式。

    ![在 Azure 入口網站中設定 APNS][6]

    ![在 Azure 入口網站中設定 APNS 憑證][7]

現在，您的通知中心已設定成使用 APNS，而且您有可用來註冊應用程式和傳送推播通知的連接字串。

## <a name="connect-your-app-to-the-notification-hub"></a>將您的應用程式連接到通知中樞

### <a name="create-a-new-project"></a>建立新專案

1. 在 Visual Studio 中建立新的 iOS 專案，並選取 [單一檢視應用程式] 範本，然後按 [下一步]

     ![Visual Studio - 選取應用程式類型][31]

2. 輸入應用程式名稱和組織識別碼，然後按 [下一步] 和 [建立]

3. 在 [方案] 檢視中，按兩下 *Into.plist*，並確定 [身分識別] 下方的套件組合識別碼符合您在建立佈建設定檔時所使用的套件組合識別碼。 在 [簽署] 下方，確定您已於 [小組] 下方選取您的開發人員帳戶，已選取 [自動管理簽署]，並已自動選取您的簽署憑證和佈建設定檔。

    ![Visual Studio- iOS 應用程式組態][32]

4. 在 [解決方案] 檢視中按兩下 *Entitlements.plist*，並確定已核取 [啟用推播通知]**。

    ![Visual Studio- iOS 權利組態][33]

5. 新增 Azure 傳訊套件。 在 [方案] 檢視中，以滑鼠右鍵按一下專案，然後選取 [新增] > [新增 NuGet 套件]。 搜尋 **Xamarin.Azure.NotificationHubs.iOS**，並將此套件新增至專案中。

6. 在您的類別中新增檔案，將其命名為 **Constants.cs**，然後新增下列變數，並以您的「中樞名稱」和先前記下的「DefaultListenSharedAccessSignature」取代字串常值預留位置。

    ```csharp
    // Azure app-specific connection string and hub path
    public const string ListenConnectionString = "<Azure DefaultListenSharedAccess Connection String>";
    public const string NotificationHubName = "<Azure Notification Hub Name>";
    ```

7. 在 **AppDelegate.cs**中，新增下列 using 陳述式：

    ```csharp
    using WindowsAzure.Messaging;
    ```

8. 宣告 **SBNotificationHub**的執行個體：

    ```csharp
    private SBNotificationHub Hub { get; set; }
    ```

9. 在 **AppDelegate.cs** 中，更新 **FinishedLaunching()** 以符合下列程式碼：

    ```csharp
    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        if (UIDevice.CurrentDevice.CheckSystemVersion(10, 0))
        {
            UNUserNotificationCenter.Current.RequestAuthorization(UNAuthorizationOptions.Alert | UNAuthorizationOptions.Sound | UNAuthorizationOptions.Sound,
                                                                    (granted, error) =>
            {
                if (granted)
                    InvokeOnMainThread(UIApplication.SharedApplication.RegisterForRemoteNotifications);
            });
        } else if (UIDevice.CurrentDevice.CheckSystemVersion (8, 0)) {
            var pushSettings = UIUserNotificationSettings.GetSettingsForTypes (
                    UIUserNotificationType.Alert | UIUserNotificationType.Badge | UIUserNotificationType.Sound,
                    new NSSet ());

            UIApplication.SharedApplication.RegisterUserNotificationSettings (pushSettings);
            UIApplication.SharedApplication.RegisterForRemoteNotifications ();
        } else {
            UIRemoteNotificationType notificationTypes = UIRemoteNotificationType.Alert | UIRemoteNotificationType.Badge | UIRemoteNotificationType.Sound;
            UIApplication.SharedApplication.RegisterForRemoteNotificationTypes (notificationTypes);
        }

        return true;
    }
    ```

10. 覆寫 **AppDelegate.cs** 中的 **RegisteredForRemoteNotifications()** 方法：

    ```csharp
    public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
    {
        Hub = new SBNotificationHub(Constants.ListenConnectionString, Constants.NotificationHubName);

        Hub.UnregisterAllAsync (deviceToken, (error) => {
            if (error != null)
            {
                System.Diagnostics.Debug.WriteLine("Error calling Unregister: {0}", error.ToString());
                return;
            }

            NSSet tags = null; // create tags if you want
            Hub.RegisterNativeAsync(deviceToken, tags, (errorCallback) => {
                if (errorCallback != null)
                    System.Diagnostics.Debug.WriteLine("RegisterNativeAsync error: " + errorCallback.ToString());
            });
        });
    }
    ```

11. 覆寫 **AppDelegate.cs** 中的 **ReceivedRemoteNotification()** 方法：

    ```csharp
    public override void ReceivedRemoteNotification(UIApplication application, NSDictionary userInfo)
    {
        ProcessNotification(userInfo, false);
    }
    ```

12. 在 **AppDelegate.cs** 中建立下列 **ProcessNotification()** 方法：

    ```csharp
    void ProcessNotification(NSDictionary options, bool fromFinishedLaunching)
    {
        // Check to see if the dictionary has the aps key.  This is the notification payload you would have sent
        if (null != options && options.ContainsKey(new NSString("aps")))
        {
            //Get the aps dictionary
            NSDictionary aps = options.ObjectForKey(new NSString("aps")) as NSDictionary;

            string alert = string.Empty;

            //Extract the alert text
            // NOTE: If you're using the simple alert by just specifying
            // "  aps:{alert:"alert msg here"}  ", this will work fine.
            // But if you're using a complex alert with Localization keys, etc.,
            // your "alert" object from the aps dictionary will be another NSDictionary.
            // Basically the JSON gets dumped right into a NSDictionary,
            // so keep that in mind.
            if (aps.ContainsKey(new NSString("alert")))
                alert = (aps [new NSString("alert")] as NSString).ToString();

            //If this came from the ReceivedRemoteNotification while the app was running,
            // we of course need to manually process things like the sound, badge, and alert.
            if (!fromFinishedLaunching)
            {
                //Manually show an alert
                if (!string.IsNullOrEmpty(alert))
                {
                    UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
                    avAlert.Show();
                }
            }
        }
    }
    ```

    > [!NOTE]
    > 您可以選擇覆寫 **FailedToRegisterForRemoteNotifications()** 以處理沒有網路連線等的情況。 這點特別重要，因為使用者可能在離線模式下 (例如飛機) 啟動您的應用程式，而您希望針對您的應用程式來處理推播傳訊案例。

13. 在您的裝置上執行應用程式。

## <a name="send-test-push-notifications"></a>傳送測試推播通知

您可以在 [Azure 入口網站] 中，使用 [測試傳送] 選項測試應用程式能否接收通知。 它會將測試推播通知傳送至您的裝置。

![Azure 入口網站 - 測試傳送][30]

推播通知通常會以後端服務傳送，例如 Mobile Apps 或使用相容程式庫的 ASP.NET。 如果您的後端無法使用程式庫，您也可以直接使用 REST API 來傳送通知訊息。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將廣播通知傳送至所有向後端註冊的 iOS 裝置。 若要了解如何將通知推送至特定 iOS 裝置，請繼續進行下列教學課程：

> [!div class="nextstepaction"]
>[將通知推送至特定裝置](notification-hubs-ios-xplat-segmented-apns-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config-cert.png
[213]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-console-app.png

[215]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler1.png
[216]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler2.png

[30]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[31]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-ios-app.png
[32]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-app-settings.png
[33]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-entitlements-settings.png


<!-- URLs. -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[Visual Studio for Mac]: https://visualstudio.microsoft.com/vs/mac/

[Local and Push Notification Programming Guide]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1
[Apple Push Notification Service]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html
[Apple Push Notification Service fwlink]: http://go.microsoft.com/fwlink/p/?LinkId=272584

[GitHub]: https://github.com/xamarin/mobile-samples/tree/master/Azure/NotificationHubs
[Azure 入口網站]: https://portal.azure.com