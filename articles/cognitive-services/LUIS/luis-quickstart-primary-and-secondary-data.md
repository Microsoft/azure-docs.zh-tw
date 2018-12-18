---
title: 教學課程 7：LUIS 中具有片語清單的簡單實體
titleSuffix: Azure Cognitive Services
description: 從語句中擷取機器學習的資料
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: 941c29506aa8f17dcb6262495b28dd26e78194d5
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036046"
---
# <a name="tutorial-7-extract-names-with-simple-entity-and-phrase-list"></a>教學課程 7：利用簡單實體和片語清單擷取名稱

在本教學課程中，使用**簡單**實體從語句中擷取雇用職位名稱的機器學習資料。 若要增加擷取精確度，請新增簡單實體特有的字詞片語清單。

本教學課程會新增用來擷取職位名稱的簡單實體。 在這個 LUIS 應用程式中，簡單實體的用途是要教導 LUIS 何謂職位名稱，以及職位名稱位於語句何處。 視字組選擇與語句長度而定，每個語句中屬於職位名稱的部分各不相同。 LUIS 需要使用職位名稱的所有意圖中的職位名稱範例。  

在下列情況下，簡單實體很適合用於這類資料：

* 資料是單一概念。
* 資料的格式不正確，例如規則運算式。
* 資料不常見，例如電話號碼或資料的預先建置實體。
* 資料未完全符合已知字組清單，例如清單實體。
* 資料不包含其他資料項目，例如複合實體或階層式實體。

**在本教學課程中，您將了解如何：**

<!-- green checkmark -->
> [!div class="checklist"]
> * 使用現有的教學課程應用程式
> * 新增會從應用程式擷取職位的簡單實體
> * 新增可提升職位字組訊號的片語清單
> * 定型 
> * 發佈 
> * 從端點取得意圖和實體

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="use-existing-app"></a>使用現有的應用程式

以上一個教學課程中建立的應用程式繼續進行，其名稱為 **HumanResources**。 

如果您沒有來自上一個教學課程的 HumanResources 應用程式，請使用下列步驟：

1.  下載並儲存[應用程式的 JSON 檔案](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/custom-domain-composite-HumanResources.json)。

2. 將 JSON 匯入新的應用程式中。

3. 從 [管理] 區段的 [版本] 索引標籤上，複製版本並將它命名為 `simple`。 複製是一個既可測試各種 LUIS 功能又不影響原始版本的絕佳方式。 因為版本名稱會作為 URL 路由的一部分，所以此名稱不能包含任何在 URL 中無效的字元。

## <a name="simple-entity"></a>簡單實體
簡單實體會偵測字組或片語中包含的單一資料概念。

請考慮下列來自聊天機器人的語句：

|語句|可擷取的職位名稱|
|:--|:--|
|我想要應徵新的會計職位。|會計|
|投遞我的履歷以應徵工程師職缺。|工程師|
|填寫職位 123456 的應徵文件|123456|

職位名稱可能是名詞、動詞或由幾個字組所形成的片語，因此職位名稱很難判斷。 例如︰

|工作|
|--|
|工程師|
|軟體工程師|
|資深軟體工程師|
|工程師團隊主管 |
|航空管制官|
|汽車司機|
|救護車司機|
|交通船|
|擠出機|
|裝配工|

此 LUIS 應用程式在數種意圖中具有職位名稱。 藉由對所有意圖語句中的這些字組加上標籤，LUIS 會更加了解何謂職位名稱，以及職位位於語句何處。

標示範例語句中的實體後，請務必新增片語清單，以提升簡單實體的訊號。 片語清單**不會**作為完全相符項目，而且不必是您預期的每個可能值。 

1. [!include[Start in Build section](../../../includes/cognitive-services-luis-tutorial-build-section.md)]

2. 在 [意圖] 頁面上，選取 [ApplyForJob] 意圖。 

