---
title: 影像搜尋端點 | Microsoft Docs
description: 影像搜尋 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: article
ms.date: 11/30/2017
ms.author: v-gedod
ms.openlocfilehash: 0d5f28bfdb45b27b04df068f75e8a20d0235d12b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367854"
---
# <a name="image-search-endpoints"></a>影像搜尋端點
**影像搜尋 API** 包含三個端點。  端點 1 會根據查詢從 Web 傳回影像。 端點 2 會傳回 [ImageInsights](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imageinsightsresponse)。  端點 3 會傳回發燒影像。
## <a name="endpoints"></a>端點
若要使用 Bing API 取得影像結果，請將要求傳送至下列其中一個端點。 使用標頭和 URL 參數以定義進一步的規格。

**端點 1：** 傳回 `?q=""` 所定義使用者搜尋查詢的相關影像。
``` 
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search
```

**端點 2：** 使用 `GET` 或 `POST`，傳回影像的相關見解。
``` 
 GET or POST https://api.cognitive.microsoft.com/bing/v7.0/images/details
```
GET 要求會傳回影像相關見解，例如包含影像的網頁。 透過 `GET` 要求包含 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#insightstoken) 參數。

或者，您可以在 `POST` 要求的本文中包含二進位影像，並將 [modules](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#modulesrequested) 參數設定為 `RecognizedEntities`。 這會傳回 [insightsToken](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v5-reference#insightstoken) 作為後續 `GET` 要求中的參數，以傳回影像中人員的相關資訊。  將 `modules` 設定為 `All` 可取得所有見解 (`POST` 結果中的 `RecognizedEntities` 除外)，而不需要使用 `insightsToken` 進行其他呼叫。 


**端點 3：** 傳回影像，這些影像是以他人所提出的搜尋要求為基礎的發燒影像。 影像可分成不同類別，例如，根據值得注意的人員或事件。
```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/trending
```

如需支援發燒影像的市場清單，請參閱[發燒影像](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/trending-images)。

如需關於標頭、參數、市場代碼、回應物件、錯誤的詳細資料，請參閱 [Bing 影像搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference) 參考。
## <a name="response-json"></a>回應 JSON
影像搜尋要求的回應包含作為 JSON 物件的結果。 如需剖析結果的範例，請參閱[教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app)和[原始程式碼](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app-source)。

## <a name="next-steps"></a>後續步驟
**Bing** API 支援根據類型傳回結果的搜尋動作。 所有搜尋端點都會傳回 JSON 回應物件形式的結果。  所有端點皆支援查詢依照經度、緯度和搜尋半徑傳回特定語言及/或位置。

如需每個端點支援的參數完整相關資訊，請參閱每種類型的參考頁面。
如需透過範例了解使用影像搜尋 API 的基本要求，請參閱[影像搜尋快速入門](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/search-the-web)。