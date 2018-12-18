---
title: 使用 Data Factory 從 MongoDB 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 MongoDB 資料庫移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 10ca7d9a-7715-4446-bf59-2d2876584550
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/13/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 7c6751a0432d66aee0ff3056b212dc1b348e333f
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045821"
---
# <a name="move-data-from-mongodb-using-azure-data-factory"></a>使用 Azure Data Factory 從 MongoDB 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-on-premises-mongodb-connector.md)
> * [第 2 版 (目前的版本)](../connector-mongodb.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 MongoDB 連接器](../connector-mongodb.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署的 MongoDB 資料庫移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從內部部署的 MongoDB 資料存放區複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從 MongoDB 資料存放區移到其他資料存放區，而不支援將資料從其他資料存放區移到 MongoDB 資料存放區。 

## <a name="prerequisites"></a>先決條件
如果是能夠連接到您內部部署 MongoDB 資料庫的 Azure Data Factory 服務，您就必須安裝下列元件：

- 支援的 MongoDB 版本為︰2.4、2.6、3.0、3.2、3.4、3.6。
- 位於裝載資料庫的同一部電腦上或個別電腦上的資料管理閘道，可避免與資料庫競用資源。 資料管理閘道是一套透過安全且可管理的方式，將內部部署資料來源連結至雲端服務的軟體。 如需資料管理閘道的詳細資料，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 一文。 如需有關為閘道設定資料管線來移動資料的逐步指示，請參閱[將資料從內部部署移到雲端](data-factory-move-data-between-onprem-and-cloud.md)。

    當您安裝閘道時，它會自動安裝用來連線至 MongoDB 的 Microsoft MongoDB ODBC 驅動程式。

    > [!NOTE]
    > 即使 MongoDB 是裝載在 Azure IaaS VM 中，您還是需要使用閘道與其連接。 如果您正嘗試連接到裝載於雲端中的 MongoDB 執行個體，您也可以在 IaaS VM 中安裝閘道器執行個體。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從內部部署的 MongoDB 資料存放區移動資料。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從內部部署 MongoDB 資料存放區複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 MongoDB 複製到 Azure Blob](#json-example-copy-data-from-mongodb-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 MongoDB 來源特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 **OnPremisesMongoDB** 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **OnPremisesMongoDb** |yes |
| 伺服器 |MongoDB 伺服器的 IP 位址或主機名稱。 |yes |
| 連接埠 |MongoDB 伺服器用來接聽用戶端連線的 TCP 連接埠。 |選用，預設值︰27017 |
| authenticationType |基本或匿名。 |yes |
| username |用來存取 MongoDB 的使用者帳戶。 |是 (如果使用基本驗證)。 |
| password |使用者的密碼。 |是 (如果使用基本驗證)。 |
| authSource |您想要用來檢查驗證所用之認證的 MongoDB 資料庫名稱。 |選用 (如果使用基本驗證)。 預設值︰使用以 databaseName 屬性指定的系統管理員帳戶和資料庫。 |
| databaseName |您想要存取之 MongoDB 資料庫的名稱。 |yes |
| gatewayName |存取資料存放區之閘道的名稱。 |yes |
| encryptedCredential |由閘道加密的認證。 |選用 |

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **MongoDbCollection** 類型資料集的 typeProperties 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| collectionName |MongoDB 資料庫中集合的名稱。 |是 |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 **typeProperties** 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

如果來源類型為 **MongoDbSource** ，則 typeProperties 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢字串。 例如：select * from MyTable。 |否 (如果已指定 **dataset** 的 **collectionName**) |



## <a name="json-example-copy-data-from-mongodb-to-azure-blob"></a>JSON 範例：將資料從 MongoDB 複製到 Azure Blob
此範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 它示範如何將資料從內部部署的 MongoDB 複製到「Azure Blob 儲存體」。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

此範例具有下列 Data Factory 實體：

1. [OnPremisesMongoDb](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [MongoDbCollection](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [MongoDbSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每個小時將資料從 MongoDB 資料庫中的查詢結果複製到 Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

第一個步驟是根據 [資料管理閘道](data-factory-data-management-gateway.md) 一文中的指示設定資料管理閘道。

**MongoDB 已連結的服務：**

```json
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties":
    {
        "type": "OnPremisesMongoDb",
        "typeProperties":
        {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",  
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
           "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<mygateway>"
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

**MongoDB 輸入資料集：** 設定 “external”: ”true” 可告知 Data Factory 服務該資料表是 Data Factory 外部的資料表，而不是由 Data Factory 中的活動所產生。

```json
{
     "name":  "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"    
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/frommongodb/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**具有 MongoDB 來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用上述輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設定為 **MongoDbSource**，且 **sink** 類型設定為 **BlobSink**。 針對 **query** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```json
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "MongoDbSource",
                        "query": "$$Text.Format('select * from  MyTable where LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "MongoDbInputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
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
                "name": "MongoDBToAzureBlob"
            }
        ],
        "start": "2016-06-01T18:00:00Z",
        "end": "2016-06-01T19:00:00Z"
    }
}
```


## <a name="schema-by-data-factory"></a>Data factory 的結構描述
Azure Data Factory 服務會使用 MongoDB 集合中最新的 100 份文件，從該集合推斷結構描述。 如果這 100 份文件未包含完整結構描述，則在複製作業期間可能會忽略某些資料行。

## <a name="type-mapping-for-mongodb"></a>MongoDB 的類型對應
如同 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移到 MongoDB 時，下列對應會用於從 MongoDB 類型對應至 .NET 類型。

| MongoDB 類型 | .NET Framework 類型 |
| --- | --- |
| Binary |Byte[] |
| BOOLEAN |BOOLEAN |
| 日期 |Datetime |
| NumberDouble |兩倍 |
| NumberInt |Int32 |
| NumberLong |Int64 |
| ObjectID |字串 |
| 字串 |字串 |
| UUID |Guid |
| Object |以「_」做為巢狀分隔符號來重新標準化為壓平合併的資料行 |

> [!NOTE]
> 若要了解使用虛擬資料表之陣列的支援，請參閱下面的 [使用虛擬資料表之複雜類型的支援](#support-for-complex-types-using-virtual-tables) 一節。

目前不支援下列 MongoDB 資料類型︰DBPointer、JavaScript、Max/Min 索引鍵、規則運算式、符號、時間戳記、未定義

## <a name="support-for-complex-types-using-virtual-tables"></a>使用虛擬資料表之複雜類型的支援
Azure Data Factory 會使用內建的 ODBC 驅動程式來連線到 MongoDB 資料庫並從中複製資料。 對於其類型會跨文件而不同的陣列或物件等複雜類型，驅動程式會將資料重新標準化為對應的虛擬資料表。 具體來說，如果資料表包含這類資料行，則驅動程式會產生下列虛擬資料表︰

* **基底資料表**，其中包含與實際資料表相同的資料 (複雜類型資料行除外)。 基底資料表使用與它所代表的實際資料表相同的名稱。
* 每個複雜類型資料行的 **虛擬資料表** ，以展開巢狀資料。 虛擬資料表會使用實際資料表名稱、分隔字元「_」和陣列或物件的名稱來命名。

虛擬資料表會參考實際資料表中的資料，讓驅動程式得以存取反正規化的資料。 如需詳細資訊，請參閱下方的＜範例＞一節。 您可以藉由查詢和聯結虛擬資料表來存取 MongoDB 陣列的內容。

您可以使用 [複製精靈](data-factory-data-movement-activities.md#create-a-pipeline-with-copy-activity) ，以直觀方式檢視 MongoDB 資料庫中的資料表清單 (包括虛擬資料表)，並預覽其中的資料。 您也可以在複製精靈中建構查詢，並進行驗證以查看結果。

### <a name="example"></a>範例
例如，下面的「ExampleTable」就是 MongoDB 資料表，其具有一個在每個儲存格中都有物件陣列的資料行 (「發票」)，以及一個具有純量類型陣列的資料行 (「評等」)。

| _id | 客戶名稱 | 發票 | 服務等級 | 評等 |
| --- | --- | --- | --- | --- |
| 1111 |ABC |[{invoice_id:”123”, item:”toaster”, price:”456”, discount:”0.2”}, {invoice_id:”124”, item:”oven”, price: ”1235”, discount: ”0.2”}] |Silver |[5,6] |
| 2222 |XYZ |[{invoice_id:”135”, item:”fridge”, price: ”12543”, discount: ”0.0”}] |Gold |[1,2] |

驅動程式會產生多個代表這個單一資料表的虛擬資料表。 第一個虛擬資料表是名為「ExampleTable」的基底資料表，如下所示。 基底資料表包含原始資料表的所有資料，但來自陣列的資料已省略，並且會在虛擬資料表中展開。

| _id | 客戶名稱 | 服務等級 |
| --- | --- | --- |
| 1111 |ABC |Silver |
| 2222 |XYZ |Gold |

下表顯示代表範例中原始陣列的虛擬資料表。 這些資料表包含下列項目：

* 對應到原始陣列資料列 (透過 _id 資料行) 之原始主要索引鍵資料行的往回參考
* 資料在原始陣列之位置的指示
* 陣列內每個元素的展開資料

資料表「ExampleTable_Invoices」：

| _id | ExampleTable_Invoices_dim1_idx | invoice_id | item | price | 折扣 |
| --- | --- | --- | --- | --- | --- |
| 1111 |0 |123 |toaster |456 |0.2 |
| 1111 |1 |124 |oven |1235 |0.2 |
| 2222 |0 |135 |fridge |12543 |0.0 |

資料表「ExampleTable_Ratings」：

| _id | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings |
| --- | --- | --- |
| 1111 |0 |5 |
| 1111 |1 |6 |
| 2222 |0 |1 |
| 2222 |1 |2 |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。

## <a name="next-steps"></a>後續步驟
請參閱 [在內部部署和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文，以取得如何建立資料管線以將資料從內部部署資料存放區移動至 Azure 資料存放區的逐步指示。
