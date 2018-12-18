---
title: 使用 Azure Data Factory 更新 Machine Learning 模型 | Microsoft Docs
description: 說明如何使用 Azure Data Factory 和 Azure Machine Learning 建立預測管線
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3eb9f765b5ffe1118b3b25aff8ecdde327021a54
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046471"
---
# <a name="updating-azure-machine-learning-models-using-update-resource-activity"></a>使用更新資源活動更新 Azure Machine Learning 模型

> [!div class="op_single_selector" title1="Transformation Activities"]
> * [Hive 活動](data-factory-hive-activity.md) 
> * [Pig 活動](data-factory-pig-activity.md)
> * [MapReduce 活動](data-factory-map-reduce.md)
> * [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)
> * [Spark 活動](data-factory-spark.md)
> * [Machine Learning Batch 執行活動](data-factory-azure-ml-batch-execution-activity.md)
> * [Machine Learning 更新資源活動](data-factory-azure-ml-update-resource-activity.md)
> * [預存程序活動](data-factory-stored-proc-activity.md)
> * [Data Lake Analytics U-SQL 活動](data-factory-usql-activity.md)
> * [.NET 自訂活動](data-factory-use-custom-activities.md)


> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中更新機器學習模型](../update-machine-learning-models.md)。

本文補充主要 Azure Data Factory - Azure Machine Learning 整合文件︰[使用 Azure Machine Learning 和 Azure Data Factory 建立預測管線](data-factory-azure-ml-batch-execution-activity.md)。 如果您尚未檢閱主要文件，請在閱讀這篇文章之前先這麼做。 

## <a name="overview"></a>概觀
經過一段時間，必須使用新的輸入資料集重新訓練 Azure ML 評分實驗中的預測模型。 完成重新訓練之後，您想要使用已重新訓練的 ML 模型來更新評分 Web 服務。 透過 Web 服務啟用重新訓練和更新 Azure ML 模型的一般步驟如下：

