---
title: 使用 Azure Data Factory 以累加方式複製資料表 | Microsoft Docs
description: 在本教學課程中，您會建立 Azure Data Factory 管線，以累加方式將資料從 Azure SQL Database 複製到 Azure Blob 儲存體。
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
ms.openlocfilehash: 08bce244dc4eafcd423123b1230fe4aa8b4ed04e
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43092034"
---
# <a name="incrementally-load-data-from-an-azure-sql-database-to-azure-blob-storage"></a>以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體
在本教學課程中，您會建立 Azure Data Factory 與管線，以將差異資料從 Azure SQL Database 中的資料表載入到 Azure Blob 儲存體。 

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備資料存放區來儲存水位線值。
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立管道。
> * 執行管道。
> * 監視管道執行。 

## <a name="overview"></a>概觀
高階解決方案圖表如下： 

![以累加方式載入資料](media\tutorial-Incrementally-copy-powershell\incrementally-load.png)

以下是建立此解決方案的重要步驟： 

1. **選取水位線資料行**。
    選取來源資料存放區中的一個資料行，可用於切割每次執行時新增或更新的記錄。 一般來說，當建立或更新資料列時，這個選取的資料行 (例如，last_modify_time 或 ID) 中的資料會持續增加。 此資料行中的最大值就作為水位線。

2. **準備資料存放區來儲存水位線值**。   
    在本教學課程中，您會將水位線值儲存在 SQL 資料庫中。
    
