---
title: 使用 Azure Data Factory 從 DB2 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 複製活動從內部部署 DB2 資料庫移動資料
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: c1644e17-4560-46bb-bf3c-b923126671f1
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 88e56f522545f9c1f38bf0d0fdbcebdc171c294b
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046525"
---
# <a name="move-data-from-db2-by-using-azure-data-factory-copy-activity"></a>使用 Azure Data Factory 複製活動從 DB2 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-onprem-db2-connector.md)
> * [第 2 版 (目前的版本)](../connector-db2.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 DB2 連接器](../connector-db2.md)。


本文將說明如何使用 Azure Data Factory 中的複製活動將內部部署 DB2 資料庫的資料複製到資料存放區。 您可以將資料複製到在[Data Factory 資料移動活動](data-factory-data-movement-activities.md#supported-data-stores-and-formats)發行項中列為支援接收的任何存放區。 本文以「資料處理站」一文為基礎，該文呈現使用複製活動移動資料的概觀，並列出支援的資料存放區組合。 

資料處理站目前僅支援資料從 DB2 資料庫移至[支援的接收資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 不支援資料從其他資料存放區移至 DB2 資料庫。

## <a name="prerequisites"></a>先決條件
資料處理站支援使用[資料管理閘道](data-factory-data-management-gateway.md)連接至內部部署 DB2 資料庫。 如需有關設定閘道資料管線來移動資料的逐步指示，請參閱[將資料從內部部署移到雲端](data-factory-move-data-between-onprem-and-cloud.md)一文。

即使 DB2 裝載於 Azure IaaS VM 中，也必須要有閘道。 您可以在資料存放區所在的 IaaS VM 上安裝閘道。 如果閘道可以連線到資料庫，您可以在不同的 VM 上安裝閘道。

資料管理閘道提供內建的 DB2 驅動程式，因此，不需要手動安裝任何驅動程式，即可從 DB2 複製資料。

> [!NOTE]
> 如需連線和閘道器問題的疑難排解秘訣，請參閱[針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues)一文。


## <a name="supported-versions"></a>支援的版本
此資料處理站 DB2 連接器支援以下 IBM DB2 平台和版本，以及支援分散式關聯資料庫架構 (DRDA) SQL 存取管理員版本 9、10 和 11：

* IBM DB2 for z/OS 版本 11.1
* IBM DB2 for z/OS 版本 10.1
* IBM DB2 for i (AS400) 版本 7.2
* IBM DB2 for i (AS400) 版本 7.1
* IBM DB2 for Linux, UNIX, and Windows (LUW) 版本 11
* IBM DB2 for LUW 版本 10.5
* IBM DB2 for LUW 版本 10.1

> [!TIP]
> 如果收到錯誤訊息「找不到對應至 SQL 陳述式執行要求的套件。 SQLSTATE=51002 SQLCODE=-805」，原因是作業系統上未針對一般使用者建立所需的套件。 若要解決此問題，請針對 DB2 伺服器類型遵循這些指示：
> - DB2 for i (AS400)：進行複製活動之前，讓進階使用者建立一般使用者的集合。 若要建立集合，請使用命令：`create collection <username>`
> - DB2 for z/OS 或 LUW：使用高權限帳戶 -- 具有套件授權單位與 BIND、BINDADD、GRANT EXECUTE TO PUBLIC 權限的進階使用者 -- 執行一次複製。 在複製期間，會自動建立所需的套件。 之後，您可以切換至一般使用者，來執行後續的複製。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具和 API，建立內含複製活動的管線，以從內部部署的 DB2 資料存放區移動資料： 

- 建立管線的最簡單方式就是使用「Azure Data Factory 複製精靈」。 如需使用複製精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。 
- 您也可以使工具來建立管線，包括 Azure 入口網站、Visual Studio、Azure PowerShell、Azure Resource Manager 範本、.NET API 及 REST API。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立連結服務，將輸入和輸出資料存放區連結到資料處理站。
2. 建立資料集，代表複製作業的輸入和輸出資料。 
3. 建立管線，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用複製精靈時，精靈會自動為您建立已連結 Data Factory 之服務、資料集及管線實體的 JSON 定義。 使用工具或 API (.NET API 除外) 時，您需使用 JSON 格式來定義資料處理站實體。 [JSON 範例：將資料從 DB2 複製到 Azure Blob 儲存體](#json-example-copy-data-from-db2-to-azure-blob)顯示用來從內部部署 DB2 資料存放區複製資料的 Data Factory 實體本身的 JSON 定義。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 DB2 資料存放區特定的 Data Factory 實體。

## <a name="db2-linked-service-properties"></a>DB2 連結服務屬性
下表列出 DB2 連結服務特定的 JSON 屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **type** |此屬性必須設為 **OnPremisesDB2**。 |yes |
| **server** |DB2 伺服器的名稱。 |yes |
| **database** |DB2 資料庫的名稱。 |yes |
| **schema** |在 DB2 資料庫中的結構描述名稱。 此屬性必須區分大小寫。 |否 |
| **authenticationType** |用來連接到 DB2 資料庫的驗證類型。 可能的值為：匿名、基本和 Windows。 |yes |
| **username** |使用者帳戶的名稱 (如果您使用基本或 Windows 驗證)。 |否 |
| **password** |使用者帳戶的密碼。 |否 |
| **gatewayName** |Data Factory 服務應該用來連接到內部部署 DB2 資料庫的閘道器名稱。 |yes |

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的**結構**、**可用性**和**原則**等區段類似於所有的資料集類型 (Azure SQL、Azure Blob 儲存體、Azure 資料表儲存體等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **RelationalTable** 類型資料集的 **typeProperties** 區段 (包含 DB2 資料集) 具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **tableName** |DB2 資料庫執行個體中連結服務所參照的資料表名稱。 此屬性必須區分大小寫。 |否 (如果指定 **RelationalSource** 類型複製活動的**查詢**屬性) |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義複製活動的區段和屬性清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 複製活動屬性 (例如**名稱**、**描述**、**輸入**資料表、**輸出**資料表，以及**原則**) 適用於所有類型的活動。 活動 **typeProperties** 區段中的可用屬性，會因為活動類型不同而有所差異。 就複製活動而言，屬性會根據資料來源和接收的類型而有所不同。

在複製活動中，如果來源類型為 **RelationalSource** (包含 DB2)，則 **typeProperties** 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| **查詢** |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：`"query": "select * from "MySchema"."MyTable""` |否 (如果已指定資料集的 **tableName** 屬性) |

> [!NOTE]
> 結構描述和資料表名稱會區分大小寫。 在查詢陳述式中，使用 "" (雙引號) 括住屬性名稱。

## <a name="json-example-copy-data-from-db2-to-azure-blob-storage"></a>JSON 範例：將資料從 DB2 複製到 Azure Blob 儲存體
此範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 此範例示範如何將資料從 DB2 資料庫複製到 Blob 儲存體。 不過，可以使用 Azure Data Factory 複製活動，將資料複製到[任何支援的資料存放區接收類型](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。

範例有下列 Data Factory 實體：

- [OnPremisesDb2](data-factory-onprem-db2-connector.md#linked-service-properties)類型的 DB2 連結服務
- [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 類型的 Azure Blob 儲存體連結服務
- [RelationalTable](data-factory-onprem-db2-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)
- [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)
- 具有使用 [RelationalSource](data-factory-onprem-db2-connector.md#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 屬性之複製活動的[管線](data-factory-create-pipelines.md)

此範例會每個小時將資料從 DB2 資料庫中的查詢結果複製到 Azure Blob。 實體定義後面的各節會說明範例中使用的 JSON 屬性。

第一個步驟是安裝和設定資料閘道。 如需相關指示，請參閱[在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文。

**DB2 連結服務**

```json
{
    "name": "OnPremDb2LinkedService",
    "properties": {
        "type": "OnPremisesDb2",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

**Azure Blob 儲存體連結服務**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorageLinkedService",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<AccountName>;AccountKey=<AccountKey>"
        }
    }
}
```

**DB2 輸入資料集**

此範例假設您已在 DB2 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestamp")。

**外部**屬性是設定為 "true"。 此設定可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。 請注意，**類型**屬性是設定為 **RelationalTable**。


```json
{
    "name": "Db2DataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremDb2LinkedService",
        "typeProperties": {},
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

**Azure Blob 輸出資料集**

藉由將 **frequency** 屬性設定 "Hour"，並將 **interval** 屬性設定為 1，資料會每小時寫入到新的 Blob。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的 **folderPath** 屬性。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobDb2DataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/db2/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**複製活動的管線**

此管線包含複製活動，該活動已設定為使用指定的輸入和輸出資料集並排定為每小時執行。 在管線的 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。 針對 **query** 屬性指定的 SQL 查詢會從 "Orders" 資料表選取資料。

```json
{
    "name": "CopyDb2ToBlob",
    "properties": {
        "description": "pipeline for the copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "select * from \"Orders\""
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "inputs": [
                    {
                        "name": "Db2DataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobDb2DataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "Db2ToBlob"
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```

## <a name="type-mapping-for-db2"></a>DB2 的類型對應
如 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會使用下列含有兩個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

複製活動將資料從 DB2 類型轉換成.NET 型別時，會使用下列對應：

| DB2 資料庫類型 | .NET Framework 類型 |
| --- | --- |
| SmallInt |Int16 |
| 整數  |Int32 |
| BigInt |Int64 |
| Real |單一 |
| 兩倍 |兩倍 |
| Float |兩倍 |
| 十進位 |十進位 |
| DecimalFloat |十進位 |
| 數值 |十進位 |
| 日期 |Datetime |
| 時間 |時間範圍 |
| Timestamp |Datetime |
| xml |Byte[] |
| Char |字串 |
| VarChar |字串 |
| LongVarChar |字串 |
| DB2DynArray |字串 |
| Binary |Byte[] |
| VarBinary |Byte[] |
| LongVarBinary |Byte[] |
| 圖形 |字串 |
| VarGraphic |字串 |
| LongVarGraphic |字串 |
| Clob |字串 |
| Blob |Byte[] |
| DbClob |字串 |
| SmallInt |Int16 |
| 整數  |Int32 |
| BigInt |Int64 |
| Real |單一 |
| 兩倍 |兩倍 |
| Float |兩倍 |
| 十進位 |十進位 |
| DecimalFloat |十進位 |
| 數值 |十進位 |
| 日期 |Datetime |
| 時間 |時間範圍 |
| Timestamp |Datetime |
| xml |Byte[] |
| Char |字串 |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-reads-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請留意可重複性，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試**原則**屬性，使得在發生失敗時，重新執行配量。 確定無論配量將執行多少次，而且無論如何重新執行配量，均讀取相同的資料。 如需詳細資訊，請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)，了解影響複製活動效能的重要因素，以及達到最佳效能的各種方法。
