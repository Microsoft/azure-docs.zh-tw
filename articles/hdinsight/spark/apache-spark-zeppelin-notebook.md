---
title: 在 Azure HDInsight 上搭配使用 Zeppelin Notebook 和 Apache Spark 叢集
description: 如何在 Azure HDInsight 上搭配使用 Zeppelin Notebook 和 Apache Spark 叢集的逐步指示。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.openlocfilehash: 9b076709ee24c61b2699672d28bd61204c88a744
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048036"
---
# <a name="use-zeppelin-notebooks-with-apache-spark-cluster-on-azure-hdinsight"></a>在 Azure HDInsight 上搭配使用 Zeppelin Notebook 和 Apache Spark 叢集

HDInsight Spark 叢集包含可用來執行 Spark 作業的 Zeppelin Notebook。 在本文中，您將學習如何在 HDInsight 叢集上使用 Zeppelin Notebook。

> [!NOTE]
> Zeppelin Notebook 僅適用於 HDInsight 3.5 上的 Spark 1.6.3 以及 HDInsight 3.6 上的 Spark 2.1.0。
>

**先決條件：**

* Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="launch-a-zeppelin-notebook"></a>啟動 Zeppelin Notebook
1. 從 Spark 叢集刀鋒視窗按一下 [叢集儀表板]，然後按一下 [Zeppelin Notebook]。 出現提示時，輸入叢集的系統管理員認證。
   
   > [!NOTE]
   > 您也可以在瀏覽器中開啟下列 URL，來連接到您叢集的 Zeppelin Notebook。 使用您叢集的名稱取代 **CLUSTERNAME** ：
   > 
   > `https://CLUSTERNAME.azurehdinsight.net/zeppelin`
   > 
   > 
1. 建立新的 Notebook。 按一下標頭窗格中的 [Notebook]，然後按一下 [建立新 Note]。
   
    ![建立新的 Zeppelin Notebook](./media/apache-spark-zeppelin-notebook/hdinsight-create-zeppelin-notebook.png "建立新的 Zeppelin Notebook")
   
    輸入 Notebook 的名稱，然後按一下 [建立記事]。
1. 此外，請確定 Notebook 標頭顯示的是已連線狀態。 右上角的綠點即表示此狀態。
   
    ![Zeppelin Notebook 狀態](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-connected.png "Zeppelin Notebook 狀態")
1. 將範例資料載入暫存資料表。 當您在 HDInsight 中建立 Spark 叢集時，系統會將範例資料檔案 **hvac.csv** 複製到相關聯的儲存體帳戶中 (位於 **\HdiSamples\SensorSampleData\hvac**)。
   
    將以下程式碼片段貼入新 Notebook 中預設建立的空白段落。
   
        %livy.spark
        //The above magic instructs Zeppelin to use the Livy Scala interpreter
   
        // Create an RDD using the default Spark context, sc
        val hvacText = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
   
        // Define a schema
        case class Hvac(date: String, time: String, targettemp: Integer, actualtemp: Integer, buildingID: String)
   
        // Map the values in the .csv file to the schema
        val hvac = hvacText.map(s => s.split(",")).filter(s => s(0) != "Date").map(
            s => Hvac(s(0), 
                    s(1),
                    s(2).toInt,
                    s(3).toInt,
                    s(6)
            )
        ).toDF()
   
        // Register as a temporary table called "hvac"
        hvac.registerTempTable("hvac")
   
    按下 **SHIFT + ENTER**，或是按一下 [播放] 按鈕來讓段落執行程式碼片段。 段落右上角的狀態應該會從「準備就緒」逐一轉變成「擱置」、「執行中」及「已完成」。 輸出會顯示在同一個段落的底部。 螢幕擷取畫面如下所示：
   
    ![從未經處理資料建立暫存資料表](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-load-data.png "從未經處理資料建立暫存資料表")
   
    您也可以為每個段落提供標題。 按一下右下角的 [設定] 圖示，然後按一下 [顯示標題]。
1. 現在，您可以針對 **hvac** 資料表執行 Spark SQL 陳述式。 將以下查詢貼入新段落。 此查詢會擷取建築物識別碼，以及在指定日期當天每棟建築物之目標溫度與實際溫度間的差異。 按下 **SHIFT + ENTER**。
   
        %sql
        select buildingID, (targettemp - actualtemp) as temp_diff, date from hvac where date = "6/1/13" 
   
    開頭的 **%Sql** 陳述式會告訴 Notebook 使用 Livy Scala 解譯器。
   
    以下螢幕擷取畫面顯示輸出。
   
    ![使用 Notebook 執行 Spark SQL 陳述式](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-1.png "使用 Notebook 執行 Spark SQL 陳述式")
   
     按一下顯示選項 (以矩形反白顯示) 以針對相同輸出切換不同的表示法。 按一下 [設定] 以選擇構成輸出中索引鍵和值的項目。 在上方的螢幕擷取畫面中，索引鍵為 **buildingID**，而值為 **temp_diff** 的平均值。
