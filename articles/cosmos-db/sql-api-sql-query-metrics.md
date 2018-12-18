---
title: 適用於 Azure Cosmos DB SQL API 的 SQL 查詢計量 | Microsoft Docs
description: 了解如何檢測和偵錯 Azure Cosmos DB 要求的 SQL 查詢效能。
keywords: sql 語法, sql 查詢, sql 查詢, json 查詢語言, 資料庫概念和 sql 查詢, 彙總函式
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: sngun
ms.openlocfilehash: 4ed0008f4b574691387d6e0ee0300b5f05f1ec1b
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34798690"
---
# <a name="tuning-query-performance-with-azure-cosmos-db"></a>使用 Azure Cosmos DB 調整查詢效能

Azure Cosmos DB 提供一個[適用於查詢資料的 SQL API](sql-api-sql-query.md)，而不需結構描述或次要索引。 本文可為開發人員提供下列資訊：

* 關於 Azure Cosmos DB 之 SQL 查詢執行如何運作的高階詳細資料
* 關於查詢要求和回應標頭以及用戶端 SDK 選項的詳細資訊
* 查詢效能的秘訣和最佳作法
* 如何運用 SQL 執行統計資料來偵錯查詢效能的範例

## <a name="about-sql-query-execution"></a>關於 SQL 查詢執行

在 Azure Cosmos DB 中，您會將資料儲存於容器中，其可擴張到任何的[儲存體大小或要求輸送量](partition-data.md)。 Azure Cosmos DB 會在涵蓋範圍內，於實體分割區之間順暢地調整資料，以處理資料成長或提高佈建的輸送量。 您可以使用 REST API 或其中一個支援的 [SQL SDK](sql-api-sdk-dotnet.md)，向任何容器發出 SQL 查詢。

分割的簡短概觀：您定義分割區索引鍵 (例如 "city")，以決定在實體分割區之間分割資料的方式。 屬於單一分割區索引鍵的資料 (例如，"city" == "Seattle") 會儲存於一個實體分割區內，但通常單一實體分割區會具有多個分割區索引鍵。 當分割區達到其儲存體大小時，服務會順暢地將該分割區分割為兩個新的分割區，並在這些分割區之間將分割區索引鍵平均分割。 由於分割區是暫時性的，因此，API 會使用「分割區索引鍵範圍」的抽象概念，來代表分割區索引鍵雜湊的範圍。 

當您向 Azure Cosmos DB 發出查詢時，SDK 會執行下列邏輯步驟：

* 剖析 SQL 查詢以決定查詢執行計畫。 
* 如果查詢所包含的篩選係以分割區索引鍵為依據 (例如 `SELECT * FROM c WHERE c.city = "Seattle"`)，即會將該查詢路由傳送至單一分割區。 如果查詢中沒有與分割區索引鍵相關的篩選，則會在所有分割區中執行該查詢，且結果會合併用戶端。
* 根據用戶端設定而定，查詢會依序或平行地在每個分割區內執行。 在每個分割區內，根據查詢複雜度、設定的頁面大小，以及集合的佈建輸送量而定，查詢可能會進行一或多次來回行程。 每次執行都會傳回查詢執行所取用的[要求單位](request-units.md)數目，以及 (選擇性地) 查詢執行統計資料。 
* SDK 會跨分割區執行查詢結果的摘要。 例如，如果查詢跨分割區包含 ORDER BY，則來自個別分割區的結果就會依序合併，以全域排序順序傳回結果。 如果查詢是 `COUNT` 之類的彙總，即會將來自個別分割區的計數加總以產生整體計數。

SDK 提供適用於查詢執行的各種選項。 例如，在 .NET 中，下列選項適用於 `FeedOptions` 類別。 下表描述這些選項，以及它們如何影響查詢執行時間。 

