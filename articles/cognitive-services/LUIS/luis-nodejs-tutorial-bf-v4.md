---
title: 使用 Node.js 的 LUIS Bot - 教學課程 - Web 應用程式 Bot - Bot Framework SDK 4.0
titleSuffix: Azure Cognitive Services
description: 使用 Node.js，建置與 Language Understanding (LUIS) 整合的聊天機器人。 此聊天機器人會使用人力資源應用程式來快速實作聊天機器人解決方案。 此 Bot 是使用 Bot Framework 第 4 版和 Azure Web 應用程式 Bot 所建置的。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: ad21754b3f55a0d14bb43a2898d5bd4b8b8150ae
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385901"
---
# <a name="tutorial-luis-bot-in-nodejs"></a>教學課程：Node.js 中的 LUIS Bot
使用 Node.js，您可以建置與 Language Understanding (LUIS) 整合的聊天機器人。 此 Bot 會使用 HomeAutomation 應用程式來實作 Bot 解決方案。 此 Bot 是使用 [Web 應用程式 Bot](https://docs.microsoft.com/azure/bot-service/) 和 [Bot Framework 第 4 版](https://github.com/Microsoft/botbuilder-js)所建置的。

**在本教學課程中，您將了解如何：**

> [!div class="checklist"]
> * 建立 Web 應用程式 Bot。 此程序會為您建立新的 LUIS 應用程式。
> * 將預先建置的領域加入新的 LUIS 模型中
> * 下載 Web Bot 服務所建立的專案
> * 在本機電腦上啟動 Bot 和模擬器
> * 修改新 LUIS 意圖的 Bot 程式碼
> * 在 Bot 中檢視語句結果

## <a name="prerequisites"></a>必要條件

<!--* Samples from 
https://github.com/Microsoft/BotBuilder-Samples/tree/v4/javascript_nodejs/12.nlp-with-luis-->
* [Bot 模擬器](https://aka.ms/abs/build/emulatordownload)
* [Visual Studio Code](https://code.visualstudio.com/Download)


## <a name="create-web-app-bot"></a>建立 Web 應用程式 Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [建立新資源]。

2. 在搜尋方塊中，搜尋並選取 [Web 應用程式 Bot]。 選取 [建立] 。

3. 在 [Bot 服務] 中提供必要資訊：

    |設定|目的|建議的設定|
    |--|--|--|
    |Bot 名稱|資源名稱|`luis-nodejs-bot-` + `<your-name>`，例如 `luis-nodejs-bot-johnsmith`|
    |訂用帳戶|要在其中建立 Bot 的訂用帳戶。|您的主要訂用帳戶。
    |資源群組|Azure 資源的邏輯群組|建立新的群組以存放與此 Bot 搭配使用的所有資源，請將群組命名為 `luis-nodejs-bot-resource-group`。|
    |位置|Azure 區域 - 這不一定要與 LUIS 撰寫或發行區域相同。|`westus`|
    |定價層|用於服務要求限制和計費。|`F0` 是免費層。
    |應用程式名稱|將 Bot 部署到雲端時，此名稱會用來作為子領域 (例如 humanresourcesbot.azurewebsites.net)。|`luis-nodejs-bot-` + `<your-name>`，例如 `luis-nodejs-bot-johnsmith`|
    |Bot 範本|Bot Framework 設定 - 請參閱下一個資料表|
    |LUIS 應用程式位置|必須與 LUIS 資源區域相同|`westus`|

4. 在 [Bot 範本設定] 中選取下列項目，然後在這些設定下選擇 [選取] 按鈕：

    |設定|目的|選取項目|
    |--|--|--|
    |SDK 版本|Bot Framework 版本|**SDK v4**|
    |SDK 語言|Bot 的程式設計語言|**Node.js**|
    |Echo/基本 Bot|Bot 類型|**基本 Bot**|
    
5. 選取 [建立] 。 這會建立 Bot 服務，並將其部署到 Azure。 此程序中的一部份會為您建立名為 `luis-nodejs-bot-XXXX` 的 LUIS 應用程式。 此名稱會以上一節中的 Bot 和應用程式名稱為基礎。

    [ ![建立 Web 應用程式 Bot](./media/bfv4-nodejs/create-web-app-service.png) ](./media/bfv4-nodejs/create-web-app-service.png#lightbox)

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
2. 在 [我的應用程式] 頁面上，選取 [建立日期] 資料行，以根據應用程式的建立日期排序。 Azure Bot 服務已在上一節中建立新的應用程式。 其名稱是 `luis-nodejs-bot-` + `<your-name>` + 4 個隨機字元所組成。
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

6. 開啟 bot.js 檔案並尋找 `const results = await this.luisRecognizer.recognize(context);`。 此階段會將進入 Bot 的使用者語句傳送到 LUIS。

    ```javascript
    /**
     * Driver code that does one of the following:
     * 1. Display a welcome card upon startup
     * 2. Use LUIS to recognize intents
     * 3. Start a greeting dialog
     * 4. Optionally handle Cancel or Help interruptions
     *
     * @param {Context} context turn context from the adapter
     */
    async onTurn(context) {
        // Create a dialog context
        const dc = await this.dialogs.createContext(context);

        if(context.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the current activity message.
            const results = await this.luisRecognizer.recognize(context);
            
            const topIntent = LuisRecognizer.topIntent(results);

            // handle conversation interrupts first
            const interrupted = await this.isTurnInterrupted(dc, results);
            if(interrupted) {
                return;
            }

            // Continue the current dialog
            const dialogResult = await dc.continue();

            switch(dialogResult.status) {
                case DialogTurnStatus.empty:
                    switch (topIntent) {
                        case GREETING_INTENT:
                            await dc.begin(GREETING_DIALOG);
                            break;

                        case NONE_INTENT:
                        default:
                            // help or no intent identified, either way, let's provide some help
                            // to the user
                            await dc.context.sendActivity(`I didn't understand what you just said to me. topIntent ${topIntent}`);
                            break;
                    }

                case DialogTurnStatus.waiting:
                    // The active dialog is waiting for a response from the user, so do nothing
                break;

                case DialogTurnStatus.complete:
                    await dc.end();
                    break;

                default:
                    await dc.cancelAll();
                    break;

            }

        } else if (context.activity.type === 'conversationUpdate' && context.activity.membersAdded[0].name === 'Bot') {
            // When activity type is "conversationUpdate" and the member joining the conversation is the bot
            // we will send our Welcome Adaptive Card.  This will only be sent once, when the Bot joins conversation
            // To learn more about Adaptive Cards, see https://aka.ms/msbot-adaptivecards for more details.
            const welcomeCard = CardFactory.adaptiveCard(WelcomeCard);
            await context.sendActivity({ attachments: [welcomeCard] });
        }
    }
    ```

    Bot 將使用者的語句傳送至 LUIS，並取得結果。 最上層的意圖會決定對話流程。 


## <a name="start-the-bot"></a>啟動 Bot
變更任何程式碼或設定之前，請先確認 Bot 能正常運作。 

1. 在 Visual Studio Code 中，開啟終端機視窗。 

2. 安裝此 Bot 的 npm 相依性。 

    ```bash
    npm install
    ```
3. 建立檔案來保存 Bot 程式碼尋找的環境變數。 開啟 `.env` 檔案。 新增下列環境變數：

    <!--there is no code language that represents an .env file correctly-->
    ```
    botFilePath=
    botFileSecret=
    ```

    您在**[下載 Web 應用程式 Bot](#download-the-web-app-bot)** 一節的步驟 1 中，已從 Azure Bot 服務的 [應用程式設定] 中複製值，請將環境變數的值設為您複製的值。

4. 在監看式模式中啟動 Bot。 在此啟動之後對程式碼所做的任何變更，都會使得應用程式自動重新啟動。

    ```bash
    npm run watch
    ```

5. Bot 啟動後，終端機視窗會顯示其中執行 Bot 的本機連接埠：

    ```
    > basic-bot@0.1.0 start C:\Users\pattiowens\repos\BFv4\luis-nodejs-bot-src
    > node ./index.js NODE_ENV=development

    restify listening to http://[::]:3978
    
    Get the Emulator: https://aka.ms/botframework-emulator
    
    To talk to your bot, open the luis-nodejs-bot-pattiowens.bot file in the Emulator
    ```

## <a name="start-the-emulator"></a>啟動模擬器

1. 起始 Bot 模擬器。 

2. 在 Bot 模擬器中，選取專案根目錄中的 *.bot 檔案。 此 `.bot` 檔案包含訊息的 Bot URL 端點：

    [ ![Bot 模擬器 v4](../../../includes/media/cognitive-services-luis/bfv4/bot-emulator-v4.png) ](../../../includes/media/cognitive-services-luis/bfv4/bot-emulator-v4.png#lightbox)

3. 輸入您在**[下載 Web 應用程式 Bot](#download-the-web-app-bot)** 一節的步驟 1 中，從 Azure Bot 服務 [應用程式設定] 中複製的 Bot 祕密。 這可讓模擬器存取 .bot 檔案中的任何加密欄位。

    ![Bot 模擬器祕密 v4](../../../includes/media/cognitive-services-luis/bfv4/bot-secret.png)


4. 在 Bot 模擬器中輸入 `Hello`，並取得基本 Bot 的適當回應。

    [ ![模擬器中的基本 Bot 回應](../../../includes/media/cognitive-services-luis/bfv4/emulator-test.png) ](../../../includes/media/cognitive-services-luis/bfv4/emulator-test.png#lightbox)

## <a name="modify-bot-code"></a>修改 Bot 程式碼 

在 `bot.js` 檔案中，新增程式碼來處理新的意圖。 

1. 在檔案頂端，尋找**支援的 LUIS 意圖**區段，並為 HomeAutomation 意圖新增常數：

    ```javascript
    // Supported LUIS Intents
    const GREETING_INTENT = 'Greeting';
    const CANCEL_INTENT = 'Cancel';
    const HELP_INTENT = 'Help';
    const NONE_INTENT = 'None';
    const TURNON_INTENT = 'HomeAutomation_TurnOn'; // new intent
    const TURNOFF_INTENT = 'HomeAutomation_TurnOff'; // new intent
    ```

    請注意，在 LUIS 入口網站應用程式中的領域和意圖之間會以底線 (`_`) 取代句點 (`.`)。 

2. 尋找 **isTurnInterrupted**，這會接收語句的 LUIS 預測，並新增一行以將結果列印到主控台。

    ```node
    /**
     * Look at the LUIS results and determine if we need to handle
     * an interruptions due to a Help or Cancel intent
     *
     * @param {DialogContext} dc - dialog context
     * @param {LuisResults} luisResults - LUIS recognizer results
     */
    async isTurnInterrupted(dc, luisResults) {
        console.log(JSON.stringify(luisResults));
    ...
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

3. 針對 `DialogTurnStatus.empty` 案例，將意圖新增至 onTurn 方法的 switch 陳述式：

    ```javascript
    switch (topIntent) {
        case GREETING_INTENT:
            await dc.begin(GREETING_DIALOG);
            break;

        // New HomeAutomation.TurnOn intent
        case TURNON_INTENT: 

            await dc.context.sendActivity(`TurnOn intent found, entities included: ${JSON.stringify(results.entities)}`);
            break;

        // New HomeAutomation.TurnOff intent
        case TURNOFF_INTENT: 

            await dc.context.sendActivity(`TurnOff intent found, entities included: ${JSON.stringify(results.entities)}`);
            break;

        case NONE_INTENT:
        default:
            // help or no intent identified, either way, let's provide some help
            // to the user
            await dc.context.sendActivity(`I didn't understand what you just said to me. topIntent ${topIntent}`);
            break;
    }
    ```

## <a name="view-results-in-bot"></a>在 Bot 中檢視結果

1. 在 Bot 模擬器中，輸入語句：`Turn on the livingroom lights to 50%`

2. Bot 會回應：

    ```JSON
    TurnOn intent found, entities included: {"$instance":{“HomeAutomation_Device”:[{“startIndex”:23,“endIndex”:29,“score”:0.9776345,“text”:“lights”,“type”:“HomeAutomation.Device”}],“HomeAutomation_Room”:[{“startIndex”:12,“endIndex”:22,“score”:0.9079433,“text”:“livingroom”,“type”:“HomeAutomation.Room”}]},“HomeAutomation_Device”:[“lights”],“HomeAutomation_Room”:[“livingroom”]}
    ```

## <a name="learn-more-about-bot-framework"></a>深入了解 Bot Framework
Azure Bot 服務會使用 Bot Framework SDK。 深入了解 SDK 和 Bot Framework：

* [Azure Bot 服務](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0) v4 文件
* [Bot Builder 範例](https://github.com/Microsoft/botbuilder-samples)
* [Bot Builder SDK](https://docs.microsoft.com/javascript/api/botbuilder-core/?view=botbuilder-ts-latest)
* [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools):

## <a name="next-steps"></a>後續步驟

您已建立了 Azure Bot 服務、複製 Bot 祕密和 .bot 檔案路徑，以及下載了程式碼的 zip 檔案。 您將預先建置的 HomeAutomation 領域新增至建立為新 Azure Bot 服務一部份的 LUIS 應用程式，然後再次訓練並發佈應用程式。 您將程式碼專案解壓縮，並建立了環境檔案 (`.env`)，然後設定 Bot 祕密和 .bot 檔案路徑。 在 bot.js 檔案中，您新增了程式碼來處理兩個新的意圖。 然後您在 Bot 模擬器中測試 Bot，以查看 LUIS 如何回應其中一個新意圖的語句。 


> [!div class="nextstepaction"]
> [在 LUIS 中建置自訂領域](luis-quickstart-intents-only.md)
