---
title: 使用 Azure Data Factory 將資料複製到 SQL Server 或從該處複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory，從內部部署或 Azure VM 中的 SQL Server 資料庫來回移動資料。
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
ms.date: 06/22/2018
ms.author: jingwang
ms.openlocfilehash: 4a0800dccca3a43d49204dfbcc32e7778449ae6e
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42442080"
---
# <a name="copy-data-to-and-from-sql-server-using-azure-data-factory"></a>使用 Azure Data Factory 將資料複製到 SQL Server 及從該處複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-sqlserver-connector.md)
> * [目前的版本](connector-sql-server.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 SQL Server 資料庫複製資料及將資料複製到該處。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 SQL Server 資料庫複製到任何支援的接收資料存放區，或將資料從任何支援的來源資料存放區複製到 SQL Server 資料庫。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 SQL Server 連接器支援：

- SQL Server 2016、2014、2012、2008 R2、2008 及 2005 版
- 使用 **SQL** 或 **Windows** 驗證來複製資料。
- 作為來源時，使用 SQL 查詢或預存程序來擷取資料。
- 在複製期間作為接收器時，使用自訂邏輯將資料附加到目的地資料表或叫用預存程序。

## <a name="prerequisites"></a>必要條件

若要從不可公開存取的 SQL Server 資料庫複製資料，您必須設定一個「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。 Integration Runtime 提供內建的 SQL Server 資料庫驅動程式，因此從 SQL Server 資料庫複製資料或將資料複製到該處時，您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 SQL Server 資料庫連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 SQL Server 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**SqlServer** | 是 |
| connectionString |指定使用 SQL 驗證或 Windows 驗證來連線至 SQL Server 資料庫時所需的 connectionString 資訊。 請參考下列範例。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| userName |如果您使用「Windows 驗證」，請指定使用者名稱。 範例︰**domainname\\username**。 |否 |
| password |指定您為 userName 指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |否 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!TIP]
>如果您遇到錯誤，其錯誤碼為 "UserErrorFailedToConnectToSqlServer"，以及「資料庫的工作階段限制為 XXX 並已達到。」訊息，請將 `Pooling=false` 新增至您的連接字串並再試一次。

**範例 1：使用 SQL 驗證**

```json
{
    "name": "SqlServerLinkedService",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**範例 2：使用 Windows 驗證**

```json
{
    "name": "SqlServerLinkedService",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=True;"
            },
             "userName": "<domain\\username>",
             "password": {
                "type": "SecureString",
                "value": "<password>"
             }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
     }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 SQL Server 資料集所支援的屬性清單。

若要從 SQL Server 資料庫複製資料或將資料複製到該處，請將資料集的類型屬性設定為 **SqlServerTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**SqlServerTable** | 是 |
| tableName |SQL Server 資料庫執行個體中已連結的服務所參考的資料表或檢視名稱。 | 是 |

**範例：**

