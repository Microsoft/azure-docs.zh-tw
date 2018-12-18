---
title: 使用 Spark 以讀取及寫入 HBase 資料 - Azure HDInsight
description: 使用 Spark HBase Connector 將資料從 Spark 叢集讀取及寫入至 HBase 叢集。
services: hdinsight
author: maxluk
ms.author: maxluk
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 07/18/2018
ms.openlocfilehash: 6f957e5841bbc75756fc42d9496bbc1057cd478e
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047755"
---
# <a name="use-spark-to-read-and-write-hbase-data"></a>使用 Spark 來讀取和寫入 HBase 資料

Apache HBase 通常會使用其低階 API (scan、get、put) 或者使用 Phoenix 以 SQL 語法來查詢。 Apache 也提供 Spark HBase Connector，這是查詢及修改 HBase 儲存之資料的方便且高效能替代方式。

## <a name="prerequisites"></a>必要條件

* 兩個個別 HDInsight 叢集，一個 HBase，以及一個已安裝 Spark 2.1 (HDInsight 3.6) 的 Spark。
* Spark 叢集必須以最少的延遲直接與 HBase 叢集通訊，因此建議的設定是在相同的虛擬網路中同時部署兩個叢集。 如需詳細資訊，請參閱[使用 Azure 入口網站在 HDInsight 中建立以 Linux 為基礎的叢集](hdinsight-hadoop-create-linux-clusters-portal.md)。
* 對於每個叢集的 SSH 存取。
* 對於每個叢集預設儲存體的存取。

## <a name="overall-process"></a>整體程序

啟用您的 Spark 叢集以查詢 HDInsight 叢集的高階程序如下：

1. 在 HBase 中準備一些範例資料。
2. 從您的 HBase 叢集設定資料夾 (/etc/hbase/conf) 取得 hbase-site.xml 檔案。
3. 將 hbase-site.xml 的複本放置在您的 Spark 2 設定資料夾 (/etc/spark2/conf)。
4. 根據 `packages` 選項中的 Maven 座標，執行參照 Spark HBase Connector 的 `spark-shell`。
5. 定義目錄，該目錄將結構描述從 Spark 對應至 HBase。
6. 使用 RDD 或 DataFrame API 與 HBase 資料進行互動。

## <a name="prepare-sample-data-in-hbase"></a>在 HBase 中準備範例資料

在此步驟中，您會在 HBase 中建立並填入簡單的資料表，然後您可以使用 Spark 查詢。

1. 使用 SSH 連線到 HBase 叢集的前端節點。 如需詳細資訊，請參閱[使用 SSH 連線至 HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)。
2. 執行 HBase Shell：

        hbase shell

3. 建立具有資料行系列 `Personal` 和 `Office` 的 `Contacts` 資料表：

        create 'Contacts', 'Personal', 'Office'

4. 載入資料的一些範例資料列：

        put 'Contacts', '1000', 'Personal:Name', 'John Dole'
        put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
        put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
        put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
        put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
        put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
        put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
        put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'

## <a name="acquire-hbase-sitexml-from-your-hbase-cluster"></a>從您的 HBase 叢集取得 hbase-site.xml

1. 使用 SSH 連線到 HBase 叢集的前端節點。
2. 將 hbase-site.xml 從本機儲存體複製到 HBase 叢集預設儲存體的根目錄：

        hdfs dfs -copyFromLocal /etc/hbase/conf/hbase-site.xml /

