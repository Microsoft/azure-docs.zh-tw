---
title: 在 Azure 上使用 Scala 與 Spark 的資料科學 | Microsoft Docs
description: 如何使用 Scala 搭配 Spark 可調整 MLlib 和 Azure HDInsight Spark 叢集上的 SparkML 封裝，處理受監督的機器學習工作。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: a7c97153-583e-48fe-b301-365123db3780
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 16e4af4dd7f5c2bd14d70cc28225dfc750ce3bea
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34838505"
---
# <a name="data-science-using-scala-and-spark-on-azure"></a>在 Azure 上使用 Scala 與 Spark 的資料科學
本文章說明如何使用 Scala 搭配 Spark 可調整 MLlib 和 Azure HDInsight Spark 叢集上的 SparkML 封裝，處理受監督的機器學習工作。 它會引導您進行構成 [資料科學程序](http://aka.ms/datascienceprocess)的各項工作︰資料擷取和探索、視覺化、特徵設計、模型化和模型取用。 本文中的模型除了兩個常見受監督的機器學習工作之外，還包括羅吉斯和線性迴歸、隨機樹系和梯度推進樹 (GBT)︰

* 迴歸問題︰計程車車程的小費金額 ($) 預測
* 二進位分類︰計程車車程的小費或不給小費 (1/0) 預測

模型化程序需要訓練和評估測試資料集和相關精確度計量。 在本文中，您會了解如何在 Azure Blob 儲存體中儲存這些模型，以及如何評分及評估模型的預測效能。 本文也涵蓋如何使用交叉驗證和超參數掃掠來最佳化模型等更進階的主題。 所使用的資料是 GitHub 上 2013 年紐約市計程車車程和費用資料集的抽樣樣本。

[Scala](http://www.scala-lang.org/)是一種以 Java 虛擬機器為基礎的語言，整合物件導向與函式型語言的概念。 這是一種可調整的語言，非常適合用於雲端中的分散式處理以及在 Azure Spark 叢集上執行。

[Spark](http://spark.apache.org/) 是一個開放原始碼平行處理架構，可支援記憶體內部處理，大幅提升巨量資料分析應用程式的效能。 Spark 處理引擎是專為速度、易用性及精密分析打造的產品。 Spark 的記憶體內分散式計算功能，使其成為機器學習和圖表計算中反覆演算法的絕佳選擇。 [Spark.ml](http://spark.apache.org/docs/latest/ml-guide.html) 封裝提供一組以資料框架為基礎的統一高階 API，可協助您建立及微調實際的機器學習管線。 [MLlib](http://spark.apache.org/mllib/) 是 Spark 的可調整機器學習程式庫，將模型化功能引進此分散式環境。


  [HDInsight Spark](../../hdinsight/spark/apache-spark-overview.md) 是開放原始碼 Spark 的 Azure 託管供應項目。 它也支援 Spark 叢集上的 Jupyter Scala Notebook，可執行 Spark SQL 互動式查詢以轉換、篩選和視覺化 Azure Blob 儲存體中儲存的資料。 本文中的 Scala 程式碼片段提供解決方案，並且顯示相關的繪圖，將安裝在 Spark 叢集上的 Jupyter Notebook 資料加以視覺化。 這些主題中的模型化步驟有程式碼向您示範如何訓練、評估、儲存和使用各類模型。

本文中的設定步驟與程式碼適用於 Azure HDInsight 3.4 Spark 1.6。 不過，本文與 [Scala Jupyter Notebook](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration%20Modeling%20and%20Scoring%20using%20Scala.ipynb) 中的程式碼皆屬泛型程式碼，應該能在任何 Spark 叢集上運作。 若未使用 HDInsight Spark，叢集設定和管理步驟可能與本文顯示的稍有不同。

> [!NOTE]
> 如需示範如何使用 Python 而非 Scala 來完成端對端資料科學程序工作的主題，請參閱 [在 Azure HDInsight 上使用 Spark 的資料科學](spark-overview.md)。
> 
> 

## <a name="prerequisites"></a>先決條件
* 您必須擁有 Azure 訂用帳戶。 如果還沒有， [請取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* 您需要 Azure HDInsight 3.4 Spark 1.6 叢集來完成下列程序。 若要建立叢集，請參閱 [開始使用：在 Azure HDInsight 上建立 Apache Spark](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md)中的指示。 在 [選取叢集類型]  功能表上設定叢集類型和版本。

![HDInsight 叢集類型組態](./media/scala-walkthrough/spark-cluster-on-portal.png)

> [!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]
> 
> 

如需紐約市計程車車程資料的說明以及如何在 Spark 叢集上從 Jupyter Notebook 執行程式碼的指示，請參閱 [在 Azure HDInsight 上使用 Spark 的資料科學概觀](spark-overview.md)中的相關章節。  

## <a name="execute-scala-code-from-a-jupyter-notebook-on-the-spark-cluster"></a>在 Spark 叢集上從 Jupyter Notebook 執行 Scala 程式碼
您可以從 Azure 入口網站啟動 Jupyter Notebook。 在儀表板上尋找 Spark 叢集，然後按一下該項目以進入您的叢集管理頁面。 接著按一下 [叢集儀表板]，然後按一下 [Jupyter Notebook] 來開啟與 Spark 叢集相關聯的 Notebook。

![叢集儀表板和 Jupyter Notebook](./media/scala-walkthrough/spark-jupyter-on-portal.png)

您也可以在 https://&lt;clustername&gt;.azurehdinsight.net/jupyter 存取 Jupyter Notebook。 將 *clustername* 取代為您叢集的名稱。 您需要有系統管理員帳戶的密碼才能存取 Jupyter Notebook。

![使用叢集名稱移至 Jupyter Notebook](./media/scala-walkthrough/spark-jupyter-notebook.png)

選取 [Scala]  會看到一個目錄，當中有一些使用 PySpark API 的預先封裝 Notebook 的範例。 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/Scala)上有使用 Scala.ipynb Notebook 的探勘模型化和評分，其中包含此 Spark 主題套件的程式碼範例。

您可以將 Notebook 直接從 GitHub 上傳至 Spark 叢集上的 Jupyter Notebook 伺服器。 在 Jupyter 首頁上，按一下 [上傳]  按鈕。 在檔案總管中，貼上 Scala Notebook 的 GitHub (原始內容) URL，然後按一下 [開啟] 。 下列 URL 有 Scala Notebook 可供使用：

[Exploration-Modeling-and-Scoring-using-Scala.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration-Modeling-and-Scoring-using-Scala.ipynb)

## <a name="setup-preset-spark-and-hive-contexts-spark-magics-and-spark-libraries"></a>安裝程式︰預設的 Spark 和 Hive 內容、Spark Magic 和 Spark 程式庫
### <a name="preset-spark-and-hive-contexts"></a>預設的 Spark 和 Hive 內容
    # SET THE START TIME
    import java.util.Calendar
    val beginningTime = Calendar.getInstance().getTime()


Jupyter Notebook 所提供的 Spark 核心有預設的內容。 您不必明確設定 Spark 或 Hive 內容，就能開始使用您所開發的應用程式。 預設內容為：

* `sc` 代表 SparkContext
* `sqlContext` 代表 HiveContext

### <a name="spark-magics"></a>Spark Magic
Spark 核心提供一些預先定義的 “Magic”，這是您可以使用 `%%`呼叫的特殊命令。 下列程式碼範例會使用兩個命令。

* `%%local` 指定後續行所列的程式碼，將在本機執行。 程式碼必須是有效的 Scala 程式碼。
* `%%sql -o <variable name>` 針對 `sqlContext` 執行 Hive 查詢。 如果傳遞 `-o` 參數，則查詢的結果會當做 Spark 資料框架，保存在 `%%local` Scala 內容中。

如需關於 Jupyter Notebook 核心，以及使用 `%%` (例如 `%%local`) 呼叫其預先定義的 "Magic" 的詳細資訊，請參閱 [HDInsight 上的 HDInsight Spark Linux 叢集可供 Jupyter Notebook 使用的核心](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md)。

### <a name="import-libraries"></a>匯入程式庫
使用下列程式碼匯入 Spark、MLlib 和其他所需的程式庫。

    # IMPORT SPARK AND JAVA LIBRARIES
    import org.apache.spark.sql.SQLContext
    import org.apache.spark.sql.functions._
    import java.text.SimpleDateFormat
    import java.util.Calendar
    import sqlContext.implicits._
    import org.apache.spark.sql.Row

    # IMPORT SPARK SQL FUNCTIONS
    import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType, FloatType, DoubleType}
    import org.apache.spark.sql.functions.rand

    # IMPORT SPARK ML FUNCTIONS
    import org.apache.spark.ml.Pipeline
    import org.apache.spark.ml.feature.{StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer, Binarizer}
    import org.apache.spark.ml.tuning.{ParamGridBuilder, TrainValidationSplit, CrossValidator}
    import org.apache.spark.ml.regression.{LinearRegression, LinearRegressionModel, RandomForestRegressor, RandomForestRegressionModel, GBTRegressor, GBTRegressionModel}
    import org.apache.spark.ml.classification.{LogisticRegression, LogisticRegressionModel, RandomForestClassifier, RandomForestClassificationModel, GBTClassifier, GBTClassificationModel}
    import org.apache.spark.ml.evaluation.{BinaryClassificationEvaluator, RegressionEvaluator, MulticlassClassificationEvaluator}

    # IMPORT SPARK MLLIB FUNCTIONS
    import org.apache.spark.mllib.linalg.{Vector, Vectors}
    import org.apache.spark.mllib.util.MLUtils
    import org.apache.spark.mllib.classification.{LogisticRegressionWithLBFGS, LogisticRegressionModel}
    import org.apache.spark.mllib.regression.{LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel}
    import org.apache.spark.mllib.tree.{GradientBoostedTrees, RandomForest}
    import org.apache.spark.mllib.tree.configuration.BoostingStrategy
    import org.apache.spark.mllib.tree.model.{GradientBoostedTreesModel, RandomForestModel, Predict}
    import org.apache.spark.mllib.evaluation.{BinaryClassificationMetrics, MulticlassMetrics, RegressionMetrics}

    # SPECIFY SQLCONTEXT
    val sqlContext = new SQLContext(sc)


## <a name="data-ingestion"></a>資料擷取
資料科學程序的第一步是內嵌您想要分析的資料。 您要從資料所在的外部來源或系統，將資料帶入資料探索和模型化環境。 在本文中，您要內嵌的資料是聯結 0.1% 取樣的計程車車程和資費檔案 (儲存為 .tsv 檔案)。 資料探索和模型化環境為 Spark。 本節包含程式碼來完成下列的工作系列︰

1. 設定資料和模型儲存體的目錄路徑。
2. 讀取輸入資料集 (儲存為 .tsv 檔案)。
3. 定義資料的結構描述並清除資料。
4. 建立已清除的資料框架，並在記憶體中加以快取。
5. 在 SQLContext 中將資料註冊為暫存資料表。
6. 查詢資料表並將結果匯入資料框架。

### <a name="set-directory-paths-for-storage-locations-in-azure-blob-storage"></a>在 Azure Blob 儲存體中設定儲存位置的目錄路徑
Spark 可以讀取和寫入 Azure Blob 儲存體。 您可以使用 Spark 來處理任何現有的資料，然後在 Blob 儲存體中再次儲存結果。

若要在 Blob 儲存體中儲存模型或檔案，您必須正確指定路徑。 使用以 `wasb:///`開頭的路徑，參考附加至 Spark 叢集的預設容器。 使用 `wasb://`參考其他位置。

下列程式碼範例指定要讀取輸入資料的位置，以及附加至 Spark 叢集 (將會儲存模型) 的 Blob 儲存體路徑。

    # SET PATHS TO DATA AND MODEL FILE LOCATIONS
    # INGEST DATA AND SPECIFY HEADERS FOR COLUMNS
    val taxi_train_file = sc.textFile("wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv")
    val header = taxi_train_file.first;

    # SET THE MODEL STORAGE DIRECTORY PATH
    # NOTE THAT THE FINAL BACKSLASH IN THE PATH IS REQUIRED.
    val modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";


### <a name="import-data-create-an-rdd-and-define-a-data-frame-according-to-the-schema"></a>根據結構描述匯入資料、建立 RDD 並定義資料框架
    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # DEFINE THE SCHEMA BASED ON THE HEADER OF THE FILE
    val sqlContext = new SQLContext(sc)
    val taxi_schema = StructType(
        Array(
            StructField("medallion", StringType, true),
            StructField("hack_license", StringType, true),
            StructField("vendor_id", StringType, true),
            StructField("rate_code", DoubleType, true),
            StructField("store_and_fwd_flag", StringType, true),
            StructField("pickup_datetime", StringType, true),
            StructField("dropoff_datetime", StringType, true),
            StructField("pickup_hour", DoubleType, true),
            StructField("pickup_week", DoubleType, true),
            StructField("weekday", DoubleType, true),
            StructField("passenger_count", DoubleType, true),
            StructField("trip_time_in_secs", DoubleType, true),
            StructField("trip_distance", DoubleType, true),
            StructField("pickup_longitude", DoubleType, true),
            StructField("pickup_latitude", DoubleType, true),
            StructField("dropoff_longitude", DoubleType, true),
            StructField("dropoff_latitude", DoubleType, true),
            StructField("direct_distance", StringType, true),
            StructField("payment_type", StringType, true),
            StructField("fare_amount", DoubleType, true),
            StructField("surcharge", DoubleType, true),
            StructField("mta_tax", DoubleType, true),
            StructField("tip_amount", DoubleType, true),
            StructField("tolls_amount", DoubleType, true),
            StructField("total_amount", DoubleType, true),
            StructField("tipped", DoubleType, true),
            StructField("tip_class", DoubleType, true)
            )
        )

    # CAST VARIABLES ACCORDING TO THE SCHEMA
    val taxi_temp = (taxi_train_file.map(_.split("\t"))
                            .filter((r) => r(0) != "medallion")
                            .map(p => Row(p(0), p(1), p(2),
                                p(3).toDouble, p(4), p(5), p(6), p(7).toDouble, p(8).toDouble, p(9).toDouble, p(10).toDouble,
                                p(11).toDouble, p(12).toDouble, p(13).toDouble, p(14).toDouble, p(15).toDouble, p(16).toDouble,
                                p(17), p(18), p(19).toDouble, p(20).toDouble, p(21).toDouble, p(22).toDouble,
                                p(23).toDouble, p(24).toDouble, p(25).toDouble, p(26).toDouble)))


    # CREATE AN INITIAL DATA FRAME AND DROP COLUMNS, AND THEN CREATE A CLEANED DATA FRAME BY FILTERING FOR UNWANTED VALUES OR OUTLIERS
    val taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

    val taxi_df_train_cleaned = (taxi_train_df.drop(taxi_train_df.col("medallion"))
            .drop(taxi_train_df.col("hack_license")).drop(taxi_train_df.col("store_and_fwd_flag"))
            .drop(taxi_train_df.col("pickup_datetime")).drop(taxi_train_df.col("dropoff_datetime"))
            .drop(taxi_train_df.col("pickup_longitude")).drop(taxi_train_df.col("pickup_latitude"))
            .drop(taxi_train_df.col("dropoff_longitude")).drop(taxi_train_df.col("dropoff_latitude"))
            .drop(taxi_train_df.col("surcharge")).drop(taxi_train_df.col("mta_tax"))
            .drop(taxi_train_df.col("direct_distance")).drop(taxi_train_df.col("tolls_amount"))
            .drop(taxi_train_df.col("total_amount")).drop(taxi_train_df.col("tip_class"))
            .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200"));

    # CACHE AND MATERIALIZE THE CLEANED DATA FRAME IN MEMORY
    taxi_df_train_cleaned.cache()
    taxi_df_train_cleaned.count()

    # REGISTER THE DATA FRAME AS A TEMPORARY TABLE IN SQLCONTEXT
    taxi_df_train_cleaned.registerTempTable("taxi_train")

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰8 秒。

### <a name="query-the-table-and-import-results-in-a-data-frame"></a>查詢資料表並將結果匯入資料框架
接著，查詢資料表中的資費、乘客和小費資料，篩選出損毀和離群資料，並列印數個資料列。

    # QUERY THE DATA
    val sqlStatement = """
        SELECT fare_amount, passenger_count, tip_amount, tipped
        FROM taxi_train
        WHERE passenger_count > 0 AND passenger_count < 7
        AND fare_amount > 0 AND fare_amount < 200
        AND payment_type in ('CSH', 'CRD')
        AND tip_amount > 0 AND tip_amount < 25
    """
    val sqlResultsDF = sqlContext.sql(sqlStatement)

    # SHOW ONLY THE TOP THREE ROWS
    sqlResultsDF.show(3)

**輸出：**

| fare_amount | passenger_count | tip_amount | tipped |
| --- | --- | --- | --- |
|        13.5 |1.0 |2.9 |1.0 |
|        16.0 |2.0 |3.4 |1.0 |
|        10.5 |2.0 |1.0 |1.0 |

## <a name="data-exploration-and-visualization"></a>資料探索和虛擬化
將資料帶入 Spark 之後，資料科學程序的下一個步驟是透過探索和視覺化以更深入瞭解資料。 本節中，您可以使用 SQL 查詢檢查計程車資料。 然後將結果匯入資料框架，以使用 Jupyter 的自動視覺化特徵繪製目標變數和潛在特徵以便進行視覺檢查。

### <a name="use-local-and-sql-magic-to-plot-data"></a>使用本機和 SQL magic 來繪製資料
根據預設，在背景工作節點上保存的工作階段內容中，可取得您從 Jupyter Notebook 執行之任何程式碼片段的輸出。 如果您想要將車程儲存至每個計算的背景工作節點，而且如果在 Jupyter 伺服器節點 (此為前端節點) 的本機上可取得計算所需的所有資料，您可以使用 `%%local` Magic 在 Jupyter 伺服器上執行程式碼片段。

* **SQL magic** (`%%sql`)。 HDInsight Spark 核心支援針對 SQLContext 進行簡單的內嵌 HiveQL 查詢。 (`-o VARIABLE_NAME`) 引數會將 SQL 查詢的輸出，保存為 Jupyter 伺服器上的 Pandas 資料框架。 這代表會在本機模式中使用此引數。
* `%%local` **magic**。 `%%local` magic 在 Jupyter 伺服器本機 (HDInsight 叢集的前端節點) 上執行程式碼。 一般而言，您會使用 `%%local` magic 來搭配含有 `-o` 參數的 `%%sql` magic。 `-o` 參數會保存本機 SQL 查詢的輸出，然後 `%%local` magic 會針對已保存在本機上的 SQL 查詢輸出，觸發下一組要在本機上執行的程式碼片段。

### <a name="query-the-data-by-using-sql"></a>使用 SQL 查詢資料
此查詢會依照費用金額、乘客計數和小費金額擷取計程車車程。

    # RUN THE SQL QUERY
    %%sql -q -o sqlResults
    SELECT fare_amount, passenger_count, tip_amount, tipped FROM taxi_train WHERE passenger_count > 0 AND passenger_count < 7 AND fare_amount > 0 AND fare_amount < 200 AND payment_type in ('CSH', 'CRD') AND tip_amount > 0 AND tip_amount < 25

在下列程式碼中， `%%local` magic 會建立本機資料框架 sqlResults。 您可以使用 sqlResults 透過 matplotlib 繪圖。

> [!TIP]
> 本文中會多次使用本機 magic。 如果資料集很大，請進行取樣以建立可容納於本機記憶體的資料框架。
> 
> 

### <a name="plot-the-data"></a>繪製資料
資料框架以 Pandas 資料框架形式出現在本機內容之後，您就可以使用 Python 程式碼進行繪圖。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER
    %%local

    # USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES.
    # CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, ETC.)
    sqlResults


 在您執行程式碼之後，Spark 核心會將 SQL (HiveQL) 查詢的輸出自動視覺化。 您可以選擇數種類型的視覺效果︰

* 資料表
* 圓形圖
* 折線圖
* 領域
* 長條圖

以下是繪製資料的程式碼：

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    import matplotlib.pyplot as plt
    %matplotlib inline

    # PLOT TIP BY PAYMENT TYPE AND PASSENGER COUNT
    ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
    ax1.set_title('Tip amount distribution')
    ax1.set_xlabel('Tip Amount ($)')
    ax1.set_ylabel('Counts')
    plt.suptitle('')
    plt.show()

    # PLOT TIP BY PASSENGER COUNT
    ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
    ax2.set_title('Tip amount by Passenger count')
    ax2.set_xlabel('Passenger count')
    ax2.set_ylabel('Tip Amount ($)')
    plt.suptitle('')
    plt.show()

    # PLOT TIP AMOUNT BY FARE AMOUNT; SCALE POINTS BY PASSENGER COUNT
    ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
    ax.set_title('Tip amount by Fare amount')
    ax.set_xlabel('Fare Amount ($)')
    ax.set_ylabel('Tip Amount ($)')
    plt.axis([-2, 80, -2, 20])
    plt.show()


**輸出：**

![小費金額長條圖](./media/scala-walkthrough/plot-tip-amount-histogram.png)

![按乘客計數排列的小費金額](./media/scala-walkthrough/plot-tip-amount-by-passenger-count.png)

![按費用金額排列的小費金額](./media/scala-walkthrough/plot-tip-amount-by-fare-amount.png)

## <a name="create-features-and-transform-features-and-then-prep-data-for-input-into-modeling-functions"></a>建立特徵和轉換特徵，然後準備資料輸入模型化函式
如需 Spark ML 和 MLlib 中以樹狀結構為基礎的模型化函式，您必須使用各種技術 (例如分類收納、索引編製、one-hot 編碼和向量化) 備妥目標和特徵。 以下是本節要遵循的程序︰

1. 將小時 **納入** 流量時間值區以建立新特徵。
2. 對分類特徵套用 **索引編製和 one-hot 編碼** 。
3. **取樣並將資料集分割** 成訓練和測試部分。
4. **指定訓練變數和特徵**，然後建立已編製索引或已進行 one-hot 編碼的訓練和測試輸入標示點彈性分散式資料集 (RDD) 或資料框架。
5. 自動 **分類及向量化特徵和目標** 以做為機器學習模型的輸入。

### <a name="create-a-new-feature-by-binning-hours-into-traffic-time-buckets"></a>將小時納入流量時間值區以建立新特徵
此程式碼示範如何將小時納入流量時間值區以建立新特徵，以及如何從記憶體快取產生的資料框架。 在重複使用 RDD 和資料框架的位置，快取可改善執行時間。 因此，您會在下列程序中的幾個階段快取 RDD 和資料框架。

    # CREATE FOUR BUCKETS FOR TRAFFIC TIMES
    val sqlStatement = """
        SELECT *,
        CASE
         WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night"
         WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush"
         WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
         WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
        END as TrafficTimeBins
        FROM taxi_train
    """
    val taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

    # CACHE THE DATA FRAME IN MEMORY AND MATERIALIZE THE DATA FRAME IN MEMORY
    taxi_df_train_with_newFeatures.cache()
    taxi_df_train_with_newFeatures.count()


### <a name="indexing-and-one-hot-encoding-of-categorical-features"></a>索引和 one-hot 編碼分類功能
MLlib 的模型化和預測函式需要先執行功能來分類要索引或編碼的分類輸入資料，才能使用這些資料。 本節示範如何索引或編碼分類特徵，以輸入模型化函式。

根據模型，您需要以不同方式索引或編碼它們。 例如，羅吉斯和線性迴歸模型需要 one-hot 編碼。 例如，有三個類別的特徵可展開成三個特徵資料行。 根據觀察值的分類，每個資料行包含 0 或 1。 MLlib 為 one-hot 編碼提供 [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) 函式。 此編碼器會將標籤索引資料行對應到二進位向量資料行，最多有一個單一值。 使用這種編碼方式，可將預期數值特徵的演算法 (例如羅吉斯迴歸) 套用至分類特徵。

您在此只會轉換四個變數 (其為字元字串) 來顯示範例。 您也可以將其他以數值表示的變數 (例如工作日) 編製索引為類別變數。

如需編製索引，請使用 `StringIndexer()`，如需進行 one-hot 編碼，請使用 MLlib 中的 `OneHotEncoder()` 函式。 以下是要索引及編碼分類功能的程式碼︰

    # CREATE INDEXES AND ONE-HOT ENCODED VECTORS FOR SEVERAL CATEGORICAL FEATURES

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # INDEX AND ENCODE VENDOR_ID
    val stringIndexer = new StringIndexer().setInputCol("vendor_id").setOutputCol("vendorIndex").fit(taxi_df_train_with_newFeatures)
    val indexed = stringIndexer.transform(taxi_df_train_with_newFeatures)
    val encoder = new OneHotEncoder().setInputCol("vendorIndex").setOutputCol("vendorVec")
    val encoded1 = encoder.transform(indexed)

    # INDEX AND ENCODE RATE_CODE
    val stringIndexer = new StringIndexer().setInputCol("rate_code").setOutputCol("rateIndex").fit(encoded1)
    val indexed = stringIndexer.transform(encoded1)
    val encoder = new OneHotEncoder().setInputCol("rateIndex").setOutputCol("rateVec")
    val encoded2 = encoder.transform(indexed)

    # INDEX AND ENCODE PAYMENT_TYPE
    val stringIndexer = new StringIndexer().setInputCol("payment_type").setOutputCol("paymentIndex").fit(encoded2)
    val indexed = stringIndexer.transform(encoded2)
    val encoder = new OneHotEncoder().setInputCol("paymentIndex").setOutputCol("paymentVec")
    val encoded3 = encoder.transform(indexed)

    # INDEX AND TRAFFIC TIME BINS
    val stringIndexer = new StringIndexer().setInputCol("TrafficTimeBins").setOutputCol("TrafficTimeBinsIndex").fit(encoded3)
    val indexed = stringIndexer.transform(encoded3)
    val encoder = new OneHotEncoder().setInputCol("TrafficTimeBinsIndex").setOutputCol("TrafficTimeBinsVec")
    val encodedFinal = encoder.transform(indexed)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰4 秒。

### <a name="sample-and-split-the-data-set-into-training-and-test-fractions"></a>取樣並將資料集分割成訓練和測試部分
此程式碼會建立隨機取樣資料 (此範例中為 25%)。 雖然此範例因為資料集的大小而不需要取樣，但是本文會示範如何取樣，讓您了解需要時如何使用它來自行解決問題。 如果樣本數較大，這可以在訓練模型時節省大量時間。 接下來將樣本數分割成訓練部分 (此範例中為 75%) 和測試部分 (此範例中為 25%)，以便在分類和迴歸模型化中使用。

在 ("rand" 資料行中) 的每一列加入一個隨機數字 (介於 0 與 1 之間)，可用來在訓練期間選取交叉驗證摺疊數。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # SPECIFY SAMPLING AND SPLITTING FRACTIONS
    val samplingFraction = 0.25;
    val trainingFraction = 0.75;
    val testingFraction = (1-trainingFraction);
    val seed = 1234;
    val encodedFinalSampledTmp = encodedFinal.sample(withReplacement = false, fraction = samplingFraction, seed = seed)
    val sampledDFcount = encodedFinalSampledTmp.count().toInt

    val generateRandomDouble = udf(() => {
        scala.util.Random.nextDouble
    })

    # ADD A RANDOM NUMBER FOR CROSS-VALIDATION
    val encodedFinalSampled = encodedFinalSampledTmp.withColumn("rand", generateRandomDouble());

    # SPLIT THE SAMPLED DATA FRAME INTO TRAIN AND TEST, WITH A RANDOM COLUMN ADDED FOR DOING CROSS-VALIDATION (SHOWN LATER)
    # INCLUDE A RANDOM COLUMN FOR CREATING CROSS-VALIDATION FOLDS
    val splits = encodedFinalSampled.randomSplit(Array(trainingFraction, testingFraction), seed = seed)
    val trainData = splits(0)
    val testData = splits(1)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰2 秒。

### <a name="specify-training-variable-and-features-and-then-create-indexed-or-one-hot-encoded-training-and-testing-input-labeled-point-rdds-or-data-frames"></a>指定訓練變數和特徵，然後建立已編製索引或已進行 one-hot 編碼的訓練和測試輸入標示點 RDD 或資料框架
本節包含的程式碼示範如何將分類的文字資料索引為標示點資料類型並加以編碼，您可以用來訓練及測試 MLlib 羅吉斯迴歸和其他分類模型。 標示點物件是 RDD，其格式化成適合 MLlib 中大部分機器學習演算法所需的輸入資料。 [標示點](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) 是本機向量 (密集或疏鬆)，與標籤/回應相關聯。

在此程式碼中，您指定目標 (相依) 變數和要用來訓練模型的特徵。 然後，建立已編製索引或已進行 one-hot 編碼的訓練和測試輸入標示點 RDD 或資料框架。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # MAP NAMES OF FEATURES AND TARGETS FOR CLASSIFICATION AND REGRESSION PROBLEMS
    val featuresIndOneHot = List("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))
    val featuresIndIndex = List("paymentIndex", "vendorIndex", "rateIndex", "TrafficTimeBinsIndex", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))

    # SPECIFY THE TARGET FOR CLASSIFICATION ('tipped') AND REGRESSION ('tip_amount') PROBLEMS
    val targetIndBinary = List("tipped").map(encodedFinalSampled.columns.indexOf(_))
    val targetIndRegression = List("tip_amount").map(encodedFinalSampled.columns.indexOf(_))

    # CREATE INDEXED LABELED POINT RDD OBJECTS
    val indexedTRAINbinary = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
    val indexedTESTbinary = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
    val indexedTRAINreg = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
    val indexedTESTreg = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))

    # CREATE INDEXED DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
    val indexedTRAINbinaryDF = indexedTRAINbinary.toDF()
    val indexedTESTbinaryDF = indexedTESTbinary.toDF()
    val indexedTRAINregDF = indexedTRAINreg.toDF()
    val indexedTESTregDF = indexedTESTreg.toDF()

    # CREATE ONE-HOT ENCODED (VECTORIZED) DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
    val assemblerOneHot = new VectorAssembler().setInputCols(Array("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount")).setOutputCol("features")
    val OneHotTRAIN = assemblerOneHot.transform(trainData)
    val OneHotTEST = assemblerOneHot.transform(testData)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰4 秒。

### <a name="automatically-categorize-and-vectorize-features-and-targets-to-use-as-inputs-for-machine-learning-models"></a>自動分類及向量化特徵和目標，以做為機器學習模型的輸入
使用 Spark ML 分類目標和特徵，以便在以樹狀結構為基礎的模型化函式中使用。 程式碼會完成兩項工作︰

* 使用臨界值 0.5，將值 0 或 1 指派給介於 0 和 1 之間的每個資料點，以建立分類的二進位目標。
* 自動將功能分類。 如果任何特徵的相異數值的數目小於 32，該特徵就會進行分類。

以下是這兩項工作的程式碼。

    # CATEGORIZE FEATURES AND BINARIZE THE TARGET FOR THE BINARY CLASSIFICATION PROBLEM

    # TRAIN DATA
    val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
    val indexerModel = indexer.fit(indexedTRAINbinaryDF)
    val indexedTrainwithCatFeat = indexerModel.transform(indexedTRAINbinaryDF)
    val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
    val indexedTRAINwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

    # TEST DATA
    val indexerModel = indexer.fit(indexedTESTbinaryDF)
    val indexedTrainwithCatFeat = indexerModel.transform(indexedTESTbinaryDF)
    val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
    val indexedTESTwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

    # CATEGORIZE FEATURES FOR THE REGRESSION PROBLEM
    # CREATE PROPERLY INDEXED AND CATEGORIZED DATA FRAMES FOR TREE-BASED MODELS

    # TRAIN DATA
    val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
    val indexerModel = indexer.fit(indexedTRAINregDF)
    val indexedTRAINwithCatFeat = indexerModel.transform(indexedTRAINregDF)

    # TEST DATA
    val indexerModel = indexer.fit(indexedTESTbinaryDF)
    val indexedTESTwithCatFeat = indexerModel.transform(indexedTESTregDF)



## <a name="binary-classification-model-predict-whether-a-tip-should-be-paid"></a>二進位分類模型：預測是否應支付小費
在本節中，您會建立三種類型的二進位分類模型來預測是否應支付小費：

* 使用 Spark ML `LogisticRegression()` 函式的**羅吉斯迴歸模型**
* 使用 Spark ML `RandomForestClassifier()` 函式的**隨機樹系分類模型**
* 使用 MLlib `GradientBoostedTrees()` 函式的**梯度推進樹分類模型**

### <a name="create-a-logistic-regression-model"></a>建立羅吉斯迴歸模型
接著，使用 Spark ML `LogisticRegression()` 函式建立羅吉斯迴歸模型。 您會在一系列步驟中建立模型建置程式碼︰

1. **訓練模型** 資料。
2. **評估模型** 。
3. **儲存模型** 供未來取用。
4. **評分模型** 。
5. **繪製結果** 。

以下是這些程序的程式碼：

    # CREATE A LOGISTIC REGRESSION MODEL
    val lr = new LogisticRegression().setLabelCol("tipped").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)
    val lrModel = lr.fit(OneHotTRAIN)

    # PREDICT ON THE TEST DATA SET
    val predictions = lrModel.transform(OneHotTEST)

    # SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
    val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
    val ROC = evaluator.evaluate(predictions)
    println("ROC on test data = " + ROC)

    # SAVE THE MODEL
    val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
    val modelName = "LogisticRegression__"
    val filename = modelDir.concat(modelName).concat(datestamp)
    lrModel.save(filename);

