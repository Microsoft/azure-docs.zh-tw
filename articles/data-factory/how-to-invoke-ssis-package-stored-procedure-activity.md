---
title: 使用預存程序活動執行 SSIS 套件 - Azure | Microsoft Docs
description: 本文描述如何使用預存程序活動，在 Azure Data Factory 管線執行 SQL Server Integration Services (SSIS) 套件。
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
ms.date: 04/17/2018
ms.author: jingwang
ms.openlocfilehash: 98a833667aa4073e05b94a62a3e3aea4355e8fb0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958958"
---
# <a name="run-an-ssis-package-with-the-stored-procedure-activity-in-azure-data-factory"></a>在 Azure Data Factory 中使用預存程序活動執行 SSIS 套件
本文描述如何使用預存程序活動，在 Azure Data Factory 管線執行 SSIS 套件。 

## <a name="prerequisites"></a>必要條件

### <a name="azure-sql-database"></a>連接字串 
這篇文章中的逐步解說會使用裝載 SSIS 目錄的 Azure SQL 資料庫。 您也可以使用 Azure SQL Database 受控執行個體。

## <a name="create-an-azure-ssis-integration-runtime"></a>建立 Azure-SSIS 整合執行階段
如果您未依照[教學課程：部署 SSIS 套件](tutorial-create-azure-ssis-runtime-portal.md)中的逐步指示進行，請建立 Azure SSIS 整合執行階段。

## <a name="data-factory-ui-azure-portal"></a>資料處理站使用者介面 (Azure 入口網站)
在本節中，您可以使用資料處理站 UI 以叫用 SSIS 封裝的預存程序活動建立資料處理站管線。

### <a name="create-a-data-factory"></a>建立 Data Factory
第一步是使用 Azure 入口網站建立資料處理站。 

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
2. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。 
3. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，輸入 **ADFTutorialDataFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您在 [名稱] 欄位看到下列錯誤，請變更資料處理站的名稱 (例如 yournameADFTutorialBulkCopyDF)。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
     ![名稱無法使用 - 錯誤](./media/how-to-invoke-ssis-package-stored-procedure-activity/name-not-available-error.png)
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
    若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
4. 針對 [版本] 選取 [V2]。
5. 選取 Data Factory 的 [位置]  。 只有受到 Data Factory 支援的位置才會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他位置。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media//how-to-invoke-ssis-package-stored-procedure-activity/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
    ![Data Factory 首頁](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)
10. 按一下 [編寫與監視] 圖格，以在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI) 應用程式。 

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>使用預存程序活動建立管線
在此步驟中，您可以使用資料處理站 UI 建立管線。 您將預存程序活動新增至管線，並設定它使用 sp_executesql 預存程序執行 SSIS 封裝。 

1. 在 [開始使用] 頁面中，按一下 [建立管線]： 

    ![開始使用頁面](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)
2. 在 [活動] 工具箱中，展開 [一般]，並將 [預存程序] 活動拖放至管線設計工具表面。 

    ![拖放功能預存程序活動](./media/how-to-invoke-ssis-package-stored-procedure-activity/drag-drop-sproc-activity.png)
3. 在第二個程序活動的 [屬性] 視窗中，切換到 [SQL 帳戶] 索引標籤，並按一下 [+ 新增]。 您可對於主控 SSIS 目錄 (SSIDB 資料庫) 的 Azure SQL 資料庫建立連線。 
   
    ![新增連結服務按鈕](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-linked-service-button.png)
4. 在 [新增連結服務] 視窗中，執行下列步驟： 

    1. 選取 [類型] 的 [Azure SQL Database]。
    2. 選取**預設** Azure Integration Runtime，以連線到裝載 `SSISDB` 資料庫的 Azure SQL Database。
    3. 針對 [伺服器名稱] 欄位，選取裝載 SSISDB 資料庫的 Azure SQL Database。
    4. 選取 [資料庫名稱] 的 [SSISDB]。
    5. 對於 [使用者名稱]，輸入可存取資料庫的使用者名稱。
    6. 對於 [密碼]，輸入使用者的密碼。 
    7. 按一下 [測試連接] 按鈕以測試資料庫連接。
    8. 按一下 [儲存] 按鈕以儲存連結服務。 

        ![Azure SQL Database 的連結服務](./media/how-to-invoke-ssis-package-stored-procedure-activity/azure-sql-database-linked-service-settings.png)
