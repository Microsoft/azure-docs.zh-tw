---
title: 如何使用對話學習模組動作的「預期實體」屬性 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何使用對話學習模組模型的「預期實體」屬性。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: fb9193066c9501341efb779b9f9e2ccace02cccf
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42888199"
---
# <a name="how-to-use-the-expected-entity-property-of-actions"></a>如何使用動作的「預期實體」屬性

本教學課程示範動作的 [Expected entity]\(預期實體\) 欄位。

## <a name="video"></a>影片

[![教學課程 4 預覽](http://aka.ms/cl-tutorial-04-preview)](http://aka.ms/blis-tutorial-04)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料
使用動作的 [Expected entity]\(預期實體\) 欄位，以和您預計使用者對動作的回應是宣告實體的系統進行通訊。

具體而言，如果動作的 [Expected entity]\(預期實體\) 欄位是設定為 $entity，則在下一個使用者語句上，系統會：

1. 首先，和平常一樣，嘗試使用機器學習架構的實體擷取模型尋找實體
2. 如果並未在步驟 1 中找到實體，則會透過啟發學習法，將整個使用者語句指派為 $entity。
3. 和平常一樣呼叫 `EntityDetectionCallback`，並繼續進行動作選取項目。

## <a name="steps"></a>步驟

### <a name="create-the-model"></a>建立模型

1. 在 Web UI 中，按一下 [新增模型]
2. 在 [名稱] 中，輸入 ExpectedEntities。 然後按一下 [建立]。

### <a name="create-an-entity"></a>建立實體

1. 按一下 [實體]，然後按一下 [新增實體]。
2. 在 [實體名稱] 中，輸入名稱。
3. 按一下 [建立專案]。

> [!NOTE]
> 實體類型為「自訂」。 這個值表示實體可以定型。  另外有一些預先建置的實體，代表其行為無法調整。  這些實體會在[預先建置的實體教學課程](./7-built-in-entities.md)中加以說明。

![](../media/tutorial4_entities.PNG)

### <a name="create-two-actions"></a>建立兩個動作

1. 按一下 [動作]，然後按一下 [新增動作]。
2. 在 [回應] 中，鍵入「What's your name?」(您叫什麼名字？)。
3. 在 [Expected entity]\(預期實體\) 中，輸入 $name。 按一下 [儲存]。
    - 這些值表示，提出此問題時，如果使用者回應未偵測到任何實體，則聊天機器人應該會假設使用者的整個回應就是這個實體。
2. 按一下 [動作]，然後按一下 [新增動作] 以建立第二個動作。
3. 在 [回應] 中，鍵入「Hello $name」($name，您好)。
    - 實體會自動新增為所需的實體。 
4. 按一下 [儲存]。

現在您有兩個動作。

![](../media/tutorial4_actions.PNG)

### <a name="train-the-bot"></a>將 Bot 定型

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 鍵入「hello」(您好)。
3. 按一下 [Score Actions]\(評分動作\)，並選取 [What's your name?]\(您叫什麼名字？\)。
    - 無法選取回應 'Hello $name' ($name，您好)，因為它需要定義實體 $name，而且 $name 不在聊天機器人的記憶體中。
2. 鍵入「david」。 
    - 名稱會醒目提示為實體。 這是由於我們先前設定啟發學習法選取回應做為實體。
5. 按一下 [Score Actions]\(評分動作\)
    - 名稱值現在會在聊天機器人記憶體中。
    - 「$name，您好」現在可做為回應。 
6. 選取 [Hello $name]\($name，您好\)。
7. 按一下 [Done Teaching]\(完成教學\)。

下列是機器學習實體擷取模型識別名稱的兩個範例，因此未觸發「預期實體」啟發學習法。

1. 按一下 [New Train Dialog]\(新增訓練對話\)。
2. 輸入「my name is david」(我的名字叫 david)。
    - 這個模型會將 david 識別為名稱實體，因為這個字組之前曾經出現。
2. 按一下 [Score Actions]\(評分動作\)
3. 選取 [Hello $name]\($name，您好\)。
4. 輸入「my name is susan」(我的名字叫 susan)。
    - 這個模型會將 susan 識別為名稱，因為這個模式已經出現過。
2. 按一下 [Score Actions]\(評分動作\)。
2. 選取 [Hello susan]\(susan，您好\)。
3. 按一下 [Done Teaching]\(完成教學\)。

在下列範例中，「預期實體」啟發學習法會觸發，但不正確。 範例接著會示範如何進行更正。

1. 鍵入「call me jose」(叫我 jose)。
    - 這個模型不會將名稱辨識為實體。
2. 按一下 [jose]，並選取姓名。
3. 按一下 [Score Actions]\(評分動作\)。
4. 選取 [hello $name]\($name，您好\)。
5. 按一下 [Done Teaching]\(完成教學\)。
1. 按一下 [New Train Dialog]\(新增訓練對話\)。
2. 輸入「hello」(哈囉)。
3. 在對於「what's your name」(您叫什麼名字) 的回應中，輸入「I am called frank」(我叫 frank)。
    - 整個片語都會反白顯示。 這是因為統計模型找不到姓名，因此引引發啟發學習法，並且選取整個答案做為姓名實體。
2. 若要修正它，請按一下反白顯示的詞，然後按一下紅色 x。 
3. 按一下以選取 frank，然後按一下姓名。
2. 按一下 [Score Actions]\(評分動作\)
3. 選取 [Hello $name]\($name，您好\)。
4. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial4_dialogs.PNG)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Negatable 實體](./5-negatable-entities.md)
