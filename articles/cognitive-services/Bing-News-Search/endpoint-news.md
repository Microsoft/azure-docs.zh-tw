---
title: Bing 新聞搜尋端點
titlesuffix: Azure Cognitive Services
description: 新聞搜尋 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: v-gedod
ms.openlocfilehash: 0ed8b9048c04c4aff5214cea697810a0c573559e
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48800540"
---
# <a name="bing-news-search-endpoints"></a>Bing 新聞搜尋端點
**新聞搜尋 API** 會傳回新聞文章、網頁、影像、影片和[實體](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/search-the-web)。 實體包含人員、地點或主題的摘要資訊。
## <a name="endpoints"></a>端點
若要使用 Bing API 取得新聞搜尋結果，請將 `GET` 要求傳送至下列其中一個端點。 標頭和 URL 參數可定義進一步的規格。

**端點 1：** 根據使用者的搜尋查詢傳回新聞項目。 如果搜尋查詢是空的，則呼叫會傳回熱門新聞文章。 查詢 `?q=""` 選項也可搭配 `/news` URL 使用。 如需了解可用性，請參閱[支援的國家/地區與市場](language-support.md#supported-markets-for-news-search-endpoint)。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search
```

**端點 2：** 依照類別傳回熱門新聞項目。 您可以使用 `category=business``category=sports` 或 `category=entertainment`，特別要求熱門商業、運動或娛樂文章。  `category` 參數只能搭配 `/news` URL 使用。 指定類別時有一些正式需求；請參閱[查詢參數](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#query-parameters)文件中的 `category`。 如需了解可用性，請參閱[支援的國家/地區與市場](language-support.md#supported-markets-for-news-endpoint)。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/news  
```

**端點 3：** 傳回社交網路上目前發燒的新聞主題。 若包含 `/trendingtopics` 選項，Bing 搜尋會忽略其他幾個參數，例如 `freshness` 和 `?q=""`。 如需了解可用性，請參閱[支援的國家/地區與市場](language-support.md#supported-markets-for-news-trending-endpoint)。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/trendingtopics
```

如需關於標頭、參數、市場代碼、回應物件、錯誤的詳細資料，請參閱 [Bing 新聞搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference) 參考。
## <a name="response-json"></a>回應 JSON
新聞搜尋要求的回應包含作為 JSON 物件的結果。 剖析結果時，需要可處理各類型元素的程序。 如需相關範例，請參閱[教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-news-search/tutorial-bing-news-search-single-page-app)和[原始程式碼](https://docs.microsoft.com/azure/cognitive-services/bing-news-search/tutorial-bing-news-search-single-page-app-source)。

## <a name="next-steps"></a>後續步驟
**Bing** API 支援根據類型傳回結果的搜尋動作。 所有搜尋端點會傳回作為 JSON 回應物件的結果。  所有端點均可支援依照經度、緯度和搜尋半徑傳回特定語言及/或位置的查詢。

如需每個端點支援的參數完整相關資訊，請參閱每種類型的參考頁面。
如需透過範例了解使用新聞搜尋 API 的基本要求，請參閱 [Bing 新聞搜尋快速入門](https://docs.microsoft.com/azure/cognitive-services/bing-news-search)。
