---
title: 如何搭配對話學習模組模型使用 API 呼叫 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用 API 呼叫。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 1d4013d736d8cfcb75874bc0c86d20b86ab4dd62
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39215834"
---
# <a name="how-to-add-api-calls-to-a-conversation-learner-model"></a>如何將 API 呼叫新增到對話學習模組模型

本教學課程示範如何將 API 呼叫新增到您的模型。 API 呼叫是您在聊天機器人中定義並寫入的函式，而且對話學習模組可以呼叫。

## <a name="video"></a>影片

[![教學課程 12 預覽](http://aka.ms/cl-tutorial-12-preview)](http://aka.ms/blis-tutorial-12)

## <a name="requirements"></a>需求
本教學課程需要「tutorialAPICalls.ts」聊天機器人正在執行。

    npm run tutorial-api-calls

## <a name="details"></a>詳細資料

- API 呼叫可以讀取和操作實體。
- API 呼叫可以存取記憶體管理員物件。
- API 呼叫也可以接受引數，這可讓您重複使用相同的 API 呼叫作為不同的用途。

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [Tutorial-12-APICalls]。 

### <a name="entities"></a>實體

我們已在模型呼叫的數字中定義一個實體。

![](../media/tutorial12_entities.PNG)

### <a name="api-calls"></a>API 呼叫
API 呼叫的程式碼是在這個檔案中定義的：C:\<installedpath\>\src\demos\tutorialAPICalls.ts。

![](../media/tutorial12_apicalls.PNG)

- 第一個 API 回呼是 RandomGreeting。 這會傳回問候語變數中定義的隨機問候語。
- Multiply API 回呼：這會將使用者提供的兩個數字相乘。 接著會傳回兩個數字相乘的結果。 這表示 API 呼叫可以接受輸入。 請注意，記憶體管理員是第一個引數。 
- ClearEntities API 回呼：清除數字實體，讓使用者輸入下一個數字。 這說明 API 呼叫可以如何操作實體。

### <a name="actions"></a>動作

我們已經建立 4 個動作。 

![](../media/tutorial12_actions.PNG)

- 除了「您想要哪個數字乘以 12？」 傳達動作之外，另外有 3 個不同的 API 呼叫說明典型的 API 呼叫模式。

- RandomGreeting：為非等候動作。 為了設定此動作，在 [建立動作] 對話方塊中，我們已選取 API_LOCAL 的動作類型，然後選取 RandomGreeting。 

![](../media/tutorial12_setupapicall.PNG)

API 旁邊的重新整理按鈕可用來停止聊天機器人，並且對於 API 進行任何變更。 按一下 [重新整理] 會取得最新的變更。

以下是我們建立相乘動作的方式：選取 API_Local 和 API 後，對於第一個輸入值 (num1string) 輸入實體 ($number)，並對於第二個輸入值 (num2string) 的輸入值 (12)。 這讓聊天機器人與 API 呼叫之間能一定程度上間接取值，以便相同的回呼可以對應系統中的一些動作，並且其指派動作的方式有所不同。

![](../media/tutorial12_actionmultiply.PNG)

### <a name="train-dialog"></a>訓練對話

讓我們逐步進行教學對話。

1. 按一下 [Train Dialogs] \(訓練對話\)，然後按一下 [New Train Dialog] \(新增訓練對話\)。
1. 輸入「hi」\(嗨\)。
2. 按一下 [Score Actions] \(評分動作\)。
3. 按一下以選取 RandomGreeting。 這將執行隨機問候語 API 呼叫。
3. 按一下以選取 [What number to do you want to multiply by 12?] \(您想要哪個數字乘以 12?\)
4. 輸入 '8'。 然後按一下 [Score Actions] \(評分動作\)。
4. 選取 [Multiply $number 12] \($number 和 12 相乘\)。 請注意相乘的結果。
5. 選取 [Clear Entities] \(清除實體\)。
    - `number` 實體的值已被清除。
3. 按一下以選取 [What number to do you want to multiply by 12?] \(您想要哪個數字乘以 12？\)
4. 按一下 [完成測試]。

![](../media/tutorial12_dialog.PNG)

您現在已了解如何註冊 API 回呼、其常見模式，以及如何定義引數，並且使值與其中的實體產生關聯。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [卡片第 1 部分](./13-cards-1.md)
