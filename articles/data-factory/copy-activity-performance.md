---
title: Azure Data Factory 中之複製活動的效能及微調指南 | Microsoft Docs
description: 了解當您使用複製活動時，Azure Data Factory 中的資料移動效能會受到哪些主要因素的影響。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/06/2018
ms.author: jingwang
ms.openlocfilehash: 958d1ea09ce4d85afc59af412e1050efc6290a1a
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39002240"
---
# <a name="copy-activity-performance-and-tuning-guide"></a>複製活動的效能及微調指南
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-copy-activity-performance.md)
> * [目前的版本](copy-activity-performance.md)


Azure Data Factory 複製活動會提供安全、可靠、高效能的頂級資料載入解決方案。 它可讓您複製每天在各式各樣雲端和內部部署資料存放區上數十 TB 的資料。 超快的資料載入效能是可確保您能夠專注於核心「巨量資料」問題的關鍵︰建置進階的分析解決方案，並從該所有資料獲得深入解析。

Azure 提供一組企業級資料儲存與資料倉儲解決方案，而「複製活動」則提供一個容易設定的高度最佳化資料載入體驗。 只要使用單一的複製活動，您便可以達成下列目的︰

* 以 **1.2 GBps** 的速度將資料載入 **Azure SQL 資料倉儲**。
* 以 **1.0 GBps** 的速度將資料載入 **Azure Blob 儲存體**
* 以 **1.0 GBps** 的速度將資料載入 **Azure Data Lake Store**

本文章說明：

