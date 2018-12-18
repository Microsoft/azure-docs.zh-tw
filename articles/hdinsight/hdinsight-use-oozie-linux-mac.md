---
title: 在 Linux 型 Azure HDInsight 上使用 Hadoop Oozie 工作流程
description: 在 Linux 型 HDInsight 上使用 Hadoop Oozie。 了解如何定義 Oozie 工作流程，以及提交 Oozie 作業。
services: hdinsight
ms.service: hdinsight
ms.custom: hdinsightactive
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 06/26/2018
ms.openlocfilehash: 052afbe185f9c66af3759775b38b6bf0f2c2e6b2
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783316"
---
# <a name="use-oozie-with-hadoop-to-define-and-run-a-workflow-on-linux-based-azure-hdinsight"></a>在 Linux 型 Azure HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程

[!INCLUDE [oozie-selector](../../includes/hdinsight-oozie-selector.md)]

了解如何在 Azure HDInsight 上搭配 Hadoop 使用 Apache Oozie。 Oozie 是可管理 Hadoop 作業的工作流程和協調系統。 Oozie 已與 Hadoop 堆疊整合，並支援下列作業：

* Apache MapReduce
* Apache Pig
* Apache Hive
* Apache Sqoop

您也可以使用 Oozie 來排程系統的特定作業，例如 Java 程式或 Shell 指令碼。

> [!NOTE]
> 還有另一個選項可以定義與 HDInsight 搭配的工作流程，那就是 Azure Data Factory。 若要深入了解 Data Factory，請參閱[搭配 Data Factory 使用 Pig 和 Hive][azure-data-factory-pig-hive]。


## <a name="prerequisites"></a>必要條件

