---
title: 使用 Azure Data Factory 從 Amazon Redshift 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 將資料從 Amazon Redshift 複製到支援的接收資料存放區。
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
ms.date: 02/07/2018
ms.author: jingwang
ms.openlocfilehash: 6d36733b63645fd86580ccdc5af756739f77338c
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048140"
---
# <a name="copy-data-from-amazon-redshift-using-azure-data-factory"></a>使用 Azure Data Factory 從 Amazon Redshift 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-amazon-redshift-connector.md)
> * [目前的版本](connector-amazon-redshift.md)


本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Amazon Redshift 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Amazon Redshift 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Amazon Redshift 連接器支援使用查詢或內建的 Redshift UNLOAD 支援，從 Redshift 擷取資料。

> [!TIP]
> 在 Redshift 中複製大量資料時，若想獲得最佳效能，請考慮透過 Amazon S3 使用內建的 Redshift UNLOAD。 如需詳細資料，請參閱「[使用 UNLOAD 複製 Amazon Redshift 中的資料](#use-unload-to-copy-data-from-amazon-redshift)」章節。

## <a name="prerequisites"></a>先決條件

* 如果您要使用[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md) 將資料複製到內部部署資料存放區，請將 Amazon Redshift 叢集的存取權授與 Integration Runtime (使用電腦的 IP 位址)。 如需相關指示，請參閱 [授權存取叢集](http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) 。
* 如果您要將資料複製到 Azure 資料存放區，請參閱 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653) 以取得 Azure 資料中心所使用的計算 IP 位址和 SQL 範圍。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Amazon Redshift 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Amazon Redshift 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**AmazonRedshift** | yes |
| 伺服器 |Amazon Redshift 伺服器的 IP 位址或主機名稱。 |yes |
| 連接埠 |Amazon Redshift 伺服器用來接聽用戶端連線的 TCP 連接埠號碼。 |否，預設值為 5439 |
| 資料庫 |Amazon Redshift 資料庫的名稱。 |yes |
| username |可存取資料庫之使用者的名稱。 |yes |
| password |使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或「自我裝載 Integration Runtime」(如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

**範例：**

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "<server name>",
            "database": "<database name>",
            "username": "<username>",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Amazon Redshift 資料集所支援的屬性清單。

若要從 Amazon Redshift 複製資料，請將資料集的類型屬性設定為 **RelationalTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**RelationalTable** | yes |
| tableName | Amazon Redshift 中的資料表名稱。 | 否 (如果已指定活動來源中的「查詢」) |

**範例**

```json
{
    "name": "AmazonRedshiftDataset",
    "properties":
    {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Amazon Redshift linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Amazon Redshift 來源所支援的屬性清單。

### <a name="amazon-redshift-as-source"></a>Amazon Redshift 作為來源

若要從 Amazon Redshift 複製資料，請將複製活動中的來源類型設定為 **AmazonRedshiftSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**AmazonRedshiftSource** | yes |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：select * from MyTable。 |否 (如果已指定資料集中的 "tableName") |
| redshiftUnloadSettings | 使用 Amazon Redshift UNLOAD 時的屬性群組。 | 否 |
| s3LinkedServiceName | 係指要作為暫時存放區的 Amazon S3 (藉由指定 "AmazonS3" 類型的已連結服務名稱)。 | 如果使用 UNLOAD，則為必要 |
| bucketName | 表示儲存暫時資料的 S3 貯體。 如果為提供，Data Factory 服務就會自動產生它。  | 如果使用 UNLOAD，則為必要 |

**範例：使用 UNLOAD 之複製活動中的 Amazon Redshift 來源**

```json
"source": {
    "type": "AmazonRedshiftSource",
    "query": "<SQL query>",
    "redshiftUnloadSettings": {
        "s3LinkedServiceName": {
            "referenceName": "<Amazon S3 linked service>",
            "type": "LinkedServiceReference"
        },
        "bucketName": "bucketForUnload"
    }
}
```

若要深入了解如何使用 UNLOAD 從 Amazon Redshift 有效率地複製資料，請參閱下一節。

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>使用 UNLOAD 複製 Amazon Redshift 中的資料

[UNLOAD](http://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) (英文) 是 Amazon Redshift 提供的機制，可以為 Amazon Simple Storage Service (Amazon S3) 中的一或多個檔案卸載查詢結果。 Amazon 建議使用此方式從 Redshift 複製大型資料集。

**範例：使用 UNLOAD、分段複製及 PolyBase 將資料從 Amazon Redshift 複製到 Azure SQL 資料倉儲**

就這個範例使用案例而言，複製活動會依照 "redshiftUnloadSettings" 中的設定，將資料從 Amazon Redshift 卸載到 Amazon S3，然後依照 "stagingSettings" 中指定的方式，將資料從 Amazon S3 複製到 Azure Blob，最後再使用 PolyBase 將資料載入到「SQL 資料倉儲」。 複製活動會正確地處理所有暫時格式。

![從 Redshift 到 SQL DW 的複製工作流程](media\copy-data-from-amazon-redshift\redshift-to-sql-dw-copy-workflow.png)

```json
"activities":[
    {
        "name": "CopyFromAmazonRedshiftToSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "AmazonRedshiftDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AmazonRedshiftSource",
                "query": "select * from MyTable",
                "redshiftUnloadSettings": {
                    "s3LinkedServiceName": {
                        "referenceName": "AmazonS3LinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "bucketName": "bucketForUnload"
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": "AzureStorageLinkedService",
                "path": "adfstagingcopydata"
            },
            "dataIntegrationUnits": 32
        }
    }
]
```

## <a name="data-type-mapping-for-amazon-redshift"></a>Amazon Redshift 的資料類型對應

從 Amazon Redshift 複製資料時，會使用下列對應從 Amazon Redshift 資料類型對應到 Azure Data Factory 過渡期資料類型。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| Amazon Redshift 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| BIGINT |Int64 |
| BOOLEAN |字串 |
| CHAR |字串 |
| 日期 |Datetime |
| DECIMAL |十進位 |
| DOUBLE PRECISION |兩倍 |
| INTEGER |Int32 |
| REAL |單一 |
| SMALLINT |Int16 |
| TEXT |字串 |
| 時間戳記 |Datetime |
| VARCHAR |字串 |

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
