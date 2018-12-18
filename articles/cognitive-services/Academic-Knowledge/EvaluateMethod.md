---
title: 學術知識 API 中的評估方法 | Microsoft Docs
description: 使用評估方法，根據 Microsoft 認知服務中的查詢運算式，傳回一組學術實體。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/27/2017
ms.author: alch
ms.openlocfilehash: 3005ae1f6df042a49db086de4982d8206f6938a4
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367843"
---
# <a name="evaluate-method"></a>評估模型

**評估** REST API 用來傳回一組以查詢運算式為基礎的學術實體。
<br>

**REST 端點：**  
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/evaluate? 
```   
<br>
## <a name="request-parameters"></a>要求參數  
Name     | 值 | 必要？  | 說明
-----------|-----------|---------|--------
**expr**       | 文字字串 | yes | 可指定應傳回哪些實體的查詢運算式。
**model**      | 文字字串 | 否  | 想要查詢的模型名稱。  目前，此值會預設為 latest。        
**attributes** | 文字字串 | 否<br>預設值：Id | 以逗號分隔的清單，可指定回應中包含的屬性值。 屬性名稱區分大小寫。
**count**        | 數字 | 否<br>預設值：10 | 要傳回的結果數目。
**offset**     | 數字 |   否<br>預設值：0    | 要傳回的第一個結果索引。
**orderby** |   文字字串 | 否<br>預設值：藉由減少 prob | 用來排序實體的屬性名稱。 (選擇性) 可以指定遞增/遞減。 格式為：name:asc 或 name:desc。
  
 <br>
## <a name="response-json"></a>回應 (JSON)
Name | 說明
-------|-----   
**expr** |  要求中的 *expr* 參數。
**entities** |  一個陣列，內含 0 或多個符合查詢運算式的實體。 每個實體都包含自然對數機率值和其他要求的屬性值。
**aborted** | 如果要求逾時，則為 true。

<br>
#### <a name="example"></a>範例：
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/evaluate?expr=
Composite(AA.AuN=='jaime teevan')&count=2&attributes=Ti,Y,CC,AA.AuN,AA.AuId
```
<br>通常會從**解譯**方法的回應中取得運算式。  但是您也可以自行撰寫查詢運算式 (請參閱[查詢運算式語法](QueryExpressionSyntax.md))。  
  
使用 count 和 offset 參數，可能會取得大量結果，但不會傳送會導致大量 (且可能很緩慢) 回應的單一要求。  在此範例中，此要求使用了**解譯** API 回應中第一個解譯的運算式作為 expr 值。 count=2 參數會指定正在要求 2 個實體結果。 而 attributes=Ti,Y,CC,AA.AuN,AA.AuId 參數會指出每個結果會要求標題、年度、引用次數、作者名稱和作者識別碼。  請參閱[實體屬性](EntityAttributes.md)以取得屬性清單。
  
```JSON
{
  "expr": "Composite(AA.AuN=='jaime teevan')",
  "entities": 
  [
    {
      "logprob": -15.08,
      "Ti": "personalizing search via automated analysis of interests and activities",
      "Y": 2005,
      "CC": 372,
      "AA": [
        {
          "AuN": "jaime teevan",
          "AuId": 1968481722
        },
        {
          "AuN": "susan t dumais",
          "AuId": 676500258
        },
        {
          "AuN": "eric horvitz",
          "AuId": 1470530979
        }
      ]
    },
    {
      "logprob": -15.389,
      "Ti": "the perfect search engine is not enough a study of orienteering behavior in directed search",
      "Y": 2004,
      "CC": 237,
      "AA": [
        {
          "AuN": "jaime teevan",
          "AuId": 1982462162
        },
        {
          "AuN": "christine alvarado",
          "AuId": 2163512453
        },
        {
          "AuN": "mark s ackerman",
          "AuId": 2055132526
        },
        {
          "AuN": "david r karger",
          "AuId": 2012534293
        }
      ]
    }
  ]
}
 ```
 
