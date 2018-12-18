---
title: 使用 Azure Data Factory 從 SAP ECC 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 SAP ECC 複製到支援的接收資料存放區。
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
ms.date: 04/26/2018
ms.author: jingwang
ms.openlocfilehash: f9f6d2e43fff9a3e57145f39863f66eed64869b2
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048578"
---
# <a name="copy-data-from-sap-ecc-using-azure-data-factory"></a>使用 Azure Data Factory 從 SAP ECC 複製資料

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 SAP ECC (SAP Enterprise Central Component) 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 SAP ECC 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此 SAP ECC 連接器支援：

- 在 SAP NetWeaver 7.0 版和更新版本上從 SAP ECC 複製資料。 
- 從 SAP ECC OData 服務 (例如 SAP 資料表/檢視表、BAPI、資料擷取器等) 所公開的任何物件複製資料，或複製傳送至 SAP PI 而可透過相對配接器以 OData 的形式接收的資料/IDOC。
- 使用基本驗證來複製資料。

## <a name="prerequisites"></a>先決條件

一般而言，SAP ECC 會透過 OData 服務經由 SAP 閘道公開實體。 若要使用此 SAP ECC 連接器，您必須：

- **設定 SAP 閘道**。 如果伺服器使用高於 7.4 版的 SAP NetWeaver，則必須已安裝 SAP 閘道器。 否則，您必須先安裝內嵌閘道或閘道中樞，再透過 OData 服務公開 SAP ECC 資料。 從[安裝指南](https://help.sap.com/saphelp_gateway20sp12/helpdata/en/c3/424a2657aa4cf58df949578a56ba80/frameset.htm)了解如何設定 SAP 閘道。

- **啟動及設定 SAP OData 服務**。 您可以透過 TCODE SICF，在短短數秒內啟動 OData 服務。 您也可以設定哪些物件需要公開。 以下是[逐步指引](https://blogs.sap.com/2012/10/26/step-by-step-guide-to-build-an-odata-service-based-on-rfcs-part-1/)範例。

## <a name="getting-started"></a>開始使用

您可以使用 .NET SDK、Python SDK、Azure PowerShell、REST API 或 Azure Resource Manager 範本來建立具有複製活動的管線。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](quickstart-create-data-factory-dot-net.md)。

下列各節將針對用來定義 SAP ECC 連接器專屬之 Data Factory 實體的屬性提供相關詳細資料。

## <a name="linked-service-properties"></a>連結服務屬性

以下是 SAP ECC 連結服務的支援屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為：**SapEcc** | yes |
| url | SAP ECC OData 服務的 URL。 | yes |
| username | 用來連線至 SAP ECC 的使用者名稱。 | 否 |
| password | 用來連線至 SAP ECC 的純文字密碼。 | 否 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：**

```json
{
    "name": "SapECCLinkedService",
    "properties": {
        "type": "SapEcc",
        "typeProperties": {
            "url": "<SAP ECC OData url e.g. http://eccsvrname:8000/sap/opu/odata/sap/zgw100_dd02l_so_srv/>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 SAP ECC 資料集所支援的屬性清單。

若要從 SAP ECC 複製資料，請將資料集的類型屬性設為 **SapEccResource**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| path | SAP ECC OData 實體的路徑。 | yes |

**範例**

```json
{
    "name": "SapEccDataset",
    "properties": {
        "type": "SapEccResource",
        "typePoperties": {
            "path": "<entity path e.g. dd04tentitySet>"
        },
        "linkedServiceName": {
            "referenceName": "<SAP ECC linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 SAP ECC 來源所支援的屬性清單。

### <a name="sap-ecc-as-source"></a>以 SAP ECC 作為來源

若要從 SAP ECC 複製資料，請將複製活動中的來源類型設為 **SapEccSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設為：**SapEccSource** | yes |
| query | 用來篩選資料的 OData 查詢選項。 Example: "$select=Name,Description&$top=10".<br/><br/>SAP ECC 連接器會從合併的 URL 複製資料：(連結服務中指定的 URL)/(資料集中指定的路徑)?(複製活動來源中指定的查詢)。 請參考 [OData URL 元件](http://www.odata.org/documentation/odata-version-3-0/url-conventions/) \(英文\)。 | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromSAPECC",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP ECC input dataset name>",
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
                "type": "SapEccSource",
                "query": "$top=10"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-ecc"></a>SAP ECC 的資料類型對應

從 SAP ECC 複製資料時，會使用下列從 SAP ECC 資料的 OData 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| OData 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |:--- |
| Edm.Binary | 字串 |
| Edm.Boolean | Bool |
| Edm.Byte | 字串 |
| Edm.DateTime | Datetime |
| Edm.Decimal | 十進位 |
| Edm.Double | 兩倍 |
| Edm.Single | 單一 |
| Edm.Guid | 字串 |
| Edm.Int16 | Int16 |
| Edm.Int32 | Int32 |
| Edm.Int64 | Int64 |
| Edm.SByte | Int16 |
| Edm.String | 字串 |
| Edm.Time | 時間範圍 |
| Edm.DateTimeOffset | DateTimeOffset |

> [!NOTE]
> 目前不支援複雜資料類型。

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
