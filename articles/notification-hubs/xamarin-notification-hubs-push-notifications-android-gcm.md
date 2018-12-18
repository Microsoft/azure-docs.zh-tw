---
title: 使用 Azure 通知中樞將通知推送至 Xamarin.Android 應用程式 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure 通知中樞將推播通知傳送至 Xamarin Android 應用程式。
author: dimazaid
manager: kpiteira
editor: spelluru
services: notification-hubs
documentationcenter: xamarin
ms.assetid: 0be600fe-d5f3-43a5-9e5e-3135c9743e54
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 94e8e813b537d304e62854b81979d433d0645115
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41917907"
---
# <a name="tutorial-push-notifications-to-xamarinandroid-apps-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至 Xamarin.Android 應用程式
[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>概觀
本教學課程示範如何使用 Azure 通知中樞將推播通知傳送至 Xamarin.Android 應用程式。 您會建立可使用 Firebase 雲端通訊 (FCM) 接收推播通知的空白 Xamarin.Android 應用程式。 您會使用通知中樞，將推播通知廣播到所有執行您的應用程式的裝置。 [NotificationHubs 應用程式][GitHub]範例中提供完成的程式碼。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立 Firebase 專案並啟用 Firebase 雲端通訊
> * 建立通知中樞
> * 建立 Xamarin.Android 應用程式，並將其連線至通知中樞
> * 從 Azure 入口網站傳送測試通知

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
- [Visual Studio 搭配 Xamarin] (在 Windows 上) 或 [Visual Studio for Mac] (在 OS X 上)。
- 有效的 Google 帳戶

## <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging"></a>建立 Firebase 專案並啟用 Firebase 雲端通訊
[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

## <a name="create-a-notification-hub"></a>建立通知中樞
[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-settings-for-the-notification-hub"></a>設定通知中樞的 GCM 設定

1. 在 [通知設定] 區段中選取 [Google] \(GCM\)。 
2. 輸入您從 Google Firebase 主控台記下的**舊有伺服器金鑰**。 
3. 在工具列上選取 [儲存]。 

    ![](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

您的通知中樞已設定成使用 FCM，而且您已擁有可用來註冊應用程式以接收通知和傳送推播通知的連接字串。

## <a name="create-xamarinandroid-app-and-connect-it-to-notification-hub"></a>建立 Xamarin.Android 應用程式，並將其連線至通知中樞

### <a name="create-visual-studio-project-and-add-nuget-packages"></a>建立 Visual Studio 專案並新增 NuGet 套件
1. 在 Visual Studio 中指向 [檔案]，選取 [新增]，然後選取 [專案]。 
   
      ![Visual Studio - 建立新的 Android 專案](./media/partner-xamarin-notification-hubs-android-get-started/new-project-dialog.png)
2. 在 [方案總管] 視窗中展開 [屬性]，然後按一下 **AndroidManifest.xml**。 更新套件名稱，使其符合您在 Google Firebase 主控台中將 Firebase 雲端通訊新增至專案時輸入的套件名稱。 

      ![GCM 中的套件名稱](./media/partner-xamarin-notification-hubs-android-get-started/package-name-gcm.png)
3. 以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件...]。 
4. 選取 [瀏覽] 索引標籤。搜尋 **Xamarin.GooglePlayServices.Base**。 在結果清單中選取 **Xamarin.GooglePlayServices.Base**。 然後，選取 [安裝]。 

      ![Google Play 服務 NuGet](./media/partner-xamarin-notification-hubs-android-get-started/google-play-services-nuget.png)
5. 在 [NuGet 套件管理員] 視窗中，搜尋 **Xamarin.Firebase.Messaging**。 在結果清單中選取 **Xamarin.Firebase.Messaging**。 然後，選取 [安裝]。 
6. 現在，請搜尋 **Xamarin.Azure.NotificationHubs.Android**。 在結果清單中選取 **Xamarin.Azure.NotificationHubs.Android**。 然後，選取 [安裝]。 

### <a name="add-the-google-services-json-file"></a>新增 Google Services JSON 檔案

1. 將您從 Google Firebase 主控台下載的 **google-services.json** 複製到專案資料夾。
2. 將 **google-services.json** 新增至專案。
3. 在 [方案總管] 視窗中選取 **google-services.json**。
4. 在 [屬性] 窗格中，將 [建置動作] 設定為 **GoogleServicesJson**。 如果您未看到 **GoogleServicesJson**，請關閉 Visual Studio 再加以重新啟動，並重新開啟專案，然後重試。 

      ![GoogleServicesJson 建置動作](./media/partner-xamarin-notification-hubs-android-get-started/google-services-json-build-action.png)

### <a name="set-up-notification-hubs-in-your-project"></a>在專案中設定通知中樞

#### <a name="registering-with-firebase-cloud-messaging"></a>向 Firebase 雲端傳訊進行註冊

開啟 **AndroidManifest.xml** 檔案，並將下列的 `<receiver>` 元素插入 `<application>` 元素：

        <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
        <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
          <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
            <category android:name="${applicationId}" />
          </intent-filter>
        </receiver>

1. 收集您的 Android 應用程式和通知中樞的下列資訊：
   
   * **接聽連接字串**：在 [Azure 入口網站]的儀表板上，選擇 [檢視連接字串]。 複製此值的 *DefaultListenSharedAccessSignature* 連線字串。
   * **中樞名稱**：您的中樞在 [Azure 入口網站]中的名稱。 例如， *mynotificationhub2*。
     
2. 在 [方案總管] 視窗中以滑鼠右鍵按一下您的**專案**，指向 [新增]，然後選取 [類別]。 
4. 為您的 Xamarin 專案建立 **Constants.cs** 類別，並定義類別中的下列常數值。 以您的值取代預留位置。
    
    ```csharp
        public static class Constants
        {
           public const string ListenConnectionString = "<Listen connection string>";
           public const string NotificationHubName = "<hub name>";
        }
    ```
3. 在 **MainActivity.cs**中新增下列 using 陳述式：
   
    ```csharp
        using Android.Util;
    ```
4. 將執行個體變數新增至 **MainActivity.cs**，以用來在應用程式執行時顯示警示對話方塊：
   
    ```csharp
        public const string TAG = "MainActivity";
    ```
5. 在 **MainActivity.cs** 中，將下列程式碼新增至 `base.OnCreate(savedInstanceState)` 後面的 `OnCreate`：

    ```csharp   
        if (Intent.Extras != null)
        {
            foreach (var key in Intent.Extras.KeySet())
            {
                if(key!=null)
                {
                    var value = Intent.Extras.GetString(key);
                    Log.Debug(TAG, "Key: {0} Value: {1}", key, value);
                }
            }
        }
    ```
7. 比照您建立 **Constants** 類別的方式，建立新的類別 **MyFirebaseIIDService**。 
8. 在 **MyFirebaseIIDService.cs** 中新增下列 using 陳述式：
   
    ```csharp
    using Android.App;
    using Android.Util;
    using WindowsAzure.Messaging;
    using Firebase.Iid;
    ```

9. 在 **MyFirebaseIIDService.cs** 中新增下列 **class** 宣告，並讓您的類別繼承自 **FirebaseInstanceIdService**：
   
    ```csharp
        [Service]
        [IntentFilter(new[] { "com.google.firebase.INSTANCE_ID_EVENT" })]
        public class MyFirebaseIIDService : FirebaseInstanceIdService
    ```
10. 在 **MyFirebaseIIDService.cs** 中，新增下列程式碼：
   
    ```csharp
        const string TAG = "MyFirebaseIIDService";
        NotificationHub hub;

        public override void OnTokenRefresh()
        {
            var refreshedToken = FirebaseInstanceId.Instance.Token;
            Log.Debug(TAG, "FCM token: " + refreshedToken);
            SendRegistrationToServer(refreshedToken);
        }

        void SendRegistrationToServer(string token)
        {
            // Register with Notification Hubs
            hub = new NotificationHub(Constants.NotificationHubName,
                                      Constants.ListenConnectionString, this);

            var tags = new List<string>() { };
            var regID = hub.Register(token, tags.ToArray()).RegistrationId;

            Log.Debug(TAG, $"Successful registration of ID {regID}");
        }
    ```
11. 為專案建立另一個新類別，將其命名為 **MyFirebaseMessagingService**。
12. 在 **MyFirebaseMessagingService.cs** 中新增下列 using 陳述式。
    
    ```csharp
        using Android.App;
        using Android.Util;
        using Firebase.Messaging;
    ```
13. 於 class 宣告之上新增下列內容，並讓您的類別繼承自 **FirebaseMessagingService**：
    
    ```csharp
        [Service]
        [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
        public class MyFirebaseMessagingService : FirebaseMessagingService
    ```    
14. 在 **MyFirebaseMessagingService.cs** 中新增下列程式碼：
    
    ```csharp
        const string TAG = "MyFirebaseMsgService";
        public override void OnMessageReceived(RemoteMessage message)
        {
            Log.Debug(TAG, "From: " + message.From);
            if(message.GetNotification()!= null)
            {
                //These is how most messages will be received
                Log.Debug(TAG, "Notification Message Body: " + message.GetNotification().Body);
                SendNotification(message.GetNotification().Body);
            }
            else 
            {
                //Only used for debugging payloads sent from the Azure portal
                SendNotification(message.Data.Values.First());

            }

        }

        void SendNotification(string messageBody)
        {
            var intent = new Intent(this, typeof(MainActivity));
            intent.AddFlags(ActivityFlags.ClearTop);
            var pendingIntent = PendingIntent.GetActivity(this, 0, intent, PendingIntentFlags.OneShot);

            var notificationBuilder = new Notification.Builder(this)
                        .SetContentTitle("FCM Message")
                        .SetSmallIcon(Resource.Drawable.ic_launcher)
                        .SetContentText(messageBody)
                        .SetAutoCancel(true)
                        .SetContentIntent(pendingIntent);

            var notificationManager = NotificationManager.FromContext(this);

            notificationManager.Notify(0, notificationBuilder.Build());
        }
    ```
15. **建置**您的專案。 
16. 在裝置或載入的模擬器上**執行**您的應用程式

## <a name="send-test-notification-from-the-azure-portal"></a>從 Azure 入口網站傳送測試通知
您可以在 [Azure 入口網站] 中，使用 [測試傳送] 選項測試應用程式能否接收通知。 它會將測試推播通知傳送至您的裝置。

![Azure 入口網站 - 測試傳送](media/partner-xamarin-notification-hubs-android-get-started/send-test-notification.png)

推播通知通常會在後端服務 (例如行動服務) 中或透過相容程式庫的 ASP.NET 傳送。 如果您的後端無法使用程式庫，您也可以直接使用 REST API 來傳送通知訊息。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已將廣播通知傳送給向後端註冊的所有 Android 裝置。 若要了解如何將通知推送至特定 Android 裝置，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[將通知推送至特定裝置](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)


<!-- Anchors. -->
[Enable Google Cloud Messaging]: #register
[Configure your Notification Hub]: #configure-hub
[Connecting your app to the Notification Hub]: #connecting-app
[Run your app with the emulator]: #run-app
[Send notifications from your back-end]: #send
[Next steps]:#next-steps

<!-- Images. -->

[11]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-configure-android.png

[13]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app1.png
[15]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app3.png

[18]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app7.png
[19]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app8.png

[20]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-android-toast.png
[22]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project1.png
[23]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project2.png
[24]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-xamarin-android-app-options.png
[25]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-google-services-json.png

[30]: ./media/notification-hubs-android-get-started/notification-hubs-test-send.png


<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-android/#create-new-service
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[Visual Studio 搭配 Xamarin]: https://docs.microsoft.com/visualstudio/install/install-visual-studio
[Visual Studio for Mac]: https://www.visualstudio.com/vs/visual-studio-mac/

[Azure 入口網站]: https://portal.azure.com/
[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Android]: http://msdn.microsoft.com/library/dn282661.aspx

[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[GitHub]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/Xamarin/GetStartedXamarinAndroid
