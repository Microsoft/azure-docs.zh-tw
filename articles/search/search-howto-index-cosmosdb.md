---
title: 為 Azure 搜尋服務的 Azure Cosmos DB 資料來源編製索引 | Microsoft Docs
description: 本文說明如何以 Azure Cosmos DB 資料來源建立 Azure 搜尋服務索引子。
author: chaosrealm
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: eugenesh
robot: noindex
ms.openlocfilehash: 8206c076f9e89753adb16854a7d981c0f80c4a3a
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34640331"
---
# <a name="connecting-cosmos-db-with-azure-search-using-indexers"></a>使用索引子連接 Cosmos DB 與 Azure 搜尋服務

在本文中，了解如何：

> [!div class="checklist"]
> * 設定使用 Azure Cosmos DB 集合作為資料來源的 [Azure 搜尋服務索引子](search-indexer-overview.md)。
> * 使用與 JSON 相容的資料類型來建立搜尋索引。
> * 設定隨需和週期性索引的索引子。
> * 以基礎資料中的變更為基礎，利用累加方式重新整理索引。

> [!NOTE]
> Azure Cosmos DB 是新一代的 DocumentDB。 雖然產品名稱變更，但 Azure 搜尋服務索引子中的 `documentdb` 語法仍存在，可在 Azure 搜尋服務和入口網站頁面中回溯相容性。 在設定索引子時，務必如本文中的指示指定 `documentdb` 語法。

在下列影片中，Azure Cosmos DB 程式總監 Andrew Liu 會示範如何將 Azure 搜尋服務索引加入 Azure Cosmos DB 容器。

