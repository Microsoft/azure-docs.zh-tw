---
title: 使用 Azure Data Factory 從 SAP HANA 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 SAP HANA 複製到支援的接收資料存放區。
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
ms.openlocfilehash: 1ded69225319e447ad210aed267741b2803889ac
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048077"
---
# <a name="copy-data-from-sap-hana-using-azure-data-factory"></a>使用 Azure Data Factory 從 SAP HANA 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-sap-hana-connector.md)
> * [目前的版本](connector-sap-hana.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 SAP HANA 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 SAP HANA 資料庫複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 SAP HANA 連接器支援：

- 從任何版本的 SAP HANA 資料庫複製資料。
- 使用 SQL 查詢從 **HANA 資訊模型** (例如「分析」和「計算」檢視) 和**資料列/資料行資料表**複製資料。
- 使用 **Basic** (基本) 或 **Windows** 驗證來複製資料。

> [!NOTE]
> 若要將資料複製**到** SAP HANA 資料存放區，請使用一般 ODBC 連接器。 如需詳細資料，請參閱 [SAP HANA 接收器](connector-odbc.md#sap-hana-sink)。 請注意，SAP HANA 連接器和 ODBC 連接器的已連接服務具有不同的類型，因此無法重複使用。

## <a name="prerequisites"></a>先決條件

若要使用這個 SAP HANA 接收器，您必須：

- 設定一個「自我裝載 Integration Runtime」。 如需詳細資料，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 一文。
- 在 Integration Runtime 電腦上安裝 SAP HANA ODBC 驅動程式。 您可以從 [SAP 軟體下載中心](https://support.sap.com/swdc)下載 SAP Hana ODBC 驅動程式。 使用關鍵字 **SAP HANA CLIENT for Windows** 搜尋。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 SAP HANA 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 SAP HANA 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**SapHana** | yes |
| 伺服器 | SAP Hana 執行個體所在之伺服器的名稱。 如果您的伺服器使用自訂連接埠，指定 `server:port`。 | yes |
| authenticationType | 用來連線到 SAP HANA 資料庫的驗證類型。<br/>允許的值為：**Basic** (基本) 和 **Windows** | yes |
| userName | 能夠存取 SAP 伺服器的使用者名稱。 | yes |
| password | 使用者的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 如[必要條件](#prerequisites)所述，必須要有一個「自我裝載 Integration Runtime」。 |yes |

**範例：**

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server>:<port (optional)>",
            "authenticationType": "Basic",
            "userName": "<username>",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 SAP HANA 資料集所支援的屬性清單。

若要從 SAP HANA 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 針對 RelationalTable 類型的 SAP HANA 資料集，不支援任何類型特定的屬性。

**範例：**

```json
{
    "name": "SAPHANADataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<SAP HANA linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 SAP HANA 來源所支援的屬性清單。

### <a name="sap-hana-as-source"></a>SAP HANA 作為來源

若要從 SAP HANA 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**RelationalSource** | yes |
| query | 指定 SQL 查詢從 SAP HANA 執行個體讀取資料。 | yes |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSAPHANA",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP HANA input dataset name>",
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
                "query": "<SQL query for SAP HANA>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-hana"></a>SAP HANA 的資料類型對應

從 SAP HANA 複製資料時，會使用下列從 SAP HANA 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| SAP HANA 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| ALPHANUM | 字串 |
| BIGINT | Int64 |
| BLOB | Byte[] |
| BOOLEAN | Byte |
| CLOB | Byte[] |
| 日期 | Datetime |
| DECIMAL | 十進位 |
| DOUBLE | 單一 |
| INT | Int32 |
| NVARCHAR | 字串 |
| REAL | 單一 |
| SECONDDATE | Datetime |
| SMALLINT | Int16 |
| TIME | 時間範圍 |
| 時間戳記 | Datetime |
| TINYINT | Byte |
| VARCHAR | 字串 |

## <a name="known-limitations"></a>已知限制

從 SAP HANA 複製資料時，有幾個已知的限制︰

- NVARCHAR 字串會截斷成 4000 個 Unicode 字元的最大長度
- 不支援 SMALLDECIMAL
- 不支援 VARBINARY
- 有效日期為 1899/12/30 到 9999/12/31 之間


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
