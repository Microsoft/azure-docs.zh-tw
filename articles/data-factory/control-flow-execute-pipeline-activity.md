---
title: 在 Azure Data Factory 中執行管線活動 | Microsoft Docs
description: 了解如何使用「執行管線活動」從一個資料處理站管線叫用另一個資料處理站管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
ms.openlocfilehash: 2aa25004fb9c2e914cd8c669095953e174686197
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051758"
---
# <a name="execute-pipeline-activity-in-azure-data-factory"></a>在 Azure Data Factory 中執行管線活動
執行管線活動可允許資料處理站管線叫用另一個管線。

## <a name="syntax"></a>語法

```json
{
    "name": "MyPipeline",
    "properties": {
        "activities": [
            {
                "name": "ExecutePipelineActivity",
                "type": "ExecutePipeline",
                "typeProperties": {
                    "parameters": {                        
                        "mySourceDatasetFolderPath": {
                            "value": "@pipeline().parameters.mySourceDatasetFolderPath",
                            "type": "Expression"
                        }
                    },
                    "pipeline": {
                        "referenceName": "<InvokedPipelineName>",
                        "type": "PipelineReference"
                    },
                    "waitOnCompletion": true
                 }
            }
        ],
        "parameters": [
            {
                "mySourceDatasetFolderPath": {
                    "type": "String"
                }
            }
        ]
    }
}
```

## <a name="type-properties"></a>類型屬性
屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
name | 執行管線活動的名稱。 | 字串 | yes
type | 必須設為：**ExecutePipeline**。 | 字串 | yes
管線 | 此管線叫用之相依管線的管線參考。 管線參考物件有兩個屬性：**referenceName** 和 **type**。 referenceName 屬性指定參考管線的名稱。 type 屬性必須設為 PipelineReference。 | PipelineReference | yes
parameters | 要傳遞至叫用之管線的參數 | 將參數名稱對應到引數值的 JSON 物件 | 否
waitOnCompletion | 定義活動執行是否等待相依的管線執行完成。 | 預設值為 false。 | BOOLEAN | 否

## <a name="sample"></a>範例
此案例有兩個管線：

- **主要管線** - 此管線有一個執行管線活動，會呼叫叫用的管線。 主要管線採用兩個參數：`masterSourceBlobContainer`、`masterSinkBlobContainer`。
- **叫用的管線** - 此管線有一個複製活動，會將資料從 Azure Blob 來源複製到 Azure Blob 接收。 叫用的管線採用兩個參數：`sourceBlobContainer`、`sinkBlobContainer`。

### <a name="master-pipeline-definition"></a>主要管線定義

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ExecutePipeline",
        "typeProperties": {
          "pipeline": {
            "referenceName": "invokedPipeline",
            "type": "PipelineReference"
          },
          "parameters": {
            "sourceBlobContainer": {
              "value": "@pipeline().parameters.masterSourceBlobContainer",
              "type": "Expression"
            },
            "sinkBlobCountainer": {
              "value": "@pipeline().parameters.masterSinkBlobContainer",
              "type": "Expression"
            }
          },
          "waitOnCompletion": true
        },
        "name": "MyExecutePipelineActivity"
      }
    ],
    "parameters": {
      "masterSourceBlobContainer": {
        "type": "String"
      },
      "masterSinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

### <a name="invoked-pipeline-definition"></a>叫用的管線定義

```json
{
  "name": "invokedPipeline",
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
        "name": "CopyBlobtoBlob",
        "inputs": [
          {
            "referenceName": "SourceBlobDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sinkBlobDataset",
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

**連結服務**

```json
{
    "name": "BlobStorageLinkedService",
    "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "value": "DefaultEndpointsProtocol=https;AccountName=*****",
        "type": "SecureString"
      }
    }
  }
}
```

**來源資料集**
```json
{
    "name": "SourceBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sourceBlobContainer",
        "type": "Expression"
      },
      "fileName": "salesforce.txt"
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

**接收資料集**
```json
{
    "name": "sinkBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sinkBlobContainer",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

### <a name="running-the-pipeline"></a>執行管線

為執行此範例中的主要管線，下列值會傳遞給 masterSourceBlobContainer 和 masterSinkBlobContainer 參數： 

```json
{
  "masterSourceBlobContainer": "executetest",
  "masterSinkBlobContainer": "executesink"
}
```

主要管線會將這些值轉送到叫用的管線，如下列範例所示： 

```json
{
    "type": "ExecutePipeline",
    "typeProperties": {
      "pipeline": {
        "referenceName": "invokedPipeline",
        "type": "PipelineReference"
      },
      "parameters": {
        "sourceBlobContainer": {
          "value": "@pipeline().parameters.masterSourceBlobContainer",
          "type": "Expression"
        },
        "sinkBlobCountainer": {
          "value": "@pipeline().parameters.masterSinkBlobContainer",
          "type": "Expression"
        }
      },

      ....
}

```
## <a name="next-steps"></a>後續步驟
請參閱 Data Factory 支援的其他控制流程活動： 

- [For Each 活動](control-flow-for-each-activity.md)
- [取得中繼資料活動](control-flow-get-metadata-activity.md)
- [查閱活動](control-flow-lookup-activity.md)
- [Web 活動](control-flow-web-activity.md)
