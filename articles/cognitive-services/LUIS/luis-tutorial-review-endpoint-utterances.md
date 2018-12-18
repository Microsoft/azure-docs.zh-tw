---
title: 教學課程 1：使用主動式學習檢閱端點語句
titleSuffix: Azure Cognitive Services
description: 藉由驗證或更正透過 LUIS 不確定的 LUIS HTTP 端點所收到的語句來改善應用程式的預測。 有些語句可能會針對意圖進行驗證，而其他語句則可能需要針對實體進行驗證。 您應該在排定的 LUIS 維護中定期檢閱端點語句。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: 1047c117228b57f7361a1e386bc6cde7acbfdde8
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042262"
---
# <a name="tutorial-1-fix-unsure-predictions"></a>教學課程 1：修正不確定的預測
在本教學課程中，藉由驗證或更正透過 LUIS 不確定的 LUIS HTTPS 端點所收到的語句來改善應用程式的預測。 有些語句可能必須針對意圖進行驗證，而其他語句則可能需要針對實體進行驗證。 您應該在排定的 LUIS 維護中定期檢閱端點語句。 

此檢閱程序是 LUIS 用來了解應用程式領域的另一種方法。 LUIS 選取了檢閱清單中顯示的語句。 此清單是：

* 應用程式所特有的。
* 為了改善應用程式的預測準確度。 
* 應定期檢閱。 

藉由檢閱端點語句，您可以確認或更正語句的預測意圖。 您也可以將未預測或預測錯誤的自訂實體加上標籤。 

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 檢閱端點語句
> * 更新片語清單
> * 訓練應用程式
> * 發佈應用程式
> * 查詢應用程式端點來查看 LUIS JSON 回應

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式

以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-sentiment-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `review`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

    如果您使用本教學課程作為新的已匯入應用程式，則您也需要訓練和發佈端點，然後將語句新增至具有[指令碼](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/demo-upload-endpoint-utterances/endpoint.js)的端點，或從瀏覽器中的端點新增語句。 要新增的語句為：

   [!code-nodejs[Node.js code showing endpoint utterances to add](~/samples-luis/examples/demo-upload-endpoint-utterances/endpoint.js?range=15-26)]

    如果您透過一系列教學課程而擁有所有版本的應用程式，您可能會驚訝地發現**檢閱端點語句**清單並不會因為版本不同而有所變更。 不論您正在編輯的版本為何，也不論在端點上發佈的應用程式版本為何，都只有一個要檢閱的語句集區。 

