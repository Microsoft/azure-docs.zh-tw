---
title: LUIS 預先建置的 number 實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的 number 實體資訊。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: 4d14a3490d87693a4785111035b62e7ae59ee669
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041841"
---
# <a name="number-entity"></a>Number 實體
數值用來量化、表達及描述各項資訊的方式眾多。 本文只涵蓋部分可能的範例。 LUIS 可解譯使用者語句的變化，並傳回一致的數值。 因為此實體已經定型，所以您不需要將包含數字的範例語句加入應用程式意圖。 

## <a name="types-of-number"></a>數字類型
數字可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-Numbers.yaml) GitHub 儲存機制管理

## <a name="examples-of-number-resolution"></a>數字解析範例

| 語句        | 實體   | 解決方案 |
| ------------- |:----------------:| --------------:|
| ```one thousand times```  | ```"one thousand"``` |   ```"1000"```      | 
| ```1,000 people```        | ```"1,000"```    |   ```"1000"```      |
| ```1/2 cup```         | ```"1 / 2"```    |    ```"0.5"```      |
|  ```one half the amount```     | ```"one half"```     |    ```"0.5"```      |
| ```one hundred fifty orders``` | ```"one hundred fifty"``` | ```"150"``` |
| ```one hundred and fifty books``` | ```"one hundred and fifty"``` | ```"150"```|
| ```a grade of one point five```| ```"one point five"``` |  ```"1.5"``` |
| ```buy two dozen eggs```    | ```"two dozen"``` | ```"24"``` |


LUIS 會在所傳回 JSON 回應的 `resolution` 欄位中，包含 **`builtin.number`** 實體的辨識值。

## <a name="resolution-for-prebuilt-number"></a>解析預先建置的數字
下列範例顯示來自 LUIS 的 JSON 回應，其中包含「兩打」語句的值 24。

```JSON
{
  "query": "order two dozen eggs",
  "topScoringIntent": {
    "intent": "OrderFood",
    "score": 0.105443209
  },
  "intents": [
    {
      "intent": "None",
      "score": 0.105443209
    },
    {
      "intent": "OrderFood",
      "score": 0.9468431361
    },
    {
      "intent": "Help",
      "score": 0.000399122015
    },
  ],
  "entities": [
    {
      "entity": "two dozen",
      "type": "builtin.number",
      "startIndex": 6,
      "endIndex": 14,
      "resolution": {
        "value": "24"
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

了解 [currency](luis-reference-prebuilt-currency.md)、[ordinal](luis-reference-prebuilt-ordinal.md) 及 [percentage](luis-reference-prebuilt-percentage.md) 相關資訊。 