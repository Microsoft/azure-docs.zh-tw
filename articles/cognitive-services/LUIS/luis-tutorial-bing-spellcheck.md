---
title: 將 Bing 拼字檢查 API v7 新增至 LUIS 查詢 | Microsoft Docs
titleSuffix: Azure
description: 將 Bing 拼字檢查 API V7 新增至 LUIS 端點查詢，可更正語句中拼錯的字組。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 02/27/2018
ms.author: diberry
ms.openlocfilehash: 1f698a17ee6f4f654ba9d1e3741b190baf318e00
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031062"
---
# <a name="correct-misspelled-words-with-bing-spell-check"></a>使用 Bing 拼字檢查更正拼錯的字組

您可以整合 LUIS 應用程式與 [Bing 拼字檢查 API V7](https://azure.microsoft.com/services/cognitive-services/spell-check/)，以在 LUIS 預測語句的分數和實體之前更正拼錯的字組。 

## <a name="create-first-key-for-bing-spell-check-v7"></a>為 Bing 拼字檢查 V7 建立第一個金鑰
您的[第一個 Bing 拼字檢查 API v7 金鑰](https://azure.microsoft.com/try/cognitive-services/?api=spellcheck-api)是免費的。 

![建立免費金鑰](./media/luis-tutorial-bing-spellcheck/free-key.png)

<a name"create-subscription-key"></a>
## <a name="create-endpoint-key"></a>建立端點金鑰
如果您的免費金鑰過期，請建立端點金鑰。

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 在左上角選取 [建立資源]。

3. 在搜尋方塊中，輸入 `Bing Spell Check API V7`。

    ![搜尋 Bing 拼字檢查 API V7](./media/luis-tutorial-bing-spellcheck/portal-search.png)

4. 選取此服務。 

5. 資訊面板會顯示在右邊，包含法律聲明等資訊。 選取 [建立] 開始訂用帳戶建立程序。 

6. 在下一個面板中，輸入您的服務設定。 等候服務建立程序完成。

    ![輸入服務設定](./media/luis-tutorial-bing-spellcheck/subscription-settings.png)

7. 選取左側導覽列上 [我的最愛] 標題底下的 [所有資源]。

8. 選取新服務。 其類型為 [認知服務] 且位置為 [全域]。 

9. 在主面板中，選取 [金鑰] 以查看新的金鑰。

    ![擷取金鑰](./media/luis-tutorial-bing-spellcheck/grab-keys.png)

10. 複製第一個金鑰。 您只需要兩個金鑰中的其中一個。 

## <a name="using-the-key-in-luis-test-panel"></a>在 LUIS 測試面板中使用金鑰
LUIS 中有兩個地點可使用金鑰。 第一個是在[測試面板](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel)中。 金鑰不會儲存到 LUIS 中，但卻是工作階段變數。 每次您希望測試面板將 Bing 拼字檢查 API v7 服務套用至語句時，都需要設定金鑰。 請參閱測試面板中有關設定金鑰的[指示](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel)。

## <a name="adding-the-key-to-the-endpoint-url"></a>將金鑰新增至端點 URL
對於您想要套用拼字校正的每項查詢，端點查詢需要以查詢字串參數傳入的金鑰。 您可能有可呼叫 LUIS 的聊天機器人，或者您可以直接呼叫 LUIS 端點 API。 不論端點的呼叫方式為何，每次呼叫都必須包含所需的資訊，拼字校正才能正常運作。

端點 URL 有數個需要正確傳遞的值。 Bing 拼字檢查 API v7 金鑰只是其中一個。 您必須將 **spellCheck** 參數設為 true，而且必須將 **bing-spell-check-subscription-key** 的值設定為金鑰值：

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appID}?subscription-key={luisKey}&spellCheck=**true**&bing-spell-check-subscription-key=**{bingKey}**&verbose=true&timezoneOffset=0&q={utterance}`

## <a name="send-misspelled-utterance-to-luis"></a>將拼錯的語句傳送至 LUIS
1. 在網頁瀏覽器中，複製上述字串，並以您自己的值取代 `region`、`appId`、`luisKey` 和 `bingKey`。 如果端點區域不同於您的發佈[區域](luis-reference-regions.md)，請務必使用端點區域。

2. 新增拼錯的語句，例如 "How far is the mountainn?"。 在英文中，包含一個 `n` 的 `mountain` 是正確的拼法。 

3. 選取 Enter 將查詢傳送至 LUIS。

4. LUIS 會以 JSON 結果回應 `How far is the mountain?`。 如果 Bing 拼字檢查 API v7 偵測到拼錯，LUIS 應用程式的 JSON 回應中的 `query` 欄位包含原始查詢，而 `alteredQuery` 欄位包含傳送至 LUIS 的更正後查詢。

```
{
  "query": "How far is the mountainn?",
  "alteredQuery": "How far is the mountain?",
  "topScoringIntent": {
    "intent": "Concierge",
    "score": 0.183866
  },
  "entities": []
}
```

## <a name="ignore-spelling-mistakes"></a>忽略拼字錯誤
如果您不想使用 Bing 拼字檢查 API v7 服務，您也可以標示有拼字錯誤的語句，以便 LUIS 學習正確的拼法及錯字。 相較於使用拼字檢查工具，此選項需要更多標記工作。

## <a name="publishing-page"></a>發佈頁面
[發佈](luis-how-to-publish-app.md) 頁面具有 **啟用 Bing 拼字檢查工具** 核取方塊。 這方便於建立金鑰並理解如何變更端點 URL。 您仍然必須使用正確的端點參數，以便更正每個語句的拼字。 

> [!div class="nextstepaction"]
> [深入了解範例語句](luis-how-to-add-example-utterances.md)
