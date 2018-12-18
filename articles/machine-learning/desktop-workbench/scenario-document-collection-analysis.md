---
title: 文件集合分析 - Azure | Microsoft Docs
description: 如何摘要及分析大型文件集合，包括片語學習、主題模型化，以及使用 Azure ML Workbench 的主題模型分析等技術。
services: machine-learning
author: kehuan
ms.author: kehuan
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, MicrosoftDocs/mlreview, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/20/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 762955103aeb48eb8a9b62f4e3ffe193bba71a38
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947212"
---
# <a name="document-collection-analysis"></a>文件集合分析

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



此案例示範如何摘要及分析大型文件集合，包括片語學習、主題模型化，以及使用 Azure ML Workbench 的主題模型分析等技術。 Azure Machine Learning Workbench 為非常大型的文件集合提供簡單的相應增加功能，並提供一些機制來訓練及調整各種計算內容中的模型，範圍從本機計算到資料科學虛擬機器以至 Spark 叢集。 透過 Azure Machine Learning Workbench 內的 Jupyter Notebook 提供簡易的開發。

## <a name="link-to-the-gallery-github-repository"></a>連結至資源庫 GitHub 存放庫

此實際案例的公用 GitHub 存放庫包含此範例所需的所有資料，包括程式碼範例：

