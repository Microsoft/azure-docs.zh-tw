---
title: 在 Azure HDInsight 上管理 Apache Spark 叢集的資源
description: 了解如何在 Azure HDInsight 上管理 Spark 叢集的資源，以便提升效能。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: jasonh
ms.openlocfilehash: d7395231662d79d284bdf061e651602dea392c28
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051493"
---
# <a name="manage-resources-for-apache-spark-cluster-on-azure-hdinsight"></a>在 Azure HDInsight 上管理 Apache Spark 叢集的資源 

您將學習存取如 Ambari UI、YARN UI，以及與您 Spark 叢集相關聯的 Spark 歷程記錄伺服器等介面的方式，以及如何調整叢集設定以取得最佳效能。

**必要條件：**

* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="open-the-ambari-web-ui"></a>開啟 Ambari Web UI

Apache Ambari 可用來監視叢集和進行設定變更。 如需詳細資訊，請參閱[使用 Azure 入口網站管理 HDInsight 中的 Hadoop 叢集](../hdinsight-administer-use-portal-linux.md#open-the-ambari-web-ui)

## <a name="open-the-spark-history-server"></a>開啟 Spark 歷程記錄伺服器

「Spark 記錄伺服器」是已完成和執行中 Spark 應用程式的 Web UI。 它是 Spark Web UI 的延伸。

**開啟 Spark 記錄伺服器 Web UI**

1. 從 [Azure 入口網站](https://portal.azure.com/)，開啟 Spark 叢集。 如需詳細資訊，請參閱[列出和顯示叢集](../hdinsight-administer-use-portal-linux.md#list-and-show-clusters)。
2. 從 [快速連結]，按一下 [叢集儀表板]，然後按一下 [Spark 記錄伺服器]

    ![Spark 歷程記錄伺服器](./media/apache-spark-resource-manager/launch-history-server.png "Spark 歷程記錄伺服器")

    出現提示時，輸入 Spark 叢集的系統管理員認證。 您也可以瀏覽至以下 URL 來開啟「Spark 記錄伺服器」：

    ```
    https://<ClusterName>.azurehdinsight.net/sparkhistory
    ```

    將 <ClusterName> 取代為您的 Spark 叢集名稱。

「Spark 記錄伺服器」Web UI 看起來像這樣：

![HDInsight Spark 記錄伺服器](./media/apache-spark-resource-manager/hdinsight-spark-history-server.png)

## <a name="open-the-yarn-ui"></a>開啟 Yarn UI
您可以使用 YARN UI 來監視目前在 Spark 叢集上執行的應用程式。

1. 從 [Azure 入口網站](https://portal.azure.com/)，開啟 Spark 叢集。 如需詳細資訊，請參閱[列出和顯示叢集](../hdinsight-administer-use-portal-linux.md#list-and-show-clusters)。
2. 從 [快速連結]，按一下 [叢集儀表板]，然後按一下 [YARN]。

    ![啟動 YARN UI](./media/apache-spark-resource-manager/launch-yarn-ui.png)

   > [!TIP]
   > 或者，您也可以從 Ambari UI 啟動 YARN UI。 若要啟動 Ambari UI，請按一下 [叢集儀表板]，然後按一下 [HDInsight 叢集儀表板]。 從 Ambari UI 中，依序按一下 [YARN]、[快速連結]、作用中的 Resource Manager，以及 [Resource Manager UI]。
   >
   >

## <a name="optimize-clusters-for-spark-applications"></a>針對 Spark 應用程式進行叢集最佳化
根據應用程式需求，可用於 Spark 組態的三個主要參數為 `spark.executor.instances`、`spark.executor.cores` 和 `spark.executor.memory`。 執行程式是針對 Spark 應用程式啟動的程序。 它會在背景工作角色節點上執行，並負責執行應用程式的工作。 執行程式的預設數目和每個叢集的執行程式大小，是根據背景工作角色節點數目和背景工作角色節點大小計算。 這項資訊會儲存在叢集前端節點上的 `spark-defaults.conf`。

這三個組態參數可以在叢集層級設定 (適用於在叢集執行的所有應用程式)，或者也可以針對每個個別應用程式指定。

### <a name="change-the-parameters-using-ambari-ui"></a>使用 Ambari UI 變更參數
1. 從 Ambari UI 按一下 [Spark]、按一下 [設定]，然後展開 [自訂 spark-defaults]。

    ![使用 Ambari UI 設定參數](./media/apache-spark-resource-manager/set-parameters-using-ambari.png)
2. 預設值適用於在叢集上同時執行 4 個 Spark 應用程式。 您可以從使用者介面變更這些值，如以下螢幕擷取畫面所示：

    ![使用 Ambari UI 設定參數](./media/apache-spark-resource-manager/set-executor-parameters.png)
3. 按一下 [儲存]  以儲存組態變更。 在頁面頂端，系統會提示您重新啟動所有受影響的服務。 按一下 [重新啟動] 。

    ![重新啟動服務](./media/apache-spark-resource-manager/restart-services.png)

### <a name="change-the-parameters-for-an-application-running-in-jupyter-notebook"></a>變更在 Jupyter Notebook 中執行的應用程式的參數
對於在 Jupyter Notebook 中執行的應用程式，您可以使用 `%%configure` magic 進行組態變更。 在理想情況下，您必須在應用程式開頭進行此類變更，才能執行您的第一個程式碼儲存格。 這樣可確保組態會在 Livy 工作階段建立時套用至其中。 如果您想要變更應用程式中稍後的階段的組態，您必須使用 `-f` 參數。 不過，這麼做會讓應用程式中的所有進度遺失。

下列程式碼片段說明如何變更在 Jupyter 中執行之應用程式的設定。

    %%configure
    {"executorMemory": "3072M", "executorCores": 4, "numExecutors":10}

組態參數必須以 JSON 字串傳遞，且必須在 magic 之後的下一行，如範例資料行中所示。

### <a name="change-the-parameters-for-an-application-submitted-using-spark-submit"></a>使用 spark-submit 變更已提交應用程式的參數
下列命令是如何針對使用 `spark-submit`提交的批次應用程式變更組態參數的範例。

    spark-submit --class <the application class to execute> --executor-memory 3072M --executor-cores 4 –-num-executors 10 <location of application jar file> <application parameters>

### <a name="change-the-parameters-for-an-application-submitted-using-curl"></a>使用 cURL 變更已提交應用程式的參數
下列命令是一個範例，說明如何為使用 cURL 提交的批次應用程式變更設定參數。

    curl -k -v -H 'Content-Type: application/json' -X POST -d '{"file":"<location of application jar file>", "className":"<the application class to execute>", "args":[<application parameters>], "numExecutors":10, "executorMemory":"2G", "executorCores":5' localhost:8998/batches

### <a name="change-these-parameters-on-a-spark-thrift-server"></a>在 Spark Thrift 伺服器上變更這些參數
Spark Thrift 伺服器提供對 Spark 叢集的 JDBC/ODBC 存取，並且用來服務 Spark SQL 查詢。 Power BI、Tableau 等工具 使用 ODBC 通訊協定與 Spark Thrift 伺服器通訊，將 Spark SQL 查詢當作 Spark 應用程式執行。 建立 Spark 叢集時，會啟動 Spark Thrift 伺服器的兩個執行個體，每個前端節點上一個執行個體。 每個 Spark Thrift 伺服器會顯示為 YARN UI 中的 Spark 應用程式。

Spark Thrift 伺服器會使用 Spark 動態執行程式配置，因此不會使用 `spark.executor.instances` 。 而是 Spark Thrift 伺服器會使用 `spark.dynamicAllocation.minExecutors` 和 `spark.dynamicAllocation.maxExecutors` 來指定執行程式計數。 會使用組態參數 `spark.executor.cores` 和 `spark.executor.memory` 以修改執行程式大小。 您可以變更這些參數，如下列步驟所示：

* 展開**進階 spark-thrift-sparkconf** 類別以更新參數 `spark.dynamicAllocation.minExecutors`、`spark.dynamicAllocation.maxExecutors` 和 `spark.executor.memory`。

    ![設定 Spark Thrift 伺服器](./media/apache-spark-resource-manager/spark-thrift-server-1.png)    
* 展開**自訂 spark-thrift-sparkconf** 類別以更新參數 `spark.executor.cores`。

    ![設定 Spark Thrift 伺服器](./media/apache-spark-resource-manager/spark-thrift-server-2.png)

### <a name="change-the-driver-memory-of-the-spark-thrift-server"></a>變更 Spark Thrift 伺服器的驅動程式記憶體
「Spark Thrift 伺服器」驅動程式記憶體是設定為前端節點 RAM 大小的 25%，其中假設前端節點的 RAM 大小總計大於 14 GB。 您可以使用 Ambari UI 來變更驅動程式記憶體設定，如以下螢幕擷取畫面所示：

* 從 Ambari UI 按一下 [Spark]、按一下 [設定]、展開 [進階 spark-env]，然後提供 [spark_thrift_cmd_opts] 的值。

    ![設定 Spark Thrift 伺服器 RAM](./media/apache-spark-resource-manager/spark-thrift-server-ram.png)

## <a name="reclaim-spark-cluster-resources"></a>回收 Spark 叢集資源
由於會進行 Spark 動態配置，因此 Thrift 伺服器所使用的唯一資源是兩個應用程式主機的資源。 若要回收這些資源，您必須停止在叢集上執行的「Thrift 伺服器」服務。

1. 從 Ambari UI 的左窗格中，按一下 [Spark] 。
2. 在下一個頁面上，按一下 [Spark Thrift 伺服器] 。

    ![重新啟動 Thrift 伺服器](./media/apache-spark-resource-manager/restart-thrift-server-1.png)
3. 您應該會看到 Spark Thrift 伺服器正在上面執行的兩個前端節點。 按一下其中一個前端節點。

    ![重新啟動 Thrift 伺服器](./media/apache-spark-resource-manager/restart-thrift-server-2.png)
4. 下一個頁面列出在該前端節點上執行的所有服務。 從清單中，按一下 Spark Thrift 伺服器旁邊的下拉式按鈕，然後按一下 [停止] 。

    ![重新啟動 Thrift 伺服器](./media/apache-spark-resource-manager/restart-thrift-server-3.png)
5. 對其他前端節點重複這些步驟。

## <a name="restart-the-jupyter-service"></a>重新啟動 Jupyter 服務
啟動 Ambari Web UI，如本文開頭所示。 從左側導覽窗格，依序按一下 [Jupyter]、[服務動作] 和 [全部重新啟動]。 這會在所有前端節點上啟動 Jupyter 服務。

![重新啟動 Jupyter](./media/apache-spark-resource-manager/restart-jupyter.png "重新啟動 Jupyter")

## <a name="monitor-resources"></a>監視資源
啟動 Yarn UI，如本文開頭所示。 在螢幕頂端的叢集計量資料表中，檢查[使用的記憶體] 的值和 [記憶體總計] 資料行。 如果這兩個值相當接近，可能會沒有足夠的資源來啟動下一個應用程式。 這同樣適用於 [使用的 VCores] 和 [VCores 總計] 資料行。 此外，在主要檢視中，如果應用程式一直維持在 [已接受] 狀態並未轉換成 [執行中] 或 [失敗] 狀態時，這也可能是表示其未取得足夠的資源來啟動。

![資源限制](./media/apache-spark-resource-manager/resource-limit.png "資源限制")

## <a name="kill-running-applications"></a>終止執行中的應用程式
1. 在 Yarn UI 中，從左窗格中，按一下 [執行中]。 從執行中應用程式的清單，決定要終止的應用程式，然後按一下 [識別碼]。

    ![終止 App1](./media/apache-spark-resource-manager/kill-app1.png "終止 App1")

2. 按一下右上角的 [終止應用程式]，然後按一下 [確定]。

    ![終止 App2](./media/apache-spark-resource-manager/kill-app2.png "終止 App2")

## <a name="see-also"></a>另請參閱
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)

### <a name="for-data-analysts"></a>針對資料分析師

* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)
* [HDInsight 中使用 Spark 的 Application Insight 遙測資料分析](apache-spark-analyze-application-insight-logs.md)
* [在 Azure HDInsight Spark 上使用 Caffe 進行分散式深入學習](apache-spark-deep-learning-caffe.md)

### <a name="for-spark-developers"></a>針對 Spark 開發人員

* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)