| 選項 | 說明 |
| ------ | ----------- |
| `EnableCrossPartitionQuery` | 必須針對任何需要跨多個分割區執行的查詢設定為 true。 這是一個明確的旗標，讓您能夠在開發期間進行明智的效能取捨。 |
| `EnableScanInQuery` | 如果您已選擇不編製索引，但仍想透過掃描繼續執行查詢，就必須設定為 true。 只有在已停用針對所要求篩選路徑編製索引的功能時才適用。 | 
| `MaxItemCount` | 在每次到伺服器的來回行程中要傳回的項目數上限。 設定為 -1，您就能讓伺服器管理項目數。 或者，您可以降低此值，以便在每次來回行程中只擷取少數項目。 
| `MaxBufferedItemCount` | 這是用戶端選項，可在執行跨分割區 ORDER BY 時，用來限制記憶體耗用量。 較高的值有助於降低跨分割區排序的延遲。 |
| `MaxDegreeOfParallelism` | 取得或設定在 Azure Cosmos DB 資料庫服務中平行查詢執行期間，在用戶端執行的並行操作次數。 正值的屬性值會將並行操作次數限制為設定的值。 如果將它設為小於 0，系統就會自動決定要執行的並行操作次數。 |
| `PopulateQueryMetrics` | 啟用在查詢執行的各種階段 (例如，編譯時間、索引迴圈時間及文件載入時間) 所花費時間的統計資料詳細記錄。 您可以與 Azure 支援人員分享來自查詢統計資料的輸出，以診斷查詢效能問題。 |
| `RequestContinuation` | 您可以藉由傳入任何查詢所傳回的不透明接續 Token，繼續進行查詢執行。 接續 Token 會封裝查詢執行所需的所有狀態。 |
| `ResponseContinuationTokenLimitInKb` | 您可以限制伺服器所傳回之接續 Token 的大小上限。 如果您的應用程式主機有回應標頭大小的限制，則您可能需要設定此項。 設定此項，可能會增加整體持續時間以及針對查詢所取用的 RU。  |

例如，假設我們的範例查詢會將集合上透過 `/city` 要求的分割區索引鍵當作分割區索引鍵，並使用 100,000 RU/秒的輸送量進行佈建。 您可以在 .NET 中使用 `CreateDocumentQuery<T>` 要求此查詢，如下所示：

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
        MaxItemCount = -1, 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();
```

如上所示的 SDK 程式碼片段會對應至下列 REST API 要求：

```
POST https://arramacquerymetrics-westus.documents.azure.com/dbs/db/colls/sample/docs HTTP/1.1
x-ms-continuation: 
x-ms-documentdb-isquery: True
x-ms-max-item-count: -1
x-ms-documentdb-query-enablecrosspartition: True
x-ms-documentdb-query-parallelizecrosspartitionquery: True
x-ms-documentdb-query-iscontinuationexpected: True
x-ms-documentdb-populatequerymetrics: True
x-ms-date: Tue, 27 Jun 2017 21:52:18 GMT
authorization: type%3dmaster%26ver%3d1.0%26sig%3drp1Hi83Y8aVV5V6LzZ6xhtQVXRAMz0WNMnUuvriUv%2b4%3d
x-ms-session-token: 7:8,6:2008,5:8,4:2008,3:8,2:2008,1:8,0:8,9:8,8:4008
Cache-Control: no-cache
x-ms-consistency-level: Session
User-Agent: documentdb-dotnet-sdk/1.14.1 Host/32-bit MicrosoftWindowsNT/6.2.9200.0
x-ms-version: 2017-02-22
Accept: application/json
Content-Type: application/query+json
Host: arramacquerymetrics-westus.documents.azure.com
Content-Length: 52
Expect: 100-continue

