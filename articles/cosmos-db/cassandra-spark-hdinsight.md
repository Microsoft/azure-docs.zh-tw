---
title: 透過 HDInsight 從 Spark on YARN 存取 Azure Cosmos DB Cassandra API
description: 本文說明如何透過 HDInsight 從 Spark on YARN 使用 Azure Cosmos DB Cassandra API
services: cosmos-db
author: anagha-microsoft
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: 34293ffc70ebbc2a8639913aefe3b62ed8e618d4
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222772"
---
# <a name="access-azure-cosmos-db-cassandra-api-from-spark-on-yarn-with-hdinsight"></a>透過 HDInsight 從 Spark on YARN 存取 Azure Cosmos DB Cassandra API

本文說明如何透過 Spark-Shell 中的 HDInsight-Spark 從 Spark on YARN 存取 Azure Cosmos DB Cassandra API。 HDInsight 是 Microsoft 在 Azure 上的 Hortonworks Hadoop PaaS，可將物件儲存體用於 HDFS，且隨附數種選項，包括 [Spark](../hdinsight/spark/apache-spark-overview.md)。  雖然本文件的內容參考 HDInsight Spark，但仍適用於所有的 Hadoop 散發套件。  

## <a name="prerequisites"></a>必要條件

* [佈建 Azure Cosmos DB Cassandra API](create-cassandra-dotnet.md#create-a-database-account)

* [檢閱連線至 Azure Cosmos DB Cassandra API 的基本概念](cassandra-spark-generic.md)

* [佈建 HDInsight-Spark 叢集](../hdinsight/spark/apache-spark-jupyter-spark-sql.md)

* [檢閱使用 Cassandra API 的程式碼範例](cassandra-spark-generic.md#next-steps)

* [如果您願意，可以使用 cqlsh 進行驗證](cassandra-spark-generic.md##connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Spark2 中的 Cassandra API 組態** - 適用於 Cassandra 的 Spark 連接器需要將 Cassandra 連線詳細資料初始化以作為 Spark 內容的一部分。 當您啟動 Jupyter Notebook 時，Spark 工作階段和內容即已初始化，建議您不要停止並重新初始化 Spark 內容，除非初始化已完成，且每個組態皆設定為 HDInsight 預設 Jupyter Notebook 啟動程序的一部分。 有個解決方法是將 Cassandra 執行個體詳細資料直接新增至 Ambari (Spark2) 服務組態。 這是個別叢集的一次性活動，在完成後需要重新啟動 Spark2 服務。
 
  1. 移至 Ambari (Spark2) 服務，然後按一下 [設定]

  2. 接著，移至自訂 spark2-defaults 並以下列方式新增屬性，然後重新啟動 Spark2 服務：

  ```scala
  spark.cassandra.connection.host=YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com<br>
  spark.cassandra.connection.port=10350<br>
  spark.cassandra.connection.ssl.enabled=true<br>
  spark.cassandra.auth.username=YOUR_COSMOSDB_ACCOUNT_NAME<br>
  spark.cassandra.auth.password=YOUR_COSMOSDB_KEY<br>
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-spark-shell"></a>從 Spark Shell 存取 Azure Cosmos DB Cassandra API

Spark Shell 可供測試/探索之用。

* 使用與叢集的 Spark 版本相容的必要 Maven 相依性啟動 Spark-Shell。

  ```scala
  spark-shell --packages "com.datastax.spark:spark-cassandra-connector_2.11:2.3.0,com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0"
  ```

* 執行某些 DDL 和 DML 作業

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}

  import spark.implicits._
  import org.apache.spark.sql.functions._
  import org.apache.spark.sql.Column
  import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType,LongType,FloatType,DoubleType, TimestampType}
  import org.apache.spark.sql.cassandra._

  //Spark connector
  import com.datastax.spark.connector._
  import com.datastax.spark.connector.cql.CassandraConnector

  //CosmosDB library for multiple retry
  import com.microsoft.azure.cosmosdb.cassandra

  // Specify connection factory for Cassandra
  spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

  // Parallelism and throughput configs
  spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
  spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
  spark.conf.set("spark.cassandra.output.concurrent.writes", "100")
  spark.conf.set("spark.cassandra.concurrent.reads", "512")
  spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
  spark.conf.set("spark.cassandra.connection.keep_alive_ms", "60000000") //Increase this number as needed
  ```

* 執行 CRUD 作業

  ```scala
  //1) Create table if it does not exist
  val cdbConnector = CassandraConnector(sc)
  cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000;"))

  //2) Delete data from potential prior runs
  cdbConnector.withSessionDo(session => session.execute("DELETE FROM books_ks.books WHERE book_id IN ('b00300','b00001','b00023','b00501','b09999','b01001','b00999','b03999','b02999','b000009');"))

  //3) Generate a few rows
  val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887,11.33),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890,22.45),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892,19.83),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893,14.22),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901,12.25)
  ).toDF("book_id", "book_author", "book_name", "book_pub_year","book_price")

  //4) Persist
  booksDF.write.mode("append").format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000")).save()

  //5) Read the data in the table
  spark.read.format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks")).load.show
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-jupyter-notebooks"></a>從 Jupyter Notebook 存取 Azure Cosmos DB Cassandra API

HDInsight-Spark 隨附 Zeppelin 和 Jupyter Notebook 服務。 兩者都是以 Web 為基礎的 Notebook 環境，且都支援 Scala 和 Python。 Notebook 非常適用於互動式探勘分析與共同作業，但不適用於作業/生產化程序。

下列 Jupyter Notebook 可上傳至您的 HDInsight Spark 叢集，並提供可與 Azure Cosmos DB Cassandra API 搭配使用的現成範例。 請務必檢閱第一個 Notebook `1.0-ReadMe.ipynb`，以檢閱連線至 Azure Cosmos DB Cassandra API 的 Spark 服務組態。

請 [azure-cosmos-db-cassandra-api-spark-notebooks-jupyter](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-jupyter/blob/master/scala/) 下方的這些 Notebook 下載到您的電腦。
  
### <a name="how-to-upload"></a>如何上傳：
當您啟動 Jupyter 時，請瀏覽至 Scala。 先建立目錄，然後將 Notebook 上傳至目錄。 上傳按鈕位於右上方。  

### <a name="how-to-run"></a>如何執行：
循序執行Notebook 和每個 Notebook 資料格。  按一下每個 Notebook 頂端的執行按鈕以執行所有資料格，或按 shift+enter 以執行個別資料格。

## <a name="access-with-azure-cosmos-db-cassandra-api-from-your-spark-scala-program"></a>從 Spark Scala 程式存取 Azure Cosmos DB Cassandra API

在生產環境中執行自動化程序時，會透過 [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html) 將 Spark 程式提交至叢集。

## <a name="next-steps"></a>後續步驟

* [如何在 IDE 中建立 Spark Scala 程式，並透過 Livy 將其提交至 HDInsight Spark 叢集執行](../hdinsight/spark/apache-spark-create-standalone-application.md)

* [如何從 Spark Scala 程式連線到 Azure Cosmos DB Cassandra API](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/master/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)

* [使用 Cassandra API 的程式碼範例完整清單](cassandra-spark-generic.md)
