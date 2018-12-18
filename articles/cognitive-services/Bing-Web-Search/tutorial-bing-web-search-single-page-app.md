---
title: 教學課程：建立單頁 Web 應用程式 - Bing Web 搜尋 API
titleSuffix: Azure Cognitive Services
description: 此單頁應用程式示範如何使用 Bing Web 搜尋 API，在單頁應用程式中擷取、剖析及顯示相關的搜尋結果。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: tutorial
ms.date: 09/12/2018
ms.author: erhopf
ms.openlocfilehash: 670f02cbd8e994664e7c4edd75940ff43f9616b6
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46126474"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-web-search-api"></a>教學課程：使用 Bing Web 搜尋 API 建立單頁應用程式

此單頁應用程式示範如何從 Bing Web 搜尋 API 擷取、剖析及顯示搜尋結果。 本教學課程使用樣本 HTML 和 CSS，並將重點放在 JavaScript 程式碼上。 HTML、CSS 和 JS 檔案均可於 [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Web-Search) 取得，其中含有快速入門指示。

此範例應用程式可以：

> [!div class="checklist"]
> * 使用搜尋選項呼叫 Bing Web 搜尋 API
> * 顯示 Web、影像、新聞和視訊結果
> * 將結果編頁
> * 管理訂用帳戶金鑰
> * 處理錯誤

若要使用此應用程式，需要具備 [Azure 認知服務帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)及 Bing 搜尋 API。 如果您還沒有帳戶，可以使用[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)來取得訂用帳戶金鑰。

## <a name="prerequisites"></a>先決條件

以下是數個您執行應用程式所需的項目：

* Node.js 8 或更新版本
* 訂用帳戶金鑰

## <a name="get-the-source-code-and-install-dependencies"></a>取得原始程式碼並安裝相依性

第一個步驟是使用範例應用程式的原始程式碼來複製存放庫。

```console
git clone https://github.com/Azure-Samples/cognitive-services-REST-api-samples.git
```

然後執行 `npm install`。 對於本教學課程，Express.js 是唯一的相依性。

```console
cd <path-to-repo>/cognitive-services-REST-api-samples/Tutorials/Bing-Web-Search
npm install
```

## <a name="app-components"></a>應用程式元件

我們要建置的範例應用程式由四個部分所組成：

* `bing-web-search.js` - 我們的 Express.js 應用程式。 它會處理要求/回應邏輯和路由。
* `public/index.html` - 我們應用程式的基本架構；它會定義向使用者呈現資料的方式。
* `public/css/styles.css` - 定義頁面樣式，例如字型、色彩、文字大小。
* `public/js/scripts.js` - 包含邏輯以對 Bing Web 搜尋 API 提出要求、管理訂用帳戶金鑰、處理和剖析回應，以及顯示結果。

本教學課程著重於 `scripts.js`，以及呼叫 Bing Web 搜尋 API 並處理回應所需的邏輯。

## <a name="html-form"></a>HTML 表單

`index.html` 包含一個表單，讓使用者能夠用來搜尋並選取搜尋選項。 `onsubmit` 屬性會在提交表單時引發，以呼叫 `scripts.js` 中所定義的 `bingWebSearch()` 方法。 它會採用三個引數︰

* 搜尋查詢
* 選取的選項
* 訂用帳戶金鑰

