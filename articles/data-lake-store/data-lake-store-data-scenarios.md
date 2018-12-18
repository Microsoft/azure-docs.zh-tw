---
title: 有關 Data Lake Storage Gen1 的資料案例 | Microsoft Docs
description: 了解可在 Data Lake Storage Gen1 (先前稱為 Azure Data Lake Store) 中用來內嵌、處理、下載及視覺化資料的各種案例和工具
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: nitinme
ms.openlocfilehash: 64c7985508ed7f03b32340cbb2c78de61242f7e1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984274"
---
# <a name="using-azure-data-lake-storage-gen1-for-big-data-requirements"></a>使用 Azure Data Lake Storage Gen1 處理巨量資料需求

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

巨量資料處理有四個主要階段︰

* 即時或以批次形式將大量資料內嵌到存放區
* 處理資料
* 下載資料
* 將資料視覺化

在此文章中，我們探討這些與 Azure Data Lake Storage Gen1 有關的階段，以了解可用來滿足您巨量資料需求的選項與工具。

## <a name="ingest-data-into-data-lake-storage-gen1"></a>將資料內嵌到 Data Lake Storage Gen1
此節強調不同的資料來源，以及將資料內嵌到 Data Lake Storage Gen1 帳戶的各種方式。

![將資料內嵌到 Data Lake Storage Gen1](./media/data-lake-store-data-scenarios/ingest-data.png "將資料內嵌到 Data Lake Storage Gen1")

### <a name="ad-hoc-data"></a>臨機操作資料
代表用來建立巨量資料應用程式原型的較小型資料集。 內嵌臨機操作資料的方式會因資料來源不同而有所差異。

| 資料來源 | 內嵌方式 |
| --- | --- |
| 本機電腦 |<ul> <li>[Azure 入口網站](data-lake-store-get-started-portal.md)</li> <li>[Azure PowerShell](data-lake-store-get-started-powershell.md)</li> <li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li> <li>[使用適用於 Visual Studio 的 Data Lake Tools](../data-lake-analytics/data-lake-analytics-data-lake-tools-get-started.md) </li></ul> |
| Azure 儲存體 Blob |<ul> <li>[Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)</li> <li>[AdlCopy 工具](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[HDInsight 叢集上執行的 DistCp](data-lake-store-copy-data-wasb-distcp.md)</li> </ul> |

### <a name="streamed-data"></a>串流資料
代表可能由應用程式、裝置、感應器等各種來源產生的資料。這些資料可透過多種工具內嵌到 Data Lake Storage Gen1。 這些工具通常能以個別事件為基礎即時擷取及處理資料，然後再以批次將事件寫入 Data Lake Storage Gen1，以供進一步處理。

以下是您可以使用的工具︰

* [Azure 串流分析](../stream-analytics/stream-analytics-data-lake-output.md) - 內嵌到「事件中樞」的事件可以透過使用 Azure Data Lake Storage Gen1 輸出寫入到 Azure Data Lake Storage Gen1。
* [Azure HDInsight Storm](../hdinsight/storm/apache-storm-write-data-lake-store.md) - 您可以從 Storm 叢集將資料直接寫入 Data Lake Storage Gen1 中。
* [EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-receive-eph.md) - 您可以從「事件中樞」接收事件，然後使用 [Data Lake Storage Gen1 .NET SDK](data-lake-store-get-started-net-sdk.md) 將事件寫入到 Data Lake Storage Gen1。

### <a name="relational-data"></a>關聯式資料
您也可以從關聯式資料庫取得資料。 每經過一段時間，關聯式資料庫就會收集大量資料，在經過巨量資料管線處理後，這些資料將可提供重要情資。 您可以使用下列工具，將此類資料移動到 Data Lake Storage Gen1。

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md)

### <a name="web-server-log-data-upload-using-custom-applications"></a>Web 伺服器記錄資料 (使用自訂應用程式上傳)
我們會特別強調此類資料集的原因在於，因為 Web 伺服器記錄資料的分析是巨量資料應用程式的常見使用案例，且需要將大量記錄檔上傳到 Data Lake Storage Gen1。 您可以使用以下任何工具來撰寫自己的指令碼或應用程式，以便上傳這類資料。

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [Azure Data Lake Storage Gen1 .NET SDK](data-lake-store-get-started-net-sdk.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md)

