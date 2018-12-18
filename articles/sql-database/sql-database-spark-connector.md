---
title: Spark 連接器與 Azure SQL Database 和 SQL Server| Microsoft Docs
description: 了解如何使用適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器
services: sql-database
author: allenwux
manager: craigg
ms.service: sql-database
ms.custom: ''
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: xiwu
ms.openlocfilehash: a422f65097466e4bbe5740c449d3ccf88701802b
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34650157"
---
# <a name="accelerate-real-time-big-data-analytics-with-spark-connector-for-azure-sql-database-and-sql-server"></a>使用適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器加速即時巨量資料分析

適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器可讓 SQL 資料庫 (包括 Azure SQL Database 和 SQL Server) 做為 Spark 作業的輸入資料來源或輸出資料接收器。 它可讓您利用巨量資料分析中的即時交易資料，並將保存臨機操作查詢或報告的結果。 相較於內建的 JDBC 連接器，此連接器提供大量插入資料至 SQL 資料庫的能力。 它可以用 10 倍到 20 倍快的效能執行逐列插入。 適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器也支援 AAD 驗證。 它可讓您使用 AAD 帳戶安全地從 Azure Databricks 連線到您的 Azure SQL Database。 它提供類似內建 JDBC 連接器的介面。 您可以輕鬆移轉現有的 Spark 作業以使用此新的連接器。

## <a name="download"></a>下載
若要開始進行，請從 GitHub 上的 [azure-sqldb-spark 存放庫](https://github.com/Azure/azure-sqldb-spark)下載「Spark 至 SQL DB」連接器。

## <a name="official-supported-versions"></a>官方支援的版本

| 元件                            |版本                  |
| :----------------------------------- | :---------------------- |
| Apache Spark                         |2.0.2 或更新版本           |
| Scala                                |2.10 或更新版本            |
| Microsoft JDBC Driver for SQL Server |6.2 或更新版本             |
| 連接字串                 |SQL Server 2008 或更新版本 |
| 連接字串                   |支援                |

適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器會利用 Microsoft JDBC Driver for SQL Server 在 Spark 背景工作節點與 SQL 資料庫之間移動資料：
 
資料流程如下：
1. Spark 主要節點會連線到 SQL Server 或 Azure SQL Database 並從特定資料表或使用特定 SQL 查詢來載入資料
2. Spark 主要節點將資料散發到背景工作節點以供轉換。 
3. 背景工作節點會連線到 SQL Server 或 Azure SQL Database 並將資料寫入資料庫。 使用者可以選擇使用逐列插入或大量插入。

下圖說明此資料流程。

   ![架構](./media/sql-database-spark-connector/architecture.png)

### <a name="build-the-spark-to-sql-db-connector"></a>建置 Spark 至 SQL DB 連接器
目前，連接器專案會使用 maven。 若要建置沒有相依性的連接器，您可以執行︰

- mvn 全新套件
- 從發行資料夾下載最新版 JAR
- 包含 SQL DB Spark JAR

## <a name="connect-spark-to-sql-db-using-the-connector"></a>使用連接器將 Spark 連線到 SQL DB
您可以從 Spark 作業連線到 Azure SQL Database 或 SQL Server、讀取或寫入資料。 您也可以在 Azure SQL Database 或 SQL Server 資料庫中執行 DML 或 DDL 查詢。

### <a name="read-data-from-azure-sql-database-or-sql-server"></a>從 Azure SQL Database 或 SQL Server 讀取資料

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "dbTable"        -> "dbo.Clients"
  "user"           -> "username",
  "password"       -> "*********",
  "connectTimeout" -> "5", //seconds
  "queryTimeout"   -> "5"  //seconds
))

val collection = sqlContext.read.sqlDB(config)
collection.show()
```
### <a name="reading-data-from-azure-sql-database-or-sql-server-with-specified-sql-query"></a>使用指定的 SQL 查詢從 Azure SQL Database 或 SQL Server 讀取資料
```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "queryCustom"  -> "SELECT TOP 100 * FROM dbo.Clients WHERE PostalCode = 98074" //Sql query
  "user"         -> "username",
  "password"     -> "*********",
))

//Read all data in table dbo.Clients
val collection = sqlContext.read.sqlDb(config)
collection.show()
```

### <a name="write-data-to-azure-sql-database-or-sql-server"></a>將資料寫入 Azure SQL Database 或 SQL Server
```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._
 
// Aquire a DataFrame collection (val collection)

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "dbTable"      -> "dbo.Clients"
  "user"         -> "username",
  "password"     -> "*********"
))

import org.apache.spark.sql.SaveMode
collection.write.mode(SaveMode.Append).sqlDB(config)
```

### <a name="run-dml-or-ddl-query-in-azure-sql-database-or-sql-server"></a>在 Azure SQL Database 或 SQL Server 中執行 DML 或 DDL 查詢
```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.query._
val query = """
              |UPDATE Customers
              |SET ContactName = 'Alfred Schmidt', City= 'Frankfurt'
              |WHERE CustomerID = 1;
            """.stripMargin

