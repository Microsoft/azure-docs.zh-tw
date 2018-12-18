---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 0e7118ff6a2860351a7bfa38637f1d767b0f4a2d
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42816221"
---
在本節中，您會更新現有 Mobile Apps 後端專案中的程式碼，以在每次新增項目時傳送推播通知。 此程序是由 Azure 通知中樞的[範本](../articles/notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)功能所提供，可啟用跨平台推播。 各種用戶端會使用範本來註冊推播通知，而單一通用推播即可送達所有的用戶端平台。

請選擇下列其中一個符合您後端專案類型(&mdash;[.NET 後端](#dotnet)或 [Node.js 後端](#nodejs))的程序。

### <a name="dotnet"></a>.NET 後端專案

1. 在 Visual Studio 中，以滑鼠右鍵按一下伺服器專案。 然後選取 [管理 NuGet 套件] 。 搜尋 `Microsoft.Azure.NotificationHubs`，然後選取 [安裝]。 此程序會安裝通知中樞程式庫，以便從後端傳送通知。
2. 在伺服器專案中，開啟 [控制器]**Controllers** > [TodoItemController.cs]。 然後新增下列 using 陳述式：

    ```csharp
    using System.Collections.Generic;
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Azure.Mobile.Server.Config;
    ```

3. 在 **PostTodoItem** 方法中，於呼叫 **InsertAsync** 之後新增下列程式碼：  

    ```csharp
    // Get the settings for the server project.
    HttpConfiguration config = this.Configuration;
    MobileAppSettingsDictionary settings =
        this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();

    // Get the Notification Hubs credentials for the mobile app.
    string notificationHubName = settings.NotificationHubName;
    string notificationHubConnection = settings
        .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

    // Create a new Notification Hub client.
    NotificationHubClient hub = NotificationHubClient
    .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

    // Send the message so that all template registrations that contain "messageParam"
    // receive the notifications. This includes APNS, GCM, WNS, and MPNS template registrations.
    Dictionary<string,string> templateParams = new Dictionary<string,string>();
    templateParams["messageParam"] = item.Text + " was added to the list.";

    try
    {
        // Send the push notification and log the results.
        var result = await hub.SendTemplateNotificationAsync(templateParams);

        // Write the success result to the logs.
        config.Services.GetTraceWriter().Info(result.State.ToString());
    }
    catch (System.Exception ex)
    {
        // Write the failure result to the logs.
        config.Services.GetTraceWriter()
            .Error(ex.Message, null, "Push.SendAsync Error");
    }
    ```

    插入新的項目時，此程序會傳送包含 item.Text 的範本通知。

4. 發佈伺服器專案。

### <a name="nodejs"></a>Node.js 後端專案

1. 如果您還沒這麼做，請[下載快速入門後端專案](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor)或使用 [Azure 入口網站中的線上編輯器](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart)。
2. 在 todoitem.js 中，以下列程式碼取代現有的程式碼：

    ```javascript
    var azureMobileApps = require('azure-mobile-apps'),
    promises = require('azure-mobile-apps/src/utilities/promises'),
    logger = require('azure-mobile-apps/src/logger');

    var table = azureMobileApps.table();

    table.insert(function (context) {
    // For more information about the Notification Hubs JavaScript SDK,
    // see http://aka.ms/nodejshubs.
    logger.info('Running TodoItem.insert');

    // Define the template payload.
    var payload = '{"messageParam": "' + context.item.text + '" }';  

    // Execute the insert. The insert returns the results as a promise.
    // Do the push as a post-execute action within the promise flow.
    return context.execute()
        .then(function (results) {
            // Only do the push if configured.
            if (context.push) {
                // Send a template notification.
                context.push.send(null, payload, function (error) {
                    if (error) {
                        logger.error('Error while sending push notification: ', error);
                    } else {
                        logger.info('Push notification sent successfully!');
                    }
                });
            }
            // Don't forget to return the results from the context.execute().
            return results;
        })
        .catch(function (error) {
            logger.error('Error while running context.execute: ', error);
        });
    });

    module.exports = table;  
    ```

    插入新的項目時，此程序會傳送包含 item.Text 的範本通知。

3. 當您在本機電腦上編輯檔案時，請重新發佈伺服器專案。
