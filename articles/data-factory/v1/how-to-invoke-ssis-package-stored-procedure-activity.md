---
title: 使用 Azure Data Factory 叫用 SSIS 套件 - 預存程序活動 | Microsoft Docs
description: 本文描述如何使用預存程序活動從 Azure Data Factory 管線叫用 SQL Server Integration Services (SSIS) 封裝。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 01/19/2018
ms.author: jingwang
ms.openlocfilehash: fe2b509b62884c1cea554bc8dc5df25489205264
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966944"
---
# <a name="invoke-an-ssis-package-using-stored-procedure-activity-in-azure-data-factory"></a>在 Azure Data Factory 使用預存程序活動叫用 SSIS 套件
本文描述如何使用預存程序活動從 Azure Data Factory 管線叫用 SSIS 封裝。 

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[使用預存程序活動來叫用 SSIS 套件](../how-to-invoke-ssis-package-stored-procedure-activity.md)。

## <a name="prerequisites"></a>必要條件

### <a name="azure-sql-database"></a>連接字串 
這篇文章中的逐步解說會使用裝載 SSIS 目錄的 Azure SQL 資料庫。 您也可以使用 Azure SQL Database 受控執行個體。

### <a name="create-an-azure-ssis-integration-runtime"></a>建立 Azure-SSIS 整合執行階段
如果您未依照[教學課程：部署 SSIS 套件](../tutorial-create-azure-ssis-runtime-portal.md)中的逐步指示進行，請建立 Azure SSIS 整合執行階段。 您無法使用第 1 版的 Data Factory 來建立 Azure-SSIS 整合執行階段。 

## <a name="azure-portal"></a>Azure 入口網站
在此節中，您可以使用 Azure 入口網站以叫用 SSIS 套件的預存程序活動建立 Data Factory 管線。

### <a name="create-a-data-factory"></a>建立 Data Factory
第一步是使用 Azure 入口網站建立資料處理站。 

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。 
2. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，輸入 **ADFTutorialDataFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您在 [名稱] 欄位看到下列錯誤，請變更資料處理站的名稱 (例如 yournameADFTutorialBulkCopyDF)。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md)一文，以了解 Data Factory 成品的命名規則。

    `Data factory name ADFTutorialDataFactory is not available`
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
    若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../../azure-resource-manager/resource-group-overview.md)。  
4. 針對 [版本] 選取 [V1]。
5. 選取 Data Factory 的 [位置]  。 只有受到 Data Factory 支援的位置才會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他位置。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media//how-to-invoke-ssis-package-stored-procedure-activity/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
    ![Data Factory 首頁](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)
10. 按一下 [編寫及部署] 以啟動「Data Factory 編輯器」。

    ![Data Factory 編輯器](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-editor.png)

### <a name="create-an-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務
建立連結服務，將主控 SSIS 目錄的 Azure SQL 資料庫連結到資料處理站。 資料處理站使用此連結服務中的資訊連線到 SSISDB 資料庫，並執行預存程序來執行 SSIS 套件。 

1. 在 [Data Factory 編輯器] 中，按一下功能表上的 [新增資料存放區]，然後按一下 [Azure SQL Database]。 

    ![新增資料存放區 -> Azure SQL Database](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-sql-database-linked-service-menu.png)
2. 在右窗格中，執行下列步驟：

    1. 以 Azure SQL 伺服器的名稱取代 `<servername>`。 
    2. 以 **SSISDB** (SSIS 目錄資料庫的名稱) 取代 `<databasename>`。 
    3. 以具有 Azure SQL 伺服器存取權的使用者名稱取代 `<username@servername>`。 
    4. 以使用者的密碼取代 `<password>`。 
    5. 按一下工具列上的 [部署] 按鈕，部署已連結的服務。 

        ![Azure SQL Database 的連結服務](./media/how-to-invoke-ssis-package-stored-procedure-activity/azure-sql-database-linked-service-definition.png)

