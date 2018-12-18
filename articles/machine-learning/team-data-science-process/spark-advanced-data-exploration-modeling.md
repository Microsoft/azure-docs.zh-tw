---
title: 使用 Spark 進階資料探索和模型化 | Microsoft Docs
description: 使用 HDInsight Spark 並透過交叉驗證和超參數最佳化定型模型，執行資料探索和二進位分類和迴歸模型化。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: f90d9a80-4eaf-437b-a914-23514390cd60
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/15/2017
ms.author: deguhath
ms.openlocfilehash: 3674a6d629789392aba341f470bae83867331dfe
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394709"
---
# <a name="advanced-data-exploration-and-modeling-with-spark"></a>使用 Spark 進階資料探索和模型化
[!INCLUDE [machine-learning-spark-modeling](../../../includes/machine-learning-spark-modeling.md)]

本逐步解說會使用 HDInsight Spark，在 NYC 計程車車程和費用 2013 資料集的取樣上，使用交叉驗證和超參數最佳化來執行資料探索和定型二進位分類和迴歸模型。 它將引導您逐步完成 [資料科學程序](http://aka.ms/datascienceprocess)、端對端、使用 HDInsight Spark 叢集處理，並使用 Azure blob 來儲存資料和模型。 程序會探索和視覺化從 Azure 儲存體 Blob 中引進的資料，然後準備資料來建立預測模型。 已使用 Python 來編寫解決方案程式碼，並顯示相關的繪圖。 這些模型是使用 Spark MLlib 工具組來執行二進位分類和迴歸模型工作。 

* **二進位分類** 工作可預測是否已支付某趟車程的小費。 
* **迴歸** 工作可根據其他小費功能來預測小費金額。 

模型化步驟也包含程式碼來示範如何定型、評估和儲存每類模型。 這個主題涵蓋一些與[使用 Spark 資料探索和模型化](spark-data-exploration-modeling.md)主題相同的內容， 但更「進階」；這是指它也會使用交叉驗證搭配超參數掃掠，以最佳方式定型精確的分類和迴歸模型。 

**交叉驗證 (CV)** 是一種技術，評估以一組已知資料定型的模型如何一般化，以預測在其上尚未定型的資料集的功能。  此處所使用的通用實作是將資料集分割成 K 摺疊，然後以循環配置資源方式定型所有摺疊，只留下其中一個摺疊。 評估測試未定型的此摺疊中獨立資料集時精確預測模型的能力。

**超參數最佳化** 是選擇用於學習演算法的超參數集的問題所在，通常具有最佳化獨立資料集上演算法效能的測量的目標。 **超參數** 是值，必須在模型定型程序之外指定。 這些值的假設可能會影響模型的彈性和精確度。 決策樹有超參數，例如，想要的深度和樹狀目錄中的分葉數目等。 支援向量機器 (SVM) 需要設定分類誤判損失詞彙。 

此處所使用的執行超參數最佳化常見做法是格線搜尋，或 **參數掃掠**。 是透過值執行詳盡搜尋，而該值是學習演算法的超參數空間的指定子集。 交叉驗證可以提供效能度量，挑出格線搜尋演算法所產生的最佳結果。 CV 與超參數掃掠搭配使用，有助於限制問題，例如定型資料模型的過度配適，以便模型會保留容量以套用至從中擷取定型資料的一般集合。

我們使用的模型包括羅吉斯和線性迴歸、隨機樹系和漸層停駐推進式決策樹︰

* [使用 SGD 的線性迴歸](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) 是一種使用隨機梯度下降 (SGD) 方法的線性迴歸模型，並使用最佳化和功能縮放比例來預測支付的小費金額。 
* [使用 LBFGS 的羅吉斯迴歸](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.classification.LogisticRegressionWithLBFGS) 或「對數優劣比」迴歸是使用相依變數來執行資料分類時，可使用的迴歸模型。 LBFGS 是牛頓最佳化演算法，可使用有限的電腦記憶體量來逼近 Broyden–Fletcher–Goldfarb–Shanno (BFGS) 演算法，且廣泛用於機器學習中。
* [隨機樹系](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) 是整體的決策樹。  隨機樹系結合許多決策樹來降低風險過度膨脹。 隨機樹系適用於迴歸和分類，可處理分類功能，也可擴充至多類別分類設定。 隨機樹系不需要調整功能，而且能夠擷取非線性和功能互動。 隨機樹系是其中一個最成功的分類和迴歸的機器學習模型。
* [漸層停駐推進式決策樹](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) 是整體的決策樹。 GBT 反覆地訓練決策樹以盡可能降低遺失函式。 GBT 適用於迴歸和分類，並可處理分類功能、不需要調整功能，而且能夠擷取非線性和功能互動。 它們也可用於多類別分類設定。

使用 CV 和超參數掃掠的模型化範例會針對二進位分類問題顯示。 更簡單的範例 (不含參數掃掠) 會出現在迴歸工作的主要主題中。 但是在附錄中，另外也會提到針對線性迴歸使用彈性 net 的驗證，和針對隨機樹系迴歸使用 CV 和參數掃掠的驗證。 **彈性 net** 是正規化迴歸方法，以符合線性迴歸模型，線性結合 L1 和 L2 度量做為 [lasso](https://en.wikipedia.org/wiki/Lasso%20%28statistics%29) 和 [ridge](https://en.wikipedia.org/wiki/Tikhonov_regularization) 方法的處罰。   

> [!NOTE]
> 雖然 Spark MLlib 工具組設計成可處理大型資料集，在這裡為了簡便我們使用極小的範例 (使用 170 個資料列，原始的 NYC 資料集約 0.1%，~30 Mb)。 此處提供的練習有效地在含有 2 個背景工作節點的 HDInsight 叢集上執行 (大約 10 分鐘)。 相同的程式碼中 (稍加修改) 可以用來處理大型資料集，適當修改的程式碼則可快取記憶體的資料以及變更叢集大小。
> 
> 

## <a name="setup-spark-clusters-and-notebooks"></a>設定：Spark 叢集和 Notebook
此逐步解說所提供的設定步驟和程式碼適用於使用 HDInsight Spark 1.6。 不過 Jupyter Notebook 可供 HDInsight Spark 1.6 版和 Spark 2.0 叢集兩者使用。 Notebook 的描述及它們的連結已在包含它們的 GitHub 儲存機制的 [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) 中提供。 此外，此處及連結的 Notebook 內的程式碼皆屬泛型程式碼，而且應該能在任何 Spark 叢集上運作。 若您不是使用 HDInsight Spark，叢集設定和管理步驟可能與這裡顯示的稍有不同。 為了方便起見，以下是可讓 Spark 1.6 版和 2.0 版在 Jupyter Notebook 伺服器的 pyspark 核心中執行的 Jupyter Notebook 連結：

### <a name="spark-16-notebooks"></a>Spark 1.6 Notebook

[pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb)：包含Notebook #1 中的主題，以及使用超參數微調和交叉驗證的模型開發。

### <a name="spark-20-notebooks"></a>Spark 2.0 Notebook

[Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb)：此檔案提供如何在 Spark 2.0 叢集中執行資料瀏覽、模型化和評分的相關資訊。

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>安裝程式︰儲存體位置、程式庫和預設 Spark 內容
Spark 可以讀取和寫入 Azure 儲存體 Blob (也稱為 WASB)。 如此可使用 Spark 處理該處儲存的任何現有資料，並在 WASB 中再次儲存結果。

若要在 WASB 中儲存模型或檔案，必須正確指定路徑。 可以使用以「wasb:///」開頭的路徑，參考連接到 Spark 叢集的預設容器。 其他位置由「wasb://」參考。

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>在 WASB 中設定儲存位置的目錄路徑
下列程式碼範例會指定要讀取資料的位置，和將儲存模型輸出的模型儲存體目錄的路徑：

    # SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

    # LOCATION OF TRAINING DATA
    taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";


    # SET THE MODEL STORAGE DIRECTORY PATH 
    # NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
    modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";

    # PRINT START TIME
    import datetime
    datetime.datetime.now()

**輸出**

datetime.datetime(2016, 4, 18, 17, 36, 27, 832799)

### <a name="import-libraries"></a>匯入程式庫
使用下列程式碼匯入必要的程式庫：

    # LOAD PYSPARK LIBRARIES
    import pyspark
    from pyspark import SparkConf
    from pyspark import SparkContext
    from pyspark.sql import SQLContext
    import matplotlib
    import matplotlib.pyplot as plt
    from pyspark.sql import Row
    from pyspark.sql.functions import UserDefinedFunction
    from pyspark.sql.types import *
    import atexit
    from numpy import array
    import numpy as np
    import datetime


### <a name="preset-spark-context-and-pyspark-magics"></a>預設 Spark 內容及 PySpark magic
Jupyter Notebook 所提供的 PySpark 核心有預設的內容。 因此您不必明確設定 Spark 或 Hive 內容，就能開始使用您所開發的應用程式。 依預設會將這些內容提供給您使用。 這些內容包括：

* sc - 代表 Spark 
* sqlContext - 代表 Hive

PySpark 核心提供一些預先定義的「magic」，這是您可以使用 %% 呼叫的特殊命令。 在這些程式碼範例中，就使用了兩個此類型的命令。

* **%%local** 指定後續行所列的程式碼要在本機執行。 程式碼必須是有效的 Python 程式碼。
* **%%sql -o <variable name>** 會針對 sqlContext 執行 Hive 查詢。 如果傳遞 -o 參數，則查詢的結果會當做 Pandas 資料框架，保存在 %%local Python 內容中。

如需關於 Jupyter Notebook 核心，以及其所提供的預先定義 "magics" 的詳細資訊，請參閱 [HDInsight 上的 HDInsight Spark Linux 叢集可供 Jupyter Notebook 使用的核心](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md)。

## <a name="data-ingestion-from-public-blob"></a>來自公用 Blob 的資料擷取：
資料科學程序的第一個步驟，是將要分析的資料從其來源位置內嵌到您的資料探索和模型化環境。 這種環境在本逐步解說中是 Spark。 本節包含程式碼來完成一系列的工作︰

* 擷取要模型化的資料範例
* 在輸入資料集中讀取 (儲存為 .tsv 檔案)
* 格式化和清除資料
* 建立並快取記憶體中的物件 (RDD 或資料框架)
* 將它註冊為 SQL 內容中的暫存資料表。

以下是資料擷取的程式碼。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # IMPORT FILE FROM PUBLIC BLOB
    taxi_train_file = sc.textFile(taxi_train_file_loc)

    # GET SCHEMA OF THE FILE FROM HEADER
    schema_string = taxi_train_file.first()
    fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
    fields[7].dataType = IntegerType() #Pickup hour
    fields[8].dataType = IntegerType() # Pickup week
    fields[9].dataType = IntegerType() # Weekday
    fields[10].dataType = IntegerType() # Passenger count
    fields[11].dataType = FloatType() # Trip time in secs
    fields[12].dataType = FloatType() # Trip distance
    fields[19].dataType = FloatType() # Fare amount
    fields[20].dataType = FloatType() # Surcharge
    fields[21].dataType = FloatType() # Mta_tax
    fields[22].dataType = FloatType() # Tip amount
    fields[23].dataType = FloatType() # Tolls amount
    fields[24].dataType = FloatType() # Total amount
    fields[25].dataType = IntegerType() # Tipped or not
    fields[26].dataType = IntegerType() # Tip class
    taxi_schema = StructType(fields)

    # PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
    taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
    taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
            .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                            float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                            float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))


    # CREATE DATA FRAME
    taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

    # CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
    taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
        .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
        .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
        .drop('direct_distance').drop('surcharge')\
        .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

    # CACHE & MATERIALIZE DATA-FRAME IN MEMORY. GOING THROUGH AND COUNTING NUMBER OF ROWS MATERIALIZES THE DATA-FRAME IN MEMORY
    taxi_df_train_cleaned.cache()
    taxi_df_train_cleaned.count()

    # REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
    taxi_df_train_cleaned.registerTempTable("taxi_train")

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

