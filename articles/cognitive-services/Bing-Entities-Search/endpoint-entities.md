---
title: 實體搜尋端點 | Microsoft Docs
description: 實體搜尋 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: article
ms.date: 12/05/2017
ms.author: v-gedod
ms.openlocfilehash: a2557c6000445544b3b47a05d7d356ccaa9928b4
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367855"
---
# <a name="entity-search-endpoint"></a>實體搜尋端點
**實體搜尋 API** 包含一個端點，該端點會根據查詢從 Web 傳回實體。

## <a name="endpoint"></a>端點
若要使用 **Bing API** 取得實體結果，請將 `GET` 要求傳送至下列端點。 使用標頭和 URL 參數以定義進一步的規格。

**端點：** 傳回 `?q=""` 所定義使用者搜尋查詢的相關實體。
```
 GET https://api.cognitive.microsoft.com/bing/v7.0/entities
```

如需關於標頭、參數、市場代碼、回應物件、錯誤的詳細資料，請參閱 [Bing 實體搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference) 參考。

## <a name="response-json"></a>回應 JSON
實體搜尋要求的回應包含作為 JSON 物件的結果。 如需結果的範例，請參閱[開始使用](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/quick-start)。

## <a name="next-steps"></a>後續步驟
**Bing** API 支援根據類型傳回結果的搜尋動作。 所有搜尋端點都會傳回 JSON 回應物件形式的結果。  所有端點皆支援查詢依照經度、緯度和搜尋半徑傳回特定語言及/或位置。

如需每個端點支援的參數完整相關資訊，請參閱每種類型的參考頁面。
如需使用實體搜尋 API 的範例，請參閱[開始使用](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/quick-start)和[調整大小和裁剪縮圖影像](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/resize-and-crop-thumbnails)。