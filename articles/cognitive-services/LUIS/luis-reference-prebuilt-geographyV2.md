---
title: Geography V2 預建實體 - LUIS 參考
titleSuffix: Azure Cognitive Services
description: 此文章包含 Language Understanding (LUIS) 中的 geographyV2 預建實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 3559bc02944f88f486104d4d9553f0c45a1f1754
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983407"
---
# <a name="geographyv2-entity"></a>GeographyV2 實體
預建 geographyV2 實體可偵測位置。 因為此實體已經定型，所以您不需要將包含 GeographyV2 的範例語句加入應用程式意圖。 英文[文化特性](luis-reference-prebuilt-entities.md)支援 GeographyV2 實體。

## <a name="subtypes"></a>子類型
地理位置都有子類型：

|子類型|目的|
|--|--|
|`poi`|感興趣的點|
|`city`|城市名稱|
|`countryRegion`|國家或地區名稱|
|`continent`|洲別名稱|
|`state`|州或省名稱|


## <a name="resolution-for-geographyv2-entity"></a>GeographyV2 實體解析
下列範例示範如何解析 **builtin.geographyV2** 實體。

```JSON
{
    "query": "Carol is visiting the sphinx in gizah egypt in africa before heading to texas",
    "topScoringIntent": {
        "intent": "None",
        "score": 0.8008023
    },
    "intents": [
        {
            "intent": "None",
            "score": 0.8008023
        }
    ],
    "entities": [
        {
            "entity": "the sphinx",
            "type": "builtin.geographyV2.poi",
            "startIndex": 18,
            "endIndex": 27
        },
        {
            "entity": "gizah",
            "type": "builtin.geographyV2.city",
            "startIndex": 32,
            "endIndex": 36
        },
        {
            "entity": "egypt",
            "type": "builtin.geographyV2.countryRegion",
            "startIndex": 38,
            "endIndex": 42
        },
        {
            "entity": "africa",
            "type": "builtin.geographyV2.continent",
            "startIndex": 47,
            "endIndex": 52
        },
        {
            "entity": "texas",
            "type": "builtin.geographyV2.state",
            "startIndex": 72,
            "endIndex": 76
        },
        {
            "entity": "carol",
            "type": "builtin.personName",
            "startIndex": 0,
            "endIndex": 4
        }
    ]
} 
```

## <a name="next-steps"></a>後續步驟

了解 [email](luis-reference-prebuilt-email.md)、[number](luis-reference-prebuilt-number.md) 及 [ordinal](luis-reference-prebuilt-ordinal.md) 實體相關資訊。 