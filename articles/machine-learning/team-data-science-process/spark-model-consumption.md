---
title: 實作 Spark 建置機器學習模型 | Microsoft Docs
description: 如何使用 Python 載入及評分儲存在 Azure Blob 儲存體 (WASB) 中的學習模型。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 626305a2-0abf-4642-afb0-dad0f6bd24e9
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/15/2017
ms.author: deguhath
ms.openlocfilehash: 95403e71367f111877c6f317c63811333bd054dd
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395185"
---
# <a name="operationalize-spark-built-machine-learning-models"></a>實作 Spark 建置機器學習模型
[!INCLUDE [machine-learning-spark-modeling](../../../includes/machine-learning-spark-modeling.md)]

本主題說明如何使用 HDInsight Spark 叢集上的 Python 實作已儲存的機器學習模型 (ML)。 它說明如何載入已使用 Spark MLlib 建立並儲存於 Azure Blob 儲存體 (WASB) 的機器學習服務模型，以及如何使用已儲存在 WASB 的資料集加以評分。 它會顯示如何前置處理輸入資料、使用 MLlib 工具組中的索引和編碼函式來轉換功能，以及如何建立可做為輸入的標示點資料物件，以便使用 ML 模型加以評分。 用於評分的模型包含線性迴歸、羅吉斯迴歸、隨機樹系模型和漸層停駐提升樹狀結構模型。

## <a name="spark-clusters-and-jupyter-notebooks"></a>Spark 叢集和 Jupyter Notebook
在本逐步解說中提供實作 ML 模型的設定步驟和程式碼，供您使用 HDInsight Spark 1.6 叢集以及 Spark 2.0 叢集。 Jupyter notebook 中也會提供這些程序的程式碼。

### <a name="notebook-for-spark-16"></a>Notebook for Spark 1.6
[pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) Jupyter Notebook 示範如何在 HDInsight 叢集上，使用 Python 讓儲存的模型能夠運作。 

### <a name="notebook-for-spark-20"></a>Notebook for Spark 2.0
若要修改此 Jupyter Notebook for Spark 1.6 來與 HDInsight Spark 2.0 叢集搭配使用，請使用[這個檔案](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py)來取代 Python 程式碼檔案。 此程式碼示範如何使用 Spark 2.0 中所建立的模型。


## <a name="prerequisites"></a>必要條件

1. 您需要 Azure 帳戶和 Spark 1.6 (或 Spark 2.0) HDInsight 叢集，才能完成此逐步解說。 請參閱[使用 Azure HDInsight 上的 Spark 的資料科學概觀](spark-overview.md)以取得這些需求。 此主題也包括這裡使用的 NYC 2013 計程車資料的描述，以及如何從 Spark 叢集的 Jupyter Notebook 執行程式碼的指示。 
2. 您也必須針對 Spark 1.6 叢集或 Spark 2.0 Notebook，透過[使用 Spark 資料探索和模型化](spark-data-exploration-modeling.md)主題運作，在這裡建立要評分的機器學習服務模型。 
3. Spark 2.0 Notebook 會針對分類工作使用額外的資料集，即 2011 年和 2012 年知名航空公司準時起飛的資料集。 Notebook 的描述及它們的連結已在包含它們的 GitHub 儲存機制的 [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) 中提供。 此外，此處及連結的 Notebook 內的程式碼皆屬泛型程式碼，而且應該能在任何 Spark 叢集上運作。 若您不是使用 HDInsight Spark，叢集設定和管理步驟可能與這裡顯示的稍有不同。 

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>安裝程式︰儲存體位置、程式庫和預設 Spark 內容
Spark 也可以讀取和寫入 Azure 儲存體 Blob (WASB)。 如此可使用 Spark 處理該處儲存的任何現有資料，並在 WASB 中再次儲存結果。

若要在 WASB 中儲存模型或檔案，必須正確指定路徑。 可以使用以「wasb:///」開頭的路徑，參考連接到 Spark 叢集的預設容器。 下列程式碼範例會指定要讀取資料的位置，和將儲存模型輸出的模型儲存體目錄的路徑。 

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>在 WASB 中設定儲存位置的目錄路徑
模型會儲存在：「wasb:///user/remoteuser/NYCTaxi/Models」。 如果未正確設定此路徑，不會載入模型進行評分。

評分的結果儲存在：「wasb:///user/remoteuser/NYCTaxi/ScoredResults」。 如果資料夾的路徑不正確，不會將結果儲存在該資料夾中。   

