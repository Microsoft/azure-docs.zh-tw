---
title: 使用 psql 來大量載入至 Apache Phoenix - Azure HDInsight
description: 使用 psql 工具將大量資料載入至 Phoenix 資料表。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/10/2017
ms.author: ashishth
ms.openlocfilehash: 4f4caec33414a9bf644e1b1860686247697b3fb4
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042279"
---
# <a name="bulk-load-data-into-phoenix-using-psql"></a>使用 psql 將資料大量載入至 Phoenix

[Apache Phoenix](http://phoenix.apache.org/) 是一個建置在 [HBase](../hbase/apache-hbase-overview.md) 上的開放原始碼、大規模平行關聯式資料庫。 Phoenix 可透過 HBase 提供類似 SQL 的查詢。 Phoenix 使用 JDBC 驅動程式，可讓使用者建立、刪除及更改 SQL 資料表、索引、檢視表和序列，以及個別或大量更新插入資料列。 Phoenix 使用 noSQL 原生編譯而不是使用 MapReduce 來編譯查詢，可在 HBase 上建立低延遲的應用程式。 Phoenix 新增了協同處理器，可支援在伺服器的位址空間中執行用戶端提供的程式碼，執行與資料共置的程式碼。 這可將用戶端/伺服器資料傳輸降到最低。  若要在 HDInsight 中使用 Phoenix 來處理資料，請先建立資料表，然後將資料載入至這些資料表。

## <a name="bulk-loading-with-phoenix"></a>使用 Phoenix 進行大量載入

有多個可將資料載入至 HBase 的方法，包括使用用戶端 API、搭配 TableOutputFormat 的 MapReduce 作業，或使用 HBase 殼層來手動輸入資料。 Phoenix 提供兩個可將 CSV 資料載入至 Phoenix 資料表的方法：一個名為 `psql` 的用戶端載入工具，以及一個以 MapReduce 為基礎的大量載入工具。

`psql` 工具是單一執行緒工具，最適用於載入數 MB 或 GB 的資料。 所有要載入的 CSV 檔案都必須具有 '.csv' 副檔名。  您也可以在 `psql` 命令列中指定含有 '.sql' 副檔名的 SQL 指令碼檔。

使用 MapReduce 進行的大量載入會用於規模大許多的資料磁碟區，通常是在生產環境案例中使用，因為 MapReduce 會使用多個執行緒。

在您開始載入資料之前，請先確認已啟用 Phoenix 且查詢逾時設定與預期的一樣。  請存取您的 HDInsight 叢集 Ambari 儀表板，然後依序選取 [HBase] 和 [Configuration] \(組態\) 索引標籤。向下捲動以確認 Apache Phoenix 已設定為 `enabled`，如下所示：

![Apache Phoenix HDInsight 叢集設定](./media/apache-hbase-phoenix-psql/ambari-phoenix.png)

### <a name="use-psql-to-bulk-load-tables"></a>使用 `psql` 來大量載入資料表

1. 建立新資料表，然後使用 `createCustomersTable.sql` 檔案名稱來儲存查詢。

    ```sql
    CREATE TABLE Customers (
        ID varchar NOT NULL PRIMARY KEY,
        Name varchar,
        Income decimal,
        Age INTEGER,
        Country varchar);
    ```

2. 將 CSV 檔案 (所顯示的範例內容) 以 `customers.csv` 形式複製到 `/tmp/` 目錄，以供載入至新建立的資料表。  使用 `hdfs` 命令將該 CSV 檔案複製到您想要的來源位置。

    ```
    1,Samantha,260000.0,18,US
    2,Sam,10000.5,56,US
    3,Anton,550150.0,Norway
    ... 4997 more rows 
    ```

    ```bash
    hdfs dfs -copyToLocal /example/data/customers.csv /tmp/
    ```

3. 建立 SQL SELECT 查詢來確認是否已正確載入輸入資料，然後使用 `listCustomers.sql` 檔案名稱來儲存查詢。 您可以使用任何 SQL 查詢。
     ```sql
    SELECT Name, Income from Customers group by Country;
    ```

4. 開啟一個「新的」Hadoop 命令視窗來大量載入資料。 首先，使用 `cd` 命令來變更至執行目錄位置，然後使用 `psql` 工具 (Python `psql.py` 命令)。 

    以下範例會預期您已使用 `hdfs` 將 `customers.csv` 檔案從儲存體帳戶複製到本機暫存目錄，如上面步驟 2 所示。

    ```bash
    cd /usr/hdp/current/phoenix-client/bin

    python psql.py ZookeeperQuorum createCustomersTable.sql /tmp/customers.csv listCustomers.sql
    ```

    > [!NOTE] 
    > 若要判斷 `ZookeeperQuorum` 名稱，請在 `/etc/hbase/conf/hbase-site.xml` 檔案中，使用屬性名稱 `hbase.zookeeper.quorum` 來找出 Zookeeper 仲裁字串。

5. 在 `psql` 作業完成之後，您應該會在命令視窗中看到一則訊息：

    ```
    CSV Upsert complete. 5000 rows upserted
    Time: 4.548 sec(s)
    ```

## <a name="use-mapreduce-to-bulk-load-tables"></a>使用 MapReduce 來大量載入資料表

若要進行遍佈整個叢集的更高輸送量載入，請使用 MapReduce 載入工具。 此載入器會先將所有資料轉換成 HFile，然後將所建立的 HFile 提供給 HBase。

1. 使用 `hadoop` 命令搭配 Phoenix 用戶端 Jar 來啟動 CSV MapReduce 載入器：

    ```bash
    hadoop jar phoenix-<version>-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table CUSTOMERS --input /data/customers.csv
    ```

2. 使用 SQL 陳述式來建立新資料表，就像上面步驟 1 中使用 `CreateCustomersTable.sql` 一樣。

3. 若要確認資料表的結構描述，請執行 `!describe inputTable`。

4. 判斷輸入資料 (例如範例 `customers.csv` 檔案) 的位置路徑。 輸入檔可能在您的 WASB/ADLS 儲存體帳戶中。 在這個範例案例中，輸入檔是在 `<storage account parent>/inputFolderBulkLoad` 目錄中。

5. 變更至 MapReduce 大量載入命令的執行目錄：

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

6. 在 `/etc/hbase/conf/hbase-site.xml` 中，使用屬性名稱 `hbase.zookeeper.quorum` 來找出您的 `ZookeeperQuorum` 值。

7. 設定 classpath，然後執行 `CsvBulkLoadTool` 工具命令：

    ```bash
    /usr/hdp/current/phoenix-client$ HADOOP_CLASSPATH=/usr/hdp/current/hbase-client/lib/hbase-protocol.jar:/etc/hbase/conf hadoop jar /usr/hdp/2.4.2.0-258/phoenix/phoenix-4.4.0.2.4.2.0-258-client.jar

    org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input /inputFolderBulkLoad/customers.csv –zookeeper ZookeeperQuorum:2181:/hbase-unsecure
    ```

8. 若要使用 MapReduce 搭配 ADLS，請找出 ADLS 根目錄，也就是 `hbase-site.xml` 中的 `hbase.rootdir` 值。 在以下命令中，ADLS 根目錄是 `adl://hdinsightconf1.azuredatalakestore.net:443/hbase1`。 在此命令中，指定 ADLS 輸入和輸出資料夾作為參數：

    ```bash
    cd /usr/hdp/current/phoenix-client

    $HADOOP_CLASSPATH=$(hbase mapredcp):/etc/hbase/conf hadoop jar /usr/hdp/2.4.2.0-258/phoenix/phoenix-4.4.0.2.4.2.0-258-client.jar

    org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/temp/input/customers.csv –zookeeper ZookeeperQuorum:2181:/hbase-unsecure --output  adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/output1
    ```

## <a name="recommendations"></a>建議

* 針對輸入和輸出資料夾都使用相同的儲存媒體 (WASB 或 ADLS)。 若要將資料從 WASB 傳輸至 ADLS，您可以使用 `distcp` 命令：

    ```bash
    hadoop distcp wasb://@.blob.core.windows.net/example/data/gutenberg adl://.azuredatalakestore.net:443/myfolder
    ```

* 使用更大型的背景工作節點。 MapReduce 大量複製程序的 map 程序會產生大量的暫時輸出，這會填滿可用的非 DFS 空間。 若要進行大量的大量載入，請使用更多且更大型的背景工作節點。 您配置給叢集的背景工作節點數目會直接影響處理速度。

* 將輸入檔切割成大約 10 GB 的區塊。 大量載入是一個需要大量儲存體的作業，因此將輸入檔分割成多個區塊可以提升效能。

* 避免產生區域伺服器作用區。 如果您的資料列索引鍵以單純方式遞增，HBase 循序寫入可能會導致產生區域伺服器作用區。 對資料列索引鍵進行 *Salt* 處理將可減少循序寫入次數。 Phoenix 提供一個可在背景中以 Salt 位元組對特定資料表的資料列索引鍵進行 Salt 處理的方法，如以下參考資料所述。

## <a name="next-steps"></a>後續步驟

* [使用 Apache Phoenix 來進行大量資料載入](http://phoenix.apache.org/bulk_dataload.html) \(英文\)
* [在 HDInsight 中搭配 Linux 型 HBase 叢集使用 Apache Phoenix](../hbase/apache-hbase-phoenix-squirrel-linux.md)
* [以 Salt 處理的資料表](https://phoenix.apache.org/salted.html) \(英文\)
* [Phoenix 文法](http://phoenix.apache.org/language/index.html) \(英文\)
