---
title: 建置您的第一個 Data Factory (Resource Manager 範本) | Microsoft Docs
description: 在本教學課程中，您會使用 Azure Resource Manager 範本來建立範例 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: ''
editor: ''
ms.assetid: eb9e70b9-a13a-4a27-8256-2759496be470
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: a04e91cea4ce1670fcc0a7be2d7591d5856b738f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106857"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-azure-resource-manager-template"></a>教學課程：使用 Azure Resource Manager 範本建置您的第一個 Azure Data Factory
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-build-your-first-pipeline.md)
> * [Azure 入口網站](data-factory-build-your-first-pipeline-using-editor.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Resource Manager 範本](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
> 
 
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[快速入門：使用 Azure Data Factory 來建立資料處理站](../quickstart-create-data-factory-dot-net.md)。

在本文中，您會使用 Azure Resource Manager 範本來建立第一個 Azure Data Factory。 若要使用其他工具/SDK 進行本教學課程，請選取下拉式清單的其中一個選項。

本教學課程中的管線有一個活動︰**HDInsight Hive 活動**。 此活動會在 Azure HDInsight 叢集上執行 Hive 指令碼，以轉換輸入資料來產生輸出資料。 管線已排定每個月在指定的開始與結束時間之間執行一次。 

> [!NOTE]
> 本教學課程中的資料管線會轉換輸入資料來產生輸出資料， 如需說明如何使用 Azure Data Factory 複製資料的教學課程，請參閱[教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
> 
> 本教學課程中的管線只有一個 HDInsightHive 類型的活動。 一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱 [Data Factory 排程和執行](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。 

## <a name="prerequisites"></a>必要條件
* 詳讀 [教學課程概觀](data-factory-build-your-first-pipeline.md) 一文並完成 **必要** 步驟。
* 按照 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 一文中的指示，在您的電腦上安裝最新版的 Azure PowerShell。
* 若要了解 Azure Resource Manager 範本，請參閱 [撰寫 Azure Resource Manager 範本](../../azure-resource-manager/resource-group-authoring-templates.md) 。 

## <a name="in-this-tutorial"></a>本教學課程內容
| 實體 | 說明 |
| --- | --- |
| Azure 儲存體連結服務 |將您的 Azure 儲存體帳戶連結至 Data Factory。 Azure 儲存體帳戶會保留此範例中管線的輸入和輸出資料。 |
| HDInsight 隨選連結服務 |將隨選 HDInsight 叢集連結至 Data Factory。 會自動為您建立叢集來處理資料，並在處理完成後將叢集刪除。 |
| Azure Blob 輸入資料集 |是指 Azure 儲存體連結服務。 連結的服務是指 Azure 儲存體帳戶，而 Azure Blob 資料集則會指定保留輸入資料儲存體中的容器、資料夾和檔案名稱。 |
| Azure Blob 輸出資料集 |是指 Azure 儲存體連結服務。 連結的服務是指 Azure 儲存體帳戶，而 Azure Blob 資料集則會指定保留輸出資料儲存體中的容器、資料夾和檔案名稱。 |
| Data Pipeline |管線有一個 HDInsightHive 類型的活動會取用輸入資料集，並產生輸出資料集。 |

資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 兩種活動類型︰[資料移動活動](data-factory-data-movement-activities.md)和[資料轉換活動](data-factory-data-transformation-activities.md)。 在本教學課程中，您可以使用一個活動建立管線 (Hive 活動)。

下節會提供完整的 Resource Manager 範本來定義 Data Factory 實體，如此您可以快速執行教學課程並測試範本。 若要了解每個 Data Factory 實體的定義方式，請參閱[範本中的 Data Factory 實體](#data-factory-entities-in-the-template)一節。

## <a name="data-factory-json-template"></a>Data Factory JSON 範本
用於定義資料處理站的最上層 Resource Manager 範本是︰ 

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { ...
    },
    "variables": { ...
    },
    "resources": [
        {
            "name": "[parameters('dataFactoryName')]",
            "apiVersion": "[variables('apiVersion')]",
            "type": "Microsoft.DataFactory/datafactories",
            "location": "westus",
            "resources": [
                { ... },
                { ... },
                { ... },
                { ... }
            ]
        }
    ]
}
```
在 **C:\ADFGetStarted** 資料夾中，使用下列內容建立名為 **ADFTutorialARM.json** 的 JSON 檔案：

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
        "storageAccountName": { "type": "string", "metadata": { "description": "Name of the Azure storage account that contains the input/output data." } },
          "storageAccountKey": { "type": "securestring", "metadata": { "description": "Key for the Azure storage account." } },
          "blobContainer": { "type": "string", "metadata": { "description": "Name of the blob container in the Azure Storage account." } },
          "inputBlobFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that has the input file." } },
          "inputBlobName": { "type": "string", "metadata": { "description": "Name of the input file/blob." } },
          "outputBlobFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that will hold the transformed data." } },
          "hiveScriptFolder": { "type": "string", "metadata": { "description": "The folder in the blob container that contains the Hive query file." } },
          "hiveScriptFile": { "type": "string", "metadata": { "description": "Name of the hive query (HQL) file." } }
    },
    "variables": {
          "dataFactoryName": "[concat('HiveTransformDF', uniqueString(resourceGroup().id))]",
          "azureStorageLinkedServiceName": "AzureStorageLinkedService",
          "hdInsightOnDemandLinkedServiceName": "HDInsightOnDemandLinkedService",
          "blobInputDatasetName": "AzureBlobInput",
          "blobOutputDatasetName": "AzureBlobOutput",
          "pipelineName": "HiveTransformPipeline"
    },
    "resources": [
      {
        "name": "[variables('dataFactoryName')]",
        "apiVersion": "2015-10-01",
        "type": "Microsoft.DataFactory/datafactories",
        "location": "West US",
        "resources": [
          {
            "type": "linkedservices",
            "name": "[variables('azureStorageLinkedServiceName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureStorage",
                  "description": "Azure Storage linked service",
                  "typeProperties": {
                    "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
                  }
            }
          },
          {
            "type": "linkedservices",
            "name": "[variables('hdInsightOnDemandLinkedServiceName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "HDInsightOnDemand",
                  "typeProperties": {
                    "version": "3.5",
                    "clusterSize": 1,
                    "timeToLive": "00:05:00",
                    "osType": "Linux",
                    "linkedServiceName": "[variables('azureStorageLinkedServiceName')]"
                  }
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobInputDatasetName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureBlob",
                  "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
                  "typeProperties": {
                    "fileName": "[parameters('inputBlobName')]",
                    "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'))]",
                    "format": {
                          "type": "TextFormat",
                          "columnDelimiter": ","
                    }
                  },
                  "availability": {
                    "frequency": "Month",
                    "interval": 1
                  },
                  "external": true
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobOutputDatasetName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "type": "AzureBlob",
                  "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
                  "typeProperties": {
                    "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'))]",
                    "format": {
                          "type": "TextFormat",
                          "columnDelimiter": ","
                    }
                  },
                  "availability": {
                    "frequency": "Month",
                    "interval": 1
                  }
            }
          },
          {
            "type": "datapipelines",
            "name": "[variables('pipelineName')]",
            "dependsOn": [
                  "[variables('dataFactoryName')]",
                  "[variables('azureStorageLinkedServiceName')]",
                  "[variables('hdInsightOnDemandLinkedServiceName')]",
                  "[variables('blobInputDatasetName')]",
                  "[variables('blobOutputDatasetName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
                  "description": "Pipeline that transforms data using Hive script.",
                  "activities": [
                {
                      "type": "HDInsightHive",
                      "typeProperties": {
                        "scriptPath": "[concat(parameters('blobContainer'), '/', parameters('hiveScriptFolder'), '/', parameters('hiveScriptFile'))]",
                        "scriptLinkedService": "[variables('azureStorageLinkedServiceName')]",
                        "defines": {
                              "inputtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('inputBlobFolder'))]",
                              "partitionedtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('outputBlobFolder'))]"
                        }
                      },
                      "inputs": [
                        {
                              "name": "[variables('blobInputDatasetName')]"
                        }
                      ],
                      "outputs": [
                        {
                              "name": "[variables('blobOutputDatasetName')]"
                        }
                      ],
                      "policy": {
                        "concurrency": 1,
                        "retry": 3
                      },
                      "scheduler": {
                        "frequency": "Month",
                        "interval": 1
                      },
                      "name": "RunSampleHiveActivity",
                      "linkedServiceName": "[variables('hdInsightOnDemandLinkedServiceName')]"
                }
                  ],
                  "start": "2017-07-01T00:00:00Z",
                  "end": "2017-07-02T00:00:00Z",
                  "isPaused": false
              }
          }
        ]
      }
    ]
}
```

> [!NOTE]
> 您可以找到另一個 Resource Manager 範本的範例，以在[教學課程︰使用 Azure Resource Manager 範本建立管線與複製活動](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)上建立 Azure Data Factory。  
> 
> 

## <a name="parameters-json"></a>參數 JSON
建立名為 **ADFTutorialARM Parameters.json** 的 JSON 檔案，其中包含 Azure Resource Manager 範本的參數。  

> [!IMPORTANT]
> 針對此參數檔案中的 **storageAccountName** 和 **storageAccountKey** 參數指定您 Azure 儲存體帳戶的名稱和金鑰。 
> 
> 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "value": "<Name of your Azure Storage account>"
        },
        "storageAccountKey": {
            "value": "<Key of your Azure Storage account>"
        },
        "blobContainer": {
            "value": "adfgetstarted"
        },
        "inputBlobFolder": {
            "value": "inputdata"
        },
        "inputBlobName": {
            "value": "input.log"
        },
        "outputBlobFolder": {
            "value": "partitioneddata"
        },
        "hiveScriptFolder": {
              "value": "script"
        },
        "hiveScriptFile": {
              "value": "partitionweblogs.hql"
        }
    }
}
```

> [!IMPORTANT]
> 您可能需要不同的參數 JSON 檔案，以供可與相同 Data Factory JSON 範本共同使用的開發、測試和生產環境使用。 使用 Power Shell 指令碼，您可以在這些環境中自動部署 Data Factory 實體。 
> 
> 

## <a name="create-data-factory"></a>建立資料處理站
1. 啟動 **Azure PowerShell** 並執行下列命令： 
   * 執行下列命令並輸入您用來登入 Azure 入口網站的使用者名稱和密碼。
    ```PowerShell
    Connect-AzureRmAccount
    ```  
   * 執行下列命令以檢視此帳戶的所有訂用帳戶。
    ```PowerShell
    Get-AzureRmSubscription
    ``` 
   * 執行下列命令以選取您要使用的訂用帳戶。 此訂用帳戶應該與您在 Azure 入口網站中使用的相同。
    ```
    Get-AzureRmSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzureRmContext
    ```   
2. 執行下列命令，使用您在步驟 1 中建立的 Resource Manager 範本來部署 Data Factory 實體。 

    ```PowerShell
    New-AzureRmResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFGetStarted\ADFTutorialARM.json -TemplateParameterFile C:\ADFGetStarted\ADFTutorialARM-Parameters.json
    ```

## <a name="monitor-pipeline"></a>監視管線
1. 登入 [Azure 入口網站](https://portal.azure.com/)後，按一下 [瀏覽]，然後選取 [Data Factory]。
     ![瀏覽 Data Factory](./media/data-factory-build-your-first-pipeline-using-arm/BrowseDataFactories.png)
2. 在 [Data Factory] 刀鋒視窗中，按一下您建立的 Data Factory (**TutorialFactoryARM**)。    
3. 在您 Data Factory 的 [Data Factory] 刀鋒視窗中，按一下 [圖表]。

     ![圖表磚](./media/data-factory-build-your-first-pipeline-using-arm/DiagramTile.png)
4. 在 [圖表檢視] 中，您會看到管線的概觀，以及在本教學課程中使用的資料集。
   
   ![圖表檢視](./media/data-factory-build-your-first-pipeline-using-arm/DiagramView.png) 
5. 在 [圖表檢視] 中，按兩下 **AzureBlobOutput**資料集。 您會看到目前正在處理的配量。
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-arm/AzureBlobOutput.png)
6. 處理完成時，您會看到配量處於 [就緒]  狀態。 建立隨選 HDInsight 叢集通常需要一些時間 (大約 20 分鐘)。 因此，管線預計需要 **大約 30 分鐘** 的時間來處理配量。
   
    ![Dataset](./media/data-factory-build-your-first-pipeline-using-arm/SliceReady.png)    
7. 當配量處於 [就緒] 狀態時，檢查您 Blob 儲存體中 **adfgetstarted** 容器內 **partitioneddata** 資料夾的輸出資料。  

如需如何使用 Azure 入口網站刀鋒視窗來監視您在本教學課程中建立的管線和資料集的指示，請參閱 [監視資料集和管線](data-factory-monitor-manage-pipelines.md) 。

您也可以使用「監視及管理應用程式」來監視您的資料管線。 如需有關使用應用程式的詳細資訊，請參閱 [使用監視應用程式來監視和管理 Azure Data Factory 管線](data-factory-monitor-manage-app.md) 。 

> [!IMPORTANT]
> 配量處理成功時就會刪除輸入檔案。 因此，如果您想要重新執行配量或再次進行本教學課程，請將輸入檔案 (input.log) 上傳至 adfgetstarted 容器的 inputdata 資料夾。
> 
> 

## <a name="data-factory-entities-in-the-template"></a>範本中的 Data Factory 實體
### <a name="define-data-factory"></a>定義資料處理站
您可以在 Resource Manager 範本中定義資料處理站，如下列範例所示︰  

```json
"resources": [
{
    "name": "[variables('dataFactoryName')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "West US"
}
```
dataFactoryName 定義為： 

```json
"dataFactoryName": "[concat('HiveTransformDF', uniqueString(resourceGroup().id))]",
```
根據資源群組識別碼，它是唯一的字串。  

### <a name="defining-data-factory-entities"></a>定義 Data Factory 實體
下列的 Data Factory 實體定義於 JSON 範本中︰ 

* [Azure 儲存體連結服務](#azure-storage-linked-service)
* [HDInsight 隨選連結服務](#hdinsight-on-demand-linked-service)
* [Azure Blob 輸入資料集](#azure-blob-input-dataset)
* [Azure Blob 輸出資料集](#azure-blob-output-dataset)
* [具有複製活動的管線](#data-pipeline)

#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
在此區段中指定您 Azure 儲存體帳戶的名稱和金鑰。 如需用來定義 Azure 儲存體連結服務之 JSON 屬性的詳細資料，請參閱 [Azure 儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)。 

```json
{
    "type": "linkedservices",
    "name": "[variables('azureStorageLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureStorage",
        "description": "Azure Storage linked service",
        "typeProperties": {
            "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
        }
    }
}
```
**connectionString** 會使用 storageAccountName 和 storageAccountKey 參數。 使用組態檔傳遞這些參數的值。 定義也會使用在範本中定義的變數︰azureStroageLinkedService 和 dataFactoryName。 

#### <a name="hdinsight-on-demand-linked-service"></a>HDInsight 隨選連結服務
請參閱[計算連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)文章，以取得關於用來定義 HDInsight 隨選連結服務之 JSON 屬性的詳細資訊。  

```json
{
    "type": "linkedservices",
    "name": "[variables('hdInsightOnDemandLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "[variables('azureStorageLinkedServiceName')]"
        }
    }
}
```
請注意下列幾點： 

* Data Factory 會使用上述 JSON，為您建立**以 Linux 為基礎的** HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。 
* 您可以使用 **自己的 HDInsight 叢集** ，不必使用隨選的 HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) 。
* HDInsight 叢集會在您於 JSON 中指定的 Blob 儲存體 (**linkedServiceName**) 建立**預設容器**。 HDInsight 不會在刪除叢集時刪除此容器。 這是設計的行為。 在使用 HDInsight 隨選連結服務時，除非有現有的即時叢集 (**timeToLive**)，否則每當需要處理配量時，就會建立 HDInsight 叢集，並在處理完成時予以刪除。
  
    隨著處理的配量越來越多，您會在 Azure Blob 儲存體中看到許多容器。 如果在疑難排解作業時不需要這些容器，建議您加以刪除以降低儲存成本。 這些容器的名稱遵循下列模式："adf**yourdatafactoryname**-**linkedservicename**-datetimestamp"。 請使用 [Microsoft 儲存體總管](http://storageexplorer.com/) 之類的工具刪除 Azure Blob 儲存體中的容器。

如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。

#### <a name="azure-blob-input-dataset"></a>Azure Blob 輸入資料集
您可以指定 blob 容器、資料夾和包含輸入資料之檔案的名稱。 請參閱 [Azure Blob 資料集屬性](data-factory-azure-blob-connector.md#dataset-properties)，以取得用來定義 Azure Blob 資料集之 JSON 屬性的詳細資訊。 

```json
{
    "type": "datasets",
    "name": "[variables('blobInputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "typeProperties": {
            "fileName": "[parameters('inputBlobName')]",
            "folderPath": "[concat(parameters('blobContainer'), '/', parameters('inputBlobFolder'))]",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true
    }
}
```
這個定義會使用下列參數範本中定義的參數︰blobContainer、inputBlobFolder 和 inputBlobName。 

#### <a name="azure-blob-output-dataset"></a>Azure Blob 輸出資料集
您可以指定 blob 容器和包含輸出資料之資料夾的名稱。 請參閱 [Azure Blob 資料集屬性](data-factory-azure-blob-connector.md#dataset-properties)，以取得用來定義 Azure Blob 資料集之 JSON 屬性的詳細資訊。  

```json
{
    "type": "datasets",
    "name": "[variables('blobOutputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "typeProperties": {
            "folderPath": "[concat(parameters('blobContainer'), '/', parameters('outputBlobFolder'))]",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        }
    }
}
```

這個定義會使用下列參數範本中定義的參數︰blobContainer 和 outputBlobFolder。 

#### <a name="data-pipeline"></a>Data Pipeline
您可以定義在隨選 Azure HDInsight 叢集上執行 Hive 指令碼以轉換資料的管線。 請參閱[管線 JSON](data-factory-create-pipelines.md#pipeline-json)，以取得用來在此範例中定義管線的 JSON 元素之描述。 

```json
{
    "type": "datapipelines",
    "name": "[variables('pipelineName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
        "[variables('azureStorageLinkedServiceName')]",
        "[variables('hdInsightOnDemandLinkedServiceName')]",
        "[variables('blobInputDatasetName')]",
        "[variables('blobOutputDatasetName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "description": "Pipeline that transforms data using Hive script.",
        "activities": [
        {
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "[concat(parameters('blobContainer'), '/', parameters('hiveScriptFolder'), '/', parameters('hiveScriptFile'))]",
                "scriptLinkedService": "[variables('azureStorageLinkedServiceName')]",
                "defines": {
                    "inputtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('inputBlobFolder'))]",
                    "partitionedtable": "[concat('wasb://', parameters('blobContainer'), '@', parameters('storageAccountName'), '.blob.core.windows.net/', parameters('outputBlobFolder'))]"
                }
            },
            "inputs": [
            {
                "name": "[variables('blobInputDatasetName')]"
            }
            ],
            "outputs": [
            {
                "name": "[variables('blobOutputDatasetName')]"
            }
            ],
            "policy": {
                "concurrency": 1,
                "retry": 3
            },
            "scheduler": {
                "frequency": "Month",
                "interval": 1
            },
            "name": "RunSampleHiveActivity",
            "linkedServiceName": "[variables('hdInsightOnDemandLinkedServiceName')]"
        }
        ],
        "start": "2017-07-01T00:00:00Z",
        "end": "2017-07-02T00:00:00Z",
        "isPaused": false
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

