---
title: 從 Azure Databricks 存取 Azure Cosmos DB Cassandra API
description: 此文章說明如何從 Azure Databricks 使用 Azure Cosmos DB Cassandra API。
services: cosmos-db
author: anagha-microsoft
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: spark-scala
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ankhanol
ms.openlocfilehash: e1d8f41c55ffd453507804b005d10620665b512c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222014"
---
# <a name="access-azure-cosmos-db-cassandra-api-data-from-azure-databricks"></a>從 Azure Databricks 存取 Azure Cosmos DB Cassandra API 資料

此文章詳細說明如何從 [Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) 上的 Spark 使用 Azure Cosmos DB Cassandra API。

## <a name="prerequisites"></a>必要條件

* [佈建 Azure Cosmos DB Cassandra API 帳戶](create-cassandra-dotnet.md#create-a-database-account)

* [檢閱連線至 Azure Cosmos DB Cassandra API 的基本概念](cassandra-spark-generic.md)

* [佈建 Azure Databricks 叢集](../azure-databricks/quickstart-create-databricks-workspace-portal.md)

* [檢閱使用 Cassandra API 的程式碼範例](cassandra-spark-generic.md#next-steps)

* [如果您願意，可以使用 cqlsh 進行驗證](cassandra-spark-generic.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **適用於 Cassandra 連接器的 Cassandra API 執行個體設定：**

  適用於 Cassandra API 的連接器需要將 Cassandra 連線詳細資料初始化以作為 Spark 內容的一部分。 當您啟動 Databricks Notebook 時，Spark 內容已初始化，因此建議您不要將它停止並重新初始化。 有一個解決方案是在叢集 Spark 設定中的叢集層級新增 Cassandra API 執行個體設定。 這在每個叢集上為一次性活動。 將下列程式碼新增至 Spark 設定，作為以空格分隔的索引鍵值組：
 
  ```scala
  spark.cassandra.connection.host YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com
  spark.cassandra.connection.port 10350
  spark.cassandra.connection.ssl.enabled true
  spark.cassandra.auth.username YOUR_COSMOSDB_ACCOUNT_NAME
  spark.cassandra.auth.password YOUR_COSMOSDB_KEY
  ```

## <a name="add-the-required-dependencies"></a>新增必要的相依性

* **Cassandra Spark 連接器：** 若要整合 Azure Cosmos DB Cassandra API 與 Spark，應該將 Cassandra 連接器連結到 Azure Databricks 叢集。 連結叢集：

  * 檢閱 Databricks 執行階段版本、Spark 版本。 接著，尋找可與 Cassandra Spark 連接器相容的 [Maven 座標](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector)，並將它連結到叢集。 請參閱[上傳 Maven 套件或 Spark 套件](https://docs.databricks.com/user-guide/libraries.html)一文，以將連接器程式庫連結至叢集。 例如，「Databricks Runtime 4.3 版」、「Spark 2.3.1」與「Scala 2.11」的 Maven 座標為 `spark-cassandra-connector_2.11-2.3.1`

* **Azure Cosmos DB Cassandra API 特有的程式庫：** 需要有自訂連線處理站，才能將 Cassandra Spark 連接器的重試原則設定到 Azure Cosmos DB Cassandra API。 新增 `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`[Maven 座標](https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.0.0/jar)，以將程式庫連結至叢集。

## <a name="sample-notebooks"></a>範例 Notebook

Github 存放庫中有一份 Azure Databricks [範例 Notebook](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/master/notebooks/scala) 清單，可供您下載。 這些範例包含如何從 Spark 連線到 Azure Cosmos DB Cassandra API，以及針對資料執行不同的 CRUD 作業。 您也可以[匯入所有 Notebook](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/master/dbc) 到您的 Databricks 叢集工作區並執行它。 

## <a name="accessing-azure-cosmos-db-cassandra-api-from-spark-scala-programs"></a>從 Spark Scala 程式存取 Azure Cosmos DB Cassandra API

要在 Azure Databricks 上當成自動化程序來執行的 Spark 程式，會使用 [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html)) 來提交，並排程要透過 Azure Databricks 作業來執行。

下列連結可協助您開始建置 Spark Scala 應用程式來與 Azure Cosmos DB Cassandra API 進行互動。
* [如何從 Spark Scala 程式連線到 Azure Cosmos DB Cassandra API](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/master/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)
* [如何在 Azure Databricks 上將 Spark Scala 應用程式當成自動化作業來執行](https://docs.azuredatabricks.net/user-guide/jobs.html) \(英文\)
* [使用 Cassandra API 的程式碼範例完整清單](cassandra-spark-generic.md#next-steps)

## <a name="next-steps"></a>後續步驟

開始使用 Java 應用程式來[建立 Cassandra API 帳戶、資料庫與資料表](create-cassandra-api-account-java.md)。
