---
title: 大規模擷取、轉換和載入 (ETL) - Azure HDInsight
description: 了解如何在 HDInsight 中搭配 Hadoop 使用 ETL。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/14/2017
ms.author: ashishth
ms.openlocfilehash: 6af55bf8311efc14687ee16c3226988fd730081d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048997"
---
# <a name="extract-transform-and-load-etl-at-scale"></a>大規模擷取、轉換和載入 (ETL)

擷取、轉換和載入 (ETL) 是一個程序，其中會從各種來源取得資料、將資料收集在標準位置中、進行清理和處理，然後最終載入至資料存放區以供查詢。 傳統 ETL 會處理匯入資料、適當地清理該資料，然後將其儲存在關聯式資料引擎中。 使用 HDInsight 時，各種 Hadoop 生態系統元件都可支援大規模執行 ETL。 

ETL 程序中的 HDInsight 使用可以由以下管線來摘要說明：

![HDInsight ETL 概觀](./media/apache-hadoop-etl-at-scale/hdinsight-etl-at-scale-overview.png)

下列各節會探索每一個 ETL 階段及其相關元件。

## <a name="orchestration"></a>協調流程

協調流程會跨 ETL 管線的所有階段。 HDInsight 中的 ETL 作業通常涉及數個彼此搭配運作的不同產品。  您可以使用 Hive 來清理資料的某個部分，而使用 Pig 來清理另一個部分。  您可以使用 Azure Data Factory 將資料從 Azure Data Lake Store 載入至 Azure SQL Database。

若要在適當的時間執行適當的作業，則需要協調流程。

### <a name="oozie"></a>Oozie

Apache Oozie 是一個可管理 Hadoop 作業的工作流程協調系統。 Oozie 會在 HDInsight 叢集內執行，並與 Hadoop 堆疊整合。 Oozie 支援 Apache MapReduce、Apache Pig、Apache Hive 及 Apache Sqoop 的 Hadoop 作業。 Oozie 也可用來排定系統特定作業，例如 Java 程式或 Shell 指令碼。

如需詳細資訊，請參閱[使用 Oozie 搭配 Hadoop 在 HDInsight 上定義和執行工作流程](../hdinsight-use-oozie-linux-mac.md)。若要深入探討如何使用 Oozie 來驅動端對端管線，請參閱[使用資料管線](../hdinsight-operationalize-data-pipeline.md)。 

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory 以平台即服務的形式提供協調流程功能。 這是一項雲端式資料整合服務，可讓您在雲端建立資料驅動工作流程，以便協調及自動進行資料移動和資料轉換。 

使用 Azure Data Factory 時，您可以：

1. 建立並排定資料驅動工作流程 (稱為管線)，以從不同的資料存放區擷取資料。
2. 使用計算服務 (例如 Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics、Azure Batch 及 Azure Machine Learning) 來處理和轉換資料。
3. 將輸出資料發佈至資料存放區，例如 Azure SQL 資料倉儲，讓商業智慧 (BI) 應用程式取用。

如需有關 Azure Data Factory 的詳細資訊，請參閱[文件](../../data-factory/introduction.md)。

## <a name="ingest-file-storage-and-result-storage"></a>擷取檔案儲存體和結果儲存體

來源資料檔通常會載入至 Azure 儲存體或 Azure Data Lake Store 中的位置。 檔案可以是任何格式，但通常會是像 CSV 的一般檔案。 

### <a name="azure-storage"></a>Azure 儲存體 