* **HDInsight 叢集**：請參閱 [開始使用 Linux 上的 HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是 HDInsight 版本 3.4 或更新版本上唯一使用的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="example-workflow"></a>範例工作流程

此文件中使用的工作流程包含兩個動作。 動作就是工作的定義，例如執行 Hive、Sqoop、MapReduce 或其他處理序：

![Workflow diagram][img-workflow-diagram]

1. Hive 動作會執行 HiveQL 指令碼，以從 HDInsight 隨附的 **hivesampletable** 擷取記錄。 每個資料列均代表來自特定行動裝置的瀏覽。 顯示的記錄格式類似下列文字：

        8       18:54:20        en-US   Android Samsung SCH-i500        California     United States    13.9204007      0       0
        23      19:19:44        en-US   Android HTC     Incredible      Pennsylvania   United States    NULL    0       0
        23      19:19:46        en-US   Android HTC     Incredible      Pennsylvania   United States    1.4757422       0       1

    本文件中使用的 Hive 指令碼會計算每個平台 (例如 Android 或 iPhone) 的總瀏覽次數，並將計數儲存到新的 Hive 資料表。

    如需 Hive 的詳細資訊，請參閱[在 HDInsight 上使用 Hive][hdinsight-use-hive]。

2. Sqoop 動作會將新的 Hive 資料表內容匯出至 Azure SQL Database 中建立的資料表。 如需 Sqoop 的詳細資訊，請參閱[搭配 HDInsight 使用 Hadoop Sqoop][hdinsight-use-sqoop]。

> [!NOTE]
> 如需 HDInsight 叢集支援的 Oozie 版本，請參閱 [HDInsight 提供之 Hadoop 叢集版本中的新增功能][hdinsight-versions]。

## <a name="create-the-working-directory"></a>建立工作目錄

Oozie 預計您會將作業所需的所有資源儲存在同一個目錄中。 這個範例會使用 **wasb:///tutorials/useoozie**。 若要建立此目錄，請完成下列步驟：

1. 使用 SSH 連線到 HDInsight 叢集：

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

    將 `sshuser` 取代為叢集的 SSH 使用者名稱。 將 `clustername` 取代為叢集的名稱。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 若要建立目錄，請使用下列命令：

    ```bash
    hdfs dfs -mkdir -p /tutorials/useoozie/data
    ```

    > [!NOTE]
    > `-p` 參數會使系統建立路徑中的所有目錄。 **data** 目錄是用來保存 **useooziewf.hql** 指令碼所使用的資料。

3. 為了確保 Oozie 可以模擬您的使用者帳戶，請使用下列命令：

    ```bash
    sudo adduser username users
    ```

    將 `username` 取代為您的 SSH 使用者名稱。

    > [!NOTE]
    > 您可以忽略指出使用者已是 `users` 群組之成員的錯誤。

## <a name="add-a-database-driver"></a>新增資料庫驅動程式

由於此工作流程會使用 Sqoop 將資料匯出至 SQL 資料庫，因此您必須提供一份要用來與 SQL 資料庫互動的 JDBC 驅動程式複本。 若要將 JDBC 驅動程式複製到工作目錄，請從 SSH 工作階段使用下列命令：

```bash
hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc*.jar /tutorials/useoozie/
```

> [!NOTE]
> 您可能會收到指出檔案已存在的訊息。

如果工作流程已使用其他資源，例如含有 MapReduce 應用程式的 jar，則您也必須新增這些資源。

## <a name="define-the-hive-query"></a>定義 Hive 查詢

請使用下列步驟來建立 Hive 查詢語言 (HiveQL) 指令碼，以定義查詢。 您將會在本文稍後的 Oozie 工作流程中使用此查詢。

1. 從 SSH 連線，使用下列命令來建立名為 `useooziewf.hql` 的檔案：

    ```bash
    nano useooziewf.hql
    ```

3. 在 GNU nano 編輯器開啟後，請使用下列查詢做為檔案的內容：

    ```hiveql
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}(deviceplatform string, count string) ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName} SELECT deviceplatform, COUNT(*) as count FROM hivesampletable GROUP BY deviceplatform;
    ```

    指令碼中使用了兩個變數：

    * `${hiveTableName}`：包含要建立的資料表名稱。

    * `${hiveDataFolder}`：包含要儲存資料表之資料檔案的位置。

    工作流程定義檔 (在此教學課程中為 workflow.xml) 會在執行階段將這些值傳遞至此 HiveQL 指令碼。

4. 若要結束編輯器，請選取 Ctrl-X。 出現提示時，選取 `Y` 以儲存檔案，輸入 `useooziewf.hql` 做為檔案名稱，然後選取 **Enter**。

5. 使用下列命令將 `useooziewf.hql` 複製到 `wasb:///tutorials/useoozie/useooziewf.hql`：

    ```bash
    hdfs dfs -put useooziewf.hql /tutorials/useoozie/useooziewf.hql
    ```

    此命令會將 `useooziewf.hql` 檔案儲存在叢集的 HDFS 相容儲存體中。

## <a name="define-the-workflow"></a>定義工作流程

Oozie 工作流程定義是以 Hadoop 流程定義語言 (hPDL)，也就是 XML 流程定義語言所撰寫的。 使用下列步驟定義工作流程：

1. 使用以下陳述式建立和編輯新的檔案：

    ```bash
    nano workflow.xml
    ```

2. 在 nano 編輯器開啟後，請輸入下列 XML 做為檔案內容：

    ```xml
    <workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
        <start to = "RunHiveScript"/>
        <action name="RunHiveScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScript}</script>
            <param>hiveTableName=${hiveTableName}</param>
            <param>hiveDataFolder=${hiveDataFolder}</param>
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
            <arg>${hiveDataFolder}</arg>
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

    此工作流程中定義了兩個動作：

   * `RunHiveScript`：這是起始動作，會執行 `useooziewf.hql` Hive 指令碼。

   * `RunSqoopExport`：此動作會使用 Sqoop 將從 Hive 指令碼建立的資料匯出到 SQL 資料庫。 只有當 `RunHiveScript` 動作成功時，才會執行此動作。

     工作流程有數個項目，例如 `${jobTracker}`。 您會以您在作業定義中使用的值取代這些項目。 您將會在本文稍後建立作業定義。

     此外也請注意 Sqoop 區段中的 `<archive>sqljdbc4.jar</archive>` 項目。 此項目會指示 Oozie 在此動作執行時，讓 Sqoop 可取得此封存。

3. 若要儲存檔案，請選取 Ctrl+X、輸入 `Y`，然後選取 **Enter**。 

4. 使用下列命令將 `workflow.xml` 檔案複製到 `/tutorials/useoozie/workflow.xml`：

    ```bash
    hdfs dfs -put workflow.xml /tutorials/useoozie/workflow.xml
    ```

## <a name="create-the-database"></a>建立資料庫

若要建立 SQL 資料庫，請依照[建立 SQL 資料庫](../sql-database/sql-database-get-started.md)文件中的步驟進行。 在建立資料庫時，請使用 `oozietest` 做為資料庫名稱。 也請記下資料庫伺服器的名稱。

### <a name="create-the-table"></a>建立資料表

> [!NOTE]
> 連接至 SQL Database 建立資料表的方法有很多種。 下列步驟會從 HDInsight 叢集使用 [FreeTDS](http://www.freetds.org/) 。


1. 使用以下命令在 HDInsight 叢集上安裝 FreeTDS：

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. 在 FreeTDS 安裝好之後，請使用下列命令來連接至先前建立的 SQL 資料庫伺服器：

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <sqlLogin> -P <sqlPassword> -p 1433 -D oozietest
    ```

    您會收到如下列文字所示的輸出：

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to oozietest
        1>

3. 在 `1>` 提示字元輸入下列幾行：

    ```sql
    CREATE TABLE [dbo].[mobiledata](
    [deviceplatform] [nvarchar](50),
    [count] [bigint])
    GO
    CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(deviceplatform)
    GO
    ```

    輸入 `GO` 陳述式後，將評估先前的陳述式。 這些陳述式會建立工作流程所使用，名為 **mobiledata** 的資料表。

    若要確認資料表是否已建立，請使用下列命令：

    ```sql
    SELECT * FROM information_schema.tables
    GO
    ```

    您會看到如下列文字所示的輸出：

        TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
        oozietest       dbo     mobiledata      BASE TABLE

4. 若要結束 tsql 公用程式，請在 `1>` 提示字元處輸入 `exit`。

## <a name="create-the-job-definition"></a>建立工作定義

工作定義描述 workflow.xml 位於何處。 它也描述工作流程使用的其他檔案 (例如 `useooziewf.hql`) 位於何處。 此外，它也會定義工作流程中所使用的屬性值，以及相關聯的檔案。

1. 若要取得預設儲存體的完整位址，請使用下列命令。 此位址會用在您於下一個步驟建立的組態檔中。

    ```bash
    sed -n '/<name>fs.default/,/<\/value>/p' /etc/hadoop/conf/core-site.xml
    ```

    此命令會傳回類似以下 XML 的資訊：

    ```xml
    <name>fs.defaultFS</name>
    <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net</value>
    ```

    > [!NOTE]
    > 若 HDInsight 叢集使用 Azure 儲存體做為預設儲存體，`<value>` 元素內容的開頭將會是 `wasb://`。 若改為使用 Azure Data Lake Store，則其開頭將會是 `adl://`。

    儲存 `<value>` 元素的內容，因為在後續步驟將會用到它。

2. 若要建立 Oozie 作業定義組態，請使用下列命令：

    ```bash
    nano job.xml
    ```

3. 在 nano 編輯器開啟後，請使用下列 XML 做為檔案的內容：

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <property>
        <name>nameNode</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net</value>
        </property>

        <property>
        <name>jobTracker</name>
        <value>headnodehost:8050</value>
        </property>

        <property>
        <name>queueName</name>
        <value>default</value>
        </property>

        <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
        </property>

        <property>
        <name>hiveScript</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/useooziewf.hql</value>
        </property>

        <property>
        <name>hiveTableName</name>
        <value>mobilecount</value>
        </property>

        <property>
        <name>hiveDataFolder</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/data</value>
        </property>

        <property>
        <name>sqlDatabaseConnectionString</name>
        <value>"jdbc:sqlserver://serverName.database.windows.net;user=adminLogin;password=adminPassword;database=oozietest"</value>
        </property>

        <property>
        <name>sqlDatabaseTableName</name>
        <value>mobiledata</value>
        </property>

        <property>
        <name>user.name</name>
        <value>YourName</value>
        </property>

        <property>
        <name>oozie.wf.application.path</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
    </configuration>
    ```

   * 將 `wasb://mycontainer@mystorageaccount.blob.core.windows.net` 的所有執行個體取代為您之前收到的預設儲存體值。

     > [!WARNING]
     > 若該路徑是 `wasb` 路徑，您必須使用完整路徑。 不要將它縮短為 `wasb:///`。

   * 將 `YourName` 取代為您的 HDInsight 叢集登入名稱。
   * 將 `serverName``adminLogin` 及 `adminPassword` 取代為您 SQL 資料庫的資訊。

     此檔案中大部分的資訊都會用來填入 workflow.xml 或 ooziewf.hql 檔案中所使用的值 (例如 `${nameNode}`)。

     > [!NOTE]
     > `oozie.wf.application.path` 項目會定義用來尋找 workflow.xml 檔案的位置。 此檔案包含此作業所執行的工作流程。

5. 若要儲存檔案，請選取 Ctrl+X、輸入 `Y`，然後選取 **Enter**。

## <a name="submit-and-manage-the-job"></a>提交和管理工作

下列步驟使用 Oozie 命令提交及管理叢集上的 Oozie 工作流程。 Oozie 命令是 [Oozie REST API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html)上的易記介面。

> [!IMPORTANT]
> 在使用 Oozie 命令時，您必須使用 HDInsight 前端節點的 FQDN。 只有從叢集才能存取此 FQDN，或者，如果叢集位於 Azure 虛擬網路，就必須從同一個網路中的其他電腦來存取。


1. 若要取得 Oozie 服務的 URL，請使用下列命令：

    ```bash
    sed -n '/<name>oozie.base.url/,/<\/value>/p' /etc/oozie/conf/oozie-site.xml
    ```

    這會傳回類似以下 XML 的資訊：

    ```xml
    <name>oozie.base.url</name>
    <value>http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie</value>
    ```

    `http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie` 部分是要搭配 Oozie 命令使用的 URL。

2. 若要建立 URL 的環境變數，請使用下列命令，這樣您就不必為每個命令輸入該 URL：

    ```bash
    export OOZIE_URL=http://HOSTNAMEt:11000/oozie
    ```

    將 URL 取代為您稍早收到的 URL。
3. 若要提交作業，請使用下列命令：

    ```bash
    oozie job -config job.xml -submit
    ```

    此命令會從 `job.xml` 載入作業資訊並將其提交給 Oozie，但不會執行它。

    在命令完成後，系統應該會傳回作業的識別碼，例如 `0000005-150622124850154-oozie-oozi-W`。 此 ID 將用來管理工作。

4. 若要檢視作業的狀態，請使用下列命令：

    ```bash
    oozie job -info <JOBID>
    ```

    > [!NOTE]
    > 將 `<JOBID>` 取代為上一個步驟中所傳回的 ID。

    這會傳回類似以下文字的資訊：

        Job ID : 0000005-150622124850154-oozie-oozi-W
        ------------------------------------------------------------------------------------------------------------------------------------
        Workflow Name : useooziewf
        App Path      : wasb:///tutorials/useoozie
        Status        : PREP
        Run           : 0
        User          : USERNAME
        Group         : -
        Created       : 2015-06-22 15:06 GMT
        Started       : -
        Last Modified : 2015-06-22 15:06 GMT
        Ended         : -
        CoordAction ID: -
        ------------------------------------------------------------------------------------------------------------------------------------

    這項工作的狀態為 `PREP`。 這個狀態表示作業雖已建立但未啟動。

5. 若要啟動作業，請使用下列命令：

    ```bash
    oozie job -start JOBID
    ```

    > [!NOTE]
    > 使用先前傳回的 ID 取代 `<JOBID>`。

    如果您在使用此命令後檢查狀態，會發現作業為執行中狀態，並傳回作業內的動作資訊。

6. 在工作順利完成後，您可以使用以下命令，確認是否已產生資料並匯出至 SQL 資料庫資料表：

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D oozietest
    ```

    在 `1>` 提示字元輸入下列查詢：

    ```sql
    SELECT * FROM mobiledata
    GO
    ```

    所傳回的資訊類似下列文字︰

        deviceplatform  count
        Android 31591
        iPhone OS       22731
        proprietary development 3
        RIM OS  3464
        Unknown 213
        Windows Phone   1791
        (6 rows affected)

如需 Oozie 命令的詳細資訊，請參閱 [Oozie 命令列工具](https://oozie.apache.org/docs/4.1.0/DG_CommandLineTool.html)。

## <a name="oozie-rest-api"></a>Oozie REST API

透過 Oozie REST API，您可以建置自己的工具來搭配 Oozie 使用。 以下為使用 Oozie REST API 時 HDInsight 的特定資訊：

* **URI**：您可以從叢集的外部 (位於 `https://CLUSTERNAME.azurehdinsight.net/oozie`) 存取 REST API。

* **驗證**：若要進行驗證，請使用 API 以及叢集 HTTP 帳戶 (admin) 和密碼。 例如︰

    ```bash
    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/oozie/versions
    ```

如需如何使用 Oozie REST API 的詳細資訊，請參閱 [Oozie Web 服務 API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html)。

## <a name="oozie-web-ui"></a>Oozie Web UI

Oozie Web UI 可讓您用網頁檢視叢集上 Oozie 作業的狀態。 透過 Web UI，您可以檢視下列資訊：

   * 工作狀態
   * 工作定義
   * 組態
   * 工作中的動作圖表
   * 工作的記錄

您也可以檢視作業內的動作詳細資料。

若要存取 Oozie Web UI，請完成下列步驟：

1. 建立 HDInsight 叢集的 SSH 通道。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH 通道](hdinsight-linux-ambari-ssh-tunnel.md)。

