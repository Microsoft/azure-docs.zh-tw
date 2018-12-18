---
title: 使用 Azure 中的 Team Data Science Process 以文字內嵌預測 Twitter 人氣 | Microsoft Docs
description: 執行資料科學專案所需的步驟。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/15/2017
ms.author: deguhath
ms.openlocfilehash: f47668cd706b78977418925d64eca583d7878cd3
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34838304"
---
# <a name="predict-twitter-sentiment-with-word-embeddings-by-using-the-team-data-science-process"></a>使用 Team Data Science Process 以文字內嵌預測 Twitter 人氣

本文章示範如何使用 _Word2Vec_ 文字內嵌演算法和_人氣特定文字內嵌 (SSWE)_ 演算法有效率地共同作業，使用 [Azure Machine Learning](../service/index.yml) 來預測 Twitter 人氣。 如需有關預測 Twitter 人氣極性的詳細資訊，請參閱 GitHub 上的 [MachineLearningSamples-TwitterSentimentPrediction 存放庫](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction)。 促進對於資料科學專案之有效小組共同作業的關鍵，是使用已建立的資料科學生命週期將專案的結構和文件標準化。 [Team Data Science Process (TDSP)](overview.md) 可提供這個類型的結構化[生命週期](lifecycle.md)。 