* [效能參考數字](#performance-reference) ；
* 可在不同案例中 (包括[資料整合單位](#data-integration-units)、[平行複製](#parallel-copy)及[分段複製](#staged-copy)) 大幅提升複製輸送量的功能；
* [效能微調指導方針](#performance-tuning-steps) 。

> [!NOTE]
> 如果您大致來說並不熟悉複製活動，請先參閱[複製活動概觀](copy-activity-overview.md)再閱讀本文。
>

## <a name="performance-reference"></a>效能參考

下表顯示根據內部測試中「單一複製活動執行」之給定來源與接收配對的複製輸送量數字 (以 **MBps** 為單位)，以供參考。 為了進行比較，該表格也會示範[資料整合單位](#data-integration-units)或[自我裝載 Integration Runtime 延展性](concepts-integration-runtime.md#self-hosted-integration-runtime) (多個節點) 的不同設定如何協助複製效能。

![效能矩陣](./media/copy-activity-performance/CopyPerfRef.png)

> [!IMPORTANT]
> 當複製活動在 Azure Integration Runtime 上執行時，允許的最小資料整合單位 (先前稱為資料移動單位) 是兩個。 如果未指定，請參閱[資料整合單位](#data-integration-units)中所使用的預設資料整合單位。

注意事項：

* 輸送量的計算公式如下：[從來源讀取的資料大小]/[複製活動執行持續時間]。
* 表中的效能參考數字是使用單一複製活動執行裡的 [TPC-H](http://www.tpc.org/tpch/) 資料集來測量的。
* 在 Azure 資料存放區中，來源和接收位於相同 Azure 區域中。
* 對於內部部署和雲端資料存放區之間的混合式複製，每個自我裝載整合執行階段節點都在與資料存放區分開的機器上執行，該機器具有以下規格。 當僅執行單一活動，複製作業將只會取用測試電腦一小部分的 CPU、記憶體或網路頻寬。
    <table>
    <tr>
        <td>CPU</td>
        <td>32 核心 2.20 GHz Intel Xeon E5-2660 v2</td>
    </tr>
    <tr>
        <td>記憶體</td>
        <td>128 GB</td>
    </tr>
    <tr>
        <td>網路</td>
        <td>網際網路介面：10 Gbps；內部網路介面：40 Gbps</td>
    </tr>
    </table>


> [!TIP]
> 您可以利用比預設最大資料整合單位 (DIU) 更多的 DIU，也就是對雲端到雲端複製活動執行使用 32 單位，以達到更高的輸送量。 例如，使用 100 DIU，您就可以用 **1.0GBps** 的速率將資料從 Azure Blob 複製到 Azure Data Lake Store。 如需此功能和支援案例的詳細資訊，請參閱[資料整合單位](#data-integration-units)一節。 請連絡 [Azure 支援](https://azure.microsoft.com/support/)來要求更多 DIU。

## <a name="data-integration-units"></a>資料整合單位

**資料整合單位 (DIU)** (先前稱為雲端資料移動單位或 DMU) 是一項量值，代表 Data Factory 中單一單位的能力 (CPU、記憶體和網路資源配置的組合)。 **DIU 僅適用於 [Azure Integration Runtime](concepts-integration-runtime.md#azure-integration-runtime)**，而非[自我裝載 Integration Runtime](concepts-integration-runtime.md#self-hosted-integration-runtime)。

**複製活動執行所需的最小資料整合單位是兩個。** 如果未指定，下表列出用於不同複製案例中的預設 DIU：

| 複製案例 | 服務決定的預設 DIU |
|:--- |:--- |
| 在以檔案為基礎的存放區之間複製資料 | 依據檔案的數量和大小，介於 4 到 32 之間。 |
| 所有其他複製案例 | 4 |

若要覆寫此預設值，請如下所示指定 **dataIntegrationUnits** 屬性的值。 **dataIntegrationUnits** 屬性的**允許值****最高為 256**。 根據您的資料模式，複製作業會在執行階段使用的 **實際 DIU 數目** 等於或小於所設定的值。 如需在為特定複製來源和接收設定更多單位時可能獲得之效能增益水準的相關資訊，請參閱 [效能參考](#performance-reference)。

監視活動執行時，您可以在複製活動輸出中查看每個複製執行實際使用的資料整合單位。 請參閱[複製活動監視](copy-activity-overview.md#monitoring)了解詳細資料。

> [!NOTE]
> 如果您需要更多 DIU 以提高輸送量，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。 目前只有當您是將多個檔案從 Blob 儲存體/Data Lake Store/Amazon S3/雲端 FTP/雲端 SFTP 複製到任一其他雲端資料存放區時，設定 8 及 8 以上的值才有作用。
>

**範例：**

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "AzureDataLakeStoreSink"
            },
            "dataIntegrationUnits": 32
        }
    }
]
```

### <a name="data-integration-units-billing-impact"></a>資料整合單位計費影響

請 **務必** 要記住，您必須根據複製作業的總時間付費。 針對資料移動付費的總持續時間，是跨 DIU 的持續時間總和。 若過去某複製作業使用 2 個雲端單位花費 1 小時，現在使用 8 個雲端單位花費 15 分鐘，則兩者的整體費用幾乎相同。

## <a name="parallel-copy"></a>平行複製

您可以使用 **parallelCopies** 屬性，來指出您想要複製活動使用的平行處理原則。 您可以將此屬性視為複製活動內，可透過平行方式從來源讀取或寫入至接收資料存放區中的執行緒數目上限。

對於每個複製活動執行，Data Factory 會決定要用來從來源資料存放區複製資料到目的地資料存放區的平行複製數目。 它會使用的預設平行複製數目取決於您所使用的來源和接收類型：

| 複製案例 | 由服務決定的預設平行複製計數 |
| --- | --- |
| 在以檔案為基礎的存放區之間複製資料 |取決於檔案大小和用來在兩個雲端資料存放區之間複製資料的資料整合單位 (DIU) 數，或自我裝載 Integration Runtime 電腦的實體組態。 |
| 將資料從任何來源資料存放區複製到 Azure 表格儲存體 |4 |
| 所有其他複製案例 |1 |

[!TIP]
> 在檔案型存放區之間複製資料時，預設行為 (自動判定) 通常能提供最佳輸送量。 

若要控制裝載資料存放區之電腦上的負載或是調整複製效能，您可以選擇覆寫預設值並指定 **parallelCopies** 屬性的值。 值必須是大於或等於 1 的整數。 在執行階段，為獲得最佳效能，複製活動會使用小於或等於設定值的值。

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "AzureDataLakeStoreSink"
            },
            "parallelCopies": 32
        }
    }
]
```

注意事項：

* 在檔案型存放區之間複製資料時，**parallelCopies** 決定檔案層級的平行處理原則。 在底層則會自動且直接在單一檔案中進行區塊化，而這是設計來為指定的來源資料存放區類型使用最適合的區塊大小，以載入平行於和垂直於 parallelCopies 的資料。 資料移動服務在執行階段用於複製作業的實際平行複製數目不會超過您擁有的檔案數目。 如果複製行為是 **mergeFile**，則複製活動無法利用檔案層級的平行處理原則。
* 當您指定 **parallelCopies** 屬性的值時，請考慮來源和接收資料存放區的負載增加，以及自我裝載整合執行階段 (若複製活動 (例如混合式複製) 需要自我裝載整合執行階段) 的負載增加。 當您有多個活動或是會對相同資料存放區執行相同活動的並行執行時，尤其如此。 如果您注意到資料存放區或自我裝載整合執行階段已無法應付負載，請減少 **parallelCopies** 值以減輕負載。
* 將資料從非檔案型存放區複製到檔案型存放區時，資料移動服務會忽略 **parallelCopies** 屬性。 即使已指定平行處理原則，也不會套用於此案例。
* **parallelCopies** 正交於 **dataIntegrationUnits**。 前者會跨所有資料整合單位計算。

## <a name="staged-copy"></a>分段複製

從來源資料存放區將資料複製到接收資料存放區時，您可以選擇使用 Blob 儲存體做為過渡暫存存放區。 暫存在下列情況下特別有用︰

- **您想要透過 PolyBase 將資料從各種資料存放區內嵌至 SQL 資料倉儲**。 SQL 資料倉儲使用 PolyBase 做為高輸送量機制，將大量資料載入 SQL 資料倉儲。 不過，來源資料必須位於 Blob 儲存體或 Azure Data Lake Store，並符合額外的條件。 當您從 Blob 儲存體或 Azure Data Lake Store 以外的資料存放區載入資料時，您可以啟用透過過渡暫存 Blob 儲存體的資料複製。 在該情況下，Data Factory 會執行必要的資料轉換，以確保它符合 PolyBase 的需求。 然後，它會使用 PolyBase 將資料有效率地載入到 SQL 資料倉儲。 如需詳細資訊，請參閱[使用 PolyBase 將資料載入 Azure SQL 資料倉儲](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse)。
- **有時透過慢速網路連接執行混合式資料移動 (也就是，從內部部署資料存放區複製到雲端資料存放區)**，需要一段時間。 若要改善效能，您可以使用分段複製壓縮內部部署資料，將資料移到雲端中的暫存資料存放區所需的時間就會減少，然後在暫存存放區中先解壓資料後，再載入到目的地資料存放區。
- **由於公司 IT 原則，您不想要在您的防火牆中開啟 80 和 443 以外的連接埠**。 例如，從內部部署資料存放區將資料複製到 Azure SQL Database 接收或 Azure SQL 資料倉儲接收時，您必須針對 Windows 防火牆和公司防火牆啟用連接埠 1433 上的輸出 TCP 通訊。 在此案例中，分段複製可利用自我裝載整合執行階段，先透過 HTTP 或 HTTPS 在連接埠 443 將資料複製到 Blob 儲存體暫存執行個體，然後從 Blob 暫存儲存體將資料載入到 SQL Database 或 SQL 資料倉儲。 在此流程中，您不需要啟用連接埠 1433。

### <a name="how-staged-copy-works"></a>分段複製的運作方式

當您啟用暫存功能時，會先從來源資料存放區複製資料到暫存 Blob 儲存體 (自備)。 接著再從暫存資料存放區複製資料到接收資料存放區。 Data Factory 會自動為您管理 2 階段流程， 也會在資料移動完成之後，清除暫存儲存體中的暫存資料。

![分段複製](media/copy-activity-performance/staged-copy.png)

當您使用暫存存放區啟用資料移動時，您可以指定是否要在從來源資料存放區將資料移動至過渡或暫存資料存放區之前壓縮資料，然後在從過渡或暫存資料存放區將資料移動至接收資料存放區之前解壓縮資料。

目前您還無法使用暫存存放區在兩個內部部署資料存放區之間複製資料。

### <a name="configuration"></a>組態

在複製活動中設定 **enableStaging** 設定，指定您是否想要讓資料在載入至目的地資料存放區之前，暫存在 Blob 儲存體中。 當您將 **enableStaging** 設定為 `TRUE` 時，請指定下一份資料表所列出的其他屬性。 如果還未指定，您也需要建立 Azure 儲存體或儲存體共用存取簽章連結服務以供暫存使用。

| 屬性 | 說明 | 預設值 | 必要 |
| --- | --- | --- | --- |
| **enableStaging** |指定您是否要透過過渡暫存存放區複製資料。 |False |否 |
| **linkedServiceName** |指定 [AzureStorage](connector-azure-blob-storage.md#linked-service-properties) 連結服務的名稱，以代表您用來做為過渡暫存存放區的儲存體執行個體。 <br/><br/> 您無法使用具有共用存取簽章的儲存體來透過 PolyBase 將資料載入至 SQL 資料倉儲。 您可以將它用於其他所有案例。 |N/A |是，當 **enableStaging** 設為 TRUE |
| **路徑** |指定要包含分段資料的 Blob 儲存體路徑。 如果未提供路徑，服務會建立容器來儲存暫存資料。 <br/><br/> 只有在使用具有共用存取簽章的儲存體時，或需要讓暫存資料位於特定位置時，才指定路徑。 |N/A |否 |
| **enableCompression** |指定將資料複製到目的地之前，是否應該壓縮資料。 此設定可減少傳輸的資料量。 |False |否 |

以下是具有上表所述屬性的「複製活動」的範例定義︰

```json
"activities":[
    {
        "name": "Sample copy activity",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "stagingcontainer/path",
                "enableCompression": true
            }
        }
    }
]
```

### <a name="staged-copy-billing-impact"></a>分段複製的計費影響

我們將會根據兩個步驟向您收費：複製持續時間和複製類型。

* 在雲端複製期間使用暫存時 (從雲端資料存放區將資料複製到其他雲端資料存放區，兩個階段均使用 Azure 整合執行階段)，所收費用為 [步驟 1 和步驟 2 的複製持續時間總和] x [雲端複製單位價格]。
* 在混合式複製期間使用暫存時 (從內部部署資料存放區將資料複製到雲端資料存放區，一個階段使用自我裝載整合執行階段)，所收費用為 [混合式複製持續時間] x [混合式複製單位價格] + [雲端複製持續時間] x [雲端複製單位價格]。

## <a name="performance-tuning-steps"></a>效能微調步驟

建議您採取下列步驟來微調 Data Factory 服務搭配使用複製活動時的效能︰

1. **建立基準**。 在開發階段，對具有代表性的資料範例使用複製活動來測試您的管線。 依照[複製活動監視](copy-activity-overview.md#monitoring)的指示收集執行詳細資料和效能特性。

2. **效能診斷與最佳化**。 如果您觀察到的效能不符預期，您必須找出效能瓶頸。 然後將效能最佳化，以消除或減少瓶頸的影響。 效能診斷的完整說明不在本文的討論之列，但以下是一些常見的考量：

   * 效能功能︰
     * [平行複製](#parallel-copy)
     * [資料整合單位](#data-integration-units)
     * [分段複製](#staged-copy)
     * [自我裝載整合執行階段延展性](concepts-integration-runtime.md#self-hosted-integration-runtime)
   * [自我裝載整合執行階段](#considerations-for-self-hosted-integration-runtime)
   * [來源](#considerations-for-the-source)
   * [接收](#considerations-for-the-sink)
   * [序列化和還原序列化](#considerations-for-serialization-and-deserialization)
   * [壓縮](#considerations-for-compression)
   * [資料行對應](#considerations-for-column-mapping)
   * [其他考量](#other-considerations)

3. **將組態擴展至整個資料集**。 當您對執行結果及效能感到滿意時，您可以將定義和管線擴展為涵蓋整個資料集。

## <a name="considerations-for-self-hosted-integration-runtime"></a>自我裝載整合執行階段的考量

若您的複製活動是在自我裝載整合執行階段上執行，請注意下列事項：

**設定**：建議您使用專用的電腦來裝載整合執行階段。 請參閱[自我裝載整合執行階段的考量](concepts-integration-runtime.md)。

**相應放大**：具有一或多個節點的單一邏輯自我裝載整合執行階段，可同時為多個複製活動執行提供服務。 如果您對於混合式資料移動 (包含大量的並行複製活動執行或需要複製的大量資料) 有很大的需求，請考慮[相應放大自我裝載整合執行階段](create-self-hosted-integration-runtime.md#high-availability-and-scalability)，以充份利用您的資源，或佈建更多資源以賦予複製能力。

## <a name="considerations-for-the-source"></a>來源的考量

### <a name="general"></a>一般

請確定基礎資料存放區未被執行於其上或對其執行的其他工作負載全面佔據。

針對 Microsoft 資料存放區，請參閱資料存放區特定的 [監視和微調主題](#performance-reference) ，以協助您了解資料存放區的效能特性、最小化回應時間和最大化輸送量。

* 如果您要從 Blob 儲存體複製資料到 SQL 資料倉儲，請考慮使用 **PolyBase** 以提升效能。 如需詳細資訊，請參閱 [使用 PolyBase 將資料載入 Azure SQL 資料倉儲](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse) 。
* 如果您將資料從 HDFS 複製到 Azure Blob/Azure Data Lake Store，請考慮使用 **DistCp** 以提高效能。 如需詳細資料，請參閱[使用 DistCp 從 HDFS 複製資料](connector-hdfs.md#use-distcp-to-copy-data-from-hdfs)。
* 如果您將資料從 Redshift 複製到 Azure SQL 資料倉儲/Azure BLob/Azure Data Lake Store，請考慮使用 **UNLOAD** 以提高效能。 如需詳細資料，請參閱[使用 UNLOAD 複製 Amazon Redshift 中的資料](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift)。

### <a name="file-based-data-stores"></a>以檔案為基礎的資料存放區

* **平均檔案大小和檔案計數**：複製活動會一次傳送一個檔案的資料。 在要移動的資料量相同的前提下，如果資料包含許多個小型檔案而非少數幾個大型檔案，其整體輸送量會較低，因為每個檔案都需要啟動程序階段。 因此，可能的話，請將小型檔案合併為較大的檔案，以提高輸送量。
* **檔案格式和壓縮**：如需可改善效能的其他方法，請參閱[序列化和還原序列化的考量](#considerations-for-serialization-and-deserialization)和[壓縮的考量](#considerations-for-compression)小節。

### <a name="relational-data-stores"></a>關聯式資料存放區

* **資料模式**︰資料表結構描述對複製輸送量會有影響。 若要複製相同的資料量，較大的資料列大小會有優於較小資料列大小的效能。 這是因為資料庫可以更有效率地擷取包含較少資料列的較少資料批次。
* **查詢或預存程序**：最佳化您在複製活動來源中指定的查詢或預存程序邏輯，以更有效率地擷取資料。

## <a name="considerations-for-the-sink"></a>接收的考量

### <a name="general"></a>一般

請確定基礎資料存放區未被執行於其上或對其執行的其他工作負載全面佔據。

針對 Microsoft 資料存放區，請參閱資料存放區特定的 [監視和微調主題](#performance-reference) 。 這些主題可協助您了解資料存放區的效能特性，以及如何最小化回應時間和最大化輸送量。

* 如果您要從 Blob 儲存體複製資料到 SQL 資料倉儲，請考慮使用 **PolyBase** 以提升效能。 如需詳細資訊，請參閱 [使用 PolyBase 將資料載入 Azure SQL 資料倉儲](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-sql-data-warehouse) 。
* 如果您將資料從 HDFS 複製到 Azure Blob/Azure Data Lake Store，請考慮使用 **DistCp** 以提高效能。 如需詳細資料，請參閱[使用 DistCp 從 HDFS 複製資料](connector-hdfs.md#use-distcp-to-copy-data-from-hdfs)。
* 如果您將資料從 Redshift 複製到 Azure SQL 資料倉儲/Azure BLob/Azure Data Lake Store，請考慮使用 **UNLOAD** 以提高效能。 如需詳細資料，請參閱[使用 UNLOAD 複製 Amazon Redshift 中的資料](connector-amazon-redshift.md#use-unload-to-copy-data-from-amazon-redshift)。

### <a name="file-based-data-stores"></a>以檔案為基礎的資料存放區

* **複製行為**：如果您從其他以檔案為基礎的資料存放區複製資料，複製活動會透過 **copyBehavior** 屬性提供三個選項。 它會保留階層、扁平化階層或合併檔案。 保留或扁平化階層幾乎不會造成效能負荷，但合併檔案則會導致效能負荷增加。
* **檔案格式和壓縮**：如需可改善效能的其他方法，請參閱[序列化和還原序列化的考量](#considerations-for-serialization-and-deserialization)和[壓縮的考量](#considerations-for-compression)小節。

### <a name="relational-data-stores"></a>關聯式資料存放區

* **複製行為**：根據為 **sqlSink** 設定的屬性，複製活動會以不同的方式將資料寫入目的地資料庫中。
  * 根據預設，資料移動服務會使用大量複製 API 以附加模式插入資料，而提供最佳效能。
  * 如果您在接收中設定了預存程序，資料庫會一次套用一個資料列的資料 (而不是大量載入)。 因此效能會大幅降低。 如果資料集較大，在適用的情況下，請考慮改為使用 **preCopyScript** 屬性。
  * 如果您設定了每個複製活動執行的 **preCopyScript** 屬性，服務會觸發指令碼，然後使用大量複製 API 插入資料。 例如，若要以最新的資料覆寫整個資料表，您可以先指定指令碼以刪除所有記錄，再從來源大量載入新資料。
* **資料模式和批次大小**：
  * 資料表結構描述對複製輸送量會有影響。 若要複製相同的資料量，較大的資料列大小會有優於較小資料列大小的效能，因為資料庫可以更有效率地認可較少的資料批次。
  * 複製活動會以一系列的批次插入資料。 您可以使用 **writeBatchSize** 屬性來設定批次中包含的資料列數。 如果您的資料具有較小的資料列，您可以將 **writeBatchSize** 屬性設為較高的值，以減少批次的額外負荷，並增加輸送量。 如果您的資料的資料列大小較大，在增加 **writeBatchSize**時請多加留意。 較大的值可能會導致複製因資料庫的超載而失敗。

### <a name="nosql-stores"></a>NoSQL 存放區

* 針對 **表格儲存體**：
  * **資料分割**：將資料寫入至交錯的資料分割會大幅降低效能。 請依資料分割索引鍵來排序來源資料，使資料能有效率地依序插入資料分割中，或者，調整邏輯以將資料寫入單一資料分割中。

## <a name="considerations-for-serialization-and-deserialization"></a>序列化和還原序列化的考量

如果您的輸入資料集或輸出資料集是檔案，就可能發生序列化和還原序列化。 請參閱[支援的檔案和壓縮格式](supported-file-formats-and-compression-codecs.md)，其中具有關於複製活動支援檔案格式的詳細資訊。

**複製行為**：

* 在以檔案為基礎的資料存放區之間複製檔案：
  * 如果輸入和輸出資料集同時具有相同的檔案格式設定，或者都沒有這些設定，資料移動服務會執行「二進位複製」，而不執行任何序列化或還原序列化。 此情況的輸送量會高於來源和接收檔案格式設定彼此不同的案例。
  * 如果輸入和輸出資料集都是文字格式，只有編碼類型不同，則資料移動服務只會執行編碼轉換， 而不會執行任何序列化和還原序列化，因此和二進位複製相較之下，會產生一些效能負荷。
  * 如果輸入和輸出資料集皆有不同的檔案格式或不同的組態 (例如分隔符號)，則資料移動服務會將來源資料還原序列化，以進行串流、轉換然後再序列化為您所指出的輸出格式。 此作業會導致遠高於其他案例的效能負荷。
* 對 (從) 不是以檔案為基礎的資料存放區複製檔案時 (例如，從以檔案為基礎的存放區複製到關聯式存放區)，必須執行序列化或還原序列化步驟。 此步驟會導致很高的效能負荷。

**檔案格式**︰您選擇的檔案格式可能會影響複製效能。 例如，Avro 是一種壓縮二進位格式，可將中繼資料和資料儲存在一起。 它廣泛支援在 Hadoop 生態系統中進行處理和查詢。 不過，Avro 的序列化和還原序列化代價較高，因為它會導致低於文字格式的複製輸送量。 在選擇整個處理流程中所使用的檔案格式時，應有整體考量。 首先要考量資料的儲存形式是來源資料存放區或是要從外部系統擷取，再考量最理想的儲存、分析處理和查詢格式，以及資料應以何種格式匯出到資料超市中，以供報告和視覺化工具使用。 有些時候，在考量整體分析程序時，讀取和寫入效能次佳的檔案格式，可能會是較好的選擇。

## <a name="considerations-for-compression"></a>壓縮的考量

如果您的輸入或輸出資料集是檔案，您可以將複製活動設定為在資料寫入至目的地時執行壓縮或解壓縮。 當您選擇壓縮時，您必須在輸入/輸出 (I/O) 與 CPU 之間進行取捨。 壓縮資料須耗用額外的計算資源。 但另一方面卻可降低網路 I/O 和儲存體用量。 根據您的資料，您可能會看到整體複製輸送量有所提升。

**轉碼器**：每種壓縮轉碼器各有優點。 例如，bzip2 的複製輸送量最低，但您卻可以在使用 bzip2 時獲得最佳的 Hive 查詢效能，因為可將其劃分來進行處理。 Gzip 是最均衡的選項，也最常被使用。 請選擇最適合您的端對端案例使用的轉碼器。

**層級**：對於每個壓縮轉碼器，您可以從兩個選項中做選擇：最快速的壓縮和最佳化的壓縮。 最快速的壓縮選項能以最快速度壓縮資料，但產生的檔案不一定經過最理想的壓縮。 最佳化的壓縮選項會花費較長的壓縮時間，並產生最少量的資料。 您可以測試這兩個選項，以查看何者在您的案例中可提供更好的整體效能。

**考量事項**：若要在內部部署存放區與雲端之間複製大量資料，請考慮使用[分段複製](#staged-copy)，並同時啟用壓縮。 當公司網路與 Azure 服務的頻寬是限制因素，而且您想要讓輸入資料集和輸出資料集都是未壓縮的形式時，使用過渡儲存體會有所幫助。

## <a name="considerations-for-column-mapping"></a>資料行對應的考量

您可以在複製活動中設定 **columnMappings** 屬性以將所有或部分的輸入資料行對應至輸出資料行。 資料移動服務從來源讀取資料之後，必須先對資料執行資料行對應，再將資料寫入至接收。 這項額外處理會降低複製輸送量。

如果您的來源資料存放區是可查詢的 (例如，如果是 SQL Database 或 SQL Server 之類的關聯式存放區，或如果是表格儲存體或 Azure Cosmos DB 之類的 NoSQL 存放區)，請考慮將資料行篩選和重新排序邏輯推送至 **查詢** 屬性，而不使用資料行對應。 如此一來，便會在資料移動服務從來源資料存放區讀取資料時發生投射，而大幅提高效率。

深入了解[複製活動結構描述對應](copy-activity-schema-and-type-mapping.md)。

## <a name="other-considerations"></a>其他考量

如果您想要複製的資料大小較大，您可以調整商務邏輯以進一步分割資料，並排定更常執行複製活動，以減少每個複製活動執行的資料大小。

請密切留意資料集數目，以及要求 Data Factory 同時連線至相同資料存放區的複製活動。 許多並行複製作業可能會導致資料存放區出現瓶頸，並導致效能降低，複製作業內部重試，在某些情況下甚至導致執行失敗。

## <a name="sample-scenario-copy-from-an-on-premises-sql-server-to-blob-storage"></a>範例案例：從內部部署 SQL Server 複製到 Blob 儲存體

**案例：** 建置從內部部署 SQL Server 將資料以 CSV 格式複製到 Blob 儲存體的管線。 為了加快複製作業速度，CSV 檔案應該壓縮為 bzip2 格式。

**測試和分析**：複製活動的輸送量小於 2 MBps，遠低於效能基準。

**效能分析和微調**：為了排解效能問題，讓我們看看資料的處理及移動方式。

1. **讀取資料**：整合執行階段開啟對 SQL Server 的連接，並傳送查詢。 SQL Server 透過內部網路將資料流傳送至整合執行階段，以進行回應。
2. **序列化和壓縮資料**︰整合執行階段將資料流序列化為 CSV 格式，並將資料壓縮為 bzip2 資料流。
3. **寫入資料**：整合執行階段透過網際網路將 bzip2 資料流上傳至 Blob 儲存體。

如您所見，資料將會以串流序列的方式處理和移動：SQL Server -> LAN -> 整合執行階段 -> WAN -> Blob 儲存體。 **整體效能受限於管線的最小輸送量**。

![資料流](./media/copy-activity-performance/case-study-pic-1.png)

下列一或多個因素可能會造成效能瓶頸：

* **來源**：SQL Server 本身的輸送量偏低，因為負載過重。
* **自我裝載整合執行階段**：
  * **LAN**：整合執行階段的位置遠離 SQL Server 電腦，且頻寬連線較低。
  * **整合執行階段**：整合執行階段已達到其負載限制，而無法執行下列作業：
    * **序列化**：將資料流序列化為 CSV 格式時，輸送量偏低。
    * **壓縮**：您選擇了緩慢的壓縮轉碼器 (例如 bzip2，其採用 Core i7，速度為 2.8 MBps)。
  * **WAN**：公司網路與 Azure 服務之間的頻寬偏低 (例如，T1 = 1,544 kbps、T2 = 6,312 kbps)。
* **接收**：Blob 儲存體的輸送量低  (但不太可能發生，因為其 SLA 保證至少有 60 MBps)。

在此情況下，bzip2 資料壓縮可能會拖慢整個管線。 改用 gzip 壓縮轉碼器可能會緩解此瓶頸。

## <a name="reference"></a>參考

以下是幾個支援的資料存放區所適用的效能監視及調整參考：

* Azure 儲存體 (包括 Blob 儲存體和表格儲存體)：[Azure 儲存體的擴充性目標](../storage/common/storage-scalability-targets.md)和 [Azure 儲存體效能和擴充性檢查清單](../storage/common/storage-performance-checklist.md)
* Azure SQL Database：您可以 [監視效能](../sql-database/sql-database-single-database-monitor.md) ，並檢查資料庫交易單位 (DTU) 百分比
* Azure SQL 資料倉儲：其能力會以資料倉儲單位 (DWU) 來測量；請參閱 [管理 Azure SQL 資料倉儲中的計算能力 (概觀)](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)
* Azure Cosmos DB：[Azure Cosmos DB 中的效能等級](../cosmos-db/performance-levels.md)
* 內部部署 SQL Server： [效能的監視與微調](https://msdn.microsoft.com/library/ms189081.aspx)
* 內部部署檔案伺服器： [檔案伺服器的效能微調](https://msdn.microsoft.com/library/dn567661.aspx)

## <a name="next-steps"></a>後續步驟
請參閱其他複製活動文章：

- [複製活動概觀](copy-activity-overview.md)
- [複製活動結構描述對應](copy-activity-schema-and-type-mapping.md)
- [複製活動容錯](copy-activity-fault-tolerance.md)
