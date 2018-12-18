---
title: 什麼是 Bing 自動建議？
titlesuffix: Azure Cognitive Services
description: 了解如何使用 Bing 自動建議 API。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-autosuggest
ms.topic: overview
ms.date: 09/12/2017
ms.author: scottwhi
ms.openlocfilehash: 18658af90bd1ff71e84a5b2332db7e30bc23f69a
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48829387"
---
# <a name="what-is-bing-autosuggest"></a>什麼是 Bing 自動建議？

如果您將查詢傳送給任何 Bing 搜尋 API，您可以使用 Bing 自動建議 API 來改善搜尋方塊體驗。 Bing 自動建議 API 會根據使用者在搜尋方塊中輸入的部分字串，傳回建議的查詢清單。 在搜尋方塊的下拉式清單中顯示建議。 建議字詞的根據是，其他使用者所搜尋過的建議查詢以及使用者意圖。

一般來說，您會在每次使用者在搜尋方塊中鍵入新字元時，呼叫此 API。 查詢字串的完整性會影響 API 傳回之建議查詢字詞的相關性。 查詢字串越完整，建議之查詢字詞的相關性就越大。 例如，API 為 *s* 可能傳回的建議也許比查詢為 *sailing dinghies* 傳回的建議相關性較小。

## <a name="getting-suggested-search-terms"></a>取得建議的搜尋字詞

下列範例顯示針對 *sail* 傳回建議查詢字串的要求。 請記得在設定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v7-reference#query) 查詢參數時，對使用者的部分查詢字詞進行 URL 編碼。 例如，如果使用者輸入「sailing les」，請將 `q` 設定為 `sailing+les` 或 `sailing%20les`。

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/suggestions?q=sail&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

下列回應會包含一份 [SearchAction](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v7-reference#searchaction) 物件清單，其中包含建議的查詢字詞。

```json
{
    "url" : "https:\/\/www.bing.com\/search?q=sailing+lessons+seattle&FORM=USBAPI",
    "displayText" : "sailing lessons seattle",
    "query" : "sailing lessons seattle",
    "searchKind" : "WebSearch"
}, ...
```

每個建議都包含 `displayText`、`query` 及 `url` 欄位。 `displayText` 欄位包含建議的查詢，您可以用來填入搜尋方塊的下拉式清單。 您必須依提供的順序顯示回應包含的所有建議。

以下舉例說明下拉式搜尋方塊與建議的查詢字詞。

![自動建議下拉式搜尋方塊清單](./media/cognitive-services-bing-autosuggest-api/bing-autosuggest-drop-down-list.PNG)

如果使用者從下拉式清單選取建議的查詢，您可以使用 `query` 欄位中的查詢字詞呼叫 [Bing Web 搜尋 API](../bing-web-search/search-the-web.md)，並自行顯示結果。 或者，您也可以改為使用 `url` 欄位中的 URL，將使用者傳送至 Bing 搜尋結果頁面。

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>後續步驟

若要快速開始使用您的第一個要求，請參閱[建立您的第一個查詢](quickstarts/csharp.md)。

請熟悉 [Bing 自動建議 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v7-reference) 參考。 此參考包含端點、標頭、您用來要求建議查詢字詞的查詢參數，以及回應物件定義的清單。

了解如何使用 [Bing Web 搜尋 API](../bing-web-search/search-the-web.md) 來搜尋 Web。

務必閱讀 [Bing 使用和顯示需求](./useanddisplayrequirements.md)，您才不會違反任何有關使用搜尋結果的規則。