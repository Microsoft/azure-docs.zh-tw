---
title: 使用資料分析管線進行作業 - Azure
description: 設定及執行範例資料管線，其是由新資料所觸發並可產生精確的結果。
services: hdinsight
ms.service: hdinsight
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/11/2018
ms.openlocfilehash: 9057d9f5d63598ea249e8f3193b84fd715018829
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43109966"
---
# <a name="operationalize-a-data-analytics-pipeline"></a>使用資料分析管線進行作業

*資料管線*是許多資料分析解決方案的基礎。 正如其名，資料管線會依需求接受原始資料、清除並調整之，然後通常會在儲存已處理的資料前執行計算或彙總。 用戶端、報告或 API 會使用已處理的資料。 不論是依照排程，還是由新資料所觸發，資料管線都必須提供可重複的結果。

本文說明如何使用在 HDInsight Hadoop 叢集上執行的 Oozie，讓您的資料管線進行作業以實施重複性。 範例案例會帶領您了解資料管線如何準備並處理航班時間序列資料。

在下列案例中，輸入資料是內含一批一個月之航班資料的一般檔案。 此航班資料包括如起點和目的地機場、飛行英哩數、起飛和抵達時間等資訊。 此管線的目標是彙總每日航線表現，其中每個航線會有一個資料列來顯示每天的平均起飛與抵達延遲時間 (以分鐘為單位) 以及該日飛行的總英哩數。

| 年 | 月 | DAY_OF_MONTH | 航空公司 |AVG_DEP_DELAY | AVG_ARR_DELAY |TOTAL_DISTANCE |
| --- | --- | --- | --- | --- | --- | --- |
| 2017 | 1 | 3 | AA | 10.142229 | 7.862926 | 2644539 |
| 2017 | 1 | 3 | AS | 9.435449 | 5.482143 | 572289 |
| 2017 | 1 | 3 | DL | 6.935409 | -2.1893024 | 1909696 |

該範例管線會等到新時間週期的航班資料送達，才會將詳細的航班資訊儲存到您的 Hive 資料倉儲中，以供長期分析。 管線也會建立較小的資料集，該資料集只會彙總每日航班資料。 此每日航班摘要資料會傳送到 SQL 資料庫來提供報告，例如提供給網站。

下圖說明此範例管線。

![航班資料管線](./media/hdinsight-operationalize-data-pipeline/pipeline-overview.png)

## <a name="oozie-solution-overview"></a>Oozie 解決方案概觀

這個管線會使用在 HDInsight Hadoop 叢集上執行的 Apache Oozie。

Oozie 會根據「動作」、「工作流程和「協調器」來描述其管線。 動作會決定實際要執行的工作，例如執行 Hive 查詢。 工作流程會定義動作順序。 協調器會定義工作流程的執行排程。 協調器也會等到新資料可用時，再啟動工作流程的執行個體。

下圖顯示此範例 Oozie 管線的高階設計。

![Oozie 航班資料管線](./media/hdinsight-operationalize-data-pipeline/pipeline-overview-oozie.png)

### <a name="provision-azure-resources"></a>佈建 Azure 資源

此管線要求 Azure SQL Database 和 HDInsight Hadoop 叢集位於相同位置。 Azure SQL Database 會同時儲存管線和 Oozie 中繼資料存放區所產生的摘要資料。

#### <a name="provision-azure-sql-database"></a>佈建 Azure SQL Database

1. 使用 Azure 入口網站，建立名為 `oozie` 的新資源群組，以包含此範例使用的所有資源。
2. 在 `oozie` 資源群組中，佈建 Azure SQL Server 和 SQL Database。 您不需要大於 S1 標準定價層的資料庫。
3. 使用 Azure 入口網站，瀏覽至新部署 SQL Database 的窗格，然後選取 [工具]。

    ![工具按鈕](./media/hdinsight-operationalize-data-pipeline/sql-db-tools.png)

4. 選取 [查詢編輯器]。

    ![查詢編輯器按鈕](./media/hdinsight-operationalize-data-pipeline/sql-db-query-editor.png)

5. 在 [查詢編輯器] 窗格中，選取 [登入]。

    ![登入按鈕](./media/hdinsight-operationalize-data-pipeline/sql-db-login1.png)

