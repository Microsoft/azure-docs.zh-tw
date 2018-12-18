---
title: 快速入門：Bing 自動建議 API
titlesuffix: Azure Cognitive Services
description: 示範如何開始使用 Bing 自動建議 API。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-autosuggest
ms.topic: quickstart
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 446d271854a4e45bcea8c261a0dc078e549f8229
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48830748"
---
# <a name="quickstart-making-your-first-autosuggest-query"></a>快速入門：進行第一個自動建議查詢

第一次進行呼叫之前，您需要取得認知服務訂用帳戶金鑰。 若要取得金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=autosuggest-api)。

若要取得網路搜尋結果，您要將 GET 要求傳送至下列端點：

```http
https://api.cognitive.microsoft.com/bing/v5.0/Suggestions
```

> [!NOTE]
> V7 預覽端點：
>
> ```http
> https://api.cognitive.microsoft.com/bing/v7.0/Suggestions
> ```

要求必須使用 HTTPS 通訊協定。

建議讓所有要求來自伺服器。 將金鑰作為用戶端應用程式的一部份散佈，會讓惡意第三方有更多機會存取到金鑰。 此外，從伺服器進行呼叫，API 未來就能以同個升級點更新版本。

要求必須指定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query) 查詢參數，其包含使用者的部份搜尋字詞。 雖是選擇性，但請在要求中指定 [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#mkt) 查詢參數，其可識別您希望從哪個市場取得結果。 如需選用查詢參數的清單，請參閱[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query-parameters)。 所有查詢參數值均須為 URL 編碼。

要求必須指定 [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#subscriptionkey) 標頭。 雖然是選擇性的，但我們仍建議使用以下標頭：

- [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#useragent)
- [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientid)
- [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientip)
- [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#location)

用戶端 IP 和位置標頭對於傳回位置感知內容很重要。

如需所有要求和回應標頭的清單，請參閱[標頭](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#headers)。

## <a name="the-request"></a>要求

要求應該包含所有建議的查詢參數和標頭。 您會在每次使用者在搜尋方塊中鍵入新字元時，呼叫此 API。 查詢字串的完整性會影響 API 傳回之建議查詢字詞的相關性。 查詢字串越完整，建議之查詢字詞的相關性就越大。 例如，API 為 *s* 可能傳回的建議也許比查詢為 *sailing dinghies* 傳回的建議相關性較小。 

下列範例顯示針對 *sail* 傳回建議查詢字串的要求。

```http
GET https://api.cognitive.microsoft.com/bing/v5.0/suggestions?q=sail&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

> [!NOTE]
> V7 預覽要求：

> ```http
> GET https://api.cognitive.microsoft.com/bing/v7.0/suggestions?q=sail&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE
> X-MSEdge-ClientIP: 999.999.999.999
> X-Search-Location: lat:47.60357;long:-122.3295;re:100
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
> Host: api.cognitive.microsoft.com
> ```

如果這是您第一次呼叫任何的 Bing API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API 且 Bing 傳回了使用者和裝置組合的用戶端識別碼，則只要包含用戶端識別碼標頭。

以下顯示先前要求的回應。 回應包括內含搜尋查詢建議清單的網路建議群組。 每個建議都包含 `displayText``query`，以及 `url` 欄位。

`displayText` 欄位包含建議的查詢，您可以用來填入搜尋方塊的下拉式清單。 您必須依提供的順序顯示回應包含的所有建議。  

如果使用者從下拉式清單選取查詢，您可以使用 `query` 欄位中的查詢字串以呼叫 [Bing 搜尋 API](../bing-web-search/search-the-web.md)，並自行顯示結果。 或者，您可以使用 `url` 欄位中的 URL，以將使用者傳送至 Bing 搜尋結果頁面。

```  
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Suggestions",
    "queryContext" : {
        "originalQuery" : "sail"
    },
    "suggestionGroups" : [{
        "name" : "Web",
        "searchSuggestions" : [{
            "url" : "https:\/\/www.bing.com\/search?q=sailing+lessons+seattle&FORM=USBAPI",
            "displayText" : "sailing lessons seattle",
            "query" : "sailing lessons seattle",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+moon+news&FORM=USBAPI",
            "displayText" : "sailor moon news",
            "query" : "sailor moon news",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+jack%27s+lincoln+city&FORM=USBAPI",
            "displayText" : "sailor jack's lincoln city",
            "query" : "sailor jack's lincoln city",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailing+anarchy&FORM=USBAPI",
            "displayText" : "sailing anarchy",
            "query" : "sailing anarchy",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale&FORM=USBAPI",
            "displayText" : "sailboats for sale",
            "query" : "sailboats for sale",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailstn.mylabsplus.com&FORM=USBAPI",
            "displayText" : "sailstn.mylabsplus.com",
            "query" : "sailstn.mylabsplus.com",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailusfood&FORM=USBAPI",
            "displayText" : "sailusfood",
            "query" : "sailusfood",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale+seattle&FORM=USBAPI",
            "displayText" : "sailboats for sale seattle",
            "query" : "sailboats for sale seattle",
            "searchKind" : "WebSearch"
        }]
    }]
}
```

## <a name="next-steps"></a>後續步驟

試用 API。 前往[自動建議 API 測試主控台](https://dev.cognitive.microsoft.com/docs/services/56c7694ecf5ff801a090fbd1/operations/56c769a2cf5ff801a090fbd2)。

如需關於取用回應物件的詳細資料，請參閱[取得建議的搜尋字詞](./get-suggested-search-terms.md) (英文)。
