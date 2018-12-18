---
title: 使用 Azure Data Factory 從 Dynamics CRM 或 Dynamics 365 (Common Data Service) 複製資料以及複製資料至 Dynamics CRM 或 Dynamics 365 | Microsoft Docs
description: 了解如何使用資料處理站管線中的複製活動，將資料從 Microsoft Dynamics CRM 或 Microsoft Dynamics 365 (Common Data Service) 複製到支援的接收資料存放區，或是從支援的來源資料存放區複製到 Dynamics CRM 或 Dynamics 365。
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
ms.date: 09/26/2018
ms.author: jingwang
ms.openlocfilehash: ce3c494dc0b8c962c8dae0af38d3cb5476cdf48b
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406170"
---
# <a name="copy-data-from-and-to-dynamics-365-common-data-service-or-dynamics-crm-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 Dynamics 365 (Common Data Service) 複製資料以及複製資料至 Dynamics 365

此文章概述如何使用 Azure Data Factory 中的「複製活動」，從 Microsoft Dynamics 365 或 Microsoft Dynamics CRM 複製資料以及複製資料至 Microsoft Dynamics 365 或 Microsoft Dynamics CRM。 此文章是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Dynamics 365 (Common Data Service) 或 Dynamics CRM 複製到任何支援的接收資料存放區。 您也可以從任何支援的來源資料存放區將資料複製到 Dynamics 365 (Common Data Service) 或 Dynamics CRM。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

這個 Dynamics 連接器支援下列 Dynamics 版本和驗證類型。 (IFD 是網際網路對向部署的縮寫。)

