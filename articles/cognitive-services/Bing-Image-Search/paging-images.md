---
title: 如何逐頁瀏覽可用的影像 | Microsoft Docs
description: 示範如何逐頁瀏覽 Bing 可傳回的所有影像。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: 3C8423F8-41E0-4F89-86B6-697E840610A7
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 04/15/2017
ms.author: scottwhi
ms.openlocfilehash: a74ee817e84be5bb563c5fdaf25afc1dc14732e5
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367846"
---
# <a name="paging-results"></a>分頁結果

當您呼叫影像搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 若要取得可用結果的預估總數，請存取解答物件的 [totalEstimatedMatches](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#totalestimatedmatches) 欄位。  
  
下列範例顯示影像解答包含的 `totalEstimatedMatches` 欄位。  
  
```json
{
    "_type" : "Images",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=73118C8...",
    "totalEstimatedMatches" : 838,
    "value" : [...]  
}  
```  
  
若要逐頁瀏覽可用的影像，請使用 [count](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#count) 和 [offset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#offset) 查詢參數。  
  
`count` 參數指定要在回應中傳回的結果數目。 您可能在回應中要求的結果數目上限為 150。 預設值為 35。 傳遞的實際數目可能小於所要求的數目。

`offset` 參數指定要略過的結果數目。 `offset` 是以零為起始，而且應該小於 (`totalEstimatedMatches` - `count`)。  
  
如果您希望每頁顯示 20 個影像，您可以將 `count` 設定為 20 並將 `offset` 設定為 0，以取得結果的第一頁。 以下顯示的範例要求 20 個於位移 40 開始的影像。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&count=20&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  

如果預設 `count` 值適用於您的實作，您只需要指定 `offset` 查詢參數。  
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&offset=40&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```  
  
如果您想要一次逐頁瀏覽 35 個影像，您應該在第一個要求將 `offset` 查詢參數設定為 0，然後在每個後續要求將 `offset` 遞增 35。 不過，後續回應中的某些結果可能與前一個回應重複。 例如，此回應中的前兩個影像可能與前一個回應中的最後兩個影像相同。

若要消除重複的結果，請使用 `Images` 物件的 [nextOffset](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#nextoffset) 欄位。 `nextOffset` 欄位會告訴您要用於下一個要求的 `offset`。 例如，如果您想要一次逐頁瀏覽 30 個影像，您可將第一個要求中的 `count` 設定為 30 並將 `offset` 設定為 0。 在下一個要求中，您可將 `count` 設定為 30 並將 `offset` 設定為前一個回應的 `nextOffset` 值。 若要向後逐頁瀏覽，我們建議維護先前位移的堆疊並快顯最近使用的堆疊。

> [!NOTE]
> 逐頁瀏覽只適用於影像搜尋 (/images/search)，不適用於影像見解或發燒影像 (/images/trending)。