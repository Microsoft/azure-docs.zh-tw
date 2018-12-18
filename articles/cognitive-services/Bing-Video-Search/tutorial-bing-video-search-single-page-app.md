---
title: 教學課程：建置單頁 Bing 影片搜尋應用程式
titlesuffix: Azure Cognitive Services
description: 說明如何在單頁 Web 應用程式中使用 Bing 影片搜尋 API。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: tutorial
ms.date: 11/01/2017
ms.author: rosh
ms.openlocfilehash: a7c6646a69aec11797d354da28baca669b802ab0
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226597"
---
# <a name="tutorial-single-page-video-search-app"></a>教學課程：單頁影片搜尋應用程式
Bing 影片搜尋 API 可讓您搜尋網頁，並取得與搜尋查詢相關的影片結果。 在本教學課程中，我們會建置單頁 Web 應用程式，以使用 Bing 影片搜尋 API 在頁面中顯示搜尋結果。 該應用程式包含 HTML、CSS 和 JavaScript 元件。

<!-- Remove until it can be replaced with a sanitized version.
![Single-page Bing Video Search app](./media/video-search-singlepage.png)
-->

> [!NOTE]
> 按下時，頁面底部的 JSON 和 HTTP 標題會顯示 JSON 回應和 HTTP 要求資訊。 這些詳細資料有助於探索服務。

![JSON、HTTP 未經處理的結果](./media/json-http-raw-results.png)

本教學課程應用程式說明如何：
> [!div class="checklist"]
> * 在 JavaScript 中執行 Bing 影片搜尋 API 呼叫
> * 將搜尋選項傳至 Bing 搜尋 API
> * 顯示影片搜尋結果，或選擇性地包含網頁、新聞或影像
> * 搜尋 24 小時、過去一週、月或所有可用時間的時段
> * 逐頁檢視搜尋結果
> * 處理 Bing 用戶端識別碼和 API 訂用帳戶金鑰
> * 處理可能會發生的錯誤

本教學課程頁完全獨立，不會使用任何外部架構、樣式表或影像檔。 它使用唯一且廣泛支援的 JavaScript 語言功能，適用於所有主要網頁瀏覽器的目前版本。

在本教學課程中，我們會討論原始程式碼的選定部分。 有完整的[原始程式碼](tutorial-bing-video-search-single-page-app-source.md)可供使用。 若要執行範例，請將原始程式碼複製並貼入文字編輯器，並將其儲存為 `bing.html`。

## <a name="app-components"></a>應用程式元件
如同任何單頁 Web 應用程式，本教學課程應用程式包含三個部分：

> [!div class="checklist"]
> * HTML - 定義頁面的結構和內容
> * CSS - 定義頁面的外觀
> * JavaScript - 定義頁面的行為

大部分的 HTML 和 CSS 是常見項目，因此本教學課程不會加以討論。 HTML 包含搜尋表單，使用者可在其中輸入查詢並選擇搜尋選項。 該表單會使用 `<form>` 標籤的 `onsubmit` 屬性，連接至執行搜尋的 JavaScript：

```html
<form name="bing" onsubmit="return bingWebSearch(this)">
```
`onsubmit` 處理常式會傳回 `false`，這可防止表單提交至伺服器。 JavaScript 程式碼會從表單收集所需的資訊，並執行搜尋。

HTML 也包含顯示搜尋結果的區域 (HTML `<div>` 標籤)。

## <a name="managing-subscription-key"></a>管理訂用帳戶金鑰

為了避免必須在程式碼中包含 Bing 搜尋 API 訂用帳戶金鑰，我們使用瀏覽器的永續性儲存體來儲存金鑰。 儲存此金鑰之前，我們會提示輸入使用者的金鑰。 若 API 稍後拒絕金鑰，我們會讓儲存的金鑰失效，以便再次提示使用者。

我們會定義 `storeValue` 和 `retrieveValue` 函式，以使用 `localStorage` 物件 (並非所有瀏覽器都支援它) 或 Cookie。 `getSubscriptionKey()` 函式使用這些函式來儲存及擷取使用者的金鑰。

