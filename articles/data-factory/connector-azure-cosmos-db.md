---
title: 使用 Data Factory 將資料複製到 Azure Cosmos DB 或從該處複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory 將資料從支援的來源資料存放區複製到 Azure Cosmos DB，或從 Cosmos DB 複製到支援的接收存放區。
services: data-factory, cosmosdb
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: multiple
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/28/2018
ms.author: jingwang
ms.openlocfilehash: 1afd64fbd7019164f0e1f5c850f2dcd8250cdbfc
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39600331"
---
# <a name="copy-data-to-or-from-azure-cosmos-db-using-azure-data-factory"></a>使用 Azure Data Factory 將資料複製到 Azure Cosmos DB 或從該處複製資料

> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-azure-documentdb-connector.md)
> * [目前的版本](connector-azure-cosmos-db.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Azure Cosmos DB (SQL API) 複製資料及將資料複製到該處。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Azure Cosmos DB 複製到任何支援的接收資料存放區，或將資料從任何支援的來源資料存放區複製到 Azure Cosmos DB。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Azure Cosmos DB 連接器支援：

- Cosmos DB [SQL API](https://docs.microsoft.com/azure/cosmos-db/documentdb-introduction)。
- 依原樣匯入/匯出 JSON 文件，或是從表格式資料集 (例如 SQL 資料庫、CSV 檔案等) 複製資料或將資料複製到該處。若要將文件依原樣複製到 JSON 檔案或另一個 Cosmos DB 集合，或從這些檔案或集合依原樣複製文件，請參閱[匯入/匯出 JSON 文件](#importexport-json-documents)。

Data Factory 可與 [Cosmos DB 大量執行程式庫](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started)整合，提供最理想的 Cosmos DB 寫入效能。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Azure Cosmos DB 專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Azure Cosmos DB 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**CosmosDb**。 | 是 |
| connectionString |指定連接到 Azure Cosmos DB 資料庫所需的資訊。 請注意，您必須如以下範例所示，在連接字串中指定資料庫資訊。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或「自我裝載 Integration Runtime」(如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：**

```json
{
    "name": "CosmosDbLinkedService",
    "properties": {
        "type": "CosmosDb",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Azure Cosmos DB 資料集所支援的屬性清單。

若要從 Azure Cosmos DB 複製資料或將資料複製到該處，請將資料集的類型屬性設定為 **DocumentDbCollection**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**DocumentDbCollection** |是 |
| collectionName |Cosmos DB 文件集合的名稱。 |是 |

**範例：**

```json
{
    "name": "CosmosDbDataset",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName":{
            "referenceName": "<Azure Cosmos DB linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "collectionName": "<collection name>"
        }
    }
}
```

### <a name="schema-by-data-factory"></a>Data factory 的結構描述

針對無結構描述的資料存放區 (例如 Azure Cosmos DB)，複製活動會以下列其中一種方式推斷結構描述。 因此，除非您想要[依原樣匯入/匯出 JSON 文件](#importexport-json-documents)，否則最佳做法是在 **structure** 區段中指定資料的結構。

*. 如果您是使用資料集定義中的 **structure** 屬性來定義結構，Data Factory 服務會將此結構接受為結構描述。 在此情況下，如果資料列不包含資料行的值，則會使用 null 值。
*. 如果您不是使用資料集定義中的 **structure** 屬性來指定結構，Data Factory 服務會使用資料的第一列來推斷結構描述。 在此情況下，如果第一個資料列不包含完整的結構描述，某些資料行會因複製作業而遺失。

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Azure Cosmos DB 來源和接收器所支援的屬性清單。

### <a name="azure-cosmos-db-as-source"></a>Azure Cosmos DB 作為來源

若要從 Azure Cosmos DB 複製資料，請將複製活動中的來源類型設定為 **DocumentDbCollectionSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**DocumentDbCollectionSource** |是 |
| query |指定 Cosmos DB 查詢來讀取資料。<br/><br/>範例： `SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |否 <br/><br/>如果未指定，執行的 SQL 陳述式：`select <columns defined in structure> from mycollection` |
| nestingSeparator |用來指出文件為巢狀文件及如何將結果集壓平合併的特殊字元。<br/><br/>例如，如果 Cosmos DB 查詢傳回巢狀結果 `"Name": {"First": "John"}`，當 nestedSeparator 是點號時，複製活動會將資料行名稱識別為 "Name.First" 且值為 "John"。 |否 (預設為點號 `.`) |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromCosmosDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Document DB input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DocumentDbCollectionSource",
                "query": "SELECT c.BusinessEntityID, c.Name.First AS FirstName, c.Name.Middle AS MiddleName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-cosmos-db-as-sink"></a>Azure Cosmos DB 作為接收器

若要將資料複製到 Azure Cosmos DB，請將複製活動中的接收類型設定為 **DocumentDbCollectionSink**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收的 type 屬性必須設定為：**DocumentDbCollectionSink** |是 |
| writeBehavior |說明如何將資料寫入 Cosmos DB。 允許的值為 `insert` 和 `upsert`。<br/>如果存在具有相同識別碼的文件時，**upsert** 的行為會用來取代文件；否則使用 insert。 請注意，如果原始文件中沒指定文件識別碼，或沒有透過資料行對應來指定文件識別碼，則 ADF 會自動產生識別碼，這表示您必須先確定文件有「識別碼」，才能讓 upsert 正常運作。 |否，預設值為 Insert |
| writeBatchSize | Data Factory 會使用 [Cosmos DB 大量執行程式庫](https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started)將資料寫入 Cosmos DB 中。 "writeBatchSize" 可用來控制每次提供給程式庫的文件大小。 您可以嘗試增加 writeBatchSize 來改善效能。 |否，預設值為 10,000 |
| nestingSeparator |來源資料行名稱中用來表示需要巢狀文件的特殊字元。 <br/><br/>例如，當 nestedSeparator 是點號時，輸出資料集結構中的 `Name.First` 會在 Cosmos DB 文件中產生下列 JSON 結構：`"Name": {"First": "[value maps to this column from source]"}`。 |否 (預設為點號 `.`) |

**範例：**

```json
"activities":[
    {
        "name": "CopyToCosmosDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Document DB output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DocumentDbCollectionSink",
                "writeBehavior": "upsert"
            }
        }
    }
]
```

## <a name="importexport-json-documents"></a>匯入/匯出 JSON 文件

使用此 Cosmos DB 連接器，您可以輕鬆地

* 將 JSON 文件從各種來源匯入到 Cosmos DB，包括 Azure Blob、Azure Data Lake Store 及 Azure Data Factory 所支援的其他檔案型存放區。
* 將 JSON 文件從 Cosmos DB 集合匯出至各種檔案型存放區。
* 在兩個 Cosmos DB 集合之間依原樣複製文件。

達成這種無從驗證結構描述的複製：

* 使用複製資料工具時，請核取 [依原樣匯出到 JSON 檔案或 Cosmos DB 集合] 選項。
* 使用活動撰寫時，請勿在 Cosmos DB 資料集內指定 "structure" (也稱為結構描述) 區段，也不要在複製活動的 Cosmos DB 來源/接收上指定 "nestingSeparator" 屬性。 從 JSON 檔案匯入或匯出到該處時，請在對應的檔案存放區資料及內，將格式類型指定為 "JsonFormat" 並正確地設定 "filePattern" (如需詳細資料，請參閱 [JSON 格式](supported-file-formats-and-compression-codecs.md#json-format)一節)，然後不要指定 "structure" (也稱為結構描述) 區段並略過其餘格式設定。

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
