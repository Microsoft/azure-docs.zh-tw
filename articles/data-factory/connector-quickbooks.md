---
title: 使用 Azure Data Factory (預覽) 從 QuickBooks 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，將資料從 QuickBooks 複製到支援的接收資料存放區。
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
ms.openlocfilehash: 83e3007a7c3198c5ae37cf95d2b21cde88bd8210
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127125"
---
# <a name="copy-data-from-quickbooks-using-azure-data-factory-preview"></a>使用 Azure Data Factory 從 QuickBooks 複製資料 (預覽)

此文章概述如何使用 Azure Data Factory 中的「複製活動」，從 QuickBooks 複製資料。 此文章是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!IMPORTANT]
> 此連接器目前為預覽版。 您可以親身體驗並提供意見反應。 如果您需要依賴解決方案中的預覽連接器，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 QuickBooks 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

此連接器目前僅支援 1.0a，這表示您需要具備一個開發人員帳戶，其中含有在 2017 年 7 月 17 日之前建立的應用程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 QuickBooks 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 QuickBooks 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**QuickBooks** | 是 |
| endpoint | QuickBooks 伺服器的端點。 (也就是 quickbooks.api.intuit.com)  | 是 |
| companyId | 要授權之 QuickBooks 公司的公司識別碼。  | 是 |
| consumerKey | 用於 OAuth 1.0 驗證的取用者金鑰。 | 是 |
| consumerSecret | 用於 OAuth 1.0 驗證的取用者祕密。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| accessToken | 用於 OAuth 1.0 驗證的存取權杖。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| accessTokenSecret | 用於 OAuth 1.0 驗證的存取權杖祕密。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |

**範例：**

```json
{
    "name": "QuickBooksLinkedService",
    "properties": {
        "type": "QuickBooks",
        "typeProperties": {
            "endpoint" : "quickbooks.api.intuit.com",
            "companyId" : "<companyId>",
            "consumerKey": "<consumerKey>",
            "consumerSecret": {
                "type": "SecureString",
                "value": "<consumerSecret>"
            },
            "accessToken": {
                 "type": "SecureString",
                 "value": "<accessToken>"
            },
            "accessTokenSecret": {
                 "type": "SecureString",
                 "value": "<accessTokenSecret>"
            },
            "useEncryptedEndpoints" : true
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 此節提供 QuickBooks 資料集所支援的屬性清單。

若要從 QuickBooks 複製資料，請將資料集的 type 屬性設定為 **QuickBooksObject**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "QuickBooksDataset",
    "properties": {
        "type": "QuickBooksObject",
        "linkedServiceName": {
            "referenceName": "<QuickBooks linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 此節提供 QuickBooks 來源所支援的屬性清單。

### <a name="quickbookssource-as-source"></a>將 QuickBooksSource 作為來源

若要從 QuickBooks 複製資料，請將複製活動中的來源類型設定為 **QuickBooksSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**QuickBooksSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM "Bill" WHERE Id = '123'"` 。 | 是 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromQuickBooks",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<QuickBooks input dataset name>",
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
                "type": "QuickBooksSource",
                "query": "SELECT * FROM \"Bill\" WHERE Id = '123' "
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
