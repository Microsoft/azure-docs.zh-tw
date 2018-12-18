---
title: 自動建議端點 | Microsoft Docs
description: 自動建議 API 端點的摘要。
services: cognitive-services
author: mikedodaro
manager: rosh
ms.service: cognitive-services
ms.component: bing-autosuggest
ms.topic: article
ms.date: 12/05/2017
ms.author: v-gedod
ms.openlocfilehash: 5aaddd09006cb6f1812bb6ae213a2f5e6720fb1b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369915"
---
# <a name="autosuggest-endpoint"></a>自動建議端點

**自動建議 API** 內含一個端點，會根據部份搜尋字詞傳回建議的查詢清單。

## <a name="endpoint"></a>端點

若要使用 Bing API 取得建議的查詢，請傳送 `GET` 要求到下列端點。 使用標頭和 URL 參數以進一步定義規格。

**端點：** 以 JSON 結果的形式，傳回根據 `?q=""` 之定義與使用者輸入內容相關的搜尋建議。

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/Suggestions 
```

如需標頭、參數、市場代碼、回應物件、錯誤等詳細資料，請參閱 [Bing 自動建議 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v7-reference) (英文)。

## <a name="response-json"></a>回應 JSON

自動建議要求的回應含有作為 JSON 物件的結果。 如需結果剖析的範例，請參閱[教學課程](tutorials/autosuggest.md)和[原始程式碼](tutorials/autosuggest-source.md)。

## <a name="next-steps"></a>後續步驟

**Bing** API 支援根據其類型傳回結果的搜尋動作。 所有搜尋端點會傳回作為 JSON 回應物件的結果。
所有端點均可支援依照經度、緯度和搜尋半徑傳回特定語言及/或位置的查詢。

如需每個端點支援之參數的完整相關資訊，請參閱每種類型的參考頁面。
如需使用自動建議 API 之基本要求的範例，請參閱[自動建議快速入門](https://docs.microsoft.com/azure/cognitive-services/Bing-Autosuggest) (英文)。