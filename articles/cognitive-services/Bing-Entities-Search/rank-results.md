---
title: 使用排名顯示回應 | Microsoft Docs
description: 示範如何使用排名來顯示 Bing 實體搜尋 API 傳回的回應。
services: cognitive-services
author: v-jerkin
manager: ehansen
ms.assetid: BBF87972-B6C3-4910-BB52-DE90893F6C71
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: article
ms.date: 12/12/2017
ms.author: v-jerkin
ms.openlocfilehash: 53354c0f78419a37e8896bb4d00e0d7aebf32203
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37059986"
---
# <a name="using-ranking-to-display-results"></a>使用排名顯示結果  

每個實體搜尋回應包含 [RankingResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankingresponse) 回應，類似於 Bing Web 搜尋回應，指定您必須顯示搜尋結果的方式。 排名回應會將結果分組為 pole、mainline 和 sidebar 內容。 pole 結果是最重要或顯著的結果，應該優先顯示。 如果您未以傳統 mainline 和 sidebar 格式顯示其餘結果，您必須提供比 sidebar 內容更高的 mainline 內容可見度。 
  
在每個群組內，[項目](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankinggroup-items)陣列會識別內容必須出現的順序。 每個項目會提供兩種方式來識別回應內的結果。  
  
-   `answerType` 和 `resultIndex` - `answerType` 欄位會識別回應 (實體或位置)，而 `resultIndex` 會識別回應內的結果 (例如一個實體)。 索引是以零為起始。  
  
-   `value` - `value` 欄位所包含的識別碼會符合回應或回應內結果的識別碼。 回應或結果其中一個 (而非兩者) 會包含識別碼。  
  
使用識別碼需要您比對排名識別碼與回應或其中一個結果的識別碼。 如果回應物件包含 `id` 欄位，則會同時顯示回應的所有結果。 例如，如果 `Entities` 物件包含 `id` 欄位，則會同時顯示所有實體發行項。 如果 `Entities` 物件不包含 `id` 欄位，則每個實體會包含 `id` 欄位，且排名回應會混合實體與位置結果。  
  
使用 `answerType` 和 `resultIndex` 是一個兩步驟程序。 首先，您會使用 `answerType` 識別包含要顯示結果的回應。 然後，您會使用 `resultIndex` 為回應的結果編製索引，以顯示結果。 (`answerType` 值是 [SearchResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#searchresponse) 物件中的欄位名稱)。如果您應該同時顯示回應的所有結果，排名回應項目不會包含 `resultIndex` 欄位。

## <a name="ranking-response-example"></a>排名回應範例

以下顯示範例 [RankingResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankingresponse)。
  
```json
{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "Jimi Hendrix"
  },
  "entities": { ... },
  "rankingResponse": {
    "sidebar": {
      "items": [
        {
          "answerType": "Entities",
          "resultIndex": 0,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#Entities.0"
          }
        },
        {
          "answerType": "Entities",
          "resultIndex": 1,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#Entities.1"
          }
        }
      ]
    }
  }
}
```

根據此排名回應，sidebar 會顯示與 Jimi Hendrix 相關的兩個實體結果。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 實體搜尋教學課程](tutorial-bing-entities-search-single-page-app.md)
