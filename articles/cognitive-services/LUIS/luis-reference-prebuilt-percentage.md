---
title: LUIS 預先建置的百分比實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的百分比實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: d445dbf69e3d2163b5d44b894f8795d41fbd34e3
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39238959"
---
# <a name="percentage-entity"></a>百分比實體
百分比數字可顯示為分數 `3 1/2` 或百分比 `2%`。 由於此實體已經定型，因此您不需要將包含百分比的範例語句加入至應用程式意圖。 [多種文化特色](luis-reference-prebuilt-entities.md)都支援百分比實體。 

## <a name="types-of-percentage"></a>百分比類型
百分比可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-Numbers.yaml#L114) GitHub 存放庫來管理

## <a name="resolution-for-prebuilt-percentage-entity"></a>解析預先建置的百分比實體
下列範例說明 **builtin.percentage** 實體的解析。

```JSON
{
  "query": "set a trigger when my stock goes up 2%",
  "topScoringIntent": {
    "intent": "SetTrigger",
    "score": 0.971157849
  },
  "intents": [
    {
      "intent": "SetTrigger",
      "score": 0.971157849
    }
  ],
  "entities": [
    {
      "entity": "2%",
      "type": "builtin.percentage",
      "startIndex": 36,
      "endIndex": 37,
      "resolution": {
        "value": "2%"
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

了解 [ordinal](luis-reference-prebuilt-ordinal.md)、[number](luis-reference-prebuilt-number.md) 和 [temperature](luis-reference-prebuilt-temperature.md) 實體相關資訊。 