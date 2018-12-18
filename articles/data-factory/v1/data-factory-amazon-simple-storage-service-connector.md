---
title: 使用 Data Factory 從 Amazon Simple Storage Service 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Amazon Simple Storage Service (S3) 移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 636d3179-eba8-4841-bcb4-3563f6822a26
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: d895dcdf9eefac01790aab6dc3f36a3feb0a8b12
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051145"
---
# <a name="move-data-from-amazon-simple-storage-service-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 Amazon Simple Storage Service 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-amazon-simple-storage-service-connector.md)
> * [第 2 版 (目前的版本)](../connector-amazon-simple-storage-service.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Amazon S3 連接器](../connector-amazon-simple-storage-service.md)。

本文說明如何使用 Azure Data Factory 中的複製活動，從 Amazon Simple Storage Service (S3) 移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從 Amazon S3 複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從 Amazon S3 移到其他資料存放區，而不支援將資料從其他資料存放區移到 Amazon S3。

## <a name="required-permissions"></a>所需的權限
若要從 Amazon S3 複製資料，請確定您已獲得下列權限︰

* 適用於 Amazon S3 物件作業的 `s3:GetObject` 和 `s3:GetObjectVersion`。
* 適用於 Amazon S3 貯體作業的 `s3:ListBucket`。 如果您要使用「Data Factory 複製精靈」，則也需要 `s3:ListAllMyBuckets`。

如需有關完整 Amazon S3 權限清單的詳細資料，請參閱[在原則中指定權限 (英文)](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html)。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具或 API，建立內含複製活動的管線，以從 Amazon S3 來源移動資料。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟，以建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具或 API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。 如需相關範例，其中含有用來從 Amazon S3 資料存放區複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 Amazon S3 複製到 Azure Blob](#json-example-copy-data-from-amazon-s3-to-azure-blob-storage) 一節。

> [!NOTE]
> 如需有關針對複製活動支援的檔案和壓縮格式的詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Amazon S3 特定的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性
已連結的服務會將資料存放區連結到 Data Factory。 您需建立 **AwsAccessKey** 類型的已連結服務，以將 Amazon S3 資料存放區連結到 Data Factory。 下表提供 Amazon S3 (AwsAccessKey) 已連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| accessKeyID |密碼存取金鑰的識別碼。 |字串 |yes |
| secretAccessKey |密碼存取金鑰本身。 |加密的密碼字串 |yes |

>[!NOTE]
>此連接器需要 IAM 帳戶的存取金鑰，才能從 Amazon S3 複製資料。 不支援[暫存安全性認證](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)。
>

下列是一個範例：

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
若要指定資料集以代表 Azure Blob 儲存體中的輸入資料，請將資料集的類型屬性設定成 **AmazonS3**。 請將資料集的 **linkedServiceName** 屬性設定成 Amazon S3 已連結服務的名稱。 如需可供定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)。 

所有資料集類型 (例如 SQL 資料庫、Azure Blob 及 Azure 資料表) 的結構、可用性及原則等區段都相似。 每個類型之資料集的 **typeProperties** 區段都不同，可提供資料存放區中資料位置的相關資訊。 **AmazonS3** 類型之資料集 (包括 Amazon S3 資料集) 的 **typeProperties** 區段具有下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| bucketName |S3 貯體名稱。 |字串 |yes |
| 索引鍵 |S3 物件索引鍵。 |字串 |否 |
| prefix |S3 物件索引鍵的前置詞。 系統會選取索引鍵以此前置詞開頭的物件。 只有當索引鍵空白時才適用。 |字串 |否 |
| version |如果已啟用 S3 版本設定功能，則為 S3 物件的版本。 |字串 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[JSON 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)小節。 <br><br> 如果您想要在檔案型存放區之間依原樣複製檔案 (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 | |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 | |


> [!NOTE]
> **bucketName + key** 可指定 S3 物件的位置，其中 bucket (貯體) 是 S3 物件的根容器，而 key 是 S3 物件的完整路徑。

### <a name="sample-dataset-with-prefix"></a>prefix 的相關範例資料集

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "testbucket",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
### <a name="sample-dataset-with-version"></a>範例資料集 (含版本)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "testbucket",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

### <a name="dynamic-paths-for-s3"></a>S3 的動態路徑
上述範例針對 Amazon S3 資料集內的 **key** 和 **bucketName** 屬性使用固定的值。

```json
"key": "testFolder/test.orc",
"bucketName": "testbucket",
```

您可以藉由使用系統變數 (例如 SliceStart)，讓 Data Factory 在執行階段動態地計算這些屬性。

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

您也可以對 Amazon S3 資料集的 **prefix** 屬性執行相同的操作。 如需支援的函式和變數清單，請參閱 [Data Factory 函式與系統變數](data-factory-functions-variables.md)。

## <a name="copy-activity-properties"></a>複製活動屬性
如需可用來定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。 活動的 **typeProperties** 區段中可用的屬性會隨著每個活動類型而有所不同。 就複製活動而言，屬性會根據來源和接收器的類型而有所不同。 當複製活動中的來源類型為 **FileSystemSource** (其中包括 Azure S3) 時，**typeProperties** 區段中會有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |指定是否要以遞迴方式列出目錄下的 S3 物件。 |true/false |否 |

## <a name="json-example-copy-data-from-amazon-s3-to-azure-blob-storage"></a>JSON 範例：將資料從 Amazon S3 複製到 Azure Blob 儲存體
此範例示範如何將資料從 Amazon S3 複製到 Azure Blob 儲存體。 不過，您可以使用 Data Factory 中的複製活動，將資料直接複製到[任何支援的接收器](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。

此範例提供下列 Data Factory 實體的 JSON 定義。 您可以透過 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)，使用這些定義來建立管線，以將資料從 Amazon S3 複製到 Blob 儲存體。   

* [AwsAccessKey](#linked-service-properties)類型的連結服務。
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [AmazonS3](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [FileSystemSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將資料從 Amazon S3 複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

### <a name="amazon-s3-linked-service"></a>Amazon S3 已連結的服務

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

### <a name="amazon-s3-input-dataset"></a>Amazon S3 輸入資料集

設定 **"external": true** 會通知 Data Factory 服務該資料集是 Data Factory 外部的資料集。 在不是由管線中的活動所產生的輸入資料集上，請將此屬性設定為 true。

```json
    {
        "name": "AmazonS3InputDataset",
        "properties": {
            "type": "AmazonS3",
            "linkedServiceName": "AmazonS3LinkedService",
            "typeProperties": {
                "key": "testFolder/test.orc",
                "bucketName": "testbucket",
                "format": {
                    "type": "OrcFormat"
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true
        }
    }
```


### <a name="azure-blob-output-dataset"></a>Azure Blob 輸出資料集

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 此資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazons3/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```


### <a name="copy-activity-in-a-pipeline-with-an-amazon-s3-source-and-a-blob-sink"></a>具有 Amazon S3 來源和 Blob 接收器的管線中複製活動

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **FileSystemSource**，而 **sink** 類型設為 **BlobSink**。

```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "FileSystemSource",
                        "recursive": true
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "AmazonS3InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "AmazonS3ToBlob"
            }
        ],
        "start": "2014-08-08T18:00:00Z",
        "end": "2014-08-08T19:00:00Z"
    }
}
```
> [!NOTE]
> 若要將來自來源資料集的資料行與來自接收資料集的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。


## <a name="next-steps"></a>後續步驟
請參閱下列文章：

* 若要了解影響 Data Factory 中資料移動 (複製活動) 效能的關鍵因素，以及各種最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。

* 如需使用複製活動來建立管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
