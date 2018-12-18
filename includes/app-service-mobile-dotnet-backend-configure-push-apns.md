---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 1b3538755233e59fb474be810e63907dfc4b8f5c
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809698"
---
**.NET 後端 (C#)**：
  
1. 在 Visual Studio 中，以滑鼠右鍵按一下伺服器專案並按一下 [管理 NuGet 封裝]，搜尋 `Microsoft.Azure.NotificationHubs`，然後按一下 [安裝]。 這會安裝通知中樞程式庫，以便從後端傳送通知。
2. 在後端的 Visual Studio 專案中，開啟 **Controllers**  >  **TodoItemController.cs**。 在檔案頂端新增下列 `using` 陳述式：

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

        // iOS payload
        var appleNotificationPayload = "{\"aps\":{\"alert\":\"" + item.Text + "\"}}";

        try
        {
            // Send the push notification and log the results.
            var result = await hub.SendAppleNativeNotificationAsync(appleNotificationPayload);

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

**Node.js 後端**：

1. 如果您還沒這麼做，請[下載快速入門專案](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart)或使用 [Azure 入口網站中的線上編輯器](../articles/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor)。    

2. 使用下列程式碼取代  todoitem.js 資料表指令碼：

    ```javascript
    var azureMobileApps = require('azure-mobile-apps'),
        promises = require('azure-mobile-apps/src/utilities/promises'),
        logger = require('azure-mobile-apps/src/logger');

    var table = azureMobileApps.table();

    // When adding record, send a push notification via APNS
    table.insert(function (context) {
        // For details of the Notification Hubs JavaScript SDK, 
        // see http://aka.ms/nodejshubs
        logger.info('Running TodoItem.insert');

        // Create a payload that contains the new item Text.
        var payload = "{\"aps\":{\"alert\":\"" + context.item.text + "\"}}";

        // Execute the insert; Push as a post-execute action when results are returned as a Promise.
        return context.execute()
            .then(function (results) {
                // Only do the push if configured
                if (context.push) {
                    context.push.apns.send(null, payload, function (error) {
                        if (error) {
                            logger.error('Error while sending push notification: ', error);
                        } else {
                            logger.info('Push notification sent successfully!');
                        }
                    });
                }
                return results;
            })
            .catch(function (error) {
                logger.error('Error while running context.execute: ', error);
            });
    });

    module.exports = table;
    ```

3. 當您在本機電腦上編輯檔案時，請重新發佈伺服器專案。
