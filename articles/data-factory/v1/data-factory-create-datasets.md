---
title: 在 Azure Data Factory 中建立資料集 | Microsoft Docs
description: 了解如何透過使用位移和 anchorDateTime 等屬性的範例，在 Azure Data Factory 中建立資料集。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 0614cd24-2ff0-49d3-9301-06052fd4f92a
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: f33ff3f588dac49e295a5aa96d71557d32407e46
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38667433"
---
# <a name="datasets-in-azure-data-factory"></a>Azure Data Factory 中的資料集
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-create-datasets.md)
> * [第 2 版 (目前的版本)](../concepts-datasets-linked-services.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的資料集](../concepts-datasets-linked-services.md)。

本文說明什麼是資料集、如何以 JSON 格式定義它們，以及如何在 Azure Data Factory 管線中使用它們。 它提供有關資料集 JSON 定義中每個區段 (例如 structure、availability 及 policy) 的詳細資料。 本文也提供在資料集 JSON 定義中使用 **offset**、**anchorDateTime** 及 **style** 屬性的範例。

> [!NOTE]
> 如果您不熟悉 Data Factory，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)來概略了解。 如果您沒有建立 Data Factory 的實作經驗，可以閱讀[資料轉換教學課程](data-factory-build-your-first-pipeline.md)和[資料移動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)來進一步了解。 

## <a name="overview"></a>概觀
資料處理站可以有一或多個管線。 「管線」是一起執行某個工作的「活動」所組成的邏輯群組。 管線中的活動會定義要在資料上執行的動作。 例如，您可以使用複製活動將資料從內部部署 SQL Server 複製到 Azure Blob 儲存體。 接著，您可以使用在 Azure HDInsight 叢集上執行 Hive 指令碼的 Hive 活動，來處理來自 Blob 儲存體的資料以產生輸出資料。 最後，您可以使用第二個複製活動將輸出資料複製到「Azure SQL 資料倉儲」，以在該處建置商業智慧 (BI) 報表解決方案。 如需有關管線和活動的詳細資訊，請參閱 [Azure Data Factory 中的管線及活動](data-factory-create-pipelines.md)。

一個活動可以接受零個或多個輸入「資料集」，並且會產生一個或多個輸出資料集。 輸入資料集代表管線中活動的輸入，輸出資料集則代表活動的輸出。 資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。 例如，Azure Blob 資料集會指定管線應從中讀取資料之 Blob 儲存體中的 Blob 容器和資料夾。 

在您建立資料集之前，請先建立一個「已連結的服務」，以將資料存放區連結到 Data Factory。 已連結的服務非常類似連接字串，可定義 Data Factory 連接到外部資源所需的連線資訊。 資料集可識別已連結的資料存放區內的資料，例如 SQL 資料表、檔案、資料夾及文件。 例如，「Azure 儲存體」已連結服務會將儲存體帳戶連結到 Data Factory。 Azure Blob 資料集代表包含要處理之輸入 Blob 的 Blob 容器和資料夾。 

以下是一個範例案例。 若要將資料從 Blob 儲存體複製到 SQL Database，您需建立兩個已連結的服務：「Azure 儲存體」和 Azure SQL Database。 接著，建立兩個資料集：Azure Blob 資料集 (此資料集參考「Azure 儲存體」已連結服務) 和「Azure SQL 資料表」資料集 (此資料集參考 Azure SQL Database 已連結服務)。 「Azure 儲存體」和 Azure SQL Database 已連結服務包含 Data Factory 在執行階段分別用來連接到「Azure 儲存體」和 Azure SQL Database 的連接字串。 Azure Blob 資料集會指定包含 Blob 儲存體中輸入 Blob 的 Blob 容器和 Blob 資料夾。 「Azure SQL 資料表」資料集會指定作為資料複製目的地的 SQL Database 中 SQL 資料表。

下圖顯示 Data Factory 中管線、活動、資料集及已連結服務之間的關聯性： 

![管線、活動、資料集、已連結的服務之間的關聯性](media/data-factory-create-datasets/relationship-between-data-factory-entities.png)

