---
title: 搜尋網路上的發燒影像 | Microsoft Docs
description: 說明如何使用 Bing 影像搜尋 API 來搜尋網路上的發燒影像。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: EAB92D35-5C0B-4A0A-8F49-02DF7FAD44B4
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: b12524cd4c1896501820209b3a45746b8f38b210
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367903"
---
# <a name="get-trending-images"></a>取得發燒影像  

若要取得今天的發燒影像，請傳送下列 GET 要求：  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/trending?mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

發燒影像 API 目前僅支援下列市場：  

- en-US (英文，美國)  
- en-CA (英文，加拿大)  
- en-AU (英文，澳大利亞)  
- zh-CN (中文，中國)

回應包含 [TrendingImages](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#trendingimages) 物件，該物件會依照類別列出影像。 使用類別的 `title` 在您的使用者體驗中將影像分組。 這些類別可能每天變更。  
  
```json
{
    "_type" : "TrendingImages",  
    "categories" : [{  
        "title" : "Popular people searches",  
        "tiles" : [{  
            "query" : {  
                "text" : "Smith",  
                "displayText" : "Mr. Smith",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=smith&FORM=..."
            },  
            "image" : {  
                "id" : "C3C60AE779A054D5CD80D3CACF0F3",  
                "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OIP.M2532...",  
                "contentUrl" : "http:\/\/www.contoso.com.au\/assets\/Uploads\/smith-SH01.jpg",  
                "thumbnail" : {  
                    "width" : 288,  
                    "height" : 300  
                }  
            }  
        },  
        . . .  
        ]  
    },  
    . . .  
    {  
        "title" : "Popular Halloween searches",  
        "tiles" : [{  
            "query" : {  
                "text" : "Halloween costumes for adults",  
                "displayText" : "Halloween costumes for adults",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Halloween+costumes..."
            },  
            "image" : {  
                "id" : "0F3395F2983003F89DCEE711B55D7FA53E4",  
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OIP.Me429c...",  
                "contentUrl" : "http:\/\/images.domain.com\/products\/8179\/1-1\/adult-squirrel...",  
                "thumbnail" : {  
                    "width" : 336,  
                    "height" : 480  
                }  
            }  
        }]  
    }]  
}  
```  
  
每個圖格都包含一個影像，以及用於取得相關影響的選項。 若要取得相關影像，您可以使用查詢 `text`，自行呼叫[影像搜尋 API](./search-the-web.md) 並顯示相關影像。 或者，您可以使用 `webSearchUrl` 中的 URL，將使用者導向 Bing 的影像搜尋結果頁面，其中包含相關的影像。 

如果您呼叫影像搜尋 API 以取得相關影像，請將 [id](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#id) 查詢參數設定為 `id` 欄位中的識別碼。 指定識別碼可確保回應包含影像 (這是回應中的第一個影像) 及其相關的影像。 此外，將 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#q) 查詢參數設定為 `query` 物件的`text` 欄位中的文字。

下列範例示範如何使用影像識別碼來取得上述發燒影像 API 回應中的 Mr. Smith 相關影像。

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=Smith&id=77FDE4A1C6529A23C7CF0EC073FAA64843E828F2&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  