> [!NOTE]
> 可從 **machine-learning-data-science-spark-data-exploration-modeling.ipynb** notebook 的最後一個儲存格的輸出，將檔案路徑位置複製並貼至此程式碼的預留位置。   
> 
> 

以下是設定目錄路徑的程式碼： 

    # LOCATION OF DATA TO BE SCORED (TEST DATA)
    taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";

    # SET THE MODEL STORAGE DIRECTORY PATH 
    # NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
    modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 

    # SET SCORDED RESULT DIRECTORY PATH
    # NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
    scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 

    # FILE LOCATIONS FOR THE MODELS TO BE SCORED
    logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
    linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
    randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
    randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
    BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
    BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

    # RECORD START TIME
    import datetime
    datetime.datetime.now()

**輸出：**

datetime.datetime(2016, 4, 25, 23, 56, 19, 229403)

### <a name="import-libraries"></a>匯入程式庫
使用下列程式碼設定 Spark 內容並匯入必要的程式庫

    #IMPORT LIBRARIES
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
* **%%sql -o <variable name>** 
* 針對 sqlContext 執行 Hive 查詢。 如果傳遞 -o 參數，則查詢的結果會當做 Pandas 資料框架，保存在 %%local Python 內容中。

如需關於 Jupyter Notebook 核心，以及其所提供的預先定義 "magics" 的詳細資訊，請參閱 [HDInsight 上的 HDInsight Spark Linux 叢集可供 Jupyter Notebook 使用的核心](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md)。

## <a name="ingest-data-and-create-a-cleaned-data-frame"></a>擷取資料並建立已清除的資料框架
本節包含一系列工作的程式碼，為擷取要評分的資料所必需。 在聯結的 0.1% 取樣的計程車車程和費用檔案中讀取 (儲存為 .tsv 檔案)、格式化資料，然後建立清空的資料框架。

已根據 [Team Data Science Process 實務：使用 HDInsight Hadoop 叢集](hive-walkthrough.md) 主題提供的程序來聯結計程車車程和費用檔案。

    # INGEST DATA AND CREATE A CLEANED DATA FRAME

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # IMPORT FILE FROM PUBLIC BLOB
    taxi_test_file = sc.textFile(taxi_test_file_loc)

    # GET SCHEMA OF THE FILE FROM HEADER
    taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)

    # PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
    taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
            .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                            float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                            float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))

    # GET SCHEMA OF THE FILE FROM HEADER
    schema_string = taxi_test_file.first()
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

    # CREATE DATA FRAME
    taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)

    # CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
    taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
        .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
        .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
        .drop('direct_distance').drop('surcharge')\
        .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

    # CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
    taxi_df_test_cleaned.cache()
    taxi_df_test_cleaned.count()

    # REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
    taxi_df_test_cleaned.registerTempTable("taxi_test")

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出：**

執行上述儲存格所花費的時間︰46.37 秒

## <a name="prepare-data-for-scoring-in-spark"></a>準備資料在 Spark 中評分
本節說明如何索引、編碼及調整分類功能，準備將它們用於分類和迴歸的 MLlib 監督式學習演算法中。

### <a name="feature-transformation-index-and-encode-categorical-features-for-input-into-models-for-scoring"></a>功能轉換：索引並編碼分類功能以輸入至模型進行評分。
本節說明如何使用 `StringIndexer` 來為分類資料編製索引，並利用 `OneHotEncoder` 輸入將特徵編碼至模組中。

[StringIndexer](http://spark.apache.org/docs/latest/ml-features.html#stringindexer) 會將標籤的字串資料行編碼至標籤索引的資料行。 索引是按標籤頻率排序。 

[OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) 會將標籤索引資料行對應到二元向量資料行 (最多有一個單一值)。 這種編碼方式允許將預期連續值功能的演算法 (例如羅吉斯迴歸) 套用至分類功能。

    #INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # LOAD PYSPARK LIBRARIES
    from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

    # CREATE FOUR BUCKETS FOR TRAFFIC TIMES
    sqlStatement = """
        SELECT *,
        CASE
         WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
         WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
         WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
         WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
        END as TrafficTimeBins
        FROM taxi_test 
    """
    taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)

    # CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
    taxi_df_test_with_newFeatures.cache()
    taxi_df_test_with_newFeatures.count()

    # INDEX AND ONE-HOT ENCODING
    stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
    model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
    indexed = model.transform(taxi_df_test_with_newFeatures)
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

    # INDEX AND ENCODE TRAFFIC TIME BINS
    stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
    model = stringIndexer.fit(encoded3)
    indexed = model.transform(encoded3)
    encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
    encodedFinal = encoder.transform(indexed)

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出：**

