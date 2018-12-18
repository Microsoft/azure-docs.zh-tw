---
title: 使用 Azure Data Factory 在 Blob 儲存體中複製資料 | Microsoft Docs
description: 建立 Azure 資料處理站，以將資料從 Azure Blob 儲存體中的某個位置複製到另一個位置。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: quickstart
ms.date: 01/22/2018
ms.author: jingwang
ms.openlocfilehash: 04fd3be1bde6f640e5966b445ff2aeb8f3c27458
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43109650"
---
# <a name="create-an-azure-data-factory-using-powershell"></a>使用 PowerShell 建立 Azure 資料處理站 
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](quickstart-create-data-factory-powershell.md)

本快速入門說明如何使用 PowerShell 來建立 Azure 資料處理站。 在此資料處理站中建立的管線會將資料從 Azure Blob 儲存體中的一個資料夾**複製**到其他資料夾。 如需如何使用 Azure Data Factory **轉換**資料的教學課程，請參閱[教學課程︰使用 Spark 轉換資料](transform-data-using-spark.md)。 

> [!NOTE]
> 本文不提供 Data Factory 服務的詳細簡介。 如需 Azure Data Factory 服務簡介，請參閱 [Azure Data Factory 簡介](introduction.md)。

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

### <a name="azure-powershell"></a>Azure PowerShell
依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

#### <a name="log-in-to-powershell"></a>登入 PowerShell

1. 在您的電腦上啟動 **PowerShell**。 讓 PowerShell 保持開啟，直到本快速入門結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。
2. 執行下列命令，並輸入您用來登入 Azure 入口網站的相同 Azure 使用者名稱和密碼：
       
    ```powershell
    Connect-AzureRmAccount
    ```        
3. 執行下列命令以檢視此帳戶的所有訂用帳戶：

    ```powershell
    Get-AzureRmSubscription
    ```
4. 如果您發現您的帳戶與多個訂用帳戶相關聯，請執行下列命令來選取您需要使用的訂用帳戶。 以您的 Azure 訂用帳戶識別碼取代 **SubscriptionId**：

    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<SubscriptionId>"       
    ```

## <a name="create-a-data-factory"></a>建立 Data Factory
1. 定義資源群組名稱的變數，以便稍後在 PowerShell 命令中使用。 將下列命令文字複製到 PowerShell，以雙引號指定 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱，然後執行命令。 例如：`"adfrg"`。 
   
     ```powershell
    $resourceGroupName = "ADFQuickStartRG";
    ```

    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令
2. 若要建立 Azure 資源群組，請執行下列命令： 

    ```powershell
    $ResGrp = New-AzureRmResourceGroup $resourceGroupName -location 'East US'
    ``` 
    如果資源群組已經存在，您可能不想覆寫它。 將不同的值指派給 `$ResourceGroupName` 變數，然後執行一次命令。 
3. 定義 Data Factory 名稱的變數。 

    > [!IMPORTANT]
    >  將資料處理站名稱更新為全域唯一的。 例如，ADFTutorialFactorySP1127。 

    ```powershell
    $dataFactoryName = "ADFQuickStartFactory";
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

## <a name="create-a-linked-service"></a>建立連結的服務

在資料處理站中建立的連結服務，會將您的資料存放區和計算服務連結到資料處理站。 在本快速入門中，您要建立連結 Azure 儲存體的服務，可用來作為來源和接收的存放區。 連結的服務具有連線資訊，可供 Data Factory 服務在執行階段中用來連線。

