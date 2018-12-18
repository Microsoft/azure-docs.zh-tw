---
title: 使用 Resource Manager 範本建立 Azure Data Factory | Microsoft Docs
description: 在本教學課程中，您會使用 Azure Resource Manager 範本來建立範例 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 01/22/2018
ms.author: douglasl
ms.openlocfilehash: a7c11e4bd3ac30c930ec717426c43d4361e32088
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43088857"
---
# <a name="tutorial-create-an-azure-data-factory-using-azure-resource-manager-template"></a>教學課程：使用 Azure Resource Manager 範本建立 Azure Data Factory
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-build-your-first-pipeline-using-arm.md)
> * [目前的版本](quickstart-create-data-factory-resource-manager-template.md) 

本快速入門說明如何使用 Azure Resource Manager 範本建立 Azure Data Factory。 在此資料處理站中建立的管線會將資料從 Azure Blob 儲存體中的一個資料夾**複製**到其他資料夾。 如需如何使用 Azure Data Factory **轉換**資料的教學課程，請參閱[教學課程︰使用 Spark 轉換資料](transform-data-using-spark.md)。 

> [!NOTE]
> 本文不提供 Data Factory 服務的詳細簡介。 如需 Azure Data Factory 服務簡介，請參閱 [Azure Data Factory 簡介](introduction.md)。

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

### <a name="azure-powershell"></a>Azure PowerShell
依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

## <a name="resource-manager-templates"></a>Resource Manager 範本
若要了解一般的 Azure Resource Manager 範本，請參閱[撰寫 Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)。 

