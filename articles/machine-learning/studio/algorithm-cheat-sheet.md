---
title: 機器學習演算法小祕技 - Azure | Microsoft Docs
description: 可列印的機器學習演算法小密技可協助您在 Azure Machine Learning Studio 中選擇適合您預測模型的演算法。
keywords: 演算法小祕技,小祕技,機器學習演算法
services: machine-learning
author: pakalra
ms.author: pakalra
manager: cgronlun
editor: cgronlun
ms.assetid: e1dc31ec-1acb-463f-ba77-de565d4ddf4d
ms.service: machine-learning
ms.component: studio
ms.topic: article
ms.date: 12/18/2017
ms.openlocfilehash: 56fadde5694dbd298c812db258105e03b7d23d45
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996735"
---
# <a name="machine-learning-algorithm-cheat-sheet-for-azure-machine-learning-studio"></a>適用於 Azure Machine Learning Studio 的機器學習演算法小祕技

**Azure Machine Learning 演算法小祕技**可協助您針對預測性分析模型選擇正確的演算法。

[Azure Machine Learning Studio](https://studio.azureml.net/) 有來自***迴歸***、***分類***、***叢集***和***異常偵測***系列的大型演算法程式庫。 每項的設計均是用來處理不同類型的機器學習服務問題。

## <a name="download-machine-learning-algorithm-cheat-sheet"></a>下載：機器學習服務演算法小祕技

**在此下載小祕技：[機器學習服務演算法小祕技 (11x17 英吋)](http://download.microsoft.com/download/A/6/1/A613E11E-8F9C-424A-B99D-65344785C288/microsoft-machine-learning-algorithm-cheat-sheet-v7.pdf)**

![機器學習演算法小祕技：了解如何選擇機器學習演算法。][cheat-sheet]

[cheat-sheet]: ./media/algorithm-cheat-sheet/machine-learning-algorithm-cheat-sheet-small_v_0_6-01.png

下載並列印 Tabloid 大小的「Machine Learning Studio 演算法小祕技」，以隨時瀏覽並獲得選擇演算法的協助。

> [!NOTE]
> 如需如何使用這個小祕技選擇正確的演算法、進一步探討不同類型的機器學習演算法，以及它們的使用方式，請參閱 [如何選擇 Microsoft Azure Machine Learning 的演算法](algorithm-choice.md)。

## <a name="notes-and-terminology-definitions-for-the-machine-learning-studio-algorithm-cheat-sheet"></a>Machine Learning Studio 演算法小祕技的注意事項與詞彙定義

* 這份演算法小祕技中提供的建議是近似經驗法則。 您可以屈從有些建議，也可以公然違反有些建議。 其主要目的是建議起點。 別擔心您的資料上會出現數種演算法間勢均力敵的競爭。 唯有了解每個演算法的原則，以及了解產生您的資料的系統。

* 每個機器學習服務演算法都有自己的風格或*歸納偏差*。 對於特定問題，適合的演算法可能有數個，但其中一個演算法可能會比其他演算法更適合。 但不一定能夠事先知道哪一種最適合。 在這類情況下，小祕技中會一起列出數個演算法。 適當策略就是嘗試一個演算法，而如果結果還不滿意，就嘗試其他演算法。 以下是 [Azure AI 資源庫](http://gallery.azure.ai/)中的實驗範例，該實驗對相同的資料嘗試數種演算法，並比較其結果：[比較多類別分類器：字母辨識](http://gallery.azure.ai/Details/a635502fc98b402a890efe21cec65b92)。

* 機器學習有三個主要類別：**經過指導的學習**、**未經指導的學習**和**增強式學習**。

  * 在**經過指導的學習**中，每個資料點都會加上標籤或與感興趣的類別或值產生關聯。  分類標籤的範例就是將影像指派為「貓」或「狗」。  值標籤的範例則是與中古汽車相關聯的銷售價格。 監督式學習的目標是研究許多像是這些已加上標籤的範例，然後能夠做出有關未來資料點的預測。 例如，識別包含正確動物的新相片，或者針對其他中古汽車指派精確的銷售價格。 這是常見且實用的機器學習服務類型。 除了 [K-Means 叢集][k-means-clustering]以外，Azure Machine Learning 中的所有模組都是經過指導的學習演算法。

  * 在**未監督的學習**中，資料點沒有與其相關聯的標籤。 然而，未經指導的學習演算法的目標在於以某種方式組織資料或描述其結構。 這可能表示將資料劃分為叢集 (如 K-Means 所為)，或尋找各種查看複雜資料的方式，使其變得更簡單。

  * 在**增強式學習**中，演算法需要選擇一個動作來回應每個資料點。 這是在機器人領域中的常見方法，其中在某個時間點的感應器讀數集就是一個資料點，而演算法必須選擇機器人的下一個動作。 它也很適合用於物聯網 (Internet of Things) 應用程式。 此學習演算法也會在短時間內收到獎勵訊號，指出決策的好壞程度。 演算法會據此修改其策略，以達到最高的獎勵。 Azure ML 中目前沒有增強式學習演算法模組。

* **貝氏法**可進行統計上獨立的資料點假設。 這表示在一個資料點中的非模型化變異不會與其他資料點相關，也就是，無法加以預測。 例如，如果要記錄的資料是下一班地鐵到達前的分鐘數，則相隔一天取得的兩個測量值屬於統計獨立。 不過，相隔一分鐘取得的兩個測量值則不屬於統計獨立 - 其中一個值可高度預測另一個值。

* **促進式決策樹迴歸**會利用特徵重疊或特徵間的互動。 這表示，在任何給定的資料點中，一項特徵的值有點能夠預測另一項特徵的值。 例如，在每日高/低溫資料中，得知當天的低溫可讓您合理猜測高溫。 兩個特徵中包含的資訊有點冗餘。

* 使用原本就是多類別的分類器，或將一組雙類別的分類器合併成一個**整體** (Ensemble)，即可將資料分類成兩個以上的類別。 在整體方法中，每個類別有個別的雙類別分類器 - 每個分類器會將資料分為兩類：「這一類」和「非這一類」。 然後這些分類器會表決資料點的正確指派。 這是[一對多的多類別][one-vs-all-multiclass]背後的運作原則。

* 數種方法 (包括羅吉斯迴歸和貝氏點機器) 採用**線性類別界限**。 也就是說，它們會假設類別之間的界限大約是直線 (或者，更一般的情況下為超平面)。 通常這是您嘗試劃分之後才會知道的資料特性，但通常可以透過事先視覺化來學習。 如果類別界限看起來非常不規則，請堅守決策樹、決策叢林、支援向量機器或類神經網路。

* 為每個類別建立**虛擬變數**，並將它設定為 1 (適用此類別時) 或 0 (不適用此類別時)，即可搭配類別變數使用類神經網路。

## <a name="next-steps"></a>後續步驟

* 如需描述演算法並提供範例的可下載資訊圖詳細資訊，請參閱[可下載的資訊圖：機器學習服務基本概念和演算法範例](basics-infographic-with-algorithm-examples.md)。

* 如需 Machine Learning Studio 中依類別排序的所有可用機器學習演算法的清單，請參閱＜Machine Learning Studio 演算法和模組說明＞中的[起始模型][initialize-model]。

* 如需 Machine Learning Studio 中按字母順序排列的完整演算法和模組清單，請參閱＜Machine Learning Studio 演算法和模組說明＞中的 [Machine Learning Studio 模組的 A-Z 清單][a-z-list]。

* 若要下載並列印提供 Machine Learning Studio 功能概觀的圖表，請參閱 [Azure Machine Learning Studio 功能的概觀圖](studio-overview-diagram.md)。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

<!-- Module References -->
[a-z-list]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/a-z-module-list
[initialize-model]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/machine-learning-initialize-model
[k-means-clustering]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/k-means-clustering
[one-vs-all-multiclass]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/one-vs-all-multiclass
