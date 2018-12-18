---
title: 知識探索服務 API 中的解譯方法 | Microsoft Docs
description: 了解如何在認知服務的知識探索服務 (KES) API 中使用解譯方法。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: ef68d98dacf393abf8d030b9312217ea380947d2
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368063"
---
# <a name="interpret-method"></a>解譯方法
「解譯」方法會拿取自然語言查詢字串，然後根據文法和索引資料來傳回格式化的使用者意圖解譯。  若要提供互動式搜尋體驗，您可以呼叫這個方法，原因是每個字元都是由使用者輸入的，並且 complete 參數會設為 1 以啟用自動完成建議。

## <a name="request"></a>要求
`http://<host>/interpret?query=<query>[&<options>]`

Name|值| 說明
----|----|----
query    | 文字字串 | 使用者所輸入的查詢。  如果 complete 設為 1，則查詢會解譯為一個前置詞，以供產生查詢自動完成建議。        
完成 | 0 (預設值) 或 1 | 1 表示根據文法與索引資料產生自動完成建議。         
count    | 數字 (預設值 = 10) | 要傳回的解譯數目上限。         
Offset   | 數字 (預設值 = 0) | 要傳回的第一個解譯索引。  例如，count=2&offset=0 會傳回解譯 0 和 1。 count=2&offset=2 會傳回解譯 2 和 3。       
timeout  | 數字 (預設值 = 1000) | 逾時 (以毫秒為單位)。 只會傳回在逾時之前找到的解譯。

使用 count 和 offset 參數，即可透過多個要求以累加方式取得大量結果。

## <a name="response-json"></a>回應 (JSON)
JSONPath     | 說明
---------|---------
$.query |要求中的 query 參數。
$.interpretations   |0 個以上方式的陣列，用來針對文法比對輸入查詢。
$.interpretations[\*].logprob   |解譯的相對對數機率 (<= 0)。  值越高表示越有可能。
$.interpretations[\*].parse |XML 字串，可顯示如何解譯查詢的每個部分。
$.interpretations[\*].rules |在文法中定義的 1 或多個規則的陣列，而這些規則會在解譯期間叫用。
$.interpretations[\*].rules[\*].name    |規則的名稱。
$.interpretations[\*].rules[\*].output  |規則的語意輸出。
$.interpretations[\*].rules[\*].output.type |語意輸出的資料類型。
$.interpretations[\*].rules[\*].output.value|語意輸出的值。  
$.aborted | 如果要求逾時，則為 true。

### <a name="parse-xml"></a>剖析 XML
剖析 XML 會對 (已完成的) 查詢標註相關資訊，以說明其與文法中的規則和索引中的屬性有何比對結果。  以下是學術刊物領域的範例：

```xml
<rule name="#GetPapers">
  papers by 
  <attr name="academic#Author.Name" canonical="heungyeung shum">harry shum</attr>
  <rule name="#GetPaperYear">
    written in
    <attr name="academic#Year">2000</attr>
  </rule>
</rule>
```

`<rule>` 元素會將查詢中與其 `name` 屬性所指定規則相符的範圍分隔開來。  當剖析涉及文法中的規則參照時，它可能會呈巢狀。

`<attr>` 元素會將查詢中與其 `name` 屬性所指定索引屬性相符的範圍分隔開來。  當比對涉及輸入查詢中的同義字時，`canonical` 屬性會包含與索引中的同義字相符的標準值。

## <a name="example"></a>範例
在學術刊物的範例中，下列要求會針對前置詞查詢 "papers by jaime"傳回最多 2 個自動完成建議：

`http://<host>/interpret?query=papers by jaime&complete=1&count=2`

回應中包含前兩個 ("count=2") 最可能完成部分查詢 "papers by jaime" 的解譯："papers by jaime teevan" 和 "papers by jaime green"。  服務產生的查詢完成，而不是只考慮作者 "jaime" 的完全相符項目，因為要求指定了 "complete=1"。 請注意，標準值 "j l green" 經由同義字 "jamie green" 比對 (如剖析所示)。


```json
{
  "query": "papers by jaime",
  "interpretations": [
    {
      "logprob": -5.615,
      "parse": "<rule name=\"#GetPapers\">papers by <attr name=\"academic#Author.Name\">jaime teevan</attr></rule>",
      "rules": [
        {
          "name": "#GetPapers",
          "output": {
            "type": "query",
            "value": "Composite(Author.Name=='jaime teevan')"
          }
        }
      ]
    },
    {
      "logprob": -5.849,
      "parse": "<rule name=\"#GetPapers\">papers by <attr name=\"academic#Author.Name\" canonical=\"j l green\">jaime green</attr></rule>",
      "rules": [
        {
          "name": "#GetPapers",
          "output": {
            "type": "query",
            "value": "Composite(Author.Name=='j l green')"
          }
        }
      ]
    }
  ]
}
```  

當語意輸出的類型為「查詢」時 (正如同此範例)，您可以透過 expr 參數將 output.value 傳遞至[評估](evaluateMethod.md) API 來擷取相符的物件。

`http://<host>/evaluate?expr=Composite(AA.AuN=='jaime teevan')`
  
