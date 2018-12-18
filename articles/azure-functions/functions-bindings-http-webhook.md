---
title: Azure Functions HTTP 觸發程序和繫結
description: 瞭解如何在 Azure Functions 中使用 HTTP 觸發程序和繫結。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函數, 事件處理, webhook, 動態計算, 無伺服器架構, HTTP, API, REST
ms.service: azure-functions
ms.devlang: multiple
ms.topic: reference
ms.date: 11/21/2017
ms.author: glenga
ms.openlocfilehash: e989152ece19168138597a96d1246ec64498ce69
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227549"
---
# <a name="azure-functions-http-triggers-and-bindings"></a>Azure Functions HTTP 觸發程序和繫結

此文章說明如何在 Azure Functions 中使用 HTTP 觸發程序和輸出繫結。

您可以自訂 HTTP 觸發程序來回應 [Webhook](https://en.wikipedia.org/wiki/Webhook)。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions.Http](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http) NuGet 套件 (1.x 版) 中提供 HTTP 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.Http) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions.Http](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http) NuGet 套件 (3.x 版) 中提供 HTTP 繫結。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Http/) GitHub 存放庫中。

[!INCLUDE [functions-package](../../includes/functions-package-auto.md)]

## <a name="trigger"></a>觸發程序

HTTP 觸發程序可讓您透過 HTTP 要求叫用函式。 您可以使用 HTTP 觸發程序來建置無伺服器 API 並回應 Webhook。 

根據預設，HTTP 觸發程序會在 Functions 1.x 中傳回「HTTP 200 正常」與空白主體，或在 Functions 1 2.x 中傳回「HTTP 204 沒有內容」與空白主體。 若要修改回應，請設定 [HTTP 輸出繫結](#http-output-binding)。

## <a name="trigger---example"></a>觸發程序 - 範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

### <a name="trigger---c-example"></a>觸發程序 - C# 範例

下列範例會顯示在查詢字串或 HTTP 要求主體中尋找 `name` 參數的 [C# 函式](functions-dotnet-class-library.md)。 請注意，傳回值用於輸出繫結，但傳回值屬性並非必要。

```cs
[FunctionName("HttpTriggerCSharp")]
public static async Task<HttpResponseMessage> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequestMessage req, 
    TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    // parse query parameter
    string name = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
        .Value;

    // Get request body
    dynamic data = await req.Content.ReadAsAsync<object>();

    // Set name to query string or body data
    name = name ?? data?.name;

    return name == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, "Hello " + name);
}
```

### <a name="trigger---c-script-example"></a>觸發程序 - C# 指令碼範例

下列範例示範 function.json 檔案中的觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 函式會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,
    "bindings": [
        {
            "authLevel": "function",
            "name": "req",
            "type": "httpTrigger",
            "direction": "in",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "name": "$return",
            "type": "http",
            "direction": "out"
        }
    ]
}
```

[設定](#trigger---configuration)章節會說明這些屬性。

以下是繫結至 `HttpRequestMessage` 的 C# 指令碼：

```csharp
using System.Net;
using System.Threading.Tasks;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");

    // parse query parameter
    string name = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
        .Value;

    // Get request body
    dynamic data = await req.Content.ReadAsAsync<object>();

    // Set name to query string or body data
    name = name ?? data?.name;

    return name == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, "Hello " + name);
}
```

您可以繫結至自訂的物件，而不是 `HttpRequestMessage`。 會從要求主體建立這個物件，並剖析成 JSON。 同樣地，類型可以傳遞至 HTTP 回應輸出繫結，並傳回為回應主體與 狀態碼 200。

```csharp
using System.Net;
using System.Threading.Tasks;

public static string Run(CustomObject req, TraceWriter log)
{
    return "Hello " + req?.name;
}

public class CustomObject {
     public String name {get; set;}
}
```

### <a name="trigger---f-example"></a>觸發程序 - F# 範例

下列範例示範 function.json 檔案中的觸發程序繫結，以及使用此繫結的 [F# 函式](functions-reference-fsharp.md)。 函式會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in"
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

[設定](#trigger---configuration)章節會說明這些屬性。

以下是 F# 程式碼：

```fsharp
open System.Net
open System.Net.Http
open FSharp.Interop.Dynamic

