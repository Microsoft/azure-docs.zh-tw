---
title: 在 Azure 上使用 PySpark 和 Scala 的 HDInsight Spark 逐步解說 | Microsoft Docs
description: 以 Team Data Science Process 為例，逐步解說如何在 Azure HDInsight Spark 上使用 PySpark 和 Scala 來執行預測性分析。
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
ms.date: 09/04/2017
ms.author: deguhath
ms.openlocfilehash: 2fd6d556e908c47baf3c58c46172c0684b66cdd9
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394244"
---
# <a name="hdinsight-spark-data-science-walkthroughs-using-pyspark-and-scala-on-azure"></a>在 Azure 上使用 PySpark 和 Scala 的 HDInsight Spark 資料科學逐步解說

這些逐步解說會在 Azure Spark 叢集上使用 PySpark 和 Scala 來執行預測性分析。 其遵循 Team Data Science Process 中所述的步驟。 如需 Team Data Science Process 的概觀，請參閱 [Data Science Process](overview.md)。 如需 Spark on HDInsight 的概觀，請參閱 [Spark on HDInsight 簡介](../../hdinsight/spark/apache-spark-overview.md)。

執行 Team Data Science Process 的其他資料科學逐步解說是依據它們使用的**平台**來歸納整理。 如需這些範例的列述，請參閱[執行 Team Data Science Process 的逐步解說](walkthroughs.md)。

## <a name="predict-taxi-tips-using-pyspark-on-azure-spark"></a>在 Azure Spark 上使用 PySpark 來預測計程車小費

[在 Azure HDInsight 上使用 Spark](spark-overview.md) 逐步解說會使用紐約計程車的資料來預測乘客是否會付小費，以及預期會給予的金額範圍。 它會在採用 [Azure HDInsight Spark 叢集](https://azure.microsoft.com/services/hdinsight/)的案例中使用 Team Data Science Process，以對公開提供使用的 NYC 計程車車程和車資資料集的資料進行儲存、探索和特徵工程設計。 此概觀主題會為您設定本逐步解說的其餘部分所使用的 HDInsight Spark 叢集和 Jupyter PySpark Notebook。 這些 Notebook 會示範如何瀏覽資料、建立和取用模型。 進階的資料探索和模型化 Notebook 顯示如何包括交叉驗證、超參數清除和模型評估。

### <a name="data-exploration-and-modeling-with-spark"></a>使用 Spark 進行資料探索和模型化 
遵循[使用 Spark MLlib 工具組來建立資料的二進位分類和迴歸模型](spark-data-exploration-modeling.md)主題的內容，來探索資料集，以及建立、評分、評估 Machine Learning 模型。

### <a name="model-consumption"></a>模型耗用量
若要瞭解如何評分本主題中所建立的分類和迴歸模型，請參閱[評分及評估 Spark 建置機器學習服務模型](spark-model-consumption.md)。

### <a name="cross-validation-and-hyperparameter-sweeping"></a>交叉驗證和超參數掃掠
如需如何使用交叉驗證和超參數掃掠訓練模型的相關資訊，請參閱[使用 Spark 進階資料探索和模型化](spark-advanced-data-exploration-modeling.md)。


## <a name="predict-taxi-tips-using-scala-on-azure-spark"></a>在 Azure Spark 上使用 Scala 來預測計程車小費

[在 Azure 上使用 Scala 與 Spark](scala-walkthrough.md) 逐步解說會使用紐約計程車的資料來預測乘客是否會付小費，以及預期會給予的金額範圍。 它會說明如何使用 Scala 搭配 Spark 機器學習程式庫 (MLlib) 和 Azure HDInsight Spark 叢集上的 SparkML 套件，處理受監督的機器學習工作。 它會引導您進行構成 [資料科學程序](http://aka.ms/datascienceprocess)的各項工作︰資料擷取和探索、視覺化、特徵設計、模型化和模型取用。 建立的模型包括羅吉斯和線性迴歸、隨機樹系和漸層停駐推進式決策樹。


## <a name="next-steps"></a>後續步驟

如需組成 Team Data Science Process 之主要元件的討論，請參閱 [Team Data Science Process 概觀](overview.md)。

如需您可以用來結構化資料科學專案之 Team Data Science Process 生命週期的討論，請參閱 [Team Data Science Process 生命週期](lifecycle.md)。 生命週期會概述專案在執行時通常會遵循之從開始到完成的步驟。 

