---
title: 教學課程 9：在 LUIS 中包括正面、負面和中性的情感分析
titleSuffix: Azure Cognitive Services
description: 在本教學課程中，建立一個應用程式，示範如何從語句中擷取正面、負面和中性人氣。 情感是從整個語句決定。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: ff5a47f977f34535c5ad1fde7e6cac5995e7f7dd
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031453"
---
# <a name="tutorial-9--extract-sentiment-of-overall-utterance"></a>教學課程 9：擷取整體語句的情感
在本教學課程中，建立一個應用程式，示範如何從語句中擷取正面、負面和中性人氣。 情感是從整個語句決定。

情感分析可判斷使用者的語句屬於正面、負面還是中性語句。 

下列語句會顯示情感範例：

|情感|分數|語句|
|:--|:--|:--|
|正面|0.91 |John W. Smith did a great job on the presentation in Paris.|
|正面|0.84 |jill-jones@mycompany.com did fabulous work on the Parker sales pitch.|

情感分析是適用於每個語句的應用程式設定。 您不必尋找語句中指出情感的字組並為其加上標籤，因為感情分析適用於整個語句。 

這是發佈設定，所以您在意圖或實體頁面上看不見它。 您可以在[互動式測試](luis-interactive-test.md#view-sentiment-results)窗格中或在端點 URL 進行測試時看到它。 

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式 
> * 新增情感分析作為發佈設定
> * 定型
> * 發佈
> * 從端點取得語句的情感

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式

以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-keyphrase-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `sentiment`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

## <a name="employeefeedback-intent"></a>新增 EmployeeFeedback 意圖 
新增意圖，以擷取公司成員的員工意見。 

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 選取 [Create new intent] \(建立新意圖\)。

3. 將新的意圖命名為 `EmployeeFeedback`。

    ![以 EmployeeFeedback 作為名稱來建立新的意圖對話方塊](./media/luis-quickstart-intent-and-sentiment-analysis/hr-create-new-intent-ddl.png)

4. 新增數個語句，以指出員工表現良好或需要改進的地方：

    請記住在此人力資源應用程式中，員工會依照名稱、電子郵件、電話分機號碼、行動電話號碼及其美國聯邦社會安全號碼，定義於清單實體 `Employee` 中。 

    |表達方式|
    |--|
    |425-555-1212 did a nice job of welcoming back a co-worker from maternity leave|
    |234-56-7891 did a great job of comforting a co-worker in their time of grief.|
    |jill-jones@mycompany.com didn't have all the required invoices for the paperwork.|
    |john.w.smith@mycompany.com turned in the required forms a month late with no signatures|
    |x23456 didn't make it to the important marketing off-site meeting.|
    |x12345 missed the meeting for June reviews.|
    |Jill Jones rocked the sales pitch at Harvard|
    |John W. Smith did a great job on the presentation at Stanford|

    [ ![在 EmployeeFeedback 意圖中有範例語句的 LUIS 應用程式螢幕擷取畫面](./media/luis-quickstart-intent-and-sentiment-analysis/hr-utterance-examples.png)](./media/luis-quickstart-intent-and-sentiment-analysis/hr-utterance-examples.png#lightbox)

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="configure-app-to-include-sentiment-analysis"></a>設定應用程式以納入情感分析
1. 選取右上方導覽列中的 [管理]，然後從左側功能表中選取 [發佈設定]。

2. 切換 [情感分析] 以啟用這項設定。 

    ![](./media/luis-quickstart-intent-and-sentiment-analysis/turn-on-sentiment-analysis-as-publish-setting.png)

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-sentiment-of-utterance-from-endpoint"></a>從端點取得語句的情感

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `Jill Jones work with the media team on the public portal was amazing`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 此語句與任何標示的語句都不同，因此這是很好的測試，且應該傳回具有所擷取情感分析的 `EmployeeFeedback` 意圖。
    
    ```JSON
    {
      "query": "Jill Jones work with the media team on the public portal was amazing",
      "topScoringIntent": {
        "intent": "EmployeeFeedback",
        "score": 0.4983256
      },
      "intents": [
        {
          "intent": "EmployeeFeedback",
          "score": 0.4983256
        },
        {
          "intent": "MoveEmployee",
          "score": 0.06617523
        },
        {
          "intent": "GetJobInformation",
          "score": 0.04631853
        },
        {
          "intent": "ApplyForJob",
          "score": 0.0103248553
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.007531875
        },
        {
          "intent": "FindForm",
          "score": 0.00344597152
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00337914471
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.0026357458
        },
        {
          "intent": "None",
          "score": 0.00214573368
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.00157622492
        },
        {
          "intent": "Utilities.Confirm",
          "score": 7.379545E-05
        }
      ],
      "entities": [
        {
          "entity": "jill jones",
          "type": "Employee",
          "startIndex": 0,
          "endIndex": 9,
          "resolution": {
            "values": [
              "Employee-45612"
            ]
          }
        },
        {
          "entity": "media team",
          "type": "builtin.keyPhrase",
          "startIndex": 25,
          "endIndex": 34
        },
        {
          "entity": "public portal",
          "type": "builtin.keyPhrase",
          "startIndex": 43,
          "endIndex": 55
        },
        {
          "entity": "jill jones",
          "type": "builtin.keyPhrase",
          "startIndex": 0,
          "endIndex": 9
        }
      ],
      "sentimentAnalysis": {
        "label": "positive",
        "score": 0.8694164
      }
    }
    ```

    sentimentAnalysis 為正面且分數為 0.86。 

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟
本教學課程會將情感分析新增為發佈設定，以整個語句擷取情感值。

> [!div class="nextstepaction"] 
> [檢閱 HR 應用程式中的端點語句](luis-tutorial-review-endpoint-utterances.md) 