3. 使用 [Azure 入口網站](https://portal.azure.com)瀏覽至 HBase 叢集。
4. 選取儲存體帳戶。 

    ![儲存體帳戶](./media/hdinsight-using-spark-query-hbase/storage-accounts.png)

5. 在清單中選取 [預設] 資料行底下具有核取記號的儲存體帳戶。

    ![預設儲存體帳戶](./media/hdinsight-using-spark-query-hbase/default-storage.png)

6. 在 [儲存體帳戶] 窗格中，選取 [Blob] 圖格。

    ![Blob 圖格](./media/hdinsight-using-spark-query-hbase/blobs-tile.png)

7. 在容器清單中，選取您 HBase 叢集所使用的容器。
8. 在檔案清單中，選取 `hbase-site.xml`。

    ![HBase-site.xml](./media/hdinsight-using-spark-query-hbase/hbase-site-xml.png)

9. 在 Blob 內容面板中，選取 [下載] 並且將 `hbase-site.xml` 儲存到本機電腦上的位置。

    ![下載](./media/hdinsight-using-spark-query-hbase/download.png)

## <a name="put-hbase-sitexml-on-your-spark-cluster"></a>將 hbase-site.xml 放置在您的 Spark 叢集

1. 使用 [Azure 入口網站](https://portal.azure.com)瀏覽至 Spark 叢集。
2. 選取儲存體帳戶。

    ![儲存體帳戶](./media/hdinsight-using-spark-query-hbase/storage-accounts.png)

3. 在清單中選取 [預設] 資料行底下具有核取記號的儲存體帳戶。

    ![預設儲存體帳戶](./media/hdinsight-using-spark-query-hbase/default-storage.png)

4. 在 [儲存體帳戶] 窗格中，選取 [Blob] 圖格。

    ![Blob 圖格](./media/hdinsight-using-spark-query-hbase/blobs-tile.png)

5. 在容器清單中，選取您 Spark 叢集所使用的容器。
6. 選取上傳。

    ![上傳](./media/hdinsight-using-spark-query-hbase/upload.png)

7. 選取您先前下載到本機電腦的 `hbase-site.xml` 檔案。

    ![上傳 hbase-site.xml](./media/hdinsight-using-spark-query-hbase/upload-selection.png)

8. 選取上傳。
9. 使用 SSH 連線到 Spark 叢集的前端節點。
10. 將 `hbase-site.xml` 從您的 Spark 叢集預設儲存體複製到叢集本機儲存體上的 Spark 2 設定資料夾：

        sudo hdfs dfs -copyToLocal /hbase-site.xml /etc/spark2/conf

## <a name="run-spark-shell-referencing-the-spark-hbase-connector"></a>執行參照 Spark HBase Connector 的 Spark Shell

1. 使用 SSH 連線到 Spark 叢集的前端節點。
2. 啟動 Spark Shell，指定 Spark HBase Connector 套件：

        spark-shell --packages com.hortonworks:shc-core:1.1.0-2.1-s_2.11 --repositories http://repo.hortonworks.com/content/groups/public/

3. 讓此 Spark Shell 執行個體保持開啟，並且繼續進行下一個步驟。

## <a name="define-a-catalog-and-query"></a>定義目錄和查詢

在這個步驟中，您會定義目錄物件，該目錄物件將結構描述從 Spark 對應至 HBase。 

1. 在開啟的 Spark Shell 中，執行下列 `import` 陳述式：

        import org.apache.spark.sql.{SQLContext, _}
        import org.apache.spark.sql.execution.datasources.hbase._
        import org.apache.spark.{SparkConf, SparkContext}
        import spark.sqlContext.implicits._

2. 針對您在 HBase 中建立的 [連絡人] 資料表定義目錄：
    1. 針對名為 `Contacts` 的 HBase 資料表定義目錄結構描述。
    2. 將 rowkey 識別為 `key`，並且將 Spark 中使用的資料行名稱對應至 HBase 中使用的資料行系列、資料行名稱以及資料行類型。
    3. rowkey 也必須詳細定義為具名資料行 (`rowkey`)，其具有 `rowkey` 的特定資料行系列 `cf`。

            def catalog = s"""{
                |"table":{"namespace":"default", "name":"Contacts"},
                |"rowkey":"key",
                |"columns":{
                |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
                |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
                |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
                |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
                |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
                |}
            |}""".stripMargin

3. 定義方法，該方法提供 HBase 中 `Contacts` 資料表周圍的 DataFrame：

            def withCatalog(cat: String): DataFrame = {
                spark.sqlContext
                .read
                .options(Map(HBaseTableCatalog.tableCatalog->cat))
                .format("org.apache.spark.sql.execution.datasources.hbase")
                .load()
            }

4. 建立 DataFrame 的執行個體：

        val df = withCatalog(catalog)

5. 查詢 DataFrame：

        df.show()

6. 您應該會看到兩個資料列的資料：

        +------+--------------------+--------------+-------------+--------------+
        |rowkey|       officeAddress|   officePhone| personalName| personalPhone|
        +------+--------------------+--------------+-------------+--------------+
        |  1000|1111 San Gabriel Dr.|1-425-000-0002|    John Dole|1-425-000-0001|
        |  8396|5415 San Gabriel Dr.|  230-555-0191|  Calvin Raji|  230-555-0191|
        +------+--------------------+--------------+-------------+--------------+

7. 註冊暫存資料表，以便使用 Spark SQL 查詢 HBase 資料表：

        df.registerTempTable("contacts")

8. 根據 `contacts` 資料表發出 SQL 查詢：

        val query = spark.sqlContext.sql("select personalName, officeAddress from contacts")
        query.show()

9. 您應該會看到如下的結果：

        +-------------+--------------------+
        | personalName|       officeAddress|
        +-------------+--------------------+
        |    John Dole|1111 San Gabriel Dr.|
        |  Calvin Raji|5415 San Gabriel Dr.|
        +-------------+--------------------+

## <a name="insert-new-data"></a>插入新資料

1. 若要插入新的連絡人記錄，請定義 `ContactRecord` 類別：

        case class ContactRecord(
            rowkey: String,
            officeAddress: String,
            officePhone: String,
            personalName: String,
            personalPhone: String
            )

2. 建立 `ContactRecord` 的執行個體並將其放置在陣列中：

        val newContact = ContactRecord("16891", "40 Ellis St.", "674-555-0110", "John Jackson","230-555-0194")

        var newData = new Array[ContactRecord](1)
        newData(0) = newContact

3. 將新資料的陣列儲存至 HBase：

        sc.parallelize(newData).toDF.write
        .options(Map(HBaseTableCatalog.tableCatalog -> catalog))
        .format("org.apache.spark.sql.execution.datasources.hbase").save()

4. 檢查結果︰
    
        df.show()

5. 您應該會看到如下的輸出：

        +------+--------------------+--------------+------------+--------------+
        |rowkey|       officeAddress|   officePhone|personalName| personalPhone|
        +------+--------------------+--------------+------------+--------------+
        |  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
        | 16891|        40 Ellis St.|  674-555-0110|John Jackson|  230-555-0194|
        |  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
        +------+--------------------+--------------+------------+--------------+

## <a name="next-steps"></a>後續步驟

* [Spark HBase Connector](https://github.com/hortonworks-spark/shc)