```json
{
    "name": "SQLServerDataset",
    "properties":
    {
        "type": "SqlServerTable",
        "linkedServiceName": {
            "referenceName": "<SQL Server linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "MyTable"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 SQL Server 來源和接收器所支援的屬性清單。

### <a name="sql-server-as-source"></a>SQL Server 作為來源

若要從 SQL Server 複製資料，請將複製活動中的來源類型設定為 **SqlSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**SqlSource** | 是 |
| SqlReaderQuery |使用自訂 SQL 查詢來讀取資料。 範例： `select * from MyTable`. |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 最後一個 SQL 陳述式必須是預存程序中的 SELECT 陳述式。 |否 |
| storedProcedureParameters |預存程序的參數。<br/>允許的值為：名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

**注意事項：**

- 如果已為 SqlSource 指定 **sqlReaderQuery**，「複製活動」就會針對 SQL Server 來源執行此查詢來取得資料。 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。
- 如果您未指定 "sqlReaderQuery" 或 "sqlReaderStoredProcedureName"，就會使用資料集 JSON 的 "structure" 區段中定義的資料行，來建構要針對 SQL Server 執行的查詢 (`select column1, column2 from mytable`)。 如果資料集定義沒有 "structure"，則會從資料表中選取所有資料行。
- 當您使用 **sqlReaderStoredProcedureName** 時，仍然必須在資料集 JSON 中指定一個虛設的 **tableName**。

**範例：使用 SQL 查詢**

```json
"activities":[
    {
        "name": "CopyFromSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Server input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**範例：使用預存程序**

```json
"activities":[
    {
        "name": "CopyFromSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Server input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**預存程序定義：**

```sql
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
     select *
     from dbo.UnitTestSrcTable
     where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sql-server-as-sink"></a>SQL Server 作為接收器

若要將資料複製到 SQL Server，請將複製活動中的接收器類型設定為 **SqlSink**。 複製活動的 **sink** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收器的 type 屬性必須設定為：**SqlSink** | 是 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中<br/>允許的值為：整數 (資料列數目)。 |否 (預設值：10000) |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。<br/>允許的值為：時間範圍。 範例：“00:30:00” (30 分鐘)。 |否 |
| preCopyScript |指定一個供「複製活動」在將資料寫入到 SQL Server 前執行的 SQL 查詢。 每一複製回合只會叫用此查詢一次。 您可以使用此屬性來清除預先載入的資料。 |否 |
| sqlWriterStoredProcedureName |預存程序的名稱，此預存程序定義如何將來源資料套用至目標資料表，例如使用您自己的商務邏輯來執行更新插入或轉換。 <br/><br/>請注意，將會**依批次叫用**此預存程序。 如果您想要進行只執行一次且與來源資料無關的作業 (例如刪除/截斷)，請使用 `preCopyScript` 屬性。 |否 |
| storedProcedureParameters |預存程序的參數。<br/>允許的值為：名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |
| sqlWriterTableType |指定要在預存程序中使用的資料表類型名稱。 複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。 然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 |否 |

> [!TIP]
> 將資料複製到 SQL Server 時，複製活動預設會將資料附加至接收資料表。 若要執行 UPSERT 或其他商務邏輯，請在 SqlSink 中使用該預存程序。 若要了解更多詳細資料，請參閱[叫用 SQL 接收器的預存程序](#invoking-stored-procedure-for-sql-sink)。

**範例 1：附加資料**

```json
"activities":[
    {
        "name": "CopyToSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Server output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlSink",
                "writeBatchSize": 100000
            }
        }
    }
]
```

**範例 2：在複製期間叫用預存程序來進行更新插入**

若要了解更多詳細資料，請參閱[叫用 SQL 接收器的預存程序](#invoking-stored-procedure-for-sql-sink)。

```json
"activities":[
    {
        "name": "CopyToSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Server output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlSink",
                "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
                "sqlWriterTableType": "CopyTestTableType",
                "storedProcedureParameters": {
                    "identifier": { "value": "1", "type": "Int" },
                    "stringData": { "value": "str1" }
                }
            }
        }
    }
]
```

## <a name="identity-columns-in-the-target-database"></a>目標資料庫中的身分識別資料行

本節提供一個範例，此範例會將資料從沒有身分識別資料行的來源資料表，複製到具有身分識別資料行的目的地資料表。

**來源資料表：**

```sql
create table dbo.SourceTbl
(
       name varchar(100),
       age int
)
```

**目的地資料表：**

```sql
create table dbo.TargetTbl
(
       identifier int identity(1,1),
       name varchar(100),
       age int
)
```

請注意，目標資料表具有身分識別資料行。

**來源資料集 JSON 定義**

```json
{
    "name": "SampleSource",
    "properties": {
        "type": " SqlServerTable",
        "linkedServiceName": {
            "referenceName": "TestIdentitySQL",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "SourceTbl"
        }
    }
}
```

**目的地資料集 JSON 定義**

```json
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "type": "SqlServerTable",
        "linkedServiceName": {
            "referenceName": "TestIdentitySQL",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "TargetTbl"
        }
    }
}
```

請注意，您的來源資料表與目標資料表的結構描述不同 (目標資料表有一個具有身分識別的額外資料行)。 在此案例中，您必須在目標資料集定義中指定 **structure** 屬性，這不包含身分識別資料行。

## <a name="invoking-stored-procedure-for-sql-sink"></a> 從 SQL 接收器叫用預存程序

將資料複製到 SQL Server 資料庫時，可以設定使用者指定的預存程序，並搭配其他參數來叫用此程序。

當內建的複製機制無法滿足需求時，可以使用預存程序。 通常是在最後一次將來源資料插入到目的地資料表之前，如果必須執行更新插入 (插入 + 更新) 或額外的處理 (合併資料行、查閱其他值、插入到多個資料表等) 時，會使用此程序。

下列範例示範如何使用預存程序，對 SQL Server 資料庫中的資料表執行更新插入。 假設輸入資料和接收器 "Marketing" 資料表各有三個資料行：ProfileID、State 及 Category。 根據 “ProfileID” 資料行執行更新插入，然後僅針對特定的類別套用。

**輸出資料集**

```json
{
    "name": "SQLServerDataset",
    "properties":
    {
        "type": "SqlServerTable",
        "linkedServiceName": {
            "referenceName": "<SQL Server linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "Marketing"
        }
    }
}
```

依下列方式定義複製活動中的 SqlSink 區段。

```json
"sink": {
    "type": "SqlSink",
    "SqlWriterTableType": "MarketingType",
    "SqlWriterStoredProcedureName": "spOverwriteMarketing",
    "storedProcedureParameters": {
        "category": {
            "value": "ProductA"
        }
    }
}
```

在資料庫中，使用與 SqlWriterStoredProcedureName 相同的名稱定義預存程序。 它會處理來自指定來源的輸入資料，並合併至輸出資料表。 預存程序中資料表類型的參數名稱應該與資料集中定義的 "tableName" 相同。

```sql
CREATE PROCEDURE spOverwriteMarketing @Marketing [dbo].[MarketingType] READONLY, @category varchar(256)
AS
BEGIN
  MERGE [dbo].[Marketing] AS target
  USING @Marketing AS source
  ON (target.ProfileID = source.ProfileID and target.Category = @category)
  WHEN MATCHED THEN
      UPDATE SET State = source.State
  WHEN NOT MATCHED THEN
      INSERT (ProfileID, State, Category)
      VALUES (source.ProfileID, source.State, source.Category)
END
```

在資料庫中，使用與 sqlWriterTableType 相同的名稱來定義資料表類型。 請注意，資料表類型的結構描述應該與輸入資料所傳回的結構描述相同。

```sql
CREATE TYPE [dbo].[MarketingType] AS TABLE(
    [ProfileID] [varchar](256) NOT NULL,
    [State] [varchar](256) NOT NULL，
    [Category] [varchar](256) NOT NULL，
)
```

預存程序功能使用 [資料表值參數](https://msdn.microsoft.com/library/bb675163.aspx)。

## <a name="data-type-mapping-for-sql-server"></a>SQL Server 的資料類型對應

從 SQL Server 複製資料或將資料複製到該處時，會使用下列從 SQL Server 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| SQL Server 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| bigint |Int64 |
| binary |Byte[] |
| bit |BOOLEAN |
| char |String、Char[] |
| 日期 |Datetime |
| DateTime |Datetime |
| datetime2 |Datetime |
| Datetimeoffset |DateTimeOffset |
| 十進位 |十進位 |
| FILESTREAM 屬性 (varbinary(max)) |Byte[] |
| Float |兩倍 |
| 映像 |Byte[] |
| int |Int32 |
| money |十進位 |
| nchar |String、Char[] |
| ntext |String、Char[] |
| numeric |十進位 |
| nvarchar |String、Char[] |
| real |單一 |
| rowversion |Byte[] |
| smalldatetime |Datetime |
| smallint |Int16 |
| smallmoney |十進位 |
| sql_variant |物件 * |
| text |String、Char[] |
| 分析 |時間範圍 |
| timestamp |Byte[] |
| tinyint |Int16 |
| uniqueidentifier |Guid |
| varbinary |Byte[] |
| varchar |String、Char[] |
| xml |xml |

## <a name="troubleshooting-connection-issues"></a>疑難排解連線問題

1. 將 SQL Server 設定成接受遠端連線。 啟動 [SQL Server Management Studio]、用滑鼠右鍵按一下 [伺服器]，然後按一下 [屬性]。 選取清單中 [連接]，然後核取 [允許此伺服器的遠端連接]。

    ![啟用遠端連線](media/copy-data-to-from-sql-server/AllowRemoteConnections.png)

    如需詳細步驟，請參閱 [設定 remote access 伺服器組態選項](https://msdn.microsoft.com/library/ms191464.aspx) 。

2. 啟動 [SQL Server 組態管理員] 。 展開您想要之執行個體的 [SQL Server 網路組態]，然後選取 [MSSQLSERVER 的通訊協定]。 您應該會在右窗格中看到通訊協定。 用滑鼠右鍵按一下 [TCP/IP]，然後按一下 [啟用] 來啟用 TCP/IP。

    ![啟用 TCP/IP](./media/copy-data-to-from-sql-server/EnableTCPProptocol.png)

    如需啟用 TCP/IP 通訊協定的詳細資料及替代方式，請參閱 [啟用或停用伺服器網路通訊協定](https://msdn.microsoft.com/library/ms191294.aspx) 。

3. 在相同的視窗中，按兩下 [TCP/IP] 來啟動 [TCP/IP 屬性] 視窗。
4. 切換到 [IP 位址] 索引標籤。向下捲動到 [IPAll] 區段。 記下 **TCP 通訊埠** (預設值是 **1433**)。
5. 在電腦上建立 **Windows 防火牆規則** ，來允許透過此連接埠的連入流量。  
6. **確認連線**：若要使用完整名稱來連線到 SQL Server，請使用來自不同機器的 SQL Server Management Studio。 例如： `"<machine>.<domain>.corp.<company>.com,1433"` 。


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
