---
title: 使用 C# 的 LUIS Bot - Web 應用程式 Bot - Bot Framework SDK 3.0
titleSuffix: Azure Cognitive Services
description: 使用 C#，建置與 Language Understanding (LUIS) 整合的聊天機器人。 此聊天機器人會使用預先建置的 HomeAutomation 領域來快速實作聊天機器人解決方案。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 0ad95d5683ea10803c2f3986be97617051f191cf
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032269"
---
# <a name="luis-bot-in-c"></a>使用 C# 的 LUIS Bot

使用 C#，建置與 Language Understanding (LUIS) 整合的聊天機器人。 此聊天機器人會使用預先建置的 HomeAutomation 領域來快速實作聊天機器人解決方案。 此 Bot 是使用 Bot Framework 3.x 和 Azure Web 應用程式 Bot 所建置的。

## <a name="prerequisite"></a>必要條件

* [HomeAutomation LUIS 應用程式](luis-get-started-create-app.md)。 來自此 LUIS 應用程式的意圖會對應至 Bot 的對話方塊處理常式。 

## <a name="luis-homeautomation-intents"></a>LUIS HomeAutomation 意圖

| 意圖 | 範例語句 | Bot 功能 |
|:----:|:----------:|---|
| HomeAutomation.TurnOn | 開燈。 | 偵測到 LUIS 意圖 `HomeAutomation.TurnOn` 時，Bot 會叫用 `OnIntent` 對話方塊處理常式。 此對話方塊可供您呼叫 IoT 服務來開啟裝置，並告訴使用者該裝置已開啟。 |
| HomeAutomation.TurnOff | 關閉臥室電燈。 | 偵測到 LUIS 意圖 `HomeAutomation.TurnOff` 時，Bot 會叫用 `OffIntent` 對話方塊處理常式。 此對話方塊可供您呼叫 IoT 服務來關閉裝置，並告訴使用者該裝置已關閉。 |

## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用 Bot 服務來建立 Language Understanding Bot

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取左上方功能表中的 [建立新資源]。

    ![建立新資源](./media/luis-tutorial-cscharp-web-bot/bot-service-creation.png)

2. 在搜尋方塊中，搜尋 **Web 應用程式 Bot**。 

    ![建立新資源](./media/luis-tutorial-cscharp-web-bot/bot-service-selection.png)

3. 在 [Web 應用程式 Bot] 視窗中，按一下 [建立]。

