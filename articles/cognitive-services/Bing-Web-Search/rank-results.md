---
title: 使用排名顯示 Web 答案 | Microsoft Docs
description: 說明如何使用排名來顯示 Bing Web 搜尋 API 傳回的答案。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: BBF87972-B6C3-4910-BB52-DE90893F6C71
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 750146f3bb28b94594a71733b68f092880360c5a
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369967"
---
# <a name="using-ranking-to-display-results"></a>使用排名顯示結果  

每個搜尋回應都包含 [RankingResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankingresponse) 答案，可指定您必須以何種方式顯示搜尋結果。 針對傳統搜尋結果頁面，排名回應會依主線內容和資訊看板內容為結果分組。 如果您未以傳統的主線和資訊看板格式顯示結果，您必須為主線內容提供比資訊看板內容更高的可見度。  
  
在每個群組 (主線或資訊看板) 內，[項目](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankinggroup-items)陣列會識別內容必須出現的順序。 每個項目都會提供下列兩種方式來識別答案內的結果。  
  
-   `answerType` 和 `resultIndex` - `answerType` 欄位會識別答案 (例如網頁或新聞)，而 `resultIndex` 會識別答案內的結果 (例如新聞發行項)。 索引是以零為起始。  
  
-   `value` - `value` 欄位所包含的識別碼會符合回應或回應內結果的識別碼。 回應或結果其中一個 (而非兩者) 會包含識別碼。  
  
使用識別碼會較為容易，因為您只需要比對排名識別碼與答案或其中一個結果的識別碼即可。 如果回應物件包含 `id` 欄位，則會同時顯示回應的所有結果。 例如，如果 `News` 物件包含 `id` 欄位，則會同時顯示所有新聞發行項。 如果 `News` 物件未包含 `id` 欄位，則每個新聞發行項都會包含 `id` 欄位，且排名回應會混合新聞發行項與其他答案的結果。  
  
使用 `answerType` 和 `resultIndex` 會稍微複雜一點。 您會使用 `answerType` 來識別哪個答案包含要顯示的結果。 然後，您會使用 `resultIndex` 為答案的結果編製索引，以顯示結果。 (`answerType` 值是 [SearchResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#searchresponse) 物件中的欄位名稱)。如果您應該同時顯示回應的所有結果，排名回應項目不會包含 `resultIndex` 欄位。  

## <a name="ranking-response-example"></a>排名回應範例

以下顯示範例 [RankingResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankingresponse)。 由於 Web 答案未包含 `id` 欄位，因此您會根據排名個別顯示所有的網頁 (每個網頁包含一個 `id` 欄位)。 而由於影像、視訊和相關搜尋答案包含 `id` 欄位，因此您會根據排名同時顯示每個答案的結果。
  
```json
{  
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214...",
        "totalEstimatedMatches" : 835000,
        "value" : [
            {
                "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
                "name" : "Motor Sports - Live at the race track ...",
                "url" : "http:\/\/www.bing.com\/cr?IG=96C4CF214A0A43...",
                "displayUrl" : "www.contoso.com\/usa\/eventsandracing\/motorsport",
                "snippet" : "Here you will find detailed information about racing...",
                "deepLinks" : [{
                    "name" : "Customer Racing",
                    "url" : "http:\/\/www.bing.com\/cr?IG=96C4CF214A0A43...",
                    "snippet" : "Customer racing news; General news..."
            },
            . . .  
        ]  
    },  
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214A...",
        "isFamilyFriendly" : true,
        "value" : [
            {
                "name" : "2016 Supercar Wallpapers",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP...",
                "datePublished" : "2017-03-25T11:14:00",
                "contentUrl" : "http:\/\/www.contoso.com\/wall...",
                "hostPageUrl" : "http:\/\/www.bing.com\/cr?IG=96C4CF214...",
                "contentSize" : "373283 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "http:\/\/www.contoso.com\/lmp-...",
                "width" : 1920,
                "height" : 1080,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 168
                },
                "insightsSourcesSummary" : {
                    "shoppingSourcesCount" : 0,
                    "recipeSourcesCount" : 0
                }
            },
            . . .  
        ]  
    },  
    "relatedSearches" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches",
        "value" : [
            {
                "text" : "vintage racing teams",
                "displayText" : "vintage racing teams",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF2..."
            },
            . . .  
        ]  
    },  
    "videos" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Videos",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/videos...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214A...",
        "isFamilyFriendly" : true,
        "value" : [
            {
                "name" : "Why We Race",
                "description" : "A new era begins in motorsports this weekend...",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF2...",
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.Vo1...",
                "datePublished" : "2014-01-25T16:31:48",
                "publisher" : [
                    {
                        "name" : "Fabrikam"
                    }
                ],
                "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=oL...",
                "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214...",
                "encodingFormat" : "mp4",
                "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=oLAZgD...",
                "width" : 480,
                "height" : 360,
                "duration" : "PT2M42S",
                "motionThumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OM...",
                "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"http:\/\/www.you...<\/iframe>",
                "allowHttpsEmbed" : true,
                "viewCount" : 47325,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 168
                },
                "allowMobileEmbed" : true,
                "isSuperfresh" : false
            },
            . . .  
        ]  
    },  
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "WebPages",
                "resultIndex" : 0,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0"
                }
            },
            {
                "answerType" : "Images",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 1,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.1"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 2,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.2"
                }
            },
            {
                "answerType" : "Videos",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Videos"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 3,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.3"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 4,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.4"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 5,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.5"
                }
            }]
        },
        "sidebar" : {
            "items" : [{
                "answerType" : "RelatedSearches",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches"
                }
            }]
        }
    }
}  
```  
  
根據此排名回應，主線將會顯示下列搜尋結果：  
  
-   第一個網頁結果 
-   所有影像  
-   第二個和第三個網頁結果  
-   所有視訊  
-   第 4 個、第 5 個和第 6 個網頁結果  
  
而資訊看板會顯示下列搜尋結果：  
  
-   所有相關搜尋  
  

## <a name="next-steps"></a>後續步驟

如需將未排名的結果升階的相關資訊，請參閱[升階未排名的答案](./filter-answers.md#promoting-answers-that-are-not-ranked)。

若想了解如何限制回應中已排名的答案數目，請參閱[限制回應中的答案數目](./filter-answers.md#limiting-the-number-of-answers-in-the-response)。

如需使用排名顯示結果的 C# 範例，請參閱 [C# 排名教學課程](./csharp-ranking-tutorial.md)。