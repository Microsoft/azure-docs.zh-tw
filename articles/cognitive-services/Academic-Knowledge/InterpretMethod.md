---
title: 學術知識 API 中的解譯方法 | Microsoft Docs
description: 使用解譯方法，根據 Microsoft 認知服務中的學術圖表資料和學術文法，傳回使用者查詢字串的格式化解譯。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/27/2017
ms.author: alch
ms.openlocfilehash: a46c792f14fabf6562666d1067ef880bd505741f
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367887"
---
# <a name="interpret-method"></a>解譯事件

**解譯** REST API 會採用使用者查詢字串 (也就是應用程式使用者所輸入的查詢)，並根據學術圖表資料和學術文法，傳回使用者意圖的格式化解譯。

若要提供互動式體驗，您可以在使用者輸入每個字元之後，重複呼叫這個方法。 在此情況下，您應該將 **complete** 參數設定為 1 來啟用自動完成建議。 如果您的應用程式不需要自動完成，您應該將 **complete** 參數設定為 0。

**REST 端點：**

    https://westus.api.cognitive.microsoft.com/academic/v1.0/interpret?

## <a name="request-parameters"></a>要求參數

Name     | 值 | 必要？  | 說明
---------|---------|---------|---------
**查詢**    | 文字字串 | yes | 使用者所輸入的查詢。  如果 complete 設為 1，則查詢會解譯為一個前置詞，以供產生查詢自動完成建議。        
**model**    | 文字字串 | 否  | 想要查詢的模型名稱。  目前，此值會預設為 latest。        
**complete** | 0 或 1 | 否<br>預設值：0  | 1 表示根據文法與圖表資料產生自動完成建議。         
**count**    | 數字 | 否<br>預設值︰10 | 要傳回的解譯數目上限。         
**offset**   | 數字 | 否<br>預設值：0  | 要傳回的第一個解譯索引。 例如，*count=2&offset=0* 會傳回解譯 0 和 1。 *count=2&offset=2* 會傳回解譯 2 和 3。       
**timeout**  | 數字 | 否<br>預設：1000 | 逾時 (以毫秒為單位)。 只會傳回在逾時之前找到的解譯。
<br>
  
## <a name="response-json"></a>回應 (JSON)
Name     | 說明
---------|---------
**查詢** |要求中的 *query* 參數。
**interpretations** |0 個或更多種根據文法比對使用者輸入的各種方法陣列。
**interpretations[x].logprob**  |解譯的相對自然對數機率。 較大的值表示越有可能。
**interpretations[x].parse**  |XML 字串，可顯示如何解譯查詢的每個部分。
**interpretations[x].rules**  |在文法中定義的 1 或多個規則的陣列，而這些規則會在解譯期間叫用。 在學術知識 API 中，一律會有 1 個規則。
**interpretations[x].rules[y].name**  |規則的名稱。
**interpretations[x].rules[y].output**  |規則的輸出。
**interpretations[x].rules[y].output.type** |規則輸出的資料類型。  在學術知識 API 中，這一律會是「查詢」。
**interpretations[x].rules[y].output.value**  |規則的輸出。 在學術知識 API 中，這是一個查詢運算式字串，可傳遞至評估和 calchistogram 方法。
**aborted** | 如果要求逾時，則為 true。

<br>
#### <a name="example"></a>範例：
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/interpret?query=papers by jaime&complete=1&count=2
 ```
<br>以下回應包含前兩個 (因為參數 *count=2*) 最可能完成部分使用者輸入 *papers by jaime* 的解譯：*papers by jaime teevan* 和 *papers by jaime green*。  服務產生的查詢完成，而不是只考慮作者 *jaime* 的完全相符項目，因為要求指定了 *complete=1*。 請注意，標準值 *j l green* 經由同義字 *jamie green* 比對 (如剖析所示)。


```JSON
{
  "query": "papers by jaime",
  "interpretations": [
    {
      "logprob": -12.728,
      "parse": "<rule name=\"#GetPapers\">papers by <attr name=\"academic#AA.AuN\">jaime teevan</attr></rule>",
      "rules": [
        {
          "name": "#GetPapers",
          "output": {
            "type": "query",
            "value": "Composite(AA.AuN=='jaime teevan')"
          }
        }
      ]
    },
    {
      "logprob": -12.774,
      "parse": "<rule name=\"#GetPapers\">papers by <attr name=\"academic#AA.AuN\" canonical=\"j l green\">jaime green</attr></rule>",
      "rules": [
        {
          "name": "#GetPapers",
          "output": {
            "type": "query",
            "value": "Composite(AA.AuN=='j l green')"
          }
        }
      ]
    }
  ]
}
```  
<br>若要擷取解譯的實體結果，請從**解譯** API 使用 *output.value*，並透過 *expr* 參數將它傳遞至**評估** API。 在此範例中，第一個解譯的查詢是： 
```
evaluate?expr=Composite(AA.AuN=='jaime teevan')
```
 