{"query":"SELECT * FROM c WHERE c.city = 'Seattle'"}
```

每個查詢執行頁面都會對應至含有 `Accept: application/query+json` 標頭的 REST API `POST`，以及主體中的 SQL 查詢。 每個查詢都會利用在用戶端與伺服器之間傳遞的 `x-ms-continuation` Token，來進行到伺服器的一或多次來回行程，以便繼續執行。 FeedOptions 中的設定選項均會以要求標頭的形式傳遞到伺服器。 例如，`MaxItemCount` 會對應到 `x-ms-max-item-count`。 

要求會傳回下列 (已截斷以提高可讀性) 的回應：

```
HTTP/1.1 200 Ok
Cache-Control: no-store, no-cache
Pragma: no-cache
Transfer-Encoding: chunked
Content-Type: application/json
Server: Microsoft-HTTPAPI/2.0
Strict-Transport-Security: max-age=31536000
x-ms-last-state-change-utc: Tue, 27 Jun 2017 21:01:57.561 GMT
x-ms-resource-quota: documentSize=10240;documentsSize=10485760;documentsCount=-1;collectionSize=10485760;
x-ms-resource-usage: documentSize=1;documentsSize=884;documentsCount=2000;collectionSize=1408;
x-ms-item-count: 2000
x-ms-schemaversion: 1.3
x-ms-alt-content-path: dbs/db/colls/sample
x-ms-content-path: +9kEANVq0wA=
x-ms-xp-role: 1
x-ms-documentdb-query-metrics: totalExecutionTimeInMs=33.67;queryCompileTimeInMs=0.06;queryLogicalPlanBuildTimeInMs=0.02;queryPhysicalPlanBuildTimeInMs=0.10;queryOptimizationTimeInMs=0.00;VMExecutionTimeInMs=32.56;indexLookupTimeInMs=0.36;documentLoadTimeInMs=9.58;systemFunctionExecuteTimeInMs=0.00;userFunctionExecuteTimeInMs=0.00;retrievedDocumentCount=2000;retrievedDocumentSize=1125600;outputDocumentCount=2000;writeOutputTimeInMs=18.10;indexUtilizationRatio=1.00
x-ms-request-charge: 604.42
x-ms-serviceversion: version=1.14.34.4
x-ms-activity-id: 0df8b5f6-83b9-4493-abda-cce6d0f91486
x-ms-session-token: 2:2008
x-ms-gatewayversion: version=1.14.33.2
Date: Tue, 27 Jun 2017 21:59:49 GMT
```

從查詢傳回的索引鍵回應標頭包括下列項目：

| 選項 | 說明 |
| ------ | ----------- |
| `x-ms-item-count` | 在回應中傳回的項目數。 這相依於所提供的 `x-ms-max-item-count`、最大回應承載大小內可容納的項目數、佈建的輸送量，以及查詢執行時間。 |  
| `x-ms-continuation:` | 要繼續執行查詢的接續 Token (如果有其他結果可供使用)。 | 
| `x-ms-documentdb-query-metrics` | 執行的查詢統計資料。 這是一個分隔的字串，包含在查詢執行的各個不同階段所花費時間的統計資料。 如果將 `x-ms-documentdb-populatequerymetrics` 設為 `True`，即會傳回。 | 
| `x-ms-request-charge` | 查詢取用的[要求單位](request-units.md)數目。 | 

如需 REST API 要求標頭和選項的詳細資訊，請參閱[使用 REST API 查詢資源](https://docs.microsoft.com/rest/api/cosmos-db/querying-cosmosdb-resources-using-the-rest-api)。

## <a name="best-practices-for-query-performance"></a>查詢效能的最佳作法
以下是影響 Azure Cosmos DB 查詢效能的最常見因素。 我們將在本文中更深入探討下列每個主題。

| 因素 | 秘訣 | 
| ------ | -----| 
| 佈建的輸送量 | 測量每個查詢的 RU，並確認您擁有查詢所需的佈建輸送量。 | 
| 分割區和分割區索引鍵 | 在篩選子句中偏好使用具有分割區索引鍵的查詢以降低延遲。 |
| SDK 與查詢選項 | 遵循 SDK 最佳作法 (例如直接連線)，並調整用戶端查詢執行選項。 |
| 網路延遲 | 在進行測量時需考量網路的額外負荷，並使用多路連接的 API 從最接近的區域進行讀取。 |
| 索引原則 | 確定您具有查詢所需的編製索引路徑/原則。 |
| 查詢執行計量 | 分析查詢執行計量，以識別可能發生重新寫入的查詢與資料圖形。  |

### <a name="provisioned-throughput"></a>佈建的輸送量
在 Cosmos DB 中，您會建立資料的容器，每一個容器都具有以每秒要求單位 (RU) 表示的保留輸送量。 讀取 1 KB 文件就是 1 RU，而每個操作 (包括查詢) 都會根據它的複雜度正規化為固定數目的 RU。 例如，如果您已針對容器佈建了 1000 RU/秒，且具有類似 `SELECT * FROM c WHERE c.city = 'Seattle'` 的查詢 (其取用 5 RU)，則您每秒可執行 (1000 RU/秒) / (5 RU/查詢) = 200 個查詢/秒之類的查詢。 

如果您每秒提交 200 個以上的查詢，則服務會在每秒的傳入要求超過 200 個時啟動速率限制。 SDK 會藉由執行輪詢/重試來自動處理這種情況，因此您可能會注意到這些查詢具有較高的延遲。 將佈建的輸送量提高為要求的值，即可改善您的查詢延遲和輸送量。 

若要深入了解要求單位，請參閱[要求單位](request-units.md)。

### <a name="partitioning-and-partition-keys"></a>分割區和分割區索引鍵
在 Azure Cosmos DB 中，查詢通常會以下列順序，從最快/最有效率到較慢/較無效率的方式執行。 

* 單一分割區索引鍵和項目索引鍵上的 GET
* 單一分割區索引鍵上具有篩選子句的查詢
* 任何屬性上不具等號比較或範圍篩選子句的查詢
* 不含篩選的查詢

需要查閱所有分割區的查詢需要較高的延遲，而且可以取用較高的 RU。 由於每個分割區都會針對所有屬性自動編製索引，因此，在此情況下，就能有效率地從索引中提供查詢。 您可以使用平行處理原則選項，更快速地進行跨越分割區的查詢。

若要深入了解分割區和分割區索引鍵，請參閱[在 Azure Cosmos DB 中進行資料分割](partition-data.md)。

### <a name="sdk-and-query-options"></a>SDK 與查詢選項
若要了解如何從 Azure Cosmos DB 取得最佳用戶端效能，請參閱[效能祕訣](performance-tips.md)和[效能測試](performance-testing.md)。 這包括使用最新的 SDK、設定平台專屬設定 (例如預設的連線數目)、記憶體回收的頻率，以及使用輕量級連線選項 (例如直接/TCP)。 


#### <a name="max-item-count"></a>最大項目計數
對於查詢，`MaxItemCount` 的值可能會對端對端查詢時間有重大影響。 伺服器每次往返將會傳回不超過 `MaxItemCount` 中的項目數 (預設值是 100 個項目)。 將此設為較高的值 (-1 是最大值，建議使用)，會限制伺服器和用戶端之間的往返次數，尤其是具有大量結果集的查詢，以改善整體查詢持續時間。

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxItemCount = -1, 
    }).AsDocumentQuery();
```

