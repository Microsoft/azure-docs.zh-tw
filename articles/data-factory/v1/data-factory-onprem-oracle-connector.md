---
title: 使用 Data Factory 從 Oracle 來回複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從內部部署 Oracle 資料庫來回複製資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 3c20aa95-a8a1-4aae-9180-a6a16d64a109
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 10535e75a32a9f95e759340cf14d693f43639473
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37856836"
---
# <a name="copy-data-to-or-from-on-premises-oracle-using-azure-data-factory"></a>使用 Azure Data Factory 從內部部署 Oracle 來回複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-onprem-oracle-connector.md)
> * [第 2 版 (目前的版本)](../connector-oracle.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Oracle 連接器](../connector-oracle.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，將資料移進/移出內部部署的 Oracle 資料庫。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

## <a name="supported-scenarios"></a>支援的案例
您可以**從 Oracle 資料庫**將資料複製到下列資料存放區：

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製**到 Oracle 資料庫**：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="prerequisites"></a>先決條件
Data Factory 支援使用資料管理閘道連接至內部部署 Oracle 來源。 請參閱[資料管理閘道](data-factory-data-management-gateway.md)一文來了解資料管理閘道，以及[將資料從內部部署移動到雲端](data-factory-move-data-between-onprem-and-cloud.md)一文來取得設定資料管線的閘道以移動資料的逐步指示。

即使 Oracle 裝載於 Azure IaaS VM 中，也必須要有閘道。 您可以將閘道安裝在與資料存放區相同或相異的 IaaS VM 上，只要閘道可以連線到資料庫即可。

> [!NOTE]
> 如需連接/閘道器相關問題的疑難排解秘訣，請參閱 [針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) 。

## <a name="supported-versions-and-installation"></a>支援的版本和安裝
此 Oracle 連接器支援兩種驅動程式版本︰

- **適用於 Oracle 的 Microsoft 驅動程式 (建議)**：從資料管理閘道 2.7 版開始，適用於 Oracle 的 Microsoft 驅動程式會自動隨閘道安裝，因此您不需要另行處理驅動程式，即能建立與 Oracle 的連線，而且使用此驅動程式時您也可以體驗到更好的複製效能。 支援以下版本的 Oracle 資料庫︰
    - Oracle 12c R1 (12.1)
    - Oracle 11g R1、R2 (11.1、11.2)
    - Oracle 10g R1、R2 (10.1、10.2)
    - Oracle 9i R1、R2 (9.0.1、9.2)
    - Oracle 8i R3 (8.1.7)

> [!NOTE]
> 不支援 Oracle Proxy 伺服器。

> [!IMPORTANT]
> 目前適用於 Oracle 的 Microsoft 驅動程式僅支援從 Oracle 複製資料，但不支援將資料寫入 Oracle。 請注意，[資料管理閘道診斷] 索引標籤中的測試連線功能不支援此驅動程式。 您可以選擇使用複製精靈來驗證連線。
>

- **.NET 的 Oracle 資料提供者︰** 您也可以選擇使用 Oracle 資料提供者，從 Oracle 複製資料/將資料複製到 Oracle。 此元件包含於 [適用於 Windows 的 Oracle 資料存取元件](http://www.oracle.com/technetwork/topics/dotnet/downloads/)中。 在安裝閘道的電腦上安裝適當版本 (32/64 位元)。 [Oracle Data Provider .NET 12.1](http://docs.oracle.com/database/121/ODPNT/InstallSystemRequirements.htm#ODPNT149) 可以存取 Oracle Database 10g Release 2 或更新版本。

    如果您選擇「XCopy 安裝」，請依照 readme.htm 中的步驟進行。 建議您選擇含有 UI 的安裝程式 (非 XCopy 安裝)。

    安裝提供者之後，請使用 [服務] 小程式 (或) 資料管理閘道組態管理員，「重新啟動」您電腦上的資料管理閘道主機服務。  

若使用複製精靈來建立複製管線，系統將會自動決定驅動程式類型。 預設將會使用 Microsoft 驅動程式，除非您的閘道版本低於 2.7 或您選擇使用 Oracle 作為接收 (Sink)。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出內部部署的 Oracle 資料庫。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您從 Oralce 資料庫將資料複製到 Azure Blob 儲存體，您會建立兩個連結服務，將 Oracle 資料庫和 Azure 儲存體帳戶連結至資料處理站。 針對 Oracle 專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。
3. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您可以建立資料集來指定包含輸入資料之 Oracle 資料庫中的資料表。 同時建立另一個資料集來指定 blob 容器和資料夾，該資料夾會保存從 Oracle 資料庫複製的資料。 針對 Oracle 專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
4. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 OracleSource 作為來源，以及使用 BlobSink 作為複製活動的接收器。 同樣地，如果您是從 Azure Blob 儲存體複製到 Oracle 資料庫，則在複製活動中使用 BlobSource 和 OracleSink。 針對 Oracle 資料庫專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，按一下上一節中資料存放區的連結。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到內部部署 Oracle 資料庫 (或從內部部署 Oracle 資料庫複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-to-and-from-oracle-database)一節。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 Oracle 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **OnPremisesOracle** |yes |
| driverType | 指定要用來從 Oracle 複製資料/將資料複製到 Oracle 資料庫的驅動程式。 允許的值為 **Microsoft** 或 **ODP** (預設值)。 如需驅動程式詳細資料，請參閱[支援的版本和安裝](#supported-versions-and-installation)一節。 | 否 |
| connectionString | 針對 connectionString 屬性指定連接到 Oracle 資料庫執行個體所需的資訊。 | yes |
| gatewayName | 用來連線內部部署 Oracle 伺服器的閘道名稱 |yes |

**範例︰使用 Microsoft 驅動程式：**

>[!TIP]
>如果您遇到指出「ORA-01025: UPI 參數超出範圍」的錯誤，而且您的 Oracle 版本為 8i，請將 `WireProtocolMode=1` 新增至連接字串並再試一次。

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**範例︰使用 ODP 驅動程式**

如需了解有哪些允許的格式，請參閱[此網站 (英文)](https://www.connectionstrings.com/oracle-data-provider-for-net-odp-net/)。

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<hostname>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<SID>)));
User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (Oracle、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。 OracleTable 資料集類型的 typeProperties 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Oracle 資料庫中連結服務所參照的資料表名稱。 |否 (如果已指定 **OracleSource** 的 **oracleReaderQuery**) |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [!NOTE]
> 複製活動只會採用一個輸入，而且只產生一個輸出。

而活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

### <a name="oraclesource"></a>oracleReaderQuery
在複製活動中，如果來源的類型為 **OracleSource**，則 **typeProperties** 區段有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| oracleReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：select * from MyTable <br/><br/>如果未指定，則為所執行的 SQL 陳述式：select * from MyTable |否 (如果已指定 **dataset** 的 **tableName**) |

### <a name="oraclesink"></a>管線
**OracleSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：00:30:00 (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：100) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |

## <a name="json-examples-for-copying-data-to-and-from-oracle-database"></a>從 Oracle 資料庫複製資料以及將資料複製到其中的 JSON 範例
下列範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 這些範例示範如何將資料從 Oracle 資料庫複製到 Azure Blob 儲存體，以及複製其中的資料。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。   

## <a name="example-copy-data-from-oracle-to-azure-blob"></a>範例：將資料從 Oracle 複製到 Azure Blob

此範例具有下列 Data Factory 實體：

1. [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [OracleSource](data-factory-onprem-oracle-connector.md#copy-activity-properties) 做為來源和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 做為接收之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將資料從內部部署 Oracle 資料庫中的資料表移動到 Blob。 如需範例中使用的各種屬性的詳細資訊，請參閱範例後面幾節中的文件。

**Oracle 連結服務：**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Azure Blob 儲存體連結服務：**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<Account key>"
        }
    }
}
```

**Oracle 輸入資料集：**

此範例假設您已在 Oracle 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2014-02-27T12:00:00",
            "frequency": "Hour"
        },
        "policy": {     
            "externalData": {        
                "retryInterval": "00:01:00",    
                "retryTimeout": "00:10:00",       
                "maximumRetry": 3       
            }     
        }
    }
}
```

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": "\t",
                "rowDelimiter": "\n"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**具有複製活動的管線：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **OracleSource**，而 **sink** 類型設為 **BlobSink**。  利用 **oracleReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
            {
                "name": "OracletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": " OracleInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "OracleSource",
                        "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

## <a name="example-copy-data-from-azure-blob-to-oracle"></a>範例：將資料從 Azure Blob 複製到 Oracle
此範例示範如何將資料從 Azure Blob 儲存體複製到內部部署 Oracle 資料庫。 不過，您可以在 Azure Data Factory 中使用複製活動，**直接**從[這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats)所說的任何來源複製資料。  

此範例具有下列 Data Factory 實體：

1. [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 做為來源和 [OracleSink](data-factory-onprem-oracle-connector.md#copy-activity-properties) 做為接收之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將資料從 Blob移動到內部部署 Oracle 資料庫中的資料表。 如需範例中使用的各種屬性的詳細資訊，請參閱範例後面幾節中的文件。

**Oracle 連結服務：**
```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<hostname>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<SID>)));
            User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Azure Blob 儲存體連結服務：**
```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<Account key>"
        }
    }
}
```

**Azure Blob 輸入資料集**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月及日部分，而檔案名稱則使用開始時間的小時部分。 “external”: “true” 設定會通知 Data Factory 服務：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                }
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

**Oracle 輸出資料集：**

此範例假設您已在 Oracle 資料表中建立資料表 "MyTable"。 請在 Oracle 中建立此資料表，其資料行的數目如您預期 Blob CSV 檔案要包含的數目。 此資料表會每小時加入新的資料列。

```json
{
    "name": "OracleOutput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Day",
            "interval": "1"
        }
    }
}
```

**具有複製活動的管線：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **OracleSink**。  

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-05T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
            {
                "name": "AzureBlobtoOracle",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "OracleOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "OracleSink"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```


