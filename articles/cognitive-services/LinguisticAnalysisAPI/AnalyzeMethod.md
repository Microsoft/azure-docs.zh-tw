---
title: 分析語言分析 API 中的方法 | Microsoft Docs
description: 如何使用語言分析 API 中的分析方法，來分析某些自然語言輸入。
services: cognitive-services
author: RichardSunMS
manager: wkwok
ms.service: cognitive-services
ms.component: linguistic-analysis
ms.topic: article
ms.date: 12/13/2016
ms.author: lesun
ms.openlocfilehash: b17a00f31845bfa05572dff7ca94e9a1ffd69586
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368078"
---
# <a name="analyze-method"></a>分析方法

**分析** REST API 可用來分析指定的自然語言輸入。
分析時可能涉及只尋找該輸入內的[句子和 Token](Sentences-and-Tokens.md)、尋找[詞性標記](POS-tagging.md)，或尋找[組成單位樹狀結構](Constituency-Parsing.md)。
您可以挑選相關分析器來指定您想要的結果。
若要列出所有可用的分析器，請參閱**[分析器](AnalyzersMethod.md)**。

請注意，您需要指定輸入字串的語言。

**REST 端點：**
```
https://westus.api.cognitive.microsoft.com/linguistics/v1.0/analyze
```
<br>

## <a name="request-parameters"></a>要求參數

Name | 類型 | 必要 | 說明
-----|-------|----------|------------
**language**    | 字串 | yes | 要用於分析的雙字母 ISO 語言代碼。 例如，英文是 "en"。
**analyzerIds** | 字串的清單 | yes | 要套用的分析器 GUID 清單。 如需詳細資訊，請參閱分析器文件。
**text**        | 字串 | yes | 要分析的原始輸入。 這可能是簡短字串 (例如，字組或片語)、完整句子，或完整的段落或語篇。

<br>
## <a name="response-json"></a>回應 (JSON)
分析輸出的陣列，要求中指定的每個屬性各有一個分析輸出。

結果外觀如下：

Name | 類型 | 說明
-----|------|--------------
analyzerId | 字串 | 所指定分析器的 GUID
結果 | 物件 | 分析器結果

請注意，結果的類型取決於輸入分析器類型。

### <a name="tokens-response-json"></a>Tokens 回應 (JSON)

Name | 類型 | 說明
-----|------|-------------
結果 | 句子物件的清單 | 在文字中識別的句子界限 |
result[x].Offset | int | 每個句子的起始字元位移 |
result[x].Len | int | 每個句子的長度 (以字元為單位) |
result[x].Tokens | Token 物件的清單 | 在句子中識別的 Token 界限 |
result[x].Tokens[y].Offset | int | Token 的起始字元位移 |
result[x].Tokens[y].Len | int | Token 的長度 (以字元為單位) |
result[x].Tokens[y].RawToken | 字串 | 在標準化之前，該 Token 內的字元 |
result[x].Tokens[y].NormalizedToken | 字串 | 字元的標準化形式，可於[剖析樹狀結構](Constituency-Parsing.md)中安全使用；例如，左括號字元 '(' 會變成 '-LRB-' |

輸入範例：`This is a test. Hello.' Hello.'
範例 JSON 回應：
```json
[
  {
    "Len": 15,
    "Offset": 0,
    "Tokens": [
      {
        "Len": 4,
        "NormalizedToken": "This",
        "Offset": 0,
        "RawToken": "This"
      },
      {
        "Len": 2,
        "NormalizedToken": "is",
        "Offset": 5,
        "RawToken": "is"
      },
      {
        "Len": 1,
        "NormalizedToken": "a",
        "Offset": 8,
        "RawToken": "a"
      },
      {
        "Len": 4,
        "NormalizedToken": "test",
        "Offset": 10,
        "RawToken": "test"
      },
      {
        "Len": 1,
        "NormalizedToken": ".",
        "Offset": 14,
        "RawToken": "."
      }
    ]
  },
  {
    "Len": 6,
    "Offset": 16,
    "Tokens": [
      {
        "Len": 5,
        "NormalizedToken": "Hello",
        "Offset": 16,
        "RawToken": "Hello"
      },
      {
        "Len": 1,
        "NormalizedToken": ".",
        "Offset": 21,
        "RawToken": "."
      }
    ]
  }
]
```


### <a name="pos-tags-response-json"></a>POS 標記回應 (JSON)

結果是字串清單的清單。
每個句子都會有 POS 標記清單，每個 Token 有一個 POS 標記。
若要尋找每個 POS 標記的對應 Token，建議您也要求 Token 化物件。

### <a name="constituency-tree-response-json"></a>組成單位樹狀結構回應 (JSON)

結果是字串的清單，在輸入中所找到的每個句子各有一個剖析樹狀結構。
剖析樹狀結構會以括號括住的形式來表示。

<br>

## <a name="example"></a>範例

`POST /analyze`

要求本文：JSON 承載
```json
{
  "language": "en",
  "analyzerIds": [
    "4FA79AF1-F22C-408D-98BB-B7D7AEEF7F04",
    "22A6B758-420F-4745-8A3C-46835A67C0D2" ],
  "text": "Hi, Tom! How are you today?" 
}
```

回應：JSON
```json
[
  {
    "analyzerId": "4FA79AF1-F22C-408D-98BB-B7D7AEEF7F04", 
    "result": [ ["NNP",",","NNP","."], ["WRB","VBP","PRP","NN","."] ]
  },
  {
    "analyzerId": "22A6B758-420F-4745-8A3C-46835A67C0D2", 
    "result":["(TOP (S (NNP Hi) (, ,) (NNP Tom) (. !)))","(TOP (SBARQ (WHADVP (WRB How)) (SQ (VP (VBP are)) (NP (PRP you)) (NN today) (. ?))))"]
  }
]
```

