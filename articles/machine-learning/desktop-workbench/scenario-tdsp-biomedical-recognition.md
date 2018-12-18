---
title: 生物醫學實體辨識 - Team Data Science Process - Azure Machine Learning | Microsoft Docs
description: 在 Azure Machine Learning Workbench 中使用生物醫學實體辨識深度學習的 Team Data Science Process 專案快速入門。
services: machine-learning
documentationcenter: ''
author: deguhath
ms.author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 775d0701f9eeaf7d26a83b814711bea630efb2a6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973757"
---
# <a name="biomedical-entity-recognition-using-team-data-science-process-tdsp-template"></a>使用 Team Data Science Process (TDSP) 範本的生物醫學實體辨識

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



實體擷取是資訊擷取的子工作 (也稱為[具名實體辨識 (NER)](https://en.wikipedia.org/wiki/Named-entity_recognition)、實體區塊化與實體識別)。 此真實案例的目的在於強調如何使用 Azure Machine Learning Workbench 來解決複雜的自然語言處理 (NLP) 工作，例如從非結構化文字擷取實體：

1. 如何使用 [Spark Word2Vec 實作](https://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec)，在大約 1800 萬筆 PubMed 摘要的文字語料庫上訓練類神經字組內嵌模型。
2. 如何在 Azure 上已啟用 GPU 的 Azure 資料科學虛擬機器 (GPU DS VM) 上，針對實體擷取建立深度長短期記憶 (LSTM) 遞迴式類神經網路模型。
2. 示範該特定領域字組內嵌模型在實體辨識工作的表現可優於一般字組內嵌模型。 
3. 示範如何使用 Azure Machine Learning Workbench 來訓練深度學習模型並加以運算化。

4. 示範 Azure Machine Learning Workbench 中的下列功能：

    * [Team Data Science Process (TDSP) 結構和範本](how-to-use-tdsp-in-azure-ml.md)的具現化
    * 自動化管理專案相依性，包括下載和安裝
    * 在不同計算環境上執行 Python 指令碼
    * 執行 Python 指令碼的歷程記錄追蹤
    * 在使用 HDInsight Spark 2.1 叢集的遠端 Spark 計算內容上執行作業
    * 在 Azure 上的遠端 GPU VM 中執行作業
    * 輕易將深度學習模型運算化作為 Azure Container Service (ACS) 上的 Web 服務

## <a name="use-case-overview"></a>使用案例概觀
生物醫學具名實體辨識是複雜生物醫學 NLP 工作的重要步驟，例如： 
* 從電子醫療或健康記錄中擷取提及具名實體的部分，例如疾病、藥物、化學藥品和徵狀。
* 藥物探索
* 了解不同實體類型之間的互動，例如藥物交互作用、藥物-疾病關係和基因-蛋白質關係。

我們的使用案例著重於如何分析大量非結構化的資料語料庫 (例如 Medline PubMed 摘要) 來訓練字組內嵌模型。 然後輸出內嵌項目會被視為自動產生的特徵，用以訓練類神經實體擷取器。

我們的結果顯示在特定領域字組內嵌特徵上進行的生物醫學實體擷取模型訓練，其表現優於在一般特徵類型上訓練的模型。 相較於一般模型可偵測到 5274 個 F1分數為 0.61 的實體，特定領域模型可以正確偵測到 7012 個 F1 分數為 0.73 的實體 (在 9475 個之中)。

下圖顯示用來處理資料和訓練模型的架構。

![架構](./media/scenario-tdsp-biomedical-recognition/architecture.png)

## <a name="data-description"></a>資料說明

### <a name="1-word2vec-model-training-data"></a>1.Word2Vec 模型訓練資料
我們已先從 [MEDLINE](https://www.nlm.nih.gov/pubs/factsheets/medline.html) 下載原始 MEDLINE 摘要資料。 此資料會以 XML 檔案的形式在其 [FTP 伺服器](https://ftp.ncbi.nlm.nih.gov/pubmed/baseline)上公開提供。 伺服器上有 892 個 XML 檔案，每個 XML 檔案有 30,000 篇文章的資訊。 「專案結構」一節中會提供更多有關資料收集步驟的詳細資訊。 每個檔案中存在的欄位如下： 
        
        abstract
        affiliation
        authors
        country 
        delete: boolean if False means paper got updated so you might have two XMLs for the same paper.
        file_name   
        issn_linking    
        journal 
        keywords    
        medline_ta: this is abbreviation of the journal nam 
        mesh_terms: list of MeSH terms  
        nlm_unique_id   
        other_id: Other IDs 
        pmc: Pubmed Central ID  
        pmid: Pubmed ID
        pubdate: Publication date
        title

### <a name="2-lstm-model-training-data"></a>2.LSTM 模型訓練資料

類神經實體擷取模型已在公開可用的資料集上進行訓練和評估。 若要取得這些資料集的詳細描述，您可參考下列來源：
 * [Bio-Entity Recognition Task at BioNLP/NLPBA 2004](http://www.nactem.ac.uk/tsujii/GENIA/ERtask/report.html)
 * [BioCreative V CDR task corpus](http://www.biocreative.org/tasks/biocreative-v/track-3-cdr/)
 * [Semeval 2013 - Task 9.1 (Drug Recognition)](https://www.cs.york.ac.uk/semeval-2013/task9/)

## <a name="link-to-the-azure-gallery-github-repository"></a>連結至 Azure 資源庫 GitHub 存放庫
以下是真實案例的公用 GitHub 存放庫連結，其中包含程式碼和更多詳細的說明： 

[https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction)


## <a name="prerequisites"></a>必要條件 

* Azure [訂用帳戶](https://azure.microsoft.com/free/)
* Azure Machine Learning Workbench. 請參閱[安裝指南](quickstart-installation.md)。 Azure Machine Learning Workbench 目前只能安裝在下列作業系統上： 
    * Windows 10 或 Windows Server 2016
    * macOS Sierra (或更新版本)


### <a name="azure-services"></a>Azure 服務
* Linux (HDI 3.6) 上 [HDInsight Spark 叢集](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-jupyter-spark-sql) Spark 2.1 版，適用於相應放大運算。 若要處理下面討論的完整 MEDLINE 摘要數量，您需要以下基本設定： 
    * 前端節點：[D13_V2](https://azure.microsoft.com/pricing/details/hdinsight/) 大小
    * 背景工作節點：至少 4 個 [D12_V2](https://azure.microsoft.com/pricing/details/hdinsight/)。 我們在工作中使用了 11 個 D12_V2 大小的背景工作節點。
* [NC6 資料科學虛擬機器 (DSVM)](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-linux-dsvm-intro)，適用於相應放大運算。

### <a name="python-packages"></a>Python 套件

所有必要的相依性會定義於案例專案資料夾下的 aml_config/conda_dependencies.yml 檔案中。 這個檔案中定義的相依性會自動佈建，以便對 Docker、VM 及 HDI 叢集目標執行。 如需 Conda 環境檔案格式的詳細資訊，請參閱[這裡](https://conda.io/docs/using/envs.html#create-environment-file-by-hand)。

* [TensorFlow](https://www.tensorflow.org/install/)
* [CNTK 2.0](https://docs.microsoft.com/cognitive-toolkit/using-cntk-with-keras)
* [Keras](https://keras.io/#installation)
* NLTK
* Fastparquet

### <a name="basic-instructions-for-azure-machine-learning-aml-workbench"></a>Azure Machine Learning (AML) Workbench 的基本指示
* [概觀](../service/overview-what-is-azure-ml.md)
* [安裝](quickstart-installation.md)
* [使用 TDSP](how-to-use-tdsp-in-azure-ml.md)
* [如何讀取和寫入檔案](how-to-read-write-files.md)
* [如何使用 Jupyter Notebook](how-to-use-jupyter-notebooks.md)
* [如何使用 GPU](how-to-use-gpu.md)

## <a name="scenario-structure"></a>案例結構
我們在案例中使用 TDSP 專案結構和文件範本 (圖 1)，其遵循 [TDSP 生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)。 我們會根據[這裡](./how-to-use-tdsp-in-azure-ml.md)提供的指示建立專案。


![填入專案資訊](./media/scenario-tdsp-biomedical-recognition/instantiation-3.png) 

逐步資料科學工作流程如下所示：

### <a name="1-data-acquisition-and-understanding"></a>1.資料取得與認知

請參閱[資料取得與認知](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/blob/master/code/01_data_acquisition_and_understanding/ReadMe.md)。

原始 MEDLINE 語料庫總共有 2700 萬筆摘要，其中大約 1000 萬篇文章有空白的抽象欄位。 Azure HDInsight Spark 用來處理無法以 [Pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html) 形式載入單一機器記憶體的巨量資料。 首先，資料會下載到 Spark 叢集中。 然後在 [Spark DataFrame](https://spark.apache.org/docs/latest/sql-programming-guide.html) 上執行下列步驟： 
* 使用 Medline XML 剖析器來剖析 XML 檔案
* 進行摘要文字前置處理，包括句子分割、Token 化和大小寫正規化。
* 排除摘要欄位空白或具有簡短文字的文章 
* 從訓練摘要建立字組詞彙
* 訓練字組內嵌類神經模型。 如需詳細資訊，請參閱 [GitHub 程式碼連結](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/blob/master/code/01_data_acquisition_and_understanding/ReadMe.md)即可開始使用。


剖析 XML 檔案之後，資料具有下列格式： 

![資料樣本](./media/scenario-tdsp-biomedical-recognition/datasample.png)

類神經實體擷取模型已在公開可用的資料集上進行訓練和評估。 若要取得這些資料集的詳細描述，您可參考下列來源：
 * [Bio-Entity Recognition Task at BioNLP/NLPBA 2004](http://www.nactem.ac.uk/tsujii/GENIA/ERtask/report.html)
 * [BioCreative V CDR task corpus](http://www.biocreative.org/tasks/biocreative-v/track-3-cdr/)
 * [Semeval 2013 - Task 9.1 (Drug Recognition)](https://www.cs.york.ac.uk/semeval-2013/task9/)

### <a name="2-modeling"></a>2.模型化

請參閱[模型化](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling)。

在模型化階段中，我們會示範如何使用在前一節下載的資料來訓練自己的字組內嵌模型，並將它用於其他下游工作。 雖然我們使用 PubMed 資料，但產生內嵌項目所用的管線為一般管線，所以可重複使用於訓練任何其他領域的字組內嵌項目。 對於即將精確表示資料的內嵌項目，有必要在大量資料上訓練 word2vec。
一旦備妥字組內嵌項目，我們即可訓練深度類神經網路模型，其使用學習到的內嵌項目來初始化內嵌圖層。 我們會將內嵌圖層標示為不可訓練，但並非強制性。 字組內嵌模型的訓練並非監督式，因此我們能夠充分利用無標籤的文字。 不過，實體辨識模型的訓練是一項監督式學習工作，其精確度取決於手動標註資料的數量和品質。 


#### <a name="21-feature-generation"></a>2.1. 特徵產生

請參閱[特徵產生](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling/01_feature_engineering)。

Word2Vec 是一種字組內嵌非監督式學習演算法，可從無標籤的訓練語料庫中訓練類神經網路模型。 它會為語料庫中的每個字組產生連續向量，代表其語意資訊。 這些模型是簡單的類神經網路，具有單一隱藏圖層。 字組向量/內嵌項目是經由反向傳播法和隨機梯度下降法來學習。 word2vec 模型有兩種類型，也就是 Skip-Gram 和 continuous-bag-of-words (如需詳細資訊，請查看[這裡](https://arxiv.org/pdf/1301.3781.pdf))。 因為我們使用 MLlib 的 word2vec 實作 (支援 Skip-gram 模型)，所我們會在此處加以概略描述 (取自[連結](https://ahmedhanibrahim.wordpress.com/2017/04/25/thesis-tutorials-i-understanding-word2vec-for-word-embedding-i/)的影像)： 

![Skip Gram 模型](./media/scenario-tdsp-biomedical-recognition/skip-gram.png)

此模型使用 Hierarchical Softmax 和 Negative 取樣來達到最佳化效能。 Hierarchical SoftMax (H-SoftMax) 是二元樹所啟發的逼近法。 H-SoftMax 基本上會使用以字組作為分葉的階層式層取代一般 SoftMax 圖層。 這可讓我們將一個字組的機率分解成一連串的計算機率，我們就不必計算耗費資源的正規化所有文字。 因為平衡二進位樹狀目錄的深度為 log2(|V|) (V 為詞彙)，所以我們只需要評估最多 log2(|V|) 節點，即可取得字組的最終機率。 在給定的內容 c 之下 w 字組的機率，就是向右轉和向左轉 (分別導向其分葉節點) 的機率乘積。 我們可以根據資料集中的字組頻率來建立 Huffman 樹狀目錄，確保較頻繁的字組可取得較短的表示法。 如需詳細資訊，請參閱[此連結](http://sebastianruder.com/word-embeddings-softmax/)。
取自[這裡](https://ahmedhanibrahim.wordpress.com/2017/04/25/thesis-tutorials-i-understanding-word2vec-for-word-embedding-i/)的影像。

##### <a name="visualization"></a>視覺效果

一旦擁有字組內嵌項目，我們想要將其視覺化，並查看語義相似字組之間的關聯性。 

![W2V 相似度](./media/scenario-tdsp-biomedical-recognition/w2v-sim.png)

我們已示範兩種將內嵌項目視覺化的方法。 第一種方法使用 PCA 來預測 2-D 向量空間的高維度向量。 這會導致資訊明顯遺失，且視覺效果不夠精確。 第二種方法是搭配使用 PCA 與 [t-SNE](https://distill.pub/2016/misread-tsne/)。 t-SNE 是非線性降維技術，非常適合將高維度的資料嵌入二維或三維空間中，而後可在散佈圖中加以視覺化。  它會以二維或三維點來為每個高維度的物件建立模型，如此一來，相似的物件是由近點建立模型，而不相似的物件則由遠點建立模型。 它適用於兩個部分。 首先，它會對較高維空間中的配對建立機率分佈，其中的相似物件被挑選的機率很高，而不相似點被挑選的機率很低。 接著，它會對低維圖中的點定義類似的機率分佈，並使兩個分佈 (圖上各點的位置) 之間的 KL 散度降到最低。 使用梯度下降法將 KL 散度降到最低，可取得低維度空間中各點的位置。 但 t-SNE 不一定永遠可靠。 在[這裡](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling/01_feature_engineering)可以找到實作詳細資訊。 


如下圖所示，t-SNE 視覺效果更能區隔字組向量與潛在叢集模式。 


* PCA 視覺效果

![PCA](./media/scenario-tdsp-biomedical-recognition/pca.png)

* t-SNE 視覺效果

![t-SNE](./media/scenario-tdsp-biomedical-recognition/tsne.png)

* 最接近「癌症」的點 (它們是所有的癌症子類型)

![最接近「癌症」的點](./media/scenario-tdsp-biomedical-recognition/nearesttocancer.png)

#### <a name="22-train-the-neural-entity-extractor"></a>2.2. 訓練類神經實體擷取器

請參閱[訓練類神經實體擷取器](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling/02_model_creation/ReadMe.md)。

前饋類神經網路架構遇到一個問題：其將每個輸入和輸出視為獨立於其他輸入和輸出。 此架構無法為序列到序列標記工作 (例如機器翻譯和實體擷取) 建立模型。 遞迴式類神經網路模型可克服此問題，因為這種模型可將到目前為止計算的資訊傳遞至下一個節點。 此特性稱之為在網路中有記憶體，因為它能夠使用先前計算的資訊，如下圖所示：

![RNN](./media/scenario-tdsp-biomedical-recognition/rnn-expanded.png)

Vanilla RNN 實際上會遇到[梯度消失問題](https://en.wikipedia.org/wiki/Vanishing_gradient_problem)，因此無法利用其先前看到的所有資訊。 只有在需要大量內容才能進行預測時，此問題才很明顯。 但是 LSTM 之類的模型不會遇到這種問題，事實上其設計可記憶長期相依性。 不同於具有單一類神經網路的 Vanilla RNN，LSTM 的每個細胞都有四個類神經網路間的交互作用。 如需 LSTM 運作方式的詳細說明，請參閱[這篇文章](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)。

![LSTM 細胞](./media/scenario-tdsp-biomedical-recognition/lstm-cell.png)

讓我們試著整合自己的 LSTM 型遞迴式類神經網路，然後嘗試從 PubMed 資料中擷取藥物、疾病和徵兆等實體類型。 第一個步驟是取得大量加上標籤的資料，如您所猜測，這不容易！ 大多數醫療資料包含有關個人的許多機密資訊，因此不可公開使用。 我們仰賴可公開使用的兩個不同資料集的組合。 第一個資料集來自 Semeval 2013 - Task 9.1 (藥物辨識)，而另一個則是來自 BioCreative V CDR Task。 我們會合併並自動標記這兩個資料集，以便偵測醫療文字中的藥物和疾病，並評估我們的字組內嵌項目。 如需實作詳細資訊，請參閱 [GitHub 程式碼連結](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling/02_model_creation)。

我們使用於所有程式碼且用於比較的模型架構呈現於下方。 隨著不同資料集變更的參數是最大序列長度 (此處為 613)。

![LSTM 模型](./media/scenario-tdsp-biomedical-recognition/d-a-d-model.png)

#### <a name="23-model-evaluation"></a>2.3. 模型評估
請參閱[模型評估](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/02_modeling/03_model_evaluation/ReadMe.md)。

我們使用共用工作 [Bio-Entity Recognition Task at Bio NLP/NLPBA 2004](http://www.nactem.ac.uk/tsujii/GENIA/ERtask/report.html) 中的評估指令碼，評估模型的求準率 (Precision)、求全率 (Recall) 和 F1 分數。 

#### <a name="in-domain-versus-generic-word-embedding-models"></a>領域內部與一般字組內嵌模型

以下是兩種特徵類型的精確度比較：(1) 在 PubMed 摘要上訓練的字組內嵌項目 (2) 在 Google 新聞上訓練的字組內嵌項目。 我們清楚地發現領域內部模型的表現優於一般模型。 因此擁有特定的字組內嵌模型，比使用一般模型更有幫助。 

* 第 1 項工作：藥物和疾病偵測

![模型比較 1](./media/scenario-tdsp-biomedical-recognition/mc1.png)

我們會以類似的方式在其他資料集上執行字組內嵌項目的評估，並發現該領域內部模型總是更好。

* 第 2 項工作：蛋白質、細胞株、細胞類型、DNA 和 RNA 偵測

![模型比較 2](./media/scenario-tdsp-biomedical-recognition/mc2.png)

* 第 3 項工作：化學藥品和疾病偵測

![模型比較 3](./media/scenario-tdsp-biomedical-recognition/mc3.png)

* 第 4 項工作：藥物偵測

![模型比較 4](./media/scenario-tdsp-biomedical-recognition/mc4.png)

* 第 5 項工作：基因偵測

![模型比較 5](./media/scenario-tdsp-biomedical-recognition/mc5.png)

#### <a name="tensorflow-versus-cntk"></a>TensorFlow 與 CNTK
所有報告的模型都已使用以 TensorFlow 作為後端的 Keras 來訓練。 在此工作完成時，具有 CNTK 後端的 Keras 不支援「反轉」。 因此，為了進行比較，我們已訓練具有 CNTK 後端的單向 LSTM 模型，並將它與具有 TensorFlow 後端的單向 LSTM 模型進行比較。 從[這裡](https://docs.microsoft.com/cognitive-toolkit/using-cntk-with-keras)安裝適用於 Keras 的 CNTK 2.0。 

![模型比較 6](./media/scenario-tdsp-biomedical-recognition/mc6.png)

我們的結論是，就每個 epoch 所需的訓練時間 (CNTK 需要 75 秒，而 Tensorflow 需要 60 秒) 和偵測到的測試實體數目而言，CNTK 與 Tensorflow 的表現一樣好。 我們會使用單向圖層進行評估。


### <a name="3-deployment"></a>3.部署

請參閱[部署](https://github.com/Azure/MachineLearningSamples-BiomedicalEntityExtraction/tree/master/code/03_deployment)。

我們已在 [Azure Container Service (ACS)](https://azure.microsoft.com/services/container-service/) 中的叢集上部署 Web 服務。 運算化環境會在叢集中佈建 Docker 和 Kubernetes，以管理 Web 服務部署。 您可以在[這裡](model-management-service-deploy.md )找到有關運算化程序的進一步資訊。


## <a name="conclusion"></a>結論

我們已詳細說明如何在 Spark 上使用 Word2Vec 演算法來訓練字組內嵌模型，然後使用擷取的內嵌項目作為特徵來訓練深度類神經網路，以便進行實體擷取。 我們已在生物醫學領域上運用訓練管線。 不過，此管線的運用相當廣泛，可偵測任何其他領域的自訂實體類型。 您只需要足夠的資料，即可針對不同領域輕鬆地修改這裡所呈現的工作流程。

## <a name="references"></a>參考

* Tomas Mikolov, Kai Chen, Greg Corrado, and Jeffrey Dean. 2013a. Efficient estimation of word representations in vector space. In Proceedings of ICLR.
* Tomas Mikolov, Ilya Sutskever, Kai Chen, Greg S Corrado, and Jeff Dean. 2013b. Distributed representations of words and phrases and their compositionality. In Proceedings of NIPS, pages 3111–3119.
* Billy Chiu, Gamal Crichton, Anna Korhonen, and Sampo Pyysalo. 2016. [How to Train Good Word Embeddings for Biomedical NLP](http://aclweb.org/anthology/W/W16/W16-2922.pdf), In Proceedings of the fifteenth Workshop on Biomedical Natural Language Processing, pages 166–174.
* [Vector Representations of Words](https://www.tensorflow.org/tutorials/word2vec)
* [遞迴式類神經網路模型](https://www.tensorflow.org/tutorials/recurrent)
* [Problems encountered with Spark ml Word2Vec](https://intothedepthsofdataengineering.wordpress.com/2017/06/26/problems-encountered-with-spark-ml-word2vec/)
* [Spark Word2Vec: lessons learned](https://intothedepthsofdataengineering.wordpress.com/2017/06/26/spark-word2vec-lessons-learned/)

