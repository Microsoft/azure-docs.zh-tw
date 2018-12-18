---
title: Azure Functions Twilio 繫結
description: 了解如何搭配使用 Twilio 繫結與 Azure Functions。
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
ms.date: 07/09/2018
ms.author: glenga
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8d973e4003f8501fb3b4d1b461703bd0b869c205
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669149"
---
# <a name="twilio-binding-for-azure-functions"></a>Azure Functions 的 Twilio 繫結

本文說明如何在 Azure Functions 中使用 [Twilio](https://www.twilio.com/) 繫結傳送文字簡訊。 Azure Functions 支援 Twilio 的輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.Twilio](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Twilio) NuGet 套件 1.x 版中提供了 Twilio 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.Twilio/) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Twilio](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Twilio) NuGet 套件 3.x 版中提供了 Twilio 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="example---functions-1x"></a>範例 - Functions 1.x

請參閱特定語言的範例：

* [C#](#c-example)
* [C# 指令碼 (.csx)](#c-script-example)
* [JavaScript](#javascript-example)

### <a name="c-example"></a>C# 範例

下列範例顯示可在受到佇列訊息觸發時傳送文字簡訊的 [C# 函式](functions-dotnet-class-library.md)。

```cs
[FunctionName("QueueTwilio")]
[return: TwilioSms(AccountSidSetting = "TwilioAccountSid", AuthTokenSetting = "TwilioAuthToken", From = "+1425XXXXXXX" )]
public static SMSMessage Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] JObject order,
    TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {order}");

    var message = new SMSMessage()
    {
        Body = $"Hello {order["name"]}, thanks for your order!",
        To = order["mobileNumber"].ToString()
    };

    return message;
}
```

本範例會使用含有方法傳回值的 `TwilioSms` 屬性。 替代方式是使用具有 `out SMSMessage` 參數或 `ICollector<SMSMessage>` 或 `IAsyncCollector<SMSMessage>` 參數的屬性。

### <a name="c-script-example"></a>C# 指令碼範例

下列範例說明 *function.json* 檔案中的 Twilio 輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會使用 `out` 參數來傳送文字簡訊。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSid": "TwilioAccountSid",
  "authToken": "TwilioAuthToken",
  "to": "+1704XXXXXXX",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

以下是 C# 指令碼：

```cs
#r "Newtonsoft.Json"
#r "Twilio.Api"

using System;
using Newtonsoft.Json;
using Twilio;

public static void Run(string myQueueItem, out SMSMessage message,  TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a 
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least 
    // initialize the SMSMessage variable.
    message = new SMSMessage();

    // A dynamic message can be set instead of the body in the output binding. In this example, we use 
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    message.Body = msg;
    message.To = order.mobileNumber;
}
```

您無法使用非同步程式碼中的 out 參數。 以下是非同步的 C# 指令碼範例：

```cs
#r "Newtonsoft.Json"
#r "Twilio.Api"

using System;
using Newtonsoft.Json;
using Twilio;

public static async Task Run(string myQueueItem, IAsyncCollector<SMSMessage> message,  TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a 
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least 
    // initialize the SMSMessage variable.
    SMSMessage smsText = new SMSMessage();

    // A dynamic message can be set instead of the body in the output binding. In this example, we use 
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    smsText.Body = msg;
    smsText.To = order.mobileNumber;

    await message.AddAsync(smsText);
}
```

### <a name="javascript-example"></a>JavaScript 範例

下列範例說明 *function.json* 檔案中的 Twilio 輸出繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSid": "TwilioAccountSid",
  "authToken": "TwilioAuthToken",
  "to": "+1704XXXXXXX",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);

    // In this example the queue item is a JSON string representing an order that contains the name of a 
    // customer and a mobile number to send text updates to.
    var msg = "Hello " + myQueueItem.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least 
    // initialize the message binding.
    context.bindings.message = {};

    // A dynamic message can be set instead of the body in the output binding. In this example, we use 
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    context.bindings.message = {
        body : msg,
        to : myQueueItem.mobileNumber
    };

    context.done();
};
```

## <a name="example---functions-2x"></a>範例 - Functions 2.x

請參閱特定語言的範例：

* [2.x C#](#2x-c-example)
* [2.x C# 指令碼 (.csx)](#2x-c-script-example)
* [2.x JavaScript](#2x-javascript-example)

### <a name="2x-c-example"></a>2.x C# 範例

下列範例顯示可在受到佇列訊息觸發時傳送文字簡訊的 [C# 函式](functions-dotnet-class-library.md)。

```cs
[FunctionName("QueueTwilio")]
[return: TwilioSms(AccountSidSetting = "TwilioAccountSid", AuthTokenSetting = "TwilioAuthToken", From = "+1425XXXXXXX" )]
public static CreateMessageOptions Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] JObject order,
    TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {order}");

    var message = new CreateMessageOptions(new PhoneNumber(order["mobileNumber"].ToString()))
    {
        Body = $"Hello {order["name"]}, thanks for your order!"
    };

    return message;
}
```

本範例會使用含有方法傳回值的 `TwilioSms` 屬性。 替代方式是使用具有 `out CreateMessageOptions` 參數或 `ICollector<CreateMessageOptions>` 或 `IAsyncCollector<CreateMessageOptions>` 參數的屬性。

### <a name="2x-c-script-example"></a>2.x C# 指令碼範例

下列範例說明 *function.json* 檔案中的 Twilio 輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會使用 `out` 參數來傳送文字簡訊。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSid": "TwilioAccountSid",
  "authToken": "TwilioAuthToken",
  "to": "+1704XXXXXXX",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

以下是 C# 指令碼：

```cs
#r "Newtonsoft.Json"
#r "Twilio.Api"

using System;
using Newtonsoft.Json;
using Twilio.Rest.Api.V2010.Account;
using Twilio.Types;

public static void Run(string myQueueItem, out CreateMessageOptions message,  TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least
    // initialize the CreateMessageOptions variable.
    message = new CreateMessageOptions(new PhoneNumber("+1704XXXXXXX"));

    // A dynamic message can be set instead of the body in the output binding. In this example, we use
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    message.Body = msg;
    message.To = order.mobileNumber;
}
```

您無法使用非同步程式碼中的 out 參數。 以下是非同步的 C# 指令碼範例：

```cs
#r "Newtonsoft.Json"
#r "Twilio.Api"

using System;
using Newtonsoft.Json;
using Twilio.Rest.Api.V2010.Account;
using Twilio.Types;

public static async Task Run(string myQueueItem, IAsyncCollector<CreateMessageOptions> message,  TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least
    // initialize the CreateMessageOptions variable.
    CreateMessageOptions smsText = new CreateMessageOptions(new PhoneNumber("+1704XXXXXXX"));

    // A dynamic message can be set instead of the body in the output binding. In this example, we use
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    smsText.Body = msg;
    smsText.To = order.mobileNumber;

    await message.AddAsync(smsText);
}
```

### <a name="2x-javascript-example"></a>2.x JavaScript 範例

下列範例說明 *function.json* 檔案中的 Twilio 輸出繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。

以下是 *function.json* 檔案中的繫結資料：

function.json 範例：

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSid": "TwilioAccountSid",
  "authToken": "TwilioAuthToken",
  "to": "+1704XXXXXXX",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    var msg = "Hello " + myQueueItem.name + ", thank you for your order.";

    // Even if you want to use a hard coded message and number in the binding, you must at least
    // initialize the message binding.
    context.bindings.message = {};

    // A dynamic message can be set instead of the body in the output binding. In this example, we use
    // the order information to personalize a text message to the mobile number provided for
    // order status updates.
    context.bindings.message = {
        body : msg,
        to : myQueueItem.mobileNumber
    };

    context.done();
};
```

## <a name="attributes"></a>屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [TwilioSms](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs) 屬性。

如需可設定的屬性內容相關資訊，請參閱[設定](#configuration)。 以下是方法簽章中的 `TwilioSms` 屬性範例：

```csharp
[FunctionName("QueueTwilio")]
[return: TwilioSms(
    AccountSidSetting = "TwilioAccountSid", 
    AuthTokenSetting = "TwilioAuthToken", 
    From = "+1425XXXXXXX" )]
public static SMSMessage Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] JObject order,
    TraceWriter log)
{
    ...
}
 ```

如需完整範例，請參閱 [C# 範例](#c-example)。

## <a name="configuration"></a>組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `TwilioSms` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**|| 必須設為 `twilioSms`。|
|**direction**|| 必須設為 `out`。|
|**name**|| 用於 Twilio 簡訊文字訊息之函式程式碼中的變數名稱。 |
|**accountSid**|**AccountSid**| 此值必須設定為保留您 Twilio 帳戶 Sid 的應用程式設定名稱。|
|**authToken**|**AuthToken**| 此值必須設定為保留您 Twilio 驗證權杖的應用程式設定名稱。|
|**to**|**To**| 此值設定為簡訊文字傳送至的電話號碼。|
|**from**|**From**| 此值設定為從中傳送簡訊文字的電話號碼。|
|**body**|**內文**| 如果您不需要在函式程式碼中動態設定 SMS 文字訊息，則此值可以用來硬式編碼 SMS 文字訊息。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)