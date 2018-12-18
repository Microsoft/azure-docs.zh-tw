---
title: 適用於 Azure Functions 的 JavaScript 開發人員參考 | Microsoft Docs
description: 了解如何使用 JavaScript 開發函式。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.assetid: 45dedd78-3ff9-411f-bb4b-16d29a11384c
ms.service: azure-functions
ms.devlang: nodejs
ms.topic: reference
ms.date: 03/04/2018
ms.author: glenga
ms.openlocfilehash: 24f7faa0fb111e4e537a7db3f5e1eea709d1ca59
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957724"
---
# <a name="azure-functions-javascript-developer-guide"></a>Azure Functions JavaScript 開發人員指南
本指南包含使用 JavaScript 撰寫 Azure Functions 的複雜性相關資訊。

JavaScript 函式是匯出的 `function`，會在觸發時執行 ([觸發程序是在 function.json 中設定](functions-triggers-bindings.md))。 每個函式都會有傳遞的 `context` 物件，其可用來接收和傳送資料繫結、記錄，以及與執行階段進行的通訊。

本文假設您已經讀過 [Azure Functions 開發人員參考](functions-reference.md)。 也建議您依照「快速入門」底下的教學課程，[建立您的第一個函式](functions-create-first-function-vs-code.md)。

## <a name="folder-structure"></a>資料夾結構

