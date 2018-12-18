---
title: 使用 Azure Data Factory 從 PayPal 複製資料 (預覽) | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 PayPal 複製到支援的接收資料存放區。
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
ms.openlocfilehash: 6d752eb5d638171aa510bbbf17a197eddd2b6f60
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127194"
---
# <a name="copy-data-from-paypal-using-azure-data-factory-preview"></a>使用 Azure Data Factory 從 PayPal 複製資料 (預覽)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 PayPal 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!IMPORTANT]
> 此連接器目前為預覽版。 您可以親身體驗並提供意見反應。 如果您需要依賴解決方案中的預覽連接器，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 PayPal 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 PayPal 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 PayPal 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**PayPal** | 是 |
| host | PayPal 執行個體的 URL。 (也就是 api.sandbox.paypal.com)  | 是 |
| clientId | 與您 PayPal 應用程式相關聯的用戶端識別碼。  | 是 |
| clientSecret | 與您 PayPal 應用程式相關聯的用戶端祕密。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |
| useHostVerification | 指定在透過 SSL 連線時，是否要求伺服器憑證中的主機名稱符合伺服器的主機名稱。 預設值為 true。  | 否 |
| usePeerVerification | 指定在透過 SSL 連線時，是否要確認伺服器的身分識別。 預設值為 true。  | 否 |

**範例：**

```json
{
    "name": "PayPalLinkedService",
    "properties": {
        "type": "PayPal",
        "typeProperties": {
            "host" : "api.sandbox.paypal.com",
            "clientId" : "<clientId>",
            "clientSecret": {
                 "type": "SecureString",
                 "value": "<clientSecret>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 PayPal 資料集所支援的屬性清單。

若要從 PayPal 複製資料，請將資料集的 type 屬性設定為 **PayPalObject**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "PayPalDataset",
    "properties": {
        "type": "PayPalObject",
        "linkedServiceName": {
            "referenceName": "<PayPal linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 PayPal 來源所支援的屬性清單。

### <a name="paypalsource-as-source"></a>將 PayPalSource 作為來源

若要從 PayPal 複製資料，請將複製活動中的來源類型設定為 **PayPalSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**PayPalSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM Payment_Experience"` 。 | 是 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromPayPal",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<PayPal input dataset name>",
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
                "type": "PayPalSource",
                "query": "SELECT * FROM Payment_Experience"
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
