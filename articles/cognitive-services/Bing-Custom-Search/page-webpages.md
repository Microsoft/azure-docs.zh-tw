---
title: Bing 自訂搜尋：逐頁瀏覽可用的網頁 | Microsoft Docs
description: 示範如何逐頁瀏覽 Bing 可傳回的所有網頁。
services: cognitive-services
author: brapel
manager: ehansen
ms.assetid: 26CA595B-0866-43E8-93A2-F2B5E09D1F3B
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 09/28/2017
ms.author: v-brapel
ms.openlocfilehash: 5692776736090c55547c2fe934e2c0aaf840d3c5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982353"
---
# <a name="paging-webpages"></a>逐頁瀏覽網頁 

當您呼叫自訂搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 若要取得可用結果的預估總數，請存取解答物件的 [totalEstimatedMatches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#totalestimatedmatches) 欄位。  
  
下列範例顯示 Web 解答包含的 `totalEstimatedMatches` 欄位。  
  
```  
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    }
}  
```  
  
若要逐頁瀏覽可用的網頁，請使用 [count](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#count) 和 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#offset) 查詢參數。  
  
`count` 參數指定要在回應中傳回的結果數目。 您可能在回應中要求的結果數目上限為 50。 預設值為 10。 傳遞的實際數目可能小於所要求的數目。

`offset` 參數指定要略過的結果數目。 `offset` 是以零為起始，而且應該小於 (`totalEstimatedMatches` - `count`)。  
  
如果您希望每頁顯示 15 個網頁，您可以將 `count` 設定為 15 並將 `offset` 設定為 0，以取得結果的第一頁。 對於後續每個頁面，您可讓 `offset` 以 15 遞增 (例如 15、30)。  
  
以下顯示的範例要求 15 個於位移 45 開始的網頁。  
  
```  
GET https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?q=sailing+dinghies&count=15&offset=45&mkt=en-us&customConfig=123456 HTTP/1.1  
Ocp-Apim-Subscription-Key: <subscription ID>
Host: api.cognitive.microsoft.com  
```  

如果預設 `count` 值適用於您的實作，您只需要指定 `offset` 查詢參數。  
  
```  
GET https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?q=sailing+dinghies&offset=45&mkt=en-us&customConfig=123456 HTTP/1.1  
Ocp-Apim-Subscription-Key: <subscription ID>  
Host: api.cognitive.microsoft.com  
```  

