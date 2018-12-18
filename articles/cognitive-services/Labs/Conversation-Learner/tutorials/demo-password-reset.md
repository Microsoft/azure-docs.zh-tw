---
title: 示範對話學習模組模型：密碼重設 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何建立示範對話學習模組模型。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: f633dd375d690a1c3e66a2a6e02ae69665dbe960
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39170628"
---
# <a name="demo-password-reset"></a>示範：密碼重設
這個示範呈現可協助密碼重設的簡單支援小組聊天機器人。 

其中顯示對話學習模組如何學習非一般對話流程、多次序列，包括領域外類別。 這個示範不使用任何程式碼或實體。

## <a name="video"></a>影片

[![示範密碼預覽](http://aka.ms/cl-demo-password-preview)](http://aka.ms/blis-demo-password)

## <a name="requirements"></a>需求
本教學課程需要執行密碼重設聊天機器人

    npm run demo-password

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [教學課程示範密碼重設]。 

### <a name="actions"></a>動作

我們已經建立使用者尋求解決密碼設定問題時可採取的動作。

![](../media/tutorial_pw_reset_actions.PNG)

### <a name="training-dialogs"></a>訓練對話

有許多訓練對話。 另外還有領域外類別的示範，例如，使用者要求「導航」就是在領域外；聊天機器人已經有一些領域外要求的範例，能夠以「我無法提供協助」來回應。

![](../media/tutorial_pw_reset_entities.PNG)

讓我們試試以教學工作階段為例。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「I lost my password」\(我遺失了我的密碼\)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [Is that for your local account or Microsoft account?]\(這適用於本機帳戶或 Microsoft 帳戶？\)
4. 輸入「Local account」\(本機帳戶\)。
5. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [Which version of Windows do you have?]\(您的 Windows 是什麼版本？\)
4. 輸入「Windows 8」。
5. 按一下 [Score Actions]\(評分動作\)。
6. 選取 [SOLUTION: how to reset password on Windows 8.]\(解決方案：如何在 Windows 8 重設密碼。\)
4. 按一下 [Done Teaching]\(完成教學\)。

讓我們試試聊天機器人如何學習領域外類別。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「web search」\(網路搜尋\)。
    - 這是領域外類別的範例。 
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [Sorry, I can't help with that.]\(很抱歉，我無法提供協助。\)
    - 請注意，這個選項的分數目前不高。 但是，經過更多教學之後，分數會變高。
4. 按一下 [Done Teaching]\(完成教學\)。

您現在已了解如何建立基本的支援小組示範，以及如何提供解決方案，並且處理領域外查詢。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [示範 - 披薩訂單](./demo-pizza-order.md)
