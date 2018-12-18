---
title: 具有 Azure Cosmos DB 和 HDInsight (Apache Spark) 的 Lambda 架構 | Microsoft Docs
description: 本文描述如何使用 Azure Cosmos DB、HDInsight 及 Spark 實作 Lambda 架構
keywords: lambda-architecture
services: cosmos-db
author: tknandu
manager: kfile
editor: ''
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 01/19/2018
ms.author: ramkris
ms.openlocfilehash: c926c67a330648e09c1fd8133164f64582ad9a34
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701070"
---
# <a name="azure-cosmos-db-implement-a-lambda-architecture-on-the-azure-platform"></a>Azure Cosmos DB：在 Azure 平台上實作 Lambda 架構 

Lambda 架構能夠對於大量資料集進行高效率的資料處理。 Lambda 架構使用批次處理、資料流處理和服務層，大幅減少與查詢巨量資料有關的延遲。 

若要在 Azure 上實作 Lambda 架構，您可以結合下列技術，以加速即時巨量資料分析：
* [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 是業界第一個全域分散式多模型資料庫服務。 
* [Apache Spark for Azure HDInsight](https://azure.microsoft.com/services/hdinsight/apache-spark/)，可執行大規模資料分析應用程式的處理架構
* Azure Cosmos DB [變更摘要](change-feed.md)，這會新資料串流至批次層，以供 HDInsight 處理
* [Spark 至 Azure Cosmos DB 連接器](spark-connector.md)

本文描述根據原始多層設計的 Lambda 架構本身的基本概念，以及簡化作業的「重新架構」Lambda 架構所具有的優點。  

## <a name="what-is-a-lambda-architecture"></a>什麼是 Lambda 架構？
Lambda 架構為泛型、可擴充且容錯的資料處理架構，可因應 [Nathan Marz](https://twitter.com/nathanmarz) 所述的批次和速度延遲情況。

![顯示 Lambda 架構的圖表](./media/lambda-architecture/lambda-architecture-intro.png)

來源： http://lambda-architecture.net/

上圖根據 [https://lambda-architecture.net](http://lambda-architecture.net/) 描述 Lambda 架構的基本原則。

 1. 所有**資料**都會被推送至*這兩個**批次層*和*速度層*。
 2. **批次層**有主要的資料集 (固定、僅附加的原始資料集) 並預先計算批次檢視。
 3. **服務層**有可供快速查詢的批次檢視。 
 4. **速度層**可補償處理時間 (對於服務層) 並且僅處理最近的資料。
 5. 合併批次檢視和即時檢視的結果，或個別進行這些結果的 ping，均可回答所有查詢。

進一步讀取時，我們可以僅採用下列項目實作此架構：

* Azure Cosmos DB 集合
* HDInsight (Apache Spark 2.1) 叢集
* Spark 連接器 [1.0](https://github.com/Azure/azure-cosmosdb-spark/tree/master/releases/azure-cosmosdb-spark_2.1.0_2.11-1.0.0)

## <a name="speed-layer"></a>速度層

就作業的觀點而言，維護兩個資料流的資料，同時確保資料的正確狀態，是相當複雜的工作。 若要簡化作業，可利用 [Azure Cosmos DB 變更摘要支援](change-feed.md)來保留「批次層」的狀態，同時對於「速度層」*變更摘要 API*，揭露 Azure Cosmos DB 變更記錄檔。  
![反白顯示新資料、速度層和 Lambda 架構主要資料集部分的圖表](./media/lambda-architecture/lambda-architecture-change-feed.png)

這些層的重要部分：

 1. 所有**資料**都「僅」會推入到 Azure Cosmos DB，因此可以避免多轉換問題。
 2. **批次層**有主要的資料集 (固定、僅附加的原始資料集) 並預先計算批次檢視。
 3. 下節將討論**服務層**。
 4. **速度層**利用 HDInsight (Apache Spark) 讀取 Azure Cosmos DB 變更摘要。 這可讓您保存您的資料，而且可讓您查詢和並行處理這些資料。
 5. 合併批次檢視和即時檢視的結果，或個別進行這些結果的 ping，均可回答所有查詢。
 
### <a name="code-example-spark-structured-streaming-to-an-azure-cosmos-db-change-feed"></a>程式碼範例：Azure Cosmos DB 變更摘要的 Spark 結構化串流
若要在**速度層**中執行 Azure Cosmos DB 變更摘要的快速原型，可以在[使用 Azure Cosmos DB 變更摘要和 Apache Spark 串流處理變更](https://github.com/Azure/azure-cosmosdb-spark/wiki/Stream-Processing-Changes-using-Azure-Cosmos-DB-Change-Feed-and-Apache-Spark) (英文) 範例中使用 Twitter 資料進行測試。 若要快速啟動您的 Twitter 輸出，請參閱[將 Twitter 的摘要串流到 Cosmos DB](https://github.com/tknandu/TwitterCosmosDBFeed) (英文) 中的程式碼範例。 對於上述範例，您會將 Twitter 資料載入 Azure Cosmos DB，然後您即可設定 HDInsight (Apache Spark) 叢集連線到變更摘要。 如需如何設定此設定的詳細資訊，請參閱 [Apache Spark 至 Azure Cosmos DB 連接器設定](https://github.com/Azure/azure-cosmosdb-spark/wiki/Spark-to-Cosmos-DB-Connector-Setup) (英文)。  

下列程式碼片段示範如何設定 `spark-shell` 執行結構化串流工作連接至 Azure Cosmos DB 變更摘要，這將檢閱即時 Twitter 資料流，以便進行執行間隔計數。

```
// Import Libraries
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.codehaus.jackson.map.ObjectMapper
import com.microsoft.azure.cosmosdb.spark.streaming._
import java.time._


// Configure connection to Azure Cosmos DB Change Feed
val sourceConfigMap = Map(
"Endpoint" -> "[COSMOSDB ENDPOINT]",
"Masterkey" -> "[MASTER KEY]",
"Database" -> "[DATABASE]",
"Collection" -> "[COLLECTION]",
"ConnectionMode" -> "Gateway",
"ChangeFeedCheckpointLocation" -> "checkpointlocation",
"changefeedqueryname" -> "Streaming Query from Cosmos DB Change Feed Interval Count")

// Start reading change feed as a stream
var streamData = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(sourceConfigMap).load()

// Start streaming query to console sink
val query = streamData.withColumn("countcol", streamData.col("id").substr(0, 0)).groupBy("countcol").count().writeStream.outputMode("complete").format("console").start()
```

如需完整的程式碼範例，請參閱 [azure-cosmosdb-spark/lambda/samples](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/lambda)，其中包括：
* [Cosmos DB 變更 Feed.scala 的串流查詢](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Streaming%20Query%20from%20Cosmos%20DB%20Change%20Feed.scala) (英文)
* [Cosmos DB 變更 Feed.scala 的串流標籤查詢](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Streaming%20Tags%20Query%20from%20Cosmos%20DB%20Change%20Feed%20.scala) (英文)

這個的輸出是 `spark-shell` 主控台，這會持續執行結構化資料流工作，對於來自 Azure Cosmos DB 變更摘要的 Twitter 資料執行間隔計算。 下圖顯示資料流工作輸出和計算間隔。

![串流輸出顯示對於來自 Azure Cosmos DB 變更摘要的 Twitter 資料執行的間隔計算](./media/lambda-architecture/lambda-architecture-speed-layer-twitter-count.png) 

如需 Azure Cosmos DB 變更摘要的詳細資訊，請參閱：

* [使用 Azure Cosmos DB 中的變更摘要支援](change-feed.md)
* [引入 Azure Cosmos DB 變更摘要處理器程式庫](https://azure.microsoft.com/blog/introducing-the-azure-cosmosdb-change-feed-processor-library/) (英文)
* [資料流處理變更：Azure CosmosDB 變更摘要 + Apache Spark](https://azure.microsoft.com/blog/stream-processing-changes-azure-cosmosdb-change-feed-apache-spark/) (英文)

## <a name="batch-and-serving-layers"></a>批次和服務層
因為新的資料載入至 Azure Cosmos DB (變更摘要用於速度層)，因此這是**主要資料集** (固定、僅附加的原始資料集) 所在的位置。 從這裡開始，使用 HDInsight (Apache Spark) 執行從**批次層**至**服務層**的預先計算函式，如下圖所示：

![反白顯示 Lambda 架構批次層和服務層的圖表](./media/lambda-architecture/lambda-architecture-batch-serve.png)

這些層的重要部分：

 1. 所有**資料**都只會推入到 Azure Cosmos DB (避免多轉換問題)。
 2. **批次層**有主要的資料集 (固定、僅附加的原始資料集) 儲存於 Azure Cosmos DB。 使用 HDI Spark，您可以預先計算將在計算的批次檢視中儲存的彙總。
 3. **服務層**是有主要資料集和計算的批次檢視這兩個集合的 Azure Cosmos DB 資料庫。
 4. 本文稍後將討論**速度層**。
 5. 合併批次檢視和即時檢視的結果，或個別進行這些結果的 ping，均可回答所有查詢。

### <a name="code-example-pre-computing-batch-views"></a>程式碼範例：預先計算批次檢視
若要展示如何針對從 Apache Spark 到 Azure Cosmos DB 的**主要資料集**執行預先計算的檢視，請從筆記本 [Lambda 架構重新架構 - 批次層](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20Layer.ipynb) (英文) 和[Lambda 架構重新架構 - 服務層的批次](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20to%20Serving%20Layer.ipynb) (英文) 使用下列程式碼片段。 在此案例中，使用 Azure Cosmos DB 中儲存的 Twitter 資料。

首先使用下列 PySpark 程式碼在 Azure Cosmos DB 中建立 Twitter 資料的設定連線，建立設定連線。

```
# Configuration to connect to Azure Cosmos DB
tweetsConfig = {
  "Endpoint" : "[Endpoint URL]",
  "Masterkey" : "[Master Key]",
  "Database" : "[Database]",
  "Collection" : "[Collection]", 
  "preferredRegions" : "[Preferred Regions]",
  "SamplingRatio" : "1.0",
  "schema_samplesize" : "200000",
  "query_custom" : "[Cosmos DB SQL Query]"
}

# Create DataFrame
tweets = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**tweetsConfig).load()

# Create Temp View (to run Spark SQL statements)
tweets.createOrReplaceTempView("tweets")
```

接著，執行下列的 Spark SQL 陳述式，以判斷推文集的前 10 個雜湊標記。 對於此 Spark SQL 查詢，我們將在 Jupyter 筆記本中執行這個查詢，而輸出長條圖不會直接遵循這個程式碼片段。

```
%%sql
select hashtags.text, count(distinct id) as tweets
from (
  select 
    explode(hashtags) as hashtags,
    id
  from tweets
) a
group by hashtags.text
order by tweets desc
limit 10
```

![顯示每個雜湊標記推文數目的圖表](./media/lambda-architecture/lambda-architecture-batch-hashtags-bar-chart.png)

現在您已有了查詢，使用 Spark 連接器將查詢儲存回集合，將輸出資料儲存至不同的集合。  在此範例中，使用 Scala 展示連線。 和上述範例類似，建立設定連線將 Apache Spark 資料框架儲存至不同的 Azure Cosmos DB 集合。

```
val writeConfigMap = Map(
    "Endpoint" -> "[Endpoint URL]",
    "Masterkey" -> "[Master Key]",
    "Database" -> "[Database]",
    "Collection" -> "[New Collection]", 
    "preferredRegions" -> "[Preferred Regions]",
    "SamplingRatio" -> "1.0",
    "schema_samplesize" -> "200000"
)

// Configuration to write
val writeConfig = Config(writeConfigMap)

```

指定 `SaveMode` (指出是否要 `Overwrite` 或 `Append` 文件) 後，建立與上述範例中的 Spark SQL 查詢類似的 `tweets_bytags` 資料框架。  建立 `tweets_bytags` 資料框架後，您可以使用已先前指定的 `writeConfig` 進行的 `write` 方法儲存該資料框架。

```
// Import SaveMode so you can Overwrite, Append, ErrorIfExists, Ignore
import org.apache.spark.sql.{Row, SaveMode, SparkSession}

// Create new DataFrame of tweets tags
val tweets_bytags = spark.sql("select hashtags.text as hashtags, count(distinct id) as tweets from ( select explode(hashtags) as hashtags, id from tweets ) a group by hashtags.text order by tweets desc")

// Save to Cosmos DB (using Append in this case)
tweets_bytags.write.mode(SaveMode.Overwrite).cosmosDB(writeConfig)
```

這個最後一個陳述式現在已將您的 Spark 資料框架儲存到新的 Azure Cosmos DB 集合；就 Lambda 架構觀點而言，這是**服務層**內的**批次檢視**。
 
#### <a name="resources"></a>資源

如需完整的程式碼範例，請參閱 [azure-cosmosdb-spark/lambda/samples](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/lambda)，其中包括：
* Lambda 架構重新架構 - 批次層 [HTML](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20Layer.html) | [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20Layer.ipynb)
* Lambda 架構重新架構 - 服務層的批次 [HTML](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20to%20Serving%20Layer.html) | [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20to%20Serving%20Layer.ipynb)

## <a name="speed-layer"></a>速度層
如先前所述，使用 Azure Cosmos DB 變更摘要庫可讓您簡化批次層和速度層之間的作業。 在此架構中，使用 Apache Spark (透過 HDInsight) 對資料執行「結構化資料流」查詢。 您也可能會想要暫時保留結構化串流查詢的結果，以便其他系統可存取此資料。

![反白顯示 Lambda 架構速度層的圖表](./media/lambda-architecture/lambda-architecture-speed.png)

若要這樣做，請建立個別的 Azure Cosmos DB 集合，以儲存結構化串流查詢的結果。  這可讓您得到此資訊的其他系統存取，而不只是 Apache Spark。 另外，使用 Cosmos DB 存留時間 (TTL) 功能，可以設定在設定的持續時間之後自動刪除您的文件。  如需 Azure Cosmos DB TTL 功能的詳細資訊，請參閱[利用存留時間讓 Azure Cosmos DB 集合中的資料自動過期](time-to-live.md)

```
// Import Libraries
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.codehaus.jackson.map.ObjectMapper
import com.microsoft.azure.cosmosdb.spark.streaming._
import java.time._


// Configure connection to Azure Cosmos DB Change Feed
val sourceCollectionName = "[SOURCE COLLECTION NAME]"
val sinkCollectionName = "[SINK COLLECTION NAME]"

val configMap = Map(
"Endpoint" -> "[COSMOSDB ENDPOINT]",
"Masterkey" -> "[COSMOSDB MASTER KEY]",
"Database" -> "[DATABASE NAME]",
"Collection" -> sourceCollectionName,
"ChangeFeedCheckpointLocation" -> "changefeedcheckpointlocation")

val sourceConfigMap = configMap.+(("changefeedqueryname", "Structured Stream replication streaming test"))

// Start to read the stream
var streamData = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(sourceConfigMap).load()
val sinkConfigMap = configMap.-("collection").+(("collection", sinkCollectionName))

// Start the stream writer to new collection
val streamingQueryWriter = streamData.writeStream.format(classOf[CosmosDBSinkProvider].getName).outputMode("append").options(sinkConfigMap).option("checkpointLocation", "streamingcheckpointlocation")
var streamingQuery = streamingQueryWriter.start()

```

## <a name="lambda-architecture-rearchitected"></a>Lambda 架構：重新架構
如先前各節所述，您可以使用下列元件簡化原始 Lambda 架構：
* Azure Cosmos DB
* Azure Cosmos DB 變更摘要庫不需要在批次層與速度層之間進行資料的多點傳送
* HDInsight 上的 Apache Spark
* 適用於 Azure Cosmos DB 的 Spark 連接器

![顯示使用 Azure Cosmos DB、Spark 和 Azure Cosmos DB 變更摘要 API 重新架構 Lambda 架構的圖表](./media/lambda-architecture/lambda-architecture-re-architected.png)

藉由這個設計，您只需要 Azure Cosmos DB 和 HDInsight 這兩個受控服務。 這兩者可解決 Lambda 架構的批次、服務和速度層。 這不僅可簡化作業，也可簡化資料流程。 
 1. 所有資料都會推送至 Azure Cosmos DB 進行處理
 2. 批次層有主要的資料集 (固定、僅附加的原始資料集) 並預先計算批次檢視
 3. 服務層有可供快速查詢的資料批次檢視。
 4. 速度層可補償處理時間 (對於服務層) 並且僅處理最近的資料。
 5. 合併批次檢視和即時檢視的結果，均可回答所有查詢。

### <a name="resources"></a>資源

 * **新資料**：[從 Twitter 到 CosmosDB 的資料流摘要](https://github.com/tknandu/TwitterCosmosDBFeed)，這是將新資料推送至 Azure Cosmos DB 的機制。
 * **批次層：** 批次層包括「主要資料集」 (固定、僅附加的原始資料集)，而且能夠對於推入至**服務層**的資料預先計算批次檢視。
    * **Lambda 架構重新架構 - 批次層**筆記本 [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20Layer.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20Layer.html) 會查詢批次檢視的「主要資料集」。
 * **服務層：****服務層**包括預先計算的資料，產生的批次檢視 (例如彙總、特定的交叉分析篩選器等) 有利於快速查詢。
    * **Lambda 架構重新架構 - 服務層的批次**筆記本 [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20to%20Serving%20Layer.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Lambda%20Architecture%20Re-architected%20-%20Batch%20to%20Serving%20Layer.html) 會將批次資料推送至服務層；也就是說，Spark 會查詢並處理推文的批次集合，然後將它儲存到另一個集合 (計算的批次)。
* **速度層：****速度層**包括利用 Azure Cosmos DB 變更摘要讀取和立即處理的 Spark。 資料也可儲存至「計算的 RT」，以便其他系統可以查詢經過處理的即時資料，而不需要執行即時查詢本身。
    * [從 Cosmos DB 變更摘要串流查詢](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Streaming%20Query%20from%20Cosmos%20DB%20Change%20Feed.scala) (英文) scala 指令碼會執行 Azure Cosmos DB 變更摘要的串流查詢，計算從 Spark 殼層的間隔計數。
    * [從 Cosmos DB 變更摘要串流標籤查詢](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/Streaming%20Tags%20Query%20from%20Cosmos%20DB%20Change%20Feed%20.scala) (英文) scala 指令碼會執行 Azure Cosmos DB 變更摘要的串流查詢，計算從 Spark 殼層的標籤間隔計數。
  
## <a name="next-steps"></a>後續步驟
如果您還沒有從 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark) GitHub 存放庫下載「Spark 至 Azure Cosmos DB」連接器，請進行下載並探索存放庫中的其他資源：
* [Lambda 架構](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/lambda) (英文)
* [分散式彙總範例](https://github.com/Azure/azure-documentdb-spark/wiki/Aggregations-Examples) (英文)
* [指令碼和 Notebook 範例](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples) (英文)
* [結構化串流示範](https://github.com/Azure/azure-cosmosdb-spark/wiki/Structured-Stream-demos) (英文)
* [變更摘要示範](https://github.com/Azure/azure-cosmosdb-spark/wiki/Change-Feed-demos) (英文)
* [使用 Azure Cosmos DB 變更摘要和 Apache Spark 串流處理變更](https://github.com/Azure/azure-cosmosdb-spark/wiki/Stream-Processing-Changes-using-Azure-Cosmos-DB-Change-Feed-and-Apache-Spark) (英文)

您也可以檢閱 [Apache Spark SQL、DataFrame 和 Dataset 指南](http://spark.apache.org/docs/latest/sql-programming-guide.html) (英文) 和 [Azure HDInsight 上的 Apache Spark](../hdinsight/spark/apache-spark-jupyter-spark-sql.md) 文章。
