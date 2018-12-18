---
title: 取得影像見解 | Microsoft Docs
titleSuffix: Bing Web Search APIs - Cognitive Services
description: 顯示如何使用 Bing 影像搜尋 API 來取得更多影像的相關資訊。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: 0BCD936E-D4C0-472D-AE40-F4B2AB6912D5
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: f651d9f773f475e633aed698e134aa6a7c07393b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369982"
---
# <a name="get-insights-about-an-image"></a>取得影像見解

> [!IMPORTANT]
> 不要使用 /images/details 端點來取得影像見解，而是應該使用[圖像式搜尋](../bing-visual-search/overview.md)，因為它可提供更完整的見解。


每個影像包括見解權杖，可讓您取得影像的相關資訊。 例如，您可以取得相關影像的集合、包含影像的網頁，或是可以購買影像所示產品的商家清單。  
  
若要取得影像見解，請擷取回應中的影像 [imageInsightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#image-imageinsightstoken) 權杖。 

```json
"value" : [{
        . . .
        "name":"sailing dinghy.jpg",
        "imageInsightsToken" : "mid_D6426898706EC7..."
        "insightsSourcesSummary" : {
            "shoppingSourcesCount" : 9,
            "recipeSourcesCount" : 0
        },
        . . .
}],
```

接下來，呼叫影像詳細資料端點，並將 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 查詢參數設為 `imageInsightsToken` 中的權杖。  

若要指定您希望取得的見解，請設定 `modules` 查詢參數。 若要取得所有見解，請將 `modules` 設為 `All`。 若要只取得標題和集合見解，請將 `modules` 設定為 `Caption%2CCollection`。 如需可能見解的完整清單，請參閱 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested)。 並非所有見解都適用於所有影像。 回應會包含您要求的所有見解 (若有)。

下列範例要求上述影像的所有可用見解。

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_D6426898706EC7...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
``` 

## <a name="getting-insights-of-a-known-image"></a>取得已知影像的見解

如果您有想要取得見解的影像 URL，請使用 [imgUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imgurl) 查詢參數，而不要使用 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 參數來指定影像。 或者，如果您有影像檔，您可以在 POST 要求主體中傳送二進位檔影像。 如果您使用 POST 要求，`Content-Type` 標頭必須設定為 `multipart/data-form`。 使用任一選項，影像大小都不能超過 1 MB。  
  
如果您有影像的 URL，則下列範例會示範如何要求影像的見解。

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&imgUrl=https%3A%2F%2Fwww.mydomain.com%2Fimages%2Fsunflower.png&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
``` 
  
## <a name="getting-all-image-insights"></a>取得所有影像見解  

若要要求影像的所有見解，請將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested) 查詢參數設定為 `All`。 若要取得相關的搜尋，要求必須包含使用者的查詢字串。 此範例顯示如何使用 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 來指定影像。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_68364D764J...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

最上層的物件是 [ImageInsightsResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsightsresponse) 物件，而非 [Images](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images) 物件。  
  
