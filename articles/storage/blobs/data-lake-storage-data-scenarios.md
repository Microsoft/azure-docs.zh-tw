---
title: 有關 Azure Data Lake Storage Gen2 的資料案例 | Microsoft Docs
description: 了解可在 Data Lake Storage Gen2 (先前稱為 Azure Data Lake Store) 中用來內嵌、處理、下載及視覺化資料的各種案例和工具
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/14/2020
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: bffa7894f7603f95c4840019be5e5670797881df
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98873241"
---
# <a name="using-azure-data-lake-storage-gen2-for-big-data-requirements"></a>使用 Azure Data Lake Storage Gen2 處理巨量資料需求

巨量資料處理有四個主要階段︰

> [!div class="checklist"]
> * 即時或以批次形式將大量資料內嵌到存放區
> * 處理資料
> * 下載資料
> * 將資料視覺化

本文將重點放在每個處理階段的選項和工具。

如需可搭配 Azure Data Lake Storage Gen2 使用的 Azure 服務完整清單，請參閱 [整合 Azure Data Lake Storage 與 azure 服務](./data-lake-storage-supported-azure-services.md)

## <a name="ingest-the-data-into-data-lake-storage-gen2"></a>將資料內嵌至 Data Lake Storage Gen2

此節強調不同的資料來源，以及將資料內嵌到 Data Lake Storage Gen2 帳戶的各種方式。

![將資料內嵌到 Data Lake Storage Gen2](./media/data-lake-storage-data-scenarios/ingest-data.png "將資料內嵌到 Data Lake Storage Gen2")

### <a name="ad-hoc-data"></a>臨機操作資料

代表用來建立巨量資料應用程式原型的較小型資料集。 內嵌臨機操作資料的方式會因資料來源不同而有所差異。 

以下是您可用來擷取臨機操作資料的工具清單。