載入、評分以及儲存結果。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # LOAD THE SAVED MODEL AND SCORE THE TEST DATA SET
    val savedModel = org.apache.spark.ml.classification.LogisticRegressionModel.load(filename)
    println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

    # SCORE THE MODEL ON THE TEST DATA
    val predictions = savedModel.transform(OneHotTEST).select("tipped","probability","rawPrediction")
    predictions.registerTempTable("testResults")

    # SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
    val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
    val ROC = evaluator.evaluate(predictions)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    # PRINT THE ROC RESULTS
    println("ROC on test data = " + ROC)


**輸出：**

測試資料的 ROC = 0.9827381497557599

在本機 Pandas 資料框架上使用 Python 來繪製 ROC 曲線。

    # QUERY THE RESULTS
    %%sql -q -o sqlResults
    SELECT tipped, probability from testResults


    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    %matplotlib inline
    from sklearn.metrics import roc_curve,auc

    sqlResults['probFloat'] = sqlResults.apply(lambda row: row['probability'].values()[0][1], axis=1)
    predictions_pddf = sqlResults[["tipped","probFloat"]]

    # PREDICT THE ROC CURVE
    # predictions_pddf = sqlResults.rename(columns={'_1': 'probability', 'tipped': 'label'})
    prob = predictions_pddf["probFloat"]
    fpr, tpr, thresholds = roc_curve(predictions_pddf['tipped'], prob, pos_label=1);
    roc_auc = auc(fpr, tpr)

    # PLOT THE ROC CURVE
    plt.figure(figsize=(5,5))
    plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
    plt.plot([0, 1], [0, 1], 'k--')
    plt.xlim([0.0, 1.0])
    plt.ylim([0.0, 1.05])
    plt.xlabel('False Positive Rate')
    plt.ylabel('True Positive Rate')
    plt.title('ROC Curve')
    plt.legend(loc="lower right")
    plt.show()