執行上述儲存格所花費的時間︰276.62 秒

## <a name="data-exploration--visualization"></a>資料探索和虛擬化
一旦將資料引進 Spark，資料科學程序的下一個步驟是透過探索和虛擬化以更深入瞭解資料。 在本節中，我們會使用 SQL 查詢檢查計程車資料，並繪製目標變數和潛在功能以進行視覺檢查。 具體來說，我們會繪製計程車車程中的乘客計數頻率、小費金額的頻率，及小費如何隨付款金額和類型而異。

### <a name="plot-a-histogram-of-passenger-count-frequencies-in-the-sample-of-taxi-trips"></a>在計程車車程範例中繪製乘客計數頻率的長條圖
此程式碼與後續程式碼片段使用了 SQL magic 來查詢範例及本機 magic 以繪製資料。

* **SQL magic (`%%sql`)** HDInsight PySpark 核心支援針對 sqlContext 進行簡單的內嵌 HiveQL 查詢。 「-o VARIABLE_NAME」引數會將 SQL 查詢的輸出，保存為 Jupyter 伺服器上的 Pandas 資料框架。 這代表會在本機模式中使用此引數。
* **`%%local` magic** 是用來在 Jupyter 伺服器本機 (HDInsight 叢集的前端節點) 上執行程式碼。 一般來說，您使用 `%%sql -o` magic 之後使用 `%%local` magic 來執行查詢。 -o 參數會將 SQL 查詢的輸出保存在本機。 接著 `%%local` magic 會針對保存在本機之 SQL 查詢的輸出，觸發下一組程式碼片段在本機執行。 執行完程式碼後，輸出將會自動以視覺化方式呈現。

此查詢會擷取按乘客計數排列的車程。 

    # PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

    # SQL QUERY
    %%sql -q -o sqlResults
    SELECT passenger_count, COUNT(*) as trip_counts FROM taxi_train WHERE passenger_count > 0 and passenger_count < 7 GROUP BY passenger_count


