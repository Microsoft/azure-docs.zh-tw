---
title: 使用 Azure Data Factory 從 OData 來源複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 OData 來源複製到支援的接收資料存放區。
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
ms.date: 05/22/2018
ms.author: jingwang
ms.openlocfilehash: aaec710dd6c12f96a479a1f41603351512da1df6
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054665"
---
# <a name="copy-data-from-odata-source-using-azure-data-factory"></a>使用 Azure Data Factory 從 OData 來源複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-odata-connector.md)
> * [目前的版本](connector-odata.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 OData 來源複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 OData 來源複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 OData 連接器支援：

- OData **3.0 和 4.0 版**。
- 使用下列驗證來複製資料︰**Anonymous** (匿名)、**Basic** (基本) 及 **Windows**。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 OData 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>已連結的服務屬性

以下是針對 OData 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為： **OData** |yes |
| url | OData 服務的根 URL。 |yes |
| authenticationType | 用來連線到 OData 來源的驗證類型。<br/>允許的值為：**Anonymous** (匿名)、**Basic** (基本) 及 **Windows**。 請注意，不支援 OAuth。 | yes |
| userName | 如果您使用基本或 Windows 驗證，請指定使用者名稱。 | 否 |
| password | 指定您為 userName 指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 否 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或「自我裝載 Integration Runtime」(如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例 1：使用 Anonymous (匿名) 驗證**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**範例 2：使用 Basic (基本) 驗證**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
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

**範例 3：使用 Windows 驗證**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of on-premises OData source>",
            "authenticationType": "Windows",
            "userName": "<domain>\\<user>",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 OData 資料集所支援的屬性清單。

若要從 OData 複製資料，請將資料集的類型屬性設定為 **ODataResource**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**ODataResource** | yes |
| path | OData 資源的路徑。 | yes |

**範例**

```json
{
    "name": "ODataDataset",
    "properties":
    {
        "type": "ODataResource",
        "linkedServiceName": {
            "referenceName": "<OData linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties":
        {
            "path": "Products"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 OData 來源所支援的屬性清單。

### <a name="odata-as-source"></a>OData 作為來源

若要從 OData 複製資料，請將複製活動中的來源類型設定為 **RelationalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**RelationalSource** | yes |
| query | 用來篩選資料的 OData 查詢選項。 範例："?$select=Name,Description&$top=5"。<br/><br/>最後，請注意，OData 連接器會從以下的組合 URL 複製資料：`[url specified in linked service]/[path specified in dataset][query specified in copy activity source]`。 請參考 [OData URL 元件](http://www.odata.org/documentation/odata-version-3-0/url-conventions/) \(英文\)。 | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromOData",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<OData input dataset name>",
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
                "query": "?$select=Name,Description&$top=5"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-odata"></a>OData 的資料類型對應

從 OData 複製資料時，會使用下列從 OData 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| OData 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| Edm.Binary | Byte[] |
| Edm.Boolean | Bool |
| Edm.Byte | Byte[] |
| Edm.DateTime | Datetime |
| Edm.Decimal | 十進位 |
| Edm.Double | 兩倍 |
| Edm.Single | 單一 |
| Edm.Guid | Guid |
| Edm.Int16 | Int16 |
| Edm.Int32 | Int32 |
| Edm.Int64 | Int64 |
| Edm.SByte | Int16 |
| Edm.String | 字串 |
| Edm.Time | 時間範圍 |
| Edm.DateTimeOffset | DateTimeOffset |

> [!Note]
> 不支援 OData 複雜資料類型 (例如「物件」)。


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