2. 建立通道後，請在網頁瀏覽器中開啟 Ambari Web UI。 Ambari 網站的 URI 為 `https://CLUSTERNAME.azurehdinsight.net`。 請將 `CLUSTERNAME` 取代為您 Linux 型 HDInsight 叢集的名稱。

3. 在頁面左邊選取 [Oozie] > [快速連結] > [Oozie Web UI]。

    ![功能表的影像](./media/hdinsight-use-oozie-linux-mac/ooziewebuisteps.png)

4. Oozie Web UI 預設會顯示執行中的工作流程作業。 若要查看所有的工作流程作業，請選取 [所有作業]。

    ![顯示的所有工作](./media/hdinsight-use-oozie-linux-mac/ooziejobs.png)

5. 若要檢視作業的詳細資訊，請選取作業。

    ![工作資訊](./media/hdinsight-use-oozie-linux-mac/jobinfo.png)

6. 您可以在 [作業資訊] 索引標籤中看到基本的作業資訊，以及作業內的個別動作。 您可以使用頂端的索引標籤，來檢視 [作業定義]、[作業組態]，以及存取 [作業記錄]，或檢視 [作業 DAG] 底下之作業的有向非循環圖 (DAG)。

   * **作業記錄**：選取 [取得記錄] 按鈕，以取得作業的所有記錄，或使用 [輸入搜尋篩選條件] 欄位來篩選記錄。

       ![作業記錄](./media/hdinsight-use-oozie-linux-mac/joblog.png)

   * **作業 DAG**：DAG 是將整個工作流程所採取的資料路徑予以圖形化的概觀。

       ![工作 DAG](./media/hdinsight-use-oozie-linux-mac/jobdag.png)