下節會提供完整的 Resource Manager 範本來定義 Data Factory 實體，如此您可以快速執行教學課程並測試範本。 若要了解每個 Data Factory 實體的定義方式，請參閱[範本中的 Data Factory 實體](#data-factory-entities-in-the-template)一節。

## <a name="data-factory-json"></a>Data Factory JSON 
在 **C:\ADFTutorial** 資料夾中，使用下列內容建立名為 **ADFTutorialARM.json** 的 JSON 檔案：

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
        "dataFactoryName": {
            "type": "string",
            "metadata": {
                "description": "Name of the data factory. Must be globally unique."
            }
        },
        "dataFactoryLocation": {
            "type": "string",
            "allowedValues": [
                "East US",
                "East US 2",
                "West Europe"
            ],
            "defaultValue": "East US",
            "metadata": {
                "description": "Location of the data factory. Currently, only East US, East US 2, and West Europe are supported. "
            }
        },
        "storageAccountName": {
            "type": "string",
            "metadata": {
                "description": "Name of the Azure storage account that contains the input/output data."
            }
        },
        "storageAccountKey": {
            "type": "securestring",
            "metadata": {
                "description": "Key for the Azure storage account."
            }
        },
        "blobContainer": {
            "type": "string",
            "metadata": {
                "description": "Name of the blob container in the Azure Storage account."
            }
        },
        "inputBlobFolder": {
            "type": "string",
            "metadata": {
                "description": "The folder in the blob container that has the input file."
            }
        },
        "inputBlobName": {
            "type": "string",
            "metadata": {
                "description": "Name of the input file/blob."
            }
        },
        "outputBlobFolder": {
            "type": "string",
            "metadata": {
                "description": "The folder in the blob container that will hold the transformed data."
            }
        },
        "outputBlobName": {
            "type": "string",
            "metadata": {
                "description": "Name of the output file/blob."
            }
        },
        "triggerStartTime": {
            "type": "string",
            "metadata": {
                "description": "Start time for the trigger."
            }
        },
        "triggerEndTime": {
            "type": "string",
            "metadata": {
                "description": "End time for the trigger."
            }
        }
    },
    "variables": {
        "azureStorageLinkedServiceName": "ArmtemplateStorageLinkedService",
        "inputDatasetName": "ArmtemplateTestDatasetIn",
        "outputDatasetName": "ArmtemplateTestDatasetOut",
        "pipelineName": "ArmtemplateSampleCopyPipeline",
        "triggerName": "ArmTemplateTestTrigger"
    },
    "resources": [{
        "name": "[parameters('dataFactoryName')]",
        "apiVersion": "2017-09-01-preview",
        "type": "Microsoft.DataFactory/factories",
        "location": "[parameters('dataFactoryLocation')]",
        "properties": {
            "loggingStorageAccountName": "[parameters('storageAccountName')]",
            "loggingStorageAccountKey": "[parameters('storageAccountKey')]"
        },
        "resources": [{
                "type": "linkedservices",
                "name": "[variables('azureStorageLinkedServiceName')]",
                "dependsOn": [
                    "[parameters('dataFactoryName')]"
                ],
                "apiVersion": "2017-09-01-preview",
                "properties": {
                    "type": "AzureStorage",
                    "description": "Azure Storage linked service",
                    "typeProperties": {
                        "connectionString": {
                            "value": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]",
                            "type": "SecureString"
                        }
                    }
                }
            },
            {
                "type": "datasets",
                "name": "[variables('inputDatasetName')]",
                "dependsOn": [
                    "[parameters('dataFactoryName')]",
                    "[variables('azureStorageLinkedServiceName')]"
                ],
                "apiVersion": "2017-09-01-preview",
                "properties": {
                    "type": "AzureBlob",
                    "typeProperties": {
                        "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'), '/')]",
                        "fileName": "[parameters('inputBlobName')]"
                    },
                    "linkedServiceName": {
                        "referenceName": "[variables('azureStorageLinkedServiceName')]",
                        "type": "LinkedServiceReference"
                    }
                }
            },
            {
                "type": "datasets",
                "name": "[variables('outputDatasetName')]",
                "dependsOn": [
                    "[parameters('dataFactoryName')]",
                    "[variables('azureStorageLinkedServiceName')]"
                ],
                "apiVersion": "2017-09-01-preview",
                "properties": {
                    "type": "AzureBlob",
                    "typeProperties": {
                        "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'), '/')]",
                        "fileName": "[parameters('outputBlobName')]"
                    },
                    "linkedServiceName": {
                        "referenceName": "[variables('azureStorageLinkedServiceName')]",
                        "type": "LinkedServiceReference"
                    }
                }
            },
            {
                "type": "pipelines",
                "name": "[variables('pipelineName')]",
                "dependsOn": [
                    "[parameters('dataFactoryName')]",
                    "[variables('azureStorageLinkedServiceName')]",
                    "[variables('inputDatasetName')]",
                    "[variables('outputDatasetName')]"
                ],
                "apiVersion": "2017-09-01-preview",
                "properties": {
                    "activities": [{
                        "type": "Copy",
                        "typeProperties": {
                            "source": {
                                "type": "BlobSource"
                            },
                            "sink": {
                                "type": "BlobSink"
                            }
                        },
                        "name": "MyCopyActivity",
                        "inputs": [{
                            "referenceName": "[variables('inputDatasetName')]",
                            "type": "DatasetReference"
                        }],
                        "outputs": [{
                            "referenceName": "[variables('outputDatasetName')]",
                            "type": "DatasetReference"
                        }]
                    }]
                }
            },
            {
                "type": "triggers",
                "name": "[variables('triggerName')]",
                "dependsOn": [
                    "[parameters('dataFactoryName')]",
                    "[variables('azureStorageLinkedServiceName')]",
                    "[variables('inputDatasetName')]",
                    "[variables('outputDatasetName')]",
                    "[variables('pipelineName')]"
                ],
                "apiVersion": "2017-09-01-preview",
                "properties": {
                    "type": "ScheduleTrigger",
                    "typeProperties": {
                        "recurrence": {
                            "frequency": "Hour",
                            "interval": 1,
                            "startTime": "[parameters('triggerStartTime')]",
                            "endTime": "[parameters('triggerEndTime')]",
                            "timeZone": "UTC"
                        }
                    },
                    "pipelines": [{
                        "pipelineReference": {
                            "type": "PipelineReference",
                            "referenceName": "ArmtemplateSampleCopyPipeline"
                        },
                        "parameters": {}
                    }]
                }
            }
        ]
    }]
}
```

## <a name="parameters-json"></a>參數 JSON
建立名為 **ADFTutorialARM Parameters.json** 的 JSON 檔案，其中包含 Azure Resource Manager 範本的參數。  

> [!IMPORTANT]
> - 針對此參數檔案中的 **storageAccountName** 和 **storageAccountKey** 參數指定您 Azure 儲存體帳戶的名稱和金鑰。 您已建立 adftutorial 容器並將範例檔案 (emp.txt) 上傳到此 Azure blob 儲存體中的輸入資料夾。 
> - 為 **dataFactoryName** 參數的資料處理站指定全域唯一的名稱。 例如：ARMTutorialFactoryJohnDoe11282017。 
> - 針對 **triggerStartTime**，指定以下格式的當天日期：`2017-11-28T00:00:00`。
> - 針對 **triggerEndTime**，指定以下格式的隔天日期：`2017-11-29T00:00:00`。 您也可檢查目前的 UTC 時間，並指定下一個小時或兩個小時作為結束時間。 例如，如果現在的 UTC 時間為上午 1:32，則指定 `2017-11-29:03:00:00` 作為結束時間。 在此情況下，觸發程序會執行管線兩次 (在上午 2 點和上午 3 點)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "dataFactoryName": {
      "value": "<datafactoryname>"
    },    
    "dataFactoryLocation": {
      "value": "East US"
    },
    "storageAccountName": {
      "value": "<yourstroageaccountname>"
    },
    "storageAccountKey": {
      "value": "<yourstorageaccountkey>"
    },
    "blobContainer": {
      "value": "adftutorial"
    },
    "inputBlobFolder": {
      "value": "input"
    },
    "inputBlobName": {
      "value": "emp.txt"
    },
    "outputBlobFolder": {
      "value": "output"
    },
    "outputBlobName": {
      "value": "emp.txt"
    },
    "triggerStartTime": {
        "value": "2017-11-28T00:00:00. Set to today"
    },
    "triggerEndTime": {
        "value": "2017-11-29T00:00:00. Set to tomorrow"
    }
  }
}
```

