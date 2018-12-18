---
title: 使用 Azure Data Factory 從 Amazon Redshift 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 複製活動從 Amazon Redshift 移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 01d15078-58dc-455c-9d9d-98fbdf4ea51e
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 7ece34809734478ddb52c12d5dbd92291231f439
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045682"
---
# <a name="move-data-from-amazon-redshift-using-azure-data-factory"></a>使用 Azure Data Factory 從 Amazon Redshift 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-amazon-redshift-connector.md)
> * [第 2 版 (目前的版本)](../connector-amazon-redshift.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Amazon Redshift 連接器](../connector-amazon-redshift.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，從 Amazon Redshift 移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。 

Data Factory 目前僅支援資料從 Amazon Redshift 移至[支援的接收資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 不支援從其他資料存放區將資料移至 Amazon Redshift。

> [!TIP]
> 當您從 Amazon Redshift 中複製大量資料時，若想獲得最佳效能，請考慮透過 Amazon Simple Storage Service (Amazon S3) 使用內建的 Redshift **UNLOAD**。 如需詳細資料，請參閱[使用 UNLOAD 複製 Amazon Redshift 中的資料](#use-unload-to-copy-data-from-amazon-redshift)。

## <a name="prerequisites"></a>先決條件
* 如果您要將資料移到內部部署的資料存放區，請在內部部署的電腦上安裝[資料管理閘道](data-factory-data-management-gateway.md)。 使用內部部署機器的 IP 位址，授與 Amazon Redshift 叢集閘道的存取權。 如需相關指示，請參閱[授權存取叢集](http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html)。
* 若要將資料移動到 Azure 資料存放區，請參閱[計算 Microsoft Azure 資料中心所使用的 IP 位址和 SQL 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具和 API，建立內含複製活動的管線，以從 Amazon Redshift 來源移動資料。

建立管線的最簡單方式就是使用「Azure Data Factory 複製精靈」。 如需使用複製精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

您也可以使用 Azure 入口網站、Visual Studio、Azure PowerShell 或其他工具來建立管線。 Azure Resource Manager 範本、.NET API 或 REST API 也可以用來建立管線。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立連結服務，將輸入和輸出資料存放區連結到資料處理站。
2. 建立資料集，代表複製作業的輸入和輸出資料。 
3. 建立管線，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用複製精靈時，精靈會自動為您建立 Data Factory 實體的 JSON 定義。 使用工具或 API (.NET API 除外) 時，您需使用 JSON 格式來定義資料處理站實體。 [JSON 範例：將資料從 Amazon Redshift 複製到 Azure Blob 儲存體](#json-example-copy-data-from-amazon-redshift-to-azure-blob)顯示用來從 Amazon Redshift 資料存放區複製資料的 Data Factory 實體本身的 JSON 定義。

下列各節描述 JSON 屬性，這些屬性是用來定義 Amazon Redshift 的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

下表提供 Amazon Redshift 服務連結服務專屬之 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **type** |屬性必須設為 **AmazonRedshift**。 |yes |
| **server** |Amazon Redshift 伺服器的 IP 位址或主機名稱。 |yes |
| **port** |Amazon Redshift 伺服器用來接聽用戶端連線的 TCP 連接埠號碼。 |否 (預設值為 5439) |
| **database** |Amazon Redshift 資料庫的名稱。 |yes |
| **username** |可存取資料庫之使用者的名稱。 |yes |
| **password** |使用者帳戶的密碼。 |yes |

## <a name="dataset-properties"></a>資料集屬性

如需定義資料集的區段和屬性清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 所有資料集類型的 **structure**、**availability** 和 **policy** 等區段都類似。 資料集類型的範例包括 Azure SQL、Azure Blob 儲存體和 Azure 資料表儲存體。

每個類型之資料集的 **typeProperties** 區段都不同，可提供存放區中資料位置的相關資訊。 **RelationalTable** 資料集類型的 **typeProperties** 區段 (包含 Amazon Redshift 資料集) 具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **tableName** |Amazon Redshift 資料庫中連結服務所參照的資料表名稱。 |否 (如果指定 **RelationalSource** 類型複製活動的**查詢**屬性) |

## <a name="copy-activity-properties"></a>複製活動屬性

如需定義活動的區段和屬性清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 **name****description**、**inputs** 資料表、**outputs** 資料表和 **policy** 屬性適用於所有活動。 **typeProperties** 區段中的可用屬性，會因為活動類型不同而有所差異。 就複製活動而言，屬性會根據資料來源和接收的類型而有所不同。

對於複製活動，當來源類型為 **AmazonRedshiftSource** 時，**typeProperties** 區段中可使用下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **查詢** | 使用自訂查詢來讀取資料。 |否 (如果已指定資料集的 **tableName** 屬性) |
| **redshiftUnloadSettings** | 在使用 Redshift **UNLOAD** 命令時包含屬性群組。 | 否 |
| **s3LinkedServiceName** | 要作為暫時存放區的 Amazon S3。 連結服務是使用類型為 **AwsAccessKey** 的 Azure Data Factory 名稱來指定。 | 使用 **redshiftUnloadSettings** 屬性時為必要 |
| **bucketName** | 表示要用來儲存暫時資料的 Amazon S3 貯體。 如果未提供此屬性，複製活動會自動產生貯體。 | 使用 **redshiftUnloadSettings** 屬性時為必要 |

您也可以將 **RelationalSource** 類型 (包括 Amazon Redshift) 與下列 **typeProperties** 區段中的屬性搭配使用。 請注意，此來源類型不支援 Redshift **UNLOAD** 命令。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **查詢** |使用自訂查詢來讀取資料。 | 否 (如果已指定資料集的 **tableName** 屬性) |

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>使用 UNLOAD 複製 Amazon Redshift 中的資料

Amazon Redshift [**UNLOAD**](http://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) 命令會將查詢的結果上傳到 Amazon S3 上的一個或多個檔案。 Amazon 建議使用此命令從 Redshift 複製大型資料集。

**範例：將資料從 Amazon Redshift 複製到 Azure SQL 資料倉儲**

此範例會將資料從 Amazon Redshift 複製到 Azure SQL 資料倉儲。 此範例使用 Redshift **UNLOAD** 命令、暫存複製資料和 Microsoft PolyBase。

在此範例使用案例中，複製活動首先將來自 Amazon Redshift 的資料上傳到 Amazon S3，如同 **redshiftUnloadSettings** 選項中所設定。 然後，將資料從 Amazon S3 複製到 **stagingSettings** 選項中指定的 Azure Blob 儲存體。 最後，PolyBase 將資料載入 SQL 資料倉儲。 複製活動會處理所有暫時格式。

![將工作流程從 Amazon Redshift 複製到 SQL 資料倉儲](media\data-factory-amazon-redshift-connector\redshift-to-sql-dw-copy-workflow.png)

```json
{
    "name": "CopyFromRedshiftToSQLDW",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "AmazonRedshiftSource",
            "query": "select * from MyTable",
            "redshiftUnloadSettings": {
                "s3LinkedServiceName":"MyAmazonS3StorageLinkedService",
                "bucketName": "bucketForUnload"
            }
        },
        "sink": {
            "type": "SqlDWSink",
            "allowPolyBase": true
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyAzureStorageLinkedService",
            "path": "adfstagingcopydata"
        },
        "cloudDataMovementUnits": 32
        .....
    }
}
```

## <a name="json-example-copy-data-from-amazon-redshift-to-azure-blob-storage"></a>JSON 範例：將資料從 Amazon Redshift 複製到 Azure Blob 儲存體
此範例示範如何將資料從 Amazon Redshift 資料庫複製到 Azure Blob 儲存體。 使用複製活動，可以將資料直接複製到任何[支援的接收資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。  

此範例具有下列 Data Factory 實體：

* [AmazonRedshift](#linked-service-properties) 類型的連結服務
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [RelationalTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)
* 具有使用 [RelationalSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md##copy-activity-properties) 屬性之複製活動的[管線](data-factory-create-pipelines.md)

此範例會每個小時將資料從 Amazon Redshift 中的查詢結果複製到 Blob。 實體定義後面的各節會說明範例中使用的 JSON 屬性。

**Amazon Redshift 連結服務**

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "< The IP address or host name of the Amazon Redshift server >",
            "port": <The number of the TCP port that the Amazon Redshift server uses to listen for client connections.>,
            "database": "<The database name of the Amazon Redshift database>",
            "username": "<username>",
            "password": "<password>"
        }
    }
}
```

**Azure Blob 儲存體連結服務**

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
**Amazon Redshift 輸入資料集**

將 **external** 設定成 "true" 會通知 Data Factory 服務該資料集是 Data Factory 外部的資料集。 此屬性設定指出該資料集不是由資料處理站中的活動產生。 在不是由管線中的活動所產生的輸入資料集上，請將此屬性設定為 true。

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Azure Blob 輸出資料集**

藉由將 **frequency** 屬性設定 "Hour"，並將 **interval** 屬性設定為 1，資料會每小時寫入到新的 Blob。 Blob 的 **folderPath** 屬性是以動態的方式評估。 屬性值是以正在處理之配量的開始時間為基礎。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazonredshift/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**管線中具有 Azure Redshift 來源 (屬於 RelationalSource 類型) 和 Azure Blob 接收器的複製活動**

管線包含複製活動，該活動已設定為使用輸入和輸出資料集。 管線排程為每小時執行一次。 在管線的 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。 針對 **query** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "AmazonRedshiftSource",
                        "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)",
                        "redshiftUnloadSettings": {
                            "s3LinkedServiceName":"myS3Storage",
                            "bucketName": "bucketForUnload"
                        }
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    },
                    "cloudDataMovementUnits": 32
                },
                "inputs": [
                    {
                        "name": "AmazonRedshiftInputDataset"
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
                "name": "AmazonRedshiftToBlob"
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```
### <a name="type-mapping-for-amazon-redshift"></a>Amazon Redshift 的類型對應
如[資料移動活動](data-factory-data-movement-activities.md)一文所述，複製活動會執行從來源類型轉換成接收類型的自動類型轉換。 類型是使用包含兩個步驟的方法來轉換：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

複製活動將資料從 Amazon Redshift 類型轉換成 .NET 類型時，會使用下列對應：

| Amazon Redshift 類型 | .NET 類型 |
| --- | --- |
| SMALLINT |Int16 |
| INTEGER |Int32 |
| BIGINT |Int64 |
| DECIMAL |十進位 |
| REAL |單一 |
| DOUBLE PRECISION |兩倍 |
| BOOLEAN |字串 |
| CHAR |字串 |
| VARCHAR |字串 |
| 日期 |Datetime |
| 時間戳記 |Datetime |
| TEXT |字串 |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-reads-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請留意可重複性，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試**原則**，使得在發生失敗時，重新執行配量。 確定無論配量將執行多少次，均讀取相同的資料。 也請確定無論重新執行配量的方式為何，均讀取相同的資料。 如需詳細資訊，請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)，了解影響複製活動效能的重要因素，以及達到最佳效能的各種方法。 

## <a name="next-steps"></a>後續步驟
如需使用複製活動來建立管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
