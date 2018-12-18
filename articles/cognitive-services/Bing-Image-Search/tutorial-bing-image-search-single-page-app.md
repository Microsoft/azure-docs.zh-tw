---
title: 教學課程：建立單頁 Web 應用程式 - Bing 影像搜尋 API
titleSuffix: Azure cognitive services
description: 「Bing 影像搜尋 API」可讓您搜尋 Web 來尋找高品質的相關影像。 您可以使用此教學課程來建置可將搜尋查詢傳送給該 API 並在網頁內顯示結果的單頁 Web 應用程式。
services: cognitive-services
author: aahi
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: tutorial
ms.date: 9/12/2018
ms.author: aahi
ms.openlocfilehash: e37cb9b9412d257ab238f23b90e4a1077070b2b6
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297446"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-image-search-api"></a>教學課程：使用 Bing 影像搜尋 API 來建立單頁應用程式

「Bing 影像搜尋 API」可讓您搜尋 Web 來尋找高品質的相關影像。 您可以使用此教學課程來建置可將搜尋查詢傳送給該 API 並在網頁內顯示結果的單頁 Web 應用程式。 此教學課程與「Bing Web 搜尋」的[相對應教學課程](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md)類似。

此教學課程應用程式說明如何：

> [!div class="checklist"]
> * 在 JavaScript 中執行 Bing 影像搜尋 API 呼叫
> * 使用搜尋選項來改善搜尋結果
> * 顯示搜尋結果並逐頁檢視
> * 要求及處理 API 訂用帳戶金鑰和 Bing 用戶端識別碼。

[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Image-Search) 上有此教學課程的完整原始程式碼。

## <a name="prerequisites"></a>先決條件

