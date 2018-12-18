---
title: 使用 Azure Data Factory 從 Netezza 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從 Netezza 將資料複製到支援的接收資料存放區。
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
ms.date: 06/15/2018
ms.author: jingwang
ms.openlocfilehash: f8c10e2200f830ea6e568e7b3fba1f0a6085cef2
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37055450"
---
# <a name="copy-data-from-netezza-using-azure-data-factory"></a>使用 Azure Data Factory 從 Netezza 複製資料 

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Netezza 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Netezza 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

您可以使用 .NET SDK、Python SDK、Azure PowerShell、REST API 或 Azure Resource Manager 範本來建立具有複製活動的管線。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](quickstart-create-data-factory-dot-net.md)。

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Netezza 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Netezza 連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為：**Netezza** | yes |
| connectionString | 連線到 Netezza 的 ODBC 連接字串。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

一般的連接字串為 `Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>`。 您可以根據您的案例設定更多屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |:--- |
| SecurityLevel | 驅動程式用來連線到資料存放區的安全性 (SSL/TLS) 層級。 例如 `SecurityLevel=preferredSecured`。 支援的值包括：<br/>- 僅限未受保護連線 (**onlyUnSecured**)：驅動程式不會使用 SSL。<br/>- **偏好未受保護連線 (preferredUnSecured) (預設值)**：如果伺服器提供選擇，則驅動程式不使用 SSL。 <br/>- **偏好受保護連線 (preferredSecured)**：如果伺服器提供選擇，則驅動程式會使用 SSL。 <br/>- **僅限受保護連線 (onlySecured)**：除非有 SSL 連線，否則驅動程式不會連線 | 否 |
| CaCertFile | 伺服器使用的 SSL 憑證完整路徑。 例如 `CaCertFile=<cert path>;`| 是，如果已啟用 SSL |

**範例：**

```json
{
    "name": "NetezzaLinkedService",
    "properties": {
        "type": "Netezza",
        "typeProperties": {
            "connectionString": {
                 "type": "SecureString",
                 "value": "Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Netezza 資料集所支援的屬性清單。

若要從 Netezza 複製資料，請將資料集的類型屬性設定為 **NetezzaTable**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "NetezzaDataset",
    "properties": {
        "type": "NetezzaTable",
        "linkedServiceName": {
            "referenceName": "<Netezza linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Netezza 來源所支援的屬性清單。

### <a name="netezza-as-source"></a>Netezza 作為來源

若要從 Netezza 複製資料，請將複製活動中的來源類型設定為 **NetezzaSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**NetezzaSource** | yes |
| query | 使用自訂 SQL 查詢來讀取資料。 例如：`"SELECT * FROM MyTable"`。 | yes |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromNetezza",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Netezza input dataset name>",
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
                "type": "NetezzaSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