## <a name="review-endpoint-utterances"></a>檢閱端點語句

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 從左側導覽選取 [檢閱端點語句]。 此清單已針對 **ApplyForJob** 意圖進行篩選。 

    [ ![[檢閱端點語句] 按鈕在左側導覽中的螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/review-endpoint-utterances-with-entity-view.png)](./media/luis-tutorial-review-endpoint-utterances/review-endpoint-utterances-with-entity-view.png#lightbox)

3. 切換 [實體檢視] 以查看加上標籤的實體。 
    
    [ ![已醒目提示 [實體檢視] 開關的 [檢閱端點語句] 螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/review-endpoint-utterances-with-token-view.png)](./media/luis-tutorial-review-endpoint-utterances/review-endpoint-utterances-with-token-view.png#lightbox)

    |語句|正確的意圖|遺漏的實體|
    |:--|:--|:--|
    |I'm looking for a job with Natural Language Processing|GetJobInfo|Job - "Natural Language Process"|

    此語句的意圖不正確，且分數小於 50%。 **ApplyForJob** 意圖有 21 個語句，相較之下，**GetJobInformation** 有 7 個語句。 隨著端點語句正確調整，應該會有更多的語句新增至 **GetJobInformation** 意圖。 這部分會留作練習讓您自行完成。 每個意圖 (**無**意圖除外) 應該會有數量大致相同的語句範例。 **無**意圖所包含的語句應該會佔應用程式中總語句的 10%。 

4. 對於 `I'm looking for a job with Natual Language Processing` 意圖，請在 [一致的意圖] 資料行中選取正確的意圖 **GetJobInformation**。 

    [ ![讓語句與意圖一致的 [檢閱端點語句] 螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/align-intent-1.png)](./media/luis-tutorial-review-endpoint-utterances/align-intent-1.png#lightbox)

5. 在同一個語句中，`Natural Language Processing` 的實體是 keyPhrase。 這反而應該是**職位**實體。 從清單中依序選取 [`Natural Language Processing`] 和 [職位] 實體。

    [ ![將語句中實體加上標籤的 [檢閱端點語句] 螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/label-entity.png)](./media/luis-tutorial-review-endpoint-utterances/label-entity.png#lightbox)

6. 在同一行中，於 [新增至一致的意圖] 資料行中選取圈選核取記號。 

    [ ![正在完成讓意圖中的語句一致的螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/align-utterance.png)](./media/luis-tutorial-review-endpoint-utterances/align-utterance.png#lightbox)

    此動作會將語句從 [檢閱端點語句] 移到 [GetJobInformation] 意圖。 端點語句現在是該意圖的語句範例。 

7. 檢閱此意圖中剩餘的語句，如果這些語句不正確，請將語句加上標籤並更正 [一致的意圖]。

8. 當所有語句都正確無誤時，請選取每個資料列的核取方塊，然後選取 [新增選取項目] 讓語句變得正確。 

    [ ![正在完成剩餘語句以成為一致意圖的螢幕擷取畫面](./media/luis-tutorial-review-endpoint-utterances/finalize-utterance-alignment.png)](./media/luis-tutorial-review-endpoint-utterances/finalize-utterance-alignment.png#lightbox)

9. 此清單應該不會再有這些語句。 如果出現更多語句，請繼續檢查清單，更正意圖並將遺漏的實體加上標籤，直到清單清空為止。 

10. 選取 [篩選] 清單中的下一個意圖，然後繼續更正語句並將實體加上標籤。 請記住，每個意圖的最後一個步驟都是選取語句資料列上的 [新增至一致的意圖] 或核取每個意圖旁的方塊，然後選取資料表上方的 [新增選取項目]。

    繼續執行，直到篩選清單中的所有意圖和實體都有空白清單為止。 這是非常小型的應用程式。 檢閱程序只會花上幾分鐘的時間。 

## <a name="update-phrase-list"></a>更新片語清單
透過新探索到的職位名稱讓片語清單隨時保持最新狀態。 

1. 在左側瀏覽窗格中選取 [片語清單]。

2. 選取 [職位] 片語清單。

3. 新增 `Natural Language Processing` 值，然後選取 [儲存]。 

## <a name="train"></a>定型

LUIS 在定型過後才會知道變更。 

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

如果您已匯入此應用程式，則必須選取 [情感分析]。

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

嘗試與更正後的語句接近的語句。 

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `Are there any natural language processing jobs in my department right now?`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 

  ```JSON
  {
    "query": "are there any natural language processing jobs in my department right now?",
    "topScoringIntent": {
      "intent": "GetJobInformation",
      "score": 0.9247605
    },
    "intents": [
      {
        "intent": "GetJobInformation",
        "score": 0.9247605
      },
      {
        "intent": "ApplyForJob",
        "score": 0.129989788
      },
      {
        "intent": "FindForm",
        "score": 0.006438211
      },
      {
        "intent": "EmployeeFeedback",
        "score": 0.00408575451
      },
      {
        "intent": "Utilities.StartOver",
        "score": 0.00194211153
      },
      {
        "intent": "None",
        "score": 0.00166400627
      },
      {
        "intent": "Utilities.Help",
        "score": 0.00118593348
      },
      {
        "intent": "MoveEmployee",
        "score": 0.0007885918
      },
      {
        "intent": "Utilities.Cancel",
        "score": 0.0006373631
      },
      {
        "intent": "Utilities.Stop",
        "score": 0.0005980781
      },
      {
        "intent": "Utilities.Confirm",
        "score": 3.719905E-05
      }
    ],
    "entities": [
      {
        "entity": "right now",
        "type": "builtin.datetimeV2.datetime",
        "startIndex": 64,
        "endIndex": 72,
        "resolution": {
          "values": [
            {
              "timex": "PRESENT_REF",
              "type": "datetime",
              "value": "2018-07-05 15:23:18"
            }
          ]
        }
      },
      {
        "entity": "natural language processing",
        "type": "Job",
        "startIndex": 14,
        "endIndex": 40,
        "score": 0.9869922
      },
      {
        "entity": "natural language processing jobs",
        "type": "builtin.keyPhrase",
        "startIndex": 14,
        "endIndex": 45
      },
      {
        "entity": "department",
        "type": "builtin.keyPhrase",
        "startIndex": 53,
        "endIndex": 62
      }
    ],
    "sentimentAnalysis": {
      "label": "positive",
      "score": 0.8251864
    }
  }
  }
  ```

  系統會預測出具有高分的正確意圖，並將**職位**實體偵測為 `natural language processing`。 

## <a name="can-reviewing-be-replaced-by-adding-more-utterances"></a>可否藉由新增更多語句來取代檢閱？ 
您可能會納悶，為什麼不新增更多的語句範例。 檢閱端點語句的目的為何？ 在真實世界的 LUIS 應用程式中，端點語句來自於使用者，而其字組選擇和排列方式皆是您未曾用過的。 如果您使用過相同的字組選擇和排列方式，則原始的預測會有較高的百分比。 

## <a name="why-is-the-top-intent-on-the-utterance-list"></a>為何語句清單上會有最高分意圖？ 
某些端點語句會在檢閱清單中擁有高預測分數。 您還是需要檢閱並確認這些語句。 這些語句之所以位於清單上，是因為下一個最高意圖的分數非常接近最高分意圖的分數。 您希望兩個最高意圖之間大約有 15% 差異。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已檢閱在 LUIS 不確定的端點所提交的語句。 這些語句經過驗證並移至正確的意圖作為範例語句後，LUIS 將會提高預測準確度。

> [!div class="nextstepaction"]
> [了解如何使用模式](luis-tutorial-pattern.md)
