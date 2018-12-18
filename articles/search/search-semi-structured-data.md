---
title: 在 Azure 搜尋服務中從 Azure 雲端儲存體搜尋半結構化資料的教學課程 | Microsoft Docs
description: 在本教學課程，了解如何使用 Azure 搜尋服務搜尋半結構化的 Azure blob 資料。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: tutorial
ms.date: 07/12/2018
ms.author: heidist
ms.openlocfilehash: a7b006bd8469ddce1415ab6cb7c52c0171ae11cd
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39005072"
---
# <a name="tutorial-search-semi-structured-data-in-azure-cloud-storage"></a>教學課程：在 Azure 雲端儲存體中搜尋半結構化資料

在這個兩部分的教學課程系列中，您可以了解如何使用 Azure 搜尋服務來搜尋半結構化及未結構化資料。 [第 1 部分](../storage/blobs/storage-unstructured-search.md)不僅引導您完成非結構化資料的搜尋步驟，也說明了本教學課程的重要必要條件 (如建立儲存體帳戶)。 

在第 2 部分，我們將焦點轉移到 JSON 這類儲存在 Azure Blob 中的半結構化資料。 半結構化資料會包含在資料內分隔內容的標籤或標記。 這些資料將不同的內容區分為必須整體編製索引的未結構化資料，以及遵循資料模型 (如關聯式資料庫結構描述)，而且可依據欄位逐一搜耙的正式結構化資料。

在第 2 部分中，您將學習如何：

> [!div class="checklist"]
> * 設定 Azure Blob 容器的 Azure 搜尋服務資料來源
> * 建立及填入 Azure 搜尋服務索引和索引子，以便搜耙容器和擷取可搜尋內容
> * 搜尋剛剛建立的索引

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

* 完成[上一個教學課程](../storage/blobs/storage-unstructured-search.md)，提供在該課程中建立的儲存體帳戶和搜尋服務。

