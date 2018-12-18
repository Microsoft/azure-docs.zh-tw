---
title: 教學課程 3：與規則運算式相符的資料 - 擷取格式正確的資料
titleSuffix: Azure Cognitive Services
description: 使用規則運算式實體從語句擷取格式一致的資料。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: 06e212ef756fda9224b38b41c69c7c4eccfb9796
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159851"
---
# <a name="tutorial-3-extract-well-formatted-data"></a>教學課程 3：擷取格式正確的資料
在本教學課程中，修改人力資源應用程式，使其使用**規則運算式**實體來從語句擷取格式一致的資料。

實體的目的是要擷取語句中包含的重要資料。 此應用程式會使用規則運算式實體來提取語句中的格式化人力資源 (HR) 表單編號。 雖然語句的意圖一定會由機器學習決定，但此特定實體類型不會進行機器學習。 

**語句範例包括：**

```
Where is HRF-123456?
Who authored HRF-123234?
HRF-456098 is published in French?
HRF-456098
HRF-456098 date?
HRF-456098 title?
```
 
在下列情況下，規則運算式是這類資料的好選擇：

* 資料的格式正確。

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增 FindForm 意圖
> * 新增規則運算式實體 
> * 定型
> * 發佈
> * 從端點取得意圖和實體

[!include[LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1. 下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-prebuilts-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `regex`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。 

## <a name="findform-intent"></a>FindForm 意圖

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 選取 [Create new intent] \(建立新意圖\)。 

3. 在快顯對話方塊方塊中輸入 `FindForm`，然後選取 [完成]。 

    ![搜尋方塊中有公用程式的建立新意圖對話方塊螢幕擷取畫面](./media/luis-quickstart-intents-regex-entity/create-new-intent-ddl.png)

4. 將語句範例新增至意圖。

    |範例語句|
    |--|
    |hrf-123456 的 URL 是什麼？|
    |hrf-345678 在哪裡？|
    |何時更新了 hrf-456098？|
    |John Smith 是否在上週更新了 hrf-234639？|
    |hrf-345123 有多少版本？|
    |誰需要授權表單 hrf-123456？|
    |有多少人需要簽署 hrf-345678？|
    |hrf-234123 日期？|
    |hrf-546234 的作者？|
    |hrf-456234 的標題？|

    [ ![醒目提示新語句的意圖頁面螢幕擷取畫面](./media/luis-quickstart-intents-regex-entity/findform-intent.png) ](./media/luis-quickstart-intents-regex-entity/findform-intent.png#lightbox)

    應用程式有從上一個教學課程中新增的預先建置編號實體，因此每個表單編號都已加上標記。 這對您的用戶端應用程式而言可能已足夠，但編號不會標示編號類型。 以適當的名稱建立新實體，可讓用戶端應用程式正確地處理 LUIS 所傳回的實體。

    [!include[Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]  

## <a name="regular-expression-entity"></a>規則運算式實體 
要比對表單編號的規則運算式實體為 `hrf-[0-9]{6}`。 此規則運算式會比對常值字元 `hrf-`，但會忽略大小寫和文化特性造成的差異。 它會完全符合 0-9 的 6 位數數字。

HRF 代表 `human resources form`。

將語句新增至意圖時，LUIS 會將語句 Token 化。 這些語句的 Token 化會在連字號前面和後面加入空格：`Where is HRF - 123456?` 規則運算式會套用至原始格式的語句 (Token 化之前)。 由於這是套用至_原始_表單，因此規則運算式不需要處理字詞界限。 

使用下列步驟建立規則運算式實體，以向 LUIS 告知 HRF-編號的格式：

1. 在左側面板中選取 [實體]。

2. 選取 [實體] 頁面上的 [建立新實體]按鈕。 

3. 在快顯對話方塊中，輸入新的實體名稱 `HRF-number`，選取 **RegEx** 作為實體類型，然後輸入 `hrf-[0-9]{6}` 作為 [Regex] 值，最後選取 [完成]。

    ![設定新實體屬性的快顯對話方塊螢幕擷取畫面](./media/luis-quickstart-intents-regex-entity/create-regex-entity.png)

4. 從左側功能表選取 [意圖]，然後選取 [FindForm] 意圖，以查看語句中標示的規則運算式。 

    [![以現有實體和 RegEx 模式標示語句的螢幕擷取畫面](./media/luis-quickstart-intents-regex-entity/labeled-utterances-for-entity.png)](./media/luis-quickstart-intents-regex-entity/labeled-utterances-for-entity.png#lightbox)

    因為實體不是機器學習的實體，標籤建立時即會套用到語句，並顯示在 LUIS 網站中。

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `When were HRF-123456 and hrf-234567 published in the last year?`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 此語句與任何標示的語句都不同，因此這是很好的測試，且應該傳回具有兩種表單編號 `HRF-123456` 和 `hrf-234567` 的 `FindForm` 意圖。

    ```JSON
    {
      "query": "When were HRF-123456 and hrf-234567 published in the last year?",
      "topScoringIntent": {
        "intent": "FindForm",
        "score": 0.9993477
      },
      "intents": [
        {
          "intent": "FindForm",
          "score": 0.9993477
        },
        {
          "intent": "ApplyForJob",
          "score": 0.0206110049
        },
        {
          "intent": "GetJobInformation",
          "score": 0.00533067342
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.004215215
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00209096959
        },
        {
          "intent": "None",
          "score": 0.0017655947
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.00109490135
        },
        {
          "intent": "Utilities.Confirm",
          "score": 0.0005704638
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.000525338168
        }
      ],
      "entities": [
        {
          "entity": "last year",
          "type": "builtin.datetimeV2.daterange",
          "startIndex": 53,
          "endIndex": 61,
          "resolution": {
            "values": [
              {
                "timex": "2017",
                "type": "daterange",
                "start": "2017-01-01",
                "end": "2018-01-01"
              }
            ]
          }
        },
        {
          "entity": "hrf-123456",
          "type": "HRF-number",
          "startIndex": 10,
          "endIndex": 19
        },
        {
          "entity": "hrf-234567",
          "type": "HRF-number",
          "startIndex": 25,
          "endIndex": 34
        },
        {
          "entity": "-123456",
          "type": "builtin.number",
          "startIndex": 13,
          "endIndex": 19,
          "resolution": {
            "value": "-123456"
          }
        },
        {
          "entity": "-234567",
          "type": "builtin.number",
          "startIndex": 28,
          "endIndex": 34,
          "resolution": {
            "value": "-234567"
          }
        }
      ]
    }
    ```

    語句中的數字傳回兩次，一次作為新的實體 `hrf-number`，一次作為預先建置的實體 `number`。 語句可以有多個實體和多個相同類型的實體，如此範例所示。 藉由使用規則運算式實體，LUIS 會擷取具名的資料，這樣更能以程式設計方式協助用戶端應用程式接收 JSON 回應。


## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟
本教學課程已建立新的意圖、新增語句範例，接著建立了規則運算式實體以從語句中擷取格式正確的資料。 定型和發佈應用程式之後，端點的查詢識別了意圖並傳回擷取的資料。

> [!div class="nextstepaction"]
> [了解清單實體](luis-quickstart-intent-and-list-entity.md)