```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "bcid_3297E6A54E4787A5F51C49D9BA342B9A*ccid_Fe2Hx...",
    "bestRepresentativeQuery" : {
        "text" : "Sailing Dinghy",
        "displayText" : "Sailing Dinghy",
        "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing+Dinghy...",
    },
    "pagesIncluding" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Powerboating Dublin, Dinghy Sailing Courses...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
                "datePublished" : "2017-01-20T00:41:00.0000000Z",
                "contentUrl" : "http:\/\/www.contoso.ie\/content...",
                "hostPageUrl" : "http:\/\/www.contoso.ie\/powerboating...",
                "contentSize" : "59063 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "www.contoso.ie\/powerboating...",
                "width" : 800,
                "height" : 600,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 225
                },
                "imageInsightsToken" : "ccid_pHjQIA0x*mid_17F61B1316A39C92214...",
                "imageId" : "17F61B1316A39C922143FFDE9DFB5B0FB41171",
                "accentColor" : "0997C2"
            },
            . . .
        ]
    },
    "relatedSearches" : {
        "value" : [
            {
                "text" : "Sailing Fun",
                "displayText" : "Sailing Fun",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing...",
                "thumbnail" : {
                    "url" : "https:\/\/tse1.mm.bing.net\/th?q=Sailing+Fun..."
                }
            },
            . . .
        ]
    },
    "visuallySimilarImages" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Weekend On the Water",
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP...",
                "datePublished" : "2010-09-05T12:00:00.0000000Z",
                "contentUrl" : "http:\/\/1.bp.contoso.com\/_dc_6...",
                "hostPageUrl" : "http:\/\/contoso.com\/2010...",
                "contentSize" : "203806 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "contoso.com\/2010...",
                "width" : 1600,
                "height" : 1249,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 234
                },
                "imageInsightsToken" : "ccid_Jg1Kwuc4*mid_5B7DA43976D3A422...",
                "imageId" : "5B7DA43976D3A422BA679A3AB019BB52C08DBC",
                "accentColor" : "0B2543"
            },
            . . .
        ]
    },
    "imageTags" : {
        "value" : [
            {
                "name" : "sail boat"
            },
            . . .
        ]
    }
}
```

## <a name="recognizing-entities-in-an-image"></a>辨識影像中的實體  

實體辨識功能會識別影像中的實體，目前只能識別人類。 若要識別影像的所有實體，請將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested) 查詢參數設定為 `RecognizedEntities`。  

> [!NOTE]
> 您不能與任何其他模組一起指定此模組。 如果您與其他模組一起指定此模組，回應不會包含辨識的實體。  
  
下列範例示範如何使用 [imgUrl](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imgurl) 參數指定影像。 請記得對查詢參數進行 URL 編碼。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=faith+hill&insightsToken=mid_68364D764J...&modules=RecognizedEntities&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  
  
以下顯示前一個要求的回應。 因為影像包含兩個人，所以回應會識別其每個人的區域。 在此情況下，已辨識出 CelebrityAnnotations 和 CelebRecognitionAnnotations 群組中的人員。 Bing 會根據人們符合原始影像的可能性列出每個群組中的人員。 清單是依信賴度的遞減順序。 CelebRecognitionAnnotations 群組提供最高等級的比對正確信賴度。  
  
```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "recognizedEntityGroups" : {
        "value" : [
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebRecognitionAnnotations"
            },
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebrityAnnotations"
            }
        ]
    }
}
```
  
`region` 欄位會識別 Bing 辨識實體的影像區域。 對於人來說，區域代表此人臉部。  
  
矩形的值相對於原始影像的高度與寬度，且是在 0.0 到 1.0 之間的範圍。 例如，如果影像是 300x200，且區域的左上角位於點 (10, 20)，右下角位於點 (290, 150)，則正規化的矩形是：  
  
-   Left: 10 / 300 = 0.03333...  
-   Top:  20 / 200 = 0.1  
-   Right: 290 / 300 = 0.9667...  
-   Bottom: 150 / 200 = 0.75  
  
