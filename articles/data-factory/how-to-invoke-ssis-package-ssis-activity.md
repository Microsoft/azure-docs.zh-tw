---
title: 使用 Execute SSIS 套件活動執行 SSIS 套件 - Azure | Microsoft Docs
description: 本文描述如何使用 Execute SSIS 套件活動，在 Azure Data Factory 管線執行 SQL Server Integration Services (SSIS) 套件。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/16/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: b2e0b65f210774f760ce2d0898c601115ab3a94d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960153"
---
# <a name="run-an-ssis-package-with-the-execute-ssis-package-activity-in-azure-data-factory"></a>在 Azure Data Factory 中使用 Execute SSIS 套件活動執行 SSIS 套件
本文描述如何使用 Execute SSIS 套件活動，在 Azure Data Factory 管線執行 SSIS 套件。 

## <a name="prerequisites"></a>必要條件

**Azure SQL Database**。 這篇文章中的逐步解說會使用裝載 SSIS 目錄的 Azure SQL 資料庫。 您也可以使用 Azure SQL Database 受控執行個體。

## <a name="create-an-azure-ssis-integration-runtime"></a>建立 Azure-SSIS 整合執行階段
如果您未依照[教學課程：部署 SSIS 套件](tutorial-create-azure-ssis-runtime-portal.md)中的逐步指示進行，請建立 Azure SSIS 整合執行階段。

## <a name="run-a-package-in-the-azure-portal"></a>在 Azure 入口網站中執行套件
在本節中，您會使用 Data Factory UI 來建立 Data Factory 管線，其中具有要執行 SSIS 套件的 Execute SSIS 套件活動。

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

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>使用 Execute SSIS 套件活動建立管線
在此步驟中，您可以使用資料處理站 UI 建立管線。 您要將 Execute SSIS 套件活動新增至管線，並設定它來執行 SSIS 套件。 

1. 在 [開始使用] 頁面中，按一下 [建立管線]： 

    ![開始使用頁面](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)
2. 在 [活動] 工具箱中展開 [一般]，然將 [執行 SSIS 套件] 活動拖放至管線設計工具介面。 

   ![將執行 SSIS 套件活動拖曳至設計工具介面](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-designer.png) 

3. 在 Execute SSIS 套件活動的屬性 **[一般]** 索引標籤上，提供活動的名稱和描述。 設定選用的逾時和重試值。

    ![在 [一般] 索引標籤上設定屬性](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png)

4. 在 Execute SSIS 套件活動的屬性 **[設定]** 索引標籤上，選取與套件部署所在 `SSISDB` 資料庫相關聯的 Azure-SSIS Integration Runtime。 使用 `<folder name>/<project name>/<package name>.dtsx` 格式來提供 `SSISDB` 資料庫中的套件路徑。 選擇性地指定 32 位元執行及預先定義或自訂記錄層級，並使用 `<folder name>/<environment name>` 格式來提供環境中的路徑。

    ![在 [設定] 索引標籤上設定屬性](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings.png)

5. 若要驗證管線設定，按一下工具列上的 [驗證]。 若要關閉 [管線驗證報告]，按一下 **>>**。

6. 按一下 [全部發行] 按鈕，將管線發行至資料處理站。 

### <a name="optionally-parameterize-the-activity"></a>選擇性將活動參數化

(選擇性) 使用 [執行 SSIS 套件活動] 方塊底部的 [檢視原始程式碼] 按鈕，或管線區域右上角的 [程式碼] 按鈕，將可以參考 Data Factory 系統變數的值、運算式或函式，以 JSON 格式指派給您的專案或封裝參數。 例如，您可以將 Data Factory 管線參數指派給 SSIS 專案或套件參數，如下列螢幕擷取畫面所示：

![編輯執行 SSIS 套件活動的 JSON 指令碼](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-parameters.png)

![將參數新增至 Execute SSIS 套件活動](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-parameters2.png)

### <a name="run-the-pipeline"></a>執行管道
在本節中，您會觸發管線執行，然後監視執行的情況。 