6. 輸入您的 SQL Database 認證，並選取 [確定]。

   ![登入表單](./media/hdinsight-operationalize-data-pipeline/sql-db-login2.png)

7. 在查詢編輯器的文字區域中，輸入下列 SQL 陳述式來建立 `dailyflights` 資料表，該資料表將會儲存管線每個執行的彙總資料。

    ```
    CREATE TABLE dailyflights
    (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        CARRIER CHAR(2),
        AVG_DEP_DELAY FLOAT,
        AVG_ARR_DELAY FLOAT,
        TOTAL_DISTANCE FLOAT
    )
    GO

    CREATE CLUSTERED INDEX dailyflights_clustered_index on dailyflights(YEAR,MONTH,DAY_OF_MONTH,CARRIER)
    GO
    ```

8. 選取 [執行] 以執行 SQL 陳述式。

    ![執行按鈕](./media/hdinsight-operationalize-data-pipeline/sql-db-run.png)

您的 Azure SQL Database 現在已準備就緒。

#### <a name="provision-an-hdinsight-hadoop-cluster"></a>佈建 HDInsight Hadoop 叢集

1. 在 Azure 入口網站中，選取 [+新增]，然後搜尋 HDInsight。
2. 選取 [建立] 。
3. 在 [基本資料] 窗格中，提供叢集的唯一名稱並選擇您的 Azure 訂用帳戶。

    ![HDInsight 叢集名稱和訂用帳戶](./media/hdinsight-operationalize-data-pipeline/hdi-name-sub.png)

4. 在 [叢集類型] 窗格中，選取 [Hadoop] 叢集類型，[Linux] 作業系統和最新版的 HDInsight 叢集。 將 [叢集層] 保持為 [標準]。

    ![HDInsight 叢集類型](./media/hdinsight-operationalize-data-pipeline/hdi-cluster-type.png)

5. 選擇 [選取] 來套用您的叢集類型選取項目。
6. 提供登入密碼並從清單中選取您的 `oozie` 資源群組，以完成 [基本資料] 窗格，然後選取 [下一步]。

    ![HDInsight 基本資料窗格](./media/hdinsight-operationalize-data-pipeline/hdi-basics.png)

7. 在 [儲存體] 窗格中，將主要儲存體類型設為 [Azure 儲存體]，選取 [建立新項目]，並提供新的帳戶名稱。

    ![HDInsight 儲存體帳戶設定](./media/hdinsight-operationalize-data-pipeline/hdi-storage.png)

8. 針對 [中繼存放區設定]，在 [選取 Hive 的 SQL 資料庫] 下，選擇您先前建立的資料庫。

    ![HDInsight Hive 中繼存放區設定](./media/hdinsight-operationalize-data-pipeline/hdi-metastore-hive.png)

9. 選取 [驗證 SQL Database]。

    ![HDInsight Hive 中繼存放區驗證](./media/hdinsight-operationalize-data-pipeline/hdi-authenticate-sql.png)

10. 輸入 SQL 資料庫使用者名稱和密碼，然後選擇 [選取]。 

       ![HDInsight Hive 中繼存放區驗證登入](./media/hdinsight-operationalize-data-pipeline/hdi-authenticate-sql-login.png)

11. 回到 [中繼存放區設定] 窗格，選取 Oozie 中繼資料存放區的資料庫並如同先前一樣進行驗證。 

       ![HDInsight 中繼存放區設定](./media/hdinsight-operationalize-data-pipeline/hdi-metastore-settings.png)

12. 選取 [下一步] 。
13. 在 [摘要] 窗格中，選取 [建立] 來部署您的叢集。

### <a name="verify-ssh-tunneling-setup"></a>確認 SSH 通道安裝

若要使用 Oozie Web 主控台來檢視協調器和工作流程執行個體的狀態，請將 SSH 通道安裝到您的 HDInsight 叢集。 如需詳細資訊，請參閱 [SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)。