7. 如果您在 [作業資訊] 索引標籤中選取其中一個動作，它將會顯示該動作的資訊。 例如，選取 **RunSqoopExport** 動作。

    ![動作資訊](./media/hdinsight-use-oozie-linux-mac/action.png)

8. 您可以查看動作的詳細資料，例如 **Console URL** (主控台 URL) 的連結。 請使用此連結來檢視作業的作業追蹤器資訊。

## <a name="schedule-jobs"></a>排程作業

您可以使用協調器指定作業的開始時間、結束時間和發生頻率。 若要定義工作流程的排程，請完成下列步驟：

1. 使用下列命令建立名為 **coordinator.xml** 的檔案：

    ```bash
    nano coordinator.xml
    ```

    使用下列 XML 做為檔案的內容：

    ```xml
    <coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
        <action>
        <workflow>
            <app-path>${workflowPath}</app-path>
        </workflow>
        </action>
    </coordinator-app>
    ```

    > [!NOTE]
    > `${...}` 變數在執行階段將被作業定義中的值取代。 這些變數分為別：
    >
    > * `${coordFrequency}`：執行作業執行個體的間隔時間。
    > * `${coordStart}`：作業開始時間。
    > * `${coordEnd}`：工作結束時間。
    > * `${coordTimezone}`：協調器作業位在固定時區，不受日光節約時間影響 (通常使用 UTC 表示)。 此時區稱為「Oozie 處理時區」。
    > * `${wfPath}`：workflow.xml 的路徑。

