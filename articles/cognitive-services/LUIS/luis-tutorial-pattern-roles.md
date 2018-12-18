---
title: 教學課程 4：內容相關資料的模式角色
titleSuffix: Azure Cognitive Services
description: 使用模式來從正確格式的語句範本擷取資料。 語句範本會使用簡單的實體和角色來擷取相關資料，例如原始位置和目的地位置。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: c7e7b100242d6ceb35172b872f2fb6ff7f4b402b
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48886155"
---
# <a name="tutorial-4-extract-contextually-related-patterns"></a>教學課程 4：擷取內容相關的模式

在本教學課程中，使用模式來從正確格式的語句範本擷取資料。 語句範本會使用簡單的實體和角色來擷取相關資料，例如原始位置和目的地位置。  使用模式時，需要針對意圖使用較少的範例語句。

角色的用途是在語句中擷取內容相關的實體。 在 `Move new employee Robert Williams from Sacramento and San Francisco` 語句中，來源城市與目的地城市的值彼此相關，並使用通用語言來表示每個位置。 


新進員工的名稱 Billy Patterson 尚不屬於**員工**清單實體的一部分。 先擷取新的員工名稱，才能將該名稱傳送到外部系統來建立公司認證。 建立公司認證之後，會將員工認證新增到**員工**清單實體。

新進員工和家人需要從目前的城市搬遷到虛構公司所在的城市。 由於新進員工可能來自任何城市，因此，必須先探索位置。 諸如清單實體之類的集合清單不會有任何作用，因為只會擷取清單中的城市。

與來源和目的地城市相關聯的角色名稱在所有實體中必須是唯一的。 確定角色是唯一的簡單方式是透過命名策略，將它們繫結至包含實體。 **NewEmployeeRelocation** 實體是一個簡單實體且含有兩個角色：**NewEmployeeReloOrigin** 和 **NewEmployeeReloDestination**。 Relo 是 relocation (重新配置) 的簡寫。

由於範例語句 `Move new employee Robert Williams from Sacramento and San Francisco` 只具備機器學習的實體，因此，請務必為意圖提供足夠的範例語句，如此才能偵測到實體。  

**當模式可讓您提供較少的範例語句時，如果偵測不到實體，模式就不會進行比對。**

如果您因為簡單實體是名稱 (例如城市) 而難以偵測簡單實體，請考慮新增類似值的片語清單。 這可藉由為 LUIS 提供關於該類型字組或片語的其他信號，來協助偵測城市名稱。 片語清單只能藉由協助偵測實體來為模式提供協助，對於要比對的模式而言，實體偵測是必要的。 

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 建立新實體
> * 建立新意圖
> * 定型
> * 發佈
> * 從端點取得意圖和實體
> * 使用角色建立模式
> * 建立城市的片語清單
> * 從端點取得意圖和實體

[!include[LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式
以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-patterns-HumanResources-v2.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `roles`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

## <a name="create-new-entities"></a>建立新實體

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 從左側瀏覽列中，選取 [實體]。 

3. 選取 [Create new entity] \(建立新實體\)。

4. 在快顯視窗中，輸入 `NewEmployee` 作為**簡單**實體。

5. 選取 [Create new entity] \(建立新實體\)。

6. 在快顯視窗中，輸入 `NewEmployeeRelocation` 作為**簡單**實體。

7. 從實體清單中，選取 [NewEmployeeRelocation]。 

8. 輸入第一個角色作為 `NewEmployeeReloOrigin`，然後選取 Enter。

9. 輸入第二個角色作為 `NewEmployeeReloDestination`，然後選取 Enter。

## <a name="create-new-intent"></a>建立新意圖
如果預先建置的 keyPhrase 實體已在開始之前移除，接著在您完成本節中的步驟之後新增回來，則在這些步驟中標示實體可能比較簡單。 

1. 從左側導覽窗格中，選取 [Intents] \(意圖\)。

2. 選取 [Create new intent] \(建立新意圖\)。 

3. 在快顯對話方塊中，輸入 `NewEmployeeRelocationProcess` 作為意圖名稱。

4. 輸入下列範例語句，標示新實體。 實體和角色值均以粗體顯示。 若切換到 [語彙基元檢視] 能更輕鬆地標示文字，請切換到該檢視。 

    當您在意圖中標示實體時並未指定實體的角色。 您稍後會在建立模式時執行該動作。 

    |語句|NewEmployee|NewEmployeeRelocation|
    |--|--|--|
    |將 **Bob Jones** 從 **Seattle** 搬遷到 **Los Colinas**|Bob Jones|Seattle、Los Colinas|
    |將 **Dave C.Cooper** 從 **Redmond** 搬遷到 **New York City**|Dave C.Cooper|Redmond、New York City|
    |將 **Jim Paul Smith** 從 **Toronto** 搬遷到 **West Vancouver**|Jim Paul Smith|Toronto、West Vancouver|
    |將 **J.Benson** 從 **Boston** 搬遷到 **Staines-upon-Thames**|J. Benson|Boston、Staines-upon-Thames|
    |將 **Travis "Trav" Hinton** 從 **Castelo Branco** 搬遷到 **Orlando**|Travis "Trav" Hinton|Castelo Branco、Orlando|
    |將 **Trevor Nottington III** 從 **Aranda de Duero** 搬遷到 **Boise**|Trevor Nottington III|Aranda de Duero、Boise|
    |將 **Dr.Greg Williams** 從 **Orlando** 搬遷到 **Ellicott City**|Dr.Greg Williams|Orlando、Ellicott City|
    |將 **Robert "Bobby" Gregson** 從 **Kansas City** 搬遷到 **San Juan Capistrano**|Robert "Bobby" Gregson|Kansas City、San Juan Capistrano|
    |將 **Patti Owens** 從 **Bellevue** 搬遷到 **Rockford**|Patti Owens|Bellevue、Rockford|
    |將 **Janet Bartlet** 從 **Tuscan** 搬遷到 **Santa Fe**|Janet Bartlet|Tuscan、Santa Fe|

    員工名稱有各種不同的前置詞、字數統計、語法和後置詞。 這對於 LUIS 了解新員工名稱的變化非常重要。 城市名稱也有各種不同的字數統計和語法。 此多樣性對於教導 LUIS 了解這些實體可能以何種方式出現在使用者的語句中非常重要。 
    
    如果任一個實體具有相同的字數且沒有任何其他變化，則您會教導 LUIS 了解此實體只有該字數且沒有任何其他變化。 LUIS 無法正確預測一組廣泛的變化，因為並未向它顯示任何變化。 

    如果您移除了 keyPhrase 實體，請立即將它新增回應用程式。

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)] 

