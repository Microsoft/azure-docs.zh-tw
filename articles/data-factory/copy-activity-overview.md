---
title: Azure Data Factory 中的複製活動 | Microsoft Docs
description: 了解 Azure Data Factory 中的複製活動，可供您將資料從支援的來源資料存放區複製到支援的接收資料存放區。
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
ms.date: 06/15/2018
ms.author: jingwang
ms.openlocfilehash: 8e34b0823b7f10455ac0b66fb0614d3946f2382e
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38542698"
---
# <a name="copy-activity-in-azure-data-factory"></a>Azure Data Factory 中的複製活動

## <a name="overview"></a>概觀

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-data-movement-activities.md)
> * [目前的版本](copy-activity-overview.md)

在 Azure Data Factory 中，您可以使用「複製活動」在內部部署與雲端資料存放區之間複製資料。 複製資料之後，可以將它進一步轉換及進行分析。 您也可以使用「複製活動」來發佈商業智慧 (BI) 及應用程式使用情況的轉換和分析結果。

![複製活動的角色](media/copy-activity-overview/copy-activity.png)

複製活動會在[整合執行階段](concepts-integration-runtime.md)上執行。 針對不同的資料複製案例，可以運用整合執行階段的不同類別：

* 在可公開存取的資料存放區之間複製資料時，可透過 **Azure 整合執行階段**執行複製活動，Azure 整合執行階段的優點是安全、可靠、可擴充和[全球通用](concepts-integration-runtime.md#integration-runtime-location)。
* 在內部部署或具有存取控制的網路 (例如，Azure 虛擬網路) 資料存放區之間複製資料時，您需要設定**自我裝載整合執行階段**以執行資料複製。

Integration Runtime 必須與每個來源及接收端資料存放區相關聯。 瞭解複製活動如何[決定使用何種 IR](concepts-integration-runtime.md#determining-which-ir-to-use)。

複製活動將資料從來源複製到接收，會經歷下列階段。 為「複製活動」提供技術支援的雲端服務會：

1. 從來源資料存放區讀取資料。
2. 執行序列化/還原序列化、壓縮/解壓縮及資料行對應等。它會根據輸入資料集、輸出資料集及「複製活動」的組態執行這些作業。
3. 將資料寫入接收/目的地資料存放區。

![複製活動概觀](media/copy-activity-overview/copy-activity-overview.png)

## <a name="supported-data-stores-and-formats"></a>支援的資料存放區和格式

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores.md)]

### <a name="supported-file-formats"></a>支援的檔案格式

您可以使用「複製活動」在兩個以檔案為基礎的資料存放區之間「依原狀複製檔案」，資料就會有效率地複製，而不需經過序列化/還原序列化。

「複製活動」也支援以指定的格式讀取和寫入檔案：**文字、JSON、Avro、ORC 和 Parquet**，並支援 **GZip、Deflate、BZip2 和 ZipDeflate** 壓縮轉碼器。 如需詳細資訊，請參閱[支援的檔案和壓縮格式](supported-file-formats-and-compression-codecs.md)。

例如，您可以執行下列複製活動：

* 複製內部部署 SQL Server 中的資料，然後以 ORC 格式寫入 Azure Data Lake Store。
* 從內部部署檔案系統複製文字 (CSV) 格式的檔案，然後以 Avro 格式寫入 Azure Blob 中。
* 從內部部署檔案系統複製壓縮檔，然後解壓縮到 Azure Data Lake Store。
* 從 Azure Blob 複製 GZip 壓縮文字 (CSV) 格式的資料，然後寫入 Azure SQL Database 中。

## <a name="supported-regions"></a>支援區域

