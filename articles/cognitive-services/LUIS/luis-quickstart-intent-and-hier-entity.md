---
title: 教學課程 5：父代/子系關聯性 - LUIS 透過內容學習資料的階層式實體
titleSuffix: Azure Cognitive Services
description: 根據內容尋找相關的資料片段。 例如，從一個建築物和辦公室移到另一個建築物和辦公室的實體移動原點和目的地位置會相關。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: a469bc600715b2e276d6654596da50d75659aadb
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831019"
---
# <a name="tutorial-5-extract-contextually-related-data"></a>教學課程 5：擷取內容相關的資料
在本教學課程中，根據內容尋找相關的資料片段。 例如，從一個建築物和辦公室移到另一個建築物和辦公室的實體移動原點和目的地位置會相關。 若要產生一份工單，則需要這兩個資料片段，而且兩者彼此相關。  

此應用程式會判斷員工是否要從原點位置 (建築物和辦公室) 移到目的地位置 (建築物和辦公室)。 它會使用階層式實體來判斷語句中的位置。 **階層式**實體的目的是根據內容尋找語句內的相關資料。 

階層式實體適用於此類型資料，因為這兩個資料片段是：

* 簡單的實體。
* 在語句的內容中彼此相關。
* 使用特定文字來表示每個位置。 這些字的範例包括：from/to、leaving/headed to、away from/toward。
* 這兩個位置通常會在相同的語句中。 
* 需要由用戶端應用程式當作一個資訊單位進行分組和處理。

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增意圖 
> * 新增具有出發地和目的地子系的位置階層式實體
> * 定型
> * 發佈
> * 從端點取得意圖和實體

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-list-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `hier`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。 

## <a name="remove-prebuilt-number-entity-from-app"></a>從應用程式中移除預先建立的數字實體
若要查看整個語句並標記階層式子系，請暫時移除預先建立的數字實體。

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 從左側功能表中選取 [實體]。

3. 選取清單中數字實體左邊的核取方塊。 選取 [刪除] 。 

## <a name="add-utterances-to-moveemployee-intent"></a>將語句新增至 MoveEmployee 意圖

1. 選取左功能表中的 [意圖]。

2. 從意圖清單中選取 [MoveEmployee]。