> [!NOTE]
> 您也可以使用 Chrome 搭配 [Foxy Proxy](https://getfoxyproxy.org/) 擴充功能，以瀏覽整個 SSH 通道的叢集 web 資源。 將其設定為透過在通道連接埠 9876 上的主機 `localhost` 來代理所有要求。 這個方法與適用於 Linux 的 Windows 子系統 (也就是在 Windows 10 上的 Bash) 相容。

1. 執行下列命令來將 SSH 通道開啟到您的叢集：

    ```
    ssh -C2qTnNf -D 9876 sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net
    ```

2. 藉由瀏覽至以下位址，在前端節點上瀏覽至 Ambari，以確認通道可運作：

    http://headnodehost:8080

3. 若要從 Ambari 內存取 [Oozie Web 主控台]，選取 [Oozie]、[快速連結]，然後選取 [Oozie Web 主控台]。

### <a name="configure-hive"></a>設定 Hive

1. 下載包含一個月航班資料的範例 CSV 檔案。 從 [HDInsight Github 存放庫](https://github.com/hdinsight/hdinsight-dev-guide)下載其 ZIP 檔案`2017-01-FlightData.zip`，並將它解壓縮為 CSV 檔案`2017-01-FlightData.csv`。 

2. 將此 CSV 檔案複製到 HDInsight 叢集所連結的 Azure 儲存體帳戶，並將它放在 `/example/data/flights` 資料夾中。

您可以在 `bash` 殼層工作階段中使用 SCP 來複製此檔案。

1. 使用 SCP，將檔案從本機電腦複製到 HDInsight 叢集前端節點的本機儲存體。

    ```bash
    scp ./2017-01-FlightData.csv sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net:2017-01-FlightData.csv
    ```

2. 使用 HDFS 命令，將檔案從前端節點的本機儲存體複製到 Azure 儲存體。

    ```bash
    hdfs dfs -put ./2017-01-FlightData.csv /example/data/flights/2017-01-FlightData.csv
    ```

此範例資料目前可供使用。 不過，管線需要兩個 Hive 資料表以供處理使用，一個用於內送資料 (`rawFlights`)，另一個用於彙總資料 (`flights`)。 如下所示，在 Ambari 中建立這些資料表。

1. 瀏覽到 [http://headnodehost:8080](http://headnodehost:8080) 以登入 Ambari。
2. 從服務清單中，選取 [Hive]。

    ![在 Ambari 中選取 Hive](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive.png)

3. 選取 Hive 檢視 2.0 標籤旁邊的 [移至檢視]。

    ![在 Ambari 中選取 Hive 檢視](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive-summary.png)

4. 在查詢文字區域中，貼上下列陳述式以建立 `rawFlights` 資料表。 `rawFlights` 資料表可為 Azure 儲存體中 `/example/data/flights` 資料夾內的 CSV 檔案提供讀時結構描述。 

    ```
    CREATE EXTERNAL TABLE IF NOT EXISTS rawflights (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        FL_DATE STRING,
        CARRIER STRING,
        FL_NUM STRING,
        ORIGIN STRING,
        DEST STRING,
        DEP_DELAY FLOAT,
        ARR_DELAY FLOAT,
        ACTUAL_ELAPSED_TIME FLOAT,
        DISTANCE FLOAT)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
    WITH SERDEPROPERTIES 
    (
        "separatorChar" = ",",
        "quoteChar"     = "\""
    ) 
    LOCATION '/example/data/flights'
    ```

5. 選取 [執行] 以建立資料表。

    ![Ambari 中的 Hive 查詢](./media/hdinsight-operationalize-data-pipeline/hdi-ambari-services-hive-query.png)

6. 若要建立 `flights` 資料表，請使用下列陳述式取代查詢文字區域中的文字。 `flights` 資料表是 Hive 管理的資料表，此表格會依據年、月和日將載入其中的資料進行分割。 此資料表將包含所有過去的航班資料，而最低資料粒度出現在每個航班一列的來源資料中。

    ```
    SET hive.exec.dynamic.partition.mode=nonstrict;

    CREATE TABLE flights
    (
        FL_DATE STRING,
        CARRIER STRING,
        FL_NUM STRING,
        ORIGIN STRING,
        DEST STRING,
        DEP_DELAY FLOAT,
        ARR_DELAY FLOAT,
        ACTUAL_ELAPSED_TIME FLOAT,
        DISTANCE FLOAT
    )
    PARTITIONED BY (YEAR INT, MONTH INT, DAY_OF_MONTH INT)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
    WITH SERDEPROPERTIES 
    (
        "separatorChar" = ",",
        "quoteChar"     = "\""
    );
    ```

7. 選取 [執行] 以建立資料表。

### <a name="create-the-oozie-workflow"></a>建立 Oozie 工作流程

管線通常會依給定的時間間隔分批處理資料。 在此情況下，管線會每日處理航班資料。 此方法可讓輸入 CSV 檔案每日、每週、每月或每年送達。

範例工作流程會以三個主要步驟每日處理航班資料：

1. 執行 Hive 查詢以從 `rawFlights` 資料表所表示的來源 CSV 檔案中擷取當天日期範圍的資料，並將資料插入 `flights` 資料表中。
2. 執行 Hive 查詢，以在當天的 Hive 中動態建立暫存資料表，其中包含一份依照日期與航空公司彙總的航班資料。
3. 使用 Apache Sqoop 將所有資料從 Hive 中的每日暫存資料表複製到 Azure SQL Database 中的目的地 `dailyflights` 資料表。 Sqoop 會從 Azure 儲存體中 Hive 資料表背後的資料中讀取來源資料列，並使用 JDBC 連線將它們載入 SQL Database 中。

Oozie 工作流程會協調這三個步驟。 

1. 在 `hive-load-flights-partition.hql` 檔案中建立查詢。

    ```
    SET hive.exec.dynamic.partition.mode=nonstrict;
    
    INSERT OVERWRITE TABLE flights
    PARTITION (YEAR, MONTH, DAY_OF_MONTH)
    SELECT  
        FL_DATE,
        CARRIER,
        FL_NUM,
        ORIGIN,
        DEST,
        DEP_DELAY,
        ARR_DELAY,
        ACTUAL_ELAPSED_TIME,
        DISTANCE,
        YEAR,
        MONTH,
        DAY_OF_MONTH
    FROM rawflights
    WHERE year = ${year} AND month = ${month} AND day_of_month = ${day};
    ```

    Oozie 變數會使用 `${variableName}` 語法。 如後續步驟所述，這些變數設定於 `job.properties` 檔案中。 Oozie 會在執行階段替換實際值。

2. 在 `hive-create-daily-summary-table.hql` 檔案中建立查詢。

    ```
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}
    (
        YEAR INT,
        MONTH INT,
        DAY_OF_MONTH INT,
        CARRIER STRING,
        AVG_DEP_DELAY FLOAT,
        AVG_ARR_DELAY FLOAT,
        TOTAL_DISTANCE FLOAT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName}
    SELECT  year, month, day_of_month, carrier, avg(dep_delay) avg_dep_delay, 
            avg(arr_delay) avg_arr_delay, sum(distance) total_distance 
    FROM flights
    GROUP BY year, month, day_of_month, carrier 
    HAVING year = ${year} AND month = ${month} AND day_of_month = ${day};
    ```

    此查詢會建立僅儲存彙總資料一天的暫存資料表，請注意 SELECT 陳述式會依航空公司和日期計算平均延遲時間和總飛行距離。 插入此資料表的資料會儲存在已知的位置 (hiveDataFolder 變數所指示的路徑)，以便在下一個步驟中作為 Sqoop 的來源。

3. 執行下列 Sqoop 命令。

    ```
    sqoop export --connect ${sqlDatabaseConnectionString} --table ${sqlDatabaseTableName} --export-dir ${hiveDataFolder} -m 1 --input-fields-terminated-by "\t"
    ```

在下列 Oozie 工作流程檔案 (名為 `workflow.xml`) 中，這三個步驟會以三個不同的動作表示。

```
<workflow-app name="loadflightstable" xmlns="uri:oozie:workflow:0.5">
    <start to = "RunHiveLoadFlightsScript"/>
    <action name="RunHiveLoadFlightsScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScriptLoadPartition}</script>
            <param>year=${year}</param>
            <param>month=${month}</param>
            <param>day=${day}</param>
        </hive>
        <ok to="RunHiveCreateDailyFlightTableScript"/>
        <error to="fail"/>
    </action>

    <action name="RunHiveCreateDailyFlightTableScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScriptCreateDailyTable}</script>
            <param>hiveTableName=${hiveDailyTableName}</param>
            <param>year=${year}</param>
            <param>month=${month}</param>
            <param>day=${day}</param>
            <param>hiveDataFolder=${hiveDataFolder}/${year}/${month}/${day}</param>
        </hive>
        <ok to="RunSqoopExport"/>
        <error to="fail"/>
    </action>

    <action name="RunSqoopExport">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.compress.map.output</name>
                <value>true</value>
            </property>
            </configuration>
            <arg>export</arg>
            <arg>--connect</arg>
            <arg>${sqlDatabaseConnectionString}</arg>
            <arg>--table</arg>
            <arg>${sqlDatabaseTableName}</arg>
            <arg>--export-dir</arg>
            <arg>${hiveDataFolder}/${year}/${month}/${day}</arg>
            <arg>-m</arg>
            <arg>1</arg>
            <arg>--input-fields-terminated-by</arg>
            <arg>"\t"</arg>
            <archive>sqljdbc41.jar</archive>
            </sqoop>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    <kill name="fail">
        <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
    </kill>
    <end name="end"/>
</workflow-app>
```

兩個 Hive 查詢可透過其在 Azure 儲存體中的路徑存取，而其餘的變數值則是由下列 `job.properties` 檔案所提供。 此檔案會將工作流程為設定為在 2017 年 1 月 3 日執行。

```
nameNode=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net
jobTracker=hn0-[CLUSTERNAME].[UNIQUESTRING].dx.internal.cloudapp.net:8050
queueName=default
oozie.use.system.libpath=true
appBase=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie
oozie.wf.application.path=${appBase}/load_flights_by_day
hiveScriptLoadPartition=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-load-flights-partition.hql
hiveScriptCreateDailyTable=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-create-daily-summary-table.hql
hiveDailyTableName=dailyflights${year}${month}${day}
hiveDataFolder=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/day/${year}/${month}/${day}
sqlDatabaseConnectionString="jdbc:sqlserver://[SERVERNAME].database.windows.net;user=[USERNAME];password=[PASSWORD];database=[DATABASENAME]"
sqlDatabaseTableName=dailyflights
year=2017
month=01
day=03
```

下表摘要說明每個屬性，並指示您可以在何處找到適合您環境的值。

| 屬性 | 值來源 |
| --- | --- |
| nameNode | HDInsight 叢集所連結 Azure 儲存體容器的完整路徑。 |
| jobTracker | 作用中叢集之 YARN 前端節點的內部主機名稱。 在 Ambari 首頁上，從服務清單中選取 YARN，然後選擇 [作用中資源管理員]。 主機名稱 URI 會顯示在頁面頂端。 附加連接埠 8050。 |
| queueName | 安排 Hive 動作時所使用的 YARN 佇列名稱。 保持為預設值 |
| oozie.use.system.libpath | 保持為 true。 |
| appBase | 在 Azure 儲存體中子資料夾的路徑，您會在此資料夾中部署 Oozie 工作流程和支援檔案。 |
| oozie.wf.application.path | 要執行之 Oozie 工作流程 `workflow.xml` 的位置。 |
| hiveScriptLoadPartition | 在 Azure 儲存體中 Hive 查詢檔案 `hive-load-flights-partition.hql` 的路徑。 |
| hiveScriptCreateDailyTable | 在 Azure 儲存體中 Hive 查詢檔案 `hive-create-daily-summary-table.hql` 的路徑。 |
| hiveDailyTableName | 要用於暫存資料表的動態產生名稱。 |
| hiveDataFolder | 在 Azure 儲存體中暫存資料表內含資料的路徑。 |
| sqlDatabaseConnectionString | Azure SQL Database 的 JDBC 語法連接字串。 |
| sqlDatabaseTableName | 在 Azure SQL Database 中摘要資料列會插入其中的資料表名稱。 保持為 `dailyflights`。 |
| 年 | 為其計算航班摘要之日期的年度元件。 保持原狀。 |
| 月 | 為其計算航班摘要之日期的月份元件。 保持原狀。 |
| day | 為其計算航班摘要之日期的日期元件。 保持原狀。 |

> [!NOTE]
> 務必先使用您環境的特定值來更新您的 `job.properties` 檔案，才可以部署和執行 Oozie 工作流程。

### <a name="deploy-and-run-the-oozie-workflow"></a>部署和執行 Oozie 工作流程

使用 bash 工作階段中的 SCP，部署 Oozie 工作流程 (`workflow.xml`)、Hive 查詢 (`hive-load-flights-partition.hql` 和 `hive-create-daily-summary-table.hql`) 和作業組態 (`job.properties`)。  在 Oozie 中，前端節點的本機儲存體上只能存在 `job.properties` 檔案。 所有其他檔案都必須都儲存在 HDFS 中，在此情況下為 Azure 儲存體。 工作流程所使用的 Sqoop 動作依賴 JDBC 驅動程式來與 SQL Database 進行通訊，該驅動程式必須從前端節點複製到 HDFS。

1. 在前端節點的本機儲存體中，與使用者的路徑底下建立 `load_flights_by_day` 子資料夾。

        ssh sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net 'mkdir load_flights_by_day'

2. 將目前目錄中的所有檔案 (`workflow.xml` 和 `job.properties` 檔案) 複製到 `load_flights_by_day` 子資料夾。

        scp ./* sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net:load_flights_by_day

3. 透過 SSH 連入前端節點並瀏覽至 `load_flights_by_day` 資料夾。

        ssh sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net
        cd load_flights_by_day

4. 將工作流程檔案複製到 HDFS。

        hdfs dfs -put ./* /oozie/load_flights_by_day

5. 將 `sqljdbc41.jar` 從本機前端節點複製到 HDFS 中的工作流程資料夾：

        hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc*.jar /oozie/load_flights_by_day

6. 執行工作流程。

        oozie job -config job.properties -run

7. 使用 Oozie Web 主控台觀察狀態。 從 Ambari 內選取 [Oozie]、[快速連結]，然後選取 [Oozie Web 主控台]。 在 [工作流程作業] 索引標籤底下，選取 [所有作業]。

    ![Oozie Web 主控台工作流程](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-workflows.png)

8. 當狀態為 [已成功]，請查詢 SQL 資料庫資料表來檢視插入的資料列。 使用 Azure 入口網站，瀏覽您的 SQL Database 窗格，選取 [工具]，然後開啟 [查詢編輯器]。

        SELECT * FROM dailyflights

現在，工作流程會為單一測試日執行，您可使用可安排工作流程的協調器來包裝該工作流程，使其每天執行。

### <a name="run-the-workflow-with-a-coordinator"></a>使用協調器執行工作流程

若要安排此工作流程，使其每天執行 (或在日期範圍內的每一天)，您可以使用協調器。 協調器是由 XML 檔案 (例如 `coordinator.xml`) 所定義：

```
<coordinator-app name="daily_export" start="2017-01-01T00:00Z" end="2017-01-05T00:00Z" frequency="${coord:days(1)}" timezone="UTC" xmlns="uri:oozie:coordinator:0.4">
    <datasets>
        <dataset name="ds_input1" frequency="${coord:days(1)}" initial-instance="2016-12-31T00:00Z" timezone="UTC">
            <uri-template>${sourceDataFolder}${YEAR}-${MONTH}-FlightData.csv</uri-template>
            <done-flag></done-flag>
        </dataset>
    </datasets>
    <input-events>
        <data-in name="event_input1" dataset="ds_input1">
            <instance>${coord:current(0)}</instance>
        </data-in>
    </input-events>
    <action>
        <workflow>
            <app-path>${appBase}/load_flights_by_day</app-path>
            <configuration>
                <property>
                    <name>year</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'yyyy')}</value>
                </property>
                <property>
                    <name>month</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'MM')}</value>
                </property>
                <property>
                    <name>day</name>
                    <value>${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>hiveScriptLoadPartition</name>
                    <value>${hiveScriptLoadPartition}</value>
                </property>
                <property>
                    <name>hiveScriptCreateDailyTable</name>
                    <value>${hiveScriptCreateDailyTable}</value>
                </property>
                <property>
                    <name>hiveDailyTableNamePrefix</name>
                    <value>${hiveDailyTableNamePrefix}</value>
                </property>
                <property>
                    <name>hiveDailyTableName</name>
                    <value>${hiveDailyTableNamePrefix}${coord:formatTime(coord:nominalTime(), 'yyyy')}${coord:formatTime(coord:nominalTime(), 'MM')}${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>hiveDataFolderPrefix</name>
                    <value>${hiveDataFolderPrefix}</value>
                </property>
                <property>
                    <name>hiveDataFolder</name>
                    <value>${hiveDataFolderPrefix}${coord:formatTime(coord:nominalTime(), 'yyyy')}/${coord:formatTime(coord:nominalTime(), 'MM')}/${coord:formatTime(coord:nominalTime(), 'dd')}</value>
                </property>
                <property>
                    <name>sqlDatabaseConnectionString</name>
                    <value>${sqlDatabaseConnectionString}</value>
                </property>
                <property>
                    <name>sqlDatabaseTableName</name>
                    <value>${sqlDatabaseTableName}</value>
                </property>
            </configuration>
        </workflow>
    </action>
</coordinator-app>
```

如您所見，大部分的協調器只會將組態資訊傳遞至工作流程執行個體。 不過，有幾個重要項目需要留意。

* 第 1 點：`coordinator-app` 元素本身的 `start` 和 `end` 屬性會控制協調器執行的時間間隔。

    ```
    <coordinator-app ... start="2017-01-01T00:00Z" end="2017-01-05T00:00Z" frequency="${coord:days(1)}" ...>
    ```

    根據 `frequency` 屬性所指定的間隔，協調器會負責排定 `start` 和 `end` 日期範圍內的動作。 每個排定的動作會如同設定依序執行工作流程。 在上述協調器定義中，協調器設定為從 2017 年 1 月 1 日到 2017 年 1 月 5 日執行動作。 [Oozie 運算式語言](http://oozie.apache.org/docs/4.2.0/CoordinatorFunctionalSpec.html#a4.4._Frequency_and_Time-Period_Representation) 頻率運算式 `${coord:days(1)}`會將頻率設定為 1 天。 這會導致協調器每天安排一次動作 (還有工作流程)。 對於過去的日期範圍，就如同此範例中，此動作會立即排定進行執行。 排定要執行動作之日期的開端稱為「名義時間」。 比方說，若要處理 2017 年 1 月 1 日的資料，協調器會安排名義時間為 2017-01-01T00:00:00 GMT 的動作。

* 第 2 點：在工作流程的日期範圍內，`dataset` 元素會指定在 HDFS 中何處尋找特定日期範圍的資料，以及設定 Oozie 如何判斷資料是否已可供處理。

    ```
    <dataset name="ds_input1" frequency="${coord:days(1)}" initial-instance="2016-12-31T00:00Z" timezone="UTC">
        <uri-template>${sourceDataFolder}${YEAR}-${MONTH}-FlightData.csv</uri-template>
        <done-flag></done-flag>
    </dataset>
    ```

    根據 `uri-template` 元素中所提供的運算式，系統會在 HDFS 中動態建置資料的路徑。 在此協調器中，一天的頻率也可搭配資料集使用。 協調器元素的開始和結束日期會控制排定動作的時間 (並定義其名義時間)，而資料集的 `initial-instance` 和 `frequency` 會控制用來建構 `uri-template` 之日期的計算。 在此情況下，將初始執行個體設定為協調器開始前一天，以確保其會取得第一天 (1/1/2017) 的資料量。 資料集的日期計算會從 `initial-instance` 的值 (12/31/2016) 開始向前延展，並以資料集頻率 (1 天) 的增量向前推進，直到其找到未超過協調器所設定之名義時間的最近日期為止 (第一個動作為 2017-01-01T00:00:00 GMT)。

    空白 `done-flag` 元素表示當 Oozie 在約定的時間檢查輸入資料是否存在時，Oozie 會由目錄或檔案的存在與否決定資料是否可用。 在此情況下是 csv 檔案的存在與否。 如果 csv 檔案存在，則 Oozie 會假設資料已就緒，而且會啟動工作流程執行個體來處理檔案。 如果不存在任何 csv 檔案，則 Oozie 會假設資料尚未就緒，而且該工作流程執行會進入等候狀態。

* 第 3 點：`data-in` 元素在取代 `uri-template` 中相關聯資料集的值時，會指定特定時間戳記作為名義時間。

    ```
    <data-in name="event_input1" dataset="ds_input1">
        <instance>${coord:current(0)}</instance>
    </data-in>
    ```

    在此情況下，將執行個體設定為運算式 `${coord:current(0)}`，該運算式會轉譯為如協調器原先所排定的方式使用動作的名義時間。 換句話說，當協調器使用名義時間 01/01/2017 來安排要執行的動作時，01/01/2017 就會用來取代 URI 範本中的 YEAR (2017) 和 MONTH (01) 變數。 為此執行個體計算 URI 範本後，Oozie 會檢查預期的目錄或檔案是否可用，並據此安排下一次的工作流程執行。

前三點合起來即會產生一個狀況：協調器會逐日安排來源資料的處理。 

* 第 1 點：協調器會從名義日期 2017-01-01 開始。

* 第 2 點：Oozie 會尋找 `sourceDataFolder/2017-01-FlightData.csv` 中可用的資料。

* 第 3 點：當 Oozie 找到該檔案時，它會安排一個工作流程執行個體來處理 2017-01-01 的資料。 Oozie 會接著繼續處理 2017-01-02 的資料。 此評估會重複直到 (但不包括) 2017-01-05 為止。

如同工作流程，協調器的組態定義於 `job.properties` 檔案中，該檔案中有工作流程所使用的設定超集合。

```
nameNode=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net
jobTracker=hn0-[CLUSTERNAME].[UNIQUESTRING].dx.internal.cloudapp.net:8050
queueName=default
oozie.use.system.libpath=true
appBase=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie
oozie.coord.application.path=${appBase}
sourceDataFolder=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/
hiveScriptLoadPartition=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-load-flights-partition.hql
hiveScriptCreateDailyTable=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/oozie/load_flights_by_day/hive-create-daily-summary-table.hql
hiveDailyTableNamePrefix=dailyflights
hiveDataFolderPrefix=wasbs://[CONTAINERNAME]@[ACCOUNTNAME].blob.core.windows.net/example/data/flights/day/
sqlDatabaseConnectionString="jdbc:sqlserver://[SERVERNAME].database.windows.net;user=[USERNAME];password=[PASSWORD];database=[DATABASENAME]"
sqlDatabaseTableName=dailyflights

```

在此 `job.properties` 檔案中引入的新屬性只有：

| 屬性 | 值來源 |
| --- | --- |
| oozie.coord.application.path | 表示包含要執行之 Oozie 協調器的 `coordinator.xml` 檔案位置。 |
| hiveDailyTableNamePrefix | 動態建立暫存資料表的資料表名稱時所使用的前置詞。 |
| hiveDataFolderPrefix | 將儲存所有暫存資料表之路徑的前置詞。 |

### <a name="deploy-and-run-the-oozie-coordinator"></a>部署和執行 Oozie 協調器

若要使用協調員執行管線，請以類似的方式繼續處理工作流程 (從包含工作流程之資料夾的上一層資料夾進行處理時除外)。 此資料夾慣例會將協調器與磁碟上的工作流程隔開，以便您將一個協調者與不同的子工作流程建立關聯。

1. 使用 SCP，將協調器檔案從本機電腦複製到叢集前端節點的本機儲存體。

    ```bash
    scp ./* sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net:~
    ```

2. 透過 SSH 連入前端節點。

    ```bash
    ssh sshuser@[CLUSTERNAME]-ssh.azurehdinsight.net 
    ```

3. 將協調器檔案複製到 HDFS。

    ```bash
    hdfs dfs -put ./* /oozie/
    ```

4. 執行協調器。

    ```bash
    oozie job -config job.properties -run
    ```

5. 使用 Oozie Web 主控台確認狀態，此次選取 [協調器作業] 索引標籤，然後選取 [所有作業]。

    ![Oozie Web 主控台協調器作業](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-coordinator-jobs.png)

6. 選取協調器執行個體，以顯示已排定的動作清單。 在此情況下，您應會看到有四個動作的名義時間落在從 1/1/2017 到 1/4/2017 的範圍內。

    ![Oozie Web 主控台協調器作業](./media/hdinsight-operationalize-data-pipeline/hdi-oozie-web-console-coordinator-instance.png)

    這份清單中的每個動作都會對應至一個工作流程執行個體，該執行個體可處理一天的資料量，其中是以名義時間所表示該日的開始。

## <a name="next-steps"></a>後續步驟

* [Apache Oozie 文件](http://oozie.apache.org/docs/4.2.0/index.html)

<!-- * Build the same pipeline [using Azure Data Factory](tbd.md).  -->
