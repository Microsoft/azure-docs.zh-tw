---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: luis
ms.topic: include
ms.custom: include file
ms.date: 08/16/2018
ms.author: diberry
ms.openlocfilehash: a1b0afce31d7202c38b049addf546350ff347719
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47044140"
---
範例語句檔案 **utterances.json** 會遵循特定格式。 

`text` 欄位包含範例語句的文字。 `intentName` 欄位必須對應到 LUIS 應用程式中的現有意圖名稱。 `entityLabels` 是必填欄位。 如果您不想要標示任何實體，請提供空白陣列。

如果 entityLabels 陣列不是空的，`startCharIndex` 和 `endCharIndex` 必須標示 `entityName` 欄位中參考的實體。 索引是以零為起始，這表示頂端範例中的 6 是指 Seattle 的 "S"，而不是大寫 S 之前的空格。如果您在文字中的某個空格處開始或結束標記，則用於新增語句的 API 呼叫會失敗。

```JSON
[
  {
    "text": "go to Seattle today",
    "intentName": "BookFlight",
    "entityLabels": [
      {
        "entityName": "Location::LocationTo",
        "startCharIndex": 6,
        "endCharIndex": 12
      }
    ]
  },
  {
    "text": "purple dogs are difficult to work with",
    "intentName": "BookFlight",
    "entityLabels": []
  }
]
```