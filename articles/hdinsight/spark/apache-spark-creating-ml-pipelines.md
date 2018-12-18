---
title: 建立 Apache Spark 機器學習服務管線 - Azure HDInsight
description: 使用 Apache Spark 機器學習服務程式庫以建立資料管線。
services: hdinsight
ms.service: hdinsight
author: maxluk
ms.author: maxluk
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: 8a1148bbb04bbad2ff25be9bcbe68d086e91027c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049248"
---
# <a name="create-a-spark-machine-learning-pipeline"></a>建立 Spark 機器學習管線

Apache Spark 可調整機器學習服務程式庫 (MLlib) 可將模型化功能引進分散式環境。 Spark 封裝 [`spark.ml`](http://spark.apache.org/docs/latest/ml-pipeline.html) 是 DataFrame 上建立的一組高階 API。 這些 API 可協助您建立及調整實用的機器學習服務管線。  *Spark 機器學習服務*是指以 MLlib DataFrame 為基礎的 API，而不是之前以 RDD 為基礎的管線 API。

機器學習服務 (ML) 管線是將多個機器學習服務演算法結合在一起的完整工作流程。 需要許多步驟以進行處理並從資料學習，需要一連串的演算法。 管線定義機器學習服務流程的階段和順序。 在 MLlib 中，管線階段會以 PipelineStages 的特定順序表示，其中 Transformer 和 Estimator 會各自執行工作。

Transformer 演算法會藉由使用 `transform()` 方法，將 DataFrame 轉換到另一個 DataFrame。 例如，功能轉換器可讀取一行 DataFrame，將其對應至另一個資料行，並利用附加至該轉換器的對應資料行，輸出新的 DataFrame。

Estimator 是學習抽象的演算法，負責對資料集進行調整或定型以產生 Transformer。 Estimator 會實作名為 `fit()` 的方法，其接受 DataFrame 並產生 DataFrame，也就是 Transformer。

Transformer 或 Estimator 的每個無狀態執行個體皆有自己唯一的識別碼，指定參數時，會使用該識別碼。 兩者都使用統一的 API 來指定這些參數。

## <a name="pipeline-example"></a>管線範例

為了示範機器學習服務管線的實際應用情況，此範例針對您的 HDInsight 叢集 (Azure 儲存體或 Data Lake Store)，使用在預設儲存體中預先載入的 `HVAC.csv` 資料檔案範例。 若要檢視檔案的內容，請瀏覽至 `/HdiSamples/HdiSamples/SensorSampleData/hvac` 目錄。 `HVAC.csv` 包含一系列各種建築物中 HVAC (暖氣、通風和空調) 系統的目標和實際溫度。 目標是在資料上定型模型，並為指定的建築物產生預測溫度。

下列程式碼：

1. 定義 `LabeledDocument`，可儲存 `BuildingID`、`SystemInfo` (系統的識別碼和存留期)，並定義 `label` (如果建築物太熱，則為 1.0，否則為 0.0)。
2. 建立採用單一行 (列) 資料的自訂剖析器函式 `parseDocument`，並比較目標溫度與實際溫度以判斷建築物是否為「熱」。
3. 擷取來源資料時，套用剖析器。
4. 建立定型資料。

```python
# The data structure (column meanings) of the data array:
# 0 Date
# 1 Time
# 2 TargetTemp
# 3 ActualTemp
# 4 System
# 5 SystemAge
# 6 BuildingID

LabeledDocument = Row("BuildingID", "SystemInfo", "label")

# Define a function that parses the raw CSV file and returns an object of type LabeledDocument
def parseDocument(line):
    values = [str(x) for x in line.split(',')]
    if (values[3] > values[2]):
        hot = 1.0
    else:
        hot = 0.0        

    textValue = str(values[4]) + " " + str(values[5])

    return LabeledDocument((values[6]), textValue, hot)

# Load the raw HVAC.csv file, parse it using the function
data = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
training = documents.toDF()
```

此範例管線有三個階段：`Tokenizer` 和 `HashingTF` (皆為 Transformer)，以及 `Logistic Regression` (Estimator)。  呼叫 `pipeline.fit(training)` 時，`training` DataFrame 中的擷取和剖析資料會流經管線。

1. 第一階段，`Tokenizer`，將 `SystemInfo` 輸入資料行 (包含系統識別碼和存留期值) 分割成 `words` 輸出資料行。 這個新的 `words` 資料行會新增至 DataFrame。 
2. 第二階段，`HashingTF`，將新的 `words` 資料行轉換至功能向量。 這個新的 `features` 資料行會新增至 DataFrame。 前兩個階段為 Transformer。 
3. 第三階段，`LogisticRegression`，為 Estimator，因此管線會呼叫 `LogisticRegression.fit()` 方法以產生 `LogisticRegressionModel`。 

```python
tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.01)

# Build the pipeline with our tokenizer, hashingTF, and logistic regression stages
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

model = pipeline.fit(training)
```

若要查看由 `Tokenizer` 和 `HashingTF` Transformer 新增的新 `words` 和 `features` 資料行，以及 `LogisticRegression` Estimator 的範例，請在原始的 DataFrame 上執行 `PipelineModel.transform()` 方法。 在實際執行程式碼中，下一步是傳入測試 DataFrame 以驗證定型。

```python
peek = model.transform(training)
peek.show()

# Outputs the following:
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+
|BuildingID|SystemInfo|label|   words|            features|       rawPrediction|         probability|prediction|
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+
|         4|     13 20|  0.0|[13, 20]|(262144,[250802,2...|[0.11943986671420...|[0.52982451901740...|       0.0|
|        17|      3 20|  0.0| [3, 20]|(262144,[89074,25...|[0.17511205617446...|[0.54366648775222...|       0.0|
|        18|     17 20|  1.0|[17, 20]|(262144,[64358,25...|[0.14620993833623...|[0.53648750722548...|       0.0|
|        15|      2 23|  0.0| [2, 23]|(262144,[31351,21...|[-0.0361327091023...|[0.49096780538523...|       1.0|
|         3|      16 9|  1.0| [16, 9]|(262144,[153779,1...|[-0.0853679939336...|[0.47867095324139...|       1.0|
|         4|     13 28|  0.0|[13, 28]|(262144,[69821,25...|[0.14630166986618...|[0.53651031790592...|       0.0|
|         2|     12 24|  0.0|[12, 24]|(262144,[187043,2...|[-0.0509556393066...|[0.48726384581522...|       1.0|
|        16|     20 26|  1.0|[20, 26]|(262144,[128319,2...|[0.33829638728900...|[0.58377663577684...|       0.0|
|         9|      16 9|  1.0| [16, 9]|(262144,[153779,1...|[-0.0853679939336...|[0.47867095324139...|       1.0|
|        12|       6 5|  0.0|  [6, 5]|(262144,[18659,89...|[0.07513008136562...|[0.51877369045183...|       0.0|
|        15|     10 17|  1.0|[10, 17]|(262144,[64358,25...|[-0.0291988646553...|[0.49270080242078...|       1.0|
|         7|      2 11|  0.0| [2, 11]|(262144,[212053,2...|[0.03678030020834...|[0.50919403860812...|       0.0|
|        15|      14 2|  1.0| [14, 2]|(262144,[109681,2...|[0.06216423725633...|[0.51553605651806...|       0.0|
|         6|       3 2|  0.0|  [3, 2]|(262144,[89074,21...|[0.00565582077537...|[0.50141395142468...|       0.0|
|        20|     19 22|  0.0|[19, 22]|(262144,[139093,2...|[-0.0769288695989...|[0.48077726176073...|       1.0|
|         8|     19 11|  0.0|[19, 11]|(262144,[139093,2...|[0.04988910033929...|[0.51246968885151...|       0.0|
|         6|      15 7|  0.0| [15, 7]|(262144,[77099,20...|[0.14854929135994...|[0.53706918109610...|       0.0|
|        13|      12 5|  0.0| [12, 5]|(262144,[89689,25...|[-0.0519932532562...|[0.48700461408785...|       1.0|
|         4|      8 22|  0.0| [8, 22]|(262144,[98962,21...|[-0.0120753606650...|[0.49698119651572...|       1.0|
|         7|      17 5|  0.0| [17, 5]|(262144,[64358,89...|[-0.0721054054871...|[0.48198145477106...|       1.0|
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+

only showing top 20 rows
```

`model` 物件現在可以用來進行預測。 如需此機器學習服務應用程式的完整範例，以及執行的逐步指示，請參閱[在 Azure HDInsight 上建立 Apache Spark 機器學習服務應用程式](apache-spark-ipython-notebook-machine-learning.md)。

## <a name="see-also"></a>另請參閱

* [在 Azure 上使用 Scala 與 Spark 的資料科學](../../machine-learning/team-data-science-process/scala-walkthrough.md)