1. 在 [Azure ML Studio](https://studio.azureml.net)中建立實驗。
2. 當您對模型感到滿意時，請使用 Azure ML Studio 對**訓練實驗**和評分/**預測實驗**發佈 Web 服務。

下表說明本範例中使用的 Web 服務。  如需詳細資訊，請參閱 [以程式設計方式重新定型機器學習服務模型](../../machine-learning/machine-learning-retrain-models-programmatically.md) 。

- **訓練 Web 服務** - 接收訓練資料並產生已訓練的模型。 重新訓練的輸出是 Azure Blob 儲存體中的 .ilearner 檔案。 當您將訓練實驗發佈為 Web 服務時，系統會自動為您建立 **預設端點** 。 您可以建立多個端點，但此範例僅使用預設端點。
- **評分 Web 服務** - 接收未標記的資料範例並進行預測。 預測的輸出可能有各種形式，例如 .csv 檔案或 Azure SQL Database 中的資料列 (視實驗的組態而定)。 當您將預測實驗發佈為 Web 服務時，系統會自動為您建立預設端點。 

下圖描述 Azure ML 中訓練與評分端點之間的關聯性。

![Web 服務](./media/data-factory-azure-ml-batch-execution-activity/web-services.png)

您可以使用 [AzureML Batch 執行活動] 來叫用**訓練 Web 服務**。 叫用訓練 Web 服務與叫用 Azure ML Web 服務 (評分 Web 服務) 以便進行資料評分相同。 上述各節詳細說明如何從 Azure Data Factory 管線叫用 Azure ML Web 服務。 

您可以使用 [Azure ML 更新資源活動] 來叫用**評分 Web 服務**，進而以新訓練的模型更新 Web 服務。 下列範例提供連結的服務定義︰ 

## <a name="scoring-web-service-is-a-classic-web-service"></a>評分 Web 服務是傳統的 Web 服務
如果評分 Web 服務是**傳統 Web 服務**，請使用 Azure 入口網站來建立第二個**非預設且可更新的端點**。 如需相關步驟，請參閱[建立端點](../../machine-learning/machine-learning-create-endpoint.md)一文。 建立非預設的可更新端點之後，執行下列步驟：

* 按一下 [批次執行] 以取得 **mlEndpoint** JSON 屬性的 URI 值。
* 按一下 [更新資源] 連結以取得 **updateResourceEndpoint** JSON 屬性的 URI 值。 API 金鑰本身位於端點頁面 (位於右下角)。

![可更新端點](./media/data-factory-azure-ml-batch-execution-activity/updatable-endpoint.png)

下列範例提供 AzureML 連結服務的範例 JSON 定義。 連結服務使用 apiKey 進行驗證。  

```json
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--scoring experiment--/jobs",
            "apiKey": "endpoint2Key",
            "updateResourceEndpoint": "https://management.azureml.net/workspaces/xxx/webservices/--scoring experiment--/endpoints/endpoint2"
        }
    }
}
```

## <a name="scoring-web-service-is-azure-resource-manager-web-service"></a>評分 Web 服務是 Azure Resource Manager Web 服務 
如果 Web 服務是會公開 Azure Resource Manager 端點的新 Web 服務類型，您不需要新增第二個「非預設」端點。 連結服務中 **updateResourceEndpoint** 的格式如下︰ 

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview. 
```

在 [Azure Machine Learning Web 服務入口網站](https://services.azureml.net/)上查詢 Web 服務時，您可以取得 URL 中預留位置的值。 新的更新資源端點類型需要 AAD (Azure Active Directory) 權杖。 在 AzureML 連結服務中指定 **servicePrincipalId** 和 **servicePrincipalKey**。 請參閱[如何建立服務主體及指派權限來管理 Azure 資源](../../azure-resource-manager/resource-group-create-service-principal-portal.md)。 以下是 AzureML 連結服務定義範例︰ 

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": "xxxxxxxxxxxx",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": "xxxxx",
            "tenant": "mycompany.com"
        }
    }
}
```

下列案例提供更多詳細資料， 其中包含從 Azure Data Factory 管線重新訓練和更新 Azure ML 模型的範例。

## <a name="scenario-retraining-and-updating-an-azure-ml-model"></a>案例：重新訓練和更新 Azure ML 模型
本節提供使用 **Azure ML 批次執行活動** 來重新訓練模型的範例管線。 管線也會使用 **Azure ML 更新資源活動** 來更新評分 Web 服務中的模型。 本節還會提供範例中所有連結服務、資料集和管線的 JSON 程式碼片段。

以下是範例管線的圖表檢視。 如您所見，Azure ML 批次執行活動會採用訓練輸入並產生訓練輸出 (iLearner 檔案)。 Azure ML 更新資源活動會採用此訓練輸出，並更新評分 Web 服務端點中的模型。 更新資源活動不會產生任何輸出。 PlaceholderBlob 只是 Azure Data Factory 服務執行管線所需的虛擬輸出資料集而已。

![管線圖](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

### <a name="azure-blob-storage-linked-service"></a>Azure Blob 儲存體連結服務：
Azure 儲存體會保留下列資料：

* 訓練資料。 Azure ML 訓練 Web 服務的輸入資料。  
* iLearner 檔案。 Azure ML 訓練 Web 服務的輸出。 此檔案也是更新資源活動的輸入。  

以下是連結服務的範例 JSON 定義：

```JSON
{
    "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=name;AccountKey=key"
        }
    }
}
```

### <a name="training-input-dataset"></a>訓練輸入資料集：
下列資料集代表 Azure ML 訓練 Web 服務的輸入訓練資料。 Azure ML 批次執行活動會以此資料集做為輸入。

```JSON
{
    "name": "trainingData",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "labeledexamples",
            "fileName": "labeledexamples.arff",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
            "interval": 1
        },
        "policy": {          
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

### <a name="training-output-dataset"></a>訓練輸出資料集：
下列資料集代表 Azure ML 訓練 Web 服務的輸出入 iLearner 檔案。 Azure ML 批次執行活動會產生此資料集。 此資料集同時也是 Azure ML 更新資源活動的輸入。

```JSON
{
    "name": "trainedModelBlob",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "trainingoutput",
            "fileName": "model.ilearner",
            "format": {
                "type": "TextFormat"
            }
        },
        "availability": {
            "frequency": "Week",
            "interval": 1
        }
    }
}
```

### <a name="linked-service-for-azure-ml-training-endpoint"></a>Azure ML 訓練端點的連結服務
下列 JSON 程式碼片段定義的 Azure 機器學習連結服務可指向訓練 Web 服務的預設端點。

```JSON
{    
    "name": "trainingEndpoint",
      "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--training experiment--/jobs",
              "apiKey": "myKey"
        }
      }
}
```

在 [Azure ML Studio] 中，依下列方式取得 **mlEndpoint** 和 **apiKey** 的值：

1. 按一下左功能表中的 [ **Web 服務** ]。
2. 按一下 Web 服務清單中的 **訓練 Web 服務** 。
3. 按一下 [API 金鑰]  文字方塊旁的 [複製]。 將剪貼簿中的金鑰貼到 Data Factory JSON 編輯器中。
4. 在 **Azure ML Studio** 中按一下 [批次執行] 連結。
5. 從 [要求] 區段複製 [要求 URI] 並將它貼到 Data Factory JSON 編輯器中。   

### <a name="linked-service-for-azure-ml-updatable-scoring-endpoint"></a>Azure ML 可更新評分端點的連結服務：
下列 JSON 程式碼片段定義的 Azure 機器學習連結服務可指向評分 Web 服務的非預設可更新端點。  

```JSON
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/00000000eb0abe4d6bbb1d7886062747d7/services/00000000026734a5889e02fbb1f65cefd/jobs?api-version=2.0",
            "apiKey": "sooooooooooh3WvG1hBfKS2BNNcfwSO7hhY6dY98noLfOdqQydYDIXyf2KoIaN3JpALu/AKtflHWMOCuicm/Q==",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "fe200044-c008-4008-a005-94000000731",
            "servicePrincipalKey": "zWa0000000000Tp6FjtZOspK/WMA2tQ08c8U+gZRBlw=",
            "tenant": "mycompany.com"
        }
    }
}
```

### <a name="placeholder-output-dataset"></a>預留位置輸出資料集：
Azure ML 更新資源活動不會產生任何輸出。 不過，Azure Data Factory 需要輸出資料集來驅動管線的排程。 因此，我們在此範例中使用虛擬/預留位置資料集。  

```JSON
{
    "name": "placeholderBlob",
    "properties": {
        "availability": {
            "frequency": "Week",
            "interval": 1
        },
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "any",
            "format": {
                "type": "TextFormat"
            }
        }
    }
}
```

### <a name="pipeline"></a>管線
管線有兩個活動：**AzureMLBatchExecution** 和 **AzureMLUpdateResource**。 Azure ML 批次執行活動會以訓練資料做為輸入並產生 iLearner 檔案做為輸出。 此活動會使用輸入訓練資料叫用訓練 Web 服務 (公開為 Web 服務的訓練實驗)，並從 Web 服務接收 iLearner 檔案。 PlaceholderBlob 只是 Azure Data Factory 服務執行管線所需的虛擬輸出資料集而已。

![管線圖](./media/data-factory-azure-ml-batch-execution-activity/update-activity-pipeline-diagram.png)

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }              
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "Training Exp for ADF ML [trained model]",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "outputs": [
                    {
                        "name": "placeholderBlob"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00Z",
           "end": "2016-02-14T00:00:00Z"
    }
}
```
