---
title: 使用 Azure 通知中樞和 Google 雲端通訊將通知推播至 Android 應用程式 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 通知中樞和 Google Firebase 雲端通訊，將通知推送至 Android 裝置。
services: notification-hubs
documentationcenter: android
keywords: 推播通知,推播通知,android 推播通知
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 8268c6ef-af63-433c-b14e-a20b04a0342a
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/05/2018
ms.author: dimazaid
ms.openlocfilehash: 1e9ed9d3500b112f98492e6a85d02a1f1843c7c3
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365572"
---
# <a name="tutorial-push-notifications-to-android-devices-by-using-azure-notification-hubs-and-google-cloud-messaging"></a>教學課程：使用 Azure 通知中樞和 Google 雲端通訊將通知推播至 Android 裝置
[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>概觀
本教學課程說明如何使用 Azure 通知中樞傳送推播通知到 Android 應用程式。
您將建立可使用 Google 雲端通訊 (GCM) 接收推播通知的空白 Android 應用程式。

> [!IMPORTANT]
> 本主題示範使用 Google 雲端通訊 (GCM) 的推播通知。 如果您是使用 Google 的 Firebase 雲端通訊 (FCM)，請參閱 [使用 Azure 通知中樞和 FCM 將推播通知傳送至 Android](notification-hubs-android-push-notification-google-fcm-get-started.md)。

您可以從 [此處](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStarted)的 GitHub 下載本教學課程的完整程式碼。

在本教學課程中，您會執行下列動作： 

> [!div class="checklist"]
> * 建立支援 Google 雲端通訊的專案。
> * 建立通知中樞
> * 將您的應用程式連接到通知中樞
> * 測試應用程式

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。 
- [Android Studio](http://go.microsoft.com/fwlink/?LinkId=389797).

## <a name="creating-a-project-that-supports-google-cloud-messaging"></a>建立支援 Google 雲端通訊的專案
[!INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]

## <a name="create-a-notification-hub"></a>建立通知中樞
[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-setting-for-the-notification-hub"></a>設定通知中樞的 GCM 設定

1. 在 [通知設定] 中選取 [Google] \(GCM\)。 
2. 輸入您從 Google Cloud Console 中取得的 **API 金鑰**。 
3. 在工具列上選取 [儲存]。 

    ![Azure 通知中樞 - Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

現在已將您的通知中樞設定成使用 GCM，而且您已擁有可用來註冊應用程式以接收和傳送推播通知的連接字串。

## <a id="connecting-app"></a>將您的應用程式連接到通知中樞
### <a name="create-a-new-android-project"></a>建立新的 Android 專案
1. 在 Android Studio 中，啟動新的 Android Studio 專案。
   
   ![Android Studio - 新增專案][13]
2. 選擇 [電話和平板電腦] 板型規格和您要支援的 [Minimum SDK]。 然後按 [下一步] 。
   
   ![Android Studio - 專案建立工作流程][14]
3. 為主要活動選擇 [空白活動]，並按 [下一步]，然後按一下 [完成]。

### <a name="add-google-play-services-to-the-project"></a>新增 Google Play 服務至專案
[!INCLUDE [Add Play Services](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

### <a name="adding-azure-notification-hubs-libraries"></a>新增 Azure 通知中樞程式庫
1. 在 **app** 的 `Build.Gradle` 檔案中，於 **dependencies** 區段中新增下列數行。
   
    ```java
        compile 'com.microsoft.azure:notification-hubs-android-sdk:0.4@aar'
        compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'
    ```
2. 加入下列儲存機制到 **dependencies** 一節之後。
   
    ```java
    repositories {
        maven {
            url "http://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

### <a name="updating-the-androidmanifestxml"></a>更新 AndroidManifest.xml。
1. 若要支援 GCM，請在程式碼中實作執行個體識別碼接聽程式服務，以便使用 [Google 的執行個體識別碼 API](https://developers.google.com/instance-id/) 來[取得註冊權杖](https://developers.google.com/cloud-messaging/android/client#sample-register)。 在本教學課程中，類別名稱為 `MyInstanceIDService`。 
   
    將下列服務定義新增至 AndroidManifest.xml 檔案的 `<application>` 標籤內。 以 `AndroidManifest.xml` 檔案頂端顯示的實際套件名稱取代 `<your package>` 預留位置。
  
    ```xml 
    <service android:name="<your package>.MyInstanceIDService" android:exported="false">
        <intent-filter>
            <action android:name="com.google.android.gms.iid.InstanceID"/>
        </intent-filter>
    </service>
    ```
2. 一旦應用程式從執行個體識別碼 API 收到 GCM 註冊權杖，就會使用權杖來 [向 Azure 通知中樞註冊](notification-hubs-push-notification-registration-management.md)。 系統會使用名為 `RegistrationIntentService` 的 `IntentService` 在背景完成註冊。 此服務負責[重新整理 GCM 註冊權杖](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)。
   
    將下列服務定義新增至 AndroidManifest.xml 檔案的 `<application>` 標籤內。 以 `AndroidManifest.xml` 檔案頂端顯示的實際套件名稱取代 `<your package>` 預留位置。 
   
    ```xml
    <service
        android:name="<your package>.RegistrationIntentService"
        android:exported="false">
    </service>
    ```
3. 定義要接收通知的接收者。 將下列接收者定義新增至 AndroidManifest.xml 檔案的 `<application>` 標籤內。 以 `AndroidManifest.xml` 檔案頂端顯示的實際套件名稱取代 `<your package>` 預留位置。
   
    ```xml
    <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <category android:name="<your package name>" />
        </intent-filter>
    </receiver>
    ```
4. 在 `</application>` 標籤下面新增下列必要的 GCM 權限。 請以 `AndroidManifest.xml` 檔案頂端顯示的套件名稱取代 `<your package>`。
   
    如需這些權限的詳細資訊，請參閱 [設定適用於 Android 的 GCM 用戶端應用程式](https://developers.google.com/cloud-messaging/android/client#manifest)。
   
    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>
    ```
### <a name="adding-code"></a>加入程式碼
1. 在 [專案檢視] 中，展開 [app] > [src] > [main] > [java]。 以滑鼠右鍵按一下 **java** 底下您的套件資料夾，並按一下 [新增]，然後按一下 [Java 類別]。 新增名為 `NotificationSettings` 的新類別。 
   
    ![Android Studio - 新增 Java 類別][6]
   
    請在 `NotificationSettings` 類別的下列程式碼中更新這三個預留位置：
   
   * **SenderId**：您稍早在 [Google Cloud Console](http://cloud.google.com/console)中取得的專案編號。
   * **HubListenConnectionString**：中樞的 **DefaultListenAccessSignature** 連接字串。 在 [Azure 入口網站]中，按一下中樞 [設定] 頁面上的 [存取原則]，即可複製該連接字串。
   * **HubName**︰使用出現在 [Azure 入口網站]中樞頁面中的通知中樞名稱。
     
     `NotificationSettings` 程式碼︰
     
    ```java
    public class NotificationSettings {
    
        public static String SenderId = "<Your project number>";
        public static String HubName = "<Your HubName>";
        public static String HubListenConnectionString = "<Your default listen connection string>";
    }
    ```
2. 新增另一個名為 `MyInstanceIDService` 的新類別。 此類別是執行個體識別碼接聽程式服務實作。
   
    此類別的程式碼會呼叫 `IntentService` 以在背景[重新整理 GCM 權杖](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)。
   
    ```java
    import android.content.Intent;
    import android.util.Log;
    import com.google.android.gms.iid.InstanceIDListenerService;

    public class MyInstanceIDService extends InstanceIDListenerService {

        private static final String TAG = "MyInstanceIDService";

        @Override
        public void onTokenRefresh() {

            Log.i(TAG, "Refreshing GCM Registration Token");

            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    };
    ```
1. 將另一個新類別新增至名為 `RegistrationIntentService`的專案。 此類別會實作 `IntentService`，可處理[重新整理 GCM 權杖](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)和[向通知中樞註冊](notification-hubs-push-notification-registration-management.md)。
   
    針對此類別使用下列程式碼。
   
    ```java
    import android.app.IntentService;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.preference.PreferenceManager;
    import android.util.Log;

    import com.google.android.gms.gcm.GoogleCloudMessaging;
    import com.google.android.gms.iid.InstanceID;
    import com.microsoft.windowsazure.messaging.NotificationHub;

    public class RegistrationIntentService extends IntentService {

        private static final String TAG = "RegIntentService";

        private NotificationHub hub;

        public RegistrationIntentService() {
            super(TAG);
        }

        @Override
        protected void onHandleIntent(Intent intent) {        
            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            String resultString = null;
            String regID = null;

            try {
                InstanceID instanceID = InstanceID.getInstance(this);
                String token = instanceID.getToken(NotificationSettings.SenderId,
                        GoogleCloudMessaging.INSTANCE_ID_SCOPE);        
                Log.i(TAG, "Got GCM Registration Token: " + token);

                // Storing the registration id that indicates whether the generated token has been
                // sent to your server. If it is not stored, send the token to your server,
                // otherwise your server should have already received the token.
                if ((regID=sharedPreferences.getString("registrationID", null)) == null) {        
                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.i(TAG, "Attempting to register with NH using token : " + token);

                    regID = hub.register(token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1", "tag2").getRegistrationId();

                    resultString = "Registered Successfully - RegId : " + regID;
                    Log.i(TAG, resultString);        
                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                } else {
                    resultString = "Previously Registered Successfully - RegId : " + regID;
                }
            } catch (Exception e) {
                Log.e(TAG, resultString="Failed to complete token refresh", e);
                // If an exception happens while fetching the new token or updating the registration data
                // on a third-party server, this ensures that we'll attempt the update at a later time.
            }

            // Notify UI that registration has completed.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(resultString);
            }
        }
    }
    ```
2. 在 `MainActivity` 類別中，在類別的開頭新增下列 `import` 陳述式。

    ```java   
    import com.google.android.gms.common.ConnectionResult;
    import com.google.android.gms.common.GoogleApiAvailability;
    import com.google.android.gms.gcm.*;
    import com.microsoft.windowsazure.notifications.NotificationsManager;
    import android.util.Log;
    import android.widget.TextView;
    import android.widget.Toast;
    import android.content.Intent;
    ```
3. 在類別頂端新增下列 Private 成員。 此程式碼會[檢查 Google 所建議的 Google Play Services 可用性](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk)。
   
    ```java
    public static MainActivity mainActivity;
    public static Boolean isVisible = false;    
    private GoogleCloudMessaging gcm;
    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    private static final String TAG = "MainActivity";
    ```
4. 在 `MainActivity` 類別中，將下列方法新增至 Google Play 服務的可用性。 
   
    ```java
    /**
        * Check the device to make sure it has the Google Play Services APK. If
        * it doesn't, display a dialog that allows users to download the APK from
        * the Google Play Store or enable it in the device's system settings.
        */
    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
                        .show();
            } else {
                Log.i(TAG, "This device is not supported by Google Play Services.");
                ToastNotify("This device is not supported by Google Play Services.");
                finish();
            }
            return false;
        }
        return true;
    }
    ```
5. 在 `MainActivity` 類別中加入下列程式碼，以在呼叫 `IntentService` 之前檢查 Google Play Services，進而取得 GCM 註冊權杖並向通知中樞註冊。
   
    ```java
    public void registerWithNotificationHubs()
    {
        Log.i(TAG, " Registering with Notification Hubs");

        if (checkPlayServices()) {
            // Start IntentService to register this application with GCM.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    }
    ```
6. 在 `MainActivity` 類別的 `OnCreate` 方法中，加入下列程式碼以便在活動建立時開始註冊程序。
   
    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainActivity = this;
        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
        registerWithNotificationHubs();
    }
    ```
7. 將上述其他方法新增至 `MainActivity` ，以驗證應用程式狀態及報告您的應用程式狀態。
   
    ```java
    @Override
    protected void onStart() {
        super.onStart();
        isVisible = true;
    }

    @Override
    protected void onPause() {
        super.onPause();
        isVisible = false;
    }

    @Override
    protected void onResume() {
        super.onResume();
        isVisible = true;
    }

    @Override
    protected void onStop() {
        super.onStop();
        isVisible = false;
    }

    public void ToastNotify(final String notificationMessage) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
                TextView helloText = (TextView) findViewById(R.id.text_hello);
                helloText.setText(notificationMessage);
            }
        });
    }
    ```
8. `ToastNotify` 方法會使用 *"Hello World"* `TextView` 控制項持續在應用程式中報告狀態和通知。 在 activity_main.xml 配置中，為該控制項加入下列識別碼。
   
    ```
    android:id="@+id/text_hello"
    ```
9. 為在 AndroidManifest.xml 中定義的接收者新增一個子類別。 將另一個新類別新增至名為 `MyHandler`的專案。
10. 在 `MyHandler.java` 頂端新增下列 import 陳述式：
    
    ```java
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.support.v4.app.NotificationCompat;
    import com.microsoft.windowsazure.notifications.NotificationsHandler;
    import android.net.Uri;
    import android.media.RingtoneManager;
    ```
11. 在 `MyHandler` 類別中新增下列程式碼，使其成為 `com.microsoft.windowsazure.notifications.NotificationsHandler` 的子類別。
    
    此程式碼會覆寫 `OnReceive` 方法，以便讓處理常式報告所收到的通知。 處理常式也會使用 `sendNotification()` 方法，將推播通知傳送給 Android 通知管理員。 當應用程式不在執行中並收到通知時，應該執行 `sendNotification()` 方法。
    
    ```java
    public class MyHandler extends NotificationsHandler {
        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;

        @Override
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String nhMessage = bundle.getString("message");
            sendNotification(nhMessage);
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(nhMessage);
            }
        }

        private void sendNotification(String msg) {

            Intent intent = new Intent(ctx, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);

            mNotificationManager = (NotificationManager)
                    ctx.getSystemService(Context.NOTIFICATION_SERVICE);

            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                    intent, PendingIntent.FLAG_ONE_SHOT);

            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder mBuilder =
                    new NotificationCompat.Builder(ctx)
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setContentTitle("Notification Hub Demo")
                            .setStyle(new NotificationCompat.BigTextStyle()
                                    .bigText(msg))
                            .setSound(defaultSoundUri)
                            .setContentText(msg);

            mBuilder.setContentIntent(contentIntent);
            mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
        }
    }
    ```
12. 在 Android Studio 的功能表列上，按一下 [建置] > [重新建置專案]，確保您的程式碼中未沒有任何錯誤。

## <a name="testing-your-app"></a>測試應用程式
### <a name="run-the-mobile-application"></a>執行行動應用程式
1. 執行 app，並注意已回報註冊成功的註冊識別碼。
   
      ![在 Android 上測試 - 通道註冊][18]
2. 輸入通知訊息，以傳送給已向中心註冊的所有 Android 裝置。
   
      ![在 Android 上測試 - 傳送訊息][19]

3. 按 [ **傳送通知**]。 任何執行應用程式的裝置都會顯示含推播通知訊息的 `AlertDialog` 執行個體。 未執行應用程式但先前已註冊推播通知的裝置，將會收到 Android 通知管理員的通知。 從左上角往下撥動，即可檢視通知訊息。
   
      ![在 Android 上測試 - 通知][21]

### <a name="test-send-push-notifications-from-the-azure-portal"></a>測試從 Azure 入口網站傳送推播通知
透過 [Azure 入口網站]傳送推播通知，即可在應用程式中測試推播通知的接收。 

1. 在 [疑難排解] 區段中，選取 [測試傳送]。 
2. 針對 [平台]，選取 [Android]。
3. 選取 [傳送] 來傳送測試通知。 
4. 確認您可在 Android 裝置上看到通知訊息。 

    ![Azure 通知中樞 - 測試傳送](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

#### <a name="push-notifications-in-the-emulator"></a>在模擬器中測試推播通知
如果您要在模擬器中測試推播通知，請確定您的模擬器映像支援您為應用程式選擇的 Google API 層級。 如果您的映像不支援原生 Google API，最後會發生 **SERVICE\_NOT\_AVAILABLE** 例外狀況。

除此之外，請確定已將 Google 帳戶新增至執行中模擬器的 [設定] > [帳戶] 之下。 否則，嘗試向 GCM 註冊可能會導致 **AUTHENTICATION\_FAILED** 例外狀況。


## <a name="optional-send-push-notifications-directly-from-the-app"></a>(選擇性) 從應用程式直接傳送推播通知
一般來說，您會使用後端伺服器傳送通知。 在某些情況下，您可能希望能夠直接從用戶端應用程式傳送推播通知。 本節說明如何使用 [Azure 通知中樞 REST API](https://msdn.microsoft.com/library/azure/dn223264.aspx)從用戶端傳送通知。

1. 在 [Android Studio 專案檢視] 中，展開 [App] > [src] > [main] > [res] > [layout]。 開啟 `activity_main.xml` 配置檔案，然後按一下 [文字] 索引標籤以更新檔案的文字內容。 以下列程式碼進行更新，這會加入新的 `Button` 和 `EditText` 控制項，以便將推播通知訊息傳送至通知中樞。 在底部將此程式碼加在 `</RelativeLayout>`之前。
   
    ```xml
    <Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/send_button"
    android:id="@+id/sendbutton"
    android:layout_centerVertical="true"
    android:layout_centerHorizontal="true"
    android:onClick="sendNotificationButtonOnClick" />

    <EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/editTextNotificationMessage"
    android:layout_above="@+id/sendbutton"
    android:layout_centerHorizontal="true"
    android:layout_marginBottom="42dp"
    android:hint="@string/notification_message_hint" />
    ```
2. 在 [Android Studio 專案檢視] 中，展開 [App] > [src] > [main] > [res] > [values]。 開啟 `strings.xml` 檔案並加入新的 `Button` 和 `EditText` 控制項所參考的字串值。 在檔案底部，將下列幾行加在 `</resources>` 之前。

    ```xml   
    <string name="send_button">Send Notification</string>
    <string name="notification_message_hint">Enter notification message text</string>
    ```
3. 在 `NotificationSetting.java` 檔案中，將下列設定新增至 `NotificationSettings` 類別。
   
    使用中樞的 **DefaultFullSharedAccessSignature** 連接字串更新 `HubFullAccess`。 按一下通知中樞 [設定] 頁面上的 [存取原則]，即可從 [Azure 入口網站]複製此連接字串。
   
    ```java
    public static String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";
    ```
4. 在 `MainActivity.java` 檔案中，在檔案的開頭新增下列 `import` 陳述式。
   
    ```java
    import java.io.BufferedOutputStream;
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    import java.io.OutputStream;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.net.URLEncoder;
    import javax.crypto.Mac;
    import javax.crypto.spec.SecretKeySpec;
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;
    ```
5. 在 `MainActivity.java` 檔案中，將下列成員加在 `MainActivity` 類別的最上方。    
   
    ```java
    private String HubEndpoint = null;
    private String HubSasKeyName = null;
    private String HubSasKeyValue = null;
    ```
6. 建立軟體存取簽章 (SaS) 權杖來驗證 POST 要求，以將訊息傳送至您的通知中樞。 剖析連接字串中的金鑰資料，然後建立 [一般概念](http://msdn.microsoft.com/library/azure/dn495627.aspx) REST API 參考中所提的 SaS 權杖。 下列程式碼是範例實作。
   
    在 `MainActivity.java` 中，將下列方法加入至 `MainActivity` 類別，以剖析連接字串。
   
    ```java
    /**
        * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx
        * to parse the connection string so a SaS authentication token can be
        * constructed.
        *
        * @param connectionString This must be the DefaultFullSharedAccess connection
        *                         string for this example.
        */
    private void ParseConnectionString(String connectionString)
    {
        String[] parts = connectionString.split(";");
        if (parts.length != 3)
            throw new RuntimeException("Error parsing connection string: "
                    + connectionString);

        for (int i = 0; i < parts.length; i++) {
            if (parts[i].startsWith("Endpoint")) {
                this.HubEndpoint = "https" + parts[i].substring(11);
            } else if (parts[i].startsWith("SharedAccessKeyName")) {
                this.HubSasKeyName = parts[i].substring(20);
            } else if (parts[i].startsWith("SharedAccessKey")) {
                this.HubSasKeyValue = parts[i].substring(16);
            }
        }
    }
    ```
7. 在 `MainActivity.java` 中，將下列方法加入至 `MainActivity` 類別，以建立 SaS 驗證權杖。
   
    ```java
    /**
        * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx to
        * construct a SaS token from the access key to authenticate a request.
        *
        * @param uri The unencoded resource URI string for this operation. The resource
        *            URI is the full URI of the Service Bus resource to which access is
        *            claimed. For example,
        *            "http://<namespace>.servicebus.windows.net/<hubName>"
        */
    private String generateSasToken(String uri) {

        String targetUri;
        String token = null;
        try {
            targetUri = URLEncoder
                    .encode(uri.toString().toLowerCase(), "UTF-8")
                    .toLowerCase();

            long expiresOnDate = System.currentTimeMillis();
            int expiresInMins = 60; // 1 hour
            expiresOnDate += expiresInMins * 60 * 1000;
            long expires = expiresOnDate / 1000;
            String toSign = targetUri + "\n" + expires;

            // Get an hmac_sha1 key from the raw key bytes
            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

            // Get an hmac_sha1 Mac instance and initialize with the signing key
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);

            // Compute the hmac on input data bytes
            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

            // Using android.util.Base64 for Android Studio instead of
            // Apache commons codec
            String signature = URLEncoder.encode(
                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

            // Construct authorization string
            token = "SharedAccessSignature sr=" + targetUri + "&sig="
                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
        } catch (Exception e) {
            if (isVisible) {
                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
            }
        }

        return token;
    }
    ```
8. 在 `MainActivity.java` 中，將下列方法新增至 `MainActivity` 類別，以使用內建 REST API 處理 [傳送通知] 按鈕點選，並將推播通知訊息傳送至中樞。
   
    ```java
    /**
        * Send Notification button click handler. This method parses the
        * DefaultFullSharedAccess connection string and generates a SaS token. The
        * token is added to the Authorization header on the POST request to the
        * notification hub. The text in the editTextNotificationMessage control
        * is added as the JSON body for the request to add a GCM message to the hub.
        *
        * @param v
        */
    public void sendNotificationButtonOnClick(View v) {
        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
        final String json = "{\"data\":{\"message\":\"" + notificationText.getText().toString() + "\"}}";

        new Thread()
        {
            public void run()
            {
                try
                {
                    // Based on reference documentation...
                    // http://msdn.microsoft.com/library/azure/dn223273.aspx
                    ParseConnectionString(NotificationSettings.HubFullAccess);
                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
                            "/messages/?api-version=2015-01");

                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();

                    try {
                        // POST request
                        urlConnection.setDoOutput(true);

                        // Authenticate the POST request with the SaS token
                        urlConnection.setRequestProperty("Authorization", 
                            generateSasToken(url.toString()));

                        // Notification format should be GCM
                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");

                        // Include any tags
                        // Example below targets 3 specific tags
                        // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags", 
                        //        "tag1 || tag2 || tag3");

                        // Send notification message
                        urlConnection.setFixedLengthStreamingMode(json.length());
                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
                        bodyStream.write(json.getBytes());
                        bodyStream.close();

                        // Get reponse
                        urlConnection.connect();
                        int responseCode = urlConnection.getResponseCode();
                        if ((responseCode != 200) && (responseCode != 201)) {
                            BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                            String line;
                            StringBuilder builder = new StringBuilder("Send Notification returned " +
                                    responseCode + " : ")  ;
                            while ((line = br.readLine()) != null) {
                                builder.append(line);
                            }

                            ToastNotify(builder.toString());
                        }
                    } finally {
                        urlConnection.disconnect();
                    }
                }
                catch(Exception e)
                {
                    if (isVisible) {
                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
                    }
                }
            }
        }.start();
    }
    ```

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已將廣播通知傳送給向後端註冊的所有 Android 裝置。 若要了解如何將通知推送至特定 Android 裝置，請繼續進行下列教學課程：  

 > [!div class="nextstepaction"] 
 > [將通知推送至特定裝置](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md) 


<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png

[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png

[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png

[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png

[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png

[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png


<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md  
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: http://go.microsoft.com/fwlink/?LinkId=389800
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md
[Azure 入口網站]: https://portal.azure.com
