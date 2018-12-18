---
title: 使用 Node.js 的 LUIS Bot - Web 應用程式 Bot - Bot Framework SDK 3.0
titleSuffix: Azure Cognitive Services
description: 使用 Bot Framework 來建置與 LUIS 應用程式整合的 Bot。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 0bd191da3f2625bc202ee66100e7dac25d9d65de
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042324"
---
# <a name="luis-bot-in-nodejs"></a>Node.js 中的 LUIS Bot

使用 Node.js，建置與 Language Understanding (LUIS) 整合的聊天機器人。 此聊天機器人會使用預先建置的 HomeAutomation 領域來快速實作聊天機器人解決方案。 此 Bot 是使用 Bot Framework 3.x 和 Azure Web 應用程式 Bot 所建置的。

## <a name="prerequisite"></a>必要條件

建立 Bot 之前，請依照[建立應用程式](./luis-get-started-create-app.md)中的步驟來建置其使用的 LUIS 應用程式。

Bot 會回應來自 LUIS 應用程式中 HomeAutomation 定義域的意圖。 針對這些意圖中的每個意圖，LUIS 應用程式都有提供一個與其對應的意圖。 Bot 則提供對話方塊來處理 LUIS 所偵測到的意圖。

| 意圖 | 範例語句 | Bot 功能 |
|:----:|:----------:|---|
| HomeAutomation.TurnOn | 開燈。 | 當偵測到 `HomeAutomation.TurnOn` 意圖時，Bot 會叫用 `TurnOnDialog`。 此對話方塊可供您叫用 IoT 服務來開啟裝置，並告訴使用者該裝置已開啟。 |
| HomeAutomation.TurnOff | 關閉臥室電燈。 | 當偵測到 `HomeAutomation.TurnOff` 意圖時，Bot 會叫用 `TurnOffDialog`。 此對話方塊可供您叫用 IoT 服務來關閉裝置，並告訴使用者該裝置已關閉。 |


## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用 Bot 服務來建立 Language Understanding Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取功能表刀鋒視窗中的 [建立新資源]，然後選取 [查看全部]。

    ![建立新資源](./media/luis-tutorial-node-bot/bot-service-creation.png)

2. 在搜尋方塊中，搜尋 **Web 應用程式 Bot**。 

    ![建立新資源](./media/luis-tutorial-node-bot/bot-service-selection.png)