**輸出：**

![小費或沒有小費的 ROC 曲線](./media/scala-walkthrough/plot-roc-curve-tip-or-not.png)

### <a name="create-a-random-forest-classification-model"></a>建立隨機樹系分類模型
接著，使用 SparkML `RandomForestClassifier()` 函式建立隨機樹系分類模型，然後對測試資料評估模型。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # CREATE THE RANDOM FOREST CLASSIFIER MODEL
    val rf = new RandomForestClassifier().setLabelCol("labelBin").setFeaturesCol("featuresCat").setNumTrees(10).setSeed(1234)

    # FIT THE MODEL
    val rfModel = rf.fit(indexedTRAINwithCatFeatBinTarget)
    val predictions = rfModel.transform(indexedTESTwithCatFeatBinTarget)

    # EVALUATE THE MODEL
    val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
    val Test_f1Score = evaluator.evaluate(predictions)
    println("F1 score on test data: " + Test_f1Score);

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    # CALCULATE BINARY CLASSIFICATION EVALUATION METRICS
    val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setRawPredictionCol("probability").setMetricName("areaUnderROC")
    val ROC = evaluator.evaluate(predictions)
    println("ROC on test data = " + ROC)


**輸出：**

測試資料的 ROC = 0.9847103571552683

### <a name="create-a-gbt-classification-model"></a>建立 GBT 分類模型
接著，使用 MLlib 的 `GradientBoostedTrees()` 函式建立 GBT 分類模型，然後對測試資料評估模型。

    # TRAIN A GBT CLASSIFICATION MODEL BY USING MLLIB AND A LABELED POINT

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # DEFINE THE GBT CLASSIFICATION MODEL
    val boostingStrategy = BoostingStrategy.defaultParams("Classification")
    boostingStrategy.numIterations = 20
    boostingStrategy.treeStrategy.numClasses = 2
    boostingStrategy.treeStrategy.maxDepth = 5
    boostingStrategy.treeStrategy.categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

    # TRAIN THE MODEL
    val gbtModel = GradientBoostedTrees.train(indexedTRAINbinary, boostingStrategy)

    # SAVE THE MODEL IN BLOB STORAGE
    val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
    val modelName = "GBT_Classification__"
    val filename = modelDir.concat(modelName).concat(datestamp)
    gbtModel.save(sc, filename);

    # EVALUATE THE MODEL ON TEST INSTANCES AND THE COMPUTE TEST ERROR
    val labelAndPreds = indexedTESTbinary.map { point =>
      val prediction = gbtModel.predict(point.features)
      (point.label, prediction)
    }
    val testErr = labelAndPreds.filter(r => r._1 != r._2).count.toDouble / indexedTRAINbinary.count()
    //println("Learned classification GBT model:\n" + gbtModel.toDebugString)
    println("Test Error = " + testErr)

    # USE BINARY AND MULTICLASS METRICS TO EVALUATE THE MODEL ON THE TEST DATA
    val metrics = new MulticlassMetrics(labelAndPreds)
    println(s"Precision: ${metrics.precision}")
    println(s"Recall: ${metrics.recall}")
    println(s"F1 Score: ${metrics.fMeasure}")

    val metrics = new BinaryClassificationMetrics(labelAndPreds)
    println(s"Area under PR curve: ${metrics.areaUnderPR}")
    println(s"Area under ROC curve: ${metrics.areaUnderROC}")

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    # PRINT THE ROC METRIC
    println(s"Area under ROC curve: ${metrics.areaUnderROC}")


