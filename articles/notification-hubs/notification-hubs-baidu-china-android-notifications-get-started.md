---
title: 透過百度開始使用 Azure 通知中樞 | Microsoft Docs
description: 在本教學課程中，您將了解如何透過百度使用 Azure 通知中樞，將通知推播到 Android 裝置。
services: notification-hubs
documentationcenter: android
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 23bde1ea-f978-43b2-9eeb-bfd7b9edc4c1
ms.service: notification-hubs
ms.devlang: java
ms.topic: hero-article
ms.tgt_pltfrm: mobile-baidu
ms.workload: mobile
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: c680de254c55adaad8cedde6cf4748efd0c2c858
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38700895"
---
# <a name="get-started-with-notification-hubs-using-baidu"></a>透過百度開始使用通知中樞
[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

百度雲端推播是中文的雲端服務，可用於將推播通知傳送至行動裝置。 

由於中國無法使用 Google Play 和 FCM (Firebase Cloud Messaging)，因此必須使用不同的應用程式市集和推送服務。 百度是其中一種方法，且目前由通知中樞所使用。

## <a name="prerequisites"></a>先決條件
本教學課程需要：

* Android SDK (我們假設您使用 Android Studio)，您可從 <a href="http://go.microsoft.com/fwlink/?LinkId=389797">Android 網站</a>下載該套件
* [百度推播 Android SDK]

> [!NOTE]
> 若要完成此教學課程，您必須具備有效的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdocumentation%2Farticles%2Fnotification-hubs-baidu-get-started%2F)。
> 
> 

## <a name="create-a-baidu-account"></a>建立百度帳戶
若要使用百度，您必須有百度帳戶。 如果您已經有帳戶，請登入 [百度入口網站] ，並略過下一個步驟。 否則，請參閱下列有關如何建立百度帳戶的指示。  

1. 移至[百度入口網站]，然後按一下 [登录] \(**登入**) 連結。 按一下 [立即注册] \(Register Now) 開始帳戶註冊程序。
   
    ![百度註冊](./media/notification-hubs-baidu-get-started/BaiduRegistration.png)

2. 輸入必要的詳細資料—電話/電子郵件地址、密碼和驗證碼，然後按一下 [注册] \(Signup)。
   
    ![百度註冊輸入](./media/notification-hubs-baidu-get-started/BaiduRegistrationInput.png)

3. 您所輸入的電子郵件地址會收到一封電子郵件，內含啟用百度帳戶的連結。
   
    ![百度註冊確認](./media/notification-hubs-baidu-get-started/BaiduConfirmation.png)

4. 登入您的電子郵件帳戶，開啟百度啟用郵件，然後按一下啟用連結以啟用您的百度帳戶。
   
    ![百度啟用電子郵件](./media/notification-hubs-baidu-get-started/BaiduActivationEmail.png)

啟用百度帳戶之後，請登入 [百度入口網站]。

## <a name="create-a-baidu-cloud-push-project"></a>建立百度雲端推播專案
建立百度雲端推播專案時，您會收到應用程式識別碼、API 金鑰和秘密金鑰。

1. 登入[百度入口網站]之後，按一下 [更多>>] \(**更多**)。
   
    ![註冊 - 更多](./media/notification-hubs-baidu-get-started/BaiduRegistrationMore.png)

2. 向下捲動至 [站长与开发者服务] \(Webmaster and Developer Services) 區段，然後按一下 [百度云推送] \(Baidu Cloud Push)。
   
    ![百度雲開放平台](./media/notification-hubs-baidu-get-started/BaiduOpenCloudPlatform.png)

3. 在下一個頁面上，按一下右上角的 [登录] \(Login)。
   
    ![百度登入](./media/notification-hubs-baidu-get-started/BaiduLogin.png)

4. 然後，按一下此頁面上的 [创建应用] \(Create Application)。

    ![百度建立應用程式](./media/notification-hubs-baidu-get-started/BaiduCreateApplication.png)

5. 在下一個頁面上，按一下 [创建新应用] \(Create New Application)。
   
    ![百度建立新的應用程式](./media/notification-hubs-baidu-get-started/BaiduCreateNewApplication.png)

6. 輸入應用程式名稱，然後按一下 [创建] \(Create)。
   
    ![](./media/notification-hubs-baidu-get-started/BaiduCreateApplicationDoCreate.png)

7. 成功建立百度雲推送專案後，您會看到包含 **AppID**、**API 金鑰**與**祕密金鑰**的頁面。 記下 API 金鑰和秘密金鑰，我們稍後會使用。
   
    ![百度推送秘密](./media/notification-hubs-baidu-get-started/BaiduGetSecrets.png)

