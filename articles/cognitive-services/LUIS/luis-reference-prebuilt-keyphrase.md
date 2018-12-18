---
title: LUIS 預先建置的 keyPhrase 實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的 keyPhrase 實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 07/09/2018
ms.author: diberry
ms.openlocfilehash: 904f327dfe20e3d0864cbf355fd10237659879ee
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39238595"
---
# <a name="keyphrase-entity"></a>keyPhrase 實體
keyPhrase 會從語句中擷取各種不同的關鍵片語。 您不需要將包含 keyPhrase 的範例語句新增到應用程式。 keyPhrase 實體會在[多種文化特性](luis-supported-languages.md#languages-supported)中受到支援，以作為[文字分析](../text-analytics/overview.md)功能的一部分。 

## <a name="resolution-for-prebuilt-keyphrase-entity"></a>解析預先建置的 keyPhrase 實體
下列範例示範如何解析 **builtin.keyPhrase** 實體。

```JSON
{
  "query": "where is the educational requirements form for the development and engineering group",
  "topScoringIntent": {
    "intent": "GetJobInformation",
    "score": 0.182757929
  },
  "entities": [
    {
      "entity": "development",
      "type": "builtin.keyPhrase",
      "startIndex": 51,
      "endIndex": 61
    },
    {
      "entity": "educational requirements",
      "type": "builtin.keyPhrase",
      "startIndex": 13,
      "endIndex": 36
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

了解 [percentage](luis-reference-prebuilt-percentage.md)、[number](luis-reference-prebuilt-number.md) 及 [age](luis-reference-prebuilt-age.md) 實體相關資訊。 