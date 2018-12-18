---
title: 取得影片深入解析 - Bing 影片搜尋
titlesuffix: Azure Cognitive Services
description: 示範如何使用 Bing 影片搜尋 API 來取得更多視訊的相關資訊。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: af6056507cb70ab33bee5955ddf7bd0f77fd04eb
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219986"
---
# <a name="get-insights-about-a-video"></a>取得影片深入解析

每個影片皆包含一個影片識別碼，您可使用該識別碼來取得關於影片的更多資訊，例如相關影片。  
  
若要取得關於影片的深入解析，請在回應中擷取其 [videoId](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#video-videoid) 權杖。 

```json
    "value" : [
        {
            . . .
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear...",
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjHBZ--g",
            "videoId" : "6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63",
            . . .
        }
    ],
```

接著，將下列 GET 要求傳送至「影片詳細資料」端點。 將 [id](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#id) 查詢參數設為 `videoId` 權杖。 若要指定您希望取得的深入解析，請設定 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#modulesrequested) 查詢參數。 若要取得所有深入解析，請將 `modules` 設為 All。 回應會包含您要求的所有深入解析 (若有)。

```
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/details?q=sailiing+dinghies&id=6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
``` 

## <a name="getting-related-videos-insights"></a>取得相關影片深入解析  

若要取得與指定影片相關的影片，請將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#modulesrequested) 查詢參數設為 RelatedVideos。
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/details?q=sailiing+dinghies&id=6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63&modules=RelatedVideos&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  
  
以下為上述要求的回應。 最上層的物件是 [VideoDetails](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videodetails) 物件，而非 [Videos](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos) 物件。  
  
```  
{
    "_type" : "Api.VideoDetails.VideoDetails",
    "relatedVideos" : {
        "value" : [
            {
                "name" : "How to sail - Reefing a Sail",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=7284B07...",
                "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OVP.zt...",
                "datePublished" : "2014-03-04T16:11:09",
                "publisher" : [
                    {
                        "name" : "Fabrikam"
                    }
                ],
                "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=...",
                "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=7284B07...",
                "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?...",
                "duration" : "PT4M56S",
                "motionThumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OM...",
                "allowHttpsEmbed" : true,
                "viewCount" : 21756,
                "videoId" : "AC1A157A4DDB571D03D6AC157A4DDB571D03D6",
                "allowMobileEmbed" : false,
                "isSuperfresh" : false
            },
            . . .
        ]
    }
}
```