### <a name="create-a-dummy-dataset-for-output"></a>建立輸出的 Dummy 資料集
輸出資料集是 Dummy 資料集，可以驅動管線的排程。 請注意，頻率設為 [小時]，間隔設為 [1]。 因此，管線在管線開始與結束時間內會每小時執行一次。 

1. 在 [Data Factory 編輯器] 的左窗格中，按一下 [...更多]  ->  [新增資料集]  ->  [Azure SQL]。

    ![更多 -> 新增資料集](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-dataset-menu.png)
2. 將下列 JSON 程式碼片段複製到右窗格中的 JSON 編輯器。 
    
    ```json
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": { },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
3. 按一下工具列上的 [部署]。 這個動作會將資料集部署至 Azure Data Factory 服務。 

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>建立具有預存程序活動的管線 
在此步驟中，您會建立具有預存程序活動的管線。 活動會叫用 sp_executesql 預存程序以執行 SSIS 套件。 

1. 在左側窗格中按一下 **[...更多]** 和 [新增管線]。
2. 將下列 JSON 程式碼片段複製到 JSON 編輯器： 

    > [!IMPORTANT]
    > 在儲存檔案之前，以 SSIS 目錄中資料夾、專案和套件的名稱取代 &lt;資料夾名稱&gt;、&lt;專案名稱&gt;、&lt;套件名稱&gt;。

    ```json
    {
        "name": "MyPipeline",
        "properties": {
            "activities": [{
                "name": "SprocActivitySample",
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_executesql",
                    "storedProcedureParameters": {
                        "stmt": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<folder name>', @project_name=N'<project name>', @package_name=N'<package name>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                    }
                },
                "outputs": [{
                    "name": "sprocsampleout"
                }],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }],
            "start": "2018-01-19T00:00:00Z",
            "end": "2018-01-19T05:00:00Z",
            "isPaused": false
        }
    }    
    ```
3. 按一下工具列上的 [部署]。 這個動作會將管線部署至 Azure Data Factory 服務。 

### <a name="monitor-the-pipeline-run"></a>監視管道執行
輸出資料集上的排程定義為每小時。 管線結束時間是開始時間之後五小時。 因此，您會看到五個管線執行。 

1. 關閉編輯器視窗，您會看到資料處理站的首頁。 按一下 [監視及管理] 圖格。 

    ![[圖表] 圖格](./media/how-to-invoke-ssis-package-stored-procedure-activity/monitor-manage-tile.png)
2. 將 [開始時間] 和 [結束時間] 更新為 **01/18/2018 08:30 AM** 和 **01/20/2018 08:30 AM**，然後按一下 [套用]。 您應該會看到與管線執行相關聯的**活動時段**。 

    ![活動時段](./media/how-to-invoke-ssis-package-stored-procedure-activity/activity-windows.png)

如需監視管線的詳細資訊，請參閱[使用監視及管理應用程式來監視及管理 Azure Data Factory 管線](data-factory-monitor-manage-app.md)。

## <a name="azure-powershell"></a>Azure PowerShell
在本節中，您可以使用 Azure PowerShell 以叫用 SSIS 封裝的預存程序活動建立資料處理站管線。

依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

### <a name="create-a-data-factory"></a>建立 Data Factory
下列程序提供建立資料處理站的步驟。 您會在此資料處理站中建立具有預存程序活動的管線。 預存程序活動會執行 SSISDB 資料庫中的預存程序來執行 SSIS 套件。

1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如：`"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令
2. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    $ResGrp = New-AzureRmResourceGroup $resourceGroupName -location 'eastus'
    ``` 
    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令。 
3. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  將資料處理站名稱更新為全域唯一的。 

    ```powershell
    $DataFactoryName = "ADFTutorialFactory";
    ```

5. 若要建立資料處理站，請從 $ResGrp 變數使用 Location 和 ResourceGroupName 屬性來執行下列 **New-AzureRmDataFactory** Cmdlet： 
    
    ```powershell       
    $df = New-AzureRmDataFactory -ResourceGroupName $ResourceGroupName -Name $dataFactoryName -Location "East US"
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果發生下列錯誤，請變更名稱，並再試一次。

    ```
    The specified Data Factory name 'ADFTutorialFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為**參與者**或**擁有者**角色，或是 Azure 訂用帳戶的**管理員**。

### <a name="create-an-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務
建立連結服務，將主控 SSIS 目錄的 Azure SQL 資料庫連結到資料處理站。 資料處理站使用此連結服務中的資訊連線到 SSISDB 資料庫，並執行預存程序來執行 SSIS 套件。 

1. 使用下列內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **AzureSqlDatabaseLinkedService.json** 的 JSON 檔案： 

    > [!IMPORTANT]
    > 儲存檔案之前，以您的 Azure SQL Database 的值取代 &lt;servername&gt;、&lt;username&gt;@&lt;servername&gt; 和 &lt;password&gt;。

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
        }
    ```