**輸出：**

ROC 曲線夏的領域 = 0.9846895479241554

## <a name="regression-model-predict-tip-amount"></a>迴歸模型：預測小費金額
在本節中，您會建立兩種類型的迴歸模型來預測小費金額︰

* 使用 Spark ML `LinearRegression()` 函式的**正規化線性迴歸模型**。 您將儲存模型，並對測試資料評估模型。
* 使用 Spark ML `GBTRegressor()` 函式的**梯度推進樹迴歸模型**。

### <a name="create-a-regularized-linear-regression-model"></a>建立正則化線性迴歸模型
    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # CREATE A REGULARIZED LINEAR REGRESSION MODEL BY USING THE SPARK ML FUNCTION AND DATA FRAMES
    val lr = new LinearRegression().setLabelCol("tip_amount").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)

    # FIT THE MODEL BY USING DATA FRAMES
    val lrModel = lr.fit(OneHotTRAIN)
    println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

    # SUMMARIZE THE MODEL OVER THE TRAINING SET AND PRINT METRICS
    val trainingSummary = lrModel.summary
    println(s"numIterations: ${trainingSummary.totalIterations}")
    println(s"objectiveHistory: ${trainingSummary.objectiveHistory.toList}")
    trainingSummary.residuals.show()
    println(s"RMSE: ${trainingSummary.rootMeanSquaredError}")
    println(s"r2: ${trainingSummary.r2}")

    # SAVE THE MODEL IN AZURE BLOB STORAGE
    val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
    val modelName = "LinearRegression__"
    val filename = modelDir.concat(modelName).concat(datestamp)
    lrModel.save(filename);

    # PRINT THE COEFFICIENTS
    println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

    # SCORE THE MODEL ON TEST DATA
    val predictions = lrModel.transform(OneHotTEST)

    # EVALUATE THE MODEL ON TEST DATA
    val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
    val r2 = evaluator.evaluate(predictions)
    println("R-sqr on test data = " + r2)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰13 秒。

    # LOAD A SAVED LINEAR REGRESSION MODEL FROM BLOB STORAGE AND SCORE A TEST DATA SET

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # LOAD A SAVED LINEAR REGRESSION MODEL FROM AZURE BLOB STORAGE
    val savedModel = org.apache.spark.ml.regression.LinearRegressionModel.load(filename)
    println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

    # SCORE THE MODEL ON TEST DATA
    val predictions = savedModel.transform(OneHotTEST).select("tip_amount","prediction")
    predictions.registerTempTable("testResults")

    # EVALUATE THE MODEL ON TEST DATA
    val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
    val r2 = evaluator.evaluate(predictions)
    println("R-sqr on test data = " + r2)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    # PRINT THE RESULTS
    println("R-sqr on test data = " + r2)