8. 按一下左窗格中的 [创建通知] \(Create Notification)，以設定專案的推播通知。
   
    ![](./media/notification-hubs-baidu-get-started/BaiduCreateNotification.png)


## <a name="configure-a-new-notification-hub"></a>設定新的通知中樞
[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]


&emsp;&emsp;6. 在您的通知中樞中，選取 [通知服務]，然後選取 [百度 (Android 中國)]。

&emsp;&emsp;![Azure 通知中樞 - 百度](./media/notification-hubs-baidu-get-started/AzureNotificationServicesBaidu.png)

&emsp;&emsp;7. 向下捲動至 [百度通知設定] 區段。 在百度雲推送專案中，輸入從百度主控台取得的 API 金鑰和秘密金鑰。 然後按一下 [儲存]。

&emsp;&emsp;![Azure 通知中樞 - 百度秘密](./media/notification-hubs-baidu-get-started/NotificationHubBaiduConfigure.png)

您的通知中樞現在已設定成使用百度。 您也有**連接字串**可用來註冊應用程式，以傳送和接收推播通知。

記下 [存取連線資訊] 視窗中的 `DefaultListenSharedAccessSignature` 和 `DefaultFullSharedAccessSignature`。

## <a name="connect-your-app-to-the-notification-hub"></a>將您的應用程式連接到通知中樞
1. 在 Android Studio 中，建立新的 Android 專案 (檔案 > 新增 > 新增專案)。

    ![Azure 通知中樞 - 百度新增專案](./media/notification-hubs-baidu-get-started/AndroidNewProject.png)

2.  輸入 [應用程式名稱]，並確定 [最低必要 SDK] 版本已設定為 [API 16: Android 4.1]。 **此外，請確定您的套件名稱 (应用包名) 與百度雲推送入口網站中的名稱相同**

    ![Azure 通知中樞 - 百度最低 SDK1](./media/notification-hubs-baidu-get-started/AndroidMinSDK.png)
    ![Azure 通知中樞 - 百度最低 SDK2](./media/notification-hubs-baidu-get-started/AndroidMinSDK2.png)

3.  按 [下一步] 並繼續遵循精靈的指示進行，直到出現 [建立活動] 視窗為止。 請確定已選取 [空的活動]，最後選取 [完成] 以建立新的 Android 應用程式。

    ![Azure 通知中樞 - 百度新增活動](./media/notification-hubs-baidu-get-started/AndroidAddActivity.png)

4.  確定已正確設定 [專案建置目標]。

5.  然後新增 Azure 通知中樞程式庫。 在應用程式的 `Build.Gradle` 檔案中，於 dependencies 區段中新增下列幾行。

    ```javascript
    compile 'com.microsoft.azure:notification-hubs-android-sdk:0.4@aar'
    compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'
    ```

    將下列存放庫新增到 dependencies 區段之後。

    ```javascript
    repositories {
        maven {
            url "http://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

    為了避免清單衝突，我們必須在 **Manifest.xml** 中新增下列程式碼。

    ```xml
    <manifest package="YOUR.PACKAGE.NAME"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android">
    ```

    以及在 `<application/>` 標籤中新增：

    ```xml
    <application
        tools:replace="android:allowBackup,icon,theme,label">
    ```

6.  下載並解壓縮 [百度推播 Android SDK]。 複製 libs 資料夾中的 `pushservice-x.y.z jar` 檔案。 然後複製 Android 應用程式的 `src/main/jniLibs` (建立新的資料夾) 資料夾中的 `.so` 檔案。

    ![Azure 通知中樞 - 百度 SDK 程式庫](./media/notification-hubs-baidu-get-started/BaiduSDKLib.png)

7. 以滑鼠右鍵按一下 libs 資料夾中的 pushervice-x.y.z.jar 檔案，並按一下 [新增為程式庫] 以在專案中包含這個程式庫。

    ![Azure 通知中樞 - 百度新增為程式庫](./media/notification-hubs-baidu-get-started/BaiduAddAsALib.jpg)

8. 開啟 Android 專案的 **AndroidManifest.xml** 檔案，並新增 Baidu SDK 所需的權限。 **以您的套件名稱取代 `YOURPACKAGENAME`**。

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_DOWNLOAD_MANAGER" />
    <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION" />
    <uses-permission android:name="android.permission.EXPAND_STATUS_BAR" />
    !! <uses-permission android:name="baidu.push.permission.WRITE_PUSHINFOPROVIDER.YOURPACKAGENAME" />
    !!<permission android:name="baidu.push.permission.WRITE_PUSHINFOPROVIDER.YOURPACKAGENAME"android:protectionLevel="normal" />

    ```