支援複製活動的服務可在 [Azure Integration Runtime 位置](concepts-integration-runtime.md#integration-runtime-location)中列出的區域和地理位置全域提供使用。 全域可用的拓撲可確保進行有效率的資料移動，通常可避免發生跨區域躍點的情況。 如需了解某區域中是否有 Data Factory 和「資料移動」可供使用，請參閱 [依區域提供的服務](https://azure.microsoft.com/regions/#services) 。

## <a name="configuration"></a>組態

若要在 Azure Data Factory 中使用複製活動，您需要：

1. **為來源和接收資料存放區建立已連結的服務。** 如需如何設定和支援的屬性，請參閱連接器發行項的＜連結服務屬性＞一節。 您可以在[支援的資料存放區和格式](#supported-data-stores-and-formats)一節找到支援的連接器清單。
2. **建立來源和接收的資料集。** 如需如何設定和支援的屬性，請參閱來源和接收連接器發行項的＜資料集屬性＞一節。
3. **建立具有複製活動的管線。** 下一節提供範例。  

### <a name="syntax"></a>語法

下列複製活動範本包含詳盡的支援屬性清單。 請指定適合您案例的屬性。

```json
"activities":[
    {
        "name": "CopyActivityTemplate",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<source dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<sink dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>",
                <properties>
            },
            "sink": {
                "type": "<sink type>"
                <properties>
            },
            "translator":
            {
                "type": "TabularTranslator",
                "columnMappings": "<column mapping>"
            },
            "dataIntegrationUnits": <number>,
            "parallelCopies": <number>,
            "enableStaging": true/false,
            "stagingSettings": {
                <properties>
            },
            "enableSkipIncompatibleRow": true/false,
            "redirectIncompatibleRowSettings": {
                <properties>
            }
        }
    }
]
```

### <a name="syntax-details"></a>語法詳細資料

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動的類型屬性必須設定為：**複製** | yes |
| 輸入 | 指定您所建立指向來源資料的資料集。 複製活動僅支援單一輸入。 | yes |
| 輸出 | 指定您所建立指向接收資料的資料集。 複製活動僅支援單一輸出。 | yes |
| typeProperties | 要設定複製活動的屬性群組。 | yes |
| 來源 | 指定關於如何擷取資料的複製來源類型和對應的屬性。<br/><br/>如需詳細資料，請參閱[支援的資料存放區和格式](#supported-data-stores-and-formats)中所列之連接器發行項的＜複製活動屬性＞一節。 | yes |
| 接收 | 指定複製接收類型和關於如何寫入資料的對應屬性。<br/><br/>如需詳細資料，請參閱[支援的資料存放區和格式](#supported-data-stores-and-formats)中所列之連接器發行項的＜複製活動屬性＞一節。 | yes |
| 轉譯程式 | 指定從來源到接收的明確資料行對應。 適用於預設複製行為無法滿足您的需求時。<br/><br/>如需詳細資料，請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)。 | 否 |
| dataIntegrationUnits | 指定 [Azure 整合執行階段](concepts-integration-runtime.md)以執行資料複製。 之前稱為「資料移動單位」(DMU)。 <br/><br/>如需詳細資訊，請參閱[資料整合單位](copy-activity-performance.md#data-integration-units)。 | 否 |
| parallelCopies | 指定從來源讀取資料和寫入資料到接收時，「複製活動」要使用的平行處理原則。<br/><br/>如需詳細資料，請參閱[平行複製](copy-activity-performance.md#parallel-copy)。 | 否 |
| enableStaging<br/>stagingSettings | 選擇在 aa blob 儲存體暫存過度資料，而不是直接從來源將資料複製到接收。<br/><br/>如需實用案例和設定的詳細資料，請參閱[分段複製](copy-activity-performance.md#staged-copy)。 | 否 |
| enableSkipIncompatibleRow<br/>redirectIncompatibleRowSettings| 選擇從來源將資料複製到接收時，要如何處理不相容的資料列。<br/><br/>如需詳細資料，請參閱[容錯](copy-activity-fault-tolerance.md)。 | 否 |

## <a name="monitoring"></a>監視

您可以在 Azure Data Factory 的 [編寫與監視] UI 上或以程式設計方式監視複製活動執行。 接著，您可以將您案例的效能和組態，與內部測試中複製活動的[效能參考](copy-activity-performance.md#performance-reference)進行比較。

### <a name="monitor-visually"></a>以視覺化方式監視

若要以視覺化方式監視複製活動執行，請移至您的資料處理站 -> [編寫與監視] -> [監視] 索引標籤，您會看到管線執行清單，而其中的 [動作] 資料行中有「檢視活動回合」連結。 

![監視管線回合](./media/load-data-into-azure-data-lake-store/monitor-pipeline-runs.png)

按一下以查看此管線執行中的活動清單。 在 [動作] 資料行中，您會有複製活動輸入、輸出、錯誤 (如果複製活動執行失敗) 及詳細資料的連結。

![監視活動回合](./media/load-data-into-azure-data-lake-store/monitor-activity-runs.png)

按一下 [動作] 下方的「詳細資料」連結，可查看複製活動的執行詳細資料及效能特性。 其中顯示的資訊包括從來源複製到接收端的資料量/資料列/檔案、輸送量、複製活動在對應期間內經歷的步驟，以及您的複製案例所使用的組態。

**範例：從 Amazon S3 複製到 Azure Data Lake Store**
![監視活動執行詳細資料](./media/copy-activity-overview/monitor-activity-run-details-adls.png)

**範例：使用分段複製從 Azure SQL Database 複製到 Azure SQL 資料倉儲**
![監視活動執行詳細資料](./media/copy-activity-overview/monitor-activity-run-details-sql-dw.png)

### <a name="monitor-programmatically"></a>以程式設計方式監視

複製活動執行詳細資料和效能特性也會在 [複製活動執行結果] -> [輸出] 區段中傳回。 以下是詳盡的清單；只會顯示您的複製案例適用的部分。 請參閱[監視快速入門](quickstart-create-data-factory-dot-net.md#monitor-a-pipeline-run)一節，了解如何監視活動執行。

| 屬性名稱  | 說明 | 單位 |
|:--- |:--- |:--- |
| dataRead | 從來源讀取的資料大小 | Int64 值 (以**位元組**為單位) |
| dataWritten | 寫入到接收的資料大小 | Int64 值 (以**位元組**為單位) |
| filesRead | 從檔案儲存體複製資料時所要複製的檔案數目。 | Int64 值 (未指定單位) |
| filesWritten | 將資料複製到檔案儲存體時所要複製的檔案數目。 | Int64 值 (未指定單位) |
| rowsCopied | 正在複製的資料列數目 (不適用於二進位複本) 。 | Int64 值 (未指定單位) |
| rowsSkipped | 略過的不相容資料列數目。 您可以將 enableSkipIncompatibleRow 設為 true 以開啟此功能。 | Int64 值 (未指定單位) |
| throughput | 傳送資料的比例 | 浮點數 (以 **KB/s** 為單位) |
| copyDuration | 複製的持續時間 | Int32 值 (以秒為單位) |
| sqlDwPolyBase | 如果將資料複製到 SQL 資料倉儲時使用 PolyBase。 | BOOLEAN |
| redshiftUnload | 如果從 Redshift 複製資料時使用 UNLOAD。 | BOOLEAN |
| hdfsDistcp | 如果從 HDFS 複製資料時使用 DistCp。 | BOOLEAN |
| effectiveIntegrationRuntime | 以 `<IR name> (<region if it's Azure IR>)` 的格式，來顯示是使用哪個 Integration Runtime 來讓活動執行。 | 文字 (字串) |
| usedDataIntegrationUnits | 複製期間的有效資料整合單位。 | Int32 值 |
| usedParallelCopies | 複製期間有效的 parallelCopies。 | Int32 值|
| redirectRowPath | 您在 redirectIncompatibleRowSettings 下設定之 Blob 儲存體中略過之不相容資料列的記錄路徑。 請參閱下列範例。 | 文字 (字串) |
| executionDetails | 複製活動經歷的階段，以及對應步驟、持續時間、使用的組態等詳細資料。不建議剖析此區段，因為它可能會變更。 | 陣列 |

```json
"output": {
    "dataRead": 107280845500,
    "dataWritten": 107280845500,
    "filesRead": 10,
    "filesWritten": 10,
    "copyDuration": 224,
    "throughput": 467707.344,
    "errors": [],
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US 2)",
    "usedDataIntegrationUnits": 32,
    "usedParallelCopies": 8,
    "executionDetails": [
        {
            "source": {
                "type": "AmazonS3"
            },
            "sink": {
                "type": "AzureDataLakeStore"
            },
            "status": "Succeeded",
            "start": "2018-01-17T15:13:00.3515165Z",
            "duration": 221,
            "usedDataIntegrationUnits": 32,
            "usedParallelCopies": 8,
            "detailedDurations": {
                "queuingDuration": 2,
                "transferDuration": 219
            }
        }
    ]
}
```

## <a name="schema-and-data-type-mapping"></a>結構描述和資料類型對應

請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，其中描述複製活動將來源資料對應到接收的方式。

## <a name="fault-tolerance"></a>容錯

根據預設，當複製活動在來源與接收之間遇到不相容的資料時，就會停止複製資料並傳回失敗。 您可以明確地設定略過並記錄不相容的資料列，而是只複製這些相容的資料以使複製成功。 如需詳細資訊，請參閱[複製活動容錯](copy-activity-fault-tolerance.md)。

## <a name="performance-and-tuning"></a>效能和微調

請參閱 [複製活動的效能及微調指南](copy-activity-performance.md)，其中說明在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素。 它也列出在內部測試期間所觀察到的效能，並討論各種可將「複製活動」效能最佳化的方式。

## <a name="incremental-copy"></a>增量複製 
Data Factory 支援以累加方式將差異資料從來源資料存放區複製到目的地資料存放區的案例。 請參閱[教學課程：以累加方式複製資料](tutorial-incremental-copy-overview.md)。 

## <a name="read-and-write-partitioned-data"></a>讀取和寫入分割的資料
在第 1 版中，Azure Data Factory 支援使用 SliceStart/SliceEnd/WindowStart/WindowEnd 系統變數讀取或寫入分割的資料。 在目前的版本中，您可以透過使用管線參數，以觸發程序的開始時間/已排程時間作為參數的值來達成此行為。 如需詳細資訊，請參閱[如何讀取或寫入分割的資料](how-to-read-write-partitioned-data.md)。

## <a name="next-steps"></a>後續步驟
請參閱下列快速入門、教學課程和範例：

- [在相同的 Azure Blob 儲存體中將資料從一個位置複製到另一個位置](quickstart-create-data-factory-dot-net.md)
- [將資料從 Azure Blob 儲存體複製到 Azure SQL Database](tutorial-copy-data-dot-net.md)
- [將資料從內部部署 SQL Server 複製到 Azure](tutorial-hybrid-copy-powershell.md)
