---
title: 如何搭配對話學習模組模型使用卡片，第 2 部分 - Microsoft 認知服務 | Microsoft Docs
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
ms.openlocfilehash: 1c7c88742c69041594006add76f7e3c642c64dec
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39170567"
---
# <a name="how-to-use-cards-part-1-of-2"></a>如何使用卡片 (第 1 部份，共 2 個部份)
本教學課程顯示如何將可填寫的表單卡片新增至您的聊天機器人。 這將顯示表單欄位如何移至實體中。

對話學習模組預設您的卡定義檔案位於稱為 cards 的目錄，也就是聊天機器人啟動位置的目錄。

## <a name="video"></a>影片

[![教學課程 14 預覽](http://aka.ms/cl-tutorial-14-preview)](http://aka.ms/blis-tutorial-14)

## <a name="requirements"></a>需求
本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料

卡是讓使用者可在對話中選取選項的 UI 項目。 

### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [Tutorial-14-Cards-2]。 

### <a name="the-card"></a>卡片

卡片的定義位於下列位置：C:\<installedpath\>\src\cards\shippingAddress.json。

此卡片會收集交貨地址的 3 個欄位：縣/市、街道，以及州/省。

![](../media/tutorial14_card.PNG)

### <a name="actions"></a>動作

我們已經建立 3 個動作。 如下所示，第一個動作是卡片。

![](../media/tutorial14_actions.PNG)

讓我們看看卡片動作類型的建立方式：

- 請注意 [地址] 的 [街道] \(屬於 Input.text 類型\) 及其識別碼。
- 同樣地，也有 [地址] 的 [縣/市] 和將 [地址] 的 [州/省] 識別碼列出的下拉式清單。

識別碼是重要，因為填入並送出欄位時，這些識別碼是聊天機器人收到這些值的實體名稱。

## <a name="entities"></a>實體
我們已定義與與上述卡片相符的 3 個實體。

![](../media/tutorial14_entities.PNG)

## <a name="actions"></a>動作

我們已定義 2 個動作。

![](../media/tutorial14_actions.PNG)

- 第一個是交貨地址卡片，其中的動作類型是 CARD，而且已經從下拉式清單選取範本作為交貨地址。
- 第二個是讀回交貨地址的簡單動作。

![](../media/tutorial14_sa_card.PNG)

### <a name="train-dialog"></a>訓練對話

讓我們逐步進行教學對話。

1. 按一下 [Train Dialogs] \(訓練對話\)，然後按一下 [New Train Dialog] \(新增訓練對話\)。
1. 輸入「hi」\(嗨\)。
2. 按一下 [Score Actions] \(評分動作\)。
3. 按一下以選取 [Shipping Address] \(交貨地址\)。
4. 填寫卡片並送出。
    - 請注意，這些值現在已移至實體記憶體。 由於表單已分割輸入，因此不需要剖析。
5. 按一下 [Score Actions] \(評分動作\)。
3. 按一下以選取 [Shipping to $Address] \(交貨至 $Address\)。
4. 按一下 [完成測試]。

![](../media/tutorial14_train_dialog.PNG)

您現在已了解如何從包含可填寫的欄位和下拉式清單出的卡取得值，以及擷取和收集聊天機器人實體中的這些值。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [分支和復原](./15-branching-and-undo.md)