## <a name="troubleshooting-tips"></a>疑難排解秘訣
### <a name="problem-1-net-framework-data-provider"></a>問題 1：.NET Framework Data Provider

您看到下列「錯誤訊息」：

    Copy activity met invalid parameters: 'UnknownParameterName', Detailed message: Unable to find the requested .Net Framework Data Provider. It may not be installed”.  

**可能的原因：**

1. 未安裝 .NET Framework Data Provider for Oracle。
2. .NET Framework Data Provider for Oracle 已安裝於 .NET Framework 2.0，而且在 .NET Framework 4.0 資料夾中找不到。

**解析/因應措施：**

1. 如果您尚未安裝 .NET Provider for Oracle，請 [安裝它](http://www.oracle.com/technetwork/topics/dotnet/downloads/) ，然後重試此案例。
2. 如果您即使在安裝提供者之後還是會收到錯誤訊息，請執行下列步驟：
   1. 從資料夾開啟 .NET 2.0 的電腦組態：<system disk>:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG\machine.config。
   2. 搜尋 **Oracle Data Provider for .NET**，而您應該能夠在 **system.data** -> **DbProviderFactories** 下方，找到如下列範例所示的項目：“<add name="Oracle Data Provider for .NET" invariant="Oracle.DataAccess.Client" description="Oracle Data Provider for .NET" type="Oracle.DataAccess.Client.OracleClientFactory, Oracle.DataAccess, Version=2.112.3.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />」的作法指南。
3. 將此項目複製到下列 v4.0 資料夾中的 machine.config 檔案 : <system disk>:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config，並將版本變更為 4.xxx.x.x。
4. 執行 `gacutil /i [provider path]`，將 “<ODP.NET 安裝路徑>\11.2.0\client_1\odp.net\bin\4\Oracle.DataAccess.dll” 安裝到全域組件快取 (GAC)。## 疑難排解祕訣

### <a name="problem-2-datetime-formatting"></a>問題 2︰日期時間格式

您看到下列「錯誤訊息」：

    Message=Operation failed in Oracle Database with the following error: 'ORA-01861: literal does not match format string'.,Source=,''Type=Oracle.DataAccess.Client.OracleException,Message=ORA-01861: literal does not match format string,Source=Oracle Data Provider for .NET,'.

**解析/因應措施：**

您可能需要根據 Oracle 資料庫中日期的設定方式，調整複製活動中的查詢字串，如下列範例 (使用 to_date 函式) 所示︰

    "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= to_date(\\'{0:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\')  AND timestampcolumn < to_date(\\'{1:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') ', WindowStart, WindowEnd)"


## <a name="type-mapping-for-oracle"></a>Oracle 的類型對應
如同 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

從 Oracle 移動資料時，會使用下列從 Oracle 資料類型到 .NET 類型的對應，反之亦然。

| Oracle 資料類型 | .NET Framework 資料類型 |
| --- | --- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(僅支援在 Oracle 10g 和更高版本使用 Microsoft 驅動程式) |
| CHAR |字串 |
| CLOB |字串 |
| 日期 |Datetime |
| FLOAT |Decimal，字串 (如果精確度 > 28) |
| INTEGER |Decimal，字串 (如果精確度 > 28) |
| 間隔年至月 |Int32 |
| 間隔日至秒鐘 |時間範圍 |
| 長 |字串 |
| 長 RAW |Byte[] |
| NCHAR |字串 |
| NCLOB |字串 |
| 數字 |Decimal，字串 (如果精確度 > 28) |
| NVARCHAR2 |字串 |
| RAW |Byte[] |
| ROWID |字串 |
| 時間戳記 |Datetime |
| 本地時區的時間戳記 |Datetime |
| 時區的時間戳記 |Datetime |
| 不帶正負號的整數 |數字 |
| VARCHAR2 |字串 |
| XML |字串 |

> [!NOTE]
> 使用 Microsoft 驅動程式時，不支援資料類型 **INTERVAL YEAR TO MONTH** 和 **INTERVAL DAY TO SECOND**。

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
