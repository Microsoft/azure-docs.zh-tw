---
title: 如何搭配對話學習模組模型使用卡片，第 1 部分 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用卡片。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 988a2433f098f41bca4796299825293efd4de44b
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171129"
---
# <a name="how-to-use-cards-part-1-of-2"></a>如何使用卡片 (第 1 部分，共 2 個部分)

本教學課程顯示如何在聊天機器人中加入並使用簡單的卡片。

> [!NOTE]
> 目前，對話學習模組預期您的卡片定義檔案位於稱為 "cards" 的目錄，也就是啟動聊天機器人的目錄。 我們將在未來提供設定此目錄的功能。

## <a name="video"></a>影片

[![教學課程 13 預覽](http://aka.ms/cl-tutorial-13-preview)](http://aka.ms/blis-tutorial-13)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料

卡片是讓使用者可在對話中選取選項的 UI 項目。 

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [Tutorial-13-Cards-1]。 

### <a name="the-card"></a>卡片

卡片的定義位於下列位置：C:\<installedpath\>\src\cards\prompt.json。

系統應該會在卡目錄中找出卡片定義。

![](../media/tutorial13_prompt.PNG)

> [!NOTE]
> 請注意文字欄位中的內文類型 `TextBlock` 和 `{{question}}` 預留位置。
> 有兩個提交按鈕，以及對於每個按鈕所提交的文字。

### <a name="actions"></a>動作

我們已經建立 3 個動作。 如下所示，第一個動作是卡片。

![](../media/tutorial13_actions.PNG)

讓我們看看卡片動作類型的建立方式：

![](../media/tutorial13_cardaction.PNG)

> [!NOTE]
> 問題輸入，以及按鈕 1 和 2。 這些是卡片中的範本參考，您會在其中輸入問題和個別答案。 您也可以參考並使用實體，或參考並使用文字和實體的混合。

眼睛圖示可顯示卡呈現的樣貌。

### <a name="train-dialog"></a>訓練對話

讓我們逐步進行教學對話。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「hi」\(嗨\)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [Prompt go left or right]\(提示向左或向右\)。
    - 按一下 [左] 或 [右] 等同於使用者分別輸入「左」或「右」。 
4. 按一下 [Score Actions]\(評分動作\)。
4. 按一下以選取 [左]。 這是非等候動作。
6. 按一下以選取 [Prompt go left or right]\(提示向左或向右\)。
4. 按一下 [右]。
5. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [右]。
6. 按一下以選取 [Prompt go left or right]\(提示向左或向右\)。
4. 按一下 [完成測試]。

您現在已了解卡的運作方式。 這些是在卡目錄中界定為 json 範本。 範本會出現在 UI 中，您可以使用字串或實體或使用兩者的混合填入。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [卡片第 2 部分](./14-cards-2.md)
