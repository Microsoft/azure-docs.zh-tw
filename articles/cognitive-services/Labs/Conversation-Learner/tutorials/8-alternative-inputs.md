---
title: 如何搭配對話學習模組使用替代輸入 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組使用替代輸入。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 8d3b3f419ceacbb9a6fe2b19cf68ea6873de536f
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171012"
---
# <a name="how-to-use-alternative-inputs"></a>如何使用替代輸入

本教學課程顯示如何在教學介面中使用使用者輸入的「替代輸入」欄位。

## <a name="video"></a>影片

[![教學課程 8 預覽](http://aka.ms/cl-tutorial-08-preview)](http://aka.ms/blis-tutorial-08)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料
「替代輸入」是使用者可能在訓練對話中的特定時間點所說的替代使用者語句。 替代輸入可讓您更簡潔地指定使用者可能會說的語句，完全不需要在個別訓練對話中列出各種變化。

## <a name="steps"></a>步驟

### <a name="create-the-model"></a>建立模型

1. 在 Web UI 中，按一下 [新增模型]
2. 在 [名稱] 中，輸入 AlternativeInputs。 然後按一下 [建立]。

### <a name="create-an-entity"></a>建立實體

1. 按一下 [實體]，然後按一下 [新增實體]。
2. 在 [實體名稱] 中，輸入城市。
3. 按一下 [建立]。

### <a name="create-three-actions"></a>建立三個動作

1. 按一下 [動作]，然後按一下 [新增動作]
2. 在 [回應] 中，鍵入「Which city do you want?」(您想要哪座城市？)。
3. 在 [不合格的實體] 中，輸入 $city。
3. Click Create

然後建立第二個動作：

1. 按一下 [動作]，然後按一下 [新增動作]。
3. 在 [回應] 中，鍵入「The weather in $city is probably sunny」($city 的天氣可能是晴天)。
4. 必要的實體，輸入 $city。
4. 按一下 [建立]。

建立第三個動作：

1. 按一下 [動作]，然後按一下 [新增動作]。
3. 在 [回應] 中，鍵入「Try asking for the weather」(嘗試詢問天氣)。
    - 這會回應「系統能夠怎麼做？」之類的使用者問題
4. 在 [不合格的實體] 中，輸入 $city。
4. Click Create

您現在有三個動作。

### <a name="train-the-bot"></a>訓練聊天機器人

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「what's the weather」(天氣如何)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [Which city do you want?]\(您想要哪座城市？\)。
2. 輸入「denver」(丹佛)。
3. 按兩下 [丹佛]，並選取城市。
    - 這會將它標示為城市實體。
5. 按一下 [Score Actions]\(評分動作\)
    - 「丹佛」現在會出現在城市實體中。 
6. 選取 [$city is probably sunny]\($city 的天氣可能是晴天\)。
7. 按一下 [Done Teaching]\(完成教學\)。

加入另一個範例對話：

1. 按一下 [新增動作]，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「what can you do?」(您該怎麼辦？)。
3. 按一下 [Score Actions]\(評分動作\)，並選取「Try asking for the weather」\(嘗試詢問天氣\)
2. 輸入「What's the weather in seattle」(西雅圖的天氣如何)。
3. 按兩下 [Seattle]\(西雅圖\)，並選取 [city]\(城市\)。
    - 這會將它標示為城市實體。
5. 按一下 [Score Actions]\(評分動作\)
    - 「西雅圖」現在會出現在城市實體中。 
6. 選取 [$city is probably sunny]\($city 的天氣可能是晴天\)。
7. 按一下 [Done Teaching]\(完成教學\)。

讓我們看看如果使用者說出類似上述的語句會發生什麼事：

1. 按一下 [新增動作]，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「help」(協助)。
3. 按一下 [Score Actions]\(評分動作\)。
    - 這兩個可能回應的分數非常接近。 這表示模型不清楚兩個動作之間的界限。
6. 按一下 [Abandon Teaching]\(放棄教學\) 和 [確認]。

![](../media/tutorial8_closescores.png)

在此情況下，這會協助將替代輸入加入到對話方塊。 您可以在進行教學時將之加入。 您也可以稍後返回並加入這些。

2. 按一下 [Train Dialogs]\(訓練對話\) 中的 [What can you do?]\(您該怎麼辦？\) 。
2. 在對話方塊中，按一下 [What can you do?]\(您該怎麼辦？\) 選取它。
    1. 在右方窗格的 [Entity Detection]\(實體偵測\) 之下，於 [Add alternative input]\(新增替代輸入\) 中，輸入一些替代選項：
    1. 輸入「what are my choices?」(我有哪些選擇？)
    2. 輸入「Tell me my choices」(讓我知道我的選擇)。
    3. 輸入「help」(協助)
    1. 按一下 [送出變更]。


![](../media/tutorial8_helpalternates.png)

2. 現在按一下 [西雅圖的天氣如何]。
    1. 在 [Add alternative input]\(新增替代輸入\) 中，輸入「forecast for seattle」(西雅圖的天氣預報)。
    2. 按兩下 [Seattle]\(西雅圖\)，並選取 [city]\(城市\)。 適用於替代輸入的實體應該會出現，而且具有相同的實體集合。 如果實體的內容不同，也不會有問題。
    3. 在 [Add alternative input]\(新增替代輸入\) 中，輸入「'will it rain today in denver」(丹佛今天 會下雨嗎)。
    4. 按一下 [denver]\(丹佛\)，並選取 [city]\(城市\)。
    5. 按一下 [送出變更] 和 [完成]。


讓我們將替代輸入加入到第一個對話方塊：

1. 按一下 [Train Dialogs]\(訓練對話\)。
2. 按一下開頭為「what's the weather」(天氣如何) 的對話。
2. 按一下以選取左窗格中的 [what's the weather]\(天氣如何\)：
    1. 在 [Add alternative input]\(新增替代輸入\) 中，輸入「weather forecast」(天氣預報)。
    2. 輸入「will it rain?」(會下雨嗎？)
    3. 按一下 [送出變更]。
4. 按一下以選取左窗格中的 [denver]\(丹佛\)：
    1. 在 [Add alternative input]\(新增替代輸入\) 中，輸入「for denver」(丹佛的)。
    2. 輸入「forecast for austin」(奧斯汀的天氣預報)。
        - 整個片語都會反白顯示。 按一下整個詞，然後按一下紅色 x。 選取 [austin]\(奧斯汀\)，然後按一下 [city]\(城市\)。
        - 按一下 [送出變更]
    1. 按一下 [完成]，使模型重新訓練。

![](../media/tutorial8_altcities.png)

讓我們試試不同的變化：

1. 按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「what are you capabilities」(您有什麼才能)。
3. 按一下 [Score Actions]\(評分動作\)。
    - 分數現在會在下一個動作中更明確地指出模型的確定性。
2. 選取「Try asking for weather」(嘗試詢問天氣)。
6. 按一下 [Done Teaching]\(完成教學\)

現在您已經看到替代輸入如何用來指示使用者可能會說的其他語句。 這些有助於您將許多對話納入於一段對話中，並列舉使用者可能說的對話，以避免建立在許多方面其實是相同的許多對話。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [記錄對話](./9-log-dialogs.md)