執行上述儲存格所花費的時間︰5.37 秒

### <a name="create-rdd-objects-with-feature-arrays-for-input-into-models"></a>使用功能陣列建立 RDD 物件以輸入至模型
本節包含程式碼，示範如何將分類的文字資料索引為 RDD 物件並加以單次編碼，以用來訓練及測試 MLlib 羅吉斯迴歸和樹狀結構型模型。 已編製索引的資料是儲存在 [彈性分散式資料集 (RDD)](http://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html) 物件中。 這些是 Spark 中的基本抽象概念。 RDD 物件代表不可變、資料分割、可與 Spark 平行操作的元素集合。

它也包含程式碼，顯示如何使用 MLlib 提供的 `StandardScalar` 來調整資料，用於使用隨機梯度下降法 (SGD) 的線性迴歸，此為訓練廣泛的機器學習服務模型的常用演算法。 [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) 是用來調整單位變異數的特徵。 調整功能，也稱為資料正規化，以確保具廣泛分散值的功能在目標函式中沒有過多權重。 

    # CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # IMPORT LIBRARIES
    from pyspark.mllib.linalg import Vectors
    from pyspark.mllib.feature import StandardScaler, StandardScalerModel
    from pyspark.mllib.util import MLUtils
    from numpy import array

    # INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
    def parseRowIndexingBinary(line):
        features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                             line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                             line.trip_distance, line.fare_amount])
        return  features

    # ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
    def parseRowOneHotBinary(line):
        features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                            line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                            line.vendorVec.toArray(), line.rateVec.toArray(), 
                                            line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
        return  features

    # ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
    def parseRowIndexingRegression(line):
        features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                             line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                             line.trip_distance, line.fare_amount])
        return  features

    # INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
    def parseRowOneHotRegression(line):
        features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                            line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                            line.vendorVec.toArray(), line.rateVec.toArray(), 
                                            line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
        return  features

    # FOR BINARY CLASSIFICATION TRAINING AND TESTING
    indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
    oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)

    # FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
    indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
    oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)

    # SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
    scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
    oneHotTESTregScaled = scaler.transform(oneHotTESTreg)

    # CACHE RDDS IN MEMORY
    indexedTESTbinary.cache();
    oneHotTESTbinary.cache();
    indexedTESTreg.cache();
    oneHotTESTreg.cache();
    oneHotTESTregScaled.cache();

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出：**

執行上述儲存格所花費的時間︰11.72 秒

## <a name="score-with-the-logistic-regression-model-and-save-output-to-blob"></a>使用羅吉斯迴歸模型進行評分，並將輸出儲存至 blob
本節的程式碼示範如何載入 Azure Blob 儲存體中已儲存的羅吉斯迴歸模型，並使用它來預測是否支付計程車車程的小費、使用標準分類度量評分，然後儲存結果並將其繪製至 Blob 儲存體。 評分的結果會儲存在 RDD 物件。 

    # SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

    # RECORD START TIME
    timestart = datetime.datetime.now()

    # IMPORT LIBRARIES
    from pyspark.mllib.classification import LogisticRegressionModel

    ## LOAD SAVED MODEL
    savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
    predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))

    ## SAVE SCORED RESULTS (RDD) TO BLOB
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
    dirfilename = scoredResultDir + logisticregressionfilename;
    predictions.saveAsTextFile(dirfilename)


    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**輸出：**

執行上述儲存格所花費的時間︰19.22 秒

## <a name="score-a-linear-regression-model"></a>評分線性迴歸模型
我們搭配使用 [LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) 與隨機梯度下降法 (SGD) 來訓練線性迴歸模型，以最佳化的方式預測支付的小費金額。 

本節的程式碼示範如何從 Azure blob 儲存體載入線性迴歸模型、使用調整變數評分，然後將結果存回 blob。

    #SCORE LINEAR REGRESSION MODEL

    # RECORD START TIME
    timestart = datetime.datetime.now()

    #LOAD LIBRARIES
    from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel

    # LOAD MODEL AND SCORE USING ** SCALED VARIABLES **
    savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
    predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))

    # SAVE RESULTS
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
    dirfilename = scoredResultDir + linearregressionfilename;
    predictions.saveAsTextFile(dirfilename)

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**輸出：**