1. 若要觸發管線執行，按一下工具列上的 [觸發程序]，然後按一下 [立即觸發]。 

    ![立即觸發](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-trigger.png)

2. 在 [管線執行] 視窗中，選取 [完成]。 

### <a name="monitor-the-pipeline"></a>監視管線

1. 切換至左側的 [監視] 索引標籤。 您會看到管線執行、其狀態，以及其他資訊 (例如執行開始時間)。 若要重新整理檢視，按一下 [重新整理]。

    ![管線執行](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

2. 按一下 [動作]資料行中的 [檢視活動執行] 連結。 您只會看到一個活動執行，因為該管線只有一個活動 (Execute SSIS 套件活動)。

    ![活動執行](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-runs.png)

3. 您可以依據 Azure SQL Server 中的 SSISDB 資料庫執行下列**查詢**，來確認套件已執行。 

    ```sql
    select * from catalog.executions
    ```

    ![確認封裝執行](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)

4. 您也可以從管線活動執行的輸出取得 SSISDB 執行識別碼，並使用識別碼來檢查更完整的執行記錄和 SSMS 中的錯誤訊息。

    ![取得執行識別碼。](media/how-to-invoke-ssis-package-ssis-activity/get-execution-id.png)

### <a name="schedule-the-pipeline-with-a-trigger"></a>使用觸發程序排程管線

您也可以建立管線的排程觸發程序，以便管線依排程執行 (每小時、每日等等)。 如需範例，請參閱[建立資料處理站 - 資料處理站 UI](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule)。

## <a name="run-a-package-with-powershell"></a>使用 PowerShell 執行套件
在本節中，您會使用 Azure PowerShell 來建立 Data Factory 管線，其中具有要執行 SSIS 套件的執行 SSIS 套件活動。 

依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。 

### <a name="create-a-data-factory"></a>建立 Data Factory
您可以使用具有 Azure SSIS IR 的同一個資料處理站，也可以建立另一個資料處理站。 下列程序提供建立資料處理站的步驟。 您可以使用此資料處理站中的執行 SSIS 套件活動建立管線。 執行 SSIS 套件活動會執行您的 SSIS 套件。 

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
    $DataFactory = Set-AzureRmDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName `
                                            -Location $ResGrp.Location `
                                            -Name $dataFactoryName 
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果發生下列錯誤，請變更名稱，並再試一次。

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```
* 若要建立 Data Factory 執行個體，您用來登入 Azure 的使用者帳戶必須為**參與者**或**擁有者**角色，或是 Azure 訂用帳戶的**管理員**。
* 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>使用 Execute SSIS 套件活動建立管線 
在此步驟中，您要使用執行 SSIS 套件活動建立管線。 此活動會執行您的 SSIS 套件。 

1. 使用類似下列範例的內容，在 **C:\ADF\RunSSISPackage** 資料夾中建立名為 **RunSSISPackagePipeline.json** 的 JSON 檔案：

    > [!IMPORTANT]
    > 儲存檔案之前，請先取代物件名稱、描述和路徑、屬性和參數值、密碼及其他變數值。 

    ```json
    {
        "name": "RunSSISPackagePipeline",
        "properties": {
            "activities": [{
                "name": "mySSISActivity",
                "description": "My SSIS package/activity description",
                "type": "ExecuteSSISPackage",
                "typeProperties": {
                    "connectVia": {
                        "referenceName": "myAzureSSISIR",
                        "type": "IntegrationRuntimeReference"
                    },
                    "runtime": "x64",
                    "loggingLevel": "Basic",
                    "packageLocation": {
                        "packagePath": "FolderName/ProjectName/PackageName.dtsx"            
                    },
                    "environmentPath":   "FolderName/EnvironmentName",
                    "projectParameters": {
                        "project_param_1": {
                            "value": "123"
                        }
                    },
                    "packageParameters": {
                        "package_param_1": {
                            "value": "345"
                        }
                    },
                    "projectConnectionManagers": {
                        "MyAdonetCM": {
                            "userName": {
                                "value": "sa"
                            },
                            "passWord": {
                                "value": {
                                    "type": "SecureString",
                                    "value": "abc"
                                }
                            }
                        }
                    },
                    "packageConnectionManagers": {
                        "MyOledbCM": {
                            "userName": {
                                "value": "sa"
                            },
                            "passWord": {
                                "value": {
                                    "type": "SecureString",
                                    "value": "def"
                                }
                            }
                        }
                    },
                    "propertyOverrides": {
                        "\\PackageName.dtsx\\MaxConcurrentExecutables ": {
                            "value": 8,
                            "isSensitive": false
                        }
                    }
                },
                "policy": {
                    "timeout": "0.01:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30
                }
            }]
        }
    }
    ```

2.  在 Azure PowerShell 中，切換至 `C:\ADF\RunSSISPackage` 資料夾。

3. 若要建立管線 **RunSSISPackagePipeline**，請執行 **Set-AzureRmDataFactoryV2Pipeline** Cmdlet。

    ```powershell
    $DFPipeLine = Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                                   -ResourceGroupName $ResGrp.ResourceGroupName `
                                                   -Name "RunSSISPackagePipeline"
                                                   -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

    以下是範例輸出：

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="run-the-pipeline"></a>執行管道
使用 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 執行管線。 Cmdlet 會傳回管線執行識別碼，方便後續監視。

```powershell
$RunId = Invoke-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                             -ResourceGroupName $ResGrp.ResourceGroupName `
                                             -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline"></a>監視管線

執行下列 PowerShell 程式碼以持續檢查管線執行狀態，直到完成複製資料為止。 在 PowerShell 視窗中複製/貼上下列指令碼，然後按 ENTER。 

```powershell
while ($True) {
    $Run = Get-AzureRmDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName `
                                               -DataFactoryName $DataFactory.DataFactoryName `
                                               -PipelineRunId $RunId

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

您也可以使用 Azure 入口網站監視管線。 如需逐步指示，請參閱[監視管線](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)：

### <a name="schedule-the-pipeline-with-a-trigger"></a>使用觸發程序排程管線
在上一個步驟中，您已依需求執行管線。 您也可以建立排程觸發程序，依排程執行管線 (每小時、每天等)。

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
    Set-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                    -DataFactoryName $DataFactory.DataFactoryName `
                                    -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
    ```
4. 觸發程序預設處於已停止狀態。 執行 **Start-AzureRmDataFactoryV2Trigger** Cmdlet 啟動觸發程序。 

    ```powershell
    Start-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                      -DataFactoryName $DataFactory.DataFactoryName `
                                      -Name "MyTrigger" 
    ```
5. 執行 **Get-AzureRmDataFactoryV2Trigger** Cmdlet，確認觸發程序已啟動。 

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName `
                                    -DataFactoryName $DataFactoryName `
                                    -Name "MyTrigger"     
    ```    
6. 在下一個小時後，執行下列命令。 例如，如果目前的時間是 UTC 下午 3:25，請在 UTC 下午 4:00 執行命令。 
    
    ```powershell
    Get-AzureRmDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName `
                                       -DataFactoryName $DataFactoryName `
                                       -TriggerName "MyTrigger" `
                                       -TriggerRunStartedAfter "2017-12-06" `
                                       -TriggerRunStartedBefore "2017-12-09"
    ```

    您可以依據 Azure SQL Server 中的 SSISDB 資料庫執行下列查詢，確認已執行套件。 

    ```sql
    select * from catalog.executions
    ```

## <a name="next-steps"></a>後續步驟
請參閱下列部落格文章：
-   [使用 ADF 管線中的 SSIS 活動來現代化及擴充您的 ETL/ELT 工作流程](https://blogs.msdn.microsoft.com/ssis/2018/05/23/modernize-and-extend-your-etlelt-workflows-with-ssis-activities-in-adf-pipelines/)
