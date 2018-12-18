---
title: 知識探索服務 API 中的評估方法 | Microsoft Docs
description: 了解如何在認知服務的知識探索服務 (KES) API 中使用評估方法。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: fc3d73b326b565cfe40d1b82cc419357b28a801a
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368030"
---
# <a name="evaluate-method"></a>評估方法
「評估」方法會根據索引資料來評估並傳回結構化查詢運算式的輸出。

通常會從解譯方法的回應中取得運算式。  但是您也可以自行撰寫查詢運算式 (請參閱[結構化查詢運算式](Expressions.md))。  

## <a name="request"></a>要求 
`http://<host>/evaluate?expr=<expr>&attributes=<attrs>[&<options>]`   

Name|值|說明
----|----|----
expr       | 文字字串 | 會選取索引實體子集的結構化查詢運算式。
屬性 | 文字字串 | 回應中所要包含屬性的逗號分隔清單。
count      | 數字 (預設值 = 10) | 要傳回的結果數目上限。
Offset     | 數字 (預設值 = 0) | 要傳回的第一個結果索引。
orderby |   文字字串 | 用來排序結果的屬性名稱，後接選擇性排序次序 (預設值 = asc)："attrname[:(asc&#124;desc)]"。  如果未指定，則會以逐漸降低的自然對數機率傳回結果。
timeout  | 數字 (預設值 = 1000) | 逾時 (以毫秒為單位)。 只會傳回在逾時之前計算的結果。

使用 count 和 offset 參數，即可透過多個要求以累加方式取得大量結果。
  
## <a name="response-json"></a>回應 (JSON)
JSONPath|說明
----|----
$.expr | 要求中的 expr 參數。
$.entities | 0 個以上符合結構化查詢運算式的物件實體陣列。 
$.aborted | 如果要求逾時，則為 true。

每個實體都包含 logprob 值和所要求屬性的值。

## <a name="example"></a>範例
在學術刊物的範例中，下列要求會傳遞結構化查詢運算式(可能來自「解譯」要求的輸出)，並針對前 2 個相符實體擷取幾個屬性：

`http://<host>/evaluate?expr=Composite(Author.Name=='jaime teevan')&attributes=Title,Y,Author.Name,Author.Id&count=2`

回應包含前 2 個 ("count=2") 最可能相符的實體。  每個實體都會傳回標題、年度、作者名稱和作者識別碼屬性。  請注意複合屬性值的結構如何與資料檔案中所指定的方式進行比對。 

```json
{
  "expr": "Composite(Author.Name=='jaime teevan')",
  "entities": 
  [
    {
      "logprob": -6.645,
      "Ti": "personalizing search via automated analysis of interests and activities",
      "Y": 2005,
      "Author": [
        {
          "Name": "jaime teevan",
          "Id": 1968481722
        },
        {
          "Name": "susan t dumais",
          "Id": 676500258
        },
        {
          "Name": "eric horvitz",
          "Id": 1470530979
        }
      ]
    },
    {
      "logprob": -6.764,
      "Ti": "the perfect search engine is not enough a study of orienteering behavior in directed search",
      "Y": 2004,
      "Author": [
        {
          "Name": "jaime teevan",
          "Id": 1982462162
        },
        {
          "Name": "christine alvarado",
          "Id": 2163512453
        },
        {
          "Name": "mark s ackerman",
          "Id": 2055132526
        },
        {
          "Name": "david r karger",
          "Id": 2012534293
        }
      ]
    }
  ]
}
```
