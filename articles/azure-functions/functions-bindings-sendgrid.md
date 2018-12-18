---
title: Azure Functions SendGrid 繫結
description: Azure Functions SendGrid 繫結參考。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 11/29/2017
ms.author: glenga
ms.openlocfilehash: 2796a7d9fb44a05e1873e2fcbde82c837db202be
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39346448"
---
# <a name="azure-functions-sendgrid-bindings"></a>Azure Functions SendGrid 繫結

本文說明如何在 Azure Functions 中使用 [SendGrid](https://sendgrid.com/docs/User_Guide/index.html) 繫結傳送電子郵件。 Azure Functions 支援適用於 SendGrid 的輸出繫結。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.SendGrid](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.SendGrid) NuGet 套件版本 2.x 中提供了 SendGrid 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.SendGrid/) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.SendGrid](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.SendGrid) NuGet 套件版本 3.x 中提供了 SendGrid 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.SendGrid/) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="example"></a>範例

請參閱特定語言的範例：

* [C#](#c-example)
* [C# 指令碼 (.csx)](#c-script-example)
* [JavaScript](#javascript-example)

### <a name="c-example"></a>C# 範例

下列範例說明的 [C# 函式](functions-dotnet-class-library.md)，可使用服務匯流排佇列觸發程序和 SendGrid 輸出繫結。

```cs
[FunctionName("SendEmail")]
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] OutgoingEmail email,
    [SendGrid(ApiKey = "CustomSendGridKeyAppSettingName")] out SendGridMessage message)
{
    message = new SendGridMessage();
    message.AddTo(email.To);
    message.AddContent("text/html", email.Body);
    message.SetFrom(new EmailAddress(email.From));
    message.SetSubject(email.Subject);
}

public class OutgoingEmail
{
    public string To { get; set; }
    public string From { get; set; }
    public string Subject { get; set; }
    public string Body { get; set; }
}
```

若您在名為「AzureWebJobsSendGridApiKey」的應用程式設定中有 API 金鑰，則可以省略設定屬性的 `ApiKey` 內容。

### <a name="c-script-example"></a>C# 指令碼範例

下列範例說明 *function.json* 檔案中的 SendGrid 輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。

以下是 *function.json* 檔案中的繫結資料：

```json 
{
    "bindings": [
        {
            "name": "message",
            "type": "sendGrid",
            "direction": "out",
            "apiKey" : "MySendGridKey",
            "to": "{ToEmail}",
            "from": "{FromEmail}",
            "subject": "SendGrid output bindings"
        }
    ]
}
```

[設定](#configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```csharp
#r "SendGrid"
using System;
using SendGrid.Helpers.Mail;

public static void Run(TraceWriter log, string input, out Mail message)
{
     message = new Mail
    {        
        Subject = "Azure news"          
    };

    var personalization = new Personalization();
    personalization.AddTo(new Email("recipient@contoso.com"));   

    Content content = new Content
    {
        Type = "text/plain",
        Value = input
    };
    message.AddContent(content);
    message.AddPersonalization(personalization);
}
```

### <a name="javascript-example"></a>JavaScript 範例

下列範例說明 *function.json* 檔案中的 SendGrid 輸出繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。

以下是 *function.json* 檔案中的繫結資料：

```json 
{
    "bindings": [
        {
            "name": "$return",
            "type": "sendGrid",
            "direction": "out",
            "apiKey" : "MySendGridKey",
            "to": "{ToEmail}",
            "from": "{FromEmail}",
            "subject": "SendGrid output bindings"
        }
    ]
}
```

[設定](#configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, input) {    
    var message = {
         "personalizations": [ { "to": [ { "email": "sample@sample.com" } ] } ],
        from: { email: "sender@contoso.com" },        
        subject: "Azure news",
        content: [{
            type: 'text/plain',
            value: input
        }]
    };

    context.done(null, message);
};
```

## <a name="attributes"></a>屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [SendGrid](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.SendGrid/SendGridAttribute.cs) 屬性。

如需可設定的屬性內容相關資訊，請參閱[設定](#configuration)。 以下是方法簽章中的 `SendGrid` 屬性範例：

```csharp
[FunctionName("SendEmail")]
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] OutgoingEmail email,
    [SendGrid(ApiKey = "CustomSendGridKeyAppSettingName")] out SendGridMessage message)
{
    ...
}
```

如需完整範例，請參閱 [C# 範例](#c-example)。

## <a name="configuration"></a>組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `SendGrid` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type**|| 必要項目 - 必須設定為 `sendGrid`。|
|**direction**|| 必要項目 - 必須設定為 `out`。|
|**name**|| 必要項目 - 函式程式碼中用於要求或要求主體的變數名稱。 當只有一個傳回值時，此值為 ```$return```。 |
|**apiKey**|**ApiKey**| 包含您 API 金鑰的應用程式設定名稱。 若未設定，預設應用程式設定名稱是「AzureWebJobsSendGridApiKey」。|
|**to**|**To**| 收件者的電子郵件地址。 |
|**from**|**From**| 寄件者的電子郵件地址。 |
|**subject**|**主旨**| 電子郵件主旨。 |
|**text**|**文字**| 電子郵件內容。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
