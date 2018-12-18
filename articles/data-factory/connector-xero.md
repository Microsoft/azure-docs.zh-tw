---
title: 使用 Azure Data Factory 複製 Xero 中的資料 (預覽) | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從 Xero 將資料複製到支援的接收資料存放區。
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
ms.openlocfilehash: d9ee9a73f4e88786ca51fe9fac50ce51e25b4dde
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46123363"
---
# <a name="copy-data-from-xero-using-azure-data-factory-preview"></a>使用 Azure Data Factory 從 Xero 複製資料 (預覽)

此文章概述如何使用 Azure Data Factory 中的「複製活動」，從 Xero 複製資料。 此文章是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

> [!IMPORTANT]
> 此連接器目前為預覽版。 您可以親身體驗並提供意見反應。 如果您需要依賴解決方案中的預覽連接器，請連絡 [Azure 支援](https://azure.microsoft.com/support/)。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Xero 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Xero 連接器支援：

- Xero [私人應用程式](https://developer.xero.com/documentation/getting-started/api-application-types)，但不支援公用應用程式。
- 所有 Xero 資料表 (API 端點)，但「報告」除外。 

Azure Data Factory 提供的內建驅動程式可啟用連線，因此使用此連接器您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Xero 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Xero 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | Type 屬性必須設定為：**Xero** | 是 |
| host | Xero 伺服器的端點 (`api.xero.com`)。  | 是 |
| consumerKey | 與 Xero 應用程式相關聯的取用者金鑰。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| privateKey | 從您 Xero 私人應用程式產生之 .pem 檔案的私密金鑰，請參閱[建立公開/私密金鑰組](https://developer.xero.com/documentation/api-guides/create-publicprivate-key)。 使用 `openssl genrsa -out privatekey.pem 512` **產生 numbit 為 512 的 privatekey.pem** 備註；不支援 1024。 包含 .pem 檔案的所有文字，包括 Unix 行尾結束符號 (\n)，請參閱以下範例。<br/><br/>將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| useEncryptedEndpoints | 指定是否使用 HTTPS 來加密資料來源端點。 預設值為 true。  | 否 |
| useHostVerification | 指定在透過 SSL 連線時，是否要求伺服器憑證中的主機名稱符合伺服器的主機名稱。 預設值為 true。  | 否 |
| usePeerVerification | 指定在透過 SSL 連線時，是否要確認伺服器的身分識別。 預設值為 true。  | 否 |

**範例：**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "host" : "api.xero.com",
            "consumerKey": {
                 "type": "SecureString",
                 "value": "<consumerKey>"
            },
            "privateKey": {
                 "type": "SecureString",
                 "value": "<privateKey>"
            }
        }
    }
}
```

**範例私密金鑰值：**

包含 .pem 檔案的所有文字，包括 Unix 行尾結束符號 (\n)。

```
"-----BEGIN RSA PRIVATE KEY-----\nMII***************************************************P\nbu****************************************************s\nU/****************************************************B\nA*****************************************************W\njH****************************************************e\nsx*****************************************************l\nq******************************************************X\nh*****************************************************i\nd*****************************************************s\nA*****************************************************dsfb\nN*****************************************************M\np*****************************************************Ly\nK*****************************************************Y=\n-----END RSA PRIVATE KEY-----"
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 此節提供 Xero 資料集所支援的屬性清單。

若要從 Xero 複製資料，請將資料集的 type 屬性設定為 **XeroObject**。 在此類型的資料集中，沒有任何其他類型特定的屬性。

**範例**

```json
{
    "name": "XeroDataset",
    "properties": {
        "type": "XeroObject",
        "linkedServiceName": {
            "referenceName": "<Xero linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 此節提供 Xero 來源所支援的屬性清單。

### <a name="xero-as-source"></a>Xero 作為來源

若要從 Xero 複製資料，請將複製活動中的來源類型設定為 **XeroSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為：**XeroSource** | 是 |
| query | 使用自訂 SQL 查詢來讀取資料。 例如： `"SELECT * FROM Contacts"` 。 | 是 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromXero",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Xero input dataset name>",
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
                "type": "XeroSource",
                "query": "SELECT * FROM Contacts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

指定 Xero 查詢時，請注意下列事項：

- 具有複雜項目的資料表將會分割成多個資料表。 例如，銀行交易有複雜的資料結構 "LineItems"，所以銀行交易的資料會對應至資料表 `Bank_Transaction` 和 `Bank_Transaction_Line_Items`，並以 `Bank_Transaction_ID` 作為外部索引鍵而連結在一起。

- Xero 資料可透過兩個結構描述取得：`Minimal` (預設值) 和 `Complete`。 Complet 結構描述包含必要呼叫資料表，其在進行所要的查詢之前需有額外的資料 (例如識別碼資料行)。

下表中的 Minimal 和 Complete 結構描述有相同的資訊。 若要減少 API 呼叫數目，請使用 Minimal 結構描述 (預設值)。

- Bank_Transactions
- Contact_Groups 
- 連絡人 
- Contacts_Sales_Tracking_Categories 
- Contacts_Phones 
- Contacts_Addresses 
- Contacts_Purchases_Tracking_Categories 
- Credit_Notes 
- Credit_Notes_Allocations 
- Expense_Claims 
- Expense_Claim_Validation_Errors
- 發票 
- Invoices_Credit_Notes
- Invoices_Prepayments 
- Invoices_Overpayments 
- Manual_Journals 
- Overpayments 
- Overpayments_Allocations 
- Prepayments 
- Prepayments_Allocations 
- Receipts 
- Receipt_Validation_Errors 
- Tracking_Categories

只有使用 Complete 結構描述時才會查詢下表：

- Complete.Bank_Transaction_Line_Items 
- Complete.Bank_Transaction_Line_Item_Tracking 
- Complete.Contact_Group_Contacts 
- Complete.Contacts_Contact_Persons 
- Complete.Credit_Note_Line_Items 
- Complete.Credit_Notes_Line_Items_Tracking 
- Complete.Expense_Claim_Payments 
- Complete.Expense_Claim_Receipts 
- Complete.Invoice_Line_Items 
- Complete.Invoices_Line_Items_Tracking
- Complete.Manual_Journal_Lines 
- Complete.Manual_Journal_Line_Tracking 
- Complete.Overpayment_Line_Items 
- Complete.Overpayment_Line_Items_Tracking 
- Complete.Prepayment_Line_Items 
- Complete.Prepayment_Line_Item_Tracking 
- Complete.Receipt_Line_Items 
- Complete.Receipt_Line_Item_Tracking 
- Complete.Tracking_Category_Options

## <a name="next-steps"></a>後續步驟
如需複製活動所支援的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
