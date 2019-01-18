---
title: 圖表搜尋方法 - 學術知識 API
titlesuffix: Azure Cognitive Services
description: 使用學術知識 API 中的圖表搜尋方法，根據特定圖表模式傳回一組學術實體。
services: cognitive-services
author: alch-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: conceptual
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: 83f29106d72f564f894c968102b703ab6bb5d8c2
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48902373"
---
# <a name="graph-search-method"></a>圖表搜尋方法

**圖表搜尋** REST API 用來傳回一組以指定的圖表模式為基礎的學術實體。  回應是一組符合使用者所指定條件約束的圖表路徑。 圖表路徑是圖表節點和邊緣的交錯序列，其形式為 _v0、e0、v1、e1、...、vn_，其中 _v0_ 是路徑的起始節點。
<br>

**REST 端點：**  
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?
```   
<br>

## <a name="request-parameters"></a>要求參數  
名稱     | 值 | 必要？  | 說明
-----------|-----------|---------|--------
**mode**       | 文字字串 | 是 | 想要使用的模型名稱。 值為 *json* 或 *lambda*。

必須透過 HTTP POST 要求呼叫圖表搜尋方法。 POST 要求應包含內容類型標頭：**application/json**。

##### <a name="json-search"></a>JSON 搜尋 

對於 json 搜尋，POST 本文是 JSON 物件。 JSON 物件描述具有使用者所指定條件約束的路徑模式 (請參閱適用於 json 搜尋的 [JSON 物件規格](JSONSearchSyntax.md))。


##### <a name="lambda-search"></a>Lambda 搜尋

對於 lambda 搜尋，POST 本文是純文字字串。 POST 本文是 LIKQ lambda 查詢字串，這是單一的 C# 陳述式 (請參閱適用於 lambda 搜尋的[查詢字串規格](LambdaSearchSyntax.md))。 

<br>
## <a name="response-json"></a>回應 (JSON)
名稱 | 說明
-------|-----   
**results** | 一個陣列，內含 0 或多個符合查詢運算式的實體。 每個實體都包含要求的屬性值。 如果已成功處理要求，就會呈現這個欄位。
**error** | HTTP 狀態碼。 如果要求失敗，就會呈現此欄位。
**message** | 錯誤訊息。 如果要求失敗，就會呈現此欄位。

如果無法在 _800 毫秒_ 內查詢處理，則會傳回 _逾時_ 錯誤。 

<br>
#### <a name="example"></a>範例：

##### <a name="json-search"></a>JSON 搜尋
```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?mode=json
```
<br>
對於 json 搜尋，如果我們想要取得標題包含 "graph engine" 並由 "bin shao" 撰寫的論文，我們可以指定查詢，如下所示。

```JSON
{
  "path": "/paper/AuthorIDs/author",
  "paper": {
    "type": "Paper",
    "NormalizedTitle": "graph engine",
    "select": [
      "OriginalTitle"
    ]
  },
  "author": {
    "return": {
      "type": "Author",
      "Name": "bin shao"
    }
  }
}
```

查詢的輸出是圖表路徑陣列。 圖表路徑是節點物件的陣列，而這些物件會對應至查詢路徑中指定的節點。 這些節點物件有至少一個 CellID 屬性，表示實體識別碼。 透過[*周遊動作物件*](JSONSearchSyntax.md)的 select 運算子來指定屬性名稱，即可擷取其他屬性。

```JSON
{
  "Results": [
    [
      {
        "CellID": 2160459668,
        "OriginalTitle": "Trinity: a distributed graph engine on a memory cloud"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 2171539317,
        "OriginalTitle": "A distributed graph engine for web scale RDF data"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 2411554868,
        "OriginalTitle": "A distributed graph engine for web scale RDF data"
      },
      {
        "CellID": 2093502026
      }
    ],
    [
      {
        "CellID": 73304046,
        "OriginalTitle": "The Trinity graph engine"
      },
      {
        "CellID": 2093502026
      }
    ]
  ]
}
 ```

##### <a name="lambda-search"></a>Lambda 搜尋 

```
https://westus.api.cognitive.microsoft.com/academic/v1.0/graph/search?mode=lambda
```
<br>
對於 lambda 搜尋，如果我們想要取得指定論文的作者識別碼，我們可以撰寫如下所示的查詢。

```
MAG.StartFrom(@"{
    type  : ""Paper"",
    match : {
        NormalizedTitle : ""trinity: a distributed graph engine on a memory cloud""
    }
}").FollowEdge("AuthorIDs").VisitNode(Action.Return)
```

lambda 搜尋查詢的輸出也是圖表路徑陣列：

```JSON
{
  "Results": [
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2142490828
      }
    ],
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2116756368
      }
    ],
    [
      {
        "CellID": 2160459668
      },
      {
        "CellID": 2093502026
      }
    ]
  ]
}
```
 
## <a name="graph-schema"></a>圖表結構描述

圖表結構描述適合用於撰寫圖表搜尋查詢。 它會顯示在下圖中。

![Microsoft 學術圖表結構描述](./Images/AcademicGraphSchema.png)
