---
title: 使用 Azure Data Factory 以累加方式複製多個資料表 | Microsoft Docs
description: 在本教學課程中，您會建立 Azure Data Factory 管線，透過累加方式將差異資料從內部部署 SQL Server 資料庫中的多個資料表，複製到 Azure SQL Database。
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: yexu
ms.openlocfilehash: 0cec1fb09503d3cc685b718c2497a363dfd15824
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48868389"
---
# <a name="incrementally-load-data-from-multiple-tables-in-sql-server-to-an-azure-sql-database"></a>以累加方式將 SQL Server 中多個資料表的資料載入到 Azure SQL Database
在本教學課程中，您會建立 Azure Data Factory 與管線，以將差異資料從內部部署 SQL Server 中的多個資料表，載入到 Azure SQL Database。    

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備來源和目的地資料存放區。
> * 建立資料處理站。
> * 建立自我裝載整合執行階段。
> * 安裝整合執行階段。 
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立、執行及監視管線。
> * 檢閱結果。
> * 新增或更新來源資料表中的資料。
> * 重新執行，並監視管線。
> * 檢閱最終結果。

## <a name="overview"></a>概觀
以下是建立此解決方案的重要步驟： 

1. **選取水位線資料行**。
    針對來源資料存放區中的每個資料表各選取一個資料行，以便用於識別每次執行時新增或更新的記錄。 一般來說，當建立或更新資料列時，這個選取的資料行 (例如，last_modify_time 或 ID) 中的資料會持續增加。 此資料行中的最大值就作為水位線。

1. **準備資料存放區來儲存水位線值**。   
    在本教學課程中，您會將水位線值儲存在 SQL 資料庫中。

1. **使用下列活動建立管線**： 
    
    a. 建立 ForEach 活動，逐一查看以參數形式傳遞到管線的來源資料表名稱清單。 此活動會針對每個來源資料表叫用下列活動，以對該資料表執行差異載入。

    b. 建立兩個查閱活動。 使用第一個查閱活動來取出最後一個水位線值。 使用第二個查閱活動來取出新的水位線值。 這些水位線值會傳遞給複製活動。

    c. 建立複製活動，以複製來源資料存放區的資料列，而這些資料列的水位線資料行值大於舊水位線值，且小於新水位線值。 然後，它會將來源資料存放區的差異資料複製到 Azure Blob 儲存體作為新檔案。

    d. 建立 StoredProcedure 活動，以更新下次執行的管線水位線值。 

    高階解決方案圖表如下： 

    ![以累加方式載入資料](media\tutorial-incremental-copy-multiple-tables-powershell\high-level-solution-diagram.png)


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **SQL Server**。 在本教學課程中，您會使用內部部署 SQL 資料庫作為來源資料存放區。 
* **Azure SQL Database**。 您會使用 SQL 資料庫作為接收資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 Azure SQL Database](../sql-database/sql-database-get-started-portal.md)，按照步驟來建立 SQL 資料庫。 

### <a name="create-source-tables-in-your-sql-server-database"></a>在 SQL Server 資料庫中建立來源資料表

1. 開啟 SQL Server Management Studio，然後連線到內部部署 SQL Server 資料庫。

1. 在**伺服器總管**中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

1. 對您的資料庫執行下列 SQL 命令，以建立名為 `customer_table` 和 `project_table` 的資料表：

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
        
    INSERT INTO customer_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'John','9/1/2017 12:56:00 AM'),
    (2, 'Mike','9/2/2017 5:23:00 AM'),
    (3, 'Alice','9/3/2017 2:36:00 AM'),
    (4, 'Andy','9/4/2017 3:21:00 AM'),
    (5, 'Anny','9/5/2017 8:06:00 AM');
    
    INSERT INTO project_table
    (Project, Creationtime)
    VALUES
    ('project1','1/1/2015 0:00:00 AM'),
    ('project2','2/2/2016 1:23:00 AM'),
    ('project3','3/4/2017 5:16:00 AM');
    
    ```

### <a name="create-destination-tables-in-your-azure-sql-database"></a>在 Azure SQL 資料庫中建立目的地資料表
1. 開啟 SQL Server Management Studio，然後連線到 SQL Server 資料庫。

1. 在**伺服器總管**中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

1. 對您的資料庫執行下列 SQL 命令，以建立名為 `customer_table` 和 `project_table` 的 SQL 資料表：  
    
    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );

    ```

