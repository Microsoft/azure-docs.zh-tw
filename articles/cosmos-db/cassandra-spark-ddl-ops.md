---
title: 從 Spark 進行 Azure Cosmos DB Cassandra API 的 DDL 作業
description: 本文詳細說明如何從 Spark 針對 Azure Cosmos DB Cassandra API 進行 Keyspace 和資料表 DDL 作業。
services: cosmos-db
author: anagha-microsoft
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: a799d85cc27575badda6892ba7baf68ca1eb1dfb
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226070"
---
# <a name="ddl-operations-in-azure-cosmos-db-cassandra-api-from-spark"></a>從 Spark 進行 Azure Cosmos DB Cassandra API 的 DDL 作業

本文詳細說明如何從 Spark 針對 Azure Cosmos DB Cassandra API 進行 Keyspace 和資料表 DDL 作業。

## <a name="cassandra-api-related-configuration"></a>Cassandra API 相關設定 

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

## <a name="keyspace-ddl-operations"></a>Keyspace DDL 作業

### <a name="create-a-keyspace"></a>建立 Keyspace

```scala
//Cassandra connector instance
val cdbConnector = CassandraConnector(sc)

// Create keyspace
cdbConnector.withSessionDo(session => session.execute("CREATE KEYSPACE IF NOT EXISTS books_ks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1 } "))
```

#### <a name="validate-in-cqlsh"></a>在 cqlsh 中驗證

在 cqlsh 中執行下列命令，您應該會看到您稍早建立的 Keyspace。

```bash
DESCRIBE keyspaces;
```

### <a name="drop-a-keyspace"></a>卸除 Keyspace

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP KEYSPACE books_ks"))
```

#### <a name="validate-in-cqlsh"></a>在 cqlsh 中驗證

```bash
DESCRIBE keyspaces;
```
## <a name="table-ddl-operations"></a>資料表 DDL 作業

<bpt id="p1">**</bpt>Considerations:<ept id="p1">**</ept>  

- 使用 create table 陳述式，即可在資料表層級指派輸送量。  
- 一個分割區索引鍵可以儲存 10 GB 的資料。  
- 一筆記錄最多可以儲存 2 MB 的資料。  
- 一個分割區索引鍵範圍可以儲存多個分割區索引鍵。

### <a name="create-a-table"></a>建立資料表

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))
```

#### <a name="validate-in-cqlsh"></a>在 cqlsh 中驗證

在 cqlsh 中執行下列命令，您應該會看到名為 "books" 的資料表： 

```bash
USE books_ks;
DESCRIBE books;
```

佈建的輸送量和預設 TTL 值不會顯示在前一個命令的輸出中，您可以從入口網站取得這些值。

### <a name="alter-table"></a>改變資料表

您可以使用 alter table 命令來改變下列值：

* 佈建的輸送量 
* 存留時間值
<br>目前不支援資料行變更。

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("ALTER TABLE books_ks.books WITH cosmosdb_provisioned_throughput=8000, WITH default_time_to_live=0;"))
```

### <a name="drop-table"></a>卸除資料表

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP TABLE IF EXISTS books_ks.books;"))
```

#### <a name="validate-in-cqlsh"></a>在 cqlsh 中驗證

在 cqlsh 中執行下列命令，您應該會看到 "books" 資料表再也無法使用：

```bash
USE books_ks;
DESCRIBE tables;
```

## <a name="next-steps"></a>後續步驟

建立 Keyspace 和資料表之後，請繼續閱讀下列文章以了解 CRUD 作業和其他功能：
 
* [建立/插入作業](cassandra-spark-create-ops.md)  
* [讀取作業](cassandra-spark-read-ops.md)  
* [作業](cassandra-spark-upsert-ops.md)  
* [刪除作業](cassandra-spark-delete-ops.md)  
* [彙總作業](cassandra-spark-aggregation-ops.md)  
* [資料表複製作業](cassandra-spark-table-copy-ops.md)  
