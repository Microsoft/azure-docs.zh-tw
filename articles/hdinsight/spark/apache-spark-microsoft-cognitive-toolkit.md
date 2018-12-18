---
title: Microsoft Cognitive Toolkit 搭配 Azure HDInsight Spark 以深入學習
description: 了解如何使用 Azure HDInsight Spark 叢集中的 Spark Python API，將定型的 Microsoft 辨識工具組深入學習模型套用至資料集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: jasonh
ms.openlocfilehash: cc36c68f4867b9b450703c881a13a65f17ebad4b
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049446"
---
# <a name="use-microsoft-cognitive-toolkit-deep-learning-model-with-azure-hdinsight-spark-cluster"></a>使用 Microsoft 辨識工具組深入了解模型與 Azure HDInsight Spark 叢集

在本文中，您會執行下列步驟。

1. 執行自訂指令碼，在 Azure HDInsight Spark 叢集上安裝 Microsoft 辨識工具組。

2. 將 Jupyter Notebook 上傳至 Spark 叢集，以了解如何使用 [Spark Python API (PySpark)](https://spark.apache.org/docs/0.9.0/python-programming-guide.html)，將定型的 Microsoft 辨識工具組深入學習模型套用至 Azure Blob 儲存體帳戶中的檔案

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 開始進行本教學課程之前，您必須擁有 Azure 訂用帳戶。 請參閱[立即建立免費的 Azure 帳戶](https://azure.microsoft.com/free)。

* **Azure HDInsight Spark 叢集**。 針對本文，建立 Spark 2.0 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="how-does-this-solution-flow"></a>此解決方案的流程為何？

此解決方案可分為本文章與您上傳作為本教學課程一部分的 Jupyter Notebook。 在本文中，您必須完成下列步驟：

* 在 HDInsight Spark 叢集上執行指令碼動作來安裝 Microsoft 辨識工具組和 Python 套件。
* 將 Jupyter Notebook 上傳至執行此解決方案的 HDInsight Spark 叢集。

Jupyter Notebook 中涵蓋下列剩餘步驟。

- 將範例映像載入 Spark 復原分散式資料集或 RDD
   - 載入模組並定義預設值
   - 在 Spark 叢集上的本機下載資料集
   - 將資料集轉換成 RDD
- 使用定型的辨識工具組模型對映像進行評分
   - 下載定型的辨識工具組模型至 Spark 叢集
   - 定義可供背景工作節點使用的函式
   - 在背景工作節點上對映像進行評分
   - 評估模型的精確度


## <a name="install-microsoft-cognitive-toolkit"></a>安裝 Microsoft 辨識工具組

您可以使用指令碼動作在 Spark 叢集上安裝 Microsoft 辨識工具組。 指令碼動作會使用自訂指令碼在叢集上安裝不是預設可用的元件。 您可以透過 Azure 入口網站使用自訂指令碼，使用 HDInsight .NET SDK 或 Azure PowerShell 都可以。 您也可以使用指令碼，在叢集建立期間安裝工具組，或在叢集已啟動並執行之後加以安裝。 

在本文中，我們在建立叢集之後，使用入口網站來安裝工具組。 如需執行自訂指令碼的其他方式，請參閱[使用指令碼動作自訂 HDInsight 叢集](../hdinsight-hadoop-customize-cluster-linux.md)。

### <a name="using-the-azure-portal"></a>使用 Azure 入口網站

如需有關如何使用 Azure 入口網站來執行指令碼動作的指示，請參閱[使用指令碼動作來自訂 HDInsight 叢集](../hdinsight-hadoop-customize-cluster-linux.md#use-a-script-action-during-cluster-creation)。 確定您提供下列輸入來安裝 Microsoft 辨識工具組。

* 提供指令碼動作名稱的值。

* 對於 **Bash 指令碼 URI**，輸入 `https://raw.githubusercontent.com/Azure-Samples/hdinsight-pyspark-cntk-integration/master/cntk-install.sh`。

* 確定您只會在前端和背景工作節點上執行指令碼，並清除所有其他核取方塊。

* 按一下頁面底部的 [新增] 。

## <a name="upload-the-jupyter-notebook-to-azure-hdinsight-spark-cluster"></a>將 Jupyter Notebook 上傳至 Azure HDInsight Spark 叢集

若要使用 Microsoft 辨識工具組搭配 Azure HDInsight Spark 叢集，您必須將 Jupyter Notebook **CNTK_model_scoring_on_Spark_walkthrough.ipynb** 載入至 Azure HDInsight Spark 叢集。 在 GitHub 上也可取得此 Notebook：[https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration)。

1. 複製 GitHub 存放庫 [https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration)。 如需複製的指示，請參閱[複製存放庫](https://help.github.com/articles/cloning-a-repository/)。

2. 從 Azure 入口網站開啟您已佈建的 Spark 叢集刀鋒視窗，按一下 [叢集儀表板]，然後按一下 [Jupyter Notebook]。

    您也可以移至 URL `https://<clustername>.azurehdinsight.net/jupyter/` 啟動 Jupyter Notebook。 將 \<clustername> 取代為 HDInsight 叢集的名稱。

3. 從 Jupyter Notebook 中，於右上角按一下 [上傳]，然後瀏覽至您複製 GitHub 存放庫的位置。

    ![將 Jupyter Notebook 上傳至 Azure HDInsight Spark 叢集](./media/apache-spark-microsoft-cognitive-toolkit/hdinsight-microsoft-cognitive-toolkit-load-jupyter-notebook.png "將 Jupyter Notebook 上傳至 Azure HDInsight Spark 叢集")

4. 再次按一下 [上傳]。

5. 將 Notebook 上傳之後，按一下 Notebook 名稱，然後遵循 Notebook 本身有關如何載入資料集的指示，並執行教學課程中。

## <a name="see-also"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)
* [HDInsight 中使用 Spark 的 Application Insight 遙測資料分析](apache-spark-analyze-application-insight-logs.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)

[hdinsight-versions]: hdinsight-component-versioning.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[azure-create-storageaccount]:../../storage/common/storage-create-storage-account.md
