---
title: 使用 Azure Data Factory 將資料複製到 Oracle 及從該處複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory 將資料從支援的來源存放區複製到 Oracle 資料庫，或從 Oracle 複製到支援的接收存放區。
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
ms.date: 06/14/2018
ms.author: jingwang
ms.openlocfilehash: ec0fc11ac2caf421f331a8fe72f1dacdf6b8a702
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42312063"
---
# <a name="copy-data-from-and-to-oracle-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 Oracle 複製資料及將資料複製到該處
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-onprem-oracle-connector.md)
> * [目前的版本](connector-oracle.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Oracle 資料庫複製資料及將資料複製到該處。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Oracle 資料庫複製到任何支援的接收資料存放區。 您也可以從任何支援的來源資料存放區將資料複製到 Oracle 資料庫。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Oracle 連接器支援下列版本的 Oracle 資料庫。 它也支援基本或 OID 驗證：

- Oracle 12c R1 (12.1)
- Oracle 11g R1、R2 (11.1、11.2)
- Oracle 10g R1、R2 (10.1、10.2)
- Oracle 9i R1、R2 (9.0.1、9.2)
- Oracle 8i R3 (8.1.7)

> [!Note]
> 不支援 Oracle Proxy 伺服器。

## <a name="prerequisites"></a>必要條件

若要從不可公開存取的 Oracle 資料庫複製資料，以及將資料複製到該處，您必須設定一個「自我裝載 Integration Runtime」。 如需整合執行階段的詳細資訊，請參閱[自我裝載 Integration Runtime](create-self-hosted-integration-runtime.md)。 整合執行階段提供內建的 Oracle 驅動程式。 因此，當您從 Oracle 複製資料或將資料複製到該處時，不需要手動安裝驅動程式。

## <a name="get-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Oracle 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Oracle 連結服務所支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設定為 **Oracle**。 | 是 |
| connectionString | 指定連線到 Oracle 資料庫執行個體所需的資訊。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。<br><br>**支援的連線類型**：您可以使用 [Oracle SID] 或 [Oracle 服務名稱] 來識別您的資料庫：<br>- 如果您使用 SID：`Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;`<br>- 如果您使用服務名稱：`Host=<host>;Port=<port>;ServiceName=<servicename>;User Id=<username>;Password=<password>;` | 是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!TIP]
>如果您遇到指出「ORA-01025: UPI 參數超出範圍」的錯誤，而且您的 Oracle 版本為 8i，請將 `WireProtocolMode=1` 新增至連接字串並再試一次。

若要啟用 Oracle 連線加密，您有兩個選項：

1.  在 Oracle 伺服器端，移至 Oracle 進階安全性 (OAS) 並設定加密設定，其支援三重 DES 加密 (3DES) 和進階加密標準 (AES)，請參閱[這裡](https://docs.oracle.com/cd/E11882_01/network.112/e40393/asointro.htm#i1008759)的詳細資料。 ADF Oracle 連接器會自動協商加密方法，以使用建立 Oracle 連線時您在 OAS 中設定的方法。

2.  在用戶端，您可以在連接字串中新增 `EncryptionMethod=1`。 這將使用 SSL/TLS 作為加密方法。 若要使用這個方法，您需要在 Oracle 伺服器端的 OAS 中停用非 SSL 加密設定，以避免加密衝突。

**範例：**

```json
{
    "name": "OracleLinkedService",
    "properties": {
        "type": "Oracle",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Oracle 資料集所支援的屬性清單。

若要從 Oracle 複製資料及將資料複製到該處，請將資料集的類型屬性設定為 **OracleTable**。 以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的 type 屬性必須設定為 **OracleTable**。 | 是 |
| tableName |Oracle 資料庫中連結服務所參照的資料表名稱。 | 是 |

**範例：**

```json
{
    "name": "OracleDataset",
    "properties":
    {
        "type": "OracleTable",
        "linkedServiceName": {
            "referenceName": "<Oracle linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "MyTable"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Oracle 來源和接收所支援的屬性清單。

### <a name="oracle-as-a-source-type"></a>Oracle 作為來源類型

若要從 Oracle 複製資料，請將複製活動中的來源類型設定為 **OracleSource**。 複製活動的 [來源] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為 **OracleSource**。 | 是 |
| oracleReaderQuery | 使用自訂 SQL 查詢來讀取資料。 例如 `"SELECT * FROM MyTable"`。 | 否 |

如果您未指定 "oracleReaderQuery"，就會使用資料集的 "structure" 區段中定義的資料行，來建構要針對 Oracle 資料庫執行的查詢 (`select column1, column2 from mytable`)。 如果資料集定義沒有 "structure"，則會從資料表中選取所有資料行。

**範例：**

```json
"activities":[
    {
        "name": "CopyFromOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Oracle input dataset name>",
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
                "type": "OracleSource",
                "oracleReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="oracle-as-a-sink-type"></a>Oracle 作為接收類型

若要將資料複製到 Oracle，請將複製活動中的接收器類型設定為 **OracleSink**。 複製活動的 [接收] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收的 type 屬性必須設定為 **OracleSink**。 | 是 |
| writeBatchSize | 當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中<br/>允許的值為整數 (資料列數目)。 |否 (預設值為 10000) |
| writeBatchTimeout | 在逾時前等待批次插入作業完成的時間。<br/>允許的值為時間範圍。 範例是 00:30:00 (30 分鐘)。 | 否 |
| preCopyScript | 指定一個供複製活動在每次執行時將資料寫入到 Oracle 前執行的 SQL 查詢。 您可以使用此屬性來清除預先載入的資料。 | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyToOracle",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Oracle output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "OracleSink"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-oracle"></a>Oracle 的資料類型對應

從 Oracle 複製資料及將資料複製到該處時，會使用下列從 Oracle 資料類型對應到 Data Factory 過渡期資料類型的對應。 若要了解複製活動如何將來源結構描述和資料類型對應至接收，請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)。

| Oracle 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(僅 Oracle 10g 及更高版本可支援) |
| CHAR |字串 |
| CLOB |字串 |
| 日期 |Datetime |
| FLOAT |Decimal，字串 (如果精確度 > 28) |
| INTEGER |Decimal，字串 (如果精確度 > 28) |
| 長 |字串 |
| 長 RAW |Byte[] |
| NCHAR |字串 |
| NCLOB |字串 |
| 數字 |Decimal，字串 (如果精確度 > 28) |
| NVARCHAR2 |字串 |
| RAW |Byte[] |
| ROWID |字串 |
| 時間戳記 |Datetime |
| 本地時區的時間戳記 |字串 |
| 時區的時間戳記 |字串 |
| 不帶正負號的整數 |數字 |
| VARCHAR2 |字串 |
| XML |字串 |

> [!NOTE]
> 不支援 INTERVAL YEAR TO MONTH 和 INTERVAL DAY TO SECOND 資料類型。


## <a name="next-steps"></a>後續步驟
如需 Data Factory 中的複製活動所支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