3. **使用下列工作流程建立管線**： 
    
    此解決方案中的管道有下列活動：
  
    * 建立兩個查閱活動。 使用第一個查閱活動來取出最後一個水位線值。 使用第二個查閱活動來取出新的水位線值。 這些水位線值會傳遞給複製活動。 
    * 建立複製活動，以複製來源資料存放區的資料列，而這些資料列的水位線資料行值大於舊水位線值，且小於新水位線值。 然後，它會將來源資料存放區的差異資料複製到 Blob 儲存體作為新檔案。 
    * 建立 StoredProcedure 活動，以更新下次執行之管線的水位線值。 


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **Azure SQL Database**。 您需要使用資料庫作為來源資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 Azure SQL Database](../sql-database/sql-database-get-started-portal.md)，按照步驟來建立 SQL 資料庫。
* **Azure 儲存體**。 您需要使用 Blob 儲存體作為接收資料存放區。 如果您沒有儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)，按照步驟來建立儲存體帳戶。 建立名為 adftutorial 的容器。 
* **Azure PowerShell**(英文)。 遵循[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示。

### <a name="create-a-data-source-table-in-your-sql-database"></a>在 SQL 資料庫中建立資料來源資料表
1. 開啟 SQL Server Management Studio。 在 [伺服器總管] 中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

2. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `data_source_table` 的資料表作為資料來源存放區： 
    
    ```sql
    create table data_source_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );

    INSERT INTO data_source_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'aaaa','9/1/2017 12:56:00 AM'),
    (2, 'bbbb','9/2/2017 5:23:00 AM'),
    (3, 'cccc','9/3/2017 2:36:00 AM'),
    (4, 'dddd','9/4/2017 3:21:00 AM'),
    (5, 'eeee','9/5/2017 8:06:00 AM');
    ```
    在本教學課程中，您會使用 LastModifytime 作為水位線資料行。 下表顯示資料來源存放區中的資料：

    ```
    PersonID | Name | LastModifytime
    -------- | ---- | --------------
    1 | aaaa | 2017-09-01 00:56:00.000
    2 | bbbb | 2017-09-02 05:23:00.000
    3 | cccc | 2017-09-03 02:36:00.000
    4 | dddd | 2017-09-04 03:21:00.000
    5 | eeee | 2017-09-05 08:06:00.000
    ```

### <a name="create-another-table-in-your-sql-database-to-store-the-high-watermark-value"></a>在 SQL 資料庫中建立另一個資料表來儲存高水位線值
1. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `watermarktable` 的資料表來儲存水位線值：  
    
    ```sql
    create table watermarktable
    (
    
    TableName varchar(255),
    WatermarkValue datetime,
    );
    ```
2. 使用來源資料存放區的資料表名稱來設定高水位線的預設值。 在本教學課程中，資料表名稱是 data_source_table。

    ```sql
    INSERT INTO watermarktable
    VALUES ('data_source_table','1/1/2010 12:00:00 AM')    
    ```
3. 檢閱資料表 `watermarktable` 中的資料。
    
    ```sql
    Select * from watermarktable
    ```
    輸出： 

    ```
    TableName  | WatermarkValue
    ----------  | --------------
    data_source_table | 2010-01-01 00:00:00.000
    ```

### <a name="create-a-stored-procedure-in-your-sql-database"></a>在 SQL 資料庫中建立預存程序 

執行下列命令，在您的 SQL 資料庫中建立預存程序：

```sql
CREATE PROCEDURE sp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN
    
    UPDATE watermarktable
    SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName
    
END
```

## <a name="create-a-data-factory"></a>建立 Data Factory
1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如 `"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    不建議您覆寫已經存在的資源群組。 將不同的值指派給 `$resourceGroupName` 變數，然後再執行一次命令。

2. 定義資料處理站位置的變數。 

    ```powershell
    $location = "East US"
    ```
3. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    New-AzureRmResourceGroup $resourceGroupName $location
    ``` 
    不建議您覆寫已經存在的資源群組。 將不同的值指派給 `$resourceGroupName` 變數，然後再執行一次命令。

4. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  更新資料處理站名稱，使其成為全域唯一的資料處理站。 例如，ADFTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFIncCopyTutorialFactory";
    ```
5. 若要建立 Data Factory，請執行下列 **Set-AzureRmDataFactoryV2** Cmdlet： 
    
    ```powershell       
    Set-AzureRmDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName 
    ```

請注意下列幾點：

* 資料處理站的名稱必須是全域唯一的名稱。 如果發生下列錯誤，請變更名稱，並再試一次：

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```

* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為參與者或擁有者角色，或是 Azure 訂用帳戶的管理員。
* 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (儲存體、SQL Database 等) 和計算 (Azure HDInsight 等) 可位於其他區域。


## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本節中，您會對儲存體帳戶和 SQL 資料庫建立連結服務。 

### <a name="create-a-storage-linked-service"></a>建立儲存體連結服務
1. 使用下列內容，在 C:\ADF 資料夾中建立名為 AzureStorageLinkedService.json 的 JSON 檔案。 (建立資料夾 ADF (如果尚未存在)。)儲存檔案之前，以您的儲存體帳戶名稱和金鑰取代 `<accountName>` 和 `<accountKey>`。

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": {
                    "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>",
                    "type": "SecureString"
                }
            }
        }
    }
    ```
2. 在 PowerShell 中，切換至 ADF 資料夾。

3. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務 AzureStorageLinkedService。 在下列範例中，您會傳遞 ResourceGroupName 和 DataFactoryName 參數的值： 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

### <a name="create-a-sql-database-linked-service"></a>建立 SQL Database 連結服務
1. 使用下列內容，在 C:\ADF 資料夾中建立名為 AzureSQLDatabaseLinkedService.json 的 JSON 檔案。 (建立資料夾 ADF (如果尚未存在)。)儲存檔案之前，以您的伺服器名稱、資料庫、使用者識別碼和密碼，取代 &lt;server&gt;、&lt;database&gt;、&lt;user id&gt; 和 &lt;password&gt;。 

    ```json
    {
        "name": "AzureSQLDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": {
                    "value": "Server = tcp:<server>.database.windows.net,1433;Initial Catalog=<database>; Persist Security Info=False; User ID=<user> ; Password=<password>; MultipleActiveResultSets = False; Encrypt = True; TrustServerCertificate = False; Connection Timeout = 30;",
                    "type": "SecureString"
                }
            }
        }
    }
    ```
2. 在 PowerShell 中，切換至 ADF 資料夾。

3. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務 AzureSQLDatabaseLinkedService。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ProvisioningState :
    ```

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集以代表來源和接收資料。 

### <a name="create-a-source-dataset"></a>建立來源資料集

1. 在相同的資料夾中，使用下列內容建立名為 SourceDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": "SourceDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "data_source_table"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
   
    ```
    在本教學課程中，您使用的資料表名稱是 data_source_table。 如果您使用不同名稱的資料表，請取代此名稱。

2. 執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 SourceDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-sink-dataset"></a>建立接收資料集

1. 在相同的資料夾中，使用下列內容建立名為 SinkDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/incrementalcopy",
                "fileName": "@CONCAT('Incremental-', pipeline().RunId, '.txt')", 
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }   
    ```

    > [!IMPORTANT]
    > 此程式碼片段假設您在 Blob 儲存體中有一個名為 adftutorial 的 Blob 容器。 建立容器 (若不存在)，或設為現有容器的名稱。 如果容器中沒有輸出資料夾 `incrementalcopy`，則會自動建立。 在本教學課程中，您會使用運算式 `@CONCAT('Incremental-', pipeline().RunId, '.txt')` 來動態產生檔案名稱。

