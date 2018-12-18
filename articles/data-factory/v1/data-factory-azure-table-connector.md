---
title: 從 Azure 資料表來回移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Azure 表格儲存體來回移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 07b046b1-7884-4e57-a613-337292416319
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 3a24e919f1bbde6188e3655399f1ef843fbec23b
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37052985"
---
# <a name="move-data-to-and-from-azure-table-using-azure-data-factory"></a>使用 Azure Data Factory 從 Azure 資料表來回移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-azure-table-connector.md)
> * [第 2 版 (目前的版本)](../connector-azure-table-storage.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Azure 表格儲存體連接器](../connector-azure-table-storage.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，將資料移進/移出「Azure 資料表儲存體」。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。 

您可以將資料從任何支援的來源資料存放區複製到「Azure 資料表儲存體」，或從「Azure 資料表儲存體」複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出「Azure 資料表儲存體」。

建立管線的最簡單方式就是使用「複製精靈」。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到「Azure 資料表儲存體」(或從「Azure 資料表儲存體」複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples)一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義「Azure 資料表儲存體」特定的 Data Factory 實體： 

## <a name="linked-service-properties"></a>連結服務屬性
您可以使用兩種類型的連結服務，將 Azure Blob 儲存體連結至 Azure Data Factory。 它們是：**AzureStorage** 連結服務和 **AzureStorageSas** 連結服務。 Azure 儲存體連結服務可將 Azure 儲存體的全域存取權提供給 Data Factory。 而 Azure 儲存體 SAS (共用存取簽章) 連結服務會將 Azure 儲存體的受限制/時間繫結存取權提供給 Data Factory。 這兩個連結服務之間沒有其他差異。 選擇符合您需求的連結服務。 以下章節提供這兩個連結服務的詳細資料。

[!INCLUDE [data-factory-azure-storage-linked-services](../../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。 **AzureTable** 類型資料集的 **typeProperties** 區段有下列屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Azure 資料表資料庫執行個體中連結服務所參照的資料表名稱。 |是。 指定 tableName 時若沒有指定 azureTableSourceQuery，資料表中的所有記錄都會複製到目的地。 如果同時指定了 azureTableSourceQuery，則資料表中符合查詢的記錄會複製到目的地。 |

### <a name="schema-by-data-factory"></a>Data factory 的結構描述
針對無結構描述的資料存放區 (如 Azure 資料表)，Data Factory 服務會以下列一種方式推斷結構描述：

1. 如果您是使用資料集定義中的 **structure** 屬性來定義結構，Data Factory 服務會將此結構接受為結構描述。 在此情況下，如果資料列的資料行沒有值，系統就會為它提供 null 值。
2. 如果您沒有使用資料集定義中的 **structure** 屬性來指定結構，Data Factory 服務將會使用資料的第一列來推斷結構描述。 在此情況下，如果第一個資料列未包含完整的結構描述，複製作業的結果中就會遺失一些資料行。

因此，對於無結構描述的資料來源來說，最佳作法是使用 **structure** 屬性來指定資料結構。

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料集，以及原則) 適用於所有類型的活動。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

**AzureTableSource** 在 typeProperties 區段中支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| AzureTableSourceQuery |使用自訂查詢來讀取資料。 |Azure 資料表查詢字串。 請參閱下一節中的範例。 |否。 指定 tableName 時若沒有指定 azureTableSourceQuery，資料表中的所有記錄都會複製到目的地。 如果同時指定了 azureTableSourceQuery，則資料表中符合查詢的記錄會複製到目的地。 |
| azureTableSourceIgnoreTableNotFound |指出是否忍受資料表不存在的例外狀況。 |TRUE<br/>FALSE |否 |

### <a name="azuretablesourcequery-examples"></a>azureTableSourceQuery 範例
如果 Azure 資料表資料行是字串類型：

```JSON
azureTableSourceQuery": "$$Text.Format('PartitionKey ge \\'{0:yyyyMMddHH00_0000}\\' and PartitionKey le \\'{0:yyyyMMddHH00_9999}\\'', SliceStart)"
```

如果 Azure 資料表資料行是日期時間類型：

```JSON
"azureTableSourceQuery": "$$Text.Format('DeploymentEndTime gt datetime\\'{0:yyyy-MM-ddTHH:mm:ssZ}\\' and DeploymentEndTime le datetime\\'{1:yyyy-MM-ddTHH:mm:ssZ}\\'', SliceStart, SliceEnd)"
```

**AzureTableSink** 在 typeProperties 區段中支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |可供接收器使用的預設資料分割索引鍵值。 |字串值。 |否 |
| azureTablePartitionKeyName |指定其值用來作為分割區索引鍵的資料行名稱。 如果未指定，則會使用 AzureTableDefaultPartitionKeyValue 做為資料分割索引鍵。 |資料行名稱。 |否 |
| azureTableRowKeyName |指定其值用來作為資料列索引鍵的資料行名稱。 如果未指定，則會針對每個資料列使用 GUID。 |資料行名稱。 |否 |
| azureTableInsertType |將資料插入 Azure 資料表的模式。<br/><br/>此屬性可控制針對輸出資料表中具有相符分割區和資料列索引鍵的現有資料列，是要取代還是合併其值。 <br/><br/>若要了解這些設定 (合併和取代) 的運作方式，請參閱[插入或合併實體](https://msdn.microsoft.com/library/azure/hh452241.aspx)和[插入或取代實體](https://msdn.microsoft.com/library/azure/hh452242.aspx)主題。 <br/><br> 此設定是在資料列層級套用，而不是在資料表層級套用，而且兩個選項都不會刪除存在於輸出資料表中但不存在於輸入中的資料列。 |合併 (預設值)<br/>取代 |否 |
| writeBatchSize |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中。 |整數 (資料列數目) |否 (預設值：10000) |
| writeBatchTimeout |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中 |時間範圍<br/><br/>範例：“00:20:00” (20 分鐘) |否 (預設為儲存體用戶端預設逾時值 90 秒) |

### <a name="azuretablepartitionkeyname"></a>azureTablePartitionKeyName
您必須先使用轉譯器 JSON 屬性將來源資料行對應至目的地資料行，才能使用目的地資料行作為 azureTablePartitionKeyName。

在下列範例中，來源資料行 DivisionID 會對應至目的地資料行：DivisionID。  

```JSON
"translator": {
    "type": "TabularTranslator",
    "columnMappings": "DivisionID: DivisionID, FirstName: FirstName, LastName: LastName"
}
```
DivisionID 被指定為分割區索引鍵。

```JSON
"sink": {
    "type": "AzureTableSink",
    "azureTablePartitionKeyName": "DivisionID",
    "writeBatchSize": 100,
    "writeBatchTimeout": "01:00:00"
}
```
## <a name="json-examples"></a>JSON 範例
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 它們會示範如何將資料複製到 Azure 表格儲存體和 Azure Blob 儲存體，以及複製其中的資料。 不過，您可以將資料從任何來源**直接**複製到任何支援的接收器。 如需詳細資訊，請參閱[使用複製活動來移動資料](data-factory-data-movement-activities.md)中的＜支援的資料存放區和格式＞一節。

## <a name="example-copy-data-from-azure-table-to-azure-blob"></a>範例：將資料從 Azure 資料表複製到 Azure Blob
下列範例顯示︰

1. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 類型的連結服務 (同時用於資料表和 Blob)。
2. [AzureTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4. 具有使用 [AzureTableSource](#activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將 Azure 資料表中屬於預設資料分割的資料複製到 Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure 儲存體連結服務：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory 支援兩種類型的 Azure 儲存體連結服務：**AzureStorage** 和 **AzureStorageSas**。 對於前者指定連接字串，包含帳戶金鑰，對於後者指定共用存取簽章 (SAS) Uri。 請參閱 [連結服務](#linked-service-properties) 章節以取得詳細資料。  

**Azure 資料表輸入資料集：**

此範例假設您已在 Azure 資料表中建立資料表 "MyTable"。

設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "AzureTableInput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
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

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**具有 AzureTableSource 和 BlobSink 的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設定為 **AzureTableSource**，且 **sink** 類型設定為 **BlobSink**。 使用 **AzureTableSourceQuery** 屬性指定的 SQL 查詢會每小時從預設分割中選取要複製的資料。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
            {
                "name": "AzureTabletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                      {
                        "name": "AzureTableInput"
                    }
                ],
                "outputs": [
                      {
                            "name": "AzureBlobOutput"
                      }
                ],
                "typeProperties": {
                      "source": {
                        "type": "AzureTableSource",
                        "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
                      },
                      "sink": {
                        "type": "BlobSink"
                      }
                },
                "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },                
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
            }
         ]    
    }
}
```

## <a name="example-copy-data-from-azure-blob-to-azure-table"></a>範例：將資料從 Azure Blob 複製到 Azure 資料表
下列範例顯示︰

1. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 類型的連結服務 (同時用於資料表和 Blob)
2. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
3. [AzureTable](#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
4. 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [AzureTableSink](#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料從 Azure Blob 複製到 Azure 資料表。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure 儲存體 (同時適用於 Azure 資料表和 Blob) 連結服務：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

Azure Data Factory 支援兩種類型的 Azure 儲存體連結服務：**AzureStorage** 和 **AzureStorageSas**。 對於前者指定連接字串，包含帳戶金鑰，對於後者指定共用存取簽章 (SAS) Uri。 請參閱 [連結服務](#linked-service-properties) 章節以取得詳細資料。

**Azure Blob 輸入資料集：**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月及日部分，而檔案名稱則使用開始時間的小時部分。 “external”: “true” 設定可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
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
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
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

**Azure 資料表輸出資料集：**

此範例會將資料複製到 Azure 資料表中名為 "MyTable" 的資料表。 請建立資料行數目與您預期 Blob CSV 檔案要包含的數目相同的 Azure 資料表。 此資料表會每小時加入新的資料列。

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**具有 BlobSource 和 AzureTableSink 的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設定為 **BlobSource**，且 **sink** 類型設定為 **AzureTableSink**。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "AzureBlobtoTable",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureTableOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "AzureTableSink",
            "writeBatchSize": 100,
            "writeBatchTimeout": "01:00:00"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },                        
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```
## <a name="type-mapping-for-azure-table"></a>Azure 資料表的類型對應
如 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會藉由下列含有兩個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換。

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移入及移出 Azure 資料表時，從 Azure 資料表 OData 類型到 .NET 類型的對應會使用下列 [Azure 資料表服務定義的對應](https://msdn.microsoft.com/library/azure/dd179338.aspx)，反向對應亦適用。

| OData 資料類型 | .NET 類型 | 詳細資料 |
| --- | --- | --- |
| Edm.Binary |byte[] |上限為 64 KB 的位元組陣列。 |
| Edm.Boolean |布林 |布林值。 |
| Edm.DateTime |Datetime |以國際標準時間 (UTC) 表示的 64 位元值。 支援的 DateTime 範圍從西元 1601 年 1 月 1 日午夜 12:00 開始 (C.E.), UTC. 此範圍結束於 9999 年 12 月 31 日。 |
| Edm.Double |double |64 位元的浮點值。 |
| Edm.Guid |Guid |128 位元的全域唯一識別碼。 |
| Edm.Int32 |Int32 |32 位元的整數。 |
| Edm.Int64 |Int64 |64 位元的整數。 |
| Edm.String |字串 |UTF 16 編碼值。 字串值最大可達 64 KB。 |

### <a name="type-conversion-sample"></a>類型轉換範例
下列範例適用於使用類型轉換從 Azure Blob 複製資料到 Azure 資料表。

假設 Blob 資料集採用 CSV 格式而且包含三個資料行。 其中一個是含有自訂日期時間格式 (使用法文縮寫星期幾名稱) 的日期時間資料行。

請依下列方式，定義「Blob 來源」資料集及資料行的類型定義。

```JSON
{
    "name": " AzureBlobInput",
    "properties":
    {
         "structure":
          [
                { "name": "userid", "type": "Int64"},
                { "name": "name", "type": "String"},
                { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
          ],
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
            "fileName":"myfile.csv",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "external": true,
        "availability":
        {
            "frequency": "Hour",
            "interval": 1,
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
如果使用從「Azure 資料表」OData 類型到 .NET 類型的類型對應，您就會在「Azure 資料表」中定義具有下列結構描述的資料表。

**Azure 資料表結構描述：**

| 資料行名稱 | 類型 |
| --- | --- |
| userid |Edm.Int64 |
| name |Edm.String |
| lastlogindate |Edm.DateTime |

接著，依下列方式定義「Azure 資料表」資料集。 您不需要指定含有類型資訊的「結構」區段，因為類型資訊已在基礎資料存放區中指定。

```JSON
{
  "name": "AzureTableOutput",
  "properties": {
    "type": "AzureTable",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

在此情況下，Data Factory 會自動進行類型轉換，包括將資料從 Blob 移到「Azure 資料表」時，含有自訂日期時間格式 (使用 "fr-fr" 文化特性) 的 Datetime 欄位。

> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>效能和微調
若要了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。