執行上述儲存格所花費的時間︰16.63 秒

## <a name="score-classification-and-regression-random-forest-models"></a>評分分類和迴歸的隨機樹系模型
本節的程式碼示範如何載入已儲存的分類和迴歸的隨機樹系模型 (儲存在 Azure blob 儲存體)、使用標準分類器和迴歸措施來評分其效能，然後將結果存回 blob 儲存體。

[隨機樹系](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) 是整體的決策樹。  隨機樹系結合許多決策樹來降低風險過度膨脹。 隨機樹系可處理分類功能、擴充至多類別分類設定、不需要調整功能，而且能夠擷取非線性和功能互動。 隨機樹系是其中一個最成功的分類和迴歸的機器學習模型。

[spark.mllib](http://spark.apache.org/mllib/) 使用連續和分類特徵來支援二元和多元分類和迴歸的隨機樹系。 

    # SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

    # RECORD START TIME
    timestart = datetime.datetime.now()

    #IMPORT MLLIB LIBRARIES    
    from pyspark.mllib.tree import RandomForest, RandomForestModel


    # CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
    savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
    predictions = savedModel.predict(indexedTESTbinary)

    # SAVE RESULTS
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
    dirfilename = scoredResultDir + rfclassificationfilename;
    predictions.saveAsTextFile(dirfilename)


    # REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
    savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
    predictions = savedModel.predict(indexedTESTreg)

    # SAVE RESULTS
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
    dirfilename = scoredResultDir + rfregressionfilename;
    predictions.saveAsTextFile(dirfilename)

    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**輸出：**

執行上述儲存格所花費的時間︰31.07 秒

## <a name="score-classification-and-regression-gradient-boosting-tree-models"></a>評分分類和迴歸的漸層停駐提升樹狀結構模型
本節的程式碼示範如何從 Azure blob 儲存體載入分類和迴歸的漸層停駐提升樹狀結構模型、使用標準分類器和迴歸措施來評分其效能，然後將結果存回 blob 儲存體。 

**spark.mllib** 使用連續和分類特徵來支援二元分類和迴歸的 GBT。 

[漸層停駐提升樹狀結構](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) 是整體的決策樹。 GBT 反覆地訓練決策樹以盡可能降低遺失函式。 GBT 可處理分類功能、不需要調整功能，而且能夠擷取非線性和功能互動。 它們也可用於多類別分類設定。

    # SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

    # RECORD START TIME
    timestart = datetime.datetime.now()

    #IMPORT MLLIB LIBRARIES
    from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

    # CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

    #LOAD AND SCORE THE MODEL
    savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
    predictions = savedModel.predict(indexedTESTbinary)

    # SAVE RESULTS
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
    dirfilename = scoredResultDir + btclassificationfilename;
    predictions.saveAsTextFile(dirfilename)


    # REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

    # LOAD AND SCORE MODEL 
    savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
    predictions = savedModel.predict(indexedTESTreg)

    # SAVE RESULTS
    datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
    btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
    dirfilename = scoredResultDir + btregressionfilename;
    predictions.saveAsTextFile(dirfilename)


    # PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
    timeend = datetime.datetime.now()
    timedelta = round((timeend-timestart).total_seconds(), 2) 
    print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**輸出：**

執行上述儲存格所花費的時間︰14.6 秒

## <a name="clean-up-objects-from-memory-and-print-scored-file-locations"></a>從記憶體清除物件並列印計分的檔案位置
    # UNPERSIST OBJECTS CACHED IN MEMORY
    taxi_df_test_cleaned.unpersist()
    indexedTESTbinary.unpersist();
    oneHotTESTbinary.unpersist();
    indexedTESTreg.unpersist();
    oneHotTESTreg.unpersist();
    oneHotTESTregScaled.unpersist();


    # PRINT OUT PATH TO SCORED OUTPUT FILES
    print "logisticRegFileLoc: " + logisticregressionfilename;
    print "linearRegFileLoc: " + linearregressionfilename;
    print "randomForestClassificationFileLoc: " + rfclassificationfilename;
    print "randomForestRegFileLoc: " + rfregressionfilename;
    print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
    print "BoostedTreeRegressionFileLoc: " + btregressionfilename;


**輸出：**

logisticRegFileLoc: LogisticRegressionWithLBFGS_2016-05-0317_22_38.953814.txt

linearRegFileLoc: LinearRegressionWithSGD_2016-05-0317_22_58.878949

randomForestClassificationFileLoc: RandomForestClassification_2016-05-0317_23_15.939247.txt

randomForestRegFileLoc: RandomForestRegression_2016-05-0317_23_31.459140.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification_2016-05-0317_23_49.648334.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression_2016-05-0317_23_56.860740.txt

## <a name="consume-spark-models-through-a-web-interface"></a>透過 Web 介面使用 Spark 模型
Spark 提供一個機制，透過 REST 介面 (包含稱為 Livy 的元件) 從遠端提交批次工作或互動式查詢。 Livy 預設在 HDInsight Spark 叢集上啟用。 如需 Livy 的詳細資訊，請參閱 [使用 Livy 遠端提交 Spark 作業](../../hdinsight/spark/apache-spark-livy-rest-interface.md)。 

您可以使用 Livy 從遠端提交作業，其批次批分儲存在 Azure blob 中的檔案，然後將結果寫入另一個 blob。 若要這樣做，需要將 Python 指令碼從   
[GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) 上傳至 Spark 叢集的 blob。 您可以使用類似 **Microsoft Azure 儲存體總管**或 **AzCopy** 的工具，將指令碼複製到叢集 blob。 在本例中，我們將指令碼上傳至 wasb:///example/python/ConsumeGBNYCReg.py。   

> [!NOTE]
> 您可在入口網站上，為 Spark 叢集關聯的儲存體帳戶尋找需要的存取金鑰。 
> 
> 

一旦上傳至這個位置，此指令碼會在分散式內容的 Spark 叢集內執行。 它會載入模型，並根據模型對輸入檔執行預測。  

您可以在 Livy 上進行簡單的 HTTPS/REST 要求，從遠端叫用此指令碼。  以下是 curl 命令，可建構 HTTP 要求以遠端叫用 Python 指令碼。 將 CLUSTERLOGIN、CLUSTERPASSWORD、CLUSTERNAME 取代為 Spark 叢集的適當值。

    # CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

    curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{\"file\": \"wasb:///example/python/ConsumeGBNYCReg.py\"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches

您可以藉由基本驗證來進行簡單 HTTPS 呼叫，使用遠端系統上的任何語言來叫用 Spark 作業。   

> [!NOTE]
> 進行此 HTTP 呼叫時可方便地使用 Python 要求程式庫，但目前預設不會在 Azure Functions 中安裝此程式庫。 因此會改用較舊的 HTTP 程式庫。   
> 
> 

以下是 HTTP 呼叫的 Python 程式碼 ︰

    #MAKE AN HTTPS CALL ON LIVY. 

    import os

    # OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILBLE BY DEFAULT
    import httplib, urllib, base64

    # REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
    host = '<spark cluster name>.azurehdinsight.net:443'
    username='<username>'
    password='<password>'

    #AUTHORIZATION
    conn = httplib.HTTPSConnection(host)
    auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
    headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}

    # SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
    # IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
    r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
    response = conn.getresponse().read()
    print(response)
    conn.close()


您也可以將此 Python 程式碼新增至 [Azure Functions](https://azure.microsoft.com/documentation/services/functions/) 以觸發 Spark 作業提交，以根據各種事件 (像是計時器、建立或更新 blob) 來評分 blob。 

如果您偏好程式碼可用的用戶端體驗，請使用 [Azure Logic Apps](https://azure.microsoft.com/documentation/services/app-service/logic/) 來叫用 Spark 批次評分，方法是在 **Logic Apps Designer** 上定義 HTTP 動作並設定它的參數。 

* 在 Azure 入口網站中，選取 [+ 新增]  ->  [Web + 行動 ]  ->  [邏輯應用程式] 來建立新的邏輯應用程式。 
* 若要引進 **Logic Apps Designer**，請輸入邏輯應用程式和 App Service 方案的名稱。
* 選取 HTTP 動作，然後輸入下圖顯示的參數︰

![Logic Apps 設計工具](./media/spark-model-consumption/spark-logica-app-client.png)

## <a name="whats-next"></a>後續步驟
**交叉驗證和超參數掃掠**：如需如何使用交叉驗證和超參數掃掠訓練模型的相關資訊，請參閱 [使用 Spark 進階資料探索和模型化](spark-advanced-data-exploration-modeling.md)

