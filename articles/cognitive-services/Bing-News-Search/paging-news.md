---
title: 如何逐頁瀏覽可用的新聞文章 | Microsoft Docs
description: 示範如何逐頁瀏覽 Bing 可傳回的所有新聞文章。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: EA388F72-FA43-493B-967C-9560B3243C62
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 2c90d468536f0864d7deac073667e29e9a54692f
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367898"
---
# <a name="paging-news"></a>逐頁瀏覽新聞

當您呼叫新聞搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 若要取得可用結果的預估總數，請存取解答物件的 [totalEstimatedMatches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#news-totalmatches) 欄位。  
  
下列範例顯示新聞解答包含的 `totalEstimatedMatches` 欄位。  
  
```  
{  
    "_type" : "News",  
    "readLink" : "https:\/\/api.cognitive.microsoft.com\/bing\/v7\/news\/search?q=sailing+dinghies",  
    "totalEstimatedMatches" : 88400,  
    "value" : [...]  
}  
```  
  
若要逐頁瀏覽可用的文章，請使用 [count](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#count) 和 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#offset) 查詢參數。  
  
`count` 參數指定要在回應中傳回的結果數目。 您可能在回應中要求的結果數目上限為 100。 預設值為 10。 傳遞的實際數目可能小於所要求的數目。

`offset` 參數指定要略過的結果數目。 `offset` 是以零為起始，而且應該小於 (`totalEstimatedMatches` - `count`)。  

如果您希望每頁顯示 20 篇文章，您可以將 `count` 設定為 20 並將 `offset` 設定為 0，以取得結果的第一頁。 對於後續每個頁面，您可讓 `offset` 以 20 遞增 (例如 20、40)。  
  
以下顯示的範例要求 20 篇於位移 40 開始的新聞文章。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search?q=sailing+dinghies&count=20&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  
  
如果預設 `count` 值適用於您的實作，您只需指定如下列範例所示的 `offset` 查詢參數：  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search?q=sailing+dinghies&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  
  
> [!NOTE]
> 逐頁瀏覽只適用於新聞搜尋 (/news/search)，但不適用於發燒主題 (/news/trendingtopics) 或新聞分類 (/news)。