---
title: 適用於 Azure Functions 的 Mobile Apps 繫結
description: 了解如何在 Azure Functions 中使用 Azure Mobile Apps 繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 11/21/2017
ms.author: glenga
ms.openlocfilehash: 6b5a45bfcfc02c6b313e868fb3ba037ce3ba48c9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967569"
---
# <a name="mobile-apps-bindings-for-azure-functions"></a>適用於 Azure Functions 的 Mobile Apps 繫結 

> [!NOTE]
> Azure Mobile Apps 繫結僅適用於 Azure Functions 1.x。 Azure Functions 2.x 中不支援這些繫結。

本文說明如何在 Azure Functions 中使用 [Azure Mobile Apps](../app-service-mobile/app-service-mobile-value-prop.md)。 Azure Functions 支援 Mobile Apps 的輸入和輸出繫結。

Mobile Apps 繫結可讓您讀取和更新行動裝置應用程式中的資料表。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.MobileApps](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.MobileApps) NuGet 套件 1.x 版中提供 Mobile Apps 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.MobileApps/) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="input"></a>輸入

Mobile Apps 輸入繫結會從行動資料表端點載入記錄，並將它傳遞到您的函式。 在 C# 和 F# 函式中，當函式成功結束時，會將記錄所做的任何變更自動傳回資料表。