**輸出：**

測試資料的 R-sqr = 0.5960320470835743

接著，查詢測試結果做為資料框架，並使用 AutoVizWidget 和 matplotlib 將其視覺化呈現。

    # RUN A SQL QUERY
    %%sql -q -o sqlResults
    select * from testResults

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER
    %%local

    # USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES
    # CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, AND SO ON)
    sqlResults

此程式碼從查詢輸出中建立本機資料框架，以及繪製資料。 `%%local` magic 建立本機資料框架 `sqlResults`，您可以用來搭配 matplotlib 進行繪製。

> [!NOTE]
> 本文中會多次使用此 Spark magic。 如果資料總量很大，您應該取樣以建立可容納於本機記憶體的資料框架。
> 
> 

使用 Python matplotlib 建立繪圖。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    sqlResults
    %matplotlib inline
    import numpy as np

    # PLOT THE RESULTS
    ax = sqlResults.plot(kind='scatter', figsize = (6,6), x='tip_amount', y='prediction', color='blue', alpha = 0.25, label='Actual vs. predicted');
    fit = np.polyfit(sqlResults['tip_amount'], sqlResults['prediction'], deg=1)
    ax.set_title('Actual vs. Predicted Tip Amounts ($)')
    ax.set_xlabel("Actual")
    ax.set_ylabel("Predicted")
    #ax.plot(sqlResults['tip_amount'], fit[0] * sqlResults['prediction'] + fit[1], color='magenta')
    plt.axis([-1, 15, -1, 8])
    plt.show(ax)