[Azure 儲存體](https://azure.microsoft.com/services/storage/blobs/)具有[特定的延展性目標](../../storage/common/storage-scalability-targets.md)。  對大多數分析節點來說，當處理許多較小的檔案時，Azure 儲存體的延展性最佳。  Azure 儲存體可保證不論有多少檔案或檔案有多大 (只要是在您的限制範圍內) 都能提供相同的效能。  這意謂著您可以儲存數 TB 的資料而仍獲得一致的效能，不論您使用的是一部份資料還是全部資料。

Azure 儲存體有數種不同類型的 Blob。  「附加 Blob」是儲存 Web 記錄或感應器資料的絕佳選項。  

多個 Blob 可以分散到多部伺服器來擴大這些 Blob 的存取途徑，但是單一 Blob 則只能由單一伺服器來提供服務。 雖然 Blog 可以在 Blog 容器中邏輯分組，但這種分組方式的意涵卻非分割。

Azure 儲存體也有一個用於 Blob 儲存體的 WebHDFS API 層。  HDInsight 中的所有服務都可以存取「Azure Blob 儲存體」中的檔案來進行資料清理和資料處理，就像這些服務使用「Hadoop 分散式檔案系統」(HDFS).時的方式一樣。

通常會使用 PowerShell、Azure 儲存體 SDK 或 AZCopy 將資料擷取至 Azure 儲存體。

### <a name="azure-data-lake-store"></a>Azure Data Lake Store

Azure Data Lake Store (ADLS) 是一個受管理的超大規模存放庫，適用於與 HDFS 相容的分析資料。  ADLS 使用一種類似於 HDFS 的設計典範，並在總容量及個別檔案大小方面提供無限的延展性。 ADLS 非常適合與大型檔案搭配運作，因為大型檔案可以跨多個節點儲存。  分割 ADLS 中的資料是在幕後執行的。  您會獲得大規模輸送量，而可以使用數千個並行執行程式來執行分析作業，有效率地讀取和寫入數百 TB 的資料。

通常會使用 Azure Data Factory、ADLS SDK、AdlCopy 服務、Apache DistCp 或 Apache Sqoop 將資料擷取至 ADLS。  要使用這當中的哪些服務大部分取決於資料的所在位置。  如果資料目前在現有的 Hadoop 叢集中，您可以使用 Apache DistCp、AdlCopy 服務或 Azure Data Factory。  如果資料在「Azure Blob 儲存體」中，則您可以使用 Azure Data Lake Store .NET SDK、Azure PowerShell 或 Azure Data Factory。

ADLS 還針對使用「Azure 事件中樞」或 Apache Storm 來執行的事件擷取進行了最佳化。

#### <a name="considerations-for-both-storage-options"></a>兩種儲存體選項的考量

如果上傳的資料集達到數 TB，網路延遲可能會是主要問題，尤其是當資料來自內部部署位置時。  在這類情況下，您可以使用下列選項：

* Azure ExpressRoute：Azure ExpressRoute 可讓您在 Azure 資料中心與內部部署的基礎結構之間建立私人連線。 這些連線為傳輸大量資料提供一個可靠的選項。 如需詳細資訊，請參閱 [Azure ExpressRoute 文件](../../expressroute/expressroute-introduction.md)。

* 「離線」上傳資料。 您可以使用 [Azure 匯入/匯出服務](../../storage/common/storage-import-export-service.md)，將含有您資料的硬碟送到 Azure 資料中心。 您的資料會先上傳到 Azure 儲存體 Blob。 接下來，您可以使用 [Azure Data Factory](../../data-factory/connector-azure-data-lake-store.md) 或 [AdlCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md) 工具，將資料從 Azure 儲存體 Blob 複製到 Data Lake Store。

### <a name="azure-sql-data-warehouse"></a>Azure SQL 資料倉儲

Azure SQL DW 是一個儲存已清理且備妥之資料以供日後分析的絕佳選擇。  Azure HDInsight 可用來為 Azure SQL DW 執行這些服務。

「Azure SQL 資料倉儲」(SQL DW) 是一個已針對分析工作負載最佳化的關聯式資料庫。  Azure SQL DW 會根據分割資料表調整規模。  資料表可以跨多個節點進行分割。  在建立 Azure SQL DW 節點時便會選取節點。  您可以在事後調整節點規模，但這是一個可能需要移動資料的作用中程序。 如需詳細資訊，請參閱 [SQL 資料倉儲 - 管理計算](../../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)。

### <a name="hbase"></a>hbase

Apache HBase 是 Azure HDInsight 中提供的索引鍵/值存放區。  Apache HBase 是開放原始碼的 NoSQL 資料庫，其建置於 Hadoop 上並模仿 Google BigTable。 HBase 可針對無結構描述資料庫中依資料行系列組織的大量非結構化及半結構化資料，提供效能隨機存取功能和極高的一致性。

資料儲存在資料表的資料列中，而資料列中的資料會依據資料行系列進行分組。 HBase 是無結構描述的資料庫，也就是說，在使用資料行之前，並不需要先定義資料行和其中儲存之資料的類型。 開放原始碼的程式碼會以線性方式延展，以處理數千個節點上的 PB 資料。 HBase 可依賴 Hadoop 生態系統中分散式應用程式所提供的資料備援、批次處理及其他功能。   

HBase 是要日後進行分析之感應器和記錄資料絕佳目的地。

HBase 延展性取決於 HDInsight 叢集中的節點數目。

### <a name="azure-sql-database-and-azure-database"></a>Azure SQL Database 與 Azure 資料庫

Azure 以平台即服務 (PAAS) 的形式提供三種不同的關聯式資料庫。

* [Azure SQL Database](../../sql-database/sql-database-technical-overview.md) 是 Microsoft SQL Server 的實作。 如需有關效能的詳細資訊，請參閱[微調 Azure SQL Database 中的資料庫效能](../../sql-database/sql-database-performance-guidance.md)。
* [適用於 MySQL 的 Azure 資料庫](../../mysql/overview.md) 是 Oracle MySQL 的實作。
* [適用於 PostgreSQL 的 Azure 資料庫](../../postgresql/quickstart-create-server-database-portal.md) 是 PostgreSQL 的實作。

這些產品可以擴增規模，也就是說可以藉由新增更多 CPU 和記憶體來調整其規模。  您也可以選擇使用進階磁碟搭配這些產品來獲得較佳的 I/O 效能。

## <a name="azure-analysis-services"></a>Azure Analysis Services 

Azure Analysis Services (AAS) 是一個用於決策支援和業務分析的分析資料引擎，可為業務報告和用戶端應用程式 (例如 Power BI、Excel、Reporting Services 報告及其他視覺化工具) 提供分析資料。

您可以變更每個個別分析 Cube 的階層來調整這些 Cube。  如需詳細資訊，請參閱 [Azure Analysis Services 價格](https://azure.microsoft.com/pricing/details/analysis-services/)。

## <a name="extract-and-load"></a>擷取和載入

在資料存在於 Azure 中之後，您便可以使用許多服務來擷取它並載入至其他產品。  HDInsight 支援 Sqoop 和 Flume。 

### <a name="sqoop"></a>Sqoop

Apache Sqoop 是一個專為在結構化、半結構化及非結構化資料來源之間有效率地傳輸資料而設計的工具。 

Sqoop 使用 MapReduce 來匯入和匯出資料，可提供平行作業和容錯功能。

### <a name="flume"></a>Flume

Apache Flume 是一個分散式、可靠且可用的服務，可有效率地收集、彙總及移動大量的記錄資料。 Flume 具有以串流資料流程為基礎的簡單彈性架構。 Flume 提供可微調的可靠性機制及許多容錯移轉與復原機制，既健全又能容錯。 Flume 使用已將線上分析應用程式納入考量的簡單可延伸資料模型。

Apache Flume 無法與 Azure HDInsight 搭配使用。  內部部署 Hadoop 安裝可以使用 Flume，將資料傳送至 Azure 儲存體 Blob 或 Azure Data Lake Store。  如需詳細資訊，請參閱[搭配 HDInsight 使用 Apache Flume](https://blogs.msdn.microsoft.com/bigdatasupport/2014/03/18/using-apache-flume-with-hdinsight/) \(英文\)。

## <a name="transform"></a>轉換

在資料存在於所選擇的位置中之後，您必須加以清理、結合或準備，才能用於特定使用模式。  Hive、Pig 及 Spark SQL 都是適用於該類工作的絕佳選擇。  HDInsight 支援所有這些產品。 

## <a name="next-steps"></a>後續步驟

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)
* [使用 Apache Hive 作為 ETL 工具](apache-hadoop-using-apache-hive-as-an-etl-tool.md) 
