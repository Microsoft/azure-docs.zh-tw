---
title: Azure Functions 的通知中樞繫結
description: 了解如何在 Azure Functions 中使用「Azure 通知中樞」繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 11/21/2017
ms.author: glenga
ms.openlocfilehash: 4208a7be3c002bcefed273015d002cb1aee0fecd
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38467659"
---
# <a name="notification-hubs-output-binding-for-azure-functions"></a>Azure Functions 的通知中樞輸出繫結

這篇文章說明如何在 Azure Functions 中使用 [Azure 通知中樞](../notification-hubs/notification-hubs-push-notification-overview.md)繫結來傳送推播通知。 Azure Functions 會支援通知中樞的輸出繫結。

必須為您要使用的平台通知服務 (PNS) 設定 Azure 通知中樞。 若要了解如何在用戶端應用程式中收到通知中樞的推播通知，請參閱[開始使用通知中心](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)，並從靠近頁面頂端的下拉式清單中選取目標用戶端平台。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.NotificationHubs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.NotificationHubs) NuGet 套件 1.x 版中提供通知中樞繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.NotificationHubs) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

Functions 2.x 中不提供此繫結。

## <a name="example---template"></a>範例 - 範本

您傳送的通知可以是原生通知或[範本通知](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)。 原生通知是以輸出繫結的 `platform` 屬性中所設定的特定用戶端平台為目標。 範本通知可用來以多個平台為目標。   

請參閱特定語言的範例：

