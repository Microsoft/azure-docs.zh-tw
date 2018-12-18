---
title: 從 Spark 對 Azure Cosmos DB Cassandra API 資料表進行彙總作業
description: 本文涵蓋從 Spark 對 Azure Cosmos DB Cassandra API 資料表進行的基本彙總作業
services: cosmos-db
author: anagha-microsoft
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.custom: basics, DDL, DML
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: 385a365ac3b81bca70a71eeed7ca1876c9df49b8
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224988"
---
# <a name="aggregate-operations-on-azure-cosmos-db-cassandra-api-tables-from-spark"></a>從 Spark 對 Azure Cosmos DB Cassandra API 資料表進行彙總作業 

本文說明從 Spark 對 Azure Cosmos DB Cassandra API 資料表進行的基本彙總作業。 

> [!NOTE]
> Azure Cosmos DB Cassandra API 中目前不支援伺服器端篩選和伺服器端彙總。

## <a name="cassandra-api-configuration"></a>Cassandra API 設定

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra

//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
//Throughput-related...adjust as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```
## <a name="sample-data-generator"></a>範例資料產生器

```scala
// Generate a simple dataset containing five values
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887,11.33),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890,22.45),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892,19.83),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893,14.22),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901,12.25)
).toDF("book_id", "book_author", "book_name", "book_pub_year","book_price")

booksDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000"))
  .save()
```

## <a name="count-operation"></a>計數運算


### <a name="rdd-api"></a>RDD API

```scala
sc.cassandraTable("books_ks", "books").count
```

**輸出：**
```
res48: Long = 5
```

### <a name="dataframe-api"></a>Dataframe API

目前不支援計算 dataframe。  以下範例說明如何在將 dataframe 保存至記憶體來作為因應措施之後，執行 dataframe 計數。

請從下列可用選項中選擇一個[儲存體選項]( https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#which-storage-level-to-choose)，以避免發生「記憶體不足」問題：

* MEMORY_ONLY：這是預設的儲存體選項。 會將 RDD 儲存成 JVM 中已還原序列化的 Java 物件。 如果記憶體無法容納 RDD，系統就不會快取某些分割區，而會在每次需要它們時即時提出建議。

* MEMORY_AND_DISK：會將 RDD 儲存成 JVM 中已還原序列化的 Java 物件。 如果記憶體無法容納 RDD，請將無法容納的分割區儲存在磁碟上，然後在每次需要時，從它們的儲存位置讀取它們。

* MEMORY_ONLY_SER (Java/Scala)：會將 RDD 儲存成已序列化的 Java 物件 (每一分割區一個位元組陣列)。 與已還原序列化的物件相比，此選項更符合空間效益，尤其是當使用快速但需要更大量 CPU 來進行讀取的序列化程式時。

* MEMORY_AND_DISK_SER (Java/Scala)：此儲存體選項與 MEMORY_ONLY_SER 類似，唯一的差異在於它會讓磁碟記憶體所無法容納的分割區溢出，而不會在需要它們時重新進行計算。

* DISK_ONLY：會將 RDD 分割區只儲存在磁碟。

* MEMORY_ONLY_2、MEMORY_AND_DISK_2…：與上述層級相同，但會在兩個叢集節點上複寫每個分割區。

* OFF_HEAP (實驗性)：與 MEMORY_ONLY_SER 類似，但會將資料儲存在堆集外記憶體中，而需要事先啟用堆集外記憶體。 

```scala
//Workaround
import org.apache.spark.storage.StorageLevel

//Read from source
val readBooksDF = spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()

//Explain plan
readBooksDF.explain

//Materialize the dataframe
readBooksDF.persist(StorageLevel.MEMORY_ONLY)

//Subsequent execution against this DF hits the cache 
readBooksDF.count

//Persist as temporary view
readBooksDF.createOrReplaceTempView("books_vw")
```

### <a name="sql"></a>SQL

```sql
select * from books_vw;
select count(*) from books_vw where book_pub_year > 1900;
select count(book_id) from books_vw;
select book_author, count(*) as count from books_vw group by book_author;
select count(*) from books_vw;
```

## <a name="average-operation"></a>平均值運算

### <a name="rdd-api"></a>RDD API

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Double) => c).mean
```

**輸出：**
```
res24: Double = 16.016000175476073
```

### <a name="dataframe-api"></a>Dataframe API

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(avg("book_price"))
  .show
```

