---
title: 什麼是 Azure HDInsight
description: HDInsight 及 Apache Hadoop 與 Apache Spark 技術堆疊和元件簡介，包括用於巨量資料分析的 Kafka、Hive、Storm 和 HBase。
ms.service: hdinsight
ms.topic: overview
ms.custom: contperf-fy21q1
ms.date: 08/21/2020
ms.openlocfilehash: d1c32bf749850ac40e23c1a9cb9c5cd7755d45c6
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98939439"
---
# <a name="what-is-azure-hdinsight"></a>什麼是 Azure HDInsight？

Azure HDInsight 是雲端中供企業使用的受控、全方位的開放原始碼分析服務。 您可以使用開放原始碼架構，例如 Hadoop、Apache Spark、Apache Hive、LLAP、Apache Kafka、Apache Storm、R 等等。

## <a name="what-is-hdinsight-and-the-hadoop-technology-stack"></a>什麼是 HDInsight 和 Hadoop 技術堆疊？

Azure HDInsight 是 Hadoop 元件的雲端發行版。 Azure HDInsight 可輕鬆快速地處理大量資料，同時節省成本。 您可以使用最熱門的開放原始碼架構，例如 Hadoop、Spark、Hive、LLAP、Kafka、Storm、R 等等。 透過這些架構，您可以啟用各種案例，例如擷取、轉換和載入 (ETL)、資料倉儲、機器學習和 IoT。

