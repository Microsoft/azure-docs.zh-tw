---
title: 使用 Azure Data Factory 從 ServiceNow 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 ServiceNow 複製到支援的接收資料存放區。
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
ms.date: 05/28/2018
ms.author: jingwang
ms.openlocfilehash: c67f6c14dc396367e0179fe5bdb4663fcb7725da
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045961"
---
# <a name="copy-data-from-servicenow-using-azure-data-factory"></a>使用 Azure Data Factory 從 ServiceNow 複製資料

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 ServiceNow 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 ServiceNow 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 ServiceNow 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 ServiceNow 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**ServiceNow** | yes |
| endpoint | ServiceNow 伺服器的端點 (`http://<instance>.service-now.com`)。  | yes |
| authenticationType | 要使用的驗證類型。 <br/>允許的值為：**Basic**、**OAuth2** | yes |
| username | 用來連線到 ServiceNow 伺服器以進行 Basic 和 OAuth2 驗證的使用者名稱。  | yes |
| password | 對應至用於進行 Basic 和 OAuth2 驗證之使用者名稱的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | yes |
| clientId | 用於 OAuth2 驗證的用戶端識別碼。  | 否 |
| clientSecret | 用於 OAuth2 驗證的用戶端祕密。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 否 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |
| useHostVerification | 指定在透過 SSL 連線時，是否要求伺服器憑證中的主機名稱符合伺服器的主機名稱。 預設值為 true。  | 否 |
| usePeerVerification | 指定在透過 SSL 連線時，是否要確認伺服器的身分識別。 預設值為 true。  | 否 |

**範例：**

```json
{
    "name": "ServiceNowLinkedService",
    "properties": {
        "type": "ServiceNow",
        "typeProperties": {
            "endpoint" : "http://<instance>.service-now.com",
            "authenticationType" : "Basic",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 ServiceNow 資料集所支援的屬性清單。

若要從 ServiceNow 複製資料，請將資料集的 type 屬性設定為 **ServiceNowObject**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "ServiceNowDataset",
    "properties": {
        "type": "ServiceNowObject",
        "linkedServiceName": {
            "referenceName": "<ServiceNow linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 ServiceNow 來源所支援的屬性清單。

### <a name="servicenow-as-source"></a>ServiceNow 作為來源

若要從 ServiceNow 複製資料，請將複製活動中的來源類型設定為 **ServiceNowSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**ServiceNowSource** | yes |
| query | 使用自訂 SQL 查詢來讀取資料。 例如：`"SELECT * FROM Actual.alm_asset"`。 | yes |

在查詢中指定 ServiceNow 的結構描述和資料行時，請注意下列事項：

- **結構描述：** 在 ServiceNow 查詢中，將結構描述指定為 `Actual` 或 `Display`，而在呼叫 [ServiceNow RESTful API](https://developer.servicenow.com/app.do#!/rest_api_doc?v=jakarta&id=r_AggregateAPI-GET) 時，您可以將它視為 `sysparm_display_value` 的參數 (true 或 false)。 
- **資料行：**`Actual` 結構描述下實際值的資料行名稱為 `[columne name]_value`，而 `Display` 結構描述下顯示值的資料行名稱為 `[columne name]_display_value`。 請注意，資料行名稱必須對應至要在查詢中使用的結構描述。

**範例查詢：**
`SELECT col_value FROM Actual.alm_asset` 或 `SELECT col_display_value FROM Display.alm_asset`

**範例：**

```json
"activities":[
    {
        "name": "CopyFromServiceNow",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ServiceNow input dataset name>",
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
                "type": "ServiceNowSource",
                "query": "SELECT * FROM Actual.alm_asset"
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