**輸出：**
```
+------------------+
|   avg(book_price)|
+------------------+
|16.016000175476073|
+------------------+
```

### <a name="sql"></a>SQL

```sql
select avg(book_price) from books_vw;
```
**輸出：**
```
16.016000175476073
```

## <a name="min-operation"></a>最小值運算

### <a name="rdd-api"></a>RDD API

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).min
```

**輸出：**
```
res31: Float = 11.33
```

### <a name="dataframe-api"></a>Dataframe API

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_id","book_price")
  .agg(min("book_price"))
  .show
```

**輸出：**
```
+---------------+
|min(book_price)|
+---------------+
|          11.33|
+---------------+
```

### <a name="sql"></a>SQL

```sql
select min(book_price) from books_vw;
```

**輸出：**
```
11.33
```

## <a name="max-operation"></a>最大值運算

### <a name="rdd-api"></a>RDD API

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).max
```

### <a name="dataframe-api"></a>Dataframe API

```scala 
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(max("book_price"))
  .show
```

**輸出：**
```
+---------------+
|max(book_price)|
+---------------+
|          22.45|
+---------------+
```

### <a name="sql"></a>SQL

```sql
select max(book_price) from books_vw;
```
**輸出：**
```22.45 ```

## <a name="sum-operation"></a>總和運算

### <a name="rdd-api"></a>RDD API

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).sum
```

**輸出：**
```
res46: Double = 80.08000087738037
```

### <a name="dataframe-api"></a>Dataframe API

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(sum("book_price"))
  .show
```
**輸出：**
```
+-----------------+
|  sum(book_price)|
+-----------------+
|80.08000087738037|
+-----------------+
```

### <a name="sql"></a>SQL

```sql
select sum(book_price) from books_vw;
```

**輸出：**
```
80.08000087738037
```

## <a name="top-or-comparable-operation"></a>排行或比較運算

### <a name="rdd-api"></a>RDD API

```scala
val readCalcTopRDD = sc.cassandraTable("books_ks", "books").select("book_name","book_price").sortBy(_.getFloat(1), false)
readCalcTopRDD.zipWithIndex.filter(_._2 < 3).collect.foreach(println)
//delivers the first top n items without collecting the rdd to the driver.
```
**輸出：**
```
(CassandraRow{book_name: A sign of four, book_price: 22.45},0)
(CassandraRow{book_name: The adventures of Sherlock Holmes, book_price: 19.83},1)
(CassandraRow{book_name: The memoirs of Sherlock Holmes, book_price: 14.22},2)
readCalcTopRDD: org.apache.spark.rdd.RDD[com.datastax.spark.connector.CassandraRow] = MapPartitionsRDD[430] at sortBy at command-2371828989676374:1
```
### <a name="dataframe-api"></a>Dataframe API

```scala
import org.apache.spark.sql.functions._

val readBooksDF = spark.read.format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load
  .select("book_name","book_price")
  .orderBy(desc("book_price"))
  .limit(3)

//Explain plan
readBooksDF.explain

//Top
readBooksDF.show
```

**輸出：**
```
== Physical Plan ==
TakeOrderedAndProject(limit=3, orderBy=[book_price#1840 DESC NULLS LAST], output=[book_name#1839,book_price#1840])
+- *(1) Scan org.apache.spark.sql.cassandra.CassandraSourceRelation@29cd5f58 [book_name#1839,book_price#1840] PushedFilters: [], ReadSchema: struct<book_name:string,book_price:float>
+--------------------+----------+
|           book_name|book_price|
+--------------------+----------+
|      A sign of four|     22.45|
|The adventures of...|     19.83|
|The memoirs of Sh...|     14.22|
+--------------------+----------+

import org.apache.spark.sql.functions._
readBooksDF: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [book_name: string, book_price: float]
```

### <a name="sql"></a>SQL

```sql
select book_name,book_price from books_vw order by book_price desc limit 3;
```

## <a name="next-steps"></a>後續步驟

若要執行資料表複製作業，請參閱：

* [資料表複製作業](cassandra-spark-table-copy-ops.md)