此程式碼從查詢輸出中建立了本機資料框架，以及繪製資料。 `%%local` magic 建立了本機資料框架「`sqlResults`」，可用於搭配 Matplotlib 進行繪製。 

> [!NOTE]
> 在本逐步解說中，會多次使用此 PySpark magic。 如果資料總量很大，您應該取樣以建立可容納於本機記憶體的資料框架。
> 
> 

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER
    %%local

    # USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
    # CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
    sqlResults

以下是繪製按乘客計數排列車程的程式碼

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    import matplotlib.pyplot as plt
    %matplotlib inline

    # PLOT PASSENGER NUMBER VS TRIP COUNTS
    x_labels = sqlResults['passenger_count'].values
    fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
    fig.set_xticklabels(x_labels)
    fig.set_title('Counts of trips by passenger count')
    fig.set_xlabel('Passenger count in trips')
    fig.set_ylabel('Trip counts')
    plt.show()

**輸出**

![按乘客計數排列的車程頻率](./media/spark-advanced-data-exploration-modeling/frequency-of-trips-by-passenger-count.png)

在 Notebook 內使用 [類型]  功能表按鈕，您可以選擇幾種不同類型的視覺效果 (資料表、圓形圖、折線圖、區域圖或橫條圖)。 橫條圖繪製結果會在此顯示。

### <a name="plot-a-histogram-of-tip-amounts-and-how-tip-amount-varies-by-passenger-count-and-fare-amounts"></a>繪製小費金額，和小費金額如何隨乘客計數和費用金額變化的長條圖。
使用 SQL 查詢來取樣資料。

    # SQL SQUERY
    %%sql -q -o sqlResults
        SELECT fare_amount, passenger_count, tip_amount, tipped
        FROM taxi_train 
        WHERE passenger_count > 0 
        AND passenger_count < 7
        AND fare_amount > 0 
        AND fare_amount < 200
        AND payment_type in ('CSH', 'CRD')
        AND tip_amount > 0 
        AND tip_amount < 25


此程式碼儲存格使用 SQL 查詢來建立三種圖的資料。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    %matplotlib inline

    # TIP BY PAYMENT TYPE AND PASSENGER COUNT
    ax1 = resultsPDDF[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
    ax1.set_title('Tip amount distribution')
    ax1.set_xlabel('Tip Amount ($)')
    ax1.set_ylabel('Counts')
    plt.suptitle('')
    plt.show()

    # TIP BY PASSENGER COUNT
    ax2 = resultsPDDF.boxplot(column=['tip_amount'], by=['passenger_count'])
    ax2.set_title('Tip amount ($) by Passenger count')
    ax2.set_xlabel('Passenger count')
    ax2.set_ylabel('Tip Amount ($)')
    plt.suptitle('')
    plt.show()

    # TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
    ax = resultsPDDF.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(resultsPDDF.passenger_count))
    ax.set_title('Tip amount by Fare amount ($)')
    ax.set_xlabel('Fare Amount')
    ax.set_ylabel('Tip Amount')
    plt.axis([-2, 120, -2, 30])
    plt.show()


**輸出：** 

![小費金額分佈](./media/spark-advanced-data-exploration-modeling/tip-amount-distribution.png)

![按乘客計數排列的小費金額](./media/spark-advanced-data-exploration-modeling/tip-amount-by-passenger-count.png)

![按費用金額排列的小費金額](./media/spark-advanced-data-exploration-modeling/tip-amount-by-fare-amount.png)

## <a name="feature-engineering-transformation-and-data-preparation-for-modeling"></a>模型化的功能工程、轉換和資料準備
本節描述並提供程序的程式碼，可用來準備資料以供 ML 模型化使用。 它示範如何執行下列工作：

* 將小時分割到流量時間分類區以建立新功能
* 索引和 on-hot 編碼分類功能
* 建立輸入到 ML 函式的標示點物件
* 建立資料的隨機子取樣，並將它分割成訓練和測試集
* 調整功能
* 快取記憶體中的物件

### <a name="create-a-new-feature-by-partitioning-traffic-times-into-bins"></a>將流量時間分割到分類區以建立新功能
此程式碼示範如何將流量時間分割到分類區以建立新功能，以及如何從記憶體快取產生的資料框架。 在重複使用彈性分散式資料集 (RDD) 和資料框架的位置，快取可改善執行時間。 因此，我們會在逐步解說中的幾個階段快取 RDD 和資料框架。

    # CREATE FOUR BUCKETS FOR TRAFFIC TIMES
    sqlStatement = """
        SELECT *,
        CASE
         WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
         WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
         WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
         WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
        END as TrafficTimeBins
        FROM taxi_train 
    """
    taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

    # CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
    # THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
    # MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
    taxi_df_train_with_newFeatures.cache()
    taxi_df_train_with_newFeatures.count()

**輸出**

126050

### <a name="index-and-one-hot-encode-categorical-features"></a>索引和 one-hot 編碼分類功能
本節說明如何索引並編碼分類功能，以輸入模型化函式中。 MLlib 的模型化和預測函式需要先執行功能來分類要索引或編碼的分類輸入資料，才能使用這些資料。 

根據模型，您需要以不同方式索引或編碼它們。 例如，羅吉斯和線性迴歸模型需要一個有效編碼方式，例如，含 3 個類別的一個功能可擴充至 3 個功能資料行，根據觀察的類別，每個資料行包含 0 或 1。 MLlib 提供 [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) 函式以執行 one-hot 編碼。 此編碼器會將標籤索引資料行對應到二進位向量資料行 (最多有一個單一值)。 這種編碼方式允許將預期數值功能的演算法 (例如羅吉斯迴歸) 套用至分類功能。

以下是要索引及編碼分類功能的程式碼︰

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer

    # INDEX AND ENCODE VENDOR_ID
    stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
    model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
    indexed = model.transform(taxi_df_train_with_newFeatures)
    encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
    encoded1 = encoder.transform(indexed)

    # INDEX AND ENCODE RATE_CODE
    stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
    model = stringIndexer.fit(encoded1)
    indexed = model.transform(encoded1)
    encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
    encoded2 = encoder.transform(indexed)

    # INDEX AND ENCODE PAYMENT_TYPE
    stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
    model = stringIndexer.fit(encoded2)
    indexed = model.transform(encoded2)
    encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
    encoded3 = encoder.transform(indexed)

    # INDEX AND TRAFFIC TIME BINS
    stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
    model = stringIndexer.fit(encoded3)
    indexed = model.transform(encoded3)
    encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
    encodedFinal = encoder.transform(indexed)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

執行上述儲存格所花費的時間︰3.14 秒

### <a name="create-labeled-point-objects-for-input-into-ml-functions"></a>建立輸入到 ML 函式的標示點物件
本節包含的程式碼示範如何將分類的文字資料索引做為標示點資料類型以及如何編碼。 這可準備它用來訓練及測試 MLlib 羅吉斯迴歸和其他分類模型。 標示點物件是彈性分散式資料集 (RDD)，其格式化成適合 MLlib 中大部分的 ML 演算法的輸入資料。 [標示點](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) 是本機向量 (密集或疏鬆)，與標籤/回應相關聯。