2. 移至位址中的 URL 結尾並輸入 `Move Wayne Berry from Miami to Mount Vernon`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 

    ```JSON
    {
      "query": "Move Wayne Berry from Newark to Columbus",
      "topScoringIntent": {
        "intent": "NewEmployeeRelocationProcess",
        "score": 0.514479756
      },
      "intents": [
        {
          "intent": "NewEmployeeRelocationProcess",
          "score": 0.514479756
        },
        {
          "intent": "Utilities.Confirm",
          "score": 0.017118983
        },
        {
          "intent": "MoveEmployee",
          "score": 0.009982505
        },
        {
          "intent": "GetJobInformation",
          "score": 0.008637771
        },
        {
          "intent": "ApplyForJob",
          "score": 0.007115978
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.006120186
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.00452428637
        },
        {
          "intent": "None",
          "score": 0.00400899537
        },
        {
          "intent": "OrgChart-Reports",
          "score": 0.00240071164
        },
        {
          "intent": "Utilities.Help",
          "score": 0.001770991
        },
        {
          "intent": "EmployeeFeedback",
          "score": 0.001697356
        },
        {
          "intent": "OrgChart-Manager",
          "score": 0.00168116146
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.00163952739
        },
        {
          "intent": "FindForm",
          "score": 0.00112958835
        }
      ],
      "entities": [
        {
          "entity": "wayne berry",
          "type": "NewEmployee",
          "startIndex": 5,
          "endIndex": 15,
          "score": 0.629158735
        },
        {
          "entity": "newark",
          "type": "NewEmployeeRelocation",
          "startIndex": 22,
          "endIndex": 27,
          "score": 0.638941
        }
      ]
    }  
    ```

意圖預測分數大約只有 50%。 如果您的用戶端應用程式需要較高的數字，則需要修正此項目。 此外，也無法預測實體。

會擷取其中一個位置，但不會擷取另一個。 

模式將協助預測分數，不過，必須先正確預測實體之後，模式才會比對語句。 

## <a name="pattern-with-roles"></a>具有角色的模式

1. 在上方瀏覽列中，選取 [建置]。

2. 在左側瀏覽列中，選取 [模式]。

3. 從 [選取意圖] 下拉式清單中，選取 [NewEmployeeRelocationProcess]。 

4. 輸入下列模式：`move {NewEmployee} from {NewEmployeeRelocation:NewEmployeeReloOrigin} to {NewEmployeeRelocation:NewEmployeeReloDestination}[.]`

    如果您針對端點進行定型、發佈及查詢，則您可能會因為看到找不到實體而感到失望，因此，模式不會進行比對，因而不會改善預測。 這是因為沒有足夠的範例語句含有已標示實體而導致的結果。 不要新增更多範例，而是新增片語清單來修正此問題。

## <a name="cities-phrase-list"></a>城市片語清單
城市就像人名一樣很棘手，因為它們可以是字組與標點符號的任意組合。 區域和世界的城市都是已知的，因此 LUIS 需要城市的片語清單，才能開始學習。 

