---
title: 使用 Azure Data Factory 從 Web 資料表移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory，來移動網頁上資料表的資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: f54a26a4-baa4-4255-9791-5a8f935898e2
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 05833599059c2724529f9fd23edcd86934793835
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048851"
---
# <a name="move-data-from-a-web-table-source-using-azure-data-factory"></a>使用 Azure Data Factory 來移動 Web 資料表的資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-web-table-connector.md)
> * [第 2 版 (目前的版本)](../connector-web-table.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Web 表格連接器](../connector-web-table.md)。

本文概述如何使用 Azure Data Factory 中的「複製活動」，將資料從網頁中的資料表移到支援的接收資料存放區。 本文是根據 [資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援作為來源/接收的資料存放區清單。

Data factory 目前只支援把 Web 資料表的資料移動到其他資料存放區，但不支援把其他資料存放區的資料移動到 Web 資料表。

> [!IMPORTANT]
> 此 Web 連接器目前只支援從 HTML 網頁擷取資料表內容。 若要從 HTTP/s 端點擷取資料，請改用 [HTTP 連接器](data-factory-http-connector.md)。

## <a name="prerequisites"></a>先決條件

若要使用此 Web 資料表連接器，您需要設定自我裝載 Integration Runtime (也稱為 Data Management Gateway)，並在接收連結服務中設定 `gatewayName` 屬性。 例如，若要從 Web 資料表複製至 Azure Blob 儲存體，請設定 Azure 儲存體連結的服務，如下所示：

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>",
      "gatewayName": "<gateway name>"
    }
  }
}
```

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從內部部署的 Cassandra 資料存放區移動資料。 

- 若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。 
- 您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從 Web 資料表複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 Web 資料表複製到 Azure Blob](#json-example-copy-data-from-web-table-to-azure-blob) 一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Web 資料表特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 Web 連結服務專屬 JSON 元素的說明。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **Web** |yes |
| Url |Web 來源的 URL |yes |
| authenticationType |匿名。 |yes |

### <a name="using-anonymous-authentication"></a>使用匿名驗證

```json
{
    "name": "web",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **WebTable** 類型資料集的 typeProperties 區段有下列屬性

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |資料集的類型。 必須設定為 **WebTable** |yes |
| path |包含資料表之資源的相對 URL。 |否。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 |
| index |資源中資料表的索引。 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。 |yes |

**範例：**

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

而活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

當複製活動中的來源類型為 **WebSource**，目前並未支援任何其他屬性。


## <a name="json-example-copy-data-from-web-table-to-azure-blob"></a>JSON 範例：將資料從 Web 資料表複製到 Azure Blob
下列範例顯示︰

1. [Web](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [WebTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [WebSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

範例會每隔一小時就把 Web 資料表的資料複製到 Azure blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

下列範例示範如何把 Web 資料表的資料複製到 Azure Blob。 不過，您可以使用 Azure Data Factory 中的複製活動，把資料直接複製到 [資料移動活動](data-factory-data-movement-activities.md) 一文中所述的任何接收器。

**Web 連結服務** 這個範例會使用有匿名驗證的 Web 連結服務。 請參閱 [Web 連結服務](#linked-service-properties) 一節，來了解您可以使用的不同驗證類型。

```json
{
    "name": "WebLinkedService",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
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
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>",
      "gatewayName": "<gateway name>"
    }
  }
}
```

**WebTable 輸入資料集** 將 **external** 設定為 **true**，等於是通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

> [!NOTE]
> 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。  
>
>

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```


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
            "folderPath": "adfgetstarted/Movies"
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

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **WebSource**，而 **sink** 類型設為 **BlobSink**。

如需 WebSource 支援的屬性清單，請參閱 [WebSource 類別屬性](#copy-activity-type-properties) 。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "WebTableToAzureBlob",
        "description": "Copy from a Web table to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "WebTableInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "WebSource"
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

## <a name="get-index-of-a-table-in-an-html-page"></a>取得 HTML 網頁中資料表的索引
1. 啟動 **Excel 2016**，然後切換到 [資料] 索引標籤。  
2. 按一下工具列上的 [開新查詢]、指向 [從其他來源]，然後按一下 [從 Web]。

    ![Power Query 功能表](./media/data-factory-web-table-connector/PowerQuery-Menu.png)
3. 在 [從 Web] 對話方塊中，輸入您要在連結服務 JSON 中使用的 **URL** (例如：https://en.wikipedia.org/wiki/)，以及您為資料集指定的路徑 (例如：AFI%27s_100_Years...100_Movies)，然後按一下 [確定]。

    ![[從 Web] 對話方塊](./media/data-factory-web-table-connector/FromWeb-DialogBox.png)

    此範例使用的 URL：https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. 當您看到 [存取 Web 內容] 對話方塊時，選取右側的 **URL**、**驗證方式**，然後按一下 [連線]。

   ![[存取 Web 內容] 對話方塊](./media/data-factory-web-table-connector/AccessWebContentDialog.png)
5. 按一下樹狀檢視中的某個**資料表**項目來查看資料表內容，然後按一下底部的 [編輯] 按鈕。  

   ![[導覽器] 對話方塊](./media/data-factory-web-table-connector/Navigator-DialogBox.png)
6. 在 [查詢編輯器] 視窗中，按一下工具列上的 [進階編輯器] 按鈕。

    ![[進階編輯器] 按鈕](./media/data-factory-web-table-connector/QueryEditor-AdvancedEditorButton.png)
7. 在 [進階編輯器] 對話方塊中，「Source」旁的數字就是索引。

    ![進階編輯器及索引](./media/data-factory-web-table-connector/AdvancedEditor-Index.png)

如果您使用的是 Excel 2013，請使用 [Microsoft Power Query for Excel](https://www.microsoft.com/download/details.aspx?id=39379) 來取得索引。 如需詳細資訊，請參閱 [連線至網頁](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) 一文。 如果您使用 [Microsoft Power BI for Desktop](https://powerbi.microsoft.com/desktop/)，步驟就很類似。

> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
