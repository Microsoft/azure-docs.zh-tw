---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 5a5d3e13f5c5f46e82e4f1620be1840db556d30e
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809457"
---
使用符合您後端專案類型的程序 (&mdash;[.NET 後端](#dotnet)或 [Node.js 後端](#nodejs))。

### <a name="dotnet"></a>.NET 後端專案

1. In Visual Studio, right-click the server project, and click **Manage NuGet Packages**. 搜尋 `Microsoft.Azure.NotificationHubs`，然後按一下 [安裝]。 這會安裝通知中樞用戶端程式庫。
2. 在 Controllers 資料夾中，開啟 TodoItemController.cs 並新增下列 `using` 陳述式：

    ```csharp
    using Microsoft.Azure.Mobile.Server.Config;
    using Microsoft.Azure.NotificationHubs;
    ```

3. 以下列程式碼取代 `PostTodoItem` 方法：  

    ```csharp
    public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
    {
        TodoItem current = await InsertAsync(item);
        // Get the settings for the server project.
        HttpConfiguration config = this.Configuration;

        MobileAppSettingsDictionary settings =
            this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();

        // Get the Notification Hubs credentials for the Mobile App.
        string notificationHubName = settings.NotificationHubName;
        string notificationHubConnection = settings
            .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

        // Create a new Notification Hub client.
        NotificationHubClient hub = NotificationHubClient
        .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

        // Android payload
        var androidNotificationPayload = "{ \"data\" : {\"message\":\"" + item.Text + "\"}}";

        try
        {
            // Send the push notification and log the results.
            var result = await hub.SendGcmNativeNotificationAsync(androidNotificationPayload);

            // Write the success result to the logs.
            config.Services.GetTraceWriter().Info(result.State.ToString());
        }
        catch (System.Exception ex)
        {
            // Write the failure result to the logs.
            config.Services.GetTraceWriter()
                .Error(ex.Message, null, "Push.SendAsync Error");
        }
        return CreatedAtRoute("Tables", new { id = current.Id }, current);
    }
    ```

4. 發佈伺服器專案。

### <a name="nodejs"></a>Node.js 後端專案

1. 如果您還沒這麼做，請[下載快速入門專案](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart)或使用 [Azure 入口網站中的線上編輯器](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor)。
2. 在 todoitem.js 檔案中，以下列程式碼取代現有的程式碼：

    ```javascript
    var azureMobileApps = require('azure-mobile-apps'),
    promises = require('azure-mobile-apps/src/utilities/promises'),
    logger = require('azure-mobile-apps/src/logger');

    var table = azureMobileApps.table();

    table.insert(function (context) {
    // For more information about the Notification Hubs JavaScript SDK,
    // see http://aka.ms/nodejshubs
    logger.info('Running TodoItem.insert');

    // Define the GCM payload.
    var payload = {
        "data": {
            "message": context.item.text
        }
    };

    // Execute the insert.  The insert returns the results as a Promise,
    // Do the push as a post-execute action within the promise flow.
    return context.execute()
        .then(function (results) {
            // Only do the push if configured
            if (context.push) {
                // Send a GCM native notification.
                context.push.gcm.send(null, payload, function (error) {
                    if (error) {
                        logger.error('Error while sending push notification: ', error);
                    } else {
                        logger.info('Push notification sent successfully!');
                    }
                });
            }
            // Don't forget to return the results from the context.execute()
            return results;
        })
        .catch(function (error) {
            logger.error('Error while running context.execute: ', error);
        });
    });

    module.exports = table;
    ```

    插入新的 todo 項目時，這會傳送包含 item.text 的 GCM 通知。

3. 在本機電腦中編輯檔案時，重新發布伺服器專案。
