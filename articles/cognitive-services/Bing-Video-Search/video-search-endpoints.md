---
title: 影片搜尋端點 - Bing 影片搜尋
titlesuffix: Azure Cognitive Services
description: Bing 影片搜尋 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: conceptual
ms.date: 12/04/2017
ms.author: rosh
ms.openlocfilehash: c153f577f76944d22f9a1b0fb4b24d332d2a02c8
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220766"
---
# <a name="video-search-endpoints"></a>影片搜尋端點
**影片搜尋 API** 包含三個端點。  端點 1 會根據查詢從 Web 傳回影片。 端點 2 會根據 `modules` URL 參數，傳回影片相關見解。  端點 3 會傳回發燒影像。

## <a name="endpoints"></a>端點
若要使用 Bing API 取得影片結果，請將 `GET` 要求傳送至下列其中一個端點。 使用標頭和 URL 參數以定義進一步的規格。

**端點1：** 傳回 `?q=""` 所定義使用者搜尋查詢的相關影片。
``` 
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search
```

**端點 2：** 傳回影片的相關見解，例如相關影片。 包含`modules`[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference#query-parameters)，這是要求的見解清單 (以逗號分隔)。
``` 
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/details
```

**端點 3：** 傳回影片，這些影片是以他人所提出的搜尋要求為基礎的發燒影片。 影片可分成不同類別，例如，根據值得注意的人員或事件。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/trending
```

如需關於標頭、參數、市場代碼、回應物件、錯誤的詳細資料，請參閱 [Bing 影片搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference) 參考。
## <a name="response-json"></a>回應 JSON
影片搜尋要求的回應包含作為 JSON 物件的結果。 如需結果剖析的範例，請參閱[教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-video-search/tutorial-bing-video-search-single-page-app)和[原始程式碼](https://docs.microsoft.com/azure/cognitive-services/bing-video-search/tutorial-bing-video-search-single-page-app-source)。

## <a name="next-steps"></a>後續步驟
**Bing** API 支援根據類型傳回結果的搜尋動作。 所有搜尋端點會傳回作為 JSON 回應物件的結果。  所有端點均可支援依照經度、緯度和搜尋半徑傳回特定語言及/或位置的查詢。

如需每個端點支援的參數完整相關資訊，請參閱每種類型的參考頁面。
如需透過範例了解使用影片搜尋 API 的基本要求，請參閱[影片搜尋快速入門](https://docs.microsoft.com/azure/cognitive-services/bing-video-search)。