| Dynamics 版本 | 驗證類型 | 已連結的服務範例 |
|:--- |:--- |:--- |
| Dynamics 365 線上版 <br> Dynamics CRM Online | Office365 | [Dynamics Online + Office365 驗證](#dynamics-365-and-dynamics-crm-online) |
| 搭配 IFD 的 Dynamics 365 內部部署版 <br> 搭配 IFD 的 Dynamics CRM 2016 內部部署版 <br> 搭配 IFD 的 Dynamics CRM 2015 內部部署版 | IFD | [搭配 IFD 的 Dynamics 內部部署版 + IFD 驗證](#dynamics-365-and-dynamics-crm-on-premises-with-ifd) |

特別針對 Dynamics 365 而言，支援下列應用程式類型：

- Dynamics 365 for Sales
- Dynamics 365 for Customer Service
- Dynamics 365 for Field Service
- Dynamics 365 for Project Service Automation
- Dynamics 365 for Marketing

不支援其他應用程式類型，例如作業和金融、Talent 等。

## <a name="get-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Dynamics 專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Dynamics 連結服務所支援的屬性。

### <a name="dynamics-365-and-dynamics-crm-online"></a>Dynamics 365 和 Dynamics CRM Online

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為 **Dynamics**。 | 是 |
| deploymentType | Dynamics 執行個體的部署類型。 如果是 Dynamics Online，就必須是 **"Online"**。 | 是 |
| serviceUri | 您 Dynamics 執行個體的服務 URL，例如 `https://adfdynamics.crm.dynamics.com`。 | 是 |
| authenticationType | 連線到 Dynamics 伺服器時所使用的驗證類型。 如果是 Dynamics Online，請指定 **"Office365"**。 | 是 |
| username | 指定要連線到 Dynamics 的使用者名稱。 | 是 |
| password | 指定您為 username 指定之使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 如果未指定，就會使用預設的 Azure Integration Runtime。 | 如果來源連結服務沒有整合執行階段，則對於來源而言為「否」；對於接收而言為「是」 |

>[!IMPORTANT]
>將資料複製到 Dynamics 時，預設的 Azure Integration Runtime 無法用來執行複製。 換句話說，如果來源連結服務沒有指定的整合執行階段，請使用 Dynamics 執行個體附近的位置明確[建立 Azure Integration Runtime](create-azure-integration-runtime.md#create-azure-ir)。 在 Dynamics 連結服務中建立關聯，如下列範例所示。

>[!NOTE]
>您可利用 Dynamics 連接器來使用選擇性 "organizationName" 屬性，以識別您的 Dynamics CRM/365 Online 執行個體。 在其持續運作的同時，建議您改為指定新的 "serviceUri" 屬性，以取得更好的效能來探索執行個體。

**範例：使用 Office365 驗證的 Dynamics 線上版**

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics online linked service using Office365 authentication",
        "typeProperties": {
            "deploymentType": "Online",
            "serviceUri": "https://adfdynamics.crm.dynamics.com",
            "authenticationType": "Office365",
            "username": "test@contoso.onmicrosoft.com",
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

### <a name="dynamics-365-and-dynamics-crm-on-premises-with-ifd"></a>搭配 IFD 的 Dynamics 365 和 Dynamics CRM 內部部署版

相較於 Dyanmics Online，額外的屬性為 "hostName" 和 "port"。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為 **Dynamics**。 | 是 |
| deploymentType | Dynamics 執行個體的部署類型。 如果是搭配 IFD 的 Dynamics 內部部署版，就必須是 **"OnPremisesWithIfd"**。| 是 |
| hostName | 內部部署 Dynamics 伺服器的主機名稱。 | 是 |
| 連接埠 | 內部部署 Dynamics 伺服器的連接埠。 | 否，預設值為 443 |
| organizationName | Dynamics 執行個體的組織名稱。 | 是 |
| authenticationType | 連線到 Dynamics 伺服器時所使用的驗證類型。 如果是搭配 IFD 的 Dynamics 內部部署版，請指定 **"Ifd"**。 | 是 |
| username | 指定要連線到 Dynamics 的使用者名稱。 | 是 |
| password | 指定您為 username 指定之使用者帳戶的密碼。 您可以選擇將這個欄位標記為 SecureString 以將它安全地儲存在 ADF，或將密碼儲存在 Azure Key Vault；然後在執行複製資料時，讓複製活動從該處提取 - 請參閱[將認證儲存在 Key Vault](store-credentials-in-key-vault.md) 以進一步了解。 | 是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 如果未指定，就會使用預設的 Azure Integration Runtime。 | 否 (來源)；是 (接收) |

>[!IMPORTANT]
>若要將資料複製到 Dynamics，請使用 Dynamics 執行個體附近的位置明確[建立 Azure Integration Runtime](create-azure-integration-runtime.md#create-azure-ir)。 在連結服務中建立關聯，如下列範例所示。

**範例：使用 IFD 驗證之搭配 IFD 的 Dynamics 內部部署版**

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics on-premises with IFD linked service using IFD authentication",
        "typeProperties": {
            "deploymentType": "OnPremisesWithIFD",
            "hostName": "contosodynamicsserver.contoso.com",
            "port": 443,
            "organizationName": "admsDynamicsTest",
            "authenticationType": "Ifd",
            "username": "test@contoso.onmicrosoft.com",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Dynamics 資料集所支援的屬性清單。

若要從 Dynamics 複製資料以及複製資料到 Dynamics，請將資料集的類型屬性設定為 **DynamicsEntity**。 以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的 type 屬性必須設定為 **DynamicsEntity**。 |是 |
| entityName | 要擷取之實體的邏輯名稱。 | 否 (來源，如果已指定活動來源中的「查詢」)；是 (接收) |

> [!IMPORTANT]
>- 從 Dynamics 複製資料時，"structure" 區段是選擇性區段，但建議在 Dynamics 資料集中需有此區段，以確保能有確定性的複製結果。 它會定義您想要複製的 Dynamics 資料之資料行名稱和資料類型。 若要深入了解，請參閱[資料集結構](concepts-datasets-linked-services.md#dataset-structure)和 [Dynamics 的資料類型對應](#data-type-mapping-for-dynamics)。
>- 在撰寫 UI 中匯入結構描述時，ADF 會推斷結構描述，方法是從 Dynamics 查詢結果取前幾個資料列作為樣本來進行結構建構初始化，在此情況下，會省略沒有值的資料行。 您可以視需要檢閱及將更多資料行新增至 Dynamics 資料集結構描述/結構，在複製執行階段即會予以採用。
>- 將資料複製到 Dynamics 時，Dynamics 資料集內不一定要有 "structure" 區段。 要複製到哪些資料行則由來源資料的結構描述決定。 如果來源是沒有標題的 CSV 檔案，在輸入資料集中使用資料行名稱和資料類型指定 "structure"。 它們會按順序一一對應至 CSV 檔案中的欄位。

**範例：**

```json
{
    "name": "DynamicsDataset",
    "properties": {
        "type": "DynamicsEntity",
        "structure": [
            {
                "name": "accountid",
                "type": "Guid"
            },
            {
                "name": "name",
                "type": "String"
            },
            {
                "name": "marketingonly",
                "type": "Boolean"
            },
            {
                "name": "modifiedon",
                "type": "Datetime"
            }
        ],
        "typeProperties": {
            "entityName": "account"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics linked service name>",
            "type": "linkedservicereference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Dynamics 來源和接收類型所支援的屬性清單。

### <a name="dynamics-as-a-source-type"></a>Dynamics 作為來源類型

若要從 Dynamics 複製資料，請將複製活動中的來源類型設定為 **DynamicsSource**。 複製活動的 [來源] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為 **DynamicsSource**。 | 是 |
| query | FetchXML 是在 Dynamics (線上版和內部部署版) 中使用的專屬查詢語言。 請參閱下列範例。 若要深入了解，請參閱[使用 FetchXML 建置查詢](https://msdn.microsoft.com/library/gg328332.aspx)。 | 否 (如果已指定資料集中的 "entityName") |

>[!NOTE]
>即使您在 FetchXML 查詢中設定的資料行投影未包含 PK 資料行，還是一律會複製 PK 資料行。

**範例：**

```json
"activities":[
    {
        "name": "CopyFromDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics input dataset>",
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
                "type": "DynamicsSource",
                "query": "<FetchXML Query>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sample-fetchxml-query"></a>範例 FetchXML 查詢

```xml
<fetch>
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="marketingonly" />
    <attribute name="modifiedon" />
    <order attribute="modifiedon" descending="false" />
    <filter type="and">
      <condition attribute ="modifiedon" operator="between">
        <value>2017-03-10 18:40:00z</value>
        <value>2017-03-12 20:40:00z</value>
      </condition>
    </filter>
  </entity>
</fetch>
```

### <a name="dynamics-as-a-sink-type"></a>Dynamics 作為接收類型

若要複製資料至 Dynamics ，將複製活動中的接收類型設定為 **DynamicsSink**。 複製活動的 [接收] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收的 type 屬性必須設定為 **DynamicsSink**。 | 是 |
| writeBehavior | 作業的寫入行為。<br/>允許的值為 **"Upsert"**。 | 是 |
| writeBatchSize | 每個批次中寫入 Dynamics 的資料列計數。 | 否 (預設值為 10) |
| ignoreNullValues | 指出在寫入作業期間是否要忽略輸入資料中的 Null 值 (索引鍵欄位除外)。<br/>允許的值為 **true** 和 **false**。<br>- **True**：執行 upsert/更新作業時，將目的地物件中的資料保持不變。 執行插入作業時，插入已定義的預設值。<br/>- **False**：執行 upsert/更新作業時，將目的地物件中的資料更新為 NULL。 執行插入作業時，插入 NULL 值。 | 否 (預設值為 false) |

>[!NOTE]
>接收 **writeBatchSize** 以及 Dynamics 接收的複製活動 **[parallelCopies](copy-activity-performance.md#parallel-copy)** 的預設值皆為 10。 因此，100 個記錄會同時提交到 Dynamics。

Dynamics 365 線上版限制[每個組織只能有 2 個並行批次呼叫](https://msdn.microsoft.com/library/jj863631.aspx#Run-time%20limitations)。 如果超出該限制，則會在執行第一個要求之前，擲回「伺服器忙碌」錯誤。 讓「writeBatchSize」保持小於或等於 10，即可避免發生這類並行呼叫節流。

**writeBatchSize** 和 **parallelCopies** 的最佳組合取決於實體的結構描述，例如資料行數目、資料列大小、連接到這些呼叫的外掛程式/工作流程/工作流程活動數目等等。10 個 writeBatchSize * 10 個 parallelCopies 的預設設定是根據 Dynamics 服務所提出的建議，雖適用於大部分 Dynamics 實體，但可能無法達到最佳效能。 您可以透過調整複製活動設定中的組合來微調效能。

**範例：**

```json
"activities":[
    {
        "name": "CopyToDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Dynamics output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DynamicsSink",
                "writeBehavior": "Upsert",
                "writeBatchSize": 10,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="data-type-mapping-for-dynamics"></a>Dynamics 的資料類型對應

從 Dynamics 複製資料時，會使用下列從 Dynamics 資料類型到 Data Factory 過渡期資料類型的對應。 若要了解複製活動如何將來源結構描述和資料類型對應至接收，請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)。

請參考以下的對應表，根據您的來源 Dynamics 資料類型，在資料集結構中設定對應的 Data Factory 資料類型。

| Dynamics 資料類型 | Data Factory 過渡期資料類型 | 支援作為來源 | 支援作為接收 |
|:--- |:--- |:--- |:--- |
| AttributeTypeCode.BigInt | long | ✓ | ✓ |
| AttributeTypeCode.Boolean | BOOLEAN | ✓ | ✓ |
| AttributeType.Customer | Guid | ✓ | | 
| AttributeType.DateTime | DateTime | ✓ | ✓ |
| AttributeType.Decimal | 十進位 | ✓ | ✓ |
| AttributeType.Double | 兩倍 | ✓ | ✓ |
| AttributeType.EntityName | 字串 | ✓ | ✓ |
| AttributeType.Integer | Int32 | ✓ | ✓ |
| AttributeType.Lookup | Guid | ✓ | ✓ (具有相關聯的單一目標) |
| AttributeType.ManagedProperty | BOOLEAN | ✓ | |
| AttributeType.Memo | 字串 | ✓ | ✓ |
| AttributeType.Money | 十進位 | ✓ | ✓ |
| AttributeType.Owner | Guid | ✓ | |
| AttributeType.Picklist | Int32 | ✓ | ✓ |
| AttributeType.Uniqueidentifier | Guid | ✓ | ✓ |
| AttributeType.String | 字串 | ✓ | ✓ |
| AttributeType.State | Int32 | ✓ | ✓ |
| AttributeType.Status | Int32 | ✓ | ✓ |


> [!NOTE]
> 不支援 Dynamics 資料類型 AttributeType.CalendarRules 和 AttributeType.PartyList。

## <a name="next-steps"></a>後續步驟
如需 Data Factory 中的複製活動所支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