## <a name="dataset-json"></a>資料集 JSON
Data Factory 中的資料集會以 JSON 格式定義如下：

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": <boolean flag to indicate external data. only for input datasets>,
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
       "policy":
        {      
        }
    }
}
```

下表描述上述 JSON 的屬性：   

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| name |資料集的名稱。 請參閱 [Azure Data Factory - 命名規則](data-factory-naming-rules.md) ，以了解命名規則。 |yes |NA |
| type |資料集的類型。 指定 Data Factory 支援的其中一種類型 (例如︰AzureBlob、AzureSqlTable)。 <br/><br/>如需詳細資料，請參閱[資料集類型](#Type)。 |yes |NA |
| structure |資料集的結構描述。<br/><br/>如需詳細資料，請參閱[資料集結構](#Structure)。 |否 |NA |
| typeProperties | 每個類型 (例如：Azure Blob、Azure SQL 資料表) 的類型屬性都不同。 如需有關支援的類型及其屬性的詳細資料，請參閱[資料集類型](#Type)。 |yes |NA |
| external | 用來指定資料集是否由 Data Factory 管線明確產生的布林值旗標。 如果活動的輸入資料集不是由目前的管線所產生，請將此旗標設定為 true。 請針對管線中第一個活動的輸入資料集，將此旗標設定為 true。  |否 |false |
| availability | 定義處理時段 (例如每小時或每天) 或用於產生資料集的切割模型。 活動執行取用和產生的每個資料單位稱為資料配量。 如果將輸出資料集的可用性設定為每天 (frequency - Day，interval - 1)，就會每天產生配量。 <br/><br/>如需詳細資料，請參閱[資料集可用性](#Availability)。 <br/><br/>如需有關資料集切割模型的詳細資料，請參閱[排程和執行](data-factory-scheduling-and-execution.md)一文。 |yes |NA |
| 原則 |定義資料集配量必須符合的準則或條件。 <br/><br/>如需詳細資料，請參閱[資料集原則](#Policy)一節。 |否 |NA |

## <a name="dataset-example"></a>資料集範例
在下列範例中，資料集代表 SQL Database 中名為 **MyTable** 的資料表。

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties":
        {
            "tableName": "MyTable"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

請注意下列幾點：

* **type** 是設定為 AzureSqlTable。
* **tableName** 類型屬性 (AzureSqlTable 類型特定) 是設定為 MyTable。
* **linkedServiceName** 係指 AzureSqlDatabase 類型的已連結服務，在接下來的 JSON 程式碼片段中將會定義。 
* **availability frequency** 是設定為 Day，**interval** 是設定為 1。 這意謂著會每天產生資料集配量。  

**AzureSqlLinkedService** 定義如下︰

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>@<servername>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

在上述 JSON 程式碼片段中：

* **type** 是設定為 AzureSqlDatabase。
* **connectionString** 類型屬性會指定用以連接到 SQL Database 的資訊。  

如您所見，已連結的服務會定義如何連接到 SQL Database。 資料集會定義使用哪個資料表作為管線中活動的輸入和輸出。   

> [!IMPORTANT]
> 除非資料集是由管線所產生，否則應該標示為「外部」。 此設定通常會套用至管線中第一個活動的輸入。   


## <a name="Type"></a> 資料集類型
資料集的類型取決於您所使用的資料存放區。 如需 Data Factory 所支援的資料存放區清單，請參閱下表。 按一下某個資料存放區，即可了解如何為該資料存放區建立已連結的服務和資料集。

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> 帶有 * 的資料存放區可以位於內部部署環境或 Azure 基礎結構即服務 (IaaS) 上。 這些資料存放區會要求您安裝[資料管理閘道](data-factory-data-management-gateway.md)。

在上一節的範例中，資料集的類型是設定為 **AzureSqlTable**。 同樣地，針對 Azure Blob 資料集，資料集的類型是設定為 **AzureBlob**，如以下 JSON 所示：

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

## <a name="Structure"></a>資料集結構
**structure** 區段是選擇性區段。 它可透過包含資料行之名稱和資料類型的集合，定義資料集的結構描述。 您可以使用 structure 區段來提供類型資訊，此資訊會用來轉換類型並將資料行從來源對應到目的地。 在下列範例中，資料集有三個資料行︰`slicetimestamp`、`projectname` 及 `pageviews`。 它們的類型分別是 String、String 及 Decimal。

```json
structure:  
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