``` javascript
// Cookie names for data we store
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

// ... omitted definitions of store value and retrieve value
// Browsers differ in their support for persistent storage by 
// local HTML files. See the source code for browser-specific
// options.

// Get stored API subscription key, or prompt if it's not found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```
HTML `<form>` 標籤 `onsubmit` 會呼叫 `bingWebSearch` 函式來傳回搜尋結果。 `bingWebSearch` 使用 `getSubscriptionKey()` 來驗證每個查詢。 如先前的定義所示，若尚未輸入金鑰．`getSubscriptionKey` 就會提示使用者輸入金鑰。 其後應儲存金鑰，以供應用程式繼續使用。

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value, 
    bingSearchOptions(this), getSubscriptionKey())">
```
## <a name="selecting-search-options"></a>選取搜尋選項
下圖顯示查詢文字方塊，以及定義搜尋的選項。

![Bing 新聞搜尋選項](media/video-search-options.png)

HTML 表單包含具有下列名稱的項目：

|元素|說明|
|-|-|
| `where` | 可供選取市場 (位置和語言) 以用於搜尋的下拉式功能表。 |
| `query` | 要輸入搜尋字詞的文字欄位。 |
| `modules` | 可供升級特定結果模組、所有結果或相關影片的核取方塊。 |
| `when` | 可選擇性地將搜尋限定為最近一天、一週或一個月的下拉式功能表。 |
| `safe` | 指明是否要使用 Bing 安全搜尋功能來篩選掉「成人」結果的核取方塊。 |
| `count` | 隱藏的欄位。 毎個要求要傳回的搜尋結果數目。 變更為每頁顯示更多或更少結果。 |
| `offset`|  隱藏的欄位。 要求中第一個搜尋結果的位移；用於分頁。 它會在新要求執行時重設為 `0`。 |

> [!NOTE]
> Bing Web 搜尋提供其他查詢參數。 我們在這裡只使用了其中一些參數。

``` javascript
// build query options from the HTML form
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));

    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++) 
        if (form.what[i].checked) what.push(form.what[i].value);
    if (what.length) {
        options.push("modules=" + what.join(","));
        options.push("answerCount=9");
    }
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    options.push("textDecorations=true");
    options.push("textFormat=HTML");
    return options.join("&");
}
```

例如，實際 API 呼叫中的 `SafeSearch` 參數可以是 `strict`、`moderate` 或 `off`，預設值為 `moderate`。 不過，我們的表單使用只有兩種狀態的核取方塊。 JavaScript 程式碼會將此設定轉換成 `strict` 或 `off` (不會使用 `moderate`)。

## <a name="performing-the-request"></a>執行要求
假設有查詢、選項字串和 API 金鑰，`BingWebSearch` 函式會使用 `XMLHttpRequest` 物件向 Bing 搜尋端點提出要求。

```javascript
// Search on the query, using search options, authenticated by the key.
function bingWebSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_headers", "paging1", "paging2", "error");

    var endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/videos/search";
    var request = new XMLHttpRequest();
    var queryurl = endpoint + "?q=" + encodeURIComponent(query) + "&" + options;

    try {
        request.open("GET", queryurl);
    } 
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // add request headers
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // event handler for successful response
    request.addEventListener("load", handleOnLoad);

    // event handler for erorrs
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // event handler for aborted request
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // send the request
    request.send();
    return false;
}
```
HTTP 要求成功完成時，JavaScript 會呼叫 `load` 事件處理常式 `handleOnLoad()`，來處理成功傳遞至 API 的 HTTP GET 要求。 

```javascript
// handle Bing search request results
function handleOnLoad() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and headers
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "Videos") {//"SearchResponse" && "rankingResponse" in jsobj) {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSubscriptionKey();

        // some error responses don't have a top-level errors object, so gin one up
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // display HTTP status code
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // add all fields from all error responses
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // also display Bing Trace ID if it isn't blocked by CORS
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // and display the error message
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> 如果搜尋作業中發生錯誤，Bing 新聞搜尋 API 會傳回非 200 HTTP 狀態碼，並在 JSON 回應中包含錯誤資訊。 此外，如果要求速率受到限制，API 會傳回空白回應。
成功的 HTTP 要求「不」一定表示搜尋本身成功。 

上述兩個函式中的大部分程式碼都是專用於錯誤處理。 下列階段可能會發生錯誤：

|階段|可能的錯誤|處理者|
|-|-|-|
|建置 JavaScript 要求物件|無效的 URL|`try`/`catch` 區塊|
|提出要求|網路錯誤、已中止連線|`error` 和 `abort` 事件處理常式|
|執行搜尋|無效的要求、無效的 JSON、速率限制|`load` 事件處理常式中的測試|

錯誤是透過呼叫 `renderErrorMessage()` 來處理，並提供有關錯誤的任何已知詳細資料。 若回應通過錯誤測試的所有挑戰，我們會呼叫 `renderSearchResults()` 在頁面中顯示搜尋結果。

## <a name="displaying-search-results"></a>顯示搜尋結果
用於顯示搜尋結果的主要函式是 `renderSearchResults()`。 此函式會接受 Bing 新聞搜尋服務所傳回的 JSON，並轉譯新聞結果和相關的搜尋 (若有的話)。

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the resuts to the mainline section
    for (section in { mainline: 0 }) {
         showDiv(section, renderResultsItems(section, results));
    }
}
```
搜尋結果會傳回成 JSON 回應中的最上層 `value` 物件。 我們將這些結果傳遞給函式 `renderResultsItems()`，它會逐一查看這些結果，並呼叫函式將每個項目轉譯成 HTML。 產生的 HTML 將傳回給 `renderSearchResults()`，在其中它會插入頁面中的 `results` 區域。

```javascript
// render search results
    function renderResultsItems(section, results) {   

        var items = results.value;
        var html = [];
        for (var i = 0; i < items.length; i++) { 
            var item = items[i];
            // collection name has lowercase first letter
            var type = "videos";
            var render = searchItemRenderers[type];
            html.push(render(item, section));  
        }
    return html.join("\n\n");
}
```

Bing 新聞搜尋 API 最多傳回四種不同的相關結果，每個都出現在它自己的最上層物件中。 如下：

|關聯|說明|
|-|-|
|`pivotSuggestions`|將原始搜尋中的樞紐字組取代為不同樞紐字組的查詢。 比方說，若您搜尋「紅色花卉」，樞紐字組可能是「紅色」，而樞紐建議可能是「黃色花卉」。|
|`queryExpansions`|藉由新增多個字詞以縮小原始搜尋範圍的查詢。 比方說，若您搜尋 "Microsoft Surface"，可能是查詢擴充可能是 "Microsoft Surface Pro"。|
|`relatedSearches`|也已由輸入原始搜尋的其他使用者輸入的查詢。 比方說，若您搜尋「雷尼爾山」，相關搜尋可能是「 聖海倫山」。|
|`similarTerms`|與原始搜尋的意義類似的查詢。 例如，若您搜尋「學校」，類似的字詞可能是「教育」。|

如 `renderSearchResults()` 先前所示，我們只會轉譯 `relatedItems` 建議，並將產生的連結置於頁面的資訊看板中。

## <a name="rendering-result-items"></a>轉譯結果項目

在 JavaScript 程式碼中有一個物件 `searchItemRenderers` 包含 *renderers:* 函式，可為每種搜尋結果產生 HTML。 影片搜尋頁面僅使用 `videos`。 各種類型的轉譯器，請參閱其他教學課程。

```javascript
searchItemRenderers = {
    news: function(item) { ... },
    webPages: function (item) { ... }, 
    images: function(item, index, count) { ... },
    videos: function (item, section, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```
轉譯器函式可接受下列參數：

|參數|說明|
|-|-|
|`item`| JavaScript 物件，其中包含項目的屬性，例如其 URL 及其描述。|
|`index`| 集合內結果項目的索引。|
|`count`| 搜尋結果項目集合中的項目數目。|

`index` 和 `count` 參數可用來編號結果、為集合開頭或結尾產生特殊 HTML、在特定數量的項目之後插入分行符號等。 如果轉譯器不需要此功能，則不需要接受這兩個參數。

`video` 轉譯器會顯示在下列 Javascript 摘錄中。 使用影片端點，所有結果都屬於 `Videos` 類型。 `searchItemRenderers` 如下列程式碼片段所示。

```javascript
// render functions for various types of search results
    searchItemRenderers = {

    videos: function (item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];

        html.push("<p class='images'>");
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escapeQuotes(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='" + item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Video page source</a> - ");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }
}
```

轉譯器函式會：
> [!div class="checklist"]
> * 建立段落標籤，將其指派給 `images` 類別，並將其推送至 HTML 陣列。
> * 計算影像的縮圖大小 (寬度固定為 60 像素，高度則按比例計算)。
> * 建置 HTML `<img>` 標籤，以顯示影像縮圖。 
> * 建置 HTML `<a>` 標籤，以連結至影像和內含影像的頁面。
> * 建置描述，以顯示影像及執行所在網站的相關資訊。

縮圖大小用於這兩個 `<img>` 標籤，以及縮圖 URL 中的 `h` 和 `w` 欄位。 [Bing 縮圖服務](resize-and-crop-thumbnails.md)接著會提供完全符合該大小的縮圖。

## <a name="persisting-client-id"></a>保存用戶端識別碼
來自 Bing 搜尋 API 的回應可能會在後續要求中包含應該傳回 API 的 `X-MSEdge-ClientID` 標頭。 若使用多個 Bing 搜尋 API，請盡可能對所有 API 使用相同的用戶端識別碼。

提供 `X-MSEdge-ClientID` 標頭可讓 Bing API 建立所有使用者搜尋的關聯，這有兩個重要的優點。

首先，它可讓 Bing 搜尋引擎將過去內容套用至搜尋結果，以尋找更符合使用者的結果。 例如，若使用者之前搜尋與航行相關的字詞，稍後搜尋「節」可能會優先傳回航行中所使用節數的相關資訊。

其次，Bing 可能會隨機選取使用者來體驗新功能，再廣泛提供這些功能。 在每個要求中提供相同的用戶端識別碼，可確保看到功能的使用者一律會看到該功能。 若沒有用戶端識別碼，使用者可能會在其搜尋結果中看到功能出現並消失，似乎很隨機。

瀏覽器安全性原則 (CORS) 可防止將 `X-MSEdge-ClientID` 標頭提供給 JavaScript 使用。 當搜尋回應的來源與要求回應的頁面不同時，就會發生這項限制。 在生產環境中，您應該裝載伺服器端指令碼，在與網頁相同的網域上執行 API 呼叫，以處理此原則。 由於指令碼的來源與網頁相同，因此 `X-MSEdge-ClientID` 標頭會接著提供給 JavaScript 使用。

> [!NOTE]
> 在生產 Web 應用程式中，您應該執行要求伺服器端。 否則，您的 Bing 搜尋 API 金鑰必須包含在網頁中，以提供給檢視來源的任何人。 您會根據 API 訂用帳戶金鑰的所有使用量付費，即使是未經授權的合作對象所提出的要求，因此請務必不要公開您的金鑰。

若要進行開發，您可以透過 CORS Proxy 提出 Bing Web 搜尋 API 要求。 來自這類 Proxy 的回應包含 `Access-Control-Expose-Headers` 標頭，可將回應標頭列入白名單並提供給 JavaScript 使用。

您可以輕鬆安裝 CORS Proxy，讓我們的教學課程應用程式存取用戶端識別碼標頭。 首先，請[安裝 Node.js](https://nodejs.org/en/download/) (若尚未安裝)。 然後在命令視窗中發出下列命令：

    npm install -g cors-proxy-server

接下來，將 HTML 檔案中的 Bing Web 搜尋端點變更為：

    http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search

最後，使用下列命令啟動 CORS Proxy：

    cors-proxy-server

當您使用教學課程應用程式時，請保持開啟命令視窗；關閉視窗會停止 Proxy。 在可展開的 [HTTP 標頭] 區段搜尋結果下，您現在可以看到 `X-MSEdge-ClientID` 標頭 (及其他標頭)，並確認每個要求的此標頭都相同。

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [Bing 影片搜尋 API 參考](//docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference)