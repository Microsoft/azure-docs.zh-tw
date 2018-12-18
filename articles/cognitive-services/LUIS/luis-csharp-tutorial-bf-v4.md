---
title: 使用 C# 的 LUIS Bot - 教學課程 - Web 應用程式 Bot - Bot Framework SDK 4.0
titleSuffix: Azure Cognitive Services
description: 使用 C#，建置與 Language Understanding (LUIS) 整合的聊天機器人。 此聊天機器人會使用人力資源應用程式來快速實作聊天機器人解決方案。 此 Bot 是使用 Bot Framework 第 4 版和 Azure Web 應用程式 Bot 所建置的。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/25/2018
ms.author: diberry
ms.openlocfilehash: 3ccec4fbd0fd69539e29e2f15f71115444bf0a48
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389287"
---
# <a name="tutorial-luis-bot-in-c"></a>教學課程：C# 中的 LUIS Bot
使用 C#，您可以建置與 Language Understanding (LUIS) 整合的聊天機器人。 此 Bot 會使用 HomeAutomation 應用程式來實作 Bot 解決方案。 此 Bot 是使用 [Web 應用程式 Bot](https://docs.microsoft.com/azure/bot-service/) 和 [Bot Framework 第 4 版](https://github.com/Microsoft/botbuilder-js)所建置的。

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 建立 Web 應用程式 Bot。 此程序會為您建立新的 LUIS 應用程式。
> * 將預先建置的領域加入新的 LUIS 模型中
> * 下載 Web Bot 服務所建立的專案
> * 在本機電腦上啟動 Bot 和模擬器
> * 修改新 LUIS 意圖的 Bot 程式碼
> * 在 Bot 中檢視語句結果

## <a name="prerequisites"></a>必要條件

* [Bot 模擬器](https://aka.ms/abs/build/emulatordownload)
* [Visual Studio](https://visualstudio.microsoft.com/downloads/)


## <a name="create-web-app-bot"></a>建立 Web 應用程式 Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [建立新資源]。

2. 在搜尋方塊中，搜尋並選取 [Web 應用程式 Bot]。 選取 [建立] 。

3. 在 [Bot 服務] 中提供必要資訊：

    |設定|目的|建議的設定|
    |--|--|--|
    |Bot 名稱|資源名稱|`luis-csharp-bot-` + `<your-name>`，例如 `luis-csharp-bot-johnsmith`|
    |訂用帳戶|要在其中建立 Bot 的訂用帳戶。|您的主要訂用帳戶。
    |資源群組|Azure 資源的邏輯群組|建立新的群組以存放與此 Bot 搭配使用的所有資源，請將群組命名為 `luis-csharp-bot-resource-group`。|
    |位置|Azure 區域 - 這不一定要與 LUIS 撰寫或發行區域相同。|`westus`|
    |定價層|用於服務要求限制和計費。|`F0` 是免費層。
    |應用程式名稱|將 Bot 部署到雲端時，此名稱會用來作為子領域 (例如 humanresourcesbot.azurewebsites.net)。|`luis-csharp-bot-` + `<your-name>`，例如 `luis-csharp-bot-johnsmith`|
    |Bot 範本|Bot Framework 設定 - 請參閱下一個資料表|
    |LUIS 應用程式位置|必須與 LUIS 資源區域相同|`westus`|

4. 在 [Bot 範本設定] 中選取下列項目，然後在這些設定下選擇 [選取] 按鈕：

    |設定|目的|選取項目|
    |--|--|--|
    |SDK 版本|Bot Framework 版本|**SDK v4**|
    |SDK 語言|Bot 的程式設計語言|**C#**|
    |Echo/基本 Bot|Bot 類型|**基本 Bot**|
    
5. 選取 [建立] 。 這會建立 Bot 服務，並將其部署到 Azure。 此程序中的一部份會為您建立名為 `luis-csharp-bot-XXXX` 的 LUIS 應用程式。 此名稱會以上一節中的 Bot 和應用程式名稱為基礎。

    [ ![建立 Web 應用程式 Bot](./media/bfv4-csharp/create-web-app-service.png) ](./media/bfv4-csharp/create-web-app-service.png#lightbox)

6. 讓此瀏覽器索引標籤保持開啟。 對於使用到 LUIS 入口網站的任何步驟，請開啟新的瀏覽器索引標籤。部署新的 Bot 服務後，請繼續下一節。

## <a name="add-prebuilt-domain-to-model"></a>將預建領域新增至模型
Bot 服務部署的一部分會以意圖和語句範例來建立新 LUIS 應用程式。 Bot 提供的意圖會對應到新 LUIS 應用程式的下列意圖： 

|基本 Bot 的 LUIS 意圖|範例語句|
|--|--|
|取消|`stop`|
|Greeting|`hello`|
|說明|`help`|
|None|應用程式領域以外的任何項目。|

將預先建置的 HomeAutomation 應用程式新增至模型，以處理類似下列的語句：`Turn off the living room lights`

1. 前往 [LUIS 入口網站](https://www.luis.ai)並登入。
2. 在 [我的應用程式] 頁面上，選取 [建立日期] 資料行，以根據應用程式的建立日期排序。 Azure Bot 服務已在上一節中建立新的應用程式。 其名稱是 `luis-csharp-bot-` + `<your-name>` + 4 個隨機字元所組成。
3. 開啟應用程式並選取頂端導覽列中的 [建置] 區段。
4. 從左側功能表中，選取 [預先建置領域]。
5. 藉由在其卡片上選取 [新增領域] 來選取 **HomeAutomation** 領域。
6. 選取右上方功能表中的 [訓練]。
7. 選取右上方功能表中的 [發佈]。 

    由 Azure Bot 服務建立的應用程式現在有新的意圖：

    |基本 Bot 的新意圖|範例語句|
    |--|--|
    |HomeAutomation.TurnOn|`turn the fan to high`
    |HomeAutomation.TurnOff|`turn off ac please`|

## <a name="download-the-web-app-bot"></a>下載 Web 應用程式 Bot 
若要開發 Web 應用程式 Bot 程式碼，請下載程式碼並在您的本機電腦上使用。 

1. 仍在 Azure 入口網站中的 Web 應用程式 Bot 資源上，選取 [應用程式設定]，並複製 **botFilePath** 和 **botFileSecret** 的值。 您稍後需要將這些值新增至環境檔案。 

2. 在 Azure 入口網站中，從 [Bot 管理] 區段中選取 [建置]。 

3. 選取 [下載 Bot 原始程式碼]。 

    [ ![下載基本 Bot 的 Web 應用程式 Bot 原始程式碼](../../../includes/media/cognitive-services-luis/bfv4/download-code.png) ](../../../includes/media/cognitive-services-luis/bfv4/download-code.png#lightbox)

4. 如果原始程式碼已壓縮為 zip 檔，將會有提供程式碼下載連結的訊息。 選取連結。 

5. 將 zip 檔儲存到本機電腦並解壓縮。 開啟專案。 

6. 開啟 bot.js 檔案並尋找 `_services.LuisServices`。 此階段會將進入 Bot 的使用者語句傳送到 LUIS。

    ```csharp
    /// <summary>
    /// Run every turn of the conversation. Handles orchestration of messages.
    /// </summary>
    /// <param name="turnContext">Bot Turn Context.</param>
    /// <param name="cancellationToken">Task CancellationToken.</param>
    /// <returns>A <see cref="Task"/> representing the asynchronous operation.</returns>
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        var activity = turnContext.Activity;

        if (activity.Type == ActivityTypes.Message)
        {
            // Perform a call to LUIS to retrieve results for the current activity message.
            var luisResults = await _services.LuisServices[LuisConfiguration].RecognizeAsync(turnContext, cancellationToken).ConfigureAwait(false);

            // If any entities were updated, treat as interruption.
            // For example, "no my name is tony" will manifest as an update of the name to be "tony".
            var topScoringIntent = luisResults?.GetTopScoringIntent();

            var topIntent = topScoringIntent.Value.intent;
            switch (topIntent)
            {
                case GreetingIntent:
                    await turnContext.SendActivityAsync("Hello.");
                    break;
                case HelpIntent:
                    await turnContext.SendActivityAsync("Let me try to provide some help.");
                    await turnContext.SendActivityAsync("I understand greetings, being asked for help, or being asked to cancel what I am doing.");
                    break;
                case CancelIntent:
                    await turnContext.SendActivityAsync("I have nothing to cancel.");
                    break;
                case NoneIntent:
                default:
                    // Help or no intent identified, either way, let's provide some help.
                    // to the user
                    await turnContext.SendActivityAsync("I didn't understand what you just said to me.");
                    break;
            }
        }
        else if (activity.Type == ActivityTypes.ConversationUpdate)
        {
            if (activity.MembersAdded.Any())
            {
                // Iterate over all new members added to the conversation.
                foreach (var member in activity.MembersAdded)
                {
                    // Greet anyone that was not the target (recipient) of this message.
                    // To learn more about Adaptive Cards, see https://aka.ms/msbot-adaptivecards for more details.
                    if (member.Id != activity.Recipient.Id)
                    {
                        var welcomeCard = CreateAdaptiveCardAttachment();
                        var response = CreateResponse(activity, welcomeCard);
                        await turnContext.SendActivityAsync(response).ConfigureAwait(false);
                    }
                }
            }
        }

    }
    ```

    Bot 將使用者的語句傳送至 LUIS，並取得結果。 最上層的意圖會決定對話流程。 


## <a name="start-the-bot"></a>啟動 Bot
變更任何程式碼或設定之前，請先確認 Bot 能正常運作。 

1. 在 Visual Studio 中開啟解決方案檔案。 

2. 建立 `appsettings.json` 檔案來保存 Bot 程式碼尋找的 Bot 變數：

    ```JSON
    {
    "botFileSecret": "",
    "botFilePath": ""

    }
    ```

    您在**[下載 Web 應用程式 Bot](#download-the-web-app-bot)** 一節的步驟 1 中，已從 Azure Bot 服務的 [應用程式設定] 中複製值，請將變數的值設為您複製的值。

3. 在 Visual Studio 中，啟動 Bot。 瀏覽器視窗會開啟 Web 應用程式 Bot 的網站，網址為 `http://localhost:3978/`。

## <a name="start-the-emulator"></a>啟動模擬器

1. 起始 Bot 模擬器。

2. 在 Bot 模擬器中，選取專案根目錄中的 *.bot 檔案。 此 `.bot` 檔案包含訊息的 Bot URL 端點：

    [ ![Bot 模擬器 v4](../../../includes/media/cognitive-services-luis/bfv4/bot-emulator-v4.png) ](../../../includes/media/cognitive-services-luis/bfv4/bot-emulator-v4.png#lightbox)

3. 輸入您在**[下載 Web 應用程式 Bot](#download-the-web-app-bot)** 一節的步驟 1 中，從 Azure Bot 服務 [應用程式設定] 中複製的 Bot 祕密。 這可讓模擬器存取 `.bot` 檔案中的任何加密欄位。

    ![Bot 模擬器祕密 v4](../../../includes/media/cognitive-services-luis/bfv4/bot-secret.png)

4. 在 Bot 模擬器中輸入 `Hello`，並取得基本 Bot 的適當回應。

    [ ![模擬器中的基本 Bot 回應](../../../includes/media/cognitive-services-luis/bfv4/emulator-test.png) ](../../../includes/media/cognitive-services-luis/bfv4/emulator-test.png#lightbox)

## <a name="modify-bot-code"></a>修改 Bot 程式碼 

在 `BasicBot.cs` 檔案中，新增程式碼來處理新的意圖。 

1. 在檔案頂端，尋找**支援的 LUIS 意圖**區段，並為 HomeAutomation 意圖新增常數：

    ```csharp
    // Supported LUIS Intents
    public const string GreetingIntent = "Greeting";
    public const string CancelIntent = "Cancel";
    public const string HelpIntent = "Help";
    public const string NoneIntent = "None";
    public const string TurnOnIntent = "HomeAutomation_TurnOn"; // new intent
    public const string TurnOffIntent = "HomeAutomation_TurnOff"; // new intent
    ```

    請注意，在 LUIS 入口網站應用程式中的領域和意圖之間會以底線 (`_`) 取代句點 (`.`)。 

2. 尋找接收語句 LUIS 預測的 **OnTurnAsync** 方法。 在 switch 陳述式中新增程式碼，以傳回兩個 HomeAutomation 意圖的 LUIS 回應。 

    ```csharp
    case TurnOnIntent:
        await turnContext.SendActivityAsync("TurnOn intent found, JSON response: " + luisResults?.Entities.ToString());
        break;
    case TurnOffIntent:
        await turnContext.SendActivityAsync("TurnOff intent found, JSON response: " + luisResults?.Entities.ToString());
        break;
    ```

    Bot 沒有與 LUIS REST API 要求完全相同的回應，因此請務必查看 JSON 回應以了解差異。 文字和意圖屬性都相同，但是實體的屬性值已修改。 

    ```JSON
    {
        "$instance": {
            "HomeAutomation_Device": [
                {
                    "startIndex": 23,
                    "endIndex": 29,
                    "score": 0.9776345,
                    "text": "lights",
                    "type": "HomeAutomation.Device"
                }
            ],
            "HomeAutomation_Room": [
                {
                    "startIndex": 12,
                    "endIndex": 22,
                    "score": 0.9079433,
                    "text": "livingroom",
                    "type": "HomeAutomation.Room"
                }
            ]
        },
        "HomeAutomation_Device": [
            "lights"
        ],
        "HomeAutomation_Room": [
            "livingroom"
        ]
    }
    ```



## <a name="view-results-in-bot"></a>在 Bot 中檢視結果

1. 在 Bot 模擬器中，輸入語句：`Turn on the livingroom lights to 50%`

2. Bot 會回應：

    ```JSON
    TurnOn intent found, JSON response: {"$instance":{“HomeAutomation_Device”:[{“startIndex”:23,“endIndex”:29,“score”:0.9776345,“text”:“lights”,“type”:“HomeAutomation.Device”}],“HomeAutomation_Room”:[{“startIndex”:12,“endIndex”:22,“score”:0.9079433,“text”:“livingroom”,“type”:“HomeAutomation.Room”}]},“HomeAutomation_Device”:[“lights”],“HomeAutomation_Room”:[“livingroom”]}
    ```    

## <a name="learn-more-about-bot-framework"></a>深入了解 Bot Framework
Azure Bot 服務會使用 Bot Framework SDK。 深入了解 SDK 和 Bot Framework：

* [Azure Bot 服務](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0) v4 文件
* [Bot Builder 範例](https://github.com/Microsoft/botbuilder-samples)
* [Bot Builder SDK](https://docs.microsoft.com/javascript/api/botbuilder-core/?view=botbuilder-ts-latest)
* [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools):

## <a name="next-steps"></a>後續步驟

您已建立了 Azure Bot 服務、複製 Bot 祕密和 `.bot` 檔案路徑，以及下載了程式碼的 zip 檔案。 您將預先建置的 HomeAutomation 領域新增至建立為新 Azure Bot 服務一部份的 LUIS 應用程式，然後再次訓練並發佈應用程式。 您將程式碼專案解壓縮，並建立了環境檔案 (`.env`)，然後設定 Bot 祕密和 `.bot` 檔案路徑。 在 bot.js 檔案中，您新增了程式碼來處理兩個新的意圖。 然後您在 Bot 模擬器中測試 Bot，以查看 LUIS 如何回應其中一個新意圖的語句。 


> [!div class="nextstepaction"]
> [在 LUIS 中建置自訂領域](luis-quickstart-intents-only.md)