9. 在應用程式元素內，於 `.MainActivity` 活動元素之後新增下列組態，並取代 yourprojectname (例如 `com.example.BaiduTest`)：

    ```xml
    <activity
        android:name="com.baidu.android.pushservice.richmedia.MediaViewActivity"
        android:configChanges="orientation|keyboardHidden"
        android:label="MediaViewActivity" />
    <activity
        android:name="com.baidu.android.pushservice.richmedia.MediaListActivity"
        android:configChanges="orientation|keyboardHidden"
        android:label="MediaListActivity"
        android:launchMode="singleTask" />
 
    <!-- Push application definition message -->
    <receiver android:name=".MyPushMessageReceiver">
        <intent-filter>

            <!-- receive push message-->
            <action android:name="com.baidu.android.pushservice.action.MESSAGE" />
            <!-- receive bind,unbind,fetch,delete.. message-->
            <action android:name="com.baidu.android.pushservice.action.RECEIVE" />
            <action android:name="com.baidu.android.pushservice.action.notification.CLICK" />
        </intent-filter>
    </receiver>

    <receiver
        android:name="com.baidu.android.pushservice.PushServiceReceiver"
        android:process=":bdservice_v1">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED" />
            <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            <action android:name="com.baidu.android.pushservice.action.notification.SHOW" />
            <action android:name="com.baidu.android.pushservice.action.media.CLICK" />
            <action android:name="android.intent.action.MEDIA_MOUNTED" />
            <action android:name="android.intent.action.USER_PRESENT" />
            <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
            <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED" />
        </intent-filter>
    </receiver>

    <receiver
        android:name="com.baidu.android.pushservice.RegistrationReceiver"
        android:process=":bdservice_v1">
        <intent-filter>
            <action android:name="com.baidu.android.pushservice.action.METHOD" />
            <action android:name="com.baidu.android.pushservice.action.BIND_SYNC" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.PACKAGE_REMOVED" />

            <data android:scheme="package" />
        </intent-filter>
    </receiver>

    <service
        android:name="com.baidu.android.pushservice.PushService"
        android:exported="true"
        android:process=":bdservice_v1">
        <intent-filter>
            <action android:name="com.baidu.android.pushservice.action.PUSH_SERVICE" />
        </intent-filter>
    </service>

    <service
        android:name="com.baidu.android.pushservice.CommandService"
        android:exported="true" />

    <!-- Adapt the ContentProvider declaration required for the Android N system, and the write permissions include the application package name-->
    <provider
        android:name="com.baidu.android.pushservice.PushInfoProvider"
        android:authorities="com.baidu.push.example.bdpush"
        android:exported="true"
        android:protectionLevel="signature"
        android:writePermission="baidu.push.permission.WRITE_PUSHINFOPROVIDER. yourprojectname  " />

    <!-- API Key of the Baidu application -->
    <meta-data
        android:name="api_key"
        !!   android:value="api_key" />
    </application>
    ```

10. 將名為 `ConfigurationSettings.java` 的新類別新增至專案。

    ```java
    public class ConfigurationSettings {
        public static String API_KEY = "...";
        public static String NotificationHubName = "...";
        public static String NotificationHubConnectionString = "...";
    }
    ```
    
    使用百度雲專案中的 API_KEY 來設定 `API_KEY` 字串的值。
    
    使用 [Azure 入口網站]中您的通知中樞名稱來設定 `NotificationHubName` 字串的值，然後以 [Azure 入口網站]中的 `DefaultListenSharedAccessSignature` 取代 `NotificationHubConnectionString`。

11. 開啟 MainActivity.java，並將下列程式碼新增至 onCreate 方法：

    ```java
    PushManager.startWork(this, PushConstants.LOGIN_TYPE_API_KEY,  API_KEY );
    ```

