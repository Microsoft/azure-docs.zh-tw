---
title: 教學課程：Bing 實體搜尋單頁 Web 應用程式
titlesuffix: Azure Cognitive Services
description: 示範如何在單頁 Web 應用程式中使用 Bing 實體搜尋 API。
services: cognitive-services
author: v-jerkin
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: tutorial
ms.date: 12/08/2017
ms.author: v-jerkin
ms.openlocfilehash: 9aabecbec144797b9fbafdff7179213b68921447
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48815540"
---
# <a name="tutorial-single-page-web-app"></a>教學課程：單頁 Web 應用程式

Bing 實體搜尋 API 可讓您搜尋網路上的「實體」和「地點」相關資訊。 您可以在指定查詢中要求任何一種結果，或兩者都要求。 以下提供地點和實體的定義。

|||
|-|-|
|實體|您可以依名稱尋找的已知人物、地點和事項|
|地點|您可以依名稱「或」類型 (義大利餐廳) 尋找的餐廳、旅館及其他本地商家|

在本教學課程中，我們會建置單頁 Web 應用程式，以使用 Bing 實體搜尋 API 直接在頁面中顯示搜尋結果。 該應用程式包含 HTML、CSS 和 JavaScript 元件。

此 API 可讓您依位置排列結果優先順序。 在行動應用程式中，您可以要求裝置提供其自身位置。 在 Web 應用程式中，您可以使用 `getPosition()` 函式。 但此呼叫僅適用於安全內容，而且可能不會提供精確位置。 此外，使用者可能想要搜尋其本身以外位置附近的實體。

我們的應用程式會為此呼叫 Bing 地圖服務，以從使用者輸入的位置取得緯度和經度。 使用者可以接著輸入地標名稱 (「太空針塔」) 或是完整或部分地址 (「紐約市」)，Bing 地圖服務 API 會提供座標。

<!-- Remove until we can replace with a sanitized version.
![[Single-page Bing Entity Search app]](media/entity-search-spa-demo.png)
-->

> [!NOTE]
> 按下時，頁面底部的 JSON 和 HTTP 標題會顯示 JSON 回應和 HTTP 要求資訊。 這些詳細資料有助於探索服務。

本教學課程應用程式說明如何：

> [!div class="checklist"]
> * 在 JavaScript 中執行 Bing 實體搜尋 API 呼叫
> * 在 JavaScript 中執行 Bing 地圖服務 `locationQuery` API 呼叫
> * 將搜尋選項傳遞至 API 呼叫
> * 顯示搜尋結果
> * 處理 Bing 用戶端識別碼和 API 訂用帳戶金鑰
> * 解決可能出現的任何錯誤

本教學課程頁完全獨立，不會使用任何外部架構、樣式表或甚至影像檔。 其中僅使用廣受支援的 JavaScript 語言功能，並且與所有主要網頁瀏覽器的目前版本搭配使用。

在本教學課程中，我們只會討論原始程式碼的特定部分。 完整的原始程式碼位於[另一個頁面上](tutorial-bing-entities-search-single-page-app-source.md)。 請將此程式碼複製並貼到文字編輯器中，然後另存為 `bing.html`。

> [!NOTE]
> 本教學課程與[單頁 Bing Web 搜尋應用程式教學課程](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md)非常類似，但只會處理實體搜尋結果。

## <a name="app-components"></a>應用程式元件

如同任何單頁 Web 應用程式，本教學課程應用程式包含三個部分：

> [!div class="checklist"]
> * HTML - 定義頁面的結構和內容
> * CSS - 定義頁面的外觀
> * JavaScript - 定義頁面的行為

本教學課程並未詳細說明大部分的 HTML 或 CSS，因為這些元件相當簡單。

HTML 包含搜尋表單，使用者可在其中輸入查詢並選擇搜尋選項。 該表單會透過 `<form>` 標籤的 `onsubmit` 屬性，連線至實際執行搜尋的 JavaScript：

```html
<form name="bing" onsubmit="return newBingEntitySearch(this)">
```

`onsubmit` 處理常式會傳回 `false`，這可防止表單提交至伺服器。 JavaScript 程式碼實際上會從表單收集所需的資訊，並執行搜尋。

搜尋會分兩個階段完成。 首先，若使用者已輸入位置限制，則會執行 Bing 地圖服務將它轉換成座標。 此查詢的回呼會接著開始執行 Bing 實體搜尋查詢。

HTML 也包含顯示搜尋結果的區域 (HTML `<div>` 標籤)。

## <a name="managing-subscription-keys"></a>管理訂用帳戶金鑰