JavaScript 專案的必要資料夾結構如下所示。 請注意，您可以變更此預設值：請參閱以下 [scriptFile](functions-reference-node.md#using-scriptfile) 小節的詳細資料。

```
FunctionsProject
 | - MyFirstFunction
 | | - index.js
 | | - function.json
 | - MySecondFunction
 | | - index.js
 | | - function.json
 | - SharedCode
 | | - myFirstHelperFunction.js
 | | - mySecondHelperFunction.js
 | - node_modules
 | - host.json
 | - package.json
 | - extensions.csproj
 | - bin
```

在專案根目錄中，有共用的 [host.json](functions-host-json.md) 檔案可用來設定函式應用程式。 每個函式都有本身程式碼檔案 (.js) 和繫結設定檔 (function.json) 的資料夾。

在函式執行階段的[版本 2.x](functions-versions.md) 中所需的繫結延伸模組，是以 `bin` 資料夾中的實際程式庫檔案在 `extensions.csproj` 檔案中所定義。 在本機開發時，您必須[註冊繫結延伸模組](functions-triggers-bindings.md#local-development-azure-functions-core-tools)。 開發 Azure 入口網站中的函式時，就會為您完成這項註冊。

## <a name="exporting-a-function"></a>匯出函數

JavaScript 函式必須透過 [`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports) (或 [`exports`](https://nodejs.org/api/modules.html#modules_exports)) 匯出。 在預設情況中，您匯出的函式應該是僅來自其檔案的匯出、名為 `run` 的匯出，或名為 `index` 的匯出。 函式的預設位置是 `index.js`，其中 `index.js` 與對應的 `function.json` 共用相同的父目錄。 請注意，`function.json` 的父目錄名稱一律是函式的名稱。 

若要設定檔案位置，並匯出函式的名稱，請參閱以下的[設定您的函式進入點](functions-reference-node.md#configure-function-entry-point)。

匯出的函式進入點必須一律採用 `context` 物件做為第一個參數。

```javascript
// You must include a context, other arguments are optional
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
    context.done();
};
// You can also use 'arguments' to dynamically handle inputs
module.exports = async function(context) {
    context.log('Number of inputs: ' + arguments.length);
    // Iterates through trigger and input binding data
    for (i = 1; i < arguments.length; i++){
        context.log(arguments[i]);
    }
};
```

觸發程序和輸入繫結 (`direction === "in"` 的繫結) 可以傳遞至函式做為參數。 這些繫結會按照在 *function.json* 中定義的順序傳遞至函式。 您也可以使用 JavaScript [`arguments`](https://msdn.microsoft.com/library/87dw3w1k.aspx) 物件動態處理輸入。 例如，如果您有 `function(context, a, b)` 並將它變更為 `function(context, a)`，您仍然可以在函式程式碼中藉由參考 `arguments[2]` 來取得 `b` 的值。

所有繫結 (不論方向為何) 也都會使用 `context.bindings` 屬性傳遞到 `context` 物件。

## <a name="context-object"></a>context 物件
執行階段使用 `context` 物件來將資料傳遞至函式並從中傳出，而且可讓您與執行階段進行通訊。

`context` 物件一律為函式的第一個參數而且一律必須包含，因為它具有像是 `context.done` 和 `context.log` 的方法，而您必須要有這些方法才能正確地使用執行階段。 您可以任意方式命名物件 (例如 `ctx` 或 `c`)。

```javascript
// You must include a context, but other arguments are optional
module.exports = function(ctx) {
    // function logic goes here :)
    ctx.done();
};
```

### <a name="contextbindings-property"></a>context.bindings 屬性

```
context.bindings
```
傳回包含所有輸入和輸出資料的具名物件。 例如，*function.json* 中的下列繫結定義可讓您使用 `context.bindings.myOutput` 從 `context.bindings.myInput` 存取佇列的內容並且將輸出指派到佇列。

```json
{
    "type":"queue",
    "direction":"in",
    "name":"myInput"
    ...
},
{
    "type":"queue",
    "direction":"out",
    "name":"myOutput"
    ...
}
```

```javascript
// myInput contains the input data, which may have properties such as "name"
var author = context.bindings.myInput.name;
// Similarly, you can set your output data
context.bindings.myOutput = { 
        some_text: 'hello world', 
        a_number: 1 };
```

請注意，您可以使用 `context.done` 方法選擇定義輸出繫結資料，而不使用 `context.binding` 物件 (如下所示)。

### <a name="contextbindingdata-property"></a>context.bindingData 屬性

```
context.bindingData
```
傳回包含觸發程序中繼資料和函式引動過程資料的具名物件 (`invocationId`、`sys.methodName`、`sys.utcNow`、`sys.randGuid`)。 如需觸發程序中繼資料的範例，請參閱此[事件中樞範例](functions-bindings-event-hubs.md#trigger---javascript-example)。

### <a name="contextdone-method"></a>context.done 方法
```
context.done([err],[propertyBag])
```

通知執行階段，您的程式碼已完成。 如果您的函式使用 JavaScript [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) 宣告 (可使用節點 8 取得 + 在 Functions 2.x 版)，您不需要使用 `context.done()`。 隱含地呼叫 `context.done` 回呼。

如果您的函式不是非同步函式，**您必須呼叫** `context.done` 來通知執行階段您的函式已完成。 如果遺漏，則執行會逾時。

`context.done` 方法可讓您將使用者定義的錯誤傳回到執行階段，並傳回包含輸出繫結資料的 JSON 物件。 傳遞到 `context.done` 的屬性會覆寫 `context.bindings` 物件上設定的任何屬性。

```javascript
// Even though we set myOutput to have:
//  -> text: 'hello world', number: 123
context.bindings.myOutput = { text: 'hello world', number: 123 };
// If we pass an object to the done function...
context.done(null, { myOutput: { text: 'hello there, world', noNumber: true }});
// the done method will overwrite the myOutput binding to be: 
//  -> text: 'hello there, world', noNumber: true
```

### <a name="contextlog-method"></a>context.log 方法  

```
context.log(message)
```
可讓您寫入預設追蹤層級的資料流函式記錄。 `context.log` 上有其他可用的記錄方法，可讓您在其他追蹤層級寫入函式記錄︰


| 方法                 | 說明                                |
| ---------------------- | ------------------------------------------ |
| **error(_message_)**   | 寫入錯誤層級或更低層級的記錄。   |
| **warn(_message_)**    | 寫入警告層級或更低層級的記錄。 |
| **info(_message_)**    | 寫入資訊層級或更低層級的記錄。    |
| **verbose(_message_)** | 寫入詳細資訊層級記錄。           |

下列範例會依警告追蹤層級寫入記錄︰

```javascript
context.log.warn("Something has happened."); 
```
您可以在 host.json 檔案中[設定用於記錄的追蹤層級閾值](#configure-the-trace-level-for-console-logging)。 如需寫入記錄的詳細資訊，請參閱以下的[寫入追蹤輸出](#writing-trace-output-to-the-console)。

參閱[監視 Azure Functions](functions-monitoring.md) 深入了解如何檢視和查詢函式記錄。

## <a name="binding-data-type"></a>繫結資料類型

若要定義輸入繫結的資料類型，請使用繫結定義中的 `dataType` 屬性。 例如，若要以二進位格式讀取 HTTP 要求的內容，請使用類型 `binary`：

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

`dataType` 的選項是：`binary`、`stream` 和 `string`。

## <a name="writing-trace-output-to-the-console"></a>將追蹤輸出寫入主控台中 

在 Functions 中，您可以使用 `context.log` 方法，將追蹤輸出寫入主控台中。 在 Functions v2.x 中，會在函數應用程式層級擷取透過 `console.log` 的追蹤輸出。 這表示，從 `console.log` 的輸出未繫結至特定函式引動過程，因此不會在特定函式的記錄中顯示。 不過，這些輸出會傳播至 Application Insights。 在 Functions v1.x 中，您不能使用 `console.log` 來寫入主控台中。 

當您呼叫 `context.log()` 時，會在預設追蹤層級 (也就是「資訊」追蹤層級) 將您的訊息寫入主控台中。 下列程式碼會依資訊追蹤層級寫入主控台中︰

```javascript
context.log({hello: 'world'});  
```

此程式碼相當於上述程式碼：

```javascript
context.log.info({hello: 'world'});  
```

此程式碼會依錯誤層級寫入主控台中︰

```javascript
context.log.error("An error has occurred.");  
```

因為 _error_ 是最高追蹤層級，所以只要啟用記錄，這項追蹤就會寫入所有追蹤層級的輸出。

所有 `context.log` 方法都與 Node.js [util.format 方法 (英文)](https://nodejs.org/api/util.html#util_util_format_format) 支援相同的參數格式。 請考慮下列程式碼，它會使用預設追蹤層級寫入函式記錄︰

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=' + req.originalUrl);
context.log('Request Headers = ' + JSON.stringify(req.headers));
```

您也能以下列格式撰寫相同的程式碼：

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);
context.log('Request Headers = ', JSON.stringify(req.headers));
```

### <a name="configure-the-trace-level-for-console-logging"></a>設定用於主控台記錄的追蹤層級

Functions 讓您定義寫入至主控台的閾值追蹤層級，這可讓您輕鬆地控制追蹤從函式寫入主控台的方式。 若要設定寫入至主控台之所有追蹤的閾值，請使用 host.json 檔案中的 `tracing.consoleLevel` 屬性。 這個設定會套用到函式應用程式中的所有函式。 下列範例會設定追蹤閾值來啟用詳細資訊記錄︰

```json
{ 
    "tracing": {      
        "consoleLevel": "verbose"     
    }
}  
```

**consoleLevel** 的值對應至 `context.log` 方法的名稱。 若要停用主控台的所有追蹤記錄，請將 **consoleLevel** 設為 _off_。 如需詳細資訊，請參閱 [host.json 參考](functions-host-json.md)。

## <a name="http-triggers-and-bindings"></a>HTTP 觸發程序和繫結

HTTP 和 Webhook 觸發程序以及 HTTP 輸出繫結會使用要求和回應物件來代表 HTTP 傳訊。  

### <a name="request-object"></a>要求物件

`context.req` (要求) 物件具有下列屬性：

| 屬性      | 說明                                                    |
| ------------- | -------------------------------------------------------------- |
| _body_        | 包含要求本文的物件。               |
| _headers_     | 包含要求標頭的物件。                   |
| _method_      | 要求的 HTTP 方法。                                |
| _originalUrl_ | 要求的 URL。                                        |
| _params_      | 包含要求之路由傳送參數的物件。 |
| _查詢_       | 包含查詢參數的物件。                  |
| _rawBody_     | 字串格式的訊息內文。                           |


### <a name="response-object"></a>回應物件

`context.res` (回應) 物件具有下列屬性：

| 屬性  | 說明                                               |
| --------- | --------------------------------------------------------- |
| _body_    | 包含回應本文的物件。         |
| _headers_ | 包含回應標頭的物件。             |
| _isRaw_   | 表示略過回應的格式。    |
| _狀態_  | 回應的 HTTP 狀態碼。                     |

### <a name="accessing-the-request-and-response"></a>存取要求和回應 

使用 HTTP 觸發程序時，您可以使用許多方式來存取 HTTP 要求和回應物件︰

+ 從 `context` 物件的 `req` 和 `res` 屬性中。 如此一來，您可以使用傳統模式來存取內容物件中的 HTTP 資料，而不需使用完整 `context.bindings.name` 模式。 下列範例示範如何存取 `context` 上的 `req` 和 `res` 物件：

    ```javascript
    // You can access your http request off the context ...
    if(context.req.body.emoji === ':pizza:') context.log('Yay!');
    // and also set your http response
    context.res = { status: 202, body: 'You successfully ordered more coffee!' }; 
    ```

+ 從具名輸入和輸出繫結。 如此一來，HTTP 觸發程序和繫結的運作方式會與任何其他繫結相同。 下列範例會使用具名 `response` 繫結來設定回應物件： 

    ```json
    {
        "type": "http",
        "direction": "out",
        "name": "response"
    }
    ```
    ```javascript
    context.bindings.response = { status: 201, body: "Insert succeeded." };
    ```
+ _[僅回應]_ 藉由呼叫`context.res.send(body?: any)`。 HTTP 回應是以做為回應主體的輸入 `body` 所建立。 隱含地呼叫 `context.done()`。

+ _[僅回應]_ 藉由呼叫`context.done()`。 特殊類型的 HTTP 繫結，會傳回傳遞到 `context.done()` 方法的回應。 下列 HTTP 輸出繫結定義 `$return` 輸出參數︰

    ```json
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
    ``` 
    ```javascript
     // Define a valid response object.
    res = { status: 201, body: "Insert succeeded." };
    context.done(null, res);   
    ```  

## <a name="node-version"></a>節點版本

下表顯示 Functions 執行階段每個主要版本所使用的 Node.js 版本：

| Functions 版本 | Node.js 版本 | 
|---|---|
| 1.x | 6.11.2 (由執行階段鎖定) |
| 2.x  | _作用中 LTS_ 和 _目前_的 Node.js 版本 (建議使用 8.11.1 和 10.6.0)。 使用 WEBSITE_NODE_DEFAULT_VERSION [應用程式設定](functions-how-to-use-azure-function-app-settings.md#settings)來設定版本。|

您可以藉由檢查以上的應用程式設定，或藉由從任何函式列印 `process.version`，來查看執行階段目前正在使用的版本。

## <a name="dependency-management"></a>相依性管理
若要使用 JavaScript 程式碼中的社群程式庫，如下列範例所示，您必須確定已在 Azure 中的函數應用程式上安裝所有的相依性。

```javascript
// Import the underscore.js library
var _ = require('underscore');
var version = process.version; // version === 'v6.5.0'

module.exports = function(context) {
    // Using our imported underscore.js library
    var matched_names = _
        .where(context.bindings.myInput.names, {first: 'Carla'});
```

請注意，您應該定義函數應用程式根目錄的 `package.json` 檔案。 定義檔案可讓應用程式中的所有函式共用相同的快取套件，以提供最佳效能。 發生版本衝突時，您可以在特定函式的資料夾中新增 `package.json` 檔案來解決它。  

有兩種方式可在您的函數應用程式上安裝套件： 

### <a name="deploying-with-dependencies"></a>使用相依性進行部署
1. 執行 `npm install` 在本機安裝所有必要的套件。

2. 部署您的程式碼，並確定 `node_modules` 資料夾包含在部署中。 


### <a name="using-kudu"></a>使用 Kudu
1. 移至 `https://<function_app_name>.scm.azurewebsites.net`。

2. 按一下 [偵錯主控台] > [CMD]。

3. 移至 `D:\home\site\wwwroot`，然後將 package.json 檔案拖曳至頁面上半部的 **wwwroot** 資料夾。  
    您也可以使用其他方法將檔案上傳至函數應用程式。 如需詳細資訊，請參閱[如何更新函式應用程式檔案](functions-reference.md#fileupdate)。 

4. 上傳 package.json 檔案之後，請在 **Kudu 遠端執行主控台**中執行 `npm install` 命令。  
    此動作會下載 package.json 檔案中指出的套件，並重新啟動函數應用程式。

## <a name="environment-variables"></a>環境變數
若要取得環境變數或應用程式設定值，請使用 `process.env`，如下列 `GetEnvironmentVariable` 函式中所示：

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.log('Node.js timer trigger function ran!', timeStamp);   
    context.log(GetEnvironmentVariable("AzureWebJobsStorage"));
    context.log(GetEnvironmentVariable("WEBSITE_SITE_NAME"));

    context.done();
};

function GetEnvironmentVariable(name)
{
    return name + ": " + process.env[name];
}
```

## <a name="configure-function-entry-point"></a>設定函式進入點

`function.json` 屬性 `scriptFile` 和 `entryPoint` 可用來對於匯出的函式設定位置和名稱。 如果已轉換 JavaScript，這些相當重要。

### <a name="using-scriptfile"></a>使用 `scriptFile`

預設會從 `index.js` 執行 JavaScript 函式，這是與對應的 `function.json` 共用相同父目錄的檔案。

`scriptFile` 可用來取得如下所示的資料夾結構：
```
FunctionApp
 | - host.json
 | - myNodeFunction
 | | - function.json
 | - lib
 | | - nodeFunction.js
 | - node_modules
 | | - ... packages ...
 | - package.json
```

`myNodeFunction` 的 `function.json` 應該包含 `scriptFile` 屬性，這個屬性指向有匯出的函式要執行的檔案。
```json
{
  "scriptFile": "../lib/nodeFunction.js",
  "bindings": [
    ...
  ]
}
```

### <a name="using-entrypoint"></a>使用 `entryPoint`

在 `scriptFile` (或 `index.js`) 中，必須使用 `module.exports` 匯出函式，才能找到並執行該函式。 觸發時執行的函式預設是僅來自該檔案的匯出，名為 `run` 的匯出，或名為 `index` 的匯出。

這可以在 `function.json` 中使用 `entryPoint` 設定：
```json
{
  "entryPoint": "logFoo",
  "bindings": [
    ...
  ]
}
```

在 Functions v2.x (在使用者函式中支援 `this` 參數) 中，函式程式碼則可能如下所示：
```javascript
class MyObj {
    constructor() {
        this.foo = 1;
    };
    
    function logFoo(context) { 
        context.log("Foo is " + this.foo); 
        context.done(); 
    }
}

const myObj = new MyObj();
module.exports = myObj;
```

在此範例中必須注意，雖然正在匯出物件，但不保證在執行之間保留狀態。

## <a name="considerations-for-javascript-functions"></a>JavaScript 函式的考量

當您使用 JavaScript 函式時，請留意下列小節中的考量事項。

### <a name="choose-single-vcpu-app-service-plans"></a>選擇單一 vCPU App Service 方案

當您建立使用 App Service 方案的函數應用程式時，建議您選取單一 vCPU 方案，而非具有多個 vCPU 的方案。 目前 Functions 在單一 vCPU VM 上執行 JavaScript 函式會較有效率，而使用較大的 VM 並不會產生預期的效能改進。 如有必要，您可以新增更多單一 vCPU VM 執行個體來手動進行相應放大，或者您可以啟用自動規模調整。 如需詳細資訊，請參閱[手動或自動調整執行個體計數規模](../monitoring-and-diagnostics/insights-how-to-scale.md?toc=%2fazure%2fapp-service-web%2ftoc.json)。    

### <a name="typescript-and-coffeescript-support"></a>TypeScript 和 CoffeeScript 支援
由於目前仍沒有針對透過執行階段自動編譯 TypeScript 或 CoffeeScript 的直接支援，因此需要在部署時期於執行階段之外處理這些支援。 

### <a name="cold-start"></a>冷啟動
在無伺服器裝載模型中開發 Azure Functions 時，可進行冷啟動。 「冷啟動」是指函數應用程式在閒置一段時間之後進行的第一次啟動，這需要較長的時間啟動。 尤其對於大型相依性樹狀結構的 JavaScript 函式，這可能會導致執行速度明顯變慢。 若要加快程序，請盡可能[以封裝檔案的形式執行函式](run-functions-from-deployment-package.md)。 許多的部署方法預設選擇此模型，但是，如果您進行許多冷啟動，而且並非從封裝檔案執行，則可以達到大幅改善的效果。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱下列資源：

* [Azure Functions 的最佳做法](functions-best-practices.md)
* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)

