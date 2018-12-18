---
title: 使用 Azure Data Factory 從 SAP Cloud for Customer 來回複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory，將資料從 SAP Cloud for Customer 複製到支援的接收資料存放區，(或) 從支援的來源資料存放區複製到 SAP Cloud for Customer。
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
ms.date: 04/17/2018
ms.author: jingwang
ms.openlocfilehash: df45613105c8fb005fc8ba0c796ef768e293c57e
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37052427"
---
# <a name="copy-data-from-sap-cloud-for-customer-c4c-using-azure-data-factory"></a>使用 Azure Data Factory 從 SAP Cloud for Customer (C4C) 複製資料

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 SAP Cloud for Customer (C4C) 來回複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 SAP Cloud for Customer 複製到任何支援的接收資料存放區，或將資料從任何支援的來源資料存放區複製到 SAP Cloud for Customer。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此連接器可讓 Azure Data Factory 從 SAP Cloud for Customer 來回複製資料，其中包括 SAP Cloud for Sales、SAP Cloud for Service 及 SAP Cloud for Social Engagement 解決方案。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 SAP Cloud for Customer 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 SAP Cloud for Customer 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為：**SapCloudForCustomer**。 | yes |
| url | SAP C4C OData 服務的 URL。 | yes |
| username | 指定要連線到 SAP C4C 的使用者名稱。 | yes |
| password | 指定您為 username 指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 如果未指定，就會使用預設的 Azure Integration Runtime。 | 否 (來源)；是 (接收) |

>[!IMPORTANT]
>若要將資料複製到 SAP Cloud for Customer，請以您 SAP Cloud for Customer 附近的位置明確[建立 Azure IR](create-azure-integration-runtime.md#create-azure-ir)，並在已連結的服務中產生關聯，如下列範例所示：

**範例：**

```json
{
    "name": "SAPC4CLinkedService",
    "properties": {
        "type": "SapCloudForCustomer",
        "typeProperties": {
            "url": "https://<tenantname>.crm.ondemand.com/sap/c4c/odata/v1/c4codata/" ,
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

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 SAP Cloud for Customer 資料集所支援的屬性清單。

若要從 SAP Cloud for Customer 複製資料，請將資料集的 type 屬性設定為 **SapCloudForCustomerResource**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的 type 屬性必須設定為：**SapCloudForCustomerResource** |yes |
| path | 指定 SAP C4C OData 實體的路徑。 |yes |

**範例：**

```json
{
    "name": "SAPC4CDataset",
    "properties": {
        "type": "SapCloudForCustomerResource",
        "typePoperties": {
            "path": "<path e.g. LeadCollection>"
        },
        "linkedServiceName": {
            "referenceName": "<SAP C4C linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 SAP Cloud for Customer 來源所支援的屬性清單。

### <a name="sap-c4c-as-source"></a>將 SAP C4C 作為來源

若要從 SAP Cloud for Customer 複製資料，請將複製活動中的來源類型設定為 **SapCloudForCustomerSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為：**SapCloudForCustomerSource**  | yes |
| query | 指定自訂 OData 查詢來讀取資料。 | 否 |

取得特定日之資料的範例查詢：`"query": "$filter=CreatedOn ge datetimeoffset'2017-07-31T10:02:06.4202620Z' and CreatedOn le datetimeoffset'2017-08-01T10:02:06.4202620Z'"`

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSAPC4C",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP C4C input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SapCloudForCustomerSource",
                "query": "<custom query e.g. $top=10>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sap-c4c-as-sink"></a>將 SAP C4C 作為接收器

若要將資料複製到 SAP Cloud for Customer，請將複製活動中的接收類型設定為 **SapCloudForCustomerSink**。 複製活動的 **sink** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為：**SapCloudForCustomerSink**  | yes |
| writeBehavior | 作業的寫入行為。 可能是 “Insert”、“Update”。 | 否。 預設值為 “Insert”。 |
| writeBatchSize | 寫入作業的批次大小。 要取得最佳效能的批次大小可能會隨著資料表或伺服器而有所不同。 | 否。 預設值為 10。 |

**範例：**

```json
"activities":[
    {
        "name": "CopyToSapC4c",
        "type": "Copy",
        "inputs": [{
            "type": "DatasetReference",
            "referenceName": "<dataset type>"
        }],
        "outputs": [{
            "type": "DatasetReference",
            "referenceName": "SapC4cDataset"
        }],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SapCloudForCustomerSink",
                "writeBehavior": "Insert",
                "writeBatchSize": 30
            },
            "parallelCopies": 10,
            "dataIntegrationUnits": 4,
            "enableSkipIncompatibleRow": true,
            "redirectIncompatibleRowSettings": {
                "linkedServiceName": {
                    "referenceName": "ErrorLogBlobLinkedService",
                    "type": "LinkedServiceReference"
                },
                "path": "incompatiblerows"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-cloud-for-customer"></a>SAP Cloud for Customer 的資料類型對應

從 SAP Cloud for Customer 複製資料時，會使用下列從 SAP Cloud for Customer 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| SAP C4C OData 資料類型 | Data Factory 過渡期資料類型 |
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


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
