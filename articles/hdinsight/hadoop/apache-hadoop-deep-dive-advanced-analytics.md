---
title: 深入探討 - 進階分析 - Azure HDInsight
description: 了解進階分析如何使用演算法來處理巨量資料。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/14/2017
ms.author: ashishth
ms.openlocfilehash: 40536aae15f2ae9fa5e186cdbea1f75f7d264437
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047135"
---
# <a name="deep-dive---advanced-analytics"></a>深入探討 - 進階分析

## <a name="what-is-advanced-analytics-for-hdinsight"></a>什麼是 HDInsight 的進階分析？

HDInsight 提供可從大量結構化、非結構化及快速移動之資料取得寶貴深入解析的功能。 進階分析會使用高擴充性架構、統計和機器學習服務模型，以及智慧型儀表板，為您提供有意義的深入解析。 機器學習服務 (或「預測性分析」) 會使用可從您資料中的關聯性進行識別並學習的演算法來進行預測，然後引導您做出決策。

## <a name="advanced-analytics-process"></a>進階分析程序

![Process](./media/apache-hadoop-deep-dive-advanced-analytics/process.png)

在您已識別出業務問題並已開始收集和處理資料之後，就必須建立代表所要預測之問題的模型。 您的模型將會使用一或多個機器學習服務演算法，來進行最符合您業務需求的預測類型。  您的大部分資料應該用來將模型定型，其餘部分則用來測試或評估該模型。 

在建立、載入、測試及評估您的模型之後，下一步就是部署該模型，讓它開始為您的問題提供解答。 最後一步是監視您模型的效能，並視需要進行微調。 

## <a name="common-types-of-algorithms"></a>常見的演算法類型

進階分析解決方案提供一組機器學習服務演算法。 以下是演算法類別和相關常見業務使用案例的摘要。

![機器學習服務使用案例](./media/apache-hadoop-deep-dive-advanced-analytics/ml-use-cases.png)

除了選取最適合的演算法之外，您還必須考量是否需要提供定型用的資料。 機器學習服務演算法歸類如下：

* 監督式 - 演算法必須先在一組已加上標籤的資料上進行定型，才能提供結果
* 半監督式 - 演算法可以由訓練人員透過互動式查詢以額外的目標增強，這在定型的初始階段無法使用
* 未監督式 - 演算法不需要定型資料 
* 增強式 - 演算法使用軟體代理程式來判斷特定內容內的理想行為 (通常用於 Robotics)


| 演算法類別| 使用 | 學習類型 | 演算法 |
| --- | --- | --- | -- |
| 分類 | 將人員或事項分類成群組 | 監督式 | 決策樹、羅吉斯迴歸、類神經網路 |
| 叢集 | 將一組範例分割成同質群組 | 未監督式 | K-Means 叢集模型 |
| 模式偵測 | 識別資料中常見的關聯 | 未監督式 | 關聯規則 |
| 迴歸 | 預測數值結果 | 監督式 | 線性迴歸、類神經網路 |
| 增強式 | 判斷機器人的最佳行為 | 增強式 | 蒙特卡羅 (Monte Carlo) 模擬、DeepMind |

## <a name="machine-learning-on-hdinsight"></a>HDInsight 上的機器學習服務

HDInsight 有多個適用於進階分析工作流程的機器學習服務選項：