2. 執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 SinkDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset    
    ```

## <a name="create-a-dataset-for-a-watermark"></a>建立水位線的資料集
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
2.  執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet 來建立資料集 WatermarkDataset。
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "WatermarkDataset" -File ".\WatermarkDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : WatermarkDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset    
    ```

## <a name="create-a-pipeline"></a>建立管線
在本教學課程中，您會建立具有兩個查閱活動、一個複製活動和一個 StoredProcedure 活動的管線，這些活動都在一個管線中鏈結。 


1. 在相同的資料夾中，使用下列內容建立 JSON 檔案 IncrementalCopyPipeline.json： 

    ```json
    {
        "name": "IncrementalCopyPipeline",
        "properties": {
            "activities": [
                {
                    "name": "LookupOldWaterMarkActivity",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                        "type": "SqlSource",
                        "sqlReaderQuery": "select * from watermarktable"
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
                            "sqlReaderQuery": "select MAX(LastModifytime) as NewWatermarkvalue from data_source_table"
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
                            "sqlReaderQuery": "select * from data_source_table where LastModifytime > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and LastModifytime <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    },
                    "dependsOn": [
                        {
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
    
                    "inputs": [
                        {
                            "referenceName": "SourceDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "SinkDataset",
                            "type": "DatasetReference"
                        }
                    ]
                },
    
                {
                    "name": "StoredProceduretoWriteWatermarkActivity",
                    "type": "SqlServerStoredProcedure",
                    "typeProperties": {
    
                        "storedProcedureName": "sp_write_watermark",
                        "storedProcedureParameters": {
                            "LastModifiedtime": {"value": "@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}", "type": "datetime" },
                            "TableName":  { "value":"@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}", "type":"String"}
                        }
                    },
    
                    "linkedServiceName": {
                        "referenceName": "AzureSQLDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
    
                    "dependsOn": [
                        {
                            "activity": "IncrementalCopyActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ]
                }
            ]
            
        }
    }
    ```
    

2. 執行 **Set-AzureRmDataFactoryV2Pipeline** Cmdlet 以建立管線 IncrementalCopyPipeline。
    
   ```powershell
   Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ``` 

   以下是範例輸出： 

   ```json
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Activities        : {LookupOldWaterMarkActivity, LookupNewWaterMarkActivity, IncrementalCopyActivity, StoredProceduretoWriteWatermarkActivity}
    Parameters        :
   ```
 
## <a name="run-the-pipeline"></a>執行管道

1. 使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管線 IncrementalCopyPipeline。 以您自己的資源群組和資料處理站名稱取代預留位置。

    ```powershell
    $RunId = Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroupName $resourceGroupName -dataFactoryName $dataFactoryName
    ``` 
2. 執行 **Get-AzureRmDataFactoryV2ActivityRun** Cmdlet 來檢查管線的狀態，直到您看到所有活動成功執行為止。 在 RunStartedAfter 和 RunStartedBefore 參數中，以您自己的適當時間取代預留位置。 在本教學課程中，您會使用 -RunStartedAfter "2017/09/14" 和 -RunStartedBefore "2017/09/15"。

    ```powershell
    Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $RunId -RunStartedAfter "<start time>" -RunStartedBefore "<end time>"
    ```

    以下是範例輸出：
 
    ```json
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupNewWaterMarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {NewWatermarkvalue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:42:42 AM
    ActivityRunEnd    : 9/14/2017 7:42:50 AM
    DurationInMs      : 7777
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupOldWaterMarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {TableName, WatermarkValue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:42:42 AM
    ActivityRunEnd    : 9/14/2017 7:43:07 AM
    DurationInMs      : 25437
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : IncrementalCopyActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, rowsCopied, copyDuration...}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:43:10 AM
    ActivityRunEnd    : 9/14/2017 7:43:29 AM
    DurationInMs      : 19769
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : StoredProceduretoWriteWatermarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {storedProcedureName, storedProcedureParameters}
    Output            : {}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:43:32 AM
    ActivityRunEnd    : 9/14/2017 7:43:47 AM
    DurationInMs      : 14467
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ```

## <a name="review-the-results"></a>檢閱結果