>[!VIDEO https://www.youtube.com/embed/OyoYu1Wzk4w]

<a name="supportedAPIs"></a>
## <a name="supported-api-types"></a>支援的 API 類型

雖然 Azure Cosmos DB 支援各種不同的資料模型和 API，但是 Azure 搜尋服務索引子的生產環境支援只會延伸至 SQL API。 MongoDB API 支援目前處於公開預覽狀態。  

即將推出其他 API 的支援。 若要協助我們設定要先支援哪些項目的優先順序，請在 User Voice 網站上投票：

* [資料表 API 資料來源支援](https://feedback.azure.com/forums/263029-azure-search/suggestions/32759746-azure-search-should-be-able-to-index-cosmos-db-tab)
* [圖形 API 資料來源支援](https://feedback.azure.com/forums/263029-azure-search/suggestions/13285011-add-graph-databases-to-your-data-sources-eg-neo4)
* [Apache Cassandra API 資料來源支援](https://feedback.azure.com/forums/263029-azure-search/suggestions/32857525-indexer-crawler-for-apache-cassandra-api-in-azu)

## <a name="prerequisites"></a>先決條件

除了 Cosmos DB 帳戶以外，您必須擁有 [Azure 搜尋服務](search-create-service-portal.md)。 

<a name="Concepts"></a>
## <a name="azure-search-indexer-concepts"></a>Azure 搜尋服務索引子概念

**資料來源**指定要編製索引的資料、認證，以及可識別資料是否變更 (例如修改或刪除集合內的文件) 的原則。 資料來源會被定義為獨立的資源，因此可供多個索引子使用。

「 **索引子** 」說明資料如何從資料來源流動到目標搜尋索引。 索引子可用來：

* 執行資料的一次性複製以填入索引。
* 依照排程將索引與資料來源中的變更同步。
* 視需要叫用索引的隨選更新。

若要設定 Azure Cosmos DB 索引子，您必須建立索引、資料來源，最後再建立索引子。 您可以使用[入口網站](search-import-data-portal.md)、[.NET SDK](/dotnet/api/microsoft.azure.search) 或 [REST API](/rest/api/searchservice/) 來建立這些物件。 

此文章說明如何使用 REST API。 如果您選擇使用入口網站，[匯入資料精靈](search-import-data-portal.md)會引導您建立所有這些資源，包含索引。

> [!TIP]
> 您可以從 Azure Cosmos DB 儀表板啟動**匯入資料**精靈，以簡化該資料來源的索引建立作業。 在左側導覽中，移至 [集合] > [新增 Azure 搜尋服務] 以便開始使用。

> [!NOTE] 
> 目前，您無法使用 Azure 入口網站或 .NET SDK 建立或編輯 **MongoDB** 資料來源。 不過，您**可以**在入口網站中監視 MongoDB 索引子的執行歷程記錄。  

<a name="CreateDataSource"></a>
## <a name="step-1-create-a-data-source"></a>步驟 1：建立資料來源
若要建立資料來源，執行：

    POST https://[service name].search.windows.net/datasources?api-version=2017-11-11
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mydocdbdatasource",
        "type": "documentdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCollection", "query": null },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        }
    }

要求的主體包含資料來源定義，其中應包含下列欄位：

* **名稱**：選擇任何名稱，以代表您的資料庫。
* **type**：必須是 `documentdb`。
* **認證**：
  
  * **connectionString**：必要。 請以下列格式指定 Azure Cosmos DB 資料庫的連線資訊：`AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>` 針對 MongoDB 集合，將 **ApiKind=MongoDb** 新增至連接字串：`AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=MongoDb` 
* **容器**：
  
  * **名稱**：必要。 指定要編製索引的資料收集識別碼。
  * **查詢**：選擇性。 您可以指定查詢將任意 JSON 文件簡維成 Azure 搜尋服務可以編製索引的一般結構描述。 針對 MongoDB 集合，不支援查詢。 
* **dataChangeDetectionPolicy**：建議使用。 請參閱[索引變更的文件](#DataChangeDetectionPolicy)小節。
* **dataDeletionDetectionPolicy**：選擇性。 請參閱[索引刪除的文件](#DataDeletionDetectionPolicy)小節。

### <a name="using-queries-to-shape-indexed-data"></a>使用查詢來形塑索引的資料
您可以指定 SQL 查詢來壓平合併巢狀屬性或陣列、投影 JSON 屬性，以及篩選要編製索引的資料。 

> [!WARNING]
> **MongoDB** 集合不支援自訂查詢：`container.query` 參數必須設定為 Null 或省略。 如果您需要使用自訂查詢，請在 [User Voice](https://feedback.azure.com/forums/263029-azure-search) 告訴我們。

範例文件︰

    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "documentdb", "search"]
    }

篩選查詢：

    SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark ORDER BY c._ts

壓平合併查詢︰

    SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
    
    
投影查詢：

    SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts


陣列壓平合併查詢︰

    SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark ORDER BY c._ts

<a name="CreateIndex"></a>
## <a name="step-2-create-an-index"></a>步驟 2：建立索引
建立目標 Azure 搜尋服務索引 (如果您尚未建立)。 您可以使用 [Azure 入口網站 UI](search-create-index-portal.md)、[建立索引 REST API](/rest/api/searchservice/create-index) 或[索引類別](/dotnet/api/microsoft.azure.search.models.index)建立索引。

下列範例會建立包含識別碼和描述欄位的索引：

    POST https://[service name].search.windows.net/indexes?api-version=2017-11-11
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }

請確定目標索引的結構描述會與來源 JSON 文件的結構描述或自訂查詢投射的輸出相容。

> [!NOTE]
> 針對分割的集合，預設文件索引鍵是 Azure Cosmos DB 的 `_rid` 屬性，Azure 搜尋服務會自動將該屬性重新命名為 `rid`，因為欄位名稱不能以底線字元開頭。 此外，Azure Cosmos DB `_rid` 值包含 Azure 搜尋服務索引鍵中無效的字元。 因此，`_rid` 值採用 Base64 編碼。
> 
> 針對 MongoDB 集合，Azure 搜尋服務會自動將 `_id` 屬性重新命名為 `doc_id`。  

### <a name="mapping-between-json-data-types-and-azure-search-data-types"></a>JSON 資料類型與 Azure 搜尋服務資料類型之間的對應
| JSON 資料類型 | 相容的目標索引欄位類型 |
| --- | --- |
| Bool |Edm.Boolean、Edm.String |
| 看起來像是整數的數字 |Edm.Int32、Edm.Int64、Edm.String |
| 看起來像是浮點的數字 |Edm.Double、Edm.String |
| 字串 |Edm.String |
| 基本類型的陣列，例如 ["a", "b", "c"] |Collection(Edm.String) |
| 看起來像是日期的字串 |Edm.DateTimeOffset、Edm.String |
| GeoJSON 物件，例如 { "type": "Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| 其他 JSON 物件 |N/A |

<a name="CreateIndexer"></a>

## <a name="step-3-create-an-indexer"></a>步驟 3：建立索引子

建立索引和資料來源之後，您就可以開始建立索引子︰

    POST https://[service name].search.windows.net/indexers?api-version=2017-11-11
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mydocdbindexer",
      "dataSourceName" : "mydocdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }

這個索引子每隔兩小時就會執行一次 (已將排程間隔設為 "PT2H")。 若每隔 30 分鐘就要執行索引子，可將間隔設為 "PT30M"。 支援的最短間隔為 5 分鐘。 排程為選擇性 - 如果省略，索引子只會在建立時執行一次。 不過，您隨時都可依需求執行索引子。   

如需建立索引子 API 的詳細資訊，請參閱 [建立索引子](https://docs.microsoft.com/rest/api/searchservice/create-indexer)。

<a id="RunIndexer"></a>
### <a name="running-indexer-on-demand"></a>視需要執行索引子
除了依照排程定期執行以外，您也可以視需要叫用索引子：

    POST https://[service name].search.windows.net/indexers/[indexer name]/run?api-version=2017-11-11
    api-key: [Search service admin key]

> [!NOTE]
> 當執行 API 成功傳回時，索引子叫用已經排程，但實際處理不會同步發生。 

您可以在入口網站中監視索引子狀態，或使用取得索引子狀態 API進行監視，我們接下將說明此 API。 

<a name="GetIndexerStatus"></a>
### <a name="getting-indexer-status"></a>取得索引子狀態
您可以擷取索引子的狀態和執行歷程記錄：

    GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2017-11-11
    api-key: [Search service admin key]

回應包含整體索引子的狀態、最後 (或進行中) 的索引子叫用，以及最新的索引子叫用歷程記錄。

    {
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
         },
        "executionHistory":[ {
            "status":"success",
             "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        }]
    }

執行歷程記錄包含多達 50 個最近完成的執行，以倒序的方式進行儲存 (因此最新的執行會排在回應中的第一位)。

<a name="DataChangeDetectionPolicy"></a>
## <a name="indexing-changed-documents"></a>索引已變更的文件
資料變更偵測原則是用來有效識別已變更的資料項目。 目前，唯一支援的原則是使用 Azure Cosmos DB 所提供之 `_ts` (時間戳記) 屬性的 `High Water Mark` 原則，指定方式如下：

    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }

為確保索引子效能良好，強烈建議使用此原則。 

如果您使用自訂查詢，請確定查詢有投射 `_ts` 屬性。

<a name="IncrementalProgress"></a>
### <a name="incremental-progress-and-custom-queries"></a>累加進度與自訂查詢
建立索引期間，採累加進度能確保當索引子執行因暫時性失敗或執行時間限制而中斷時，索引子仍能夠在下次執行時從中斷處繼續建立索引，而不需要重新建立整個集合的索引。 這在建立大型集合的索引時尤其重要。 

若要在使用自訂查詢時啟用累加進度，請確保您的查詢是依 `_ts` 欄排序查詢結果。 如此會啟用定期檢查點設置，出現失敗時，Azure 搜尋服務會以此提供累加進度來應對。   

在某些情況下，即使您的查詢含有 `ORDER BY [collection alias]._ts` 子句，Azure 搜尋服務仍可能無法推斷此查詢是否依 `_ts` 排序。 您可以使用 `assumeOrderByHighWaterMarkColumn` 這個設定屬性讓 Azure 搜尋服務知道查詢結果已經過排序。 若要指定這項提示，請依下列指示更新您的索引子： 

    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "assumeOrderByHighWaterMarkColumn" : true } }
    } 

