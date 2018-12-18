---
title: 如何搭配對話學習模組模型使用等候與非等候動作 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用等候與非等候動作。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: a8f7ccf79e750c9f3c21c25c50c3e275db7e4195
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173079"
---
# <a name="wait-and-non-wait-actions"></a>等候與非等候動作

本教學課程會顯示對話學習模組中的等候與非等候動作兩者的差異。

## <a name="video"></a>影片

[![教學課程 2 預覽](http://aka.ms/cl-tutorial-02-preview)](http://aka.ms/blis-tutorial-02)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料

- 等候動作：系統採取「等候」動作之後，將停止採取動作，並等候使用者輸入。
- 非等候動作：系統採取「非等候」動作之後，將立即選擇其他動作 (而不等待使用者輸入)。

## <a name="steps"></a>步驟

### <a name="create-a-new-model"></a>建立新模型

1. 在 Web UI 中，按一下 [新增模型]
2. 在 [名稱] 中，輸入 WaitNonWait。 然後按一下 [建立]。

### <a name="create-the-first-wait-action"></a>建立第一個等候動作

1. 按一下 [動作]，然後按一下 [新增動作]。
2. 在 [回應] 中，輸入「Which animal do you want?」\(您想要什麼動物？\)。
    - 這是等候動作，因此維持勾選 [等候回應] 方塊。
3. 按一下 [建立]。

### <a name="create-a-non-wait-action"></a>建立非等候動作

1. 按一下 [新增動作]
2. 在 [回應] 中，鍵入「Cows say moo」\(牛說哞\)。
3. 取消勾選 [等候回應] 核取方塊。
4. Click Create

### <a name="create-a-second-non-wait-action"></a>建立第二個非等候動作

1. 按一下 [新增動作]
2. 在 [回應] 中，鍵入「Ducks say quack」\(鴨子說呱\)。
3. 取消勾選 [等候回應] 核取方塊。
4. Click Create

![](../media/tutorial2_actions.PNG)

### <a name="train-the-bot"></a>將 Bot 定型

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「hello」\(您好\)
3. 按一下 [Score Actions]\(評分動作\)，然後選取 [Which animal do you want?]\(您想要什麼動物？\)。
4. 輸入「cow」\(哞\)
5. 按一下 [Score Actions]\(評分動作\)，然後選取 [Cows say moo]\(牛說哞\)。
    - 聊天機器人將不會等候輸入，而將採取下一個動作。
2. 選取 [Which animal do you want?]\(您想要什麼動物？\)。
3. 輸入「duck」\(鴨子\)
5. 按一下 [Score Actions]\(評分動作\)，然後選取 [Ducks say quack]\(鴨子說呱\)。

![](../media/tutorial2_dialogs.PNG)

> [!NOTE]
> 聊天機器人對於等候與非等候動作的回應順序。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [實體簡介](./3-introduction-to-entities.md)
