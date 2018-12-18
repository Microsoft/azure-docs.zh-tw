---
title: 如何搭配其他聊天機器人建置技術使用對話學習模組 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配其他聊天機器人建置技術使用對話學習模組。
services: cognitive-services
author: mattm
manager: larsliden
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 07/13/2018
ms.author: v-jaswel
ms.openlocfilehash: a03596ff8383a085314508d4a25d0ba89bcc3094
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39174477"
---
# <a name="how-to-use-conversation-learner-with-other-bot-building-technologies"></a>如何搭配其他聊天機器人建置技術使用對話學習模組

本教學課程涵蓋如何搭配其他聊天機器人建置技術使用對話學習模組，以及如何在這些技術之間共用記憶體 (或狀態)。 

## <a name="video"></a>影片

[![教學課程 15 預覽](http://aka.ms/cl-tutorial-15-preview)](http://aka.ms/blis-tutorial-15)

## <a name="requirements"></a>需求
本教學課程需要使用聊天機器人模擬器建立記錄對話，而非記錄對話 Web UI。  

本教學課程要求混合式教學課程聊天機器人必須是執行中狀態：

    npm run tutorial-hybrid

## <a name="details"></a>詳細資料

當對話學習模組擁有控制權時，相對於對話學習模組工作階段的所有狀態都必須儲存於對話學習模組的記憶體管理員中。 這是必要，因為機器學習會使用狀態來判斷對話的驅動方式。 外部狀態可以傳遞到要在工作階段開始時呼叫之 OnSessionStartCallback 中的對話學習模組。 當工作階段終止時，可以由 OnSessionEndCallback 傳回內部。

您幾乎可將對話學習模組視為函式呼叫，以取得一些初始狀態並傳回值。

在此範例中，您將使用兩個不同的系統來建立混合式聊天機器人：
1. 對話學習模組模型 <br />
使用對話學習模組模型，根據目前的工作階段來判斷聊天機器人的下一個動作。
聊天機器人的這個部分會取得初始狀態 `isOpen` (指出商店是否處於營業狀態) 的一部分 ，並傳回狀態 `purchaseItem` (使用者所購買項目的名稱) 的另一部分。

2. 文字比對 <br />
只查看特定字串和回應的內送文字。
聊天機器人的這個部分會管理聊天機器人的其他儲存機制，並負責啟動 CL 工作階段。 具體來說，它會管理三個變數：`usingConversationLearner`、`storeIsOpen` 和 `purchaseItem`。

讓我們來看看這個示範中所使用的模型。

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [Tutorial-15-Hybrid]。

## <a name="entities"></a>實體

開啟實體頁面，並注意下列兩個實體：`isOpen` 和 `purchaseItem`

若要了解如何使用這些實體，請開啟檔案：`C:\<installedpath\>\src\demos\tutorialHybrid.ts` 以查看回呼。

請注意，`OnSessionStartCallback` 中的程式碼會將來自 BotBuilder 對話儲存體的 `storeIsOpen` 值複製為 `isOpen` 實體的值，如此一來，它就可供對話學習模組使用。 查看下列程式碼：

![](../media/tutorial17_sessionstart.PNG)

同樣地，`OnSessionEndCallback` 中的程式碼 (如果工作階段已因為學習到的活動且不僅是逾時而結束) 會將實體 `purchaseItem` 的值複製到 BotBuilder 儲存體 `purchaseItem`。 查看下列程式碼：

![](../media/tutorial17_sessionend.PNG)

現在，讓我們查看動作。

## <a name="actions"></a>動作

請注意，此模型有四個動作。

動作的預期規則如下：

- 如果已設定 `isOpen` 實體，聊天機器人將會詢問 "What would you like to buy?" (您想要買什麼?)， 並將其儲存於 `puchaseItem` 位置
- 如果未設定 `isOpen`，聊天機器人就會說 "I’m sorry we’re closed" (很抱歉，本日營業時間結束。)
- 其他兩個動作都屬於 `END_SESSION` 類型。
- END_SESSION 動作會向 ConversationLearner 指出對話已完成。

### <a name="overall-bot-logic"></a>聊天機器人的完整邏輯

首先您會看到，如果已設定聊天機器人的狀態 `usingConversationLearner` 旗標，我們會將控制權交給對話學習模組。 如果未設定，就會將控制權交給其他項目。  在此範例中，我們會顯示簡單的文字比對，但這可能是任何其他的聊天機器人技術，包括 LUIS、QnA Maker，甚至是對話學習模組的另一個執行個體。

我們需要為使用者提供一種方式來讓商店開始營業和休息，因此，我們會利用 "open store" (開始營業) 和 "close store" (休息) 進行字串比較，並設定 "storeIsOpen" 旗標。

接下來，需要設法觸發將控制權交給對話學習模組模型。 當我們比對 "shop" 字串時，會執行下列作業：
- 在聊天機器人的記憶體中，設定 `usingConversationLearner` 旗標。
- 在對話學習模組模型上，呼叫 "StartSession" 方法。  這將會觸發 "onSessionStartCallback"，它會將 `isOpen` 實體值初始化

查看下面的內容：

![](../media/tutorial17_useConversationLearner.PNG)

我們也會進行與 "history" 的文字比對，它將會顯示最後一個購買項目。
最後，如果輸入了任何其他項目，我們就會顯示可用的使用者命令。

## <a name="train-dialog"></a>訓練對話

針對本教學課程，已經將模型預先定型。  我們將測試完整的聊天機器人，以查看實務上對開始和結束工作階段回呼的影響。

## <a name="testing-the-bot"></a>測試聊天機器人

不同於單一對話學習模組模型聊天機器人，您將無法在對話學習模組 UI 中測試此項目，因為它只會顯示由對話學習模組模型所處理的項目。

### <a name="install-the-bot-framework-emulator"></a>安裝 Bot Framework 模擬器

- 移至 [https://github.com/Microsoft/BotFramework-Emulator](https://github.com/Microsoft/BotFramework-Emulator)。
- 下載並安裝模擬器。

### <a name="configure-the-emulator"></a>設定模擬器

- 開啟模擬器，並確定 URL 的目標是您聊天機器人執行所在的相同連接埠。 例如：`http://localhost:3978/api/messages`

### <a name="test"></a>測試 

#### <a name="scenario-1-store-is-closed"></a>案例 1：商店休息中
1. 輸入 'shop' (商店)。 這會透過文字比對來處理，並將控制權交給對話學習模組模型。
2. 輸入「hello」(哈囉)。  因為未設定 `isOpen` 值，聊天機器人會說 "I’m sorry we’re closed" (很抱歉，本日營業時間結束)，然後結束工作階段。

#### <a name="scenario-2-store-is-open"></a>案例 2：商店開始營業
3. 輸入 'open store' (開店)。  這會將 `isOpen` 設為 True。
4. 輸入 'shop' (商店)。
5. 輸入「hello」(哈囉)。  因為已將 `isOpen` 值設為 True，聊天機器人將會說 "What would you like to buy?" (您想要買什麼？)
6. 輸入 'chair' (椅子)。 'chair' (椅子) 將儲存到 CL 記憶體作為 `purchaseItem` 實體。 系統會叫用結束工作階段回呼，將此值複製到對話存放區。
7. 輸入 'history' (歷程記錄)。  聊天機器人將會說 'You bought chair' (您購買了椅子)，因為這是您的最後一個 `purchaseItem`。

## <a name="conclusion"></a>結論

根據您已在上方了解到的資訊，您應該能夠將合對話學習模組與任何其他的聊天機器人技術組合在一起