以下是要為二進位分類進行索引及編碼文字功能的程式碼。

    # FUNCTIONS FOR BINARY CLASSIFICATION

    # LOAD LIBRARIES
    from pyspark.mllib.regression import LabeledPoint
    from numpy import array

    # INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
    def parseRowIndexingBinary(line):
        features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.pickup_hour, line.weekday,
                             line.passenger_count, line.trip_time_in_secs, line.trip_distance, line.fare_amount])
        labPt = LabeledPoint(line.tipped, features)
        return  labPt

    # ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
    def parseRowOneHotBinary(line):
        features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                            line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                   line.vendorVec.toArray(), line.rateVec.toArray(), line.paymentVec.toArray()), axis=0)
        labPt = LabeledPoint(line.tipped, features)
        return  labPt


以下是要為線性迴歸分析進行編碼及建立類別文字功能的程式碼。

    # FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

    # ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
    def parseRowIndexingRegression(line):
        features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                             line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                             line.trip_distance, line.fare_amount])
        labPt = LabeledPoint(line.tip_amount, features)
        return  labPt

    # INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
    def parseRowOneHotRegression(line):
        features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                            line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                            line.vendorVec.toArray(), line.rateVec.toArray(), 
                                            line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
        labPt = LabeledPoint(line.tip_amount, features)
        return  labPt


### <a name="create-a-random-sub-sampling-of-the-data-and-split-it-into-training-and-testing-sets"></a>建立資料的隨機子取樣，並將它分割成訓練和測試集
此程式碼會建立隨機取樣資料 (這裡使用 25%)。 雖然您不需要這個範例 (因為資料集的大小)，我們將在這裡示範如何取樣， 這樣您就了解如何在需要時使用它來自行解決問題。 當樣本非常大時，這樣可在訓練模型時節省大量時間。 接下來我們將範例分割成訓練部分 (這裡為 75%) 和測試部分 (這裡為 25%)，以便在分類和迴歸模型化中使用。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # SPECIFY SAMPLING AND SPLITTING FRACTIONS
    from pyspark.sql.functions import rand

    samplingFraction = 0.25;
    trainingFraction = 0.75; testingFraction = (1-trainingFraction);
    seed = 1234;
    encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)

    # SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST, WITH A RANDOM COLUMN ADDED FOR DOING CV (SHOWN LATER)
    # INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS
    dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
    trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);

    # CACHE TRAIN AND TEST DATA
    trainData.cache()
    testData.cache()

    # FOR BINARY CLASSIFICATION TRAINING AND TESTING
    indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
    indexedTESTbinary = testData.map(parseRowIndexingBinary)
    oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
    oneHotTESTbinary = testData.map(parseRowOneHotBinary)

    # FOR REGRESSION TRAINING AND TESTING
    indexedTRAINreg = trainData.map(parseRowIndexingRegression)
    indexedTESTreg = testData.map(parseRowIndexingRegression)
    oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
    oneHotTESTreg = testData.map(parseRowOneHotRegression)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出**

執行上述儲存格所花費的時間︰0.31 秒

### <a name="feature-scaling"></a>調整功能
調整功能，也稱為資料正規化，以確保具廣泛分散值的功能在目標函式中沒有過多權重。 用於調整功能的程式碼會使用 [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) ，將功能調整至單位變異數。 這是由 MLlib 提供，用於使用隨機梯度下降 (SGD) 的線性迴歸。 SGD 為訓練廣泛的其他機器學習模型的常用演算法，例如正則化迴歸或支援向量機器 (SVM)。   

> [!TIP]
> 我們找到了適用於調整功能的 LinearRegressionWithSGD 演算法。   
> 
> 

以下是要用於正規化線性 SGD 演算法的比例調整變數的程式碼。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.regression import LabeledPoint
    from pyspark.mllib.linalg import Vectors
    from pyspark.mllib.feature import StandardScaler, StandardScalerModel
    from pyspark.mllib.util import MLUtils

    # SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
    label = oneHotTRAINreg.map(lambda x: x.label)
    features = oneHotTRAINreg.map(lambda x: x.features)
    scaler = StandardScaler(withMean=False, withStd=True).fit(features)
    dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
    oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

    label = oneHotTESTreg.map(lambda x: x.label)
    features = oneHotTESTreg.map(lambda x: x.features)
    scaler = StandardScaler(withMean=False, withStd=True).fit(features)
    dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
    oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出**

執行上述儲存格所花費的時間︰11.67 秒

### <a name="cache-objects-in-memory"></a>快取記憶體中的物件
快取用於分類、迴歸和縮放功能的輸入資料框架物件，可降低訓練和測試 ML 演算法所花費的時間。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # FOR BINARY CLASSIFICATION TRAINING AND TESTING
    indexedTRAINbinary.cache()
    indexedTESTbinary.cache()
    oneHotTRAINbinary.cache()
    oneHotTESTbinary.cache()

    # FOR REGRESSION TRAINING AND TESTING
    indexedTRAINreg.cache()
    indexedTESTreg.cache()
    oneHotTRAINreg.cache()
    oneHotTESTreg.cache()

    # SCALED FEATURES
    oneHotTRAINregScaled.cache()
    oneHotTESTregScaled.cache()

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出** 

執行上述儲存格所花費的時間︰0.13 秒

## <a name="predict-whether-or-not-a-tip-is-paid-with-binary-classification-models"></a>使用二進位分類模型來預測是否支付小費
本節說明如何為二進位分類工作使用三個模型，預測是否支付計程車車程的小費。 顯示模型如下︰

* 羅吉斯迴歸 
* 隨機樹系
* 漸層停駐提升樹狀結構

每個模型建置程式碼區段會分成幾個步驟︰ 

1. **模型定型** 資料
2. **模型評估** 
3. **儲存模型** 

我們會示範如何以兩種方式使用參數掃掠執行交叉驗證 (CV)︰

1. 使用 **泛型** 自訂程式碼，可以套用至 MLlib 中的任何演算法，以及套用至演算法中的任何參數集。 
2. 使用 **pySpark CrossValidator 管線函式**。 請注意，CrossValidator 對 Spark 1.5.0 有幾項限制︰ 
   
   * 管線模型無法儲存/保存供未來取用。
   * 無法用於模型中的每個參數。
   * 無法用於每個 MLlib 演算法。

### <a name="generic-cross-validation-and-hyperparameter-sweeping-used-with-the-logistic-regression-algorithm-for-binary-classification"></a>針對二進位分類搭配使用一般交叉驗證和超參數掃掠與羅吉斯迴歸演算法
本節的程式碼顯示如何定型、評估及使用 [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) 來儲存羅吉斯迴歸模型，可預測是否針對 NYC 計程車車程和費用資料集的某趟車程支付小費。 模型是使用交叉驗證 (CV) 和超參數掃掠定型，這兩種方法是使用自訂程式碼實作，可以套用至 MLlib 中的任何學習演算法。   

> [!NOTE]
> 執行此自訂 CV 程式碼可能需要數分鐘的時間。
> 
> 

