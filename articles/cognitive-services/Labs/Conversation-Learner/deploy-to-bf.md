---
title: 如何部署對話學習模組聊天機器人 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 學習如何部署對話學習模組聊天機器人。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: bb977df92cf0ada1e50a929a9ea714313a70165a
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171471"
---
# <a name="how-to-deploy-a-conversation-learner-bot"></a>如何部署對話學習模組聊天機器人

本文件說明如何將對話學習模組聊天機器人部署於本機或部署於 Azure。

## <a name="prerequisite-determine-the-model-id"></a>必要條件：判斷模型識別碼 

若要在對話學習模組 UI 之外執行聊天機器人，您必須設定聊天機器人將使用的對話學習模組模型識別碼，亦即對話學習模組雲端中機器學習模型的識別碼  (相反地，透過對話學習模組 UI 執行聊天機器人時，UI 會選擇模型識別碼)。  

取得模型識別碼的方法如下：

1. 啟動聊天機器人和對話學習模組 UI。  請參閱快速入門指南的完整指示；摘要如下：

    在一個命令視窗中：

    ```
    [open a command window]
    cd my-bot
    npm start
    ```

    在另一個命令視窗中

    ```bash
    [open second command prompt window]
    cd cl-bot-01
    npm run ui
    ```

2. 將瀏覽器開啟至 http://localhost:5050 

3. 按一下要取得識別碼的對話學習模組模型

4. 按一下左側瀏覽列中的 [設定]。

5. 「模型識別碼」GUID 會顯示在頁面頂端附近。

## <a name="option-1-deploying-a-conversation-learner-bot-to-run-locally"></a>選項 1：部署對話學習模組聊天機器人在本機執行

這會將聊天機器人部署到本機電腦，並顯示如何使用 Bot Framework 模擬器存取聊天機器人。

### <a name="configure-your-bot-for-access-outside-the-conversation-learner-ui"></a>對於對話學習模組 UI 外部的存取設定聊天機器人

在本機執行聊天機器人時，將應用程式識別碼新增至聊天機器人的 `.env` 檔案：

    ```
    CONVERSATION_LEARNER_MODEL_ID=<YOUR_MODEL_ID>
    ```

然後啟動聊天機器人：

    ```
    [open a command window]
    cd my-bot
    npm start
    ```

聊天機器人此時將在本機執行。  您可以使用 Bot Framework 模擬器存取聊天機器人。

### <a name="download-and-install-the-emulator"></a>下載並安裝模擬器

    ```
    git clone https://github.com/Microsoft/BotFramework-Emulator
    npm install
    npm run build
    npm start
    ```

### <a name="connect-the-emulator-to-your-bot"></a>將模擬器連接到聊天機器人

1. 在模擬器左上方的「輸入端點 URL」方塊中，輸入 `http://127.0.0.1:3978/api/messages`。  將其他欄位保留空白，並按一下 [連線]。

2. 您現在正與聊天機器人交談。

## <a name="option-2-deploy-to-azure"></a>選項 2：部署至 Azure

採用與發佈其他任何聊天機器人類似的方式，發佈對話學習模組聊天機器人。 大致而言，您會將程式碼上傳至託管的網站，並設定適當的組態值，然後向各種通道註冊聊天機器人。 這段影片詳細說明如何使用 Azure Bot 服務發佈聊天機器人。

部署並執行聊天機器人後，即可使用 Azure Bot Channel Registration 將不同的通道連接到聊天機器人，例如 Facebook、Teams、Skype 等等。 如需該程序的說明文件，請參閱： https://docs.microsoft.com/en-us/bot-framework/bot-service-quickstart-registration

以下是將對話學習模組聊天機器人部署至 Azure 的逐步指示。  這些指示假設可從以雲端為基礎的來源 (例如 VSTS、GitHub、BitBucket 或 OneDrive) 取得您的聊天機器人，並且將設定您的聊天機器人進行持續部署。

1. 登入 Azure 入口網站，網址為 https://portal.azure.com

2. 建立新的「Web 應用程式聊天機器人」資源 

    1. 命名該聊天機器人
    2. 按一下 [Bot Template]\(聊天機器人範本\)，選擇 [Node.js]，再選擇 [基本]，然後按一下 [選取] 按鈕
    3. 按一下 [建立] 來建立 Web 應用程式聊天機器人。
    4. 等候 Web 應用程式聊天機器人資源建立。

3. 在 Azure 入口網站中，編輯您剛才建立的 Web 應用程式聊天機器人資源。

    1. 按一下左側的 [應用程式設定] 瀏覽項目
    1. 向下捲動至 [應用程式設定] 區段
    2. 新增下列設定：

        環境變數 | value
        --- | --- 
        CONVERSATION_LEARNER_SERVICE_URI | "https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/"
        CONVERSATION_LEARNER_MODEL_ID      | 應用程式識別碼 GUID (從模型之 [設定] 下方的對話學習模組 UI 中取得)
        LUIS_AUTHORING_KEY               | 此模型的 LUIS 撰寫金鑰
    
    4. 按一下靠近頁面頂端的 [儲存]
    5. 開啟左邊的 [編譯] 瀏覽項目
    6. 按一下 [設定連續部署] 
    7. 按一下部署下的 [安裝] 圖示
    8. 按一下 [必要的設定]
    9. 選取提供聊天機器人程式碼的來源，並設定該來源。
