---
title: 在 Machine Learning 中解譯模型結果 | Microsoft Docs
description: 如何針對使用和視覺化評分模型輸出的演算法選擇最佳的參數設定。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 6230e5ab-a5c0-4c21-a061-47675ba3342c
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.openlocfilehash: e62f4716a95a6c1840afde27361eeb756abff335
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836999"
---
# <a name="interpret-model-results-in-azure-machine-learning"></a>在 Azure Machine Learning 中解譯模型結果
本主題說明如何視覺化和解譯 Azure Machine Learning Studio 中的預測結果。 在您訓練好模型並完成其預測 (「模型評分」) 之後，您必須了解和解譯預測結果。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

Azure Machine Learning 中有四個主要的機器學習類型：

* 分類
* 叢集
* 迴歸
* 推薦系統

用來預測這些模型的模組如下︰

* [評分模型][score-model]模組，用於分類和迴歸
* [指派至叢集][assign-to-clusters]模組，用於加入叢集
* [評分 Matchbox 推薦][score-matchbox-recommender]，用於推薦系統

本文件說明如何針對每個模組解譯預測結果。 如需這些模組的概觀，請參閱[如何選擇參數來最佳化 Azure Machine Learning 中的演算法](algorithm-parameters-optimize.md)。

本主題說明預測解譯，但是未說明模型評估。 如需如何評估模型的詳細資訊，請參閱[如何在 Azure Machine Learning 中評估模型效能](evaluate-model-performance.md)。

如果您是 Azure Machine Learning 的新手，並且需要建立簡單實驗以開始使用的說明，請參閱[在 Azure Machine Learning Studio 中建立簡單實驗](create-experiment.md)。

## <a name="classification"></a>分類
分類問題方面有兩個子類別：

* 只有兩個分類的問題 (雙類別或二進位分類)
* 兩個以上分類的問題 (多類別分類)

Azure Machine Learning 有不同的模組可以處理各種類型的分類，但解譯其預設結果的方法相似。

### <a name="two-class-classification"></a>雙類別分類
**範例實驗**

