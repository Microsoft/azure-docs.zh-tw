---
title: 教學課程 6：使用 LUIS 複合實體擷取複合資料
titleSuffix: Azure Cognitive Services
description: 新增複合實體，以便將擷取的各類型資料組合為單一包含實體。 用戶端應用程式可藉由組合資料，輕鬆地擷取不同資料類型的相關資料。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: 99e0b22b663f6edab9646111b390186a6f89a90f
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035176"
---
# <a name="tutorial-6-group-and-extract-related-data"></a>教學課程 6：將相關的資料組成群組並加以擷取
在本教學課程中，新增複合實體，以便將擷取的各類型資料組合為單一包含實體。 用戶端應用程式可藉由組合資料，輕鬆地擷取不同資料類型的相關資料。

複合實體的用途是將相關實體群組為父類別實體。 在建立複合項目之前，這些資訊會以個別實體的形式存在。 它類似於階層式實體，但可包含不同類型的實體。 

複合實體很適合用於這類資料，因為此資料：

* 彼此相關。 
* 使用各種實體類型。
* 需要由用戶端應用程式當作一個資訊單位進行分組和處理。

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增複合實體 
> * 定型
> * 發佈
> * 從端點取得意圖和實體

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-hier-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `composite`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。


## <a name="composite-entity"></a>複合實體
當個別實體可以邏輯方式組成群組，而此邏輯群組對用戶端應用程式有助益時，請建立複合實體。 

在此應用程式中，員工名稱會定義於 **Employee** 清單實體中，並包含名稱、電子郵件地址、公司電話分機、行動電話號碼和美國聯邦稅務識別碼的同義字。 

**MoveEmployee** 意圖具有範例語句，可要求員工從某棟建築物和辦公室搬遷到另一棟。 建築物名稱是英文字母："A"、"B" 等，而辦公室是數字："1234"、"13245"。 

**MoveEmployee** 意圖中的範例語句包括：

|範例語句|
|--|
|Move John W. Smith to a-2345 (將 John W. Smith 搬遷到 a-2345)|
|shift x12345 to h-1234 tomorrow|
 
搬遷要求應包含員工 (使用任何同義字)，以及最終的建築物和辦公室位置。 要求也可以包含原始辦公室，以及應進行搬遷的日期。 

從端點擷取的資料應該包含此資訊，並在 `RequestEmployeeMove` 複合實體中傳回它：

```JSON
"compositeEntities": [
  {
    "parentType": "RequestEmployeeMove",
    "value": "jill jones from a - 1234 to z - 2345 on march 3 2 p . m",
    "children": [
      {
        "type": "builtin.datetimeV2.datetime",
        "value": "march 3 2 p.m"
      },
      {
        "type": "Locations::Destination",
        "value": "z - 2345"
      },
      {
        "type": "Employee",
        "value": "jill jones"
      },
      {
        "type": "Locations::Origin",
        "value": "a - 1234"
      }
    ]
  }
]
```

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 在 [意圖] 頁面上，選取 [MoveEmployee] 意圖。 

