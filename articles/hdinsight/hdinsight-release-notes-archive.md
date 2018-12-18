---
title: Azure HDInsight 的封存版本資訊
description: Azure HDInsight 的封存版本資訊與版本。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 03/20/2018
ms.author: jasonh
ms.openlocfilehash: f3be4b5e74c0c5cac9acd281b86e55c96b011763
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49363787"
---
# <a name="archived-release-notes-for-azure-hdinsight"></a>Azure HDInsight 的封存版本資訊

如需**最新的** Azure HDInsight 版本更新，請參閱 [HDInsight 版本資訊](hdinsight-release-notes.md)。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 版本控制文件](hdinsight-component-versioning.md)。

## <a name="notes-for-06272018---release-of-new-open-source-versions-adls-gen2-etc-on-hdinsight-36"></a>2018 年 06 月 27 日備註 - 在 HDInsight 3.6 上發布新的開放原始碼版本、ADLS Gen2 等
2018 年 6 月發布的 HDInsight 是一個重要版本，可為我們的客戶提供許多新的更新和功能。 如需詳細資料，請閱讀這篇[文章](https://azure.microsoft.com/blog/enterprises-get-deeper-insights-with-hadoop-and-spark-updates-on-azure-hdinsight/)。

以下為重點整理。 有關詳細的發布說明、已修復的錯誤 (bug)、已知問題等，請閱讀 [Azure HDInsight 的版本資訊](hdinsight-release-notes.md)。

- **更新 Hadoop 和其他開放原始碼專案** – 除了 20 多個開放原始碼專案中的 1000 多個錯誤 (bug) 修復程式外，此更新還包含新版 Spark (2.3) 和 Kafka (1.0)。
- **將 R Server 9.1 更新為機器學習服務 9.3** – 在此版本中，我們為資料科學家和工程師提供最佳的增強式開放原始碼及演算法創新，而且作業方便，這些全部都以他們的偏好語言和 Apache Spark 的速度提供。 此版本擴展 R Server 中提供的功能，並進一步支援 Python，因而將叢集名稱從 R Server 改為 ML 服務。 
- **支援 Azure Data Lake Storage Gen2** – HDInsight 將支援 Azure Data Lake Storage Gen2 的預覽版本。 在可用區域中，客戶能夠選擇 ADLS Gen2 帳戶作為其 HDInsight 叢集的存放區。
- **HDInsight 企業安全性套件更新 (預覽)** – (預覽) 虛擬網路服務端點支援 Azure blob 儲存體、ADLS Gen1、Cosmos DB 及 Azure DB。 

## <a name="notes-for-03202018---release-of-spark-22-on-hdinsight-36"></a>2018/03/20 相關資訊 - HDInsight 3.6 上的 Spark 2.2 版本

- Spark 2.2.0 改善了跨 Spark 核心、SQL、ML 的穩定性，並將結構化串流帶到 GA 狀態。 Spark 2.2.0 現在已可在 HDInsight 3.6 上使用。


## <a name="notes-for-08012017-release-of-hdinsight"></a>HDInsight 08/01/2017 版本的相關資訊

| 標題 | 說明 | 受影響的區域  | 叢集類型  | 
| --- | --- | --- | --- | --- |
| Microsoft R Server 9.1 on HDInsight 版本 |HDInsight 現在支援在 HDInsight 上佈建 R Server 9.1 叢集。 如需 Microsoft R Server 9.1 版本的詳細資訊，請參閱[這個部落格](https://blogs.technet.microsoft.com/dataplatforminsider/2017/04/19/introducing-microsoft-r-server-9-1-release/)。 |服務 |R 伺服器 |
| HDInsight 3.6 現在包含較新的 Hadoop 堆疊版本|<ul><li>如需更新後版本的詳細清單，請參閱 [HDInsight 中可用的 Hadoop 元件版本](hdinsight-component-versioning.md#hadoop-components-available-with-different-hdinsight-versions)。</li><li>如需已在 Hadoop 堆疊的最新版本中修正的錯誤清單，請參閱 [Apache 修補程式資訊](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_release-notes/content/patch_parent.html)。</li><li>如需 HDP 2.6.1 (現在已可在 HDInsight 3.6 中使用) 之間的中斷性變更清單，請參閱 [https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_release-notes/content/behavior_changes.html](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_release-notes/content/behavior_changes.html)。</li><li>如需 HDP 2.6.1 中的已知問題清單，請參閱[已知問題](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_release-notes/content/known_issues.html)。</li></ul> |服務 |全部 |N/A |
| 互動式 Hive (預覽) 叢集的更新 |<ul><li><b>功能改進。</b> 實作快取的中繼存放區，以藉由快取中繼資料來減少後端 SQL 的負載，並可改善所有中繼資料作業的效能。  這項改進現在是所有互動式 Hive 叢集的預設值。 如需詳細資訊，請參閱 [https://issues.apache.org/jira/browse/HIVE-16520](https://issues.apache.org/jira/browse/HIVE-16520)。</li><li><b>功能改進。</b> 動態分割區載入已經過最佳化。 如需詳細資訊，請參閱 [https://issues.apache.org/jira/browse/HIVE-14204] (https://issues.apache.org/jira/browse/HIVE-14204)。</li><li><b>功能改進。</b> 將 Linux 上的 HDInsight 設定最佳化。</li><li><b>錯誤修正。</b> `CredentialProviderFactory$getProviders` 不是安全執行緒。 」 現已修正此問題。 如需詳細資訊，請參閱 [https://issues.apache.org/jira/browse/HADOOP-14195](https://issues.apache.org/jira/browse/HADOOP-14195)。</li><li><b>錯誤修正。</b> WASB 驅動程式 `liststatus` API 若大量使用 CPU 會導致 ATS 效能不佳。 」 現已修正此問題。 如需詳細資訊，請參閱 [https://github.com/Azure/azure-storage-java/pull/154](https://github.com/Azure/azure-storage-java/pull/154)。</li></ul> |服務 |互動式 Hive (預覽) |
| Hadoop 叢集的更新 |改進 Templeton 工作作業的可靠性。 如需詳細資訊，請參閱 [https://issues.apache.org/jira/browse/HIVE-15947](https://issues.apache.org/jira/browse/HIVE-15947) |服務 |Hadoop |
| YARN 更新 | HDInsight 現已建立 250 GB 的 Ambari 資料庫 (未增加成本)，以讓客戶獲得更好的體驗。 這項變更應可防止 ATS 用完，從而獲得更好的效能。 |服務 |全部 |
| Spark 更新 | Spark 2.1.1 的版本。 如需詳細資訊，請參閱 [Spark 2.1.1 版](https://spark.apache.org/releases/spark-release-2-1-1.html)。 | 服務 | Spark |

  



## <a name="04062017---general-availability-of-hdinsight-36"></a>2017/04/06 - HDInsight 3.6 正式運作

* 此版本中，Azure HDInsight 新增以 HDP 2.6 為基礎的 3.6 版。 HDP 2.6 版本資訊可在[這裡](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_release-notes/content/ch_relnotes.html)找到，而 HDInsight 版本的詳細資訊則可以在[這裡](hdinsight-component-versioning.md)找到。 HDInsight 3.6 適用於下列工作負載：

    * Hadoop v2.7.3
    * HBase v1.1.2
    * Storm v1.1.0
    * Spark v2.1.0
    * Interactive Hive v2.1.0

* **Hive View 2.0 的支援**。 這應該會改善 Interactive Hive 的使用者體驗。 如需詳細資訊，請參閱 [Hortonworks 文件](http://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-views/content/ch_using_hive_view.html)。

* **使用 Hive LLAP 提高效能**。 如需詳細資訊，請參閱 [Hortonworks 文件](https://hortonworks.com/blog/top-5-performance-boosters-with-apache-hive-llap/)。

* **Hive 的新功能**。 請參閱 [Hortonworks 文件](https://hortonworks.com/apache/hive/#section_4)。

* **Hive CLI 取代**︰Hive CLI 已被取代，建議客戶改用 Beeline。 如需詳細資訊，請參閱 [Apache 文件](https://cwiki.apache.org/confluence/display/Hive/Replacing+the+Implementation+of+Hive+CLI+Using+Beeline)。 如需有關如何使用 Beeline 搭配 HDInsight 的指示，請參閱[搭配 HDInsight Hadoop 叢集使用 Beeline](hadoop/apache-hadoop-use-hive-beeline.md)。

* **Apache Phoenix 和 HBase 的新功能**。
    * 儲存配額支援︰通常用在多租用戶環境中，允許在個別資料表和個別命名空間等級上維持有限的儲存空間。
    * Phoenix 編製索引改進︰累加索引建立和從先前的失敗中重建/繼續編製索引。
    * Phoenix 資料完整性工具︰支援驗證結構描述、索引和其他中繼資料。


* **HBase 問題**︰執行 CSV 大量上傳 MapReduce 作業時，下列語法可能會導致錯誤。

        HADOOP_CLASSPATH=$(hbase mapredcp):/path/to/hbase/conf hadoop jar phoenix-<version>-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table EXAMPLE --input /data/example.csv

    請改用下列語法：

        HADOOP_CLASSPATH=/path/to/hbase-protocol.jar:/path/to/hbase/conf hadoop jar phoenix-<version>-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table EXAMPLE --input /data/example.csv


## <a name="02282017---release-of-spark-21-on-hdinsight-36-preview"></a>2017/02/28 - 發行 Spark 2.1 on HDInsight 3.6 (預覽版)
* [Spark 2.1](http://spark.apache.org/releases/spark-release-2-1-0.html) 會改善舊版的許多穩定性和可用性問題。 它也會有跨所有 Spark 工作負載的新功能，例如 Spark Core、SQL、ML 和串流處理。
* 結構化串流會利用事件時間的浮水印和 Kafka 0.10 連接器支援改良延展性。
* Spark SQL 分割現在會使用新的可調整資料分割處理機制來處理。 在[這裡](http://spark.apache.org/releases/spark-release-2-1-0.html)查看如何升級的其他詳細資料。
* Spark 2.1 on Azure HDInsight 3.6 預覽目前不支援使用 ODBC 驅動程式的 BI 工具連線。
* 此預覽版本不支援從 Spark 2.1 叢集的 Azure Data Lake Store 存取。


## <a name="11182016---release-of-spark-201-on-hdinsight-35"></a>2016/11/18 - 發行 Spark 2.0.1 on HDInsight 3.5
Spark 2.0.1 現已可在 Spark 叢集上取得 (HDInsight 3.5 版)。

## <a name="11162016---release-of-r-server-90-on-hdinsight-35-spark-20"></a>2016/11/16 - 發行 R Server 9.0 on HDInsight 3.5 (Spark 2.0)
*   R 伺服器叢集現在包括適用於下列兩個版本的選項︰HDI 3.5 (Spark 2.0) 上的 R 伺服器 9.0 和 HDI 3.4 (Spark 1.6) 上的 R 伺服器 8.0。
*   HDI 3.5 (Spark 2.0) 上的 R 伺服器 9.0 是以 R 3.3.2 為基礎，且包含新的 ScaleR 資料來源函式，其名稱為 RxHiveData 和 RxParquetData，可用來將資料從 Hive 和 Parquet 直接載入到 Spark 資料框架供 ScaleR 進行分析。 如需詳細資訊，請透過使用 **?RxHiveData** 和 **?RxParquetData** 命令參閱 R 中這些函式的內嵌說明。
*   RStudio 伺服器 Community 版本現在預設 (具有選擇退出選項) 會在佈建流程中安裝在 [叢集組態] 刀鋒視窗上。

## <a name="11092016---release-of-spark-20-on-hdinsight"></a>2016/11/09 - 發行 Spark 2.0 on HDInsight
* HDInsight 3.5 上的 Spark 2.0 叢集現在支援 Livy 和 Jupyter 服務。

## <a name="10262016---release-of-r-server-on-hdinsight"></a>2016/10/26 - 發行 R Server on HDInsight
* 用於邊緣節點存取的 URI 已變更為 **clustername**-ed-ssh.azurehdinsight.net
* HDInsight 上 R 伺服器的叢集佈建已簡化。
* HDInsight 上的 R 伺服器現在可做為一般 HDInsight「R 伺服器」叢集類型，不會再安裝為個別的 HDInsight 應用程式。 邊緣節點和 R 伺服器二進位檔現在會佈建為 R 伺服器叢集部署的一部分。 這可改善佈建速度和可靠性。 R 伺服器的定價模式也會隨之更新。
* R 伺服器叢集類型價格現在會根據標準層價格加上 R 伺服器額外費用價格。 這項變更不會影響 R 伺服器的實際價格，它只會變更帳單中費用的呈現方式。 所有現有的 R 伺服器叢集會繼續運作，而 Resource Manager 範本則會繼續運作，直到取代通知發出為止。 **雖然如此，仍建議更新已編寫指令碼的部署，以使用新的 Resource Manager 範本。**






