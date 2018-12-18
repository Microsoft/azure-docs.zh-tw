---
title: 使用 Azure Data Factory 從 Cassandra 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從 Cassandra 將資料複製到支援的接收資料存放區。
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
ms.date: 06/07/2018
ms.author: jingwang
ms.openlocfilehash: a0095ae4aa50845a24cabb981399ac4035afdebe
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051445"
---
# <a name="copy-data-from-cassandra-using-azure-data-factory"></a>使用 Azure Data Factory 從 Cassandra 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-onprem-cassandra-connector.md)
> * [目前的版本](connector-cassandra.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Cassandra 資料庫複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以從 Cassandra 資料庫將資料複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此 Cassandra 連接器支援：

- Cassandra **2.x 版和 3.x 版**。
- 使用 **Basic** (基本) 或 **Anonymous** (匿名) 驗證來複製資料。

>[!NOTE]
>針對在自我裝載 Integration Runtime 上執行的活動，從 IR 3.7 版及更新版本開始支援 Cassandra 3.x。

## <a name="prerequisites"></a>先決條件

若要從不可公開存取的 Cassandra 資料庫複製資料，您應該設定一個「自我裝載整合執行階段」。 如需詳細資料，請參閱[自我裝載整合執行階段](create-self-hosted-integration-runtime.md)一文。 「整合執行階段」提供內建的 Cassandra 驅動程式，因此從 Cassandra 複製資料或將資料複製到該處時，您不需要手動安裝任何驅動程式。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Cassandra 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Cassandra 連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |類型屬性必須設定為：**Cassandra** |yes |
| host |一或多個 Cassandra 伺服器 IP 位址或主機名稱。<br/>指定以逗號分隔的 IP 位址或主機名稱清單，以同時連線到所有伺服器。 |yes |
| 連接埠 |Cassandra 伺服器用來接聽用戶端連線的 TCP 連接埠。 |否 (預設值為 9042) |
| authenticationType | 用來連接到 Cassandra 資料庫的驗證類型。<br/>允許的值為：**Basic** (基本) 和 **Anonymous** (匿名)。 |yes |
| username |指定使用者帳戶的使用者名稱。 |是，如果 authenticationType 設定為 [基本]。 |
| password |指定使用者帳戶的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是，如果 authenticationType 設定為 [基本]。 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用「自我裝載 Integration Runtime」或 Azure Integration Runtime (如果您的資料存放區是可公開存取的)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>目前不支援使用 SSL 與 Cassandra 連線。

**範例：**

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "Cassandra",
        "typeProperties": {
            "host": "<host>",
            "authenticationType": "Basic",
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Cassandra 資料集所支援的屬性清單。

若要從 Cassandra 複製資料，請將資料集的類型屬性設定為 **CassandraTable**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**CassandraTable** | yes |
| keyspace |Cassandra 資料庫中的 Keyspace 或結構描述名稱。 |否 (如果已指定「CassandraSource」的「查詢」) |
| tableName |Cassandra 資料庫中資料表的名稱。 |否 (如果已指定「CassandraSource」的「查詢」) |

**範例：**

```json
{
    "name": "CassandraDataset",
    "properties": {
        "type": "CassandraTable",
        "linkedServiceName": {
            "referenceName": "<Cassandra linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "keySpace": "<keyspace name>",
            "tableName": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性


如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Cassandra 來源所支援的屬性清單。

### <a name="cassandra-as-source"></a>Cassandra 作為來源

若要從 Cassandra 複製資料，請將複製活動中的來源類型設定為 **CassandraSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**CassandraSource** | yes |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢或 CQL 查詢。 請參閱 [CQL 參考資料](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)。 <br/><br/>在使用 SQL 查詢時，指定 **keyspace name.table 名稱** 來代表您想要查詢的資料表。 |否 (如果已指定資料集中的「tableName」和「keyspace」)。 |
| consistencyLevel |一致性層級可指定必須先有多少複本回應讀取要求，才會將資料傳回用戶端應用程式。 Cassandra 會檢查要讓資料滿足讀取要求的指定複本數目。 如需詳細資訊，請參閱 [設定資料一致性](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) 。<br/><br/>允許的值為：**ONE**、**TWO**、**THREE**、**QUORUM**、**ALL**、**LOCAL_QUORUM**、**EACH_QUORUM** 和 **LOCAL_ONE**。 |否 (預設值為 `ONE`) |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromCassandra",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Cassandra input dataset name>",
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
                "type": "CassandraSource",
                "query": "select id, firstname, lastname from mykeyspace.mytable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-cassandra"></a>Cassandra 的資料類型對應

從 Cassandra 複製資料時，會使用下列從 Cassandra 資料類型對應到 Azure Data Factory 過渡期資料類型的對應。 請參閱[結構描述和資料類型對應](copy-activity-schema-and-type-mapping.md)，以了解複製活動如何將來源結構描述和資料類型對應至接收器。

| Cassandra 資料類型 | Data Factory 過渡期資料類型 |
|:--- |:--- |
| ASCII |字串 |
| BIGINT |Int64 |
| BLOB |Byte[] |
| BOOLEAN |BOOLEAN |
| DECIMAL |十進位 |
| DOUBLE |DOUBLE |
| FLOAT |單一 |
| INET |字串 |
| INT |Int32 |
| TEXT |字串 |
| 時間戳記 |Datetime |
| TIMEUUID |Guid |
| UUID |Guid |
| VARCHAR |字串 |
| VARINT |DECIMAL |

> [!NOTE]
> 如需集合類型 (對應、集、清單等)，請參閱 [使用虛擬資料表處理 Cassandra 集合類型](#work-with-collections-using-virtual-table) 一節。
>
> 不支援使用者定義的類型。
>
> 二進位資料行與字串資料行的長度不能超過 4000。
>

## <a name="work-with-collections-using-virtual-table"></a>使用虛擬資料表處理集合

Azure Data Factory 會使用內建的 ODBC 驅動程式來連線到 Cassandra 資料庫並從中複製資料。 針對包含對應、集和清單在內的集合類型，此驅動程式會將資料重新標準化為對應的虛擬資料表。 具體來說，如果資料表包含任何集合資料行，則此驅動程式會產生下列虛擬資料表︰

* **基底資料表**，其中包含與實際資料表相同的資料 (集合資料行除外)。 基底資料表使用與它所代表的實際資料表相同的名稱。
* 每個集合資料行的 **虛擬資料表** ，以展開巢狀資料。 代表集合的虛擬資料表會使用實際資料表名稱、分隔字元「vt」和資料行名稱來命名。

虛擬資料表會參考實際資料表中的資料，讓驅動程式得以存取反正規化的資料。 如需詳細資訊，請參閱＜範例＞一節。 您可以藉由查詢和聯結虛擬資料表來存取 Cassandra 集合的內容。

### <a name="example"></a>範例

例如，以下的「ExampleTable」就是 Cassandra 資料庫資料表，其中包含名為「pk_int」的整數主索引鍵資料行、名為「值」的文字資料行、「清單」資料行、「對應」資料行和「集」資料行 (名為「StringSet」)。

| pk_int | 值 | 列出 | 對應 | StringSet |
| --- | --- | --- | --- | --- |
| 1 |"sample value 1" |["1", "2", "3"] |{"S1": "a", "S2": "b"} |{"A", "B", "C"} |
| 3 |"sample value 3" |["100", "101", "102", "105"] |{"S1": "t"} |{"A", "E"} |

驅動程式會產生多個代表這個單一資料表的虛擬資料表。 虛擬資料表中的外部索引鍵資料行會參考實際資料表中的主索引鍵資料行，並指出虛擬資料表資料列所對應的實際資料表資料列。

第一個虛擬資料表是名為「ExampleTable」的基底資料表，如下表所示： 

| pk_int | 值 |
| --- | --- |
| 1 |"sample value 1" |
| 3 |"sample value 3" |

基底資料表包含與原始資料庫資料表相同的資料，但集合除外，集合會被此資料表省略，而在其他虛擬資料表中展開。

下表顯示會重新標準化「清單」、「對應」與「StringSet」資料行中資料的虛擬資料表。 名稱結尾為「_index」或「_key」的資料行代表資料在原始清單或對應內的位置。 名稱結尾為「_value」的資料行包含從集合展開的資料。

**資料表「ExampleTable_vt_List」：**

| pk_int | List_index | List_value |
| --- | --- | --- |
| 1 |0 |1 |
| 1 |1 |2 |
| 1 |2 |3 |
| 3 |0 |100 |
| 3 |1 |101 |
| 3 |2 |102 |
| 3 |3 |103 |

**資料表「ExampleTable_vt_Map」：**

| pk_int | Map_key | Map_value |
| --- | --- | --- |
| 1 |S1 |具有使用  |
| 1 |S2 |b |
| 3 |S1 |t |

**資料表「ExampleTable_vt_StringSet」：**

| pk_int | StringSet_value |
| --- | --- |
| 1 |具有使用  |
| 1 |b |
| 1 |C |
| 3 |具有使用  |
| 3 |E |

## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