3. 在工具列上選取放大鏡圖示來篩選語句清單。 

    [![](media/luis-tutorial-composite-entity/hr-moveemployee-magglass.png "在 'MoveEmployee' 意圖上已醒目提示放大鏡按鈕的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-moveemployee-magglass.png#lightbox)

4. 在篩選條件文字方塊中輸入 `tomorrow`，以尋找語句 `shift x12345 to h-1234 tomorrow`。

    [![](media/luis-tutorial-composite-entity/hr-filter-by-tomorrow.png "在 'MoveEmployee' 意圖上已醒目提示 'tomorrow' 篩選條件的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-filter-by-tomorrow.png#lightbox)

    另一個方法是選取 [實體篩選條件]，然後從清單中選取 [datetimeV2]，依 datetimeV2 篩選實體。 

5. 選取第一個實體 `Employee`，然後從快顯功能表清單中選取 [包裝於複合實體中]。 

    [![](media/luis-tutorial-composite-entity/hr-create-entity-1.png "在 'MoveEmployee' 意圖上選取複合中已醒目提示之第一個實體的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-create-entity-1.png#lightbox)


6. 接著，立即選取語句中的最後一個實體 `datetimeV2`。 系統會在所選取的文字下方繪製綠色橫條，來表示複合實體。 在快顯功能表中，輸入複合名稱 `RequestEmployeeMove`，然後選取 Enter。 

    [![](media/luis-tutorial-composite-entity/hr-create-entity-2.png "在 'MoveEmployee' 意圖上選取複合中已醒目提示的最後一個實體和建立實體的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-create-entity-2.png#lightbox)

7. 在 [您想要建立何種類型的實體?] 中，幾乎所有所需的欄位都位於清單中。 只會遺漏原始位置。 選取 [新增子實體]，從現有實體清單中選取 [Locations::Origin]，然後選取 [完成]。 

    請注意，預先建置的實體、數字會新增至複合實體。 如果您讓預先建置的實體出現在複合實體的開頭與結尾語彙基元之間，則複合實體必須包含這些預先建置的實體。 如果未包含預先建置的實體，則無法正確預測複合實體，但可正確預測每個個別的元素。

    ![在 'MoveEmployee' 意圖上於快顯視窗中新增另一個實體的 LUIS 螢幕擷取畫面](media/luis-tutorial-composite-entity/hr-create-entity-ddl.png)

8. 選取工具列上的放大鏡來移除篩選條件。 

9. 從篩選條件中移除 `tomorrow` 這個字，您便可再度看見所有範例語句。 

## <a name="label-example-utterances-with-composite-entity"></a>使用複合實體來標示範例語句


1. 在每個範例語句中，選取應該出現在複合中的最左邊實體。 接著，選取 [包裝於複合實體中]。

    [![](media/luis-tutorial-composite-entity/hr-label-entity-1.png "在 'MoveEmployee' 意圖上選取複合中已醒目提示之第一個實體的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-label-entity-1.png#lightbox)

2. 選取複合實體中的最後一個文字 ，然後從快顯功能表中選取 [RequestEmployeeMove]。 

    [![](media/luis-tutorial-composite-entity/hr-label-entity-2.png "在 'MoveEmployee' 意圖上選取複合中已醒目提示之最後一個實體的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-label-entity-2.png#lightbox)

3. 確認已使用複合實體來標示意圖中的所有語句。 

    [![](media/luis-tutorial-composite-entity/hr-all-utterances-labeled.png "'MoveEmployee' 上已標示所有語句的 LUIS 螢幕擷取畫面")](media/luis-tutorial-composite-entity/hr-all-utterances-labeled.png#lightbox)

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體 

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `Move Jill Jones from a-1234 to z-2345 on March 3 2 p.m.`。 最後一個查詢字串參數是 `q`，也就是語句查詢。 

    由於此測試是要確認已正確擷取複合，因此，測試可以包含現有的範例語句或新語句。 在複合實體中包含所有子實體是個很好的測試。

    ```JSON
    {
      "query": "Move Jill Jones from a-1234 to z-2345 on March 3  2 p.m",
      "topScoringIntent": {
        "intent": "MoveEmployee",
        "score": 0.9959525
      },
      "intents": [
        {
          "intent": "MoveEmployee",
          "score": 0.9959525
        },
        {
          "intent": "GetJobInformation",
          "score": 0.009858314
        },
        {
          "intent": "ApplyForJob",
          "score": 0.00728598563
        },
        {
          "intent": "FindForm",
          "score": 0.0058053555
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.005371796
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00266987388
        },
        {
          "intent": "None",
          "score": 0.00123299169
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.00116407464
        },
        {
          "intent": "Utilities.Confirm",
          "score": 0.00102653319
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.0006628214
        }
      ],
      "entities": [
        {
          "entity": "march 3 2 p.m",
          "type": "builtin.datetimeV2.datetime",
          "startIndex": 41,
          "endIndex": 54,
          "resolution": {
            "values": [
              {
                "timex": "XXXX-03-03T14",
                "type": "datetime",
                "value": "2018-03-03 14:00:00"
              },
              {
                "timex": "XXXX-03-03T14",
                "type": "datetime",
                "value": "2019-03-03 14:00:00"
              }
            ]
          }
        },
        {
          "entity": "jill jones",
          "type": "Employee",
          "startIndex": 5,
          "endIndex": 14,
          "resolution": {
            "values": [
              "Employee-45612"
            ]
          }
        },
        {
          "entity": "z - 2345",
          "type": "Locations::Destination",
          "startIndex": 31,
          "endIndex": 36,
          "score": 0.9690751
        },
        {
          "entity": "a - 1234",
          "type": "Locations::Origin",
          "startIndex": 21,
          "endIndex": 26,
          "score": 0.9713137
        },
        {
          "entity": "-1234",
          "type": "builtin.number",
          "startIndex": 22,
          "endIndex": 26,
          "resolution": {
            "value": "-1234"
          }
        },
        {
          "entity": "-2345",
          "type": "builtin.number",
          "startIndex": 32,
          "endIndex": 36,
          "resolution": {
            "value": "-2345"
          }
        },
        {
          "entity": "3",
          "type": "builtin.number",
          "startIndex": 47,
          "endIndex": 47,
          "resolution": {
            "value": "3"
          }
        },
        {
          "entity": "2",
          "type": "builtin.number",
          "startIndex": 50,
          "endIndex": 50,
          "resolution": {
            "value": "2"
          }
        },
        {
          "entity": "jill jones from a - 1234 to z - 2345 on march 3 2 p . m",
          "type": "RequestEmployeeMove",
          "startIndex": 5,
          "endIndex": 54,
          "score": 0.4027723
        }
      ],
      "compositeEntities": [
        {
          "parentType": "RequestEmployeeMove",
          "value": "jill jones from a - 1234 to z - 2345 on march 3 2 p . m",
          "children": [
            {
              "type": "builtin.datetimeV2.datetime",
              "value": "march 3 2 p.m"
            },
            {
              "type": "Locations::Destination",
              "value": "z - 2345"
            },
            {
              "type": "Employee",
              "value": "jill jones"
            },
            {
              "type": "Locations::Origin",
              "value": "a - 1234"
            }
          ]
        }
      ]
    }
    ```

  這個語句會傳回複合實體陣列。 系統會為每個實體指定類型和值。 若要針對每個子實體獲取更高的精確度，請使用來自複合陣列項目的類型和值組合，來尋找實體陣列中的對應項目。  

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

本教學課程建立了複合實體，以便封裝現有的實體。 這可讓用戶端應用程式在不同的資料類型中尋找一組相關資料，以便繼續交談。 此人力資源應用程式的用戶端應用程式可以詢問需要開始及結束搬遷的日期和時間。 也可以詢問搬遷的其他搬運事宜，例如實體電話。 

> [!div class="nextstepaction"] 
> [了解如何新增簡單實體和片語清單](luis-quickstart-primary-and-secondary-data.md)  