* 安裝 REST 用戶端並了解如何建構 HTTP 要求。 基於本教學課程的目的，我們使用的是 [Postman](https://www.getpostman.com/) \(英文\)。 如果您已經很熟悉某個特定的 REST 用戶端，即可隨意使用不同的用戶端。

> [!NOTE]
> 本教學課程仰賴 JSON 陣列支援，該功能目前是 Azure 搜尋服務中的預覽功能， 因此入口網站並未提供。 基於此因素，我們使用提供這項功能的預覽 REST API 和 REST 用戶端工具來呼叫 API。

## <a name="set-up-postman"></a>設定 Postman

啟動 Postman 及設定 HTTP 要求。 如果您不熟悉此工具，請參閱[使用 Fiddler 或 Postman 探索 Azure 搜尋服務 REST API](search-fiddler.md) 來取得詳細資訊。

本教學課程中每個呼叫的要求方法都是 "POST"。 標頭金鑰為 "Content-type" 和 "api-key"。 標頭金鑰的值分別為 "application/json" 和您的「管理金鑰」(管理金鑰是您搜尋主索引鍵的預留位置)。 主體是您放置呼叫實際內容的地方。 根據使用的用戶端而定，您用以建構查詢的方式可能會有一些變化，但那些都是基本的。

  ![半結構化搜尋](media/search-semi-structured-data/postmanoverview.png)

針對本教學課程中涵蓋的 REST 呼叫，必須提供您的搜尋 api-key。 您可以在搜尋服務內部的 [金鑰] 下方尋找您的 api-key。 這個 api-key 必須位於本教學課程引導您進行之每個 API 呼叫的標頭中 (使用它來取代上一個螢幕擷取畫面中的「管理金鑰」)。 保留此金鑰，因為您需要針對每個呼叫使用它。

  ![半結構化搜尋](media/search-semi-structured-data/keys.png)

## <a name="download-the-sample-data"></a>下載範例資料

目前已為您備妥範例資料集。 **下載 [clinical-trials-json.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials-json.zip)** 並將它解壓縮至它本身的資料夾。

範例中所含的是範例 JSON 檔案，其原本是從 [clinicaltrials.gov](https://clinicaltrials.gov/ct2/results) \(英文\) 取得的文字檔。 我們已將其轉換為 JSON，方便供您使用。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](http://portal.azure.com)。

## <a name="upload-the-sample-data"></a>上傳範例資料

在 Azure 入口網站中，瀏覽回到在[上一個教學課程](../storage/blobs/storage-unstructured-search.md)中建立的儲存體帳戶。 接著開啟 [資料] 容器，然後按一下 [上傳]。

按一下 [進階]、輸入 "clinical-trials-json"，然後上傳您已下載的所有 JSON 檔案。

  ![半結構化搜尋](media/search-semi-structured-data/clinicalupload.png)

上傳完成之後，檔案應該會出現在資料容器內部它們本身的子資料夾中。

## <a name="connect-your-search-service-to-your-container"></a>將您的搜尋服務連接到您的容器

我們使用 Postman 來對您的搜尋服務進行三個 API 呼叫，以建立資料來源、索引及索引子。 資料來源包括指向您儲存體帳戶和您 JSON 資料的指標。 您的搜尋服務會在載入資料時進行連線。

查詢字串必須包含 **api-version=2016-09-01-Preview**，而每個呼叫都應傳回 **201 Created**。 公開推出的 api-version 尚未具備以 jsonArray 方式處理 json 的功能，目前只有預覽的 api-version 能夠做到。

從您的 REST 用戶端執行下列三個 API 呼叫。

### <a name="create-a-datasource"></a>建立資料來源

資料來源會指定要編製索引的資料。

此呼叫的端點為 `https://[service name].search.windows.net/datasources?api-version=2016-09-01-Preview`。 使用您的搜尋服務名稱來取代 `[service name]`。

針對此呼叫，您需要您的儲存體帳戶名稱和儲存體帳戶金鑰。 儲存體帳戶金鑰可以在 Azure入口網站中，於儲存體帳戶的**存取金鑰**內部找到。 下面的影像顯示該位置：

  ![半結構化搜尋](media/search-semi-structured-data/storagekeys.png)

執行呼叫之前，請務必取代呼叫主體中的 `[storage account name]` 和 `[storage account key]`。

```json
{
    "name" : "clinical-trials-json",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=[storage account name];AccountKey=[storage account key];" },
    "container" : { "name" : "data", "query" : "clinical-trials-json" }
}
```

回應看起來應如下所示：

```json
{
    "@odata.context": "https://exampleurl.search.windows.net/$metadata#datasources/$entity",
    "@odata.etag": "\"0x8D505FBC3856C9E\"",
    "name": "clinical-trials-json",
    "description": null,
    "type": "azureblob",
    "subtype": null,
    "credentials": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=[mystorageaccounthere];AccountKey=[[myaccountkeyhere]]];"
    },
    "container": {
        "name": "data",
        "query": "clinical-trials-json"
    },
    "dataChangeDetectionPolicy": null,
    "dataDeletionDetectionPolicy": null
}
```

### <a name="create-an-index"></a>建立索引
    
第二個 API 呼叫會建立索引。 索引會指定所有參數及其屬性。

此呼叫的 URL 是 `https://[service name].search.windows.net/indexes?api-version=2016-09-01-Preview`。 使用您的搜尋服務名稱來取代 `[service name]`。

首先，取代 URL。 接著複製下列程式碼並貼至您的主體，然後執行查詢。

```json
{
  "name": "clinical-trials-json-index",  
  "fields": [
  {"name": "FileName", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
  {"name": "Description", "type": "Edm.String", "searchable": true, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
  {"name": "MinimumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
  {"name": "Title", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
  {"name": "URL", "type": "Edm.String", "searchable": false, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
  {"name": "MyURL", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
  {"name": "Gender", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
  {"name": "MaximumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
  {"name": "Summary", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
  {"name": "NCTID", "type": "Edm.String", "key": true, "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
  {"name": "Phase", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
  {"name": "Date", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
  {"name": "OverallStatus", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
  {"name": "OrgStudyId", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": false},
  {"name": "HealthyVolunteers", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
  {"name": "Keywords", "type": "Collection(Edm.String)", "searchable": true, "retrievable": true, "facetable": true, "filterable": false, "sortable": false},
  {"name": "metadata_storage_last_modified", "type":"Edm.DateTimeOffset", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
  {"name": "metadata_storage_size", "type":"Edm.String", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
  {"name": "metadata_content_type", "type":"Edm.String", "searchable": true, "retrievable": true, "filterable": true, "sortable": false}
  ],
  "suggesters": [
  {
    "name": "sg",
    "searchMode": "analyzingInfixMatching",
    "sourceFields": ["Title"]
  }
  ]
}
```

回應看起來應如下所示：

```json
{
    "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexes/$entity",
    "@odata.etag": "\"0x8D505FC00EDD5FA\"",
    "name": "clinical-trials-json-index",
    "fields": [
        {
            "name": "FileName",
            "type": "Edm.String",
            "searchable": false,
            "filterable": false,
            "retrievable": true,
            "sortable": true,
            "facetable": false,
            "key": false,
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "analyzer": null,
            "synonymMaps": []
        },
        {
            "name": "Description",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "retrievable": false,
            "sortable": false,
            "facetable": false,
            "key": false,
            "indexAnalyzer": null,
            "searchAnalyzer": null,
            "analyzer": null,
            "synonymMaps": []
        },
        ...
          "scoringProfiles": [],
    "defaultScoringProfile": null,
    "corsOptions": null,
    "suggesters": [],
    "analyzers": [],
    "tokenizers": [],
    "tokenFilters": [],
    "charFilters": []
}
```

### <a name="create-an-indexer"></a>建立索引子

索引子會將資料來源連接至目標搜尋索引，並選擇性地提供排程來將資料重新整理自動化。

此呼叫的 URL 是 `https://[service name].search.windows.net/indexers?api-version=2016-09-01-Preview`。 使用您的搜尋服務名稱來取代 `[service name]`。

首先，取代 URL。 接著複製下列程式碼並貼至您的主體，然後執行查詢。

```json
{
  "name" : "clinical-trials-json-indexer",
  "dataSourceName" : "clinical-trials-json",
  "targetIndexName" : "clinical-trials-json-index",
  "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
}
```

回應看起來應如下所示：

```json
{
    "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexers/$entity",
    "@odata.etag": "\"0x8D505FDE143D164\"",
    "name": "clinical-trials-json-indexer",
    "description": null,
    "dataSourceName": "clinical-trials-json",
    "targetIndexName": "clinical-trials-json-index",
    "schedule": null,
    "parameters": {
        "batchSize": null,
        "maxFailedItems": null,
        "maxFailedItemsPerBatch": null,
        "base64EncodeKeys": null,
        "configuration": {
            "parsingMode": "jsonArray"
        }
    },
    "fieldMappings": [],
    "enrichers": [],
    "disabled": null
}
```

## <a name="search-your-json-files"></a>搜尋您的 JSON 檔案

您的搜尋服務現已連線至資料容器，因此可以開始搜尋檔案。

開啟 Azure 入口網站，並瀏覽回到您的搜尋服務。 就像您在上一個教學課程中所做的。

  ![未結構化搜尋](media/search-semi-structured-data/indexespane.png)

### <a name="user-defined-metadata-search"></a>使用者定義的中繼資料搜尋

如前所述，您可以使用數種方式來查詢資料：全文檢索搜尋、系統內容或使用者定義的中繼資料。 如果已在建立目標索引期間將系統內容和使用者定義的中繼資料標記為 **retrievable**，就只能使用 `$select` 參數來搜尋它們。 索引中的參數一旦建立之後，就無法改變。 不過，可以新增其他參數。

有一個基本查詢範例是 `$select=Gender,metadata_storage_size`，它會將傳回限制為這兩個參數。

  ![半結構化搜尋](media/search-semi-structured-data/lastquery.png)

有一個更複雜的查詢範例是 `$filter=MinimumAge ge 30 and MaximumAge lt 75`，它只會傳回參數 MinimumAge 大於或等於 30 且 MaximumAge 小於 75 的結果。

  ![半結構化搜尋](media/search-semi-structured-data/metadatashort.png)

如果您想要自行實驗並嘗試其他更多查詢，請隨意嘗試。 了解您可以使用邏輯運算子 (and、or、not) 和比較運算子 (eq、ne、gt、lt、ge、le)。 字串比較是區分大小寫的。

`$filter` 參數只能與在索引建立期間標記為可篩選的中繼資料一起使用。

## <a name="clean-up-resources"></a>清除資源

在完成教學課程後，最快速的清除方式是刪除包含 Azure 搜尋服務的資源群組。 您現在可以刪除資源群組，以永久刪除當中所包含的所有項目。 在入口網站中，資源群組名稱位在 Azure 搜尋服務的 [概觀] 頁面上。

## <a name="next-steps"></a>後續步驟

您可以將 AI 功能的演算法附加至索引子的管線。 下一個步驟中，繼續進行下列教學課程。

> [!div class="nextstepaction"]
> [在 Azure Blob 儲存體中為文件編製索引](search-howto-indexing-azure-blob-storage.md)