5. 在 [屬性] 視窗中，從 [SQL 帳戶] 索引標籤切換到 [預存程序] 索引標籤，並執行下列步驟： 

    1. 選取 [編輯]。 
    2. 對於 [預存程序名稱]，輸入 `sp_executesql`。 
    3. 按一下 [預存程序參數] 區段中的 [+ 新增]。 
    4. 對於參數的 [名稱]，輸入 **stmt**。 
    5. 針對參數的 [類型]，輸入 [字串]。 
    6. 針對參數的 [值]，輸入下列 SQL 查詢：

        在 SQL 查詢中，指定 **folder_name**、**project_name** 和 **package_name** 參數的正確值。 

        ```sql
        DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER name in SSIS Catalog>', @project_name=N'<PROJECT name in SSIS Catalog>', @package_name=N'<PACKAGE name>.dtsx', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END
        ```

        ![Azure SQL Database 的連結服務](./media/how-to-invoke-ssis-package-stored-procedure-activity/stored-procedure-settings.png)
6. 若要驗證管線設定，按一下工具列上的 [驗證]。 若要關閉 [管線驗證報告]，按一下 **>>**。

    ![驗證管線](./media/how-to-invoke-ssis-package-stored-procedure-activity/validate-pipeline.png)
7. 按一下 [全部發行] 按鈕，將管線發行至資料處理站。 

    ![發佈](./media/how-to-invoke-ssis-package-stored-procedure-activity/publish-all-button.png)    

### <a name="run-and-monitor-the-pipeline"></a>執行並監視管線
在本節中，您會觸發管線執行，然後監視執行的情況。 

1. 若要觸發管線執行，按一下工具列上的 [觸發程序]，然後按一下 [立即觸發]。 

    ![立即觸發](media/how-to-invoke-ssis-package-stored-procedure-activity/trigger-now.png)