您可以使用 Bing 在後續見解呼叫中傳回的區域。 例如，針對已辨識的實體取得在視覺效果上類似的影像。 如需詳細資訊，請參閱[裁剪影像以使用視覺效果類似和實體辨識模組](#croppingimages)。 以下顯示區域欄位和您用來裁剪影像的查詢參數之間的對應。  
  
-   Left 對應至 [cal](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cal)  
-   Top 對應至 [cat](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cat)  
-   Right 對應至 [car](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#car)  
-   Bottom 對應至 [cab](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cab)  

## <a name="finding-visually-similar-images"></a>尋找在視覺效果上類似的影像  

若要尋找與原始影像在視覺效果上類似的影像，請將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested) 查詢參數設定為 SimilarImages。  
  
下列要求會示範如何取得在視覺效果上類似的影像。 要求會使用 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 查詢參數來識別原始影像。 若要改善相關性，您應該包含使用者的查詢字串。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?insightsToken=mid_68364D764J...&modules=SimilarImages&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

  
以下顯示前一個要求的回應。  
  
```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarImages" : {
        "value" : [
            {
                "name" : "typical Hawaiian Sunset! :) | Scenes of Hawaii",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=detailv2...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP.Mda2a86...",
                 . . .
            }
        ]
    }
```
  
## <a name="cropping-images-to-use-with-visually-similar-and-entity-recognition-modules"></a>裁剪影像以使用視覺效果類似和實體辨識模組  

若要指定 Bing 用來判斷影像是否在視覺效果上類似的影像區域，或用來執行實體辨識的影像區域，請使用 [cal](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cal)、[cat](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cat)、[cab](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#cab) 和 [car](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#car) 查詢參數。 根據預設，Bing 會使用整個影像。  
  
參數會指定 Bing 用來進行比較的區域左上角和右下角。 請將值指定為原始影像的寬度和高度的分數。 分數值從左上角的 (0.0, 0.0) 開始，到右下角的 (1.0, 1.0) 結束。 例如，若要指定左上角從頂端往下四分之一、從左邊算起四分之一處開始，請將 `cal` 設定為 0.25，將 `cat` 設定為 0.25。  
  
下列呼叫順序顯示指定裁剪區域的效果。 第一個呼叫未包含裁剪，Bing 會辨識影像中間並排站著的兩個人。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  

回應顯示兩個可辨識的實體。  
  
```json 
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value": [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                },  
                {  
                    "region" : {  
                        "left" : 0.25,  
                        "top" : 0.2488889,  
                        "right" : 0.4466667,  
                        "bottom" : 0.5111111  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Marcus Appel",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
            "name" : "CelebRecognitionAnnotations"
        }]
    }  
}  
```  
  
第二個呼叫以垂直方式從中間裁剪影像，Bing 會辨識影像右邊有一個人。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?cal=0.5&cat=0.0&car=1.0&cab=1.0&modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

回應顯示一個可辨識的實體。  
  
```json  
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value" : [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
                "name" : "CelebRecognitionAnnotations"  
            }
        ]
    }
}  
```  
  
## <a name="finding-visually-similar-products"></a>尋找在視覺效果上類似的產品  

若要尋找包含與原始影像中產品視覺效果類似之產品的影像，請將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested) 查詢參數設定為 SimilarProducts。  
  
下列要求會示範如何取得視覺效果類似之產品的影像。 要求會使用 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 查詢參數來識別前一個要求中傳回的原始影像。 若要改善相關性，您應該包含使用者的查詢字串。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=anne+klein+dresses&modules=SimilarProducts&insightsToken=ccid_WOeyfoSp*mid_4B0A357&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```
  
以下顯示前一個要求的回應。 回應包含類似產品的影像，並指出有多少商家在線上提供該產品、是否有產品評等，以及找到的最低價格 (請參閱 `aggregateOffer` 欄位)。  
  
```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarProducts" : {
        "value" : [
            {
                "name" : "Sequin One-Shoulder Twist-Drape Dress",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=de...",  
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP.M85bdee...",  
                . . .
            },
            . . .
        ]
    }
}
```
  
若要取得一份在線上提供該產品的商家清單 (請參閱 [offerCount](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#offer-offercount) 欄位)，請再次呼叫 API，然後將 `modules` 設定為 ShoppingSources。 然後，將 `insightsToken` 查詢參數設定為在產品摘要影像中找到的權杖。  
  
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=ShoppingSources&insightsToken=ccid_hb3uRvUk*mid_BF5C252A47F2C765...&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

以下為上述要求的回應。  
  
```json  
{  
    "_type" : "ImageInsights",  
    "shoppingSources" : {  
        "offers" : [{  
            "url" : "http:\/\/www.contoso.com\/dp\/B00O...",  
            "seller" : {  
                "name" : "Contoso",  
                "image" : {  
                    "url" : "https:\/\/tse3.mm.bing.net\/th?id=A10d50fe..."  
                }  
            },  
            "price" : 126.87,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        },  
        {  
            "url" : "http:\/\/www.adatum.com\/product\/heritage...\/",  
            "seller" : {  
                "name" : "fabrikam.com"  
            },  
            "price" : 495,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        }]  
    }  
}  
```