3. 在語句 `I want to apply for the new accounting job` 中選取 [`accounting`]，於快顯功能表最上面的欄位中輸入 `Job`，然後選取快顯功能表中的 [建立新實體]。 

    [![](media/luis-quickstart-primary-and-secondary-data/hr-create-entity.png "已醒目提示 [ApplyForJob] 意圖與建立實體步驟的 LUIS 螢幕擷取畫面")](media/luis-quickstart-primary-and-secondary-data/hr-create-entity.png#lightbox)

4. 在快顯視窗中，確認實體名稱和類型，然後選取 [完成]。

    ![建立簡單實體快顯強制回應對話方塊與職位名稱和簡單類型](media/luis-quickstart-primary-and-secondary-data/hr-create-simple-entity-popup.png)

5. 在語句 `Submit resume for engineering position` 中，將 `engineering` 這個字組標記為 [職位] 實體。 選取 `engineering` 字組，然後從快顯功能表中選取 [職位]。 

    [![](media/luis-quickstart-primary-and-secondary-data/hr-label-simple-entity.png "已醒目提示並標記職位實體的 LUIS 螢幕擷取畫面")](media/luis-quickstart-primary-and-secondary-data/hr-label-simple-entity.png#lightbox)

    所有語句都會加上標籤，但只有五個語句不足以讓 LUIS 認識職位相關字組和片語。 使用編號值的職位會使用規則運算式實體，因此不需要更多範例。 屬於字組或片語的職位需要至少 15 個以上的範例。 

6. 新增更多語句，並將職位字組或片語標記為**職位**實體。 職位是就業服務的通用就業類型。 如果您想讓職位與特定產業相關聯，職位字組就應該反映這一點。 

    |語句|職位實體|
    |:--|:--|
    |我要應徵研發部門的項目經理職位|項目經理|
    |以下是我的幫廚應用程式。|幫廚|
    |隨附我的夏令營輔導員履歷。|夏令營輔導員|
    |這是我的履歷， 應徵的職位是行政助理。|行政助理|
    |我想要應徵銷售部門的管理職位。|管理、銷售部門|
    |這是我的新會計職位履歷。|會計|
    |內附我的吧檯助理應徵文件。|吧檯助理|
    |我要提交屋頂工和創作者的應徵文件。|屋頂工、創作者|
    |這是我的的履歷， 我想應徵公車司機。|公車司機|
    |我是合格護士。 這是我的履歷。|合格護士|
    |我想要提交我在報上看到的教學職位文件。|教書|
    |這是我的履歷， 我想應徵蔬果裝料工。|裝料工|
    |應徵貼瓷磚工作。|圖格|
    |已隨附景觀設計師履歷。|景觀設計師|
    |已隨附我的生物學教授履歷。|生物學教授|
    |我想要應徵攝影方面的職位。|攝影|git 

## <a name="label-entity-in-example-utterances"></a>標記範例語句中的實體

藉由標記或「標示」，實體可顯示在範例語句中找到實體的 LUIS。

1. 選取左功能表中的 [意圖]。

2. 從意圖清單中選取 [GetJobInformation]。 

3. 將語句範例中的職位加上標籤：

    |語句|職位實體|
    |:--|:--|
    |資料庫中是否有任何工作？|資料庫|
    |尋找會計職責方面的新局面|會計|
    |有哪些適合資深工程師的職位？|資深工程師|

    尚有其他語句範例，但這些語句未包含職位字組。

## <a name="train"></a>定型

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="get-intent-and-entities-from-endpoint"></a>從端點取得意圖和實體 

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 結尾並輸入 `Here is my c.v. for the programmer job`。 最後一個 querystring 參數是 `q`，也就是 **query** 語句。 此語句與任何標示的語句都不同，因此這是很好的測試，且應該傳回 `ApplyForJob` 語句。

    ```JSON
    {
      "query": "Here is my c.v. for the programmer job",
      "topScoringIntent": {
        "intent": "ApplyForJob",
        "score": 0.9826467
      },
      "intents": [
        {
          "intent": "ApplyForJob",
          "score": 0.9826467
        },
        {
          "intent": "GetJobInformation",
          "score": 0.0218927357
        },
        {
          "intent": "MoveEmployee",
          "score": 0.007849265
        },
        {
          "intent": "Utilities.StartOver",
          "score": 0.00349470088
        },
        {
          "intent": "Utilities.Confirm",
          "score": 0.00348804821
        },
        {
          "intent": "None",
          "score": 0.00319909188
        },
        {
          "intent": "FindForm",
          "score": 0.00222647213
        },
        {
          "intent": "Utilities.Help",
          "score": 0.00211193133
        },
        {
          "intent": "Utilities.Stop",
          "score": 0.00172086991
        },
        {
          "intent": "Utilities.Cancel",
          "score": 0.00138010911
        }
      ],
      "entities": [
        {
          "entity": "programmer",
          "type": "Job",
          "startIndex": 24,
          "endIndex": 33,
          "score": 0.5230502
        }
      ]
    }
    ```
    
    LUIS 找到了正確的意圖 **ApplyForJob**，並擷取正確的實體 **Job**，其值為 `programmer`。


## <a name="names-are-tricky"></a>名稱很棘手
LUIS 應用程式深信它找到了正確的意圖，並擷取出職位名稱，但這些名稱很棘手。 請試試語句 `This is the lead welder paperwork`。  

在下列 JSON 中，LUIS 會回應正確意圖 `ApplyForJob`，但未擷取 `lead welder` 職位名稱。 

```JSON
{
  "query": "This is the lead welder paperwork.",
  "topScoringIntent": {
    "intent": "ApplyForJob",
    "score": 0.468558252
  },
  "intents": [
    {
      "intent": "ApplyForJob",
      "score": 0.468558252
    },
    {
      "intent": "GetJobInformation",
      "score": 0.0102701457
    },
    {
      "intent": "MoveEmployee",
      "score": 0.009442534
    },
    {
      "intent": "Utilities.StartOver",
      "score": 0.00639619166
    },
    {
      "intent": "None",
      "score": 0.005859333
    },
    {
      "intent": "Utilities.Cancel",
      "score": 0.005087704
    },
    {
      "intent": "Utilities.Stop",
      "score": 0.00315379258
    },
    {
      "intent": "Utilities.Help",
      "score": 0.00259344373
    },
    {
      "intent": "FindForm",
      "score": 0.00193389168
    },
    {
      "intent": "Utilities.Confirm",
      "score": 0.000420796918
    }
  ],
  "entities": []
}
```

由於名稱可以是任何字組，LUIS 若有可提升訊號的字組片語清單，就能更正確地預測實體。

## <a name="to-boost-signal-add-phrase-list"></a>若要提升訊號，請新增片語清單

從 LUIS-Samples Github 存放庫開啟 [jobs-phrase-list.csv](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/quickstarts/job-phrase-list.csv)。 此清單有超過一千個職位字組和片語。 請看一下清單以找出對您有意義的職位字組清單。 如果清單中沒有您的字組或片語，請自行新增。

1. 在 LUIS 應用程式的 [建置] 區段中，選取 [提升應用程式效能] 功能表底下的 [片語清單]。

2. 選取 [建立新的片語清單]。 

3. 將新的片語清單命名為 `Job`，然後將 jobs-phrase-list.csv 中的清單複製到 [值] 文字方塊。 選取 Enter 鍵。 

    [![](media/luis-quickstart-primary-and-secondary-data/hr-create-phrase-list-1.png "[建立新的片語清單] 對話方塊快顯的螢幕擷取畫面")](media/luis-quickstart-primary-and-secondary-data/hr-create-phrase-list-1.png#lightbox)

    如果您想在片語清單中新增更多字組，請檢閱**相關值**，並新增相關字組。 

4. 選取 [儲存] 以啟動該片語清單。

    [![](media/luis-quickstart-primary-and-secondary-data/hr-create-phrase-list-2.png "[建立新的片語清單] 對話方塊快顯與片語清單值方塊中字組的螢幕擷取畫面")](media/luis-quickstart-primary-and-secondary-data/hr-create-phrase-list-2.png#lightbox)

5. 重新[訓練](#train-the-luis-app)並[發行](#publish-the-app-to-get-the-endpoint-URL)應用程式以使用片語清單。

6. 在端點使用相同語句重新查詢：`This is the lead welder paperwork.`

    JSON 回應包含所擷取的實體：

    ```JSON
    {
        "query": "This is the lead welder paperwork.",
        "topScoringIntent": {
            "intent": "ApplyForJob",
            "score": 0.920025647
        },
        "intents": [
            {
            "intent": "ApplyForJob",
            "score": 0.920025647
            },
            {
            "intent": "GetJobInformation",
            "score": 0.003800706
            },
            {
            "intent": "Utilities.StartOver",
            "score": 0.00299335527
            },
            {
            "intent": "MoveEmployee",
            "score": 0.0027167045
            },
            {
            "intent": "None",
            "score": 0.00259556063
            },
            {
            "intent": "FindForm",
            "score": 0.00224019377
            },
            {
            "intent": "Utilities.Stop",
            "score": 0.00200693542
            },
            {
            "intent": "Utilities.Cancel",
            "score": 0.00195913855
            },
            {
            "intent": "Utilities.Help",
            "score": 0.00162656687
            },
            {
            "intent": "Utilities.Confirm",
            "score": 0.0002851904
            }
        ],
        "entities": [
            {
            "entity": "lead welder",
            "type": "Job",
            "startIndex": 12,
            "endIndex": 22,
            "score": 0.8295959
            }
        ]
    }
    ```

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，人力資源應用程式會使用機器學習的簡單實體來尋找語句中的職位名稱。 職位名稱可以是這麼多樣化的字組或片語，因此應用程式需要片語清單才能提升職位名稱字組。 

> [!div class="nextstepaction"]
> [新增預先建置的 keyPhrase 實體](luis-quickstart-intent-and-key-phrase.md)