雙類別分類問題的範例是鳶尾花的分類。 作法是根據特徵來分類鳶尾花。 Azure Machine Learning 中提供的鳶尾花資料集是熱門[鳶尾花資料集](http://en.wikipedia.org/wiki/Iris_flower_data_set)的子集，僅包含兩個花卉物種 (類別 0 和 1) 的執行個體。 每個花卉有四個特徵 (萼片長度、萼片寬度、花瓣長度及花瓣寬度)。

![鳶尾花實驗的螢幕擷取畫面](./media/interpret-model-results/1.png)

圖 1. 鳶尾花雙類別分類問題實驗

已執行實驗以解決此問題，如「圖 1」所示。 已訓練及評分雙類別促進式決策樹模型。 您現在可以從[評分模型][score-model]模組將預測結果視覺化，方法是按一下[評分模型][score-model]模組的輸出連接埠，然後按一下 [視覺化]。

![評分模型模組](./media/interpret-model-results/1_1.png)

這樣會帶出評分結果，如圖 2 所示。

![鳶尾花雙類別分類實驗的結果](./media/interpret-model-results/2.png)

圖 2. 視覺化雙類別分類中的評分模型結果

**結果解譯**

結果資料表中有六個資料行。 左側四個資料行是四個特徵。 右側兩個資料行 (「評分標籤」和「評分機率」) 是預測結果。 「評分機率」資料行顯示花卉屬於正類別 (類別 1) 的機率。 例如，資料行中的第一個數字 (0.028571) 表示第一個花卉屬於類別 1 的機率有 0.028571。 「評分標籤」資料行顯示每個花卉的預測類別。 這是根據「評分機率」資料行。 如果花卉的評分機率大於 0.5，則會預測為類別 1。 否則會預測為類別 0。

**Web 服務發佈**

了解預測結果並且完全評判之後，可以將實驗發佈為 Web 服務，以便您在各種應用程式中進行部署及呼叫，以取得任何新的鳶尾花的類別預測。 若要了解如何將訓練實驗變更為評分實驗並且發佈為 Web 服務，請參閱[發佈 Azure Machine Learning Web 服務](walkthrough-5-publish-web-service.md)。 此程序可提供給您如圖 3 所示的評分實驗。

![評分實驗的螢幕擷取畫面](./media/interpret-model-results/3.png)

圖 3. 鳶尾花雙類別分類問題實驗評分

您現在必須設定 Web 服務的輸入和輸出。 輸入是[評分模型][score-model]的右側輸入連接埠，這是鳶尾花的特徵輸入。 輸出的選擇取決於您是對於預測類別 (評分標籤)、評分機率或兩者感到興趣。 此範例假設您對兩者都感到興趣。 若要選取想要的輸出資料行，請使用[選取資料集中的資料行][select-columns]模組。 依序按一下 [選取資料集中的資料行][select-columns] 和 **啟動資料行選取器**，然後選取 [評分標籤] 和 [評分機率]。 設定 [選取資料集中的資料行][select-columns] 的輸出連接埠並再次執行之後，您應該就可以按一下 [發佈 WEB 服務]，將評分實驗發佈為 Web 服務。 最終實驗如「圖 4」所示。

![鳶尾花雙類別分類實驗](./media/interpret-model-results/4.png)

圖 4. 鳶尾花雙類別分類問題的最終評分實驗

執行 Web 服務並且輸入測試執行個體的一些特徵值之後，結果會傳回兩個數字。 第一個數字是評分標籤，而第二個數字是評分機率。 此花卉預測為類別 1，其機率為 0.9655。

![測試解譯評分模型](./media/interpret-model-results/4_1.png)

![測試結果評分](./media/interpret-model-results/5.png)

圖 5. 鳶尾花雙類別分類的 Web 服務結果

### <a name="multi-class-classification"></a>多類別分類
**範例實驗**

在此實驗中，您將執行字母辨識工作，做為多元分類的範例。 分類器會嘗試根據一些從手寫影像中擷取的手寫屬性值，預測特定字母 (類別)。

![字母辨識範例](./media/interpret-model-results/5_1.png)

在定型資料中，有 16 個擷取自手寫字母影像的特徵。 26 個字母形成 26 個類別。 圖 6 顯示的實驗將以訓練多元分類模型進行字母辨識，並在測試資料集上針對相同的特徵集進行預測。

![字母辨識多類別分類實驗](./media/interpret-model-results/6.png)

圖 6. 字母辨識多類別分類問題實驗

從[評分模型][score-model]模組將結果視覺化，方法是按一下[評分模型][score-model]模組的輸出連接埠，然後按一下 [視覺化]，您應會看見如圖 7 所示的內容。

![評分模型模組](./media/interpret-model-results/7.png)

圖 7. 視覺化多類別分類中的評分模型結果

**結果解譯**

左側 16 個資料行代表測試集的特徵值。 類別 "XX" 之名稱為「評分機率」之類的資料行，就像是雙類別案例的「評分機率」資料行。 它們會顯示對應項目落在特定類別的機率。 例如，對於第一個項目，有 0.003571 的機率是 “A”，有 0.000451 的機率是 “B”，依此類推。 最後的資料行 (評分標籤) 與雙類別案例的「評分標籤」相同。 它會選取具有最大評分機率的類別做為對應項目的預測類別。 例如，對於第一個項目，評分標籤為 “F”，因為它的最大機率是 “F” (0.916995)。

**Web 服務發佈**

您也可以取得每個項目的評分標籤以及評分標籤的機率。 基本邏輯是尋找所有評分機率當中最大的機率。 若要這麼做，您需要使用[執行 R 指令碼][execute-r-script]模組。 R 程式碼如圖 8 所示，實驗的結果如圖 9 所示。

![R 程式碼範例](./media/interpret-model-results/8.png)

圖 8. R 程式碼，用以擷取評分標籤和標籤的相關聯機率

![實驗結果](./media/interpret-model-results/9.png)

圖 9. 字母辨識多類別分類問題的最終評分實驗

發佈及執行 Web 服務並輸入一些輸入特徵值之後，傳回的結果如圖 10 所示。 此手寫字母具有其擷取的 16 個特徵，預測為 “T” 的機率是 0.9715。

![測試解譯評分模組](./media/interpret-model-results/9_1.png)

![測試結果](./media/interpret-model-results/10.png)

圖 10. 多類別分類的 Web 服務結果

## <a name="regression"></a>迴歸
迴歸問題與分類問題不同。 在分類問題中，您會嘗試預測離散案例，例如鳶尾花屬於哪個類別。 但如以下迴歸問題範例所示，您會嘗試預測連續變數，例如汽車的價格。

**範例實驗**

使用汽車價格預測做為您的迴歸範例。 您會嘗試根據汽車的特徵預測其價格，例如製造、燃料類型、車體類型和驅動輪。 實驗如「圖 11」所示。

![汽車價格迴歸實驗](./media/interpret-model-results/11.png)

圖 11. 汽車價格迴歸問題實驗

視覺化[評分模型][score-model]模組，結果如圖 12 所示。

![汽車價格預測問題的評分結果](./media/interpret-model-results/12.png)

圖 12. 汽車價格預測問題的評分結果

**結果解譯**

「評分標籤」是此評分結果中的結果資料行。 數字是每部汽車的預測價格。

**Web 服務發佈**

您可以將迴歸實驗發佈至 Web 服務，並且針對汽車價格預測呼叫，方式與雙類別分類使用案例中的方式相同。

![汽車價格迴歸問題的評分實驗](./media/interpret-model-results/13.png)

圖 13. 汽車價格迴歸問題的評分實驗

執行 Web 服務，傳回的結果如「圖 14」所示。 此汽車的預測價格為 $15,085.52。

![測試解譯評分模組](./media/interpret-model-results/13_1.png)

![評分模組結果](./media/interpret-model-results/14.png)

圖 14. 汽車價格迴歸問題的 Web 服務結果

## <a name="clustering"></a>叢集
**範例實驗**

讓我們再次使用鳶尾花資料集來建立叢集實驗。 您可以在這裡篩選資料集中的類別標籤，使其僅具有特徵並可用於叢集。 在此鳶尾花使用案例中，將訓練處理期間的叢集數指定為 2，表示您想要將花卉叢集為兩個類別。 實驗如「圖 15」所示。

![鳶尾花叢集問題實驗](./media/interpret-model-results/15.png)

圖 15. 鳶尾花叢集問題實驗

叢集與分類的不同之處在於訓練資料集本身沒有實況標籤。 將訓練資料集執行個體群組至不同的叢集。 在訓練處理期間，模型會為項目加上標籤，方法是學習其特徵之間的差異。 之後，定型模型可進一步用來分類未來的項目。 在叢集問題當中，我們感興趣的結果有兩個部分。 第一個部分是為訓練資料集加上標籤，而第二個部分是使用定型模型來分類新的資料集。

您可以按一下[訓練叢集模型][train-clustering-model]的左側輸出連接埠，然後按一下 [視覺化]，將結果的第一個部分視覺化。 視覺化如圖 16 所示。

![叢集結果](./media/interpret-model-results/16.png)

圖 16. 視覺化訓練資料集的叢集結果

結果的第二個部分，使用已訓練的叢集模型叢集新的項目，如「圖 17」所示。

![視覺化叢集結果](./media/interpret-model-results/17.png)

圖 17. 視覺化新資料集的叢集結果

**結果解譯**

雖然兩個部分的結果出自於不同的實驗階段，但是其外觀相同，並且是以相同的方式進行解譯。 前面四個資料行是特徵。 最後一個資料行 (指派) 是預測結果。 已指派相同數字的項目預測為在相同叢集中，也就是說，它們某種程度上有相似之處 (此實驗使用預設的歐幾里德距離度量)。 因為您將叢集數目指定為 2，則 [指派] 中的項目會標示為 0 或 1。

**Web 服務發佈**

您可以將叢集實驗發佈至 Web 服務，並且針對叢集預測呼叫，方式與雙類別分類使用案例中的方式相同。

![鳶尾花叢集問題的評分實驗](./media/interpret-model-results/18.png)

圖 18. 鳶尾花叢集問題的評分實驗

執行 Web 服務之後，傳回的結果如圖 19 所示。 此花卉預測為在叢集 0 中。

![測試解譯評分模組](./media/interpret-model-results/18_1.png)

![評分模組結果](./media/interpret-model-results/19.png)

圖 19. 鳶尾花雙類別分類的 Web 服務結果

## <a name="recommender-system"></a>推薦系統
**範例實驗**

對於推薦系統，您可以使用餐廳推薦問題做為範例：您可以根據餐廳的評等歷史為客戶推薦餐廳。 輸入資料是由三個部分組成：

* 來自客戶的餐廳評等
* 客戶特色資料
* 餐廳特色資料

我們可以使用 Azure Machine Learning 的[訓練 Matchbox 推薦][train-matchbox-recommender]模組做許多事情：

* 為指定使用者和項目預測評等
* 對指定使用者推薦項目
* 尋找指定使用者相關的使用者
* 尋找指定項目相關的項目

您可以選擇想要執行的動作，方法是從 [推薦預測種類] 功能表中的四個選項進行選取。 您可以在這裡逐步完成這四個案例。

![Matchbox 推薦](./media/interpret-model-results/19_1.png)

典型的推薦系統 Azure Machine Learning 實驗如圖 20 所示。 如需如何使用這些推薦系統模組的詳細資訊，請參閱[訓練 Matchbox 推薦][train-matchbox-recommender]和[評分 Matchbox 推薦][score-matchbox-recommender]。

![推薦系統實驗](./media/interpret-model-results/20.png)

圖 20. 推薦系統實驗

**結果解譯**

**為指定使用者和項目預測評等**

藉由選取 [推薦預測種類] 之下的 [評等預測]，您會要求推薦系統預測指定使用者和項目的評等。 [評分 Matchbox 推薦][score-matchbox-recommender]輸出的視覺化如圖 21 所示。

![推薦系統的評分結果 - 評等預測](./media/interpret-model-results/21.png)

圖 21. 視覺化推薦系統的評分結果 - 評等預測

前兩個資料行是輸入資料提供的使用者-項目組。 第三個資料行是對於特定項目之使用者的預測評等。 例如，在第一個資料列中，預測客戶 U1048 將餐廳 135026 評等為 2。

**對指定使用者推薦項目**

藉由選取 [推薦預測種類] 之下的 [項目推薦]，您會要求推薦系統對指定使用者推薦項目。 此案例中需要選擇的最後一個參數是「推薦項目選取」。 選項 [ **從評等項目 (針對模型評估)** ] 主要適用於訓練處理期間的模型評估。 對於此預測階段，我們選擇 [從所有項目] 。 [評分 Matchbox 推薦][score-matchbox-recommender]輸出的視覺化如圖 22 所示。

![推薦系統的評分結果 - 項目推薦](./media/interpret-model-results/22.png)

圖 22. 視覺化推薦系統的評分結果 - 項目推薦

六個資料行中的第一個資料行代表用於推薦項目的指定使用者識別碼，由輸入資料提供。 其他五個資料行代表對使用者推薦的項目 (以相關性的遞減順序排序)。 例如，在第一個資料列中，對客戶 U1048 最推薦的餐廳是 134986，接下來是 135018、134975、135021 及 132862。

**尋找指定使用者相關的使用者**

藉由選取 [推薦預測種類] 之下的 [相關使用者]，您會要求推薦系統尋找指定使用者的相關使用者。 相關使用者是具有類似偏好的使用者。 此案例中需要選擇的最後一個參數是「相關使用者選取」。 [從評等項目的使用者 (針對模型評估)] 選項主要適用於訓練處理期間的模型評估。 對於此預測階段選擇 [從所有使用者]。 [評分 Matchbox 推薦][score-matchbox-recommender]輸出的視覺化如圖 23 所示。

![推薦系統的評分結果 - 相關使用者](./media/interpret-model-results/23.png)

圖 23. 視覺化推薦系統的評分結果 - 相關使用者

六個資料行中的第一個資料行顯示尋找相關使用者所需的指定使用者識別碼，由輸入資料提供。 其他五個資料行會儲存使用者的預測相關使用者 (以相關性的遞減順序排序)。 例如，在第一個資料列中，對於客戶 U1048 最相關的客戶是 U1051，接下來是 U1066、U1044、U1017 及 U1072。

**尋找指定項目相關的項目**

藉由選取 [推薦預測種類] 之下的 [相關項目]，您會要求推薦系統尋找指定項目的相關項目。 相關項目是相同使用者最有可能喜歡的項目。 此案例中需要選擇的最後一個參數是「相關項目選取」。 選項 [ **從評等項目 (針對模型評估)** ] 主要適用於訓練處理期間的模型評估。 對於此預測階段，我們選擇 [ **從所有項目** ]。 [評分 Matchbox 推薦][score-matchbox-recommender]輸出的視覺化如圖 24 所示。

![推薦系統的評分結果 - 相關項目](./media/interpret-model-results/24.png)

圖 24. 視覺化推薦系統的評分結果 - 相關項目

六個資料行中的第一個資料行代表尋找相關項目所需的指定項目識別碼，由輸入資料提供。 其他五個資料行會儲存項目的預測相關項目 (以相關性的遞減順序排序)。 例如，在第一個資料列中，對項目 135026 最相關的項目是 135074，接下來是 135035、132875、135055 及 134992。

**Web 服務發佈**

將這些實驗發佈為 Web 服務以取得預測的處理類似於四個案例。 我們在這裡採用第二個案例 (對指定的使用者推薦項目) 做為範例。 您可以遵循其他三個案例的相同程序。

將已訓練的推薦系統儲存為定型模型，並將輸入資料篩選為要求的單一使用者識別碼資料行，您可以連結如圖 25 所示的實驗，並且將其發佈為 Web 服務。

![餐廳推薦問題的評分實驗](./media/interpret-model-results/25.png)

圖 25. 餐廳推薦問題的評分實驗

執行 Web 服務，傳回的結果如圖 26 所示。 對於使用者 U1048 的五個推薦餐廳是 134986、135018、134975、135021 及 132862。

![推薦系統服務的範例](./media/interpret-model-results/25_1.png)

![範例實驗結果](./media/interpret-model-results/26.png)

圖 26. 餐廳推薦問題的 Web 服務結果

<!-- Module References -->
[assign-to-clusters]: https://msdn.microsoft.com/library/azure/eed3ee76-e8aa-46e6-907c-9ca767f5c114/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[score-matchbox-recommender]: https://msdn.microsoft.com/library/azure/55544522-9a10-44bd-884f-9a91a9cec2cd/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[train-clustering-model]: https://msdn.microsoft.com/library/azure/bb43c744-f7fa-41d0-ae67-74ae75da3ffd/
[train-matchbox-recommender]: https://msdn.microsoft.com/library/azure/fa4aa69d-2f1c-4ba4-ad5f-90ea3a515b4c/