2. 在 **Azure PowerShell** 中，切換至 **C:\ADF\RunSSISPackage** 資料夾。
3. 執行 **New-AzureRmDataFactoryLinkedService** Cmdlet 以建立連結服務：**AzureSqlDatabaseLinkedService**。 

    ```powershell
    New-AzureRmDataFactoryLinkedService $df -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-an-output-dataset"></a>建立輸出資料表
輸出資料集是 Dummy 資料集，可以驅動管線的排程。 請注意，頻率設為 [小時]，間隔設為 [1]。 因此，管線在管線開始與結束時間內會每小時執行一次。 

1. 使用下列內容建立 OuputDataset.json 檔案： 
    
    ```json
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": { },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
2. 執行 **New-AzureRmDataFactoryDataset** Cmdlet 以建立資料集。 

    ```powershell
    New-AzureRmDataFactoryDataset $df -File ".\OutputDataset.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>建立具有預存程序活動的管線 
在此步驟中，您會建立具有預存程序活動的管線。 活動會叫用 sp_executesql 預存程序以執行 SSIS 套件。 

1. 使用下列內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **MyPipeline.json** 的 JSON 檔案：

    > [!IMPORTANT]
    > 在儲存檔案之前，以 SSIS 目錄中資料夾、專案和套件的名稱取代 &lt;資料夾名稱&gt;、&lt;專案名稱&gt;、&lt;套件名稱&gt;。

    ```json
    {
        "name": "MyPipeline",
        "properties": {
            "activities": [{
                "name": "SprocActivitySample",
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_executesql",
                    "storedProcedureParameters": {
                        "stmt": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<folder name>', @project_name=N'<project name>', @package_name=N'<package name>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                    }
                },
                "outputs": [{
                    "name": "sprocsampleout"
                }],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }],
            "start": "2017-10-01T00:00:00Z",
            "end": "2017-10-01T05:00:00Z",
            "isPaused": false
        }
    }    
    ```

2. 若要建立管線：**RunSSISPackagePipeline**，請執行 **New-AzureRmDataFactoryPipeline** Cmdlet。

    ```powershell
    $DFPipeLine = New-AzureRmDataFactoryPipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

### <a name="monitor-the-pipeline-run"></a>監視管道執行

2. 執行 **Get-AzureRmDataFactorySlice**，以取得輸出資料集** (管線的輸出資料表) 所有配量的詳細資料。

    ```PowerShell
    Get-AzureRmDataFactorySlice $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```
    請注意，您在此處指定的 StartDateTime 與在管線 JSON 中指定的開始時間是相同的。 
3. 執行 **Get-AzureRmDataFactoryRun** ，以取得特定配量的活動執行詳細資料。

    ```PowerShell
    Get-AzureRmDataFactoryRun $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```

    您可以繼續執行此 Cmdlet 直到您看到配量處於**就緒**狀態或**失敗**狀態。 

    您可以依據 Azure SQL Server 中的 SSISDB 資料庫執行下列查詢，確認已執行套件。 

    ```sql
    select * from catalog.executions
    ```

## <a name="next-steps"></a>後續步驟
如需預存程序活動的詳細資訊，請參閱[預存程序活動](data-factory-stored-proc-activity.md)一文。

