---
title: '教學課程：使用 HDInsight 上的 Hive 執行擷取、轉換、載入 (ETL) 作業 - Azure '
description: 了解如何從原始 CSV 資料集擷取資料、使用 HDInsight 上的 Hive 加以轉換，然後使用 Sqoop 將已轉換的資料載入 Azure SQL 資料庫中。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.date: 05/07/2018
ms.author: jasonh
ms.custom: H1Hack27Feb2017,hdinsightactive,mvc
ms.openlocfilehash: 9ea9fbdbe07013131f35a7a0c3d6ab4af19f7056
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990938"
---
# <a name="tutorial-extract-transform-and-load-data-using-apache-hive-on-azure-hdinsight"></a>教學課程：使用 Azure HDInsight 上的 Apache Hive 來擷取、轉換和載入資料

在本教學課程中，您會取用原始 CSV 資料檔、將其匯入 HDInsight 叢集儲存體中，然後使用 Azure HDInsight 上的 Apache Hive 轉換資料。 資料轉換後，您會使用 Apache Sqoop 將該資料載入 Azure SQL 資料庫中。 在本文中，您會使用可公開取用的航班資料。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是 Azure HDInsight 版本 3.4 或更新版本上唯一使用的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

本教學課程涵蓋下列工作： 

> [!div class="checklist"]
> * 下載範例航班資料
> * 將資料上傳至 HDInsight 叢集
> * 使用 Hive 轉換資料
> * 在 Azure SQL 資料庫中建立資料表
> * 使用 Sqoop 將資料匯出至 Azure SQL 資料庫


下圖顯示一般 ETL 應用程式流程。

![使用 Azure HDInsight 上的 Apache Hive 執行 ETL 作業](./media/hdinsight-analyze-flight-delay-data-linux/hdinsight-etl-architecture.png "使用 Azure HDInsight 上的 Apache Hive 執行 ETL 作業")

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

* **HDInsight 上的 Linux 型 Hadoop 叢集**。 請參閱[開始在 HDInsight 中使用 Hadoop](hadoop/apache-hadoop-linux-tutorial-get-started.md)，以取得如何建立新的 Linux 型 HDInsight 叢集的步驟。

* **Azure SQL Database**。 您會使用 Azure SQL Database 做為目的地資料存放區。 如果您沒有 SQL 資料庫，請參閱[在 Azure 入口網站中建立 Azure SQL Database](../sql-database/sql-database-get-started.md)。

