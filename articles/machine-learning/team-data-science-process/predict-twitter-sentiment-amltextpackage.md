---
title: 使用適用於文字分析的 Azure Machine Learning (AML) 套件 (AMLPTA) 和 Team Data Science Process (TDSP) 進行 Twitter 人氣分類 | Microsoft Docs
description: 說明如何使用 TDSP (Team Data Science Process) 和 AMLPTA 進行人氣分類
services: machine-learning, team-data-science-process
documentationcenter: ''
author: deguhath
manager: deguhath
editor: cgronlun
ms.assetid: b8fbef77-3e80-4911-8e84-23dbf42c9bee
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: deguhath
ms.openlocfilehash: acad053712d237fdb86934faa7c5c4546c630387
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394550"
---
# <a name="twitter-sentiment-classification-with-azure-machine-learning-aml-package-for-text-analytics-amlpta-and-team-data-science-process-tdsp"></a>使用適用於文字分析的 Azure Machine Learning (AML) 套件 (AMLPTA) 和 Team Data Science Process (TDSP) 進行 Twitter 人氣分類

## <a name="introduction"></a>簡介
結構的標準化和資料科學專案的記載 (錨定至已建立的[資料科學生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)) 是促進資料科學小組有效共同作業的關鍵。

我們先前發行了[適用於 TDSP 專案結構和範本的 GitHub 存放庫](https://github.com/Azure/Azure-TDSP-ProjectTemplate)。 我們現在已能夠建立使用 [Azure Machine Learning 的 TDSP 結構和文件範本](https://github.com/amlsamples/tdsp)具現化的 Azure Machine Learning 專案。 [這裡](https://docs.microsoft.com/azure/machine-learning/preview/how-to-use-tdsp-in-azure-ml)提供如何在 Azure Machine Learning 中使用 TDSP 結構和範本的指示。 

在此範例中，我們會示範如何使用「適用於文字分析的 Azure Machine Learning 套件」和 TDSP 來開發和部署預測模型，以進行 Twitter 人氣分類。


## <a name="use-case"></a>使用案例
### <a name="twitter-sentiment-polarity-sample"></a>Twitter 人氣極性範例
本文會使用範例，示範如何具現化並執行 Machine Learning 專案。 此範例會在 Azure Machine Learning Workbench 中使用 TDSP 結構和範本。 本逐步解說中提供完整範例。 模型化工作會使用推文中的文字來預測人氣極性 (正數或負數)。 本文概述逐步解說中所述的資料模型化工作。 本逐步解說涵蓋下列工作：

1. 機器學習模型的資料探勘、定型和部署，可處理使用案例概觀中所述的預測問題。 Twitter 人氣資料會用於這些工作。
2. 藉由針對此專案從 Azure Machine Learning 使用 TDSP 範本來執行專案。 對於專案執行和報告，會使用 TDSP 生命週期。
3. 直接從 Azure Container Services 中的 Azure Machine Learning 運作解決方案。

此專案會點出如何使用 Azure Machine Learning 的文字分析套件。


## <a name="link-to-github-repository"></a>連結至 GitHub 存放庫
GitHub 存放庫的連結[在此](https://github.com/Azure/MachineLearningSamples-AMLTextPackage-TwitterSentimentPrediction)。 

### <a name="purpose"></a>目的
此範例的主要目的在於示範如何在 Azure Machine Learning Workbench 中使用 Team Data Science Process (TDSP) 結構和範本，具現化和執行機器學習專案。 為此，我們會使用 [Twitter 人氣資料](http://cs.stanford.edu/people/alecmgo/trainingandtestdata.zip)。 模型化工作是為了使用推文中的文字來預測人氣極性 (正數或負數)。

### <a name="scope"></a>影響範圍
- 機器學習模型的資料探勘、訓練和部署，可處理「使用案例概觀」中所述的預測問題。
- 使用 Azure Machine Learning 中此專案的 Team Data Science Process (TDSP) 範本，在 Azure Machine Learning 中執行專案。 對於專案執行和報告，我們將會使用 TDSP 生命週期。
- 直接從 Azure Container Services 中的 Azure Machine Learning 將解決方案運算化。

此專案強調 Azure Machine Learning 的好幾項功能，例如 TDSP 結構具現化和使用、執行 Azure Machine Learning Workbench 中的程式碼，以及使用 Docker 和 Kubernetes 在 Azure Container Services 中輕易進行運算化。

## <a name="team-data-science-process-tds"></a>Team Data Science Process (TDSP)
我們使用 TDSP 專案結構和文件範本來執行此範例。 此範例遵循 [TDSP 生命週期](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/lifecycle)。 我們會根據[這裡](https://github.com/amlsamples/tdsp/blob/master/docs/how-to-use-tdsp-in-azure-ml.md)提供的指示建立專案。


<img src="./media/predict-twitter-sentiment-amltextpackage/tdsp-lifecycle2.png" alt="tdsp-lifecycle" width="800" height="600">


## <a name="use-case-overview"></a>使用案例概觀
此工作是為了使用從 Twitter 文字擷取出的字組內嵌功能來預測每個 Twitter 的人氣二元極性。 如需詳細說明，請參閱此[存放庫](https://github.com/Azure/MachineLearningSamples-AMLTextPackage-TwitterSentimentPrediction)。

### <a name="data-acquisition-and-understandinghttpsgithubcomazuremachinelearningsamples-amltextpackage-twittersentimentpredictiontreemastercode01dataacquisitionandunderstanding"></a>[資料取得與認知](https://github.com/Azure/MachineLearningSamples-AMLTextPackage-TwitterSentimentPrediction/tree/master/code/01_data_acquisition_and_understanding)
此範例中的第一個步驟是下載 sentiment140 資料集，並將其分成定型和測試資料集。 Sentiment140 資料集包含推文實際內容 (移除了表情符號) 以及每則推文的極性 (負面=0、正面=4)，其中的中性推文則會予以移除。 所產生的定型資料具有一百三十萬個資料列，測試資料具有三十二萬個資料列。

### <a name="modelinghttpsgithubcomazuremachinelearningsamples-amltextpackage-twittersentimentpredictiontreemastercode02modeling"></a>[模型化](https://github.com/Azure/MachineLearningSamples-AMLTextPackage-TwitterSentimentPrediction/tree/master/code/02_modeling)
範例的這個部分會進一步分成三個子部分： 
- **功能工程**：對應於使用不同的字組內嵌演算法 (名稱為 Word2Vec) 來產生功能。 
- **模型建立**：可處理不同模型的定型 (例如，「羅吉斯迴歸」和「漸層停駐提升」)，這些模型可用來預測輸入文字的人氣。 
- **模型評估**：對測試資料套用已定型的模型。

#### <a name="feature-engineering"></a>特徵設計
我們使用 <b>Word2Vec</b> 來產生字組內嵌。 我們會先在 Skip-gram 模式中使用 Word2Vec 演算法，如 [Mikolov, Tomas, et al.Distributed representations of words and phrases and their compositionality.Advances in neural information processing systems.2013.](https://arxiv.org/abs/1310.4546) 所述，來產生字組內嵌。

Skip-gram 是淺層的類神經網路，會將編碼為一個熱向量的目標字組作為輸入，然後用它來預測鄰近的字組。 如果 V 是詞彙的大小，則輸出層的大小會是 __C*V__，其中 C 是內容視窗的大小。 下圖會顯示 Skip-gram 型架構。
 
<table class="image" align="center">
<caption align="bottom">Skip-gram 模型</caption>
<tr><td><img src="https://s3-ap-south-1.amazonaws.com/av-blog-media/wp-content/uploads/2017/06/05000515/Capture2-276x300.png" alt="Skip-gram model"/></td></tr>
</table>

Word2Vec 演算法和 Skip-gram 模型的詳細資料不在此範例的討論範圍內，讀者若有興趣，請瀏覽下列連結以取得更多詳細資料。 02_A_Word2Vec.py 程式碼參考了 [tensorflow 範例](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/tutorials/word2vec/word2vec_basic.py)

* [以向量表示字組](https://www.tensorflow.org/tutorials/word2vec)
* [Word2vec 的確切運作方式？](http://www.1-4-5.net/~dmm/ml/how_does_word2vec_work.pdf)
* [雜訊對比估計和負面取樣的注意事項](http://demo.clab.cs.cmu.edu/cdyer/nce_notes.pdf)

在定型程序完成之後，會針對模型化階段產生兩個 TSV 格式的內嵌檔案。

#### <a name="model-training"></a>模型訓練
使用 SSWE 或 Word2Vec 演算法產生字組向量之後，下一步是將分類模型定型以預測實際的人氣極性。 我們會對兩個模型 (羅吉斯迴歸和卷積神經網路 (CNN)) 套用兩種功能 (Word2Vec 和 SSWE)。 

#### <a name="model-evaluation"></a>模型評估
我們會提供如何對測試資料集載入和評估多個已定型模型的程式碼。


### <a name="deploymenthttpsgithubcomazuremachinelearningsamples-amltextpackage-twittersentimentpredictiontreemastercode03deployment"></a>[部署](https://github.com/Azure/MachineLearningSamples-AMLTextPackage-TwitterSentimentPrediction/tree/master/code/03_deployment)
在這個部分，我們會提供指標來指示該如何讓預先定型的人氣預測模型，在 Azure Container Service (AKS) 中叢集上的 Web 服務中運作。 運算化環境會在叢集中佈建 Docker 和 Kubernetes，以管理 Web 服務部署。 您可以在[這裡](https://docs.microsoft.com/azure/machine-learning/preview/model-management-service-deploy)找到有關運算化程序的進一步資訊。

## <a name="conclusion"></a>結論
我們已詳細說明如何使用 Word2Vec 將字組內嵌模型定型，然後使用擷取的內嵌項目作為特徵將兩個不同的模型定型，以預測 Twitter 文字資料的人氣分數。 其中一個模型會部署在 Azure Container Service (AKS) 中。 

## <a name="next-steps"></a>後續步驟
請進一步閱讀[適用於文字分析的 Azure Machine Learning 套件 (AMLPTA)](https://docs.microsoft.com/python/api/overview/azure-machine-learning/textanalytics?view=azure-ml-py-latest) 和 [Team Data Science Process (TDSP)](https://aka.ms/tdsp) 上的文件來開始了解。

## <a name="references"></a>參考
* [Team Data Science Process](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/overview) 
* [如何在 Azure Machine Learning 中使用 Team Data Science Process (TDSP)](https://aka.ms/how-to-use-tdsp-in-aml)
* [Azure Machine Learning 的 TDSP 專案範本](https://aka.ms/tdspamlgithubrepo)
* [Azure ML WorkBench](https://docs.microsoft.com/azure/machine-learning/preview/)
* [Mikolov, Tomas 等人Distributed representations of words and phrases and their compositionality.Advances in neural information processing systems.2013.](https://arxiv.org/abs/1310.4546)
