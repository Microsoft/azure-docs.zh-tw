---
title: LUIS 預先建置的 dimension 實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的 dimension 實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: 3c923e7791c58255690100b04700577eb5c3f5dd
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39237259"
---
# <a name="dimension-entity"></a>Dimension 實體
預先建置的 dimension 實體可偵測各種維度類型，無論 LUIS 應用程式文化特色為何。 因為此實體已經定型，所以您不需要將包含維度的範例語句加入應用程式意圖。 [多種文化特色](luis-reference-prebuilt-entities.md)都支援 dimension 實體。 

## <a name="types-of-dimension"></a>維度類型

維度可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-NumbersWithUnit.yaml) GitHub 儲存機制管理


## <a name="resolution-for-dimension-entity"></a>解析 dimension 實體
下列範例顯示解析的 **builtin.dimension** 實體。

```JSON
{
  "query": "it takes more than 10 1/2 miles of cable and wire to hook it all up , and 23 computers.",
  "topScoringIntent": {
    "intent": "None",
    "score": 0.762141049
  },
  "intents": [
    {
      "intent": "None",
      "score": 0.762141049
    }
  ],
  "entities": [
    {
      "entity": "10 1/2 miles",
      "type": "builtin.dimension",
      "startIndex": 19,
      "endIndex": 30,
      "resolution": {
        "unit": "Mile",
        "value": "10.5"
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

了解 [email](luis-reference-prebuilt-email.md)、[number](luis-reference-prebuilt-number.md) 及 [ordinal](luis-reference-prebuilt-ordinal.md) 實體相關資訊。 