**輸出：**

![小費金額︰實際與預測](./media/scala-walkthrough/plot-actual-vs-predicted-tip-amount.png)

### <a name="create-a-gbt-regression-model"></a>建立 GBT 迴歸模型
使用 SparkML `GBTRegressor()` 函式建立 GBT 迴歸模型，然後對測試資料評估模型。

[梯度推進樹](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) 集合了所有決策樹。 GBT 反覆地訓練決策樹以盡可能降低遺失函式。 您可以使用 GBT 進行迴歸和分類。 GBT 可以處理分類特徵、不需要調整特徵，而且可以擷取非線性和特徵互動。 您也可以在多類別分類設定中使用 GBT。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # TRAIN A GBT REGRESSION MODEL
    val gbt = new GBTRegressor().setLabelCol("label").setFeaturesCol("featuresCat").setMaxIter(10)
    val gbtModel = gbt.fit(indexedTRAINwithCatFeat)

    # MAKE PREDICTIONS
    val predictions = gbtModel.transform(indexedTESTwithCatFeat)

    # COMPUTE TEST SET R2
    val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
    val Test_R2 = evaluator.evaluate(predictions)


    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    # PRINT THE RESULTS
    println("Test R-sqr is: " + Test_R2);


**輸出：**

測試 R-sqr：0.7655383534596654