let Run(req: HttpRequestMessage) =
    async {
        let q =
            req.GetQueryNameValuePairs()
                |> Seq.tryFind (fun kv -> kv.Key = "name")
        match q with
        | Some kv ->
            return req.CreateResponse(HttpStatusCode.OK, "Hello " + kv.Value)
        | None ->
            let! data = Async.AwaitTask(req.Content.ReadAsAsync<obj>())
            try
                return req.CreateResponse(HttpStatusCode.OK, "Hello " + data?name)
            with e ->
                return req.CreateErrorResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
    } |> Async.StartAsTask
```

您需要 `project.json` 檔案，以使用 NuGet 來參考 `FSharp.Interop.Dynamic` 和 `Dynamitey` 組件，如下列範例所示：

```json
{
  "frameworks": {
    "net46": {
      "dependencies": {
        "Dynamitey": "1.0.2",
        "FSharp.Interop.Dynamic": "3.0.0"
      }
    }
  }
}
```

### <a name="trigger---javascript-example"></a>觸發程序 - JavaScript 範例

下列範例示範的是使用繫結之 function.json 檔案，以及 [JavaScript 函式](functions-reference-node.md)中的觸發程序繫結。 函式會尋找 `name` 參數，其位於查詢字串或 HTTP 要求的主體。

以下是 *function.json* 檔案：

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

[設定](#trigger---configuration)章節會說明這些屬性。

以下是 JavaScript 程式碼：

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

### <a name="trigger---java-example"></a>觸發程序 - Java 範例

下列範例示範 *function.json* 檔案中的觸發程序繫結，以及使用此繫結的 [Java 函式](functions-reference-java.md)。 此函式會傳回 HTTP 狀態碼 200 回應，以及會在觸發端要求本文前面加上 "Hello, " 問候語的要求本文。


以下是 *function.json* 檔案：

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "anonymous",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

以下是 Java 程式碼：

```java
@FunctionName("hello")
public HttpResponseMessage<String> hello(@HttpTrigger(name = "req", methods = {"post"}, authLevel = AuthorizationLevel.ANONYMOUS), Optional<String> request,
                        final ExecutionContext context) 
    {
        // default HTTP 200 response code
        return String.format("Hello, %s!", request);
    }
}
```

## <a name="trigger---attributes"></a>觸發程序 - 屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [HttpTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions.Http/HttpTriggerAttribute.cs) 屬性。

您可以設定授權層級和屬性建構參數中可允許的 HTTP 方法，並有 webhook 類型和路由範本的內容。 如需這些設定的詳細資訊，請參閱[觸發程序 - 組態](#trigger---configuration)。 以下是方法簽章中的 `HttpTrigger` 屬性：

```csharp
[FunctionName("HttpTriggerCSharp")]
public static HttpResponseMessage Run(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequestMessage req)
{
    ...
}
 ```

如需完整範例，請參閱[觸發程序 - C# 範例](#trigger---c-example)。

## <a name="trigger---configuration"></a>觸發程式 - 設定

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `HttpTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
| **type** | n/a| 必要項目 - 必須設定為 `httpTrigger`。 |
| **direction** | n/a| 必要項目 - 必須設定為 `in`。 |
| **name** | n/a| 必要項目 - 函式程式碼中用於要求或要求主體的變數名稱。 |
| <a name="http-auth"></a>**authLevel** |  **AuthLevel** |會判斷要求中必須存在哪些金鑰 (若有的話) 才能叫用函式。 授權層級可為下列其中一個值： <ul><li><code>anonymous</code>&mdash;不需要 API 金鑰。</li><li><code>function</code>&mdash;需要函式專屬的 API 金鑰。 如果沒有提供任何值，此為預設值。</li><li><code>admin</code>&mdash;需要主要金鑰。</li></ul> 如需詳細資訊，請參閱有關[授權金鑰](#authorization-keys)章節。 |
| **methods** |**方法** | 函式將回應的 HTTP 方法陣列。 如果未指定，函式將會回應所有的 HTTP 方法。 請參閱[自訂 HTTP 端點](#customize-the-http-endpoint)。 |
| **route** | **路由** | 會定義路由範本，從而控制函式所要回應的要求 URL。 如果沒有提供任何值，預設值為 `<functionname>`。 如需詳細資訊，請參閱[自訂 HTTP 端點](#customize-the-http-endpoint)。 |
| **webHookType** | **WebHookType** | _只有針對 1.x 版執行階段才有支援。_<br/><br/>會設定 HTTP 觸發程序作為指定提供者的 [webhook](https://en.wikipedia.org/wiki/Webhook) 接收器。 如果設定這個屬性，請勿設定 `methods` 屬性。 Webhook 類型可以是下列值其中之一：<ul><li><code>genericJson</code>&mdash;一般用途的 Webhook 端點，不需要特定提供者的邏輯。 此設定會將要求限制為只有那些使用 HTTP POST 和包含 `application/json` 內容類型的要求。</li><li><code>github</code>&mdash;函式會回應 [GitHub Webhook](https://developer.github.com/webhooks/)。 請勿使用 _authLevel_ 屬性搭配 GitHub Webhook。 如需詳細資訊，請參閱此文章稍後的 GitHub Webhook 一節。</li><li><code>slack</code>&mdash;函式會回應 [Slack Webhook](https://api.slack.com/outgoing-webhooks)。 請勿使用 _authLevel_ 屬性搭配 Slack Webhook。 如需詳細資訊，請參閱此文章稍後的 Slack Webhook 一節。</li></ul>|

## <a name="trigger---usage"></a>觸發程序 - 使用方式

針對 C# 和 F# 函式，您可以宣告觸發程序輸入的類型為 `HttpRequestMessage` 或自訂類型。 如果您選擇 `HttpRequestMessage`，就會取得要求物件的完整存取權。 針對自訂的類型，執行階段會嘗試剖析 JSON 要求本文來設定物件屬性。

對於 JavaScript 函式，Functions 執行階段提供要求主體而非要求物件。 如需詳細資訊，請參閱 [JavaScript 觸發程序範例](#trigger---javascript-example)。


### <a name="customize-the-http-endpoint"></a>自訂 HTTP 端點

根據預設，當您為 HTTP 觸發程序建立函式時，將可透過下列形式的路由來定址該函式：

    http://<yourapp>.azurewebsites.net/api/<funcname> 

您可以在 HTTP 觸發程序的輸入繫結上使用選擇性的 `route` 屬性來自訂此路由。 舉例來說，下列 *function.json* 檔案定義了 HTTP 觸發程序的 `route` 屬性：

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

在使用此設定的情況下，現在便可使用下列路由來定址該函式，而不需使用原始路由。

```
http://<yourapp>.azurewebsites.net/api/products/electronics/357
```

這可讓該函式程式碼在位址中支援兩個參數，即 _category_ 和 _id_。您可以將任何 [Web API 路由條件約束](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints)與您的參數搭配使用。 下列 C# 函式程式碼會同時利用這兩個參數。

```csharp
public static Task<HttpResponseMessage> Run(HttpRequestMessage req, string category, int? id, 
                                                TraceWriter log)
{
    if (id == null)
        return  req.CreateResponse(HttpStatusCode.OK, $"All {category} items were requested.");
    else
        return  req.CreateResponse(HttpStatusCode.OK, $"{category} item with id = {id} has been requested.");
}
```

以下是使用相同路由參數的 Node.js 函式程式碼。

```javascript
module.exports = function (context, req) {

    var category = context.bindingData.category;
    var id = context.bindingData.id;

    if (!id) {
        context.res = {
            // status defaults to 200 */
            body: "All " + category + " items were requested."
        };
    }
    else {
        context.res = {
            // status defaults to 200 */
            body: category + " item with id = " + id + " was requested."
        };
    }

    context.done();
} 
```

所有函式路由預設前面都會加上 *api*。 您也可以在 [host.json](functions-host-json.md) 檔案中使用 `http.routePrefix` 屬性來自訂或移除前置詞。 下列範例會在 *host.json* 檔案中使用空字串作為前置詞來移除 *api* 路由前置詞。

```json
{
    "http": {
    "routePrefix": ""
    }
}
```

### <a name="authorization-keys"></a>授權金鑰

Functions 可讓您使用金鑰來提高開發期間存取 HTTP 函式端點的困難度。  標準 HTTP 觸發程序可要求在要求中有這樣的 API 金鑰存在。 

> [!IMPORTANT]
> 雖然金鑰可能有助於在開發期間遮蔽您的 HTTP 端點，但這並不適合用來作為在生產環境中保護 HTTP 觸發程序的方式。 若要深入了解，請參閱[在生產環境中保護 HTTP 端點](#secure-an-http-endpoint-in-production)。

> [!NOTE]
> 在 Functions 1.x 執行階段中，Webhook 提供者可以使用金鑰以多種方式授權要求，端視提供者支援的方式而定。 [Webhook 和金鑰](#webhooks-and-keys)中提供了這方面的相關說明。 2.x 版執行階段並未內建對 Webhook 提供者的支援。

金鑰類型有兩種：

* **主機金鑰**：這些金鑰由函數應用程式中所有的函式共用。 當做為 API 金鑰使用時，這些金鑰會允許存取函數應用程式中的任何函式。
* **函式金鑰**：這些金鑰僅適用於據以定義它們的特定函式。 當做為 API 金鑰使用時，這些金鑰僅允許存取該函式。

每個金鑰均為具名以供參考，並且在函式和主機層級有一預設金鑰 (名稱為 "default")。 函式金鑰的優先順序高於主機金鑰。 當您使用相同的名稱來定義兩個金鑰時，一律會使用函式金鑰。

每個函數應用程式都有特殊的**主要金鑰**。 此金鑰是一個名為 `_master` 的主機金鑰，它會提供對執行階段 API 的系統管理存取權。 無法撤銷此金鑰。 當您將授權等級設定為 `admin` 時，要求就必須使用主要金鑰；任何其他金鑰則會導致授權失敗。

> [!CAUTION]  
> 由於主要金鑰會在您的函數應用程式中授與提高的權限，因此您不應該與第三方共用此金鑰，或是在原生用戶端應用程式中散發它。 當您選擇管理授權層級時，請務必謹慎。

### <a name="obtaining-keys"></a>取得金鑰

金鑰會當作您函數應用程式的一部分儲存於 Azure 中，並在加密後靜置。 若要檢視您的金鑰，請建立新的金鑰或將金鑰輪替為新的值，瀏覽至您在 [Azure 入口網站](https://portal.azure.com)中的其中一個 HTTP 觸發函式，然後選取 [管理]。

![在入口網站中管理函式金鑰。](./media/functions-bindings-http-webhook/manage-function-keys.png)

目前沒有任何支援的 API 可透過程式設計方式取得函式金鑰。

### <a name="api-key-authorization"></a>API 金鑰授權

大多數 HTTP 觸發程序範本都會要求在要求中有 API 金鑰。 因此，您的 HTTP 要求通常看起來會像以下 URL：

    https://<yourapp>.azurewebsites.net/api/<function>?code=<ApiKey>

金鑰可包含在名為 `code` 的查詢字串變數中，如以上所示。 它也可以包含在 `x-functions-key` HTTP 標頭中。 金鑰的值可以是針對函式定義的任何函式金鑰，或是任何主機金鑰。

您可以允許匿名要求，這不需要金鑰。 您也可以要求使用主要金鑰。 您可以使用繫結 JSON 中的 `authLevel` 屬性來變更預設授權層級。 如需詳細資訊，請參閱[觸發程序 - 組態](#trigger---configuration)。

> [!NOTE]
> 在本機執行函式時，不論指定的驗證等級設定為何，都會停用授權。 發佈至 Azure 之後，就會強制執行您觸發程序中的 `authLevel` 設定。



### <a name="secure-an-http-endpoint-in-production"></a>在生產環境中保護 HTTP 端點

若要在生產環境中完全保護您的函式端點，您應該考慮實作下列其中一個函數應用程式等級安全性選項：

* 為您的函數應用程式開啟「App Service 驗證/授權」。 App Service 平台可讓您使用 Azure Active Directory (AAD) 和數個協力廠商身分識別提供者來驗證用戶端。 您可以使用此平台來為函式實作自訂授權規則，並可使用來自函式程式碼的使用者資訊。 若要深入了解，請參閱 [Azure App Service 中的驗證與授權](../app-service/app-service-authentication-overview.md)。

* 使用「Azure API 管理」(APIM) 來驗證要求。 APIM 為傳入要求提供多種 API 安全性選項。 若要深入了解，請參閱 [API 管理驗證原則](../api-management/api-management-authentication-policies.md)。 備妥 APIM 之後，您可以設定讓函數應用程式只接受來自您 APIM 執行個體 IP 位址的要求。 若要深入了解，請參閱 [IP 位址限制](ip-addresses.md#ip-address-restrictions)。

* 將您的函數應用程式部署至 Azure App Service Environment (ASE)。 ASE 提供一個可供執行您函式的專用主控環境。 ASE 可讓您設定一個單一前端閘道，可用來驗證所有傳入要求。 如需詳細資訊，請參閱[設定 App Service Environment 的 Web 應用程式防火牆 (WAF)](../app-service/environment/app-service-app-service-environment-web-application-firewall.md)。

當使用這其中一種函數應用程式等級的安全性方法時，您應該將 HTTP 觸發的函式驗證等級設定為 `anonymous`。

### <a name="webhooks"></a>Webhook

> [!NOTE]
> Webhook 模式僅適用於 1.x 版 Functions 執行階段。

Webhook 模式可為 Webhook 承載提供額外的驗證。 在 2.x 版中，基底 HTTP 觸發程序仍然可運作，並且對 Webhook 來說是建議採用的方法。

#### <a name="github-webhooks"></a>GitHub Webhook

若要回應 GitHub Webhook，請先建立含有 HTTP 觸發程序的函式，然後將 **webHookType** 屬性設定為 `github`。 接著將其 URL 和 API 金鑰複製到您 GitHub 存放庫的 [新增 Webhook] 頁面。 

![](./media/functions-bindings-http-webhook/github-add-webhook.png)

如需範例，請參閱[建立由 GitHub Webhook 所觸發的函式](functions-create-github-webhook-triggered-function.md)。

#### <a name="slack-webhooks"></a>Slack Webhook

Slack webhook 會為您產生權杖，而不是由您指定，因此您必須使用 Slack 的權杖來設定函式專屬的金鑰。 請參閱[授權金鑰](#authorization-keys)。

### <a name="webhooks-and-keys"></a>Webhook 和金鑰

Webhook 授權是由 Webhook 接收器元件 (HTTP 觸發程序的一部分) 處理，處理機制則會以 Webhook 類型作為基礎而有所不同。 每個機制都依賴金鑰。 根據預設，將會使用名稱為 "default" 的函式金鑰。 如需使用不同的金鑰，請設定 Webhook 提供者以下列其中一種方式將金鑰名稱隨著要求一起傳送：

* **查詢字串**：提供者會在 `clientid` 查詢字串參數中傳遞金鑰名稱，例如 `https://<yourapp>.azurewebsites.net/api/<funcname>?clientid=<keyname>`。
* **要求標頭**︰提供者在 `x-functions-clientid` 標頭中傳遞金鑰名稱。

## <a name="trigger---limits"></a>觸發程序 - 限制

HTTP 要求長度的限制為 100 MB (104,857,600 個位元組)，而 URL 長度的限制為 4 KB (4,096 個位元組)。 這些限制由執行階段 [Web.config 檔案](https://github.com/Azure/azure-webjobs-sdk-script/blob/v1.x/src/WebJobs.Script.WebHost/Web.config)的 `httpRuntime` 元素所指定。

如果使用 HTTP 觸發程序的函式未在約 2.5 分鐘內完成，閘道將會逾時並傳回 HTTP 502 錯誤。 函式會繼續執行，但無法傳回 HTTP 回應。 對於長時間執行的函式，建議您遵循非同步模式，並傳回可以偵測要求狀態的位置。 如需函式可以執行多久的相關資訊，請參閱[級別和裝載 - 使用情況方案](functions-scale.md#consumption-plan)。 

## <a name="trigger---hostjson-properties"></a>觸發程序 - host.json 屬性

[host.json](functions-host-json.md) 檔案包含控制 HTTP 觸發程序行為的設定。

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="output"></a>輸出

使用 HTTP 輸出繫結來回應 HTTP 要求傳送者。 此繫結需要 HTTP 觸發程序，並可讓您自訂與觸發程序要求相關聯的回應。 如果未提供 HTTP 輸出繫結，則 HTTP 觸發程序會在 Functions 1.x 中傳回「HTTP 200 正常」與空白主體，或在 Functions 1 2.x 中傳回「HTTP 204 沒有內容」與空白主體。

## <a name="output---configuration"></a>輸出 - 設定

下表說明您在 function.json 檔案中設定的繫結設定屬性。 就 C# 類別程式庫而言，沒有任何屬性 (Attribute) 的屬性 (Property) 與這些 *function.json* 屬性 (Property) 對應。 

|屬性  |說明  |
|---------|---------|
| **type** |必須設為 `http`。 |
| **direction** | 必須設為 `out`。 |
|**name** | 函式程式碼中用於回應的變數名稱，或要使用傳回值的 `$return`。 |

## <a name="output---usage"></a>輸出 - 使用方式

若要傳送 HTTP 回應，請使用語言標準回應模式。 在 C# 或 C# 指令碼 中，讓函式傳回類型成為 `HttpResponseMessage` 或 `Task<HttpResponseMessage>`。 在 C# 中，傳回值屬性並非必要。

如需範例回應，請參閱[觸發程序範例](#trigger---example)。

## <a name="next-steps"></a>後續步驟

[深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
