---
title: 使用 Azure Data Factory 從 MongoDB 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 MongoDB 複製到支援的接收資料存放區。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/05/2018
ms.author: jingwang
ms.openlocfilehash: debb27f49c730df4a8bef42b1f1ef9ec50f1faf0
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054053"
---
# <a name="copy-data-from-mongodb-using-azure-data-factory"></a>使用 Azure Data Factory 從 MongoDB 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-on-premises-mongodb-connector.md)
> * [目前的版本](connector-mongodb.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 MongoDB 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 MongoDB 資料庫複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 MongoDB 連接器支援：

- MongoDB **2.4、2.6、3.0、3.2、3.4、3.6 版**。
- 使用 **Basic** (基本) 或 **Anonymous** (匿名) 驗證來複製資料。

## <a name="prerequisites"></a>先決條件

若要從不可公開存取的 MongoDB 資料庫複製資料，您必須設定一個「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。 Integration Runtime 提供內建的 MongoDB 驅動程式，因此從 MongoDB 複製資料時，您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 MongoDB 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 MongoDB 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |類型屬性必須設定為：**MongoDb** |yes |
| 伺服器 |MongoDB 伺服器的 IP 位址或主機名稱。 |yes |
| 連接埠 |MongoDB 伺服器用來接聽用戶端連線的 TCP 連接埠。 |否 (預設值為 27017) |
| databaseName |您想要存取之 MongoDB 資料庫的名稱。 |yes |
| authenticationType | 用來連線到 MongoDB 資料庫的驗證類型。<br/>允許的值為：**Basic** (基本) 和 **Anonymous** (匿名)。 |yes |
| username |用來存取 MongoDB 的使用者帳戶。 |是 (如果使用基本驗證)。 |
| password |使用者的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 (如果使用基本驗證)。 |
| authSource |您想要用來檢查驗證所用之認證的 MongoDB 資料庫名稱。 |否。 就基本驗證而言，預設會使用以 databaseName 屬性指定的系統管理員帳戶和資料庫。 |
| enableSsl | 指定是否使用 SSL 來加密與伺服器的連線。 預設值為 False。  | 否 |
| allowSelfSignedServerCert | 指定是否允許來自伺服器的自我簽署憑證。 預設值為 False。  | 否 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：**

```json
{
    "name": "MongoDBLinkedService",
    "properties": {
        "type": "MongoDb",
        "typeProperties": {
            "server": "<server name>",
            "databaseName": "<database name>",
            "authenticationType": "Basic",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 MongoDB 資料集所支援的屬性清單。

若要從 MongoDB 複製資料，請將資料集的類型屬性設定為 **MongoDbCollection**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**MongoDbCollection** | yes |
| collectionName |MongoDB 資料庫中集合的名稱。 |yes |

**範例：**

```json
{
     "name":  "MongoDbDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": {
            "referenceName": "<MongoDB linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "collectionName": "<Collection name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 MongoDB 來源所支援的屬性清單。

### <a name="mongodb-as-source"></a>MongoDB 作為來源

若要從 MongoDB 複製資料，請將複製活動中的來源類型設定為 **MongoDbSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**MongoDbSource** | yes |
| query |使用自訂的 SQL-92 查詢來讀取資料。 例如：select * from MyTable。 |否 (如果已指定資料集中 "collectionName") |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromMongoDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<MongoDB input dataset name>",
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
                "type": "MongoDbSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

> [!TIP]
> 指定 SQL 查詢時，請注意 DateTime 格式。 例如：`SELECT * FROM Account WHERE LastModifiedDate >= '2018-06-01' AND LastModifiedDate < '2018-06-02'`，或使用參數 `SELECT * FROM Account WHERE LastModifiedDate >= '@{formatDateTime(pipeline().parameters.StartTime,'yyyy-MM-dd HH:mm:ss')}' AND LastModifiedDate < '@{formatDateTime(pipeline().parameters.EndTime,'yyyy-MM-dd HH:mm:ss')}'`

## <a name="schema-by-data-factory"></a>Data factory 的結構描述

Azure Data Factory 服務會使用 MongoDB 集合中**最新的 100 份文件**，從該集合推斷結構描述。 如果這 100 份文件未包含完整結構描述，則在複製作業期間可能會忽略某些資料行。

## <a name="data-type-mapping-for-mongodb"></a>MongoDB 的資料類型對應

從 MongoDB 複製資料時，會使用下列從 MongoDB 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| MongoDB 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| Binary |Byte[] |
| BOOLEAN |BOOLEAN |
| 日期 |Datetime |
| NumberDouble |兩倍 |
| NumberInt |Int32 |
| NumberLong |Int64 |
| ObjectID |字串 |
| 字串 |字串 |
| UUID |Guid |
| Object |以 "_" 作為巢狀分隔符號來重新標準化為壓平合併資料行 |

> [!NOTE]
> 若要了解對使用虛擬資料表之陣列的支援，請參閱[對使用虛擬資料表之複雜類型的支援](#support-for-complex-types-using-virtual-tables)一節。
>
> 目前不支援下列 MongoDB 資料類型︰DBPointer、JavaScript、Max/Min 索引鍵、規則運算式、符號、時間戳記、未定義。

## <a name="support-for-complex-types-using-virtual-tables"></a>使用虛擬資料表之複雜類型的支援

Azure Data Factory 會使用內建的 ODBC 驅動程式來連線到 MongoDB 資料庫並從中複製資料。 對於其類型會跨文件而不同的陣列或物件等複雜類型，驅動程式會將資料重新標準化為對應的虛擬資料表。 具體來說，如果資料表包含這類資料行，則驅動程式會產生下列虛擬資料表︰

* **基底資料表**，其中包含與實際資料表相同的資料 (複雜類型資料行除外)。 基底資料表使用與它所代表的實際資料表相同的名稱。
* 每個複雜類型資料行的 **虛擬資料表** ，以展開巢狀資料。 虛擬資料表會使用實際資料表的名稱、分隔字元 "_" 及陣列或物件的名稱來命名。

虛擬資料表會參考實際資料表中的資料，讓驅動程式得以存取反正規化的資料。 您可以藉由查詢和聯結虛擬資料表來存取 MongoDB 陣列的內容。

### <a name="example"></a>範例

例如，以下的 ExampleTable 是一個 MongoDB 資料表，其中包含一個在每個儲存格中都有物件陣列的資料行 (發票)，以及一個具有純量類型陣列的資料行 (評等)。

| _id | 客戶名稱 | 發票 | 服務等級 | 評等 |
| --- | --- | --- | --- | --- |
| 1111 |ABC |[{invoice_id:"123", item:"toaster", price:"456", discount:"0.2"}, {invoice_id:"124", item:"oven", price: "1235", discount: "0.2"}] |Silver |[5,6] |
| 2222 |XYZ |[{invoice_id:"135", item:"fridge", price: "12543", discount: "0.0"}] |Gold |[1,2] |

驅動程式會產生多個代表這個單一資料表的虛擬資料表。 第一個虛擬資料表是名為 "ExampleTable" 的基底資料表，如範例所示。 基底資料表包含原始資料表的所有資料，但來自陣列的資料已省略，並且會在虛擬資料表中展開。

| _id | 客戶名稱 | 服務等級 |
| --- | --- | --- |
| 1111 |ABC |Silver |
| 2222 |XYZ |Gold |

下表顯示代表範例中原始陣列的虛擬資料表。 這些資料表包含下列項目：

* 對應到原始陣列資料列 (透過 _id 資料行) 之原始主要索引鍵資料行的往回參考
* 資料在原始陣列之位置的指示
* 陣列內每個元素的展開資料

**"ExampleTable_Invoices" 資料表：**

| _id | ExampleTable_Invoices_dim1_idx | invoice_id | item | price | 折扣 |
| --- | --- | --- | --- | --- | --- |
| 1111 |0 |123 |toaster |456 |0.2 |
| 1111 |1 |124 |oven |1235 |0.2 |
| 2222 |0 |135 |fridge |12543 |0.0 |

**"ExampleTable_Ratings" 資料表：**

| _id | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings |
| --- | --- | --- |
| 1111 |0 |5 |
| 1111 |1 |6 |
| 2222 |0 |1 |
| 2222 |1 |2 |


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
