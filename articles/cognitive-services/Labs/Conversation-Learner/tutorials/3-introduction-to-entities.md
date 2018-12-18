---
title: 如何搭配對話學習模組模型使用實體 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何搭配對話學習模組模型使用實體。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: f851d43d69999a848dea01c9457a379adb63353b
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39172376"
---
# <a name="introduction-to-entities"></a>實體簡介

本教學課程介紹實體，並說明如何在動作中使用 [不合格的實體] 和 [必要實體] 欄位。

## <a name="video"></a>影片

[![教學課程 3 預覽](http://aka.ms/cl-tutorial-03-preview)](http://aka.ms/blis-tutorial-03)

## <a name="requirements"></a>需求

本教學課程需要執行一般教學課程聊天機器人

    npm run tutorial-general

## <a name="details"></a>詳細資料

本教學課程將說明實體的兩個常見用法。  首先，實體可以從使用者訊息中擷取子字串，例如在「西雅圖的天氣如何？」中識別出城市。  其次，實體可以限制動作可供使用的時機。  具體來說，動作可以將實體列示為「必要」或「不合格」的實體：
- 動作的必要實體必須存在於 Bot 的記憶體中，動作才可供使用
- 不合格的實體必須*不*存在於 Bot 的記憶體中，動作才可供使用

另有其他教學課程會說明實體的其他層面，例如預先建置的實體、多重值和 Negatable 實體、程式設計實體，以及在程式碼中操作實體。

## <a name="steps"></a>步驟

### <a name="create-the-model"></a>建立模型

1. 在 Web UI 中，按一下 [新增模型]
2. 在 [名稱] 中，輸入 IntroToEntities。 接著，按一下 [建立]。

### <a name="create-entity"></a>建立實體

1. 按一下 [實體]，然後按一下 [新增實體]。
2. 在 [實體名稱] 中，輸入城市。
3. Click Create

> [!NOTE]
> 實體類型是「自訂」，這表示實體可以定型。  另外有一些預先建立的實體，代表其行為無法調整，會在另一篇教學課程內加以說明。

### <a name="create-two-actions"></a>建立兩個動作

1. 按一下 [動作]，然後按一下 [新增動作]
2. 在 [回應] 中，輸入「我不知道您要的城市」。
3. 在 [不合格的實體] 中，輸入 $city。 按一下 [儲存]。
    - 這表示，如果此實體定義於 Bot 的記憶體中，這個動作即*無法*使用。
2. 按一下 [動作]，然後按一下 [新增動作] 以建立第二個動作。
3. 在 [回應] 中，輸入「'$city 的天氣可能是晴天」。
4. 在 [必要實體] 中，已自動新增城市實體，因為會參考它。
5. 按一下 [Save] \(儲存)。

現在您有兩個動作。

![](../media/tutorial3_actions.PNG)

### <a name="train-the-bot"></a>將 Bot 定型

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
2. 輸入「您好」。
3. 按一下 [評分動作]，並選取 [我不知道您要選擇的城市?]
    - 您無法選取需要城市實體的回應，因為城市實體並未定義於聊天機器人的記憶體中。
2. 選取 [我不知道您要的城市]。
4. 輸入「西雅圖」。 將西雅圖醒目提示，然後按一下城市。
5. 按一下 [Score Actions]\(評分動作\)
    - 城市值現在會在聊天機器人的記憶體中。
    - 「'$city 的天氣可能是晴天」現在已是可用的回應。 
6. 選取 ['$city 的天氣可能是晴天]。

我們假設使用者輸入「加以重複」。 
1. 輸入這些文字。 城市實體及其值會在記憶體中且可供使用。
2. 選取 ['$city 的天氣可能是晴天]。

![](../media/tutorial3_entities.PNG)

您現在已建立實體，並在使用者訊息中標示其執行個體。  您也已使用實體在 Bot 記憶體中的存在性/不存在性，透過動作的不合格和必要實體欄位來控制動作可供使用的時機。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [預期的實體](./4-expected-entity.md)
