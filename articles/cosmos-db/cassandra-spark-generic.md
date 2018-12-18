---
title: 從 Spark 使用 Azure Cosmos DB Cassandra API
description: 本文是從 Spark 進行 Cosmos DB Cassandra API 整合的主頁面。
services: cosmos-db
author: anagha-microsoft
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: f2f5aebf32cf5860ca8fc32ab741177c6df15c60
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227192"
---
# <a name="connect-to-azure-cosmos-db-cassandra-api-from-spark"></a>從 Spark 連線至 Azure Cosmos DB Cassandra API

本文是從 Spark 進行 Azure Cosmos DB Cassandra API 整合的一系列文章之一。 這些文章涵蓋連線能力、資料定義語言 (DDL) 作業、基本的資料操作語言 (DML) 作業，以及從 Spark 進行進階 Azure Cosmos DB Cassandra API 整合。 

## <a name="prerequisites"></a>必要條件
* [佈建 Azure Cosmos DB Cassandra API 帳戶。](create-cassandra-dotnet.md#create-a-database-account)

* 佈建您選擇的 Spark 環境 [[Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal) | [Azure HDInsight-Spark](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql) | 其他]。

## <a name="dependencies-for-connectivity"></a>連線能力的相依項目
* **適用於 Cassandra 的 Spark 連接器：** Spark 連接器用來連線到 Azure Cosmos DB Cassandra API。  請找出位於 [Maven 中心]( https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector)且與 Spark 環境的 Spark 和 Scala 版本相容的連接器版本，並加以使用。

* **適用於 Cassandra API 的 Azure Cosmos DB 協助程式程式庫：** 除了 Spark 連接器之外，您還需要 Azure Cosmos DB 中另一個稱為 [azure-cosmos-cassandra-spark-helper]( https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.0.0/jar) 的程式庫。 此程式庫包含連線處理站和自訂重試原則類別。

  Azure Cosmos DB 中的重試原則設定為處理 HTTP 狀態碼 429 (「要求速率很大」) 的例外狀況。 Azure Cosmos DB Cassandra API 會將這些例外狀況轉譯成 Cassandra 原生通訊協定上的多載錯誤，您可以使用輪詢進行重試。 由於 Azure Cosmos DB 會使用佈建的輸送量模型，因此輸入/輸出速率增加時，也會發生要求速率限制例外狀況。 重試原則可保護您的 Spark 工作以免遭受資料暴增的影響，而資料暴增會暫時超出配置給集合的輸送量。

  > [!NOTE] 
  > 重試原則只能保護您的 Spark 工作以免遭受暫時暴增的影響。 如果您未設定執行工作負載所需的足夠 RU，則重試原則不適用，而且重試原則類別會重新擲回例外狀況。

* **Azure Cosmos DB 帳戶連線詳細資料：** Azure Cassandra API 帳戶名稱、帳戶端點和金鑰。
    
## <a name="spark-connector-throughput-configuration-parameters"></a>Spark 連接器輸送量設定參數

下表列出連接器所提供的 Azure Cosmos DB Cassandra API 特定輸送量設定參數。 如需所有設定參數的詳細清單，請參閱 Spark Cassandra 連接器 GitHub 存放庫的[設定參考](https://github.com/datastax/spark-cassandra-connector/blob/master/doc/reference.md)頁面。

| **屬性名稱** | **預設值** | **說明** |
|---------|---------|---------|
| spark.cassandra.output.batch.size.rows |  1 |每個單一批次的資料列數目。 將此參數設定為 1。 此參數用來為繁重的工作負載達到更高的輸送量。 |
| spark.cassandra.connection.connections_per_executor_max  | None | 每個執行程式的每個節點連線數目上限。 10*n 相當於 n 個節點的 Cassandra 叢集中每個節點有 10 個連線。 因此，如果針對 5 個節點的 Cassandra 叢集，您需要每個執行程式的每個節點有 5 個連線，則您應該將此設定設為 25。 請根據 Spark 工作設定的平行處理原則程度或執行程式數目來修改此值。   |
| spark.cassandra.output.concurrent.writes  |  100 | 定義每個執行程式可能發生的平行寫入數目。 因為您將 "batch.size.rows" 設定為 1，所以請務必據以相應增加此值。 請根據您想要針對工作負載達到的平行處理原則程度或輸送量來修改此值。 |
| spark.cassandra.concurrent.reads |  512 | 定義每個執行程式可能發生的平行讀取數目。 請根據您想要針對工作負載達到的平行處理原則程度或輸送量來修改此值  |
| spark.cassandra.output.throughput_mb_per_sec  | None | 定義每個執行程式的總寫入輸送量。 此參數可用作 Spark 工作輸送量的上限，並以 Cosmos DB 集合的佈建輸送量為基礎。   |
| spark.cassandra.input.reads_per_sec| None   | 定義每個執行程式的總讀取輸送量。 此參數可用作 Spark 工作輸送量的上限，並以 Cosmos DB 集合的佈建輸送量為基礎。  |
| spark.cassandra.output.batch.grouping.buffer.size |  1000  | 定義每個單一 Spark 工作可在傳送至 Cassandra API 之前，先存放在記憶體中的批次數目 |
| spark.cassandra.connection.keep_alive_ms | 60000 | 定義未使用的連線可供使用的期間。 | 

請根據您對 Spark 工作預期的工作負載以及您為 Cosmos DB 帳戶佈建的輸送量，調整這些參數的輸送量和平行處理原則程度。

## <a name="connecting-to-azure-cosmos-db-cassandra-api-from-spark"></a>從 Spark 連線至 Azure Cosmos DB Cassandra API

### <a name="cqlsh"></a>cqlsh
下列命令將詳細說明如何從 cqlsh 連線至 Azure CosmosDB Cassandra API。  當您在 Spark 中執行範例時，這對於驗證相當有用。<br>
**從 Linux/Unix/Mac：**

```bash
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false
cqlsh.py YOUR-COSMOSDB-ACCOUNT-NAME.cassandra.cosmosdb.windows-ppe.net 10350 -u YOUR-COSMOSDB-ACCOUNT-NAME -p YOUR-COSMOSDB-ACCOUNT-KEY --ssl
```

### <a name="1--azure-databricks"></a>1.Azure Databricks
下文說明用於連線至 Azure Cosmos DB Cassandra API 的 Azure Databricks 叢集佈建、叢集設定，以及涵蓋 DDL 作業、DML 作業和其他項目的數個範例 Notebook。<BR>
[從 Azure Databricks 使用 Azure Cosmos DB Cassandra API](cassandra-spark-databricks.md)<BR>
  
### <a name="2--azure-hdinsight-spark"></a>2.Azure HDInsight-Spark
下文說明用於連線至 Azure Cosmos DB Cassandra API 的 HDinsight-Spark 服務、佈建、叢集設定，以及涵蓋 DDL 作業、DML 作業和其他項目的數個範例 Notebook。<BR>
[從 Azure HDInsight-Spark 使用 Azure Cosmos DB Cassandra API](cassandra-spark-hdinsight.md)
 
### <a name="3--spark-environment-in-general"></a>3.一般情況下的 Spark 環境
上述各節是專門針對以 Azure Spark 為基礎的 PaaS 服務，而本節則涵蓋所有一般 Spark 環境。  連接器相依項目、匯入和 Spark 工作階段設定詳述如下。 ＜後續步驟＞一節涵蓋適用於 DDL 作業、DML 作業和其他項目的程式碼範例。  

#### <a name="connector-dependencies"></a>連接器相依項目：

1. 新增 Maven 座標，以取得[適用於 Spark 的 Cassandra 連接器](cassandra-spark-generic.md#dependencies-for-connectivity)
2. 針對適用於 Cassandra API 的 [Azure Cosmos DB 協助程式程式庫](cassandra-spark-generic.md#dependencies-for-connectivity)新增 Maven 座標

#### <a name="imports"></a>匯入：

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra
```

#### <a name="spark-session-configuration"></a>Spark 工作階段設定：

```scala
//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

//Throughput-related. You can adjust the values as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```

## <a name="next-steps"></a>後續步驟

下列文章會示範 Spark 如何與 Azure Cosmos DB Cassandra API 整合。 
 
* [DDL 作業](cassandra-spark-ddl-ops.md)
* [建立/插入作業](cassandra-spark-create-ops.md)
* [讀取作業](cassandra-spark-read-ops.md)
* [作業](cassandra-spark-upsert-ops.md)
* [刪除作業](cassandra-spark-delete-ops.md)
* [彙總作業](cassandra-spark-aggregation-ops.md)
* [資料表複製作業](cassandra-spark-table-copy-ops.md)
