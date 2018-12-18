---
title: 教學課程 4：完全相符的文字項目 - LUIS 清單實體
titleSuffix: Azure Cognitive Services
description: 取得符合預先定義項目清單的資料。 清單上的每個項目可以擁有也完全相符的同義字
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: b4fdf094653a4b16dead6397fe8e1a9f1a0258b9
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162078"
---
# <a name="tutorial-4-extract-exact-text-matches"></a>教學課程 4：擷取完全相符的文字項目
在本教學課程中，了解如何取得與預先定義的項目清單相符的資料。 清單上的每個項目可以包含同義字清單。 在人力資源應用程式中，員工的識別可以透過數個關鍵資訊來實現，例如姓名、電子郵件、電話號碼和美國聯邦稅務識別碼。 

人力資源應用程式必須判斷哪些員工會在不同大樓間移動。 針對關於員工移動的語句，LUIS 會判斷意圖並擷取員工，讓用戶端應用程式可以建立用來移動員工的標準順序。

此應用程式會使用清單實體來擷取員工。 使用姓名、公司電話分機號碼、行動電話號碼、電子郵件或美國聯邦社會安全號碼即可參考員工。 

在下列情況下，清單實體是這類資料的好選擇：

* 資料值是一組已知的值。
* 此組合不會超過此實體類型的最大 LUIS [界限](luis-boundaries.md)。
* 語句中的文字是與同義字或正式名稱完全相符的項目。 

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增 MoveEmployee 意圖
> * 新增清單實體 
> * 定型 
> * 發佈
> * 從端點取得意圖和實體

