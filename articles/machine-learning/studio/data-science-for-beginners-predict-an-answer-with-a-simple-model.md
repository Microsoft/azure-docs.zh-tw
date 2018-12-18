---
title: 利用簡單的迴歸模型預測答案 - Azure Machine Learning | Microsoft Docs
description: 如何在「適用於初學者的資料科學」影片 4 中，建立簡單迴歸模型來預測價格。 包含線性迴歸以及目標資料。
keywords: 建立模型, 簡單模型, 價格預測, 簡單迴歸模型
services: machine-learning
documentationcenter: na
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cjgronlund
ms.assetid: a28f1fab-e2d8-4663-aa7d-ca3530c8b525
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2018
ms.openlocfilehash: ad1b8369358f7811a02d344fdc0306662413a404
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833820"
---
# <a name="predict-an-answer-with-a-simple-model"></a>利用簡單模型預測答案
## <a name="video-4-data-science-for-beginners-series"></a>影片 4：適用於初學者的資料科學系列
在「適用於初學者的資料科學」影片 4 中，了解如何建立簡單迴歸模型來預測鑽石價格。 我們將利用目標資料來繪製迴歸模型。

若要充分運用這系列影片，請觀賞所有影片。 [瀏覽影片清單](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/data-science-for-beginners-series-predict-an-answer-with-a-simple-model/player]
>
>

## <a name="other-videos-in-this-series"></a>系列中的其他影片
 是一個資料科學的快速簡介，包含五個簡短影片。

* 影片 1：[資料科學可以回答的 5 個問題](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 分 14 秒)*
* 影片 2： [您的資料已經可以進行資料科學了嗎？](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 分 56 秒)*
* 影片 3：[詢問您可以使用資料回答的問題](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 分 17 秒)*
* 影片 4：利用簡單模型預測答案
* 影片 5：[複製其他人的工作進行資料科學](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 分 18 秒)*

## <a name="transcript-predict-an-answer-with-a-simple-model"></a>文字記錄：利用簡單模型預測答案
歡迎觀賞「適用於初學者的資料科學」系列的第四個影片。 在這個影片中，我們將建置簡單模型並進行預測。

「模型」  是我們相關資料的簡化敘述。 我將告訴您我的意思。

## <a name="collect-relevant-accurate-connected-enough-data"></a>收集相關、精確、連貫、足夠的資料
假設我想要購買鑽石。 我有一只戒指，它屬於我的祖母，其中鑲嵌了一個 1.35 克拉的鑽石，而我想要了解它的價值。 我帶著筆記本和筆到珠寶店，並記下架上所有鑽石的價格，以及它們的重量有多少克拉數。 從第一顆鑽石開始 - 它是 1.01 克拉，美金 7,366 元。

現在我已順利完成，並針對店內的所有其他鑽石執行此動作。

![鑽石資料欄](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/diamond-data.png)

請注意，我們的清單包含兩欄。 每欄都有不同的屬性 (克拉數的重量和價格)，而且每一行都是代表單一顆鑽石的單一資料點。

我們實際上在此處建立了一個小型資料集 - 一個表格。 請注意，它符合我們對於品質的準則：

* 資料是 **相關** 的 - 重量明確地與價格相關
* 它是 **精確** 的 - 我們已再次檢查我們寫下的價格
* 它是 **連貫** 的 - 在這些欄中沒有任何空格
* 此外，誠如所見，有 **足夠** 的資料可回答我們的問題

## <a name="ask-a-sharp-question"></a>詢問明確的問題
現在我們將以明確的方式提出問題：「要花多少費用才能購買 1.35 克拉的鑽石？」

我們的清單中沒有 1.35 克拉的鑽石，因此，必須使用剩餘的資料來取得問題的答案。

## <a name="plot-the-existing-data"></a>繪製現有的資料
我們要做的第一件事是繪製一條水平數線 (稱為軸) 來繪製重量的圖表。 重量的範圍從 0 到 2，因此，我們將繪製一條涵蓋此範圍的線，並針對每半克拉劃上刻度。

接下來，我們將繪製一條垂直軸來記錄價格，並將它連接到水平的重量軸。 這將以美元為單位。 現在我們有一組座標軸。

![重量軸和價格軸](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/weight-and-price-axes.png)