### <a name="create-another-table-in-the-azure-sql-database-to-store-the-high-watermark-value"></a>在 Azure SQL Database 中建立另一個資料表來儲存高水位線值
1. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `watermarktable` 的資料表來儲存水位線值： 
    
    ```sql
    create table watermarktable
    (
    
        TableName varchar(255),
        WatermarkValue datetime,
    );
    ```
1. 在水位線資料表中插入這兩個來源資料表的初始水位線值。

    ```sql

    INSERT INTO watermarktable
    VALUES 
    ('customer_table','1/1/2010 12:00:00 AM'),
    ('project_table','1/1/2010 12:00:00 AM');
    
    ```

### <a name="create-a-stored-procedure-in-the-azure-sql-database"></a>在 Azure SQL 資料庫中建立預存程序 

執行下列命令，在您的 SQL 資料庫中建立預存程序。 這個預存程序會在每次管線執行之後更新水位線值。 

```sql
CREATE PROCEDURE sp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

    UPDATE watermarktable
    SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName

END

```

### <a name="create-data-types-and-additional-stored-procedures-in-the-azure-sql-database"></a>在 Azure SQL 資料庫中建立資料類型和其他預存程序
執行下列查詢，在您的 SQL 資料庫中建立兩個預存程序和兩個資料類型。 它們用來將來源資料表的資料合併到目的地資料表。

```sql
CREATE TYPE DataTypeforCustomerTable AS TABLE(
    PersonID int,
    Name varchar(255),
    LastModifytime datetime
);

GO

CREATE PROCEDURE sp_upsert_customer_table @customer_table DataTypeforCustomerTable READONLY
AS

BEGIN
  MERGE customer_table AS target
  USING @customer_table AS source
  ON (target.PersonID = source.PersonID)
  WHEN MATCHED THEN
      UPDATE SET Name = source.Name,LastModifytime = source.LastModifytime
  WHEN NOT MATCHED THEN
      INSERT (PersonID, Name, LastModifytime)
      VALUES (source.PersonID, source.Name, source.LastModifytime);
END

GO

CREATE TYPE DataTypeforProjectTable AS TABLE(
    Project varchar(255),
    Creationtime datetime
);

GO

CREATE PROCEDURE sp_upsert_project_table @project_table DataTypeforProjectTable READONLY
AS

BEGIN
  MERGE project_table AS target
  USING @project_table AS source
  ON (target.Project = source.Project)
  WHEN MATCHED THEN
      UPDATE SET Creationtime = source.Creationtime
  WHEN NOT MATCHED THEN
      INSERT (Project, Creationtime)
      VALUES (source.Project, source.Creationtime);
END

```

