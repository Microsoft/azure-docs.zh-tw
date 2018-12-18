---
title: 包含檔案
description: 包含檔案
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 03/30/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 19352df7abff23ed44521a11e7907c84c8c0327f
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "33835819"
---
在本節中，您會從 .NET 主控台應用程式將即時新聞以加註標記的範本通知形式傳送。 

1. 在 Visual Studio 中，建立新的 Visual C# 主控台應用程式：
   
      ![主控台應用程式連結][13]

2. 在 Visual Studio 主功能表上，選取 [工具] > [Library Package Manager] >  [Package Manager Console]，然後在主控台視窗中輸入下列字串：
   
        Install-Package Microsoft.Azure.NotificationHubs
   
3. 選取 [輸入]。  
    此動作會使用 [Microsoft.Azure.Notification 中樞 NuGet 封裝]來新增對 Azure 通知中樞 SDK 的參考。

4. 開啟 Program.cs 檔案，並新增下列 `using` 陳述式：
   
    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

5. 在 `Program` 類別中，新增或取代 (如果方法已存在) 下列方法：
   
    ```csharp
    private static async void SendTemplateNotificationAsync()
    {
        // Define the notification hub.
        NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");

        // Create an array of breaking news categories.
        var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};

        // Send the notification as a template notification. All template registrations that contain
        // "messageParam" and the proper tags will receive the notifications.
        // This includes APNS, GCM, WNS, and MPNS template registrations.

        Dictionary<string, string> templateParams = new Dictionary<string, string>();

        foreach (var category in categories)
        {
            templateParams["messageParam"] = "Breaking " + category + " News!";
            await hub.SendTemplateNotificationAsync(templateParams, category);
        }
    }
    ```   
   
    此程式碼會分別將範本通知傳送給字串陣列中的六個標籤。 使用標籤可確保裝置只會收到已登錄類別的通知。

5. 在上述程式碼中，請使用您的通知中樞名稱及通知中樞儀表板的 *DefaultFullSharedAccessSignature* 連接字串，來取代 `<hub name>` 和 `<connection string with full access>` 預留位置。

6. 在 [主要] 方法中新增下列程式碼行：
   
    ```csharp
    SendTemplateNotificationAsync();
    Console.ReadLine();
    ```

7. 建置主控台應用程式。

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

<!-- URLs. -->
[Get started with Notification Hubs]: ../articles/notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Notification Hubs REST interface]: http://msdn.microsoft.com/library/windowsazure/dn223264.aspx
[Add push notifications for Mobile Apps]: ../articles/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push.md
[How to use Notification Hubs from Java or PHP]: ../articles/notification-hubs/notification-hubs-java-push-notification-tutorial.md
[Microsoft.Azure.Notification 中樞 NuGet 封裝]: http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/