1. 在 **C:\ADFv2QuickStartPSH** 資料夾中，使用以下內容建立名為 **AzureStorageLinkedService.json** 的 JSON 檔案：(如果 ADFv2QuickStartPSH 資料夾不存在，則加以建立。)。 

    > [!IMPORTANT]
    > 儲存檔案前，以 Azure 儲存體帳戶的名稱和金鑰取代 &lt;accountName&gt; 和 &lt;accountKey&gt;。

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": {
                    "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>;EndpointSuffix=core.windows.net",
                    "type": "SecureString"
                }
            }
        }
    }
    ```
    如果使用 [記事本]，請在 [另存新檔] 對話方塊的 [存檔類型] 欄位中選取 [所有檔案]。 否則，它可能會將 `.txt` 副檔名新增至檔案。 例如： `AzureStorageLinkedService.json.txt`。 如果您在 [檔案總管] 中建立檔案，然後在 [記事本] 中開啟該檔案，您可能不會看到 `.txt` 副檔名，因為預設已設定 [隱藏已知檔案類型的副檔名] 選項。 先移除 `.txt` 副檔名，再繼續下一個步驟。
2. 在 **PowerShell** 中，切換到 **ADFv2QuickStartPSH** 資料夾。

    ```powershell
    Set-Location 'C:\ADFv2QuickStartPSH'
    ```
3. 執行 **Set-AzureRmDataFactoryV2LinkedService** Cmdlet 來建立連結服務：**AzureStorageLinkedService**。 

    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureStorageLinkedService" -DefinitionFile ".\AzureStorageLinkedService.json"
    ```

    以下是範例輸出：

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

## <a name="create-a-dataset"></a>建立資料集
在此步驟中，您可以定義資料集來代表要從來源複製到接收的資料。 資料集是 **AzureBlob** 的類型。 它會參考您在前一個步驟中建立的 **Azure 儲存體連結服務**。 它會接受參數來建構 **folderPath** 屬性。 針對輸入資料集，管線中的複製活動會傳遞輸入路徑作為此參數的值。 同樣地，針對輸出資料集，複製活動會傳遞輸出路徑作為此參數的值。 

1. 在 **C:\ADFv2QuickStartPSH** 資料夾中，使用下列內容建立名為 **BlobDataset.json** 的 JSON 檔案：

    ```json
    {
        "name": "BlobDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "@{dataset().path}"
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            },
            "parameters": {
                "path": {
                    "type": "String"
                }
            }
        }
    }
    ```

2. 若要建立資料集：**BlobDataset**，請執行 **Set-AzureRmDataFactoryV2Dataset** Cmdlet。

    ```powershell
    Set-AzureRmDataFactoryV2Dataset -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "BlobDataset" -DefinitionFile ".\BlobDataset.json"
    ```

    以下是範例輸出：

    ```
    DatasetName       : BlobDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset
    ```

## <a name="create-a-pipeline"></a>建立管線
  
在本快速入門中，您要透過採用兩個參數 -輸入 Blob 路徑和輸出 Blob 路徑的一個活動來建立此管線。 這些參數的值是在觸發/執行管線時設定。 複製活動會使用在前一個步驟中建立作為輸入和輸出的同一個 Blob 資料集。 將該資料集用作輸入資料集時，即會指定輸入路徑。 並且，將該資料集用作輸出資料集時，即會指定輸出路徑。 

1. 在 **C:\ADFv2QuickStartPSH** 資料夾中，使用下列內容建立名為 **Adfv2QuickStartPipeline.json** 的 JSON 檔案：

    ```json
    {
        "name": "Adfv2QuickStartPipeline",
        "properties": {
            "activities": [
                {
                    "name": "CopyFromBlobToBlob",
                    "type": "Copy",
                    "inputs": [
                        {
                            "referenceName": "BlobDataset",
                            "parameters": {
                                "path": "@pipeline().parameters.inputPath"
                            },
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "BlobDataset",
                            "parameters": {
                                "path": "@pipeline().parameters.outputPath"
                            },
                            "type": "DatasetReference"
                        }
                    ],
                    "typeProperties": {
                        "source": {
                            "type": "BlobSource"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    }
                }
            ],
            "parameters": {
                "inputPath": {
                    "type": "String"
                },
                "outputPath": {
                    "type": "String"
                }
            }
        }
    }
    ```

2. 若要建立管線：**Adfv2QuickStartPipeline**，請執行 **Set-AzureRmDataFactoryV2Pipeline** Cmdlet。

    ```powershell
    $DFPipeLine = Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "Adfv2QuickStartPipeline" -DefinitionFile ".\Adfv2QuickStartPipeline.json"
    ```

    以下是範例輸出：

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

## <a name="create-a-pipeline-run"></a>建立管線執行

在此步驟中，您會以來源和接收 Blob 路徑的實際值設定管線參數 **inputPath** 和 **outputPath** 的值。 接著，您可以使用這些引數來建立管線執行。 

