---
title: 包含檔案
description: 包含檔案
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/02/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 527567ee3f3a939c7358fb6a62271cbe38e16974
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42915179"
---
當您傳送範本通知時，您只需要提供一組屬性。 在此案例中，這組屬性包含已針對目前新聞進行當地語系化的版本。

```json
{
    "News_English": "World News in English!",
    "News_French": "World News in French!",
    "News_Mandarin": "World News in Mandarin!"
}
```

### <a name="send-notifications-using-a-c-console-app"></a>使用 C# 主控台應用程式傳送通知

本節將使用主控台應用程式示範傳送通知的方式。 程式碼會將通知廣播到 Windows 市集和 iOS 裝置。 在您先前建立的主控台應用程式中，使用下列程式碼修改 `SendTemplateNotificationAsync` 方法：

```csharp
private static async void SendTemplateNotificationAsync()
{
    // Define the notification hub.
    NotificationHubClient hub = 
        NotificationHubClient.CreateClientFromConnectionString(
            "<connection string with full access>", "<hub name>");

    // Sending the notification as a template notification. All template registrations that contain 
    // "messageParam" or "News_<local selected>" and the proper tags will receive the notifications. 
    // This includes APNS, GCM, WNS, and MPNS template registrations.
    Dictionary<string, string> templateParams = new Dictionary<string, string>();

    // Create an array of breaking news categories.
    var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};
    var locales = new string[] { "English", "French", "Mandarin" };

    foreach (var category in categories)
    {
        templateParams["messageParam"] = "Breaking " + category + " News!";

        // Sending localized News for each tag too...
        foreach( var locale in locales)
        {
            string key = "News_" + locale;

            // Your real localized news content would go here.
            templateParams[key] = "Breaking " + category + " News in " + locale + "!";
        }

        await hub.SendTemplateNotificationAsync(templateParams, category);
    }
}
```

SendTemplateNotificationAsync 方法會將已當地語系化的新聞片段傳遞至**所有**裝置，無論其平台為何。 通知中樞會建置並傳遞正確的原生承載給訂閱特定標記的所有裝置。

### <a name="sending-notification-with-mobile-services"></a>使用行動服務傳送通知

在行動服務排程器中，使用下列指令碼：

```csharp
var azure = require('azure');
var notificationHubService = azure.createNotificationHubService('<hub name>', '<connection string with full access>');
var notification = {
        "News_English": "World News in English!",
        "News_French": "World News in French!",
        "News_Mandarin", "World News in Mandarin!"
}
notificationHubService.send('World', notification, function(error) {
    if (!error) {
        console.warn("Notification successful");
    }
});
```
