---
title: LUIS 預先建置的 currency 實體參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中預先建置的 currency 實體資訊。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: 89350ad6a49ceef8169d1693dafdedf702992504
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033884"
---
# <a name="currency-entity"></a>Currency 實體
預先建置的 currency 實體可偵測許多面額與國家的貨幣，無論 LUIS 應用程式文化特色為何。 因為此實體已經定型，所以您不需要將包含貨幣的範例語句加入應用程式意圖。 [多種文化特色](luis-reference-prebuilt-entities.md)都支援 currency 實體。 

## <a name="types-of-currency"></a>貨幣類型
貨幣可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-NumbersWithUnit.yaml#L26) GitHub 儲存機制管理

## <a name="resolution-for-currency-entity"></a>解析 currency 實體
下列範例顯示解析的 **builtin.currency** 實體。

```JSON
{
  "query": "search for items under $10.99",
  "topScoringIntent": {
    "intent": "SearchForItems",
    "score": 0.926173568
  },
  "intents": [
    {
      "intent": "SearchForItems",
      "score": 0.926173568
    },
    {
      "intent": "None",
      "score": 0.07376878
    }
  ],
  "entities": [
    {
      "entity": "$10.99",
      "type": "builtin.currency",
      "startIndex": 23,
      "endIndex": 28,
      "resolution": {
        "unit": "Dollar",
        "value": "10.99"
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

了解 [datetimeV2](luis-reference-prebuilt-datetimev2.md)、[dimension](luis-reference-prebuilt-dimension.md) 及 [email](luis-reference-prebuilt-email.md) 實體相關資訊。 