1. 從左側功能表的 [改善應用程式效能] 區段中，選取 [片語清單]。 

2. 將清單命名為 `Cities`，並為清單新增下列 `values`：

    |片語清單的值|
    |--|
    |Seattle|
    |San Diego|
    |New York City|
    |洛杉磯|
    |Portland|
    |Philadephia|
    |邁阿密|
    |達拉斯|

    請勿新增世界上的每個城市，也不要新增區域中的每個城市。 LUIS 必須能夠從清單中推論出城市的概況。 請務必讓 [這些值是可交換的] 保持選取狀態。 此設定表示清單上的字組會被視為同義字。 

3. 訓練和發佈應用程式。

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 移至位址中的 URL 結尾並輸入 `Move wayne berry from miami to mount vernon`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 

    ```JSON
    {
      "query": "Move Wayne Berry from Miami to Mount Vernon",
      "topScoringIntent": {
        "intent": "NewEmployeeRelocationProcess",
        "score": 0.9999999
      },
      "intents": [
        {
          "intent": "NewEmployeeRelocationProcess",
          "score": 0.9999999
        },
        {
          "intent": "Utilities.Confirm",
          "score": 1.49678385E-06
        },
        {
          "intent": "MoveEmployee",
          "score": 8.240291E-07
        },
        {
          "intent": "GetJobInformation",
          "score": 6.3131273E-07
        },
        {
          "intent": "None",
          "score": 4.25E-09
        },
        {
          "intent": "OrgChart-Manager",
          "score": 2.8E-09
        },
        {
          "intent": "OrgChart-Reports",
          "score": 2.8E-09
        },
        {
          "intent": "EmployeeFeedback",
          "score": 1.64E-09
        },
        {
          "intent": "Utilities.StartOver",
          "score": 1.64E-09
        },
        {
          "intent": "Utilities.Help",
          "score": 1.48181822E-09
        },
        {
          "intent": "Utilities.Stop",
          "score": 1.48181822E-09
        },
        {
          "intent": "Utilities.Cancel",
          "score": 1.35E-09
        },
        {
          "intent": "FindForm",
          "score": 1.23846156E-09
        },
        {
          "intent": "ApplyForJob",
          "score": 5.692308E-10
        }
      ],
      "entities": [
        {
          "entity": "wayne berry",
          "type": "builtin.keyPhrase",
          "startIndex": 5,
          "endIndex": 15
        },
        {
          "entity": "miami",
          "type": "builtin.keyPhrase",
          "startIndex": 22,
          "endIndex": 26
        },
        {
          "entity": "wayne berry",
          "type": "NewEmployee",
          "startIndex": 5,
          "endIndex": 15,
          "score": 0.9410646,
          "role": ""
        },
        {
          "entity": "miami",
          "type": "NewEmployeeRelocation",
          "startIndex": 22,
          "endIndex": 26,
          "score": 0.9853915,
          "role": "NewEmployeeReloOrigin"
        },
        {
          "entity": "mount vernon",
          "type": "NewEmployeeRelocation",
          "startIndex": 31,
          "endIndex": 42,
          "score": 0.986044347,
          "role": "NewEmployeeReloDestination"
        }
      ],
      "sentimentAnalysis": {
        "label": "neutral",
        "score": 0.5
      }
}
    ```

意圖分數現在會高出許多，而且角色名稱為實體回應的一部分。

## <a name="hierarchical-entities-versus-roles"></a>階層式實體與角色

在[階層式教學課程](luis-quickstart-intent-and-hier-entity.md)中，**MoveEmployee** 意圖偵測到現有員工從某一棟建築物和辦公室搬遷另一棟的時機。 範例語句含有來源和目的地位置，但未使用角色。 相反地，來源和目的地都是階層式實體的子系。 

在本教學課程中，人力資源應用程式會偵測將新員工從某個城市搬遷到另一個的相關語句。 語句的這兩種類型相同，但會使用不同的 LUIS 能力來加以解決。

|教學課程|範例語句|來源和目的地位置|
|--|--|--|
|[階層式 (沒有角色)](luis-quickstart-intent-and-hier-entity.md)|將 Jill Jones 從 **a-2349** 搬遷到 **b-1298**|a-2349、b-1298|
|本教學課程 (含有角色)|將 Billy Patterson 從 **Yuma** 搬遷到 **Denver**。|Yuma、Denver|

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

本教學課程已新增具有角色的實體和具有語句範例的意圖。 正確使用實體的第一個端點預測已預測出意圖但信賴分數不高。 只偵測到兩個實體的其中一個。 接下來，本教學課程新增了一種使用實體角色的模式，以及用來提升語句中城市名稱價值的片語清單。 第二個端點預測傳回了高信賴分數，並找到這兩個實體角色。 

> [!div class="nextstepaction"]
> [了解 LUIS 應用程式的最佳做法](luis-concept-best-practices.md)
