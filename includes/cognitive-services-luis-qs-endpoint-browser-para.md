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
ms.openlocfilehash: e507a7c45e286473abe9b9e4365e80fb29eba2a4
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47043985"
---
若要了解 LUIS 預測端點所傳回的內容，請在網頁瀏覽器中檢視預測結果。 若要查詢公用應用程式，您需要自己的金鑰和應用程式識別碼。 公用 IoT 應用程式識別碼 (`df67dcdb-c37d-46af-88e1-8b97951ca1c2`) 會提供為第一個步驟中 URL 的一部分。

**GET** 端點要求的 URL 格式為：

```JSON
https://<region>.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<YOUR-KEY>&q=<user-utterance>
```

1. 公用 IoT 應用程式的端點採用以下格式：`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2?subscription-key=<YOUR_KEY>&q=turn on the bedroom light`

    複製此 URL，並以您的金鑰取代 `<YOUR_KEY>` 值。

2. 將 URL 貼入瀏覽器視窗，然後按 Enter。 瀏覽器會顯示 JSON 結果，指出 LUIS 偵測到 `HomeAutomation.TurnOn` 意圖 (最高意圖) 和 `HomeAutomation.Room` 實體 (值為 `bedroom`)。

    ```JSON
    {
      "query": "turn on the bedroom light",
      "topScoringIntent": {
        "intent": "HomeAutomation.TurnOn",
        "score": 0.809439957
      },
      "entities": [
        {
          "entity": "bedroom",
          "type": "HomeAutomation.Room",
          "startIndex": 12,
          "endIndex": 18,
          "score": 0.8065475
        }
      ]
    }
    ```

3. 將 URL 中的 `q=` 參數值變更為 `turn off the living room light`，然後按 Enter。 結果現在指出 LUIS 偵測到 `HomeAutomation.TurnOff` 意圖 (最高意圖) 和 `HomeAutomation.Room` 實體 (值為 `living room`)。 

    ```JSON
    {
      "query": "turn off the living room light",
      "topScoringIntent": {
        "intent": "HomeAutomation.TurnOff",
        "score": 0.984057844
      },
      "entities": [
        {
          "entity": "living room",
          "type": "HomeAutomation.Room",
          "startIndex": 13,
          "endIndex": 23,
          "score": 0.9619945
        }
      ]
    }
    ```