> [!IMPORTANT]
> 您可能需要不同的參數 JSON 檔案，以供可與相同 Data Factory JSON 範本共同使用的開發、測試和生產環境使用。 使用 Power Shell 指令碼，您可以在這些環境中自動部署 Data Factory 實體。 

## <a name="deploy-data-factory-entities"></a>部署 Data Factory 實體 
在 PowerShell 中，執行下列命令，使用您稍早在本快速入門中建立的 Resource Manager 範本來部署 Data Factory 實體。 

```PowerShell
New-AzureRmResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFTutorial\ADFTutorialARM.json -TemplateParameterFile C:\ADFTutorial\ADFTutorialARM-Parameters.json
```

您會看到類似以下的範例： 

```
DeploymentName          : MyARMDeployment
ResourceGroupName       : ADFTutorialResourceGroup
ProvisioningState       : Succeeded
Timestamp               : 11/29/2017 3:11:13 AM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                 Type            Value
                          ===============      ============    ==========
                          dataFactoryName      String          <data factory name>
                          dataFactoryLocation  String          East US
                          storageAccountName   String          <storage account name>
                          storageAccountKey    SecureString
                          blobContainer        String          adftutorial
                          inputBlobFolder      String          input
                          inputBlobName        String          emp.txt
                          outputBlobFolder     String          output
                          outputBlobName       String          emp.txt
                          triggerStartTime     String          11/29/2017 12:00:00 AM
                          triggerEndTime       String          11/29/2017 4:00:00 AM

Outputs                 :
DeploymentDebugLogLevel :
```

## <a name="start-the-trigger"></a>啟動觸發程序

此範本會部署下列 Data Factory 實體： 

- Azure 儲存體連結服務
- Azure Blob 資料集 (輸入和輸出)
- 具有複製活動的管線
- 可觸發管線的觸發程序