1. 在 **C:\ADFv2QuickStartPSH** 資料夾中，使用下列內容建立名為 **PipelineParameters.json** 的 JSON 檔案：

    ```json
    {
        "inputPath": "adftutorial/input",
        "outputPath": "adftutorial/output"
    }
    ```
2. 執行 **Invoke-AzureRmDataFactoryV2Pipeline** Cmdlet 來建立管線執行，並傳入參數值。 Cmdlet 會傳回管線執行識別碼，方便後續監視。

    ```powershell
    $RunId = Invoke-AzureRmDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineName $DFPipeLine.Name -ParameterFile .\PipelineParameters.json
    ```

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1. 執行下列 PowerShell 程式碼以持續檢查管線執行狀態，直到完成複製資料為止。 在 PowerShell 視窗中複製/貼上下列指令碼，然後按 ENTER。 

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

    以下是管線執行的範例輸出：

    ```
    Pipeline is running...status: InProgress
    Pipeline run finished. The status is:  Succeeded
    
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : SPTestFactory0928
    RunId             : 0000000000-0000-0000-0000-0000000000000
    PipelineName      : Adfv2QuickStartPipeline
    LastUpdated       : 9/28/2017 8:28:38 PM
    Parameters        : {[inputPath, adftutorial/input], [outputPath, adftutorial/output]}
    RunStart          : 9/28/2017 8:28:14 PM
    RunEnd            : 9/28/2017 8:28:38 PM
    DurationInMs      : 24151
    Status            : Succeeded
    Message           :
    ```

    如果您看到錯誤：
    ```
    Activity CopyFromBlobToBlob failed: Failed to detect region of linked service 'AzureStorage' : 'AzureStorageLinkedService' with error '[Region Resolver] Azure Storage failed to get address for DNS. Warning: System.Net.Sockets.SocketException (0x80004005): No such host is known
    ```
    請執行下列步驟： 
    1. 在 AzureStorageLinkedService.json 中，確認您 Azure 儲存體帳戶的名稱和金鑰是正確的。 
    2. 請確認連接字串的格式是正確的。 例如，AccountName 和 AccountKey 屬性會以分號 (`;`) 字元分隔。 
    3. 如果您的帳戶名稱和帳戶金鑰周圍有角括號，請將它們移除。 
    4. 下列為範例連接字串： 

        ```json
        "connectionString": {
            "value": "DefaultEndpointsProtocol=https;AccountName=mystorageaccountname;AccountKey=mystorageacountkey;EndpointSuffix=core.windows.net",
            "type": "SecureString"
        }
        ```
    5. 遵循[建立連結的服務](#create-a-linked-service)一節中的步驟重新建立連結的服務。 
    6. 遵循[建立管線執行](#create-a-pipeline-run)重新執行管線。 
    7. 再次執行目前的監視命令來監視新的管線執行。 
1. 執行下列指令碼來取出複製活動執行詳細資料，例如，讀取/寫入資料的大小。

    ```powershell
    Write-Output "Activity run details:"
    $Result = Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineRunId $RunId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $Result

    Write-Output "Activity 'Output' section:"
    $Result.Output -join "`r`n"

    Write-Output "Activity 'Error' section:"
    $Result.Error -join "`r`n"
    ```
3. 確認您看到的輸出類似下列活動執行結果範例輸出：

    ```json
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : SPTestFactory0928
    ActivityName      : CopyFromBlobToBlob
    PipelineRunId     : 00000000000-0000-0000-0000-000000000000
    PipelineName      : Adfv2QuickStartPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, copyDuration, throughput...}
    LinkedServiceName :
    ActivityRunStart  : 9/28/2017 8:28:18 PM
    ActivityRunEnd    : 9/28/2017 8:28:36 PM
    DurationInMs      : 18095
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity 'Output' section:
    "dataRead": 38
    "dataWritten": 38
    "copyDuration": 7
    "throughput": 0.01
    "errors": []
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (West US)"
    "usedDataIntegrationUnits": 2
    "billedDuration": 14
    ```

[!INCLUDE [data-factory-quickstart-verify-output-cleanup.md](../../includes/data-factory-quickstart-verify-output-cleanup.md)] 

## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 瀏覽[教學課程](tutorial-copy-data-dot-net.md)以了解使用 Data Factory 的更多案例。 
