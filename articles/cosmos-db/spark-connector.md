---
title: 將 Apache Spark 連線到 Azure Cosmos DB | Microsoft Docs
description: 了解可讓您將 Apache Spark 連線到 Azure Cosmos DB 的 Azure Cosmos DB Spark 連接器。 您可以在 Microsoft 的多租用戶、全域分散式資料庫系統上執行分散式彙總。
keywords: Apache Spark
services: cosmos-db
author: tknandu
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: ramkris
ms.openlocfilehash: e7ed0049e64a7740063f2fab7bdfddff38d45ed9
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287705"
---
# <a name="accelerate-big-data-analytics-by-using-the-apache-spark-to-azure-cosmos-db-connector"></a>使用 Apache Spark 至 Azure Cosmos DB 連接器來加速巨量資料分析
 
Apache Spark 至 Azure Cosmos DB 連接器可讓 Azure Cosmos DB 成為 Apache Spark 作業的輸入或輸出。 將 [Spark](http://spark.apache.org/) 連線到 [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 可讓您加速解決瞬息萬變的資料科學問題。 您可以使用 Azure Cosmos DB 來快速保存及查詢資料。 此連接器可有效地使用原生 Azure Cosmos DB 受控索引。 該索引可在針對快速變更的全域分散式資料執行分析和向下推展的述詞篩選時，更新資料行。 這種資料的範圍涵蓋物聯網 (IoT)、資料科學及分析案例。

## <a name="connector-components"></a>連接器元件

Spark 至 Azure Cosmos DB 連接器具有下列元件：

* [Azure Cosmos DB](http://documentdb.com) 可讓您在任意數目的全體地理區域，佈建及彈性地調整輸送量和儲存空間。  

* [Apache Spark](http://spark.apache.org/) 是功能強大的開放原始碼處理引擎，專為速度、易用性和精密分析所打造。  

* [Azure Databricks 上的 Apache Spark 叢集](https://docs.azuredatabricks.net/getting-started/index.html)可讓您在 Spark 叢集上執行 Spark 作業。

## <a name="connect-apache-spark-to-azure-cosmos-db"></a>將 Apache Spark 連線至 Azure Cosmos DB

有兩種方法可以連接 Apache Spark 與 Azure Cosmos DB：

- [Azure Cosmos DB SQL Python SDK](https://github.com/Azure/azure-documentdb-python) 是 Python 型連接器，也稱為 pyDocumentDB。  

- [Azure Cosmos DB SQL Java SDK](https://github.com/Azure/azure-documentdb-java) 是 Java 型連接器。


**支援的版本**

| 元件 | 版本 |
|---------|-------|
|Apache Spark| 2.1.x、2.2.x、2.3.x |
| Scala|2.11|
| Azure Databricks 執行階段版本 | > 3.4 |
| Azure Cosmos DB SQL Java SDK | 1.16.2 |

## <a name="connect-by-using-python-or-pydocumentdb-sdk"></a>使用 Python 或 pyDocumentDB SDK 連線

下圖顯示 pyDocumentDB SDK 實作的架構：

![透過 pyDocumentDB DB 的 Spark 至 Azure Cosmos DB 資料流程圖](./media/spark-connector/spark-pydocumentdb.png)


### <a name="data-flow"></a>資料流

PyDocumentDB 實作的資料流程如下所示：

* Spark 的主要節點會透過 pyDocumentDB 連線至 Azure Cosmos DB 閘道節點。 您會指定僅限 Spark 和 Azure Cosmos DB 連線。 對於個別主要和閘道節點的連線皆為透明的。  

* 閘道節點針對 Azure Cosmos DB 執行查詢，其中該查詢後續會針對資料節點中集合的分割區執行。 這些查詢的回應會傳回給閘道節點，並將該結果集傳回給 Spark 主要節點。  

* 後續查詢 (例如，針對 Spark 資料框架) 皆會傳送到 Spark 背景工作角色節點進行處理。  

Spark 與 Azure Cosmos DB 之間的通訊，僅限在 Spark 主要節點和 Azure Cosmos DB 閘道節點才能進行。 由於系統允許這兩個節點之間的傳輸層，因此查詢速度很快。

執行下列步驟，使用 pyDocumentDB SDK 來將 Spark 連線至 Azure Cosmos DB：

1. 建立 [Azure Databricks 工作區](../azure-databricks/quickstart-create-databricks-workspace-portal.md#create-an-azure-databricks-workspace)和 [Spark 叢集](../azure-databricks/quickstart-create-databricks-workspace-portal.md#create-a-spark-cluster-in-databricks)。 Azure Databricks 執行階段 4.0 版包含該工作區內的 Apache Spark 2.3.0 和 Scala 2.11。  

2. 當叢集已建立且正在執行時，請移至 [工作區] > [建立] > [程式庫]。  
3. 從 [新的程式庫] 對話方塊中，選擇 [上傳 Python Egg 或 PyPi] 作為來源。 提供 **pydocumentdb** 作為名稱，然後選取 [安裝程式庫]。 pyDocumentdb SDK 已經發佈到 pip 套件，因此您可以找到並安裝它。 

   ![[新增程式庫] 對話方塊的螢幕擷取畫面](./media/spark-connector/create-library.png)

4. 安裝程式庫之後，將它連結至您稍早建立的叢集。  

5. 移至 [工作區] > [建立] > [Notebook]。  

6. 在 [建立 Notebook] 對話方塊中輸入易記的名稱，然後選擇 [Python] 作為程式設計語言。 從下拉式清單中選取您稍早建立的叢集，然後選取 [建立]。  

7. 使用裝載於 “doctorwho” Azure Cosmos DB 帳戶中的航班範例資料來執行少數的 Spark 查詢。 (這個帳戶可公開存取。)[azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark/tree/master) GitHub 存放庫會裝載 HTML 版的 Notebook。 下載存放庫檔案並移至 `\samples\Documentation_Samples\Read_Batch_PyDocumentDB.html`。 您可以將 Notebook 匯入至 Azure Databricks 帳戶並加以執行。 下一節將詳細說明程式碼區塊的功能。

下列程式碼片段示範如何匯入 pyDocumentDB SDK，並在 Spark 內容中執行查詢。 如程式碼片段所述，pyDocumentDB SDK 包含連線至 Azure Cosmos DB 帳戶所需的連線參數。 它會匯入必要的程式庫，並設定主要金鑰和主機，以建立 Azure Cosmos DB 用戶端 (pydocumentdb.document_client)。


```python
# Import Necessary Libraries
import pydocumentdb
from pydocumentdb import document_client
from pydocumentdb import documents
import datetime

# Configuring the connection policy (allowing for endpoint discovery)
connectionPolicy = documents.ConnectionPolicy()
connectionPolicy.EnableEndpointDiscovery
connectionPolicy.PreferredLocations = ["Central US", "East US 2", "Southeast Asia", "Western Europe","Canada Central"]

# Set keys to connect to Azure Cosmos DB
masterKey = 'le1n99i1w5l7uvokJs3RT5ZAH8dc3ql7lx2CG0h0kK4lVWPkQnwpRLyAN0nwS1z4Cyd1lJgvGUfMWR3v8vkXKA=='
host = 'https://doctorwho.documents.azure.com:443/'
client = document_client.DocumentClient(host, {'masterKey': masterKey}, connectionPolicy)

```

接下來，您可以執行查詢。 下列程式碼片段會連線至 doctorwho 帳戶中的 airports.codes 集合，並執行查詢以擷取華盛頓州的機場城市。 

```python
# Configure Database and Collections
databaseId = 'airports'
collectionId = 'codes'

# Configurations the Azure Cosmos DB client will use to connect to the database and collection
dbLink = 'dbs/' + databaseId
collLink = dbLink + '/colls/' + collectionId

# Set query parameter
querystr = "SELECT c.City FROM c WHERE c.State='WA'"

```

執行查詢之後，結果是已轉換成 Python 清單的 “query_iterable.QueryIterable”。 這份清單會接著轉換成 Spark 資料框架。 

```python
# Query documents
query = client.QueryDocuments(collLink, querystr, options=None, partition_key=None)

# Query for partitioned collections
# query = client.QueryDocuments(collLink, query, options= { 'enableCrossPartitionQuery': True }, partition_key=None)

# Create `df` Spark DataFrame from `elements` Python list
df = spark.createDataFrame(list(query))

# Show data
df.show()
```

### <a name="considerations-when-using-pydocumentdb-sdk"></a>使用 pyDocumentDB SDK 時的考量
在下列案例中建議使用 pyDocumentDB SDK 來將 Spark 連線至 Azure Cosmos DB：

* 您想要使用 Python。  

* 您要將相當小的結果集從 Azure Cosmos DB 傳回 Spark。 請注意，Azure Cosmos DB 中的基礎資料集可能相當大，而您正在針對 Azure Cosmos DB 來源套用篩選條件或執行述詞篩選條件。

## <a name="connect-by-using-the-java-sdk"></a>使用 Java SDK 連線

下圖顯示 Azure Cosmos DB SQL Java SDK 實作的架構，並在 Spark 背景工作節點之間移動資料：

![Spark 至 Azure Cosmos DB 連接器中的資料流程圖](./media/spark-connector/spark-connector.png)

### <a name="data-flow"></a>資料流

Java SDK 實作的資料流程如下所示：

* 從 Spark 主要節點連線到 Azure Cosmos DB 閘道節點，以取得分割區對應。 您會指定僅限 Spark 和 Azure Cosmos DB 連線。 對於個別主要和閘道節點的連線皆為透明的。  

* 此資訊會提供回 Spark 主要節點。 此時，您應該可以剖析查詢，來判斷您需要存取 Azure Cosmos DB 中的哪些分割區和其位置。  

* 此資訊會傳輸到 Spark 背景工作角色節點。  

* Spark 背景工作角色節點會直接連線至 Azure Cosmos DB 分割區以擷取資料，並將資料傳回背景工作角色節點中的 Spark 分割區。  

Spark 與 Azure Cosmos DB 之間的通訊速度大幅提升，這是因為資料移動是在 Spark 背景工作角色節點與 Azure Cosmos DB 資料節點 (分割區) 之間進行。 在這個範例中，您會將一些範例 Twitter 資料傳送至 Azure Cosmos DB 帳戶，並使用該資料來執行 Spark 查詢。 使用下列步驟，將範例 Twitter 資料寫入至 Azure Cosmos DB：

1. 建立 [Azure Cosmos DB SQL API 帳戶](create-sql-api-dotnet.md#create-a-database-account)，並在帳戶中[新增集合](create-sql-api-dotnet.md#add-a-collection)。  

2. 從 GitHub 下載 [TwitterCosmosDBFeed](https://github.com/tknandu/TwitterCosmosDBFeed) 範例。 您可使用此摘要將範例 Twitter 資料寫入至 Azure Cosmos DB。  

3. 開啟命令提示字元，然後執行下列命令來安裝 Tweepy 和 pyDocumentdb 模組：

   ```bash
   pip install tweepy==3.3.0
   pip install pyDocumentDB
   ```

4. 將 Twitter 摘要範例的內容解壓縮，然後開啟 config.py 檔案。 更新 masterKey、host、databaseId、collectionId 和 preferredLocations 值。  

5. 移至 `http://apps.twitter.com/`，並註冊 Twitter 摘要指令碼作為新的應用程式。 選擇適用於您應用程式的名稱和應用程式之後，您將會得到**取用者金鑰、取用者祕密、存取權杖和存取權杖祕密**。 複製這些值並在 config.py 檔案中更新它們，以程式設計方式提供對 Twitter 的應用程式存取權。   

6. 儲存 config.py 檔案。 開啟命令提示字元，然後使用下列命令來執行 Python 應用程式：

   ```bash
   Python driver.py
   ```

7. 移至入口網站中的 Azure Cosmos DB 集合，然後確認 Twitter 資料會寫入至集合。

### <a name="find-and-attach-the-java-sdk-to-the-spark-cluster"></a>尋找 Java SDK 並連結至 Spark 叢集

1. 建立 [Azure Databricks 工作區](../azure-databricks/quickstart-create-databricks-workspace-portal.md#create-an-azure-databricks-workspace)和 [Spark 叢集](../azure-databricks/quickstart-create-databricks-workspace-portal.md#create-a-spark-cluster-in-databricks)。 Azure Databricks 執行階段 4.0 版包含該工作區內的 Apache Spark 2.3.0 和 Scala 2.11。  

2. 當叢集已建立且正在執行時，請移至 [工作區] > [建立] > [程式庫]。  

3. 從 [新增程式庫] 對話方塊中，選擇 [Maven 座標] 作為來源。 提供座標值 **com.microsoft.azure:azure-cosmosdb-spark_2.3.0_2.11:1.2.0**，然後選取 [建立程式庫]。 Maven 相依性已經解析，並將套件新增至您的工作區。 在前述的 Maven 座標格式中，2.3.0 代表 Spark 版本、2.11 代表 Scala 版本，而 1.2.0 代表 Azure Cosmos DB 連接器版本。 

4. 安裝程式庫之後，將它連結至您稍早建立的叢集。 

本文示範在下列案例中 Spark 連接器 Java SDK 的使用方式：

* 從 Azure Cosmos DB 讀取 Twitter 資料。  

* 讀取正在串流處理至 Azure Cosmos DB 的 Twitter 資料。  

* 將 Twitter 資料寫入至 Azure Cosmos DB。 

### <a name="read-twitter-data-from-azure-cosmos-db"></a>從 Azure Cosmos DB 讀取 Twitter 資料
 
在本節中，您會執行 Spark 查詢，從 Azure Cosmos DB 讀取 Twitter 資料的批次。 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark/tree/master) GitHub 存放庫會裝載 HTML 版的 Notebook。 下載存放庫檔案並移至 `\samples\Documentation_Samples\Read_Batch_Twitter_Data.html`。 您可以將 Notebook 匯入至 Azure Databricks 帳戶，以及更新帳戶 URI、主要金鑰、資料庫和集合名稱。 您可以執行 Notebook，或如下所示建立它：

1. 移至您的 Azure Databricks 帳戶，然後選取 [工作區] > [建立] > [Notebook]。 

2. 在 [建立 Notebook] 對話方塊中輸入易記的名稱，然後選擇 [Python] 作為程式設計語言。 從下拉式清單中選取您稍早建立的叢集，然後選取 [建立]。  

3. 更新端點、主要金鑰、資料庫和集合值以連線至帳戶。 使用 spark.read.format() 命令讀取推文。

   ```python
   # Configuration Map
   tweetsConfig = {
   "Endpoint" : "<Your Azure Cosmos DB endpoint>",
   "Masterkey" : "<Primary key of your Azure Cosmos DB account>",
   "Database" : "<Your Azure Cosmos DB database name>",
   "Collection" : "<Your Azure Cosmos DB collection name>", 
   "preferredRegions" : "East US",
   "SamplingRatio" : "1.0",
   "schema_samplesize" : "200000",
   "query_custom" : "SELECT c.id, c.created_at, c.user.screen_name, c.user.lang, c.user.location, c.text, c.retweet_count, c.entities.hashtags, c.entities.user_mentions, c.favorited, c.source FROM c"
   }
   # Read Tweets
   tweets = spark.read.format("com.microsoft.azure.cosmosdb.spark").options(**tweetsConfig).load()
   tweets.createOrReplaceTempView("tweets")
   #tweets.cache()

   ```

4. 執行查詢，從快取的資料中依不同的主題標籤取得推文計數。 

   ```python
   %sql
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

Java SDK 支援下列值來進行設定對應： 

|設定  |說明  |
|---------|---------|
|query_maxdegreeofparallelism  | 設定平行查詢執行期間，在用戶端執行的並行作業數目。 如果將它設為大於 0 的值，它會將並行作業數目限制為指派的值。 如果將它設為小於 0，系統就會自動決定要執行的並行操作次數。 當連接器將每個集合分割區與執行程式相對應時，此值不會對讀取作業產生任何影響。        |
|query_maxbuffereditemcount     |    設定平行查詢執行期間，可在用戶端緩衝處理的項目數上限。 如果將它設為大於 0 的值，它會將緩衝處理的項目數限制為指派的值。 如果將它設為小於 0，系統就會自動決定要進行緩衝處理的項目數。     |
|query_enablescan    |   設定此選項，可在因為已在要求的路徑選擇退出編製索引而無法提供服務的查詢上啟用掃描。       |
|query_disableruperminuteusage  |  如果已用完每秒的一般佈建要求單位 (RU)，即會停用每分的 RU 容量來提供查詢。       |
|query_emitverbosetraces   |   設定選項，以允許查詢發出詳細資訊追蹤來進行調查。      |
|query_pagesize  |   針對每個查詢要求設定查詢結果頁面的大小。 若要針對輸送量進行最佳化，請使用大型分頁大小來減少擷取結果的往返次數。      |
|query_custom  |  從 Azure Cosmos DB 擷取資料時，設定 Azure Cosmos DB 查詢來覆寫預設查詢。 請注意，當您提供此值時，它也會用來取代具有下推述詞的查詢。     |

根據案例而定，您應使用不同的組態值來將效能和輸送量最佳化。 請注意，設定金鑰目前不區分大小寫，而且設定值一律為字串。

### <a name="read-twitter-data-that-is-streaming-to-azure-cosmos-db"></a>讀取正在串流處理至 Azure Cosmos DB 的 Twitter 資料

在本節中，您會執行 Spark 查詢，以讀取串流處理 Twitter 資料的變更摘要。 當您執行本節中的查詢時，請確定您的 Twitter 摘要應用程式正在執行且會將資料提取到 Azure Cosmos DB。 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark/tree/master) GitHub 存放庫會裝載 HTML 版的 Notebook。 下載存放庫檔案並移至 `\samples\Documentation_Samples\Read_Stream_Twitter_Data.html`。 您可以將 Notebook 匯入至 Azure Databricks 帳戶，以及更新帳戶 URI、主要金鑰、資料庫和集合名稱。 您可以執行 Notebook，或如下所示建立它：

1. 移至您的 Azure Databricks 帳戶，然後選取 [工作區] > [建立] > [Notebook]。  

2. 在 [建立 Notebook] 對話方塊中輸入易記的名稱，然後選擇 [Scala] 作為程式設計語言。 從下拉式清單中選取您稍早建立的叢集，然後選取 [建立]。  

3. 更新端點、主要金鑰、資料庫和集合值以連線至帳戶。

   ```scala
   // This script does the following:
   // - creates a structured stream from a Twitter feed CosmosDB collection (on top of change feed)
   // - get the count of tweets
   import com.microsoft.azure.cosmosdb.spark._
   import com.microsoft.azure.cosmosdb.spark.schema._
   import com.microsoft.azure.cosmosdb.spark.config.Config
   import org.codehaus.jackson.map.ObjectMapper
   import com.microsoft.azure.cosmosdb.spark.streaming._
   import java.time._

   val sourceConfigMap = Map(
   "Endpoint" -> "<Your Azure Cosmos DB endpoint>",
   "Masterkey" -> "<Primary key of your Azure Cosmos DB account>",
   "Database" -> "<Your Azure Cosmos DB database name>",
   "Collection" -> "<Your Azure Cosmos DB collection name>", 
   "ConnectionMode" -> "Gateway",
   "ChangeFeedCheckpointLocation" -> "/tmp",
   "changefeedqueryname" -> "Streaming Query from Cosmos DB Change Feed Internal Count")
   ```
4. 使用 spark.readStream.format() 命令，開始讀取變更摘要作為串流：

   ```scala
   var streamData = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(sourceConfigMap).load()
   ```
5. 開始將查詢串流處理至您的主控台：

   ```scala
   //**RUN THE ABOVE FIRST AND KEEP BELOW IN SEPARATE CELL
   val query = streamData.withColumn("countcol", streamData.col("id").substr(0,0)).groupBy("countcol").count().writeStream.outputMode("complete").format("console").start()
   ```

Java SDK 支援下列值來進行設定對應：

|設定  |說明  |
|---------|---------|
|readchangefeed   |  表示集合內容擷取自 CosmosDB 變更摘要。 預設值為 False。       |
|changefeedqueryname |   識別查詢的自訂字串。 連接器會分別針對不同的變更摘要查詢持續追蹤集合接續 Token。 如果 readchangefeed 為 True，這就是不得接受空白值的必要組態。      |
|changefeedcheckpointlocation  |   可在發生節點失敗時保存接續 Token 的本機檔案儲存體路徑。      |
|changefeedstartfromthebeginning  |  設定是否應該從一開始 (True) 或從目前所在點 (False) 啟動變更摘要。 根據預設，它會從目前所在點 (False) 啟動。       |
|rollingchangefeed  |   布林值，表示變更摘要是否應該來自最後一個查詢。 預設值為 False，表示變更將從集合的第一次讀取開始計數。      |
|changefeedusenexttoken  |   支援處理失敗案例的布林值。 表示已正常處理目前的變更摘要批次。 復原分散式資料集應使用下一個接續 Token 來取得後續的變更批次。      |
| InferStreamSchema | 布林值，表示是否應該在串流處理一開始就，針對串流處理資料的結構描述進行取樣。 此值預設會設定為 true。 如果此參數設為 True，而且串流處理資料的結構描述會在資料取樣之後變更，則將在串流處理的資料框架中捨棄新加入的屬性。 <br/><br/> 如果您想要使串流處理的資料框架成為無從驗證的結構描述，請將此參數設為 False。 在此模式中，從 Azure Cosmos DB 變更摘要讀取的文件主體都會包裝到 body 屬性中。 這個屬性位於結果串流處理資料框架中 (系統屬性值除外)。
 |

### <a name="connection-settings"></a>連線設定

Java SDK 支援下列連線設定：

|設定  |說明  |
|---------|---------|
|connectionmode   |  設定內部 DocumentClient 應該用來與 Azure Cosmos DB 通訊的連線模式。 允許的值為 **DirectHttps** (預設值) 和 **Gateway**。 DirectHttps 連線模式會將要求直接路由傳送至 CosmosDB 分割區，並提供某些延遲優點。       |
|connectionmaxpoolsize   |  設定內部 DocumentClient 所使用的連線集區大小值。 預設值是 100。       |
|connectionidletimeout  |  設定閒置連線的逾時值 (秒)。 預設值為 60。       |
|query_maxretryattemptsonthrottledrequests    |  設定重試次數上限。 您會在因為用戶端上的速率限制而導致要求失敗時使用此值。 若未指定，預設值為 1000 次重試嘗試。       |
|query_maxretrywaittimeinseconds   |  設定最大重試時間 (秒)。 此值預設為 1000 秒。       |

### <a name="write-twitter-data-to-azure-cosmos-db"></a>將 Twitter 資料寫入至 Azure Cosmos DB 

在本節中，您可以執行 Spark 查詢，以將 Twitter 資料批次寫入至相同資料庫中的新集合。 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark/tree/master) GitHub 存放庫會裝載 HTML 版的 Notebook。 下載存放庫檔案並移至 `\samples\Documentation_Samples\Write_Batch_Twitter_Data.html`。 您可以將 Notebook 匯入至 Azure Databricks 帳戶，以及更新帳戶 URI、主要金鑰、資料庫和集合名稱。 您可以執行 Notebook，或如下所示建立它：

1. 移至您的 Azure Databricks 帳戶，然後選取 [工作區] > [建立] > [Notebook]。  

2. 在 [建立 Notebook] 對話方塊中輸入易記的名稱，然後選擇 [Scala] 作為程式設計語言。 從下拉式清單中選取您稍早建立的叢集，然後選取 [建立]。  

3. 更新端點、主要金鑰、資料庫和集合值以連線至帳戶，來讀取和寫入 Twitter 資料。

   ```scala
   %scala
   // Import Necessary Libraries
   import org.joda.time._
   import org.joda.time.format._
   import com.microsoft.azure.cosmosdb.spark.schema._
   import com.microsoft.azure.cosmosdb.spark._
   import com.microsoft.azure.cosmosdb.spark.config.Config

   // Maps
   val readConfigMap = Map(
   "Endpoint" -> "<Your Azure Cosmos DB endpoint>",
   "Masterkey" -> "<Primary key of your Azure Cosmos DB account>",
   "Database" -> "<Your Azure Cosmos DB database name>",
   "Collection" -> "<Your Azure Cosmos DB source collection name>", 
   "preferredRegions" -> "East US",
   "SamplingRatio" -> "1.0",
   "schema_samplesize" -> "200000",
   "query_custom" -> "SELECT c.id, c.created_at, c.user.screen_name, c.user.location, c.text, c.retweet_count, c.entities.hashtags, c.entities.user_mentions, c.favorited, c.source FROM c"
   )
   val writeConfigMap = Map(
   "Endpoint" -> "<Your Azure Cosmos DB endpoint>",
   "Masterkey" -> "<Primary key of your Azure Cosmos DB account>",
   "Database" -> "<Your Azure Cosmos DB database name>",
   "Collection" -> "<Your Azure Cosmos DB destination collection name>", 
   "preferredRegions" -> "East US",
   "SamplingRatio" -> "1.0",
   "schema_samplesize" -> "200000"
   ) 

   // Configs
   // get read
   val readConfig = Config(readConfigMap)
   val tweets = spark.read.cosmosDB(readConfig)
   tweets.createOrReplaceTempView("tweets")
   tweets.cache()

   // get write
   val writeConfig = Config(writeConfigMap)
   ```
4. 執行查詢，從快取的資料中依不同的主題標籤取得推文計數。 

   ```scala
   %sql
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

5. 建立推文標記的新資料框架，並將資料儲存至新的集合。 執行下列程式碼之後，您可以返回入口網站，並確認資料會寫入至集合。 

   ```scala
   %scala
   // Import SaveMode so you can Overwrite, Append, ErrorIfExists, Ignore
   import org.apache.spark.sql.{Row, SaveMode, SparkSession}

   // Create new DataFrame of tweets tags
   val tweets_bytags = spark.sql("select '2018-06-12' as currdt, hashtags.text as hashtags, count(distinct id) as tweets from ( select explode(hashtags) as hashtags, id from tweets ) a group by hashtags.text order by tweets desc limit 10")

   // Save to Cosmos DB (using Append in this case)
   // Ensure the baseConfig contains a Read-Write Key
   // The key provided in our examples is a Read-Only Key

   tweets_bytags.write.mode(SaveMode.Overwrite).cosmosDB(writeConfig)
   ```

Java SDK 支援下列值來進行設定對應：

|設定  |說明  |
|---------|---------|
| BulkImport | 布林值，表示是否應該使用 BulkExecutor 程式庫來匯入資料。 此值預設會設定為 true。 |
|WritingBatchSize  |   表示當您將資料寫入至 Azure Cosmos DB 集合時要使用的批次大小。 <br/><br/> 如果 BulkImport 參數設為 True，則 WritingBatchSize 參數表示提供來作為 BulkExecutor 程式庫之 importAll API 輸入的文件批次大小。 根據預設，此值會設為 100K。 <br/><br/> 如果 BulkImport 參數設為 False，則 WritingBatchSize 參數表示當您寫入至 Azure Cosmos DB 集合時要使用的批次大小。 連接器會在批次中以非同步方式傳送 createDocument 和 upsertDocument 要求。 只要叢集資源可供使用，批次大小越大，您可達到的輸送量就越多。 另一方面，請指定較小數字的批次大小來限制速率及 RU 耗用量。 根據預設，寫入批次大小會設為 500。  |
|Upsert   |  布林值字串，表示當您寫入至 Azure CosmosDB 集合時，是否要使用 upsertDocument，而不是使用 CreateDocument。   |
| WriteThroughputBudget |  整數字串，表示除了配置給集合的總輸送量，您想要配置給大量擷取 Spak 作業的每秒 RU 數目。 |


### <a name="write-twitter-data-that-is-streaming-to-azure-cosmos-db"></a>寫入正在串流處理至 Azure Cosmos DB 的 Twitter 資料 

在本節中，您可以執行 Spark 查詢，以將串流處理 Twitter 資料的變更摘要寫入至相同資料庫中的新集合。 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark/tree/master) GitHub 存放庫會裝載 HTML 版的 Notebook。 下載存放庫檔案並移至 `\samples\Documentation_Samples\Write_Stream_Twitter_Data.html`。 您可以將 Notebook 匯入至 Azure Databricks 帳戶，以及更新帳戶 URI、主要金鑰、資料庫和集合名稱。 您可以執行 Notebook，或如下所示建立它：

1. 移至您的 Azure Databricks 帳戶，然後選取 [工作區] > [建立] > [Notebook]。  

2. 在 [建立 Notebook] 對話方塊中輸入易記的名稱，然後選擇 [Scala] 作為程式設計語言。 從下拉式清單中選取您稍早建立的叢集，然後選取 [建立]。  

3. 更新端點、主要金鑰、資料庫和集合值以連線至帳戶，來讀取和寫入 Twitter 資料。

   ```scala
   import com.microsoft.azure.cosmosdb.spark._
   import com.microsoft.azure.cosmosdb.spark.schema._
   import com.microsoft.azure.cosmosdb.spark.config.Config
   import com.microsoft.azure.cosmosdb.spark.streaming._

   // Configure connection to Azure Cosmos DB Change Feed (Trades)
   val ConfigMap = Map(
   // Account settings
   "Endpoint" -> "<Your Azure Cosmos DB endpoint>",
   "Masterkey" -> "<Primary key of your Azure Cosmos DB account>",
   "Database" -> "<Your Azure Cosmos DB database name>",
   "Collection" -> "<Your Azure Cosmos DB source collection name>", 
   // Change feed settings
   "ReadChangeFeed" -> "true",
   "ChangeFeedStartFromTheBeginning" -> "true",
   "ChangeFeedCheckpointLocation" -> "dbfs:/cosmos-feed",
   "ChangeFeedQueryName" -> "Structured Stream Read",
   "InferStreamSchema" -> "true"
   )
   ```
4. 使用 spark.readStream.format() 命令，開始讀取變更摘要作為串流：
 
   ```scala
   // Start reading change feed of trades as a stream
   var streamdata = spark
     .readStream
     .format(classOf[CosmosDBSourceProvider].getName)
     .options(ConfigMap)
     .load()
   ```

5. 定義目的地集合的組態，並使用 writeStream.format() 方法來開始串流處理作業：

   ```scala
   val sinkConfigMap = Map(
   "Endpoint" -> "<Your Azure Cosmos DB endpoint>",
   "Masterkey" -> "<Primary key of your Azure Cosmos DB account>",
   "Database" -> "<Your Azure Cosmos DB database name>",
   "Collection" -> "<Your Azure Cosmos DB destination collection name>", 
   "checkpointLocation" -> "streamingcheckpointlocation6",
   "WritingBatchSize" -> "100",
   "Upsert" -> "true")

   // Start the stream writer
   val streamingQueryWriter = streamdata
    .writeStream
    .format(classOf[CosmosDBSinkProvider].getName)
    .outputMode("append")
    .options(sinkConfigMap)
    .start()
    ```

Java SDK 支援下列值來進行設定對應：

|設定  |說明  |
|---------|---------|
|Upsert   |  布林值字串，表示當您寫入至 Azure CosmosDB 集合時，是否要使用 upsertDocument，而不是使用 CreateDocument。   |
|checkpointlocation  |   可在發生節點失敗時保存接續 Token 的本機檔案儲存體路徑。   |
|WritingBatchSize  |  表示將資料寫入至 Azure Cosmos DB 集合時要使用的批次大小。 連接器會在批次中以非同步方式傳送 createDocument 和 upsertDocument 要求。 只要叢集資源可供使用，批次大小越大，您可達到的輸送量就越多。 另一方面，請指定較小數字的批次大小來限制速率及 RU 耗用量。 根據預設，寫入批次大小會設為 500。  |


### <a name="considerations-when-using-java-sdk"></a>使用 Java SDK 時的考量

在下列案例中建議使用 Java SDK 來將 Spark 連線至 Azure Cosmos DB：

* 您想要使用 Python 或 Scala。  

* 您有大量的資料要在 Apache Spark 與 Azure Cosmos DB 之間傳輸。 Java SDK 的效能優於 pyDocumentDB。 如需查詢效能差異的詳細資訊，請參閱[查詢測試回合 Wiki](https://github.com/Azure/azure-cosmosdb-spark/wiki/Query-Test-Runs) \(英文\)。

## <a name="next-steps"></a>後續步驟

如果您還沒這麼做，請從 [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark) GitHub 存放庫下載 Spark 至 Azure Cosmos DB 連接器。 探索存放庫中的下列額外資源：

* [彙總範例](https://github.com/Azure/azure-cosmosdb-spark/wiki/Aggregations-Examples)
* [指令碼和 Notebook 範例](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples) (英文)

您也可以檢閱 [Apache Spark SQL、DataFrame 和 Dataset 指南](http://spark.apache.org/docs/latest/sql-programming-guide.html) (英文) 和 [Azure HDInsight 上的 Apache Spark](../hdinsight/spark/apache-spark-jupyter-spark-sql.md) 文章。