## <a name="advanced-modeling-utilities-for-optimization"></a>可供最佳化的進階模型化公用程式
在本節中，您將使用開發人員經常使用的機器學習公用程式來最佳化模型。 具體來說，您可以使用參數掃掠和交叉驗證，以三種不同的方式來最佳化機器學習模型︰

* 將資料分為訓練集和驗證集，對訓練集使用超參數掃掠來最佳化模型，然後對驗證集進行評估 (線性迴歸)
* 利用 Spark ML 的 CrossValidator 函式 (二進位分類)，使用交叉驗證和超參數掃掠來最佳化模型
* 使用自訂交叉驗證和參數掃掠程式碼，利用任何機器學習函式和參數集來最佳化模型 (線性迴歸)

**交叉驗證** 是一種技術，評估已對一組已知資料訓練過的模型將能多廣泛地預測尚未對其訓練過的資料集特徵。 這種技術背後的基本概念是模型已針對已知資料的資料集訓練，然後針對獨立的資料集測試其預測的精確度。 常見的實作是將資料集分割成 *k*個摺疊，然後以循環配置資源方式對除了一個摺疊的所有摺疊訓練模型。

**超參數最佳化** 的問題是要選擇一組超參數來學習演算法，目標通常是要最佳化獨立資料集上演算法效能的測量。 超參數是您在模型訓練程序之外必須指定的值。 對於超參數值的假設可能會影響模型的彈性和精確度。 決策樹有超參數，例如想要的樹深度和葉數目等。 您必須為支援向量機器 (SVM) 設定分類誤判損失詞彙。

執行超參數最佳化一個常見的方法是使用格線搜尋，也稱為 **參數掃掠**。 在格線搜尋中，會對某個學習演算法的超參數空間指定子集的所有值執行詳盡搜尋。 交叉驗證可以提供效能度量，挑出格線搜尋演算法所產生的最佳結果。 如果您使用交叉驗證超參數掃掠，可以協助限制例如將模型過度擬合到訓練資料的問題。 如此一來，模型會保留空間來套用到擷取訓練資料的一般資料集。

### <a name="optimize-a-linear-regression-model-with-hyper-parameter-sweeping"></a>利用超參數掃掠來最佳化線性迴歸模型
接著，將資料分為訓練集和驗證集，對訓練集使用超參數掃掠來最佳化模型，然後對驗證集進行評估 (線性迴歸)。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # RENAME `tip_amount` AS A LABEL
    val OneHotTRAINLabeled = OneHotTRAIN.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
    val OneHotTESTLabeled = OneHotTEST.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
    OneHotTRAINLabeled.cache()
    OneHotTESTLabeled.cache()

    # DEFINE THE ESTIMATOR FUNCTION: `THE LinearRegression()` FUNCTION
    val lr = new LinearRegression().setLabelCol("label").setFeaturesCol("features").setMaxIter(10)

    # DEFINE THE PARAMETER GRID
    val paramGrid = new ParamGridBuilder().addGrid(lr.regParam, Array(0.1, 0.01, 0.001)).addGrid(lr.fitIntercept).addGrid(lr.elasticNetParam, Array(0.1, 0.5, 0.9)).build()

    # DEFINE THE PIPELINE WITH A TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET), AND THEN THE SPECIFY ESTIMATOR, EVALUATOR, AND PARAMETER GRID
    val trainPct = 0.75
    val trainValidationSplit = new TrainValidationSplit().setEstimator(lr).setEvaluator(new RegressionEvaluator).setEstimatorParamMaps(paramGrid).setTrainRatio(trainPct)

    # RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
    val model = trainValidationSplit.fit(OneHotTRAINLabeled)

    # MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
    val testResults = model.transform(OneHotTESTLabeled).select("label", "prediction")

    # COMPUTE TEST SET R2
    val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
    val Test_R2 = evaluator.evaluate(testResults)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");

    println("Test R-sqr is: " + Test_R2);


**輸出：**

測試 R-sqr：0.6226484708501209