1. 您也可以在查詢中使用變數來執行 Spark SQL 陳述式。 下一個程式碼片段示範如何利用您想要查詢的可能值，來定義查詢中的變數 **Temp**。 當您第一次執行查詢時，下拉式清單會自動填入您指定的變數值。
   
        %sql
        select buildingID, date, targettemp, (targettemp - actualtemp) as temp_diff from hvac where targettemp > "${Temp = 65,65|75|85}" 
   
    將此程式碼片段貼到新的段落中，然後按下 **SHIFT + ENTER**。 以下螢幕擷取畫面顯示輸出。
   
    ![使用 Notebook 執行 Spark SQL 陳述式](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-2.png "使用 Notebook 執行 Spark SQL 陳述式")
   
    對於後續的查詢，您可以從下拉式清單選取新的值，然後再次執行查詢。 按一下 [設定] 以選擇構成輸出中索引鍵和值的項目。 上述螢幕擷取畫面使用 **buildingID** 做為索引鍵、平均 **temp_diff** 做為值，而 **targettemp** 做為群組。
1. 重新啟動 Livy 解譯器以結束應用程式。 若要進行此操作，請在右上角按一下登入的使用者名稱，然後按一下 [解譯器]，以開啟解譯器設定。
   
    ![啟動解譯器](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Hive 輸出")
1. 捲動到 Livy 解譯器設定，然後按一下 [重新啟動]。
   
    ![重新啟動 Livy 解譯器](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-restart-interpreter.png "重新啟動 Zeppelin 解譯器")

## <a name="how-do-i-use-external-packages-with-the-notebook"></a>如何搭配 Notebook 使用外部套件？
您可以在 HDInsight (Linux) 上的 Apache Spark 叢集中設定 Zeppelin Notebook，以使用不是叢集中現成隨附的由社群所提供的外部套件。 您可以搜尋 [Maven 儲存機制](http://search.maven.org/) 來取得可用套件的完整清單。 您也可以從其他來源取得可用套件清單。 例如，從 [Spark 套件](http://spark-packages.org/)可以取得社群提供套件的完整清單。

在本文中，您將了解如何搭配 Jupyter Notebook 使用 [spark-csv](http://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) 套件。

1. 開啟解譯器設定。 在右上角按一下登入的使用者名稱，然後按一下 [解譯器]。
   
    ![啟動解譯器](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Hive 輸出")
1. 捲動到 Livy 解譯器設定，然後按一下 [編輯]。
   
    ![變更解譯器設定](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-1.png "變更解譯器設定")
1. 新增稱為 **livy.spark.jars.packages** 的金鑰，並以 `group:id:version` 的格式設定其值。 因此，如果您想要使用 [spark-csv](http://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) 套件，您必須將金鑰值設為 `com.databricks:spark-csv_2.10:1.4.0`。
   
    ![變更解譯器設定](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-2.png "變更解譯器設定")
   
    按一下 [儲存]，然後重新啟動 Livy 解譯器。
1. **秘訣**︰如果您想要了解如何得出上面所輸入的金鑰值，其方法如下。
   
    a. 在「Maven 儲存機制」中找出套件。 針對本教學課程，我們使用 [spark-csv](http://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar)。
   
    b. 從儲存機制收集 [GroupId]、[ArtifactId] 及 [版本] 的值。
   
    ![搭配 Jupyter Notebook 使用外部封裝](./media/apache-spark-zeppelin-notebook/use-external-packages-with-jupyter.png "搭配 Jupyter Notebook 使用外部封裝")
   
    c. 串連三個值，其中以冒號分隔 (**:**)。
   
        com.databricks:spark-csv_2.10:1.4.0

## <a name="where-are-the-zeppelin-notebooks-saved"></a>Zeppelin Notebook 儲存在哪裡？
Zeppelin Notebook 會儲存到叢集前端節點。 因此，如果您刪除叢集，Notebook 會一併刪除。 如果您想要保留 Notebook 以供稍後用於其他叢集上，您必須在作業執行完成後將 Notebook 匯出。 若要匯出 Notebook，請按一下 [匯出] 圖示，如下圖所示。

![下載 Notebook](./media/apache-spark-zeppelin-notebook/zeppelin-download-notebook.png "下載 Notebook")

這會將 Notebook 以 JSON 檔案的形式儲存在下載位置中。

## <a name="livy-session-management"></a>Livy 工作階段管理
當您在 Zeppelin Notebook 中執行第一個程式碼段落時，HDInsight Spark 叢集中便會建立新的 Livy 工作階段。 此工作階段可供您後續建立的所有 Zeppelin Notebook 共用。 如果 Livy 工作階段因為某些原因而遭到刪除 (叢集重新開機等)，您就無法從 Zeppelin Notebook 執行作業。

在這種情況下，您必須先執行下列步驟，然後才能開始從 Zeppelin Notebook 執行作業。 

1. 從 Zeppelin Notebook 重新啟動 Livy 解譯器。 若要進行此操作，請在右上角按一下登入的使用者名稱，然後按一下 [解譯器]，以開啟解譯器設定。
   
    ![啟動解譯器](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Hive 輸出")
1. 捲動到 Livy 解譯器設定，然後按一下 [重新啟動]。
   
    ![重新啟動 Livy 解譯器](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-restart-interpreter.png "重新啟動 Zeppelin 解譯器")
1. 從現有的 Zeppelin Notebook 執行程式碼單元。 這會在 HDInsight 叢集中建立新的 Livy 工作階段。

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
* [Use HDInsight Tools Plugin for IntelliJ IDEA to create and submit Spark Scala applicatons (使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式)](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
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







