---
title: 教學課程 2：預先建置的意圖和實體 - 使用預先建置的常見語句 - 擷取 LUIS 中的常見資料
titleSuffix: Azure Cognitive Services
description: 將預先建置的意圖和實體新增至「人力資源」教學課程應用程式，以快速預測意圖及擷取資料。 您不需使用預先建置的實體來標示任何語句。 系統會自動偵測實體。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: d42aed76ecdbc2bd840e17517db2ca0b6ba11aa0
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034428"
---
# <a name="tutorial-2-identify-common-intents-and-entities"></a>教學課程 2：找出常見的意圖和實體
在本教學課程中，修改人力資源應用程式。 將預先建置的意圖和實體新增至「人力資源」教學課程應用程式，以快速預測意圖及擷取資料。 您不需使用預先建置的實體來標示任何語句，因為系統會自動偵測實體。

常見主體網域和資料類型的預先建置模型可協助您快速建置模型，以及提供模型的外觀範例。 

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增預先建置的意圖 
> * 新增預先建置的實體 
> * 定型 
> * 發佈 
> * 從端點取得意圖和實體

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-intent-only-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `prebuilts`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。 

## <a name="add-prebuilt-intents"></a>新增預先建置的意圖
LUIS 提供數個預先建置的意圖來協助處理常見的使用者意圖。  

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 選取 [新增預先建置的意圖]。 

3. 搜尋 `Utilities`。 

    [ ![搜尋方塊中有 Utilities 的預先建置意圖對話方塊螢幕擷取畫面](./media/luis-tutorial-prebuilt-intents-and-entities/prebuilt-intent-utilities.png)](./media/luis-tutorial-prebuilt-intents-and-entities/prebuilt-intent-utilities.png#lightbox)

4. 選取下列意圖，然後選取 [Done] \(完成\)： 

    * Utilities.Cancel
    * Utilities.Confirm
    * Utilities.Help
    * Utilities.StartOver
    * Utilities.Stop


## <a name="add-prebuilt-entities"></a>新增預先建置的實體
LUIS 提供數個預先建置的實體來擷取常見的資料。 

1. 從左側導覽功能表中選取 [Entities] \(實體\)。

2. 選取 [管理預先建置的實體] 按鈕。

3. 從預先建置的實體清單中選取 [number] 和 [datetimeV2]，然後選取 [Done] \(完成\)。

    ![預先建置的實體對話方塊中已選取 number 的螢幕擷取畫面](./media/luis-tutorial-prebuilt-intents-and-entities/select-prebuilt-entities.png)

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至瀏覽器位址列中的 URL 結尾並輸入 `I want to cancel on March 3`。 最後一個查詢字串參數是 `q`，也就是語句**查詢**。 

    ```JSON
    {
      "query": "I want to cancel on March 3",
      "topScoringIntent": {
        "intent": "Utilities.Cancel",
        "score": 0.7818295
      },
      "intents": [
        {
          "intent": "Utilities.Cancel",
          "score": 0.7818295
        },
        {
          "intent": "ApplyForJob",
          "score": 0.0237864349
        },
        {
          "intent": "GetJobInformation",
          "score": 0.017576348
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.0130122062
        },
        {
          "intent": "Utilities.Help",
          "score": 0.006731322
        },
        {
          "intent": "None",
          "score": 0.00524190161
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.004912514
        },
        {
          "intent": "Utilities.Confirm",
          "score": 0.00092950504
        }
      ],
      "entities": [
        {
          "entity": "march 3",
          "type": "builtin.datetimeV2.date",
          "startIndex": 20,
          "endIndex": 26,
          "resolution": {
            "values": [
              {
                "timex": "XXXX-03-03",
                "type": "date",
                "value": "2018-03-03"
              },
              {
                "timex": "XXXX-03-03",
                "type": "date",
                "value": "2019-03-03"
              }
            ]
          }
        },
        {
          "entity": "3",
          "type": "builtin.number",
          "startIndex": 26,
          "endIndex": 26,
          "resolution": {
            "value": "3"
          }
        }
      ]
    }
    ```

    結果預測出 Utilities.Cancel 意圖，並擷取了 3 月 3 日的日期和數字 3。 

    3 月 3 日有兩個值，原因是語句中並未指出 3 月 3 日是在過去還是未來。 如有需要，用戶端應用程式可自行決定是要做出假設還是問清楚。 

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

藉由新增預先建置的意圖和實體，用戶端應用程式可決定常見的使用者用意及擷取常見的資料類型。 

> [!div class="nextstepaction"]
> [在應用程式中新增規則運算式實體](luis-quickstart-intents-regex-entity.md)