```html
<form name="bing" onsubmit="return bingWebSearch(this.query.value,
    bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="query-options"></a>查詢選項

HTML 表單包含對應到 [Bing Web 搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#query-parameters) \(英文\) 中查詢參數的選項。 下表提供使用者如何使用範例應用程式來篩選搜尋結果的明細：

| 參數 | 說明 |
|-----------|-------------|
| `query` | 可輸入查詢字串的文字欄位。 |
| `where` | 可選取市場 (位置和語言) 的下拉式選單。 |
| `what` | 要將特定結果類型升階的核取方塊。 例如，將影像升階會在搜尋結果中提高影像的排名。 |
| `when` | 可讓使用者將搜尋結果限制為今天、本週或本月的下拉式選單。 |
| `safe` | 可啟用 Bing SafeSearch 的核取方塊，其會篩選出成人內容。 |
| `count` | 隱藏的欄位。 毎個要求要傳回的搜尋結果數目。 變更此值，以便為每頁顯示更多或更少結果。 |
| `offset` | 隱藏的欄位。 要求中第一個搜尋結果的位移，其可用於分頁。 它會隨著每個新要求重設為 `0`。 |

> [!NOTE]
> Bing Web 搜尋 API 會提供其他查詢參數來協助精簡搜尋結果。 此範例只使用數個參數。 如需可用參數的完整清單，請參閱 [Bing Web 搜尋 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#query-parameters) \(英文\)。

`bingSearchOptions()` 函式會轉換這些選項，以符合 Bing 搜尋 API 所需的格式。

```javascript
// Build query options from selections in the HTML form.
function bingSearchOptions(form) {

    var options = [];
    // Where option.
    options.push("mkt=" + form.where.value);
    // SafeSearch option.
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    // Freshness option.
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++)
        if (form.what[i].checked) what.push(form.what[i].value);
    // Promote option.
    if (what.length) {
        options.push("promote=" + what.join(","));
        options.push("answerCount=9");
    }
    // Count option.
    options.push("count=" + form.count.value);
    // Offset option.
    options.push("offset=" + form.offset.value);
    // Hardcoded text decoration option.
    options.push("textDecorations=true");
    // Hardcoded text format option.
    options.push("textFormat=HTML");
    return options.join("&");
}
```

您可以使用 `moderate` (此為 Bing Web 搜尋的預設設定)，將 `SafeSearch` 設定為 `strict`、`moderate` 或 `off`。 此表單使用具備兩種狀態的核取方塊。 在此程式碼片段中，會將 SafeSearch 設定為 `strict` 或 `off`，但不使用 `moderate`。

如果選取了任何 [升階] 核取方塊，即會將 `answerCount` 參數加入至查詢。 如果使用 `promote` 參數，則必須使用 `answerCount`。 在此程式碼片段中，會將值設定為 `9`，以傳回所有可用的結果類型。
> [!NOTE]
> 將結果類型升階，不「保證」它將包含於搜尋結果中。 實際情況是，升階會提高此類結果相對於其正常狀態下的排名。 若要將搜尋限定於特定類型的結果，請使用 `responseFilter` 查詢參數，或呼叫更明確的端點，例如 Bing 影像搜尋或 Bing 新聞搜尋。

`textDecoration` 和 `textFormat` 查詢參數都會硬式編碼於指令碼中，並使搜尋字詞在搜尋結果中以粗體顯示。 這些都不是必要參數。

## <a name="manage-subscription-keys"></a>管理訂用帳戶金鑰

為了避免對 Bing 搜尋 API 訂用帳戶金鑰進行硬式編碼，此範例應用程式會使用瀏覽器的永續性儲存體來儲存訂用帳戶金鑰。 如果未儲存任何訂用帳戶金鑰，系統會提示使用者輸入一個。 如果 API 拒絕該訂用帳戶金鑰，系統會提示使用者重新輸入訂用帳戶金鑰。

`getSubscriptionKey()` 函式會使用 `storeValue` 和 `retrieveValue` 函式，來儲存及擷取使用者的訂用帳戶金鑰。 這些函式會使用 `localStorage` 物件 (如果支援) 或 Cookie。

```javascript
// Cookie names for stored data.
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/search";

// See source code for storeValue and retrieveValue definitions.