val config = Config(Map(
  "url"          -> "mysqlserver.database.windows.net",
  "databaseName" -> "MyDatabase",
  "user"         -> "username",
  "password"     -> "*********",
  "queryCustom"  -> query
))

sqlContext.SqlDBQuery(config)
```

## <a name="connect-spark-to-azure-sql-database-using-aad-authentication"></a>使用 AAD 驗證將 Spark 連線到 Azure SQL Database
您可以使用 Azure Active Directory (AAD) 驗證以連線到 SQL Database。 使用 AAD 驗證以集中管理資料庫使用者的身分識別，以及做為 SQL Server 驗證的替代方案。
### <a name="connecting-using-activedirectorypassword-authentication-mode"></a>使用 ActiveDirectoryPassword 驗證模式來連線
#### <a name="setup-requirement"></a>安裝需求
如果您使用 ActiveDirectoryPassword 驗證模式，必須下載 [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java) 及其相依項目，並將它們包含在 Java 建置路徑中。

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"            -> "mysqlserver.database.windows.net",
  "databaseName"   -> "MyDatabase",
  "user"           -> "username ",
  "password"       -> "*********",
  "authentication" -> "ActiveDirectoryPassword",
  "encrypt"        -> "true"
))

val collection = sqlContext.read.SqlDB(config)
collection.show()
```

### <a name="connecting-using-access-token"></a>使用存取權杖來連線
#### <a name="setup-requirement"></a>安裝需求
如果您使用存取權杖型驗證模式，必須下載 [azure-activedirectory-library-for-java](https://github.com/AzureAD/azure-activedirectory-library-for-java)及其相依項目，並將它們包含在 Java 建置路徑中。

請參閱[使用 Azure Active Directory 驗證向 SQL Database 進行驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)，以了解如何取得 Azure SQL Database 的存取權杖。

```scala
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

val config = Config(Map(
  "url"                   -> "mysqlserver.database.windows.net",
  "databaseName"          -> "MyDatabase",
  "accessToken"           -> "access_token ",
  "hostNameInCertificate" -> "*.database.windows.net",
  "encrypt"               -> "true"
))

val collection = sqlContext.read.SqlDB(config)
collection.show()
```

## <a name="write-data-to-azure-sql-database-or-sql-server-using-bulk-insert"></a>使用大量插入將資料寫入 Azure SQL Database 或 SQL Server
傳統的 jdbc 連接器使用逐列插入將資料寫入 Azure SQL Database 或 SQL Server。 您可以使用 Spark 至 SQL DB 連接器，藉由大量插入將資料寫入 SQL Database。 在載入大型資料集或將資料載入使用資料行存放區索引的資料表時，它會大幅改善寫入效能。

```scala
import com.microsoft.azure.sqldb.spark.bulkcopy.BulkCopyMetadata
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._

/** 
  Add column Metadata.
  If not specified, metadata is automatically added
  from the destination table, which may suffer performance.
*/
var bulkCopyMetadata = new BulkCopyMetadata
bulkCopyMetadata.addColumnMetadata(1, "Title", java.sql.Types.NVARCHAR, 128, 0)
bulkCopyMetadata.addColumnMetadata(2, "FirstName", java.sql.Types.NVARCHAR, 50, 0)
bulkCopyMetadata.addColumnMetadata(3, "LastName", java.sql.Types.NVARCHAR, 50, 0)

val bulkCopyConfig = Config(Map(
  "url"               -> "mysqlserver.database.windows.net",
  "databaseName"      -> "MyDatabase",
  "user"              -> "username",
  "password"          -> "*********",
  "databaseName"      -> "zeqisql",
  "dbTable"           -> "dbo.Clients",
  "bulkCopyBatchSize" -> "2500",
  "bulkCopyTableLock" -> "true",
  "bulkCopyTimeout"   -> "600"
))

df.bulkCopyToSqlDB(bulkCopyConfig, bulkCopyMetadata)
//df.bulkCopyToSqlDB(bulkCopyConfig) if no metadata is specified.
```

## <a name="next-steps"></a>後續步驟
如果您還沒有從 [azure-sqldb-spark GitHub 存放庫](https://github.com/Azure/azure-sqldb-spark) 下載適用於 Azure SQL Database 和 SQL Server 的 Spark 連接器，請進行下載並探索存放庫中的其他資源：

-   [Azure Databricks Notebook 範例](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/notebooks)
- [指令碼範例 (Scala)](https://github.com/Azure/azure-sqldb-spark/tree/master/samples/scripts)

您也可以檢閱 [Apache Spark SQL、DataFrame 和 Dataset 指南](http://spark.apache.org/docs/latest/sql-programming-guide.html) (英文) 和 [Azure Databricks 文件](https://docs.microsoft.com/azure/azure-databricks/)。

