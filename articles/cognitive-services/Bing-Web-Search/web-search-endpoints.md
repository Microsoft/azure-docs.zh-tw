---
title: Web 搜尋端點 | Microsoft Docs
description: Web 搜尋 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 11/28/2017
ms.author: v-gedod
ms.openlocfilehash: 72fbe1a0eb4379ad032e649f7299e37a0214adfb
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367946"
---
# <a name="web-search-endpoint"></a>Web 搜尋端點
**Web 搜尋 API** 會傳回網頁、新聞、影像、影片和[實體](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/search-the-web)。 實體包含人員、地點或主題的摘要資訊。
## <a name="endpoint"></a>端點
若要使用 Bing API 取得 Web 搜尋結果，請將 `GET` 要求傳送至下列端點。 標頭和 URL 參數可定義進一步的規格。

**端點**：傳回 `?q=""` 所定義使用者搜尋查詢的相關 Web 結果。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/search
```
端點：如需標頭、參數、市場代碼、回應物件、錯誤等詳細資料，請參閱 [Bing Web API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference) 參考。

## <a name="response-json"></a>回應 JSON
Web 搜尋要求的回應包含作為 JSON 物件的結果。 剖析結果時，需要可處理各類型元素的程序。 如需相關範例，請參閱[教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-web-search/tutorial-bing-web-search-single-page-app)和[原始程式碼](https://docs.microsoft.com/azure/cognitive-services/bing-web-search/tutorial-bing-web-search-single-page-app-source)。

## <a name="next-steps"></a>後續步驟
**Bing** API 支援根據類型傳回結果的搜尋動作。 所有搜尋端點都會傳回 JSON 回應物件形式的結果。  所有端點皆支援查詢依照經度、緯度和搜尋半徑傳回特定語言及/或位置。

如需每個端點支援的參數完整相關資訊，請參閱每種類型的參考頁面。
如需透過範例了解使用 Web 搜尋 API 的基本要求，請參閱[搜尋 Web 快速入門](https://docs.microsoft.com/azure/cognitive-services/bing-web-search/search-the-web)。