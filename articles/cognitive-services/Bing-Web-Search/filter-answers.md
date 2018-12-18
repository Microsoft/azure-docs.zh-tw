---
title: 篩選 Bing 傳回的 Web 答案 | Microsoft Docs
description: 示範如何使用 responseFilter 來篩選 Bing Web 搜尋 API 所傳回的答案。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: 8B837DC2-70F1-41C7-9496-11EDFD1A888D
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 01/12/2017
ms.author: scottwhi
ms.openlocfilehash: 64095089e4c0841aa1f77165969221836c747738
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42888568"
---
# <a name="filtering-the-answers-that-the-search-response-includes"></a>篩選搜尋回應所包含的答案  

當您查詢 Web 時，Bing 會傳回它認為與該搜尋相關的所有內容。 例如，如果搜尋查詢為 "sailing + dinghies"，回應可能包含下列答案：

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43C...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    },
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA5CA6464E5D...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [...]
        }
    }
}    
```

如果您對於特定類型的內容 (例如影像、影片和新聞) 感興趣，則可以使用 [responseFilter](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#responsefilter) \(英文\) 查詢參數，只要求那些答案。 如果 Bing 找到指定答案的相關內容，Bing 就會傳回它。 回應篩選是以逗號分隔的答案清單。 下列範例示範如何使用 `responseFilter` 來要求 sailing dinghies (小帆艇) 的影像、影片和新聞。 當您為查詢字串編碼時，逗號會變更為 %2C。  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&responseFilter=images%2Cvideos%2Cnews&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

以下顯示前一個查詢的回應。 如您所見，Bing 找不到相關的影片和新聞結果，因此，回應不會包含它們。

```json
{
    "_type" : "SearchResponse",
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3AD78B183C56456C...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "Images",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images"
                }
            }]
        }
    }
}
```

如果您想要在回應中排除特定類型的內容 (例如映像)，則在 responseFilter 值前面加上連字號 (減號) 前置詞，便可將其排除。 請以逗號分隔所要排除的類型： 

```
&responseFilter=-images,-videos
```

雖然 Bing 未在前一個回應中傳回影片和新聞結果，但不表示影片和新聞內容不存在。 這只是表示此頁面未包含它們。 不過，如果您[翻閱](./paging-webpages.md)更多結果，後續的頁面可能就會包含它們。 此外，如果您直接呼叫[影片搜尋 API](../bing-video-search/search-the-web.md) 和[新聞搜尋 API](../bing-news-search/search-the-web.md) 端點，回應很可能就會包含結果。 

不建議您使用 `responseFilter` 來從單一 API 取得結果。 如果您想要來自單一 Bing API 的內容，請直接呼叫該 API。 例如，如果只要接收影像，將要求傳送到影像搜尋 API 端點 `https://api.cognitive.microsoft.com/bing/v7.0/images/search` 或其他[影像](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#endpoints)端點之一。 呼叫單一 API 非常重要，不只是出於效能考量，而且還因為內容特定的 API 會提供更豐富的結果。 例如，您可以使用不適用 Web 搜尋 API 的篩選來篩選結果。  
  
若要取得特定網域中的搜尋結果，請在查詢字串中包括 `site:` 查詢運算子。  

```
https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us
```

> [!NOTE] 
> 視查詢而定，如果您使用 `site:` 查詢運算子，不論 [safeSearch](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#safesearch) 設定為何，回應都有可能包含成人內容。 只有在您了解網站上的內容，而且您的案例支援成人內容的可能性時，才得以使用 `site:`。 
  
## <a name="limiting-the-number-of-answers-in-the-response"></a>限制回應中的答案數目

Bing 會根據順位在回應中包含答案。 例如，如果您查詢 *sailing+dinghies*，Bing 會傳回 `webpages`、`images`、`videos` 及 `relatedSearches`。

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "relatedSearches" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

若要將 Bing 傳回的答案數目限制為前兩個答案 (webpages 和 images)，請將 [answerCount](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#answercount) \(英文\) 查詢參數設為 2。 

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

回應只會包含 `webPages` 和 `images`。

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "rankingResponse" : {...}
}
```

如果您將 `responseFilter` 查詢參數新增到前一個查詢，並將它設為 webpages 和 news，回應就只會包含 webpages，因為不會將 news 設為順位。

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "rankingResponse" : {...}
}
```

## <a name="promoting-answers-that-are-not-ranked"></a>提升未設為順位的答案

如果 Bing 針對查詢所傳回的前幾個順位的答案為 webpages、images、videos 及 relatedSearches，則回應會包含那些答案。 如果您將 [answerCount](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#answercount) \(英文\) 設為二 (2)，Bing 就會傳回前兩個順位的答案：webpages 和 images。 如果您想要 Bing 在回應中包含 images 和 videos，請指定 [promote](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#promote) \(英文\) 查詢參數，並將它設為 images 和 videos。 

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&promote=images%2Cvideos&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

以下為上述要求的回應。 Bing 會傳回前兩個答案 webpages 和 images，並將 videos 提升到答案中。

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailiing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

如果您將 `promote` 設為 news，回應就不會包含 news 答案，因為它不是已設為順位的答案。您只能提升已設為順位的答案。

您想要提升的答案不會計入 `answerCount` 限制。 例如，如果已設為順位的答案為 news、images 及 videos，而且您將 `answerCount` 設為 1 並將 `promote` 設為 news，回應就會包含 news 和 images。 或者，如果已設為順位的答案為 videos、images 及 news，則回應會包含 videos 和 news。

只有當您指定 `answerCount` 查詢參數時，才能使用 `promote`。
