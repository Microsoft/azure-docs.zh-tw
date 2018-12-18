---
title: 適用於 Azure Functions 的 Microsoft Graph 繫結
description: 了解如何在 Azure Functions 中使用 Microsoft Graph 觸發程序和繫結。
services: functions
author: mattchenderson
manager: jeconnoc
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 12/20/2017
ms.author: mahender
ms.openlocfilehash: 3d6d4f2e3d89e1d8abf647b21e35fcdfec020b1d
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44722260"
---
# <a name="microsoft-graph-bindings-for-azure-functions"></a>適用於 Azure Functions 的 Microsoft Graph 繫結

這篇文章說明如何在 Azure Functions 中設定並使用 Microsoft Graph 觸發程序和繫結。 您可以進行這些動作，使用 Azure Functions 來處理來自 [Microsoft Graph](https://graph.microsoft.io) 的資料、深入資訊和事件。

Microsoft Graph 擴充功能會提供下列繫結：
- [驗證權杖輸入繫結](#token-input)可讓您與任何 Microsoft Graph API 進行互動。
- [Excel 資料表輸入繫結](#excel-input)可讓您從 Excel 中讀取資料。
- [Excel 資料表輸出繫結](#excel-output)可讓您修改 Excel 資料。
- [OneDrive 檔案輸入繫結](#onedrive-input)可讓您從 OneDrive 讀取檔案。
- [OneDrive 檔案輸出繫結](#onedrive-output)可讓您寫入 OneDrive 中的檔案。
- [Outlook 訊息輸出繫結](#outlook-output)可讓您透過 Outlook 傳送電子郵件。
- [Microsoft Graph webhook 觸發程序和繫結](#webhooks)的集合可讓您從 Microsoft Graph 回應事件。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!Note]
> Microsoft Graph 繫結目前處於 Azure Functions 2.x 版的預覽階段。 Functions 1.x 版不加以支援。

## <a name="packages"></a>封裝

[Microsoft.Azure.WebJobs.Extensions.AuthTokens](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.AuthTokens/) NuGet 套件中提供驗證權杖輸入繫結。 [Microsoft.Azure.WebJobs.Extensions.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.MicrosoftGraph/) 套件中提供其他 Microsoft Graph 繫結。 套件的原始程式碼位於 [azure-functions-microsoftgraph-extension](https://github.com/Azure/azure-functions-microsoftgraph-extension/) GitHub 存放庫中。

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="setting-up-the-extensions"></a>設定擴充功能

Microsoft Graph 繫結可透過_繫結擴充功能_提供。 繫結擴充功能是 Azure Functions 執行階段的選用元件。 本節將示範如何設定 Microsoft Graph 和驗證權杖的擴充功能。

### <a name="enabling-functions-20-preview"></a>啟用 Functions 2.0 預覽

繫結擴充功能僅適用於 Azure Functions 2.0 預覽。 

如需如何設定函式應用程式以使用 Functions 執行階段預覽 2.0 版的相關資訊，請參閱[如何設定 Azure Functions 執行階段目標版本](set-runtime-version.md)。

### <a name="installing-the-extension"></a>安裝擴充功能

若要從 Azure 入口網站安裝擴充功能，請瀏覽至會參考它的範本或繫結。 建立新的函式，並在 [範本選擇] 畫面中選擇 "Microsoft Graph" 情節。 從這個情節選取其中一個範本。 或者，您可以瀏覽至現有函式的 [整合] 索引標籤，然後選取本文中涵蓋的其中一個繫結。

在這兩種情況下，會出現警告，指定要安裝的擴充功能。 按一下 [安裝] 取得擴充功能。 每個擴充功能只需要針對每個函式應用程式安裝一次。 

> [!Note] 
> 入口網站安裝程序在取用方案上可能需要 10 分鐘。

如果您使用 Visual Studio，可以安裝[本文先前所列的 NuGet 套件](#packages)來取得擴充功能。

### <a name="configuring-authentication--authorization"></a>設定驗證/授權

本文中概述的繫結需要使用身分識別。 這可讓 Microsoft Graph 強制執行權限和稽核互動。 識別可以是使用者存取您的應用程式或應用程式本身。 若要設定這個身分識別，請使用 Azure Active Directory 來設定 [App Service 驗證/授權](https://docs.microsoft.com/azure/app-service/app-service-authentication-overview)。 您也必須要求函式所需的任何資源權限。

> [!Note] 
> Microsoft Graph 擴充功能僅支援 Azure AD 驗證。 使用者必須使用公司或學校帳戶登入。

如果您正在使用 Azure 入口網站，您會在安裝擴充功能的提示下方看到警告。 警告會提示您設定 App Service 驗證/授權並要求範本或繫結所需的任何權限。 視情況按一下 [立即設定 Azure AD] 或 [立即新增權限]。



<a name="token-input"></a>
## <a name="auth-token"></a>驗證權杖

驗證權杖輸入繫結會取得 Azure AD 指定資源的權杖，並將其提供給您的程式碼作為字串。 資源可以是應用程式有權限任何的項目。 

本節包含下列小節：

* [範例](#auth-token---example)
* [屬性](#auth-token---attributes)
* [組態](#auth-token---configuration)
* [使用量](#auth-token---usage)

### <a name="auth-token---example"></a>驗證權杖 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#auth-token---c-script-example)
* [JavaScript](#auth-token---javascript-example)

#### <a name="auth-token---c-script-example"></a>驗證權杖 - C# 指令碼範例

下列範例可取得使用者設定檔資訊。

*function.json* 檔案會使用權杖輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "token",
      "direction": "in",
      "name": "graphToken",
      "resource": "https://graph.microsoft.com",
      "identity": "userFromRequest"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會使用權杖向 Microsoft Graph 進行 HTTP 呼叫，並傳回結果：

```csharp
using System.Net; 
using System.Net.Http; 
using System.Net.Http.Headers; 

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, string graphToken, TraceWriter log)
{
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", graphToken);
    return await client.GetAsync("https://graph.microsoft.com/v1.0/me/");
}
```

#### <a name="auth-token---javascript-example"></a>驗證權杖 - JavaScript 範例

下列範例可取得使用者設定檔資訊。

*function.json* 檔案會使用權杖輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "token",
      "direction": "in",
      "name": "graphToken",
      "resource": "https://graph.microsoft.com",
      "identity": "userFromRequest"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會使用權杖向 Microsoft Graph 進行 HTTP 呼叫，並傳回結果。

```js
const rp = require('request-promise');

module.exports = function (context, req) {
    let token = "Bearer " + context.bindings.graphToken;

    let options = {
        uri: 'https://graph.microsoft.com/v1.0/me/',
        headers: {
            'Authorization': token
        }
    };
    
    rp(options)
        .then(function(profile) {
            context.res = {
                body: profile
            };
            context.done();
        })
        .catch(function(err) {
            context.res = {
                status: 500,
                body: err
            };
            context.done();
        });
};
```

### <a name="auth-token---attributes"></a>驗證權杖 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [Token](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/TokenBinding/TokenAttribute.cs) 屬性。

### <a name="auth-token---configuration"></a>驗證權杖 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Token` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於驗證權杖的變數名稱。 請參閱[從程式碼使用驗證權杖輸入繫結](#token-input-code)。|
|**type**||必要項目 - 必須設定為 `token`。|
|**direction**||必要項目 - 必須設定為 `in`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|**userId**|UserId  |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**Resource**|**資源**|必要項目 - 正在要求權杖的 Azure AD 資源 URL。|

<a name="token-input-code"></a>
### <a name="auth-token---usage"></a>驗證權杖 - 使用方式

繫結本身不需要任何 Azure AD 權限，但根據使用權杖的方式，您可能必須要求其他權限。 檢查您想要使用權杖存取的資源需求。

一律會向程式碼顯示權杖作為字串。




<a name="excel-input"></a>
## <a name="excel-input"></a>Excel 輸入

Excel 資料表輸入繫結會讀取儲存在 OneDrive 中 Excel 資料表的內容。

本節包含下列小節：

* [範例](#excel-input---example)
* [屬性](#excel-input---attributes)
* [組態](#excel-input---configuration)
* [使用量](#excel-input---usage)

### <a name="excel-input---example"></a>Excel 輸入 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#excel-input---c-script-example)
* [JavaScript](#excel-input---javascript-example)

#### <a name="excel-input---c-script-example"></a>Excel 輸入 - C# 指令碼範例

下列 *function.json* 檔案會使用 Excel 輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "excel",
      "direction": "in",
      "name": "excelTableData",
      "path": "{query.workbook}",
      "identity": "UserFromRequest",
      "tableName": "{query.table}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

下列 C# 指令碼程式碼會讀取指定資料表的內容，然後將其傳回給使用者：

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives; 

public static IActionResult Run(HttpRequest req, string[][] excelTableData, TraceWriter log)
{
    return new OkObjectResult(excelTableData);
}
```

#### <a name="excel-input---javascript-example"></a>Excel 輸入 - JavaScript 範例

下列 *function.json* 檔案會使用 Excel 輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "excel",
      "direction": "in",
      "name": "excelTableData",
      "path": "{query.workbook}",
      "identity": "UserFromRequest",
      "tableName": "{query.table}"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

下列 JavaScript 程式碼會讀取指定資料表的內容，然後將其傳回給使用者。

```js
module.exports = function (context, req) {
    context.res = {
        body: context.bindings.excelTableData
    };
    context.done();
};
```

### <a name="excel-input---attributes"></a>Excel 輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [Excel](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/ExcelAttribute.cs) 屬性。

### <a name="excel-input---configuration"></a>Excel 輸入 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Excel` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於 Excel 資料表的變數名稱。 請參閱[從程式碼使用 Excel 資料表輸入繫結](#excel-input-code)。|
|**type**||必要項目 - 必須設定為 `excel`。|
|**direction**||必要項目 - 必須設定為 `in`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|**userId**|UserId  |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**路徑**|**路徑**|必要項目 - OneDrive 中的 Excel 活頁簿路徑。|
|**worksheetName**|**WorksheetName**|資料表所在的工作表。|
|**tableName**|**TableName**|資料表的名稱。 如果未指定，就會使用工作表的內容。|

<a name="excel-input-code"></a>
### <a name="excel-input---usage"></a>Excel 輸入 - 使用方式

這個繫結需要下列 Azure AD 權限︰
|資源|權限|
|--------|--------|
|Microsoft Graph|讀取使用者檔案|

繫結會向 .NET 函式公開下列類型：
- string[][]
- Microsoft.Graph.WorkbookTable
- 自訂物件類型 (使用結構化模型繫結)










<a name="excel-output"></a>
## <a name="excel-output"></a>Excel 輸出

Excel 輸出繫結會修改儲存在 OneDrive 中 Excel 資料表的內容。

本節包含下列小節：

* [範例](#excel-output---example)
* [屬性](#excel-output---attributes)
* [組態](#excel-output---configuration)
* [使用量](#excel-output---usage)

### <a name="excel-output---example"></a>Excel 輸出 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#excel-output---c-script-example)
* [JavaScript](#excel-output---javascript-example)

#### <a name="excel-output---c-script-example"></a>Excel 輸出 - C# 指令碼範例

下列範例會將資料列新增至 Excel 資料表。

*function.json* 檔案會使用 Excel 輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "newExcelRow",
      "type": "excel",
      "direction": "out",
      "identity": "userFromRequest",
      "updateType": "append",
      "path": "{query.workbook}",
      "tableName": "{query.table}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會以查詢字串的輸入作為基礎，將新的資料列新增至資料表 (假設為單一資料行)：

```csharp
using System.Net;
using System.Text;

public static async Task Run(HttpRequest req, IAsyncCollector<object> newExcelRow, TraceWriter log)
{
    string input = req.Query
        .FirstOrDefault(q => string.Compare(q.Key, "text", true) == 0)
        .Value;
    await newExcelRow.AddAsync(new {
        Text = input
        // Add other properties for additional columns here
    });
    return;
}
```

#### <a name="excel-output---javascript-example"></a>Excel 輸出 - JavaScript 範例

下列範例會將資料列新增至 Excel 資料表。

*function.json* 檔案會使用 Excel 輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "newExcelRow",
      "type": "excel",
      "direction": "out",
      "identity": "userFromRequest",
      "updateType": "append",
      "path": "{query.workbook}",
      "tableName": "{query.table}"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

下列 JavaScript 程式碼會以查詢字串的輸入作為基礎，將新的資料列新增至資料表 (假設為單一資料行)。

```js
module.exports = function (context, req) {
    context.bindings.newExcelRow = {
        text: req.query.text
        // Add other properties for additional columns here
    }
    context.done();
};
```

### <a name="excel-output---attributes"></a>Excel 輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [Excel](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/ExcelAttribute.cs) 屬性。

### <a name="excel-output---configuration"></a>Excel 輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Excel` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於驗證權杖的變數名稱。 請參閱[從程式碼使用 Excel 資料表輸出繫結](#excel-output-code)。|
|**type**||必要項目 - 必須設定為 `excel`。|
|**direction**||必要項目 - 必須設定為 `out`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|UserId |**userId** |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**路徑**|**路徑**|必要項目 - OneDrive 中的 Excel 活頁簿路徑。|
|**worksheetName**|**WorksheetName**|資料表所在的工作表。|
|**tableName**|**TableName**|資料表的名稱。 如果未指定，就會使用工作表的內容。|
|**updateType**|**UpdateType**|必要項目 - 要對資料表進行變更的類型。 可以是下列其中一個值：<ul><li><code>update</code> - 取代 OneDrive 中的資料表內容。</li><li><code>append</code> - 藉由建立新的資料列，在 OneDrive 中將承載新增至資料表結尾。</li></ul>|

<a name="excel-output-code"></a>
### <a name="excel-output---usage"></a>Excel 輸出 - 使用方式

這個繫結需要下列 Azure AD 權限︰
|資源|權限|
|--------|--------|
|Microsoft Graph|可以完整存取使用者檔案|

繫結會向 .NET 函式公開下列類型：
- string[][]
- Newtonsoft.Json.Linq.JObject
- Microsoft.Graph.WorkbookTable
- 自訂物件類型 (使用結構化模型繫結)





<a name="onedrive-input"></a>
## <a name="file-input"></a>檔案輸入

OneDrive 檔案輸入繫結會讀取儲存在 OneDrive 中的檔案內容。

本節包含下列小節：

* [範例](#file-input---example)
* [屬性](#file-input---attributes)
* [組態](#file-input---configuration)
* [使用量](#file-input---usage)

### <a name="file-input---example"></a>檔案輸入 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#file-input---c-script-example)
* [JavaScript](#file-input---javascript-example)

#### <a name="file-input---c-script-example"></a>檔案輸入 - C# 指令碼範例

下列範例會讀取儲存在 OneDrive 中的檔案。

*function.json* 檔案會使用 OneDrive 檔案輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "myOneDriveFile",
      "type": "onedrive",
      "direction": "in",
      "path": "{query.filename}",
      "identity": "userFromRequest"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會讀取查詢字串中指定的檔案，並記錄其長度：

```csharp
using System.Net;

public static void Run(HttpRequestMessage req, Stream myOneDriveFile, TraceWriter log)
{
    log.Info(myOneDriveFile.Length.ToString());
}
```

#### <a name="file-input---javascript-example"></a>檔案輸入 - JavaScript 範例

下列範例會讀取儲存在 OneDrive 中的檔案。

*function.json* 檔案會使用 OneDrive 檔案輸入繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "myOneDriveFile",
      "type": "onedrive",
      "direction": "in",
      "path": "{query.filename}",
      "identity": "userFromRequest"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

下列 JavaScript 程式碼會讀取查詢字串中指定的檔案，並傳回其長度。

```js
module.exports = function (context, req) {
    context.res = {
        body: context.bindings.myOneDriveFile.length
    };
    context.done();
};
```

### <a name="file-input---attributes"></a>檔案輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [OneDrive](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/OneDriveAttribute.cs) 屬性。

### <a name="file-input---configuration"></a>檔案輸入 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `OneDrive` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於檔案的變數名稱。 請參閱[從程式碼使用 OneDrive 檔案輸入繫結](#onedrive-input-code)。|
|**type**||必要項目 - 必須設定為 `onedrive`。|
|**direction**||必要項目 - 必須設定為 `in`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|**userId**|UserId  |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**路徑**|**路徑**|必要項目 - OneDrive 中的檔案路徑。|

<a name="onedrive-input-code"></a>
### <a name="file-input---usage"></a>檔案輸入 - 使用方式

這個繫結需要下列 Azure AD 權限︰
|資源|權限|
|--------|--------|
|Microsoft Graph|讀取使用者檔案|

繫結會向 .NET 函式公開下列類型：
- byte[]
- Stream
- 字串
- Microsoft.Graph.DriveItem






<a name="onedrive-output"></a>
## <a name="file-output"></a>檔案輸出

OneDrive 檔案輸出繫結會修改儲存在 OneDrive 中的檔案內容。

本節包含下列小節：

* [範例](#file-output---example)
* [屬性](#file-output---attributes)
* [組態](#file-output---configuration)
* [使用量](#file-output---usage)

### <a name="file-output---example"></a>檔案輸出 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#file-output---c-script-example)
* [JavaScript](#file-output---javascript-example)

#### <a name="file-output---c-script-example"></a>檔案輸出 - C# 指令碼範例

下列範例會寫入儲存在 OneDrive 中的檔案。

*function.json* 檔案會使用 OneDrive 輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "myOneDriveFile",
      "type": "onedrive",
      "direction": "out",
      "path": "FunctionsTest.txt",
      "identity": "userFromRequest"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會從查詢字串取得文字，並將它寫入位於呼叫者 OneDrive 的根目錄文字檔 (如先前範例中定義的 FunctionsTest.txt) 中：

```csharp
using System.Net;
using System.Text;

public static async Task Run(HttpRequest req, TraceWriter log, Stream myOneDriveFile)
{
    string data = req.Query
        .FirstOrDefault(q => string.Compare(q.Key, "text", true) == 0)
        .Value;
    await myOneDriveFile.WriteAsync(Encoding.UTF8.GetBytes(data), 0, data.Length);
    myOneDriveFile.Close();
    return;
}
```

#### <a name="file-output---javascript-example"></a>檔案輸出 - JavaScript 範例

下列範例會寫入儲存在 OneDrive 中的檔案。

*function.json* 檔案會使用 OneDrive 輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "myOneDriveFile",
      "type": "onedrive",
      "direction": "out",
      "path": "FunctionsTest.txt",
      "identity": "userFromRequest"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會從查詢字串取得文字，並將它寫入位於呼叫者 OneDrive 的根目錄文字檔 (如上述組態中定義的 FunctionsTest.txt) 中。

```js
module.exports = function (context, req) {
    context.bindings.myOneDriveFile = req.query.text;
    context.done();
};
```

### <a name="file-output---attributes"></a>檔案輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [OneDrive](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/OneDriveAttribute.cs) 屬性。

### <a name="file-output---configuration"></a>檔案輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `OneDrive` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於檔案的變數名稱。 請參閱[從程式碼使用 OneDrive 檔案輸出繫結](#onedrive-output-code)。|
|**type**||必要項目 - 必須設定為 `onedrive`。|
|**direction**||必要項目 - 必須設定為 `out`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|UserId |**userId** |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**路徑**|**路徑**|必要項目 - OneDrive 中的檔案路徑。|

<a name="onedrive-output-code"></a>
#### <a name="file-output---usage"></a>檔案輸出 - 使用方式

這個繫結需要下列 Azure AD 權限︰
|資源|權限|
|--------|--------|
|Microsoft Graph|可以完整存取使用者檔案|

繫結會向 .NET 函式公開下列類型：
- byte[]
- Stream
- 字串
- Microsoft.Graph.DriveItem





<a name="outlook-output"></a>
## <a name="outlook-output"></a>Outlook 輸出

Outlook 訊息輸出繫結會透過 Outlook 傳送電子郵件訊息。

本節包含下列小節：

* [範例](#outlook-output---example)
* [屬性](#outlook-output---attributes)
* [組態](#outlook-output---configuration)
* [使用量](#outlook-outnput---usage)

### <a name="outlook-output---example"></a>Outlook 輸出 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#outlook-output---c-script-example)
* [JavaScript](#outlook-output---javascript-example)

#### <a name="outlook-output---c-script-example"></a>Outlook 輸出 - C# 指令碼範例

下列範例會透過 Outlook 傳送電子郵件。

*function.json* 檔案會使用 Outlook 訊息輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "message",
      "type": "outlook",
      "direction": "out",
      "identity": "userFromRequest"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會從呼叫者將電子郵件傳送至查詢字串中指定的收件者：

```csharp
using System.Net;

public static void Run(HttpRequest req, out Message message, TraceWriter log)
{ 
    string emailAddress = req.Query["to"];
    message = new Message(){
        subject = "Greetings",
        body = "Sent from Azure Functions",
        recipient = new Recipient() {
            address = emailAddress
        }
    };
}

public class Message {
    public String subject {get; set;}
    public String body {get; set;}
    public Recipient recipient {get; set;}
}

public class Recipient {
    public String address {get; set;}
    public String name {get; set;}
}
```

#### <a name="outlook-output---javascript-example"></a>Outlook 輸出 - JavaScript 範例

下列範例會透過 Outlook 傳送電子郵件。

*function.json* 檔案會使用 Outlook 訊息輸出繫結定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "name": "message",
      "type": "outlook",
      "direction": "out",
      "identity": "userFromRequest"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會從呼叫者將電子郵件傳送至查詢字串中指定的收件者：

```js
module.exports = function (context, req) {
    context.bindings.message = {
        subject: "Greetings",
        body: "Sent from Azure Functions with JavaScript",
        recipient: {
            address: req.query.to 
        } 
    };
    context.done();
};
```

### <a name="outlook-output---attributes"></a>Outlook 輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [Outlook](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/OutlookAttribute.cs) 屬性。

### <a name="outlook-output---configuration"></a>Outlook 輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `Outlook` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於電子郵件訊息的變數名稱。 請參閱[從程式碼使用 Outlook 訊息輸出繫結](#outlook-output-code)。|
|**type**||必要項目 - 必須設定為 `outlook`。|
|**direction**||必要項目 - 必須設定為 `out`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|**userId**|UserId  |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |

<a name="outlook-output-code"></a>
### <a name="outlook-output---usage"></a>Outlook 輸出 - 使用方式

這個繫結需要下列 Azure AD 權限︰
|資源|權限|
|--------|--------|
|Microsoft Graph|以使用者的身分傳送電子郵件|

繫結會向 .NET 函式公開下列類型：
- Microsoft.Graph.Message
- Newtonsoft.Json.Linq.JObject
- 字串
- 自訂物件類型 (使用結構化模型繫結)






## <a name="webhooks"></a>Webhook

Webhook 可讓您回應 Microsoft Graph 中的事件。 若要支援 webhook，必須建立函式、重新整理並回應 _webhook 訂用帳戶_。 完整的 Webhook 解決方案需要下列繫結的組合：
- [Microsoft Graph webhook 觸發程序](#webhook-trigger)可讓您回應傳入的 webhook。
- [Microsoft Graph webhook 訂用帳戶輸入繫結](#webhook-input)可讓您列出現有的訂用帳戶，並選擇性地加以重新整理。
- [Microsoft Graph webhook 訂用帳戶輸出繫結](#webhook-output)可讓您建立或刪除 webhook 訂用帳戶。

繫結本身不需要任何 Azure AD 權限，但您必須要求與所需回應之資源類型相關的權限。 如需每個資源類型所需權限的清單，請參閱[訂用帳戶權限](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/subscription_post_subscriptions)。

如需 Webhook 的詳細資訊，請參閱[在 Microsoft Graph 中使用 Webhook]。





## <a name="webhook-trigger"></a>Webhook 觸發程序

Microsoft Graph Webhook 觸發程序可讓函式回應從 Microsoft Graph 傳入的 Webhook。 這個觸發程序的每個執行個體都可回應一種 Microsoft Graph 資源類型。

本節包含下列小節：

* [範例](#webhook-trigger---example)
* [屬性](#webhook-trigger---attributes)
* [組態](#webhook-trigger---configuration)
* [使用量](#webhook-trigger---usage)

### <a name="webhook-trigger---example"></a>Webhook 觸發程序 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#webhook-trigger---c-script-example)
* [JavaScript](#webhook-trigger---javascript-example)

#### <a name="webhook-trigger---c-script-example"></a>Webhook 觸發程序 - C# 指令碼範例

下列範例會處理 Outlook 傳入訊息的 Webhook。 若要使用 Webhook 觸發程序，您需要[建立訂用帳戶](#webhook-output---example)，並可[重新整理該訂用帳戶](#webhook-subscription-refresh)以防止它過期。

*function.json* 檔案會定義 Webhook 觸發程序：

```json
{
  "bindings": [
    {
      "name": "msg",
      "type": "GraphWebhookTrigger",
      "direction": "in",
      "resourceType": "#Microsoft.Graph.Message"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會回應傳入的電子郵件訊息，並記錄收件者所傳送及主旨中包含 "Azure Functions" 之電子郵件的內文：

```csharp
#r "Microsoft.Graph"
using Microsoft.Graph;
using System.Net;

public static async Task Run(Message msg, TraceWriter log)  
{
    log.Info("Microsoft Graph webhook trigger function processed a request.");

    // Testable by sending oneself an email with the subject "Azure Functions" and some text body
    if (msg.Subject.Contains("Azure Functions") && msg.From.Equals(msg.Sender)) {
        log.Info($"Processed email: {msg.BodyPreview}");
    }
}
```

#### <a name="webhook-trigger---javascript-example"></a>Webhook 觸發程序 - JavaScript 範例

下列範例會處理 Outlook 傳入訊息的 Webhook。 若要使用 Webhook 觸發程序，您需要[建立訂用帳戶](#webhook-output---example)，並可[重新整理該訂用帳戶](#webhook-subscription-refresh)以防止它過期。

*function.json* 檔案會定義 Webhook 觸發程序：

```json
{
  "bindings": [
    {
      "name": "msg",
      "type": "GraphWebhookTrigger",
      "direction": "in",
      "resourceType": "#Microsoft.Graph.Message"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會回應傳入的電子郵件訊息，並記錄收件者所傳送及主旨中包含 "Azure Functions" 之電子郵件的內文：

```js
module.exports = function (context) {
    context.log("Microsoft Graph webhook trigger function processed a request.");
    const msg = context.bindings.msg
    // Testable by sending oneself an email with the subject "Azure Functions" and some text body
    if((msg.subject.indexOf("Azure Functions") > -1) && (msg.from === msg.sender) ) {
      context.log(`Processed email: ${msg.bodyPreview}`);
    }
    context.done();
};
```

### <a name="webhook-trigger---attributes"></a>Webhook 觸發程序 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [GraphWebHookTrigger](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/GraphWebHookTriggerAttribute.cs) 屬性。

### <a name="webhook-trigger---configuration"></a>Webhook 觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `GraphWebHookTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於電子郵件訊息的變數名稱。 請參閱[從程式碼使用 Outlook 訊息輸出繫結](#outlook-output-code)。|
|**type**||必要項目 - 必須設定為 `graphWebhook`。|
|**direction**||必要項目 - 必須設定為 `trigger`。|
|**resourceType**|**ResourceType**|必要項目 - 這個函式應回應 webhook 的圖表資源。 可以是下列其中一個值：<ul><li><code>#Microsoft.Graph.Message</code> - 對 Outlook 訊息所做的變更。</li><li><code>#Microsoft.Graph.DriveItem</code> - 對 OneDrive 根項目所做的變更。</li><li><code>#Microsoft.Graph.Contact</code> - 對 Outlook 中個人連絡人所做的變更。</li><li><code>#Microsoft.Graph.Event</code> - 對 Outlook 行事曆項目所做的變更。</li></ul>|

> [!Note]
> 函式應用程式只能有一個針對指定 `resourceType` 值註冊的函式。

### <a name="webhook-trigger---usage"></a>Webhook 觸發程序 - 使用方式

繫結會向 .NET 函式公開下列類型：
- 與資源型別相關的 Microsoft Graph SDK 類型，例如 `Microsoft.Graph.Message` 或 `Microsoft.Graph.DriveItem`。
- 自訂物件類型 (使用結構化模型繫結)




<a name="webhook-input"></a>
## <a name="webhook-input"></a>Webhook 輸入

Microsoft Graph Webhook 輸入繫結可讓您擷取由此函式應用程式管理的訂用帳戶清單。 繫結會讀取自函式應用程式儲存體，因此不會反映從應用程式外部建立的其他訂用帳戶。

本節包含下列小節：

* [範例](#webhook-input---example)
* [屬性](#webhook-input---attributes)
* [組態](#webhook-input---configuration)
* [使用量](#webhook-input---usage)

### <a name="webhook-input---example"></a>Webhook 輸入 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#webhook-input---c-script-example)
* [JavaScript](#webhook-input---javascript-example)

#### <a name="webhook-input---c-script-example"></a>Webhook 輸入 - C# 指令碼範例

下列範例會取得呼叫使用者的所有訂用帳戶，並加以刪除。

*function.json* 檔案會透過訂用帳戶輸入繫結以及使用刪除動作的訂用帳戶輸出繫結，來定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "existingSubscriptions",
      "direction": "in",
      "filter": "userFromRequest"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "subscriptionsToDelete",
      "direction": "out",
      "action": "delete",
      "identity": "userFromRequest"
    },
    {
      "type": "http",
      "name": "res",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會取得訂用帳戶，並加以刪除：

```csharp
using System.Net;

public static async Task Run(HttpRequest req, string[] existingSubscriptions, IAsyncCollector<string> subscriptionsToDelete, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");
    foreach (var subscription in existingSubscriptions)
    {
        log.Info($"Deleting subscription {subscription}");
        await subscriptionsToDelete.AddAsync(subscription);
    }
}
```

#### <a name="webhook-input---javascript-example"></a>Webhook 輸入 - JavaScript 範例

下列範例會取得呼叫使用者的所有訂用帳戶，並加以刪除。

*function.json* 檔案會透過訂用帳戶輸入繫結以及使用刪除動作的訂用帳戶輸出繫結，來定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "existingSubscriptions",
      "direction": "in",
      "filter": "userFromRequest"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "subscriptionsToDelete",
      "direction": "out",
      "action": "delete",
      "identity": "userFromRequest"
    },
    {
      "type": "http",
      "name": "res",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會取得訂用帳戶，並加以刪除：

```js
module.exports = function (context, req) {
    const existing = context.bindings.existingSubscriptions;
    var toDelete = [];
    for (var i = 0; i < existing.length; i++) {
        context.log(`Deleting subscription ${existing[i]}`);
        todelete.push(existing[i]);
    }
    context.bindings.subscriptionsToDelete = toDelete;
    context.done();
};
```

### <a name="webhook-input---attributes"></a>Webhook 輸入 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [GraphWebHookSubscription](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/GraphWebHookSubscriptionAttribute.cs) 屬性。

### <a name="webhook-input---configuration"></a>Webhook 輸入 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `GraphWebHookSubscription` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於電子郵件訊息的變數名稱。 請參閱[從程式碼使用 Outlook 訊息輸出繫結](#outlook-output-code)。|
|**type**||必要項目 - 必須設定為 `graphWebhookSubscription`。|
|**direction**||必要項目 - 必須設定為 `in`。|
|**filter**|**Filter**| 如果設定為 `userFromRequest`，繫結就只會擷取呼叫使用者擁有的訂用帳戶 (僅在搭配 [HTTP 觸發程序]時有效)。| 

### <a name="webhook-input---usage"></a>Webhook 輸入 - 使用方式

繫結會向 .NET 函式公開下列類型：
- string[]
- 自訂物件類型陣列
- Newtonsoft.Json.Linq.JObject[]
- Microsoft.Graph.Subscription[]





## <a name="webhook-output"></a>Webhook 輸出

Webhook 訂用帳戶輸出繫結可讓您建立、刪除和重新整理 Microsoft Graph 中的 Webhook 訂用帳戶。

本節包含下列小節：

* [範例](#webhook-output---example)
* [屬性](#webhook-output---attributes)
* [組態](#webhook-output---configuration)
* [使用量](#webhook-output---usage)

### <a name="webhook-output---example"></a>Webhook 輸出 - 範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#webhook-output---c-script-example)
* [JavaScript](#webhook-output---javascript-example)

#### <a name="webhook-output---c-script-example"></a>Webhook 輸出 - C# 指令碼範例

下列範例會建立訂用帳戶。 您可以[重新整理訂用帳戶](#webhook-subscription-refresh)以防止訂用帳戶過期。

*function.json* 檔案會使用建立動作，利用訂用帳戶輸出繫結來定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "clientState",
      "direction": "out",
      "action": "create",
      "subscriptionResource": "me/mailFolders('Inbox')/messages",
      "changeTypes": [
        "created"
      ],
      "identity": "userFromRequest"
    },
    {
      "type": "http",
      "name": "$return",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會註冊 Webhook，當呼叫使用者收到 Outlook 訊息時會通知此函式應用程式：

```csharp
using System;
using System.Net;

public static HttpResponseMessage run(HttpRequestMessage req, out string clientState, TraceWriter log)
{
  log.Info("C# HTTP trigger function processed a request.");
    clientState = Guid.NewGuid().ToString();
    return new HttpResponseMessage(HttpStatusCode.OK);
}
```

#### <a name="webhook-output---javascript-example"></a>Webhook 輸出 - JavaScript 範例

下列範例會建立訂用帳戶。 您可以[重新整理訂用帳戶](#webhook-subscription-refresh)以防止訂用帳戶過期。

*function.json* 檔案會使用建立動作，利用訂用帳戶輸出繫結來定義 HTTP 觸發程序︰

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "clientState",
      "direction": "out",
      "action": "create",
      "subscriptionResource": "me/mailFolders('Inbox')/messages",
      "changeTypes": [
        "created"
      ],
      "identity": "userFromRequest"
    },
    {
      "type": "http",
      "name": "$return",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會註冊 Webhook，當呼叫使用者收到 Outlook 訊息時會通知此函式應用程式：

```js
const uuidv4 = require('uuid/v4');

module.exports = function (context, req) {
    context.bindings.clientState = uuidv4();
    context.done();
};
```

### <a name="webhook-output---attributes"></a>Webhook 輸出 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [GraphWebHookSubscription](https://github.com/Azure/azure-functions-microsoftgraph-extension/blob/master/src/MicrosoftGraphBinding/Bindings/GraphWebHookSubscriptionAttribute.cs) 屬性。

### <a name="webhook-output---configuration"></a>Webhook 輸出 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `GraphWebHookSubscription` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**name**||必要項目 - 函式程式碼中用於電子郵件訊息的變數名稱。 請參閱[從程式碼使用 Outlook 訊息輸出繫結](#outlook-output-code)。|
|**type**||必要項目 - 必須設定為 `graphWebhookSubscription`。|
|**direction**||必要項目 - 必須設定為 `out`。|
|**身分識別**|**身分識別**|必要項目 - 要用來執行動作的身分識別。 可以是下列其中一個值：<ul><li><code>userFromRequest</code> - 僅在使用 [HTTP 觸發程序]時才有效。 會使用呼叫使用者的身分識別。</li><li><code>userFromId</code> - 使用先前已登入之使用者的身分識別與指定的識別碼。 請參閱 <code>userId</code> 屬性。</li><li><code>userFromToken</code> - 使用指定權杖所代表的身分識別。 請參閱 <code>userToken</code> 屬性。</li><li><code>clientCredentials</code> - 使用函式應用程式的身分識別。</li></ul>|
|**userId**|UserId  |只有當_身分識別_設為 `userFromId` 時才需要。 與先前已登入之使用者相關聯的使用者主體識別碼。|
|**userToken**|**UserToken**|只有當_身分識別_設為 `userFromToken` 時才需要。 函式應用程式有效的權杖。 |
|**action**|**Action**|必要項目 - 指定繫結應該要執行的動作。 可以是下列其中一個值：<ul><li><code>create</code> - 註冊新的訂用帳戶。</li><li><code>delete</code> - 刪除指定的訂用帳戶。</li><li><code>refresh</code> - 重新整理指定的訂用帳戶以避免過期。</li></ul>|
|**subscriptionResource**|**SubscriptionResource**|只有當_動作_設為 `create` 時才需要。 指定要監視以進行變更的 Microsoft Graph 資源。 請參閱[在 Microsoft Graph 中使用 webhook]。 |
|**changeType**|**ChangeType**|只有當_動作_設為 `create` 時才需要。 指出會引發通知之訂閱資源中的變更類型。 支援的值為：`created`、`updated`、`deleted`。 可以使用逗號分隔清單來組合多個值。|

### <a name="webhook-output---usage"></a>Webhook 輸出 - 使用方式

繫結會向 .NET 函式公開下列類型：
- 字串
- Microsoft.Graph.Subscription




<a name="webhook-examples"></a>
## <a name="webhook-subscription-refresh"></a>Webhook 訂用帳戶重新整理

有兩種方法可重新整理訂用帳戶：

- 使用應用程式識別碼來處理所有訂用帳戶。 這將需要 Azure Active Directory 管理員的同意。這可供所有 Azure Functions 所支援的語言使用。
- 使用與每個訂用帳戶相關聯的身分識別，方法是手動繫結每個使用者識別碼。 這將需要一些自訂程式碼來執行繫結。 只有 .NET 函式才能使用。

本節包含下列各種方法的範例：

* [應用程式身分識別範例](#webhook-subscription-refresh---app-identity-example)
* [使用者身分識別範例](#webhook-subscription-refresh---user-identity-example)

### <a name="webhook-subscription-refresh---app-identity-example"></a>Webhook 訂用帳戶重新整理 - 應用程式身分識別範例

請參閱特定語言的範例：

* [C# 指令碼 (.csx)](#app-identity-refresh---c-script-example)
* [JavaScript](#app-identity-refresh---javascript-example)

### <a name="app-identity-refresh---c-script-example"></a>應用程式身分識別重新整理 - C# 指令碼範例

下列範例會使用應用程式識別碼來重新整理訂用帳戶。

*function.json* 會使用訂用帳戶輸入繫結和訂用帳戶輸出繫結來定義計時器觸發程序：

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 * * */2 * *"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "existingSubscriptions",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "subscriptionsToRefresh",
      "direction": "out",
      "action": "refresh",
      "identity": "clientCredentials"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會重新整理訂用帳戶：

```csharp
using System;

public static void Run(TimerInfo myTimer, string[] existingSubscriptions, ICollector<string> subscriptionsToRefresh, TraceWriter log)
{
    // This template uses application permissions and requires consent from an Azure Active Directory admin.
    // See https://go.microsoft.com/fwlink/?linkid=858780
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
    foreach (var subscription in existingSubscriptions)
    {
      log.Info($"Refreshing subscription {subscription}");
      subscriptionsToRefresh.Add(subscription);
    }
}
```

### <a name="app-identity-refresh---c-script-example"></a>應用程式身分識別重新整理 - C# 指令碼範例

下列範例會使用應用程式識別碼來重新整理訂用帳戶。

*function.json* 會使用訂用帳戶輸入繫結和訂用帳戶輸出繫結來定義計時器觸發程序：

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 * * */2 * *"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "existingSubscriptions",
      "direction": "in"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "subscriptionsToRefresh",
      "direction": "out",
      "action": "refresh",
      "identity": "clientCredentials"
    }
  ],
  "disabled": false
}
```

JavaScript 程式碼會重新整理訂用帳戶：

```js
// This template uses application permissions and requires consent from an Azure Active Directory admin.
// See https://go.microsoft.com/fwlink/?linkid=858780

module.exports = function (context) {
    const existing = context.bindings.existingSubscriptions;
    var toRefresh = [];
    for (var i = 0; i < existing.length; i++) {
        context.log(`Refreshing subscription ${existing[i]}`);
        toRefresh.push(existing[i]);
    }
    context.bindings.subscriptionsToRefresh = toRefresh;
    context.done();
};
```

### <a name="webhook-subscription-refresh---user-identity-example"></a>Webhook 訂用帳戶重新整理 - 使用者身分識別範例

下列範例會使用使用者身分識別來重新整理訂用帳戶。

*function.json* 檔案會定義計時器觸發程序，並將訂用帳戶輸入繫結延遲至函式程式碼：

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 * * */2 * *"
    },
    {
      "type": "graphWebhookSubscription",
      "name": "existingSubscriptions",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

C# 指令碼程式碼會重新整理訂用帳戶，並在程式碼中使用每個使用者的身分識別建立輸出繫結：

```csharp
using System;

public static async Task Run(TimerInfo myTimer, UserSubscription[] existingSubscriptions, IBinder binder, TraceWriter log)
{
  log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
    foreach (var subscription in existingSubscriptions)
    {
        // binding in code to allow dynamic identity
        using (var subscriptionsToRefresh = await binder.BindAsync<IAsyncCollector<string>>(
            new GraphWebhookSubscriptionAttribute() {
                Action = "refresh",
                Identity = "userFromId",
                UserId = subscription.UserId
            }
        ))
        {
            log.Info($"Refreshing subscription {subscription}");
            await subscriptionsToRefresh.AddAsync(subscription);
        }

    }
}

public class UserSubscription {
    public string UserId {get; set;}
    public string Id {get; set;}
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

[HTTP 觸發程序]: functions-bindings-http-webhook.md
[在 Microsoft Graph 中使用 webhook]: https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/webhooks