**使用 CV 及超參數掃掠來定型羅吉斯迴歸模型**

    # LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

    # GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD LIBRARIES
    from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
    from pyspark.mllib.evaluation import BinaryClassificationMetrics

    # CREATE PARAMETER GRID FOR LOGISTIC REGRESSION PARAMETER SWEEP
    from sklearn.grid_search import ParameterGrid
    grid = [{'regParam': [0.01, 0.1], 'iterations': [5, 10], 'regType': ["l1", "l2"], 'tolerance': [1e-3, 1e-4]}]
    paramGrid = list(ParameterGrid(grid))
    numModels = len(paramGrid)

    # SET NUM FOLDS AND NUM PARAMETER SETS TO SWEEP ON
    nFolds = 3;
    h = 1.0 / nFolds;
    metricSum = np.zeros(numModels);

    # BEGIN CV WITH PARAMETER SWEEP
    for i in range(nFolds):
        # Create training and x-validation sets
        validateLB = i * h
        validateUB = (i + 1) * h
        condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
        validation = trainData.filter(condition)
        # Create LabeledPoints from data-frames
        if i > 0:
            trainCVLabPt.unpersist()
            validationLabPt.unpersist()
        trainCV = trainData.filter(~condition)
        trainCVLabPt = trainCV.map(parseRowOneHotBinary)
        trainCVLabPt.cache()
        validationLabPt = validation.map(parseRowOneHotBinary)
        validationLabPt.cache()
        # For parameter sets compute metrics from x-validation
        for j in range(numModels):
            regt = paramGrid[j]['regType']
            regp = paramGrid[j]['regParam']
            iters = paramGrid[j]['iterations']
            tol = paramGrid[j]['tolerance']
            # Train logistic regression model with hypermarameter set
            model = LogisticRegressionWithLBFGS.train(trainCVLabPt, regType=regt, iterations=iters,  
                                                      regParam=regp, tolerance = tol, intercept=True)
            predictionAndLabels = validationLabPt.map(lambda lp: (float(model.predict(lp.features)), lp.label))
            # Use ROC-AUC as accuracy metrics
            validMetrics = BinaryClassificationMetrics(predictionAndLabels)
            metric = validMetrics.areaUnderROC
            metricSum[j] += metric

    avgAcc = metricSum / nFolds;
    bestParam = paramGrid[np.argmax(avgAcc)];

    # UNPERSIST OBJECTS
    trainCVLabPt.unpersist()
    validationLabPt.unpersist()

    # TRAIN ON FULL TRAIING SET USING BEST PARAMETERS FROM CV/PARAMETER SWEEP
    logitBest = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, regType=bestParam['regType'], 
                                                  iterations=bestParam['iterations'], 
                                                  regParam=bestParam['regParam'], tolerance = bestParam['tolerance'], 
                                                  intercept=True)


    # PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
    # NOTE: There are 20 coefficient terms for the 10 features, 
    #       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
    print("Coefficients: " + str(logitBest.weights))
    print("Intercept: " + str(logitBest.intercept))

    # PRINT ELAPSED TIME    
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

係數：[0.0082065285375, -0.0223675576104, -0.0183812028036, -3.48124578069e-05, -0.00247646947233, -0.00165897881503, 0.0675394837328, -0.111823113101, -0.324609912762, -0.204549780032, -1.36499216354, 0.591088507921, -0.664263411392, -1.00439726852, 3.46567827545, -3.51025855172, -0.0471341112232, -0.043521833294, 0.000243375810385, 0.054518719222]

截距：-0.0111216486893

執行上述儲存格所花費的時間︰14.43 秒

**使用標準度量評估二進位分類模型**

本章節中的程式碼示範如何針對測試資料集評估羅吉斯迴歸模型，包括 ROC 曲線的繪製。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    #IMPORT LIBRARIES
    from sklearn.metrics import roc_curve,auc
    from pyspark.mllib.evaluation import BinaryClassificationMetrics
    from pyspark.mllib.evaluation import MulticlassMetrics

    # PREDICT ON TEST DATA WITH BEST/FINAL MODEL
    predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitBest.predict(lp.features)), lp.label))

    # INSTANTIATE METRICS OBJECT
    metrics = BinaryClassificationMetrics(predictionAndLabels)

    # AREA UNDER PRECISION-RECALL CURVE
    print("Area under PR = %s" % metrics.areaUnderPR)

    # AREA UNDER ROC CURVE
    print("Area under ROC = %s" % metrics.areaUnderROC)
    metrics = MulticlassMetrics(predictionAndLabels)

    # OVERALL STATISTICS
    precision = metrics.precision()
    recall = metrics.recall()
    f1Score = metrics.fMeasure()
    print("Summary Stats")
    print("Precision = %s" % precision)
    print("Recall = %s" % recall)
    print("F1 Score = %s" % f1Score)

    # OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
    logitBest.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
    predictionAndLabelsDF = predictionAndLabels.toDF()
    predictionAndLabelsDF.registerTempTable("tmp_results");

    # PRINT ELAPSED TIME    
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

PR 下的領域 = 0.985336538462

ROC 下的領域 = 0.983383274312

摘要狀態

精確度 = 0.984174341679

回收 = 0.984174341679

F1 分數 = 0.984174341679

執行上述儲存格所花費的時間︰2.67 秒

**繪製 ROC 曲線。**

predictionAndLabelsDF 已在先前的儲存格中已註冊為 tmp_results 資料表。 tmp_results 可用來執行查詢，並將結果輸出至 sqlResults 資料框架供繪製之用。 程式碼如下。

    # QUERY RESULTS                              
    %%sql -q -o sqlResults
    SELECT * from tmp_results


以下是建立預測和繪製 ROC 曲線的程式碼。

    # MAKE PREDICTIONS AND PLOT ROC-CURVE

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES                              
    %%local
    %matplotlib inline
    from sklearn.metrics import roc_curve,auc

    #PREDICTIONS
    predictions_pddf = sqlResults.rename(columns={'_1': 'probability', '_2': 'label'})
    prob = predictions_pddf["probability"] 
    fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
    roc_auc = auc(fpr, tpr)

    # PLOT ROC CURVES
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


**輸出**

![泛型方法的羅吉斯迴歸 ROC 曲線](./media/spark-advanced-data-exploration-modeling/logistic-regression-roc-curve.png)

**blob 中供未來取用的保留模型**

本章節中的程式碼示範如何儲存羅吉斯迴歸模型以供取用。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.classification import LogisticRegressionModel

    # PERSIST MODEL
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
    dirfilename = modelDir + logisticregressionfilename;

    logitBest.save(sc, dirfilename);

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds";


**輸出**

執行上述儲存格所花費的時間︰34.57 秒

### <a name="use-mllibs-crossvalidator-pipeline-function-with-logistic-regression-elastic-regression-model"></a>搭配使用 MLlib 的 CrossValidator 管線函式與 LogisticRegression (彈性迴歸) 模型
本節的程式碼顯示如何定型、評估及使用 [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) 來儲存羅吉斯迴歸模型，可預測是否針對 NYC 計程車車程和費用資料集的某趟車程支付小費。 模型是使用交叉驗證 (CV) 和超參數掃掠定型，這兩種方法是使用 CV 的 MLlib CrossValidator 管線函式和參數掃掠進行實作。   