// Get stored subscription key, or prompt if it isn't found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // Always set the cookie in order to update the expiration date.
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```

如我們先前所見，提交表單時，`onsubmit` 會引發並呼叫 `bingWebSearch`。 此函式會初始化並傳送要求。 `getSubscriptionKey` 會在每個提交上呼叫來驗證要求。

## <a name="call-bing-web-search"></a>呼叫 Bing Web 搜尋

假設有查詢、options 字串和 API 金鑰，`BingWebSearch` 函式就會建立 `XMLHttpRequest` 物件來呼叫 Bing Web 搜尋端點。

```javascript
// Perform a search constructed from the query, options, and subscription key.
function bingWebSearch(query, options, key) {
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // Initialize the request.
    try {
        request.open("GET", queryurl);
    }
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // Add request headers.
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // Event handler for successful response.
    request.addEventListener("load", handleBingResponse);

    // Event handler for errors.
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // Event handler for an aborted request.
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // Send the request.
    request.send();
    return false;
}
```

當要求成功之後，`load` 事件處理常式就會觸發並呼叫 `handleBingResponse` 函式。 `handleBingResponse` 會剖析結果物件、顯示結果，並包含失敗要求的錯誤邏輯。

```javascript
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // Try to parse results object.
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
        return;
    }

    // Show raw JSON and the HTTP request.
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // If the HTTP response is 200 OK, try to render the results.
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "SearchResponse" && "rankingResponse" in jsobj) {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is considered an error.
    else {
        // 401 is unauthorized; force a re-prompt for the user's subscription
        // key on the next request.
        if (this.status === 401) invalidateSubscriptionKey();

        // Some error responses don't have a top-level errors object, if absent
        // create one.
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // Display the HTTP status code.
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // Add all fields from all error responses.
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // Display Bing Trace ID if it isn't blocked by CORS.
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // Display the error message.
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> 成功的 HTTP 要求「不」代表搜尋本身成功。 如果搜尋作業中發生錯誤，Bing Web 搜尋 API 會傳回非 200 HTTP 狀態碼，並在 JSON 回應中包含錯誤資訊。 若要求速率受到限制，API 會傳回空白回應。

上述兩個函式中的大部分程式碼都是專用於錯誤處理。 下列階段可能會發生錯誤：

| 階段 | 可能的錯誤 | 處理者 |
|-------|--------------------|------------|
| 建置要求物件 | 無效的 URL | `try` / `catch` 區塊 |
| 提出要求 | 網路錯誤、已中止連線 | `error` 和 `abort` 事件處理常式 |
| 執行搜尋 | 無效的要求、無效的 JSON、速率限制 | `load` 事件處理常式中的測試 |

錯誤會藉由呼叫 `renderErrorMessage()` 來處理。 如果回應通過所有錯誤測試，即會呼叫 `renderSearchResults()` 來顯示搜尋結果。

## <a name="display-search-results"></a>顯示搜尋結果

對於 Bing Web 搜尋 API 所傳回的結果有[使用和顯示需求](useanddisplayrequirements.md)。 由於回應可能包含各種結果類型，因而不足以逐一查看最上層 `WebPages`集合。 範例應用程式會改為使用 `RankingResponse`，來將結果排序為規格。

> [!NOTE]
> 如果您只想要單一結果類型，請使用 `responseFilter` 查詢參數，或考慮使用其他 Bing 搜尋端點之一，例如 Bing 影像搜尋。

每個回應都有一個 `RankingResponse` 物件，其中最多可能包含三個集合：`pole`、`mainline` 和 `sidebar`。 `pole` (若存在) 是最相關的搜尋結果，必須優先顯示。 `mainline` 包含大多數的搜尋結果，並在 `pole` 之後立即顯示。 `sidebar` 包含輔助搜尋結果。 可能的話，應該將這些結果顯示於資訊看板中。 如果因螢幕限制而無法顯示資訊看板，則這些結果應顯示在 `mainline` 結果後面。

每個 `RankingResponse` 都包含 `RankingItem` 陣列，其會指定必須排序結果的方式。 我們的範例應用程式會使用 `answerType` 和 `resultIndex` 參數來識別結果。

> [!NOTE]
> 還有其他方式可用以識別並排名結果。 如需詳細資訊，請參閱[使用排名顯示結果](rank-results.md)。

讓我們看看此程式碼：

```javascript
// Render the search results from the JSON response.
function renderSearchResults(results) {

    // If spelling was corrected, update the search field.
    if (results.queryContext.alteredQuery)
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // Add Prev / Next links with result count.
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the results for each section.
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

`renderResultsItems()` 函式會依序逐一查看每個 `RankingResponse` 集合中的項目、使用 `answerType` 和 `resultIndex` 值來將每個排名結果對應至搜尋結果，然後呼叫適當的轉譯函式以產生 HTML。 若未針對項目指定 `resultIndex`，則 `renderResultsItems()` 會逐一查看該類型的所有結果，並為每個項目呼叫轉譯函式。 產生的 HTML 會插入到 `index.html` 中適當的 `<div>` 元素。

```javascript
// Render search results from the RankingResponse object per rank response and
// use and display requirements.
function renderResultsItems(section, results) {

    var items = results.rankingResponse[section].items;
    var html = [];
    for (var i = 0; i < items.length; i++) {
        var item = items[i];
        // Collection name has lowercase first letter while answerType has uppercase
        // e.g. `WebPages` RankingResult type is in the `webPages` top-level collection.
        var type = item.answerType[0].toLowerCase() + item.answerType.slice(1);
        if (type in results && type in searchItemRenderers) {
            var render = searchItemRenderers[type];
            // This ranking item refers to ONE result of the specified type.
            if ("resultIndex" in item) {
                html.push(render(results[type].value[item.resultIndex], section));
            // This ranking item refers to ALL results of the specified type.
            } else {
                var len = results[type].value.length;
                for (var j = 0; j < len; j++) {
                    html.push(render(results[type].value[j], section, j, len));
                }
            }
        }
    }
    return html.join("\n\n");
}
```

## <a name="review-renderer-functions"></a>檢視轉譯器函式

在我們的範例應用程式中，`searchItemRenderers` 物件所包含的函式會針對每種類型的搜尋結果產生 HTML。

```javascript
// Render functions for each result type.
searchItemRenderers = {
    webPages: function(item) { ... },
    news: function(item) { ... },
    images: function(item, section, index, count) { ... },
    videos: function(item, section, index, count) { ... },
    relatedSearches: function(item, section, index, count) { ... }
}
```

> [!IMPORTANT]
> 此範例應用程式具有適用於網頁、新聞、影像、視訊和相關搜尋的轉譯器。 您的應用程式將需要轉譯器以用於可能接收到的各種類型結果，其中可能包括計算、拼字建議、實體、時區和定義。

部分轉譯函式只接受 `item` 參數。 其餘函式會接受其他參數，可用來根據內容以不同方式轉譯項目。 未使用此資訊的轉譯器不需要接受這些參數。

內容引數包括：

| 參數  | 說明 |
|------------|-------------|
| `section` | 顯示項目的結果區段 (`pole`、`mainline` 或 `sidebar`)。 |
| `index`<br>`count` | 如果 `RankingResponse` 項目指定要顯示給定集合中的所有結果，則可供使用，否則會使用 `undefined`。 其集合內的項目索引，以及該集合中的項目總數。 您可以使用此資訊來為結果編號，以便為第一個或最後一個結果 (或其他結果) 產生不同的 HTML。 |

在此範例應用程式中，`images` 和 `relatedSearches` 轉譯器都會使用內容引數來自訂產生的 HTML。 以下我們將深入觀察 `images` 轉譯器：

```javascript
searchItemRenderers = {
    // Render image result with thumbnail.
    images: function(item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];
        if (section === "sidebar") {
            if (index) html.push("<br>");
        } else {
            if (!index) html.push("<p class='images'>");
        }
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        return html.join("");
    },
    // Other renderers are omitted from this sample...
}
```

影像轉譯器：

* 計算影像縮圖大小 (寬度會變動，高度則固定為 60 像素)。
* 根據內容，在影像結果前面插入 HTML。
* 建置 HTML `<a>` 標籤，以連結至內含影像的頁面。
* 建置 HTML `<img>` 標籤，以顯示影像縮圖。

影像轉譯器會使用 `section` 和 `index` 變數，根據結果的顯示位置以不同方式顯示結果。 資訊看板中的影像結果之間會插入分行符號 (`<br>` 標記)，讓資訊看板顯示影像的資料行。 在其他區段中，第一個影像結果 `(index === 0)`前面會有 `<p>` 標記。

縮圖大小用於這兩個 `<img>` 標籤，以及縮圖 URL 中的 `h` 和 `w` 欄位。 `title` 和 `alt` 屬性 (影像的文字描述) 會從影像的名稱和 URL 中的主機名稱建構。

以下是如何在範例應用程式中顯示影像的範例：

![[Bing 影像結果]](media/cognitive-services-bing-web-api/web-search-spa-images.png)

## <a name="persist-the-client-id"></a>保存用戶端識別碼

來自 Bing 搜尋 API 的回應可能包含應在每個後續要求中傳回至 API 的 `X-MSEdge-ClientID` 標頭。 如果您的應用程式使用一個以上的 Bing 搜尋 API，請確定會在服務之間隨著每個要求傳送相同的用戶端識別碼。

提供 `X-MSEdge-ClientID` 標頭可讓 Bing API 關聯到使用者的搜尋。 首先，它可讓 Bing 搜尋引擎將過去的內容套用至搜尋，以尋找更能滿足要求的結果。 例如，若使用者之前搜尋與航行相關的字詞，稍後搜尋「節」可能會優先傳回航行中所使用節數的相關資訊。 其次，Bing 可能會隨機選取使用者來體驗新功能，再廣泛提供這些功能。 隨著每個要求提供相同的用戶端識別碼，可確保已選擇看到功能的使用者將一律看到該功能。 若沒有用戶端識別碼，使用者可能會在其搜尋結果中隨機看到功能出現並消失。

瀏覽器安全性原則 (例如，跨原始來源資源共用 (CORS)) 可能會防止範例應用程式存取 `X-MSEdge-ClientID` 標頭。 當搜尋回應的來源與要求回應的頁面不同時，就會發生此限制。 在生產環境中，您應該裝載伺服器端指令碼，在與網頁相同的網域上執行 API 呼叫，以處理此原則。 由於指令碼的來源與網頁相同，因此 `X-MSEdge-ClientID` 標頭會接著提供給 JavaScript 使用。

> [!NOTE]
> 在生產 Web 應用程式中，無論如何都應該執行要求伺服器端。 否則，您的 Bing 搜尋 API 訂用帳戶金鑰必須包含在網頁中，以提供給檢視來源的任何人。 您會根據 API 訂用帳戶金鑰的所有使用量付費，即使是未經授權的合作對象所提出的要求，因此請務必不要公開您的金鑰。

您可以基於開發目的，透過 CORS Proxy 提出要求。 來自這個類型 Proxy 的回應包含 `Access-Control-Expose-Headers` 標頭，可將回應標頭列入白名單並提供給 JavaScript 使用。

您可以輕鬆安裝 CORS Proxy，讓我們的範例應用程式能夠存取用戶端識別碼標頭。 請執行這個命令：

```console
npm install -g cors-proxy-server
```

接下來，將 `script.js` 中的 Bing Web 搜尋端點變更為：

```javascript
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search
```

使用此命令來啟動 CORS Proxy：

```console
cors-proxy-server
```

當您使用範例應用程式時，請將命令視窗保持開啟；關閉該視窗即會停止 Proxy。 在搜尋結果下方可展開的 [HTTP 標頭] 區段中，應該可以看見 `X-MSEdge-ClientID` 標頭。 請確認它在每個要求中都一樣。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing Web 搜尋 API v7參考](//docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)
