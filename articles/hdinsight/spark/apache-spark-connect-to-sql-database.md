---
title: 使用 Apache Spark 對 Azure SQL 資料庫讀取及寫入資料
description: 了解如何設定 HDInsight Spark 叢集與 Azure SQL 資料庫之間的連線，以對 SQL 資料庫讀取資料、寫入資料和串流資料
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2018
ms.openlocfilehash: 2aec894da6b4e5ffd59fee12bc8476b25955c991
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045791"
---
# <a name="use-hdinsight-spark-cluster-to-read-and-write-data-to-azure-sql-database"></a>使用 HDInsight Spark 叢集對 Azure SQL 資料庫讀取及寫入資料

了解如何連線 Azure HDInsight 中的 Apache Spark 叢集與 Azure SQL 資料庫，然後將資料讀取、寫入和串流到 SQL 資料庫中。 本文中的指示使用 Jupyter Notebook 執行 Scala 程式碼片段。 不過，您可以在 Scala 或 Python 中建立獨立應用程式，並執行相同的工作。 

## <a name="prerequisites"></a>必要條件

* **Azure HDInsight Spark 叢集**。  請依照[在 HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)中的指示操作。

* **Azure SQL 資料庫**。 請依照[建立 Azure SQL 資料庫](../../sql-database/sql-database-get-started-portal.md)中的指示操作。 請確實使用範例 **AdventureWorksLT** 結構描述和資料來建立資料庫。 此外，請確實建立允許用戶端 IP 位址對伺服器上的 SQL 資料庫進行存取的伺服器層級防火牆規則。 在相同的文章中可找到新增防火牆規則的指示。 建立 Azure SQL 資料庫之後，請確實將下列值存放在方便取用之處。 您從 Spark 叢集連線至資料庫時需要用到這些值。

    * 裝載 Azure SQL 資料庫的伺服器名稱
    * Azure SQL Database 名稱
    * Azure SQL 資料庫管理使用者名稱/密碼

* **SQL Server Management Studio**。 請依照[使用 SSMS 進行連線及查詢資料](../../sql-database/sql-database-connect-query-ssms.md)中的指示操作。

## <a name="create-a-jupyter-notebook"></a>建立 Jupyter Notebook

首先請建立與 Spark 叢集相關聯的 Jupyter Notebook。 您可以使用此 Notebook 執行本文中使用的程式碼片段。 

