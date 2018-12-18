---
title: 將推播通知新增至 Xamarin.Forms 應用程式 | Microsoft Docs
description: 了解如何使用 Azure 服務將多平台推播通知傳送至 Xamarin.Forms 應用程式。
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: crdun
editor: ''
ms.assetid: d9b1ba9a-b3f2-4d12-affc-2ee34311538b
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin
ms.devlang: dotnet
ms.topic: article
ms.date: 10/12/2016
ms.author: crdun
ms.openlocfilehash: 2658dcb0d206086aad1443e2dc720b6f1f55906f
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818153"
---
# <a name="add-push-notifications-to-your-xamarinforms-app"></a>將推播通知新增至 Xamarin.Forms 應用程式

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## <a name="overview"></a>概觀

在本教學課程中，您會將推播通知新增至 [Xamarin.Forms 快速入門](app-service-mobile-xamarin-forms-get-started.md)所產生的所有專案。 這表示每次插入記錄時，就會將推播通知傳送到所有跨平台用戶端。

如果您不要使用下載的快速入門伺服器專案，將需要推播通知擴充套件。 如需詳細資訊，請參閱[使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

## <a name="prerequisites"></a>必要條件

若為 iOS，您需要 [Apple Developer Program 成員資格](https://developer.apple.com/programs/ios/)和實體 iOS 裝置。 [iOS 模擬器不支援推播通知](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator.html)。

## <a name="configure-hub"></a>設定通知中樞

[!INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

## <a name="update-the-server-project-to-send-push-notifications"></a>更新伺服器專案以傳送推播通知

[!INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

## <a name="configure-and-run-the-android-project-optional"></a>設定和執行 Android 專案 (選擇性)

完成本節可以為適用於 Android 的 Xamarin.Forms Droid 專案啟用推播通知。

### <a name="enable-firebase-cloud-messaging-fcm"></a>啟用 Firebase 雲端傳訊 (FCM)

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

### <a name="configure-the-mobile-apps-back-end-to-send-push-requests-by-using-fcm"></a>設定 Mobile Apps 後端以使用 FCM 傳送推送要求

[!INCLUDE [app-service-mobile-android-configure-push](../../includes/app-service-mobile-android-configure-push.md)]

### <a name="add-push-notifications-to-the-android-project"></a>將推播通知新增至 Android 專案

為後端設定了 FCM 後，您就可以在用戶端中新增元件和程式碼，以向 FCM 註冊。 您也可以透過 Mobile Apps 後端向 Azure 通知中樞註冊推播通知，並接收通知。

1. 在 **Droid** 專案中，以滑鼠右鍵按一下 [參考] > [管理 NuGet 套件...]。
1. 在 [NuGet 套件管理員] 視窗中，搜尋 **Xamarin.Firebase.Messaging** 套件，並將其新增至專案。
1. 在 **Droid** 專案的專案屬性中，將應用程式設定為使用 Android 7.0 版或更新版本進行編譯。
1. 將從 Firebase 主控台下載的 **google-services.json** 檔案新增至 **Droid** 專案的根目錄，並將其建置動作設定為 **GoogleServicesJson**。 如需詳細資訊，請參閱[新增 Google Services JSON 檔案](https://developer.xamarin.com/guides/android/data-and-cloud-services/google-messaging/remote-notifications-with-fcm/#Add_the_Google_Services_JSON_File)。

#### <a name="registering-with-firebase-cloud-messaging"></a>向 Firebase 雲端傳訊進行註冊

1. 開啟 **AndroidManifest.xml** 檔案，並將下列的 `<receiver>` 元素插入 `<application>` 元素：

    ```xml
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="${applicationId}" />
        </intent-filter>
    </receiver>
    ```

#### <a name="implementing-the-firebase-instance-id-service"></a>實作 Firebase 執行個體識別碼服務

1. 將新的類別新增至名為 `FirebaseRegistrationService` 的 **Droid** 專案，並確定下列 `using` 陳述式出現在檔案頂端：

    ```csharp
    using System.Threading.Tasks;
    using Android.App;
    using Android.Util;
    using Firebase.Iid;
    using Microsoft.WindowsAzure.MobileServices;
    ```

1. 將空的 `FirebaseRegistrationService` 類別取代為下列程式碼：

    ```csharp
    [Service]
    [IntentFilter(new[] { "com.google.firebase.INSTANCE_ID_EVENT" })]
    public class FirebaseRegistrationService : FirebaseInstanceIdService
    {
        const string TAG = "FirebaseRegistrationService";

        public override void OnTokenRefresh()
        {
            var refreshedToken = FirebaseInstanceId.Instance.Token;
            Log.Debug(TAG, "Refreshed token: " + refreshedToken);
            SendRegistrationTokenToAzureNotificationHub(refreshedToken);
        }

        void SendRegistrationTokenToAzureNotificationHub(string token)
        {
            // Update notification hub registration
            Task.Run(async () =>
            {
                await AzureNotificationHubService.RegisterAsync(TodoItemManager.DefaultManager.CurrentClient.GetPush(), token);
            });
        }
    }
    ```

    `FirebaseRegistrationService` 類別負責產生安全性權杖，以授與應用程式存取 FCM 的權限。 應用程式從 FCM 收到註冊權杖時會叫用 `OnTokenRefresh` 方法。 此方法會從 `FirebaseInstanceId.Instance.Token` 屬性取出權杖，而 FCM 會以非同步方式更新該屬性。 `OnTokenRefresh` 方法並不常受到叫用，因為只有在安裝或解除安裝應用程式、使用者刪除應用程式資料、應用程式清除執行個體識別碼，或權杖的安全性遭到洩露時，才會更新權杖。 此外，FCM 執行個體識別碼服務會要求應用程式定期重新整理其權杖，通常是每 6 個月一次。

    `OnTokenRefresh` 方法也會叫用 `SendRegistrationTokenToAzureNotificationHub` 方法，用來讓使用者的註冊權杖與 Azure 通知中樞產生關聯。

#### <a name="registering-with-the-azure-notification-hub"></a>向 Azure 通知中樞進行註冊

1. 將新的類別新增至名為 `AzureNotificationHubService` 的 **Droid** 專案，並確定下列 `using` 陳述式出現在檔案頂端：

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Android.Util;
    using Microsoft.WindowsAzure.MobileServices;
    using Newtonsoft.Json.Linq;
    ```

1. 將空的 `AzureNotificationHubService` 類別取代為下列程式碼：

    ```csharp
    public class AzureNotificationHubService
    {
        const string TAG = "AzureNotificationHubService";

        public static async Task RegisterAsync(Push push, string token)
        {
            try
            {
                const string templateBody = "{\"data\":{\"message\":\"$(messageParam)\"}}";
                JObject templates = new JObject();
                templates["genericMessage"] = new JObject
                {
                    {"body", templateBody}
                };

                await push.RegisterAsync(token, templates);
                Log.Info("Push Installation Id: ", push.InstallationId.ToString());
            }
            catch (Exception ex)
            {
                Log.Error(TAG, "Could not register with Notification Hub: " + ex.Message);
            }
        }
    }
    ```

    `RegisterAsync` 方法會使用 Firebase 註冊權杖建立 JSON 格式的簡易通知訊息範本，並註冊為可從通知中樞接收範本通知。 這可確保從 Azure 通知中樞送出的任何通知會送往註冊權杖所代表的裝置。

#### <a name="displaying-the-contents-of-a-push-notification"></a>顯示推播通知的內容

1. 將新的類別新增至名為 `FirebaseNotificationService` 的 **Droid** 專案，並確定下列 `using` 陳述式出現在檔案頂端：

    ```csharp
    using Android.App;
    using Android.Content;
    using Android.Media;
    using Android.Support.V7.App;
    using Android.Util;
    using Firebase.Messaging;
    ```

1. 將空的 `FirebaseNotificationService` 類別取代為下列程式碼：

    ```csharp
    [Service]
    [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
    public class FirebaseNotificationService : FirebaseMessagingService
    {
        const string TAG = "FirebaseNotificationService";

        public override void OnMessageReceived(RemoteMessage message)
        {
            Log.Debug(TAG, "From: " + message.From);

            // Pull message body out of the template
            var messageBody = message.Data["message"];
            if (string.IsNullOrWhiteSpace(messageBody))
                return;

            Log.Debug(TAG, "Notification message body: " + messageBody);
            SendNotification(messageBody);
        }

        void SendNotification(string messageBody)
        {
            var intent = new Intent(this, typeof(MainActivity));
            intent.AddFlags(ActivityFlags.ClearTop);
            var pendingIntent = PendingIntent.GetActivity(this, 0, intent, PendingIntentFlags.OneShot);

            var notificationBuilder = new NotificationCompat.Builder(this)
                .SetSmallIcon(Resource.Drawable.ic_stat_ic_notification)
                .SetContentTitle("New Todo Item")
                .SetContentText(messageBody)
                .SetContentIntent(pendingIntent)
                .SetSound(RingtoneManager.GetDefaultUri(RingtoneType.Notification))
                .SetAutoCancel(true);

            var notificationManager = NotificationManager.FromContext(this);
            notificationManager.Notify(0, notificationBuilder.Build());
        }
    }
    ```

    應用程式從 FCM 收到通知時所叫用的 `OnMessageReceived` 方法會擷取訊息內容，並呼叫 `SendNotification` 方法。 此方法會將訊息內容轉換成應用程式執行時所啟動的本機通知，並在通知區域中顯示通知。

現在，您已經準備好在 Android 裝置或模擬器上執行的應用程式中測試推播通知。

### <a name="test-push-notifications-in-your-android-app"></a>在 Android 應用程式中測試推播通知

只有要在模擬器上測試時，才需要前兩個步驟。

1. 請確定您要在利用 Google Play Services 設定的裝置或模擬器上進行部署或偵錯。 藉由檢查裝置或模擬器上已安裝 **Play**  應用程式，即可確認這點。
2. 按一下 [應用程式] > [設定] > [新增帳戶] 將 Google 帳戶加入 Android 裝置。 然後遵循提示在裝置中新增現有 Google 帳戶，或是建立一個新的帳戶。
3. 在 Visual Studio 或 Xamarin Studio 中，以滑鼠右鍵按一下 **Droid** 專案，然後按一下 [設定為啟始專案]。
4. 按一下 [執行] 以建置專案，並在 Android 裝置或模擬器上啟動應用程式。
5. 在應用程式中輸入一項工作，然後按一下加號 (**+**) 圖示。
6. 確認在加入項目時收到通知。

## <a name="configure-and-run-the-ios-project-optional"></a>設定和執行 iOS 專案 (選擇性)

這一節適用於對 iOS 裝置執行 Xamarin iOS 專案。 如果未使用 iOS 裝置，可以略過這一節。

[!INCLUDE [Enable Apple Push Notifications](../../includes/notification-hubs-enable-apple-push-notifications.md)]

#### <a name="configure-the-notification-hub-for-apns"></a>設定 APNS 的通知中樞

[!INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

接下來，您將在 Xamarin Studio 或 Visual Studio 中設定 iOS 專案設定。

[!INCLUDE [app-service-mobile-xamarin-ios-configure-project](../../includes/app-service-mobile-xamarin-ios-configure-project.md)]

#### <a name="add-push-notifications-to-your-ios-app"></a>將推播通知加入至 iOS 應用程式

1. 在 **iOS** 專案中開啟 AppDelegate.cs，並將下列陳述式新增到程式碼檔案頂端。

    ```csharp
    using Newtonsoft.Json.Linq;
    ```

2. 在 **AppDelegate** 類別中，新增 **RegisteredForRemoteNotifications** 事件的覆寫以註冊通知：

    ```csharp
    public override void RegisteredForRemoteNotifications(UIApplication application,
        NSData deviceToken)
    {
        const string templateBodyAPNS = "{\"aps\":{\"alert\":\"$(messageParam)\"}}";

        JObject templates = new JObject();
        templates["genericMessage"] = new JObject
            {
                {"body", templateBodyAPNS}
            };

        // Register for push with your mobile app
        Push push = TodoItemManager.DefaultManager.CurrentClient.GetPush();
        push.RegisterAsync(deviceToken, templates);
    }
    ```

3. 此外，也在 **AppDelegate** 中新增 **DidReceiveRemoteNotification** 事件處理常式的下列覆寫：

    ```csharp
    public override void DidReceiveRemoteNotification(UIApplication application,
        NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
    {
        NSDictionary aps = userInfo.ObjectForKey(new NSString("aps")) as NSDictionary;

        string alert = string.Empty;
        if (aps.ContainsKey(new NSString("alert")))
            alert = (aps[new NSString("alert")] as NSString).ToString();

        //show alert
        if (!string.IsNullOrEmpty(alert))
        {
            UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
            avAlert.Show();
        }
    }
    ```

    當應用程式執行時，此方法會處理傳入的通知。

4. 在 **AppDelegate** 類別中，將下列程式碼新增到 **FinishedLaunching** 方法：

    ```csharp
    // Register for push notifications.
    var settings = UIUserNotificationSettings.GetSettingsForTypes(
        UIUserNotificationType.Alert
        | UIUserNotificationType.Badge
        | UIUserNotificationType.Sound,
        new NSSet());

    UIApplication.SharedApplication.RegisterUserNotificationSettings(settings);
    UIApplication.SharedApplication.RegisterForRemoteNotifications();
    ```

    這能夠支援遠端通知，並要求推播註冊。

您的應用程式現在已更新為支援推播通知。

#### <a name="test-push-notifications-in-your-ios-app"></a>在 iOS 應用程式中測試推播通知

1. 以滑鼠右鍵按一下 iOS 專案，然後按一下 [設為起始專案]。
2. 在 Visual Studio 中按下 [執行] 按鈕或 **F5** 以建置專案，並在 iOS 裝置上啟動應用程式。 然後，按一下 [確定] 以接收推撥通知。

   > [!NOTE]
   > 您必須明確地接受來自應用程式的推播通知。 只有在應用程式第一次執行時，才會發生此要求。

3. 在應用程式中輸入一項工作，然後按一下加號 (**+**) 圖示。
4. 確認您已接收到通知，然後按一下 [確定] 以關閉通知。

## <a name="configure-and-run-windows-projects-optional"></a>設定和執行 Windows 專案 (選擇性)

本節說明執行適用於 Windows 裝置的 Xamarin.Forms WinApp 和 WinPhone81 專案。 這些步驟也支援通用 Windows 平台 (UWP) 專案。 如果未使用 Windows 裝置，可以略過這一節。

#### <a name="register-your-windows-app-for-push-notifications-with-windows-notification-service-wns"></a>為 Windows 應用程式向 Windows 通知服務 (WNS) 註冊推播通知

[!INCLUDE [app-service-mobile-register-wns](../../includes/app-service-mobile-register-wns.md)]

#### <a name="configure-the-notification-hub-for-wns"></a>設定 WNS 的通知中樞

[!INCLUDE [app-service-mobile-configure-wns](../../includes/app-service-mobile-configure-wns.md)]

#### <a name="add-push-notifications-to-your-windows-app"></a>將推播通知加入至 Windows 應用程式

1. 在 Visual Studio 中，開啟 Windows 專案中的 **App.xaml.cs**，並新增下列陳述式。

    ```csharp
    using Newtonsoft.Json.Linq;
    using Microsoft.WindowsAzure.MobileServices;
    using System.Threading.Tasks;
    using Windows.Networking.PushNotifications;
    using <your_TodoItemManager_portable_class_namespace>;
    ```

    使用包含 `TodoItemManager` 類別的可攜式專案命名空間來取代 `<your_TodoItemManager_portable_class_namespace>`。

2. 在 App.xaml.cs 中，新增下列 **InitNotificationsAsync** 方法：

    ```csharp
    private async Task InitNotificationsAsync()
    {
        var channel = await PushNotificationChannelManager
            .CreatePushNotificationChannelForApplicationAsync();

        const string templateBodyWNS =
            "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(messageParam)</text></binding></visual></toast>";

        JObject headers = new JObject();
        headers["X-WNS-Type"] = "wns/toast";

        JObject templates = new JObject();
        templates["genericMessage"] = new JObject
        {
            {"body", templateBodyWNS},
            {"headers", headers} // Needed for WNS.
        };

        await TodoItemManager.DefaultManager.CurrentClient.GetPush()
            .RegisterAsync(channel.Uri, templates);
    }
    ```

    這個方法會取得推播通知通道，並註冊範本以接收來自通知中樞的範本通知。 支援 messageParam  的範本通知會傳送到此用戶端。

3. 在 App.xaml.cs 中，新增 `async` 修飾詞以更新 **OnLaunched** 事件處理常式方法定義。 然後，在方法的結尾新增下列程式碼行：

    ```csharp
    await InitNotificationsAsync();
    ```

    如此可確保每次啟動應用程式時都會建立或重新整理推播通知註冊。 必須如此以保證 WNS 推送通道永遠在作用中。  

4. 在 Visual Studio 的 [方案總管] 中，開啟 **Package.appxmanifest** 檔案，然後把 [通知] 下方的 [支援快顯通知] 設為 [是]。
5. 建置應用程式並確認沒有錯誤。 您用戶端應用程式現在應該註冊 Mobile Apps 後端的範本通知。 針對方案中每個 Windows 專案重複操作這一節。

#### <a name="test-push-notifications-in-your-windows-app"></a>在 Windows 應用程式中測試推播通知

1. 在 Visual Studio 中，以滑鼠右鍵按一下 Windows 專案，然後按一下 [設定為啟始專案]。
2. 按 [執行] 按鈕，以建立專案並啟動應用程式。
3. 在應用程式中輸入新 todoitem 的名稱，然後按一下加號 (**+**) 圖示來加入它。
4. 確認在加入項目時收到通知。

## <a name="next-steps"></a>後續步驟

您可以深入了解推播通知︰

* [從 Azure Mobile Apps 傳送推播通知](https://developer.xamarin.com/guides/xamarin-forms/cloud-services/push-notifications/azure/)
* [Firebase 雲端傳訊](https://developer.xamarin.com/guides/android/data-and-cloud-services/google-messaging/firebase-cloud-messaging/)
* [使用 Firebase 雲端傳訊進行遠端通知](https://developer.xamarin.com/guides/android/data-and-cloud-services/google-messaging/remote-notifications-with-fcm/)
* [診斷推播通知問題](../notification-hubs/notification-hubs-push-notification-fixer.md)  
  通知遭到捨棄或未抵達裝置有各種原因。 本主題說明如何分析及找出推播通知失敗的根本原因。

您也可以繼續進行下列其中一個教學課程：

* [將驗證新增至應用程式中](app-service-mobile-xamarin-forms-get-started-users.md)  
  ：了解如何使用識別提供者來驗證應用程式的使用者。
* [啟用應用程式的離線同步處理](app-service-mobile-xamarin-forms-get-started-offline-data.md)  
  了解如何使用 Mobile Apps 後端，為應用程式新增離線支援。 使用離線同步處理時，使用者可與行動應用程式進行互動&mdash;檢視、新增或修改資料&mdash;即使沒有網路連線進也可行。

<!-- Images. -->

<!-- URLs. -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532
[apns object]: http://go.microsoft.com/fwlink/p/?LinkId=272333
