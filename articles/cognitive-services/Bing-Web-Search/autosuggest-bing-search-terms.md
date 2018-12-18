---
title: 自動建議 Bing 搜尋字詞
titleSuffix: Microsoft Cognitive Services
description: 將 Bing Web 搜尋 API 與 Bing 自動建議 API 配對，為使用者提供改善的搜尋體驗。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 8/13/2018
ms.author: erhopf
ms.openlocfilehash: df8a57b3136abfacce971f4d01ccb2296dfa784c
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42890073"
---
# <a name="autosuggest-bing-search-terms"></a>自動建議 Bing 搜尋字詞

若您提供使用者可在其中輸入其搜尋字詞的搜尋方塊，請使用 [Bing 自動建議 API](../bing-autosuggest/get-suggested-search-terms.md) 來改善使用經驗。 API 會根據部分搜尋字詞傳回建議的查詢字串，作為使用者類型。

在使用者輸入搜尋字詞之後，必須在設定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#query) (英文) 查詢參數之前進行 URL 編碼。 例如，如果使用者輸入 *sailing dinghies*，請將 `q` 設定為 `sailing+dinghies` 或 `sailing%20dinghies`。

若查詢字詞包含拼字錯誤，搜尋回應會包含 [QueryContext](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#querycontext) (英文) 物件。 物件會顯示 Bing 用於搜尋的原始拼字和已更正的拼字。

```json
"queryContext": {
    "originalQuery": "sialing dingy for sale",
    "alteredQuery": "sailing dinghy for sale",
    "alterationOverrideQuery": "+sialing +dingy for sale"
}
```

您可以使用此資訊讓使用者知道，您在顯示搜尋結果時已修改他們的查詢字串。

![查詢內容 UX 範例](./media/cognitive-services-bing-web-api/bing-query-context.PNG)  

## <a name="next-steps"></a>後續步驟  

* 檢閱範例 [Bing Web 搜尋 API 回應](search-responses.md)。  

## <a name="see-also"></a>另請參閱  

* [Bing Web 搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)