1. 在 Blob 儲存體 (接收存放區) 中，您會看到資料已複製到 SinkDataset 中定義的檔案。 在目前的教學課程中，檔案名稱是 `Incremental- d4bf3ce2-5d60-43f3-9318-923155f61037.txt`。 開啟檔案，您可以看到檔案中的記錄與 SQL 資料庫中的資料相同。

    ```
    1,aaaa,2017-09-01 00:56:00.0000000
    2,bbbb,2017-09-02 05:23:00.0000000
    3,cccc,2017-09-03 02:36:00.0000000
    4,dddd,2017-09-04 03:21:00.0000000
    5,eeee,2017-09-05 08:06:00.0000000
    ``` 
2. 檢查 `watermarktable` 中的最新值。 您會看到水位線值已更新。

    ```sql
    Select * from watermarktable
    ```
    
    以下是範例輸出：
 
    TableName | WatermarkValue
    --------- | --------------
    data_source_table   2017-09-05  8:06:00.000

### <a name="insert-data-into-the-data-source-store-to-verify-delta-data-loading"></a>將資料插入資料來源存放區以確認差異資料載入

1. 將新資料插入 SQL 資料庫 (資料來源存放區)。

    ```sql
    INSERT INTO data_source_table
    VALUES (6, 'newdata','9/6/2017 2:23:00 AM')
    
    INSERT INTO data_source_table
    VALUES (7, 'newdata','9/7/2017 9:01:00 AM')
    ``` 

    SQL 資料庫中更新的資料如下：

    ```
    PersonID | Name | LastModifytime
    -------- | ---- | --------------
    1 | aaaa | 2017-09-01 00:56:00.000
    2 | bbbb | 2017-09-02 05:23:00.000
    3 | cccc | 2017-09-03 02:36:00.000
    4 | dddd | 2017-09-04 03:21:00.000
    5 | eeee | 2017-09-05 08:06:00.000
    6 | newdata | 2017-09-06 02:23:00.000
    7 | newdata | 2017-09-07 09:01:00.000
    ```
2. 使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet，再次執行管線 IncrementalCopyPipeline。 以您自己的資源群組和資料處理站名稱取代預留位置。

    ```powershell
    $RunId = Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroupName $resourceGroupName -dataFactoryName $dataFactoryName
    ```
3. 執行 **Get-AzureRmDataFactoryV2ActivityRun** Cmdlet 來檢查管線的狀態，直到您看到所有活動成功執行為止。 在 RunStartedAfter 和 RunStartedBefore 參數中，以您自己的適當時間取代預留位置。 在本教學課程中，您會使用 -RunStartedAfter "2017/09/14" 和 -RunStartedBefore "2017/09/15"。

    ```powershell
    Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $RunId -RunStartedAfter "<start time>" -RunStartedBefore "<end time>"
    ```

    以下是範例輸出：
 
    ```json
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupNewWaterMarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {NewWatermarkvalue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:52:26 AM
    ActivityRunEnd    : 9/14/2017 8:52:58 AM
    DurationInMs      : 31758
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupOldWaterMarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {TableName, WatermarkValue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:52:26 AM
    ActivityRunEnd    : 9/14/2017 8:52:52 AM
    DurationInMs      : 25497
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : IncrementalCopyActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, rowsCopied, copyDuration...}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:53:00 AM
    ActivityRunEnd    : 9/14/2017 8:53:20 AM
    DurationInMs      : 20194
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : StoredProceduretoWriteWatermarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {storedProcedureName, storedProcedureParameters}
    Output            : {}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:53:23 AM
    ActivityRunEnd    : 9/14/2017 8:53:41 AM
    DurationInMs      : 18502
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ```
4. 在 blob 儲存體中，您會看到已建立另一個檔案。 在本教學課程中，新的檔案名稱是 `Incremental-2fc90ab8-d42c-4583-aa64-755dba9925d7.txt`。 開啟該檔案，您會在其中看到兩列記錄。

5. 檢查 `watermarktable` 中的最新值。 您會看到水位線值再次更新。

    ```sql
    Select * from watermarktable
    ```
    範例輸出： 
    
    TableName | WatermarkValue
    --------- | ---------------
    data_source_table | 2017-09-07 09:01:00.000

     
## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 準備資料存放區來儲存水位線值。 
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立管道。
> * 執行管道。
> * 監視管道執行。 

在本教學課程中，管線已從 SQL 資料庫中的單一資料表將資料複製到 Blob 儲存體。 請前進到下列教學課程，了解如何將資料從內部部署 SQL Server 資料庫中的多個資料表複製到 SQL 資料庫。 

> [!div class="nextstepaction"]
>[以累加方式將 SQL Server 中多個資料表的資料載入到 Azure SQL Database](tutorial-incremental-copy-multiple-tables-powershell.md)



