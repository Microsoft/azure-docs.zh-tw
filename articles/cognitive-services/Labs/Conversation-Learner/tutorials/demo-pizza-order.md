---
title: 示範對話學習模組模型：披薩訂單 - Microsoft 認知服務 | Microsoft Docs
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
ms.openlocfilehash: 052ef249f3367a562e5598b90533c0e52ed75df4
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171379"
---
# <a name="demo-pizza-order"></a>示範：披薩訂單
這段示範說明披薩訂購聊天機器人。 這支援藉由下列功能訂購一份披薩：

- 辨識使用者語句中的披薩配料
- 檢查披薩配料是否有庫存或無庫存，並進行適當的回應
- 記住前一份訂單的披薩配料，並使用相同的配料開始新的訂單

## <a name="video"></a>影片

[![示範披薩預覽](http://aka.ms/cl-demo-pizza-preview)](http://aka.ms/blis-demo-pizza)

## <a name="requirements"></a>需求
本教學課程需要執行披薩訂單聊天機器人

    npm run demo-pizza

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [TutorialDemo Pizza Order]。 

## <a name="entities"></a>實體

您已經建立三個實體。

- Toppings：這個實體將會累積使用者要求的配料。 其中包含有庫存的有效配料。 這會檢查配料是否有庫存或無庫存。
- OutofStock：這個實體會用來回覆使用者選取的配料沒有庫存。
- LastToppings：下達訂單之後，此實體會用來在訂單上向使用者提供配料清單。

![](../media/tutorial_pizza_entities.PNG)

### <a name="actions"></a>動作

您已經建立一組動作，其中包括詢問使用者想要在披薩中加入哪些配料、告訴他們目前已加入的配料等等。

另外還有兩個 API 呼叫：

- FinalizeOrder： 下達披薩訂單
- UseLastToppings：從前一份訂單移轉配料 

![](../media/tutorial_pizza_actions.PNG)

### <a name="training-dialogs"></a>訓練對話
您已經定義少數定型對話。 

![](../media/tutorial_pizza_dialogs.PNG)

讓我們試試以教學工作階段為例。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「order a pizza」(訂購披薩)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [what would you like on your pizza?]\(您要在披薩上加什麼？\)
4. 輸入「mushrooms and cheese」(蘑菇和起士)。
    - 請注意 LUIS 已將兩者標示為配料。 如果不正確，您可以按一下以反白顯示，然後進行修正。
    - 實體旁的「+」號表示已加入到配料集合。
5. 按一下 [Score Actions]\(評分動作\)。
    - 請注意，`mushrooms` 和 `cheese` 不在配料的記憶體中。
3. 按一下以選取 [you have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)
    - 請注意這是非等候動作，因此聊天機器人將要求下一個動作。
6. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
7. 輸入「remove mushrooms and add peppers」(不要蘑菇而且要加黑胡椒)。
    - 請注意，`mushroom` 旁邊有 '-' 號，因為將移除該配料。 此外，`peppers` 具有 '+' 號，可將它加入配料。
2. 按一下 [Score Actions]\(評分動作\)。
    - 請注意，`peppers` 現在是粗體，因為這是新配料。 此外，已劃掉 `mushrooms`。
8. 按一下以選取 [you have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)
6. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
7. 輸入「加入豌豆」。
    - `Peas` 是沒有庫存的配料範例。 但仍會將它標示為配料。
2. 按一下 [Score Actions]\(評分動作\)。
    - `Peas` 會顯示為 OutOfStock。
    - 若要查看這是如何發生的，請開啟位於 `C:\<\installedpath>\src\demos\demoPizzaOrder.ts` 的程式碼。 查看 EntityDetectionCallback 方法。 對於每個配料都會呼叫這個方法來查看否有庫存。 如果沒有，則會從配料集合中清除，並新增至 OutOfStock 實體。 inStock 變數是在該方法之上定義，其中有一份庫存配料清單。
6. 選取 [We don't have $OutOfStock]\(我們沒有 $OutOfStock\)。
7. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
8. 輸入「no」(否)。
9. 按一下 [Score Actions]\(評分動作\)。
10. 選取 [FinalizeOrder] API 呼叫。 
    - 這會呼叫程式碼中定義的「FinalizeOrder」函式。 這會清除配料，並且回覆「your order is on its way」(您的訂單正在處理中)。 
2. 輸入「order another」(訂購另一份)。 我們會開始新的訂單。
9. 按一下 [Score Actions]\(評分動作\)。
    - 'cheese' 和 'peppers' 均位於記憶體中，因為這些是前一份訂單的配料。
1. 選取 [Would you like $LastToppings]\(您是否要 $LastToppings\)。
2. 輸入「yes」(是)。
3. 按一下 [Score Actions]\(評分動作\)。
    - 聊天機器人想要進行 UseLastToppings 動作。 這是兩個回呼方法的第二個。 這會將最後一份訂單的配料複製到配料，並清除最後的配料。 這會記住最後一份訂單，而且，如果使用者說還要另一份披薩，會提供這些配料做為選項。
2. 按一下以選取 [you have $Toppings on your pizza]\(您的披薩上有 '$Toppings\)。
3. 選取 [Would you like anything else?]\(您還需要其他任何服務嗎？\)
8. 輸入「no」(否)。
4. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial_pizza_callbackcode.PNG)

![](../media/tutorial_pizza_apicalls.PNG)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [示範 - VR 應用程式啟動器](./demo-vr-app-launcher.md)
