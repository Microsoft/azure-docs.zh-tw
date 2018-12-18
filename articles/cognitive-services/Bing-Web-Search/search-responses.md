---
title: Bing Web 搜尋 API 回應 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 了解 Bing Web 搜尋 API 所提供的回答類型和回應。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 8/13/2018
ms.author: erhopf
ms.openlocfilehash: 13e9792f3d5765047dabb4cdef59e85a47a69aba
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42890067"
---
# <a name="bing-web-search-responses"></a>Bing Web 搜尋回應  

當您將搜尋要求傳送給 Bing Web 搜尋時，它會在回應本文中傳回 [`SearchResponse`](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#searchresponse) 物件。 物件中包含每項回答 (Bing 判斷與查詢有關) 的欄位。 此範例說明如果 Bing 傳回所有回答時的回應物件：

```json
{
    "_type": "SearchResponse",
    "queryContext": {...},
    "webPages": {...},
    "images": {...},
    "relatedSearches": {...},
    "videos": {...},
    "news": {...},
    "spellSuggestion": {...},
    "computation": {...},
    "timeZone": {...},
    "rankingResponse": {...}
}, ...
```

一般而言，Bing Web 搜尋會傳回回答的子集。 例如，如果查詢字詞是 sailing dinghies，則回應可能會包含 `webPages`、`images` 和 `rankingResponse`。 除非您已使用 [responseFilter](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#responsefilter) 來篩選網頁，否則回應永遠會包含 `webpages` 和 `rankingResponse` 回答。

## <a name="webpages-answer"></a>網頁回答

[網頁](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#webanswer) (英文) 回答包含網頁 (Bing Web 搜尋判斷與查詢有關) 的連結清單。 清單中每個[網頁](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#webpage) (英文) 都包含頁面的名稱、URL、顯示 URL、內容的簡短描述以及 Bing 找到內容時的日期。

```json
{
    "id": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
    "name": "Dinghy sailing",
    "url": "https:\/\/www.bing.com\/cr?IG=3A43CA5...",
    "displayUrl": "https:\/\/en.contoso.com\/wiki\/Dinghy_sailing",
    "snippet": "Dinghy sailing is the activity of sailing small boats...",
    "dateLastCrawled": "2017-04-05T16:25:00"
}, ...
```

使用 `name` 和 `url` 來建立帶領使用者前往網頁的超連結。

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display the webpage in a search results page.

![Rendered webpage example](./media/cognitive-services-bing-web-api/bing-rendered-webpage-example.PNG)
-->

## <a name="images-answer"></a>影像回應

[影像](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images) (英文) 回答包含 Bing 認為與查詢相關的影像清單。 清單中的每個[影像](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#image) (英文) 都包含影像的 URL、大小、維度及其編碼格式。 Image 物件也包含影像縮圖的 URL 和縮圖的維度。

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=3A43CA5CA64...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/upload.contoso.com\/sailing\/...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=3A43CA5CA6464....",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "http:\/\/en.contoso.com\/wiki\/File...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    }
}, ...
```

根據使用者的裝置而定，您通常會顯示縮圖子集，以及可讓使用者檢視其餘影像的選項。

<!-- Remove until this can be replaced with a sanitized version.
![List of thumbnail images](./media/cognitive-services-bing-web-api/bing-web-image-thumbnails.PNG)
-->

當使用者將游標暫留在縮圖上時，您也可以展開縮圖。 如果要展開影像，請務必標註影像的屬性。 例如，藉由從 `hostPageDisplayUrl` 擷取主機，並將它顯示在影像下方。 如需調整縮圖大小的詳細資訊，請參閱[調整縮圖大小及裁切](./resize-and-crop-thumbnails.md)。

<!-- Remove until this can be replaced with a sanitized version.
![Expanded view of thumbnail image](./media/cognitive-services-bing-web-api/bing-web-image-thumbnail-expansion.PNG)
-->

如果使用者按一下縮圖，則使用 `webSearchUrl` 以將使用者帶到 Bing 對於影像的搜尋結果頁面，其中包含影像的拼貼。

如需影像回答和影像的詳細資訊，請參閱[影像搜尋 API](../bing-image-search/search-the-web.md)。

## <a name="related-searches-answer"></a>相關搜尋回答

[relatedSearches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#searchresponse-relatedsearches) (英文) 回答中包含其他使用者進行的最熱門相關查詢清單。 清單中每個[查詢](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#query_obj) (英文) 都包含查詢字串 (`text`)、具有搜尋結果醒目提示字元的查詢字串 (`displayText`)，以及前往 Bing 針對該查詢搜尋結果的 URL (`webSearchUrl`)。

```json
{
    "text": "dinghy racing teams",
    "displayText": "dinghy racing teams",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=96C4CF214A0..."
}, ...
```

使用 `displayText` 查詢字串和 `webSearchUrl` URL 建立超連結，將使用者帶到 Bing 針對相關查詢的搜尋結果頁面。 您也可以在您自己的 Web 搜尋 API 查詢中使用 `text` 查詢字串，並且顯示結果。

如需如何在 `displayText` 中處理醒目提示標記的詳細資訊，請參閱[搜尋結果醒目提示](./hit-highlighting.md)。

以下顯示在 Bing.com 中相關查詢使用方式的範例。

![Bing 上的相關搜尋範例](./media/cognitive-services-bing-web-api/bing-web-rendered-relatedsearches.GIF)

## <a name="videos-answer"></a>影片回答

[影片](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) (英文) 回答中包含 Bing 認為與查詢相關的影片清單。 清單中的每段[影片](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video) (英文) 都包含影片的 URL、持續時間、維度及編碼格式。 影片物件也包含了影片縮圖的 URL 和縮圖的維度。

```json
{
    "name": "Sailing dinghy",
    "description": "Northwind Traders is a 12 foot gunter rigged...",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D84...",
    "thumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OVP.wsKiL...",
    "datePublished": "2013-11-06T01:56:28",
    "publisher": [{
        "name": "Fabrikam"
    }],
    "contentUrl": "https:\/\/www.fabrikam.com\/watch?v=MrVBWZpJjX",
    "hostPageUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D8400DB...",
    "encodingFormat": "mp4",
    "hostPageDisplayUrl": "https:\/\/www.fabrikam.com\/watch?v=MrBWZpJjXo",
    "width": 1280,
    "height": 720,
    "duration": "PT3M47S",
    "motionThumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OM.oa...",
    "embedHtml": "<iframe width=\"1280\" height=\"720\" src=\"http:\/\/www....><\/iframe>",
    "allowHttpsEmbed": true,
    "viewCount": 19089,
    "thumbnail": {
        "width": 300,
        "height": 168
    },
    "allowMobileEmbed": true,
    "isSuperfresh": false
}, ...
```

根據使用者的裝置而定，您通常會顯示影片子集，以及可讓使用者檢視其餘影片的選項。 您也會顯示影片的縮圖，具有影片長度、描述 (名稱) 以及屬性 (發行者)。

<!-- Remove until this can be replaced with a sanitized version.
![List of video thumbnails](./media/cognitive-services-bing-web-api/bing-web-video-thumbnails.PNG)
-->

當使用者將滑鼠停留在縮圖上時，您可以使用 `motionThumbnailUrl` 來播放影片的縮圖版本。 當您顯示它時，請務必將其歸類為動態縮圖。

<!-- Remove until this can be replaced with a sanitized version.
![Motion thumbnail of a video](./media/cognitive-services-bing-web-api/bing-web-video-motion-thumbnail.PNG)
-->

如果使用者按一下縮圖，以下是影片檢視選項：

- 使用 `hostPageUrl` 可在主機網站 (例如，YouTube) 上檢視影片
- 使用 `webSearchUrl` 可在 Bing 影片瀏覽器中檢視影片
- 使用 `embedHtml` 可將影片內嵌到您自己的體驗中

如需影片回答和影片的詳細資訊，請參閱[影片搜尋 API](../bing-video-search/search-the-web.md)。

## <a name="news-answer"></a>新聞回答

[新聞](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#news) (英文) 回答中包含 Bing 認為與查詢相關的新聞文章清單。 清單中每篇[新聞文章](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#newsarticle) (英文) 都包含文章的名稱、描述以及文章在主機網站上的 URL。 如果文章包含影像，物件就會包含影像的縮圖。

```json
{
    "name": "WC Sailing Qualifies for America Trophy with...",
    "url": "http:\/\/www.bing.com\/cr?IG=3445EEF15DAF4FFFBF7...",
    "image": {
        "contentUrl": "http:\/\/www.contoso.com\/sports\/sail...",
        "thumbnail": {
            "contentUrl": "https:\/\/www.bing.com\/th?id=ON.1...",
            "width": 400,
            "height": 272
        }
    },
    "description": "The WC sailing team qualified for a...",
    "provider": [{
        "_type": "Organization",
        "name": "contoso.com"
    }],
    "datePublished": "2017-04-16T21:56:00"
}, ...
```

根據使用者的裝置而定，您通常會顯示新聞文章子集，以及可讓使用者檢視其餘文章的選項。 使用 `name` 和 `url` 來建立帶領使用者前往主機網站上新聞文章的超連結。 如果文章包含影像，讓影像可以使用 `url` 進行點按。 請務必使用 `provider` 來歸類文章。

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display articles in a search results page.

![List of news articles](./media/cognitive-services-bing-web-api/bing-web-news-list.PNG)
-->

如需新聞回答和新聞文章的詳細資訊，請參閱[新聞搜尋 API](../bing-news-search/search-the-web.md)。

## <a name="computation-answer"></a>計算回答

如果使用者輸入數學運算式或單位轉換查詢，回應可能會包含[計算](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#computation) (英文) 回答。 `computation` 回答包含標準化運算式及其結果。

單位轉換查詢是將一個單位轉換為另一個單位。 例如，「10 公尺是多少英吋？」或「1/4 杯是多少茶匙？」

以下顯示針對「10 公尺是多少英吋？」的 `computation` 回答

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "10 meters",
    "value": "32.808399 feet"
}, ...
```

以下顯示數學查詢及其對應 `computation` 回答的範例。

```
Query: (5+3)(10/2)+8
Encoded query: %285%2B3%29%2810%2F2%29%2B8
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "((5+3)*(10\/2))+8",
    "value": "48"
}
```

```
Query: sqrt(4^2+8^2)
Encoded query: sqrt%284^2%2B8^2%29
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "sqrt((4^2)+(8^2))",
    "value": "8.94427191"
}
```

```
Query: 30 6/8 - 18 8/16
Encoded query: 30%206%2F8%20-%2018%208%2F16
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#WolframAlpha",
    "expression": "30 6\/8-18 8\/16",
    "value": "12.25"
}
```

```
Query: 8^2+11^2-2*8*11*cos(37)
Encoded query: 8^2%2B11^2-2*8*11*cos%2837%29
```

```json
"computation": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
        "expression": "(8^2)+(11^2)-(2*8*11*cos(37))",
        "value": "44.4401502"
}
```

數學運算式可能包含下列符號：

|符號|說明|
|------------|-----------------|
|+|加法|
|-|減法|
|/|除法|
|*|乘法|
|^|乘冪|
|!|階乘|
|.|十進位|
|()|優先順序群組|
|[]|函式|

數學運算式可能包含下列常數：

|符號|說明|
|------------|-----------------|
|Pi|3.14159...|
|角度|角度|
|i|虛數|
|e|e，2.71828...|
|GoldenRatio|黃金比例，1.61803...|

數學運算式可能包含下列函式：

|符號|說明|
|------------|-----------------|
|Sqrt|平方根|
|Sin[x]、Cos[x]、Tan[x]<br />Csc[x]、Sec[x]、Cot[x]|三角函數 (以弧度為單位的引數)|
|ArcSin[x]、ArcCos[x]、ArcTan[x]<br />ArcCsc[x]、ArcSec[x]、ArcCot[x]|反三角函數 (以弧度為單位的給定結果)|
|Exp[x]、E^x|指數函式|
|Log[x]|自然對數|
|Sinh[x]、Cosh[x]、Tanh[x]<br />Csch[x]、Sech[x]、Coth[x]|雙曲函式|
|ArcSinh[x]、ArcCosh[x]、ArcTanh[x]<br />ArcCsch[x]、ArcSech[x]、ArcCoth[x]|反雙曲函式|

不支援包含變數 (例如，4x+6=18，其中 x 是變數) 的數學運算式。

## <a name="timezone-answer"></a>時區回答

如果使用者輸入時間或日期查詢，回應可能會包含[時區](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#timezone) (英文) 回答。 此回答支援隱含或明確查詢。 隱含查詢，例如「現在幾點？」，會根據使用者的位置傳回當地時間。 明確查詢，例如「西雅圖現在幾點？」，會傳回美國華盛頓州西雅圖的當地時間。

`timeZone` 回答可提供位置的名稱、指定位置目前的 UTC 日期和時間，以及 UTC 時差。 如果位置的界限在多個時區內，則回答會包含目前的 UTC 日期和時間以及界限內的所有時區。 例如，因為佛羅里達州位於兩個時區內，回應會包含兩個時區的當地日期和時間。  

如果查詢要求州或國家/地區的時間，Bing 會判斷位於位置地理界限內的主要城市，並且在 `primaryCityTime` 欄位中傳回。 如果界限包含多個時區，其餘時區會在 `otherCityTimes` 欄位中傳回。

以下顯示查詢傳回 `timeZone` 回答的範例。

```
Query: What time is it?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Redmond, Washington, United States",
        "time": "2015-10-27T08:38:12.1189231Z",
        "utcOffset": "UTC-7"
    }
}

Query: What time is it in the Pacific time zone?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Pacific Time Zone",
        "time": "2015-10-23T12:33:19.0728146Z",
        "utcOffset": "UTC-7"
    }
}

Query: Time in Florida?

"timeZone": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
        "primaryCityTime": {
            "location": "Tallahassee, Florida, United States",
            "time": "2015-10-23T13:04:56.6774389Z",
            "utcOffset": "UTC-4"
        },
        "otherCityTimes": [{
            "location": "Pensacola",
            "time": "2015-10-23T12:04:56.6664294Z",
            "utcOffset": "UTC-5"
        }]
}

Query: What time is it in the U.S.

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Washington, D.C., United States",
        "time": "2015-10-23T15:27:59.8892745Z",
        "utcOffset": "UTC-4"
    },
    "otherCityTimes": [{
        "location": "Honolulu",
        "time": "2015-10-23T09:27:59.8892745Z",
        "utcOffset": "UTC-10"
    },
    {
        "location": "Anchorage",
        "time": "2015-10-23T11:27:59.8892745Z",
        "utcOffset": "UTC-8"
    },
    {
        "location": "Phoenix",
        "time": "2015-10-23T12:27:59.8892745Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Los Angeles",
        "time": "2015-10-23T12:27:59.8942788Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Denver",
        "time": "2015-10-23T13:27:59.8812681Z",
        "utcOffset": "UTC-6"
    },
    {
        "location": "Chicago",
        "time": "2015-10-23T14:27:59.8892745Z",
        "utcOffset": "UTC-5"
    }]
}
```

## <a name="spellsuggestion-answer"></a>SpellSuggestion 回答

如果 Bing 判斷使用者可能想要搜尋不同的項目，則回應會包含 [SpellSuggestions](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#spellsuggestions) (英文) 物件。 例如，如果使用者搜尋 carlos pen，Bing 可能會判斷使用者其實是想要搜尋 Carlos Pena (根據過去其他人對於 carlos pen 的搜尋)。 以下顯示範例拼字回應。

```json
"spellSuggestions": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#SpellSuggestions",
    "value": [{
        "text": "carlos pena",
        "displayText": "carlos pena"
    }]
}, ...
```

以下顯示 Bing 如何使用拼字建議。

![Bing 拼字建議範例](./media/cognitive-services-bing-web-api/bing-web-spellingsuggestion.GIF)  

## <a name="next-steps"></a>後續步驟  

* 檢閱[要求節流](throttling-requests.md)文件。  

## <a name="see-also"></a>另請參閱  

* [Bing Web 搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)