部署的觸發程序處於已停止狀態。 啟動觸發程序的其中一個方式是使用 **Start-AzureRmDataFactoryV2Trigger** PowerShell cmdlet。 下列程序會提供詳細的步驟： 

1. 在 PowerShell 視窗中，建立變數，以保留資源群組的名稱。 將下列指令碼複製到 PowerShell 視窗中，然後按 ENTER。 如果您為 New-AzureRmResourceGroupDeployment 命令指定了不同的資源群組名稱，請在此更新此值。 

    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup"
    ``` 
1. 建立變數以保留資料處理站的名稱。 指定您在 ADFTutorialARM-Parameters.json 檔案中指定的相同名稱。   

    ```powershell
    $dataFactoryName = "<yourdatafactoryname>"
    ```
3. 設定觸發程序名稱的變數。 觸發程序的名稱已硬式編碼在 Resource Manager 範本檔案 (ADFTutorialARM.json) 中。

    ```powershell
    $triggerName = "ArmTemplateTestTrigger"
    ```
4. 在指定資料處理站和觸發程序的名稱之後，執行下列 PowerShell 命令，以取得**觸發程序的狀態**：

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $triggerName
    ```

    以下是範例輸出： 

    ```json
    TriggerName       : ArmTemplateTestTrigger
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ARMFactory1128
    Properties        : Microsoft.Azure.Management.DataFactory.Models.ScheduleTrigger
    RuntimeState      : Stopped
    ```
    
    請注意，觸發程序的執行階段狀態為 [已停止]。 
5. **啟動觸發程序**。 觸發程序會在整點執行範本中定義的管線。 也就是，如果您在下午 2:25 執行此命令，則觸發程序會在下午 3 點第一次執行管線。 然後，它會每小時執行管線，直到您為觸發程序指定的結束時間為止。 

    ```powershell
    Start-AzureRmDataFactoryV2Trigger -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -TriggerName $triggerName
    ```
    
    以下是範例輸出： 
    
    ```
    Confirm
    Are you sure you want to start trigger 'ArmTemplateTestTrigger' in data factory 'ARMFactory1128'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
    True
    ```
6. 再次執行 Get-AzureRmDataFactoryV2Trigger 命令，確認觸發程序已啟動。  

    ```powershell
    Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -TriggerName $triggerName
    ```
    
    以下是範例輸出：
    
    ```
    TriggerName       : ArmTemplateTestTrigger
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ARMFactory1128
    Properties        : Microsoft.Azure.Management.DataFactory.Models.ScheduleTrigger
    RuntimeState      : Started
    ```