若要查看 HDInsight 上可用的 Hadoop 技術堆疊元件，請參閱 [HDInsight 可用的元件和版本](./hdinsight-component-versioning.md)。 若要深入了解 HDInsight 中的 Hadoop，請參閱 [HDInsight 的 Azure 功能頁面](https://azure.microsoft.com/services/hdinsight/)。

## <a name="what-is-big-data"></a>什麼是巨量資料？

比起以往，巨量資料的收集量快速增加，收集速度加快，收集格式也愈來愈多。 它可以屬於過去 (亦即已儲存) 或即時 (亦即從來源串流而來)。 請參閱[使用 HDInsight 的案例](#scenarios-for-using-hdinsight)，了解巨量資料的最常見使用案例。

## <a name="why-should-i-use-azure-hdinsight"></a>為什麼應該使用 Azure HDInsight？

此區段會列出 Azure HDInsight 的功能。

|功能  |描述  |
|---------|---------|
|雲端原生     |     Azure HDInsight 可讓您在 Azure 上建立 Hadoop、Spark、 [互動式查詢 (LLAP)](./interactive-query/apache-interactive-query-get-started.md)、Kafka、Storm、HBase 和 ML 服務的最佳化叢集。 HDInsight 也提供所有生產工作負載的端對端 SLA。  |
|低成本且可調整     | HDInsight 可讓您相應增加或減少工作負載。 您可以依照需求建立叢集，且只支付您所使用的部分來降低成本。 您也可以建置資料管線來施行您的作業。 低耦合的計算和儲存體可提供更好的效能和彈性。 |
|安全且符合規範    | HDInsight 可讓您使用 Azure 虛擬網路、加密，以及與 Azure Active Directory 整合來保護企業資料資產。 HDInsight 也符合最受歡迎的產業和政府合規性標準。        |
|監視    | Azure HDInsight 與 Azure 監視器記錄整合後會提供單一介面，以便監視所有的叢集。        |
|正式上市 | HDInsight 的適用區域超過任何其他巨量資料分析供應項目。 Azure HDInsight 也會適用於 Azure Government、中國和德國，可讓您符合您在重要主權區域中的企業需求。 |  
|生產力     |  Azure HDInsight 可讓您在慣用的開發環境中，使用多種 Hadoop 與 Spark 的生產工具。 這些開發環境包括適用於 Scala、Python、R、Java 和 .NET 支援的 Visual Studio、VSCode、Eclipse 和 IntelliJ。 資料科學家也可以使用熱門 Notebook (例如 Jupyter 和 Zeppelin) 共同作業。    |
|擴充性     |  您可以透過使用指令碼動作安裝的元件 (Hue、Presto 等)、新增邊緣節點，或與其他巨量資料認證的應用程式整合，來擴充 HDInsight 叢集。 透過單鍵部署，HDInsight 即可與最受歡迎的巨量資料解決方案緊密整合。|

## <a name="scenarios-for-using-hdinsight"></a>使用 HDInsight 的案例

Azure HDInsight 可在巨量資料處理的各種案例中使用。 它可以是歷程資料 (已收集及儲存的資料) 或即時資料 (從來源直接串流處理的資料)。 下列類別概述處理這類資料的案例：

### <a name="batch-processing-etl"></a>批次處理 (ETL)

在擷取、轉換及載入 (ETL) 程序中，非結構化或結構化資料會擷取自異質資料來源。 然後轉換成結構化格式，並載入資料存放區。 您可以將已轉換的資料用於資料科學或資料倉儲上。

### <a name="data-warehousing"></a>資料倉儲

您可以使用 HDInsight 對任何格式的結構化或非結構化資料執行 PB 規模的互動式查詢。 您也可以建置模型，將這些查詢連線至 BI 工具。

HDInsight 架構：資料倉儲

### <a name="internet-of-things-iot"></a>物聯網 (IoT)

您可以使用 HDInsight 來處理從不同裝置類型即時接收的串流資料。 如需詳細資訊，請[閱讀 Azure 的此部落格文章，其中宣佈了在 HDInsight 上使用 Azure 受控磁碟的 Apache Kafka 公開預覽](https://azure.microsoft.com/blog/announcing-public-preview-of-apache-kafka-on-hdinsight-with-azure-managed-disks/)。

HDInsight 架構：物聯網

### <a name="data-science"></a>資料科學

您可以使用 HDInsight 建置應用程式，從資料中擷取重要的深入解析。 您也可以在其上使用 Azure Machine Learning，以預測您未來的業務趨勢。 如需詳細資訊，請參閱[此客戶案例](https://customers.microsoft.com/story/pros)。

HDInsight 架構：資料科學

### <a name="hybrid"></a>混合式

您可以使用 HDInsight 將現有的內部部署巨量資料基礎結構延伸至 Azure，充分利用雲端的進階分析功能。

HDInsight 架構：混合式

## <a name="cluster-types-in-hdinsight"></a>HDInsight 中的叢集類型

HDInsight 包含特定叢集類型和叢集自訂功能，例如新增元件、公用程式及語言的功能。 HDInsight 提供下列叢集類型：

|叢集類型 | 描述 |
|---|---|
|[Apache Hadoop](./hadoop/apache-hadoop-introduction.md)|使用 HDFS、YARN 資源管理和簡單 MapReduce 程式設計模型的架構，用來平行處理和分析批次資料。|
|[Apache Spark](./spark/apache-spark-overview.md)|一個開放原始碼平行處理架構，可支援記憶體內部處理，大幅提升巨量資料分析應用程式的效能。 請參閱[什麼是 HDInsight 中的 Apache Spark？](./spark/apache-spark-overview.md)。|
|[Apache HBase](./hbase/apache-hbase-overview.md)|建置於 Hadoop 上的 NoSQL 資料庫，可針對大量非結構化及半結構化資料，提供隨機存取功能和強大一致性 - 可能是數十億的資料列乘以數十億的資料行。 請參閱[什麼是 HDInsight 上的 HBase？](./hbase/apache-hbase-overview.md)|
|[ML 服務](./r-server/r-server-overview.md)|可用來裝載和管理並行、分散式 R 程序的伺服器。 這項新功能可讓資料科學家、統計學家以及 R 程式設計人員隨其所需存取 HDInsight 上可調整大小的分散式分析方法。 請參閱 [HDInsight 上的 ML 服務概觀](./r-server/r-server-overview.md)。|
|[Apache Storm](./storm/apache-storm-overview.md)|分散式、即時的運算系統，可快速處理大型的資料串流。 Storm 以受控叢集的形式在 HDInsight 中提供。 請參閱＜ [使用 Storm 和 Hadoop 來分析即時感應器資料](./storm/apache-storm-overview.md)＞。|
|[Apache 互動式查詢](./interactive-query/apache-interactive-query-get-started.md)|更快速之互動式 Hive 查詢的記憶體內快取。 請參閱[在 HDInsight 中使用互動式查詢](./interactive-query/apache-interactive-query-get-started.md)。|
|[Apache Kafka](./kafka/apache-kafka-introduction.md)|用來建立串流資料管線和應用程式的開放原始碼平台。 Kafka 也提供訊息佇列功能，可讓您發佈和訂閱資料串流。 請參閱 [HDInsight 上的 Apache Kafka 簡介](./kafka/apache-kafka-introduction.md)。|

## <a name="open-source-components-in-hdinsight"></a>HDInsight 中的開放原始碼元件

Azure HDInsight 可讓您透過開放原始碼架構 (例如 Hadoop、Spark、Hive、LLAP、Kafka、Storm、HBase 和 R) 建立叢集。根據預設，這些叢集隨附叢集上包含的其他開放原始碼元件，例如 Apache Ambari5、Avro5、Apache Hive3、HCatalog2、Apache Mahout2、Apache Hadoop MapReduce3、Apache Hadoop YARN2、Apache Phoenix3、Apache Pig3、Apache Sqoop3、Apache Tez3、Apache Oozie2 和 Apache ZooKeeper5。  

## <a name="programming-languages-in-hdinsight"></a>HDInsight 中的程式設計語言

HDInsight 叢集 (包括 Spark、HBase、Kafka、Hadoop 等) 支援許多種程式設計語言。 某些程式設計語言並未預設安裝。 針對未預設安裝的程式庫、模組或套件，請使用指令碼動作來安裝元件。

|程式設計語言  |資訊  |
|---------|---------|
|預設的程式設計語言支援     | 根據預設，HDInsight 叢集可支援：<ul><li>Java</li><li>Python</li><li>.NET</li><li>Go</li></ul>  |
|Java 虛擬機器 (JVM) 語言     | Java 虛擬機器 (JVM) 上可以執行許多 Java 以外的語言。 不過，如果您要執行這些語言，您可能必須在叢集上安裝其他元件。 HDInsight 叢集上支援下列以 JVM 為基礎的語言： <ul><li>Clojure</li><li>Jython (適用於 Java 的 Python)</li><li>Scala</li></ul>     |
|Hadoop 專屬語言     | HDInsight 叢集支援下列 Hadoop 技術堆疊專屬語言： <ul><li>適用於 Pig 工作的 Pig Latin</li><li>適用於 Hive 工作和 SparkSQL 的 HiveQL</li></ul>        |

## <a name="development-tools-for-hdinsight"></a>適用於 HDInsight 的開發工具

您可以使用 HDInsight 開發工具 (包括 IntelliJ、Eclipse、Visual Studio Code 和 Visual Studio)，透過與 Azure 的完美整合，以撰寫並提交 HDInsight 資料查詢和作業。

* 適用於 IntelliJ10 的 Azure 工具組
* 適用於 Eclipse6 的 Azure 工具組
* 適用於 VS Code13 的 Azure HDInsight 工具
* 適用於 Visual Studio9 的 Azure Data Lake 工具

## <a name="business-intelligence-on-hdinsight"></a>HDInsight 上的商業智慧

熟悉的商業智慧 (BI) 工具可使用 Power Query 增益集或 Microsoft Hive ODBC 驅動程式來擷取、分析和報告與 HDInsight 整合的資料：

* [使用資料視覺效果工具搭配 Azure HDInsight 的 Apache Spark BI](./spark/apache-spark-use-bi-tools.md)

* [在 Azure HDInsight 中使用 Microsoft Power BI 將 Apache Hive 資料視覺化](./hadoop/apache-hadoop-connect-hive-power-bi.md)

* [在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](./interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)

* [使用 Power Query 將 Excel 連線到 Apache Hadoop](./hadoop/apache-hadoop-connect-excel-power-query.md) (需要 Windows)

* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 Apache Hadoop](./hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md) (需要 Windows)


## <a name="in-region-data-residency"></a>區域內資料落地 

Spark、Hadoop、LLAP、Storm 和 MLService 不會儲存客戶資料，因此這些服務會自動滿足區域內的資料落地需求，包括 [信任中心](https://azuredatacentermap.azurewebsites.net/)中指定的需求。 

Kafka 和 HBase 會儲存客戶資料。 Kafka 和 HBase 會自動將此資料儲存在單一區域中，因此這項服務可滿足區域內的資料落地需求，包括 [信任中心](https://azuredatacentermap.azurewebsites.net/)中指定的需求。 


熟悉的商業智慧 (BI) 工具可使用 Power Query 增益集或 Microsoft Hive ODBC 驅動程式來擷取、分析和報告與 HDInsight 整合的資料。

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 中建立 Apache Hadoop 叢集](./hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* 建立 Apache Spark 叢集 - 入口網站
* [Azure HDInsight 中的企業安全性](./domain-joined/hdinsight-security-overview.md)
