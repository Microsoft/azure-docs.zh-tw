---
title: 在 Spark 中搭配 Python 程式庫分析網站記錄 - Azure
description: 此 Notebook 示範如何使用自訂程式庫搭配 Azure HDInsight 上的 Spark 來分析記錄資料。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: jasonh
ms.openlocfilehash: a22e1f90c01b6b0e871516816815286a4f6671a2
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045711"
---
# <a name="analyze-website-logs-using-a-custom-python-library-with-spark-cluster-on-hdinsight"></a>使用自訂 Python 程式庫搭配 HDInsight 上的 Spark 叢集來分析網站記錄

此 Notebook 示範如何使用自訂程式庫搭配 HDInsight 上的 Spark 來分析記錄資料。 我們使用的自訂程式庫是名為 **iislogparser.py**的 Python 程式庫。

> [!TIP]
> 本教學課程也適用於您在 HDInsight 中所建立 Spark (Linux) 叢集上的 Jupyter Notebook。 Notebook 的體驗能讓您從 Notebook 本身執行 Python 程式碼片段。 如果要從 Notebook 中執行本教學課程，請建立 Spark 叢集、啟動 Jupyter Notebook (`https://CLUSTERNAME.azurehdinsight.net/jupyter`)，然後執行 **PySpark** 資料夾下的 Notebook **使用自訂 library.ipynb 透過 Spark 分析記錄**。
>
>

**必要條件：**

您必須滿足以下條件：

* Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。