3. 新增下列範例語句：

    |範例語句|
    |--|
    |Move John W. Smith **to** a-2345|
    |Direct Jill Jones **to** b-3499|
    |Organize the move of x23456 **from** hh-2345 **to** e-0234|
    |Begin paperwork to set x12345 **leaving** a-3459 **headed to** f-34567|
    |Displace 425-555-0000 **away from** g-2323 **toward** hh-2345|

    [ ![MoveEmployee 意圖中有新語句的 LUIS 螢幕擷取畫面](./media/luis-quickstart-intent-and-hier-entity/hr-enter-utterances.png)](./media/luis-quickstart-intent-and-hier-entity/hr-enter-utterances.png#lightbox)

    在[清單實體](luis-quickstart-intent-and-list-entity.md)教學課程中，會依照名稱、電子郵件地址、電話分機、行動電話號碼或美國聯邦社會安全號碼指定員工。 這些員工編號會使用於語句中。 先前的範例語句包含不同的方式可記下原點和目的地位置 (以粗體顯示標記)。 有些語句特意只有目的地。 這有助於 LUIS 了解如何在未指定原點時將這些位置放在語句中。     

    [!include[Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]  

## <a name="create-a-location-entity"></a>建立位置實體
LUIS 必須藉由在語句中標記原點和目的地，進而了解位置為何。 如果您需要在語彙基元 (原始) 檢視中查看語句，請在導覽列中選取標示為 [實體檢視] 的語句上方的切換鍵。 切換參數之後，控制項會標示為 [語彙基元檢視]。

請考慮使用下列語句：

```JSON
mv Jill Jones from a-2349 to b-1298
```

此語句已指定兩個位置：`a-2349` 和 `b-1298`。 假設字母會對應至建築物名稱，而數字則表示該建築物內的辦公室。 兩者都群組為階層式實體 (`Locations`) 的子系是合理的，因為必須從語句中擷取這兩個資料片段，才能在用戶端應用程式中完成要求，而且兩者彼此相關。 
 
如果只有一個階層式實體的子系 (出發或目的地位置) 存在，仍然會進行擷取。 只要擷取一個或部分子系時，不需要找到所有子系。 

1. 在 `Displace 425-555-0000 away from g-2323 toward hh-2345` 語句中，選取 `g-2323` 這個字。 隨即出現頂端有文字方塊的下拉式功能表。 在文字方塊中輸入實體名稱 `Locations`，然後在下拉式功能表中選取 [建立新的實體]。 

    [![](media/luis-quickstart-intent-and-hier-entity/hr-create-new-entity-1.png "在意圖頁面上建立新實體的螢幕擷取畫面")](media/luis-quickstart-intent-and-hier-entity/hr-create-new-entity-1.png#lightbox)

2. 在快顯視窗中，選取 [階層式] 實體類型，並使用 `Origin` 和 `Destination` 作為子實體。 選取 [完成] 。

    ![](media/luis-quickstart-intent-and-hier-entity/hr-create-new-entity-2.png "新位置實體的實體建立快顯對話方塊螢幕擷取畫面")

3. `g-2323` 的標籤標示為 `Locations`，因為 LUIS 不知道該字詞是出發或目的地位置，或兩者皆非。 選取 `g-2323`，然後選取 [位置]，接著遵循右側功能表並選取 `Origin`。

    [![](media/luis-quickstart-intent-and-hier-entity/hr-label-entity.png "用以變更位置實體子系的實體標記快顯對話方塊螢幕擷取畫面")](media/luis-quickstart-intent-and-hier-entity/hr-label-entity.png#lightbox)

5. 選取語句中的建築物和辦公室，然後選取 [位置]，並遵循右側功能表來選取 `Origin` 或 `Destination`以標示所有其他語句中的其他位置。 標示所有位置後，[語彙基元檢視] 中的語句就開始看起來像是一個模式。 

    [![](media/luis-quickstart-intent-and-hier-entity/hr-entities-labeled.png "語句中已標記位置實體的螢幕擷取畫面")](media/luis-quickstart-intent-and-hier-entity/hr-entities-labeled.png#lightbox)

## <a name="add-prebuilt-number-entity-to-app"></a>將預先建立的數字實體新增至應用程式
將預先建立的數字實體新增回應用程式。

1. 從左側導覽功能表中選取 [Entities] \(實體\)。

2. 選取 [Manage prebuilt entities] \(管理預先建置的實體\) 按鈕。

3. 從預先建立的實體清單中選取 [數字]，然後選取 [完成]。

    ![預先建置的實體對話方塊中已選取 number 的螢幕擷取畫面](./media/luis-quickstart-intent-and-hier-entity/hr-add-number-back-ddl.png)

## <a name="train-the-luis-app"></a>進行 LUIS 應用程式定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-the-app-to-get-the-endpoint-url"></a>發佈應用程式以取得端點 URL

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="query-the-endpoint-with-a-different-utterance"></a>使用不同的語句來查詢端點

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]


2. 移至位址中的 URL 尾端並輸入 `Please relocation jill-jones@mycompany.com from x-2345 to g-23456`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 此語句與任何已標記的語句都不同，因此這是一個良好的測試，而應該會將 `MoveEmployee` 意圖與所擷取的階層式實體一起傳回。

    ```JSON
    {
      "query": "Please relocation jill-jones@mycompany.com from x-2345 to g-23456",
      "topScoringIntent": {
        "intent": "MoveEmployee",
        "score": 0.9966052
      },
      "intents": [
        {
          "intent": "MoveEmployee",
          "score": 0.9966052
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.0325253047
        },
        {
          "intent": "FindForm",
          "score": 0.006137873
        },
        {
          "intent": "GetJobInformation",
          "score": 0.00462633232
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.00415637763
        },
        {
          "intent": "ApplyForJob",
          "score": 0.00382325822
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00249120337
        },
        {
          "intent": "None",
          "score": 0.00130756292
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.00119622645
        },
        {
          "intent": "Utilities.Confirm",
          "score": 1.26910036E-05
        }
      ],
      "entities": [
        {
          "entity": "jill - jones @ mycompany . com",
          "type": "Employee",
          "startIndex": 18,
          "endIndex": 41,
          "resolution": {
            "values": [
              "Employee-45612"
            ]
          }
        },
        {
          "entity": "x - 2345",
          "type": "Locations::Origin",
          "startIndex": 48,
          "endIndex": 53,
          "score": 0.8520272
        },
        {
          "entity": "g - 23456",
          "type": "Locations::Destination",
          "startIndex": 58,
          "endIndex": 64,
          "score": 0.974032
        },
        {
          "entity": "-2345",
          "type": "builtin.number",
          "startIndex": 49,
          "endIndex": 53,
          "resolution": {
            "value": "-2345"
          }
        },
        {
          "entity": "-23456",
          "type": "builtin.number",
          "startIndex": 59,
          "endIndex": 64,
          "resolution": {
            "value": "-23456"
          }
        }
      ]
    }
    ```
    
    已預測正確的意圖，而且實體陣列在對應的 **entity** 屬性中具有原點和目的地值。
    

## <a name="could-you-have-used-a-regular-expression-for-each-location"></a>是否已針對每個位置使用規則運算式？
是，使用原點和目的地角色建立規則運算式實體，並在模式中使用它。

此範例中的位置 (例如 `a-1234`) 會遵循特定格式：一或兩個子母加上破折號，後面接著一系列 4 或 5 個數字。 此資料的描述為每個位置各有一個角色的規則運算式實體。 這些角色僅適用於模式。 您可以根據這些語句建立模式，然後建立位置格式的規則運算式，並將它新增至模式。 

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟
本教學課程建立了新意圖，並針對原點和目的地位置的內容學習資料新增了範例語句。 應用程式一旦經過訓練並發佈，用戶端應用程式即可使用該資訊來建立包含相關資訊的移動票證。

> [!div class="nextstepaction"] 
> [了解如何新增複合實體](luis-tutorial-composite-entity.md) 