---
title: 學術知識 API 中的 JSON 搜尋語法 | Microsoft Docs
description: 了解您可以在 Microsoft 認知服務的學術知識 API 中使用的 JSON 搜尋語法。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: a4b9cf535dae60258d71c43bba6f9eec1444bd41
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367831"
---
# <a name="json-search-syntax"></a>JSON 搜尋語法

```javascript
/* Query Object:
   Suppose we have a query path /v0/e0/v1/e1/...
   A query object contains constraints to be applied to graph nodes in a path.
   Constraints are given in <"node identifier", {constraint object}> key-value pairs: 
*/
{
    /* query path can also be set in the query object */
    path: "/v0/e0/v1/e1/.../vn/",
    v0: { /* A Constraint Object */ },
    v1: { /* A Constraint Object */ },
}
```

查詢路徑中的節點名稱 (_v0、v1 ..._) 會當作可在查詢物件中參考的節點識別碼；路徑中的邊緣名稱 (_e0、e1 ..._) 代表對應邊緣的類型。 我們可以使用星號 _*_ 作為節點或邊緣名稱 (必須提供的起始節點除外)，以宣告這類元素沒有任何條件約束。 例如，查詢路徑 `/v0/*/v1/e1/*/` 會從圖表中擷取路徑，但不限制邊緣類型 _(v0、v1)_。 同時，此查詢沒有路徑目的地 (最後一個節點) 的條件約束。

當路徑只包含一個節點 (假設 _v0_) 時，查詢只會傳回滿足這些條件約束的所有實體。 套用至起始節點的條件約束物件稱為「起始詢物件」，其規格如下所示。

```javascript
/* Starting Query Object:
   Specifies constraints on the starting node
*/
{
    /* "match" operator searches for entities with the specified properties. 
       All properties defined in the graph schema be queried such as "Name" and "NormalizedTitle".
     */
    "match": { 
        "Name" : "some name",
        ...
    },
    /* "id" operator directly specifies the IDs of the starting nodes. 
       When it is present, the "match" operator is ignored. 
     */
    "id": [ id1, id2, id3... ],
    /* "type" operator limits results to a certain type of entities,
       for example, "Author" or "Paper".
     */
    "type": "Author",
    /* "select" operator pulls properties from the database and 
       returns them to the client.
     */
    "select": ["PaperRank", ...]
}
```

當路徑不只包含起始節點時，查詢處理器會遵循指定的路徑模式執行圖表周遊。 當它抵達節點時，將會觸發使用者指定的周遊動作，也就是在目前節點停止並返回，或是繼續探索圖表。 若未指定任何周遊動作，將會採用預設動作。 對於中繼節點，預設動作是繼續探索圖表。 對於路徑的最後一個節點，預設動作是停止並返回。 指定周遊動作的條件約束物件稱為「周遊動作物件」。 其規格如下所示：

```javascript
/* Traversal Action Object:
   Specifies graph traversal actions on a node (except for the starting node).
 */
{
    /* Set the continue condition here. */
    continue: { 
        /* simple property exact match */
        "property_key" : "property_value", 
        /* Advanced query operators */
        /* -- Numerical comparisons */
        "property_key_2" : { "gt" /* or simply ">" */ : 123 },
        /* -- Substring query */
        "property_key_3" : { "substring" : "456" },
        /* -- CellID query */
        "id": [ 111, 222, 333... ],
        /* -- Entity type query */
        "type": "cell_type",
        /* -- Property existance query */
        "has" : "property_key_4",
        /* -- Logical operators */
        /* ---- Note that, by default the operators are combined with AND semantics */
        
        /* -- OR operator */
        "or": {
          /* Query operators... */
        },
        /* -- NOT operator */
        "not": {
            /* Query operators... */
        }
    },
    /* Set the return condition here. */
    return: {
        /* Same operators as the continue object */
    },
    /* The selected properties to return. */
    select: ["property_key_1", ...]
}
```

json 搜尋查詢的 POST 本文應包含至少一個 path 模式。 周遊動作物件是選擇性的。 以下有兩個範例。

```JSON
{
  "path": "/series/ConferenceInstanceIDs/conference/FieldOfStudyIDs/field",
  "series": {
    "type": "ConferenceSeries",
    "FullName": "graph",
    "select": [ "FullName", "ShortName" ]
  },
  "conference": {
    "type": "ConferenceInstance",
    "select": [ "FullName" ]
  },
  "field": {
    "type": "FieldOfStudy",
    "select": [ "Name" ],
    "return": { "Name": { "substring" : "World Wide Web" } }
  }
}
```

```JSON
{
  "path": "/author/PaperIDs/paper",
  "author": {
    "type": "Author",
    "select": [ "DisplayAuthorName" ],
    "match": { "Name": "leslie lamport" }
  },
  "paper": {
    "type": "Paper",
    "select": [ "OriginalTitle", "CitationCount" ],
    "return": { "CitationCount": { "gt": 100 } }
  }
}
```

