---
title: 在 HDInsight 上使用 Spark MLlib 建立機器學習的範例 - Azure
description: 了解如何使用 Spark MLlib 建立一個透過羅吉斯迴歸使用分類來分析資料集的機器學習應用程式。
keywords: spark 機器學習, spark 機器學習範例
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/18/2018
ms.author: jasonh
ms.openlocfilehash: 78f9240e6b01bafc68b71d20044c7ec7458cc972
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047291"
---
# <a name="use-spark-mllib-to-build-a-machine-learning-application-and-analyze-a-dataset"></a>使用 Spark MLlib 建置機器學習應用程式及分析資料集

了解如何使用 Spark [MLlib](https://spark.apache.org/mllib/) 建立機器學習應用程式，以在開啟的資料集上進行簡單的預測分析。 從 Spark 的內建機器學習程式庫，此範例會透過羅吉斯迴歸使用「分類」。 

> [!TIP]
> 本範例也作為您在 HDInsight 中所建立之 Spark (Linux) 叢集上的 Jupyter Notebook 提供使用。 Notebook 的體驗能讓您從 Notebook 本身執行 Python 程式碼片段。 若要依照 Notebook 中的教學課程執行，請建立 Spark 叢集並啟動 Jupyter Notebook (`https://CLUSTERNAME.azurehdinsight.net/jupyter`)。 然後，執行 **Python** 資料夾下的 Notebook **Spark 機器學習 - 使用 MLlib.ipynb 進行食品檢查資料的預測分析**。
>
>

MLlib 是核心 Spark 程式庫之一，提供許多可用於機器學習工作的公用程式，包括具有下列用途的公用程式：

* 分類
* 迴歸
* 叢集
* 主題模型化
* 奇異值分解 (SVD) 和主體元件分析 (PCA)
* 假設測試和計算範例統計資料

## <a name="understand-classification-and-logistic-regression"></a>了解分類和羅吉斯迴歸
分類是常見的機器學習工作，是指將輸入資料依類別排序的程序。 它是以分類演算法指出如何為您所提供的輸入資料指派「標籤」的作業。 例如，試想某個機器學習演算法以股市資訊作為輸入，並且將股票分成兩個類別：該賣的股票和該留的股票。

羅吉斯迴歸是您用於分類的演算法。 Spark 的羅吉斯迴歸 API 可用於*二元分類*，或用來將輸入資料歸類到兩個群組之一。 如需羅吉斯迴歸的詳細資訊，請參閱 [Wikipedia](https://en.wikipedia.org/wiki/Logistic_regression)。

總之，羅吉斯迴歸的程序會產生一個*羅吉斯函數* ，此函數可用來預測輸入向量可能屬於哪一個群組的機率。  

## <a name="predictive-analysis-example-on-food-inspection-data"></a>食品檢查資料的預測分析範例
在此範例中，您將使用 Spark 對透過[芝加哥市資料入口網站](https://data.cityofchicago.org/) \(英文\) 取得的食品檢查資料 (**Food_Inspections1.csv**) 執行某項預測分析。 此資料集包含在芝加哥進行餐飲業檢查的相關資訊，其中包括每個受檢餐飲業者、發現的違規情事 (如果有的話) 和檢查結果的相關資訊。 與叢集相關聯的儲存體帳戶已有 CSV 資料檔，此叢集位於 **/HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections1.csv**。

在下列步驟中，您會開發一個模型來觀察能否通過食品檢查的標準為何。

## <a name="create-a-spark-mllib-machine-learning-app"></a>建立 Spark MLlib 機器學習應用程式

1. 使用 PySpark 核心建立 Jupyter Notebook。 如需指示，請參閱[建立 Jupyter Notebook](./apache-spark-jupyter-spark-sql.md#create-a-jupyter-notebook)。

2. 匯入此應用程式所需的類型。 複製以下程式碼並貼入空白儲存格，然後按下 **SHIFT + ENTER** 鍵。

    ```PySpark
    from pyspark.ml import Pipeline
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml.feature import HashingTF, Tokenizer
    from pyspark.sql import Row
    from pyspark.sql.functions import UserDefinedFunction
    from pyspark.sql.types import *
    ```
    由於使用 PySpark 核心，因此不需要明確建立任何內容。 當您執行第一個程式碼儲存格時，系統會自動為您建立 Spark 和 Hive 內容。 

## <a name="construct-the-input-dataframe"></a>建構輸入資料框架

由於未經處理資料採用 CSV 格式，因此您可以使用 Spark 內容，將檔案以非結構化文字的形式提取至記憶體中，然後再使用 Python 的 CSV 程式庫剖析每一行資料。

1. 請執行下列幾行，透過匯入和剖析輸入資料來建立具有恢復功能的分散式資料集 (RDD)。

    ```PySpark
    def csvParse(s):
        import csv
        from StringIO import StringIO
        sio = StringIO(s)
        value = csv.reader(sio).next()
        sio.close()
        return value
    
    inspections = sc.textFile('wasb:///HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections1.csv')\
                    .map(csvParse)
    ```

2. 執行下列程式碼，可從 RDD 擷取一個資料列，因此您可以查看資料結構描述：

    ```PySpark
    inspections.take(1)
    ```

    輸出如下：

    ```
    [['413707',
        'LUNA PARK INC',
        'LUNA PARK  DAY CARE',
        '2049789',
        "Children's Services Facility",
        'Risk 1 (High)',
        '3250 W FOSTER AVE ',
        'CHICAGO',
        'IL',
        '60625',
        '09/21/2010',
        'License-Task Force',
        'Fail',
        '24. DISH WASHING FACILITIES: PROPERLY DESIGNED, CONSTRUCTED, MAINTAINED, INSTALLED, LOCATED AND OPERATED - Comments: All dishwashing machines must be of a type that complies with all requirements of the plumbing section of the Municipal Code of Chicago and Rules and Regulation of the Board of Health. OBSEVERD THE 3 COMPARTMENT SINK BACKING UP INTO THE 1ST AND 2ND COMPARTMENT WITH CLEAR WATER AND SLOWLY DRAINING OUT. INST NEED HAVE IT REPAIR. CITATION ISSUED, SERIOUS VIOLATION 7-38-030 H000062369-10 COURT DATE 10-28-10 TIME 1 P.M. ROOM 107 400 W. SURPERIOR. | 36. LIGHTING: REQUIRED MINIMUM FOOT-CANDLES OF LIGHT PROVIDED, FIXTURES SHIELDED - Comments: Shielding to protect against broken glass falling into food shall be provided for all artificial lighting sources in preparation, service, and display facilities. LIGHT SHIELD ARE MISSING UNDER HOOD OF  COOKING EQUIPMENT AND NEED TO REPLACE LIGHT UNDER UNIT. 4 LIGHTS ARE OUT IN THE REAR CHILDREN AREA,IN THE KINDERGARDEN CLASS ROOM. 2 LIGHT ARE OUT EAST REAR, LIGHT FRONT WEST ROOM. NEED TO REPLACE ALL LIGHT THAT ARE NOT WORKING. | 35. WALLS, CEILINGS, ATTACHED EQUIPMENT CONSTRUCTED PER CODE: GOOD REPAIR, SURFACES CLEAN AND DUST-LESS CLEANING METHODS - Comments: The walls and ceilings shall be in good repair and easily cleaned. MISSING CEILING TILES WITH STAINS IN WEST,EAST, IN FRONT AREA WEST, AND BY THE 15MOS AREA. NEED TO BE REPLACED. | 32. FOOD AND NON-FOOD CONTACT SURFACES PROPERLY DESIGNED, CONSTRUCTED AND MAINTAINED - Comments: All food and non-food contact equipment and utensils shall be smooth, easily cleanable, and durable, and shall be in good repair. SPLASH GUARDED ARE NEEDED BY THE EXPOSED HAND SINK IN THE KITCHEN AREA | 34. FLOORS: CONSTRUCTED PER CODE, CLEANED, GOOD REPAIR, COVING INSTALLED, DUST-LESS CLEANING METHODS USED - Comments: The floors shall be constructed per code, be smooth and easily cleaned, and be kept clean and in good repair. INST NEED TO ELEVATE ALL FOOD ITEMS 6INCH OFF THE FLOOR 6 INCH AWAY FORM WALL.  ',
        '41.97583445690982',
        '-87.7107455232781',
        '(41.97583445690982, -87.7107455232781)']]
    ```

    輸出可讓您了解輸入檔案的結構描述。 它包括每個餐飲業者的名稱、類型，地址，檢查資料和位置等等。 

3. 執行下列程式碼來建立資料框架 (df) 和暫存資料表 (CountResults)，其中具有一些可用於預測分析的資料行。 您可以使用 `sqlContext` 對結構化資料執行轉換。 

    ```PySpark
    schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), False),
    StructField("results", StringType(), False),
    StructField("violations", StringType(), True)])
    
    df = spark.createDataFrame(inspections.map(lambda l: (int(l[0]), l[1], l[12], l[13])) , schema)
    df.registerTempTable('CountResults')
    ```

    資料框架中的四個相關資料行為 **id**、**name**、**results** 與 **violations**。

4. 執行下列程式碼，可取得一小部分的資料範例：

    ```PySpark
    df.show(5)
    ```

    輸出如下：

    ```
    +------+--------------------+-------+--------------------+
    |    id|                name|results|          violations|
    +------+--------------------+-------+--------------------+
    |413707|       LUNA PARK INC|   Fail|24. DISH WASHING ...|
    |391234|       CAFE SELMARIE|   Fail|2. FACILITIES TO ...|
    |413751|          MANCHU WOK|   Pass|33. FOOD AND NON-...|
    |413708|BENCHMARK HOSPITA...|   Pass|                    |
    |413722|           JJ BURGER|   Pass|                    |
    +------+--------------------+-------+--------------------+
    ```

## <a name="understand-the-data"></a>了解資料

我們要著手了解資料集的內容為何。 

1. 執行下列程式碼，可顯示**結果 (results)** 資料行中的相異值：

    ```PySpark
    df.select('results').distinct().show()
    ```

    輸出如下：

    ```
    +--------------------+
    |             results|
    +--------------------+
    |                Fail|
    |Business Not Located|
    |                Pass|
    |  Pass w/ Conditions|
    |     Out of Business|
    +--------------------+
    ```

2. 執行下列程式碼，可將這些結果的分佈視覺化：

    ```PySpark
    %%sql -o countResultsdf
    SELECT results, COUNT(results) AS cnt FROM CountResults GROUP BY results
    ```

    `%%sql` magic 後面緊接著 `-o countResultsdf` 可確保查詢的輸出會保存在 Jupyter 伺服器的本機上 (通常是叢集的前端節點)。 輸出會使用指定的名稱 [countResultsdf](http://pandas.pydata.org/) ，當做 **Pandas**資料框架保存。 如需 `%%sql` magic 及 PySpark 核心提供的其他 magic 的詳細資訊，請參閱 [使用 Spark HDInsight 叢集之 Jupyter Notebook 上可用的核心](apache-spark-jupyter-notebook-kernels.md#parameters-supported-with-the-sql-magic)。

    輸出如下：

    ![SQL 查詢輸出](./media/apache-spark-machine-learning-mllib-ipython/spark-machine-learning-query-output.png "SQL 查詢輸出")


3. 您也可以使用 [Matplotlib](https://en.wikipedia.org/wiki/Matplotlib) (用於建構資料視覺效果的程式庫) 建立繪圖。 因為必須從保存在本機上的 **countResultsdf** 資料框架建立繪圖，所以程式碼片段的開頭必須為 `%%local` magic。 這可確保程式碼是在 Jupyter 伺服器的本機上執行。

    ```PySpark
    %%local
    %matplotlib inline
    import matplotlib.pyplot as plt

    labels = countResultsdf['results']
    sizes = countResultsdf['cnt']
    colors = ['turquoise', 'seagreen', 'mediumslateblue', 'palegreen', 'coral']
    plt.pie(sizes, labels=labels, autopct='%1.1f%%', colors=colors)
    plt.axis('equal')
    ```

    輸出如下：

    ![Spark 機器學習應用程式輸出 - 包含五個不同檢查結果的圓形圖](./media/apache-spark-machine-learning-mllib-ipython/spark-machine-learning-result-output-1.png "Spark 機器學習結果輸出")

    一項檢查可以有 5 個不同的結果：

    - 找不到該業者
    - 不合格
    - 通過
    - 有條件通過
    - 已結束營業

    若要預測食物檢查結果，您需要根據違規事項來開發模型。 由於羅吉斯迴歸是一種二元分類方法，因此將結果資料分成兩個類別，是很合理的：**不合格**和**通過**：

    - 通過
        - 通過
        - 有條件通過
    - 不合格
        - 不合格
    - 捨棄
        - 找不到該業者
        - 已結束營業

    其他結果的資料 (「找不到該業者」或「已結束營業」) 並不是很有用處，而且這些資料在結果中所佔的比例非常小。

4. 執行下列程式碼，可將現有資料框架 (`df`) 轉換為新的資料框架，其中每項檢查都以一組「標籤-違規」來表示。 在此案例中，標籤 `0.0` 代表失敗，標籤 `1.0` 代表成功，標籤 `-1.0` 代表此二者以外的某種結果。 

    ```PySpark
    def labelForResults(s):
        if s == 'Fail':
            return 0.0
        elif s == 'Pass w/ Conditions' or s == 'Pass':
            return 1.0
        else:
            return -1.0
    label = UserDefinedFunction(labelForResults, DoubleType())
    labeledData = df.select(label(df.results).alias('label'), df.violations).where('label >= 0')
    ```

5. 執行下列程式碼，可顯示一個加上標籤的資料列：

    ```PySpark
    labeledData.take(1)
    ```

    輸出如下：

    ```
    [Row(label=0.0, violations=u"41. PREMISES MAINTAINED FREE OF LITTER, UNNECESSARY ARTICLES, CLEANING  EQUIPMENT PROPERLY STORED - Comments: All parts of the food establishment and all parts of the property used in connection with the operation of the establishment shall be kept neat and clean and should not produce any offensive odors.  REMOVE MATTRESS FROM SMALL DUMPSTER. | 35. WALLS, CEILINGS, ATTACHED EQUIPMENT CONSTRUCTED PER CODE: GOOD REPAIR, SURFACES CLEAN AND DUST-LESS CLEANING METHODS - Comments: The walls and ceilings shall be in good repair and easily cleaned.  REPAIR MISALIGNED DOORS AND DOOR NEAR ELEVATOR.  DETAIL CLEAN BLACK MOLD LIKE SUBSTANCE FROM WALLS BY BOTH DISH MACHINES.  REPAIR OR REMOVE BASEBOARD UNDER DISH MACHINE (LEFT REAR KITCHEN). SEAL ALL GAPS.  REPLACE MILK CRATES USED IN WALK IN COOLERS AND STORAGE AREAS WITH PROPER SHELVING AT LEAST 6' OFF THE FLOOR.  | 38. VENTILATION: ROOMS AND EQUIPMENT VENTED AS REQUIRED: PLUMBING: INSTALLED AND MAINTAINED - Comments: The flow of air discharged from kitchen fans shall always be through a duct to a point above the roofline.  REPAIR BROKEN VENTILATION IN MEN'S AND WOMEN'S WASHROOMS NEXT TO DINING AREA. | 32. FOOD AND NON-FOOD CONTACT SURFACES PROPERLY DESIGNED, CONSTRUCTED AND MAINTAINED - Comments: All food and non-food contact equipment and utensils shall be smooth, easily cleanable, and durable, and shall be in good repair.  REPAIR DAMAGED PLUG ON LEFT SIDE OF 2 COMPARTMENT SINK.  REPAIR SELF CLOSER ON BOTTOM LEFT DOOR OF 4 DOOR PREP UNIT NEXT TO OFFICE.")]
    ```

## <a name="create-a-logistic-regression-model-from-the-input-dataframe"></a>從輸入資料框架建立羅吉斯迴歸模型

最後一項工作，是將加上標籤的資料轉換成可依羅吉斯迴歸進行分析的格式。 羅吉斯迴歸演算法的輸入必須是一組「標籤-特性向量配對」，其中「特性向量」是代表輸入點的數字向量。 因此，您需要將半結構化且包含許多自然語言註解的「違規情事」資料行，轉換成機器可輕易辨識的實數陣列。

可用來處理自然語言的標準機器學習方法之一，是為每個不同的字指派一個「索引」，然後將向量傳至機器學習演算法，使每個索引的值包含該字在文字字串中出現的相對頻率。

MLlib 可提供簡單的方法來執行此作業。 首先，將每個違規情事字串語彙基元化，以取得字串中的個別字。 然後，使用 `HashingTF` 將每組語彙基元轉換為特性向量，接著可以將它傳遞給羅吉斯迴歸演算法以構建模型。 您將使用「管線」循序執行上述所有步驟。

```PySpark
tokenizer = Tokenizer(inputCol="violations", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.01)
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

model = pipeline.fit(labeledData)
```

## <a name="evaluate-the-model-using-another-dataset"></a>使用另一個資料集來評估模型

您可以使用先前建立的模型，根據所觀察的違規情事，*預測*新的檢查會有何種結果。 您已在資料集 **Food_Inspections1.csv** 上訓練此模型。 您可以使用第二個資料集 **Food_Inspections2.csv**，評估此模型對於新資料的強度。 第二個資料集 (**Food_Inspections2.csv**) 在與叢集相關聯的預設儲存體容器中。

1. 執行下列程式碼，可建立新的資料框架 **predictionsDf**，其中包含模型所產生的預測。 該程式碼片段也會根據資料框架，建立暫存資料表 **Predictions**。

    ```PySpark
    testData = sc.textFile('wasb:///HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections2.csv')\
                .map(csvParse) \
                .map(lambda l: (int(l[0]), l[1], l[12], l[13]))
    testDf = spark.createDataFrame(testData, schema).where("results = 'Fail' OR results = 'Pass' OR results = 'Pass w/ Conditions'")
    predictionsDf = model.transform(testDf)
    predictionsDf.registerTempTable('Predictions')
    predictionsDf.columns
    ```

    您應該會看到如下的輸出：

    ```
    # -----------------
    # THIS IS AN OUTPUT
    # -----------------

    ['id',
        'name',
        'results',
        'violations',
        'words',
        'features',
        'rawPrediction',
        'probability',
        'prediction']
    ```

1. 請看其中一個預測。 執行此程式碼片段：

    ```PySpark
    predictionsDf.take(1)
    ```

   針對測試資料集中的第一個項目有一個預測。
1. `model.transform()` 方法會將相同的轉換套用至具有相同結構描述的任何新資料，並做出關於如何分類資料的預測。 您可以做一些簡單的統計，了解一下預測精準度如何：

    ```PySpark
    numSuccesses = predictionsDf.where("""(prediction = 0 AND results = 'Fail') OR
                                            (prediction = 1 AND (results = 'Pass' OR
                                                                results = 'Pass w/ Conditions'))""").count()
    numInspections = predictionsDf.count()

    print "There were", numInspections, "inspections and there were", numSuccesses, "successful predictions"
    print "This is a", str((float(numSuccesses) / float(numInspections)) * 100) + "%", "success rate"
    ```

    輸出顯示如下：

    ```
    # -----------------
    # THIS IS AN OUTPUT
    # -----------------

    There were 9315 inspections and there were 8087 successful predictions
    This is a 86.8169618894% success rate
    ```

    透過 Spark 使用羅吉斯迴歸，您找出了英文版的違規情事說明與指定企業是否能通過食品檢查之間的關聯性，而建立了兩者關聯性的精確模型。

## <a name="create-a-visual-representation-of-the-prediction"></a>建立預測的視覺表示法
您現在可以建構最終的視覺效果，以利研判此測試的結果。

1. 您可以從擷取稍早建立的 **Predictions** 暫存資料表中不同的預測和結果開始。 下列查詢會將輸出分隔為 *true_positive*、*false_positive*、*true_negative* 和 *false_negative*。 在下面的查詢中，可以使用 `-q` 關閉視覺效果，並 (使用 `-o`) 將輸出儲存為可和 `%%local` magic 搭配使用的資料框架。

    ```PySpark
    %%sql -q -o true_positive
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 0 AND results = 'Fail'
    ```

    ```PySpark
    %%sql -q -o false_positive
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 0 AND (results = 'Pass' OR results = 'Pass w/ Conditions')
    ```

    ```PySpark
    %%sql -q -o true_negative
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 1 AND results = 'Fail'
    ```

    ```PySpark
    %%sql -q -o false_negative
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 1 AND (results = 'Pass' OR results = 'Pass w/ Conditions')
    ```

1. 最後，使用 **Matplotlib**用下列程式碼片段產生繪圖。

    ```PySpark
    %%local
    %matplotlib inline
    import matplotlib.pyplot as plt

    labels = ['True positive', 'False positive', 'True negative', 'False negative']
    sizes = [true_positive['cnt'], false_positive['cnt'], false_negative['cnt'], true_negative['cnt']]
    colors = ['turquoise', 'seagreen', 'mediumslateblue', 'palegreen', 'coral']
    plt.pie(sizes, labels=labels, autopct='%1.1f%%', colors=colors)
    plt.axis('equal')
    ```

    您應該會看見下列輸出：

    ![Spark 機器學習應用程式輸出 - 包含失敗食品檢查百分比的圓形圖。](./media/apache-spark-machine-learning-mllib-ipython/spark-machine-learning-result-output-2.png "Spark 機器學習結果輸出")

    在此圖中，「肯定」結果是指未通過的食品檢查，否定結果則是指通過的檢查。

## <a name="shut-down-the-notebook"></a>關閉 Notebook
應用程式執行完畢之後，您應該關閉 Notebook 以釋放資源。 若要這樣做，請從 Notebook 的 [檔案] 功能表中，按一下 [關閉並停止]。 這樣會關機並關閉 Notebook。

## <a name="seealso"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
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
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
