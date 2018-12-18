---
title: 使用 Azure Data Factory 更新 Machine Learning 模型 | Microsoft Docs
description: 說明如何使用 Azure Data Factory 和 Machine Learning 建立預測管線
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: shlo
ms.openlocfilehash: 4eed11b312bce27dc0cd98daa3e2599a28fcabbd
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524425"
---
# <a name="update-azure-machine-learning-models-by-using-update-resource-activity"></a>使用更新資源活動更新 Azure Machine Learning 模型
本文補充主要 Azure Data Factory - Azure Machine Learning 整合文件︰[使用 Azure Machine Learning 和 Azure Data Factory 建立預測管線](transform-data-using-machine-learning.md)。 如果您尚未檢閱主要文件，請在閱讀這篇文章之前先這麼做。 

## <a name="overview"></a>概觀
進行 Azure Machine Learning 模型的實作程序時，需要定型並儲存您的模型。 接著，使用它來建立預測性 Web 服務。 接著才能在網站、儀表板及行動應用程式取用 Web 服務。

您使用 Machine Learning 建立的模型通常不是靜態。 因為當有新資料或 API 取用者有自己的資料時，模型就必須重新定型。 如需如何在 Azure Machine Learning 中重新定型模型的詳細資料，請參閱[重新定型 Machine Learning 模型](../machine-learning/machine-learning-retrain-machine-learning-model.md)。 

重新定型可能經常會發生。 使用批次執行活動與更新資源活動，您可以使用 Data Factory 實作 Azure Machine Learning 模型重新定型，並更新預測 Web 服務。 

下圖描述定型與預測 Web 服務之間的關聯性。 

![Web 服務](./media/update-machine-learning-models/web-services.png)

## <a name="azure-machine-learning-update-resource-activity"></a>Azure Machine Learning 更新資源活動 

下列 JSON 程式碼片段定義 Azure Machine Learning 批次執行活動。

```json
{
    "name": "amlUpdateResource",
    "type": "AzureMLUpdateResource",
    "description": "description",
    "linkedServiceName": {
        "type": "LinkedServiceReference",
        "referenceName": "updatableScoringEndpoint2"
    },
    "typeProperties": {
        "trainedModelName": "ModelName",
        "trainedModelLinkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "StorageLinkedService"
                },
        "trainedModelFilePath": "ilearner file path"
    }
}
```




| 屬性                      | 說明                              | 必要 |
| :---------------------------- | :--------------------------------------- | :------- |
| name                          | 管線中的活動名稱     | 是      |
| 說明                   | 說明活動用途的文字。  | 否       |
| type                          | 對於 Azure Machine Learning 更新資源活動，活動類型為 **AzureMLUpdateResource**。 | 是      |
| 預設容器             | 包含 updateResourceEndpoint 屬性的 Azure Machine Learning 連結服務。 | 是      |
| trainedModelName              | 要更新之 Web 服務實驗中的「定型模型」模組名稱 | 是      |
| trainedModelLinkedServiceName | 具備由更新作業上傳之 ilearner 檔案的 Azure 儲存體連結服務名稱 | 是      |
| trainedModelFilePath          | trainedModelLinkedService 中的相對檔案路徑，表示由更新作業上傳的 ilearner 檔案 | 是      |


## <a name="end-to-end-workflow"></a>端對端工作流程

實作重新定型模組和更新預測 Web 服務的整個程序會叫用下列步驟： 

- 使用**批次執行活動**叫用**定型 Web 服務**。 叫用定型 Web 服務與[使用 Azure Machine Learning 和 Data Factory 批次執行活動建立預測管線](transform-data-using-machine-learning.md)中說明的叫用預測 Web 服務相同。 定型 Web 服務的輸出是 iLearner 檔案，您可以使用該檔案來更新預測 Web 服務。 
- 您可以使用 [更新資源活動] 來叫用**預測 Web 服務**的**更新資源端點**，以將 Web 服務更新為新定型的模型。 

## <a name="azure-machine-learning-linked-service"></a>Azure Machine Learning 連結服務

對於上述的端對端工作流程，您必須建立兩個 Azure Machine Learning 連結服務： 

1. Azure Machine Learning 連結服務至定型 Web 服務，此連結服務以[使用 Azure Machine Learning 和 Data Factory 批次執行活動建立預測管線](transform-data-using-machine-learning.md)中所述的相同方式，由批次執行活動使用。 差別在於定型 Web 服務的輸出是 iLearner 檔案，更新資源活動使用該檔案以更新預測 Web 服務。 
2. Azure Machine Learning 連結服務至預測 Web 服務的更新資源端點。 更新資源活動會使用此連結服務，以利用上述步驟傳回的 iLearner 檔案，更新預測 Web 服務。 

