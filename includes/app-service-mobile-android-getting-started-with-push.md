---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 654bc3745768fccea41d7c3991142bf7183b54be
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809509"
---
1. 在您的 **app** 專案中開啟 `AndroidManifest.xml` 檔案。 在 `application` 起始標籤後新增下列程式碼：

    ```xml
    <service android:name=".ToDoMessagingService">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT"/>
        </intent-filter>
    </service>
    <service android:name=".ToDoInstanceIdService">
        <intent-filter>
            <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
        </intent-filter>
    </service>
    ```

2. 開啟檔案 `ToDoActivity.java` 並進行下列變更：

    - 新增匯入陳述式：

        ```java
        import com.google.firebase.iid.FirebaseInstanceId;
        ```

    - 將 `MobileServiceClient` 的定義從 [私人] 變更為 [公用靜態]，如下所示：

        ```java
        private static MobileServiceClient mClient;
        ```

    - 新增 `registerPush` 方法：

        ```java
        public static void registerPush() {
            final String token = FirebaseInstanceId.getInstance().getToken();
            if (token != null) {
                new AsyncTask<Void, Void, Void>() {
                    protected Void doInBackground(Void... params) {
                        mClient.getPush().register(token);
                        return null;
                    }
                }.execute();
            }
        }
        ```

    - 更新 `ToDoActivity` 類別的 **onCreate** 方法。 確定在 `MobileServiceClient` 具現化之後新增此程式碼。

        ```java
        registerPush();
        ```

3. 新增類別來處理通知。 在 [專案總管] 中，開啟 **app** > **java** > **your-project-namespace** 節點，然後以滑鼠右鍵按一下套件名稱節點。 按一下 [新增]，然後按一下 [Java 類別]。 在 [名稱] 中，輸入 `ToDoMessagingService`，然後按一下 [確定]。 然後，將類別宣告取代為：

    ```java
    import android.app.Notification;
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;

    import com.google.firebase.messaging.FirebaseMessagingService;
    import com.google.firebase.messaging.RemoteMessage;

    public class ToDoMessagingService extends FirebaseMessagingService {

        private static final int NOTIFICATION_ID = 1;

        @Override
        public void onMessageReceived(RemoteMessage remoteMessage) {
            String message = remoteMessage.getData().get("message");
            if (message != null) {
                sendNotification("Notification Hub Demo", message);
            }
        }

        private void sendNotification(String title, String messageBody) {
            PendingIntent contentIntent = PendingIntent.getActivity(this, 0, new Intent(this, ToDoActivity.class), 0);
            Notification.Builder notificationBuilder = new Notification.Builder(this)
                    .setSmallIcon(R.drawable.ic_launcher)
                    .setContentTitle(title)
                    .setContentText(messageBody)
                    .setContentIntent(contentIntent);
            NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            if (notificationManager != null) {
                notificationManager.notify(NOTIFICATION_ID, notificationBuilder.build());
            }
        }
    }
    ```

4. 新增另一個類別來處理權杖的更新。 建立 `ToDoInstanceIdService` java 類別，並將類別宣告取代為：

    ```java
    import com.google.firebase.iid.FirebaseInstanceIdService;

    public class ToDoInstanceIdService extends FirebaseInstanceIdService {

        @Override
        public void onTokenRefresh() {
            ToDoActivity.registerPush();
        }
    }
    ```

您的應用程式現在已更新為支援推播通知。
