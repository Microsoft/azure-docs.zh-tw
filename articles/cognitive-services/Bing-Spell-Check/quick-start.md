---
title: 快速入門：Bing 拼字檢查 API
titlesuffix: Azure Cognitive Services
description: 示範如何使用 Bing 拼字檢查 API 開始使用。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-spell-check
ms.topic: quickstart
ms.date: 06/21/2016
ms.author: scottwhi
ms.openlocfilehash: 29ee7cb4ee648d20b425939553ba31cd9ac150f0
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804186"
---
# <a name="quickstart-your-first-spell-check-request"></a>快速入門：您的第一個拼字檢查要求

第一次進行呼叫之前，您需要取得認知服務訂用帳戶金鑰。 若要取得金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=spellcheck-api)。

若要檢查文字字串是否有拼字和文法錯誤，您會將 GET 要求傳送至下列端點：  
  
```
https://api.cognitive.microsoft.com/bing/v5.0/spellcheck
```

> [!NOTE]
> V7 預覽端點：
> 
> ```
> https://api.cognitive.microsoft.com/bing/v7.0/spellcheck
> ```  
  
要求必須使用 HTTPS 通訊協定。

建議讓所有要求來自伺服器。 將金鑰作為用戶端應用程式的一部份散佈，會讓惡意第三方有更多機會存取到金鑰。 此外，從伺服器進行呼叫可為 API 的未來版本提供單一升級點。

要求必須指定 [text](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#text) 查詢參數，其包含要證明的文字字串。 雖然是選用項目，但要求應該也指定 [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#mkt) 查詢參數，其可識別您希望結果來自哪個市場。 如需選用查詢參數 (例如 `mode`) 的清單，請參閱[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#query-parameters)。 所有查詢參數值均須為 URL 編碼。  
  
要求必須指定 [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#subscriptionkey) 標頭。 雖然是選擇性的，但我們仍建議使用以下標頭：  
  
-   [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#useragent)  
-   [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#clientid)  
-   [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#clientip)  
-   [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#location)  

如需所有要求和回應標頭的清單，請參閱[標頭](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v5-reference#headers)。

## <a name="the-request"></a>要求

以下顯示的要求包含所有建議的查詢參數和標頭。 如果這是您第一次呼叫任何的 Bing API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API 且 Bing 傳回了使用者和裝置組合的用戶端識別碼，則只要包含用戶端識別碼。 
  
```  
GET https://api.cognitive.microsoft.com/bing/v5.0/spellcheck?text=when+its+your+turn+turn,+john,+come+runing&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

> [!NOTE]
> V7 預覽要求：

> ```  
> GET https://api.cognitive.microsoft.com/bing/v7.0/spellcheck?text=when+its+your+turn+turn,+john,+come+runing&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE  
> X-MSEdge-ClientIP: 999.999.999.999  
> X-Search-Location: lat:47.60357;long:-122.3295;re:100  
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
> Host: api.cognitive.microsoft.com  
> ```  

以下顯示前一個要求的回應。 此範例也示範 Bing 特定回應標頭。

```
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{  
    "_type" : "SpellCheck",  
    "flaggedTokens" : [{  
        "offset" : 5,  
        "token" : "its",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "it's",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 25,  
        "token" : "john",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "John",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 19,  
        "token" : "turn",  
        "type" : "RepeatedToken",  
        "suggestions" : [{  
            "suggestion" : "",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 35,  
        "token" : "runing",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "running",  
            "score" : 1  
        }]  
    }]  
}  
```  


## <a name="next-steps"></a>後續步驟

試用 API。 移至[拼字檢查 API 測試主控台](https://dev.cognitive.microsoft.com/docs/services/56e73033cf5ff80c2008c679/operations/57855119bca1df1c647bc358)。 

如需關於取用回應物件的詳細資料，請參閱[拼字檢查文字字串](./proof-text.md)。

