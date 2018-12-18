---
title: 測試 Azure Functions | Microsoft Docs
description: 使用 Postman、cURL、和 Node.js 來測試您的 Azure Functions。
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, 函數, 事件處理, webhook, 動態計算, 無伺服器架構, 測試
ms.assetid: c00f3082-30d2-46b3-96ea-34faf2f15f77
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 02/02/2017
ms.author: glenga
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a57d5f8d857a8cfcdc81e86650466aec740f41e3
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286804"
---
# <a name="strategies-for-testing-your-code-in-azure-functions"></a>在 Azure Functions 中測試程式碼的策略

本主題會示範測試函式的各種方法，包括下列一般方式：

+ HTTP 型的工具，例如 cURL、Postman，甚至是適用於 Web 型觸發程序的網頁瀏覽器
+ 用於測試以 Azure 儲存體為基礎之觸發程序的 Azure 儲存體總管
+ Azure Functions 入口網站中的 [測試] 索引標籤
+ 計時器觸發函式
+ 測試應用程式或架構

所有測試方法都是使用會透過查詢字串參數或要求主體接受輸入的 HTTP 觸發程序函式。 您會在第一節中使用 Azure 入口網站建立此函式。

## <a name="create-a-simple-function-for-testing-using-the-azure-portal"></a>對於使用 Azure 入口網站進行測試建立簡單函式
在本教學課程中的大多數時間裡，我們將使用建立函式時可使用的 HttpTrigger JavaScript 函式範本的稍經修改版本。 如果您需要建立函式的協助，請檢閱這個[教學課程](functions-create-first-azure-function.md)。 在 [Azure 入口網站]中建立測試函式時，選擇 **HttpTrigger- JavaScript** 範本。

預設函式範本基本上是 "hello world" 函式，可從要求主體或查詢字串參數 `name=<your name>`回應名稱。  我們將更新程式碼，以讓您在要求主體中以 JSON 內容的形式提供名稱和地址。 然後函式會在可使用時將這些內容回應給用戶端。   

使用我們將用來測試的下列程式碼更新函式︰

```javascript
module.exports = function (context, req) {
    context.log("HTTP trigger function processed a request. RequestUri=%s", req.originalUrl);
    context.log("Request Headers = " + JSON.stringify(req.headers));
    var res;

    if (req.query.name || (req.body && req.body.name)) {
        if (typeof req.query.name != "undefined") {
            context.log("Name was provided as a query string param...");
            res = ProcessNewUserInformation(context, req.query.name);
        }
        else {
            context.log("Processing user info from request body...");
            res = ProcessNewUserInformation(context, req.body.name, req.body.address);
        }
    }
    else {
        res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done(null, res);
};
function ProcessNewUserInformation(context, name, address) {
    context.log("Processing user information...");
    context.log("name = " + name);
    var echoString = "Hello " + name;
    var res;

    if (typeof address != "undefined") {
        echoString += "\n" + "The address you provided is " + address;
        context.log("address = " + address);
    }
    res = {
        // status: 200, /* Defaults to 200 */
        body: echoString
    };
    return res;
}
```

## <a name="test-a-function-with-tools"></a>使用工具測試函式
在 Azure 入口網站之外，您可以使用數種工具來針對測試觸發函式。 這些工具包含 HTTP 測試工具 (包括 UI 型和命令列)、Azure 儲存體存取工具，以及甚至簡易的網頁瀏覽器。

### <a name="test-with-a-browser"></a>使用瀏覽器測試
網頁瀏覽器是透過 HTTP 觸發函數的簡易方法。 您可以針對不需要主體承載並僅使用查詢字串參數的 GET 要求使用瀏覽器。

若要測試稍早定義的函式，請從入口網站複製**函式 URL** 。 其具備下列格式：

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

將 `name` 參數附加至查詢字串。 針對 `<Enter a name here>` 預留位置使用實際的名稱。

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>&name=<Enter a name here>

將此 URL 貼入瀏覽器，應該會得到預似以下的回應。

![Chrome 瀏覽器索引標籤具有測試回應的螢幕擷取畫面](./media/functions-test-a-function/browser-test.png)