4. 在 [Bot 服務] 中提供必要資訊，然後按一下 [建立]。 這會建立 Bot 服務和 LUIS 應用程式，並將其部署到 Azure。 如果您想要使用[語音促發](https://docs.microsoft.com/bot-framework/bot-service-manage-speech-priming)，請先檢閱[區域需求](luis-resources-faq.md#what-luis-regions-support-bot-framework-speech-priming)，再建立您的 Bot。 
    * 將 [應用程式名稱] 設定為您 Bot 的名稱。 將 Bot 部署到雲端時，此名稱會用來作為子網域 (例如 mynotesbot.azurewebsites.net)。 <!-- This name is also used as the name of the LUIS app associated with your bot. Copy it to use later, to find the LUIS app associated with the bot. -->
    * 選取訂用帳戶、[資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)、App Service 方案，以及[位置](https://azure.microsoft.com/regions/)。
    * 針對 [Bot 範本]，請選取：
        * **SDK v3**
        * **C#**
        * **語言理解**
    * 選取 [LUIS 應用程式位置]。 這是用來建立應用程式的撰寫[區域](luis-reference-regions.md)。
    * 選取法律聲明的確認核取方塊。 法律聲明的條款在此核取方塊下方。

    ![Bot 服務](./media/luis-tutorial-cscharp-web-bot/bot-service-setting-callout-template.png)


5. 確認已部署 Bot 服務。
    * 按一下 [通知] (位於 Azure 入口網站頂端邊緣的鈴鐺圖示)。 通知會從 [部署已開始] 變更為 [部署成功]。
    * 在通知變更為 [部署成功] 之後，按一下該通知上的 [前往資源]。

> [!Note]
> 這個 Web 應用程式 Bot 建立程序也會為您建立一個新的 LUIS 應用程式。 此應用程式已為您進行定型並發佈。 

## <a name="try-the-default-bot"></a>試用預設 Bot

選取 [通知] 核取方塊來確認已部署 Bot。 通知會從 [部署進行中] 變更為 [部署成功]。 按一下 [前往資源] 按鈕以開啟 Bot 的資源。

部署 Bot 之後，按一下 [在網路聊天中測試] 來開啟 [網路聊天] 窗格。 在網路聊天中輸入 "hello"。

  ![在網路聊天中測試 Bot](./media/luis-tutorial-cscharp-web-bot/bot-service-web-chat.png)

Bot 會說出 "You have reached Greeting. You said: hello" 來作為回應。  此回應確認了 Bot 已收到您的訊息，並已將其傳遞給它所建立的預設 LUIS 應用程式。 此預設 LUIS 應用程式偵測到 Greeting 意圖。 在下一個步驟中，您會將 Bot 連線至您先前建立的 LUIS 應用程式，而不是預設的 LUIS 應用程式。

## <a name="connect-your-luis-app-to-the-bot"></a>將您的 LUIS 應用程式連線至 Bot

開啟 [應用程式設定]，然後編輯 [LuisAppId] 欄位以包含您 LUIS 應用程式的應用程式識別碼。 如果您是在美國西部以外的區域建立 HomeAutomation LUIS 應用程式，則也必須變更 [LuisAPIHostName]。 [LuisAPIKey] 目前設定為您的撰寫金鑰。 當您的流量超出免費層配額時，請將此金鑰變更為您的端點金鑰。 

  ![在 Azure 中更新 LUIS 應用程式識別碼](./media/luis-tutorial-cscharp-web-bot/bot-service-app-settings.png)

> [!Note]
> 如果您沒有[家庭自動化應用程式](luis-get-started-create-app.md)的 LUIS 應用程式識別碼，請使用您用來登入 Azure 的相同帳戶來登入 [LUIS](luis-reference-regions.md) 網站。 
> 1. 按一下 [My apps] \(我的應用程式\)。 
> 2. 尋找您先前建立並包含來自 HomeAutomation 定義域之意圖和實體的 LUIS 應用程式。
> 3. 在 LUIS 應用程式的 [Settings] \(設定\) 頁面中，尋找並複製應用程式識別碼。 確定該應用程式已[定型](luis-interactive-test.md)並[發佈](luis-how-to-publish-app.md)。 

    > [!WARNING]
    > If you delete your app ID or LUIS key, the bot will stop working.

## <a name="modify-the-bot-code"></a>修改 Bot 程式碼

1. 按一下 [建置]，然後按一下 [開啟線上程式碼編輯器]。

   ![開啟線上程式碼編輯器](./media/luis-tutorial-cscharp-web-bot/bot-service-build.png)

2. 在 `build.cmd` 上按一下滑鼠右鍵，然後選擇 [從主控台執行]來建置應用程式。 服務會自動為您完成數個建置步驟。 當完成狀態為「順利完成」時，即表示建置完成。

3. 在程式碼編輯器中，開啟 `/Dialogs/BasicLuisDialog.cs`。 其中包含下列程式碼：

   [!code-csharp[Default BasicLuisDialog.cs](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/Default_BasicLuisDialog.cs "Default BasicLuisDialog.cs")]

## <a name="change-code-to-homeautomation-intents"></a>將程式碼變更為 HomeAutomation 意圖


1. 移除用於 **Greeting**、**Cancel** 及 **Help** 的三個意圖屬性和方法。 在 HomeAutomation 預先建置的定義域中並不使用這些意圖。 請務必保留 **None** 意圖屬性和方法。 

2. 將相依性與其他相依性一起新增至檔案頂端：

   [!code-csharp[Dependencies](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs?range=4-5&dedent=8 "dependencies")]

3. 在 `BasicLuisDialog ` 類別頂端，新增常數來管理字串：

   [!code-csharp[Add Intent and Entity Constants](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs?range=23-32&dedent=8 "Add Intent and Entity Constants")]

4. 在 `BasicLuisDialog ` 類別內，新增新意圖 `HomeAutomation.TurnOn` 和 `HomeAutomation.TurnOff` 的程式碼：

   [!code-csharp[Add Intents](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs?range=61-71&dedent=8 "Add Intents")]

5. 在 `BasicLuisDialog ` 類別內，新增程式碼來取得 LUIS 所找到的任何實體：

   [!code-csharp[Collect entities](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs?range=34-53&dedent=8 "Collect entities")]

6. 變更 `BasicLuisDialog ` 類別中的 **ShowLuisResult** 方法來捨入分數、收集實體，以及在 Chatbot 中顯示回應訊息：

   [!code-csharp[Display message in chatbot](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs?range=73-83&dedent=8 "Display message in chatbot")]

## <a name="build-the-bot"></a>建置 Bot
在程式碼編輯器中，於 `build.cmd` 上按一下滑鼠右鍵，然後選取 [從主控台執行]。

![建置 Web Bot ](./media/luis-tutorial-cscharp-web-bot/bot-service-build-run-from-console.png)

程式碼檢視畫面會改換成顯示建置進度和結果的終端機視窗。

![建置 Web Bot 成功](./media/luis-tutorial-cscharp-web-bot/bot-service-build-success.png)

> [!TIP]
> 有一個建置 Bot 的替代方法，就是選取頂端藍色列中的 Bot 名稱，然後選取 [開啟 Kudu 主控台]。 主控台會開啟至 **D:\home**。 
> 
> 輸入 `cd site\wwwroot` 來將目錄變更為 **D:\home\site\wwwroot**
>
> 輸入 `build.cmd` 來執行建置指令碼

## <a name="test-the-bot"></a>測試 Bot

在 Azure 入口網站中，按一下 [在網路聊天中測試] 來測試 Bot。 輸入 "Turn on the lights" 和 "turn off my heater" 之類的訊息，來叫用您已新增至 Bot 的意圖。

   ![在網路聊天中測試 HomeAutomation Bot](./media/luis-tutorial-cscharp-web-bot/bot-service-chat-results.png)

> [!TIP]
> 您無須對 Bot 程式碼進行任何修改，即可將 LUIS 應用程式重新定型。 請參閱[新增範例語句](https://docs.microsoft.com/azure/cognitive-services/LUIS/add-example-utterances) \(英文\) 和[對您的 LUIS 應用程式進行定型和測試](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-interactive-test) \(英文\)。 

## <a name="download-the-bot-to-debug"></a>下載 Bot 來進行偵錯
如果您的 Bot 無法運作，請將專案下載到您的本機電腦，然後繼續[偵錯](https://docs.microsoft.com/bot-framework/bot-service-debug-bot#debug-a-c-bot)。 

## <a name="learn-more-about-bot-framework"></a>深入了解 Bot Framework
深入了解 [Bot Framework](https://dev.botframework.com/)，以及 [3.x](https://github.com/Microsoft/BotBuilder) 和 [4.x](https://github.com/Microsoft/botbuilder-dotnet) SDK。

## <a name="next-steps"></a>後續步驟

新增 LUIS 意圖和 Bot 服務對話方塊來處理 **Help** **Cancel**及 **Greeting** 意圖。 請務必進行 Web 應用程式 Bot 定型、發佈及[建置](#build-the-bot)。 LUIS 與 Bot 兩者必須具有相同的意圖。

> [!div class="nextstepaction"]
> [新增意圖](./luis-how-to-add-intents.md)
> [語音促發](https://docs.microsoft.com/bot-framework/bot-service-manage-speech-priming)


<!-- Links -->
[Github-BotFramework-Emulator-Download]: https://aka.ms/bot-framework-emulator
[Github-LUIS-Samples]: https://github.com/Microsoft/LUIS-Samples
[Github-LUIS-Samples-cs-hotel-bot]: https://github.com/Microsoft/LUIS-Samples/tree/master/bot-integration-samples/hotel-finder/csharp
[Github-LUIS-Samples-cs-hotel-bot-readme]: https://github.com/Microsoft/LUIS-Samples/blob/master/bot-integration-samples/hotel-finder/csharp/README.md
[BFPortal]: https://dev.botframework.com/
[RegisterInstructions]: https://docs.microsoft.com/bot-framework/portal-register-bot
[BotFramework]: https://docs.microsoft.com/bot-framework/
[VisualStudio]: https://www.visualstudio.com/

<!-- tested on Win10 -->
