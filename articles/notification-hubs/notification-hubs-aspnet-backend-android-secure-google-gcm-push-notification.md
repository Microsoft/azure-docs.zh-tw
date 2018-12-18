---
title: 使用 Azure 通知中樞傳送安全的推播通知
description: 了解如何從 Azure 將安全的推播通知傳送至 Android 應用程式。 程式碼範例是以 Java 及 C# 撰寫。
documentationcenter: android
keywords: 推播通知,推播訊息,android 推播通知
author: dimazaid
manager: kpiteira
editor: spelluru
services: notification-hubs
ms.assetid: daf3de1c-f6a9-43c4-8165-a76bfaa70893
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: android
ms.devlang: java
ms.topic: article
ms.date: 04/25/2018
ms.author: dimazaid
ms.openlocfilehash: 58f6967c59a5060baa10ff83752b9c6ed08226cb
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38698023"
---
# <a name="sending-secure-push-notifications-with-azure-notification-hubs"></a>使用 Azure 通知中樞傳送安全的推播通知
> [!div class="op_single_selector"]
> * [Windows Universal](notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md)
> * [iOS](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md)
> * [Android](notification-hubs-aspnet-backend-android-secure-google-gcm-push-notification.md)
> 
> 

## <a name="overview"></a>概觀
> [!IMPORTANT]
> 若要完成此教學課程，您必須具備有效的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A643EE910&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdocumentation%2Farticles%2Fpartner-xamarin-notification-hubs-ios-get-started)。
> 
> 

Microsoft Azure 中的推播通知支援可讓您存取易於使用、多重平台的大規模推播訊息基礎結構，因而可大幅簡化消費者和企業應用程式在行動平台上的推播通知實作。

基於法規或安全性限制，應用程式有時會想要在通知中加入無法透過標準推播通知基礎結構傳送的內容。 本教學課程說明如何透過用戶端 Android 裝置和應用程式後端之間的安全、已驗證連線來傳送敏感資訊，以達到相同體驗。

概括而言，流程如下所示：

1. 應用程式後端：
   * 在後端資料庫中儲存安全裝載。
   * 將此通知的識別碼傳送至 Android 裝置 (不會傳送安全資訊)。
2. 收到通知時，裝置上的應用程式會執行下列動作：
   * Android 裝置會連絡後端並要求安全裝載。
   * 應用程式會以通知的形式在裝置上顯示裝載。

請務必注意在上述流程 (與本教學課程) 中，假設使用者登入後，裝置會將驗證權杖儲存在本機儲存體中。 由於裝置可使用此權杖擷取通知的安全裝載，所以這個可保證順暢的體驗。 如果您的應用程式沒有將驗證權杖儲存在裝置上，或如果這些權杖可能會過期，裝置應用程式應在收到推播通知時顯示一般通知，以提示使用者啟動應用程式。 應用程式會接著驗證使用者，並顯示通知裝載。

本教學課程示範如何傳送安全的推播通知。 本教學課程以 [通知使用者](notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md) 教學課程為基礎，如果您還未完成該教學課程中的步驟，請先完成。

> [!NOTE]
> 本教學課程假設您已建立並設定通知中樞，如 [開始使用通知中樞 (Android)](notification-hubs-android-push-notification-google-gcm-get-started.md)中所述。
> 
> 

[!INCLUDE [notification-hubs-aspnet-backend-securepush](../../includes/notification-hubs-aspnet-backend-securepush.md)]

## <a name="modify-the-android-project"></a>修改 Android 專案
現在您已將應用程式後端修改為只傳送推播通知的*識別碼*，就必須變更 Android 應用程式來處理該通知，並回呼後端以擷取要顯示的安全訊息。
為了達成此目標，您必須確定您的 Android 應用程式知道在收到推播通知時，如何使用後端自我驗證。

為了將驗證標頭值儲存在您的應用程式共用喜好設定中，現在將修改登入流程。 類別機制可用來儲存任何驗證權杖 (例如 OAuth tokens)，應用程式在不需要使用者認證的情況下，必須使用這些驗證權杖。

1. 在您的 Android 應用程式專案中，在 **MainActivity** 類別開頭處新增下列常數：
   
        public static final String NOTIFY_USERS_PROPERTIES = "NotifyUsersProperties";
        public static final String AUTHORIZATION_HEADER_PROPERTY = "AuthorizationHeader";
2. 同樣地，在 **MainActivity** 類別中，更新 `getAuthorizationHeader()` 方法以包含下列程式碼：
   
        private String getAuthorizationHeader() throws UnsupportedEncodingException {
            EditText username = (EditText) findViewById(R.id.usernameText);
            EditText password = (EditText) findViewById(R.id.passwordText);
            String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
            basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
   
            SharedPreferences sp = getSharedPreferences(NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
            sp.edit().putString(AUTHORIZATION_HEADER_PROPERTY, basicAuthHeader).commit();
   
            return basicAuthHeader;
        }
3. 在 **MainActivity** 檔案開頭處新增下列 `import` 陳述式：
   
        import android.content.SharedPreferences;

現在，變更收到通知時所呼叫的處理常式。

1. 在 **MyHandler** 類別中，變更 `OnReceive()` 方法以包含下列程式碼：
   
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String secureMessageId = bundle.getString("secureId");
            retrieveNotification(secureMessageId);
        }
2. 然後新增 `retrieveNotification()` 方法，以部署後端時所取得的後端端點取代預留位置 `{back-end endpoint}`：
   
        private void retrieveNotification(final String secureMessageId) {
            SharedPreferences sp = ctx.getSharedPreferences(MainActivity.NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
            final String authorizationHeader = sp.getString(MainActivity.AUTHORIZATION_HEADER_PROPERTY, null);
   
            new AsyncTask<Object, Object, Object>() {
                @Override
                protected Object doInBackground(Object... params) {
                    try {
                        HttpUriRequest request = new HttpGet("{back-end endpoint}/api/notifications/"+secureMessageId);
                        request.addHeader("Authorization", "Basic "+authorizationHeader);
                        HttpResponse response = new DefaultHttpClient().execute(request);
                        if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                            Log.e("MainActivity", "Error retrieving secure notification" + response.getStatusLine().getStatusCode());
                            throw new RuntimeException("Error retrieving secure notification");
                        }
                        String secureNotificationJSON = EntityUtils.toString(response.getEntity());
                        JSONObject secureNotification = new JSONObject(secureNotificationJSON);
                        sendNotification(secureNotification.getString("Payload"));
                    } catch (Exception e) {
                        Log.e("MainActivity", "Failed to retrieve secure notification - " + e.getMessage());
                        return e;
                    }
                    return null;
                }
            }.execute(null, null, null);
        }

本方法會呼叫應用程式後端，使用儲存在共用喜好設定中的認證來擷取通知內容，並以正常通知的方式顯示該通知。 應用程式使用者所看到的通知會與任何其他推播通知沒有兩樣。

比較理想的案例是處理遺失驗證標頭屬性或遭到後端拒絕的情況。 這些案例的特定處理絕大部分會依您的目標使用者經驗而定。 其中一個選項就是透過一般提示顯示通知，方便使用者進行驗證並擷取實際通知。

## <a name="run-the-application"></a>執行應用程式
若要執行應用程式，請執行下列動作：

1. 確定 Azure 中已部署 **AppBackend** 。 如果使用 Visual Studio，則執行 **AppBackend** Web API 應用程式。 [ASP.NET Web] 頁面便會隨即顯示。
2. 在 Eclipse 中，在實體 Android 裝置或模擬器上執行此應用程式。
3. 在 Android 應用程式 UI 中，輸入使用者名稱和密碼。 這些可以是任何字串，但必須是相同值。
4. 在 Android 應用程式 UI 中，按一下 [登入] 。 然後按一下 [傳送推播] 。

