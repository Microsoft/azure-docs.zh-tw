---
title: 搜尋網路上的發燒影片 - Bing 影片搜尋
titlesuffix: Azure Cognitive Services
description: 說明如何使用 Bing 影片搜尋 API 來搜尋網路上的發燒影片。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 8a6ccc9ea8cf9468d7638360c9db8131bc6dc5be
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222032"
---
# <a name="get-trending-videos"></a>取得發燒影片  

若要取得今天的發燒影片，請傳送下列 GET 要求：  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/trending?mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

下列市場支援發燒影片。  
 
-   en-AU (英文，澳大利亞)  
-   en-CA (英文，加拿大)  
-   en-GB (英文，英國)  
-   en-ID (英文，印尼)  
-   en-IE (英文，愛爾蘭)  
-   en-IN (英文，印度)  
-   en-NZ (英文，紐西蘭)  
-   en-PH (英文，菲律賓)  
-   en-SG (英文，新加坡)  
-   en-US (英文，美國)  
-   en-WW (英文，全球彙總代碼)  
-   en-ZA (英文，南非)  
-   zh-CN (中文，中國)

  
下列範例顯示包含發燒影片的回應。  

```  
{  
    "_type" : "TrendingVideos",  
    "bannerTiles" : [
        {  
            "query" : {  
                "text" : "Hello - Smith",  
                "displayText" : "Hello - Smith",  
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3E8F5..."
            },  
            "image" : {  
                "description" : "Image from: contosowallpapers.com",  
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=RsA%2fdPlTmx4zS...",  
                "headLine" : "\"Hello\" is a song by...",  
                "contentUrl" : "http:\/\/www.contosowallpapers.com\/wp-content..."  
            }  
        },  
        . . .  
    ],  
    "categories" : [
        {  
            "title" : "Music videos",  
            "subcategories" : [
                {  
                    "tiles" : [
                        {  
                            "query" : {  
                                "text" : "Song Title - Artist Name",  
                                "displayText" : "Song Title - Artist Name",  
                                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3E8F5..."
                            },  
                            "image" : {  
                                "description" : "Image from: contoso.com",  
                                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=...",  
                                "contentUrl" : "http:\/\/images6.contoso.com\/image..."  
                            }  
                        },  
                        . . .  
                    ],
                    "title" : "Top "  
                },
                {
                    "tiles" : [...],
                    "title" : "Trending"
                },
                . . .
            ],  
        },
        {
            "title" : "Viral videos",
            "subcategories" : [
                {
                    "tiles" : [...],
                    "title" : "Trending"
                },
                . . .
            ],  
        },
        . . .  
    ]  
}  
  
```  
此回應包含依類別和子類別列出的影片清單。 例如，如果類別清單包含了 Music Videos 類別，而其中一個子類別是 Top，您可在您的使用者體驗中建立 Top Music Videos 類別。 您可接著使用 `thumbnailUrl`、`displayText` 和 `webSearchUrl` 欄位，在每個類別 (例如 Top Music Videos) 之下建立可點選的圖格。 當使用者按一下此圖格時，他們就會前往可播放影片的 Bing 影片瀏覽器。

回應也會包含橫幅影片，也就是最熱門的發燒影片。 橫幅影片可能來自一或多個類別。  
  
