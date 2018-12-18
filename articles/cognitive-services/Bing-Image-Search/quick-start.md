---
title: 影像搜尋 API 快速入門 | Microsoft Docs
description: 示範如何開始使用 Bing 影像搜尋 API。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: BE2B2F8C-20B5-4E0B-AEC8-EAD505BA4C85
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 9e211cf5acd17ab80948d0b7161bdd2a9220c4a6
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368002"
---
# <a name="your-first-images-search-query"></a>第一個影像搜尋查詢

第一次進行呼叫之前，您需要取得 Bing 搜尋認知服務訂用帳戶金鑰。 若要取得金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)。

若要取得影像搜尋結果，您要將 GET 要求傳送至下列端點：  
  
```
https://api.cognitive.microsoft.com/bing/v7.0/images/search
```
  
要求必須使用 HTTPS 通訊協定。

建議讓所有要求來自伺服器。 將金鑰作為用戶端應用程式的一部份散佈，會讓惡意第三方有更多機會存取到金鑰。 此外，從伺服器進行呼叫，API 未來就能以同個升級點更新版本。

要求必須指定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#query) 查詢參數，其中含有使用者的搜尋字詞。 雖是選擇性，但請在要求中指定 [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#mkt) 查詢參數，其可識別您希望從哪個市場取得結果。 如需選擇性查詢參數 (例如 `freshness` 和 `size`) 的清單，請參閱[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#query-parameters)。 所有查詢參數值均須為 URL 編碼。  
  
要求必須指定 [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#subscriptionkey) 標頭。 雖然是選擇性的，但我們仍建議使用以下標頭：  
  
-   [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#useragent)  
-   [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#clientid)  
-   [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#clientip)  
-   [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#location)  

用戶端 IP 和位置標頭對於傳回位置感知內容很重要。  

如需所有要求和回應標頭的清單，請參閱[標頭](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#headers)。

## <a name="the-request"></a>要求

以下顯示搜尋要求，其包含所有建議的查詢參數和標頭。 如果這是您第一次呼叫任何的 Bing API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API 且 Bing 傳回了使用者和裝置組合的用戶端識別碼，則只要包含用戶端識別碼。 
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

以下顯示前一個要求的回應。

```json
{
    "_type" : "Images",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=73118C8B4E3...",
    "totalEstimatedMatches" : 838,
    "value" : [
        {
            "name" : "File:Rich Passage Minto Sailing Dinghy.jpg - Wikipedia",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=73118C8B4E3...",
            "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP.GNarK7m...",
            "datePublished" : "2011-10-29T11:26:00",
            "contentUrl" : "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3...",
            "hostPageUrl" : "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3687...",
            "contentSize" : "79239 B",
            "encodingFormat" : "jpeg",
            "hostPageDisplayUrl" : "en.wikipedia.org\/wiki\/File:Rich_Passage...",
            "width" : 526,
            "height" : 688,
            "thumbnail" : {
                "width" : 229,
                "height" : 300
            },
            "imageInsightsToken" : "ccid_GNarK7ma*mid_CCF85447ADA6...",
            "insightsSourcesSummary" : {
                "shoppingSourcesCount" : 0,
                "recipeSourcesCount" : 0
            },
            "imageId" : "CCF85447ADA6FFF9E96E7DF0B796F7A86E34593",
            "accentColor" : "376094"
        },
        . . .
    ],
    "queryExpansions" : [
        {
            "text" : "Small Sailing Dinghies",
            "displayText" : "Small",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=73118C8B4...",
            "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api...",
            "thumbnail" : {
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?q=..."
            }
        },
        . . .
    ],
    "nextOffset" : 10,
    "pivotSuggestions" : [
        {
            "pivot" : "sailing",
            "suggestions" : []
        },
        {
            "pivot" : "dinghies",
            "suggestions" : [
                {
                    "text" : "Sailing Tender",
                    "displayText" : "Tender",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=73118...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?q=..."
                    }
                },
                . . .
            ]
        }
    ],
    "displayShoppingSourcesBadges" : false,
    "displayRecipeSourcesBadges" : true
}
```

## <a name="next-steps"></a>後續步驟

試用 API。 移至[影像搜尋 API 測試主控台](https://dev.cognitive.microsoft.com/docs/services/8336afba49a84475ba401758c0dbf749/operations/571fab09dbe2d933e891028f)。 

如需關於取用回應物件的詳細資料，請參閱[搜尋網路](./search-the-web.md) (英文)。

如需如何取得影像深入解析 (例如包含此影像或影像中所辨識人員的網頁) 的詳細資訊，請參閱[影像深入解析](./image-insights.md)。  
  
如需社交媒體上發燒影像的詳細資料，請參閱[發燒影像](./trending-images.md) (英文)。  
