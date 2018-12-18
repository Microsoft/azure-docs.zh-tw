---
title: 在 Azure 上使用 Hive 的 HDInsight Hadoop 資料科學逐步解說 | Microsoft Docs
description: 以 Team Data Science Process 為例，逐步解說如何在 Azure HDInsight Hadoop 上使用 Hive 來執行預測性分析。
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
ms.openlocfilehash: 2aa64d6dc144e85e1a0f03b8548fa47dba91a2eb
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393154"
---
# <a name="hdinsight-hadoop-data-science-walkthroughs-using-hive-on-azure"></a>在 Azure 上使用 Hive 的 HDInsight Hadoop 資料科學逐步解說 

這些逐步解說會使用 Hive 搭配 HDInsight Hadoop 叢集來執行預測性分析。 其遵循 Team Data Science Process 中所述的步驟。 如需 Team Data Science Process 的概觀，請參閱 [Data Science Process](overview.md)。 如需 Azure HDInsight 的簡介，請參閱 [Azure HDInsight、Hadoop 技術堆疊和 Hadoop 叢集簡介](../../hdinsight/hadoop/apache-hadoop-introduction.md)。

執行 Team Data Science Process 的其他資料科學逐步解說是依據它們使用的**平台**來歸納整理。 如需這些範例的列述，請參閱[執行 Team Data Science Process 的逐步解說](walkthroughs.md)。


## <a name="predict-taxi-tips-using-hive-with-hdinsight-hadoop"></a>使用 Hive 搭配 HDInsight Hadoop 來預測計程車小費

[使用 HDInsight Hadoop 叢集](hive-walkthrough.md)逐步解說會使用紐約計程車行的資料來預測： 

- 是否給予小費 
- 小費金額的分佈

此案例是透過 Hive 搭配 [Azure HDInsight Hadoop 叢集](https://azure.microsoft.com/services/hdinsight/)來實作。 您將了解如何對公開使用之 NYC 計程車車程和車資資料集中的資料，進行儲存、探索及特性工程設計。 您也可以使用 Azure Machine Learning 來建置及部署模型。

## <a name="predict-advertisement-clicks-using-hive-with-hdinsight-hadoop"></a>使用 Hive 搭配 HDInsight Hadoop 來預測廣告點擊數

[在 1 TB 資料集上使用 Azure HDInsight Hadoop 叢集](hive-criteo-walkthrough.md)逐步解說會使用公開使用的 [Criteo](http://labs.criteo.com/downloads/download-terabyte-click-logs/) 點擊資料集，來預測是否給予小費及預期的金額範圍。 此案例是透過 Hive 搭配 [Azure HDInsight Hadoop 叢集](https://azure.microsoft.com/services/hdinsight/)來實作，以對資料進行儲存、探索、特性工程設計及縮減取樣。 它使用 Azure Machine Learning 來建立、定型二元分類模型並對其進行評分，以預測使用者是否點擊廣告。 此逐步解說最後會示範如何將其中一個模型發佈為 Web 服務。


## <a name="next-steps"></a>後續步驟

如需組成 Team Data Science Process 之主要元件的討論，請參閱 [Team Data Science Process 概觀](overview.md)。

如需您可以用來結構化資料科學專案之 Team Data Science Process 生命週期的討論，請參閱 [Team Data Science Process 生命週期](lifecycle.md)。 生命週期會概述專案在執行時通常會遵循之從開始到完成的步驟。 

