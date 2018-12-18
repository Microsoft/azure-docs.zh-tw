---
title: 使用 Azure Data Factory 從 DB2 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從 DB2 將資料複製到支援的接收資料存放區。
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
ms.date: 08/17/2018
ms.author: jingwang
ms.openlocfilehash: f9d1d2181649cf24784dc7ad11638946c9ee4406
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42145544"
---
# <a name="copy-data-from-db2-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 DB2 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-onprem-db2-connector.md)
> * [目前的版本](connector-db2.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 DB2 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以從 DB2 資料庫將資料複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此 DB2 連接器支援以下 IBM DB2 平台和版本，以及支援分散式關聯資料庫架構 (DRDA) SQL 存取管理員 (SQLAM) 版本 9、10 和 11：

* IBM DB2 for z/OS 11.1
* IBM DB2 for z/OS 10.1
* IBM DB2 for i 7.2
* IBM DB2 for i 7.1
* IBM DB2 for LUW 11
* IBM DB2 for LUW 10.5
* IBM DB2 for LUW 10.1

> [!TIP]
> 如果收到錯誤訊息表示：「找不到對應至 SQL 陳述式執行要求的套件。 SQLSTATE=51002 SQLCODE=-805」，原因是這類作業系統上未針對一般使用者建立所需的套件。 請根據您的 DB2 伺服器類型，遵循下面的指示作業：
> - DB2 for i (AS400)：使用複製活動之前，讓進階使用者建立登入使用者集合。 命令：`create collection <username>`
> - DB2 for z/OS 或 LUW：使用高權限帳戶 - 具有封裝授權單位與 BIND、BINDADD、GRANT EXECUTE TO PUBLIC 權限的進階使用者或管理員 - 執行一次複製活動，然後就會在複製期間自動建立所需的封裝。 之後，您可以切換至一般使用者，來執行後續的複製。

## <a name="prerequisites"></a>必要條件

若要從不可公開存取的 DB2 資料庫複製資料，您應該設定一個「自我裝載整合執行階段」。 若要了解自我裝載整合執行階段，請參閱[自我裝載整合執行階段](create-self-hosted-integration-runtime.md)一文。 「整合執行階段」提供內建的 DB2 驅動程式，因此從 DB2 複製資料時，您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 DB2 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 DB2 連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為：**DB2** | 是 |
| 伺服器 |DB2 伺服器的名稱。 您可以指定在伺服器名稱後面加上以冒號隔開的連接埠號碼，例如 `server:port`。 |是 |
| 資料庫 |DB2 資料庫的名稱。 |是 |
| authenticationType |用來連接到 DB2 資料庫的驗證類型。<br/>允許的值為**基本**。 |是 |
| username |指定要連線到 DB2 資料庫的使用者名稱。 |是 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：**

```json
{
    "name": "Db2LinkedService",
    "properties": {
        "type": "Db2",
        "typeProperties": {
            "server": "<servername:port>",
            "database": "<dbname>",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 DB2 資料集所支援的屬性清單。

若要從 DB2 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**RelationalTable** | 是 |
| tableName | DB2 資料庫中的表格名稱。 | 否 (如果已指定活動來源中的「查詢」) |

**範例**

```json
{
    "name": "DB2Dataset",
    "properties":
    {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<DB2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 DB2 來源所支援的屬性清單。

### <a name="db2-as-source"></a>DB2 作為來源

若要從 DB2 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**RelationalSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""` 。 | 否 (如果已指定資料集中的 "tableName") |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromDB2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<DB2 input dataset name>",
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
                "type": "RelationalSource",
                "query": "SELECT * FROM \"DB2ADMIN\".\"Customers\""
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-db2"></a>DB2 的資料類型對應

從 DB2 複製資料時，會使用下列從 DB2 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| DB2 資料庫類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| BigInt |Int64 |
| Binary |Byte[] |
| Blob |Byte[] |
| Char |字串 |
| Clob |字串 |
| 日期 |DateTime |
| DB2DynArray |字串 |
| DbClob |字串 |
| 十進位 |十進位 |
| DecimalFloat |十進位 |
| 兩倍 |兩倍 |
| Float |兩倍 |
| 圖形 |字串 |
| 整數  |Int32 |
| LongVarBinary |Byte[] |
| LongVarChar |字串 |
| LongVarGraphic |字串 |
| 數值 |十進位 |
| Real |單一 |
| SmallInt |Int16 |
| 時間 |時間範圍 |
| Timestamp |Datetime |
| VarBinary |Byte[] |
| VarChar |字串 |
| VarGraphic |字串 |
| xml |Byte[] |


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
