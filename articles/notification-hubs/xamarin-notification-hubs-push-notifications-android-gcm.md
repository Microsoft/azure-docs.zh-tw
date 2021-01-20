---
title: 使用 Azure 通知中樞將推播通知傳送至 Xamarin.Android 應用程式 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure 通知中樞將推播通知傳送至 Xamarin Android 應用程式。
author: sethmanheim
manager: femila
editor: jwargo
services: notification-hubs
documentationcenter: xamarin
ms.assetid: 0be600fe-d5f3-43a5-9e5e-3135c9743e54
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/12/2021
ms.author: matthewp
ms.reviewer: jowargo
ms.lastreviewed: 08/01/2019
ms.openlocfilehash: e7d4206de1e097c30e9f5e96bbd935e94892ce0e
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98221029"
---
# <a name="tutorial-send-push-notifications-to-xamarinandroid-apps-using-notification-hubs"></a>教學課程：使用通知中樞將推播通知傳送至 Xamarin.Android 應用程式

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>概觀

本教學課程示範如何使用 Azure 通知中樞將推播通知傳送至 Xamarin.Android 應用程式。 您會建立可使用 Firebase 雲端通訊 (FCM) 接收推播通知的空白 Xamarin.Android 應用程式。 您會使用通知中樞，將推播通知廣播到所有執行您的應用程式的裝置。 [NotificationHubs 應用程式](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/Xamarin/GetStartedXamarinAndroid)範例中提供完成的程式碼。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立 Firebase 專案並啟用 Firebase 雲端通訊
> * 建立通知中樞
> * 建立 Xamarin.Android 應用程式，並將其連線至通知中樞
> * 從 Azure 入口網站傳送測試通知

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費 Azure 帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [Visual Studio 搭配 Xamarin] (在 Windows 上) 或 [Visual Studio for Mac] (在 OS X 上)。
* 有效的 Google 帳戶

## <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging"></a>建立 Firebase 專案並啟用 Firebase 雲端通訊

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging-xamarin.md)]

## <a name="create-a-notification-hub"></a>建立通知中樞

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcmfcm-settings-for-the-notification-hub"></a>設定通知中樞的 GCM/FCM 設定