> [!NOTE]
> 執行此 MLlib CV 程式碼可能需要數分鐘的時間。
> 
> 

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml import Pipeline
    from pyspark.ml.evaluation import BinaryClassificationEvaluator
    from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
    from sklearn.metrics import roc_curve,auc

    # DEFINE ALGORITHM / MODEL
    lr = LogisticRegression()

    # DEFINE GRID PARAMETERS
    paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
                                  .addGrid(lr.maxIter, (5, 10))\
                                  .addGrid(lr.tol, (1e-4, 1e-5))\
                                  .addGrid(lr.elasticNetParam, (0.25,0.75))\
                                  .build()

    # DEFINE CV WITH PARAMETER SWEEP
    cv = CrossValidator(estimator= lr,
                        estimatorParamMaps=paramGrid,
                        evaluator=BinaryClassificationEvaluator(),
                        numFolds=3)

    # CONVERT TO DATA-FRAME: THIS DOES NOT RUN ON RDDs
    trainDataFrame = sqlContext.createDataFrame(oneHotTRAINbinary, ["features", "label"])

    # TRAIN WITH CROSS-VALIDATION
    cv_model = cv.fit(trainDataFrame)


    ## PREDICT AND EVALUATE ON TEST DATA-SET

    # USE TEST DATASET FOR PREDICTION
    testDataFrame = sqlContext.createDataFrame(oneHotTESTbinary, ["features", "label"])
    test_predictions = cv_model.transform(testDataFrame)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**輸出**

執行上述儲存格所花費的時間︰107.98 秒

**繪製 ROC 曲線。**

predictionAndLabelsDF 已在先前的儲存格中已註冊為 tmp_results 資料表。 tmp_results 可用來執行查詢，並將結果輸出至 sqlResults 資料框架供繪製之用。 程式碼如下。

    # QUERY RESULTS
    %%sql -q -o sqlResults
    SELECT label, prediction, probability from tmp_results

以下是繪製 ROC 曲線的程式碼。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES 
    %%local
    from sklearn.metrics import roc_curve,auc

    # ROC CURVE
    prob = [x["values"][1] for x in sqlResults["probability"]]
    fpr, tpr, thresholds = roc_curve(sqlResults['label'], prob, pos_label=1);
    roc_auc = auc(fpr, tpr)

    #PLOT
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


**輸出**

![使用 MLlib 的 CrossValidator 的羅吉斯迴歸 ROC 曲線](./media/spark-advanced-data-exploration-modeling/mllib-crossvalidator-roc-curve.png)

### <a name="random-forest-classification"></a>隨機樹系分類
本節的程式碼顯示如何訓練評估及儲存隨機樹系迴歸，可預測是否針對 NYC 計程車車程和費用資料集的某趟車程支付小費。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.tree import RandomForest, RandomForestModel
    from pyspark.mllib.util import MLUtils
    from pyspark.mllib.evaluation import BinaryClassificationMetrics
    from pyspark.mllib.evaluation import MulticlassMetrics

    # SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
    categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

    # TRAIN RANDOMFOREST MODEL
    rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
                                           categoricalFeaturesInfo=categoricalFeaturesInfo,
                                           numTrees=25, featureSubsetStrategy="auto",
                                           impurity='gini', maxDepth=5, maxBins=32)
    ## UN-COMMENT IF YOU WANT TO PRING TREES
    #print('Learned classification forest model:')
    #print(rfModel.toDebugString())

    # PREDICT ON TEST DATA AND EVALUATE
    predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
    predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

    # AREA UNDER ROC CURVE
    metrics = BinaryClassificationMetrics(predictionAndLabels)
    print("Area under ROC = %s" % metrics.areaUnderROC)

    # PERSIST MODEL IN BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    rfclassificationfilename = "RandomForestClassification_" + datestamp;
    dirfilename = modelDir + rfclassificationfilename;

    rfModel.save(sc, dirfilename);

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

ROC 下的領域 = 0.985336538462

執行上述儲存格所花費的時間︰26.72 秒

### <a name="gradient-boosting-trees-classification"></a>漸層停駐提升樹狀結構類別
本節的程式碼顯示如何訓練評估及儲存漸層停駐提升樹狀結構模型，可預測是否針對 NYC 計程車車程和費用資料集的某趟車程支付小費。

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

    # SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
    categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

    gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                    numIterations=10)
    ## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
    #print('Learned classification GBT model:')
    #print(bgtModel.toDebugString())

    # PREDICT ON TEST DATA AND EVALUATE
    predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
    predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

    # Area under ROC curve
    metrics = BinaryClassificationMetrics(predictionAndLabels)
    print("Area under ROC = %s" % metrics.areaUnderROC)

    # PERSIST MODEL IN A BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
    dirfilename = modelDir + btclassificationfilename;

    gbtModel.save(sc, dirfilename)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出**

ROC 下的領域 = 0.985336538462

執行上述儲存格所花費的時間︰28.13 秒

## <a name="predict-tip-amount-with-regression-models-not-using-cv"></a>使用迴歸模型預測小費金額 (非使用 CV)
本節說明如何使用三種迴歸工作模型：根據其他小費功能預測支付的計程車車程的小費金額。 顯示模型如下︰

* 正規化線性迴歸
* 隨機樹系
* 漸層停駐提升樹狀結構

這些模型將在簡介中說明。 每個模型建置程式碼區段會分成幾個步驟︰ 

1. **模型定型** 資料
2. **模型評估** 
3. **儲存模型**    

> AZURE 附註︰交叉驗證未與本節中的三個迴歸模型搭配使用，因為這是用來顯示羅吉斯迴歸模型詳細資料的方法。 本主題的＜附錄＞示範如何針對線性迴歸使用 CV 與 Elastic Net。
> 
> AZURE NOTE︰在我們的經驗中，交集的 LinearRegressionWithSGD 模型可能會發生問題，且必須小心變更/最佳化參數以取得有效的模型。 調整變數對於聚合很有用。 本主題中的＜附錄＞所顯示的 Elastic Net 迴歸也可用來取代 LinearRegressionWithSGD。
> 
> 

### <a name="linear-regression-with-sgd"></a>使用 SGD 的線性迴歸
本節的程式碼示範如何使用縮放功能來訓練線性迴歸 (使用隨機梯度下降 (SGD) 以最佳化)，以及如何評分、評估並將模型儲存在 Azure Blob 儲存體 (WASB)。