* [C# 指令碼 - out 參數](#c-script-template-example---out-parameter)
* [C# 指令碼 - 非同步](#c-script-template-example---asynchronous)
* [C# 指令碼 - JSON](#c-script-template-example---json)
* [C# 指令碼 - 程式庫類型](#c-script-template-example---library-types)
* [F#](#f-template-example)
* [JavaScript](#javascript-template-example)

### <a name="c-script-template-example---out-parameter"></a>C# 指令碼範本範例 - out 參數

這個範例會傳送在範本中包含 `message` 預留位置的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

```cs
using System;
using System.Threading.Tasks;
using System.Collections.Generic;

public static void Run(string myQueueItem,  out IDictionary<string, string> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    notification = GetTemplateProperties(myQueueItem);
}

private static IDictionary<string, string> GetTemplateProperties(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["message"] = message;
    return templateProperties;
}
```

### <a name="c-script-template-example---asynchronous"></a>C# 指令碼範本範例 - 非同步

如果您使用非同步程式碼，則不允許使用 out 參數。 在此情況下，請使用 `IAsyncCollector` 來傳回您的範本通知。 下列程式碼是上述程式碼的非同步範例。 

```cs
using System;
using System.Threading.Tasks;
using System.Collections.Generic;

public static async Task Run(string myQueueItem, IAsyncCollector<IDictionary<string,string>> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    log.Info($"Sending Template Notification to Notification Hub");
    await notification.AddAsync(GetTemplateProperties(myQueueItem));    
}

private static IDictionary<string, string> GetTemplateProperties(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["user"] = "A new user wants to be added : " + message;
    return templateProperties;
}
```

### <a name="c-script-template-example---json"></a>C# 指令碼範本範例 - JSON

這個範例會使用有效的 JSON 字串來傳送在範本中包含 `message` 預留位置的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

```cs
using System;

public static void Run(string myQueueItem,  out string notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    notification = "{\"message\":\"Hello from C#. Processed a queue item!\"}";
}
```

### <a name="c-script-template-example---library-types"></a>C# 指令碼範本範例 - 程式庫類型

這個範例示範如何使用 [Microsoft Azure 通知中樞程式庫](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)中定義的類型。 

```cs
#r "Microsoft.Azure.NotificationHubs"

using System;
using System.Threading.Tasks;
using Microsoft.Azure.NotificationHubs;

public static void Run(string myQueueItem,  out Notification notification, TraceWriter log)
{
   log.Info($"C# Queue trigger function processed: {myQueueItem}");
   notification = GetTemplateNotification(myQueueItem);
}

private static TemplateNotification GetTemplateNotification(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["message"] = message;
    return new TemplateNotification(templateProperties);
}
```

### <a name="f-template-example"></a>F # 範本範例

這個範例會傳送包含 `location` 和 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

```fsharp
let Run(myTimer: TimerInfo, notification: byref<IDictionary<string, string>>) =
    notification = dict [("location", "Redmond"); ("message", "Hello from F#!")]
```

### <a name="javascript-template-example"></a>JavaScript 範本範例

這個範例會傳送包含 `location` 和 `message` 的[範本註冊](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)通知。

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if(myTimer.isPastDue)
    {
        context.log('Node.js is running late!');
    }
    context.log('Node.js timer trigger function ran!', timeStamp);  
    context.bindings.notification = {
        location: "Redmond",
        message: "Hello from Node!"
    };
    context.done();
};
```

## <a name="example---apns-native"></a>範例 - APNS 原生

這個 C# 指令碼範例會示範如何傳送原生 APNS 通知。 

```cs
#r "Microsoft.Azure.NotificationHubs"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.NotificationHubs;
using Newtonsoft.Json;

public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a new user to be processed in the form of a JSON string with 
    // a "name" value.
    //
    // The JSON format for a native APNS notification is ...
    // { "aps": { "alert": "notification message" }}  

    log.Info($"Sending APNS notification of a new user");    
    dynamic user = JsonConvert.DeserializeObject(myQueueItem);    
    string apnsNotificationPayload = "{\"aps\": {\"alert\": \"A new user wants to be added (" + 
                                        user.name + ")\" }}";
    log.Info($"{apnsNotificationPayload}");
    await notification.AddAsync(new AppleNotification(apnsNotificationPayload));        
}
```

## <a name="example---gcm-native"></a>範例 - GCM 原生

這個 C# 指令碼範例會示範如何傳送原生 GCM 通知。 

```cs
#r "Microsoft.Azure.NotificationHubs"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.NotificationHubs;
using Newtonsoft.Json;

public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a new user to be processed in the form of a JSON string with 
    // a "name" value.
    //
    // The JSON format for a native GCM notification is ...
    // { "data": { "message": "notification message" }}  

    log.Info($"Sending GCM notification of a new user");    
    dynamic user = JsonConvert.DeserializeObject(myQueueItem);    
    string gcmNotificationPayload = "{\"data\": {\"message\": \"A new user wants to be added (" + 
                                        user.name + ")\" }}";
    log.Info($"{gcmNotificationPayload}");
    await notification.AddAsync(new GcmNotification(gcmNotificationPayload));        
}
```

## <a name="example---wns-native"></a>範例 - WNS 原生

這個 C# 指令碼範例示範如何使用 [Microsoft Azure 通知中樞程式庫](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)中定義的類型來傳送原生 WNS 快顯通知。 

```cs
#r "Microsoft.Azure.NotificationHubs"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.NotificationHubs;
using Newtonsoft.Json;

public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a new user to be processed in the form of a JSON string with 
    // a "name" value.
    //
    // The XML format for a native WNS toast notification is ...
    // <?xml version="1.0" encoding="utf-8"?>
    // <toast>
    //      <visual>
    //     <binding template="ToastText01">
    //       <text id="1">notification message</text>
    //     </binding>
    //   </visual>
    // </toast>

    log.Info($"Sending WNS toast notification of a new user");    
    dynamic user = JsonConvert.DeserializeObject(myQueueItem);    
    string wnsNotificationPayload = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                                    "<toast><visual><binding template=\"ToastText01\">" +
                                        "<text id=\"1\">" + 
                                            "A new user wants to be added (" + user.name + ")" + 
                                        "</text>" +
                                    "</binding></visual></toast>";

    log.Info($"{wnsNotificationPayload}");
    await notification.AddAsync(new WindowsNotification(wnsNotificationPayload));        
}
```

## <a name="attributes"></a>屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [NotificationHub](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs) 屬性。

屬性的建構函式參數和屬性會在[組態](#configuration)一節中加以敘述。

## <a name="configuration"></a>組態

下表說明您在 function.json 檔案中設定的繫結設定屬性內容和 `NotificationHub` 屬性：

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** |n/a| 必須設定為 "notificationHub"。 |
|**direction** |n/a| 必須設定為 "out"。 | 
|**name** |n/a| 函式程式碼中用於通知中樞訊息的變數名稱。 |
|**tagExpression** |**TagExpression** | 標籤運算式可讓您指定將通知傳遞到一組裝置，這些裝置已註冊要接收與標籤運算式相符的通知。  如需詳細資訊，請參閱 [路由與標籤運算式](../notification-hubs/notification-hubs-tags-segment-push-message.md)。 |
|**hubName** | **HubName** | Azure 入口網站中通知中樞資源的名稱。 |
|**連接** | **ConnectionStringSetting** | 包含「通知中樞」連接字串的應用程式設定名稱。  必須針對通知中樞將連接字串設為 DefaultFullSharedAccessSignature 值。 請參閱本文稍後的[連線字串設定](#connection-string-setup)。|
|**platform** | **平台** | 此平台屬性指出作為您通知目標的用戶端平台。 依照預設，如果輸出繫結中省略平台屬性，範本通知可用來以「Azure 通知中樞」上所設定的任何平台為目標。 如需有關一般使用範本搭配「Azure 通知中樞」來傳送跨平台通知的詳細資訊，請參閱[範本](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md)。 platform 在設定時必須是以下其中一個值： <ul><li><code>apns</code>&mdash;Apple Push Notification Service。 如果有關設定適用於 APNS 的通知中樞及在用戶端 App 中接收通知的詳細資訊，請參閱[使用 Azure 通知中樞將推播通知傳送至 iOS](../notification-hubs/notification-hubs-ios-apple-push-notification-apns-get-started.md)。</li><li><code>adm</code>&mdash;[Amazon 裝置通訊](https://developer.amazon.com/device-messaging)。 如果有關設定適用於 ADM 的通知中樞及在 Kindle App 中接收通知的詳細資訊，請參閱[開始使用適用於 Kindle 應用程式的通知中樞](../notification-hubs/notification-hubs-kindle-amazon-adm-push-notification.md)。</li><li><code>gcm</code>&mdash;[Google 雲端通訊](https://developers.google.com/cloud-messaging/)。 也支援 Firebase Cloud Messaging (新版 GCM)。 如需詳細資訊，請參閱[使用 Azure 通知中樞將推播通知傳送至 Android](../notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started.md)。</li><li><code>wns</code>&mdash;以 Windows 平台為目標的 [Windows 推播通知服務](https://msdn.microsoft.com/windows/uwp/controls-and-patterns/tiles-and-notifications-windows-push-notification-services--wns--overview)。 WNS 也支援 Windows Phone 8.1 和更新版本。 如需詳細資訊，請參閱[開始使用適用於 Windows 通用平台應用程式的通知中樞](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)。</li><li><code>mpns</code>&mdash;[Microsoft 推播通知服務](https://msdn.microsoft.com/library/windows/apps/ff402558.aspx)。 此平台支援 Windows Phone 8 和舊版 Windows Phone 平台。 如需詳細資訊，請參閱[在 Windows Phone 上使用 Azure 通知中樞傳送推播通知](../notification-hubs/notification-hubs-windows-mobile-push-notifications-mpns.md)。</li></ul> |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

### <a name="functionjson-file-example"></a>function.json 檔案範例

以下是 function.json 檔案中「通知中樞」繫結的範例。

```json
{
  "bindings": [
    {
      "type": "notificationHub",
      "direction": "out",
      "name": "notification",
      "tagExpression": "",
      "hubName": "my-notification-hub",
      "connection": "MyHubConnectionString",
      "platform": "gcm"
    }
  ],
  "disabled": false
}
```

### <a name="connection-string-setup"></a>連接字串設定

若要使用通知中樞輸出繫結，必須設定中樞的連接字串。 您可以直接從 Azure 入口網站中的 [整合] 索引標籤選取現有通知中樞或建立新的通知中樞。 您也可以手動設定連接字串。 

若要將連接字串設定為現有通知中樞：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)中的通知中樞，選擇 [存取原則]，然後選取 **DefaultFullSharedAccessSignature** 原則旁邊的 [複製] 按鈕。 這會將 DefaultFullSharedAccessSignature 原則的連接字串複製到通知中樞。 這個連接字串可讓您的函式將通知訊息傳送至中樞。
    ![複製通知中樞連接字串](./media/functions-bindings-notification-hubs/get-notification-hub-connection.png)
1. 瀏覽至 Azure 入口網站中的函式應用程式，選擇 [應用程式設定]，新增 **MyHubConnectionString** 之類的金鑰，貼上針對通知中樞所複製的 DefaultFullSharedAccessSignature 來作為值，然後按一下 [儲存]。

此應用程式設定的名稱是 function.json 或 .NET 屬性中輸出繫結連線設定的內容。 請參閱本文中稍早的[組態區段](#configuration)。

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="exceptions-and-return-codes"></a>例外狀況和傳回碼

| 繫結 | 參考 |
|---|---|
| 通知中樞 | [操作指南](https://docs.microsoft.com/rest/api/notificationhubs/) \(英文\) |

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

