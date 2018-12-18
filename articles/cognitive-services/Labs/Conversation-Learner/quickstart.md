---
title: 如何使用 Node.js 建立對話學習模組模型 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何使用 Node.js 建立對話學習模組模型。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: bc0cf0900ec9f87c75091b3bf219d92e0859aa1f
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39282134"
---
# <a name="create-a-conversation-learner-model-using-nodejs"></a>如何使用 Node.js 建立對話學習模組模型

對話學習模組可降低建置 Bot 的複雜度。 它可讓混合式開發工作流程允許使用手動撰寫的程式碼和機器學習，以減少撰寫 Bot 所需的程式碼數量。 您仍然可以為模型的某些固定部分撰寫程式碼，例如，檢查使用者是否已登入，或發出 API 要求來檢查商店庫存。 不過，狀態與動作選取項目的其他變更則可以自領域專家或開發人員所提供的範例對話方塊中學習。

## <a name="invitation-required"></a>需要邀請

*需要具有邀請才能存取專案對話學習模組。*

專案對話學習模組包含已新增至 Bot 的 SDK，以及 SDK 用以存取機器學習的雲端服務。  目前，存取專案對話學習模組雲端服務需要邀請。  如果您尚未獲得邀請，請[申請邀請](https://aka.ms/conversation-learner-request-invite)。  如果您尚未收到邀請，則無法存取雲端 API。

## <a name="prerequisites"></a>必要條件

- Node 8.5.0 或更高版本及 npm 5.3.0 或更高版本。 請從 [https://nodejs.org](https://nodejs.org) 安裝。
  
- LUIS 撰寫金鑰：

  1. 登入 [http://www.luis.ai](http://www.luis.ai)。

  2. 按一下右上角的名稱，然後按一下 [設定]

  3. 撰寫金鑰隨即顯示在產生的頁面上

  (您的 LUIS 撰寫金鑰提供 2 個角色。  首先，它會作為您的對話學習模組撰寫金鑰。  其次，對話學習模組會使用 LUIS 進行實體擷取；LUIS 撰寫金鑰則用來代表您建立 LUIS 模型)

- Google Chrome 網頁瀏覽器。 請從 [https://www.google.com/chrome/index.html](https://www.google.com/chrome/index.html) 安裝。

- git。 請從 [https://git-scm.com/downloads](https://git-scm.com/downloads) 安裝。

- VSCode。 請從 [https://code.visualstudio.com/](https://code.visualstudio.com/) 安裝。 請注意，建議使用此項目，但並非必要。

## <a name="quick-start"></a>快速入門 

1. 安裝並建置：

    ```bash    
    git clone https://github.com/Microsoft/ConversationLearner-Samples cl-bot-01
    cd cl-bot-01
    npm install
    npm run build
    ```

    > [!NOTE]
    > 在 `npm install` 期間，如果發生下列錯誤，您可以予以忽略：`gyp ERR! stack Error: Can't find Python executable`

2. 設定：

   在 `cl-bot-01` 目錄中，建立稱為 `.env` 的檔案。  此檔案的內容應該是：

   ```
   NODE_ENV=development
   LUIS_AUTHORING_KEY=<your LUIS authoring key>
   ```

3. 啟動 Bot：

    ```
    npm start
    ```

    這會執行 `cl-bot-01/src/app.ts` 中的一般空白 Bot。

3. 執行對話學習模組 UI：

    ```bash
    [open second command prompt window]
    cd cl-bot-01
    npm run ui
    ```

4. 將瀏覽器開啟至 http://localhost:5050 

您現在正在使用對話學習模組，並且可以建立及教導對話學習模組模型。  

> [!NOTE]
> 在啟動時，專案對話學習模組可透過邀請供您使用。  如果 http://localhost:5050 顯示 HTTP `403` 錯誤，這表示您的帳戶尚未獲得邀請。  請[申請邀請](https://aka.ms/conversation-learner-request-invite)。

## <a name="tutorials-demos-and-switching-between-bots"></a>教學課程、示範以及在 Bot 之間切換

上述指示啟動了一般空白 Bot。  若要改為執行教學課程或示範 Bot：

1. 如果您已開啟對話學習模組 Web UI，請返回位於 http://localhost:5050/home 的模型清單。
    
2. 如果正在執行另一個 Bot (如 `npm start` 或 `npm run demo-pizza`)，請將其停止。  您不需要停止 UI 處理序，或關閉網頁瀏覽器。

3. 從命令列執行示範 Bot (上述的步驟 2)。  示範包括：

  ```bash
  npm run tutorial-general
  npm run tutorial-entity-detection
  npm run tutorial-session-callbacks
  npm run tutorial-api-calls
  npm run tutorial-hybrid
  npm run demo-password
  npm run demo-pizza
  npm run demo-storage
  npm run demo-vrapp
  ```

4. 如果您尚未開啟，請在 Chrome 中載入 http://localhost:5050/home，以切換到對話學習模組 Web UI。 

5. 按一下 [匯入教學課程] (只需執行一次)。  這需要約一分鐘，就會將所有教學課程的對話學習模組模型複製到您的對話學習模組帳戶。

6. 按一下對話學習模組 UI 中與您所啟動的示範相對應的示範模型。

示範的來源檔案位在 `cl-bot-01/src/demos`

## <a name="create-a-bot-which-includes-back-end-code"></a>建立包括後端程式碼的 Bot

1. 如果您已開啟對話學習模組 Web UI，請返回位於 http://localhost:5050/home 的模型清單。
    
2. 如果正在執行 Bot (如 `npm run demo-pizza`)，請將其停止。  您不需要停止 UI 處理序，或關閉網頁瀏覽器。

3. 若有需要，請編輯 `cl-bot-01/src/app.ts` 中的程式碼。

4. 重新建置並重新啟動 Bot：

    ```bash    
    npm run build
    npm start
    ```

5. 如果您尚未開啟，請在 Chrome 中載入 http://localhost:5050/home，以切換到對話學習模組 Web UI。 

6. 在 UI 中建立新的對話學習模組模型，並開始進行教導。

7. 若要在 `cl-bot-01/src/app.ts` 中變更程式碼，請從步驟 2 開始重複上述步驟。

## <a name="vscode"></a>VSCode

在 VSCode 的 `cl-bot-01/src/app.ts` 中，每個示範及「空白 Bot」都有執行設定。  請開啟 VSCode 中的 `cl-bot-01` 資料夾。

## <a name="advanced-configuration"></a>進階組態

有一個範本 `.env.example` 檔案顯示您可以設定哪些環境變數來設定範例。

您可以藉由將 `.env` 檔案增至專案的根目錄來調整這些連接埠，以避免在電腦上執行的其他服務之間發生衝突：

```bash
cp .env.example .env
```

這會使用標準設定，此設定可讓您在本機上執行 Bot，並開始使用對話學習模組  (稍後若要將您的 Bot 部署到 Bot Framework，則需要對這個檔案進行一些編輯)。

## <a name="support"></a>支援

- 使用 "microsoft cognitive" 在 [Stack Overflow](https://stackoverflow.com) 上標記問題
- 在我們的 [User Voice 頁面](https://aka.ms/conversation-learner-uservoice)上要求功能
- 在我們的 [github 存放庫](https://github.com/Microsoft/ConversationLearner-Samples)中提出問題

## <a name="contributing"></a>參與

此專案採用 [Microsoft 開放原始碼管理辦法](https://opensource.microsoft.com/codeofconduct/)。 如需詳細資訊，請參閱[管理辦法常見問題集](https://opensource.microsoft.com/codeofconduct/faq/)，如有任何其他問題或意見請連絡 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## <a name="source-repositories"></a>來源存放庫

- [conversationlearner-samples](https://github.com/Microsoft/ConversationLearner-Samples)
- [conversationlearner-sdk](https://github.com/Microsoft/ConversationLearner-SDK)
- [conversationlearner-models](https://github.com/Microsoft/ConversationLearner-Models)
- [conversationlearner-ui](https://github.com/Microsoft/ConversationLearner-UI)
- [conversationlearner-webchat](https://github.com/Microsoft/ConversationLearner-WebChat)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Hello world](./tutorials/1-hello-world.md)
