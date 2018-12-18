---
title: 10 分鐘建立您的第一個 LUIS 應用程式
titleSuffix: Azure Cognitive Services
description: 建立 LUIS 應用程式，該應用程式會使用預先建置網域 `HomeAutomation` 來開啟或關閉燈光和應用程式。 這個預先建置網域可為您提供意圖、實體和範例語句。 完成之後，您會擁有一個在雲端中執行的 LUIS 端點。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 12a660b49d1a81865c34ceda38f041de9be31eb1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037468"
---
# <a name="quickstart-use-prebuilt-home-automation-app"></a>快速入門：使用預先建置的家庭自動化應用程式

在本快速入門中，建立 LUIS 應用程式，該應用程式會使用預先建置網域 `HomeAutomation` 來開啟或關閉燈光和應用程式。 這個預先建置網域可為您提供意圖、實體和範例語句。 完成之後，您會擁有一個在雲端中執行的 LUIS 端點。

## <a name="prerequisites"></a>必要條件

在本文中，您需要免費 LUIS 帳戶，在位於 [http://www.luis.ai](http://www.luis.ai) (英文) 的 LUIS 入口網站上建立。 

## <a name="create-a-new-app"></a>建立新的應用程式
您可以在 [我的應用程式] 上建立和管理應用程式。 

1. 登入 LUIS 入口網站。

2. 選取 [建立新的應用程式]。

    [![](media/luis-quickstart-new-app/app-list.png "應用程式清單的螢幕擷取畫面")](media/luis-quickstart-new-app/app-list.png)

3. 在對話方塊中，將您的應用程式命名為 "Home Automation"。

    [![](media/luis-quickstart-new-app/create-new-app-dialog.png "建立新應用程式快顯對話方塊的螢幕擷取畫面")](media/luis-quickstart-new-app/create-new-app-dialog.png)

4. 選擇您的應用程式文化特性。 請針對這個「家庭自動化」應用程式，選擇 [英文]。 然後選取 [完成]。 LUIS 會建立「家庭自動化」應用程式。 

    >[!NOTE]
    >建立應用程式之後便無法變更文化特性 (Culture)。 

## <a name="add-prebuilt-domain"></a>新增預建網域

在左側導覽窗格中選取 [預先建置網域]。 然後搜尋 "Home"。 選取 [新增網域]。

[![](media/luis-quickstart-new-app/home-automation.png "螢幕擷取畫面，在預先建置網域功能表中叫出的家庭自動化網域")](media/luis-quickstart-new-app/home-automation.png)

當成功新增網域時，預先建置網域方塊會顯示 [移除網域] 按鈕。

[![](media/luis-quickstart-new-app/remove-domain.png "螢幕擷取畫面，具有移除按鈕的家庭自動化網域")](media/luis-quickstart-new-app/remove-domain.png)

## <a name="intents-and-entities"></a>意圖和實體

在左側瀏覽窗格中選取 [意圖]，以檢閱 HomeAutomation 網域意圖。 每個意圖都有範例語句。

> [!NOTE]
> 「無」是所有 LUIS 應用程式都會提供的意圖。 您可以使用它來處理未對應至應用程式所提供功能的語句。 

選取 **HomeAutomation.TurnOff** 意圖。 您可以看到意圖包含以實體標示的語句清單。

[![](media/luis-quickstart-new-app/home-automation-turnon.png "HomeAutomation.TurnOff 意圖的螢幕擷取畫面")](media/luis-quickstart-new-app/home-automation-turnon.png)

## <a name="train-the-luis-app"></a>進行 LUIS 應用程式定型

[!include[LUIS How to Train steps](../../../includes/cognitive-services-luis-tutorial-how-to-train.md)]

## <a name="test-your-app"></a>測試應用程式
一旦您定型您的應用程式，就可以進行測試。 選取頂端導覽中的 [測試]。 將測試語句 (例如「關閉燈光」) 輸入到 [互動測試] 窗格中，然後按下 Enter 鍵。 

```
Turn off the lights
```

檢查最高得分意圖是否對應至您對於每個測試語句所預期的意圖。

在此範例中，「關閉燈光」會正確識別為 "HomeAutomation.TurnOff" 的最高得分意圖。

[![](media/luis-quickstart-new-app/test.png "螢幕擷取畫面，具有醒目提示語句的測試面板")](media/luis-quickstart-new-app/test.png)


再次選取 [測試] 以摺疊測試窗格。 

<a name="publish-your-app"></a>

## <a name="publish-the-app-to-get-the-endpoint-url"></a>發佈應用程式以取得端點 URL

[!include[LUIS How to Publish steps](../../../includes/cognitive-services-luis-tutorial-how-to-publish.md)]

## <a name="query-the-endpoint-with-a-different-utterance"></a>使用不同的語句來查詢端點

1. [!include[LUIS How to get endpoint first step](../../../includes/cognitive-services-luis-tutorial-how-to-get-endpoint.md)] 

2. 移至位址中的 URL 結尾並輸入 `turn off the living room light`，然後按 Enter。 瀏覽器會顯示 HTTP 端點的 JSON 回應。

    [![](media/luis-quickstart-new-app/turn-off-living-room.png "螢幕擷取畫面，瀏覽器中的 JSON 結果偵測到意圖 TurnOff")](media/luis-quickstart-new-app/turn-off-living-room.png)
    
## <a name="clean-up-resources"></a>清除資源

[!include[LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

您可以從程式碼呼叫端點：

> [!div class="nextstepaction"]
> [使用程式碼呼叫 LUIS 端點](luis-get-started-cs-get-intent.md)