structure 中的每個資料行都包含下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| name |資料行的名稱。 |yes |
| type |資料行的資料類型。  |否 |
| culture |當類型為 .NET 類型 (`Datetime` 或 `Datetimeoffset`) 時，所要使用的 .NET 型文化特性。 預設值為 `en-us`。 |否 |
| format |當類型為 .NET 類型 (`Datetime` 或 `Datetimeoffset`) 時，所要使用的格式字串。 |否 |

下列方針可協助您判斷何時要包括 structure 資訊，以及在 **structure** 區段中要包含哪些資訊。

* **針對結構化資料來源**，請只有在您想要將來源資料行對應到接收資料行且其名稱不相同時，才指定 structure 區段。 這類結構化資料來源會將資料結構描述和類型資訊與資料本身儲存在一起。 結構化資料來源的範例包括 SQL Server、Oracle 及 Azure 資料表。 
  
    由於結構化資料來源已經有可用的類型資訊，因此當您包含 structure 區段時，便不應包含類型資訊。
* **針對在讀取時驗證結構描述 (schema on read) 的資料來源 (具體而言即 Blob 儲存體)**，您可以選擇儲存資料，而不將任何結構描述或類型資訊與資料儲存在一起。 針對這些類型的資料來源，當您想要將來源資料行與接收資料行對應時，請包含 structure。 當資料集是複製活動的輸入，並且來源資料集的資料類型應該轉換成接收器的原生類型時，也請包含 structure。 
    
    Data Factory 支援使用下列值在 structure 中提供類型資訊：**Int16、Int32、Int64、Single、Double、Decimal、Byte[]、Boolean、String、Guid、Datetime、Datetimeoffset 及 Timespan**。 這些值是符合 Common Language Specification (CLS) 規範的 .NET 型類型值。

Data Factory 會在將資料從來源資料存放區移到接收資料存放區時，自動執行類型轉換。 
  

## <a name="dataset-availability"></a>資料集可用性
資料集中的 **availability** 區段會定義資料集的處理時段 (例如每小時、每天或每週)。 如需有關活動時段的詳細資訊，請參閱[排程和執行](data-factory-scheduling-and-execution.md)。

下列 availability 區段指定會每小時產生輸出資料集，或每小時會有可用的輸入資料集：

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 1    
}
```

如果管線有下列開始和結束時間：  

```json
    "start": "2016-08-25T00:00:00Z",
    "end": "2016-08-25T05:00:00Z",
