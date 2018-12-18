---
title: 使用 Azure Data Factory 建立預測資料管線 | Microsoft Docs
description: 說明如何使用 Azure Data Factory 和 Azure Machine Learning 建立預測管線
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 4fad8445-4e96-4ce0-aa23-9b88e5ec1965
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 62712a201e8f9717f3e0215c4c29130d3bddcdf2
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37050206"
---
# <a name="create-predictive-pipelines-using-azure-machine-learning-and-azure-data-factory"></a>使用 Azure Machine Learning 和 Azure Data Factory 來建立預測管線

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

## <a name="introduction"></a>簡介
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中使用機器學習來轉換資料](../transform-data-using-machine-learning.md)。


### <a name="azure-machine-learning"></a>Azure Machine Learning
[Azure 機器學習服務](https://azure.microsoft.com/documentation/services/machine-learning/) 可讓您建置、測試以及部署預測性分析解決方案。 從高階觀點而言，由下列三個步驟完成這個動作：

1. **建立訓練實驗**。 您可以使用 Azure ML Studio 來進行此步驟。 ML Studio 是共同作業的視覺化開發環境，可供您使用訓練資料來訓練和測試預測性分析模型。
2. **將其轉換為評分實驗**。 一旦您的模型已使用現有資料訓練，並做好使用該模型為新資料評分的準備之後，您準備並簡化用於評分實驗。
3. **將其部署為 Web 服務**。 只要按一下，您就可以將評分實驗當做 Azure Web 服務發佈。 您可以透過此 Web 服務端點將資料傳送給您的模型，並從模型接收結果預測。  

### <a name="azure-data-factory"></a>Azure Data Factory
Data Factory 是雲端架構資料整合服務，用來協調以及自動**移動**和**轉換**資料。 您可以使用 Azure Data Factory 建立資料整合方案，以從各種資料存放區內嵌資料、轉換/處理資料，並將結果資料發佈至資料存放區。

Data Factory 服務可讓您建立資料管線，以移動和轉換資料，然後依指定的排程 (每小時、每天、每週等) 執行管線。 它也提供豐富的視覺效果來顯示資料管線之間的歷程和相依性，並可透過單一整合檢視監視您所有的資料管線，以輕鬆地找出問題並設定監視警示。

請參閱 [Azure Data Factory 簡介](data-factory-introduction.md)和[建置您的第一個管線](data-factory-build-your-first-pipeline.md)文章，快速地開始使用 Azure Data Factory 服務。

### <a name="data-factory-and-machine-learning-together"></a>Data Factory 和 Machine Learning 一起合作
Azure Data Factory 可讓您輕鬆地建立管線，使用已發佈的 [Azure Machine Learning][azure-machine-learning] Web 服務進行預測性分析。 在 Azure Data Factory 管線中使用 [批次執行活動]  ，您可以叫用 Azure ML Web 服務以對批次中的資料進行預測。 如需詳細資訊，請參閱 [使用批次執行活動叫用 Azure ML Web 服務](#invoking-an-azure-ml-web-service-using-the-batch-execution-activity) 一節。

經過一段時間，必須使用新的輸入資料集重新訓練 Azure ML 評分實驗中的預測模型。 您可以執行下列步驟，從 Data Factory 管線重新訓練 Azure ML 模型：

1. 將訓練實驗 (而非預設實驗) 發佈為 Web 服務。 在 Azure ML Studio 中進行此步驟的方法，和先前案例中將預測實驗公開為 Web 服務的方法一樣。
2. 使用 Azure ML 批次執行活動，對訓練實驗叫用 Web 服務。 基本上，您可以使用 Azure ML 批次執行活動來叫用訓練 Web 服務和評分 Web 服務。

完成重新訓練之後，您可以使用「Azure ML 更新資源活動」，使用新訓練的模型來更新評分 Web 服務 (以 Web 服務公開的預測實驗)。 如需詳細資料，請參閱[使用更新資源活動更新模型](data-factory-azure-ml-update-resource-activity.md)一文。

## <a name="invoking-a-web-service-using-batch-execution-activity"></a>使用批次執行活動來叫用 Web 服務
您可以使用 Azure Data Factory 協調資料的移動和處理作業，然後使用 Azure Machine Learning 批次執行。 以下是最上層的步驟︰

1. 建立 Azure Machine Learning 連結服務。 您需要下列值︰

   1. **要求 URI** 。 您可以按一下 Web 服務頁面中的 **批次執行** 連結，即可找到要求 URI。
   2. **API 金鑰** 。 按一下已發佈的 Web 服務，即可找到 API 金鑰。
   3. 使用 **AzureMLBatchExecution** 活動。

      ![機器學習服務儀表板](./media/data-factory-azure-ml-batch-execution-activity/AzureMLDashboard.png)

      ![批次 URI](./media/data-factory-azure-ml-batch-execution-activity/batch-uri.png)

### <a name="scenario-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>案例：使用 Web 服務輸入/輸出 (參考 Azure Blob 儲存體中的資料) 的實驗
在此案例中，Azure Machine Learning Web 服務會使用 Azure Blob 儲存體中的檔案資料執行預測，並將預測結果儲存在 Blob 儲存體中。 下列 JSON 使用 AzureMLBatchExecution 活動定義 Data Factory 管線。 此活動以 **DecisionTreeInputBlob** 資料集做為輸入，以 **DecisionTreeResultBlob** 資料集做為輸出。 **DecisionTreeInputBlob** 會做為輸入以使用 **webServiceInput** JSON 屬性傳遞至 Web 服務。 **DecisionTreeResultBlob** 會做為輸出以使用 **webServiceOutputs** JSON 屬性傳遞至 Web 服務。  

> [!IMPORTANT]
> 如果 Web 服務接受多個輸入，請使用 **webServiceInputs** 屬性，而不要使用 **webServiceInput**。 如需如何使用 webServiceInputs 屬性的範例，請參閱 [Web 服務需要多個輸入](#web-service-requires-multiple-inputs) 一節。
>
> **webServiceInput**/**webServiceInputs** 和 **webServiceOutputs** 屬性 (位於 **typeProperties** 中) 所參考的資料集必須也包含在活動的 **inputs** 和 **outputs** 中。
>
> 在您的 Azure ML 實驗中，Web 服務輸入和輸出連接埠及全域參數有您可以自訂的預設名稱 ("input1"、"input2")。 您用於 webServiceInputs、webServiceOutputs 及 globalParameters 設定的名稱必須與實驗中的名稱完全相同。 您可以檢視您 Azure ML 端點之 [批次執行說明] 頁面上的範例要求承載，來確認預期的對應。
>
>

```JSON
{
  "name": "PredictivePipeline",
  "properties": {
    "description": "use AzureML model",
    "activities": [
      {
        "name": "MLActivity",
        "type": "AzureMLBatchExecution",
        "description": "prediction analysis on batch input",
        "inputs": [
          {
            "name": "DecisionTreeInputBlob"
          }
        ],
        "outputs": [
          {
            "name": "DecisionTreeResultBlob"
          }
        ],
        "linkedServiceName": "MyAzureMLLinkedService",
        "typeProperties":
        {
            "webServiceInput": "DecisionTreeInputBlob",
            "webServiceOutputs": {
                "output1": "DecisionTreeResultBlob"
            }                
        },
        "policy": {
          "concurrency": 3,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        }
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```
> [!NOTE]
> 只有當輸入及輸出屬於 AzureMLBatchExecution 活動時，才可以當做參數傳遞至 Web 服務。 例如，在上面的 JSON 片段中，DecisionTreeInputBlob 是 AzureMLBatchExecution 活動的輸入，其透過 webServiceInput 參數傳遞至 Web 服務做為輸入。   
>
>

### <a name="example"></a>範例
此範例使用 Azure 儲存體來存放輸入和輸出資料。

建議您在瀏覽此範例之前，先瀏覽[透過 Data Factory 建立第一個管線][adf-build-1st-pipeline]教學課程。 在此範例中使用 Data Factory 編輯器來建立 Data Factory 構件 (連結服務、資料集、管線)。   

1. 為您的 **Azure 儲存體**建立**連結服務**。 如果輸入和輸出檔案在不同的儲存體帳戶中，您就需要兩個連結服務。 以下是 JSON 範例：

    ```JSON
    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=[acctName];AccountKey=[acctKey]"
        }
      }
    }
    ```
2. 建立**輸入** Azure Data Factory **資料集**。 與某些其他 Data Factory 資料集不同的是，這些資料集必須同時包含 **folderPath** 和 **fileName** 值。 您可以使用資料分割，讓每個批次執行 (每一個資料配量) 處理或產生唯一的輸入和輸出檔案。 您可能需要包含某個上游活動，以將輸入轉換成 CSV 檔案格式，並將它放在每個配量的儲存體帳戶中。 在此情況下，您不需包含下列範例所示的 **external** 及 **externalData** 設定，而您的 DecisionTreeInputBlob 會是不同活動的輸出資料集。

    ```JSON
    {
      "name": "DecisionTreeInputBlob",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "azuremltesting/input",
          "fileName": "in.csv",
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ","
          }
        },
        "external": true,
        "availability": {
          "frequency": "Day",
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

    您的輸入 csv 檔案必須要有資料行標題資料列。 如果使用 [複製活動] 建立 csv 或將其移至 Blob 儲存體，則接收屬性 **blobWriterAddHeader** 應該設為 **true**。 例如︰

    ```JSON
    sink:
    {
        "type": "BlobSink",     
        "blobWriterAddHeader": true
    }
    ```

    如果 csv 檔案沒有標頭資料列，您可能會看到下列錯誤：**活動中的錯誤：讀取字串時發生錯誤。非預期的權杖：StartObject。路徑 ''、行 1、位置 1**。
3. 建立**輸出** Azure Data Factory **資料集**。 此範例使用資料分割來為每一個配量執行建立一個唯一輸出路徑。 如果沒有資料分割，活動就會覆寫檔案。

    ```JSON
    {
      "name": "DecisionTreeResultBlob",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "azuremltesting/scored/{folderpart}/",
          "fileName": "{filepart}result.csv",
          "partitionedBy": [
            {
              "name": "folderpart",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "yyyyMMdd"
              }
            },
            {
              "name": "filepart",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "HHmmss"
              }
            }
          ],
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ","
          }
        },
        "availability": {
          "frequency": "Day",
          "interval": 15
        }
      }
    }
    ```
4. 建立 **AzureMLLinkedService** 類型的**連結服務**，並提供 API 金鑰和模型批次執行 URL。

    ```JSON
    {
      "name": "MyAzureMLLinkedService",
      "properties": {
        "type": "AzureML",
        "typeProperties": {
          "mlEndpoint": "https://[batch execution endpoint]/jobs",
          "apiKey": "[apikey]"
        }
      }
    }
    ```
5. 最後，撰寫一個包含 **AzureMLBatchExecution** 活動的管線。 在執行階段，管線會執行下列步驟︰

   1. 從輸入資料集取得輸入檔案的位置。
   2. 叫用 Azure Machine Learning 批次執行 API
   3. 將批次執行輸出複製到輸出資料集中指定的 Blob。

      > [!NOTE]
      > AzureMLBatchExecution 活動可以有零個或多個輸入，以及一個或多個輸出。
      >
      >

    ```JSON
    {
        "name": "PredictivePipeline",
        "properties": {
            "description": "use AzureML model",
            "activities": [
            {
                "name": "MLActivity",
                "type": "AzureMLBatchExecution",
                "description": "prediction analysis on batch input",
                "inputs": [
                {
                    "name": "DecisionTreeInputBlob"
                }
                ],
                "outputs": [
                {
                    "name": "DecisionTreeResultBlob"
                }
                ],
                "linkedServiceName": "MyAzureMLLinkedService",
                "typeProperties":
                {
                    "webServiceInput": "DecisionTreeInputBlob",
                    "webServiceOutputs": {
                        "output1": "DecisionTreeResultBlob"
                    }                
                },
                "policy": {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            }
            ],
            "start": "2016-02-13T00:00:00Z",
            "end": "2016-02-14T00:00:00Z"
        }
    }
    ```

      **開始**和**結束**日期時間必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2014-10-14T16:32:41Z。 **結束**時間是選用項目。 如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。 如需 JSON 屬性的詳細資料，請參閱 [JSON 指令碼參考](https://msdn.microsoft.com/library/dn835050.aspx) 。

      > [!NOTE]
      > 您可自行選擇是否指定 AzureMLBatchExecution 活動的輸入。
      >
      >

### <a name="scenario-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>案例：使用讀取器/寫入器模組參考各種儲存體資料的實驗
建立 Azure ML 實驗時的另一個常見案例，是使用讀取器和寫入器模組。 讀取器模組是用來將資料載入實驗，而寫入器模組則是用於儲存您的實驗資料。 如需讀取器和寫入器模組的詳細資料，請參閱 MSDN Library 上的[讀取器](https://msdn.microsoft.com/library/azure/dn905997.aspx)和[寫入器](https://msdn.microsoft.com/library/azure/dn905984.aspx)主題。     

使用讀取器和寫入器模組時，較好的做法是針對這些讀取器/寫入器模組的每一個屬性，使用 Web 服務參數。 這些 Web 參數可讓您在執行階段設定值。 例如，建立實驗時，您可以利用讀取器模組使用 Azure SQL Database：XXX.database.windows.net。 部署 Web 服務之後，您需要啟用 Web 服務的取用者，藉此指定另一個稱為 YYY.database.windows.net 的 Azure SQL Server。 您可以使用 Web 服務參數來設定此值。

> [!NOTE]
> Web 服務的輸入和輸出與 Web 服務參數不同。 在第一個案例中，您已了解如何為 Azure ML Web 服務指定輸入和輸出。 在此案例中，您會傳遞 Web 服務的參數，以對應至讀取器/寫入器模組的屬性。
>
>

讓我們看看使用 Web 服務參數的案例。 您已部署 Azure Machine Learning Web 服務，其使用讀取器模組所讀取的資料，是來自其中一個受 Azure Machine Learning 支援的資料來源 (例如：Azure SQL Database)。 批次執行之後，會使用寫入器模組寫入結果 (Azure SQL Database)。  實驗中沒有定義任何 Web 服務的輸入和輸出。 在此情況下，我們建議您為讀取器和寫入器模組設定相關 Web 服務參數。 此組態允許在使用 AzureMLBatchExecution 活動時設定讀取器/寫入器模組。 如以下活動 JSON 所示，在 **globalParameters** 區段中指定 Web 服務參數。

```JSON
"typeProperties": {
    "globalParameters": {
        "Param 1": "Value 1",
        "Param 2": "Value 2"
    }
}
```

您也可以使用 [Data Factory 函式](data-factory-functions-variables.md) 傳遞 Web 服務參數的值，如下列範例所示：

```JSON
"typeProperties": {
    "globalParameters": {
       "Database query": "$$Text.Format('SELECT * FROM myTable WHERE timeColumn = \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(WindowStart, 0))"
    }
}
```

> [!NOTE]
> Web 服務參數區分大小寫，因此，請確定您在活動 JSON 中所指定的名稱符合 Web 服務所公開的名稱。
>
>

### <a name="using-a-reader-module-to-read-data-from-multiple-files-in-azure-blob"></a>使用讀取器模組讀取 Azure Blob 中多個檔案的資料
具有 Pig 和 Hive 等活動的巨量資料管線可以產生沒有副檔名的一個或多個輸出檔案。 例如，當您指定外部 Hive 資料表時，外部 Hive 資料表的資料可以儲存在 Azure Blob 儲存體中，並命名為：000000_0。 您可以在實驗中使用讀取器模組讀取多個檔案，並將該模組用於預測。

在 Azure Machine Learning 實驗中使用讀取器模組時，您可以指定 Azure Blob 做為輸入。 Azure Blob 儲存體中的檔案可以是在 HDInsight 上執行的 Pig 和 Hive 指令碼所產生的輸出檔 (範例：000000_0)。 讀取器模組可讓您藉由設定 **容器、目錄或 Blob 的路徑**來讀取檔案 (沒有副檔名)。 **容器路徑**指向容器，**目錄/Blob** 則指向包含檔案的資料夾，如下圖所示。 星號 (也就是 \*) **會指定容器/資料夾中的所有檔案 (也就是 data/aggregateddata/year=2014/month-6/\*)** 將讀取為實驗的一部分。

![Azure Blob 屬性](./media/data-factory-create-predictive-pipelines/azure-blob-properties.png)

### <a name="example"></a>範例
#### <a name="pipeline-with-azuremlbatchexecution-activity-with-web-service-parameters"></a>AzureMLBatchExecution 活動使用 Web 服務參數的管線

```JSON
{
  "name": "MLWithSqlReaderSqlWriter",
  "properties": {
    "description": "Azure ML model with sql azure reader/writer",
    "activities": [
      {
        "name": "MLSqlReaderSqlWriterActivity",
        "type": "AzureMLBatchExecution",
        "description": "test",
        "inputs": [
          {
            "name": "MLSqlInput"
          }
        ],
        "outputs": [
          {
            "name": "MLSqlOutput"
          }
        ],
        "linkedServiceName": "MLSqlReaderSqlWriterDecisionTreeModel",
        "typeProperties":
        {
            "webServiceInput": "MLSqlInput",
            "webServiceOutputs": {
                "output1": "MLSqlOutput"
            }
              "globalParameters": {
                "Database server name": "<myserver>.database.windows.net",
                "Database name": "<database>",
                "Server user account name": "<user name>",
                "Server user account password": "<password>"
              }              
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        },
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```

在上述 JSON 範例中：

* 已部署的 Azure Machine Learning Web 服務使用讀取器和寫入器模組，讀取 Azure SQL Database 的資料，或將資料寫入其中。 此 Web 服務會公開下列 4 個參數：資料庫伺服器名稱、資料庫名稱、伺服器使用者帳戶名稱和伺服器使用者帳戶密碼。  
* **開始**和**結束**日期時間必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2014-10-14T16:32:41Z。 **結束**時間是選用項目。 如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。 如需 JSON 屬性的詳細資料，請參閱 [JSON 指令碼參考](https://msdn.microsoft.com/library/dn835050.aspx) 。

### <a name="other-scenarios"></a>其他案例
#### <a name="web-service-requires-multiple-inputs"></a>Web 服務需要多個輸入
如果 Web 服務接受多個輸入，請使用 **webServiceInputs** 屬性，而不要使用 **webServiceInput**。 **webServiceInputs** 所參考的資料集也必須包含在活動的 **inputs** 中。

在您的 Azure ML 實驗中，Web 服務輸入和輸出連接埠及全域參數有您可以自訂的預設名稱 ("input1"、"input2")。 您用於 webServiceInputs、webServiceOutputs 及 globalParameters 設定的名稱必須與實驗中的名稱完全相同。 您可以檢視您 Azure ML 端點之 [批次執行說明] 頁面上的範例要求承載，來確認預期的對應。

```JSON
{
    "name": "PredictivePipeline",
    "properties": {
        "description": "use AzureML model",
        "activities": [{
            "name": "MLActivity",
            "type": "AzureMLBatchExecution",
            "description": "prediction analysis on batch input",
            "inputs": [{
                "name": "inputDataset1"
            }, {
                "name": "inputDataset2"
            }],
            "outputs": [{
                "name": "outputDataset"
            }],
            "linkedServiceName": "MyAzureMLLinkedService",
            "typeProperties": {
                "webServiceInputs": {
                    "input1": "inputDataset1",
                    "input2": "inputDataset2"
                },
                "webServiceOutputs": {
                    "output1": "outputDataset"
                }
            },
            "policy": {
                "concurrency": 3,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "02:00:00"
            }
        }],
        "start": "2016-02-13T00:00:00Z",
        "end": "2016-02-14T00:00:00Z"
    }
}
```

#### <a name="web-service-does-not-require-an-input"></a>Web 服務不需要輸入
Azure ML 批次執行 Web 服務可用來執行任何可能不需要任何輸入的工作流程，例如 R 或 Python 指令碼。 或者，您也可以使用不會公開任何 GlobalParameters 的讀取器模組來設定實驗。 在此情況下，AzureMLBatchExecution 活動的設定如下：

```JSON
{
    "name": "scoring service",
    "type": "AzureMLBatchExecution",
    "outputs": [
        {
            "name": "myBlob"
        }
    ],
    "typeProperties": {
        "webServiceOutputs": {
            "output1": "myBlob"
        }              
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

#### <a name="web-service-does-not-require-an-inputoutput"></a>Web 服務不需要輸入/輸出
Azure ML 批次執行 Web 服務可能未設定任何 Web 服務輸出。 在此範例中，沒有任何 Web 服務輸入或輸出，也不會設定任何 GlobalParameters。 仍會在活動本身設定輸出，但不會將它指定為 webServiceOutput。

```JSON
{
    "name": "retraining",
    "type": "AzureMLBatchExecution",
    "outputs": [
        {
            "name": "placeholderOutputDataset"
        }
    ],
    "typeProperties": {
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

#### <a name="web-service-uses-readers-and-writers-and-the-activity-runs-only-when-other-activities-have-succeeded"></a>Web 服務會使用讀取器和寫入器，而且只有在其他活動皆成功時，才會執行活動
Azure ML Web 服務的讀取器和寫入器模組可能會設定為不一定要利用任何 GlobalParameters 來執行。 但是，您可能想要在管線中內嵌服務呼叫來使用資料集相依性，只有在完成一些上游處理之後，才會叫用該服務。 您也可以在使用此方法完成批次執行之後觸發一些其他動作。 在此情況下，您可以使用活動輸入和輸出來表達相依性，而不需將它們任一個命名為 Web 服務輸入或輸出。

```JSON
{
    "name": "retraining",
    "type": "AzureMLBatchExecution",
    "inputs": [
        {
            "name": "upstreamData1"
        },
        {
            "name": "upstreamData2"
        }
    ],
    "outputs": [
        {
            "name": "downstreamData"
        }
    ],
    "typeProperties": {
     },
    "linkedServiceName": "mlEndpoint",
    "policy": {
        "concurrency": 1,
        "executionPriorityOrder": "NewestFirst",
        "retry": 1,
        "timeout": "02:00:00"
    }
},
```

**心得** 如下︰

* 如果您的實驗端點使用 webServiceInput：它可透過 Blob 資料集來表示，並且會包含於活動輸入以及 webServiceInput 屬性中。 否則，即會省略 webServiceInput 屬性。
* 如果您的實驗端點使用 webServiceOutput：它們可透過 Blob 資料集來表示，並且會包含於活動輸出以及 webServicepOutputs 屬性中。 活動輸出和 webServiceOutputs 會以此實驗中的每個輸出名稱來對應。 否則，即會省略 webServiceOutputs 屬性。
* 如果您的實驗端點會公開 globalParameter，則會在活動 globalParameters 屬性中提供它們以做為金鑰值組。 否則，即會省略 globalParameters 屬性。 金鑰會區分大小寫。 [Azure Data Factory 函式](data-factory-functions-variables.md) 可能會在值中使用。
* 您可以將額外的資料集包含於活動輸入和輸出屬性中，而不需在活動的 typeProperties 中加以參考。 這些資料集會使用配量相依性來管理執行，但其他的則會被 AzureMLBatchExecution 活動所忽略。


## <a name="updating-models-using-update-resource-activity"></a>使用更新資源活動來更新模型
完成重新訓練之後，您可以使用「Azure ML 更新資源活動」，使用新訓練的模型來更新評分 Web 服務 (以 Web 服務公開的預測實驗)。 如需詳細資料，請參閱[使用更新資源活動更新模型](data-factory-azure-ml-update-resource-activity.md)一文。

### <a name="reader-and-writer-modules"></a>讀取器和寫入器模組
使用 Azure SQL 讀取器和寫入器，是常見的 Web 服務參數使用案例。 讀取器模組可用來將資料從 Azure Machine Learning Studio 外部的資料管理服務載入至實驗。 寫入器模組則用來將資料從實驗儲存至 Azure Machine Learning Studio 外部的資料管理服務。  

如需 Azure Blob/Azure SQL 讀取器/寫入器的詳細資料，請參閱 MSDN Library 上的[讀取器](https://msdn.microsoft.com/library/azure/dn905997.aspx)和[寫入器](https://msdn.microsoft.com/library/azure/dn905984.aspx)主題。 上一節中的範例已使用 Azure Blob 讀取器與 Azure Blob 寫入器。 本節討論如何使用 Azure SQL 讀取器和 Azure SQL 寫入器。

## <a name="frequently-asked-questions"></a>常見問題集
**問：** 我擁有巨量資料管線所產生的多個檔案。 我可以使用 AzureMLBatchExecution 活動來處理所有檔案嗎？

**答：** 是。 如需詳細資料，請參閱 **使用讀取器模組讀取 Azure Blob 中多個檔案的資料** 一節。

## <a name="azure-ml-batch-scoring-activity"></a>AzureML 批次計分活動
如果您使用 **AzureMLBatchScoring** 活動來與 Azure Machine Learning 整合，建議您使用最新的 **AzureMLBatchExecution** 活動。

2015 年 8 月發行的 Azure SDK 和 Azure PowerShell 中導入了 AzureMLBatchExecution 活動。

如果您想要繼續使用 AzureMLBatchScoring 活動，請繼續閱讀本節。  

### <a name="azure-ml-batch-scoring-activity-using-azure-storage-for-inputoutput"></a>使用 Azure 儲存體進行輸入/輸出的 AzureML 批次計分活動

```JSON
{
  "name": "PredictivePipeline",
  "properties": {
    "description": "use AzureML model",
    "activities": [
      {
        "name": "MLActivity",
        "type": "AzureMLBatchScoring",
        "description": "prediction analysis on batch input",
        "inputs": [
          {
            "name": "ScoringInputBlob"
          }
        ],
        "outputs": [
          {
            "name": "ScoringResultBlob"
          }
        ],
        "linkedServiceName": "MyAzureMLLinkedService",
        "policy": {
          "concurrency": 3,
          "executionPriorityOrder": "NewestFirst",
          "retry": 1,
          "timeout": "02:00:00"
        }
      }
    ],
    "start": "2016-02-13T00:00:00Z",
    "end": "2016-02-14T00:00:00Z"
  }
}
```

### <a name="web-service-parameters"></a>Web 服務參數
若要指定 Web 服務參數的值，請將 **typeProperties** 區段新增至管線 JSON 中的 **AzureMLBatchScoringActivty** 區段，如下列範例所示：

```JSON
"typeProperties": {
    "webServiceParameters": {
        "Param 1": "Value 1",
        "Param 2": "Value 2"
    }
}
```
您也可以使用 [Data Factory 函式](data-factory-functions-variables.md) 傳遞 Web 服務參數的值，如下列範例所示：

```JSON
"typeProperties": {
    "webServiceParameters": {
       "Database query": "$$Text.Format('SELECT * FROM myTable WHERE timeColumn = \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(WindowStart, 0))"
    }
}
```

> [!NOTE]
> Web 服務參數區分大小寫，因此，請確定您在活動 JSON 中所指定的名稱符合 Web 服務所公開的名稱。
>
>

## <a name="see-also"></a>另請參閱
* [Azure 部落格文章：開始使用 Azure Data Factory 和 Azure Machine Learning](https://azure.microsoft.com/blog/getting-started-with-azure-data-factory-and-azure-machine-learning-4/)

[adf-build-1st-pipeline]: data-factory-build-your-first-pipeline.md

[azure-machine-learning]: http://azure.microsoft.com/services/machine-learning/
