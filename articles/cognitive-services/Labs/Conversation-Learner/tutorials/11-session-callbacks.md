---
title: 如何搭配對話學習模組模型使用工作階段回呼 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用工作階段回呼。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 0f51b232470e4e4da3f25d40d025dd3b09dd1204
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171910"
---
# <a name="how-to-use-session-callbacks-with-a-conversation-learner-model"></a>如何搭配對話學習模組模型使用工作階段回呼

本教學課程說明 onSessionStart 和 onSessionEnd 回呼。

## <a name="video"></a>影片

[![教學課程 11 預覽](http://aka.ms/cl-tutorial-11-preview)](http://aka.ms/blis-tutorial-11)

## <a name="requirements"></a>需求
本教學課程要求 `tutorialSessionCallbacks` 聊天機器人必須為執行中狀態。

    npm run tutorial-session-callbacks

## <a name="details"></a>詳細資料
本教學課程涵蓋工作階段的概念、如何預設處理工作階段，以及如何覆寫該行為。

工作階段是與聊天機器人進行的一次對話。 其中可以有多次來回的交談，但是對話之間沒有長時間的空白 (像是 30 分鐘)。  關於預設工作階段逾時持續期間，請參閱說明頁的「限制」。

如果中斷相當長的時間，聊天機器人會進行下一個工作階段。  啟動新的工作階段會使遞迴式類神經網路恢復其初始狀態。  根據預設，這也會清除所有的實體值，不過可以變更此行為 (如下所述)。

### <a name="open-the-demo"></a>開啟示範

在模型清單中，按一下 [Tutorial-11-SessionCallbacks]。 

### <a name="entities"></a>實體

在模型中定義四個實體。

![](../media/tutorial11_entities.PNG)

必須注意 BotName 是程式設計實體。  這個實體將會由聊天機器人在工作階段開始時設定。

### <a name="actions"></a>動作

在模型中定義四個動作。

![](../media/tutorial11_actions.PNG)

首先，本教學課程將示範如何在工作階段開始時控制實體值，例如，在使用者說任何話之前設定 BotName 實體。

其次，本教學課程將示範如何將一個工作階段的值沿用到下一個工作階段。  在本教學課程中，我們假設使用者名稱和電話號碼從一個工作階段到下一個工作階段維持相同，但是其位置可能會變更。  因此，我們會跨工作階段沿用名稱和電話號碼，但是清除使用者位置。

### <a name="train-dialog"></a>訓練對話

下列是範例對話。 這是一個工作階段，也就是此次對話沒有任何長時間中斷。

![](../media/tutorial11_traindialog.PNG)

### <a name="code-for-the-callbacks"></a>回呼的程式碼

回呼方法的程式碼位於下列檔案： c:\<installedpath>\src\demos\tutorialSessionCallbacks.ts。

![](../media/tutorial11_code.PNG)

這兩種方法都是選用方法。

- OnSessionStartCallback：這個方法會設定 BotName 實體。
- OnSessionEndCallback：您可以指定要保留的實體。 這將清除使用者名稱和使用者電話以外的所有實體。

### <a name="try-the-bot"></a>測試聊天機器人

切換至 Web UI，並按一下 [記錄對話]。

1. 輸入「hello」(哈囉)。
2. 系統：「Hi, I'm Botty. What's your name?」(您好，我是 Botty。您叫什麼名字？) 其名稱 Botty 來自 OnSessionStartCallback。
3. 輸入「jason」。
4. 系統：「Hi jason. What's your phone number?」(jason，您好。您的電話號碼是多少？)
5. 輸入「555-555-5555」。
6. 系統：「Can you tell Botty your location, jason?」(jason，您能否告訴 Botty 您的位置？)
7. 鍵入「Redmond」(雷德蒙德)。

這是一個工作階段。 若要啟動新的工作階段，我們需要結束這個工作階段。 

1. 按一下 [工作階段逾時]。 這會將您移至下一個工作階段。
    - 提供的 [工作階段逾時] 用於偵錯用途。  在實際的工作階段中，需要大約 30 分鐘的長時間暫停。  關於預設工作階段逾時持續期間，請參閱說明頁的「限制」。
1. 輸入「hi」\(嗨\)。
2. 系統：「Can you tell Botty your location, jason?」(jason，您能否告訴 Botty 您的位置？)
    - 系統已記住名稱和電話號碼。
2. 輸入新位置：「Seattle」(西雅圖)。
3. 系統：「So, jason you are in Seattle」(所以，jason，您位於西雅圖)。
4. 按一下 [完成測試]。

讓我們切換回 [記錄對話]。 請注意，最後一個對話已分割成兩個，因為每個記錄對話均對應於一個工作階段。  

![](../media/tutorial11_splitdialogs.PNG)

- 在第一次互動中，Botty 已設定完成，但是名稱和電話號碼未設定完成。
- 第二次互動顯示名稱和電話號碼。

您現在已經知道工作階段預設的處理方式，以及如何覆寫預設行為。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [API 呼叫](./12-api-calls.md)