## <a name="save-raw-data-as-an-rdd"></a>將原始資料儲存為 RDD
在本節中，我們會使用與 HDInsight 中的 Apache Spark 叢集相關聯的 [Jupyter](https://jupyter.org) Notebook，來執行會處理原始範例資料，並將這些資料儲存成 Hive 資料表的作業。 範例資料是所有叢集在預設情況下均有的 .csv 檔案 (hvac.csv)。

一旦將資料儲存成 Hive 資料表之後，下一節我們將使用 Power BI 和 Tableau 等 BI 工具連接 Hive 資料表。

1. 在 [Azure 入口網站](https://portal.azure.com/)的開始面板中，按一下您的 Spark 叢集格圖格 (如果您已將其釘選到開始面板)。 您也可以按一下 [瀏覽全部] > [HDInsight 叢集] 來瀏覽至您的叢集。   
2. 從 Spark 叢集刀鋒視窗按一下 [叢集儀表板]，然後按一下 [Jupyter Notebook]。 出現提示時，輸入叢集的系統管理員認證。

   > [!NOTE]
   > 您也可以在瀏覽器中開啟下列 URL，來連接到您的叢集的 Jupyter Notebook。 使用您叢集的名稱取代 **CLUSTERNAME** ：
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`
   >
   >
3. 建立新的 Notebook。 按一下 [新增]，然後按一下 [PySpark]。

    ![建立新的 Jupyter Notebook](./media/apache-spark-custom-library-website-log-analysis/hdinsight-create-jupyter-notebook.png "建立新的 Jupyter Notebook")
4. 系統隨即會建立新 Notebook，並以 Untitled.pynb 的名稱開啟。 在頂端按一下 Notebook 名稱，然後輸入好記的名稱。

    ![提供 Notebook 的名稱](./media/apache-spark-custom-library-website-log-analysis/hdinsight-name-jupyter-notebook.png "提供 Notebook 的名稱")
5. 您使用 PySpark 核心建立 Notebook，因此不需要明確建立任何內容。 當您執行第一個程式碼儲存格時，系統會自動為您建立 Spark 和 Hive 內容。 首先，您可以匯入此案例需要的類型。 將下列程式碼片段貼到空白儲存格中，然後按 **SHIFT + ENTER**。

        from pyspark.sql import Row
        from pyspark.sql.types import *


1. 使用叢集上已有的範例記錄資料來建立 RDD。 您可以在 **\HdiSamples\HdiSamples\WebsiteLogSampleData\SampleLog\909f2b.log** 上存取與叢集相關聯之預設儲存體帳戶中的資料。

        logs = sc.textFile('wasb:///HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log')


1. 擷取範例記錄組，以確認先前的步驟已順利完成。

        logs.take(5)

    您應該會看到如下所示的輸出：

        # -----------------
        # THIS IS AN OUTPUT
        # -----------------

        [u'#Software: Microsoft Internet Information Services 8.0',
         u'#Fields: date time s-sitename cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Cookie) cs(Referer) cs-host sc-status sc-substatus sc-win32-status sc-bytes cs-bytes time-taken',
         u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step2.png X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 53175 871 46',
         u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step3.png X-ARR-LOG-ID=9eace870-2f49-4efd-b204-0d170da46b4a 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 51237 871 32',
         u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step4.png X-ARR-LOG-ID=4bea5b3d-8ac9-46c9-9b8c-ec3e9500cbea 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 72177 871 47']

## <a name="analyze-log-data-using-a-custom-python-library"></a>使用自訂 Python 程式庫來分析記錄資料
1. 在上述輸出中，前幾行包含標頭資訊，其餘各行則符合該標頭中所說明的結構描述。 剖析這類記錄檔可能是複雜的作業。 因此，我們使用自訂 Python 程式庫 (**iislogparser.py**)，它可大幅簡化這類記錄檔的剖析。 依預設，此程式庫會包含在位於以下位置的 HDInsight 上的 Spark 叢集中： **/HdiSamples/HdiSamples/WebsiteLogSampleData/iislogparser.py**。

    不過，此程式庫不在 `PYTHONPATH` 中，因此無法藉由 `import iislogparser` 之類的匯入陳述式來使用它。 若要使用此程式庫，我們必須將它散發到所有的背景工作角色節點。 執行下列程式碼片段。

        sc.addPyFile('wasb:///HdiSamples/HdiSamples/WebsiteLogSampleData/iislogparser.py')


1. `iislogparser` 提供一個函數 `parse_log_line`，在記錄行為標頭資料列時會傳回 `None`，在發現記錄行時則傳回 `LogLine` 類別的執行個體。 使用 `LogLine` 類別，僅從 RDD 擷取記錄行：

        def parse_line(l):
            import iislogparser
            return iislogparser.parse_log_line(l)
        logLines = logs.map(parse_line).filter(lambda p: p is not None).cache()
2. 擷取幾個記錄行，以確認步驟已順利完成。

       logLines.take(2)

   輸出應該類似如下範例：

       # -----------------
       # THIS IS AN OUTPUT
       # -----------------

       [2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step2.png X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 53175 871 46,
        2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step3.png X-ARR-LOG-ID=9eace870-2f49-4efd-b204-0d170da46b4a 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 51237 871 32]
3. `LogLine` 類別也有一些有用的方法，像是 `is_error()`，它會傳回記錄項目是否有錯誤碼。 它可用來計算已擷取的記錄行中有多少錯誤，然後將所有錯誤記錄到另一個檔案。

       errors = logLines.filter(lambda p: p.is_error())
       numLines = logLines.count()
       numErrors = errors.count()
       print 'There are', numErrors, 'errors and', numLines, 'log entries'
       errors.map(lambda p: str(p)).saveAsTextFile('wasb:///HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b-2.log')

   您應該會看到如下的輸出：

       # -----------------
       # THIS IS AN OUTPUT
       # -----------------

       There are 30 errors and 646 log entries
4. 您也可以使用 **Matplotlib** 來建構資料的視覺效果。 例如，如果您想要找出要求長時間執行的原因，您可以尋找平均佔用最多服務資源的檔案。
   下列程式碼片段會擷取耗費最多時間為要求提供服務的前 25 個資源。

       def avgTimeTakenByKey(rdd):
           return rdd.combineByKey(lambda line: (line.time_taken, 1),
                                   lambda x, line: (x[0] + line.time_taken, x[1] + 1),
                                   lambda x, y: (x[0] + y[0], x[1] + y[1]))\
                     .map(lambda x: (x[0], float(x[1][0]) / float(x[1][1])))

       avgTimeTakenByKey(logLines.map(lambda p: (p.cs_uri_stem, p))).top(25, lambda x: x[1])

   您應該會看到如下的輸出：

       # -----------------
       # THIS IS AN OUTPUT
       # -----------------

       [(u'/blogposts/mvc4/step13.png', 197.5),
        (u'/blogposts/mvc2/step10.jpg', 179.5),
        (u'/blogposts/extractusercontrol/step5.png', 170.0),
        (u'/blogposts/mvc4/step8.png', 159.0),
        (u'/blogposts/mvcrouting/step22.jpg', 155.0),
        (u'/blogposts/mvcrouting/step3.jpg', 152.0),
        (u'/blogposts/linqsproc1/step16.jpg', 138.75),
        (u'/blogposts/linqsproc1/step26.jpg', 137.33333333333334),
        (u'/blogposts/vs2008javascript/step10.jpg', 127.0),
        (u'/blogposts/nested/step2.jpg', 126.0),
        (u'/blogposts/adminpack/step1.png', 124.0),
        (u'/BlogPosts/datalistpaging/step2.png', 118.0),
        (u'/blogposts/mvc4/step35.png', 117.0),
        (u'/blogposts/mvcrouting/step2.jpg', 116.5),
        (u'/blogposts/aboutme/basketball.jpg', 109.0),
        (u'/blogposts/anonymoustypes/step11.jpg', 109.0),
        (u'/blogposts/mvc4/step12.png', 106.0),
        (u'/blogposts/linq8/step0.jpg', 105.5),
        (u'/blogposts/mvc2/step18.jpg', 104.0),
        (u'/blogposts/mvc2/step11.jpg', 104.0),
        (u'/blogposts/mvcrouting/step1.jpg', 104.0),
        (u'/blogposts/extractusercontrol/step1.png', 103.0),
        (u'/blogposts/sqlvideos/sqlvideos.jpg', 102.0),
        (u'/blogposts/mvcrouting/step21.jpg', 101.0),
        (u'/blogposts/mvc4/step1.png', 98.0)]
5. 您也可以用繪圖的形式呈現這項資訊。 建立繪圖的第一個步驟是建立暫存資料表 **AverageTime**。 此資料表會依時間將記錄分組，以查看在任何特定時間中是否出現異常的延遲尖峰。

       avgTimeTakenByMinute = avgTimeTakenByKey(logLines.map(lambda p: (p.datetime.minute, p))).sortByKey()
       schema = StructType([StructField('Minutes', IntegerType(), True),
                            StructField('Time', FloatType(), True)])

       avgTimeTakenByMinuteDF = sqlContext.createDataFrame(avgTimeTakenByMinute, schema)
       avgTimeTakenByMinuteDF.registerTempTable('AverageTime')
6. 接著，您可以執行下列 SQL 查詢取得 **AverageTime** 資料表中的所有記錄。

       %%sql -o averagetime
       SELECT * FROM AverageTime

   `%%sql` magic 後面緊接著 `-o averagetime` 可確保查詢的輸出會保存在 Jupyter 伺服器的本機上 (通常是叢集的前端節點)。 輸出內容會以具有指定名稱 [averagetime](http://pandas.pydata.org/) 的 **Pandas**資料框架保存。

   您應該會看到如下的輸出：

   ![SQL 查詢輸出](./media/apache-spark-custom-library-website-log-analysis/hdinsight-jupyter-sql-qyery-output.png "SQL 查詢輸出")

   如需 `%%sql` magic 的詳細資訊，請參閱 [%%sql magic 支援的參數](apache-spark-jupyter-notebook-kernels.md#parameters-supported-with-the-sql-magic)。
7. 現在您可以使用 Matplotlib (用於建構資料視覺效果的程式庫) 建立繪圖。 因為必須從保存在本機上的 **averagetime** 資料框架建立繪圖，所以程式碼片段的開頭必須為 `%%local` magic。 這可確保程式碼是在 Jupyter 伺服器的本機上執行。

       %%local
       %matplotlib inline
       import matplotlib.pyplot as plt

       plt.plot(averagetime['Minutes'], averagetime['Time'], marker='o', linestyle='--')
       plt.xlabel('Time (min)')
       plt.ylabel('Average time taken for request (ms)')

   您應該會看到如下的輸出：

   ![Matplotlib 輸出](./media/apache-spark-custom-library-website-log-analysis/hdinsight-apache-spark-web-log-analysis-plot.png "Matplotlib 輸出")
8. 應用程式執行完畢之後，您應該要關閉 Notebook 來釋放資源。 若要這樣做，請從 Notebook 的 [檔案] 功能表中，按一下 [關閉並停止]。 這樣就能夠結束並關閉 Notebook。

## <a name="seealso"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](../hdinsight-apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
