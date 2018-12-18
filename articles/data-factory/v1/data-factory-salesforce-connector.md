---
title: 使用 Data Factory 從 Salesforce 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Salesforce 移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: dbe3bfd6-fa6a-491a-9638-3a9a10d396d1
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: a9dba65591479033a892615ff053eebd0862851e
ms.sourcegitcommit: b9786bd755c68d602525f75109bbe6521ee06587
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39125665"
---
# <a name="move-data-from-salesforce-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 Salesforce 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-salesforce-connector.md)
> * [第 2 版 (目前的版本)](../connector-salesforce.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Salesforce 連接器](../connector-salesforce.md)。


本文概述如何在 Azure Data Factory 使用複製活動，將資料從 Salesforce 複製到 [支援的來源與接收](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 資料表的 [接收] 欄底下列出的任何資料存放區。 本文是根據 [資料移動活動](data-factory-data-movement-activities.md) 一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

Azure Data Factory 目前只支援將資料從 Salesforce 移動到 [支援的接收資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)，但不支援將資料從其他資料存放區移動到 Salesforce。

## <a name="supported-versions"></a>支援的版本
此連接器使用下列其中一個 Salesforce 版本︰Developer Edition、Professional Edition、Enterprise Edition 或 Unlimited Edition。 並且支援從 Salesforce 生產、沙箱和自訂網域複製。

## <a name="prerequisites"></a>必要條件
* 必須啟用 API 權限。 請參閱 [如何在 Salesforce 中透過權限集啟用 API 存取權？](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)
* 若要將資料從 Salesforce 複製到內部部署資料存放區，您必須在內部部署環境中至少安裝資料管理閘道 2.0。

## <a name="salesforce-request-limits"></a>Salesforce 要求限制
Salesforce 對於 API 要求總數和並行 API 要求均有限制。 請注意下列幾點：

- 如果並行要求數目超過限制，便會進行節流，而且您會看到隨機失敗。
- 如果要求總數超過限制，Salesforce 帳戶將會封鎖 24 小時。

在上述兩種情況中，您也可能會收到「REQUEST_LIMIT_EXCEEDED」錯誤。 如需詳細資訊，請參閱 [Salesforce 開發人員限制](http://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf)文章的＜API 要求限制＞一節。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從 Salesforce 移動資料。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從 Salesforce 複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 Salesforce 複製到 Azure Blob](#json-example-copy-data-from-salesforce-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Salesforce 特定的 Data Factory 實體： 

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 Salesforce 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **Salesforce**。 |是 |
| environmentUrl | 指定 Salesforce 執行個體的 URL。 <br><br> - 預設為「https://login.salesforce.com」。 <br> - 若要從沙箱複製資料，請指定「https://test.salesforce.com」。 <br> - 若要從自訂網域複製資料，舉例來說，請指定 "https://[網域].my.salesforce.com"。 |否 |
| username |指定使用者帳戶的使用者名稱。 |是 |
| password |指定使用者帳戶的密碼。 |是 |
| securityToken |指定使用者帳戶的安全性權杖。 如需如何重設/取得安全性權杖的指示，請參閱 [取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) 。 若要整體了解安全性權杖，請參閱[安全性和 API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)。 |是 |

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱 [建立資料集](data-factory-create-datasets.md) 一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **RelationalTable** 類型資料集的 typeProperties 區段有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Salesforce 中資料表的名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱 [建立管線](data-factory-create-pipelines.md) 一文。 名稱、描述、輸入和輸出資料表以及各種原則等屬性都適用於所有活動類型。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

在複製活動中，如果來源類型為 **RelationalSource** (包含 Salesforce)，則 typeProperties 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢或 [Salesforce 物件查詢語言 (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm) 查詢。 例如：`select * from MyTable__c`。 |否 (如果已指定 **dataset** 的 **tableName**) |

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## <a name="query-tips"></a>查詢秘訣
### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>在 DateTime 資料行上使用 Where 子句來擷取資料
指定 SOQL 或 SQL 查詢時，請注意 DateTime 格式差異。 例如︰

* **SOQL 範例**：`$$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', WindowStart, WindowEnd)`
* **SQL 範例**：
    * **使用複製精靈指定查詢︰**`$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)`
    * **使用 JSON 編輯指定查詢 (適當地逸出字元)︰**`$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\\'{0:yyyy-MM-dd HH:mm:ss}\\'}} AND LastModifiedDate < {{ts\\'{1:yyyy-MM-dd HH:mm:ss}\\'}}', WindowStart, WindowEnd)`

### <a name="retrieving-data-from-salesforce-report"></a>從 Salesforce 報表擷取資料
例如，您可以藉由以 `{call "<report name>"}` 方式指定查詢，從 Salesforce 報表擷取資料。 `"query": "{call \"TestReport\"}"`。

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>從 Salesforce 資源回收筒擷取已刪除的記錄
若要從「Salesforce 資源回收筒」查詢虛刪除記錄，您可以在查詢中指定 **"IsDeleted = 1"**。 例如，

* 若只要查詢已刪除的記錄，請指定 "select * from MyTable__c **where IsDeleted= 1**"
* 若要查詢所有記錄 (包括現有和已刪除的記錄)，請指定 "select * from MyTable__c **where IsDeleted = 0 or IsDeleted = 1**"

## <a name="json-example-copy-data-from-salesforce-to-azure-blob"></a>JSON 範例：將資料從 Salesforce 複製到 Azure Blob
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 這些範例示範如何將資料從 Salesforce 複製到 Azure Blob 儲存體。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。   

以下是為了實作案例所必須建立的 Data Factory 構件。 清單後面的各節會提供有關這些步驟的詳細資料。

* [Salesforce](#linked-service-properties)
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* [RelationalTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)
* 具有使用 [RelationalSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)

**Salesforce 連結服務**

此範例使用 **Salesforce** 連結服務。 如需此連結服務所支援的屬性，請參閱 [Salesforce 連結服務](#linked-service-properties) 一節。  如需如何重設/取得安全性權杖的指示，請參閱 [取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) 。

```json
{
    "name": "SalesforceLinkedService",
    "properties":
    {
        "type": "Salesforce",
        "typeProperties":
        {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```
**Azure 儲存體連結服務**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
    "type": "AzureStorage",
    "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
    }
}
```
**Salesforce 輸入資料集**

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"  
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

將 **external** 設定為 **true** 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/alltypes_c"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**具有複製活動的管線**

此管線包含「複製活動」，該活動已設定為使用輸入和輸出資料集，並且排定為每小時執行。 在管線 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。

如需 RelationalSource 支援的屬性清單，請參閱 [RelationalSource 類型屬性](#copy-activity-properties) 。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2016-06-01T18:00:00",
        "end":"2016-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
        {
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "SalesforceInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"                
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }
        ]
    }
}
```
> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)


### <a name="type-mapping-for-salesforce"></a>Salesforce 的類型對應
| Salesforce 類型 | 以 .NET 為基礎的類型 |
| --- | --- |
| 自動編號 |字串 |
| 核取方塊 |BOOLEAN |
| 貨幣 |十進位 |
| 日期 |Datetime |
| 日期/時間 |Datetime |
| 電子郵件 |字串 |
| id |字串 |
| 查閱關聯性 |字串 |
| 複選挑選清單 |字串 |
| 數字 |十進位 |
| 百分比 |十進位 |
| 電話 |字串 |
| 挑選清單 |字串 |
| 文字 |字串 |
| 文字區域 |字串 |
| 文字區域 (完整) |字串 |
| 文字區域 (豐富) |字串 |
| 文字 (加密) |字串 |
| URL |字串 |

> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>效能和微調
若要了解 Azure Data Factory 中影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法，請參閱 [複製活動的效能及微調指南](data-factory-copy-activity-performance.md) 。