[!include[LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-regex-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `list`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。 


## <a name="moveemployee-intent"></a>MoveEmployee 意圖

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 選取 [Create new intent] \(建立新意圖\)。 

3. 在快顯對話方塊方塊中輸入 `MoveEmployee`，然後選取 [完成]。 

    ![建立新意圖對話方塊的螢幕擷取畫面](./media/luis-quickstart-intent-and-list-entity/hr-create-new-intent-ddl.png)

4. 將語句範例新增至意圖。

    |範例語句|
    |--|
    |move John W. Smith from B-1234 to H-4452|
    |mv john.w.smith@mycompany.com from office b-1234 to office h-4452|
    |shift x12345 to h-1234 tomorrow|
    |place 425-555-1212 in HH-2345|
    |move 123-45-6789 from A-4321 to J-23456|
    |mv Jill Jones from D-2345 to J-23456|
    |shift jill-jones@mycompany.com to M-12345|
    |x23456 to M-12345|
    |425-555-0000 to h-4452|
    |234-56-7891 to hh-2345|

    [ ![醒目提示新語句的意圖頁面螢幕擷取畫面](./media/luis-quickstart-intent-and-list-entity/hr-enter-utterances.png) ](./media/luis-quickstart-intent-and-list-entity/hr-enter-utterances.png#lightbox)

    請記住，先前的教學課程中已新增 number 和 datetimeV2，系統會將任何語句範例中所找到的這兩個項目自動標記。

    [!include[Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]  

## <a name="employee-list-entity"></a>員工清單實體
現在 **MoveEmployee** 意圖已具有語句範例，LUIS 接著必須了解員工是什麼。 

每個項目的主要「正式」名稱都是員工編號。 在此領域中，每個正式名稱同義字的範例如下： 

|同義字用途|同義字值|
|--|--|
|名稱|John W. Smith|
|電子郵件地址|john.w.smith@mycompany.com|
|電話分機|x12345|
|個人行動電話號碼|425-555-1212|
|美國聯邦社會安全碼|123-45-6789|


1. 在左側面板中選取 [實體]。

2. 選取 [Create new entity] \(建立新實體\)。

3. 在實體快顯對話方塊中，輸入 `Employee` 作為實體名稱，以及輸入 [清單] 作為實體類型。 選取 [完成] 。  

    [![](media/luis-quickstart-intent-and-list-entity/hr-list-entity-ddl.png "建立新實體快顯對話方塊的螢幕擷取畫面")](media/luis-quickstart-intent-and-list-entity/hr-list-entity-ddl.png#lightbox)

4. 在 [員工實體] 頁面上，輸入 `Employee-24612` 作為新值。

    [![](media/luis-quickstart-intent-and-list-entity/hr-emp1-value.png "輸入值的螢幕擷取畫面")](media/luis-quickstart-intent-and-list-entity/hr-emp1-value.png#lightbox)

5. 針對 [同義字]，新增下列值：

    |同義字用途|同義字值|
    |--|--|
    |名稱|John W. Smith|
    |電子郵件地址|john.w.smith@mycompany.com|
    |電話分機|x12345|
    |個人行動電話號碼|425-555-1212|
    |美國聯邦社會安全碼|123-45-6789|

    [![](media/luis-quickstart-intent-and-list-entity/hr-emp1-synonyms.png "輸入同義字的螢幕擷取畫面")](media/luis-quickstart-intent-and-list-entity/hr-emp1-synonyms.png#lightbox)

6. 輸入 `Employee-45612` 作為新值。

7. 針對 [同義字]，新增下列值：

    |同義字用途|同義字值|
    |--|--|
    |名稱|Jill Jones|
    |電子郵件地址|jill-jones@mycompany.com|
    |電話分機|x23456|
    |個人行動電話號碼|425-555-0000|
    |美國聯邦社會安全碼|234-56-7891|

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)] 

2. 移至位址中的 URL 結尾並輸入 `shift 123-45-6789 from Z-1242 to T-54672`。 最後一個 querystring 參數是 `q`，也就是 **q**uery 語句。 此語句與任何標示的語句都不同，因此這是很好的測試，且應該傳回 `MoveEmployee` 意圖及所擷取的 `Employee`。

  ```JSON
  {
    "query": "shift 123-45-6789 from Z-1242 to T-54672",
    "topScoringIntent": {
      "intent": "MoveEmployee",
      "score": 0.9882801
    },
    "intents": [
      {
        "intent": "MoveEmployee",
        "score": 0.9882801
      },
      {
        "intent": "FindForm",
        "score": 0.016044287
      },
      {
        "intent": "GetJobInformation",
        "score": 0.007611245
      },
      {
        "intent": "ApplyForJob",
        "score": 0.007063288
      },
      {
        "intent": "Utilities.StartOver",
        "score": 0.00684710965
      },
      {
        "intent": "None",
        "score": 0.00304174074
      },
      {
        "intent": "Utilities.Help",
        "score": 0.002981
      },
      {
        "intent": "Utilities.Confirm",
        "score": 0.00212222221
      },
      {
        "intent": "Utilities.Cancel",
        "score": 0.00191026414
      },
      {
        "intent": "Utilities.Stop",
        "score": 0.0007461446
      }
    ],
    "entities": [
      {
        "entity": "123 - 45 - 6789",
        "type": "Employee",
        "startIndex": 6,
        "endIndex": 16,
        "resolution": {
          "values": [
            "Employee-24612"
          ]
        }
      },
      {
        "entity": "123",
        "type": "builtin.number",
        "startIndex": 6,
        "endIndex": 8,
        "resolution": {
          "value": "123"
        }
      },
      {
        "entity": "45",
        "type": "builtin.number",
        "startIndex": 10,
        "endIndex": 11,
        "resolution": {
          "value": "45"
        }
      },
      {
        "entity": "6789",
        "type": "builtin.number",
        "startIndex": 13,
        "endIndex": 16,
        "resolution": {
          "value": "6789"
        }
      },
      {
        "entity": "-1242",
        "type": "builtin.number",
        "startIndex": 24,
        "endIndex": 28,
        "resolution": {
          "value": "-1242"
        }
      },
      {
        "entity": "-54672",
        "type": "builtin.number",
        "startIndex": 34,
        "endIndex": 39,
        "resolution": {
          "value": "-54672"
        }
      }
    ]
  }
  ```

  找到員工並以 `Employee` 類型傳回，其解析值是 `Employee-24612`。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟
本教學課程已建立新的意圖、新增語句範例，接著建立了清單實體以從語句中擷取完全相符的文字項目。 定型和發佈應用程式之後，端點的查詢識別了意圖並傳回擷取的資料。

> [!div class="nextstepaction"]
> [在應用程式中新增階層式實體](luis-quickstart-intent-and-hier-entity.md)