> [!TIP]
> 在我們的經驗中，交集的 LinearRegressionWithSGD 模型可能會發生問題，且必須小心變更/最佳化參數以取得有效的模型。 調整變數對於聚合很有用。
> 
> 

    # LINEAR REGRESSION WITH SGD 

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD LIBRARIES
    from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
    from pyspark.mllib.evaluation import RegressionMetrics
    from scipy import stats

    # USE SCALED FEATURES TO TRAIN MODEL
    linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)

    # PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
    # NOTE: There are 20 coefficient terms for the 10 features, 
    #       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
    print("Coefficients: " + str(linearModel.weights))
    print("Intercept: " + str(linearModel.intercept))

    # SCORE ON SCALED TEST DATA-SET & EVALUATE
    predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
    testMetrics = RegressionMetrics(predictionAndLabels)

    print("RMSE = %s" % testMetrics.rootMeanSquaredError)
    print("R-sqr = %s" % testMetrics.r2)

    # SAVE MODEL IN BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
    dirfilename = modelDir + linearregressionfilename;

    linearModel.save(sc, dirfilename)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出**

係數：[0.0141707753435, -0.0252930927087, -0.0231442517137, 0.247070902996, 0.312544147152, 0.360296120645, 0.0122079566092, -0.00456498588241, -0.0898228505177, 0.0714046248793, 0.102171263868, 0.100022455632, -0.00289545676449, -0.00791124681938, 0.54396316518, -0.536293513569, 0.0119076553369, -0.0173039244582, 0.0119632796147, 0.00146764882502]

截距：0.854507624459

RMSE = 1.23485131376

R-sqr = 0.597963951127

執行上述儲存格所花費的時間︰38.62 秒

### <a name="random-forest-regression"></a>隨機樹系迴歸
本節的程式碼顯示如何訓練評估及儲存隨機樹系模型，可預測 NYC 計程車車程資料的小費金額。   

> [!NOTE]
> 在附錄中提供使用自訂程式碼的交叉驗證與參數掃掠。
> 
> 

    #PREDICT TIP AMOUNTS USING RANDOM FOREST

    # RECORD START TIME
    timestart= datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.tree import RandomForest, RandomForestModel
    from pyspark.mllib.util import MLUtils
    from pyspark.mllib.evaluation import RegressionMetrics


    # TRAIN MODEL
    categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
    rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                        numTrees=25, featureSubsetStrategy="auto",
                                        impurity='variance', maxDepth=10, maxBins=32)
    # UN-COMMENT IF YOU WANT TO PRING TREES
    #print('Learned classification forest model:')
    #print(rfModel.toDebugString())

    # PREDICT AND EVALUATE ON TEST DATA-SET
    predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
    predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

    testMetrics = RegressionMetrics(predictionAndLabels)
    print("RMSE = %s" % testMetrics.rootMeanSquaredError)
    print("R-sqr = %s" % testMetrics.r2)

    # SAVE MODEL IN BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    rfregressionfilename = "RandomForestRegression_" + datestamp;
    dirfilename = modelDir + rfregressionfilename;

    rfModel.save(sc, dirfilename);

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出**

RMSE = 0.931981967875

R-sqr = 0.733445485802

執行上述儲存格所花費的時間︰25.98 秒

### <a name="gradient-boosting-trees-regression"></a>漸層停駐提升樹狀結構迴歸
本節的程式碼顯示如何訓練評估及儲存漸層停駐提升樹狀結構模型，可預測 NYC 計程車車程資料的小費金額。

**訓練及評估**

    #PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

    # RECORD START TIME
    timestart= datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
    from pyspark.mllib.util import MLUtils

    # TRAIN MODEL
    categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
    gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
                                                    numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)

    # EVALUATE A TEST DATA-SET
    predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
    predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

    testMetrics = RegressionMetrics(predictionAndLabels)
    print("RMSE = %s" % testMetrics.rootMeanSquaredError)
    print("R-sqr = %s" % testMetrics.r2)

    # PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES
    test_predictions= sqlContext.createDataFrame(predictionAndLabels)
    test_predictions_pddf = test_predictions.toPandas()

    # SAVE MODEL IN BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
    dirfilename = modelDir + btregressionfilename;
    gbtModel.save(sc, dirfilename)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

RMSE = 0.928172197114

R-sqr = 0.732680354389

執行上述儲存格所花費的時間︰20.9 秒

**圖**

tmp_results 已在先前的儲存格中註冊為 Hive 資料表。 來自資料表的結果已輸出至 sqlResults  資料框架以供繪製之用。 程式碼如下

    # PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES

    # SELECT RESULTS
    %%sql -q -o sqlResults
    SELECT * from tmp_results


以下是使用 Jupyter 伺服器繪製資料的程式碼。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    import numpy as np

    # PLOT
    ax = sqlResults.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
    fit = np.polyfit(sqlResults['_1'], sqlResults['_2'], deg=1)
    ax.set_title('Actual vs. Predicted Tip Amounts ($)')
    ax.set_xlabel("Actual")
    ax.set_ylabel("Predicted")
    ax.plot(sqlResults['_1'], fit[0] * sqlResults['_1'] + fit[1], color='magenta')
    plt.axis([-1, 15, -1, 15])
    plt.show(ax)

![Actual-vs-predicted-tip-amounts](./media/spark-advanced-data-exploration-modeling/actual-vs-predicted-tips.png)

## <a name="appendix-additional-regression-tasks-using-cross-validation-with-parameter-sweeps"></a>附錄︰使用交叉驗證與參數掃掠的額外迴歸工作
本附錄包含程式碼，示範如何針對線性迴歸使用彈性 net 以執行 CV，以及如何針對隨機樹系迴歸使用自訂程式碼，以參數掃掠執行 CV。

### <a name="cross-validation-using-elastic-net-for-linear-regression"></a>針對線性迴歸使用彈性 net 進行交叉驗證
本章節中的程式碼示範如何針對線性迴歸使用彈性 net 進行交叉驗證，以及如何針對測試資料評估模型。

    ###  CV USING ELASTIC NET FOR LINEAR REGRESSION

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.ml.regression import LinearRegression
    from pyspark.ml import Pipeline
    from pyspark.ml.evaluation import RegressionEvaluator
    from pyspark.ml.tuning import CrossValidator, ParamGridBuilder

    # DEFINE ALGORITHM/MODEL
    lr = LinearRegression()

    # DEFINE GRID PARAMETERS
    paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
                                  .addGrid(lr.maxIter, (5, 10))\
                                  .addGrid(lr.tol, (1e-4, 1e-5))\
                                  .addGrid(lr.elasticNetParam, (0.25,0.75))\
                                  .build() 

    # DEFINE PIPELINE 
    # SIMPLY THE MODEL HERE, WITHOUT TRANSFORMATIONS
    pipeline = Pipeline(stages=[lr])

    # DEFINE CV WITH PARAMETER SWEEP
    cv = CrossValidator(estimator= lr,
                        estimatorParamMaps=paramGrid,
                        evaluator=RegressionEvaluator(),
                        numFolds=3)

    # CONVERT TO DATA FRAME, AS CROSSVALIDATOR WON'T RUN ON RDDS
    trainDataFrame = sqlContext.createDataFrame(oneHotTRAINreg, ["features", "label"])

    # TRAIN WITH CROSS-VALIDATION
    cv_model = cv.fit(trainDataFrame)


    # EVALUATE MODEL ON TEST SET
    testDataFrame = sqlContext.createDataFrame(oneHotTESTreg, ["features", "label"])

    # MAKE PREDICTIONS ON TEST DOCUMENTS
    # cvModel uses the best model found (lrModel).
    predictionAndLabels = cv_model.transform(testDataFrame)

    # CONVERT TO DF AND SAVE REGISER DF AS TABLE
    predictionAndLabels.registerTempTable("tmp_results");

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

