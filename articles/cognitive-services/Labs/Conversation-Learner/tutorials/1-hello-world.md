---
title: 如何建立 "Hello World" 對話學習模組模型 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何建立 "Hello World" 對話學習模組模型。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 70b8f25bd699cbdb069892d65bf766ef3953f59d
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39170866"
---
# <a name="how-to-create-a-hello-world-model-with-conversation-learner"></a>如何使用對話學習模組建立 "Hello World" 模型

本教學課程顯示如何開始使用對話學習模組，包括建立動作、以互動方式進行教導，以及修正終端使用者記錄的對話。

## <a name="video"></a>影片

[![教學課程 1 預覽](http://aka.ms/cl-tutorial-01-preview)](http://aka.ms/blis-tutorial-01)


## <a name="requirements"></a>需求
如果尚未準備好，請先確定所有安裝步驟均已完成，包括使用 LUIS 撰寫金鑰建立 `.env` 檔案。  如需詳細資訊，請參閱[快速入門](https://github.com/Microsoft/ConversationLearner-Samples)。

本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="steps"></a>步驟

在 Web UI 中的首頁開始。

### <a name="create-the-model"></a>建立模型
1. 按一下 [新增模型]
2. 在 [名稱] 欄位中，輸入 Hello World
3. Click Create

### <a name="create-an-action"></a>建立動作

1. 按一下 Hello World 模型加以啟動
2. 按一下 [動作]，然後按一下 [新增動作]
    - 動作可以是對話學習模組傳回給使用者、API 呼叫或卡片的文字訊息。
3. 在 [回應] 中，鍵入「Hello World!」
    - 這是聊天機器人將傳回的回應
4. Click Create

您已建立聊天機器人可以執行的第一個動作，也就是傳回文字回應。

### <a name="train-the-bot"></a>訓練聊天機器人

#### <a name="create-the-first-dialog"></a>建立第一個對話方塊

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)
2. 輸入使用者將在對話開始時說出的語句範例，例如，「您好」。
3. 按一下 [Score Actions]\(評分動作\)
4. 選取 [Hello World!]
    - 這會建立一個回合的範例對話。 
2. 輸入「goodbye」(再見)
3. 按一下 [Score Actions]\(評分動作\)
4. 按一下 [新增動作]，然後在 [回應] 中輸入「Goodbye!」(再見！) 然後按一下 [建立]
5. 按一下 [Done Teaching]\(完成教學\)。 這將結束此訓練對話。

現在系統中會有一個教學對話方塊。

#### <a name="continue-teaching-the-bot"></a>繼續訓練聊天機器人
讓我們再進行一次訓練，並查看聊天機器人的回應。

1. 按一下 [New Train Dialog]\(新增訓練對話\)
2. 輸入「hi there」(嗨，你好)
    - 這和第一個對話很類似，我們預期可以從聊天機器人得到不錯的分數。
2. 按一下 [Score Actions]\(評分動作\)
    - 位置和分數可能已經不夠精確，因而需要更多教學。
3. 按一下「Hello World!」旁邊的 [選取]
4. 然後輸入「bye」(拜拜)
5. 按一下 [Score Actions]\(評分動作\)
6. 選取「Goodbye!」(再見！)
7. 按一下 [Done Teaching]\(完成教學\)

![](../media/tutorial1_actions.PNG)

我們將執行另一個教學工作階段，以查看聊天機器人如何運作。

使用「hi」\(嗨\)和「byebye」\(拜拜\)重複上述步驟，並注意您按一下 [Score Actions]\(評分動作\) 時聊天機器人的位置和分數出現的變化。

您現在可以使用「howdy」\(你好\)和「good bye」\(再見\) 重複這些步驟，並注意聊天機器人學習這次互動的分數改進。

![](../media/tutorial1_dialogs.PNG)

### <a name="test-the-bot-as-an-end-user"></a>以終端使用者身分測試聊天機器人

1. 按一下 [Log Dialogs]\(記錄對話方塊\)，然後按一下 [New Log Dialog]\(新增記錄對話方塊\)
2. 鍵入「hello there」(哈囉，你好)
3. 然後輸入「bye」(拜拜)

您也可以使用「bye」(拜拜) 開始對話，並注意聊天機器人的回應。

### <a name="view-conversations-in-the-log-dialogs"></a>檢視 [Log Dialogs]\(記錄對話方塊\) 中的對話

在 [對話記錄方塊] 中，您可以檢視對話的清單、更新，並且將互動另存為訓練對話。 若要這樣做：

1. 按一下對話的記錄
2. 如果對話順利，請按一下上一個動作，例如「再見」。
3. 按一下以選取的建議的回應。 
    - 您也可以選取或加入另一個動作。
4. 然後按一下 [完成]，將此另存為訓練對話。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [等候與非等候動作](./2-wait-vs-nonwait-actions.md)