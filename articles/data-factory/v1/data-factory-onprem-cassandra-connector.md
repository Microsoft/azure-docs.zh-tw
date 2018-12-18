---
title: 使用 Data Factory 從 Cassandra 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從內部部署的 Cassandra 資料庫移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 085cc312-42ca-4f43-aa35-535b35a102d5
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c4ed3a22d3ad4e227178e8ac265cc97050e31ee6
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054304"
---
# <a name="move-data-from-an-on-premises-cassandra-database-using-azure-data-factory"></a>使用 Azure Data Factory 從內部部署的 Cassandra 資料庫移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-onprem-cassandra-connector.md)
> * [第 2 版 (目前的版本)](../connector-cassandra.md)

> [!NOTE]
> 本文適用於第 1 版的 Data Factory。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Cassandra 連接器](../connector-cassandra.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署的 Cassandra 資料庫移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從內部部署的 Cassandra 資料存放區複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從 Cassandra 資料存放區移到其他資料存放區，而不支援將資料從其他資料存放區移到 Cassandra 資料存放區。 

## <a name="supported-versions"></a>支援的版本
Cassandra 連接器支援下列 Cassandra 版本：2.x 和 3.x。 針對在自我裝載 Integration Runtime 上執行的活動，從 IR 3.7 版及更新版本開始支援 Cassandra 3.x。

## <a name="prerequisites"></a>先決條件
若要讓 Azure Data Factory 服務能夠連接到內部部署的 Cassandra 資料庫，您必須在裝載資料庫的同一部電腦上或在個別的電腦上安裝「資料管理閘道」，以避免發生與資料庫競用資源的情況。 「資料管理閘道」是一個元件，可透過既安全又受控方式，將內部部署的資料來源連接到雲端服務。 如需資料管理閘道的詳細資料，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 一文。 如需有關為閘道設定資料管線來移動資料的逐步指示，請參閱[將資料從內部部署移到雲端](data-factory-move-data-between-onprem-and-cloud.md)。

您必須使用閘道來連接到 Cassandra 資料庫，即使該資料庫裝載在雲端中 (例如在 Azure IaaS VM 上) 也一樣。 您可以讓閘道位於裝載資料庫的同一部 VM 上，也可以讓它位於個別的 VM 上，只要該閘道可以連接到資料庫即可。  

當您安裝閘道時，它會自動安裝用來連線至 Cassandra 資料庫的 Microsoft Cassandra ODBC 驅動程式。 因此，從 Cassandra 資料庫複製資料時，您不需要在閘道電腦上手動安裝任何驅動程式。 

> [!NOTE]
> 如需連接/閘道器相關問題的疑難排解秘訣，請參閱 [針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) 。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從內部部署的 Cassandra 資料存放區移動資料。 

- 若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。 
- 您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從內部部署 Cassandra 資料存放區複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 Cassandra 複製到 Azure Blob](#json-example-copy-data-from-cassandra-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Cassandra 資料存放區特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 Cassandra 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **OnPremisesCassandra** |yes |
| host |一或多個 Cassandra 伺服器 IP 位址或主機名稱。<br/><br/>指定以逗號分隔的 IP 位址或主機名稱清單，以同時連線到所有伺服器。 |yes |
| 連接埠 |Cassandra 伺服器用來接聽用戶端連線的 TCP 連接埠。 |否，預設值：9042 |
| authenticationType |基本或匿名 |yes |
| username |指定使用者帳戶的使用者名稱。 |是，如果 authenticationType 設定為 [基本]。 |
| password |指定使用者帳戶的密碼。 |是，如果 authenticationType 設定為 [基本]。 |
| gatewayName |用來連線到內部部署 Cassandra 資料庫的閘道器名稱。 |yes |
| encryptedCredential |由閘道加密認證。 |否 |

>[!NOTE]
>目前不支援使用 SSL 與 Cassandra 連線。

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **CassandraTable** 類型資料集的 typeProperties 區段有下列屬性

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| keyspace |Cassandra 資料庫中的 Keyspace 或結構描述名稱。 |是 (如果未定義 **CassandraSource** 的**查詢**)。 |
| tableName |Cassandra 資料庫中資料表的名稱。 |是 (如果未定義 **CassandraSource** 的**查詢**)。 |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

而活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

如果來源類型為 **CassandraSource**，則 typeProperties 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢或 CQL 查詢。 請參閱 [CQL 參考資料](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)。 <br/><br/>在使用 SQL 查詢時，指定 **keyspace name.table 名稱** 來代表您想要查詢的資料表。 |否 (如果已定義資料集上的 tableName 和 keyspace)。 |
| consistencyLevel |一致性層級可指定必須先有多少複本回應讀取要求，才會將資料傳回用戶端應用程式。 Cassandra 會檢查要讓資料滿足讀取要求的指定複本數目。 |ONE、TWO、THREE、QUORUM、ALL、LOCAL_QUORUM、EACH_QUORUM、LOCAL_ONE。 如需詳細資訊，請參閱 [設定資料一致性](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) 。 |否。 預設值為 ONE。 |

## <a name="json-example-copy-data-from-cassandra-to-azure-blob"></a>JSON 範例︰將資料從 Cassandra 複製到 Azure Blob
此範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 它示範如何將資料從內部部署的 Cassandra 資料庫複製到「Azure Blob 儲存體」。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

> [!IMPORTANT]
> 此範例提供 JSON 程式碼片段。 其中並不包含建立 Data Factory 的逐步指示。 如需逐步指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。

此範例具有下列 Data Factory 實體：

* [OnPremisesCassandra](#linked-service-properties)類型的連結服務。
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [CassandraTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [CassandraSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

**Cassandra 已連結的服務：**

此範例使用 **Cassandra** 連結服務。 如需此連結服務所支援的屬性，請參閱 [Cassandra 連結服務](#linked-service-properties) 一節。  

```json
{
    "name": "CassandraLinkedService",
    "properties":
    {
        "type": "OnPremisesCassandra",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "host": "mycassandraserver",
            "port": 9042,
            "username": "user",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```

**Azure 儲存體連結服務：**

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

**Cassandra 輸入資料集：**

```json
{
    "name": "CassandraInput",
    "properties": {
        "linkedServiceName": "CassandraLinkedService",
        "type": "CassandraTable",
        "typeProperties": {
            "tableName": "mytable",
            "keySpace": "mykeyspace"
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

**Azure Blob 輸出資料集：**

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
            "folderPath": "adfgetstarted/fromcassandra"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**具有 Cassandra 來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **CassandraSource**，而 **sink** 類型設為 **BlobSink**。

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
            "name": "CassandraToAzureBlob",
            "description": "Copy from Cassandra to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "CassandraInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "CassandraSource",
                    "query": "select id, firstname, lastname from mykeyspace.mytable"

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

### <a name="type-mapping-for-cassandra"></a>Cassandra 的類型對應
| Cassandra 類型 | 以 .Net 為基礎的類型 |
| --- | --- |
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
>

## <a name="work-with-collections-using-virtual-table"></a>使用虛擬資料表處理集合
Azure Data Factory 會使用內建的 ODBC 驅動程式來連線到 Cassandra 資料庫並從中複製資料。 針對包含對應、集和清單在內的集合類型，此驅動程式會將資料重新標準化為對應的虛擬資料表。 具體來說，如果資料表包含任何集合資料行，則此驅動程式會產生下列虛擬資料表︰

* **基底資料表**，其中包含與實際資料表相同的資料 (集合資料行除外)。 基底資料表使用與它所代表的實際資料表相同的名稱。
* 每個集合資料行的 **虛擬資料表** ，以展開巢狀資料。 代表集合的虛擬資料表會使用實際資料表名稱、分隔字元 “vt”和資料行名稱來命名。

虛擬資料表會參考實際資料表中的資料，讓驅動程式得以存取反正規化的資料。 如需詳細資訊，請參閱＜範例＞一節。 您可以藉由查詢和聯結虛擬資料表來存取 Cassandra 集合的內容。

您可以使用[複製精靈](data-factory-data-movement-activities.md#create-a-pipeline-with-copy-activity)，以直觀方式檢視 Cassandra 資料庫中的資料表清單 (包括虛擬資料表)，並預覽其中的資料。 您也可以在複製精靈中建構查詢，並進行驗證以查看結果。

### <a name="example"></a>範例
例如，以下的「ExampleTable」就是 Cassandra 資料庫資料表，其中包含名為「pk_int」的整數主索引鍵資料行、名為「值」的文字資料行、「清單」資料行、「對應」資料行和「集」資料行 (名為「StringSet」)。

| pk_int | 值 | 列出 | 對應 | StringSet |
| --- | --- | --- | --- | --- |
| 1 |"sample value 1" |["1", "2", "3"] |{"S1": "a", "S2": "b"} |{"A", "B", "C"} |
| 3 |"sample value 3" |["100", "101", "102", "105"] |{"S1": "t"} |{"A", "E"} |

驅動程式會產生多個代表這個單一資料表的虛擬資料表。 虛擬資料表中的外部索引鍵資料行會參考實際資料表中的主索引鍵資料行，並指出虛擬資料表資料列所對應的實際資料表資料列。

第一個虛擬資料表是名為「ExampleTable」的基底資料表，如下表所示。 基底資料表包含與原始資料庫資料表相同的資料，但集合除外，集合會被此資料表省略，而在其他虛擬資料表中展開。

| pk_int | 值 |
| --- | --- |
| 1 |"sample value 1" |
| 3 |"sample value 3" |

下表顯示會重新標準化「清單」、「對應」與「StringSet」資料行中資料的虛擬資料表。 名稱結尾為「_index」或「_key」的資料行代表資料在原始清單或對應內的位置。 名稱結尾為「_value」的資料行包含從集合展開的資料。

#### <a name="table-exampletablevtlist"></a>資料表「ExampleTable_vt_List」：
| pk_int | List_index | List_value |
| --- | --- | --- |
| 1 |0 |1 |
| 1 |1 |2 |
| 1 |2 |3 |
| 3 |0 |100 |
| 3 |1 |101 |
| 3 |2 |102 |
| 3 |3 |103 |

#### <a name="table-exampletablevtmap"></a>資料表「ExampleTable_vt_Map」：
| pk_int | Map_key | Map_value |
| --- | --- | --- |
| 1 |S1 |具有使用  |
| 1 |S2 |b |
| 3 |S1 |t |

#### <a name="table-exampletablevtstringset"></a>資料表「ExampleTable_vt_StringSet」：
| pk_int | StringSet_value |
| --- | --- |
| 1 |具有使用  |
| 1 |b |
| 1 |C |
| 3 |具有使用  |
| 3 |E |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