3. 在 [Bot 服務] 刀鋒視窗中提供必要資訊，然後選取 [建立]。 這會建立 Bot 服務和 LUIS 應用程式，並將其部署到 Azure。 如果您想要使用[語音促發](https://docs.microsoft.com/bot-framework/bot-service-manage-speech-priming)，請先檢閱[區域需求](luis-resources-faq.md#what-luis-regions-support-bot-framework-speech-priming)，再建立您的 Bot。 
    * 將 [應用程式名稱] 設定為您 Bot 的名稱。 將 Bot 部署到雲端時，此名稱會用來作為子網域 (例如 mynotesbot.azurewebsites.net)。 <!-- This name is also used as the name of the LUIS app associated with your bot. Copy it to use later, to find the LUIS app associated with the bot. -->
    * 選取訂用帳戶、[資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)、App Service 方案，以及[位置](https://azure.microsoft.com/regions/)。
    * 針對 [Bot 範本]，請選取：
        * **SDK v3**
        * **Node.js**
        * **語言理解**
    * 選取 [LUIS 應用程式位置]。 這是用來建立應用程式的撰寫[區域][LUIS]。
    * 選取法律聲明的確認核取方塊。 法律聲明的條款在此核取方塊下方。

    ![Bot 服務刀鋒視窗](./media/luis-tutorial-node-bot/bot-service-setting-callout-template.png)


4. 確認已部署 Bot 服務。
    * 選取 [通知] (位於 Azure 入口網站頂端邊緣的鈴鐺圖示)。 通知會從 [部署已開始] 變更為 [部署成功]。
    * 在通知變更為 [部署成功] 之後，選取該通知上的 [前往資源]。

## <a name="try-the-default-bot"></a>試用預設 Bot

勾選 [通知] 來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 選取 [前往資源] 按鈕以開啟 Bot 的資源刀鋒視窗。

<!-- this step isn't supposed to be necessary -->
## <a name="install-npm-resources"></a>安裝 NPM 資源
使用下列步驟來安裝 NPM 套件：

1. 從「Web 應用程式 Bot」的 [Bot 管理] 區段中選取 [建置]。 

2. 一個新的第二瀏覽器視窗隨即開啟。 選取 [開啟線上程式碼編輯器]。

3. 在頂端導覽列中，選取 Web 應用程式 Bot 名稱 `homeautomationluisbot`。 

4. 在下拉式清單中，選取 [開啟 Kudu 主控台]。

5. 一個新的瀏覽器視窗隨即開啟。 在主控台中，輸入下列命令：

    ```
    cd site\wwwroot && npm install
    ```

    等候安裝程序完成。 返回第一個瀏覽器視窗。 

## <a name="test-in-web-chat"></a>在網路聊天中測試
註冊 Bot 之後，選取 [在網路聊天中測試] 來開啟 [網路聊天] 窗格。 在網路聊天中輸入 "hello"。

  ![在網路聊天中測試 Bot](./media/luis-tutorial-node-bot/bot-service-web-chat.png)

Bot 會說出 "You have reached Greeting. You said: hello" 來作為回應。 這確認了 Bot 已收到您的訊息，並已將其傳遞給它所建立的預設 LUIS 應用程式。 此預設 LUIS 應用程式偵測到 Greeting 意圖。 在下一個步驟中，您會將 Bot 連線至您先前建立的 LUIS 應用程式，而不是預設的 LUIS 應用程式。

## <a name="connect-your-luis-app-to-the-bot"></a>將您的 LUIS 應用程式連線至 Bot

在第一個瀏覽器視窗中開啟 [應用程式設定]，然後編輯 [LuisAppId] 欄位以包含您 LUIS 應用程式的應用程式識別碼。

  ![在 Azure 中更新 LUIS 應用程式識別碼](./media/luis-tutorial-node-bot/bot-service-app-id.png)

如果您沒有 LUIS 應用程式識別碼，請使用您用來登入 Azure 的相同帳戶來登入 [LUIS](luis-reference-regions.md) 網站。 選取 [My apps] \(我的應用程式\)。 

1. 尋找您先前建立並包含來自 HomeAutomation 定義域之意圖和實體的 LUIS 應用程式。

2. 在 LUIS 應用程式的 [Settings] \(設定\) 頁面中，尋找並複製應用程式識別碼。

3. 如果您尚未進行應用程式定型，請選取右上方的 [Train] \(定型\) 按鈕來進行應用程式定型。

4. 如果您尚未發佈應用程式，請選取頂端導覽列中的 [PUBLISH] \(發佈\) 來開啟 [Publish] \(發佈\) 頁面。 選取 [Production] \(生產\) 位置和 [Publish] \(發佈\) 按鈕。

## <a name="modify-the-bot-code"></a>修改 Bot 程式碼

移至第二個瀏覽器視窗 (如果仍處於開啟狀態) 或在第一個瀏覽器視窗中，選取 [建置]，然後選取 [開啟線上程式碼編輯器]。

   ![開啟線上程式碼編輯器](./media/luis-tutorial-node-bot/bot-service-build.png)

在程式碼編輯器中，開啟 `app.js`。 其中包含下列程式碼：

```javascript
/*-----------------------------------------------------------------------------
A simple Language Understanding (LUIS) bot for the Microsoft Bot Framework. 
-----------------------------------------------------------------------------*/

var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})
```

系統會忽略 app.js 中現有的意圖。 您可以保留它們。 

## <a name="add-a-dialog-that-matches-homeautomationturnon"></a>新增與 HomeAutomation.TurnOn 相符的對話方塊

複製下列程式碼，然後新增至 `app.js`。

```javascript
bot.dialog('TurnOn',
    (session) => {
        session.send('You reached the TurnOn intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'HomeAutomation.TurnOn'
})
```

連結至對話方塊之 [triggerAction][triggerAction] 上的 [matches][matches] 選項會指定意圖的名稱。 每當 Bot 收到來自使用者的語句時，辨識器都會執行。 如果它所偵測到的最高分意圖與繫結至對話方塊的 `triggerAction` 相符，Bot 就會叫用該對話方塊。

## <a name="add-a-dialog-that-matches-homeautomationturnoff"></a>新增與 HomeAutomation.TurnOff 相符的對話方塊

複製下列程式碼，然後新增至 `app.js`。

```javascript
bot.dialog('TurnOff',
    (session) => {
        session.send('You reached the TurnOff intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'HomeAutomation.TurnOff'
})
```
## <a name="test-the-bot"></a>測試 Bot

在 Azure 入口網站中，選取 [在網路聊天中測試] 來測試 Bot。 嘗試輸入 "Turn on the lights" 和 "turn off my heater" 之類的訊息，來叫用您已新增至 Bot 的意圖。
   ![在網路聊天中測試 HomeAutomation Bot](./media/luis-tutorial-node-bot/bot-service-chat-results.png)

> [!TIP]
> 如果您發現 Bot 並未總是辨識出正確的意圖或實體，請提供更多範例語句來進行 LUIS 應用程式定型，以改善應用程式的效能。 您無須對 Bot 程式碼進行任何修改，即可將 LUIS 應用程式重新定型。 請參閱[新增範例語句](https://docs.microsoft.com/azure/cognitive-services/LUIS/add-example-utterances) \(英文\) 和[對您的 LUIS 應用程式進行定型和測試](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-interactive-test) \(英文\)。

## <a name="learn-more-about-bot-framework"></a>深入了解 Bot Framework
深入了解 [Bot Framework](https://dev.botframework.com/)，以及 [3.x](https://github.com/Microsoft/BotBuilder) 和 [4.x](https://github.com/Microsoft/botbuilder-js) SDK。

## <a name="next-steps"></a>後續步驟

<!-- From trying the bot, you can see that the recognizer can trigger interruption of the currently active dialog. Allowing and handling interruptions is a flexible design that accounts for what users really do. Learn more about the various actions you can associate with a recognized intent.--> 您可以嘗試將其他意圖 (例如 Help、Cancel 及 Greeting) 新增至 LUIS 應用程式。 然後為新意圖新增對話方塊，並使用 Bot 來測試它們。 

<!-- 
> [!NOTE] 
> The default LUIS app that the template created contains example utterances for Cancel, Greeting, and Help intents. In the list of apps, find the app that begins with the name specified in **App name** in the **Bot Service** blade when you created the Bot Service. -->

> [!div class="nextstepaction"]
> [新增意圖](./luis-how-to-add-intents.md)
> [語音促發](https://docs.microsoft.com/bot-framework/bot-service-manage-speech-priming)


[intentDialog]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: https://docs.microsoft.com/azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://github.com/Microsoft/BotBuilder/tree/master/Node/examples/feature-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://github.com/Microsoft/BotBuilder/blob/master/Node/examples/basics-naturalLanguage/app.js

[LUISBotSample]: https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS

[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html


<!-- Old Links -->
[Github-BotFramework-Emulator-Download]: https://aka.ms/bot-framework-emulator
[Github-LUIS-Samples]: https://github.com/Microsoft/LUIS-Samples
[Github-LUIS-Samples-node-hotel-bot]: https://github.com/Microsoft/LUIS-Samples/tree/master/bot-integration-samples/hotel-finder/nodejs
[NodeJs]: https://nodejs.org/
[BFPortal]: https://dev.botframework.com/
[RegisterInstructions]: https://docs.microsoft.com/bot-framework/portal-register-bot
[BotFramework]: https://docs.microsoft.com/bot-framework/
[LUIS]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions

