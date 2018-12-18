---
title: 什麼是 Bing Web 搜尋？
titleSuffix: Azure Cognitive Services
description: Bing Web 搜尋 API 是一項可即時解答使用者查詢的 RESTful 服務。 搜尋結果可經由簡單的設定，將網頁、影像、影片、新聞、翻譯等項目包含在內。 結果會以 JSON 的形式提供，並以搜尋相關性和您的 Bing Web 搜尋訂用帳戶為基礎。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: overview
ms.date: 08/14/2018
ms.author: erhopf
ms.openlocfilehash: 8482afae2889ccf2153e1864e4a65b635b6ab4c7
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46125233"
---
# <a name="what-is-bing-web-search"></a>什麼是 Bing Web 搜尋？

Bing Web 搜尋 API 是一項可即時解答使用者查詢的 RESTful 服務。 搜尋結果可經由簡單的設定，將網頁、影像、影片、新聞、翻譯等項目包含在內。 結果會以 JSON 的形式提供，並以搜尋相關性和您的 Bing Web 搜尋訂用帳戶為基礎。

如果應用程式需要存取所有與使用者的搜尋查詢有關的內容，就非常適合使用此 API。 如果您要建置的應用程式只需要特定類型的結果，請考慮使用 [Bing 影像搜尋 API](../Bing-Image-Search/overview.md)、[Bing 影片搜尋 API](../Bing-Video-Search/search-the-web.md) 或 [Bing 新聞搜尋 API](../Bing-News-Search/search-the-web.md)。 請參閱[認知服務 API](https://docs.microsoft.com/azure/cognitive-services#cognitive-services-apis)，以取得 Bing 搜尋 API 的完整清單。

想了解其運作方式嗎？ 請觀看我們的 [Bing Web 搜尋 API 示範](https://azure.microsoft.com/services/cognitive-services/bing-web-search-api/)。

## <a name="features"></a>特性  

除了即時解答以外，Bing Web 搜尋還提供可讓您為使用者自訂搜尋結果的其他特性和功能。

| 功能 | 說明 |
|---------|-------------|
| [即時建議搜尋字詞](../bing-autosuggest/get-suggested-search-terms.md) | 使用 Bing 自動建議 API 隨著使用者的輸入顯示建議的搜尋字詞，以改善您的應用程式體驗。 |
| [依內容類型篩選及限制結果](filter-answers.md) | 使用篩選條件和查詢參數，自訂及精簡網頁、影像、影片、安全搜尋的搜尋結果。 |
| [Unicode 字元的搜尋結果醒目提示](hit-highlighting.md) | 透過搜尋結果醒目提示，在對使用者顯示搜尋結果之前先從中找出並移除不必要的 Unicode 字元。 |
| [依國家/地區、區域和/或市場將搜尋結果當地語系化](supported-countries-markets.md) | Bing Web 搜尋可支援三十多個國家/地區或區域。 使用這項功能可精簡特定國家/區域或市場的搜尋結果。 |
| [使用 Bing 統計資料分析搜尋計量](bing-web-stats.md) | Bing 統計資料是付費訂閱，可提供關於呼叫量、熱門查詢字串、地理分佈等項目的分析。 |

## <a name="workflow"></a>工作流程

Bing Web 搜尋 API 可輕易地從任何可發出 HTTP 要求及剖析 JSON 回應的程式設計語言呼叫。 此服務可使用 [REST API](quickstarts/python.md) 或 [Bing Web 搜尋 SDK](web-sdk-python-quickstart.md) 來存取。  

1. 建立具備 Bing 搜尋 API 存取權的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 如果您沒有 Azure 訂用帳戶，您可以建立[免費帳戶](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)。  
2. [將要求傳送至 Bing Web 搜尋 API](quickstarts/python.md)。
3. 剖析 JSON 回應。

## <a name="next-steps"></a>後續步驟

* 使用我們的 [Python 快速入門](quickstarts/python.md)，發出您對 Bing Web 搜尋 API 的第一個呼叫。  
* [建置單頁 Web 應用程式](tutorial-bing-web-search-single-page-app.md)。
* 檢閱 [Web 搜尋 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)文件。  
* 深入了解 Bing Web 搜尋的[使用和顯示需求](UseAndDisplayRequirements.md)。  
