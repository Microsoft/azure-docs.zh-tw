---
title: 從 Azure Cosmos DB 來回移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Azure Cosmos DB 集合來回移動資料
services: data-factory, cosmosdb
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: c9297b71-1bb4-4b29-ba3c-4cf1f5575fac
ms.service: multiple
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: f3ebd704129aabecffdaa2589b8b086803a2d092
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046593"
---
# <a name="move-data-to-and-from-azure-cosmos-db-using-azure-data-factory"></a>使用 Azure Data Factory 從 Azure Cosmos DB 來回移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-azure-documentdb-connector.md)
> * [第 2 版 (目前的版本)](../connector-azure-cosmos-db.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Azure Cosmos DB 連接器](../connector-azure-cosmos-db.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，將資料移進/移出 Azure Cosmos DB (SQL API)。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。 

您可以將資料從任何支援的來源資料存放區複製到 Azure Cosmos DB，或從 Azure Cosmos DB 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 

> [!IMPORTANT]
> Azure Cosmos DB 連接器只支援 SQL API。

若要將資料依原樣複製到 JSON 檔案或另一個 Cosmos DB 集合，或從這些檔案或集合依原樣複製資料，請參閱[匯入/匯出 JSON 文件](#importexport-json-documents)。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出 Azure Cosmos DB。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到 Cosmos DB (或從 Cosmos DB 複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples)一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Cosmos DB 特定的 Data Factory 實體： 

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 Azure Cosmos DB 連結服務專屬 JSON 元素的描述。

| **屬性** | **說明** | **必要** |
| --- | --- | --- |
| type |類型屬性必須設為： **DocumentDb** |yes |
| connectionString |指定連接到 Azure Cosmos DB 資料庫所需的資訊。 |yes |

範例：

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需可用來定義資料集的完整區段和屬性清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。 **DocumentDbCollection** 類型資料集的 typeProperties 區段具有下列屬性。

| **屬性** | **說明** | **必要** |
| --- | --- | --- |
| collectionName |Cosmos DB 文件集合的名稱。 |yes |

範例：

```JSON
{
  "name": "PersonCosmosDbTable",
  "properties": {
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
### <a name="schema-by-data-factory"></a>Data factory 的結構描述
針對無結構描述的資料存放區 (如 Azure Cosmos DB)，Data Factory 服務會以下列一種方式推斷結構描述：  

1. 如果您是使用資料集定義中的 **structure** 屬性來定義結構，Data Factory 服務會將此結構接受為結構描述。 在此情況下，如果資料列不包含資料行的值，則會使用 null 值。
2. 如果您不是使用資料集定義中的 **structure** 屬性來指定結構，Data Factory 服務會使用資料的第一列來推斷結構描述。 在此情況下，如果第一個資料列不包含完整的結構描述，某些資料行會因複製作業而遺失。

因此，對於無結構描述的資料來源來說，最佳作法是使用 **structure** 屬性來指定資料結構。

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [!NOTE]
> 複製活動只會採用一個輸入，而且只產生一個輸出。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同，而在複製活動的案例中，可用的屬性會根據來源與接收的類型而有所不同。

在複製活動的案例中，如果來源類型為 **DocumentDbCollectionSource**，則 **typeProperties** 區段可使用下列屬性：

| **屬性** | **說明** | **允許的值** | **必要** |
| --- | --- | --- | --- |
| query |指定查詢來讀取資料。 |Azure Cosmos DB 所支援的查詢字串。 <br/><br/>範例： `SELECT c.BusinessEntityID, c.PersonType, c.NameStyle, c.Title, c.Name.First AS FirstName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |否 <br/><br/>如果未指定，執行的 SQL 陳述式：`select <columns defined in structure> from mycollection` |
| nestingSeparator |用來表示文件為巢狀文件的特殊字元 |任何字元。 <br/><br/>Azure Cosmos DB 是 JSON 文件的 NoSQL 存放區 (允許巢狀結構)。 Azure Data Factory 可讓使用者透過 nestingSeparator (也就是上述範例中的 “.”) 表示階層 。 使用分隔符號，複製活動將會根據資料表定義中的 “Name.First”、“Name.Middle” 和 “Name.Last”，產生含有三個子元素 (First、Middle 和 Last) 的 "Name" 物件。 |否 |

**DocumentDbCollectionSink** 支援下列屬性：

| **屬性** | **說明** | **允許的值** | **必要** |
| --- | --- | --- | --- |
| nestingSeparator |來源資料行名稱中用來表示需要巢狀文件的特殊字元。 <br/><br/>就上述範例而言：輸出資料表中的 `Name.First` 會在 Cosmos DB 文件中產生下列 JSON 結構：<br/><br/>"Name": {<br/>    "First": "John"<br/>}, |用來分隔巢狀層級的字元。<br/><br/>預設值為 `.` (點)。 |用來分隔巢狀層級的字元。 <br/><br/>預設值為 `.` (點)。 |
| writeBatchSize |為了建立文件而傳送到 Azure Cosmos DB 服務的平行要求數目。<br/><br/>使用這個屬性從 Cosmos DB 來回複製資料時，可以微調效能。 增加 writeBatchSize 時，您可預期有更好的效能，因為對 Cosmos DB 傳送了更多的平行要求。 不過，您必須避免可能擲回錯誤訊息的節流：「要求速率很高」。<br/><br/>節流是由許多因素所決定，包括文件大小、文件中的詞彙數目、目標集合的檢索原則等。對於複製作業，您可以使用更好的集合 (例如 S3) 以取得最多可用輸送量 (2,500 要求單位/秒)。 |整數  |否 (預設值：5) |
| writeBatchTimeout |在逾時前等待作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |

## <a name="importexport-json-documents"></a>匯入/匯出 JSON 文件
使用此 Cosmos DB 連接器，您可以輕鬆地

* 將 JSON 文件從各種來源匯入到 Cosmos DB，包括 Azure Blob、Azure Data Lake、內部部署的檔案系統，或 Azure Data Factory 所支援的其他檔案型存放區。
* 將 JSON 文件從 Cosmos DB 集合匯出至各種檔案型存放區。
* 在兩個 Cosmos DB 集合之間依原樣移轉資料。

達成這種無從驗證結構描述的複製 
* 使用複製精靈時，核取 [依原樣匯出到 JSON 檔案或 Cosmos DB 集合] 選項。
* 使用 JSON 編輯時，請勿在 Cosmos DB 資料集中指定 "structure" 區段，也不要在複製活動的 Cosmos DB 來源/接收器上指定 "nestingSeparator" 屬性。 若要從 JSON 檔案匯入或匯出到這些檔案，請在檔案存放區資料集內，將格式類型指定為 "JsonFormat"、設定 "filePattern"，然後略過其餘格式設定，如需詳細資料，請參閱 [JSON 格式](data-factory-supported-file-and-compression-formats.md#json-format)一節。

## <a name="json-examples"></a>JSON 範例
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 它們示範如何將資料複製到 Azure Cosmos DB 和 Azure Blob 儲存體，以及從中複製資料。 不過，您可以使用 Azure Data Factory 中的「複製活動」，將資料從任何來源「直接」複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats)所述的任何接收器。

## <a name="example-copy-data-from-azure-cosmos-db-to-azure-blob"></a>範例：將資料從 Azure Cosmos DB 複製到 Azure Blob
下列範例顯示：

1. [DocumentDb](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [DocumentDbCollection](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [DocumentDbCollectionSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會將 Azure Cosmos DB 中的資料複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure Cosmos DB 連結服務︰**

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```
**Azure Blob 儲存體連結服務：**

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
**Azure DocumentDB 輸入資料集：**

此範例假設您在 Azure Cosmos DB 資料庫中擁有名為 Person 的集合。

設定 “external”: ”true” 和指定 externalData 原則即可通知 Data Factory 服務：這是 Azure Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "PersonCosmosDbTable",
  "properties": {
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Azure Blob 輸出資料集：**

資料會每小時利用可反映特定日期時間及小時資料粒度的 Blob 路徑，複製到新的 Blob。

```JSON
{
  "name": "PersonBlobTableOut",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "docdb",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "nullValue": "NULL"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
Cosmos DB 資料庫中 Person 集合中的範例 JSON 文件：

```JSON
{
  "PersonId": 2,
  "Name": {
    "First": "Jane",
    "Middle": "",
    "Last": "Doe"
  }
}
```
Cosmos DB 支援在階層式 JSON 文件上使用類似 SQL 的語法來查詢文件。

範例： 

```sql
SELECT Person.PersonId, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person
```

下列管線會將資料從 Azure Cosmos DB 資料庫中的 Person 集合複製到 Azure Blob。 複製活動中已指定為輸入和輸出資料集。  

```JSON
{
  "name": "DocDbToBlobPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "DocumentDbCollectionSource",
            "query": "SELECT Person.Id, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person",
            "nestingSeparator": "."
          },
          "sink": {
            "type": "BlobSink",
            "blobWriterAddHeader": true,
            "writeBatchSize": 1000,
            "writeBatchTimeout": "00:00:59"
          }
        },
        "inputs": [
          {
            "name": "PersonCosmosDbTable"
          }
        ],
        "outputs": [
          {
            "name": "PersonBlobTableOut"
          }
        ],
        "policy": {
          "concurrency": 1
        },
        "name": "CopyFromDocDbToBlob"
      }
    ],
    "start": "2015-04-01T00:00:00Z",
    "end": "2015-04-02T00:00:00Z"
  }
}
```
## <a name="example-copy-data-from-azure-blob-to-azure-cosmos-db"></a>範例：將資料從 Azure Blob 複製到 Azure Cosmos DB 
下列範例顯示：

1. [DocumentDb](#azure-documentdb-linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [DocumentDbCollection](#azure-documentdb-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [DocumentDbCollectionSink](#azure-documentdb-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例將資料從 Azure Blob 複製到 Azure Cosmos DB。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure Blob 儲存體連結服務：**

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
**Azure Cosmos DB 連結服務︰**

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```
**Azure Blob 輸入資料集：**

```JSON
{
  "name": "PersonBlobTableIn",
  "properties": {
    "structure": [
      {
        "name": "Id",
        "type": "Int"
      },
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "MiddleName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "fileName": "input.csv",
      "folderPath": "docdb",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "nullValue": "NULL"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
**Azure Cosmos DB 輸出資料集︰**

此範例會將資料複製到名為 "Person" 的集合。

```JSON
{
  "name": "PersonCosmosDbTableOut",
  "properties": {
    "structure": [
      {
        "name": "Id",
        "type": "Int"
      },
      {
        "name": "Name.First",
        "type": "String"
      },
      {
        "name": "Name.Middle",
        "type": "String"
      },
      {
        "name": "Name.Last",
        "type": "String"
      }
    ],
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
下列管線會將資料從 Azure Blob 複製到 Cosmos DB 資料庫中的 Person 集合。 複製活動中已指定為輸入和輸出資料集。

```JSON
{
  "name": "BlobToDocDbPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "DocumentDbCollectionSink",
            "nestingSeparator": ".",
            "writeBatchSize": 2,
            "writeBatchTimeout": "00:00:00"
          }
          "translator": {
              "type": "TabularTranslator",
              "ColumnMappings": "FirstName: Name.First, MiddleName: Name.Middle, LastName: Name.Last, BusinessEntityID: BusinessEntityID, PersonType: PersonType, NameStyle: NameStyle, Title: Title, Suffix: Suffix, EmailPromotion: EmailPromotion, rowguid: rowguid, ModifiedDate: ModifiedDate"
          }
        },
        "inputs": [
          {
            "name": "PersonBlobTableIn"
          }
        ],
        "outputs": [
          {
            "name": "PersonCosmosDbTableOut"
          }
        ],
        "policy": {
          "concurrency": 1
        },
        "name": "CopyFromBlobToDocDb"
      }
    ],
    "start": "2015-04-14T00:00:00Z",
    "end": "2015-04-15T00:00:00Z"
  }
}
```
如果範例 Blob 輸入為

```
1,John,,Doe
```
則 Cosmos DB 中的輸出 JSON 會是：

```JSON
{
  "Id": 1,
  "Name": {
    "First": "John",
    "Middle": null,
    "Last": "Doe"
  },
  "id": "a5e8595c-62ec-4554-a118-3940f4ff70b6"
}
```
Azure Cosmos DB 是 JSON 文件的 NoSQL 存放區 (允許巢狀結構)。 Azure Data Factory 可讓使用者透過 **nestingSeparator** (也就是此範例中的 “.”) 表示階層 。 使用分隔符號，複製活動將會根據資料表定義中的 “Name.First”、“Name.Middle” 和 “Name.Last”，產生含有三個子元素 (First、Middle 和 Last) 的 "Name" 物件。

## <a name="appendix"></a>附錄
1. **問：** 複製活動支援現有記錄的更新嗎？

    **答：** 否。
2. **問：** 重試複製 Azure Cosmos DB 時，會如何處理已經複製的記錄？

    **回：** 如果記錄有 [識別碼] 欄位，而複製作業嘗試插入具有相同識別碼的記錄，則複製作業會擲回錯誤。  
3. **問：** Data Factory 支援[範圍或雜湊式資料分割](../../cosmos-db/sql-api-partition-data.md)嗎？

    **答：** 否。
4. **問：** 我可以為資料表指定多個 Azure Cosmos DB 集合嗎？

    **答：** 否。 目前只能指定一個集合。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
