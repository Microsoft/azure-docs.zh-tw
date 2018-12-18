---
title: 使用 Azure Data Factory 從 Azure SQL 資料倉儲來回複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory 將資料從支援的來源存放區複製到「Azure SQL 資料倉儲」，或從「SQL 資料倉儲」複製到支援的接收存放區。
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
ms.date: 07/28/2018
ms.author: jingwang
ms.openlocfilehash: ef1bd613943543f78d358064f4abefc6fa31b63e
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842330"
---
#  <a name="copy-data-to-or-from-azure-sql-data-warehouse-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料複製到 Azure SQL 資料倉儲或從該處複製資料 
> [!div class="op_single_selector" title1="Select the version of Data Factory service you're using:"]
> * [第 1 版](v1/data-factory-azure-sql-data-warehouse-connector.md)
> * [目前的版本](connector-azure-sql-data-warehouse.md)

本文說明如何使用 Azure Data Factory 中的「複製活動」，從 Azure SQL 資料倉儲來回複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Azure SQL 資料倉儲複製到任何支援的接收資料存放區。 您也可以將資料從任何支援的來源資料存放區複製到 Azure SQL 資料倉儲。 如需複製活動所支援作為來源或接收的資料存放區清單，請參閱[支援的資料存放區和格式](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個「Azure SQL 資料倉儲」連接器支援下列功能：

- 使用 SQL 驗證和 Azure Active Directory (Azure AD) 應用程式權杖驗證與服務主體或受控服務識別 (MSI) 來複製資料。
- 作為來源時，使用 SQL 查詢或預存程序來擷取資料。
- 做為接收時，使用 PolyBase 或大量插入來載入資料。 建議您使用 PolyBase 以獲得較佳的複製效能。

> [!IMPORTANT]
> 請注意，PolyBase 只支援 SQL 驗證，不支援 Azure AD 驗證。

> [!IMPORTANT]
> 如果您使用Azure Data Factory Integration Runtime 複製資料，請設定 [Azure SQL 伺服器防火牆](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)，讓 Azure 服務可存取伺服器。
> 如果您使用自我裝載整合執行階段來複製資料，請將 Azure SQL 伺服器防火牆設定為允許適當的 IP 範圍。 此範圍包括用來連線到 Azure SQL Database 的機器 IP。

## <a name="get-started"></a>開始使用

> [!TIP]
> 若要達到最佳效能，請使用 PolyBase 將資料載入 Azure SQL 資料倉儲。 如需詳細資訊，請參閱 [使用 PolyBase 將資料載入 Azure SQL 資料倉儲](#use-polybase-to-load-data-into-azure-sql-data-warehouse) 。 如需使用案例的逐步解說，請參閱[使用 Azure Data Factory 在 15 分鐘內將 1 TB 載入至 Azure SQL 資料倉儲](load-azure-sql-data-warehouse.md)。

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性會定義「Azure SQL 資料倉儲」連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是 Azure SQL 資料倉儲連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為 **AzureSqlDW**。 | 是 |
| connectionString | 針對 **connectionString** 屬性指定連線到 Azure SQL 資料倉儲執行個體所需的資訊。 將此欄位標記為 **SecureString**，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| servicePrincipalId | 指定應用程式的用戶端識別碼。 | 當您搭配服務主體使用 Azure AD 驗證時為是。 |
| servicePrincipalKey | 指定應用程式的金鑰。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 當您搭配服務主體使用 Azure AD 驗證時為是。 |
| tenant | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | 當您搭配服務主體使用 Azure AD 驗證時為是。 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載整合執行階段 (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 | 否 |

針對不同的驗證類型，請分別參閱下列有關先決條件和 JSON 範例的章節：

- [SQL 驗證](#sql-authentication)
- Azure AD 應用程式權杖驗證：[服務主體](#service-principal-authentication)
- Azure AD 應用程式權杖驗證：[受控服務識別](#managed-service-identity-authentication)

>[!TIP]
>如果您遇到錯誤，其錯誤碼為 "UserErrorFailedToConnectToSqlServer"，以及「資料庫的工作階段限制為 XXX 並已達到。」訊息，請將 `Pooling=false` 新增至您的連接字串並再試一次。

### <a name="sql-authentication"></a>SQL 驗證

#### <a name="linked-service-example-that-uses-sql-authentication"></a>使用 SQL 驗證的連結服務範例

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>服務主體驗證

若要使用以服務主體為基礎的 Azure AD 應用程式權杖驗證，請遵循下列步驟：

1. 從 Azure 入口網站**[建立 Azure Active Directory 應用程式](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)**。 請記下應用程式名稱，以及下列可定義連結服務的值：

    - 應用程式識別碼
    - 應用程式金鑰
    - 租用戶識別碼

1. 如果您尚未這麼做，請在 Azure 入口網站上針對您的 Azure SQL 伺服器**[佈建 Azure Active Directory 系統管理員](../sql-database/sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)**。 Azure AD 系統管理員可以是 Azure AD 使用者或 Azure AD 群組。 如果您已對具有 MSI 的群組授與系統管理員角色，請略過步驟 3 和 4。 系統管理員將擁有完整的資料庫存取權。

1. 為服務主體**[建立自主資料庫使用者](../sql-database/sql-database-aad-authentication-configure.md#create-contained-database-users-in-your-database-mapped-to-azure-ad-identities)**。 以至少具有 ALTER ANY USER 權限的 AAD 身分識別，使用 SSMS 這類工具連線至您想要從中來回複製資料的資料倉儲。 執行下列 T-SQL：
    
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER;
    ```

1. 如同您一般對 SQL 使用者或其他人所做的一樣，**將所需的權限授與服務主體**。 執行下列程式碼：

    ```sql
    EXEC sp_addrolemember [role name], [your application name];
    ```

1. 在 Azure Data Factory 中，**設定 Azure SQL 資料倉儲連結服務**。


#### <a name="linked-service-example-that-uses-service-principal-authentication"></a>使用服務主體驗證的連結服務範例

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
            },
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-service-identity-authentication"></a>受控服務識別驗證

資料處理站可與[受控服務識別](data-factory-service-identity.md)相關聯，用後者來表示此特定處理站。 您可以使用此服務識別來進行 Azure SQL 資料倉儲驗證。 指定的處理站可以使用此身分識別來存取資料倉儲和從中來回複製資料。

> [!IMPORTANT]
> 請注意，PolyBase 目前不支援 MSI 驗證。

若要使用以 MSI 為基礎的 Azure AD 應用程式權杖驗證，請遵循下列步驟：

1. **在 Azure AD 中建立群組。** 讓處理站 MSI 成為群組成員。

    1. 從 Azure 入口網站尋找資料處理站服務識別。 移至資料處理站的 [屬性]。 複製 SERVICE IDENTITY ID。

    1. 安裝 [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2) 模組。 使用 `Connect-AzureAD` 命令登入。 執行下列命令來建立群組，並將資料處理站 MSI 新增為成員。
    ```powershell
    $Group = New-AzureADGroup -DisplayName "<your group name>" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
    Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId "<your data factory service identity ID>"
    ```

1. 如果您尚未這麼做，請在 Azure 入口網站上針對您的 Azure SQL 伺服器**[佈建 Azure Active Directory 系統管理員](../sql-database/sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)**。

1. 為 Azure AD 群組**[建立自主資料庫使用者](../sql-database/sql-database-aad-authentication-configure.md#create-contained-database-users-in-your-database-mapped-to-azure-ad-identities)**。 以至少具有 ALTER ANY USER 權限的 AAD 身分識別，使用 SSMS 這類工具連線至您想要從中來回複製資料的資料倉儲。 執行下列 T-SQL。 
    
    ```sql
    CREATE USER [your Azure AD group name] FROM EXTERNAL PROVIDER;
    ```

1. 像您一般對 SQL 使用者和其他人所做的一樣，**授與 Azure AD 群組所需的權限**。 例如，執行下列程式碼。

    ```sql
    EXEC sp_addrolemember [role name], [your Azure AD group name];
    ```

1. 在 Azure Data Factory 中，**設定 Azure SQL 資料倉儲連結服務**。

#### <a name="linked-service-example-that-uses-msi-authentication"></a>使用 MSI 驗證的連結服務範例

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](https://docs.microsoft.com/en-us/azure/data-factory/concepts-datasets-linked-services)一文。 本節提供「Azure SQL 資料倉儲」資料集所支援的屬性清單。

若要從「Azure SQL 資料倉儲」複製資料或將資料複製到該處，請將資料集的**類型**屬性設定為 **AzureSqlDWTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的**類型**屬性必須設定為 **AzureSqlDWTable**。 | 是 |
| tableName | 「Azure SQL 資料倉儲」執行個體中連結服務所參考的資料表或檢視名稱。 | 是 |

#### <a name="dataset-properties-example"></a>資料集屬性範例

```json
{
    "name": "AzureSQLDWDataset",
    "properties":
    {
        "type": "AzureSqlDWTable",
        "linkedServiceName": {
            "referenceName": "<Azure SQL Data Warehouse linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "MyTable"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供「Azure SQL 資料倉儲」來源和接收所支援的屬性清單。

### <a name="azure-sql-data-warehouse-as-the-source"></a>Azure SQL 資料倉儲作為來源

若要從「Azure SQL 資料倉儲」複製資料，請將複製活動來源中的**類型**屬性設定為 **SqlDWSource**。 複製活動的 [來源] 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的**類型**屬性必須設定為 **SqlDWSource**。 | 是 |
| SqlReaderQuery | 使用自訂 SQL 查詢來讀取資料。 範例： `select * from MyTable`. | 否 |
| sqlReaderStoredProcedureName | 從來源資料表讀取資料的預存程序名稱。 最後一個 SQL 陳述式必須是預存程序中的 SELECT 陳述式。 | 否 |
| storedProcedureParameters | 預存程序的參數。<br/>允許的值為名稱或值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 | 否 |

### <a name="points-to-note"></a>注意事項

- 如果已為 **SqlSource** 指定 **sqlReaderQuery**，「複製活動」就會針對「Azure SQL 資料倉儲」來源執行這項查詢以取得資料。 或者，您可以指定預存程序。 指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數)。
- 如果您未指定 **sqlReaderQuery** 或 **sqlReaderStoredProcedureName**，就會使用資料集 JSON **structure** 區段中定義的資料行來建構查詢。 `select column1, column2 from mytable` 會針對 Azure SQL 資料倉儲執行。 如果資料集定義沒有 **structure**，則會從資料表中選取所有資料行。
- 當您使用 **sqlReaderStoredProcedureName** 時，仍然必須在資料集 JSON 中指定一個虛設的 **tableName**。

#### <a name="sql-query-example"></a>SQL 查詢範例

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="stored-procedure-example"></a>預存程序範例

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
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

### <a name="stored-procedure-definition"></a>預存程序定義

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

### <a name="azure-sql-data-warehouse-as-sink"></a>Azure SQL 資料倉儲作為接收端

若要將資料複製到「Azure SQL 資料倉儲」，請將複製活動中的接收類型設定為 **SqlDWSink**。 複製活動的 [接收] 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收端的**類型**屬性必須設定為 **SqlDWSink**。 | 是 |
| allowPolyBase | 指出是否使用 PolyBase (適用的話) 而不是使用 BULKINSERT 機制。 <br/><br/> 建議使用 PolyBase 將資料載入 SQL 資料倉儲。 請參閱[使用 PolyBase 將資料載入 Azure SQL 資料倉儲](#use-polybase-to-load-data-into-azure-sql-data-warehouse)一節中的條件約束和詳細資料。<br/><br/>允許的值為 **True** 和 **False** (預設值)。  | 否 |
| polyBaseSettings | 可以在 **allowPolybase** 屬性設定為 **true** 時指定的一組屬性。 | 否 |
| rejectValue | 指定在查詢失敗前可以拒絕的資料列數目或百分比。<br/><br/>在 [CREATE EXTERNAL TABLE (Transact-SQL)](https://msdn.microsoft.com/library/dn935021.aspx)的＜引數＞一節中，深入了解 PolyBase 的拒絕選項。 <br/><br/>允許的值為 0 (預設值)、1、2 等其他值。 |否 |
| rejectType | 指定 **rejectValue** 選項為常值或百分比。<br/><br/>允許的值為**值** (預設值) 和**百分比**。 | 否 |
| rejectSampleValue | 決定在 PolyBase 重新計算已拒絕的資料列百分比之前，所要擷取的資料列數目。<br/><br/>允許的值為 1、2 等其他值。 | 是，如果 **rejectType** 是**百分比**。 |
| useTypeDefault | 指定當 PolyBase 從文字檔擷取資料時，如何處理分隔符號文字檔中的遺漏值。<br/><br/>從 [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://msdn.microsoft.com/library/dn935026.aspx) 的＜引數＞一節深入了解這個屬性。<br/><br/>允許的值為 **True** 和 **False** (預設值)。 | 否 |
| writeBatchSize | 當緩衝區大小達到 **writeBatchSize** 時，將資料插入 SQL 資料表中。 只有在未使用 PolyBase 時才適用。<br/><br/>允許的值為**整數** (資料列數目)。 | 否。 預設值為 10000。 |
| writeBatchTimeout | 在逾時前等待批次插入作業完成的時間。只有在未使用 PolyBase 時才適用。<br/><br/>允許的值為**時間範圍**。 範例：“00:30:00” (30 分鐘)。 | 否 |
| preCopyScript | 指定一個供「複製活動」在每次執行時將資料寫入到「Azure SQL 資料倉儲」前執行的 SQL 查詢。 使用此屬性來清除預先載入的資料。 | 否 | (#repeatability-during-copy)。 | 查詢陳述式。 | 否 |

#### <a name="sql-data-warehouse-sink-example"></a>SQL 資料倉儲接收範例

```json
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

若要深入了解如何使用 PolyBase 來有效率地載入「SQL 資料倉儲」，請參閱下一節。

## <a name="use-polybase-to-load-data-into-azure-sql-data-warehouse"></a>使用 PolyBase 將資料載入 Azure SQL 資料倉儲

使用 [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) 是以高輸送量將大量資料載入 Azure SQL 資料倉儲的有效方法。 使用 PolyBase 而不是預設的 BULKINSERT 機制，將可看到輸送量大幅提升。 如需詳細的比較，請參閱[效能參考](copy-activity-performance.md#performance-reference)。 如需使用案例的逐步解說，請參閱[將 1 TB 載入至 Azure SQL 資料倉儲](https://docs.microsoft.com/en-us/azure/data-factory/v1/data-factory-load-sql-data-warehouse)。

* 如果您的來源資料位在 Azure Blob 儲存體或 Azure Data Lake Store，且其格式與 PolyBase 相容，即可使用 PolyBase 直接複製到 Azure SQL 資料倉儲。 如需詳細資料，請參閱**[使用 PolyBase 直接複製](#direct-copy-by-using-polybase)**。
* 如果您的來源資料存放區與格式不受 PolyBase 支援，您可以改用**[使用 PolyBase 分段複製](#staged-copy-by-using-polybase)** 功能。 分段複製功能也能提供更好的輸送量。 它會自動將資料轉換成與 PolyBase 相容的格式。 並會將資料儲存在 Azure Blob 儲存體中。 然後，它會將資料載入 SQL 資料倉儲。

> [!IMPORTANT]
> 請注意，PolyBase 目前不支援以 MSI 為基礎的 Azure AD 應用程式權杖驗證。

### <a name="direct-copy-by-using-polybase"></a>使用 PolyBase 直接複製

SQL 資料倉儲 PolyBase 直接支援 Azure Blob 和 Azure Data Lake Store。 它會使用服務主體作為來源，並具有特定的檔案格式需求。 如果您的來源資料符合本節所述準則，即可使用 PolyBase 從來源資料存放區直接複製到 Azure SQL 資料倉儲。 否則，請利用[使用 PolyBase 分段複製](#staged-copy-by-using-polybase)。

> [!TIP]
> 若要有效率地將資料從 Data Lake Store 複製到 SQL 資料倉儲，深入了解[使用 Data Lake Store 與 SQL 資料倉儲時，Azure Data Factory 能讓您更輕鬆容易發現資料中的重要資訊](https://blogs.msdn.microsoft.com/azuredatalake/2017/04/08/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse/)。

如果不符合需求，Azure Data Factory 會檢查設定，並自動切換回適用於資料移動的 BULKINSERT 機制。

1. [來源連結的服務] 類型是採用帳戶金鑰驗證的 Azure Blob 儲存體 (**AzureBLobStorage**/**AzureStorage**) 或採用服務主體驗證的 Azure Data Lake Storage Gen1 (**AzureDataLakeStore**)。
2. **輸入資料集**類型為 **AzureBlob** 或 **AzureDataLakeStoreFile**。 `type` 屬性下方的格式類型為 **OrcFormat**、**ParquetFormat** 或 **TextFormat**，並且具備下列組態：

   1. `fileName` 不包含萬用字元篩選條件。
   2. `rowDelimiter` 必須為 **\n**。
   3. `nullValue` 是設定為**空字串** ("") 或保留預設值，且 `treatEmptyAsNull` 不是設定為 false。
   4. `encodingName` 會設定為 **utf-8**，也就是預設值。
   5. 未指定 `escapeChar`、`quoteChar` 與 `skipLineCount`。 PolyBase 支援略過標頭列，這在 ADF 中可設定為 `firstRowAsHeader`。
   6. `compression` 可以是「無壓縮」、**GZip** 或 **Deflate**。

    ```json
    "typeProperties": {
       "folderPath": "<blobpath>",
       "format": {
           "type": "TextFormat",
           "columnDelimiter": "<any delimiter>",
           "rowDelimiter": "\n",
           "nullValue": "",
           "encodingName": "utf-8",
           "firstRowAsHeader": <any>
       },
       "compression": {
           "type": "GZip",
           "level": "Optimal"
       }
    },
    ```

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "BlobDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            }
        }
    }
]
```

### <a name="staged-copy-by-using-polybase"></a>使用 PolyBase 分段複製

當來源資料不符合上一節中的準則時，請啟用透過過渡暫存 Azure Blob 儲存體執行個體複製資料。 這不可以是 Azure 進階儲存體。 在此情況下，Azure Data Factory 會自動執行資料轉換，以符合 PolyBase 的資料格式需求。 然後，它會使用 PolyBase 將資料載入 SQL 資料倉儲。 最後，它會清除 Blob 儲存體中的暫存資料。 如需透過暫存 Azure Blob 儲存體執行個體複製資料的詳細資訊，請參閱[分段複製](copy-activity-performance.md#staged-copy)。

若要使用此功能，請建立 [Azure 儲存體連結服務](connector-azure-blob-storage.md#linked-service-properties)，讓其參考具有過渡 Blob 儲存體的 Azure 儲存體帳戶。 然後指定複製活動的 `enableStaging` 和 `stagingSettings` 屬性，如下列程式碼所示：

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

## <a name="best-practices-for-using-polybase"></a>使用 PolyBase 的最佳做法

除了＜[Azure SQL 資料倉儲的最佳做法](../sql-data-warehouse/sql-data-warehouse-best-practices.md)＞中所述的方法外，下列章節還提供其他最佳做法。

### <a name="required-database-permission"></a>必要的資料庫權限

若要使用 PolyBase，將資料載入 SQL 資料倉儲的使用者必須具備目標資料庫的 ["CONTROL" 權限](https://msdn.microsoft.com/library/ms191291.aspx)。 達到此目標的其中一個方法是將該使用者新增為 **db_owner** 角色的成員。 在 [SQL 資料倉儲概觀](../sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization)中了解如何執行此作業。

### <a name="row-size-and-data-type-limits"></a>資料列大小和資料類型限制

PolyBase 負載的限制為小於 1 MB 的資料列。 這些資料列不可載入 VARCHR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX)。 如需詳細資訊，請參閱 [SQL 資料倉儲服務容量限制](../sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads)。

當您來源資料中的資料列大於 1 MB 時，您可能要將來源資料表垂直分割成幾個小的資料表。 務必確認每一列的大小不會超過限制。 然後可以使用 PolyBase 載入較小的資料表，並且在 Azure SQL 資料倉儲中將其合併在一起。

### <a name="sql-data-warehouse-resource-class"></a>SQL 資料倉儲資源類別

若要達到最佳的可能輸送量，請透過 PolyBase 將較大型資源類別指派給要將資料載入 SQL 資料倉儲的使用者。

### <a name="tablename-in-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲中的 **tableName**

下表是如何在 JSON 資料集中指定 **tableName** 屬性的範例。 其中會顯示數個結構描述和資料表名稱的組合。

| DB 結構描述 | 資料表名稱 | **tableName** JSON 屬性 |
| --- | --- | --- |
| dbo | MyTable | MyTable 或 dbo.MyTable 或 [dbo].[MyTable] |
| dbo1 | MyTable | dbo1.MyTable 或 [dbo1].[MyTable] |
| dbo | My.Table | [My.Table] 或 [dbo].[My.Table] |
| dbo1 | My.Table | [dbo1].[My.Table] |

如果您看到下列錯誤，可能是您為 **tableName** 屬性指定的值有問題。 請參閱前面的資料表，以正確的方式指定 **tableName** JSON 屬性的值。

```
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

### <a name="columns-with-default-values"></a>包含預設值的資料行

Data Factory 中的 PolyBase 功能目前只接受與目標資料表中相同的資料行數目。 範例是內含四個資料行的資料表，且其中一個資料行已使用預設值進行定義。 輸入資料仍需要有四個資料行。 3 個資料行的輸入資料集會產生類似下列訊息的錯誤︰

```
All columns of the table must be specified in the INSERT BULK statement.
```

NULL 值是一種特殊形式的預設值。 如果資料欄可以是 Null，Blob 中該欄的輸入資料可能會是空白。 但輸入資料集中不能缺少輸入資料。 PolyBase 會在 Azure SQL 資料倉儲中為遺漏值插入 NULL。

## <a name="data-type-mapping-for-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲的資料類型對應

從「Azure SQL 資料倉儲」複製資料或將資料複製到該處時，會使用下列從「Azure SQL 資料倉儲」資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收。

| Azure SQL 資料倉儲資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| bigint | Int64 |
| binary | Byte[] |
| bit | BOOLEAN |
| char | String、Char[] |
| 日期 | Datetime |
| DateTime | Datetime |
| datetime2 | Datetime |
| Datetimeoffset | DateTimeOffset |
| 十進位 | 十進位 |
| FILESTREAM 屬性 (varbinary(max)) | Byte[] |
| Float | 兩倍 |
| 映像 | Byte[] |
| int | Int32 |
| money | 十進位 |
| nchar | String、Char[] |
| ntext | String、Char[] |
| numeric | 十進位 |
| nvarchar | String、Char[] |
| real | 單一 |
| rowversion | Byte[] |
| smalldatetime | Datetime |
| smallint | Int16 |
| smallmoney | 十進位 |
| sql_variant | 物件 * |
| text | String、Char[] |
| 分析 | 時間範圍 |
| timestamp | Byte[] |
| tinyint | Byte |
| uniqueidentifier | Guid |
| varbinary | Byte[] |
| varchar | String、Char[] |
| xml | xml |

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收端的資料存放區清單，請參閱[支援的資料存放區和格式](copy-activity-overview.md##supported-data-stores-and-formats)。