使用 _TDSP 範本_建立資料科學專案，會為 Azure Machine Learning 專案提供標準化架構。 TDSP 小組先前發行了[適用於 TDSP 專案結構和範本的 GitHub 存放庫](https://github.com/Azure/Azure-TDSP-ProjectTemplate)。 現在使用 [Azure Machine Learning 的 TDSP 範本](https://github.com/amlsamples/tdsp)所具現化的 Machine Learning 專案已啟用。 如需指示，請參閱如何在 Azure Machine Learning 中使用[具有 TDSP 範本的 TDSP 結構專案](../desktop-workbench/how-to-use-tdsp-in-azure-ml.md)。 


## <a name="twitter-sentiment-polarity-sample"></a>Twitter 人氣極性範例

本文會使用範例，示範如何具現化並執行 Machine Learning 專案。 此範例會在 Azure Machine Learning Workbench 中使用 TDSP 結構和範本。 [本逐步解說](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/blob/master/docs/deliverable_docs/Step_By_Step_Tutorial.md)中提供完整範例。 模型化工作會使用推文中的文字來預測人氣極性 (正數或負數)。 本文概述逐步解說中所述的資料模型化工作。 本逐步解說涵蓋下列工作：

- 機器學習模型的資料探勘、定型和部署，可處理使用案例概觀中所述的預測問題。 [Twitter 人氣資料](http://cs.stanford.edu/people/alecmgo/trainingandtestdata.zip)會用於這些工作。
- 藉由針對此專案從 Azure Machine Learning 使用 TDSP 範本來執行專案。 對於專案執行和報告，會使用 TDSP 生命週期。
- 直接從 Azure Container Services 中的 Azure Machine Learning 運作解決方案。

專案會反白顯示 Azure Machine Learning 的下列功能：

- 具現化及使用 TDSP 結構。
- 在 Azure Machine Learning Workbench 中執行程式碼。
- 使用 Docker 和 Kubernetes 在 Container Service 中輕鬆進行運作。

## <a name="team-data-science-process"></a>Team Data Science Process
若要執行此範例，您可以在 Azure Machine Learning Workbench 中使用 TDSP 專案結構和文件範本。 此範例會實作 [TDSP 生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)，如下圖所示：

![TDSP 生命週期](./media/predict-twitter-sentiment/tdsp-lifecycle.PNG)

TDSP 專案在 Azure Machine Learning Workbench 中是根據[這些指示](https://github.com/amlsamples/tdsp/blob/master/docs/how-to-use-tdsp-in-azure-ml.md)所建立的，如下圖所示：

![在 Azure Machine Learning Workbench 中建立 TDSP](./media/predict-twitter-sentiment/tdsp-instantiation.PNG) 


## <a name="data-acquisition-and-preparationhttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode01dataacquisitionandunderstanding"></a>[資料取得與準備](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/01_data_acquisition_and_understanding)
此範例中的第一個步驟是下載 sentiment140 資料集，並將資料分成定型和測試資料集。 sentiment140 資料集包含推文中的實際內容 (但已移除表情符號)。 資料集也包含每個推文的極性 (負數 = 0，正數 = 4)，但移除了中性推文。 資料分割之後，定型資料具有一百三十萬個資料列，測試資料具有三十二萬個資料列。


## <a name="model-developmenthttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode02modeling"></a>[模型開發](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/02_modeling)

此範例的下一個步驟是開發資料的模型。 模型化工作分成三個部分：

- 功能工程：使用不同的文字內嵌演算法產生模型的功能。 
- 模型建立：將不同的模型定型以預測輸入文字的人氣。 這些模型的範例包括_羅吉斯迴歸_和_漸層停駐提升_。
- 模型評估：評估測試資料的定型模型。


### <a name="feature-engineeringhttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode02modeling01featureengineering"></a>[功能工程](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/02_modeling/01_FeatureEngineering)

Word2Vec 和 SSWE 演算法會用來產生文字內嵌。 


#### <a name="word2vec-algorithm"></a>Word2Vec 演算法

Word2Vec 演算法會在 Skip-Gram 模型中使用。 此模型會在 Tomas Mikolov 等人的論文中說明。"[Distributed Representations of Words and Phrases and their Compositionality.Advances in neural information processing systems.](https://arxiv.org/abs/1310.4546)" 2013.

Skip-Gram 模型是淺層類神經網路。 輸入是目標文字，編碼為獨熱向量，用來預測附近的文字。 如果 **V** 是詞彙的大小，則輸出層的大小是 __C*V__，其中 C 是內容視窗的大小。 下圖顯示根據 Skip-Gram 模型的架構：

![Skip-Gram 模型](./media/predict-twitter-sentiment/skip-gram-model.PNG)

Word2Vec 演算法和 Skip-Gram 模型的詳細機制已超越此範例的範圍。 如需詳細資訊，請參閱下列參考資料：

- [具有參考 TensorFlow 範例的 02_A_Word2Vec.py 程式碼](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/tutorials/word2vec/word2vec_basic.py) 
- [Vector Representations of Words](https://www.tensorflow.org/tutorials/word2vec)
- [Word2vec 的確切運作方式？](http://www.1-4-5.net/~dmm/ml/how_does_word2vec_work.pdf)
- [雜訊對比估計和負面取樣的注意事項](http://demo.clab.cs.cmu.edu/cdyer/nce_notes.pdf)




#### <a name="sentiment-specific-word-embedding-algorithm"></a>人氣特定文字內嵌演算法
SSWE 演算法會嘗試解決 Word2Vec 演算法的弱點，這個弱點是具有類似內容和相反極性的文字會具有類似的文字向量。 這個相似性會造成 Word2Vec 演算法對於人氣分析之類的工作無法準確執行。 SSWE 演算法會嘗試處理此弱點，方法是將句子極性和文字的內容同時納入其損失函式。

此範例會使用 SSWE 演算法的變化，如下所示：

- 原始 _ngram_ 和損毀 _ngram_ 都會當做輸入使用。
- 排名樣式關鍵損失函式會同時用於語法損失和語意損失。
- 終極損失函式是語法損失和語意損失的加權組合。
- 為了簡單起見，只有語意交叉熵會用來作為損失函式。

此範例示範即使使用更簡單的損失函式，SSWE 內嵌的效能還是優於 Word2Vec 內嵌。 下圖顯示用來產生人氣特定文字內嵌的迴旋模型：

![由 SSWE 啟發的類神經網路模型](./media/predict-twitter-sentiment/embedding-model-2.PNG)

在定型程序完成之後，會針對模型化階段產生定位字元分隔值 (TSV) 格式的兩個內嵌檔案。

如需 SSWE 演算法的詳細資訊，請參閱 Duyu Tang 等人的論文。"[Learning Sentiment-Specific Word Embedding for Twitter Sentiment Classification](http://www.aclweb.org/anthology/P14-1146)." Association for Computational Linguistics (1). 2014.


### <a name="model-creationhttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode02modeling02modelcreation"></a>[模型建立](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/02_modeling/02_ModelCreation)
使用 SSWE 或 Word2Vec 演算法產生文字向量之後，便會將分類模型定型以預測實際的人氣極性。 兩種類型的功能：Word2Vec 和 SSWE，會套用至兩個模型：漸層停駐提升模型和羅吉斯迴歸模型。 因此，會將四個不同的模型定型。


### <a name="model-evaluationhttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode02modeling03modelevaluation"></a>[模型評估](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/02_modeling/03_ModelEvaluation)
將模型定型之後，模型會用來測試 Twitter 文字資料，並評估每個模型的效能。 此範例會評估下列四個模型：

- 透過 SSWE 內嵌的漸層停駐提升。
- 透過 SSWE 內嵌的羅吉斯迴歸。
- 透過 Word2Vec 內嵌的漸層停駐提升。
- 透過 Word2Vec 內嵌的羅吉斯迴歸。

下圖顯示四個模型的效能比較：

![接收者作業特性 (ROC) 模型比較](./media/predict-twitter-sentiment/roc-model-comparison.PNG)

使用曲線下面積 (AUC) 計量來比較模型時，具有 SSWE 功能的漸層停駐提升模型會提供最佳的效能。


## <a name="deploymenthttpsgithubcomazuremachinelearningsamples-twittersentimentpredictiontreemastercode03deployment"></a>[部署](https://github.com/Azure/MachineLearningSamples-TwitterSentimentPrediction/tree/master/code/03_deployment)

最後一個步驟是將定型人氣預測模型部署至 Azure Container Service 中叢集上的 Web 服務。 這個範例會使用具有 SSWE 內嵌演算法的漸層停駐提升模型作為定型模型。 運作環境會在叢集中佈建 Docker 和 Kubernetes，以管理 Web 服務部署，如下圖所示： 

![Kubernetes 儀表板](./media/predict-twitter-sentiment/kubernetes-dashboard.PNG)

如需有關運作程序的詳細資訊，請參閱[將 Azure Machine Learning 模型部署為 Web 服務](../desktop-workbench/model-management-service-deploy.md)。

## <a name="conclusion"></a>結論

在本文中，您已學習如何使用 Word2Vec 和人氣特定文字內嵌演算法來將文字內嵌模型定型。 擷取的內嵌會作為功能來將數個模型定型，以預測 Twitter 文字資料的人氣分數。 SSWE 功能與漸層停駐提升模型搭配使用，提供了最佳的效能。 然後會使用 Azure Machine Learning Workbench，將模型部署為 Container Service 中的即時 Web 服務。


## <a name="references"></a>參考

* [Team Data Science Process](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/overview) 
* [如何在 Azure Machine Learning 中使用 Team Data Science Process (TDSP)](https://aka.ms/how-to-use-tdsp-in-aml)
* [Azure Machine Learning 的 TDSP 專案範本](https://aka.ms/tdspamlgithubrepo)
* [Azure Machine Learning Workbench](../service/index.yml)
* [UCI ML 存放庫中的美國所得資料集](https://archive.ics.uci.edu/ml/datasets/adult)
* [使用 TDSP 範本的生物醫學實體辨識](../desktop-workbench/scenario-tdsp-biomedical-recognition.md)
* [Mikolov, Tomas 等人"Distributed Representations of Words and Phrases and their Compositionality.Advances in neural information processing systems."2013.](https://arxiv.org/abs/1310.4546)
* [Tang, Duyu 等人"Learning Sentiment-Specific Word Embedding for Twitter Sentiment Classification."ACL (1).2014.](http://www.aclweb.org/anthology/P14-1146)