若要上傳 Web 伺服器記錄資料及上傳其他類型的資料 (如社交情緒資料)，撰寫自己的自訂指令碼/應用程式是個不錯方法，因為您可以彈性地將自己的資料上傳元件納入較大型的巨量資料應用程式中。 在某些情況下，這段程式碼可能會採用指令碼或簡易命令列公用程式的形式。 在其他情況下，程式碼可用來將巨量資料處理整合到商務應用程式或解決方案中。

### <a name="data-associated-with-azure-hdinsight-clusters"></a>與 Azure HDInsight 叢集相關聯的資料
大部分的 HDInsight 叢集類型 (Hadoop、HBase、Storm) 能以資料儲存存放庫的形式支援 Data Lake Storage Gen1。 HDInsight 叢集能從 Azure 儲存體 Blob (WASB) 存取資料。 為了提高效能，您可以將資料從 WASB 複製到與叢集相關聯的 Data Lake Storage Gen1 帳戶。 您可以使用下列工具來複製資料。

* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)
* [AdlCopy 服務](data-lake-store-copy-data-azure-storage-blob.md)
* [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)

### <a name="data-stored-in-on-premises-or-iaas-hadoop-clusters"></a>儲存於內部部署環境或 IaaS Hadoop 叢集中的資料
您可能會使用 HDFS，在本機電腦上將大量資料儲存於現有的 Hadoop 叢集中。 Hadoop 叢集可能位於內部部署環境中，也可能位於 Azure 上的 IaaS 叢集內。 可能有一些要以一次性方法或週期性方式將此類資料複製到 Azure Data Lake Storage Gen1 的需求。 有各種不同的選項可用來達到此目的。 以下是替代項目和相關考量的清單。

| 方法 | 詳細資料 | 優點 | 考量 |
| --- | --- | --- | --- |
| 使用 Azure Data Factory (ADF)，將資料從 Hadoop 叢集直接複製到 Azure Data Lake Storage Gen1 |[ADF 支援 HDFS 做為資料來源](../data-factory/connector-hdfs.md) |ADF 針對 HDFS 提供全新支援，以及一流的端對端管理與監視 |需要將「資料管理閘道」部署在內部部署環境或 IaaS 叢集中 |
| 從 Hadoop 將資料匯出為檔案。 接著，使用適當的機制將檔案複製到 Azure Data Lake Storage Gen1。 |您可以使用下列工具，將檔案複製到 Azure Data Lake Storage Gen1︰ <ul><li>[適用於 Windows OS 的 Azure PowerShell](data-lake-store-get-started-powershell.md)</li><li>[Azure CLI](data-lake-store-get-started-cli-2.0.md)</li><li>使用任何 Data Lake Storage Gen1 SDK 的自訂應用程式</li></ul> |快速開始使用。 可以執行自訂的上傳 |牽涉到多種技術的多步驟程序。 考慮到自訂的工具性質，管理和監視會在經過一段時間之後逐漸變成是一項挑戰 |
| 使用 Distcp，將資料從 Hadoop 複製到 Azure 儲存體。 接著，使用適當的機制將資料從 Azure 儲存體複製到 Data Lake Storage Gen1。 |您可以使用下列工具，將資料從 Azure 儲存體複製到 Data Lake Storage Gen1︰ <ul><li>[Azure Data Factory](../data-factory/copy-activity-overview.md)</li><li>[AdlCopy 工具](data-lake-store-copy-data-azure-storage-blob.md)</li><li>[HDInsight 叢集上執行的 Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)</li></ul> |您可以使用開放原始碼工具。 |牽涉到多種技術的多步驟程序 |

### <a name="really-large-datasets"></a>大型資料集
若要上傳動輒數 TB 的資料集，使用上述方法有時候可能會過於緩慢且昂貴。 此時，您可以使用下列選項。