* **Azure CLI**。 如果您尚未安裝 Azure CLI，請參閱[安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 以取得相關步驟。

* **SSH 用戶端**。 如需詳細資訊，請參閱[使用 SSH 連線至 HDInsight (Hadoop)](hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="download-the-flight-data"></a>下載航班資料

1. 瀏覽至[創新技術研究管理部運輸統計處][rita-website]。

2. 在此頁面上選取下列值：

   | 名稱 | 值 |
   | --- | --- |
   | 篩選年份 |2013 |
   | 篩選期間 |一月 |
   | 欄位 |Year、FlightDate、UniqueCarrier、Carrier、FlightNum、OriginAirportID、Origin、OriginCityName、OriginState、DestAirportID、Dest、DestCityName、DestState、DepDelayMinutes、ArrDelay、ArrDelayMinutes、CarrierDelay、WeatherDelay、NASDelay、SecurityDelay、LateAircraftDelay。 |
   清除所有其他欄位。 

3. 選取 [下載]。 您會取得含有您所選資料欄位的 .zip 檔案。

## <a name="upload-data-to-an-hdinsight-cluster"></a>將資料上傳至 HDInsight 叢集

有許多方法可將資料上傳至與 HDInsight 叢集相關聯的儲存體。 在本節中，您會使用 `scp` 來上傳資料。 若要了解其他上傳資料的方式，請參閱[將資料上傳至 HDInsight](hdinsight-upload-data.md)。

1. 開啟命令提示字元，並使用下列命令將 .zip 檔案上傳至 HDInsight 叢集前端節點：

    ```bash
    scp <FILENAME>.zip <SSH-USERNAME>@<CLUSTERNAME>-ssh.azurehdinsight.net:<FILENAME.zip>
    ```

    以 .zip 檔案名稱取代 FILENAME。 以 HDInsight 叢集的 SSH 登入取代 *USERNAME* 。 將 *CLUSTERNAME* 取代為 HDInsight 叢集的名稱。

   > [!NOTE]
   > 如果您使用密碼來驗證您的 SSH 登入，系統會提示您輸入密碼。 如果您使用的是公用金鑰，您可能必須使用 `-i` 參數並指定對應的私密金鑰路徑。 例如： `scp -i ~/.ssh/id_rsa FILENAME.zip USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:`。

2. 完成上傳之後，使用 SSH 連線至叢集。 在命令提示字元中輸入下列命令：

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

3. 使用以下命令解壓縮 .zip 檔案：

    ```bash
    unzip FILENAME.zip
    ```

    此命令會解壓縮一個 .csv 檔案 (約為 60MB)。

4. 使用下列命令在 HDInsight 儲存體上建立目錄，然後將 .csv 檔案複製到該目錄︰

    ```bash
    hdfs dfs -mkdir -p /tutorials/flightdelays/data
    hdfs dfs -put <FILENAME>.csv /tutorials/flightdelays/data/
    ```

## <a name="transform-data-using-a-hive-query"></a>使用 Hive 查詢轉換資料

有許多方法可在 HDInsight 叢集上執行 Hive 作業。 在本節中，您會使用 Beeline 來執行 Hive 作業。 若想了解其他用來執行 Hive 作業的方法，請參閱[使用 HDInsight 上的 Hive](./hadoop/hdinsight-use-hive.md)。

在執行 Hive 作業的過程中，您會將 .csv 檔案中的資料匯入至名為 **Delays** 的 Hive 資料表。

1. 從 HDInsight 叢集既有的 SSH 提示字元中，使用下列命令建立並編輯名為 **flightdelays.hql** 的新檔案：

    ```bash
    nano flightdelays.hql
    ```

2. 使用下列文字做為此檔案的內容：

    ```hiveql
    DROP TABLE delays_raw;
    -- Creates an external table over the csv file
    CREATE EXTERNAL TABLE delays_raw (
        YEAR string,
        FL_DATE string,
        UNIQUE_CARRIER string,
        CARRIER string,
        FL_NUM string,
        ORIGIN_AIRPORT_ID string,
        ORIGIN string,
        ORIGIN_CITY_NAME string,
        ORIGIN_CITY_NAME_TEMP string,
        ORIGIN_STATE_ABR string,
        DEST_AIRPORT_ID string,
        DEST string,
        DEST_CITY_NAME string,
        DEST_CITY_NAME_TEMP string,
        DEST_STATE_ABR string,
        DEP_DELAY_NEW float,
        ARR_DELAY_NEW float,
        CARRIER_DELAY float,
        WEATHER_DELAY float,
        NAS_DELAY float,
        SECURITY_DELAY float,
        LATE_AIRCRAFT_DELAY float)
    -- The following lines describe the format and location of the file
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
    LOCATION '/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays AS
    SELECT YEAR AS year,
        FL_DATE AS flight_date,
        substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier,
        substring(CARRIER, 2, length(CARRIER) -1) AS carrier,
        substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num,
        ORIGIN_AIRPORT_ID AS origin_airport_id,
        substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code,
        substring(ORIGIN_CITY_NAME, 2) AS origin_city_name,
        substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr,
        DEST_AIRPORT_ID AS dest_airport_id,
        substring(DEST, 2, length(DEST) -1) AS dest_airport_code,
        substring(DEST_CITY_NAME,2) AS dest_city_name,
        substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr,
        DEP_DELAY_NEW AS dep_delay_new,
        ARR_DELAY_NEW AS arr_delay_new,
        CARRIER_DELAY AS carrier_delay,
        WEATHER_DELAY AS weather_delay,
        NAS_DELAY AS nas_delay,
        SECURITY_DELAY AS security_delay,
        LATE_AIRCRAFT_DELAY AS late_aircraft_delay
    FROM delays_raw;
    ```

2. 若要儲存檔案，請按 **Esc**，然後輸入 `:x`。

3. 若要啟動 Hive 並執行 **flightdelays.hql** 檔案，請使用下列命令：

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
    ```

4. 在 __flightdelays.hql__ 指令碼執行完畢之後，請使用下列命令來開啟互動式 Beeline 工作階段：

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

5. 當您收到 `jdbc:hive2://localhost:10001/>` 提示字元時，請使用以下查詢從匯入的航班延誤資料中擷取資料：

    ```hiveql
    INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    SELECT regexp_replace(origin_city_name, '''', ''),
        avg(weather_delay)
    FROM delays
    WHERE weather_delay IS NOT NULL
    GROUP BY origin_city_name;
    ```

    此查詢會擷取因氣候因素而延誤的城市清單，以及平均延誤時間，並會儲存到 `/tutorials/flightdelays/output`。 稍後，Sqoop 會從此位置讀取該資料，並匯出到 Azure SQL Database。

6. 若要結束 Beeline，請在提示字元中輸入 `!quit` 。

## <a name="create-a-sql-database-table"></a>建立 SQL 資料庫資料表

本節假設您已建立事件 Azure SQL 資料庫。 如果您還沒有 SQL 資料庫，請使用[在 Azure 入口網站中建立 Azure SQL Database](../sql-database/sql-database-get-started.md) 中的資訊來建立一個資料庫。

如果您已經有 SQL 資料庫，就必須取得伺服器名稱。 若要在 [Azure 入口網站](https://portal.azure.com)中找到伺服器名稱，請選取 [SQL Database]，然後篩選您選擇使用的資料庫名稱。 伺服器名稱會列在 [伺服器名稱] 資料行中。

![取得 Azure SQL 伺服器詳細資料](./media/hdinsight-analyze-flight-delay-data-linux/get-azure-sql-server-details.png "取得 Azure SQL 伺服器詳細資料")

> [!NOTE]
> 連接至 SQL Database 並建立資料表的方法有很多種。 下列步驟會從 HDInsight 叢集使用 [FreeTDS](http://www.freetds.org/) 。


1. 若要安裝 FreeTDS，請從 SSH 連線對叢集使用下列命令：

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

3. 安裝完成後，請使用下列命令來連線到 SQL Database 伺服器。 使用 SQL Database 伺服器名稱取代 **serverName** 。 使用 SQL Database 的登入取代 **adminLogin** 和 **adminPassword**。 使用資料庫名稱取代 **databaseName** 。

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -p 1433 -D <databaseName>
    ```

    出現提示時，請輸入 SQL Database 管理員的登入密碼。

    您會收到如以下文字的輸出：

    ```
    locale is "en_US.UTF-8"
    locale charset is "UTF-8"
    using default charset "UTF-8"
    Default database being set to sqooptest
    1>
    ```

4. 在 `1>` 提示字元輸入下列幾行：

    ```hiveql
    CREATE TABLE [dbo].[delays](
    [origin_city_name] [nvarchar](50) NOT NULL,
    [weather_delay] float,
    CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED   
    ([origin_city_name] ASC))
    GO
    ```

    輸入 `GO` 陳述式後，將評估先前的陳述式。 此查詢會建立名為 **delays** 的資料表 (具有叢集索引)。

    使用下列查詢來確認已建立資料表：

    ```hiveql
    SELECT * FROM information_schema.tables
    GO
    ```

    輸出大致如下：

    ```
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    databaseName       dbo             delays        BASE TABLE
    ```

5. Enter `exit` at the `1>` 以結束 tsql 公用程式。

## <a name="export-data-to-sql-database-using-sqoop"></a>使用 Sqoop 將資料匯出至 SQL 資料庫

在前幾節中，您在 `/tutorials/flightdelays/output` 上複製了已轉換的資料。 在本節中，您會使用 Sqoop 將資料從 '/tutorials/flightdelays/output` 匯出至您在 Azure SQL 資料庫中建立的資料表。 

1. 使用下列命令以確認 Sqoop 看得見您的 SQL 資料庫：

    ```bash
    sqoop list-databases --connect jdbc:sqlserver://<serverName>.database.windows.net:1433 --username <adminLogin> --password <adminPassword>
    ```

    此命令會傳回一份資料庫清單，包含您稍早在其中建立 delays 資料表的資料庫。

2. 使用以下命令，將資料從 hivesampletable 匯出至延遲資料表：

    ```bash
    sqoop export --connect 'jdbc:sqlserver://<serverName>.database.windows.net:1433;database=<databaseName>' --username <adminLogin> --password <adminPassword> --table 'delays' --export-dir '/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
    ```

    Sqoop 會連線到包含 delays 資料表的資料庫，並將資料從 `/tutorials/flightdelays/output` 目錄匯出至 delays 資料表。

3. 在 sqoop 命令完成後，使用 tsql 公用程式連線至資料庫：

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D <databaseName>
    ```

    使用下列陳述式來確認資料已匯出到延遲資料表：

    ```sql
    SELECT * FROM delays
    GO
    ```

    您應會看到資料表中的資料清單。 此資料表包含城市名稱以及該城市的平均航班延誤時間。 

    輸入 `exit` 以結束 tsql 公用程式。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 HDInsight 中的 Apache Hadoop 叢集執行擷取、轉換和載入資料作業。 接著請進入下一個教學課程，了解如何使用 Azure Data Factory 隨需建立 HDInsight Handoop 叢集。

> [!div class="nextstepaction"]
>[使用 Azure Data Factory 在 HDInsight 中建立隨選 Handooop 叢集](hdinsight-hadoop-create-linux-clusters-adf.md)

若要了解更多 HDInsight 中資料的使用方式，請參閱下列文章：

* [教學課程：使用 Azure HDInsight 上的 Apache Hive 來擷取、轉換和載入資料](../storage/data-lake-storage/tutorial-extract-transform-load-hive.md)
* [搭配 HDInsight 使用 Hivet][hdinsight-use-hive]
* [搭配 HDInsight 使用 Pig][hdinsight-use-pig]
* [在 HDInsight 上開發 Hadoop 的 Java MapReduce 程式][hdinsight-develop-mapreduce]
* [開發 HDInsight 的 Python 串流處理 MapReduce 程式][hdinsight-develop-streaming]
* [搭配 HDInsight 使用 Oozie][hdinsight-use-oozie]
* [搭配 HDInsight 使用 Sqoop][hdinsight-use-sqoop]



[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/


[rita-website]: http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[hdinsight-use-oozie]: hdinsight-use-oozie-linux-mac.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-get-started]: hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-use-sqoop]:hadoop/apache-hadoop-use-sqoop-mac-linux.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
[hdinsight-develop-streaming]:hadoop/apache-hadoop-streaming-python.md
[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[hadoop-hiveql]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
