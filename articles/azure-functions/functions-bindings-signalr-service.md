---
title: Azure Functions SignalR Service 繫結
description: 了解如何搭配使用 SignalR Service 繫結與 Azure Functions。
services: functions
documentationcenter: na
author: anthonychu
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 09/23/2018
ms.author: antchu
ms.openlocfilehash: 435a581b065b94399006e60fec8d007296734373
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993947"
---
# <a name="signalr-service-bindings-for-azure-functions"></a>適用於 Azure Functions 的 SignalR Service 繫結

本文說明如何藉由在 Azure Functions 中使用 SignalR Service 繫結，來驗證及傳送即時訊息給連線至 [Azure SignalR Service](https://azure.microsoft.com/services/signalr-service/) 的用戶端。 Azure Functions 支援 SignalR Service 的輸入和輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.SignalRService](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.SignalRService) NuGet 套件版本 1.0.0-preview1-* 中提供了 SignalR Service 繫結。 套件的原始程式碼位於 [azure-functions-signalrservice-extension](https://github.com/Azure/azure-functions-signalrservice-extension) GitHub 存放庫中。

> [!NOTE]
> Azure SignalR Service 已正式推出。 不過，適用於 Azure Functions 的 SignalR Service 繫結目前處於預覽狀態。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]


## <a name="signalr-connection-info-input-binding"></a>SignalR 連線資訊輸入繫結

在用戶端可以連線到 Azure SignalR Service 之前，它必須擷取服務端點 URL 和有效的存取權杖。 SignalRConnectionInfo 輸入繫結會產生 SignalR Service 端點 URL 和有效的存取權杖，可用來連線到服務。 因為權杖是限時的，且可用來驗證連線的特定使用者，所以您不應該快取權杖或者在用戶端之間共用權杖。 使用此繫結的 HTTP 觸發程序可以供用戶端用來擷取連線資訊。

請參閱特定語言的範例：