## <a name="monitor-the-pipeline"></a>監視管線
1. 登入 [Azure 入口網站](https://portal.azure.com/)後，按一下 [所有服務]，以 **data fa** 等關鍵字進行搜尋，然後選取 [資料處理站]。

    ![瀏覽資料處理站功能表](media/quickstart-create-data-factory-resource-manager-template/browse-data-factories-menu.png)
2. 在 [資料管理站] 頁面中，按一下您建立的資料管理站。 如有需要，以您的資料管理站名稱篩選清單。  

    ![選取 Data Factory](media/quickstart-create-data-factory-resource-manager-template/select-data-factory.png)
3. 在資料管理站頁面上，按一下 [監視及管理] 圖格。 

    ![監視及管理圖格](media/quickstart-create-data-factory-resource-manager-template/monitor-manage-tile.png)
4. [資料整合應用程式] 應該在網頁瀏覽器的個別索引標籤中開啟。 如果監視索引標籤不在作用中，請切換到 [監視索引標籤]。請注意，**排程器觸發程序**已觸發管線執行。 

    ![監視管線執行](media/quickstart-create-data-factory-resource-manager-template/monitor-pipeline-run.png)    

    > [!IMPORTANT]
    > 您只會在整點看到管線執行 (例如：4 AM、5 AM、6 AM 等)。 按一下工具列上的 [重新整理]，以在時間達到下一個小時時重新整理清單。 
5. 按一下 [動作] 資料行中的連結。 

    ![管線動作連結](media/quickstart-create-data-factory-resource-manager-template/pipeline-actions-link.png)
6. 您會看到與管線執行相關聯的活動執行。 本快速入門中，管線只有一個以下類型的活動：複製。 因此，您會看到該活動的執行。 

    ![活動執行](media/quickstart-create-data-factory-resource-manager-template/activity-runs.png)
1. 按一下 [輸出] 資料行底下的連結。 您會在 [輸出] 視窗中看到複製作業的輸出。 按一下最大化按鈕，以查看完整的輸出。 您可以關閉最大化的輸出視窗，或將它關閉。 

    ![[輸出] 視窗](media/quickstart-create-data-factory-resource-manager-template/output-window.png)
7. 一旦您看到成功/失敗執行，請停止觸發程序。 觸發程序會一小時執行一次管線。 管線會將相同的檔案從每次執行的輸入資料夾複製到輸出資料夾。 若要停止觸發程序，請在 PowerShell 視窗中執行下列命令。 
    
    ```powershell
    Stop-AzureRmDataFactoryV2Trigger -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $triggerName
    ```

[!INCLUDE [data-factory-quickstart-verify-output-cleanup.md](../../includes/data-factory-quickstart-verify-output-cleanup.md)] 

## <a name="data-factory-entities-in-the-template"></a>實體的 JSON 定義
下列的 Data Factory 實體定義於 JSON 範本中︰ 

- [Azure 儲存體連結服務](#azure-storage-linked-service)
- [Azure Blob 輸入資料集](#azure-blob-input-dataset)
- [Azure Blob 輸出資料集](#azure-blob-output-dataset)
- [具有複製活動的管線](#data-pipeline)
- [觸發程序](#trigger)

#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 您已建立容器並將資料上傳到此儲存體帳戶，作為必要條件的一部分。 在此區段中指定您 Azure 儲存體帳戶的名稱和金鑰。 如需用來定義 Azure 儲存體連結服務之 JSON 屬性的詳細資料，請參閱 [Azure 儲存體連結服務](connector-azure-blob-storage.md#linked-service-properties)。 

```json
{
    "type": "linkedservices",
    "name": "[variables('azureStorageLinkedServiceName')]",
    "dependsOn": [
        "[parameters('dataFactoryName')]"
    ],
    "apiVersion": "2017-09-01-preview",
    "properties": {
        "type": "AzureStorage",
        "description": "Azure Storage linked service",
        "typeProperties": {
            "connectionString": {
                "value": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]",
                "type": "SecureString"
            }
        }
    }
}
```

connectionString 會使用 storageAccountName 和 storageAccountKey 參數。 使用組態檔傳遞這些參數的值。 定義也會使用在範本中定義的變數︰azureStroageLinkedService 和 dataFactoryName。 

#### <a name="azure-blob-input-dataset"></a>Azure Blob 輸入資料集
Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 在 Azure Blob 資料集定義中，您可指定 Blob 容器、資料夾和包含輸入資料之檔案的名稱。 請參閱 [Azure Blob 資料集屬性](connector-azure-blob-storage.md#dataset-properties)，以取得用來定義 Azure Blob 資料集之 JSON 屬性的詳細資訊。 

```json
{
    "type": "datasets",
    "name": "[variables('inputDatasetName')]",
    "dependsOn": [
    "[parameters('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2017-09-01-preview",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
        "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'), '/')]",
        "fileName": "[parameters('inputBlobName')]"
    },
    "linkedServiceName": {
        "referenceName": "[variables('azureStorageLinkedServiceName')]",
        "type": "LinkedServiceReference"
    }
    }
},

```

#### <a name="azure-blob-output-dataset"></a>Azure Blob 輸出資料集
您可在 Azure Blob 儲存體中指定資料夾名稱，該資料夾可保留從輸入資料夾複製的資料。 請參閱 [Azure Blob 資料集屬性](connector-azure-blob-storage.md#dataset-properties)，以取得用來定義 Azure Blob 資料集之 JSON 屬性的詳細資訊。 

```json
{
    "type": "datasets",
    "name": "[variables('outputDatasetName')]",
    "dependsOn": [
    "[parameters('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2017-09-01-preview",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
        "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'), '/')]",
        "fileName": "[parameters('outputBlobName')]"
    },
    "linkedServiceName": {
        "referenceName": "[variables('azureStorageLinkedServiceName')]",
        "type": "LinkedServiceReference"
    }
    }
}
```

#### <a name="data-pipeline"></a>Data Pipeline
定義可將資料從 Azure blob 資料集複製到 Azure blob 資料集的管線。 請參閱[管線 JSON](concepts-pipelines-activities.md#pipeline-json)，以取得用來在此範例中定義管線的 JSON 元素之描述。 

```json
{
    "type": "pipelines",
    "name": "[variables('pipelineName')]",
    "dependsOn": [
    "[parameters('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]",
    "[variables('inputDatasetName')]",
    "[variables('outputDatasetName')]"
    ],
    "apiVersion": "2017-09-01-preview",
    "properties": {
    "activities": [
        {
        "type": "Copy",
        "typeProperties": {
            "source": {
            "type": "BlobSource"
            },
            "sink": {
            "type": "BlobSink"
            }
        },
        "name": "MyCopyActivity",
        "inputs": [
            {
            "referenceName": "[variables('inputDatasetName')]",
            "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
            "referenceName": "[variables('outputDatasetName')]",
            "type": "DatasetReference"
            }
        ]
        }
    ]
    }
}
```

#### <a name="trigger"></a>觸發程序
定義一小時執行一次管線的觸發程序。 部署的觸發程序處於已停止狀態。 使用 **Start-AzureRmDataFactoryV2Trigger** Cmdlet 來啟動觸發程序。 如需觸發程序的詳細資訊，請參閱[管線執行和觸發程序](concepts-pipeline-execution-triggers.md#triggers)一文。 

```json
{
    "type": "triggers",
    "name": "[variables('triggerName')]",
    "dependsOn": [
        "[parameters('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]",
        "[variables('inputDatasetName')]",
        "[variables('outputDatasetName')]",
        "[variables('pipelineName')]"
    ],
    "apiVersion": "2017-09-01-preview",
    "properties": {
        "type": "ScheduleTrigger",
        "typeProperties": {
            "recurrence": {
                "frequency": "Hour",
                "interval": 1,
                "startTime": "2017-11-28T00:00:00",
                "endTime": "2017-11-29T00:00:00",
                "timeZone": "UTC"               
            }
        },
        "pipelines": [{
            "pipelineReference": {
                "type": "PipelineReference",
                "referenceName": "ArmtemplateSampleCopyPipeline"
            },
            "parameters": {}
        }]
    }
}
```

## <a name="reuse-the-template"></a>重複使用範本
在教學課程中，您可以建立定義 Data Factory 實體的範本和傳遞參數值的範本。 若要使用相同的範本將 Data Factory 實體部署至不同的環境，您可以針對每個環境建立參數檔案，並在部署到該環境時使用它。     

範例：  

```PowerShell
New-AzureRmResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Dev.json

New-AzureRmResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Test.json

New-AzureRmResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json -TemplateParameterFile ADFTutorialARM-Parameters-Production.json
```
請注意，第一個命令會使用開發環境的參數檔案，第二個會使用測試環境的參數檔案，而第三個會使用生產環境的參數檔案。  

您也可以重複使用範本來執行重複的工作。 例如，使用一個或多個管線建立許多資料處理站，這些管線會實作相同的邏輯，但每個資料處理站會使用不同的 Azure 儲存體帳戶。 在此案例中，您會在具有不同參數檔案的相同環境中 (開發、測試或生產) 使用相同的範本來建立資料處理站。 


## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 瀏覽[教學課程](tutorial-copy-data-dot-net.md)以了解使用 Data Factory 的更多案例。 
