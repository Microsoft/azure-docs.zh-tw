---
title: 使用 Azure Data Factory 從 MySQL 移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 MySQL Database 移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 452f4fce-9eb5-40a0-92f8-1e98691bea4c
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/06/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 34de57188dffb7375889ed9ed89a759238b035ac
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046879"
---
# <a name="move-data-from-mysql-using-azure-data-factory"></a>使用 Azure Data Factory 從 MySQL 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-onprem-mysql-connector.md)
> * [第 2 版 (目前的版本)](../connector-mysql.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 MySQL 連接器](../connector-mysql.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署的 MySQL 資料庫移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從內部部署的 MySQL 資料存放區複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從 MySQL 資料存放區移到其他資料存放區，而不支援將資料從其他資料存放區移到 MySQL 資料存放區。 

## <a name="prerequisites"></a>先決條件
Data Factory 服務支援使用資料管理閘道器連接至內部部署 MySQL 來源。 請參閱 [在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文來了解資料管理閘道和設定閘道的逐步指示。

即使 MySQL 資料庫裝載於 Azure IaaS 虛擬機器 (VM) 中，也必須要有閘道。 您可以將閘道安裝在與資料存放區相同或相異的 VM 上，只要閘道可以連線到資料庫即可。

> [!NOTE]
> 如需連接/閘道器相關問題的疑難排解秘訣，請參閱 [針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) 。

## <a name="supported-versions-and-installation"></a>支援的版本和安裝
若要讓資料管理閘道連線至 MySQL 資料庫，您必須在與資料管理閘道相同的系統上，安裝 [Microsoft Windows 的 MySQL Connector/Net](https://dev.mysql.com/downloads/connector/net/) \(英文\) (介於 6.6.5 和 6.10.7 之間的版本)。 這個 32 位元驅動程式可與 64 位元資料管理閘道相容。 支援 MySQL 版本 5.1 和以上版本。

> [!TIP]
> 如果您遇到錯誤「驗證失敗，因為遠端合作對象已關閉傳輸串流」，請考慮將 MySQL 連接器/Net 更新為更高版本。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從內部部署的 Cassandra 資料存放區移動資料。 

- 若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。 
- 您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從內部部署 MySQL 資料存放區複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 MySQL 複製到 Azure Blob](#json-example-copy-data-from-mysql-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 MySQL 資料存放區特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 MySQL 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 類型 |類型屬性必須設為： **OnPremisesMySql** |yes |
| 伺服器 |MySQL 伺服器的名稱。 |yes |
| 資料庫 |MySQL 資料庫的名稱。 |yes |
| 結構描述 |在資料庫中的結構描述名稱。 |否 |
| authenticationType |用來連接到 MySQL 資料庫的驗證類型。 可能的值為：`Basic`。 |yes |
| username |指定要連線到 MySQL 資料庫的使用者名稱。 |yes |
| password |指定您所指定使用者帳戶的密碼。 |yes |
| gatewayName |Data Factory 服務應該用來連接到內部部署 MySQL 資料庫的閘道器名稱。 |yes |

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **RelationalTable** 資料集類型的 typeProperties 區段 (包含 MySQL 資料集) 具有下列屬性

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |MySQL 資料庫執行個體中連結服務所參照的資料表名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

而活動的 **typeProperties** 區段中，可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

當複製活動中的來源類型為 **RelationalSource** (包括 MySQL) 時，typeProperties 區段中會有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：select * from MyTable。 |否 (如果已指定 **dataset** 的 **tableName**) |


## <a name="json-example-copy-data-from-mysql-to-azure-blob"></a>JSON 範例：將資料從 MySQL 複製到 Azure Blob
此範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 它示範如何將資料從內部部署的 MySQL 資料庫複製到「Azure Blob 儲存體」。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

> [!IMPORTANT]
> 此範例提供 JSON 程式碼片段。 其中並不包含建立 Data Factory 的逐步指示。 如需逐步指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。

此範例具有下列 Data Factory 實體：

1. [OnPremisesMySql](data-factory-onprem-mysql-connector.md#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [RelationalTable](data-factory-onprem-mysql-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [RelationalSource](data-factory-onprem-mysql-connector.md#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每個小時將資料從 MySQL 資料庫中的查詢結果複製到 Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

第一步是設定資料管理閘道。 如需相關指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 。

**MySQL 已連結的服務：**

```JSON
    {
      "name": "OnPremMySqlLinkedService",
      "properties": {
        "type": "OnPremisesMySql",
        "typeProperties": {
          "server": "<server name>",
          "database": "<database name>",
          "schema": "<schema name>",
          "authenticationType": "<authentication type>",
          "userName": "<user name>",
          "password": "<password>",
          "gatewayName": "<gateway>"
        }
      }
    }
```

**Azure 儲存體連結服務：**

```JSON
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

**MySQL 輸入資料集：**

此範例假設您已在 MySQL 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 可讓 Data Factory 服務知道資料表是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

```JSON
    {
        "name": "MySqlDataSet",
        "properties": {
            "published": false,
            "type": "RelationalTable",
            "linkedServiceName": "OnPremMySqlLinkedService",
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

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
    {
        "name": "AzureBlobMySqlDataSet",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "mycontainer/mysql/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**具有複製活動的管線：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。 針對 **query** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```JSON
    {
        "name": "CopyMySqlToBlob",
        "properties": {
            "description": "pipeline for copy activity",
            "activities": [
                {
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "RelationalSource",
                            "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                        },
                        "sink": {
                            "type": "BlobSink",
                            "writeBatchSize": 0,
                            "writeBatchTimeout": "00:00:00"
                        }
                    },
                    "inputs": [
                        {
                            "name": "MySqlDataSet"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobMySqlDataSet"
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
                    "name": "MySqlToBlob"
                }
            ],
            "start": "2014-06-01T18:00:00Z",
            "end": "2014-06-01T19:00:00Z"
        }
    }
```


### <a name="type-mapping-for-mysql"></a>MySQL 的類型對應
如 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會藉由下列含有兩個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移到 MySQL 時，會使用下列從 MySQL 類型到 .NET 類型的對應。

| MySQL 資料庫類型 | .NET Framework 類型 |
| --- | --- |
| 不帶正負號的 bigint |十進位 |
| bigint |Int64 |
| bit |十進位 |
| blob |Byte[] |
| 布林 |BOOLEAN |
| char |字串 |
| 日期 |DateTime |
| Datetime |DateTime |
| decimal |十進位 |
| 雙精度 |兩倍 |
| 兩倍 |兩倍 |
| 列舉 |字串 |
| float |單一 |
| 不帶正負號的 int |Int64 |
| int |Int32 |
| 不帶正負號的整數 |Int64 |
| integer |Int32 |
| 長 varbinary |Byte[] |
| 長 varchar |字串 |
| longblob |Byte[] |
| longtext |字串 |
| mediumblob |Byte[] |
| 不帶正負號的 mediumint |Int64 |
| mediumint |Int32 |
| mediumtext |字串 |
| numeric |十進位 |
| real |兩倍 |
| set |字串 |
| 不帶正負號的 smallint |Int32 |
| smallint |Int16 |
| text |字串 |
| 分析 |時間範圍 |
| timestamp |DateTime |
| tinyblob |Byte[] |
| 不帶正負號的 tinyint |Int16 |
| tinyint |Int16 |
| tinytext |字串 |
| varchar |字串 |
| 年 |int |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
