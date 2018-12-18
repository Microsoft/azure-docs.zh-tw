---
title: 已備妥資料來進行資料科學嗎？ 資料評估 - Azure Machine Learning | Microsoft Docs
description: 您資料備妥可進行資料科學前，必須符合的四個準則。 這段影片提供具體範例，可協助進行基本的資料評估。
keywords: 相關的資料, 評估資料, 準備資料, 資料準則, 備妥資料
services: machine-learning
documentationcenter: na
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cjgronlund
ms.assetid: d502062c-da70-4b21-9054-0bfd9902612e
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2018
ms.openlocfilehash: 5ab3c7716485053432240cb74be8ebc60c9ad274
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834612"
---
# <a name="is-your-data-ready-for-data-science"></a>已備妥資料來進行資料科學嗎？
## <a name="video-2-data-science-for-beginners-series"></a>影片 2：適用於初學者的資料科學系列
了解如何評估您的資料，以確定它符合進行資料科學的基本準則。

若要充分運用這系列影片，請觀賞所有影片。 [瀏覽影片清單](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Shows/SupervisionNotRequired/9/player]
>
>

## <a name="other-videos-in-this-series"></a>系列中的其他影片
 是一個資料科學的快速簡介，包含五個簡短影片。

* 影片 1：[資料科學可以回答的 5 個問題](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 分 14 秒)*
* 影片 2：您的資料已經可以進行資料科學了嗎？
* 影片 3：[詢問您可以使用資料回答的問題](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 分 17 秒)*
* 影片 4：[利用簡單模型預測答案](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 分 42 秒)*
* 影片 5：[複製其他人的工作進行資料科學](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 分 18 秒)*

## <a name="transcript-is-your-data-ready-for-data-science"></a>文字記錄：已備妥資料來進行資料科學嗎？
歡迎觀賞「已備妥資料來進行資料科學嗎？」 這是「適用於初學者的資料科學」系列的第二個影片。  

您必須先提供一些可運用的高品質原始資料，資料科學才能提供您想要的答案。 就像製作比薩，您一開始使用的原料越好，最終產品就越好。 

## <a name="criteria-for-data"></a>資料準則
在資料科學中，有某些要素必須同時提取，包括：

* 相關
* 連線
* 精確
* 足以使用

## <a name="is-your-data-relevant"></a>您的資料有相關性嗎？
因此第一個要素 - 您需要相關的資料。

![相關的資料與無關的資料 - 評估資料](./media/data-science-for-beginners-is-your-data-ready-for-data-science/relevant-and-irrelevant-data.png)

左側的表格中顯示了七個人在波士頓酒吧外測量的血液酒精濃度、紅襪隊在他們的最近一場比賽中的打擊率，以及最近一家便利商店的牛奶價格。

這一切都是合法的資料。 唯一的缺點是它們並不相關。 這些數字之間沒有任何明顯的關聯性。 如果某人提供了目前的牛奶價格和紅襪隊的打擊率，您絕對沒有辦法猜到其血液酒精濃度。

現在看看右邊表格。 這次測量了每個人的身高體重，並計算了他們喝了多少飲料。  現在每一列的數字都彼此相關。 如果我提供了我的身高體重以及我已喝了多少瑪格麗特，您就可以猜出我的血液酒精濃度。

## <a name="do-you-have-connected-data"></a>是否有連貫的資料？
下一個要素是連貫的資料。

![連接的資料與中斷連接的資料 - 資料準則、備妥資料](./media/data-science-for-beginners-is-your-data-ready-for-data-science/connected-vs-disconnected-data.png)

以下是一些關於漢堡品質的相關資料：燒烤溫度、肉餅重量，以及當地食品雜誌的評價。 但請注意左邊表格中的空格。

大部分的資料集遺漏了某些值。 通常都會有這類空格，而且有方法可以解決它們。 但是，如果遺漏太多，您的資料會開始看起來像是瑞士乾酪。

如果您查看左邊表格，有這麼多遺漏的資料，就很難在燒烤溫度和肉餅重量之間推論出任何種類的關聯性。 此範例顯示中斷的資料。

不過，右邊是完整且完全的表格 - 連貫的資料範例。

## <a name="is-your-data-accurate"></a>是您的資料精確嗎？
下一個要素是精確度。 以下是四個要達到的目標。

![準確的資料與不準確的資料 - 資料準則](./media/data-science-for-beginners-is-your-data-ready-for-data-science/inaccurate-vs-accurate-data.png)

看看右上方的目標。 正確地密集射中了靶心周圍。 當然這是精確的。 奇怪的是，在資料科學的語言中，目標正下方的表現也會被視為精確。

如果您繪製出這些箭頭的中心，會看到它們非常接近靶心。 箭頭全都分佈於目標四周，因此被視為不準確，但它們都集中在靶心周圍，因此可視為精確。

現在看一下左上方的目標。 此處，箭射中得非常接近，是一個緊密的群組。 它們是準確的，但並不精確，因為中心偏離了靶心。 左下方目標中的箭就是不準確且不精確。 這位弓箭手需要多加練習。

## <a name="do-you-have-enough-data-to-work-with"></a>您是否有足夠資料可供使用？
最後，4 號要素是足夠的資料。

![您是否有足夠資料可進行分析？ 資料評估](./media/data-science-for-beginners-is-your-data-ready-for-data-science/barely-enough-data.png)

將表格中的每一個資料點視為繪圖中的筆刷筆劃。 如果只有幾個筆刷筆劃，繪圖就會變得模糊 - 很難判斷它是什麼。

如果您加入更多筆刷筆劃，則您的繪圖就會開始變得更明確。

當您勉強有足夠的筆劃時，就只能看見足夠的資訊來進行一些廣泛的決策。 它是我想要造訪的地方嗎？ 它看起來很明亮，看起來像潔淨的水 – 沒錯，這就是我想要度假的地方。

當您加入更多資料時，圖片會變得越來越清晰，而您可以做出更詳細的決策。 現在您可以看見左岸的三家旅館。 您可以注意前景中那一家的建築特色。 您可能也會因為景觀而選擇入住三樓。

利用相關、連結、精確與足夠的資料，您擁有需要的所有要素，來進行一些高品質資料科學。

請務必查看 Microsoft Azure Machine Learning 中「適用於初學者的資料科學」的其他 4 部影片。

## <a name="next-steps"></a>後續步驟
* [嘗試使用 Machine Learning Studio 進行您的第一個資料科學實驗](create-experiment.md)
* [在 Microsoft Azure 上取得 Machine Learning 簡介](what-is-machine-learning.md)
