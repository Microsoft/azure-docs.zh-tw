---
title: 使用變更追蹤和 Azure Data Factory 以累加方式複製資料 | Microsoft Docs
description: '在本教學課程中，您會建立 Azure Data Factory 管線，透過累加方式將差異資料從內部部署 SQL Server 資料庫中的多個資料表，複製到 Azure SQL Database。 '
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
ms.openlocfilehash: 246b423e69fa8fb73db45f44fa17c1bc65407681
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43090720"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-change-tracking-information"></a>使用變更追蹤資訊，以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體 
在本教學課程中，您會建立一個 Azure Data Factory 並讓其具有管線，以根據來源 Azure SQL Database 中的**變更追蹤**資訊，將差異資料載入到 Azure Blob 儲存體。  

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備來源資料存放區
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收和變更追蹤資料集。
> * 建立、執行和監視完整複製管線
> * 新增或更新來源資料表中的資料
> * 建立、執行和監視累加複製管線

## <a name="overview"></a>概觀
在資料整合解決方案中，我們經常會見到在初次載入資料之後，再以累加方式載入資料的情形。 在某些情況下，您的來源資料存放區於某一期間內所變更的資料，可以輕鬆地到加以切割 (例如，LastModifyTime、CreationTime)。 但也有某些情況是，您並無法明確地識別自上次處理資料後所產生的差異資料。 Azure SQL Database 和 SQL Server 等資料存放區所支援的變更追蹤技術，可供您用來識別差異資料。  本教學課程說明如何搭配使用 Azure Data Factory 與 SQL 變更追蹤技術，以透過累加方式從 Azure SQL Database 將差異資料載入至 Azure Blob 儲存體。  如需更為具體的 SQL 變更追蹤技術資訊，請參閱 [SQL Server 中的變更追蹤](/sql/relational-databases/track-changes/about-change-tracking-sql-server)。 

## <a name="end-to-end-workflow"></a>端對端工作流程
以下是使用變更追蹤技術，以累加方式載入資料的典型端對端工作流程步驟。

> [!NOTE]
> Azure SQL Database 和 SQL Server 都支援變更追蹤技術。 本教學課程會使用 Azure SQL Database 作為來源資料存放區。 但您也可以使用內部部署的 SQL Server。 

1. **初始載入歷史資料** (執行一次)：
    1. 在來源 Azure SQL Database 中啟用變更追蹤技術。
    2. 取得 Azure SQL Database 中的 SYS_CHANGE_VERSION 初始值作為基準，以擷取變更的資料。
    3. 將完整資料從 Azure SQL Database 載入到 Azure Blob 儲存體。 
2. **依排程累加載入差異資料** (在初始載入資料後定期執行)：
    1. 取得 SYS_CHANGE_VERSION 的舊值和新值。
    3. 藉由將 **sys.change_tracking_tables** 中有所變更資料列 (介於兩個 SYS_CHANGE_VERSION 值之間) 的主索引鍵，聯結到**來源資料表**中的資料來載入差異資料，然後將差異資料移動到目的地。
    4. 針對要在下一次載入的差異更新其 SYS_CHANGE_VERSION。

## <a name="high-level-solution"></a>高階解決方案
在本教學課程中，您會建立兩個管線以執行下列兩項作業：  

1. **初始載入：** 您會使用複製活動建立管線，以將整個資料從來源資料存放區 (Azure SQL Database) 複製到目的地資料存放區 (Azure Blob 儲存體)。

    ![完整載入資料](media/tutorial-incremental-copy-change-tracking-feature-powershell/full-load-flow-diagram.png)
1.  **累加載入：** 您會使用下列活動建立管線，然後定期執行該管線。 
    1. 建立**兩個查閱活動**，從 Azure SQL Database 取得舊的和新的 SYS_CHANGE_VERSION，並將其傳遞給複製活動。
    2. 建立**一個複製活動**，將兩個 SYS_CHANGE_VERSION 值之間所插入/更新/刪除的資料，從 Azure SQL Database 複製到 Azure Blob 儲存體。
    3. 建立**一個預存程序活動**，以更新下一次管線執行的 SYS_CHANGE_VERSION 值。

    ![累加載入流程圖](media/tutorial-incremental-copy-change-tracking-feature-powershell/incremental-load-flow-diagram.png)


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* Azure PowerShell。 依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。
* **Azure SQL Database**。 您需要使用資料庫作為**來源**資料存放區。 如果您沒有 Azure SQL Database，請參閱[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)一文，按照步驟建立資料庫。
* **Azure 儲存體帳戶**。 您需要使用 Blob 儲存體作為**接收**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。 建立名為 **adftutorial** 的容器。 