您也可以重複使用範本來執行重複的工作。 例如，您需要使用一個或多個管線建立許多資料處理站，這些管線會實作相同的邏輯，但每個資料處理站會使用不同的 Azure 儲存體和 Azure SQL Database 帳戶。 在此案例中，您會在具有不同參數檔案的相同環境中 (開發、測試或生產) 使用相同的範本來建立資料處理站。 

## <a name="resource-manager-template-for-creating-a-gateway"></a>可供建立閘道的 Resource Manager 範本
以下是可供在後端建立邏輯閘道的範例 Resource Manager 範本。 在內部部署電腦或 Azure IaaS VM 上安裝閘道，並使用金鑰向 Data Factory 服務註冊此閘道。 如需詳細資訊，請參閱 [在內部部署和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 。

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
    },
    "variables": {
        "dataFactoryName":  "GatewayUsingArmDF",
        "apiVersion": "2015-10-01",
        "singleQuote": "'"
    },
    "resources": [
        {
            "name": "[variables('dataFactoryName')]",
            "apiVersion": "[variables('apiVersion')]",
            "type": "Microsoft.DataFactory/datafactories",
            "location": "eastus",
            "resources": [
                {
                    "dependsOn": [ "[concat('Microsoft.DataFactory/dataFactories/', variables('dataFactoryName'))]" ],
                    "type": "gateways",
                    "apiVersion": "[variables('apiVersion')]",
                    "name": "GatewayUsingARM",
                    "properties": {
                        "description": "my gateway"
                    }
                }            
            ]
        }
    ]
}
```
此範本會利用名為 GatewayUsingARM 的閘道建立名為 GatewayUsingArmDF 的 Data Factory。 

## <a name="see-also"></a>另請參閱
| 話題 | 說明 |
|:--- |:--- |
| [管線](data-factory-create-pipelines.md) |本文協助您了解 Azure Data Factory 中的管線和活動，以及如何使用這些來為您的案例或業務建構端對端的資料導向工作流程。 |
| [資料集](data-factory-create-datasets.md) |本文協助您了解 Azure Data Factory 中的資料集。 |
| [排程和執行](data-factory-scheduling-and-execution.md) |本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 |
| [使用監視應用程式來監視和管理管線](data-factory-monitor-manage-app.md) |本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 |