| 資料來源 | 內嵌方式 |
| --- | --- |
| 本機電腦 |[Azure PowerShell](data-lake-storage-directory-file-acl-powershell.md)<br><br>[Azure CLI](data-lake-storage-directory-file-acl-cli.md)<br><br>[儲存體總管](https://azure.microsoft.com/features/storage-explorer/)<br><br>[AzCopy 工具](../common/storage-use-azcopy-v10.md)|
| Azure 儲存體 Blob |[Azure Data Factory](../../data-factory/connector-azure-data-lake-store.md)<br><br>[AzCopy 工具](../common/storage-use-azcopy-v10.md)<br><br>[HDInsight 叢集上執行的 DistCp](data-lake-storage-use-distcp.md)|

### <a name="streamed-data"></a>串流資料

這代表可由各種來源（例如應用程式、裝置、感應器等）產生的資料。這項資料可透過各種不同的工具內嵌至 Data Lake Storage Gen2。 這些工具通常能以個別事件為基礎即時擷取及處理資料，然後再以批次將事件寫入 Data Lake Storage Gen2，以供進一步處理。

以下是您可用來擷取串流資料的工具清單。

|工具 | 指引 |
|---|--|
|Azure 串流分析|[快速入門：使用 Azure 入口網站建立串流分析作業](../../stream-analytics/stream-analytics-quick-create-portal.md) <br> [輸出至 Azure Data Lake Gen2](../../stream-analytics/stream-analytics-define-outputs.md)|
|Azure HDInsight Storm | [從 HDInsight 上的 Apache Storm 寫入 Apache Hadoop HDFS](../../hdinsight/storm/apache-storm-write-data-lake-store.md) |

### <a name="relational-data"></a>關聯式資料

您也可以從關聯式資料庫取得資料。 每經過一段時間，關聯式資料庫就會收集大量資料，在經過巨量資料管線處理後，這些資料將可提供重要情資。 您可以使用下列工具，將此類資料移動到 Data Lake Storage Gen2。

以下是您可用來擷取關聯式資料的工具清單。

|工具 | 指引 |
|---|--|
|Azure Data Factory | [Azure Data Factory 中的複製活動](../../data-factory/copy-activity-overview.md) |

### <a name="web-server-log-data-upload-using-custom-applications"></a>Web 伺服器記錄資料 (使用自訂應用程式上傳)

我們會特別強調此類資料集的原因在於，因為 Web 伺服器記錄資料的分析是巨量資料應用程式的常見使用案例，且需要將大量記錄檔上傳到 Data Lake Storage Gen2。 您可以使用以下任何工具來撰寫自己的指令碼或應用程式，以便上傳這類資料。

以下是您可用來擷取網頁伺服器記錄資料的工具清單。

|工具 | 指引 |
|---|--|
|Azure Data Factory | [Azure Data Factory 中的複製活動](../../data-factory/copy-activity-overview.md)  |
|Azure CLI|[Azure CLI](data-lake-storage-directory-file-acl-cli.md)|
|Azure PowerShell|[Azure PowerShell](data-lake-storage-directory-file-acl-powershell.md)|

若要上傳 Web 伺服器記錄資料及上傳其他類型的資料 (如社交情緒資料)，撰寫自己的自訂指令碼/應用程式是個不錯方法，因為您可以彈性地將自己的資料上傳元件納入較大型的巨量資料應用程式中。 在某些情況下，這段程式碼可能會採用指令碼或簡易命令列公用程式的形式。 在其他情況下，程式碼可用來將巨量資料處理整合到商務應用程式或解決方案中。

### <a name="data-associated-with-azure-hdinsight-clusters"></a>與 Azure HDInsight 叢集相關聯的資料

大部分的 HDInsight 叢集類型 (Hadoop、HBase、Storm) 能以資料儲存存放庫的形式支援 Data Lake Storage Gen2。 HDInsight 叢集能從 Azure 儲存體 Blob (WASB) 存取資料。 為了提高效能，您可以將資料從 WASB 複製到與叢集相關聯的 Data Lake Storage Gen2 帳戶。 您可以使用下列工具來複製資料。

以下是您可用來擷取 HDInsight 叢集相關資料的工具清單。

|工具 | 指引 |
|---|--|
|Apache DistCp | [使用 DistCp 在 Azure 儲存體 Blob 與 Azure Data Lake Storage Gen2 之間複製資料](./data-lake-storage-use-distcp.md) |
|AzCopy 工具 | [使用 AzCopy 轉送資料](../common/storage-use-azcopy-v10.md) |
|Azure Data Factory | [使用 Azure Data Factory 將資料複製到 Azure Data Lake Storage Gen2 或從中複製資料](../../data-factory/load-azure-data-lake-storage-gen2.md) |

### <a name="data-stored-in-on-premises-or-iaas-hadoop-clusters"></a>儲存於內部部署環境或 IaaS Hadoop 叢集中的資料

您可能會使用 HDFS，在本機電腦上將大量資料儲存於現有的 Hadoop 叢集中。 Hadoop 叢集可能位於內部部署環境中，也可能位於 Azure 上的 IaaS 叢集內。 可能有一些要以一次性方法或週期性方式將此類資料複製到 Azure Data Lake Storage Gen2 的需求。 有各種不同的選項可用來達到此目的。 以下是替代項目和相關考量的清單。

| 方法 | 詳細資料 | 優點 | 考量 |
| --- | --- | --- | --- |
| 使用 Azure Data Factory (ADF)，將資料從 Hadoop 叢集直接複製到 Azure Data Lake Storage Gen2 |[ADF 支援 HDFS 做為資料來源](../../data-factory/connector-hdfs.md) |ADF 針對 HDFS 提供全新支援，以及一流的端對端管理與監視 |需要將「資料管理閘道」部署在內部部署環境或 IaaS 叢集中 |
| 使用 Distcp，將資料從 Hadoop 複製到 Azure 儲存體。 接著，使用適當的機制將資料從 Azure 儲存體複製到 Data Lake Storage Gen2。 |您可以使用下列工具，將資料從 Azure 儲存體複製到 Data Lake Storage Gen2︰ <ul><li>[Azure Data Factory](../../data-factory/copy-activity-overview.md)</li><li>[AzCopy 工具](../common/storage-use-azcopy-v10.md)</li><li>[HDInsight 叢集上執行的 Apache DistCp](data-lake-storage-use-distcp.md)</li></ul> |您可以使用開放原始碼工具。 |牽涉到多種技術的多步驟程序 |

### <a name="really-large-datasets"></a>大型資料集

若要上傳動輒數 TB 的資料集，使用上述方法有時候可能會過於緩慢且昂貴。 在這種情況下，您可以使用 Azure ExpressRoute。  

Azure ExpressRoute 可讓您在 Azure 資料中心與內部部署的基礎結構之間建立私人連線。 這是傳輸大量資料的可靠選項。 若要深入了解，請參閱 [Azure ExpressRoute 文件](../../expressroute/expressroute-introduction.md)。

## <a name="process-the-data"></a>處理資料

一旦 Data Lake Storage Gen2 中的資料可以使用，您就可以使用支援的巨量資料應用程式來針對這些資料執行分析。 

![分析 Data Lake Storage Gen2 中的資料](./media/data-lake-storage-data-scenarios/analyze-data.png "分析 Data Lake Storage Gen2 中的資料")

以下是您可用來對儲存在 Data Lake Storage Gen2 中的資料執行資料分析工作的工具清單。

|工具 | 指引 |
|---|--|
|Azure HDInsight | [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen2](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md) |
|Azure Databricks | [Azure Data Lake Storage Gen2](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-datalake-gen2.html) \(部分機器翻譯\)<br><br>[快速入門：使用 Azure Databricks 分析 Azure Data Lake Storage Gen2 中的資料](./data-lake-storage-quickstart-create-databricks-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)<br><br>[教學課程：使用 Azure Databricks 擷取、轉換和載入資料](/azure/databricks/scenarios/databricks-extract-load-sql-data-warehouse?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) (機器翻譯)|

## <a name="visualize-the-data"></a>將資料視覺化

使用 Power BI 連接器來建立儲存在 Data Lake Storage Gen2 中之資料的視覺標記法。 請參閱 [使用 Power BI 來分析 Azure Data Lake Storage Gen2 中的資料](/power-query/connectors/datalakestorage)。

## <a name="download-the-data"></a>下載資料

在以下案例中，您可能也會想要從 Azure Data Lake Storage Gen2 下載資料或移動資料：

* 將資料移動到其他儲存機制，以便與現有的資料處理管線連結。 例如，您可能想要將資料從 Data Lake Storage Gen2 移至 Azure SQL Database 或 SQL Server 實例。

* 在建置應用程式原型時，將資料下載到本機電腦，以便在 IDE 環境中處理。

![從 Data Lake Storage Gen2 傳出資料](./media/data-lake-storage-data-scenarios/egress-data.png "從 Data Lake Storage Gen2 傳出資料")

以下是您可用來從 Data Lake Storage Gen2 下載資料的工具清單。

|工具 | 指引 |
|---|--|
|Azure Data Factory | [Azure Data Factory 中的複製活動](../../data-factory/copy-activity-overview.md) |
|Apache DistCp | [使用 DistCp 在 Azure 儲存體 Blob 與 Azure Data Lake Storage Gen2 之間複製資料](./data-lake-storage-use-distcp.md) |
|Azure 儲存體總管|[使用 Azure 儲存體總管來管理 Azure Data Lake Storage Gen2 中的目錄、檔案和 ACL](data-lake-storage-explorer.md) (機器翻譯)|
|AzCopy 工具|[使用 AzCopy 和 Blob 儲存體傳輸資料](../common/storage-use-azcopy-v10.md#transfer-data)|