我們現在將利用這個資料，並將它變成「散佈圖」 。 這是以視覺化方式檢視數值資料集的好方法。

針對第一個資料點，我們在 1.01 克拉處仔細打量出一條垂直線。 然後，在 7,366 美元處仔細打量出一條水平線。 接著在它們的交會處畫上一點。 這代表我們的第一顆鑽石。

現在逐一取得此清單上的每一顆鑽石，然後執行相同動作。 完成之後，這就是我們所得到的結果：一連串的點，每一個都代表一顆鑽石。

![散佈圖](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/scatter-plot.png)

## <a name="draw-the-model-through-the-data-points"></a>通過資料點繪製模型
現在如果您查看點與傾向，集合看起來就像一條很模糊的粗線。 我們可以記上標記，並通過它繪製一條直線。

藉由繪製一條線，我們建立了一個「模型」 。 將此想像為現實世界，並為它建立一個極度簡化的草圖版本。 現在草圖有誤 - 這條線並未通過所有資料點。 但它是非常實用的簡化。

![線性迴歸線](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/linear-regression-line.png)

這條線不會完全通過所有點，事實上是沒問題。 資料科學家利用下列方式來說明這一點，假設有一個模型 (就是這條線)，接著每個點都有一些與它相關聯的「雜訊」或「變異數」。 有一個基本的完美關聯性，接著是加入了雜訊和不確定性的真實世界本質。

因為我們正試著回答「多少？」的問題，這就稱為「迴歸」。 而且由於我們使用直線，所以它是「線性迴歸」。

## <a name="use-the-model-to-find-the-answer"></a>使用模型尋找答案
現在我們已有模型，接著詢問它一個問題：1.35 克拉的鑽石價值多少？

為了回答問題，我們仔細打量出 1.35 克拉的所在位置，並繪製一條垂直線。 在它與模型線交會的地方，我們繪製了一條垂直線到美元軸。 它正好命中 10,000。 碰！ 這就是答案了︰一顆 1.35 克拉的鑽石價值約 10,000 美元。

![在模型中尋找答案](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/find-the-answer.png)

## <a name="create-a-confidence-interval"></a>建立信賴區間
很自然地就想要知道此預測的精確度為何。 了解 1.35 克拉的鑽石是否非常接近 10,000 美元，抑或高出許多或低很多，是非常實用的。 為了理解這一點，讓我們在迴歸線四周畫一條包絡線，其中包含大部分的點。 這條包絡線就稱為「信賴區間」 ︰我們非常確信價格會落在此包絡線內，因為在過去它們大多數都是這樣。 我們可以從 1.35 克拉線與此包絡線頂端和底部交會的地方繪製兩條以上的垂直線。

![信賴區間](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/confidence-interval.png)

現在讓我們來談論關於信賴區間的資訊：我們可以很有自信地表示 1.35 克拉鑽石的價格大約是 10,000 美元，但它最低可能是 8,000 美元，最高可能是 12,000 美元。

## <a name="were-done-with-no-math-or-computers"></a>我們不需用到任何數學或電腦就能完成
我們完成了資料科學家需要報酬才會進行的工作，而且只需繪製下列動作就能完成：

* 我們詢問可以使用資料回答的問題
* 我們使用「線性迴歸」建立了「模型」
* 我們進行「預測」，並使用「信賴區間」來完成

而且並未使用數學或電腦來執行此動作。

現在，如果我們當時有更多資訊，例如...

* 鑽石切割
* 色澤變化 (鑽石接近白色的程度)
* 鑽石中內含物的數目

...則會有更多欄。 在此情況下，數學就會變得很有幫助。 如果您有兩欄以上，就很難在紙張上繪製點。 數學可使該條線或該平面非常準確地符合您的資料。

此外，如果不是只有少數的鑽石，而是有兩千個或兩百萬個，則您可以使用電腦更快速地進行此動作。

現在，我們已討論過如何進行線性迴歸，而且已使用資料進行預測。

請務必查看 Microsoft Azure Machine Learning 中「適用於初學者的資料科學」的其他影片。

## <a name="next-steps"></a>後續步驟
* [嘗試使用 Machine Learning Studio 進行您的第一個資料科學實驗](create-experiment.md)
* [在 Microsoft Azure 上取得 Machine Learning 簡介](what-is-machine-learning.md)