### <a name="create-a-data-source-table-in-your-azure-sql-database"></a>在 Azure SQL 資料庫中建立資料來源資料表
1. 啟動 **SQL Server Management Studio**，然後連線到 Azure SQL Server。 
2. 在**伺服器總管**中，以滑鼠右鍵按一下您的**資料庫**，然後選擇 [新增查詢]。
3. 對 Azure SQL 資料庫執行下列 SQL 命令，以建立名為 `data_source_table` 的資料表作為資料來源存放區。  
    
    ```sql
    create table data_source_table
    (
        PersonID int NOT NULL,
        Name varchar(255),
        Age int
        PRIMARY KEY (PersonID)
    );

    INSERT INTO data_source_table
        (PersonID, Name, Age)
    VALUES
        (1, 'aaaa', 21),
        (2, 'bbbb', 24),
        (3, 'cccc', 20),
        (4, 'dddd', 26),
        (5, 'eeee', 22);

    ```
4. 執行下列 SQL 查詢，在您的資料庫和來源資料表 (data_source_table) 啟用**變更追蹤**機制： 

    > [!NOTE]
    > - 將 &lt;your database name&gt; 替換為具有 data_source_table 的 Azure SQL Database 名稱。 
    > - 在目前的範例中，有變更的資料會保留兩天。 如果您每隔三天以上才會載入變更的資料，則裡面可能不會包含某些已變更的資料。  您必須將 CHANGE_RETENTION 的值變更為更大的數字。 或者，請確保您用來載入已變更資料的期間是在兩天內。 如需詳細資訊，請參閱[啟用資料庫的變更追蹤](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server#enable-change-tracking-for-a-database)
 
    ```sql
    ALTER DATABASE <your database name>
    SET CHANGE_TRACKING = ON  
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  
  
    ALTER TABLE data_source_table
    ENABLE CHANGE_TRACKING  
    WITH (TRACK_COLUMNS_UPDATED = ON)
    ```
5. 執行下列查詢，建立新資料表並使用預設值儲存 ChangeTracking_version： 

    ```sql
    create table table_store_ChangeTracking_version
    (
        TableName varchar(255),
        SYS_CHANGE_VERSION BIGINT,
    );

    DECLARE @ChangeTracking_version BIGINT
    SET @ChangeTracking_version = CHANGE_TRACKING_CURRENT_VERSION();  

    INSERT INTO table_store_ChangeTracking_version
    VALUES ('data_source_table', @ChangeTracking_version)
    ```
    
    > [!NOTE]
    > 如果在您啟用 SQL Database 的變更追蹤後，資料並未變更，變更追蹤版本的值會是 0。
6. 執行下列查詢，在您的 Azure SQL Database 中建立預存程序。 該管線會叫用此預存程序，以更新您在上一個步驟建立的資料表變更追蹤版本。 

    ```sql
    CREATE PROCEDURE Update_ChangeTracking_Version @CurrentTrackingVersion BIGINT, @TableName varchar(50)
    AS
    
    BEGIN
    
        UPDATE table_store_ChangeTracking_version
        SET [SYS_CHANGE_VERSION] = @CurrentTrackingVersion
    WHERE [TableName] = @TableName
    
    END    
    ```

### <a name="azure-powershell"></a>Azure PowerShell
依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

## <a name="create-a-data-factory"></a>建立 Data Factory
1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如：`"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$resourceGroupName` 變數，然後執行一次命令
2. 定義 Data Factory 位置的變數： 

    ```powershell
    $location = "East US"
    ```
3. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    New-AzureRmResourceGroup $resourceGroupName $location
    ``` 
    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$resourceGroupName` 變數，然後執行一次命令。 
3. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  將資料處理站名稱更新為全域唯一的。  

    ```powershell
    $dataFactoryName = "IncCopyChgTrackingDF";
    ```
5. 若要建立 Data Factory，請執行下列 **Set-AzureRmDataFactoryV2** Cmdlet： 
    
    ```powershell       
    Set-AzureRmDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName 
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果發生下列錯誤，請變更名稱，並再試一次。

    ```
    The specified Data Factory name 'ADFIncCopyChangeTrackingTestFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為**參與者**或**擁有者**角色，或是 Azure 訂用帳戶的**管理員**。
* 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。


## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本節中，您會對 Azure 儲存體帳戶和 Azure SQL 資料庫建立連結服務。 

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務。
在此步驟中，您會將您的 Azure 儲存體帳戶連結到 Data Factory。

1. 在 **C:\ADFTutorials\IncCopyChangeTrackingTutorial** 資料夾中，使用下列內容建立名為 **AzureStorageLinkedService.json** 的 JSON 檔案：(如果資料夾尚不存在，請加以建立)。 儲存檔案之前，以您的 Azure 儲存體帳戶名稱和金鑰取代 `<accountName>`、`<accountKey>`。

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
2. 在 **Azure PowerShell** 中，切換至 **C:\ADFTutorials\IncCopyChgTrackingTutorial** 資料夾。
3. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務：**AzureStorageLinkedService**。 在下列範例中，您會傳遞 **ResourceGroupName** 和 **DataFactoryName** 參數的值。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

### <a name="create-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務。
在此步驟中，您會將您的 Azure SQL Database 連結到您的 Data Factory。

1. 在 **C:\ADFTutorials\IncCopyChangeTrackingTutorial** 資料夾中，使用下列內容建立名為 **AzureSQLDatabaseLinkedService.json** 的 JSON 檔案：將 **&lt;server&gt;、&lt;database name&gt;、&lt;user id&gt; 和 &lt;password&gt;** 替換為 Azure SQL Server 名稱、資料庫名稱、使用者識別碼和密碼，再儲存檔案。 

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
2. 在 **Azure PowerShell** 中，執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務：**AzureSQLDatabaseLinkedService**。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    以下是範例輸出：

    ```json
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集以代表資料來源和資料目的地。 以及用來儲存 SYS_CHANGE_VERSION 的位置。

### <a name="create-a-source-dataset"></a>建立來源資料集
在此步驟中，您會建立資料集來代表來源資料。 

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

2.  執行 Set-AzureRmDataFactoryV2Dataset Cmdlet 來建立資料集：SourceDataset
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-sink-dataset"></a>建立接收資料集
在此步驟中，您會建立資料集來代表從來源資料存放區複製的資料。 

1. 在相同的資料夾中，使用下列內容建立名為 SinkDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/incchgtracking",
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

    您會在 Azure Blob 儲存體中建立 adftutorial 容器，以滿足其中一項必要條件。 建立容器 (若不存在)，或設為現有容器的名稱。 在本教學課程中，您會使用下列運算式來動態產生輸出檔案名稱：@CONCAT('Incremental-', pipeline().RunId, '.txt')。
2.  執行 Set-AzureRmDataFactoryV2Dataset Cmdlet 來建立資料集：SinkDataset
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

### <a name="create-a-change-tracking-dataset"></a>建立變更追蹤資料集
在此步驟中，您會建立資料集來儲存變更追蹤版本。  

1. 在相同的資料夾中，使用下列內容建立名為 ChangeTrackingDataset.json 的 JSON 檔案： 

    ```json
    {
        "name": " ChangeTrackingDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "table_store_ChangeTracking_version"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
    ```

    您必須建立 table_store_ChangeTracking_version 資料表以滿足其中一項必要條件。
2.  執行 Set-AzureRmDataFactoryV2Dataset Cmdlet 來建立資料集：WatermarkDataset
    
    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "ChangeTrackingDataset" -File ".\ChangeTrackingDataset.json"
    ```

    以下是此 Cmdlet 的範例輸出：
    
    ```json
    DatasetName       : ChangeTrackingDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

## <a name="create-a-pipeline-for-the-full-copy"></a>建立完整複本的管線
在此步驟中，您會使用複製活動建立管線，將整個資料從來源資料存放區 (Azure SQL Database) 複製到目的地資料存放區 (Azure Blob 儲存體)。

1. 在相同的資料夾中，使用下列內容建立 JSON 檔案 FullCopyPipeline.json： 

    ```json
    {
        "name": "FullCopyPipeline",
        "properties": {
            "activities": [{
                "name": "FullCopyActivity",
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "SqlSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
    
                "inputs": [{
                    "referenceName": "SourceDataset",
                    "type": "DatasetReference"
                }],
                "outputs": [{
                    "referenceName": "SinkDataset",
                    "type": "DatasetReference"
                }]
            }]
        }
    }
    ```
2. 執行 Set-AzureRmDataFactoryV2Pipeline Cmdlet 以建立管線：FullCopyPipeline。
    
   ```powershell
    Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "FullCopyPipeline" -File ".\FullCopyPipeline.json"
   ``` 

   以下是範例輸出： 

   ```json
    PipelineName      : FullCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {FullCopyActivity}
    Parameters        :
   ```
 
### <a name="run-the-full-copy-pipeline"></a>執行完整的複製管線
使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管線：**FullCopyPipeline**。 

```powershell
Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "FullCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName        
``` 

### <a name="monitor-the-full-copy-pipeline"></a>監視完整的複製管線

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [所有服務]，以 `data factories` 關鍵字進行搜尋，然後選取 [資料處理站]。 

    ![Data Factory 功能表](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-data-factories-menu-1.png)
3. 在 Data Factory 清單中搜尋**您的 Data Factory**，然後加以選取以啟動 [Data Factory] 頁面。 

    ![搜尋您的 Data Factory](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-search-data-factory-2.png)
4. 在資料管理站頁面上，按一下 [監視及管理] 圖格。 

    ![監視及管理圖格](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-monitor-manage-tile-3.png)    
5. **資料整合應用程式**會在不同的索引標籤中啟動。您可以看到所有**管線執行**及其狀態。 請注意，在下列範例中，管線執行狀態是 [成功]。 按一下 [參數] 資料行中的連結，即可檢查傳遞到管線的參數。 如果有錯誤，您就會在 [錯誤] 資料行中看到連結。 按一下 [動作] 資料行中的連結。 

    ![管線執行](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-pipeline-runs-4.png)    
6. 當您按一下 [動作] 資料行中的連結時，您會看到下列頁面，其中顯示管線的所有**活動執行**。 

    ![活動執行](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-activity-runs-5.png)
7. 若要切換回 [管線執行] 檢視，請按一下 [管線]，如下圖所示。 


### <a name="review-the-results"></a>檢閱結果
您會在 `adftutorial` 容器的 `incchgtracking` 資料夾中看到名為 `incremental-<GUID>.txt` 的檔案。 

![完整複製的輸出檔案](media\tutorial-incremental-copy-change-tracking-feature-powershell\full-copy-output-file.png)

此檔案應該會有 Azure SQL Database 中的資料：

```
1,aaaa,21
2,bbbb,24
3,cccc,20
4,dddd,26
5,eeee,22
```

## <a name="add-more-data-to-the-source-table"></a>新增更多資料至來源資料表

對 Azure SQL Database 執行下列查詢，以新增資料列並加以更新。 

```sql
INSERT INTO data_source_table
(PersonID, Name, Age)
VALUES
(6, 'new','50');


UPDATE data_source_table
SET [Age] = '10', [name]='update' where [PersonID] = 1

``` 

## <a name="create-a-pipeline-for-the-delta-copy"></a>建立差異複本的管線
在此步驟中，您會建立具有下列活動的管線，並定期執行此管線。 **查閱活動**會從 Azure SQL Database 取得舊的和新的 SYS_CHANGE_VERSION，並將它傳遞給複製活動。 **複製活動**會將兩個 SYS_CHANGE_VERSION 值之間所插入/更新/刪除的資料，從 Azure SQL Database 複製到 Azure Blob 儲存體。 **預存程序活動**會更新下一次管線執行的 SYS_CHANGE_VERSION 值。

1. 在相同的資料夾中，使用下列內容建立 JSON 檔案 IncrementalCopyPipeline.json： 

    ```json
    {
            "name": "IncrementalCopyPipeline",
            "properties": {
                "activities": [
                {
                        "name": "LookupLastChangeTrackingVersionActivity",
                        "type": "Lookup",
                        "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select * from table_store_ChangeTracking_version"
                            },
        
                            "dataset": {
                            "referenceName": "ChangeTrackingDataset",
                            "type": "DatasetReference"
                            }
                        }
                    },
                    {
                        "name": "LookupCurrentChangeTrackingVersionActivity",
                        "type": "Lookup",
                        "typeProperties": {
                            "source": {
                                "type": "SqlSource",
                                "sqlReaderQuery": "SELECT CHANGE_TRACKING_CURRENT_VERSION() as CurrentChangeTrackingVersion"
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
                                "sqlReaderQuery": "select data_source_table.PersonID,data_source_table.Name,data_source_table.Age, CT.SYS_CHANGE_VERSION, SYS_CHANGE_OPERATION from data_source_table RIGHT OUTER JOIN CHANGETABLE(CHANGES data_source_table, @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.SYS_CHANGE_VERSION}) as CT on data_source_table.PersonID = CT.PersonID where CT.SYS_CHANGE_VERSION <= @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}"
                            },
                            "sink": {
                                "type": "BlobSink"
                            }
                        },
                        "dependsOn": [
                            {
                                "activity": "LookupLastChangeTrackingVersionActivity",
                                "dependencyConditions": [
                                    "Succeeded"
                                ]
                            },
                            {
                                "activity": "LookupCurrentChangeTrackingVersionActivity",
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
                        "name": "StoredProceduretoUpdateChangeTrackingActivity",
                        "type": "SqlServerStoredProcedure",
                        "typeProperties": {
    
                            "storedProcedureName": "Update_ChangeTracking_Version",
                            "storedProcedureParameters": {
                            "CurrentTrackingVersion": {"value": "@{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}", "type": "INT64" },
                                "TableName":  { "value":"@{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.TableName}", "type":"String"}
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
2. 執行 Set-AzureRmDataFactoryV2Pipeline Cmdlet 以建立管線：FullCopyPipeline。
    
   ```powershell
    Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ``` 

   以下是範例輸出： 

   ```json
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : IncCopyChgTrackingDF
    Activities        : {LookupLastChangeTrackingVersionActivity, LookupCurrentChangeTrackingVersionActivity, IncrementalCopyActivity, StoredProceduretoUpdateChangeTrackingActivity}
    Parameters        :
   ```

### <a name="run-the-incremental-copy-pipeline"></a>執行累加複製管線
使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管道：**IncrementalCopyPipeline**。 

```powershell
Invoke-AzureRmDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroup $resourceGroupName -dataFactoryName $dataFactoryName     
``` 


### <a name="monitor-the-incremental-copy-pipeline"></a>監視累加複製管線
1. 在**資料整合應用程式**中，重新整理 [管線執行] 檢視。 確認您有在清單中看到 IncrementalCopyPipeline。 按一下 [動作] 資料行中的連結。  

    ![管線執行](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-pipeline-runs-6.png)    
2. 當您按一下 [動作] 資料行中的連結時，您會看到下列頁面，其中顯示管線的所有**活動執行**。 

    ![活動執行](media\tutorial-incremental-copy-change-tracking-feature-powershell\monitor-activity-runs-7.png)
3. 若要切換回 [管線執行] 檢視，請按一下 [管線]，如下圖所示。 

### <a name="review-the-results"></a>檢閱結果
您會在 `adftutorial` 容器的 `incchgtracking` 資料夾中看到第二個檔案。 

![累加複製的輸出檔案](media\tutorial-incremental-copy-change-tracking-feature-powershell\incremental-copy-output-file.png)

此檔案應該只會有 Azure SQL Database 中的差異資料。 具有 `U` 的記錄是資料庫中更新的資料列，具有 `I` 的記錄則是一個新增的資料列。 

```
1,update,10,2,U
6,new,50,1,I
```
前三個資料行是 data_source_table 中已變更的資料。 最後兩個資料行是變更追蹤系統資料表中的中繼資料。 第四個資料行是每個已變更資料列的 SYS_CHANGE_VERSION。 第五個資料行是作業：U = 更新、I = 插入。  如需變更追蹤資訊的詳細資料，請參閱 [CHANGETABLE](/sql/relational-databases/system-functions/changetable-transact-sql)。 

```
==================================================================
PersonID Name    Age    SYS_CHANGE_VERSION    SYS_CHANGE_OPERATION
==================================================================
1        update  10     2                     U
6        new     50     1                     I
```

    
## <a name="next-steps"></a>後續步驟
進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[在雲端中使用 Spark 叢集轉換資料](tutorial-transform-data-spark-powershell.md)



