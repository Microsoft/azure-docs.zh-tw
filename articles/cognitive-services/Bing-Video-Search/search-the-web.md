---
title: 什麼是 Bing 影片搜尋？
titlesuffix: Azure Cognitive Services
description: 說明如何使用 Bing 影片搜尋 API 來搜尋網路上的影片。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: overview
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: cf37db9bffa8b2a54a6327c29ec806e0eefc8c91
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225407"
---
# <a name="what-is-bing-video-search"></a>什麼是 Bing 影片搜尋？

Bing 影片搜尋 API 可提供和 [Bing 影片](https://www.bing.com/videos)類似 (但並非完全一樣) 的體驗。 Bing 影片搜尋 API 可讓您將搜尋查詢傳送至 Bing，並取得相關影片的清單。

如果您要建立僅限影片的搜尋結果頁面，以尋找與使用者的搜尋查詢相關的影片，請呼叫此 API，而非呼叫比較一般的 [Bing Web 搜尋 API](../bing-web-search/search-the-web.md)。 如果您想要影片和其他類型的內容 (例如網頁、新聞和影像)，請呼叫 Bing Web 搜尋 API。

## <a name="suggesting--using-search-terms"></a>建議和使用搜尋字詞

若您提供使用者可在其中輸入其搜尋字詞的搜尋方塊，請使用 [Bing 自動建議 API](../bing-autosuggest/get-suggested-search-terms.md) 來改善使用經驗。 API 會根據部分搜尋字詞傳回建議的查詢字串，作為使用者類型。

在使用者輸入其搜尋字詞之後，URL 會先將此字詞編碼，再設定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#query) 查詢參數。 例如，如果使用者輸入 *sailing dinghies*，請將 `q` 設定為 `sailing+dinghies` 或 `sailing%20dinghies`。

## <a name="getting-videos"></a>取得影片

若要從 Web 取得與使用者的搜尋字詞相關聯的影片，請傳送下列 GET 要求：

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

所有要求都必須是從伺服器進行。

如果這是您第一次呼叫任何的 Bing API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API 且 Bing 傳回了使用者和裝置組合的用戶端識別碼，則只要包含用戶端識別碼。

若要從特定網域取得影片，請使用 [site:](http://msdn.microsoft.com/library/ff795613.aspx) 查詢運算子。

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

回應包含一個 [Videos](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) 答案，其中包含 Bing 認為與查詢相關的影片清單。 清單中的每個 [Video](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video) 物件都包含影片的 URL、持續時間、維度、編碼格式和其他屬性。 影片物件也包含了影片縮圖的 URL 和縮圖的維度。

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545...",
    "totalEstimatedMatches" : 1000,
    "value" : [
        {
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear when...",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7...",
            "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.DYW...",
            "datePublished" : "2014-03-04T11:51:53",
            "publisher" : [
                {
                    "name" : "Fabrikam"
                }
            ],
            "creator" : 
            {
                "name" : "Marcus Appel"
            },
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjZ--g",
            "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D569...",
            "encodingFormat" : "h264",
            "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjZ--g",
            "width" : 1280,
            "height" : 720,
            "duration" : "PT2M47S",
            "motionThumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OM.Y62...",
            "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"https:...><\/iframe>",
            "allowHttpsEmbed" : true,
            "viewCount" : 8743,
            "thumbnail" : 
            {
                "width" : 300,
                "height" : 168
            },
            "videoId" : "6DB795E11A6E3CBAAD636DB795E113CBAAD63",
            "allowMobileEmbed" : true,
            "isSuperfresh" : false
        },
        ...
    ],
    "queryExpansions" : [...],
    "nextOffsetAddCount" : 0,
    "pivotSuggestions" : [...]
}
```

您可以顯示所有影片縮圖的拼貼，或顯示縮圖的子集。 如果您顯示子集，請提供使用者檢視其餘影片的選項。 您必須以回應中提供的順序來顯示影片。 如需調整縮圖大小的詳細資訊，請參閱[調整縮圖大小及裁切](./resize-and-crop-thumbnails.md)。 

當使用者將滑鼠停留在縮圖上時，您可以使用 [motionThumbnailUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-motionthumbnailurl) 來播放影片的縮圖版本。 當您顯示它時，請務必將其歸類為動態縮圖。

<!-- Removing until the images can be sanitized.
![Motion thumbnail of a video](../bing-web-search/media/cognitive-services-bing-web-api/bing-web-video-motion-thumbnail.PNG)
-->

如果使用者按一下縮圖，以下是影片檢視選項：

- 使用 [hostPageUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-hostpageurl) 可在主機網站 (例如，YouTube) 上檢視影片
- 使用 [webSearchUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-websearchurl) 可在 Bing 影片瀏覽器中檢視影片
- 使用 [embdedHtml](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-embedhtml) 可將影片內嵌到您自己的體驗中 

在播放影片時，請務必使用發行者和建立者來歸納影片。

如需關於使用 [videoId](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-videoid) 來深入了解影片的詳細資訊，請參閱[影片深入解析](./video-insights.md)。

## <a name="filtering-videos"></a>篩選影片

根據預設，影片搜尋 API 會傳回與查詢相關的所有影片。 如果您只需要長度不超過五分鐘的免費影片，您可以使用下列篩選查詢參數：

- [pricing](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#pricing)&mdash;依定價篩選影片 (例如，免費或必須付費的影片)
- [resolution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#resolution)&mdash;依解析度篩選影片 (例如，720p 或更高解析度的影片)
- [videoLength](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videolength)&mdash;依影片長度篩選影片 (例如，長度少於五分鐘的影片)
- [freshness](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#freshness)&mdash;依存留期篩選影片 (例如，Bing 在過去一週探索的影片)

若要取得特定網域中的影片，請在查詢字串中加入 [site:](http://msdn.microsoft.com/library/ff795613.aspx) 查詢運算子。

> [!NOTE]
> 視查詢而定，如果您使用 `site:` 查詢運算子，不論 [safeSearch](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#safesearch) 設定為何，回應都有可能包含成人內容。 只有在您了解網站上的內容，而且您的案例支援成人內容的可能性時，才得以使用 `site:`。

下列範例說明如何從 ContosoSailing.com 取得解析度不低於 720p、且 Bing 在過去一個月曾探索過的免費影片。

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies+site:contososailing.com&pricing=free&freshness=month&resolution=720p&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

## <a name="expanding-the-query"></a>擴展查詢

如果 Bing 可以擴展查詢來縮小原始搜尋範圍，則 [Videos](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) 物件會包含 `queryExpansions` 欄位。 例如，如果原本的查詢是 *Cleaning Gutters*，擴展的查詢可能是：Gutter Cleaning **Tools**、Cleaning Gutters **From the Ground**、Gutter Cleaning **Machine** 和 **Easy** Gutter Cleaning。

下列範例說明 *Cleaning Gutters* 的擴展查詢。

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC5...",
    "totalEstimatedMatches" : 1000,
    "value" : [...],
    "nextOffsetAddCount" : 4,
    "queryExpansions" : [
        {
            "text" : "Gutter Cleaning Tools",
            "displayText" : "Tools",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FB....",
            "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5...",
            "thumbnail" : {
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?q=Gutter..."
            }
        },
        ...
    ]
    "pivotSuggestions" : [...],
}
```

`queryExpansions` 欄位包含 [Query](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#query_obj) 物件的清單。 `text` 欄位包含擴展的查詢，而 `displayText` 欄位包含擴展字詞。 如果擴展查詢字串就是使用者所要尋找的內容，您可以使用文字和縮圖欄位對使用者顯示擴展查詢字串。 使用 `webSearchUrl` URL 或 `searchLink` URL，讓縮圖和文字可以點選。 使用 `webSearchUrl` 將 Bing 搜尋結果傳送給使用者，而如果您提供自己的結果頁面，則使用 `searchLink`。

## <a name="pivoting-the-query"></a>樞紐分析查詢

如果 Bing 可以分割原始搜尋查詢，則 [Videos](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) 物件會包含 `pivotSuggestions` 欄位。 例如，如果原始查詢為 *Cleaning Gutters*，則 Bing 可能會將查詢分割為 *Cleaning* 和 *Gutters*。

下列範例說明 *Cleaning Gutters* 的樞紐建議。

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC...",
    "totalEstimatedMatches" : 1000,
    "value" : [...],
    "nextOffsetAddCount" : 0,
    "queryExpansions" : [...],
    "pivotSuggestions" : [
        {
            "pivot" : "cleaning",
            "suggestions" : [
                {
                    "text" : "Gutter Repair",
                    "displayText" : "Repair",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5\/videos...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=Gutter..."
                    }
                },
                ...
            ]
        },
        {
            "pivot" : "gutters",
            "suggestions" : [
                {
                    "text" : "Window Cleaning",
                    "displayText" : "Window",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC59...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?q=Window..."
                    }
                },
                ...
            ]
        }
    ]
}
```

對於每個樞紐，回應會包含一份[查詢](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#query_obj)物件清單，其中包含建議的查詢。 `text` 欄位包含建議的查詢，而 `displayText` 欄位包含一個字詞，其可取代原始查詢中的樞紐。 例如，Window Cleaning。

如果擴展查詢字串就是使用者所要尋找的內容，您可以使用 `text` 和 `thumbnail` 欄位對使用者顯示擴展查詢字串。 使用 `webSearchUrl` URL 或 `searchLink` URL，讓縮圖和文字可以點選。 使用 `webSearchUrl` 將 Bing 搜尋結果傳送給使用者，而如果您提供自己的結果頁面，則使用 `searchLink`。

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>後續步驟

若要快速開始使用您的第一個要求，請參閱[建立您的第一個要求](./quick-start.md)。

若要取得您的訂用帳戶金鑰，請參閱[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-video-search-api)。

請熟悉 [Bing 影片搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference) 參考。 此參考包含端點、標頭，以及您用來要求搜尋結果的查詢參數清單。 它也包含回應物件的定義。 

若要改善使用者搜尋方塊體驗，請參閱 [Bing 自動建議 API](../bing-autosuggest/get-suggested-search-terms.md)。 當使用者輸入其查詢字詞時，您可以呼叫此 API 以取得其他人所使用的相關查詢字詞。

務必閱讀 [Bing 使用和顯示需求](./useanddisplayrequirements.md)，您才不會違反任何有關使用搜尋結果的規則。

當您呼叫影片搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 回應的 `totalEstimatedMatches` 欄位包含可以檢視的影片數目估計。 如需關於如何逐頁檢視其餘影片的詳細資訊，請參閱[逐頁檢視影片](./paging-videos.md)。

如需關於取得影片深入解析的詳細資訊，請參閱[影片深入解析](./video-insights.md)。

如需關於取得發燒影片的詳細資訊，請參閱[發燒影片](./trending-videos.md)。