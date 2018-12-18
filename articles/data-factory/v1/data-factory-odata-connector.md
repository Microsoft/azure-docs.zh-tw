---
title: 從 OData 來源移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory，來移動 OData 來源的資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: de28fa56-3204-4546-a4df-21a21de43ed7
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: be2150147d950d708404aff53ce0aa4e0776ac33
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051162"
---
# <a name="move-data-from-a-odata-source-using-azure-data-factory"></a>使用 Azure Data Factory 來移動 OData 來源的資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-odata-connector.md)
> * [第 2 版 (目前的版本)](../connector-odata.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 OData 連接器](../connector-odata.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署的 OData 來源移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從 OData 來源複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從 OData 來源移到其他資料存放區，而不支援將資料從其他資料存放區移到 OData 來源。 

## <a name="supported-versions-and-authentication-types"></a>支援的版本和驗證類型
此 OData 連接器支援 OData 版本 3.0 和 4.0，而您可以從雲端 OData 和內部部署 OData 來源複製資料。 若為後者，您必須安裝資料管理閘道。 如需資料管理閘道的詳細資訊，請參閱 [在內部部署和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。

支援下列驗證類型：

* 若要存取**雲端** OData 摘要，您可以使用匿名、基本 (使用者名稱和密碼) 或 Azure Active Directory 架構的 OAuth 驗證。
* 若要存取**內部部署** OData 摘要，您可以使用匿名、基本 (使用者名稱和密碼) 或 Windows 驗證。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從 OData 來源移動資料。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從 OData 來源複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 OData 來源複製到 Azure Blob](#json-example-copy-data-from-odata-source-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 OData 來源特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>已連結的服務屬性
下表提供 OData 連結服務專屬 JSON 元素的說明。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **OData** |yes |
| url |OData 服務的 URL。 |yes |
| authenticationType |用來連線到 OData 來源的驗證類型。 <br/><br/> 若為雲端 OData，可能的值為 Anonymous、Basic 和 OAuth (請注意，Azure Data Factory 目前僅支援 Azure Active Directory 架構的 OAuth)。 <br/><br/> 若為內部部署 OData，可能的值為 Anonymous、Basic 和 Windows。 |yes |
| username |如果您要使用 Basic 驗證，請指定使用者名稱。 |是 (只在您使用基本驗證時) |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |是 (只在您使用基本驗證時) |
| authorizedCredential |如果您使用 OAuth，按一下 Data Factory 複製精靈或編輯器中的 [授權] 按鈕，然後輸入您的認證，接著將會自動產生這個屬性的值。 |是 (只有在您使用 OAuth 驗證時) |
| gatewayName |Data Factory 服務應該用來連接到內部部署 OData 服務的閘道器名稱。 只在要從內部部署 OData 來源複製資料時才指定。 |否 |

### <a name="using-basic-authentication"></a>使用基本驗證
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Basic",
            "username": "username",
            "password": "password"
        }
    }
}
```

### <a name="using-anonymous-authentication"></a>使用匿名驗證
```json
{
    "name": "ODataLinkedService",
        "properties":
    {
        "type": "OData",
        "typeProperties":
        {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

### <a name="using-windows-authentication-accessing-on-premises-odata-source"></a>使用存取內部部署 OData 來源的 Windows 驗證
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of on-premises OData source e.g. Dynamics CRM>",
            "authenticationType": "Windows",
            "username": "domain\\user",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-oauth-authentication-accessing-cloud-odata-source"></a>使用 OAuth 驗證存取雲端 OData 來源
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of cloud OData source e.g. https://<tenant>.crm.dynamics.com/XRMServices/2011/OrganizationData.svc>",
            "authenticationType": "OAuth",
            "authorizedCredential": "<auto generated by clicking the Authorize button on UI>"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **ODataResource** (包含 OData 資料集) 類型資料集的 typeProperties 區段有下列屬性

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| path |OData 資源的路徑 |否 |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

如果來源類型為 **RelationalSource** (包含 OData)，則 typeProperties 區段可使用下列屬性：

| 屬性 | 說明 | 範例 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |「?$select=Name, Description&$top=5」 |否 |

## <a name="type-mapping-for-odata"></a>OData 的類型對應
如 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會藉由下列含有兩個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換。

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

從 OData 移動資料時，OData 類型至 .NET 類型會使用下列對應。

| OData 資料類型 | .NET 類型 |
| --- | --- |
| Edm.Binary |Byte[] |
| Edm.Boolean |Bool |
| Edm.Byte |Byte[] |
| Edm.DateTime |Datetime |
| Edm.Decimal |十進位 |
| Edm.Double |兩倍 |
| Edm.Single |單一 |
| Edm.Guid |Guid |
| Edm.Int16 |Int16 |
| Edm.Int32 |Int32 |
| Edm.Int64 |Int64 |
| Edm.SByte |Int16 |
| Edm.String |字串 |
| Edm.Time |時間範圍 |
| Edm.DateTimeOffset |DateTimeOffset |

> [!Note]
> 不支援 OData 複雜資料類型，例如，物件。

## <a name="json-example-copy-data-from-odata-source-to-azure-blob"></a>JSON 範例：將資料從 OData 來源複製到 Azure Blob
此範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 這些範例示範如何把 OData 來源的資料複製到 Azure Blob 儲存體。 不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。 範例有下列 Data Factory 實體：

1. [OData](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [ODataResource](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [RelationalSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

範例會每隔一小時依照 OData 來源，把查詢來的資料複製到 Azure Blob 中。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**OData 已連結的服務：** 此範例會使用匿名驗證。 請參閱 [OData 連結服務](#linked-service-properties) 一節，來了解您可以使用的不同驗證類型。

```json
{
    "name": "ODataLinkedService",
        "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
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

**OData 輸入資料集：**

設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```json
{
    "name": "ODataDataset",
    "properties":
    {
        "type": "ODataResource",
        "typeProperties":
        {
                "path": "Products"
        },
        "linkedServiceName": "ODataLinkedService",
        "structure": [],
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3                
        }
    }
}
```

在資料集定義中指定 **path** 的動作是可以省略的。

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
    "name": "AzureBlobODataDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/odata/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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


**具有 OData 來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。 針對 **query** 屬性所指定的 SQL 查詢，會從 OData 來源選取最新的資料。

```json
{
    "name": "CopyODataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "?$select=Name, Description&$top=5",
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "ODataDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobODataDataSet"
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
                "name": "ODataToBlob"
            }
        ],
        "start": "2017-02-01T18:00:00Z",
        "end": "2017-02-03T19:00:00Z"
    }
}
```

在管線定義中指定 **query** 的動作是可以省略的。 Data Factory 服務用來擷取資料的 **URL** 就是：在連結服務中所指定的 URL (必要) + 在資料集所指定的路徑 (可省略) + 管線中的查詢 (可省略)。


### <a name="type-mapping-for-odata"></a>OData 的類型對應
如同 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

當您移動 OData 資料存放區的資料時，OData 資料類型會對應到 .NET 類型。

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行與接收資料集內的資料行對應，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