2. 若要儲存檔案，請選取 Ctrl+X、輸入 `Y`，然後選取 **Enter**。

3. 若要將該檔案複製到此作業的工作目錄，請使用下列命令：

    ```bash
    hadoop fs -put coordinator.xml /tutorials/useoozie/coordinator.xml
    ```

4. 若要修改 `job.xml` 檔案，請使用下列命令：

    ```
    nano job.xml
    ```

    進行下列變更：

   * 若要指示 Oozie 執行協調器檔案而非工作流程，請將 `<name>oozie.wf.application.path</name>` 變更為 `<name>oozie.coord.application.path</name>`。

   * 若要設定協調器使用的 `workflowPath` 變數，請新增下列 XML：

        ```xml
        <property>
            <name>workflowPath</name>
            <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
        ```

       將 `wasb://mycontainer@mystorageaccount.blob.core.windows` 文字取代為在 job.xml 檔案之其他項目中使用的值。

   * 若要定義協調器的開始時間、結束時間和頻率，請新增下列 XML：

        ```xml
        <property>
            <name>coordStart</name>
            <value>2017-05-10T12:00Z</value>
        </property>

        <property>
            <name>coordEnd</name>
            <value>2017-05-12T12:00Z</value>
        </property>

        <property>
            <name>coordFrequency</name>
            <value>1440</value>
        </property>

        <property>
            <name>coordTimezone</name>
            <value>UTC</value>
        </property>
        ```

       這些值會將開始時間設為 2017 年 5 月 10 日下午 12:00，將結束時間設為 2017 年 5 月 12 日。 用來執行此作業的間隔會設為每日。 頻率的單位是分鐘，因此 24 小時 x 60 分鐘 = 1440 分鐘。 最後，時區會設為 UTC。