* [2.x C#](#2x-c-input-example)
* [2.x JavaScript](#2x-javascript-input-example)

### <a name="2x-c-input-example"></a>2.x C# 輸入範例

下列範例示範 [C# 函式](functions-dotnet-class-library.md)，該函式會使用輸入繫結來取得 SignalR 連線資訊，並且透過 HTTP 傳回。

```cs
[FunctionName("GetSignalRInfo")]
public static SignalRConnectionInfo GetSignalRInfo(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req, 
    [SignalRConnectionInfo(HubName = "chat")]SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

#### <a name="authenticated-tokens"></a>已驗證權杖

如果函式是由已驗證的用戶端觸發，您可以將使用者識別碼宣告新增至產生的權杖。 您可以使用 [App Service 驗證] (../app-service/app-service-authentication-overview.md)，輕易地將驗證新增至函式應用程式。

App Service 驗證會設定名為 `x-ms-client-principal-id` 和 `x-ms-client-principal-name` 的 HTTP 標頭，這些標頭分別包含已驗證使用者的用戶端主體識別碼和名稱。 您可以使用[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)：`{headers.x-ms-client-principal-id}` 或 `{headers.x-ms-client-principal-name}`，將繫結的 `UserId` 屬性設為任一標頭的值。 

```cs
[FunctionName("GetSignalRInfo")]
public static SignalRConnectionInfo GetSignalRInfo(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req, 
    [SignalRConnectionInfo
        (HubName = "chat", UserId = "{headers.x-ms-client-principal-id}")]
        SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier claim set to the authenticated user
    return connectionInfo;
}
```

### <a name="2x-javascript-input-example"></a>2.x JavaScript 輸入範例

下列範例示範 function.json 檔案中的 SignalR 連線資訊輸入繫結，以及使用繫結來傳回連線資訊的 [JavaScript 函式](functions-reference-node.md)。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req, connectionInfo) {
    context.res = { body: connectionInfo };
    context.done();
};
```

#### <a name="authenticated-tokens"></a>已驗證權杖

如果函式是由已驗證的用戶端觸發，您可以將使用者識別碼宣告新增至產生的權杖。 您可以使用 [App Service 驗證] (../app-service/app-service-authentication-overview.md)，輕易地將驗證新增至函式應用程式。

App Service 驗證會設定名為 `x-ms-client-principal-id` 和 `x-ms-client-principal-name` 的 HTTP 標頭，這些標頭分別包含已驗證使用者的用戶端主體識別碼和名稱。 您可以使用[繫結運算式](functions-triggers-bindings.md#binding-expressions-and-patterns)：`{headers.x-ms-client-principal-id}` 或 `{headers.x-ms-client-principal-name}`，將繫結的 `userId` 屬性設為任一標頭的值。 

function.json 範例：

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req, connectionInfo) {
    // connectionInfo contains an access key token with a name identifier 
    // claim set to the authenticated user
    context.res = { body: connectionInfo };
    context.done();
};
```

## <a name="signalr-output-binding"></a>SignalR 輸出繫結

若使用 SignalR 輸出繫結，即可使用 Azure SignalR Service 來傳送一或多則訊息。 您可以將訊息廣播到所有已連線的用戶端，或者您可以只將訊息廣播到已對指定使用者驗證的已連線用戶端。

請參閱特定語言的範例：

* [2.x C#](#2x-c-output-example)
* [2.x JavaScript](#2x-javascript-output-example)

### <a name="2x-c-output-example"></a>2.x C# 輸出範例

#### <a name="broadcast-to-all-clients"></a>廣播到所有用戶端

下列範例示範 [C# 函式](functions-dotnet-class-library.md)，該函式會使用連結至所有已連線用戶端的輸出繫結，來傳送訊息。 `Target` 是要在每個用戶端上叫用的方法名稱。 `Arguments` 屬性是要傳遞至用戶端方法的零或多個物件陣列。

```cs
[FunctionName("SendMessage")]
public static Task SendMessage(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]object message, 
    [SignalR(HubName = "chat")]IAsyncCollector<SignalRMessage> signalRMessages)
{
    return signalRMessages.AddAsync(
        new SignalRMessage 
        {
            Target = "newMessage", 
            Arguments = new [] { message } 
        });
}
```

#### <a name="send-to-a-user"></a>傳送給使用者

您可以藉由設定 SignalR 訊息的 `UserId` 屬性，僅將訊息傳送給已對使用者驗證的連線。

```cs
[FunctionName("SendMessage")]
public static Task SendMessage(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]object message, 
    [SignalR(HubName = "chat")]IAsyncCollector<SignalRMessage> signalRMessages)
{
    return signalRMessages.AddAsync(
        new SignalRMessage 
        {
            // the message will only be sent to these user IDs
            UserId = "userId1",
            Target = "newMessage", 
            Arguments = new [] { message } 
        });
}
```

### <a name="2x-javascript-output-example"></a>2.x JavaScript 輸出範例

#### <a name="broadcast-to-all-clients"></a>廣播到所有用戶端

下列範例示範 function.json 檔案中的 SignalR 輸出繫結，以及透過 Azure SignalR Service 使用繫結來傳送訊息的 [JavaScript 函式](functions-reference-node.md)。 將輸出繫結設為一或多則 SignalR 訊息的陣列。 SignalR 訊息是由 `target` 屬性 (會指定要在每個用戶端上叫用的方法名稱) 和 `arguments` 屬性 (要傳遞至用戶端方法作為引數的物件陣列) 組成。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
  "type": "signalR",
  "name": "signalRMessages",
  "hubName": "<hub_name>",
  "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
  "direction": "out"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req) {
    context.bindings.signalRMessages = [{
        "target": "newMessage",
        "arguments": [ req.body ]
    }];
    context.done();
};
```

#### <a name="send-to-a-user"></a>傳送給使用者

您可以藉由設定 SignalR 訊息的 `userId` 屬性，僅將訊息傳送給已對使用者驗證的連線。

function.json 維持不變。 以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, req) {
    context.bindings.signalRMessages = [{
        // message will only be sent to these user IDs
        "userId": "userId1",
        "target": "newMessage",
        "arguments": [ req.body ]
    }];
    context.done();
};
```

## <a name="configuration"></a>組態

### <a name="signalrconnectioninfo"></a>SignalRConnectionInfo

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `SignalRConnectionInfo` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**|| 必須設為 `signalRConnectionInfo`。|
|**direction**|| 必須設為 `in`。|
|**name**|| 函式程式碼中用於連線資訊物件的變數名稱。 |
|**hubName**|**HubName**| 此值必須設為 SignalR 中樞 (針對該中樞產生連線資訊) 的名稱。|
|**userId**|UserId| 選擇性：要在存取金鑰權杖中設定的使用者識別碼宣告值。 |
|**connectionStringSetting**|**ConnectionStringSetting**| 包含 SignalR Service 連接字串 (預設值為 "AzureSignalRConnectionString") 的應用程式設定名稱 |

### <a name="signalr"></a>SignalR

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `SignalR` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**|| 必須設為 `signalR`。|
|**direction**|| 必須設為 `out`。|
|**name**|| 函式程式碼中用於連線資訊物件的變數名稱。 |
|**hubName**|**HubName**| 此值必須設為 SignalR 中樞 (針對該中樞產生連線資訊) 的名稱。|
|**connectionStringSetting**|**ConnectionStringSetting**| 包含 SignalR Service 連接字串 (預設值為 "AzureSignalRConnectionString") 的應用程式設定名稱 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

