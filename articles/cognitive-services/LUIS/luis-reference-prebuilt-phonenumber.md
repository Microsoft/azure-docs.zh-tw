---
title: LUIS 預先建置的 phonenumber 實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的 phonenumber 實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: 1ae14f72f0dc610b9399e675f49ef5fff51a6965
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39238986"
---
# <a name="phonenumber-entity"></a>Phonenumber 實體
`phonenumber` 實體會擷取各種不同的電話號碼，包括國碼/區碼。 因為此實體已經定型，所以您不需要將範例語句加入應用程式。 只有 `en-us` 文化特色才支援 `phonenumber` 實體。 

## <a name="types-of-phonenumber"></a>電話號碼類型
電話號碼可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/Base-PhoneNumbers.yaml) GitHub 儲存機制管理

## <a name="resolution-for-prebuilt-phonenumber-entity"></a>解析預先建置的 phonenumber 實體
下列範例顯示解析的 **builtin.phonenumber** 實體。

```JSON
{
  "query": "my mobile is 00 44 161 1234567",
  "topScoringIntent": {
    "intent": "None",
    "score": 0.8448457
  },
  "intents": [
    {
      "intent": "None",
      "score": 0.8448457
    }
  ],
  "entities": [
    {
      "entity": "00 44 161 1234567",
      "type": "builtin.phonenumber",
      "startIndex": 13,
      "endIndex": 29,
      "resolution": {
        "value": "00 44 161 1234567"
      }
    }
  ]
}
```


## <a name="next-steps"></a>後續步驟

了解 [percentage](luis-reference-prebuilt-percentage.md)、[number](luis-reference-prebuilt-number.md) 及 [temperature](luis-reference-prebuilt-temperature.md) 實體相關資訊。 