5. 若要儲存檔案，請選取 Ctrl+X、輸入 `Y`，然後選取 **Enter**。

6. 若要執行工作，請使用以下命令：

    ```
    oozie job -config job.xml -run
    ```

    此命令會提交並啟動工作。

7. 如果您移至 Oozie Web UI，並選取 [協調器作業] 索引標籤，您會看到如下圖所示的資訊：

    ![協調器作業索引標籤](./media/hdinsight-use-oozie-linux-mac/coordinatorjob.png)

    [Next Materialization (下次具體化)] 項目包含下次工作執行的時間。

8. 與先前的工作流程作業類似，若您在 Web UI 中選取作業項目，它便會顯示作業的資訊：

    ![協調器作業資訊](./media/hdinsight-use-oozie-linux-mac/coordinatorjobinfo.png)

    > [!NOTE]
    > 此影像只會顯示作業的成功執行項目，而不是排程工作流程內的個別動作。 若要查看個別動作，請選取其中一個**動作**項目。

    ![動作資訊](./media/hdinsight-use-oozie-linux-mac/coordinatoractionjob.png)

## <a name="troubleshooting"></a>疑難排解

透過 Oozie UI，您可以檢視 Oozie 記錄。 Oozie UI 也包含工作流程所啟動之 MapReduce 工作的 JobTracker 記錄連結。 疑難排解的模式應該是：

   1. 在 Oozie Web UI 中檢視作業。

   2. 如果發生錯誤或特定動作失敗，請選取該動作，以查看 [Error Message] \(錯誤訊息)  欄位是否有提供失敗的詳細資訊。

   3. 如果有提供，請使用動作的 URL 以檢視動作的更多詳細資料，例如 JobTracker 記錄。

以下是您可能會遇到的特定錯誤，以及解決這些問題的方法。

### <a name="ja009-cannot-initialize-cluster"></a>JA009：無法初始化叢集

**徵兆**：作業狀態會變更為 **SUSPENDED**。 作業的詳細資料會將 `RunHiveScript` 狀態顯示為 **START_MANUAL**。 選取該動作會顯示下列錯誤訊息：

    JA009: Cannot initialize Cluster. Please check your configuration for map

