---
title: 使用 Data Factory 將資料複製到 Azure 資料表儲存體或從該處複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory 將資料從支援的來源存放區複製到「Azure 資料表儲存體」，或從「資料表儲存體」複製到支援的接收存放區。
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
ms.date: 08/17/2018
ms.author: jingwang
ms.openlocfilehash: 0399836191050996ac3eaf0fbe59496e10e2b426
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42141918"
---
# <a name="copy-data-to-and-from-azure-table-storage-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料複製到 Azure 資料表儲存體或從該處複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-azure-table-connector.md)
> * [目前的版本](connector-azure-table-storage.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，將資料複製到「Azure 資料表」儲存體及從該處複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從支援的來源資料存放區複製到資料表儲存體。 您也可以將資料從資料表儲存體複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個「Azure 資料表」連接器支援使用帳戶金鑰和服務共用存取簽章驗證來複製資料。

## <a name="get-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義資料表儲存體專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

### <a name="use-an-account-key"></a>使用帳戶金鑰

您可以使用帳戶金鑰來建立 Azure 儲存體連結服務。 它可將儲存體的全域存取權提供給資料處理站。 以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為 **AzureTableStorage**。 |是 |
| connectionString | 針對 connectionString 屬性指定連線到儲存體所需的資訊。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載整合執行階段 (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>如果您使用 "AzureStorage" 類型連結服務，它仍會如現狀般受到支援，但建議您從現在開始使用這個新的 "AzureTableStorage" 連結服務類型。

**範例：**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-shared-access-signature-authentication"></a>使用共用存取簽章驗證

您也可以使用共用存取簽章來建立儲存體連結服務。 它提供受限制/時間界限存取權，讓資料處理站存取儲存體中的所有/特定資源。

共用存取簽章可提供您儲存體帳戶中資源的委派存取。 您可以使用它來將儲存體帳戶中物件的有限權限授與用戶端，讓該用戶端可以在一段指定時間內使用一組指定的權限進行存取。 您不需要共用您的帳戶存取金鑰。 共用存取簽章是一種 URI，此 URI 會在其查詢參數中包含對儲存體資源進行驗證式存取所需的一切資訊。 若要使用共用存取簽章存取儲存體資源，用戶端只需在適當的建構函式或方法中傳入共用存取簽章即可。 如需共用存取簽章的詳細資訊，請參閱[共用存取簽章：了解共用存取簽章模型](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

> [!NOTE]
> Data Factory 現已支援**服務共用存取簽章**和**帳戶共用存取簽章**。 如需這兩種類型及其建構方式的詳細資訊，請參閱[共用存取簽章的類型](../storage/common/storage-dotnet-shared-access-signature-part-1.md#types-of-shared-access-signatures)。 

> [!TIP]
> 若要為儲存體帳戶產生服務共用存取簽章，您可以執行下列 PowerShell 命令。 取代預留位置，並授與所需的權限。
> `$context = New-AzureStorageContext -StorageAccountName <accountName> -StorageAccountKey <accountKey>`
> `New-AzureStorageContainerSASToken -Name <containerName> -Context $context -Permission rwdl -StartTime <startTime> -ExpiryTime <endTime> -FullUri`

若要使用共用存取簽章驗證，以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設為 **AzureTableStorage**。 |是 |
| sasUri | 指定儲存體資源 (例如 Blob、容器或資料表) 的共用存取簽章 URI。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載 Integration Runtime (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>如果您使用 "AzureStorage" 類型連結服務，它仍會如現狀般受到支援，但建議您從現在開始使用這個新的 "AzureTableStorage" 連結服務類型。

**範例：**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureTableStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

當您建立共用存取簽章 URI 時，請考慮下列各點：

- 根據連結的服務 (讀取、寫入、讀取/寫入) 在資料處理站中的使用方式，在物件上設定適當的讀取/寫入權限。
- 設定適當的 [過期時間]。 請確定儲存體物件的存取權不會在管線的作用中期間內過期。
- 應視需要在正確的資料表層級建立 URI。

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供「Azure 資料表」資料集所支援的屬性清單。

若要將資料複製到「Azure 資料表」以及從該處複製資料，請將資料集的類型屬性設定為 **AzureTable**。 以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的 type 屬性必須設定為 **AzureTable**。 |是 |
| tableName |資料表儲存體資料庫執行個體中連結服務所參照的資料表名稱。 |是 |

**範例：**

```json
{
    "name": "AzureTableDataset",
    "properties":
    {
        "type": "AzureTable",
        "linkedServiceName": {
            "referenceName": "<Azure Table storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "MyTable"
        }
    }
}
```

### <a name="schema-by-data-factory"></a>Data factory 的結構描述

針對無結構描述的資料存放區 (如 Azure 資料表)，Data Factory 會以下列一種方式推斷結構描述：

* 如果您是使用資料集定義中的 **structure** 屬性來定義結構，Data Factory 會將此結構接受為結構描述。 在此情況下，如果資料列的資料行沒有值，系統就會為它提供 null 值。
* 如果您沒有使用資料集定義中的 **structure** 屬性來指定結構，Data Factory 服務將會使用資料的第一列來推斷結構描述。 在此情況下，如果第一個資料列未包含完整的結構描述，複製作業的結果中就會遺失一些資料行。

對於無結構描述的資料來源來說，最佳做法是使用 **structure** 屬性來指定資料結構。

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供「Azure 資料表」來源和接收所支援的屬性清單。

### <a name="azure-table-as-a-source-type"></a>Azure 資料表作為來源類型

若要從「Azure 資料表」複製資料，請將複製活動中的來源類型設定為 **AzureTableSource**。 複製活動的 [來源] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為 **AzureTableSource**。 |是 |
| AzureTableSourceQuery |使用自訂的資料表儲存體查詢來讀取資料。 請參閱下一節中的範例。 |否 |
| azureTableSourceIgnoreTableNotFound |指出是否讓資料表例外狀況不存在。<br/>允許的值為 **True** 和 **False** (預設值)。 |否 |

### <a name="azuretablesourcequery-examples"></a>azureTableSourceQuery 範例

如果 Azure 資料表資料行是日期時間類型：

```json
"azureTableSourceQuery": "LastModifiedTime gt datetime'2017-10-01T00:00:00' and LastModifiedTime le datetime'2017-10-02T00:00:00'"
```

如果 Azure 資料表資料行是字串類型：

```json
"azureTableSourceQuery": "LastModifiedTime ge '201710010000_0000' and LastModifiedTime le '201710010000_9999'"
```

如果您使用管線參數，請根據上述範例將日期時間值轉換成適當的格式。

### <a name="azure-table-as-a-sink-type"></a>Azure 資料表作為接收類型

若要將資料複製到 Azure 資料表，將複製活動中的接收類型設定為 **AzureTableSink**。 複製活動的 [接收] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收的 type 屬性必須設定為 **AzureTableSink**。 |是 |
| azureTableDefaultPartitionKeyValue |可供接收使用的預設資料分割索引鍵值。 |否 |
| azureTablePartitionKeyName |指定其值用來作為分割區索引鍵的資料行名稱。 如果未指定，則會使用 "AzureTableDefaultPartitionKeyValue" 作為分割區索引鍵。 |否 |
| azureTableRowKeyName |指定其值用來作為資料列索引鍵的資料行名稱。 如果未指定，則會針對每個資料列使用 GUID。 |否 |
| azureTableInsertType |將資料插入 Azure 資料表的模式。 此屬性可控制針對輸出資料表中具有相符分割區和資料列索引鍵的現有資料列，是要取代還是合併其值。 <br/><br/>允許的值為 **merge** (預設值) 和 **replace**。 <br/><br> 這個設定會在資料列層級套用，而不是在資料表層級套用。 這兩個選項都不會刪除輸出資料表中不存在於輸入的資料列。 若要了解合併和取代設定的運作方式，請參閱[插入或合併實體](https://msdn.microsoft.com/library/azure/hh452241.aspx)和[插入或取代實體](https://msdn.microsoft.com/library/azure/hh452242.aspx)。 |否 |
| writeBatchSize |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中。<br/>允許的值為整數 (資料列數目)。 |否 (預設值為 10000) |
| writeBatchTimeout |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中。<br/>允許的值為時間範圍。 範例是 "00:20:00" (20 分鐘)。 |否 (預設值為 90 秒，儲存體用戶端的預設逾時) |

**範例：**

```json
"activities":[
    {
        "name": "CopyToAzureTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Table output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureTableSink",
                "azureTablePartitionKeyName": "<column name>",
                "azureTableRowKeyName": "<column name>"
            }
        }
    }
]
```

### <a name="azuretablepartitionkeyname"></a>azureTablePartitionKeyName

您必須先使用 **"translator"** 屬性將來源資料行對應至目的地資料行，才能使用目的地資料行作為 azureTablePartitionKeyName。

在下列範例中，來源資料行 DivisionID 會對應至目的地資料行 DivisionID：

```json
"translator": {
    "type": "TabularTranslator",
    "columnMappings": "DivisionID: DivisionID, FirstName: FirstName, LastName: LastName"
}
```

"DivisionID" 會指定為分割區索引鍵。

```json
"sink": {
    "type": "AzureTableSink",
    "azureTablePartitionKeyName": "DivisionID"
}
```

## <a name="data-type-mapping-for-azure-table"></a>Azure 資料表的資料類型對應

從「Azure 資料表」複製資料或將資料複製到該處時，會使用下列從「Azure 資料表」資料類型對應到 Data Factory 過渡期資料類型的對應。 若要了解複製活動如何將來源結構描述和資料類型對應至接收，請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)。

將資料移入及移出 Azure 資料表時，從 Azure 資料表 OData 類型到 .NET 類型的對應會使用下列 [Azure 資料表定義的對應](https://msdn.microsoft.com/library/azure/dd179338.aspx)，反向對應亦適用。

| Azure 資料表資料類型 | Data Factory 過渡期資料類型 | 詳細資料 |
|:--- |:--- |:--- |
| Edm.Binary |byte[] |上限為 64 KB 的位元組陣列。 |
| Edm.Boolean |布林 |布林值。 |
| Edm.DateTime |Datetime |以國際標準時間 (UTC) 表示的 64 位元值。 支援的 DateTime 範圍從西元 1601 年 1 月 1 日午夜開始。 (C.E.), UTC. 此範圍結束於 9999 年 12 月 31 日。 |
| Edm.Double |double |64 位元的浮點值。 |
| Edm.Guid |Guid |128 位元的全域唯一識別碼。 |
| Edm.Int32 |Int32 |32 位元的整數。 |
| Edm.Int64 |Int64 |64 位元的整數。 |
| Edm.String |字串 |UTF 16 編碼值。 字串值最大可達 64 KB。 |

## <a name="next-steps"></a>後續步驟
如需 Data Factory 中的複製活動所支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