#### <a name="max-degree-of-parallelism"></a>平行處理原則的最大程度
針對查詢，請調整 `MaxDegreeOfParallelism` 來識別應用程式的最佳設定，特別是當您執行跨分割區查詢時 (不含分割區索引鍵值上的篩選)。 `MaxDegreeOfParallelism` 可控制平行工作的最大數目，例如，要平行造訪的最大分割區數目。 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();
```

假設
* D = 平行工作的預設最大數目 (= 用戶端機器中的處理器數目總計)
* P = 使用者指定的平行工作最大數目
* N = 針對回答查詢需要造訪的分割區數目

以下是平行查詢對於不同 P 值之行為的影響。
* (P == 0) => 序列模式
* (P == 1) => 一項工作的最大值
* (P > 1) => 平行工作的最小值 (P, N) 
* (P < 1) => 平行工作的最小值 (N, D)

如需 SDK 版本資訊，以及所實作類別和方法的詳細資訊，請參閱 [SQL SDK](sql-api-sdk-dotnet.md)

### <a name="network-latency"></a>網路延遲
若要了解如何設定全域散發，並連線到最接近的區域，請參閱 [Azure Cosmos DB 全域散發](tutorial-global-distribution-sql-api.md)。 當您需要進行多次來回行程或從查詢擷取大型結果集時，網路延遲會對查詢效能產生顯著影響。 

查詢執行計量上的區段說明如何擷取查詢的伺服器執行時間 ( `totalExecutionTimeInMs`)，以便您可以區分查詢執行所花費的時間以及網路傳輸所花費的時間。

### <a name="indexing-policy"></a>編製索引原則
若要了解編製索引路徑、種類和模式，以及它們如何影響執行查詢，請參閱[設定編製索引原則](indexing-policies.md)。 根據預設，編製索引原則會針對字串使用雜湊索引編製，這對於等號比較查詢很有效率，但不適用於範圍查詢/排序依據查詢。 如果您需要針對字串進行範圍查詢，我們建議針對所有字串指定範圍索引類型。 

## <a name="query-execution-metrics"></a>查詢執行計量
您可以藉由傳入選擇性的 `x-ms-documentdb-populatequerymetrics` 標頭 (.NET SDK 中的 `FeedOptions.PopulateQueryMetrics`)，來取得查詢執行的詳細計量。 `x-ms-documentdb-query-metrics` 中傳回的值含有下列索引鍵/值組，適用於對查詢執行進行進階疑難排解。 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;

```

