---
title: 從 Spark 將資料更新插入至 Azure Cosmos DB Cassandra API
description: 此文章詳細說明如何從 Spark 更新插入至 Azure Cosmos DB Cassandra API 中的資料表
services: cosmos-db
author: anagha-microsoft
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: 5bb4d066d6003dde38b02a3f4ac6c66463dd5ebe
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221452"
---
# <a name="upsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>從 Spark 將資料更新插入至 Azure Cosmos DB Cassandra API

此文章說明如何從 Spark 將資料更新插入至 Azure Cosmos DB Cassandra API。

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

## <a name="dataframe-api"></a>Dataframe API

### <a name="create-a-dataframe"></a>建立資料框架 

```scala
// (1) Update: Changing author name to include prefix of "Sir"
// (2) Insert: adding a new book

val booksUpsertDF = Seq(
    ("b00001", "Sir Arthur Conan Doyle", "A study in scarlet", 1887),
    ("b00023", "Sir Arthur Conan Doyle", "A sign of four", 1890),
    ("b01001", "Sir Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
    ("b00501", "Sir Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
    ("b00300", "Sir Arthur Conan Doyle", "The hounds of Baskerville", 1901),
    ("b09999", "Sir Arthur Conan Doyle", "The return of Sherlock Holmes", 1905)
    ).toDF("book_id", "book_author", "book_name", "book_pub_year")
booksUpsertDF.show()
```

### <a name="upsert-data"></a>更新插入資料

```scala
// Upsert is no different from create
booksUpsertDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .save()
```

### <a name="update-data"></a>更新資料

```scala
//This runs on the driver, leverage only for one off updates
cdbConnector.withSessionDo(session => session.execute("update books_ks.books set book_price=99.33 where book_id ='b00300';"))
```

## <a name="rdd-api"></a>RDD API
> [!NOTE]
> 從 RDD API 更新插入等同於建立作業 

## <a name="next-steps"></a>後續步驟

繼續閱讀下列文章，以對 Azure Cosmos DB Cassandra API 資料表中儲存的資料執行其他作業：
 
* [刪除作業](cassandra-spark-delete-ops.md)
* [彙總作業](cassandra-spark-aggregation-ops.md)
* [資料表複製作業](cassandra-spark-table-copy-ops.md)