### <a name="optimize-the-binary-classification-model-by-using-cross-validation-and-hyper-parameter-sweeping"></a>使用交叉驗證和超參數掃掠來最佳化二進位分類模型
本節示範如何使用交叉驗證和超參數掃掠來最佳化二進位分類模型。 這會使用 Spark ML `CrossValidator` 函式。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # CREATE DATA FRAMES WITH PROPERLY LABELED COLUMNS TO USE WITH THE TRAIN AND TEST SPLIT
    val indexedTRAINwithCatFeatBinTargetRF = indexedTRAINwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
    val indexedTESTwithCatFeatBinTargetRF = indexedTESTwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
    indexedTRAINwithCatFeatBinTargetRF.cache()
    indexedTESTwithCatFeatBinTargetRF.cache()

    # DEFINE THE ESTIMATOR FUNCTION
    val rf = new RandomForestClassifier().setLabelCol("label").setFeaturesCol("features").setImpurity("gini").setSeed(1234).setFeatureSubsetStrategy("auto").setMaxBins(32)

    # DEFINE THE PARAMETER GRID
    val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(4,8)).addGrid(rf.numTrees, Array(5,10)).addGrid(rf.minInstancesPerNode, Array(100,300)).build()

    # SPECIFY THE NUMBER OF FOLDS
    val numFolds = 3

    # DEFINE THE TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET)
    val CrossValidator = new CrossValidator().setEstimator(rf).setEvaluator(new BinaryClassificationEvaluator).setEstimatorParamMaps(paramGrid).setNumFolds(numFolds)

    # RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
    val model = CrossValidator.fit(indexedTRAINwithCatFeatBinTargetRF)

    # MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
    val testResults = model.transform(indexedTESTwithCatFeatBinTargetRF).select("label", "prediction")

    # COMPUTE THE TEST F1 SCORE
    val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
    val Test_f1Score = evaluator.evaluate(testResults)

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


**輸出：**

執行資料格的時間︰33 秒。

### <a name="optimize-the-linear-regression-model-by-using-custom-cross-validation-and-parameter-sweeping-code"></a>使用自訂交叉驗證和參數掃掠程式碼來最佳化線性迴歸模型
接著，使用自訂程式碼來最佳化模型，並使用最精確的準則來找出最佳的模型參數。 然後，建立最終模型、對測試資料評估模型，以及在 Blob 儲存體中儲存模型。 最後，載入模型、對測試資料進行評分，以及評估精確度。

    # RECORD THE START TIME
    val starttime = Calendar.getInstance().getTime()

    # DEFINE THE PARAMETER GRID AND THE NUMBER OF FOLDS
    val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(5,10)).addGrid(rf.numTrees, Array(10,25,50)).build()

    val nFolds = 3
    val numModels = paramGrid.size
    val numParamsinGrid = 2

    # SPECIFY THE NUMBER OF CATEGORIES FOR CATEGORICAL VARIABLES
    val categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

    var maxDepth = -1
    var numTrees = -1
    var param = ""
    var paramval = -1
    var validateLB = -1.0
    var validateUB = -1.0
    val h = 1.0 / nFolds;
    val RMSE  = Array.fill(numModels)(0.0)

    # CREATE K-FOLDS
    val splits = MLUtils.kFold(indexedTRAINbinary, numFolds = nFolds, seed=1234)


    # LOOP THROUGH K-FOLDS AND THE PARAMETER GRID TO GET AND IDENTIFY THE BEST PARAMETER SET BY LEVEL OF ACCURACY
    for (i <- 0 to (nFolds-1)) {
        validateLB = i * h
        validateUB = (i + 1) * h
        val validationCV = trainData.filter($"rand" >= validateLB  && $"rand" < validateUB)
        val trainCV = trainData.filter($"rand" < validateLB  || $"rand" >= validateUB)
        val validationLabPt = validationCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
        val trainCVLabPt = trainCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
        validationLabPt.cache()
        trainCVLabPt.cache()

        for (nParamSets <- 0 to (numModels-1)) {
            for (nParams <- 0 to (numParamsinGrid-1)) {
                param = paramGrid(nParamSets).toSeq(nParams).param.toString.split("__")(1)
                paramval = paramGrid(nParamSets).toSeq(nParams).value.toString.toInt
                if (param == "maxDepth") {maxDepth = paramval}
                if (param == "numTrees") {numTrees = paramval}
            }
            val rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                      numTrees=numTrees, maxDepth=maxDepth,
                                                      featureSubsetStrategy="auto",impurity="variance", maxBins=32)
            val labelAndPreds = validationLabPt.map { point =>
                                                     val prediction = rfModel.predict(point.features)
                                                     ( prediction, point.label )
                                                    }
            val validMetrics = new RegressionMetrics(labelAndPreds)
            val rmse = validMetrics.rootMeanSquaredError
            RMSE(nParamSets) += rmse
        }
        validationLabPt.unpersist();
        trainCVLabPt.unpersist();
    }
    val minRMSEindex = RMSE.indexOf(RMSE.min)

    # GET THE BEST PARAMETERS FROM A CROSS-VALIDATION AND PARAMETER SWEEP
    var best_maxDepth = -1
    var best_numTrees = -1
    for (nParams <- 0 to (numParamsinGrid-1)) {
        param = paramGrid(minRMSEindex).toSeq(nParams).param.toString.split("__")(1)
        paramval = paramGrid(minRMSEindex).toSeq(nParams).value.toString.toInt
        if (param == "maxDepth") {best_maxDepth = paramval}
        if (param == "numTrees") {best_numTrees = paramval}
    }

    # CREATE THE BEST MODEL WITH THE BEST PARAMETERS AND A FULL TRAINING DATA SET
    val best_rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                      numTrees=best_numTrees, maxDepth=best_maxDepth,
                                                      featureSubsetStrategy="auto",impurity="variance", maxBins=32)

    # SAVE THE BEST RANDOM FOREST MODEL IN BLOB STORAGE
    val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
    val modelName = "BestCV_RF_Regression__"
    val filename = modelDir.concat(modelName).concat(datestamp)
    best_rfModel.save(sc, filename);

    # PREDICT ON THE TRAINING SET WITH THE BEST MODEL AND THEN EVALUATE
    val labelAndPreds = indexedTESTreg.map { point =>
                                            val prediction = best_rfModel.predict(point.features)
                                            ( prediction, point.label )
                                           }

    val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
    val test_rsqr = new RegressionMetrics(labelAndPreds).r2

    # GET THE TIME TO RUN THE CELL
    val endtime = Calendar.getInstance().getTime()
    val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
    println("Time taken to run the above cell: " + elapsedtime + " seconds.");


    # LOAD THE MODEL
    val savedRFModel = RandomForestModel.load(sc, filename)

    val labelAndPreds = indexedTESTreg.map { point =>
                                            val prediction = savedRFModel.predict(point.features)
                                            ( prediction, point.label )
                                           }
    # TEST THE MODEL
    val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
    val test_rsqr = new RegressionMetrics(labelAndPreds).r2


**輸出：**

執行資料格的時間︰61 秒。

## <a name="consume-spark-built-machine-learning-models-automatically-with-scala"></a>透過 Scala 自動取用 Spark 建置的機器學習模型
如需能引導您完成在 Azure 中構成資料科學程序之工作的主題概觀，請參閱 [Team Data Science Process](http://aka.ms/datascienceprocess)。

[Team Data Science Process 逐步解說](walkthroughs.md) 說明的其他端對端逐步解說示範 Team Data Science Process 針對特定案例的步驟。 這些逐步解說也示範如何將雲端和內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。

[評分 Spark 建置的機器學習模型](spark-model-consumption.md) 示範如何利用 Spark 內建並儲存於 Azure Blob 儲存體的機器學習模型，使用 Scala 程式碼來自動載入及評分新資料集。 您可以遵循此處提供的指示，只要將本文中的 Python 程式碼取代為 Scala 程式碼，即可自動取用。

