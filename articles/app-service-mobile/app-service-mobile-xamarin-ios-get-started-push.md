---
title: 使用 Azure App Service 將推播通知新增至 Xamarin.iOS 應用程式
description: 了解如何使用 Azure App Service 將推播通知傳送至 Xamarin.iOS 應用程式。
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 2921214a-49f8-45e1-a306-a85ce21defca
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: article
ms.date: 10/12/2016
ms.author: crdun
ms.openlocfilehash: 32a8c36d223e2b0c12f5d82ec748af66ae841b01
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42819017"
---
# <a name="add-push-notifications-to-your-xamarinios-app"></a>將推播通知新增至 Xamarin.iOS 應用程式

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## <a name="overview"></a>概觀

在本教學課程中，您會將推播通知新增至 [Xamarin.iOS 快速入門](app-service-mobile-xamarin-ios-get-started.md)專案，以便在每次插入一筆記錄時傳送推播通知至裝置。

如果您不要使用下載的快速入門伺服器專案，將需要推播通知擴充套件。 如需詳細資訊，請參閱[使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

## <a name="prerequisites"></a>必要條件

* 完成 [建立 Xamarin.iOS 應用程式](app-service-mobile-xamarin-ios-get-started.md) 教學課程。
* 實體的 iOS 裝置。 iOS 模擬器不支援推播通知。

## <a name="register-the-app-for-push-notifications-on-apples-developer-portal"></a>在 Apple 的開發人員入口網站註冊應用程式以取得推播通知

[!INCLUDE [Enable Apple Push Notifications](../../includes/enable-apple-push-notifications.md)]

## <a name="configure-your-mobile-app-to-send-push-notifications"></a>設定您的行動應用程式以傳送推播通知

[!INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

## <a name="update-the-server-project-to-send-push-notifications"></a>更新伺服器專案以傳送推播通知

[!INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

## <a name="configure-your-xamarinios-project"></a>設定您的 Xamarin.iOS 專案

[!INCLUDE [app-service-mobile-xamarin-ios-configure-project](../../includes/app-service-mobile-xamarin-ios-configure-project.md)]

## <a name="add-push-notifications-to-your-app"></a>將推播通知新增至應用程式

1. 在 **QSTodoService** 中新增下列屬性，以便讓 **AppDelegate** 能夠取得行動用戶端：

    ```csharp
    public MobileServiceClient GetClient {
        get
        {
            return client;
        }
        private set
        {
            client = value;
        }
    }
    ```

2. 將下列 `using` 陳述式加入至 **AppDelegate.cs** 檔案頂端。

    ```csharp
    using Microsoft.WindowsAzure.MobileServices;
    using Newtonsoft.Json.Linq;
    ```

3. 在 **AppDelegate** 中，覆寫 **FinishedLaunching** 事件：

   ```csharp
    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        // registers for push for iOS8
        var settings = UIUserNotificationSettings.GetSettingsForTypes(
            UIUserNotificationType.Alert
            | UIUserNotificationType.Badge
            | UIUserNotificationType.Sound,
            new NSSet());

        UIApplication.SharedApplication.RegisterUserNotificationSettings(settings);
        UIApplication.SharedApplication.RegisterForRemoteNotifications();

        return true;
    }
    ```

4. 在相同檔案中，覆寫 `RegisteredForRemoteNotifications` 事件。 透過此程式碼，您將註冊能夠在伺服器所支援的所有平台間傳送的簡單範本通知。

    如需有關通知中樞範本的詳細資訊，請參閱 [範本](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)。

    ```csharp
    public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
    {
        MobileServiceClient client = QSTodoService.DefaultService.GetClient;

        const string templateBodyAPNS = "{\"aps\":{\"alert\":\"$(messageParam)\"}}";

        JObject templates = new JObject();
        templates["genericMessage"] = new JObject
        {
            {"body", templateBodyAPNS}
        };

        // Register for push with your mobile app
        var push = client.GetPush();
        push.RegisterAsync(deviceToken, templates);
    }
    ```

5. 然後，覆寫 **DidReceivedRemoteNotification** 事件：

   ```csharp
    public override void DidReceiveRemoteNotification (UIApplication application, NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
    {
        NSDictionary aps = userInfo.ObjectForKey(new NSString("aps")) as NSDictionary;

        string alert = string.Empty;
        if (aps.ContainsKey(new NSString("alert")))
            alert = (aps [new NSString("alert")] as NSString).ToString();

        //show alert
        if (!string.IsNullOrEmpty(alert))
        {
            UIAlertView avAlert = new UIAlertView("Notification", alert, null, "OK", null);
            avAlert.Show();
        }
    }
    ```

您的應用程式現在已更新為支援推播通知。

## <a name="test"></a>在應用程式中測試推播通知

1. 按 [執行] 按鈕以建置專案並在可執行 iOS 的裝置上啟動應用程式，然後按一下 [確定] 以接受推播通知。

   > [!NOTE]
   > 您必須明確地接受來自應用程式的推播通知。 只有在應用程式第一次執行時，才會發生此要求。

2. 在應用程式中輸入一項工作，然後按一下加號 (**+**) 圖示。
3. 確認您已接收到通知，然後按一下 [確定]  以關閉通知。
4. 重複執行步驟 2 並立即關閉應用程式，接著確認通知已顯示。

您已成功完成此教學課程。
