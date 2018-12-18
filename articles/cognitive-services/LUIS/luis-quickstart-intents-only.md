---
title: 教學課程 1：在自訂 LUIS 應用程式中尋找用意
titleSuffix: Azure Cognitive Services
description: 建立自訂應用程式來預測使用者的用意。 此應用程式不會擷取語句文字中的各種資料元素 (例如電子郵件地址或日期)，因此是最簡單的 LUIS 應用程式類型。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/09/2018
ms.author: diberry
ms.openlocfilehash: b229dbc90f3f6ecc226c88ee393114f233bcf1a2
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035398"
---
# <a name="tutorial-1-build-custom-app-to-determine-user-intentions"></a>教學課程 1：建置自訂應用程式來判斷使用者的用意

在本教學課程中，您會建立自訂人力資源 (HR) 應用程式，以根據語句 (文字) 預測使用者的用意。 完成之後，您會擁有一個在雲端中執行的 LUIS 端點。

應用程式的目的是要判斷交談式自然語言文字的用意。 這些用意可歸類為各種**意圖**。 此應用程式有幾個意圖。 第一個意圖 **`GetJobInformation`** 可識別出使用者想要知道公司內部職缺的相關資訊。 第二個的意圖 **`None`** 使用於來自此應用程式「網域」 (範圍) 以外使用者的任何語句。 而後，針對任何有關應徵職位的語句新增了第三個意圖 **`ApplyForJob`**。 此第三個意圖不同於 `GetJobInformation`，因為某人在應徵職位時應該已經知道職位資訊。 不過，根據字組選擇來判斷意圖可能有點困難，因為兩者都與職位有關。

在 LUIS 傳回 JSON 回應之後，隨著此要求完成 LUIS。 LUIS 不會提供使用者語句的回答，只會在自然語言中識別所要求的是哪一類資訊。 

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 建立新的應用程式 
> * 建議意圖
> * 新增範例語句
> * 訓練應用程式
> * 發佈應用程式
> * 從端點取得意圖

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="create-a-new-app"></a>建立新的應用程式

1. 使用 [https://www.luis.ai](https://www.luis.ai) 的 URL 登入 LUIS 入口網站。 

2. 選取 [建立新的應用程式]。  

    [![](media/luis-quickstart-intents-only/app-list.png "Language Understanding (LUIS) 我的應用程式頁面的螢幕擷取畫面")](media/luis-quickstart-intents-only/app-list.png#lightbox)

3. 在快顯對話方塊中，輸入名稱 `HumanResources`，並保留預設文化特性 [英文]。 讓描述保留空白。

    ![LUIS 新增應用程式](./media/luis-quickstart-intents-only/create-app.png)

    接著，應用程式會顯示 [None] 意圖的 [意圖] 頁面。

## <a name="getjobinformation-intent"></a>GetJobInformation 意圖

1. 選取 [Create new intent] \(建立新意圖\)。 輸入新的意圖名稱 `GetJobInformation`。 每當使用者想要知道公司內職缺的相關資訊時，系統就會預測此意圖。

    ![](media/luis-quickstart-intents-only/create-intent.png "Language Understanding (LUIS) 新增意圖對話方塊的螢幕擷取畫面")

2. 藉由提供_範例語句_，您可訓練 LUIS 應針對此意圖預測哪些種類的語句。 將您預期使用者會要求的數個範例語句新增至此意圖，例如：

    | 範例語句|
    |--|
    |今天是否有發佈任何新職位？|
    |有哪些適合資深工程師的職位？|
    |是否有資料庫方面的工作？|
    |尋找會計職責方面的新局面|
    |職位清單在哪|
    |有新職位嗎？|
    |西雅圖辦公室是否有任何新職位？|

    [![](media/luis-quickstart-intents-only/utterance-getstoreinfo.png "針對 MyStore 意圖輸入新語句的螢幕擷取畫面")](media/luis-quickstart-intents-only/utterance-getstoreinfo.png#lightbox)

    [!include[Do not use too few utterances](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]    


## <a name="none-intent"></a>None 意圖 
用戶端應用程式必須知道語句是否在應用程式的主體網域外部。 如果 LUIS 針對語句傳回 [None] 意圖時，您的用戶端應用程式可以詢問使用者是否想要結束交談。 如果使用者不想要結束交談，用戶端應用程式也可以提供更多指示，以便繼續交談。 

這些範例語句 (在主體網域外部) 會群組成 [None] 意圖。 請勿將它保留空白。 

1. 選取左面板中的 [意圖]。

2. 選取 [無] 意圖。 新增您的使用者可能輸入但與您的人力資源應用程式無關的三個語句。 如果應用程式與職位公告有關，以下是一些 **None** 語句：

    | 範例語句|
    |--|
    |狗一直叫很煩人|
    |幫我訂披薩|
    |海裡面的企鵝|


## <a name="train"></a>定型 

[!INCLUDE [LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish"></a>發佈

[!INCLUDE [LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)] 

## <a name="get-intent"></a>取得意圖

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 移至位址中的 URL 尾端並輸入 `I'm looking for a job with Natural Language Processing`。 最後一個查詢字串參數是 `q`，也就是語句**查詢**。 此語句與所有範例語句不同。 這是很好的測試，而且應傳回 `GetJobInformation` 意圖作為評分最高的意圖。 

    ```JSON
    {
      "query": "I'm looking for a job with Natural Language Processing",
      "topScoringIntent": {
        "intent": "GetJobInformation",
        "score": 0.8965092
      },
      "intents": [
        {
          "intent": "GetJobInformation",
          "score": 0.8965092
        },
        {
          "intent": "None",
          "score": 0.147104025
        }
      ],
      "entities": []
    }
    ```

    結果會包含應用程式中的**所有意圖** (目前為 2 個)。 實體陣列是空的，因為此應用程式目前沒有任何實體。 

    JSON 結果會將評分最高的意圖識別為 **`topScoringIntent`** 屬性。 所有分數都介於 1 到 0，分數越好者越接近 1。 

## <a name="applyforjob-intent"></a>ApplyForJob 意圖
返回 LUIS 網站並建立新的意圖，以判斷使用者語句是否關於應徵職位。

1. 在右上方的功能表中選取 [建置]，以返回應用程式建置。

2. 選取左功能表中的 [意圖]。

3. 選取 **建立新意圖**，然後輸入名稱 `ApplyForJob`。 

    ![用來建立新意圖的 LUIS 對話方塊](./media/luis-quickstart-intents-only/create-applyforjob-intent.png)

4. 將您預期使用者會要求的數個語句新增至意圖，例如：

    | 範例語句|
    |--|
    |我想要應徵新的會計職位|
    |填寫職位 123456 的應徵文件|
    |投遞履歷以應徵工程師職缺|
    |這是我的履歷， 應徵的職位是 654234|
    |職位 567890 和我的文件|

    [![](media/luis-quickstart-intents-only/utterance-applyforjob.png "針對 ApplyForJob 意圖輸入新語句的螢幕擷取畫面")](media/luis-quickstart-intents-only/utterance-applyforjob.png#lightbox)

    LUIS 目前不確定意圖是否正確，所以標示的意圖會有紅色外框。 請訓練應用程式，讓 LUIS 知道語句位於正確意圖。 

## <a name="train-again"></a>再次訓練

[!include[LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="publish-again"></a>再次發佈

[!include[LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)] 

## <a name="get-intent-again"></a>再次取得意圖

1. [!INCLUDE [LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)]

2. 在新的瀏覽器視窗中，於 URL 結尾輸入 `Can I submit my resume for job 235986`。 

    ```JSON
    {
      "query": "Can I submit my resume for job 235986",
      "topScoringIntent": {
        "intent": "ApplyForJob",
        "score": 0.9166808
      },
      "intents": [
        {
          "intent": "ApplyForJob",
          "score": 0.9166808
        },
        {
          "intent": "GetJobInformation",
          "score": 0.07162977
        },
        {
          "intent": "None",
          "score": 0.0262826588
        }
      ],
      "entities": []
    }
    ```

    結果會包含新的意圖 **ApplyForJob** 以及現有意圖。 

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

本教學課程已在端點上建立人力資源 (HR) 應用程式、建立 2 個意圖、新增每個意圖的範例語句、進行訓練、發佈及測試。 這些是建置 LUIS 模型的基本步驟。 

> [!div class="nextstepaction"]
> [在此應用程式中新增預先建置的意圖和實體](luis-tutorial-prebuilt-intents-entities.md)