### <a name="azure-powershell"></a>Azure PowerShell
依照[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示，安裝最新的 Azure PowerShell 模組。

## <a name="create-a-data-factory"></a>建立 Data Factory
1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如 `"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    不建議您覆寫已經存在的資源群組。 將不同的值指派給 `$resourceGroupName` 變數，然後再執行一次命令。

1. 定義資料處理站位置的變數。 

    ```powershell
    $location = "East US"
    ```
1. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    New-AzureRmResourceGroup $resourceGroupName $location
    ``` 
    不建議您覆寫已經存在的資源群組。 將不同的值指派給 `$resourceGroupName` 變數，然後再執行一次命令。

1. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  更新資料處理站名稱，使其成為全域唯一的資料處理站。 例如，ADFIncMultiCopyTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFIncMultiCopyTutorialFactory";
    ```
1. 若要建立 Data Factory，請執行下列 **Set-AzureRmDataFactoryV2** Cmdlet： 
    
    ```powershell       
    Set-AzureRmDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

請注意下列幾點：

* 資料處理站的名稱必須是全域唯一的名稱。 如果發生下列錯誤，請變更名稱，並再試一次：

    ```
    The specified Data Factory name 'ADFIncMultiCopyTutorialFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為參與者或擁有者角色，或是 Azure 訂用帳戶的管理員。
* 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、SQL Database 等) 和計算 (Azure HDInsight 等) 可位於其他區域。

[!INCLUDE [data-factory-create-install-integration-runtime](../../includes/data-factory-create-install-integration-runtime.md)]



## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本節中，您會對內部部署 SQL Server 資料庫和 SQL 資料庫建立連結服務。 

### <a name="create-the-sql-server-linked-service"></a>建立 SQL Server 連結服務
在此步驟中，您要將內部部署 SQL Server 資料庫連結至資料處理站。

1. 使用下列內容，在 C:\ADFTutorials\IncCopyMultiTableTutorial 資料夾中建立名為 SqlServerLinkedService.json 的 JSON 檔案。 以您用於連線到 SQL Server 的驗證作為基礎，選取右側區段。 建立本機資料夾 (如果尚未存在)。 

    > [!IMPORTANT]
    > 以您用於連線到 SQL Server 的驗證作為基礎，選取右側區段。

    如果您使用 SQL 驗證，請複製下列 JSON 定義：

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<servername>;Database=<databasename>;User ID=<username>;Password=<password>;Timeout=60"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }
   ```    
    如果您使用 Windows 驗證，請複製下列 JSON 定義：

    ```json
    {
        "properties": {
            "type": "SqlServer",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=<server>;Database=<database>;Integrated Security=True"
                },
                "userName": "<user> or <domain>\\<user>",
                "password": {
                    "type": "SecureString",
                    "value": "<password>"
                }
            },
            "connectVia": {
                "type": "integrationRuntimeReference",
                "referenceName": "<integration runtime name>"
            }
        },
        "name": "SqlServerLinkedService"
    }    
    ```
    > [!IMPORTANT]
    > - 以您用於連線到 SQL Server 的驗證作為基礎，選取右側區段。
    > - 以您的整合執行階段名稱取代 &lt;整合執行階段名稱>。
    > - 儲存檔案之前，以您的 SQL Server 資料庫值取代 &lt;servername>、&lt;databasename>、&lt;username> 和 &lt;password>。
    > - 如果您需要在使用者帳戶或伺服器名稱中使用斜線字元 (`\`)，請使用逸出字元 (`\`)。 例如 `mydomain\\myuser`。

1. 在 PowerShell 中，切換至 C:\ADFTutorials\IncCopyMultiTableTutorial 資料夾。

1. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務 AzureStorageLinkedService。 在下列範例中，您會傳遞 ResourceGroupName 和 DataFactoryName 參數的值： 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerLinkedService" -File ".\SqlServerLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : SqlServerLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFIncMultiCopyTutorialFactory1201
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerLinkedService
    ```

### <a name="create-the-sql-database-linked-service"></a>建立 SQL 資料庫連結服務
1. 使用下列內容，在 C:\ADFTutorials\IncCopyMultiTableTutorial 資料夾中建立名為 AzureSQLDatabaseLinkedService.json 的 JSON 檔案。 (建立資料夾 ADF (如果尚未存在)。)儲存檔案之前，以您的 SQL Server 資料庫名稱、資料庫名稱、使用者識別碼和密碼，取代 &lt;server&gt;&lt;database name&gt;、&lt;user id&gt; 和 &lt;password&gt;。 

    ```json
    {
        "name": "AzureSQLDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": {
                    "value": "Server = tcp:<server>.database.windows.net,1433;Initial Catalog=<database name>; Persist Security Info=False; User ID=<user name>; Password=<password>; MultipleActiveResultSets = False; Encrypt = True; TrustServerCertificate = False; Connection Timeout = 30;",
                    "type": "SecureString"
                }
            }
        }
    }
    ```
1. 在 PowerShell 中，執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務 AzureSQLDatabaseLinkedService。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFIncMultiCopyTutorialFactory1201
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集以代表資料來源和資料目的地，以及要儲存水位線的位置。

### <a name="create-a-source-dataset"></a>建立來源資料集

1. 在相同的資料夾中，使用下列內容建立名為 SourceDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": "SourceDataset",
        "properties": {
            "type": "SqlServerTable",
            "typeProperties": {
                "tableName": "dummyName"
            },
            "linkedServiceName": {
                "referenceName": "SqlServerLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
   
    ```

    資料表名稱是虛擬名稱。 管線中的複製活動會使用 SQL 查詢來載入資料，而不會載入整個資料表。

1. 執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 SourceDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFIncMultiCopyTutorialFactory1201
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-sink-dataset"></a>建立接收資料集

1. 在相同的資料夾中，使用下列內容建立名為 SinkDataset.json 的 JSON 檔案。 tableName 元素是由管線在執行階段動態設定。 管線中的 ForEach 活動會逐一查看資料表名稱清單，並將資料表名稱傳遞至每個反覆項目中的這個資料集。 

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": {
                    "value": "@{dataset().SinkTableName}",
                    "type": "Expression"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            },
            "parameters": {
                "SinkTableName": {
                    "type": "String"
                }
            }
        }
    }
    ```

1. 執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 SinkDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFIncMultiCopyTutorialFactory1201
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-a-watermark"></a>建立水位線的資料集
在此步驟中，您會建立資料集來儲存高水位線值。 

1. 在相同的資料夾中，使用下列內容建立名為 WatermarkDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": " WatermarkDataset ",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "watermarktable"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }    
    ```
1. 執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 WatermarkDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "WatermarkDataset" -File ".\WatermarkDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : WatermarkDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : <data factory name>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset    
    ```

## <a name="create-a-pipeline"></a>建立管線
管線會使用資料表名稱清單作為參數。 ForEach 活動會逐一查看資料表名稱清單，並執行下列作業： 

1. 使用查閱活動擷取舊的水位線值 (初始值，或最後一次反覆運算中使用的值)。

1. 使用查閱活動擷取新的水位線值 (來源資料表中水位線資料行的最大值)。

1. 使用複製活動，在來源資料庫到目的地資料庫的這兩個水位線值之間複製資料。

1. 使用 StoredProcedure 活動，更新要在下一個反覆項目的第一個步驟中使用的舊水位線值。 

### <a name="create-the-pipeline"></a>建立管線
1. 在相同的資料夾中，使用下列內容建立名為 IncrementalCopyPipeline.json 的 JSON 檔案： 

    ```json
    {
        "name": "IncrementalCopyPipeline",
        "properties": {
            "activities": [{
    
                "name": "IterateSQLTables",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "false",
                    "items": {
                        "value": "@pipeline().parameters.tableList",
                        "type": "Expression"
                    },
    
                    "activities": [
                        {
                            "name": "LookupOldWaterMarkActivity",
                            "type": "Lookup",
                            "typeProperties": {
                                "source": {
                                    "type": "SqlSource",
                                    "sqlReaderQuery": "select * from watermarktable where TableName  =  '@{item().TABLE_NAME}'"
                                },
    
                                "dataset": {
                                    "referenceName": "WatermarkDataset",
                                    "type": "DatasetReference"
                                }
                            }
                        },
                        {
                            "name": "LookupNewWaterMarkActivity",
                            "type": "Lookup",
                            "typeProperties": {
                                "source": {
                                    "type": "SqlSource",
                                    "sqlReaderQuery": "select MAX(@{item().WaterMark_Column}) as NewWatermarkvalue from @{item().TABLE_NAME}"
                                },
    
                                "dataset": {
                                    "referenceName": "SourceDataset",
                                    "type": "DatasetReference"
                                }
                            }
                        },
    
                        {
                            "name": "IncrementalCopyActivity",
                            "type": "Copy",
                            "typeProperties": {
                                "source": {
                                    "type": "SqlSource",
                                    "sqlReaderQuery": "select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and @{item().WaterMark_Column} <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'"
                                },
                                "sink": {
                                    "type": "SqlSink",
                                    "SqlWriterTableType": "@{item().TableType}",
                                    "SqlWriterStoredProcedureName": "@{item().StoredProcedureNameForMergeOperation}"
                                }
                            },
                            "dependsOn": [{
                                    "activity": "LookupNewWaterMarkActivity",
                                    "dependencyConditions": [
                                        "Succeeded"
                                    ]
                                },
                                {
                                    "activity": "LookupOldWaterMarkActivity",
                                    "dependencyConditions": [
                                        "Succeeded"
                                    ]
                                }
                            ],
    
                            "inputs": [{
                                "referenceName": "SourceDataset",
                                "type": "DatasetReference"
                            }],
                            "outputs": [{
                                "referenceName": "SinkDataset",
                                "type": "DatasetReference",
                                "parameters": {
                                    "SinkTableName": "@{item().TableType}"
                                }
                            }]
                        },
    
                        {
                            "name": "StoredProceduretoWriteWatermarkActivity",
                            "type": "SqlServerStoredProcedure",
                            "typeProperties": {
    
                                "storedProcedureName": "sp_write_watermark",
                                "storedProcedureParameters": {
                                    "LastModifiedtime": {
                                        "value": "@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}",
                                        "type": "datetime"
                                    },
                                    "TableName": {
                                        "value": "@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}",
                                        "type": "String"
                                    }
                                }
                            },
    
                            "linkedServiceName": {
                                "referenceName": "AzureSQLDatabaseLinkedService",
                                "type": "LinkedServiceReference"
                            },
    
                            "dependsOn": [{
                                "activity": "IncrementalCopyActivity",
                                "dependencyConditions": [
                                    "Succeeded"
                                ]
                            }]
                        }
    
                    ]
    
                }
            }],
    
            "parameters": {
                "tableList": {
                    "type": "Object"
                }
            }
        }
    }
    ```
1. 執行 **Set-AzureRmDataFactoryV2Pipeline** Cmdlet 以建立管線 IncrementalCopyPipeline。
    
   ```powershell
   Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ``` 

   以下是範例輸出： 

   ```json
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFIncMultiCopyTutorialFactory1201
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
   ```
 
## <a name="run-the-pipeline"></a>執行管道

1. 在相同的資料夾中，使用下列內容建立名為 Parameters.json 的參數檔案：

    ```json
    {
        "tableList": 
        [
            {
                "TABLE_NAME": "customer_table",
                "WaterMark_Column": "LastModifytime",
                "TableType": "DataTypeforCustomerTable",
                "StoredProcedureNameForMergeOperation": "sp_upsert_customer_table"
            },
            {
                "TABLE_NAME": "project_table",
                "WaterMark_Column": "Creationtime",
                "TableType": "DataTypeforProjectTable",
                "StoredProcedureNameForMergeOperation": "sp_upsert_project_table"
            }
        ]
    }
    ```
1. 使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管線 IncrementalCopyPipeline。 以您自己的資源群組和資料處理站名稱取代預留位置。

    ```powershell
    $RunId = Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"        
    ``` 

## <a name="monitor-the-pipeline"></a>監視管線

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 選取 [所有服務]，以關鍵字「資料處理站」進行搜尋，然後選取 [資料處理站]。 

    ![Data Factory 功能表](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-data-factories-menu-1.png)

1. 在 Data Factory 清單中搜尋您的 Data Factory，然後加以選取以開啟 [Data Factory] 頁面。 

    ![搜尋您的 Data Factory](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-search-data-factory-2.png)

1. 在 [Data Factory] 頁面上，選取 [監視及管理]。 

    ![監視及管理圖格](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-monitor-manage-tile-3.png)

1. [資料整合應用程式] 會在不同的索引標籤中開啟。您可以看到所有管線執行及其狀態。 請注意，在下列範例中，管線執行狀態是 [成功]。 若要檢查傳遞到管線的參數，請選取 [參數] 資料行中的連結。 如果發生錯誤，您就會在 [錯誤] 資料行中看到連結。 選取 [動作] 資料行中的連結。 

    ![管線執行回合](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-pipeline-runs-4.png)    
1. 當您選取 [動作] 資料行中的連結時，您會看到下列頁面，其中顯示管線的所有活動執行： 

    ![活動執行](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-activity-runs-5.png)

1. 若要回到 [管線執行] 檢視，請選取 [管線]，如下圖所示。 

## <a name="review-the-results"></a>檢閱結果
在 SQL Server Management Studio 中，對目標 SQL 資料庫執行下列查詢，以確認資料已從來源資料表複製到目的地資料表： 

**查詢** 
```sql
select * from customer_table
```

**輸出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           Alice   2017-09-03 02:36:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

**查詢**

```sql
select * from project_table
```

**輸出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
```

**查詢**

```sql
select * from watermarktable
```

**輸出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-05 08:06:00.000
project_table   2017-03-04 05:16:00.000
```

請注意，這兩個資料表的水位線值已更新。 

## <a name="add-more-data-to-the-source-tables"></a>新增更多資料至來源資料表

對來源 SQL Server 資料庫執行下列查詢，以更新 customer_table 中的現有資料列。 在 project_table 中插入新的資料列。 

```sql
UPDATE customer_table
SET [LastModifytime] = '2017-09-08T00:00:00Z', [name]='NewName' where [PersonID] = 3

INSERT INTO project_table
(Project, Creationtime)
VALUES
('NewProject','10/1/2017 0:00:00 AM');
``` 

## <a name="rerun-the-pipeline"></a>重新執行管線

1. 現在，執行下列 PowerShell 命令以重新執行管線：

    ```powershell
    $RunId = Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupname -dataFactoryName $dataFactoryName -ParameterFile ".\Parameters.json"
    ```
1. 遵循[監視管線](#monitor-the-pipeline)一節中的指示，以監視管線執行。 因為管線狀態為 [進行中]，所以您會在 [動作] 底下看到另一個動作連結，其可供取消管線執行。 

    ![進行中監視管線執行](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-pipeline-runs-6.png)

1. 選取 [重新整理] 可重新整理清單，直到管線執行成功。 

    ![重新整理管線執行](media\tutorial-incremental-copy-multiple-tables-powershell\monitor-pipeline-runs-succeded-7.png)

1. 選擇性地，選取 [動作] 底下的 [檢視活動執行] 連結，可查看與此管線執行相關聯的所有活動執行。 

## <a name="review-the-final-results"></a>檢閱最終結果
在 SQL Server Management Studio 中，對目標資料庫執行下列查詢，以確認經過更新/全新的資料已從來源資料表複製到目的地資料表。 

**查詢** 
```sql
select * from customer_table
```

**輸出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           NewName 2017-09-08 00:00:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

請注意，**PersonID** 為 3 的 **Name** 和 **LastModifytime** 的新值。 

**查詢**

```sql
select * from project_table
```

**輸出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
NewProject  2017-10-01 00:00:00.000
```

請注意，project_table 中已新增 **NewProject** 項目。 

**查詢**

```sql
select * from watermarktable
```

**輸出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-08 00:00:00.000
project_table   2017-10-01 00:00:00.000
```

請注意，這兩個資料表的水位線值已更新。
     
## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 準備來源和目的地資料存放區。
> * 建立資料處理站。
> * 建立自我裝載的整合執行階段 (IR)。
> * 安裝整合執行階段。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立、執行及監視管線。
> * 檢閱結果。
> * 新增或更新來源資料表中的資料。
> * 重新執行，並監視管線。
> * 檢閱最終結果。

進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[使用變更追蹤技術，以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體](tutorial-incremental-copy-change-tracking-feature-powershell.md)


