---
title: 教學課程 8：LUIS 中的關鍵片語擷取
titleSuffix: Azure Cognitive Services
description: 使用預先建立的實體 keyPhrase，從語句中擷取關鍵主題。 您不需使用預先建置的實體來標示任何語句。 系統會自動偵測實體。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: 8fa183c22b9b6830c57b0a16b7f5d20ca38e3ef3
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166515"
---
# <a name="tutorial-8-extract-key-phrases-of-utterance"></a>教學課程 8：擷取語句的關鍵片語
在本教學課程中，您將使用預先建置的 keyPhrase 實體，從語句中擷取關鍵主題。 您不需使用預先建置的實體來標示任何語句。 系統會自動偵測實體。

下列語句顯示關鍵片語範例：

|語句|KeyPhrase 實體值|
|--|--|
|下一個年度是否有提供較低扣除額的新醫療計劃？|「較低扣除額」<br>「新醫療計劃」<br>「年度」|
|高扣除額的醫療計劃是否涵蓋視力治療？|「高扣除額的醫療計劃」<br>「視力治療」|

用戶端應用程式可以使用這些值以及所擷取的其他實體，來決定對話的下一個步驟。

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增 KeyPhrase 實體 
> * 定型
> * 發佈
> * 從端點取得意圖和實體

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式

以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-simple-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `keyphrase`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

## <a name="add-keyphrase-entity"></a>新增 KeyPhrase 實體 
新增預先建立的 keyPhrase 實體，從語句中擷取主題。

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 從左側功能表中選取 [實體]。

3. 選取 [Manage prebuilt entities] \(管理預先建置的實體\)。

4. 在快顯對話方塊中，選取 [keyPhrase]，然後選取 [完成]。 

    [ ![[實體] 清單快顯對話方塊的螢幕擷取畫面](./media/luis-quickstart-intent-and-key-phrase/hr-add-or-remove-prebuilt-entities.png)](./media/luis-quickstart-intent-and-key-phrase/hr-add-or-remove-prebuilt-entities.png#lightbox)

5. 從左功能表選取 [意圖]，然後選取 [Utilities.Confirm] 意圖。 數個語句中會標示 KeyPhrase 實體。 

    [ ![語句中已標示 keyPhrases 的 Utilities.Confirm 意圖螢幕擷取畫面](./media/luis-quickstart-intent-and-key-phrase/hr-keyphrase-labeled.png)](./media/luis-quickstart-intent-and-key-phrase/hr-keyphrase-labeled.png#lightbox)

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `does form hrf-123456 cover the new dental benefits and medical plan`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 
    
    ```JSON
    {
      "query": "does form hrf-123456 cover the new dental benefits and medical plan",
      "topScoringIntent": {
        "intent": "FindForm",
        "score": 0.9300641
      },
      "intents": [
        {
          "intent": "FindForm",
          "score": 0.9300641
        },
        {
          "intent": "ApplyForJob",
          "score": 0.0359598845
        },
        {
          "intent": "GetJobInformation",
          "score": 0.0141798034
        },
        {
          "intent": "MoveEmployee",
          "score": 0.0112197418
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.00507669244
        },
        {
          "intent": "None",
          "score": 0.00238501839
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00202810857
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.00102957746
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.0008688423
        },
        {
          "intent": "Utilities.Confirm",
          "score": 3.557994E-05
        }
      ],
      "entities": [
        {
          "entity": "hrf-123456",
          "type": "HRF-number",git 
          "startIndex": 10,
          "endIndex": 19
        },
        {
          "entity": "new dental benefits",
          "type": "builtin.keyPhrase",
          "startIndex": 31,
          "endIndex": 49
        },
        {
          "entity": "medical plan",
          "type": "builtin.keyPhrase",
          "startIndex": 55,
          "endIndex": 66
        },
        {
          "entity": "hrf",
          "type": "builtin.keyPhrase",
          "startIndex": 10,
          "endIndex": 12
        },
        {
          "entity": "-123456",
          "type": "builtin.number",
          "startIndex": 13,
          "endIndex": 19,
          "resolution": {
            "value": "-123456"
          }
        }
      ]
    }
    ```

    在搜尋表單時，使用者所提供的資訊超過尋找表單所需要的資訊。 多餘的資訊會以 **builtin.keyPhrase** 的形式傳回。 用戶端應用程式可以使用這項多餘的資訊來建立後續問題，例如「您是否要和人力資源代表談談新的看牙福利」，也可以提供有更多選項的功能表，包括「更多關於新看牙福利或醫療方案的資訊。」

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

本教學課程已新增預先建置的 keyPhrase 實體，在不需要對語句加上標籤的情況下，快速地提供語句中的關鍵片語。 

> [!div class="nextstepaction"]
> [在應用程式中新增情感分析](luis-quickstart-intent-and-sentiment-analysis.md)