12. 新增名為 `MyPushMessageReceiver.java` 的類別，並在其中新增下列程式碼。 此類別會處理接收自百度推播伺服器的推播通知。

    ```java
    package your.package.name;

    import android.content.Context;
    import android.content.Intent;
    import android.os.AsyncTask;
    import android.text.TextUtils;
    import android.util.Log;

    import com.baidu.android.pushservice.PushMessageReceiver;
    import com.microsoft.windowsazure.messaging.NotificationHub;
    import org.json.JSONException;
    import org.json.JSONObject;

    import java.util.List;

    public class MyPushMessageReceiver extends PushMessageReceiver {

        public static final String TAG = MyPushMessageReceiver.class
                .getSimpleName();
        public static NotificationHub hub = null;
        public static String mChannelId, mUserId;

        @Override
        public void onBind(Context context, int errorCode, String appid,
                        String userId, String channelId, String requestId) {
            String responseString = "onBind errorCode=" + errorCode + " appid="
                    + appid + " userId=" + userId + " channelId=" + channelId
                    + " requestId=" + requestId;
            Log.d(TAG, responseString);

            if (errorCode == 0) {
                // Binding successful
                Log.d(TAG, " Binding successful");
            }
            try {
                if (hub == null) {
                    hub = new NotificationHub(
                            ConfigurationSettings.NotificationHubName,
                            ConfigurationSettings.NotificationHubConnectionString,
                            context);
                    Log.i(TAG, "Notification hub initialized");
                }
            } catch (Exception e) {
                Log.e(TAG, e.getMessage());
            }
            mChannelId = channelId;
            mUserId = userId;

            registerWithNotificationHubs();
        }
        private void registerWithNotificationHubs() {

            new AsyncTask<Void, Void, Void>() {
                @Override
                protected Void doInBackground(Void... params) {
                    try {
                        hub.registerBaidu(mUserId, mChannelId);
                        Log.i(TAG, "Registered with Notification Hub - '"
                                + ConfigurationSettings.NotificationHubName + "'"
                                + " with UserId - '"
                                + mUserId + "' and Channel Id - '"
                                + mChannelId + "'");
                    } catch (Exception e) {
                        Log.e(TAG, e.getMessage());
                    }
                    return null;
                }
            }.execute(null, null, null);
        }

        @Override
        public void onMessage(Context context, String message,
                            String customContentString) {
            String messageString = " onMessage=\"" + message
                    + "\" customContentString=" + customContentString;
            Log.d(TAG, messageString);
            if (!TextUtils.isEmpty(customContentString)) {
                JSONObject customJson = null;
                try {
                    customJson = new JSONObject(customContentString);
                    String myvalue = null;
                    if (!customJson.isNull("mykey")) {
                        myvalue = customJson.getString("mykey");
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }

        }

        @Override
        public void onNotificationArrived(Context context, String title, String description, String customContentString) {
            String notifyString = " Notice Arrives onNotificationArrived  title=\"" + title
                    + "\" description=\"" + description + "\" customContent="
                    + customContentString;
            Log.d(TAG, notifyString);
            if (!TextUtils.isEmpty(customContentString)) {
                JSONObject customJson = null;
                try {
                    customJson = new JSONObject(customContentString);
                    String myvalue = null;
                    if (!customJson.isNull("mykey")) {
                        myvalue = customJson.getString("mykey");
                    }
                } catch (JSONException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onNotificationClicked(Context context, String title, String description, String customContentString) {
            String notifyString = " onNotificationClicked title=\"" + title + "\" description=\""
                    + description + "\" customContent=" + customContentString;
            Log.d(TAG, notifyString);
            Intent intent = new Intent(context.getApplicationContext(),MainActivity.class);
            intent.putExtra("title",title);
            intent.putExtra("description",description);
            intent.putExtra("isFromNotify",true);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.getApplicationContext().startActivity(intent);

        }

        @Override
        public void onSetTags(Context context, int errorCode,
                            List<String> successTags, List<String> failTags, String requestId) {
            String responseString = "onSetTags errorCode=" + errorCode
                    + " successTags=" + successTags + " failTags=" + failTags
                    + " requestId=" + requestId;
            Log.d(TAG, responseString);

        }

        @Override
        public void onDelTags(Context context, int errorCode,
                            List<String> successTags, List<String> failTags, String requestId) {
            String responseString = "onDelTags errorCode=" + errorCode
                    + " successTags=" + successTags + " failTags=" + failTags
                    + " requestId=" + requestId;
            Log.d(TAG, responseString);

        }

        @Override
        public void onListTags(Context context, int errorCode, List<String> tags,
                            String requestId) {
            String responseString = "onListTags errorCode=" + errorCode + " tags="
                    + tags;
            Log.d(TAG, responseString);

        }

        @Override
        public void onUnbind(Context context, int errorCode, String requestId) {
            String responseString = "onUnbind errorCode=" + errorCode
                    + " requestId = " + requestId;
            Log.d(TAG, responseString);

            if (errorCode == 0) {
                // Unbinding is successful
                Log.d(TAG, " Unbinding is successful ");
            }
        }
    }
    ```

