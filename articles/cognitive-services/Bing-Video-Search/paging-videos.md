---
title: 如何逐頁瀏覽可用的影片 - Bing 影片搜尋
titlesuffix: Azure Cognitive Services
description: 示範如何逐頁瀏覽 Bing 可傳回的所有影片。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: 9b030312c562d1c0a6cbacfc7f424289dee2e8de
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225560"
---
# <a name="paging-videos"></a>逐頁瀏覽影片

當您呼叫影片搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 若要取得可用結果的預估總數，請存取解答物件的 [totalEstimatedMatches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos-totalestimatedmatches) 欄位。  
  
下列範例顯示影片解答包含的 `totalEstimatedMatches` 欄位。  
  
```  
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D56...",
    "totalEstimatedMatches" : 1000,
    "value" : [...]
}  
```  
  
若要逐頁瀏覽可用的影片，請使用 [count](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#count) 和 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#offset) 查詢參數。  
  
`count` 參數指定要在回應中傳回的結果數目。 您可能在回應中要求的結果數目上限為 105。 預設值為 35。 傳遞的實際數目可能小於所要求的數目。

`offset` 參數指定要略過的結果數目。 `offset` 是以零為起始，而且應該小於 (`totalEstimatedMatches` - `count`)。  
  
如果您希望每頁顯示 20 部影片，您可以將 `count` 設定為 20 並將 `offset` 設定為 0，以取得結果的第一頁。 對於後續每個頁面，您可讓 `offset` 以 20 遞增 (例如 20、40)。  

以下顯示的範例要求 20 部於位移 40 開始的影片。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&count=20&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  

如果預設 `count` 值適用於您的實作，您只需要指定 `offset` 查詢參數。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  

通常，如果您要一次逐頁瀏覽 35 部影像，您可在第一個要求將 `offset` 查詢參數設定為 0，然後在每個後續要求將 `offset` 遞增 35。 不過，後續回應中的某些結果可能與前一個回應重複。 例如，此回應中的前兩部影片可能與前一個回應中的最後兩部影片相同。

若要消除重複的結果，請使用 `Videos` 物件的 [nextOffset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#videos-nextoffset) 欄位。

例如，如果您想要一次逐頁瀏覽 30 部影片，您可將第一個要求中的 `count` 設定為 30 並將 `offset` 設定為 0。 在下一個要求中，您會將 `offset` 查詢參數設定為 `nextOffset` 值。


> [!NOTE]
> 逐頁瀏覽只適用於影片搜尋 (/videos/search)，而不適用於影片見解 (/videos/details) 或發燒影片 (/videos/trending)。