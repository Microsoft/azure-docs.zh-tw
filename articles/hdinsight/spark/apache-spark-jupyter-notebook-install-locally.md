---
title: 在本機安裝 Jupyter 並連線到 Azure HDInsight 中的 Spark
description: 了解如何在電腦本機安裝 Jupyter Notebook，並連線到 Apache Spark 叢集。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: jasonh
ms.openlocfilehash: 83e9596f37850ef5b26b530cd4424a024355fc8a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042550"
---
# <a name="install-jupyter-notebook-on-your-computer-and-connect-to-apache-spark-on-hdinsight"></a>在電腦上安裝 Jupyter Notebook，並連線到 HDInsight 上的 Apache Spark

在這篇文章中，您會了解如何搭配含有 Spark magic 的自訂 PySpark (適用於 Python) 和 Spark (適用於 Scala) 核心來安裝 Jupyter 筆記本，然後將筆記本連接到 HDInsight 叢集。 在您的本機電腦上安裝 Jupyter 可以有數種原因，而且也會面臨數種挑戰。 如需詳細資訊，請參閱本文章結尾的[為什麼我應該在我的電腦上安裝 Jupyter](#why-should-i-install-jupyter-on-my-computer) 一節。

在電腦上安裝 Jupyter 和 Spark magic 涉及三個主要步驟。

* 安裝 Jupyter Notebook
* 安裝含有 Spark magic 的 PySpark 和 Spark 核心
* 設定 Spark magic 以存取 HDInsight 上的 Spark 叢集

如需有關可供 HDInsight 叢集之相關 Jupyter Notebook 使用的自訂核心和 Spark magic 的詳細資訊，請參閱 [HDInsight 上的 Apache Spark Linux 叢集可供 Jupyter Notebook 使用的核心](apache-spark-jupyter-notebook-kernels.md)。

## <a name="prerequisites"></a>必要條件
此處所列的必要條件不是針對安裝 Jupyter。 這些是用來在安裝 Notebook 之後將 Jupyter Notebook 連接到 HDInsight 叢集。

* Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="install-jupyter-notebook-on-your-computer"></a>在電腦上安裝 Jupyter Notebook

您必須先安裝 Python，才能安裝 Jupyter Notebook。 Python 和 Jupyter 皆為 [Anaconda 發行版本](https://www.continuum.io/downloads)的一部分。 當您安裝 Anaconda 時，會安裝某個 Python 發行版本。 安裝 Anaconda 之後，您便可以執行適當的命令來新增 Jupyter 安裝。

1. 下載您平台適用的 [Anaconda 安裝程式](https://www.continuum.io/downloads) ，然後執行安裝程式。 執行安裝精靈時，請確定您選取將 Anaconda 新增至 PATH 變數的選項。
1. 執行下列命令來安裝 Jupyter。

        conda install jupyter

    如需有關安裝 Jupyter 的詳細資訊，請參閱[使用 Anaconda 來安裝 Jupyter](http://jupyter.readthedocs.io/en/latest/install.html) \(英文\)。

## <a name="install-the-kernels-and-spark-magic"></a>安裝核心和 Spark magic

如需有關如何安裝 Spark magic、PySpark 和 Spark 核心的指示，請遵循 GitHub 上 [sparkmagic 文件](https://github.com/jupyter-incubator/sparkmagic#installation)中的安裝指示。 Spark magic 文件中的第一個步驟會要求您安裝 Spark magic。 根據您將要連線的 HDInsight 叢集版本，將連結中的第一個步驟以下列命令取代。 之後，遵循 Spark magic 文件中的其餘步驟。 如果您想要安裝其他核心，則必須執行 Spark magic 安裝指示小節中的步驟 3。

* 若為叢集 3.4 版，請執行 `pip install sparkmagic==0.2.3` 來安裝 sparkmagic 0.2.3

* 若為叢集 3.5 版和 3.6 版，請執行 `pip install sparkmagic==0.11.2` 來安裝 sparkmagic 0.11.2

## <a name="configure-spark-magic-to-connect-to-hdinsight-spark-cluster"></a>設定 Spark magic 以連線到 HDInsight Spark 叢集

在本節中，您會設定稍早安裝的 Spark magic，以連線到您必須已在 Azure HDInsight 中建立的 Apache Spark 叢集。

1. Jupyter 組態資訊通常儲存在使用者主目錄中。 若要在任何作業系統平台上找出您的主目錄，輸入下列命令。

    啟動 Python 殼層。 在命令視窗中，輸入下列命令：

        python

    在 Python 殼層中，輸入下列命令來找出主目錄。

        import os
        print(os.path.expanduser('~'))

1. 瀏覽至主目錄，然後建立一個叫做 **.sparkmagic** 的資料夾 (如果尚未存在)。
1. 在該資料夾中，建立一個叫做 **config.json** 的檔案，然後在檔案中新增下列 JSON 程式碼片段。

        {
          "kernel_python_credentials" : {
            "username": "{USERNAME}",
            "base64_password": "{BASE64ENCODEDPASSWORD}",
            "url": "https://{CLUSTERDNSNAME}.azurehdinsight.net/livy"
          },
          "kernel_scala_credentials" : {
            "username": "{USERNAME}",
            "base64_password": "{BASE64ENCODEDPASSWORD}",
            "url": "https://{CLUSTERDNSNAME}.azurehdinsight.net/livy"
          }
        }

1. 以適當的值替代 **{USERNAME}**、**{CLUSTERDNSNAME}** 和 **{BASE64ENCODEDPASSWORD}**。 您可以在慣用的程式設計語言中或在線上，使用一些公用程式來產生 base64 編碼的密碼，以做為您的實際密碼。

1. 在 `config.json` 中設定正確的活動訊號設定。 您應該依照加入稍早的 `kernel_python_credentials` 和 `kernel_scala_credentials` 程式碼片段，在相同層級新增這些設定。 如需有關加入活動訊號設定的方法與位置範例，請參閱此[範例 config.json (sample config.json)](https://github.com/jupyter-incubator/sparkmagic/blob/master/sparkmagic/example_config.json)。

    * 若為 `sparkmagic 0.2.3` (叢集 3.4 版)，請加入︰

            "should_heartbeat": true,
            "heartbeat_refresh_seconds": 5,
            "heartbeat_retry_seconds": 1

    * 若為 `sparkmagic 0.11.2` (叢集 3.5 版和 3.6 版)，請加入︰

            "heartbeat_refresh_seconds": 5,
            "livy_server_heartbeat_timeout_seconds": 60,
            "heartbeat_retry_seconds": 1

    >[!TIP]
    >傳送活動訊號可確保不會流失工作階段。 當電腦進入睡眠或已關機時，則不會傳送活動訊號，導致工作階段被清除。 若為叢集 3.4 版，如果想要停用此行為，您可以從 Ambari UI 將 Livy 組態 `livy.server.interactive.heartbeat.timeout` 設定為 `0`。 若為叢集 3.5 版，如果您未設定上述的 3.5 組態，則不會刪除工作階段。

1. 啟動 Jupyter。 從命令提示字元使用下列命令。

        jupyter notebook

1. 確認您可以使用 Jupyter Notebook 來連接到叢集，並且可以使用核心隨附的 Spark magic。 請執行下列步驟：

    a. 建立新的 Notebook。 從右下角，按一下 [新增]。 您應該會看到預設核心 **Python2**，以及您安裝的兩個新核心 **PySpark** 和 **Spark**。 按一下 [PySpark] 。

    ![Jupyter Notebook 中的核心](./media/apache-spark-jupyter-notebook-install-locally/jupyter-kernels.png "Jupyter Notebook 中的核心")

    b. 執行下列程式碼片段。

        %%sql
        SELECT * FROM hivesampletable LIMIT 5

    如果您可以順利擷取輸出，即表示已測試您對 HDInsight 叢集的連線。

    >[!TIP]
    >如果您想要更新 Notebook 組態以連接到不同的叢集，請將 config.json 更新成一組新的值，如上述的步驟 3 所示。

## <a name="why-should-i-install-jupyter-on-my-computer"></a>為什麼我應該在我的電腦上安裝 Jupyter？
您想要在您的電腦上安裝 Jupyter，然後將其連接至 HDInsight 上的 Spark 叢集有數種原因。

* 雖然 Azure HDInsight 的 Spark 叢集上已經有 Jupyter Notebook 可用，但在電腦上安裝 Jupyter 可讓您選擇在本機建立 Notebook、針對正在執行的叢集測試您的應用程式，然後將 Notebook 上傳到叢集。 若要將 Notebook 上傳到叢集，您可以使用在叢集上執行的 Jupyter Notebook 來上傳它們，或將它們儲存到與叢集關聯之儲存體帳戶的 [/HdiNotebooks] 資料夾中。 如需有關 Notebook 在叢集上的儲存方式詳細資訊，請參閱 [Jupyter Notebook 會儲存在哪裡](apache-spark-jupyter-notebook-kernels.md#where-are-the-notebooks-stored)？
* 使用本機可用的 Notebook，您可以根據您的應用程式需求，連接至不同的 Spark 叢集。
* 您可以使用 GitHub 實作來源控制系統，並且具有 Notebook 的版本控制。 您也可以有共同作業環境，其中多個使用者可以使用相同的 Notebook。
* 您甚至不需要啟用叢集，即可在本機使用 Notebook。 您只需要叢集針對 Notebook 進行測試，不必以手動方式管理您的 Notebook 或開發環境。
* 設定自己的本機開發環境比設定叢集的 Jupyter 安裝更容易。  您可以利用您在本機安裝的所有軟體，而不需要設定一或多個遠端叢集。

> [!WARNING]
> 將 Jupyter 安裝在本機電腦上，多個使用者可以同時在相同的 Spark 叢集上執行相同的 Notebook。 在這種情況下，會建立多個 Livy 工作階段。 如果您遇到問題，而且想要偵錯，則追蹤哪個 Livy 工作階段屬於哪個使用者會是複雜的工作。
>
>

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
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
