---
title: 快速入門：Bing 實體搜尋 API
description: 示範如何使用 Bing 實體搜尋 API 開始使用。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: quickstart
ms.date: 07/06/2017
ms.author: scottwhi
ms.openlocfilehash: ffc9ebb21c6646b1a39af4659053adf4157d204b
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48813952"
---
# <a name="quickstart-making-your-first-bing-entity-search-request"></a>快速入門：進行您的第一次 Bing 實體搜尋要求

Bing 實體搜尋 API 會將搜尋查詢傳送至 Bing，並取得包含實體和地點的結果。 地點結果包含餐廳、旅館或其他本地商家。 針對地點，查詢可指定本地商家名稱或要求清單 (例如，我附近的餐廳)。 實體結果包含人員、地點或事項。 此內容中的地點是觀光景點、州、國家/地區等。 

## <a name="first-steps"></a>第一個步驟

第一次進行呼叫之前，您需要取得認知服務訂用帳戶金鑰。 若要取得金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-entities-search-api)。 (如果實體搜尋 API 未顯示在頂端，請按一下 [搜尋] 索引標籤，然後向下捲動直到您看到它。)

## <a name="the-endpoint"></a>端點

若要取得實體和地點搜尋結果，請將 GET 要求傳送至下列端點：  
  
```
https://api.cognitive.microsoft.com/bing/v7.0/entities
```

要求必須使用 HTTPS 通訊協定。

我們建議所有要求均應來自伺服器。 隨著用戶端應用程式散佈金鑰，會讓惡意的第三方有更多機會存取該應用程式。 此外，從伺服器進行呼叫可為 API 的未來版本提供單一升級點。

## <a name="specifying-query-parameters-and-headers"></a>指定查詢參數和標頭

要求必須指定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#query) 查詢參數，其包含使用者的搜尋字詞。 要求也必須指定 [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#mkt) 查詢參數，其可識別您希望結果來自哪個市場。 如需選用查詢參數的清單，請參閱[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#query-parameters)。 對所有查詢參數進行 URL 編碼。  
  
要求必須指定 [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#subscriptionkey) 標頭。 雖然是選擇性的，但我們仍建議使用以下標頭：  
  
-   [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#useragent)  
-   [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#clientid)  
-   [X-MSEdge-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#clientip)  
-   [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#location)  

用戶端 IP 和位置標頭對於傳回位置感知內容很重要。  

如需所有要求和回應標頭的清單，請參閱[標頭](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#headers)。

## <a name="the-request"></a>要求

以下顯示的實體要求包含所有建議的查詢參數和標頭。 

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/entities?q=mount+rainier&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

如果這是您第一次呼叫任何的 Bing API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API 且 Bing 傳回了使用者和裝置組合的用戶端識別碼，則只要包含用戶端識別碼。

## <a name="the-response"></a>回應

以下顯示前一個要求的回應。 此範例也示範 Bing 特定回應標頭。 如需回應物件的相關資訊，請參閱 [SearchResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#searchresponse)。

```
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "mount rainier"
    },
    "entities" : {
        "queryScenario" : "DominantEntity",
        "value" : [{
            "contractualRules" : [{
                "_type" : "ContractualRules\/LicenseAttribution",
                "targetPropertyName" : "description",
                "mustBeCloseToContent" : true,
                "license" : {
                    "name" : "CC-BY-SA",
                    "url" : "http:\/\/creativecommons.org\/licenses\/by-sa\/3.0\/"
                },
                "licenseNotice" : "Text under CC-BY-SA license"
            },
            {
                "_type" : "ContractualRules\/LinkAttribution",
                "targetPropertyName" : "description",
                "mustBeCloseToContent" : true,
                "text" : "en.wikipedia.org",
                "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
            },
            {
                "_type" : "ContractualRules\/MediaAttribution",
                "targetPropertyName" : "image",
                "mustBeCloseToContent" : true,
                "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
            }],
            "webSearchUrl" : "https:\/\/www.bing.com\/search?q=Mount%20Rainier...",
            "name" : "Mount Rainier",
            "image" : {
                "name" : "Mount Rainier",
                "thumbnailUrl" : "https:\/\/www.bing.com\/th?id=A21890c0e1f...",
                "provider" : [{
                    "_type" : "Organization",
                    "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
                }],
                "hostPageUrl" : "http:\/\/upload.wikimedia.org\/wikipedia...",
                "width" : 110,
                "height" : 110
            },
            "description" : "Mount Rainier, Mount Tacoma, or Mount Tahoma is the highest...",
            "entityPresentationInfo" : {
                "entityScenario" : "DominantEntity",
                "entityTypeHints" : ["Attraction"],
                "entityTypeDisplayHint" : "Mountain"
            },
            "bingId" : "9ae3e6ca-81ea-6fa1-ffa0-42e1d78906"
        }]
    }
}
```



## <a name="next-steps"></a>後續步驟

試用 API。 移至[實體搜尋 API 測試主控台](https://dev.cognitive.microsoft.com/docs/services/7a3fb374be374859a823b79fd938cc65/)。 

如需關於取用回應物件的詳細資料，請參閱[搜尋網路上的實體和地點](./search-the-web.md)。

務必閱讀 [Bing 使用和顯示需求](./use-display-requirements.md)，您才不會違反任何有關使用搜尋結果的規則。
