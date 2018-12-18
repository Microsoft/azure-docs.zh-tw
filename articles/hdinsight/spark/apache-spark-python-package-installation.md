---
title: 指令碼動作 - 在 Azure HDInsight 上搭配 Jupyter 安裝 Python 套件
description: 說明如何使用指令碼動作以設定讓 HDInsight Spark 叢集隨附之 Jupyter Notebook 使用外部 Python 套件的逐步指示。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/09/2018
ms.author: jasonh
ms.openlocfilehash: c8d0b172682654c858a97b4ca2df99ec5079adaa
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041144"
---
# <a name="use-script-action-to-install-external-python-packages-for-jupyter-notebooks-in-apache-spark-clusters-on-hdinsight"></a>使用指令碼動作在 HDInsight Linux 上的 Apache Spark 叢集中安裝 Jupyter Notebook 的外部 Python 封裝
> [!div class="op_single_selector"]
> * [使用資料格魔術](apache-spark-jupyter-notebook-use-external-packages.md)
> * [使用指令碼動作](apache-spark-python-package-installation.md)
>
>

了解如何使用指令碼動作，將 HDInsight (Linux) 上的 Apache Spark 叢集設定為使用外部、社群的提供 **Python** 套件，而不是叢集中現成隨附的套件。

> [!NOTE]
> 您也可以藉由使用 `%%configure` 魔術，設定 Jupyter Notebook 來使用外部的封裝。 如需指示，請參閱[在 HDInsight 上的 Apache Spark 叢集中搭配 Jupyter Notebook 使用外部封裝](apache-spark-jupyter-notebook-use-external-packages.md)。
> 
> 

您可以搜尋[套件索引](https://pypi.python.org/pypi)，以取得可用的套件完整清單。 您也可以從其他來源取得可用套件清單。 例如，您可以安裝經由 [Anaconda](https://docs.continuum.io/anaconda/pkg-docs) 或 [conda-forge](https://conda-forge.org/feedstocks/) 提供的套件。

在本文中，您將學習如何在叢集上使用指令碼動作安裝 [TensorFlow](https://www.tensorflow.org/) 套件，並使用透過 Jupyter Notebook 來使用它。

## <a name="prerequisites"></a>必要條件
您必須滿足以下條件：

* Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

   > [!NOTE]
   > 如果您在 HDInsight Linux 上還沒有 Spark 叢集，您可以在叢集建立期間執行指令碼動作。 瀏覽有關於[如何使用自訂指令碼動作](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux)的文件。
   > 
   > 

## <a name="use-external-packages-with-jupyter-notebooks"></a>搭配 Jupyter Notebook 使用外部套件

1. 在 [Azure 入口網站](https://portal.azure.com/)的開始面板中，按一下您的 Spark 叢集磚 (如果您已將其釘選到開始面板)。 您也可以按一下 [瀏覽全部] > [HDInsight 叢集] 來瀏覽至您的叢集。   

2. 從 Spark 叢集刀鋒視窗的左窗格中，按一下 [指令碼動作]。 執行自訂動作，將 TensorFlow 安裝在前端節點和背景工作角色節點。 您可以在以下網址參考 Bash 指令碼： https://hdiconfigactions.blob.core.windows.net/linuxtensorflow/tensorflowinstall.sh 瀏覽有關於[如何使用自訂指令碼動作](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux)的文件。

   > [!NOTE]
   > 叢集中有兩個 Python 安裝。 Spark 會使用位於 `/usr/bin/anaconda/bin` 的 Anaconda Python 安裝。 在您的自訂動作中，透過 `/usr/bin/anaconda/bin/pip` 和 `/usr/bin/anaconda/bin/conda` 來參考該安裝。
   > 
   > 

3. 開啟 PySpark Jupyter Notebook

    ![建立新的 Jupyter Notebook](./media/apache-spark-python-package-installation/hdinsight-spark-create-notebook.png "建立新的 Jupyter Notebook")

4. 系統隨即會建立新 Notebook，並以 Untitled.pynb 的名稱開啟。 在頂端按一下 Notebook 名稱，然後輸入好記的名稱。

    ![提供 Notebook 的名稱](./media/apache-spark-python-package-installation/hdinsight-spark-name-notebook.png "提供 Notebook 的名稱")

5. 現在，您將 `import tensorflow` 並執行 hello world 範例。 

    要複製的程式碼︰

        import tensorflow as tf
        hello = tf.constant('Hello, TensorFlow!')
        sess = tf.Session()
        print(sess.run(hello))

    結果如下所示：
    
    ![TensorFlow 程式碼執行](./media/apache-spark-python-package-installation/execution.png "執行 TensorFlow 程式碼")

## <a name="seealso"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [在 HDInsight 上的 Apache Spark 叢集中搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