此範例使用 Chrome 瀏覽器，它會將傳回字串包裝在 XML 中。 其他瀏覽器只會顯示字串值。

在入口網站 [記錄] 視窗中，執行函式時會記錄類似下面的輸出︰

    2016-03-23T07:34:59  Welcome, you are now connected to log-streaming service.
    2016-03-23T07:35:09.195 Function started (Id=61a8c5a9-5e44-4da0-909d-91d293f20445)
    2016-03-23T07:35:10.338 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==&name=Glenn from a browser
    2016-03-23T07:35:10.338 Request Headers = {"cache-control":"max-age=0","connection":"Keep-Alive","accept":"text/html","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T07:35:10.338 Name was provided as a query string param.
    2016-03-23T07:35:10.338 Processing User Information...
    2016-03-23T07:35:10.369 Function completed (Success, Id=61a8c5a9-5e44-4da0-909d-91d293f20445)

### <a name="test-with-postman"></a>使用 Postman 測試
測試多數函數的建議工具是 Postman，它能與 Chrome 瀏覽器整合。 若要安裝 Postman，請參閱 [取得 Postman](https://www.getpostman.com/)。 Postman 可對 HTTP 要求的許多其他屬性提供控制。

> [!TIP]
> 請使用您最熟悉的 HTTP 測試工具。 以下是 Postman 的一些替代方案︰  
>
> * [Fiddler](http://www.telerik.com/fiddler)  
> * [Paw](https://luckymarmot.com/paw)  
>
>

若要在 Postman 中測試要求主體的函數︰

1. 從 Chrome 瀏覽器視窗左上角的 [應用程式] 按鈕啟動 Postman。
2. 複製您的**函式 Url**，並將它貼到 Postman 中。 它包含存取程式碼查詢字串參數。
3. 將 HTTP 方法變更為 **POST**。
4. 按一下 [主體] > [原始]，並新增類似以下的 JSON 要求主體︰

    ```json
    {
        "name" : "Wes testing with Postman",
        "address" : "Seattle, WA 98101"
    }
    ```
5. 按一下 [傳送] 。

下圖顯示在本教學課程中測試簡單的回應函數範例。

![Postman 使用者介面的螢幕擷取畫面](./media/functions-test-a-function/postman-test.png)

在入口網站 [記錄] 視窗中，執行函式時會記錄類似下面的輸出︰

    2016-03-23T08:04:51  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:04:57.107 Function started (Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
    2016-03-23T08:04:57.763 HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==
    2016-03-23T08:04:57.763 Request Headers = {"cache-control":"no-cache","connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T08:04:57.763 Processing user info from request body...
    2016-03-23T08:04:57.763 Processing User Information...
    2016-03-23T08:04:57.763 name = Wes testing with Postman
    2016-03-23T08:04:57.763 address = Seattle, W.A. 98101
    2016-03-23T08:04:57.795 Function completed (Success, Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)

### <a name="test-with-curl-from-the-command-line"></a>從命令列透過 cURL 進行測試
測試軟體時往往只需要命令列即可協助偵錯您的應用程式。 對於測試函式也是如此。 請注意，以 Linux 為基礎的系統預設可以使用 cURL。 在 Windows 上，您必須先下載並安裝 [cURL 工具 (英文)](https://curl.haxx.se/)。

若要測試稍早定義的函式，請從入口網站複製**函式 URL**。 其具備下列格式：

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

這是用來觸發函式的 URL。 在命令列上使用 cURL 命令來測試它，以針對函式執行 GET (`-G` 或 `--get`) 要求︰

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

這個特定範例需要在 cURL 命令中將查詢字串參數傳遞做為資料 (`-d`)︰

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code> -d name=<Enter a name here>

執行該命令後，您會在命令列上看到函式的下列輸出：

![命令提示字元輸出的螢幕擷取畫面](./media/functions-test-a-function/curl-test.png)

在入口網站 [記錄] 視窗中，執行函式時會記錄類似下面的輸出︰

    2016-04-05T21:55:09  Welcome, you are now connected to log-streaming service.
    2016-04-05T21:55:30.738 Function started (Id=ae6955da-29db-401a-b706-482fcd1b8f7a)
    2016-04-05T21:55:30.738 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/HttpTriggerNodeJS1?code=XXXXXXX&name=Azure Functions
    2016-04-05T21:55:30.738 Function completed (Success, Id=ae6955da-29db-401a-b706-482fcd1b8f7a)

### <a name="test-a-blob-trigger-by-using-storage-explorer"></a>使用儲存體總管測試 blob 觸發程序
您可以使用 [Azure 儲存體總管](http://storageexplorer.com/)來測試 blob 觸發程序函式。

1. 在您函式應用程式的 [Azure 入口網站]中，建立 C#、F# 或 JavaScript Blob 觸發程序函式。 將要監視的路徑設定為您的 blob 容器名稱。 例如︰

        files
2. 按一下 **+** 按鈕以選取或建立您想要使用的儲存體帳戶。 接著，按一下 [建立]。
3. 建立含有下列內容的文字檔，並儲存該檔案：

        A text file for blob trigger function testing.
4. 執行 [Azure 儲存體總管](http://storageexplorer.com/)，並連線要監視的儲存體帳戶中的 blob 容器。
5. 按一下 [上傳] 以上傳文字檔案。

    ![儲存體總管的螢幕擷取畫面](./media/functions-test-a-function/azure-storage-explorer-test.png)

預設 blob 觸發程序的函式程式碼將會在記錄中報告 blob 的處理：

    2016-03-24T11:30:10  Welcome, you are now connected to log-streaming service.
    2016-03-24T11:30:34.472 Function started (Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)
    2016-03-24T11:30:34.472 C# Blob trigger function processed: A text file for blob trigger function testing.
    2016-03-24T11:30:34.472 Function completed (Success, Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)

## <a name="test-a-function-within-functions"></a>在 Functions 內測試函數
Azure Functions 入口網站是為了讓您測試 HTTP 和計時器觸發的函數而設計。 您也可以建立函數以觸發其他您正在測試的函數。

### <a name="test-with-the-functions-portal-run-button"></a>使用 Functions 入口網站的執行按鈕測試
入口網站提供 [執行] 按鈕，可讓您執行一些有限的測試。 您可以使用按鈕提供要求主體，但無法提供查詢字串參數或更新要求標頭。

藉由在 [要求主體] 欄位中新增類似以下的 JSON 字串，來測試我們稍早建立的 HTTP 觸發程序函式。 然後按一下 [執行] 按鈕。

```json
{
    "name" : "Wes testing Run button",
    "address" : "USA"
}
```

在入口網站 [記錄] 視窗中，執行函式時會記錄類似下面的輸出︰

    2016-03-23T08:03:12  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:03:17.357 Function started (Id=753a01b0-45a8-4125-a030-3ad543a89409)
    2016-03-23T08:03:18.697 HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/wesmchttptriggernodejs1
    2016-03-23T08:03:18.697 Request Headers = {"connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T08:03:18.697 Processing user info from request body...
    2016-03-23T08:03:18.697 Processing User Information...
    2016-03-23T08:03:18.697 name = Wes testing Run button
    2016-03-23T08:03:18.697 address = USA
    2016-03-23T08:03:18.744 Function completed (Success, Id=753a01b0-45a8-4125-a030-3ad543a89409)


### <a name="test-with-a-timer-trigger"></a>使用計時器觸發程序測試
有些函式無法適當地使用先前所述的工具進行測試。 例如，請考量將訊息放入 [Azure 佇列儲存體](../storage/queues/storage-dotnet-how-to-use-queues.md)時執行的佇列觸發程序函式。 您一律可以撰寫程式碼來將訊息放入佇列，本文稍後會提供在主控台專案中進行此動作的範例。 不過，還有另一種方法可用來直接使用函式進行測試。  

您可以使用設定了佇列輸出繫結的計時器觸發程序。 該計時器觸發程序程式碼會接著將測試訊息寫入至佇列。 本節會逐步解說範例。

如需使用 Azure Function 搭配使用繫結的深入資訊，請參閱 [Azure Functions 開發人員參考](functions-reference.md)。

#### <a name="create-a-queue-trigger-for-testing"></a>建立用於測試的佇列觸發程序
為了示範此方法，我們會先建立要對名為 `queue-newusers`的佇列進行測試的佇列觸發程序函式。 此函式會處理放入新使用者佇列儲存體的名稱和地址資訊。

> [!NOTE]
> 如果您使用不同的佇列名稱，請確定您使用的名稱符合 [命名佇列和中繼資料](https://msdn.microsoft.com/library/dd179349.aspx) 的規則。 否則，您會收到錯誤。
>
>

1. 在函式應用程式的 [Azure 入口網站]中，按一下 [新增函式] > [QueueTrigger - C#]。
2. 輸入要使用佇列函式監視的佇列名稱：

        queue-newusers
3. 按一下 **+** 按鈕以選取或建立您想要使用的儲存體帳戶。 接著，按一下 [建立]。
4. 將此入口網站瀏覽器視窗保持開啟，使得您可以監視預設的佇列函式範本程式碼的記錄項目。

#### <a name="create-a-timer-trigger-to-drop-a-message-in-the-queue"></a>建立要將訊息放在佇列中的計時器觸發程序
1. 在新的瀏覽器視窗中開啟 [Azure 入口網站]，並瀏覽至您的函式應用程式。
2. 按一下 [新增函式] > [TimerTrigger - C#]。 輸入 cron 運算式來設定計時器程式碼會測試您的佇列函式的頻率。 然後按一下 [ **建立**]。 如果您想要每隔 30 秒執行測試，您可以使用下列 [CRON 運算式](https://wikipedia.org/wiki/Cron#CRON_expression)：

        */30 * * * * *
3. 按一下新計時器觸發程序的 [整合]  索引標籤。
4. 在 [輸出] 下，按一下 [+ 新輸出]。 然後按一下 [佇列] 和 [選取]。
5. 記下您用於**佇列訊息物件**的名稱。 您可以在計時器函式程式碼中使用它。

        myQueue
6. 輸入會傳送訊息的目標佇列名稱︰

        queue-newusers
7. 按一下 **+** 按鈕以選取您先前用於佇列觸發程序的儲存體帳戶。 然後按一下 [儲存] 。
8. 按一下計時器觸發程序的 [開發]  索引標籤。
9. 只要使用稍早顯示的相同佇列訊息物件名稱，您即可以對 C# 計時器函式使用下列程式碼。 然後按一下 [儲存] 。

    ```cs
    using System;

    public static void Run(TimerInfo myTimer, out String myQueue, TraceWriter log)
    {
        String newUser =
        "{\"name\":\"User testing from C# timer function\",\"address\":\"XYZ\"}";

        log.Verbose($"C# Timer trigger function executed at: {DateTime.Now}");   
        log.Verbose($"{newUser}");   

        myQueue = newUser;
    }
    ```

目前，如果您已使用範例 cron 運算式，C# 計時器函式會每隔 30 秒執行。 計時器函式的記錄會報告每次執行︰

    2016-03-24T10:27:02  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.004 Function started (Id=04061790-974f-4043-b851-48bd4ac424d1)
    2016-03-24T10:27:30.004 C# Timer trigger function executed at: 3/24/2016 10:27:30 AM
    2016-03-24T10:27:30.004 {"name":"User testing from C# timer function","address":"XYZ"}
    2016-03-24T10:27:30.004 Function completed (Success, Id=04061790-974f-4043-b851-48bd4ac424d1)

在佇列函式的瀏覽器視窗中，您會看到正在處理的每個訊息︰

    2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
    2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"User testing from C# timer function","address":"XYZ"}
    2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)

## <a name="test-a-function-with-code"></a>使用程式碼測試函式
您需要建立外部應用程式或架構以測試您的函式。

### <a name="test-an-http-trigger-function-with-code-nodejs"></a>使用 Node.js 程式碼測試 HTTP 觸發程序函式
您可以使用 Node.js App 來執行 HTTP 要求，以測試您的函式。
請務必設定：

* 要求選項中的 `host` 設為您的函式應用程式主機。
* 在 `path` 中設定您的函數名稱。
* 在 `path` 中設定您的存取程式碼 (`<your code>`)。

程式碼範例：

```javascript
var http = require("http");

var nameQueryString = "name=Wes%20Query%20String%20Test%20From%20Node.js";

var nameBodyJSON = {
    name : "Wes testing with Node.JS code",
    address : "Dallas, T.X. 75201"
};

var bodyString = JSON.stringify(nameBodyJSON);

var options = {
  host: "functions841def78.azurewebsites.net",
  //path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9&" + nameQueryString,
  path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9",
  method: "POST",
  headers : {
      "Content-Type":"application/json",
      "Content-Length": Buffer.byteLength(bodyString)
    }    
};

callback = function(response) {
  var str = ""
  response.on("data", function (chunk) {
    str += chunk;
  });

  response.on("end", function () {
    console.log(str);
  });
}

var req = http.request(options, callback);
console.log("*** Sending name and address in body ***");
console.log(bodyString);
req.end(bodyString);
```


輸出：

    C:\Users\Wesley\testing\Node.js>node testHttpTriggerExample.js
    *** Sending name and address in body ***
    {"name" : "Wes testing with Node.JS code","address" : "Dallas, T.X. 75201"}
    Hello Wes testing with Node.JS code
    The address you provided is Dallas, T.X. 75201

在入口網站 [記錄] 視窗中，執行函式時會記錄類似下面的輸出︰

    2016-03-23T08:08:55  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:08:59.736 Function started (Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
    2016-03-23T08:09:01.153 HTTP trigger function processed a request. RequestUri=http://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1/?code=XXXXXXXXXX==
    2016-03-23T08:09:01.153 Request Headers = {"connection":"Keep-Alive","host":"functionsExample.azurewebsites.net"}
    2016-03-23T08:09:01.153 Name not provided as query string param. Checking body...
    2016-03-23T08:09:01.153 Request Body Type = object
    2016-03-23T08:09:01.153 Request Body = [object Object]
    2016-03-23T08:09:01.153 Processing User Information...
    2016-03-23T08:09:01.215 Function completed (Success, Id=607b891c-08a1-427f-910c-af64ae4f7f9c)


### <a name="test-a-queue-trigger-function-with-code-c"></a>使用 C# 程式碼測試佇列觸發程序函式 #
我們稍早提到，您可以透過使用程式碼在您的佇列中放置訊息來測試佇列觸發程序。 下列範例程式碼是基於[開始使用 Azure 佇列儲存體](../storage/queues/storage-dotnet-how-to-use-queues.md)教學課程中提供的 C# 程式碼。 從該連結也可以取得其他語言的程式碼。

若要在主控台應用程式中測試此程式碼，您必須︰

* [在 app.config 檔案中設定儲存體連接字串](../storage/queues/storage-dotnet-how-to-use-queues.md)。
* 傳遞 `name` 和 `address` 做為應用程式的參數。 例如： `C:\myQueueConsoleApp\test.exe "Wes testing queues" "in a console app"`。 (在執行階段期間，此程式碼會接受新使用者名稱和地址做為命令列引數。)

範例 C# 程式碼︰

```cs
static void Main(string[] args)
{
    string name = null;
    string address = null;
    string queueName = "queue-newusers";
    string JSON = null;

    if (args.Length > 0)
    {
        name = args[0];
    }
    if (args.Length > 1)
    {
        address = args[1];
    }

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference(queueName);

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExists();

    // Create a message and add it to the queue.
    if (name != null)
    {
        if (address != null)
            JSON = String.Format("{{\"name\":\"{0}\",\"address\":\"{1}\"}}", name, address);
        else
            JSON = String.Format("{{\"name\":\"{0}\"}}", name);
    }

    Console.WriteLine("Adding message to " + queueName + "...");
    Console.WriteLine(JSON);

    CloudQueueMessage message = new CloudQueueMessage(JSON);
    queue.AddMessage(message);
}
```

在佇列函式的瀏覽器視窗中，您會看到正在處理的每個訊息︰

    2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
    2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"Wes testing queues","address":"in a console app"}
    2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)


<!-- URLs. -->

[Azure 入口網站]: https://portal.azure.com