```

輸出資料集會在管線的開始和結束時間內每小時產生。 因此，這個管線會產生五個資料集配量，每個活動時段 (上午 12 點 - 上午 1 點、上午 1 點 - 上午 2 點、上午 2 點 - 上午 3 點、上午 3 點 - 上午 4 點、上午 4 點 - 上午 5 點) 各一個資料集。 

下表描述您可以在 [可用性] 區段中使用的屬性：

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| frequency |指定資料集配量生產的時間單位。<br/><br/><b>支援的頻率</b>：Minute、Hour、Day、Week、Month |yes |NA |
| interval |指定頻率的倍數。<br/><br/>「頻率 x 間隔」會決定產生配量的頻率。 例如，如果您需要將資料集以每小時為單位來切割，請將 <b>frequency</b> 設定為 <b>Hour</b>，將 <b>interval</b> 設定為 <b>1</b>。<br/><br/>請注意，如果您將 **frequency** 指定為 **Minute**，則應該將 interval 設定為不小於 15。 |yes |NA |
| style |指定應該在間隔的開始還是結束時產生配量。<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul>如果將 **frequency** 設定為 **Month**，並將 **style** 設定為 **EndOfInterval**，就會在當月的最後一天產生配量。 如果將 **style** 設定為 **StartOfInterval**，則會在當月的第一天產生配量。<br/><br/>如果將 **frequency** 設定為 **Day**，並將 **style** 設定為 **EndOfInterval**，就會在當天的最後一小時產生配量。<br/><br/>如果 **frequency** 設定為 **Hour**，並將 **style** 設定為 **EndOfInterval**，則會在該小時結束時產生配量。 例如，就下午 1 點 - 2 點期間的配量而言，會在下午 2 點產生配量。 |否 |EndOfInterval |
| anchorDateTime |定義排程器用來計算資料集配量界限的時間絕對位置。 <br/><br/>請注意，如果此屬性有比所指定頻率更細微的日期部分，則系統會忽略那些更細微的部分。 例如，如果 **interval** 為 **hourly** (frequency: hour 且 interval: 1)，而且 **anchorDateTime** 包含**分鐘和秒鐘**，則系統會忽略 **anchorDateTime** 的分鐘和秒鐘部分。 |否 |01/01/0001 |
| Offset |所有資料集配量的開始和結束移位所依據的時間範圍。 <br/><br/>請注意，如果同時指定 **anchorDateTime** 和 **offset**，則結果會是合併的位移。 |否 |NA |

### <a name="offset-example"></a>位移範例
根據預設，每日 (`"frequency": "Day", "interval": 1`) 配量的開始時間是「國際標准時間」(UTC) 上午 12 點 (午夜)。 如果您希望將開始時間改為 UTC 時間上午 6 點，請依照下列程式碼片段所示設定位移： 

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
### <a name="anchordatetime-example"></a>anchorDateTime 範例
在下列範例中，會每隔 23 小時產生一次資料集。 第一個配量會在 **anchorDateTime** 所指定的時間 (已設定為 `2017-04-19T08:00:00` (UTC)) 開始。

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 23,    
    "anchorDateTime":"2017-04-19T08:00:00"    
}
```

### <a name="offsetstyle-example"></a>位移/樣式範例
下列資料集是每月資料集，會在每月 3 號的上午 8:00 (`3.08:00:00`) 產生：

```json
"availability": {
    "frequency": "Month",
    "interval": 1,
    "offset": "3.08:00:00", 
    "style": "StartOfInterval"
}
```

## <a name="Policy"></a>資料集原則
資料集定義中的 **policy** 區段會定義資料集配量必須符合的準則或條件。

### <a name="validation-policies"></a>驗證原則
| 原則名稱 | 說明 | 適用於 | 必要 | 預設值 |
| --- | --- | --- | --- | --- |
| minimumSizeMB |驗證「Azure Blob 儲存體」中的資料是否符合大小下限需求 (以 MB 為單位)。 |Azure Blob 儲存體 |否 |NA |
| minimumRows |驗證 **Azure SQL Database** 或 **Azure 資料表**中的資料是否包含最小的資料列數。 |<ul><li>Azure SQL Database</li><li>Azure 資料表</li></ul> |否 |NA |

#### <a name="examples"></a>範例
**minimumSizeMB：**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

**minimumRows：**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

### <a name="external-datasets"></a>外部資料集
外部資料集是並非由 Data Factory 中的執行中管線所產生的資料集。 如果資料集標示為**外部**，則可定義 **ExternalData** 原則來影響資料集配量可用性的行為。

除非資料集是由 Data Factory 所產生，否則應該標示為「外部」。 除非會使用活動或管線鏈結，否則此設定通常會套用到管線中第一個活動的輸入。

