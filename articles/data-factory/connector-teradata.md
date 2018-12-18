---
title: 使用 Azure Data Factory 從 Teradata 複製資料 | Microsoft Docs
description: 了解 Data Factory 服務的「Teradata 連接器」，此連接器可讓您將資料從 Teradata 資料庫複製到 Data Factory 所支援作為接收器的資料存放區。
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
ms.date: 02/07/2018
ms.author: jingwang
ms.openlocfilehash: a2928b202f56674c69e6431201db6d846a9feb9a
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045750"
---
# <a name="copy-data-from-teradata-using-azure-data-factory"></a>使用 Azure Data Factory 從 Teradata 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-onprem-teradata-connector.md)
> * [目前的版本](connector-teradata.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Teradata 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Teradata 資料庫複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Teradata 連接器支援：

- Teradata **12 版和更新版本**。
- 使用 **Basic** (基本) 或 **Windows** 驗證來複製資料。

## <a name="prerequisites"></a>先決條件

若要使用這個 Teradata 接收器，您必須：

- 設定一個「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。
- 在 Integration Runtime 電腦上安裝 [Teradata 的 .Net 資料提供者](http://go.microsoft.com/fwlink/?LinkId=278886) 14 版或更新版本。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Teradata 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Teradata 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**Teradata** | yes |
| 伺服器 | Teradata 伺服器的名稱。 | yes |
| authenticationType | 用來連接到 Teradata 資料庫的驗證類型。<br/>允許的值為：**Basic** (基本) 和 **Windows**。 | yes |
| username | 指定連線到 Teradata 資料庫時所要使用的使用者名稱。 | yes |
| password | 指定您為使用者名稱所指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 如[必要條件](#prerequisites)所述，必須要有一個「自我裝載 Integration Runtime」。 |yes |

**範例：**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "server": "<server>",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Teradata 資料集所支援的屬性清單。

若要從 Teradata 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**RelationalTable** | yes |
| tableName | Teradata 資料庫中的資料表名稱。 | 否 (如果已指定活動來源中的「查詢」) |

**範例：**

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Teradata 來源所支援的屬性清單。

### <a name="teradata-as-source"></a>Teradata 作為來源

若要從 Teradata 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**RelationalSource** | yes |
| query | 使用自訂 SQL 查詢來讀取資料。 例如：`"SELECT * FROM MyTable"`。 | 否 (如果已指定資料集中的 "tableName") |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromTeradata",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Teradata input dataset name>",
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
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-teradata"></a>Teradata 的資料類型對應

從 Teradata 複製資料時，會使用下列從 Teradata 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| Teradata 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| BigInt |Int64 |
| Blob |Byte[] |
| Byte |Byte[] |
| ByteInt |Int16 |
| Char |字串 |
| Clob |字串 |
| 日期 |Datetime |
| 十進位 |十進位 |
| 雙精準度 |雙精準度 |
| 圖形 |字串 |
| 整數  |Int32 |
| 間隔日 |時間範圍 |
| 間隔日至小時 |時間範圍 |
| 間隔日至分鐘 |時間範圍 |
| 間隔日至秒鐘 |時間範圍 |
| 間隔小時 |時間範圍 |
| 間隔小時至分鐘 |時間範圍 |
| 間隔小時至秒鐘 |時間範圍 |
| 間隔分鐘 |時間範圍 |
| 間隔分鐘至秒鐘 |時間範圍 |
| 間隔月 |字串 |
| 間隔第二 |時間範圍 |
| 間隔年 |字串 |
| 間隔年至月 |字串 |
| 數字 |雙精確度 |
| Period(Date) |字串 |
| Period(Time) |字串 |
| Period(Time With Time Zone) |字串 |
| Period(Timestamp) |字串 |
| Period(Timestamp With Time Zone) |字串 |
| SmallInt |Int16 |
| 時間 |時間範圍 |
| 時區的時間 |字串 |
| Timestamp |Datetime |
| 時區的時間戳記 |DateTimeOffset |
| VarByte |Byte[] |
| VarChar |字串 |
| VarGraphic |字串 |
| xml |字串 |


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