## <a name="send-notifications-to-your-app"></a>將通知傳送至您的應用程式

您可以在 [Azure 入口網站]中快速測試通知的接收：使用通知中樞設定畫面中的 [傳送] 按鈕 (如下列畫面所示)︰

![](./media/notification-hubs-baidu-get-started/BaiduTestSendButton.png)
![](./media/notification-hubs-baidu-get-started/BaiduTestSend.png)

推播通知通常會以後端服務傳送，例如行動服務或使用相容程式庫的 ASP.NET。 如果程式庫不適用於您的後端，您可以直接使用 REST API 來傳送通知訊息。

為了簡單起見，本教學課程使用主控台應用程式來示範如何透過 .NET SDK 傳送通知。 不過我們建議以[使用通知中樞將通知推播給使用者](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)教學課程作為下一個步驟，以便從 ASP.NET 後端傳送通知。 

以下是不同的傳送通知方法：
* **REST 介面**：您可以在使用 [REST 介面](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx)的任何後端平台上支援通知。
* **Microsoft Azure 通知中樞 .NET SDK**︰在適用於 Visual Studio 的 NuGet 封裝管理員中，執行 [Install-Package Microsoft.Azure.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)。
* **Node.js**：[如何從 Node.js 使用通知中樞](notification-hubs-nodejs-push-notification-tutorial.md)。
* **Mobile Apps**：如需如何從已與通知中樞整合的 Azure App Service Mobile Apps 傳送通知的範例，請參閱[將推播通知新增至行動應用程式](../app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push.md)。
* **Java/PHP**︰如需使用 REST API 傳送通知的範例，請參閱＜如何從 Java/PHP 使用通知中樞＞([Java](notification-hubs-java-push-notification-tutorial.md) | [PHP](notification-hubs-php-push-notification-tutorial.md))。

## <a name="optional-send-notifications-from-a-net-console-app"></a>(選擇性) 從 .NET 主控台應用程式傳送通知。
在本節中，我們會說明如何使用.NET 主控台應用程式傳送通知。

1. 建立新的 Visual C# 主控台應用程式：
   
    ![](./media/notification-hubs-baidu-get-started/ConsoleProject.png)

2. 在 [套件管理員主控台] 視窗中，將 [預設專案]  設為新的主控台應用程式專案，然後在主控台視窗中執行下列命令：
   
        Install-Package Microsoft.Azure.NotificationHubs
   
    此指示會使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 套件</a>新增對 Azure 通知中樞 SDK 的參考。
   
    ![](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-package-manager.png)

3. 開啟 `Program.cs` 檔案，並新增下列 using 陳述式：
   
    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

4. 在 `Program` 類別中，新增下列方法，並以您具有的值取代 `DefaultFullSharedAccessSignatureSASConnectionString` 和 `NotificationHubName`。
   
    ```csharp
    private static async void SendNotificationAsync()
    {
        NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("DefaultFullSharedAccessSignatureSASConnectionString", "NotificationHubName");
        string message = "{\"title\":\"((Notification title))\",\"description\":\"Hello from Azure\"}";
        var result = await hub.SendBaiduNativeNotificationAsync(message);
    }
    ```

5. 在您的 `Main` 方法中新增下列程式碼行：

    ```csharp
    SendNotificationAsync();
    Console.ReadLine();
    ```

## <a name="test-your-app"></a>測試應用程式

若要使用真正的手機來測試應用程式，您只需要使用 USB 纜線將其連接到電腦即可。 此動作會將應用程式載入連結的手機。

若要使用模擬器測試此應用程式，請按一下 Android Studio 頂端工具列上的 [執行]，然後選取您的應用程式︰這會啟動模擬器、載入和執行應用程式。

應用程式會從百度推播通知服務中擷取 `userId` 和 `channelId`，並向通知中樞註冊。

若要傳送測試通知，您可以使用 [Azure 入口網站]的 [偵錯] 索引標籤。 如果建置 Visual Studio 的 .NET 主控台應用程式時，僅須在 Visual Studio 中按 F5 鍵，即可執行該應用程式。 應用程式會傳送通知，而該通知會顯示在裝置或模擬器上方的通知區域中。

<!-- URLs. -->
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[百度推播 Android SDK]: http://push.baidu.com/sdk/push_client_sdk_for_android
[Azure 入口網站]: https://portal.azure.com/
[百度入口網站]: http://www.baidu.com/