* [機器學習服務和 Spark](#machine-learning-and-spark)
* [R 和 ML 服務](#r-and-r-server)
* [Azure Machine Learning 和 Hive](#azure-machine-learning-and-hive)
* [Spark 和深度學習](#spark-and-deep-learning)

### <a name="machine-learning-and-spark"></a>機器學習服務和 Spark

[HDInsight Spark](../spark/apache-spark-overview.md) 是 Azure 代管的 [Spark](http://spark.apache.org/) 供應項目方案，是一個使用記憶體內處理來大幅提升巨量資料分析效能的統一、開放原始碼平行資料處理架構。 Spark 處理引擎是專為速度、易用性及精密分析打造的產品。 Spark 的記憶體內分散式計算功能，使其成為機器學習和圖表計算中所使用反覆演算法的絕佳選擇。 

有三個可將演算法模型化功能導入此分散式環境的可調整機器學習服務程式庫：

* [**MLlib**](https://spark.apache.org/docs/latest/ml-guide.html) - MLlib 包含建置在 Spark RDD 上的原始 API。
* [**SparkML**](https://spark.apache.org/docs/1.2.2/ml-guide.html) - SparkML 是較新的套件，可提供建置在 Spark DataFrames 上的較高階 API 來建構 ML 管線。
* [**MMLSpark**](https://github.com/Azure/mmlspark)  - 適用於 Apache Spark 的 Microsoft Machine Learning 程式庫 (MMLSpark) 是設計來提升資料科學家在 Spark 上的生產力，不僅可提高實驗率，還可在非常大型的資料集上運用最先進的機器學習服務技術，包括深度學習。 MMLSpark 程式庫簡化了在 PySpark 中建置模型的常見模型化工作。 

### <a name="r-and-ml-services"></a>R 和 ML 服務

ML 服務為 HDInsight 的一部分，您可以使用現成可與大量資料集和模型搭配使用的 [ML 服務](../r-server/r-server-overview.md)來建立 HDInsight 叢集。 這個新功能可為資料科學家和統計人員提供一個熟悉的 R 介面，此介面可視需要透過 HDInsight 進行調整，而不會產生叢集設定和維護的額外負荷。

### <a name="azure-machine-learning-and-hive"></a>Azure Machine Learning 和 Hive

[Azure Machine Learning Studio](https://studio.azureml.net/) 除了提供完全受管理的服務供您用來將預測性模型部署為可供取用的 Web 服務之外，也提供可建立預測性分析模型的工具。 Azure Machine Learning 提供可在雲端建立完整預測性分析解決方案的工具，可讓您快速建立、測試預測性模型、將預測性模型變成可操作的模型並對其進行管理。 您可以從大型演算法程式庫中選取、使用 Web 型工作室來建置模型，然後將模型輕鬆部署成 Web 服務。

### <a name="spark-and-deep-learning"></a>Spark 和深度學習

[深度學習](https://www.microsoft.com/research/group/dltc/)是機器學習服務的一個分支，使用以人類大腦的生物學程序為靈感來源的*深度類神經網路* (DNN)。 許多研究人員都將深度學習視為一個有前景的人工智慧方法。 深度學習的一些範例包括口語語言翻譯工具、影像辨識系統，以及機器推理。 為了協助推展自己在深度學習方面的工作，Microsoft 開發了免費、容易使用的開放原始碼 [Microsoft Cognitive Toolkit](https://www.microsoft.com/en-us/cognitive-toolkit/)。 各種 Microsoft 產品、世界各地需要大規模部署深度學習的公司，以及對最新演算法和技術感興趣的學生，都廣泛使用此工具組。 

## <a name="scenario---score-images-to-identify-patterns-in-urban-development"></a>案例 - 進行影像評分來識別都市發展模式

我們將檢閱一個使用 HDInsight 的進階分析機器學習服務管線範例。

在此案例中，您將了解如何使用 HDInsight Spark 叢集上的 PySpark，將在深度學習架構 (Microsoft 的 Cognitive Toolkit (CNTK)) 中產生的 DNN 變成可操作，來為儲存在「Azure Blob 儲存體」帳戶中的大型影像集合進行評分。 此方法適用於一般 DNN 使用案例、空拍影像分類，並可用來識別最近的都市發展模式。  您將使用預先定型的影像分類模型。 此模型已在 [CIFAR-10 資料集](https://www.cs.toronto.edu/~kriz/cifar.html)上預先定型，並已套用至 10,000 個保留的影像。

這個進階分析案例中有三個主要工作：

1. 使用 Apache Spark 2.1.0 散發套件來建立 Azure HDInsight Hadoop 叢集。 
2. 執行自訂指令碼，以在 Azure HDInsight Spark 叢集的所有節點上安裝 Microsoft Cognitive Toolkit。 
3. 將預先建置的 Jupyter Notebook 上傳至 HDInsight Spark 叢集，以使用 Spark Python API (PySpark) 將定型的 Microsoft Cognitive Toolkit 深度學習模型套用至「Azure Blob 儲存體帳戶」中的檔案。 

此範例會使用 Alex Krizhevsky、Vinod Nair 及 Geoffrey Hinton 所編譯並散發的 CIFAR-10 影像集。 CIFAR-10 資料集包含 60,000 個分屬 10 個互斥類別的 32×32 彩色影像：

![映像](./media/apache-hadoop-deep-dive-advanced-analytics/ml-images.png)

如需有關此資料集的詳細資料，請參閱 Alex Krizhevsky 的[從小影像學習多層特徵](https://www.cs.toronto.edu/~kriz/learning-features-2009-TR.pdf) \(英文\)。

此影像集已分割成由 50,000 影像組成的定型集和由 10,000 個影像組成的測試集。 第一個集合已用來依循 Cognitive Toolkit GitHub 儲存機制中的[這個教學課程](https://github.com/Microsoft/CNTK/tree/master/Examples/Image/Classification/ResNet)，使用 Microsoft Cognitive Toolkit 將深度達 20 層的卷積剩餘網路 (ResNet) 模型定型。 剩餘的 10,000 個影像則已用來測試模型的正確性。 這正是分散式運算開始發揮作用之處：影像的前處理和評分工作是可高度平行化的工作。 有了已儲存的定型模型在手，我們使用了：

* PySpark 將影像和定型模型散發至叢集的背景工作節點。
* Python 在 HDInsight Spark 叢集的每個節點上進行影像前處理。
* Cognitive Toolkit 在每個節點上載入模型，並對經過前處理的影像進行評分。
* Jupyter Notebook 來執行 PySpark 指令碼、彙總結果，以及使用 [Matplotlib](https://matplotlib.org/) 以視覺方式呈現模型效能。

在一個含有 4 個背景工作節點的叢集上，10,000 個影像的整個前處理/評分工作花費不到 1 分鐘即可完成。 此模型可準確預測大約 9,100 (91%) 個影像的標籤。 混淆矩陣可說明最常見的分類錯誤。 例如，以下矩陣顯示與其他標籤組相比，將狗錯誤標示成貓 (以及將貓錯誤標示成狗) 的發生頻率較高。

![結果](./media/apache-hadoop-deep-dive-advanced-analytics/ml-results.png)

### <a name="try-it-out"></a>立即試用！

請依循[這個教學課程](../spark/apache-spark-microsoft-cognitive-toolkit.md)以端對端方式實作此解決方案：設定一個 HDInsight Spark 叢集、安裝 Cognitive Toolkit，然後執行可為 10,000 CIFAR 影像進行評分的 Jupyter Notebook。

## <a name="next-steps"></a>後續步驟

Hive 和 Azure Machine Learning

* [Hive 和 Azure Machine Learning (端對端)](../../machine-learning/team-data-science-process/hive-walkthrough.md)
* [在 1 TB 資料集上使用 Azure HDInsight Hadoop 叢集](../../machine-learning/team-data-science-process/hive-criteo-walkthrough.md)

Spark 和 MLlib

* [使用 HDInsight 上的 Spark 進行機器學習](../../machine-learning/team-data-science-process/spark-overview.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](../spark/apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](../spark/apache-spark-machine-learning-mllib-ipython.md)

深度學習、Cognitive Toolkit 及其他方法

* [在 Azure HDInsight Spark 上使用 Cognitive Toolkit 和 TensorFlow 以困窘的方式進行平行影像分類](https://blogs.technet.microsoft.com/machinelearning/2017/04/12/embarrassingly-parallel-image-classification-using-cognitive-toolkit-tensorflow-on-azure-hdinsight-spark/)
* [資料科學 Azure 虛擬機器](../../machine-learning/data-science-virtual-machine/overview.md)
* [Azure HDInsight 上的 H2O.ai 簡介](https://azure.microsoft.com/blog/introducing-h2o-ai-with-on-azure-hdinsight-to-bring-the-most-robust-ai-platform-for-enterprises/) \(英文\)