> [!NOTE]
> 此應用程式同時需要 Bing 搜尋 API 和 Bing 地圖服務 API 的訂用帳戶金鑰。 您可以使用[試用版的 Bing 搜尋金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)和[基本 Bing 地圖服務金鑰](https://www.microsoft.com/maps/create-a-bing-maps-key)。

為了避免必須在程式碼中包含 Bing 搜尋和 Bing 地圖服務 API 訂用帳戶金鑰，我們使用瀏覽器的永續性儲存體來儲存這些金鑰。 若尚未儲存任何一個金鑰，則會出現提示要求儲存以供稍後使用。 若 API 稍後拒絕金鑰，我們會讓儲存的金鑰失效，以便在使用者下次搜尋時，要求使用者提供金鑰。

我們會定義 `storeValue` 和 `retrieveValue` 函式，以使用 `localStorage` 物件 (若瀏覽器支援的話) 或 Cookie。 我們的 `getSubscriptionKey()` 函式使用這些函式來儲存及擷取使用者的金鑰。

```javascript
// cookie names for data we store
SEARCH_API_KEY_COOKIE = "bing-search-api-key";
MAPS_API_KEY_COOKIE   = "bing-maps-api-key";
CLIENT_ID_COOKIE      = "bing-search-client-id";

// API endpoints
SEARCH_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/entities";
MAPS_ENDPOINT   = "http://dev.virtualearth.net/REST/v1/Locations";

// ... omitted definitions of storeValue() and retrieveValue()

// get stored API subscription key, or prompt if it's not found
function getSubscriptionKey(cookie_name, key_length, api_name) {
    var key = retrieveValue(cookie_name);
    while (key.length !== key_length) {
        key = prompt("Enter " + api_name + " API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(cookie_name, key);
    return key;
}

function getMapsSubscriptionKey() {
    return getSubscriptionKey(MAPS_API_KEY_COOKIE, 64, "Bing Maps");
}

function getSearchSubscriptionKey() {
    return getSubscriptionKey(SEARCH_API_KEY_COOKIE, 32, "Bing Search");
}
```

HTML `<body>` 標籤包含 `onload` 屬性，以在頁面完成載入時呼叫 `getSearchSubscriptionKey()` 和 `getMapsSubscriptionKey()`。 若使用者尚未輸入其金鑰，這些呼叫會立即提示使用者輸入。

```html
<body onload="document.forms.bing.query.focus(); getSearchSubscriptionKey(); getMapsSubscriptionKey();">
```

## <a name="selecting-search-options"></a>選取搜尋選項

![[Bing 實體搜尋表單]](media/entity-search-spa-form.png)

HTML 表單包含下列控制項：

| | |
|-|-|
|`where`|可供選取市場 (位置和語言) 以用於搜尋的下拉式功能表。|
|`query`|要在其中輸入搜尋字詞的文字欄位。|
|`safe`|指出是否開啟安全搜尋的核取方塊 (這會限制「成人」結果)|
|`what`|可供選擇以搜尋實體、地點或兩者的功能表。|
|`mapquery`|使用者可在此文字欄位中輸入完整或部分地址、地標等，以協助 Bing 實體搜尋傳回更相關的結果。|

> [!NOTE]
> 地點結果目前僅適用於美國。 `where` 和 `what` 功能表具有可強制執行這項限制的程式碼。 若您在 `what` 功能表中選取 [地點] 時選擇非美國市場，`what` 會變更為任何項目。 若您在 `where` 功能表中選取非美國市場時選擇 [地點]，`where` 會變更為美國。

我們的 JavaScript 函式 `bingSearchOptions()` 會將這些欄位轉換成 Bing 搜尋 API 的部分查詢字串。

```javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.what.selectedIndex) options.push("responseFilter=" + form.what.value);
    return options.join("&");
}
```

例如，安全搜尋功能可以是 `strict`、`moderate` 或 `off`，預設為 `moderate`。 但我們的表單使用只有兩種狀態的核取方塊。 JavaScript 程式碼會將此設定轉換成 `strict` 或 `off` (我們不使用 `moderate`)。

`bingSearchOptions()` 中不會處理 `mapquery` 欄位，因為它適用於 Bing 地圖服務位置查詢，不適用於 Bing 實體搜尋。

## <a name="obtaining-a-location"></a>取得位置

Bing 地圖服務 API 提供 [`locationQuery` 方法](//msdn.microsoft.com/library/ff701711.aspx)，我們使用此方法來尋找使用者輸入位置的緯度和經度。 這些座標會接著在使用者的要求下，傳遞至 Bing 實體搜尋 API。 搜尋結果會優先使用接近指定位置的實體和地點。

我們無法在 Web 應用程式中使用一般 `XMLHttpRequest` 查詢來存取 Bing 地圖服務 API，因為服務不支援跨原始來源查詢。 幸好，它支援 JSONP (「P」表示「已填補」)。 JSONP 回應是包裝成函式呼叫的一般 JSON 回應。 要求是透過使用 `<script>` 標籤插入文件來提出 (載入指令碼不需遵守瀏覽器安全性原則)。

`bingMapsLocate()` 函式會為查詢建立並插入 `<script>` 標籤。 查詢字串的 `jsonp=bingMapsCallback` 區段指定要透過回應呼叫的函式名稱。

```javascript
function bingMapsLocate(where) {

    where = where.trim();
    var url = MAPS_ENDPOINT + "?q=" + encodeURIComponent(where) + 
                "&jsonp=bingMapsCallback&maxResults=1&key=" + getMapsSubscriptionKey();

    var script = document.getElementById("bingMapsResult")
    if (script) script.parentElement.removeChild(script);

    // global variable holds reference to timer that will complete the search if the maps query fails
    timer = setTimeout(function() {
        timer = null;
        var form = document.forms.bing;
        bingEntitySearch(form.query.value, "", bingSearchOptions(form), getSearchSubscriptionKey());
    }, 5000);

    script = document.createElement("script");
    script.setAttribute("type", "text/javascript");
    script.setAttribute("id", "bingMapsResult");
    script.setAttribute("src", url);
    script.setAttribute("onerror", "BingMapsCallback(null)");
    document.body.appendChild(script);

    return false;
}
```

> [!NOTE]
> 若 Bing 地圖服務 API 沒有回應，則絕不會呼叫 `bingMapsCallBack()` 函式。 通常，這表示不會呼叫 `bingEntitySearch()`，而且實體搜尋結果不會顯示。 若要避免這種情況發生，`bingMapsLocate()` 也會設定計時器在五秒後呼叫 `bingEntitySearch()`。 回呼函數中有一個邏輯可避免執行實體搜尋兩次。

當查詢完成時，會依照要求呼叫 `bingMapsCallback()` 函式。

```javascript
function bingMapsCallback(response) {

    if (timer) {    // we beat the timer; stop it from firing
        clearTimeout(timer);
        timer = null;
    } else {        // the timer beat us; don't do anything
        return; 
    }

    var location = "";
    var name = "";
    var radius = 1000;

    if (response) {
        try {
            if (response.statusCode === 401) {
                invalidateMapsKey();
            } else if (response.statusCode === 200) {
                var resource = response.resourceSets[0].resources[0];
                var coords   = resource.point.coordinates;
                name         = resource.name;

                // the radius is the largest of the distances between the location and the corners
                // of its bounding box (in case it's not in the center) with a minimum of 1 km
                try {
                    var bbox    = resource.bbox;
                    radius  = Math.max(haversineDistance(bbox[0], bbox[1], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[1]),
                                       haversineDistance(bbox[0], bbox[3], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[3]), 1000);
                } catch(e) {  }
                var location = "lat:" + coords[0] + ";long:" + coords[1] + ";re:" + Math.round(radius);
            }
        }
        catch (e) { }   // response is unexpected. this isn't fatal, so just don't provide location
    }

    var form = document.forms.bing;
    if (name) form.mapquery.value = name;
    bingEntitySearch(form.query.value, location, bingSearchOptions(form), getSearchSubscriptionKey());

}
```

除了緯度和經度，Bing 實體搜尋查詢還需要「半徑範圍」，以指出位置資訊的精確度。 我們使用 Bing 地圖服務回應中提供的「週框方塊」來計算半徑範圍。 週框方塊是圍繞整個位置的矩形。 例如，若使用者輸入 `NYC`，結果會大致包含紐約市的中心座標及圍繞城市的週框方塊。 

我們會先使用函式 `haversineDistance()` (未顯示)，計算從主要座標到週框方塊四個邊角的距離。 我們使用四個距離中的最大值作為半徑範圍。 最小半徑範圍為一公里。 若回應中未提供週框方塊，此值也會作為預設值。

取得座標和半徑範圍之後，我們會接著呼叫 `bingEntitySearch()` 以執行實際搜尋。

## <a name="performing-the-search"></a>執行搜尋

指定查詢、位置、選項字串和 API 金鑰之後，`BingEntitySearch()` 函式會提出 Bing 實體搜尋要求。

```javascript
// perform a search given query, location, options string, and API keys
function bingEntitySearch(query, latlong, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "error");

    var request = new XMLHttpRequest();
    var queryurl = SEARCH_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

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

    if (latlong) request.setRequestHeader("X-Search-Location", latlong);

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

HTTP 要求成功完成時，JavaScript 會呼叫我們的 `load` 事件處理常式 `handleBingResponse()` 函式，來處理成功傳遞至 API 的 HTTP GET 要求。 

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
            if (jsobj._type === "SearchResponse") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    if (divHidden("pole") && divHidden("mainline") && divHidden("sidebar")) 
        showDiv("noresults", "No results.<p><small>Looking for restaurants or other local businesses? Those currently areen't supported outside the US.</small>");
    }

    // Any other HTTP status is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSearchKey();

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
> 成功的 HTTP 要求「不」一定表示搜尋本身成功。 若搜尋作業中發生錯誤，Bing 實體搜尋 API 會傳回非 200 HTTP 狀態碼，並在 JSON 回應中包含錯誤資訊。 此外，若要求速率受到限制，API 會傳回空白回應。

上述兩個函式中大部分的程式碼都是專門用來處理錯誤的。 下列階段可能會發生錯誤：

|階段|可能的錯誤|處理者|
|-|-|-|
|建置 JavaScript 要求物件|無效的 URL|`try`/`catch` 區塊|
|提出要求|網路錯誤、已中止連線|`error` 和 `abort` 事件處理常式|
|執行搜尋|無效的要求、無效的 JSON、速率限制|`load` 事件處理常式中的測試|

錯誤是透過呼叫 `renderErrorMessage()` 來處理，並提供有關錯誤的任何已知詳細資料。 若回應通過錯誤測試的所有挑戰，我們會呼叫 `renderSearchResults()` 在頁面中顯示搜尋結果。

## <a name="displaying-search-results"></a>顯示搜尋結果

Bing 實體搜尋 API [要求您依指定順序顯示結果](use-display-requirements.md)。 由於 API 可能傳回兩種不同的回應，因此逐一查看 JSON 回應中的最上層 `Entities` 或 `Places` 集合及顯示這些結果並不夠。 (若您只需要一種結果，請使用 `responseFilter` 查詢參數)。

相反地，我們會在搜尋結果中使用 `rankingResponse` 集合，以排序要顯示的結果。 此物件是指 `Entitiess` 及/或 `Places` 集合中的項目。

`rankingResponse` 最多可能包含三個搜尋結果集合，並已指定 `pole`、`mainline` 和 `sidebar`。 

`pole` (若存在) 是最相關的搜尋結果，應該優先顯示。 `mainline` 會參照大量搜尋結果。 Mainline 結果應該在 `pole` 之後立即顯示 (若 `pole` 不存在，則優先顯示)。 

最後， `sidebar` 是指輔助搜尋結果。 它們可能會在實際 sidebar 中顯示，或直接在 mainline 結果之後顯示。 我們已為教學課程應用程式選擇後者。

`rankingResponse` 集合中的每個項目是指兩個不同但方式相同的實際搜尋結果項目。

| | |
|-|-|
|`id`|`id` 看起來像是 URL，但不應該用於連結。 排名結果的 `id` 類型符合回應集合中任一搜尋結果項目的 `id`，「或」整個回應集合 (例如`Entities`)。
|`answerType`<br>`resultIndex`|`answerType` 會參照包含結果的最上層答案集合 (例如 `Entities`)。 `resultIndex` 會參照該集合內的結果索引。 若省略 `resultIndex`，排名結果是指整個集合。

> [!NOTE]
> 如需這部分搜尋回應的詳細資訊，請參閱[排名結果](rank-results.md)。

您可以使用對您的應用程式而言最方便的任何方法，來尋找參考的搜尋結果項目。 在我們的教學課程程式碼中，我們使用 `answerType` 和 `resultIndex` 來尋找每個搜尋結果。

最後，讓我們來看看函式 `renderSearchResults()`。 此函式會逐一查看三個 `rankingResponse` 集合，這分別代表搜尋結果的三個區段。 針對每個區段，我們會呼叫 `renderResultsItems()` 以轉譯該區段的結果。

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // if spelling was corrected, update search field
    if (results.queryContext.alteredQuery) 
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // for each possible section, render the resuts from that section
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

## <a name="rendering-result-items"></a>轉譯結果項目

在我們的 JavaScript 程式碼中有一個物件 `searchItemRenderers`，其中包含 *renderers:* 函式，可為每種搜尋結果產生 HTML。

```javascript
searchItemRenderers = { 
    entities: function(item) { ... },
    places: function(item) { ... }
}
```

轉譯器函式可接受下列參數：

| | |
|-|-|
|`item`|JavaScript 物件，其中包含項目的屬性，例如其 URL 及其描述。|
|`index`|集合內結果項目的索引。|
|`count`|搜尋結果項目集合中的項目數目。|

`index` 和 `count` 參數可用來編號結果、為集合開頭或結尾產生特殊 HTML、在特定數量的項目之後插入分行符號等。 若轉譯器不需要此功能，則不需要接受這兩個參數。 事實上，我們不會將其用於教學課程應用程式的轉譯器中。

讓我們仔細看看 `entities` 轉譯器：

```javascript
    entities: function(item) {
        var html = [];
        html.push("<p class='entity'>");
        if (item.image) {
            var img = item.image;
            if (img.hostPageUrl) html.push("<a href='" + img.hostPageUrl + "'>");
            html.push("<img src='" + img.thumbnailUrl +  "' title='" + img.name + "' height=" + img.height + " width= " + img.width + ">");
            if (img.hostPageUrl) html.push("</a>");
            if (img.provider) {
                var provider = img.provider[0];
                html.push("<small>Image from ");
                if (provider.url) html.push("<a href='" + provider.url + "'>");
                html.push(provider.name ? provider.name : getHost(provider.url));
                if (provider.url) html.push("</a>");
                html.push("</small>");
            }
        }
        html.push("<p>");
        if (item.entityPresentationInfo) {
            var pi = item.entityPresentationInfo;
            if (pi.entityTypeHints || pi.entityTypeDisplayHint) {
                html.push("<i>");
                if (pi.entityTypeDisplayHint) html.push(pi.entityTypeDisplayHint);
                else if (pi.entityTypeHints) html.push(pi.entityTypeHints.join("/"));
                html.push("</i> - ");
            }
        }
        html.push(item.description);
        if (item.webSearchUrl) html.push("&nbsp;<a href='" + item.webSearchUrl + "'>More</a>")
        if (item.contractualRules) {
            html.push("<p><small>");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++) {
                var rule = item.contractualRules[i];
                var link = [];
                if (rule.license) rule = rule.license;
                if (rule.url) link.push("<a href='" + rule.url + "'>");
                link.push(rule.name || rule.text || rule.targetPropertyName + " source");
                if (rule.url) link.push("</a>");
                rules.push(link.join(""));
            }
            html.push("License: " + rules.join(" - "));
            html.push("</small>");
        }
        return html.join("");
    }, // places renderer omitted
```

我們的實體轉譯器函式會：

> [!div class="checklist"]
> * 建置 HTML `<img>` 標籤，以顯示影像縮圖 (若有的話)。 
> * 建置 HTML `<a>` 標籤，以連結至內含影像的頁面。
> * 建置描述，以顯示影像及執行所在網站的相關資訊。
> * 使用顯示提示 (若有的話) 納入實體的分類。
> * 包含 Bing 搜尋的連結，以取得有關實體的詳細資訊。
> * 顯示資料來源所需的任何授權或屬性資訊。

## <a name="persisting-client-id"></a>保存用戶端識別碼

來自 Bing 搜尋 API 的回應可能會包含應在後續要求中傳回至 API 的 `X-MSEdge-ClientID` 標頭。 若使用多個 Bing 搜尋 API，請盡可能對所有 API 使用相同的用戶端識別碼。

提供 `X-MSEdge-ClientID` 標頭可讓 Bing API 建立所有使用者搜尋的關聯，這有兩個重要的優點。

首先，它可讓 Bing 搜尋引擎將過去內容套用至搜尋結果，以尋找更符合使用者的結果。 例如，若使用者之前搜尋與航行相關的字詞，稍後搜尋「碼頭」可能會優先傳回帆船停駐地點的相關資訊。

其次，Bing 可能會隨機選取使用者來體驗新功能，再廣泛提供這些功能。 在每個要求中提供相同的用戶端識別碼，可確保已選擇看到功能的使用者一律會看到功能。 若沒有用戶端識別碼，使用者可能會在其搜尋結果中隨機看到功能出現並消失。

瀏覽器安全性原則 (CORS) 可防止將 `X-MSEdge-ClientID` 標頭提供給 JavaScript 使用。 當搜尋回應的來源與要求回應的頁面不同時，就會發生這項限制。 在生產環境中，您應該裝載伺服器端指令碼，在與網頁相同的網域上執行 API 呼叫，以處理此原則。 由於指令碼的來源與網頁相同，因此 `X-MSEdge-ClientID` 標頭會接著提供給 JavaScript 使用。

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
> [Bing 實體搜尋 API 參考](//docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference)

> [!div class="nextstepaction"]
> [Bing 地圖服務 API 文件](//msdn.microsoft.com/library/dd877180.aspx)
