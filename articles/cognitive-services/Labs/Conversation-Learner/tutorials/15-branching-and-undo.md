---
title: 如何搭配對話學習模組模型使用分支與復原作業 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用分支與復原作業。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 05140693026e21a73b756ed0ea7bc9936bef067e
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173293"
---
# <a name="how-to-use-branching-and-undo-operations"></a>如何使用分支與復原作業
在本教學課程中，我們將介紹復原與分支作業。


## <a name="details"></a>詳細資料
- 復原：可讓開發人員「復原」使用者輸入或動作選擇。 在背景，「復原」實際上會建立新的對話，並重新播放到上一個步驟。  這表示將再次呼叫對話中的實體偵測回呼和 API 呼叫。

- 分支：建立新的訓練對話，前面的對話內容跟現有的訓練對話相同，省下手動重新輸入對話的時間。 在背景，「分支」會建立新的對話，並重新播放現有訓練對話到選取的步驟。  這表示將再次呼叫對話中的實體偵測回呼和 API 呼叫。


## <a name="requirements"></a>需求
本教學課程需要執行披薩訂單聊天機器人：

    npm run demo-pizza

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [TutorialDemo Pizza Order]。 

如需披薩訂單示範的詳細資訊，請參閱披薩訂單教學課程。

## <a name="undo"></a>復原

我們會復原一部分的對話，並從該步驟重新建立對話。

### <a name="training-dialogs"></a>訓練對話
讓我們開始訓練工作階段。 

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「order a pizza」(訂購披薩)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [what would you like on your pizza?]\(您要在披薩上加什麼？\)
4. 輸入「mushrooms and cheese」(蘑菇和起士)。
5. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [you have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)。
6. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
7. 輸入「remove mushrooms and add peppers」(不要蘑菇而且要加黑胡椒)。
    - 選取蘑菇並取消勾選配料實體。 我們正在建立將復原的動作。
2. 按一下 [Undo Step]\(復原步驟\)。
    - 最後一個項目已遭移除，而且我們回到了 'Would you like anything else?' (您還需要其他任何服務嗎？)  (請參考下方的螢幕擷取畫面)
2. 輸入「remove mushrooms and add peppers」(不要蘑菇而且要加黑胡椒)。
8. 按一下以選取 [you have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)
    - 請確定已正確選取兩個實體。
2. 按一下 [Score Actions]\(評分動作\)。 您現在可以繼續使用修正後的對話。
4. 按一下 [Done Teaching]\(完成教學\)。

現在您已了解如何使用復原來移除使用者輸入與動作。

![](../media/tutorial15_undo.PNG)

## <a name="branch"></a>分支

作為示範，讓我們開啟現有的訓練對話，並藉由分支建立另一個訓練對話。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按 [新訂單] 來開啟現有的對話。 
2. 按一下對話中的最後一個「否」(請參閱下方的螢幕擷取畫面)。
3. 按一下 [分支]。
    - 'no' (否) 已被移除，而且此時已將整段對話複製到新的對話中。 
    - 如此便不需要重新輸入先前的對話回合，即可從這個時間點探索新的「分支」。
1. 輸入「yes」(是)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 選取 [You have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)。
6. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
7. 輸入「no」(否)。
4. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial15_branch.PNG)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [版本控制和標記](./16-versioning-and-tagging.md)