* **使用 Azure ExpressRoute**。 Azure ExpressRoute 可讓您在 Azure 資料中心與內部部署的基礎結構之間建立私人連線。 這是傳輸大量資料的可靠選項。 如需詳細資訊，請參閱 [Azure ExpressRoute 文件](../expressroute/expressroute-introduction.md)。
* **「離線」上傳資料**。 如果因為任何原因而無法使用 Azure ExpressRoute，您可以使用 [Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md) ，將含有您資料的硬碟送到 Azure 資料中心。 您的資料會先上傳到 Azure 儲存體 Blob。 接著，您可以使用 [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) 或 [AdlCopy 工具](data-lake-store-copy-data-azure-storage-blob.md)，將資料從 Azure 儲存體 Blob 複製到 Data Lake Storage Gen1。

  > [!NOTE]
  > 使用「匯入/匯出」服務時，運送到 Azure 資料中心之磁碟上的檔案大小應不大於 195 GB。
  >
  >

## <a name="process-data-stored-in-data-lake-storage-gen1"></a>處理儲存在 Data Lake Storage Gen1 中的資料
一旦 Data Lake Storage Gen1 中的資料可以使用，您就可以使用支援的巨量資料應用程式來針對這些資料執行分析。 目前，您可以使用 Azure HDInsight 和 Azure Data Lake 分析來針對儲存在 Data Lake Storage Gen1中的資料執行資料分析工作。

![分析 Data Lake Storage Gen1 中的資料](./media/data-lake-store-data-scenarios/analyze-data.png "分析 Data Lake Storage Gen1 中的資料")

您可以查看下列範例。

* [建立以 Data Lake Storage Gen1 作為儲存體的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-portal.md)
* [搭配 Data Lake Storage Gen1 使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

## <a name="download-data-from-data-lake-storage-gen1"></a>從 Data Lake Storage Gen1 下載資料
在以下案例中，您可能也會想要從 Azure Data Lake Storage Gen1 下載資料或移動資料：

* 將資料移動到其他存放庫，以便與現有的資料處理管線連結。 例如，您可能會想要將資料從 Data Lake Storage Gen1 移動到 Azure SQL Database 或內部部署 SQL Server。
* 在建置應用程式原型時，將資料下載到本機電腦，以便在 IDE 環境中處理。

![從 Data Lake Storage Gen1 輸出資料](./media/data-lake-store-data-scenarios/egress-data.png "從 Data Lake Storage Gen1 輸出資料")

在這些案例中，您可以使用下列任何選項。

* [Apache Sqoop](data-lake-store-data-transfer-sql-sqoop.md)
* [Azure Data Factory](../data-factory/copy-activity-overview.md)
* [Apache DistCp](data-lake-store-copy-data-wasb-distcp.md)

您也可以使用下列方法來撰寫自己的指令碼/應用程式，以便從 Data Lake Storage Gen1 下載資料。

* [Azure CLI](data-lake-store-get-started-cli-2.0.md)
* [Azure PowerShell](data-lake-store-get-started-powershell.md)
* [Azure Data Lake Storage Gen1 .NET SDK](data-lake-store-get-started-net-sdk.md)

## <a name="visualize-data-in-data-lake-storage-gen1"></a>視覺化 Data Lake Storage Gen1 中的資料
您可以混合使用多種服務，利用視覺化的方式呈現儲存在 Data Lake Storage Gen1 中的資料。

![視覺化 Data Lake Storage Gen1 中的資料](./media/data-lake-store-data-scenarios/visualize-data.png "視覺化 Data Lake Storage Gen1 中的資料")

* 您可以從使用 [Azure Data Factory 將資料從 Data Lake Storage Gen1 移到 Azure SQL 資料倉儲](../data-factory/copy-activity-overview.md)開始
* 之後，您可以 [將 Power BI 與 Azure SQL 資料倉儲整合](../sql-data-warehouse/sql-data-warehouse-get-started-visualize-with-power-bi.md) ，以視覺化方式呈現資料。