| Name | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| dataDelay |要延遲所指定配量之外部資料可用性檢查的時間。 例如，您可以使用此設定來延遲每小時檢查。<br/><br/>此設定僅適用於當前的時間。  例如，如果現在時間是下午 1:00，且此值是 10 分鐘，驗證就會在下午 1:10 開始。<br/><br/>請注意，此設定不會影響過去的配量。 針對**配量結束時間** + **dataDelay** < **現在**的配量，處理時將不會有任何延遲。<br/><br/>時間如果大於 23:59 小時，應該使用 `day.hours:minutes:seconds` 格式來指定。 例如，若要指定 24 小時，不要使用 24:00:00。 請改用 1.00:00:00。 如果您使用 24:00:00，這會視同 24 天 (24.00:00:00)。 如為 1 天又 4 小時，請指定 1:04:00:00。 |否 |0 |
| retryInterval |失敗與下一次嘗試之間的等待時間。 此設定適用於當前的時間。 如果上一次嘗試失敗，則下一次嘗試將會在 **retryInterval** 期間之後。 <br/><br/>如果現在是下午 1:00，我們會開始第一次嘗試。 如果完成第一次驗證檢查的持續時間是 1 分鐘且作業失敗，則下一次重試會在 1:00 + 1 分鐘 (持續時間) + 1 分鐘 (重試間隔) = 下午 1:02。 <br/><br/>若是過去的配量，則不會有任何延遲。 重試會立即發生。 |否 |00:01:00 (1 分鐘) |
| retryTimeout |每次重試嘗試的逾時。<br/><br/>如果此屬性是設定為 10 分鐘，則應該在 10 分鐘內完成驗證。 如果花超過 10 分鐘來執行驗證，則重試會逾時。<br/><br/>如果所有驗證嘗試都逾時，配量就會標示為 **TimedOut**。 |否 |00:10:00 (10 分鐘) |
| maximumRetry |檢查外部資料可用性的次數。 允許的最大值為 10。 |否 |3 |


## <a name="create-datasets"></a>建立資料集
您可以使用下列其中一項工具或 SDK 來建立資料集： 

- 複製精靈 
- Azure 入口網站
- Visual Studio
- PowerShell
- Azure Resource Manager 範本
- REST API
- .NET API

如需使用上述其中一項工具或 SDK 來建立管線和資料集的逐步指示，請參閱下列教學課程：
 
- [使用資料轉換活動來建置管線](data-factory-build-your-first-pipeline.md)
- [使用資料移動活動來建置管線](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

建置及部署管線之後，您便可以使用 Azure 入口網站刀鋒視窗或「監視與管理」應用程式，來管理及監視管線。 如需逐步指示，請參閱下列主題： 

- [使用 Azure 入口網站刀鋒視窗來監視和管理管線](data-factory-monitor-manage-pipelines.md)
- [使用監視及管理應用程式，以監視和管理管線](data-factory-monitor-manage-app.md)


## <a name="scoped-datasets"></a>範圍資料集
您可以使用 **datasets** 屬性來建立範圍設定為管線的資料集。 這些資料集僅供此管線內的活動使用，不供其他管線中的活動使用。 下列範例會定義一個管線，此管線具有兩個要在管線內使用的資料集 (InputDataset-rdc 和 OutputDataset-rdc)。  

> [!IMPORTANT]
> 只有單次管線 (**pipelineMode** 已設定為 **OneTime**) 才支援範圍資料集。 如需詳細資訊，請參閱 [一次性管線](data-factory-create-pipelines.md#onetime-pipeline) 。
>
>

```json
{
    "name": "CopyPipeline-rdc",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-rdc"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-rdc"
                    }
                ],
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1,
                    "style": "StartOfInterval"
                },
                "name": "CopyActivity-0"
            }
        ],
        "start": "2016-02-28T00:00:00Z",
        "end": "2016-02-28T00:00:00Z",
        "isPaused": false,
        "pipelineMode": "OneTime",
        "expirationTime": "15.00:00:00",
        "datasets": [
            {
                "name": "InputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "InputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/input",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": true,
                    "policy": {}
                }
            },
            {
                "name": "OutputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "OutputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/output",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": false,
                    "policy": {}
                }
            }
        ]
    }
}
```

## <a name="next-steps"></a>後續步驟
- 如需有關管線的詳細資訊，請參閱[建立管線](data-factory-create-pipelines.md)。 
- 如需有關管線的排程和執行方式的詳細資訊，請參閱 [Azure Data Factory 中的排程和執行](data-factory-scheduling-and-execution.md)。 