1. 從 [Azure 入口網站](https://portal.azure.com/)，開啟您的叢集。 

1. 從 [快速連結] 區段按一下 [叢集儀表板]，以開啟 [叢集儀表板] 檢視。  如果您沒有看見 [快速連結]，請按一下刀鋒視窗上左側功能表中的 [概觀]。

    ![Spark 上的叢集儀表板](./media/apache-spark-connect-to-sql-database/hdinsight-cluster-dashboard-on-spark.png "Spark 上的叢集儀表板") 

1. 按一下 [Jupyter Notebook]。 出現提示時，輸入叢集的系統管理員認證。

    ![Spark 上的 Jupyter Notebook](./media/apache-spark-connect-to-sql-database/hdinsight-jupyter-notebook-on-spark.png "Spark 上的 Jupyter Notebook")
   
   > [!NOTE]
   > 您也可以在瀏覽器中開啟下列 URL，以存取 Spark 叢集上的 Jupyter Notebook。 使用您叢集的名稱取代 **CLUSTERNAME** ：
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`
   > 
   > 

1. 在 Jupyter Notebook 中，按一下右上角的 [新增]，然後按一下 [Spark] 以建立 Scala Notebook。 HDInsight Spark 叢集上的 Jupyter Notebook 也會提供適用於 Python2 應用程式的 **PySpark** 核心，以及適用於 Python3 應用程式的 **PySpark3** 核心。 在本文中，我們會建立 Scala Notebook。
   
    ![Spark 上的 Jupyter Notebook 核心](./media/apache-spark-connect-to-sql-database/kernel-jupyter-notebook-on-spark.png "Spark 上的 Jupyter Notebook 核心")

    如需核心的詳細資訊，請參閱[在 HDInsight 中搭配使用 Jupyter Notebook 核心與 Apache Spark 叢集](apache-spark-jupyter-notebook-kernels.md)。

   > [!NOTE]
   > 在本文中我們將使用 Spark (Scala) 核心，因為目前只有 Scala 和 Java 支援將 Spark 中的資料串流至 SQL 資料庫的作業。 即使對 SQL 的讀取和寫入可使用 Python 來執行，但為了本文的一致性，我們對三項作業都會使用 Scala。
   >

1. 這時會開啟一個採用預設名稱「**未命名**」的新 Notebook。 請按一下 Notebook 名稱，並輸入您選擇的名稱。

    ![提供 Notebook 的名稱](./media/apache-spark-connect-to-sql-database/hdinsight-spark-jupyter-notebook-name.png "提供 Notebook 的名稱")

現在您可以開始建立應用程式。
    
## <a name="read-data-from-azure-sql-database"></a>從 Azure SQL 資料庫讀取資料

在本節中，您會從位於 AdventureWorks 資料庫的資料表 (例如 **SalesLT.Address**) 中讀取資料。

1. 在新 Jupyter Notebook 的程式碼單元中貼上下列程式碼片段，並以您的 Azure SQL 資料庫的值取代預留位置值。

       // Declare the values for your Azure SQL database

       val jdbcUsername = "<SQL DB ADMIN USER>"
       val jdbcPassword = "<SQL DB ADMIN PWD>"
       val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
       val jdbcPort = 1433
       val jdbcDatabase ="<AZURE SQL DB NAME>"

    按 **SHIFT + ENTER** 以執行程式碼單元。  

1. 使用下列程式碼片段來建置可傳遞至 Spark 資料框架 API 的 JDBC URL，並建立用來存放參數的 `Properties` 物件。 請在程式碼單元中貼上此程式碼片段，然後按 **SHIFT + ENTER** 加以執行。

       import java.util.Properties

       val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
       val connectionProperties = new Properties()
       connectionProperties.put("user", s"${jdbcUsername}")
       connectionProperties.put("password", s"${jdbcPassword}")         

1. 使用下列程式碼片段，利用您 Azure SQL Database 中的資料表所包含的資料來建立資料框架。 在此程式碼片段中，我們使用可納入 **AdventureWorksLT** 資料庫中的 **SalesLT.Address** 資料表。 請在程式碼單元中貼上此程式碼片段，然後按 **SHIFT + ENTER** 加以執行。

       val sqlTableDF = spark.read.jdbc(jdbc_url, "SalesLT.Address", connectionProperties)

1. 現在，您可以執行資料框架的作業，例如取得資料結構描述：

       sqlTableDF.printSchema
   
    您會看到如下的輸出：

    ![提供 Notebook 的名稱](./media/apache-spark-connect-to-sql-database/read-from-sql-schema-output.png "提供 Notebook 的名稱")

1. 您也可以執行像是擷取前 10 個資料列的作業。

       sqlTableDF.show(10)

1. 或者，您可以從資料集擷取特定資料行。

       sqlTableDF.select("AddressLine1", "City").show(10)

## <a name="write-data-into-azure-sql-database"></a>將資料寫入 Azure SQL 資料庫中

在本節中，我們將使用叢集上提供的範例 CSV 檔案建立 Azure SQL 資料庫中的資料表，並在其中填入資料。 範例 CSV 檔案 (**HVAC.csv**) 可從所有 HDInsight 叢集取得，位置是 `HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv`。

1. 在新 Jupyter Notebook 的程式碼單元中貼上下列程式碼片段，並以您的 Azure SQL 資料庫的值取代預留位置值。

       // Declare the values for your Azure SQL database

       val jdbcUsername = "<SQL DB ADMIN USER>"
       val jdbcPassword = "<SQL DB ADMIN PWD>"
       val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
       val jdbcPort = 1433
       val jdbcDatabase ="<AZURE SQL DB NAME>"

    按 **SHIFT + ENTER** 以執行程式碼單元。  

1. 下列程式碼片段會建置可傳遞至 Spark 資料框架 API 的 JDBC URL，並建立用來存放參數的 `Properties` 物件。 請在程式碼單元中貼上此程式碼片段，然後按 **SHIFT + ENTER** 加以執行。

       import java.util.Properties

       val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
       val connectionProperties = new Properties()
       connectionProperties.put("user", s"${jdbcUsername}")
       connectionProperties.put("password", s"${jdbcPassword}")

1. 使用下列程式碼片段來擷取 HVAC.csv 中資料的結構描述，並使用該結構描述，從資料框架 `readDf` 中的 CSV 載入資料。 請在程式碼單元中貼上此程式碼片段，然後按 **SHIFT + ENTER** 加以執行。

       val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
       val readDf = spark.read.format("csv").schema(userSchema).load("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

1. 使用 `readDf` 資料框架建立暫存資料表 `temphvactable`。 然後，使用該暫存資料表建立登錄區資料表 `hvactable_hive`。

       readDf.createOrReplaceTempView("temphvactable")
       spark.sql("create table hvactable_hive as select * from temphvactable")

1. 最後，使用該登錄區資料表在 Azure SQL 資料庫中建立資料表。 下列程式碼片段會在 Azure SQL 資料庫中建立 `hvactable`。

       spark.table("hvactable_hive").write.jdbc(jdbc_url, "hvactable", connectionProperties)

1. 使用 SSMS 連線至 Azure SQL 資料庫，並確認您在該處看到 `dbo.hvactable`。

    a. 啟動 SSMS，並提供如以下螢幕擷取畫面所示的連線詳細資料，以連線至 Azure SQL 資料庫。

    ![使用 SSMS 連線至 SQL 資料庫](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms.png "使用 SSMS 連線至 SQL 資料庫")

    b. 在 [物件總管] 中展開 Azure SQL 資料庫和資料表節點，以檢視已建立的 **dbo.hvactable**。

    ![使用 SSMS 連線至 SQL 資料庫](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms-locate-table.png "使用 SSMS 連線至 SQL 資料庫")

1. 在 SSMS 中執行查詢，以查看資料表中的資料行。

        SELECT * from hvactable

## <a name="stream-data-into-azure-sql-database"></a>將資料串流至 Azure SQL 資料庫

在本節中，我們會將資料串流至您在上一節建立於 Azure SQL 資料庫中的 **hvactable**。

1. 首先，請確定 **hvactable** 中沒有任何記錄。 請使用 SSMS 對資料表執行下列查詢。

       DELETE FROM [dbo].[hvactable]

1. 在 HDInsight Spark 叢集上建立新的 Jupyter Notebook。 在程式碼單元中貼上下列程式碼片段，然後按 **SHIFT + ENTER**：

       import org.apache.spark.sql._
       import org.apache.spark.sql.types._
       import org.apache.spark.sql.functions._
       import org.apache.spark.sql.streaming._
       import java.sql.{Connection,DriverManager,ResultSet}

1. 我們要將 **HVAC.csv** 中的資料串流至 hvactable。 HVAC.csv 檔案可從叢集上的 /HdiSamples/HdiSamples/SensorSampleData/HVAC/ 取得。 在下列程式碼片段中，我們會先取得要串流處理之資料的結構描述。 接著，我們會使用該結構描述建立串流資料框架。 請在程式碼單元中貼上此程式碼片段，然後按 **SHIFT + ENTER** 加以執行。

       val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
       val readStreamDf = spark.readStream.schema(userSchema).csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/") 
       readStreamDf.printSchema

1. 輸出會顯示 **HVAC.csv** 的結構描述。 **Hvactable** 也具有相同的結構描述。 輸出會列出資料表中的資料行。

    ![資料表的結構描述](./media/apache-spark-connect-to-sql-database/schema-of-table.png "資料表的結構描述")

1. 最後，使用下列程式碼片段從 HVAC.csv 讀取資料，並將其串流至 Azure SQL 資料庫中的 **hvactable**。 在程式碼單元中貼上程式碼片段，並以您的 Azure SQL 資料庫的值取代預留位置值，然後按 **SHIFT + ENTER** 加以執行。

       val WriteToSQLQuery  = readStreamDf.writeStream.foreach(new ForeachWriter[Row] {
          var connection:java.sql.Connection = _
          var statement:java.sql.Statement = _
          
          val jdbcUsername = "<SQL DB ADMIN USER>"
          val jdbcPassword = "<SQL DB ADMIN PWD>"
          val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
          val jdbcPort = 1433
          val jdbcDatabase ="<AZURE SQL DB NAME>"
          val driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
          val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
  
         def open(partitionId: Long, version: Long):Boolean = {
           Class.forName(driver)
           connection = DriverManager.getConnection(jdbc_url, jdbcUsername, jdbcPassword)
           statement = connection.createStatement
           true
         }
  
         def process(value: Row): Unit = {
           val Date  = value(0)
           val Time = value(1)
           val TargetTemp = value(2)
           val ActualTemp = value(3)
           val System = value(4)
           val SystemAge = value(5)
           val BuildingID = value(6)  
    
           val valueStr = "'" + Date + "'," + "'" + Time + "'," + "'" + TargetTemp + "'," + "'" + ActualTemp + "'," + "'" + System + "'," + "'" + SystemAge + "'," + "'" + BuildingID + "'"
           statement.execute("INSERT INTO " + "dbo.hvactable" + " VALUES (" + valueStr + ")")   
           }

         def close(errorOrNull: Throwable): Unit = {
            connection.close
          }
         })
        
         var streamingQuery = WriteToSQLQuery.start()

1. 在 SQL Server Management Studio (SSMS) 中執行下列查詢，以確認資料已串流至 **hvactable**。 每當您執行此查詢時，都會顯示資料表中增加的資料列數。

        SELECT COUNT(*) FROM hvactable

## <a name="next-steps"></a>後續步驟

* [使用 HDInsight Spark 叢集分析 Data Lake Store 中的資料](apache-spark-use-with-data-lake-store.md)
* [使用 EventHub 處理結構化串流事件](apache-spark-eventhub-structured-streaming.md)
* [搭配使用 Spark 結構化串流與 HDInsight 上的 Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)
