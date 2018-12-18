---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 3ced8c9fdadf547ec234c6d1bd5f3ddb8af7bc05
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42816257"
---
1. 在名為 `ToDoBroadcastReceiver`的專案中建立新類別。
2. 在 **ToDoBroadcastReceiver** 類別中新增下列 using 陳述式：

    ```csharp
    using Gcm.Client;
    using Microsoft.WindowsAzure.MobileServices;
    using Newtonsoft.Json.Linq;
    ```

3. 在 **using** 陳述式與 **namespace** 宣告之間新增下列權限要求：

    ```csharp
    [assembly: Permission(Name = "@PACKAGE_NAME@.permission.C2D_MESSAGE")]
    [assembly: UsesPermission(Name = "@PACKAGE_NAME@.permission.C2D_MESSAGE")]
    [assembly: UsesPermission(Name = "com.google.android.c2dm.permission.RECEIVE")]
    //GET_ACCOUNTS is only needed for android versions 4.0.3 and below
    [assembly: UsesPermission(Name = "android.permission.GET_ACCOUNTS")]
    [assembly: UsesPermission(Name = "android.permission.INTERNET")]
    [assembly: UsesPermission(Name = "android.permission.WAKE_LOCK")]
    ```

4. 使用下列程式碼取代現有的 **ToDoBroadcastReceiver** 類別定義：

    ```csharp
    [BroadcastReceiver(Permission = Gcm.Client.Constants.PERMISSION_GCM_INTENTS)]
    [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_MESSAGE }, 
        Categories = new string[] { "@PACKAGE_NAME@" })]
    [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_REGISTRATION_CALLBACK }, 
        Categories = new string[] { "@PACKAGE_NAME@" })]
    [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_LIBRARY_RETRY }, 
    Categories = new string[] { "@PACKAGE_NAME@" })]
    public class ToDoBroadcastReceiver : GcmBroadcastReceiverBase<PushHandlerService>
    {
        // Set the Google app ID.
        public static string[] senderIDs = new string[] { "<PROJECT_NUMBER>" };
    }
    ```

    在上述程式碼中，您必須使用在 Google 開發人員入口網站佈建您的應用程式時由 Google 指派給您的專案編號來取代 *`<PROJECT_NUMBER>`*。 

5. 在 ToDoBroadcastReceiver.cs 專案檔案中，加入下列可定義 **PushHandlerService** 類別的程式碼：

    ```csharp
    // The ServiceAttribute must be applied to the class.
    [Service]
    public class PushHandlerService : GcmServiceBase
    {
        public static string RegistrationID { get; private set; }
        public PushHandlerService() : base(ToDoBroadcastReceiver.senderIDs) { }
    }
    ```

    請注意，此類別衍生自 **GcmServiceBase** 而且必須對此類別套用 **Service** 屬性。

    > [!NOTE]
    > **GcmServiceBase** 類別實作 **OnRegistered()**、**OnUnRegistered()**、**OnMessage()** 及 **OnError()** 方法。 您必須覆寫 **PushHandlerService** 類別中的這些方法。

6. 在 **PushHandlerService** 類別中加入下列程式碼，以覆寫 **OnRegistered** 事件處理常式。

    ```csharp
    protected override void OnRegistered(Context context, string registrationId)
    {
        System.Diagnostics.Debug.WriteLine("The device has been registered with GCM.", "Success!");

        // Get the MobileServiceClient from the current activity instance.
        MobileServiceClient client = ToDoActivity.CurrentActivity.CurrentClient;
        var push = client.GetPush();

        // Define a message body for GCM.
        const string templateBodyGCM = "{\"data\":{\"message\":\"$(messageParam)\"}}";

        // Define the template registration as JSON.
        JObject templates = new JObject();
        templates["genericMessage"] = new JObject
        {
            {"body", templateBodyGCM }
        };

        try
        {
            // Make sure we run the registration on the same thread as the activity, 
            // to avoid threading errors.
            ToDoActivity.CurrentActivity.RunOnUiThread(

                // Register the template with Notification Hubs.
                async () => await push.RegisterAsync(registrationId, templates));

            System.Diagnostics.Debug.WriteLine(
                string.Format("Push Installation Id", push.InstallationId.ToString()));
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(
                string.Format("Error with Azure push registration: {0}", ex.Message));
        }
    }
    ```

    此方法使用傳回的 GCM 註冊識別碼，向 Azure 註冊以取得推送通知。 只能在建立註冊之後加入標記。 如需詳細資訊，請參閱 [做法：將標記新增至裝置安裝以啟用發送到標記](../articles/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags)。

7. 以下列程式碼覆寫 **PushHandlerService** 中的 **OnMessage** 方法：

    ```csharp
    protected override void OnMessage(Context context, Intent intent)
    {
        string message = string.Empty;

        // Extract the push notification message from the intent.
        if (intent.Extras.ContainsKey("message"))
        {
            message = intent.Extras.Get("message").ToString();
            var title = "New item added:";

            // Create a notification manager to send the notification.
            var notificationManager = 
                GetSystemService(Context.NotificationService) as NotificationManager;

            // Create a new intent to show the notification in the UI. 
            PendingIntent contentIntent =
                PendingIntent.GetActivity(context, 0,
                new Intent(this, typeof(ToDoActivity)), 0);

            // Create the notification using the builder.
            var builder = new Notification.Builder(context);
            builder.SetAutoCancel(true);
            builder.SetContentTitle(title);
            builder.SetContentText(message);
            builder.SetSmallIcon(Resource.Drawable.ic_launcher);
            builder.SetContentIntent(contentIntent);
            var notification = builder.Build();

            // Display the notification in the Notifications Area.
            notificationManager.Notify(1, notification);

        }
    }
    ```

8. 使用下列程式碼覆寫 **OnUnRegistered()** 和 **OnError()** 方法。

    ```csharp
    protected override void OnUnRegistered(Context context, string registrationId)
    {
        throw new NotImplementedException();
    }

    protected override void OnError(Context context, string errorId)
    {
        System.Diagnostics.Debug.WriteLine(
            string.Format("Error occurred in the notification: {0}.", errorId));
    }
    ```