對於第二個 Azure Machine Learning 連結服務，當您的 Azure Machine Learning Web 服務是傳統的 Web 服務或新的 Web 服務時，設定是不同的。 差異會在下列各節分別進行討論。 

## <a name="web-service-is-new-azure-resource-manager-web-service"></a>Web 服務是新的 Azure Resource Manager Web 服務 

如果 Web 服務是會公開 Azure Resource Manager 端點的新 Web 服務類型，您不需要新增第二個「非預設」端點。 連結服務中 **updateResourceEndpoint** 的格式如下︰ 

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview. 
```

在 [Azure Machine Learning Web 服務入口網站](https://services.azureml.net/)上查詢 Web 服務時，您可以取得 URL 中預留位置的值。 

新類型的更新資源端點需要服務主體驗證。 若要使用服務主體驗證，請在 Azure Active Directory (Azure AD) 中註冊應用程式實體，並授與 Web 服務所屬之訂用帳戶或資源群組的**參與者**或**擁有者**角色。 請參閱[如何建立服務主體及指派權限來管理 Azure 資源](../azure-resource-manager/resource-group-create-service-principal-portal.md)。 請記下以下的值，您可以使用這些值來定義連結服務：

- 應用程式識別碼
- 應用程式金鑰 
- 租用戶識別碼

以下是連結服務定義範例︰ 

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000  000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": {
            "type": "SecureString",
            "value": "APIKeyOfEndpoint1"
            },
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview. ",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": {
            "type": "SecureString",
            "value": "servicePrincipalKey"
            },
            "tenant": "mycompany.com"
        }
    }
}
```

下列案例提供更多詳細資料， 其中包含從 Azure Data Factory 管線重新訓練和更新 Azure ML 模型的範例。


## <a name="sample-retraining-and-updating-an-azure-machine-learning-model"></a>範例：重新定型和更新 Azure Machine Learning 模型

本節提供使用 **Azure ML 批次執行活動** 來重新訓練模型的範例管線。 管線也會使用 **Azure ML 更新資源活動** 來更新評分 Web 服務中的模型。 本節還會提供範例中所有連結服務、資料集和管線的 JSON 程式碼片段。

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
下列 JSON 程式碼片段定義 Azure Machine Learnin 連結服務可指向評分 Web 服務的可更新端點。  

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

### <a name="pipeline"></a>管線
管線有兩個活動：**AzureMLBatchExecution** 和 **AzureMLUpdateResource**。 批次執行活動會以定型資料作為輸入，並產生 iLearner 檔案作為輸出。 接著，更新資源活動會採用該 iLearner 檔案來更新預測 Web 服務。 

```JSON
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "amlBEGetilearner",
                "description": "Use AML BES to get the ileaner file from training web service",
                "type": "AzureMLBatchExecution",
                "linkedServiceName": {
                    "referenceName": "trainingEndpoint",
                    "type": "LinkedServiceReference"
                },
                "typeProperties": {
                    "webServiceInputs": {
                        "input1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            }, 
                            "FilePath":"azuremltesting/input"
                        }, 
                        "input2": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference" 
                            }, 
                            "FilePath":"azuremltesting/input"
                        }        
                    },
                    "webServiceOutputs": {
                        "output1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"   
                            }, 
                            "FilePath":"azuremltesting/output"
                        }        
                    }
                }
            },
            {
                "name": "amlUpdateResource",
                "type": "AzureMLUpdateResource",
                "description": "Use AML Update Resource to update the predict web service",
                "linkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "updatableScoringEndpoint2"
                },
                "typeProperties": {
                    "trainedModelName": "ADFV2Sample Model [trained model]",
                    "trainedModelLinkedServiceName": {
                                "type": "LinkedServiceReference",
                                "referenceName": "StorageLinkedService"
                            },
                    "trainedModelFilePath": "azuremltesting/output/newModelForArm.ilearner"
                },
                "dependsOn": [ 
                    { 
                        "activity": "amlbeGetilearner", 
                        "dependencyConditions": [ "Succeeded" ] 
                    }
                 ]

            }
        ]
    }
}
```
## <a name="next-steps"></a>後續步驟
請參閱下列文章，其說明如何以其他方式轉換資料： 

* [U-SQL 活動](transform-data-using-data-lake-analytics.md)
* [Hive 活動](transform-data-using-hadoop-hive.md)
* [Pig 活動](transform-data-using-hadoop-pig.md)
* [MapReduce 活動](transform-data-using-hadoop-map-reduce.md)
* [Hadoop 串流活動](transform-data-using-hadoop-streaming.md)
* [Spark 活動](transform-data-using-spark.md)
* [.NET 自訂活動](transform-data-using-dotnet-custom-activity.md)
* [預存程序活動](transform-data-using-stored-procedure.md)
