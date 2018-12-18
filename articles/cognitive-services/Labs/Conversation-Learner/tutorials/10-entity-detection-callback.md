---
title: 如何搭配對話學習模組模型使用實體偵測回呼 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用實體偵測回呼。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: f168018a23d03ffb957da2dd1f67881420a21208
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171098"
---
# <a name="how-to-use-entity-detection-callback"></a>如何使用實體偵測回呼

本教學課程會顯示實體偵測回呼，並呈現解析實體的常見模式。

## <a name="video"></a>影片

[![教學課程 10 預覽](http://aka.ms/cl-tutorial-10-preview)](http://aka.ms/blis-tutorial-10)

## <a name="requirements"></a>需求
本教學課程要求 `tutorialEntityDetectionCallback` 聊天機器人必須為執行中狀態。

    npm run tutorial-entity-detection

## <a name="details"></a>詳細資料
實體偵測回呼允許使用自訂程式碼處理與實體有關的商務規則。 此示範會使用回呼和程式設計的實體，將使用者輸入的名稱解析為正式名稱，例如，將「大蘋果」解析為「紐約」。

### <a name="open-the-demo"></a>開啟示範

在模型清單中，按一下 [Tutorial-10-EntityDetectionCallback]。 

### <a name="entities"></a>實體

模型中定義三個實體。

![](../media/tutorial10_entities.PNG)

1. City 是使用者將提供來作為文字輸入的自訂實體。
2. CityUnknown 是程式設計實體。 這個實體將由系統填入。 如果系統不知道是哪一個城市，系統會複製使用者輸入的文字。
3. CityResolved 是系統知道的城市。 這個實體將是城市的正式名稱，例如，「大蘋果」將解析為「紐約」。

### <a name="actions"></a>動作

在模型中定義三個動作。

![](../media/tutorial10_actions.PNG)

1. 第一個動作是「您想要哪座城市？」
2. 第二個是「我不知道 $CityUknown 這座城市。 您想要哪座城市？」
3. 第三個是「您說 $City，而我解析成 $CityResolved。」

### <a name="callback-code"></a>回呼代碼

讓我們看看代碼。 您可以在 C:\<installedpath>\src\demos\tutorialSessionCallbacks.ts 檔案中找到 EntityDetectionCallback 方法。

![](../media/tutorial10_callbackcode.PNG)

實體解析發生後，會呼叫此函式。
 
- 首先進行的動作是清除 $CityUknown。 $CityUknown 在一開始就會被清除，只會維持單一回合。
- 然後，取得已辨識的城市清單。 選擇並嘗試解析第一個城市。
- 解析它的函式 (resolveCity) 已在上述程式碼中進一步定義。 該函示有標準城市名稱的清單。 它會在清單中找到城市名稱並傳回。 若找不到，則會查看「cityMap」並傳回對應的名稱。 如果仍然找不到城市，會傳回 null。
- 最後，如果城市已解析為名稱，我們會將該名稱儲存於 $CityKnown 實體。 若未解析，則將清除使用者說的話並填入 $CityUknown 實體。

### <a name="train-dialogs"></a>訓練對話

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「hello」(您好)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [Which city do you want?]\(您想要哪座城市？\)。
2. 鍵入「new york」(紐約)。
    - 此文字會被識別為城市實體。
5. 按一下 [Score Actions]\(評分動作\)
    - 已填入 `City` 和 `CityResolved`。
6. 選取 [You said $City, and I resolved that to $CityResolved]\(您說 $City，而我解析成 $CityResolved\)。
7. 按一下 [Done Teaching]\(完成教學\)。

加入另一個範例對話：

1. 按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「hello」(您好)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [Which city do you want?]\(您想要哪座城市？\)。
2. 輸入「big apple」(大蘋果)。
    - 此文字會被識別為城市實體。
5. 按一下 [Score Actions]\(評分動作\)
    - `CityResolved` 會顯示程式碼執行的效果。
6. 選取 [You said $City, and I resolved that to $CityResolved]\(您說 $City，而我解析成 $CityResolved\)。
7. 按一下 [Done Teaching]\(完成教學\)。

加入另一個範例對話：

1. 按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「hello」(您好)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [Which city do you want?]\(您想要哪座城市？\)。
2. 輸入「foo」。
    - 這是系統不知道的城市的範例。 
5. 按一下 [Score Actions]\(評分動作\)
6. 選取 I don't know this city, $CityUknown. Which city do you want?\(我不知道 $CityUknown 這座城市。您想要哪座城市？\)。
7. 鍵入「new york」(紐約)。
8. 按一下 [Score Actions]\(評分動作\)。
    - `CityUknown` 已清除，並填入 `CityResolved`。
6. 選取 [You said $City, and I resolved that to $CityResolved]\(您說 $City，而我解析成 $CityResolved\)。
7. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial10_bigapple.PNG)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [工作階段回呼](./11-session-callbacks.md)