2. 在 [管線執行] 視窗中，選取 [完成]。 
3. 切換至左側的 [監視] 索引標籤。 您會看到管線執行、其狀態，以及其他資訊 (例如執行開始時間)。 若要重新整理檢視，按一下 [重新整理]。

    ![管線執行](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

3. 按一下 [動作]資料行中的 [檢視活動執行] 連結。 管線只有一個活動 (預存程序活動) 時，您只會看到一個活動執行。

    ![活動執行](./media/how-to-invoke-ssis-package-stored-procedure-activity/activity-runs.png)

4. 您可以依據 Azure SQL Server 中的 SSISDB 資料庫執行下列**查詢**，來確認套件已執行。 

    ```sql
    select * from catalog.executions
    ```

    ![確認封裝執行](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)


> [!NOTE]
> 您也可以建立管線的排程觸發程序，以便管線依排程執行 (每小時、每日等等)。 如需範例，請參閱[建立資料處理站 - 資料處理站 UI](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule)。

## <a name="azure-powershell"></a>Azure PowerShell
在本節中，您可以使用 Azure PowerShell 以叫用 SSIS 封裝的預存程序活動建立資料處理站管線。 

依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。 

### <a name="create-a-data-factory"></a>建立 Data Factory
您可以使用具有 Azure SSIS IR 的同一個資料處理站，也可以建立另一個資料處理站。 下列程序提供建立資料處理站的步驟。 您會在此資料處理站中建立具有預存程序活動的管線。 預存程序活動會執行 SSISDB 資料庫中的預存程序來執行 SSIS 套件。 

1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如：`"adfrg"`。 
   
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

5. 若要建立 Data Factory，請從 $ResGrp 變數使用 Location 和 ResourceGroupName 屬性來執行下列 **Set-AzureRmDataFactoryV2** Cmdlet： 
    
    ```powershell       
    $DataFactory = Set-AzureRmDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName -Location $ResGrp.Location -Name $dataFactoryName 
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果發生下列錯誤，請變更名稱，並再試一次。

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為**參與者**或**擁有者**角色，或是 Azure 訂用帳戶的**管理員**。
* 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

### <a name="create-an-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務
建立連結服務，將主控 SSIS 目錄的 Azure SQL 資料庫連結到資料處理站。 資料處理站使用此連結服務中的資訊連線到 SSISDB 資料庫，並執行預存程序來執行 SSIS 套件。 

1. 使用下列內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **AzureSqlDatabaseLinkedService.json** 的 JSON 檔案： 

    > [!IMPORTANT]
    > 儲存檔案之前，以您的 Azure SQL Database 的值取代 &lt;servername&gt;、&lt;username&gt; 和 &lt;password&gt;。

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": {
                    "type": "SecureString",
                    "value": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
                }
            }
        }
    }
    ```

2. 在 **Azure PowerShell** 中，切換至 **C:\ADF\RunSSISPackage** 資料夾。

3. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務：**AzureSqlDatabaseLinkedService**。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>使用預存程序活動建立管線 
在此步驟中，您會建立具有預存程序活動的管線。 活動會叫用 sp_executesql 預存程序執行 SSIS 封裝。 

1. 使用下列內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **RunSSISPackagePipeline.json** 的 JSON 檔案：

    > [!IMPORTANT]
    > 以 SSIS 目錄中的資料夾名稱、專案名稱和封裝名稱取代 &lt;FOLDER NAME&gt;、&lt;PROJECT NAME&gt;、&lt;PACKAGE NAME&gt;，再儲存檔案。 

    ```json
    {
        "name": "RunSSISPackagePipeline",
        "properties": {
            "activities": [
                {
                    "name": "My SProc Activity",
                    "description":"Runs an SSIS package",
                    "type": "SqlServerStoredProcedure",
                    "linkedServiceName": {
                        "referenceName": "AzureSqlDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "typeProperties": {
                        "storedProcedureName": "sp_executesql",
                        "storedProcedureParameters": {
                            "stmt": {
                                "value": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER NAME>', @project_name=N'<PROJECT NAME>', @package_name=N'<PACKAGE NAME>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                            }
                        }
                    }
                }
            ]
        }
    }
    ```

2. 若要建立管線：**RunSSISPackagePipeline**，請執行 **Set-AzureRmDataFactoryV2Pipeline** Cmdlet。

    ```powershell
    $DFPipeLine = Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

    以下是範例輸出：

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-a-pipeline-run"></a>建立管線執行
使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管線。 Cmdlet 會傳回管線執行識別碼，方便後續監視。

```powershell
$RunId = Invoke-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline-run"></a>監視管道執行

執行下列 PowerShell 程式碼以持續檢查管線執行狀態，直到完成複製資料為止。 在 PowerShell 視窗中複製/貼上下列指令碼，然後按 ENTER。 

```powershell
while ($True) {
    $Run = Get-AzureRmDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

### <a name="create-a-trigger"></a>建立觸發程序
在上一個步驟中，您已依需求叫用管線。 您也可以建立排程觸發程序，依排程執行管線 (每小時、每天等)。

1. 使用下列內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **MyTrigger.json** 的 JSON 檔案： 

    ```json
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Hour",
                    "interval": 1,
                    "startTime": "2017-12-07T00:00:00-08:00",
                    "endTime": "2017-12-08T00:00:00-08:00"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "RunSSISPackagePipeline"
                    },
                    "parameters": {}
                }
            ]
        }
    }    
    ```
2. 在 **Azure PowerShell** 中，切換至 **C:\ADF\RunSSISPackage** 資料夾。
3. 執行 **Set-AzureRmDataFactoryV2Trigger** Cmdlet，以建立觸發程序。 

    ```powershell
    Set-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
    ```
4. 觸發程序預設處於已停止狀態。 執行 **Start-AzureRmDataFactoryV2Trigger** Cmdlet 啟動觸發程序。 

    ```powershell
    Start-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" 
    ```
5. 執行 **Get-AzureRmDataFactoryV2Trigger** Cmdlet，確認觸發程序已啟動。 

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"     
    ```    
6. 在下一個小時後，執行下列命令。 例如，如果目前的時間是 UTC 下午 3:25，請在 UTC 下午 4:00 執行命令。 
    
    ```powershell
    Get-AzureRmDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-06" -TriggerRunStartedBefore "2017-12-09"
    ```

    您可以依據 Azure SQL Server 中的 SSISDB 資料庫執行下列查詢，確認已執行套件。 

    ```sql
    select * from catalog.executions
    ```


## <a name="next-steps"></a>後續步驟
您也可以使用 Azure 入口網站監視管線。 如需逐步指示，請參閱[監視管線](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)：