1. 選取左側功能表上的 [設定] 區段中的 [Google (GCM/FCM)]。
2. 輸入您從 Google Firebase 主控台記下的 **伺服器金鑰**。
3. 在工具列上選取 [儲存]。

    ![Azure 入口網站中的通知中樞螢幕擷取畫面，並以紅色框線醒目提示 Google G C M F C M 選項。](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

您的通知中樞已設定成使用 FCM，而且您已擁有可用來註冊應用程式以接收通知和傳送推播通知的連接字串。

## <a name="create-a-xamarinandroid-app-and-connect-it-to-notification-hub"></a>建立 Xamarin.Android 應用程式，並將其連線至通知中樞

### <a name="create-visual-studio-project-and-add-nuget-packages"></a>建立 Visual Studio 專案並新增 NuGet 套件

> [!NOTE]
> 本教學課程所述的步驟適用於 Visual Studio 2017。 

1. 在 Visual Studio 中，開啟 [檔案] 功能表，選取 [新增]，然後選取 [專案]。 在 [新增專案] 視窗中，執行下列步驟：
    1. 依序展開 [已安裝] 和 [Visual C#]，然後按一下 [Android]。
    2. 從清單中選取 [Android 應用程式 (Xamarin)]。
    3. 輸入專案的 [名稱]  。
    4. 選取專案的 [位置]  。
    5. 選取 [確定]

        ![[新增專案] 對話方塊](./media/partner-xamarin-notification-hubs-android-get-started/new-project-dialog-new.png)
2. 在 [新的 Android 應用程式] 對話方塊中，選取 [空白應用程式]，然後選取 [確定]。

    ![醒目提示空白應用程式範本的螢幕擷取畫面。](./media/partner-xamarin-notification-hubs-android-get-started/new-android-app-dialog.png)
3. 在 [方案總管] 視窗中展開 [屬性]，然後按一下 **AndroidManifest.xml**。 更新套件名稱，使其符合您在 Google Firebase 主控台中將 Firebase 雲端通訊新增至專案時輸入的套件名稱。

    ![GCM 中的套件名稱](./media/partner-xamarin-notification-hubs-android-get-started/package-name-gcm.png)
4. 依照下列步驟，將專案的目標 Android 版本設定為 **android 10.0** ： 
    1. 以滑鼠右鍵按一下專案，然後選取 [屬性]。 
    1. 針對 [ **使用 Android 版本編譯： (目標 framework)** ] 欄位，選取 [ **Android 10.0**]。 
    1. 在訊息方塊上選取 [是]，繼續變更目標 Framework。
1. 遵循下列步驟將必要的 NuGet 套件新增至專案：
    1. 以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件...]。
    1. 切換至 [已安裝] 索引標籤，選取 [Xamarin.Android.Support.Design]，然後選取右窗格中的 [更新] 將套件更新為最新版本。
    1. 切換至 [瀏覽] 索引標籤。搜尋 **Xamarin.GooglePlayServices.Base**。 在結果清單中選取 **Xamarin.GooglePlayServices.Base**。 然後，選取 [安裝]。

        ![Google Play 服務 NuGet](./media/partner-xamarin-notification-hubs-android-get-started/google-play-services-nuget.png)
    6. 在 [NuGet 套件管理員] 視窗中，搜尋 **Xamarin.Firebase.Messaging**。 在結果清單中選取 **Xamarin.Firebase.Messaging**。 然後，選取 [安裝]。
    7. 現在，請搜尋 **Xamarin.Azure.NotificationHubs.Android**。 在結果清單中選取 **Xamarin.Azure.NotificationHubs.Android**。 然後，選取 [安裝]。

### <a name="add-the-google-services-json-file"></a>新增 Google Services JSON 檔案

1. 將您從 Google Firebase 主控台下載的 `google-services.json` 複製到專案資料夾。
2. 將 `google-services.json` 新增至專案。
3. 在 [方案總管] 視窗中選取 `google-services.json`。
4. 在 [屬性] 窗格中，將 [建置動作] 設定為 **GoogleServicesJson**。 如果您未看到 **GoogleServicesJson**，請關閉 Visual Studio 再加以重新啟動，並重新開啟專案，然後重試。

    ![GoogleServicesJson 建置動作](./media/partner-xamarin-notification-hubs-android-get-started/google-services-json-build-action.png)

### <a name="set-up-notification-hubs-in-your-project"></a>在專案中設定通知中樞

#### <a name="registering-with-firebase-cloud-messaging"></a>向 Firebase 雲端傳訊進行註冊

1. 如果您要從 Google 雲端通訊遷移至 Firebase，您的專案檔案 `AndroidManifest.xml` 可能會包含過期的 GCM 設定，這可能會導致通知重複。 編輯檔案，並移除區段內的下列幾行 `<application>` （如果有的話）：

    ```xml
    <receiver
        android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver"
        android:exported="false" />
    <receiver
        android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver"
        android:exported="true"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
            <category android:name="${applicationId}" />
        </intent-filter>
    </receiver>
    ```

2. 在 **應用程式元素前面** 新增下列陳述式。

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    ```

3. 收集您的 Android 應用程式和通知中樞的下列資訊：

   * **接聽連接字串**：在 [Azure 入口網站]的儀表板上，選擇 [檢視連接字串]。 複製此值得 `DefaultListenSharedAccessSignature` 連接字串。
   * **中樞名稱**：您的中樞在 [Azure 入口網站]中的名稱。 例如， *mynotificationhub2*。
4. 在 [方案總管] 視窗中，以滑鼠右鍵按一下您的 **專案**，選取 [新增]，然後選取 [類別]。
5. 為您的 Xamarin 專案建立 `Constants.cs` 類別，並定義類別中的下列常數值。 以您的值取代預留位置。

    ```csharp
    public static class Constants
    {
        public const string ListenConnectionString = "<Listen connection string>";
        public const string NotificationHubName = "<hub name>";
    }
    ```

6. 將下列 using 陳述式新增至 `MainActivity.cs`：

    ```csharp
    using Azure.Messaging.NotificationHubs;
    ```

7. 將下列屬性新增至 >mainactivity 類別：

    ```csharp
    internal static readonly string CHANNEL_ID = "my_notification_channel";

8. In `MainActivity.cs`, add the following code to `OnCreate` after `base.OnCreate(savedInstanceState)`:

    ```csharp
    // Listen for push notifications
    NotificationHub.SetListener(new AzureListener());

    // Start the SDK
    NotificationHub.Start(this.Application, HubName, ConnectionString);
    ```

9. 將名為 `AzureListener` 的類別新增至專案。
10. 將下列 using 陳述式新增至 `AzureListener.cs`。

    ```csharp
    using Android.Content;
    using WindowsAzure.Messaging.NotificationHubs;
    ```

11. 在您的類別宣告上方新增下列內容，並讓您的類別繼承自 `Java.Lang.Object` 並執行 `INotificationListener` ：

    ```csharp
    public class AzureListener : Java.Lang.Object, INotificationListener
    ```

12. 將下列程式碼新增至 `MyFirebaseMessagingService` 類別內，以處理所接收的訊息。

    ```csharp
        public void OnPushNotificationReceived(Context context, INotificationMessage message)
        {
            var intent = new Intent(this, typeof(MainActivity));
            intent.AddFlags(ActivityFlags.ClearTop);
            var pendingIntent = PendingIntent.GetActivity(this, 0, intent, PendingIntentFlags.OneShot);
    
            var notificationBuilder = new NotificationCompat.Builder(this, MainActivity.CHANNEL_ID);
    
            notificationBuilder.SetContentTitle(message.Title)
                        .SetSmallIcon(Resource.Drawable.ic_launcher)
                        .SetContentText(message.Body)
                        .SetAutoCancel(true)
                        .SetShowWhen(false)
                        .SetContentIntent(pendingIntent);
    
            var notificationManager = NotificationManager.FromContext(this);
    
            notificationManager.Notify(0, notificationBuilder.Build());
        }
    ```

1. **建置** 您的專案。
1. 在裝置或載入的模擬器上 **執行** 您的應用程式

## <a name="send-test-notification-from-the-azure-portal"></a>從 Azure 入口網站傳送測試通知

您可以在 [Azure 入口網站] 中，使用 [測試傳送] 選項測試應用程式能否接收通知。 它會將測試推播通知傳送至您的裝置。

![Azure 入口網站 - 測試傳送](media/partner-xamarin-notification-hubs-android-get-started/send-test-notification.png)

推播通知通常會在後端服務 (例如行動服務) 中或透過相容程式庫的 ASP.NET 傳送。 如果您的後端無法使用程式庫，您也可以直接使用 REST API 來傳送通知訊息。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將廣播通知傳送給向後端註冊的所有 Android 裝置。 若要了解如何將通知推送至特定 Android 裝置，請繼續進行下列教學課程：

> [!div class="nextstepaction"]
>[將通知推送至特定裝置](push-notifications-android-specific-devices-firebase-cloud-messaging.md)

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
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-android/#create-new-service
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[Visual Studio 搭配 Xamarin]: /visualstudio/install/install-visual-studio
[Visual Studio for Mac]: https://www.visualstudio.com/vs/visual-studio-mac/
[Azure 入口網站]: https://portal.azure.com/
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Android]: /previous-versions/azure/dn282661(v=azure.100)
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[GitHub]: https://github.com/Azure/azure-notificationhubs-android