<a name="DataDeletionDetectionPolicy"></a>
## <a name="indexing-deleted-documents"></a>索引已刪除的文件
從集合中刪除資料列時，通常也會想刪除搜尋索引內的那些資料列。 資料刪除偵測原則可用來有效識別刪除的資料項目。 目前，唯一支援的原則是「 `Soft Delete` 」原則 (刪除會標示為某種形式的旗標)，指定方式如下：

    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }

如果您使用自訂查詢，請確定查詢有投射到 `softDeleteColumnName` 參考的屬性。

下列範例會建立包含虛刪除原則的資料來源：

    POST https://[service name].search.windows.net/datasources?api-version=2017-11-11
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mydocdbdatasource",
        "type": "documentdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myDocDbEndpoint.documents.azure.com;AccountKey=myDocDbAuthKey;Database=myDocDbDatabaseId"
        },
        "container": { "name": "myDocDbCollectionId" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }

## <a name="NextSteps"></a>後續步驟
恭喜！ 您已了解如何使用索引子來整合 Azure Cosmos DB 與 Azure 搜尋服務。

* 若要深入了解 Azure Cosmos DB，請參閱 [Azure Cosmos DB 服務頁面](https://azure.microsoft.com/services/cosmos-db/)。
* 若要深入了解 Azure 搜尋服務，請參閱 [[搜尋服務] 頁面](https://azure.microsoft.com/services/search/)。
