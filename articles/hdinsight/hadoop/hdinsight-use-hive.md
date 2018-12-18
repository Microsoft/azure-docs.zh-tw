---
title: 什麼是 Apache Hive 和 HiveQL - Azure HDInsight
description: Apache Hive 是適用於 Hadoop 的資料倉儲系統。 您可以使用 HiveQL (這類似於 TRANSACT-SQL) 查詢 Hive 中儲存的資料。 在本文件中，您將了解如何使用 Hive 和 HiveQL 搭配 Azure HDInsight。
keywords: hiveql,什麼是 hive,hadoop hiveql,如何使用 hive,了解 hive,什麼是s hive
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 04/23/2018
ms.openlocfilehash: 446bb23e15d908c8afe189a33e4d8a70faad284a
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126632"
---
# <a name="what-is-apache-hive-and-hiveql-on-azure-hdinsight"></a>Azure HDInsight 上的 Apache Hive 和 HiveQL 是什麼？

[Apache Hive](http://hive.apache.org/) 是適用於 Hadoop 的資料倉儲系統。 Hive 可執行資料摘要、查詢以及資料分析。 Hive 查詢是以 HiveQL 撰寫而成，這是類似 SQL 的查詢語言。

Hive 可讓您將結構投影在大量非結構化資料上。 定義結構後，您不需具備 Jave 或 MapReduce 相關知識，即可使用 HiveQL來查詢資料。

HDInsight 提供數種已針對特定工作負載進行微調的叢集類型。 下列叢集類型最常用於 Hive 查詢︰

* __互動式查詢__︰提供[低延遲分析處理 (LLAP)](https://cwiki.apache.org/confluence/display/Hive/LLAP) 功能的 Hadoop 叢集，可改善互動式查詢的回應時間。 如需詳細資訊，請參閱[開始使用 HDInsight 中的互動式查詢](../interactive-query/apache-interactive-query-get-started.md)文件。

* __Hadoop__︰已針對批次處理工作負載進行微調的 Hadoop 叢集。 如需詳細資訊，請參閱[開始使用 HDInsight 中的 Hadoop](../hadoop/apache-hadoop-linux-tutorial-get-started.md) 文件。

* __Spark__：Apache Spark 有可用於 Hive 的內建功能。 如需詳細資訊，請參閱[開始使用 HDInsight 上的 Spark](../spark/apache-spark-jupyter-spark-sql.md) 文件。

* __HBase__：HiveQL 可以用來查詢 HBase 中儲存的資料。 如需詳細資訊，請參閱[開始使用 HDInsight 上的 HBase](../hbase/apache-hbase-tutorial-get-started-linux.md) 文件。

## <a name="how-to-use-hive"></a>如何使用 Hive

使用下表了解搭配使用 Hive 與 HDInsight 的各種方式︰

| **使用此方法**，如果您想要... | ...**互動式**查詢 | ...**批次** 處理 | ...搭配此 **叢集作業系統** | ...從此 **用戶端作業系統** |
|:--- |:---:|:---:|:--- |:--- |
| [適用於 Visual Studio Code 的 HDInsight 工具](../hdinsight-for-vscode.md) |✔ |✔ |Linux | Linux、Unix、Mac OS X 或 Windows |
| [HDInsight Tools for Visual Studio](../hadoop/apache-hadoop-use-hive-visual-studio.md) |✔ |✔ |Linux 或 Windows* |Windows |
| [Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md) |✔ |✔ |Linux |任何 (以瀏覽器為基礎) |
| [Beeline 用戶端](../hadoop/apache-hadoop-use-hive-beeline.md) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [REST API](../hadoop/apache-hadoop-use-hive-curl.md) |&nbsp; |✔ |Linux 或 Windows* |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](../hadoop/apache-hadoop-use-hive-powershell.md) |&nbsp; |✔ |Linux 或 Windows* |Windows |

> [!IMPORTANT]
> \* Linux 是 HDInsight 3.4 版或更新版本上唯一使用的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="hiveql-language-reference"></a>HiveQL 語言參考

HiveQL 語言參考可在[語言手冊 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) 中找到。

## <a name="hive-and-data-structure"></a>Hive 和資料結構

Hive 了解如何使用結構化和半結構化資料。 例如，以特定字元分隔欄位的文字檔案。 下列 HiveQL 陳述式會使用以空格分隔的資料建立資料表︰

```hiveql
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
```

Hive 也支援自訂複雜或不規則結構化資料的 **序列化/反序列化程式 (SerDe)** 。 如需詳細資訊，請參閱[如何搭配 HDInsight 使用自訂 JSON SerDe](http://blogs.msdn.com/b/bigdatasupport/archive/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight.aspx) 文件 (英文)。

如需 Hive 所支援檔案格式的詳細資訊，請參閱[語言手冊 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)。

### <a name="hive-internal-tables-vs-external-tables"></a>Hive 內部和外部資料表比較。

您可以使用 Hive 建立兩種類型的資料表：

* __內部__︰資料會儲存在 Hive 資料倉儲中。 資料倉儲位於叢集之預設儲存體上的 `/hive/warehouse/`。

    符合下列其中一項條件時，請使用內部資料表：

    * 資料是暫存的。
    * 您想要 Hive 管理資料表和資料的生命週期。

* __外部__︰資料會儲存在資料倉儲之外。 資料可以儲存在叢集可存取的任何儲存體上。

    符合下列其中一項條件時，請使用外部資料表：

    * 資料也使用於 Hive 之外。 例如，資料檔案由其他程序所更新 (不會鎖定檔案)。
    * 即使在刪除資料表後，資料都必須保留在基礎位置。
    * 您需要自訂位置，例如非預設儲存體帳戶。
    * Hive 以外的程式管理資料格式、位置等等。

如需詳細資訊，請參閱 [Hive 內部和外部資料表簡介][cindygross-hive-tables]部落格文章。

## <a name="user-defined-functions-udf"></a>使用者定義函數 (UDF)

Hive 也可透過 **使用者定義函數 (UDF)** 延伸。 UDF 可讓您在 HiveQL 中實作功能或不易模型化的邏輯。 如需將 UDF 與 Hive 搭配使用的範例，請參閱以下文件：

* [將 Java 使用者定義的函式與 Hive 搭配使用](../hadoop/apache-hadoop-hive-java-udf.md)

* [搭配使用 Python 使用者定義函式與 Hive](../hadoop/python-udf-hdinsight.md)

* [搭配使用 C# 使用者定義函式與 Hive](../hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [如何將自訂 Hive 使用者定義的函式新增至 HDInsight](http://blogs.msdn.com/b/bigdatasupport/archive/2014/01/14/how-to-add-custom-hive-udfs-to-hdinsight.aspx)

* [可將日期/時間格式轉換成 Hive 時間戳記的範例 Hive 使用者定義函式](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## <a id="data"></a>範例資料

HDInsight 上的 Hive 已預先載入名為 `hivesampletable` 的內部資料表。 HDInsight 也提供可搭配 Hive 使用的範例資料集。 這些資料集會儲存在 `/example/data` 和 `/HdiSamples` 目錄中。 這些目錄存在於您叢集的預設儲存體中。

## <a id="job"></a>範例 Hive 查詢

下列 HiveQL 陳述式將資料行投影在 `/example/data/sample.log` 檔案︰

```hiveql
set hive.execution.engine=tez;
DROP TABLE log4jLogs;
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs 
    WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' 
    GROUP BY t4;
```

在上一個範例中，HiveQL 陳述式會執行下列動作：

* `set hive.execution.engine=tez;`：設定執行引擎以使用 Tez。 使用 Tez 可提升查詢效能。 如需 Tez 的詳細資訊，請參閱 [使用 Apache Tez 以提升效能](#usetez) 一節。

    > [!NOTE]
    > 只有在使用以 Windows 為基礎的 HDInsight 叢集時，才需要此陳述式。 Tez 是以 Linux 為基礎的 HDInsight 預設的執行引擎。

* `DROP TABLE`︰如果資料表已存在，請刪除資料表。

* `CREATE EXTERNAL TABLE`：在 Hive 中建立新的**外部**資料表。 外部資料表只會將資料表定義儲存在 Hive 中。 資料會留在原來的位置，並保持原始格式。

* `ROW FORMAT`：告訴 Hive 如何格式化資料。 在此情況下，每個記錄中的欄位會以空格隔開。

* `STORED AS TEXTFILE LOCATION`：將資料的儲存位置告訴 Hive (`example/data` 目錄)，且資料會儲存為文字。 資料可以在目錄的一個檔案中，也可以分散在多個檔案中。

* `SELECT`：選取資料行 **t4** 包含 **[ERROR]** 值的所有資料列計數。 這個陳述式會傳回值 **3**，因為有三個資料列包含此值。

* `INPUT__FILE__NAME LIKE '%.log'` - Hive 嘗試將結構描述套用至目錄中的所有檔案。 在此情況下，目錄包含不符合結構描述的檔案。 若要防止結果中出現亂碼資料，此陳述式會告訴 Hive 我們只應該從檔名以 log 結尾的檔案傳回資料。

> [!NOTE]
> 當您預期會由外部來源來更新基礎資料時，請使用外部資料表。 例如，自動化的資料上傳程序，或 MapReduce 作業。
>
> 捨棄外部資料表並「不會」  刪除資料，只會刪除資料表定義。

若要建立**內部**資料表，而不是外部資料表，請使用下列 HiveQL：

```hiveql
set hive.execution.engine=tez;
CREATE TABLE IF NOT EXISTS errorLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
STORED AS ORC;
INSERT OVERWRITE TABLE errorLogs
SELECT t1, t2, t3, t4, t5, t6, t7 
    FROM log4jLogs WHERE t4 = '[ERROR]';
```

這些陳述式會執行下列動作：

* `CREATE TABLE IF NOT EXISTS`︰如果資料表不存在，請建立資料表。 因為未使用 **EXTERNAL** 關鍵字，這個陳述式會建立內部資料表。 資料表會儲存在 Hive 資料倉儲中，並完全受到 Hive 所管理。

* `STORED AS ORC`：以最佳化資料列單欄式 (Optimized Row Columnar, ORC) 格式儲存資料。 ORC 是高度最佳化且有效率的 Hive 資料儲存格式。

* `INSERT OVERWRITE ... SELECT`：從 **log4jLogs** 資料表選取包含 **[ERROR]** 的資料列，然後將資料插入 **errorLogs** 資料表。

> [!NOTE]
> 與外部資料表不同之處在於，捨棄內部資料表也會刪除基礎資料。

## <a name="improve-hive-query-performance"></a>改善 Hive 查詢效能

### <a id="usetez"></a>Apache Tez

[Apache Tez](http://tez.apache.org) 是可讓資料高用量應用程式 (例如 Hive)，以大規模而更有效率方式執行作業的架構。 對於以 Linux 為基礎的 HDInsight 叢集，Tez 預設為開啟。

> [!NOTE]
> 對於 Windows 型的 HDInsight 叢集，Tez 目前預設為關閉，因而必須啟用。 若要充分發揮 Tez 的效益，您必須設定 Hive 查詢的下列值：
>
> `set hive.execution.engine=tez;`
>
> 對於以 Linux 為基礎的 HDInsight 叢集，Tez 是預設引擎。

[Tez 上的 Hive 設計文件](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)包含實作選擇和調整組態的詳細資料。

為了使用 Tez 來協助偵錯作業，HDInsight 提供下列 Web UI，讓您檢視 Tez 作業的詳細資料：

* [在以 Linux 為基礎的 HDInsight 上使用 Ambari Tez 檢視](../hdinsight-debug-ambari-tez-view.md)

* [在以 Windows 為基礎的 HDInsight 上使用 Tez UI](../hdinsight-debug-tez-ui.md)

### <a name="low-latency-analytical-processing-llap"></a>低延遲分析處理 (LLAP)

[LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP) (有時也稱為 Live Long and Process) 是 Hive 2.0 的新功能，允許在記憶體中快取查詢。 LLAP 讓 Hive 查詢的速讀變快，在某些情況下可達到[比 Hive 1.x 快 26 倍](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/)。

HDInsight 可提供互動式查詢叢集類型的 LLAP。 如需詳細資訊，請參閱[開始使用互動式查詢](../interactive-query/apache-interactive-query-get-started.md)文件。

## <a name="scheduling-hive-queries"></a>排程 Hive 查詢

有數項服務可讓您使用排程或隨需的工作流程來執行 Hive 查詢。

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory 可讓您使用 HDInsight 作為 Data Factory 管線的一部分。 若想進一步了解如何使用管線中的 Hive，請參閱[使用 Azure Data Factory 中的 Hive 活動轉換資料](../../data-factory/transform-data-using-hadoop-hive.md)文件。

### <a name="hive-jobs-and-sql-server-integration-services"></a>Hive 工作和 SQL Server 整合服務

您可以使用 SQL Server Integration Services (SSIS) 來執行 Hive 作業。 適用於 SSIS 的 Azure Feature Pack 中提供下列元件可搭配 HDInsight 上的 Hive 工作使用。

* [Azure HDInsight Hive 工作][hivetask]

* [Azure 訂用帳戶連接管理員][connectionmanager]

如需詳細資訊，請參閱 [Azure Feature Pack][ssispack] 文件。

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie 是可管理 Hadoop 作業的工作流程和協調系統。 如需搭配使用 Oozie 與 Hive 的詳細資訊，請參閱[使用 Oozie 定義和執行工作流程](../hdinsight-use-oozie-linux-mac.md)文件。

## <a id="nextsteps"></a>後續步驟

現在您已了解什麼是 Hive 以及如何搭配 HDInsight 中的 Hadoop 使用它，接著請使用下列連結探索 Azure HDInsight 的其他使用方式。

* [將資料上傳至 HDInsight][hdinsight-upload-data]
* [搭配 HDInsight 使用 Pig][hdinsight-use-pig]
* [搭配 HDInsight 使用 MapReduce 作業][hdinsight-use-mapreduce]

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: http://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/
[hivetask]: http://msdn.microsoft.com/library/mt146771(v=sql.120).aspx
[connectionmanager]: http://msdn.microsoft.com/library/mt146773(v=sql.120).aspx
[ssispack]: http://msdn.microsoft.com/library/mt146770(v=sql.120).aspx

[hdinsight-use-pig]: hdinsight-use-pig.md
[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md


[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: ../hdinsight-upload-data.md

[Powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-here-strings]: http://technet.microsoft.com/library/ee692792.aspx


[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx
