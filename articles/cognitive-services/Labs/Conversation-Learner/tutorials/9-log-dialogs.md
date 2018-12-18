---
title: 如何在對話學習模組模型中記錄對話 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何在對話學習模組模型中記錄對話。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 6ceeb9683a979256a8a52347fc74ab758fd1d348
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171430"
---
# <a name="how-to-log-dialogs-in-a-conversation-learner-model"></a>如何在對話學習模組模型中記錄對話

本教學課程顯示如何在對話學習模組介面內執行終端使用者測試、如何記錄對話，以及如何修正記錄的對話來改進您的模型。

## <a name="video"></a>影片

[![教學課程 9 預覽](http://aka.ms/cl-tutorial-09-preview)](http://aka.ms/blis-tutorial-09)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料
您可以使用記錄對話方塊檢閱和修正與使用者進行的對話。  具體來說，您可以修正實體標籤與動作選取項目，以改善訓練模型和整體系統的效能。 

## <a name="steps"></a>步驟

### <a name="create-the-model"></a>建立模型

1. 在 Web UI 中，按一下 [新增模型]
2. 在 [名稱] 中，輸入 LogDialogs。 然後按一下 [建立]。

### <a name="create-an-entity"></a>建立實體

1. 按一下 [實體]，然後按一下 [新增實體]。
2. 在 [實體名稱] 中，輸入城市。
3. 按一下 [建立]。

### <a name="create-two-actions"></a>建立兩個動作

1. 按一下 [動作]，然後按一下 [新增動作]
2. 在 [回應] 中，鍵入「Which city?」(哪座城市？)。
3. 在 [不合格的實體] 中，輸入 $city。
3. Click Create

然後建立第二個動作：

1. 按一下 [動作]，然後按一下 [新增動作]。
3. 在 [回應] 中，鍵入「The weather in $city is probably sunny」($city 的天氣可能是晴天)。
4. 必要的實體，輸入 $city。
4. 按一下 [建立]。

您現在有兩個動作。

### <a name="train-the-bot"></a>訓練聊天機器人

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「what's the weather」(天氣如何)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [Which city?]\(哪座城市？\)。
2. 輸入「Seattle」(西雅圖)。
3. 按兩下 [Seattle]\(西雅圖\)，並選取 [city]\(城市\)。
    - 這會將它標示為城市實體。
5. 按一下 [Score Actions]\(評分動作\)
6. 選取 [$city is probably sunny]\($city 的天氣可能是晴天\)。
7. 按一下 [Done Teaching]\(完成教學\)。

加入另一個範例對話：

1. 按一下 [新增動作]，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「what's the weather in Seattle?」(西雅圖的天氣如何？)。 注意西雅圖已標記為實體。
5. 按一下 [Score Actions]\(評分動作\) 
6. 選取 [$city is probably sunny]\($city 的天氣可能是晴天\)。
7. 按一下 [Done Teaching]\(完成教學\)。

### <a name="try-the-bot-as-the-user"></a>以使用者身分嘗試聊天機器人
先假設我們已向使用者部署此聊天機器人。

1. 按一下 [Log Dialogs]\(記錄對話方塊\)。
2. 按一下 [New Chat Session]\(新增交談工作階段\)。
    - 這會在 UI 的左邊呈現使用者在網站聊天控制項中遇到的聊天機器人。 您可以忽略右邊的空白區域。
3. 鍵入「hello」(您好)。
4. 聊天機器人回應：「which city?」(哪座城市？)
4. 鍵入「Boston」(波士頓)。
5. 聊天機器人回應：「which city?」(哪座城市？)
    - 這似乎不正確。 讓我們儲存這段對話。
2. 按一下 [完成測試]。

讓我們開始新的工作階段：

2. 按一下 [New Chat Session]\(新增交談工作階段\)。
3. 鍵入「forecast for Boston」(波士頓的天氣預報)。
4. 聊天機器人回應：「which city?」(哪座城市？)
2. 按一下 [Done Teaching]\(完成教學\)。

現在讓我們修正第二個對話方塊：

1. 按一下 [Log Dialogs]\(記錄對話方塊\) 下的 [forecast for Boston]\(波士頓的天氣預報\)。
    - 這會開啟對話。
    - 如果您按一下對話的使用者端 (在「波士頓的天氣預報」上)，您可以變更實體標籤。
    - 如果您按一下系統端 (在「哪座城市」上)，您可以變更選取的動作。
5. 按一下 [forecast for Boston]\(波士頓的天氣預報\)。 
    - 根本原因是並未將波士頓標記為實體。 我們需要變更它。
    - 按兩下 [Boston]\(波士頓\)，然後選取城市。
    - 按一下 [送出變更]，並按一下 [儲存]。 這將按照您進行的變更來建立訓練對話，然後將您移至您變更時的訓練對話方塊。
6. 選取 [$city is probably sunny]\($city 的天氣可能是晴天\)。
7. 按一下 [Done Teaching]\(完成教學\)。 如果您立即移至訓練對話，您會看到新動作加入。

![](../media/tutorial9_logdiag1.PNG)

現在讓我們修正另一個對話方塊：

1. 按一下 [Log Dialogs]\(記錄對話方塊\) 之下的 [hello]\(哈囉\)。
    - 這會開啟對話。
3. 'hello' (哈囉) 的回應為 'which city' (哪座城市)。 但是，我們想要將它變更為更有意義的回應。 更好的答案會是「哈囉，我是天氣聊天機器人」之類的回應。 不過由於沒有動作這麼做，因此我們必須建立一個動作。
4. 按一下 [動作]。
    - 在 [回應] 中，鍵入「I'm the weather bot. I can help with forecasts.」(我是天氣聊天機器人。我可以協助進行天氣預報。)
6. 取消勾選 [等候回應] 核取方塊使其成為非等候動作。
7. 按一下 [建立]。
8. 然後按一下以選取新動作。 然後按一下 [儲存]。
    - 這會使您返回訓練工作階段中的該時間點。
6. 按一下以選取 ['which city?]\(哪座城市？\)
7. 鍵入「Boston」(波士頓)。 按兩下將波士頓標記為實體 (如果尚未標記)。
8. 按一下 [Score Actions]\(評分動作\)。
9. 按一下以選取 [The weather in $city is probably sunny.]\($city 的天氣可能是晴天。\)
10. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial9_addnewaction.PNG)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [實體偵測回呼](./10-entity-detection-callback.md)