## <a name="input---example"></a>輸入 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#input---c-script-example)
* [JavaScript](#input---javascript-example)

### <a name="input---c-script-example"></a>輸入 - C# 指令碼範例

下列範例示範 function.json 檔案中的 Mobile Apps 輸入繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會由包含記錄識別碼的佇列訊息觸發。 函式會讀取指定的記錄並修改其 `Text` 屬性。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "name": "myQueueItem",
    "queueName": "myqueue-items",
    "connection":"",
    "type": "queueTrigger",
    "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id" : "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
[設定](#input---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
#r "Newtonsoft.Json"    
using Newtonsoft.Json.Linq;

public static void Run(string myQueueItem, JObject record)
{
    if (record != null)
    {
        record["Text"] = "This has changed.";
    }    
}
```

### <a name="input---javascript"></a>輸入 - JavaScript

下列範例示範 function.json 檔案中的 Mobile Apps 輸入繫結，以及使用此繫結的 [JavaScript 指令碼函式](functions-reference-node.md)。 函式會由包含記錄識別碼的佇列訊息觸發。 函式會讀取指定的記錄並修改其 `Text` 屬性。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "name": "myQueueItem",
    "queueName": "myqueue-items",
    "connection":"",
    "type": "queueTrigger",
    "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id" : "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
[設定](#input---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, myQueueItem) {    
    context.log(context.bindings.record);
    context.done();
};
```

## <a name="input---attributes"></a>輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) 屬性。

如需您可以設定之屬性內容的相關資訊，請參閱[下列組態區段](#input---configuration)。

## <a name="input---configuration"></a>輸入 - 組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `MobileTable` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
| **type**|| 必須設定為 "mobileTable"|
| **direction**||必須設定為 "in"|
| **name**|| 函式簽章中的輸入參數名稱。|
|**tableName** |**TableName**|行動裝置應用程式資料表的名稱|
| **id**| **Id** | 要擷取之記錄的識別碼。 可以是靜態，或以叫用函式的觸發程序作為基礎。 例如，如果您對函式使用佇列觸發程序，`"id": "{queueTrigger}"` 就會使用佇列訊息的字串值作為要擷取的記錄識別碼。|
|**連接**|**連接**|包含行動裝置應用程式 URL 的應用程式設定名稱。 函式會使用此 URL 針對您的行動裝置應用程式建構所需的 REST 作業。 在包含您的行動裝置應用程式 URL 的函式應用程式中建立應用程式設定，然後在輸入繫結的 `connection` 屬性中，指定應用程式設定的名稱。 URL 看起來像這樣：`http://<appname>.azurewebsites.net`。
|**apiKey**|**ApiKey**|包含行動裝置應用程式 API 金鑰的應用程式設定名稱。 如果您[在您的 Node.js 行動裝置應用程式中實作 API 金鑰](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key)，或[在您的 .NET 行動裝置應用程式中實作 API 金鑰](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key)，請提供 API 金鑰。 請提供金鑰，在包含 API 金鑰的函式應用程式中建立應用程式設定，然後在具有應用程式設定名稱的輸入繫結中新增 `apiKey` 屬性。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> 請勿與行動裝置應用程式用戶端共用 API 金鑰。 只應該安全地散佈給服務端用戶端，如 Azure Functions。 Azure Functions 將會您的連接資訊和 API 金鑰儲存為應用程式設定，使得不會將讓它們簽入至您的原始檔控制儲存機制。 這可保護您的敏感資訊。

## <a name="input---usage"></a>輸入 - 使用方式

在 C# 函式中，找到具有指定識別碼的記錄時，它會傳遞到名為 [JObject](http://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm) 的參數。 找不到記錄時，參數值為 `null`。 

在 JavaScript 函式中，記錄會傳遞至 `context.bindings.<name>` 物件。 找不到記錄時，參數值為 `null`。 

在 C# 和 F# 函式中，當函式成功結束時，會將對輸入記錄 (輸入參數) 所做的任何變更自動傳送回資料表。 您無法修改 JavaScript 函式中的記錄。

## <a name="output"></a>輸出

使用 Mobile Apps 輸出繫結將新記錄寫入至 Mobile Apps 資料表。  

## <a name="output---example"></a>輸出 - 範例

請參閱特定語言的範例：

* [C#](#output---c-example)
* [C# 指令碼 (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)

### <a name="output---c-example"></a>輸出 - C# 範例

下列範例示範的 [C# 函式](functions-dotnet-class-library.md)是由佇列訊息觸發，並在行動應用程式資料表中建立記錄。

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    return new { Text = $"I'm running in a C# function! {myQueueItem}" };
}
```

### <a name="output---c-script-example"></a>輸出 - C# 指令碼範例

下列範例示範 function.json 檔案中的 Mobile Apps 輸出繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會由佇列訊息觸發，並使用 `Text` 屬性的硬式編碼值來建立新的記錄。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "name": "myQueueItem",
    "queueName": "myqueue-items",
    "connection":"",
    "type": "queueTrigger",
    "direction": "in"
    },
    {
    "name": "record",
    "type": "mobileTable",
    "tableName": "MyTable",
    "connection": "My_MobileApp_Url",
    "apiKey": "My_MobileApp_Key",
    "direction": "out"
    }
]
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 C# 指令碼程式碼：

```cs
public static void Run(string myQueueItem, out object record)
{
    record = new {
        Text = $"I'm running in a C# function! {myQueueItem}"
    };
}
```

### <a name="output---javascript-example"></a>輸出 - JavaScript 範例

下列範例示範 function.json 檔案中的 Mobile Apps 輸出繫結，以及使用此繫結的 [JavaScript 指令碼函式](functions-reference-node.md)。 函式會由佇列訊息觸發，並使用 `Text` 屬性的硬式編碼值來建立新的記錄。

以下是 *function.json* 檔案中的繫結資料：

```json
{
"bindings": [
    {
    "name": "myQueueItem",
    "queueName": "myqueue-items",
    "connection":"",
    "type": "queueTrigger",
    "direction": "in"
    },
    {
    "name": "record",
    "type": "mobileTable",
    "tableName": "MyTable",
    "connection": "My_MobileApp_Url",
    "apiKey": "My_MobileApp_Key",
    "direction": "out"
    }
],
"disabled": false
}
```

[設定](#output---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function (context, myQueueItem) {

    context.bindings.record = {
        text : "I'm running in a Node function! Data: '" + myQueueItem + "'"
    }   

    context.done();
};
```

## <a name="output---attributes"></a>輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) 屬性。

如需可設定的屬性內容相關資訊，請參閱[輸出 - 組態](#output---configuration)。 以下是方法簽章中的 `MobileTable` 屬性範例：

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    ...
}
```

如需完整範例，請參閱[輸出 - C# 範例](#output---c-example)。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `MobileTable` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
| **type**|| 必須設定為 "mobileTable"|
| **direction**||必須設定為 "out"|
| **name**|| 函式簽章中的輸出參數名稱。|
|**tableName** |**TableName**|行動裝置應用程式資料表的名稱|
|**連接**|**MobileAppUriSetting**|包含行動裝置應用程式 URL 的應用程式設定名稱。 函式會使用此 URL 針對您的行動裝置應用程式建構所需的 REST 作業。 在包含您的行動裝置應用程式 URL 的函式應用程式中建立應用程式設定，然後在輸入繫結的 `connection` 屬性中，指定應用程式設定的名稱。 URL 看起來像這樣：`http://<appname>.azurewebsites.net`。
|**apiKey**|**ApiKeySetting**|包含行動裝置應用程式 API 金鑰的應用程式設定名稱。 如果您[在您的 Node.js 行動裝置應用程式後端中實作 API 金鑰](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key)，或[在您的 .NET 行動裝置應用程式後端中實作 API 金鑰](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key)，請提供 API 金鑰。 請提供金鑰，在包含 API 金鑰的函式應用程式中建立應用程式設定，然後在具有應用程式設定名稱的輸入繫結中新增 `apiKey` 屬性。 |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> 請勿與行動裝置應用程式用戶端共用 API 金鑰。 只應該安全地散佈給服務端用戶端，如 Azure Functions。 Azure Functions 將會您的連接資訊和 API 金鑰儲存為應用程式設定，使得不會將讓它們簽入至您的原始檔控制儲存機制。 這可保護您的敏感資訊。

## <a name="output---usage"></a>輸出 - 使用方式

在 C# 指令碼函式中，使用類型 `out object` 的具名輸出參數來存取輸出記錄。 在 C# 類別庫中，`MobileTable` 屬性可與下列任何類型搭配使用：

* `ICollector<T>` 或 `IAsyncCollector<T>`，其中 `T` 為 `JObject` 或任何包含 `public string Id` 屬性的類型。
* `out JObject`
* `out T` 或 `out T[]`，其中 `T` 為任何包含 `public string Id` 屬性的類型。

在 Node.js 函式中，使用 `context.bindings.<name>` 來存取輸出記錄。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