[https://github.com/Azure/MachineLearningSamples-DocumentCollectionAnalysis](https://github.com/Azure/MachineLearningSamples-DocumentCollectionAnalysis)

## <a name="overview"></a>概觀

對於每天收集的大量資料 (尤其是非結構化文字資料)，組織、搜尋及了解這些大量文字是一大挑戰。 此文件集合分析案例示範一個有效率且自動化的端對端工作流程，以供分析大型文件集合並可進行下游 NLP 工作。

此案例所傳遞的重要元素如下：

1. 學習文件中的主要多字片語。

1. 探索文件集合中呈現的基礎主題。

1. 依主題分佈表示文件。

1. 呈現根據主題內容來組織、搜尋和摘要文件的方法。

此案例中呈現的方法能夠完成各種重要的產業工作負載，例如探索主題趨勢異常、文件集合摘要，以及類似文件搜尋。 它可運用到許多不同類型的文件分析，例如政府法規、新聞報導、產品評論、客戶意見反應及科學研究文章。

此案例中使用的機器學習技術/演算法包括：

1. 文字處理和清除

1. 片語學習

1. 主題模型化

1. 語料庫摘要

1. 主題趨勢和異常偵測

## <a name="prerequisites"></a>必要條件

要執行此範例所需符合的必要條件如下：

* 確定您已依照[安裝和建立快速入門](quickstart-installation.md)，正確地安裝 [Azure Machine Learing Workbench](../service/overview-what-is-azure-ml.md)。

* 此範例可在任何計算環境中執行。 不過，建議將它執行於多核心機器上，且該機器至少有 16 GB 的記憶體和 5 GB 的磁碟空間。

## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「文件集合分析」並選取範本
5.  按一下 [建立] 

## <a name="data-description"></a>資料說明

此案例中使用的資料集包含美國國會所採取之每個立法行動的文字摘要和相關中繼資料。 資料是從 [GovTrack.us](https://www.govtrack.us/) 收集而來，該網站可追蹤美國國會的活動並協助美國公民參與其國家立法程序。 使用手動指令碼可透過[此連結](https://www.govtrack.us/data/congress/)下載大量資料，但該指令碼並未包含在此案例中。 在 [GovTrack API 文件](https://www.govtrack.us/developers)中可找到中如何下載資料的詳細資訊。

### <a name="data-source"></a>資料來源

在此案例中收集的未經處理資料是由美國國會從 1973 年到 2017 年 6 月 (第 93 屆至第 115 屆國會) 提出的一系列立法行動 (提議的法案和決議案)。 所收集的資料採用 JSON 格式，而且包含一組豐富的立法行動相關資訊。 如需資料欄位的詳細說明，請參閱[此 GitHub 連結](https://github.com/unitedstates/congress/wiki/bills)。 為了在此案例中進行示範，我們只會從 JSON 檔案擷取一部分的資料欄位。 此案例提供一個預先編譯的 TSV 檔案 `CongressionalDataAll_Jun_2017.tsv`，其中包含這些立法動作的記錄。 您可經由 Notebook 資料夾下的 Notebook `1_Preprocess_Text.ipynb` 或 Python 套件中的 `preprocessText.py` 自動下載 TSV 檔案。

### <a name="data-structure"></a>資料結構

資料檔案中有 9 個資料欄位。 資料欄位名稱和說明如下所列。

| 欄位名稱 | 類型 | 說明 | 包含遺漏值 |
|------------|------|-------------|---------------|
| `ID` | 字串 | 法案/決議案的識別碼。 此欄位的格式為 [bill_type][number]-[congress]。 例如，"hconres1-93" 表示法案類型為 "hconres" (代表 House Concurrent Resolution，請參閱[本文件](https://github.com/unitedstates/congress/wiki/bills#basic-information))，法案編號為 '1' 且國會屆次為 '93'。 | 否 |
| `Text` | 字串 | 法案/決議案的內容。 | 否 |
| `Date` | 字串 | 法案/決議案的最初提案日期。 格式為 'yyyy-mm-dd'。 | 否 |
| `SponsorName` | 字串 | 提議法案/決議案的主要發起人名稱。 | 是 |
| `Type` | 字串 | 主要發起人的頭銜類型，'rep' (議員) 或 'sen' (參議員)。 | 是 |
| `State` | 字串 | 主要發起人的州別。 | 是 |
| `District` | 整數  | 主要發起人的轄區編號 (如果發起人的頭銜是議員的話)。 | 是 |
| `Party` | 字串 | 主要發起人的黨派。 | 是 |
| `Subjects` | 字串 | 由國會圖書館累積新增至法案的主題字詞。 字詞會以逗號串連。 這些字詞是由國會圖書館人員所轉寫，在首度發佈法案資訊時通常不存在。 可隨時加以新增。 因此，在法案的生命週期結束前，某些主題可能不再相關。 | 是 |

## <a name="scenario-structure"></a>案例結構

文件的集合分析範例可分成兩種交付項目。 第一種是一系列的 iPython Notebook，以供顯示整個工作流程的逐步說明。 第二種是 Python 套件，以及如何使用該套件的一些程式碼範例。 Python 套件很普通，足以為許多使用案例提供服務。

此範例中的檔案整理如下。

| 檔案名稱 | 類型 | 說明 |
|-----------|------|-------------|
| `aml_config` | 資料夾 | Azure Machine Learning Workbench 組態資料夾，請參閱[此文件](./experimentation-service-configuration-reference.md)以取得詳細的實驗執行組態 |
| `Code` | 資料夾 | 用來儲存 Python 指令碼和 Python 套件的程式碼資料夾 |
| `Data` | 資料夾 | 用來儲存中繼檔案的資料夾 |
| `notebooks` | 資料夾 | Jupyter Notebook 資料夾 |
| `Code/documentAnalysis/__init__.py` | Python 檔案 | Python 套件 init 檔案 |
| `Code/documentAnalysis/configs.py` | Python 檔案 | 由文件分析 Python 套件使用的組態檔，包括預先定義的常數 |
| `Code/documentAnalysis/preprocessText.py` | Python 檔案 | 此 Python 檔案用於下游工作的資料前置處理 |
| `Code/documentAnalysis/phraseLearning.py` | Python 檔案 | 此 Python 檔案用來學習資料中的片語及轉換未經處理的資料 |
| `Code/documentAnalysis/topicModeling.py` | Python 檔案 | 此 Python 檔案用來訓練潛在狄氏配置 (LDA) 主題模型 |
| `Code/step1.py` | Python 檔案 | 文件集合分析的步驟 1：文字前置處理 |
| `Code/step2.py` | Python 檔案 | 文件集合分析的步驟 2：片語學習 |
| `Code/step3.py` | Python 檔案 | 文件集合分析的步驟 3：訓練和評估 LDA 主題模型 |
| `Code/runme.py` | Python 檔案 | 範例中的一個檔案中執行所有步驟 |
| `notebooks/1_Preprocess_Text.ipynb` | iPython Notebook | 前置處理文字及轉換未經處理的資料 |
| `notebooks/2_Phrase_Learning.ipynb` | iPython Notebook | 從文字資料學習片語 (在資料轉換後) |
| `notebooks/3_Topic_Model_Training.ipynb` | iPython Notebook | 訓練 LDA 主題模型 |
| `notebooks/4_Topic_Model_Summarization.ipynb` | iPython Notebook | 根據定型的 LDA 主題模型摘要文件集合的內容 |
| `notebooks/5_Topic_Model_Analysis.ipynb` | iPython Notebook | 分析文字文件集合的主題內容，以及讓主題資訊與文件集合相關聯的其他中繼資料相互關聯 |
| `notebooks/6_Interactive_Visualization.ipynb` | iPython Notebook | 已學習主題模型的互動式視覺效果 |
| `notebooks/winprocess.py` | Python 檔案 | Notebook 用於多工處理的 python 指令碼 |
| `README.md` | Markdown 檔案 | Markdown 讀我檔案 |

### <a name="data-ingestion-and-transformation"></a>資料擷取和轉換

已編譯的資料集 `CongressionalDataAll_Jun_2017.tsv` 會儲存在 Blob 儲存體中，並可從 Notebook 和 Python 指令碼存取。 資料擷取和轉換有兩個步驟：文字資料前置處理，以及片語學習。

#### <a name="preprocess-text-data"></a>文字資料前置處理

前置處理步驟會運用自然語言處理技術來清除和準備未經處理的文字資料。 它可作為非監督式片語學習和潛在主題模型化的先驅。 在 Notebook `1_Preprocess_Text.ipynb` 中可找到詳細的逐步說明。 另外還有 Python 指令碼 `step1.py` 對應至這個 Notebook。

在此步驟中，TSV 資料檔案會從 Azure Blob 儲存體下載並匯入為 Pandas DataFrame。 DataFrame 中的每個資料列元素都是一長字串的連貫文字或一份「文件」。 然後每份文件會分成文字區塊，有可能是句子、片語或子片語。 分割的用意是要禁止片語學習程序在學習片語時，使用交叉句子或交叉片語的字組字串。

Notebook 和 Python 套件中都有針對前置處理步驟定義的多個函式。 呼叫這兩行程式碼即可達成大部分的作業。

```python
# Read raw data into a Pandas DataFrame
textDF = getData()

# Write data frame with preprocessed text out to TSV file
cleanedDataFrame = CleanAndSplitText(textDF, saveDF=True)
```

#### <a name="phrase-learning"></a>片語學習

片語學習步驟會實作一個基本架構，以學習大型文件集合中的主要片語。 標題為 "[Modeling Multiword Phrases with Constrained Phrases Tree for Improved Topic Modeling of Conversational Speech](http://people.csail.mit.edu/hazen/publications/Hazen-SLT-2012.pdf)" 的論文中有其說明，這篇論文起初是在 2012 IEEE Workshop on Spoken Language Technology 中提出。 片語學習步驟的詳細實作會顯示在 iPython Notebook `2_Phrase_Learning.ipynb` 和 Python 指令碼 `step2.py` 中。

此步驟將已清除的文字當作輸入，並學習大型文件集合中的最主要片語。 片語學習是一個反覆程序，可分成三個工作：計算 n-gram、依組成字組的加權點式交互資訊將潛在片語排名，以及將片語重寫成文字。 這三項工作會在多次反覆運算中循序執行，直到已學習指定的片語為止。

在文件分析 Python 套件中，Python 類別 `PhraseLearner` 已定義於 `phraseLearning.py` 檔案中。 以下是用來學習片語的程式碼片段。

```python
# Instantiate a PhraseLearner and run a configuration
phraseLearner = PhraseLearner(cleanedDataFrame, "CleanedText", numPhrase,
                        maxPhrasePerIter, maxPhraseLength, minInstanceCount)

# The chunks of text in a list
textData = list(phraseLearner.textFrame['LowercaseText'])

# Learn most salient phrases present in a large collection of documents
phraseLearner.RunConfiguration(textData,
            phraseLearner.learnedPhrases,
            addSpace=True,
            writeFile=True,
            num_workers=cpu_count()-1)
```

> [!NOTE]
> 片語學習步驟是透過多工處理來實作。 不過，CPU 核心愈多**不**表示執行時間愈快。 在我們的測試中，使用超過 8 個核心並不會改善效能，因為多工處理有額外的負荷。 在具有 8 個核心 (3.6 GHz) 的電腦上學習 25,000 個片語大約需要兩個半小時。
>

### <a name="topic-modeling"></a>主題模型化

學習潛在主題模型使用 LDA 是此案例中的第三個步驟。 這個步驟需要有 [Gensim](https://radimrehurek.com/gensim/) Python 套件，才能學習 [LDA 主題模型](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation)。 此步驟的對應 Notebook 是 `3_Topic_Model_Training.ipynb`。 您也可以參考 `step3.py`，了解如何使用文件分析套件。

在此步驟中，主要工作是學習 LDA 主題模型並微調超參數。 訓練 LDA 模型時，需要微調多個參數，但最重要的參數是主題數目。 太少主題可能無法深入了解文件集合；而選擇太多主題將會導致語料庫「過度分群」成許多小型、高度相似的主題。 此案例的目的在於示範如何微調主題的數目。 Azure Machine Learning Workbench 提供使用不同的計算組態，對不同的計算內容執行實驗的自由。

文件分析 Python 套件中已定義了幾個函式，協助使用者找出最佳的主題數目。 第一種方法是藉由評估主題模型的連貫性。 支援的評估矩陣有四個：`u_mass`、`c_v`、`c_uci` 和 `c_npmi`。 [這篇論文](http://svn.aksw.org/papers/2015/WSDM_Topic_Evaluation/public.pdf)會討論這四個計量的詳細資訊。 第二種方法是評估留出語料庫的困惑度 (perplexity)。

為了進行困惑度評估，'U' 形曲線應該要找出最佳的主題數目。 而轉彎處是最佳選擇。 建議您以不同的隨機種子評估很多次，並取得平均值。 連貫性評估應該是 'n' 形，這表示連貫性會隨著主題數目增加而提高，而後降低。 困惑度和 `c_v` 連貫性的範例圖如下所示。

![困惑度](./media/scenario-document-collection-analysis/Perplexity_Value.png)

![c_v Coherence](./media/scenario-document-collection-analysis/c_v_Coherence.png)

在此案例中，困惑度會在超過 200 個主題後大幅提升，而連貫性值會在超過 200 個主題後大幅減少。 根據這些圖形以及對更多一般主題或叢集式主題的需求，選擇 200 個主題應該是一個不錯的選項。

您可以在一次實驗執行中訓練一個 LDA 主題模型，或在一次實驗執行中以不同的主題數目組態來訓練和評估多個 LDA 模型。 建議您執行一個組態很多次，然後取得平均的連貫性及/或困惑度評估。 在 `step3.py` 檔案中可找到如何使用文件分析套件的詳細資訊。 程式碼片段範例如下所示。

```python
topicmodeler = TopicModeler(docs,
        stopWordFile=FUNCTION_WORDS_FILE,
        minWordCount=MIN_WORD_COUNT,
        minDocCount=MIN_DOC_COUNT,
        maxDocFreq=MAX_DOC_FREQ,
        workers=cpu_count()-1,
        numTopics=NUM_TOPICS,
        numIterations=NUM_ITERATIONS,
        passes=NUM_PASSES,
        chunksize=CHUNK_SIZE,
        random_state=RANDOM_STATE,
        test_ratio=test_ratio)

# Train an LDA topic model
lda = topicmodeler.TrainLDA(saveModel=saveModel)

# Evaluate coherence metrics
coherence = topicmodeler.EvaluateCoherence(lda, coherence_types)

# Evaluate perplexity on a held-out corpus
perplex = topicmodeler.EvaluatePerplexity(lda)
```

> [!NOTE]
> 訓練 LDA 主題模型的執行時間取決於多個因素，例如語料庫大小、超參數組態，以及電腦上的核心數目。 使用多個 CPU 核心可更快地訓練模型。 不過，採用相同的超參數組態，較多核心表示訓練期間的更新比較少。 建議您具有**至少 100 項更新來訓練收斂的 LDA 模型**。 [這篇文章](https://groups.google.com/forum/#!topic/gensim/ojySenxQHi4)和[這篇文章](http://miningthedetails.com/blog/python/lda/GensimLDA/)會探討更新數目與超參數數目之間的關聯性。 在我們的測試中，在具有 16 個核心 (2.0 GHz) 的電腦上使用 `workers=15`、`passes=10`、`chunksize=1000` 的組態來訓練具有 200 個主題的 LDA 模型大約需要 3 小時。
>

### <a name="topic-summarization-and-analysis"></a>主題摘要和分析

主題摘要和分析包含兩個 Notebook，而文件分析套件中沒有對應的函式。

在 `4_Topic_Model_Summarization.ipynb` 中，它會根據定型的 LDA 主題模型來摘要文件內容。 摘要會套用至在步驟 3 中學習的 LDA 主題模型。 它會顯示如何使用主體來記載純度 (purity) 量值，以測量主題的重要性或品質。 此純度量值假設可支配所在文件的潛在主題，比起每週散佈於許多文件的潛在主題更具語意重要性。 這個概念是在 "[Latent Topic Modeling for Audio Corpus Summarization](http://people.csail.mit.edu/hazen/publications/Hazen-Interspeech11.pdf)" 這篇論文中引進。

Notebook `5_Topic_Model_Analysis.ipynb` 顯示如何分析文件集合的主題內容，以及讓主題資訊與文件集合相關聯的其他中繼資料相互關聯。 此 Notebook 中引進了幾個標繪圖，可協助使用者深入了解已學習的主題和文件集合。

Notebook `6_Interactive_Visualization.ipynb` 示範如何以互動方式視覺化已學習主題模型。 它包含四個互動式視覺效果工作。

## <a name="conclusion"></a>結論

此真實案例強調如何使用知名的文字分析技術 (在此例中是片語學習和 LDA 主題模型化) 來產生穩健的模型，以及 Azure Machine Learning Workbench 如何協助追蹤模型效能和順暢地執行較大規模的學習模組。 詳細資訊：

* 使用片語學習和主題模型化來處理文件集合和建立穩健的模型。 如果文件集合很龐大，則 Azure Machine Learning Workbench 可以輕鬆地將它相應增加和相應放大。此外，使用者可以輕鬆自在地從 Azure Machine Learning Workbench 對不同的計算內容執行實驗。

* Azure Machine Learning Workbench 提供選項：逐步執行 Runbook 和使用 Python 指令碼執行整個實驗。

* 使用 Azure Machine Learning Workbench 進行超參數微調，可找出學習所需的最佳主題數目。 Azure Machine Learning Workbench 可協助追蹤模型效能和順暢地執行較大規模的不同學習模組。

* Azure Machine Learning Workbench 可以管理執行歷程記錄和已學習的模型。 它可讓資料科學家快速找出效能最佳的模型，並找出用來產生這些模型的指令碼和資料。

## <a name="references"></a>參考

* **Timothy J. Hazen, Fred Richardson**, [_Modeling Multiword Phrases with Constrained Phrases Tree for Improved Topic Modeling of Conversational Speech_](http://people.csail.mit.edu/hazen/publications/Hazen-SLT-2012.pdf). Spoken Language Technology Workshop (SLT), 2012 IEEE. IEEE, 2012.

* **Timothy J. Hazen**, [_Latent Topic Modeling for Audio Corpus Summarization_](http://people.csail.mit.edu/hazen/publications/Hazen-Interspeech11.pdf). 12th Annual Conference of the International Speech Communication Association. 2011.

* **Michael Roder, Andreas Both, Alexander Hinneburg**, [_Exploring the Space of Topic Coherence Measures_](http://svn.aksw.org/papers/2015/WSDM_Topic_Evaluation/public.pdf). Proceedings of the eighth ACM international conference on Web search and data mining. ACM, 2015.