**原因**：**job.xml** 檔案中使用的 Azure Blob 儲存體位址未包含儲存體容器或儲存體帳戶名稱。 Blob 儲存體位址的格式必須是 `wasb://containername@storageaccountname.blob.core.windows.net`。

**解決方法**：變更作業所使用的 Blob 儲存體位址。

### <a name="ja002-oozie-is-not-allowed-to-impersonate-ltuser"></a>JA002：不允許 Oozie 模擬 &lt;USER>

**徵兆**：作業狀態會變更為 **SUSPENDED**。 作業的詳細資料會將 `RunHiveScript` 狀態顯示為 **START_MANUAL**。 如果您選取該動作，它將會顯示下列錯誤訊息：

    JA002: User: oozie is not allowed to impersonate <USER>

**原因**：目前的權限設定不允許 Oozie 模擬指定的使用者帳戶。

**解決方法**：Oozie 可以模擬 **users** 群組中的使用者。 使用 `groups USERNAME` 查看使用者帳戶所屬的群組。 如果使用者不是 **users** 群組的成員，請使用以下命令將使用者新增至群組：

    sudo adduser USERNAME users

> [!NOTE]
> 這可能需要幾分鐘，HDInsight 才能辨識出使用者已新增至該群組。

### <a name="launcher-error-sqoop"></a>啟動器錯誤 (Sqoop)

**徵兆**：作業狀態會變更為 **KILLED**。 作業的詳細資料會將 `RunSqoopExport` 狀態顯示為 **ERROR**。 如果您選取該動作，它將會顯示下列錯誤訊息：

    Launcher ERROR, reason: Main class [org.apache.oozie.action.hadoop.SqoopMain], exit code [1]

**原因**：Sqoop 無法載入存取資料庫時所需的資料庫驅動程式。

**解決方法**：當您從 Oozie 作業使用 Sqoop 時，您必須將資料庫驅動程式與作業所使用的其他資源 (例如 workflow.xml) 包含在一起。 此外，請從 workflow.xml 的 `<sqoop>...</sqoop>` 區段，參考含有資料庫驅動程式的封存。

例如，您可以針對本文件中的工作使用下列步驟：

1. 將 `sqljdbc4.1.jar` 檔案複製到 **/tutorials/useoozie** 目錄中：

    ```bash
    hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc41.jar /tutorials/useoozie/sqljdbc41.jar
    ```

2. 修改 `workflow.xml`，在 `</sqoop>` 上方新的一行上新增下列 XML：

    ```xml
    <archive>sqljdbc41.jar</archive>
    ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何定義 Oozie 工作流程，以及如何執行 Oozie 工作。 若要深入了解如何使用 HDInsight，請參閱下列文章：

* [搭配 HDInsight 使用以時間為基礎的 Hadoop Oozie 協調器][hdinsight-oozie-coordinator-time]
* [在 HDInsight 中上傳 Hadoop 作業的資料][hdinsight-upload-data]
* [搭配使用 Sqoop 與 HDInsight 中的 Hadoop][hdinsight-use-sqoop]
* [搭配使用 Hive 與 HDInsight 上的 Hadoop][hdinsight-use-hive]
* [搭配使用 Pig 與 HDInsight 上的 Hadoop][hdinsight-use-pig]
* [開發 HDInsight 的 Java MapReduce 程式][hdinsight-develop-mapreduce]

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563
[azure-data-factory-pig-hive]: ../data-factory/transform-data.md
[hdinsight-oozie-coordinator-time]: hdinsight-use-oozie-coordinator-time.md
[hdinsight-versions]:  hdinsight-component-versioning.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started]: hdinsight-get-started.md
[hdinsight-use-sqoop]:hadoop/apache-hadoop-use-sqoop-mac-linux.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started-emulator]: hdinsight-get-started-emulator.md
[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[sqldatabase-create-configue]: sql-database-create-configure.md
[sqldatabase-get-started]: sql-database-get-started.md

[azure-create-storageaccount]:../storage/common/storage-create-storage-account.md

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://azure.microsoft.com/downloads/
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/library/ee176961.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
