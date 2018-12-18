---
title: 使用 Azure Data Factory 建立預測資料管線 | Microsoft Docs
description: 了解如何在 Azure Data Factory 中使用 Azure Machine Learning - 批次執行活動建立預測管線。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: douglasl
ms.openlocfilehash: 87a71cff07d18dde25fa5c58b3718e7a57e3ce8d
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046008"
---
# <a name="create-predictive-pipelines-using-azure-machine-learning-and-azure-data-factory"></a>使用 Azure Machine Learning 和 Azure Data Factory 來建立預測管線
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-azure-ml-batch-execution-activity.md)
> * [目前的版本](transform-data-using-machine-learning.md)

[Azure 機器學習服務](https://azure.microsoft.com/documentation/services/machine-learning/) 可讓您建置、測試以及部署預測性分析解決方案。 從高階觀點而言，由下列三個步驟完成這個動作：

1. **建立訓練實驗**。 您可以使用 Azure ML Studio 來進行此步驟。 ML Studio 是共同作業的視覺化開發環境，可供您使用訓練資料來訓練和測試預測性分析模型。
2. **將其轉換為評分實驗**。 一旦您的模型已使用現有資料訓練，並做好使用該模型為新資料評分的準備之後，您準備並簡化用於評分實驗。
3. **將其部署為 Web 服務**。 只要按一下，您就可以將評分實驗當做 Azure Web 服務發佈。 您可以透過此 Web 服務端點將資料傳送給您的模型，並從模型接收結果預測。  

### <a name="data-factory-and-machine-learning-together"></a>Data Factory 和 Machine Learning 一起合作
Azure Data Factory 可讓您輕鬆地建立管線，使用已發佈的 [Azure Machine Learning][azure-machine-learning] Web 服務進行預測性分析。 在 Azure Data Factory 管線中使用 [批次執行活動]  ，您可以叫用 Azure ML Web 服務以對批次中的資料進行預測。 

經過一段時間，必須使用新的輸入資料集重新訓練 Azure ML 評分實驗中的預測模型。 您可以執行下列步驟，從 Data Factory 管線重新訓練 Azure ML 模型：

1. 將訓練實驗 (而非預設實驗) 發佈為 Web 服務。 在 Azure ML Studio 中進行此步驟的方法，和先前案例中將預測實驗公開為 Web 服務的方法一樣。
2. 使用 Azure ML 批次執行活動，對訓練實驗叫用 Web 服務。 基本上，您可以使用 Azure ML 批次執行活動來叫用訓練 Web 服務和評分 Web 服務。

完成重新訓練之後，您可以使用「Azure ML 更新資源活動」，使用新訓練的模型來更新評分 Web 服務 (以 Web 服務公開的預測實驗)。 如需詳細資料，請參閱[使用更新資源活動更新模型](update-machine-learning-models.md)一文。

## <a name="azure-machine-learning-linked-service"></a>Azure Machine Learning 連結服務

您可建立 **Azure Machine Learning** 連結服務，以將 Azure Machine Learning Web 服務連結至 Azure Data Factory。 Azure Machine Learning 批次執行活動使用的連結活動與[更新資源活動](update-machine-learning-models.md)。 


```JSON
{
    "type" : "linkedServices",
    "name": "AzureMLLinkedService",
    "apiVersion" : "2017-09-01-preview",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "URL to Azure ML Predictive Web Service",
            "apiKey": {
                "type": "SecureString",
                "value": "api key"
            }
        }
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

如需了解 JSON 定義中的屬性說明，請參閱[計算連結服務](compute-linked-services.md)一文。 

Azure Machine Learning 支援傳統 Web 服務和新 Web 服務，以用於您的預測性實驗。 您可以從 Data Factory 選擇正確的服務。 若要取得建立 Azure Machine Learning 連結服務所需的資訊，請移至 https://services.azureml.net，其中列出所有 (新) Web 服務和傳統 Web 服務。 按一下您要存取的 Web 服務，然後按一下 [取用] 頁面。 為 **apiKey** 屬性複製**主索引鍵**，並為 **mlEndpoint** 屬性複製**批次要求**。 

![Azure Machine Learning Web 服務](./media/transform-data-using-machine-learning/web-services.png)

## <a name="azure-machine-learning-batch-execution-activity"></a>Azure Machine Learning 批次執行活動

下列 JSON 程式碼片段定義 Azure Machine Learning 批次執行活動。 活動定義具有您稍早建立之 Azure Machine Learning 連結服務的參考。 

```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "<web service input name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                }, 
                "FilePath":"path1"
            }, 
            "<web service input name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference" 
                }, 
                "FilePath":"path2"
            }        
        },
        "webServiceOutputs": {
            "<web service output name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"   
                }, 
                "FilePath":"path3"
            }, 
            "<web service output name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"   
                }, 
                "FilePath":"path4"
            }         
        },
        "globalParameters": {
            "<Parameter 1 Name>": "<parameter value>",
            "<parameter 2 name>": "<parameter 2 value>"
        }
    }
}
```



| 屬性          | 說明                              | 必要 |
| :---------------- | :--------------------------------------- | :------- |
| name              | 管線中的活動名稱     | yes      |
| 說明       | 說明活動用途的文字。  | 否       |
| type              | 對於 Data Lake Analytics U-SQL 活動，活動類型為 **AzureMLBatchExecution**。 | yes      |
| 預設容器 | 將服務連結至 Azure Machine Learning 連結服務。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。 | yes      |
| webServiceInputs  | 對應 Azure Machine Learning Web 服務輸入之名稱的索引鍵/值組。 索引鍵必須符合已發佈 Azure Machine Learning Web 服務中定義的輸入參數。 值是指定輸入 Blob 位置的 Azure 儲存體連結服務和 FilePath 屬性組。 | 否       |
| webServiceOutputs | 對應 Azure Machine Learning Web 服務輸出之名稱的索引鍵/值組。 索引鍵必須符合已發佈 Azure Machine Learning Web 服務中定義的輸出參數。 值是指定輸出 Blob 位置的 Azure 儲存體連結服務和 FilePath 屬性組。 | 否       |
| globalParameters  | 傳遞至 Azure ML 批次執行服務端點的索引鍵/值組。 索引鍵必須符合已發佈 Azure ML Web 服務中定義的 Web 服務參數名稱。 值會在 Azure ML 批次執行要求的 GlobalParameters 屬性中傳遞 | 否       |

### <a name="scenario-1-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>案例 1：使用 Web 服務輸入/輸出 (參考 Azure Blob 儲存體中的資料) 的實驗

在此案例中，Azure Machine Learning Web 服務會使用 Azure Blob 儲存體中的檔案資料執行預測，並將預測結果儲存在 Blob 儲存體中。 下列 JSON 使用 AzureMLBatchExecution 活動定義 Data Factory 管線。 Azure 部落格儲存體中的輸入和輸出資料使用 LinkedName 和 FilePath 來引用。 範例中，輸入和輸出的連結服務不同，您可以為 Data Factory 的每個輸入/輸出使用不同的連結服務，以便能夠選取正確的檔案，並傳送至 Azure ML Web 服務。 

> [!IMPORTANT]
> 在您的 Azure ML 實驗中，Web 服務輸入和輸出連接埠及全域參數有您可以自訂的預設名稱 ("input1"、"input2")。 您用於 webServiceInputs、webServiceOutputs 及 globalParameters 設定的名稱必須與實驗中的名稱完全相同。 您可以檢視您 Azure ML 端點之 [批次執行說明] 頁面上的範例要求承載，來確認預期的對應。
>
> 

```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "input1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                }, 
                "FilePath":"amltest/input/in1.csv"
            }, 
            "input2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference" 
                }, 
                "FilePath":"amltest/input/in2.csv"
            }        
        },
        "webServiceOutputs": {
            "outputName1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"   
                }, 
                "FilePath":"amltest2/output/out1.csv"
            }, 
            "outputName2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"   
                }, 
                "FilePath":"amltest2/output/out2.csv"
            }         
        }
    }
}
```
### <a name="scenario-2-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>案例 2：使用讀取器/寫入器模組參考各種儲存體資料的實驗
建立 Azure ML 實驗時的另一個常見案例是使用「匯入資料」和「輸出資料」模組。 「匯入資料」模組是用來將資料載入實驗，而「輸出資料」模組則是用於儲存您的實驗資料。 如需「匯入資料」和「輸出資料」模組的詳細資料，請參閱 MSDN 文件庫上的[匯入資料](https://msdn.microsoft.com/library/azure/dn905997.aspx)和[輸出資料](https://msdn.microsoft.com/library/azure/dn905984.aspx)主題。     

使用「匯入資料」和「輸出資料」模組時，較好的做法是針對這些模組的每一個屬性，使用 Web 服務參數。 這些 Web 參數可讓您在執行階段設定值。 例如，您可以透過使用 Azure SQL Database：XXX.database.windows.net 的「匯入資料」模組，建立實驗。 部署 Web 服務之後，您需要啟用 Web 服務的取用者，藉此指定另一個稱為 `YYY.database.windows.net` 的 Azure SQL Server。 您可以使用 Web 服務參數來設定此值。

> [!NOTE]
> Web 服務的輸入和輸出與 Web 服務參數不同。 在第一個案例中，您已了解如何為 Azure ML Web 服務指定輸入和輸出。 在此案例中，您會傳遞 Web 服務的參數，以對應至「匯入資料」/「輸出資料」模組的屬性。
>
> 

讓我們看看使用 Web 服務參數的案例。 您已部署 Azure Machine Learning Web 服務，其使用讀取器模組所讀取的資料，是來自其中一個受 Azure Machine Learning 支援的資料來源 (例如：Azure SQL Database)。 批次執行之後，會使用寫入器模組寫入結果 (Azure SQL Database)。  實驗中沒有定義任何 Web 服務的輸入和輸出。 在此情況下，我們建議您為讀取器和寫入器模組設定相關 Web 服務參數。 此組態允許在使用 AzureMLBatchExecution 活動時設定讀取器/寫入器模組。 如以下活動 JSON 所示，在 **globalParameters** 區段中指定 Web 服務參數。

```JSON
"typeProperties": {
    "globalParameters": {
        "Database server name": "<myserver>.database.windows.net",
        "Database name": "<database>",
        "Server user account name": "<user name>",
        "Server user account password": "<password>"
    }
}
```


> [!NOTE]
> Web 服務參數區分大小寫，因此，請確定您在活動 JSON 中所指定的名稱符合 Web 服務所公開的名稱。
>

完成重新訓練之後，您可以使用「Azure ML 更新資源活動」，使用新訓練的模型來更新評分 Web 服務 (以 Web 服務公開的預測實驗)。 如需詳細資料，請參閱[使用更新資源活動更新模型](update-machine-learning-models.md)一文。



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