* 最新版的 [Node.js](https://nodejs.org/)。
* 適用於 Node.js 的 [Express.js](https://expressjs.com/) 架構。 GitHub 範例讀我檔案中會提供原始程式碼的安裝指示。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="manage-and-store-user-subscription-keys"></a>管理及儲存使用者訂用帳戶金鑰

此應用程式使用網頁瀏覽器的永續性儲存體來儲存 API 訂用帳戶金鑰。 如果未儲存任何金鑰，網頁將會提示使用者提供金鑰，並儲存該金鑰以供稍後使用。 如果該金鑰稍後被 API 拒絕，應用程式就會從儲存體中移除它。


請定義 `storeValue` 和 `retrieveValue` 函式以使用 `localStorage` 物件 (如果瀏覽器支援此物件) 或 Cookie。

```javascript
// Cookie names for data being stored
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";
// The Bing Image Search API endpoint
BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/images/search";

try { //Try to use localStorage first
    localStorage.getItem;   

    window.retrieveValue = function (name) {
        return localStorage.getItem(name) || "";
    }
    window.storeValue = function(name, value) {
        localStorage.setItem(name, value);
    }
} catch (e) {
    //If the browser doesn't support localStorage, try a cookie
    window.retrieveValue = function (name) {
        var cookies = document.cookie.split(";");
        for (var i = 0; i < cookies.length; i++) {
            var keyvalue = cookies[i].split("=");
            if (keyvalue[0].trim() === name) return keyvalue[1];
        }
        return "";
    }
    window.storeValue = function (name, value) {
        var expiry = new Date();
        expiry.setFullYear(expiry.getFullYear() + 1);
        document.cookie = name + "=" + value.trim() + "; expires=" + expiry.toUTCString();
    }
}
```

`getSubscriptionKey()` 函式會使用 `retrieveValue` 來嘗試擷取先前儲存的金鑰。 如果找不到任何金鑰，就會提示使用者提供其金鑰，並使用 `storeValue` 來儲存該金鑰。

```javascript

// Get the stored API subscription key, or prompt if it's not found
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

HTML `<form>` 標籤 `onsubmit` 會呼叫 `bingWebSearch` 函式來傳回搜尋結果。 `bingWebSearch` 使用 `getSubscriptionKey` 來驗證每個查詢。 如先前的定義所示，若尚未輸入金鑰．`getSubscriptionKey` 就會提示使用者輸入金鑰。 其後應儲存金鑰，以供應用程式繼續使用。

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value,
bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="send-search-requests"></a>傳送搜尋要求

此應用程式會使用 HTML `<form>` 來一開始傳送使用者搜尋要求，其中會使用 `onsubmit` 屬性來呼叫 `newBingImageSearch()`。

```html
<form name="bing" onsubmit="return newBingImageSearch(this)">
```

`onsubmit` 處理常式預設會傳回 `false`，以防止提交表單。

## <a name="select-search-options"></a>選取搜尋選項

![[Bing 影像搜尋表單]](media/cognitive-services-bing-images-api/image-search-spa-form.png)

「Bing 影像搜尋 API」提供數個[篩選查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#filter-query-parameters)，可針對搜尋結果縮小範圍及進行篩選。 此應用程式中的 HTML 表單會使用及顯示下列參數選項：

|              |                                                                                                                                                                                    |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `where`      | 可供選取市場 (位置和語言) 以用於搜尋的下拉式功能表。                                                                                             |
| `query`      | 要在其中輸入搜尋字詞的文字欄位。                                                                                                                                 |
| `aspect`     | 選項按鈕用於選擇找到的映像比例：大致是方形、寬或高。                                                                                     |
| `color`      |                                                                                                                                                                                    |
| `when`       | 下拉式功能表可選擇性地將搜尋限制在最近一天、一週或一個月。                                                                                          |
| `safe`       | 指出是否要使用 Bing 的安全搜尋功能來篩選掉「成人」結果的核取方塊。                                                                                      |
| `count`      | 隱藏的欄位。 毎個要求要傳回的搜尋結果數目。 變更為每頁顯示更多或更少結果。                                                            |
| `offset`     | 隱藏的欄位。 要求中第一個搜尋結果的位移；用於分頁。 它會在新要求執行時重設為 `0`。                                                           |
| `nextoffset` | 隱藏的欄位。 收到搜尋結果後，此欄位會設為回應中的 `nextOffset` 值。 使用此欄位可避免連續頁面上的重複結果。 |
| `stack`      | 隱藏的欄位。 JSON 編碼之上一頁搜尋結果的位移清單，用於導覽回上一個頁面。                                                      |

`bingSearchOptions()` 函式會將這些選項的格式製作成部分查詢字串，以供在應用程式的 API 要求中使用。  

```javascript
// Build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var aspect = "all";
    for (var i = 0; i < form.aspect.length; i++) {
        if (form.aspect[i].checked) {
            aspect = form.aspect[i].value;
            break;
        }
    }
    options.push("aspect=" + aspect);
    if (form.color.value) options.push("color=" + form.color.value);
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    return options.join("&");
}
```

## <a name="performing-the-request"></a>執行要求

在使用搜尋查詢、選項字串及 API 金鑰的情況下，`BingImageSearch()` 函式會使用 XMLHttpRequest 物件向「Bing 影像搜尋」端點提出要求。


```javascript
// perform a search given query, options string, and API key
function bingImageSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("results", "related", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // open the request
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
    request.addEventListener("load", handleBingResponse);

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

順利完成 HTTP 要求時，JavaScript 會呼叫「載入」事件處理常式 `handleBingResponse()` 來處理成功的 HTTP GET 要求。

```javascript
// handle Bing search request results
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and HTTP request
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "Images") {
                if (jsobj.nextOffset) document.forms.bing.nextoffset.value = jsobj.nextOffset;
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
> 成功的 HTTP 要求可能會包含失敗的搜尋資訊。 如果在進行搜尋作業時發生錯誤，「Bing 影像搜尋 API」將會傳回非 200 HTTP 狀態碼，並在 JSON 回應中包含錯誤資訊。 此外，若要求的速率受到限制，API 將會傳回空白回應。

## <a name="display-the-search-results"></a>顯示搜尋結果

`renderSearchResults()` 函式會顯示搜尋結果，此函式會接受「Bing 影像搜尋」服務所傳回的 JSON，並針對所傳回的任何影像和相關搜尋，呼叫適當的轉譯器函式。

```javascript
function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    showDiv("results", renderImageResults(results.value));
    if (results.relatedSearches)
        showDiv("sidebar", renderRelatedItems(results.relatedSearches));
}
```

影像搜尋結果會包含在 JSON 回應內的最上層 `value` 物件中。 這些會傳遞給 `renderImageResults()`，它會逐一查看結果並將每個項目轉換成 HTML。

```javascript
function renderImageResults(items) {
    var len = items.length;
    var html = [];
    if (!len) {
        showDiv("noresults", "No results.");
        hideDivs("paging1", "paging2");
        return "";
    }
    for (var i = 0; i < len; i++) {
        html.push(searchItemRenderers.images(items[i], i, len));
    }
    return html.join("\n\n");
}
```

「Bing 影像搜尋 API」可以傳回四種類型的搜尋建議來協助引導使用者的搜尋體驗，其中每種建議都在自己的最上層物件中：

| 建議         | 說明                                                                                                                                                                                                         |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pivotSuggestions` | 將原始搜尋中的樞紐字組取代為不同樞紐字組的查詢。 比方說，若您搜尋「紅色花卉」，樞紐字組可能是「紅色」，而樞紐建議可能是「黃色花卉」。 |
| `queryExpansions`  | 藉由新增多個字詞以縮小原始搜尋範圍的查詢。 比方說，若您搜尋 "Microsoft Surface"，可能是查詢擴充可能是 "Microsoft Surface Pro"。                                   |
| `relatedSearches`  | 也已由輸入原始搜尋的其他使用者輸入的查詢。 比方說，若您搜尋「雷尼爾山」，相關搜尋可能是「 聖海倫山」。                       |
| `similarTerms`     | 與原始搜尋的意義類似的查詢。 例如，如果您搜尋「小貓」，類似的字詞可能是「可愛」。                                                                   |

此應用程式只會轉譯 `relatedItems` 建議，並將產生的連結置於頁面的資訊看板中。

## <a name="rendering-search-results"></a>轉譯搜尋結果

在此應用程式中，`searchItemRenderers` 物件會包含可為每一種搜尋結果產生 HTML 的轉譯器函式。

```javascript
searchItemRenderers = {
    images: function(item, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```

這些轉譯器函式可接受下列參數：

| 參數         | 說明                                                                                              |
|---------|----------------------------------------------------------------------------------------------|
| `item`  | JavaScript 物件，其中包含項目的屬性，例如其 URL 及其描述。 |
| `index` | 集合內結果項目的索引。                                          |
| `count` | 搜尋結果項目集合中的項目數目。                                  |

`index` 和 `count` 參數可用來將結果編號、為集合產生 HTML，以及組織內容。 具體而言，它會：

* 計算影像縮圖大小 (寬度變動，最小值為 120 像素，而高度固定為 90 像素)。
* 建置 HTML `<img>` 標籤，以顯示影像縮圖。
* 建置 HTML `<a>` 標籤，以連結至影像和內含影像的頁面。
* 建置描述，以顯示影像及執行所在網站的相關資訊。

```javascript
    images: function (item, index, count) {
        var height = 120;
        var width = Math.max(Math.round(height * item.thumbnail.width / item.thumbnail.height), 120);
        var html = [];
        if (index === 0) html.push("<p class='images'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<p class='images' style='max-width: " + width + "px'>");
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + "'>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Image</a> - ");
        html.push("<a href='" + item.hostPageUrl + "'>Page</a></nobr><br>");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }, // relatedSearches renderer omitted
```

縮圖影像的 `height` 和 `width` 會同時用於 `<img>` 標記及縮圖 URL 中的 `h` 和 `w` 欄位。 這讓 Bing 能夠傳回與該大小完全相同的[縮圖](resize-and-crop-thumbnails.md)。

## <a name="persisting-client-id"></a>保存用戶端識別碼

來自 Bing 搜尋 API 的回應可能會包含應在後續要求中傳回至 API 的 `X-MSEdge-ClientID` 標頭。 若使用多個 Bing 搜尋 API，請盡可能對所有 API 使用相同的用戶端識別碼。

提供 `X-MSEdge-ClientID` 標頭可讓 Bing API 將使用者的所有搜尋建立關聯，這在下列情況下相當有用：

首先，它可讓 Bing 搜尋引擎將過去內容套用至搜尋結果，以尋找更符合使用者的結果。 例如，若使用者之前搜尋與航行相關的字詞，稍後搜尋「節」可能會優先傳回航行中所使用節數的相關資訊。

其次，Bing 可能會隨機選取使用者來體驗新功能，再廣泛提供這些功能。 在每個要求中提供相同的用戶端識別碼，可確保已選擇看到功能的使用者一律會看到功能。 若沒有用戶端識別碼，使用者可能會在其搜尋結果中隨機看到功能出現並消失。

瀏覽器安全性原則 (CORS) 可防止將 `X-MSEdge-ClientID` 標頭提供給 JavaScript 使用。 當搜尋回應的來源與要求回應的頁面不同時，就會發生此限制。 在生產環境中，您應該裝載伺服器端指令碼，在與網頁相同的網域上執行 API 呼叫，以處理此原則。 由於指令碼的來源與網頁相同，因此 `X-MSEdge-ClientID` 標頭會接著提供給 JavaScript 使用。

> [!NOTE]
> 在生產 Web 應用程式中，無論如何都應該執行要求伺服器端。 否則，您的 Bing 搜尋 API 金鑰必須包含在網頁中，以提供給檢視來源的任何人。 您會根據 API 訂用帳戶金鑰的所有使用量付費，即使是未經授權的合作對象所提出的要求，因此請務必不要公開您的金鑰。

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
> [使用 Bing 影像搜尋 API 來擷取影像詳細資料](tutorial-image-post.md)

## <a name="see-also"></a>另請參閱

* [Bing 影像搜尋 API 參考](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