| 計量 | 單位 | 說明 | 
| ------ | -----| ----------- |
| `totalExecutionTimeInMs` | 毫秒 | 查詢執行時間 | 
| `queryCompileTimeInMs` | 毫秒 | 查詢編譯時間  | 
| `queryLogicalPlanBuildTimeInMs` | 毫秒 | 建置邏輯查詢計劃的時間 | 
| `queryPhysicalPlanBuildTimeInMs` | 毫秒 | 建置實體查詢計劃的時間 | 
| `queryOptimizationTimeInMs` | 毫秒 | 最佳化查詢所花費的時間 | 
| `VMExecutionTimeInMs` | 毫秒 | 查詢執行階段所花費的時間 | 
| `indexLookupTimeInMs` | 毫秒 | 實體索引層內所花費的時間 | 
| `documentLoadTimeInMs` | 毫秒 | 載入文件所花費的時間  | 
| `systemFunctionExecuteTimeInMs` | 毫秒 | 執行系統 (內建) 函式所花費的總時間，以毫秒為單位  | 
| `userFunctionExecuteTimeInMs` | 毫秒 | 執行使用者定義函式所花費的總時間，以毫秒為單位 | 
| `retrievedDocumentCount` | count | 擷取的文件總數  | 
| `retrievedDocumentSize` | 位元組 | 擷取的文件大小總計，以位元組為單位  | 
| `outputDocumentCount` | count | 輸出文件數目 | 
| `writeOutputTimeInMs` | 毫秒 | 查詢執行時間，以毫秒為單位 | 
| `indexUtilizationRatio` | 比率 (<=1) | 符合篩選的文件數目與所載入文件數目的比率  | 

用戶端 SDK 可能會在內部進行多個查詢操作，以在每個分割區內提供查詢。 如果結果總數超過 `x-ms-max-item-count`、如果查詢超出分割區的佈建輸送量、如果查詢承載達到每個頁面的大小上限，或者如果查詢達到系統已配置的逾時限制，用戶端就會針對每個分割區進行多個呼叫。 每個部分查詢執行都會傳回該頁面的 `x-ms-documentdb-query-metrics`。 

以下是一些範例查詢，以及如何解譯從查詢執行傳回的部分計量資訊： 

| 查詢 | 範例計量 | 說明 | 
| ------ | -----| ----------- |
| `SELECT TOP 100 * FROM c` | `"RetrievedDocumentCount": 101` | 擷取的文件數目為 100 + 1，可符合 TOP 子句。 查詢時間大部分花費在 `WriteOutputTime` 和 `DocumentLoadTime`，因為它是一次掃描。 | 
| `SELECT TOP 500 * FROM c` | `"RetrievedDocumentCount": 501` | RetrievedDocumentCount 現在較高 (500+1 以符合 TOP 子句)。 | 
| `SELECT * FROM c WHERE c.N = 55` | `"IndexLookupTime": "00:00:00.0009500"` | 在 IndexLookupTime 中大約花費了 0.9 毫秒進行索引鍵查閱，因為它會透過 `/N/?` 進行索引查閱。 | 
| `SELECT * FROM c WHERE c.N > 55` | `"IndexLookupTime": "00:00:00.0017700"` | 在範圍索引中，於 IndexLookupTime 中所花費的時間稍微多一點 (1.7 毫秒)，因為它會根據 `/N/?` 進行索引查閱。 | 
| `SELECT TOP 500 c.N FROM c` | `"IndexLookupTime": "00:00:00.0017700"` | 在 `DocumentLoadTime` 上所花費的時間與上一個查詢相同，但 `WriteOutputTime` 較低，因為我們只要投影一個屬性。 | 
| `SELECT TOP 500 udf.toPercent(c.N) FROM c` | `"UserDefinedFunctionExecutionTime": "00:00:00.2136500"` | 在 `UserDefinedFunctionExecutionTime` 中大約花費了 213 毫秒，以針對 `c.N` 的每個值執行 UDF。 |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(c.Name, 'Den')` | `"IndexLookupTime": "00:00:00.0006400", "SystemFunctionExecutionTime": "00:00:00.0074100"` | 在 `IndexLookupTime` 中針對 `/Name/?` 大約花費了 0.6 毫秒。 `SystemFunctionExecutionTime` 中，大部分的查詢執行時間 (~7 毫秒)。 |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(LOWER(c.Name), 'den')` | `"IndexLookupTime": "00:00:00", "RetrievedDocumentCount": 2491,  "OutputDocumentCount": 500` | 查詢會當作掃描來執行，因為它使用 `LOWER`，而且會傳回 2491 份擷取文件中的 500 份。 |


## <a name="next-steps"></a>後續步驟
* 若要了解支援的 SQL 查詢運算子和關鍵字，請參閱 [SQL 查詢](sql-api-sql-query.md)。 
* 若要了解要求單位，請參閱[要求單位](request-units.md)。
* 若要了解編製索引原則，請參閱[編製索引原則](indexing-policies.md) 