執行上述儲存格所花費的時間︰161.21 秒

**使用 R-SQR 度量進行評估**

tmp_results 已在先前的儲存格中註冊為 Hive 資料表。 來自資料表的結果已輸出至 sqlResults  資料框架以供繪製之用。 程式碼如下

    # SELECT RESULTS
    %%sql -q -o sqlResults
    SELECT label,prediction from tmp_results


以下是計算 R-sqr 的程式碼。

    # RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
    %%local
    from scipy import stats

    #R-SQR TEST METRIC
    corstats = stats.linregress(sqlResults['label'],sqlResults['prediction'])
    r2 = (corstats[2]*corstats[2])
    print("R-sqr = %s" % r2)


**輸出**

R-sqr = 0.619184907088

### <a name="cross-validation-with-parameter-sweep-using-custom-code-for-random-forest-regression"></a>針對隨機樹系迴歸使用自訂程式碼以參數掃掠進行交叉驗證。
本章節中的程式碼示範如何針對隨機樹系迴歸使用自訂程式碼以參數掃掠執行交叉驗證，以及如何針對測試資料評估模型。

    # RECORD START TIME
    timestart= datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    # GET ACCURARY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET
    from pyspark.mllib.tree import RandomForest, RandomForestModel
    from pyspark.mllib.util import MLUtils
    from pyspark.mllib.evaluation import RegressionMetrics
    from sklearn.grid_search import ParameterGrid

    ## CREATE PARAMETER GRID
    grid = [{'maxDepth': [5,10], 'numTrees': [25,50]}]
    paramGrid = list(ParameterGrid(grid))

    ## SPECIFY LEVELS OF CATEGORICAL VARIBLES
    categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

    # SPECIFY NUMFOLDS AND ARRAY TO HOLD METRICS
    nFolds = 3;
    numModels = len(paramGrid)
    h = 1.0 / nFolds;
    metricSum = np.zeros(numModels);

    for i in range(nFolds):
        # Create training and x-validation sets
        validateLB = i * h
        validateUB = (i + 1) * h
        condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
        validation = trainData.filter(condition)
        # Create labeled points from data-frames
        if i > 0:
            trainCVLabPt.unpersist()
            validationLabPt.unpersist()
        trainCV = trainData.filter(~condition)
        trainCVLabPt = trainCV.map(parseRowIndexingRegression)
        trainCVLabPt.cache()
        validationLabPt = validation.map(parseRowIndexingRegression)
        validationLabPt.cache()
        # For parameter sets compute metrics from x-validation
        for j in range(numModels):
            maxD = paramGrid[j]['maxDepth']
            numT = paramGrid[j]['numTrees']
            # Train logistic regression model with hypermarameter set
            rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                        numTrees=numT, featureSubsetStrategy="auto",
                                        impurity='variance', maxDepth=maxD, maxBins=32)
            predictions = rfModel.predict(validationLabPt.map(lambda x: x.features))
            predictionAndLabels = validationLabPt.map(lambda lp: lp.label).zip(predictions)
            # Use ROC-AUC as accuracy metrics
            validMetrics = RegressionMetrics(predictionAndLabels)
            metric = validMetrics.rootMeanSquaredError
            metricSum[j] += metric

    avgAcc = metricSum/nFolds;
    bestParam = paramGrid[np.argmin(avgAcc)];

    # UNPERSIST OBJECTS
    trainCVLabPt.unpersist()
    validationLabPt.unpersist()

    ## TRAIN FINAL MODL WIHT BEST PARAMETERS
    rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                        numTrees=bestParam['numTrees'], featureSubsetStrategy="auto",
                                        impurity='variance', maxDepth=bestParam['maxDepth'], maxBins=32)

    # EVALUATE MODEL ON TEST DATA
    predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
    predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

    #PRINT TEST METRICS
    testMetrics = RegressionMetrics(predictionAndLabels)
    print("RMSE = %s" % testMetrics.rootMeanSquaredError)
    print("R-sqr = %s" % testMetrics.r2)

    # PRINT ELAPSED TIME
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出**

RMSE = 0.906972198262

R-sqr = 0.740751197012

執行上述儲存格所花費的時間︰69.17 秒

### <a name="clean-up-objects-from-memory-and-print-model-locations"></a>從記憶體清除物件並列印模型位置
使用 `unpersist()` 刪除記憶體中快取的物件。

    # UNPERSIST OBJECTS CACHED IN MEMORY

    # REMOVE ORIGINAL DFs
    taxi_df_train_cleaned.unpersist()
    taxi_df_train_with_newFeatures.unpersist()
    trainData.unpersist()
    trainData.unpersist()

    # FOR BINARY CLASSIFICATION TRAINING AND TESTING
    indexedTRAINbinary.unpersist()
    indexedTESTbinary.unpersist()
    oneHotTRAINbinary.unpersist()
    oneHotTESTbinary.unpersist()

    # FOR REGRESSION TRAINING AND TESTING
    indexedTRAINreg.unpersist()
    indexedTESTreg.unpersist()
    oneHotTRAINreg.unpersist()
    oneHotTESTreg.unpersist()

    # SCALED FEATURES
    oneHotTRAINregScaled.unpersist()
    oneHotTESTregScaled.unpersist()


**輸出**

PythonRDD[122] at RDD at PythonRDD.scala: 43

**列印出要在耗用量 Notebook 中使用的模型檔案路徑。 **若要耗用與評分獨立資料集，您需要在「耗用量 Notebook」中複製並貼上這些檔案名稱。

    # PRINT MODEL FILE LOCATIONS FOR CONSUMPTION
    print "logisticRegFileLoc = modelDir + \"" + logisticregressionfilename + "\"";
    print "linearRegFileLoc = modelDir + \"" + linearregressionfilename + "\"";
    print "randomForestClassificationFileLoc = modelDir + \"" + rfclassificationfilename + "\"";
    print "randomForestRegFileLoc = modelDir + \"" + rfregressionfilename + "\"";
    print "BoostedTreeClassificationFileLoc = modelDir + \"" + btclassificationfilename + "\"";
    print "BoostedTreeRegressionFileLoc = modelDir + \"" + btregressionfilename + "\"";


**輸出**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-05-0316_47_30.096528"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-05-0316_51_28.433670"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-05-0316_50_17.454440"

randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-05-0316_51_57.331730"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-05-0316_50_40.138809"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-05-0316_52_18.827237"

## <a name="whats-next"></a>後續步驟
現在已使用 Spark MlLib 建立迴歸和分類模型，您已瞭解如何評分及評估這些模型。

**模型耗用量︰** 若要瞭解如何評分及評估本主題中所建立的分類和迴歸模型，請參閱 [評分及評估 Spark 建置機器學習服務模型](spark-model-consumption.md)。

