---
title: Azure Cosmos DB 常見問題集 | Microsoft Docs
description: 取得關於 Azure Cosmos DB (一種全域散發的多模型資料庫服務) 之常見問題集的解答。 了解產能、效能層級和調整。
keywords: 資料庫問題, 常見問題集, Database questions, frequently asked questions, documentdb, azure, Microsoft azure
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: sngun
ms.openlocfilehash: de67d71a209ea0889b4dc5c87fed254700cb9916
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395123"
---
# <a name="azure-cosmos-db-faq"></a>Azure Cosmos DB 常見問題集
## <a name="azure-cosmos-db-fundamentals"></a>Azure Cosmos DB 基本概念

### <a name="what-happened-to-the-documentdb-api"></a>DocumentDB API 有何改變？

Azure Cosmos DB DocumentDB API 或 SQL (DocumentDB) API 現在也稱為 Azure Cosmos DB SQL API。 您不需要變更任何項目，即可繼續執行使用 DocumentDB API 建置的應用程式。 功能保持不變。

如果您之前已有 DocumentDB API 帳戶，現在就能擁有 SQL API 帳戶，且不會向您收取任何費用。 

### <a name="what-happened-to-azure-documentdb-as-a-service"></a>Azure DocumentDB 服務有何改變？

Azure DocumentDB 服務現在是 Azure Cosmos DB 服務的一部分，且本身以 SQL API 形式呈現。 針對 Azure DocumentDB 所建置的應用程式可在不變更 Azure Cosmos DB SQL API 的情況下執行。 此外，Azure Cosmos DB 支援 Gremlin API、資料表 API、MongoDB API 和 Cassandra API (預覽)。

### <a name="what-are-the-typical-use-cases-for-azure-cosmos-db"></a>Azure Cosmos DB 有哪些一般使用案例？
對於新的 Web、行動裝置、遊戲和 IoT 應用程式，若其中自動調整規模、可預測的效能、毫秒回應時間的快速排序，以及查詢無結構描述之資料的能力都很重要，則 Azure Cosmos DB 是個不錯的選擇。 Azure Cosmos DB 適合用來進行快速開發，且支援應用程式資料模型的持續反覆運算。 管理使用者產生之內容和資料的應用程式，就是 [Azure Cosmos DB 的常見使用案例](use-cases.md)。 

### <a name="how-does-azure-cosmos-db-offer-predictable-performance"></a>Azure Cosmos DB 如何提供可預測的效能？
[要求單位](request-units.md) (RU) 是 Azure Cosmos DB 的輸送量計算單位。 1 RU 的輸送量會對應至 1KB 文件的 GET 輸送量。 根據完成作業所需的輸送量，Azure Cosmos DB 中的每個作業 (包括讀取、寫入、SQL 查詢和預存程序執行) 具有決定性的 RU 值。 您可以就單一 RU 計量來思考，而不是思考 CPU、IO 和記憶體以及它們分別如何影響您的應用程式輸送量。

您可以按照每秒輸送量 RU 的佈建輸送量保留每個 Azure Cosmos DB 容器。 對於任何規模的應用程式，您可以將個別要求設為基準以測量其 RU 值，以及佈建容器來處理所有要求的要求單位總數。 您也可以隨著應用程式發展需求，相應增加或減少容器的輸送量。 如需有關要求單位的詳細資訊，以及幫助您判斷容器需求的協助，請參閱[估計輸送量需求](request-units.md#estimating-throughput-needs)並嘗試使用[輸送量計算機 (英文)](https://www.documentdb.com/capacityplanner)。 這裡的「容器」一詞指的是 SQL API 集合、Gremlin API 圖形、MongoDB API 集合及資料表 API 資料表。 

### <a name="how-does-azure-cosmos-db-support-various-data-models-such-as-keyvalue-columnar-document-and-graph"></a>Azure Cosmos DB 如何支援各種資料模型，例如索引鍵/值、單欄式資料、文件和圖形？

索引鍵/值、單欄式資料、文件和圖形都是原生支援的模型，因為 Azure Cosmos DB 內建 ARS (原子、記錄、序列) 設計。 原子、記錄、序列可以輕易地對應並且投射至各種資料模型。 目前已有適用於各種模型子集合的 API (SQL、MongoDB、資料表、Gremlin API)，未來會有更多用於其他特定資料模型的 API。

Azure Cosmos DB 不需要結構描述的索引引擎能夠自動建立所內嵌之資料的索引，且不需要查詢任何結構描述或開發者的次要索引。 引擎依賴一組邏輯索引配置 (反向式、單欄式、樹狀目錄)，可將儲存配從索引和查詢的處理子系統分離。 Cosmos DB 也可以擴充支援一些有線網路通訊協定和 API，並將它們有效地轉譯成核心資料模型 (1) 和邏輯索引配置，(2) 造就其原生支援多個資料模型的獨特能力。

### <a name="is-azure-cosmos-db-hipaa-compliant"></a>Azure Cosmos DB 符合 HIPAA 規範嗎？
是，Azure Cosmos DB 符合 HIPAA 規範。 HIPAA 建立了使用、洩漏與保護個別辨識健全狀況資訊的需求。 如需詳細資訊，請參閱 [Microsoft 信任中心](https://www.microsoft.com/en-us/TrustCenter/Compliance/HIPAA)。

### <a name="what-are-the-storage-limits-of-azure-cosmos-db"></a>Azure Cosmos DB 的儲存體限制有哪些？
對於容器可以儲存在 Azure Cosmos DB 中的資料總量，沒有任何限制。

### <a name="what-are-the-throughput-limits-of-azure-cosmos-db"></a>Azure Cosmos DB 的輸送量限制有哪些？
對於容器可以在 Azure Cosmos DB 中支援的總輸送量，沒有任何限制。 其主要概念在於，將您的工作負載大致平均分配在數量夠大的資料分割索引鍵之間。

### <a name="are-direct-and-gateway-connectivity-modes-encrypted-"></a>直接和閘道連線模式是否會加密？ 
是，這兩種模式一律會完整加密。 

### <a name="how-much-does-azure-cosmos-db-cost"></a>Azure Cosmos DB 的費用是多少？
如需詳細資料，請參閱 [Azure Cosmos DB 定價詳細資料](https://azure.microsoft.com/pricing/details/cosmos-db/)頁面。 Azure Cosmos DB 使用量費用取決於已佈建的容器數目、容器在線上的時數，以及每個容器的佈建輸送量。 這裡的「容器」一詞指的是 SQL API 集合、Gremlin API (Graph)、MongoDB API 集合及資料表 API 資料表。 

### <a name="is-a-free-account-available"></a>有免費的帳戶嗎？
是，您可以免費註冊限時的帳戶，沒有承諾用量。 若要註冊，請造訪[免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)，或在[試用 Azure Cosmos DB 常見問題集](#try-cosmos-db)中深入了解。

如果您不熟悉 Azure，可以註冊 [Azure 免費帳戶](https://azure.microsoft.com/free/)，就能得到 30 天免費試用以及信用額度，讓您試用所有 Azure 服務。 如果您有 Visual Studio 訂用帳戶，也享有[免費的 Azure 信用額度](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)，可用於所有 Azure 服務。 

您也可以免費使用 [Azure Cosmos DB 模擬器](local-emulator.md)在本機開發及測試應用程式，而不需建立 Azure 訂用帳戶。 如果您滿意應用程式在 Azure Cosmos DB 模擬器中的運作方式，就可以切換成使用雲端的 Azure Cosmos DB 帳戶。

### <a name="how-can-i-get-additional-help-with-azure-cosmos-db"></a>如何取得 Azure Cosmos DB 的其他協助？

若要詢問技術問題，您可以將問題張貼到下列兩個問答論壇之一：
* [MSDN 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurecosmosdb)
* [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-cosmosdb)。 Stack Overflow 最適合用於詢問程式設計問題。 確保您的問題[契合主題](https://stackoverflow.com/help/on-topic)且[盡可能提供愈多的詳細資料，讓問題更加清楚並可得到答覆](https://stackoverflow.com/help/how-to-ask)。 

若要要求新功能，請在 [Uservoice](https://feedback.azure.com/forums/263030-azure-cosmos-db) 上建立新的要求。

若要修正您的帳戶問題，請在 Azure 入口網站中提出[支援要求](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)。

您可以將其他問題提交給小組 (利用 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com))；不過，這不是技術支援小組的別名。 


## <a id="try-cosmos-db"></a>試用 Azure Cosmos DB 訂用帳戶

您現在不需要訂用帳戶即可享用限時的 Azure Cosmos DB 體驗，不僅免費，也沒有任何承諾用量。 若要註冊試用 Azure Cosmos DB 訂用帳戶，請移至[免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)。 此訂用帳戶與 [Azure 免費試用](https://azure.microsoft.com/free/)有所區別，有了 Azure 免費試用或 Azure 付費訂用帳戶之後，還是可以使用這個訂用帳戶。 

試用 Azure Cosmos DB 訂用帳戶會顯示在 Azure 入口網站中，與您的使用者識別碼相關聯的其他訂用帳戶旁。 

下列條件適用於試用 Azure Cosmos DB 訂用帳戶：

* SQL、Gremlin API 和資料表帳戶的每個訂用帳戶一個容器。
* MongoDB 帳戶的每個訂用帳戶最多 3 個集合。
* 10 GB 儲存體容量。
* 全域複寫適用於下列 [Azure 區域](https://azure.microsoft.com/regions/)：美國中部、北歐和東南亞
* 輸送量上限 5K RU/s。
* 訂用帳戶將於 24 小時之後到期，可以擴充到最多總時數 48 小時。
* 無法為試用 Azure Cosmos DB 帳戶建立 Azure 支援票證，不過，會以現有支援計劃為訂閱者提供支援。 

## <a name="set-up-azure-cosmos-db"></a>設定 Azure Cosmos DB
### <a name="how-do-i-sign-up-for-azure-cosmos-db"></a>如何註冊 Azure Cosmos DB？
Azure 入口網站中已提供 Azure Cosmos DB。 首先，請註冊 Azure 訂用帳戶。 註冊後，您就可以將 SQL API、Gremlin API、資料表 API、MongoDB API 或 Cassandra API 帳戶新增至您的 Azure 訂用帳戶。

### <a name="what-is-a-master-key"></a>什麼是主要金鑰？
主要金鑰是可存取帳戶的所有資源的安全性權杖。 擁有此金鑰的人，對資料庫帳戶中的所有資源具有讀取和寫入存取權。 分配主要金鑰時，務必謹慎。 [Azure 入口網站][azure-portal]的 [金鑰] 刀鋒視窗提供主要金鑰和次要金鑰。 如需金鑰的詳細資訊，請參閱 [檢視、複製和重新產生存取金鑰](manage-account.md#keys)。

### <a name="what-are-the-regions-that-preferredlocations-can-be-set-to"></a>設定 PreferredLocations 的目標區域有哪些？ 
PreferredLocations 值可以設定為任何提供 Cosmos DB 的 Azure 區域。 如需可用區域的清單，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。

### <a name="is-there-anything-i-should-be-aware-of-when-distributing-data-across-the-world-via-the-azure-datacenters"></a>當我透過 Azure 的資料中心在世界各地散發資料時，應注意哪些事項？ 
所有的 Azure 區域均提供 Azure Cosmos DB ，如同 [Azure 區域](https://azure.microsoft.com/regions/)頁面所指出。 由於它是核心服務，因此每個新資料中心都會有一個 Azure Cosmos DB。 

設定區域時，請記住 Azure Cosmos DB 涉及主權和政府雲端。 也就是如果您在[主權區域](https://azure.microsoft.com/global-infrastructure/)中建立帳戶，便無法覆寫到該[主權區域](https://azure.microsoft.com/global-infrastructure/)之外的位置。 同樣地，您也無法從外部帳戶覆寫到其他主權位置。 

### <a name="is-it-possible-to-switch-from-container-level-throughput-provisioning-to-database-level-throughput-provisioning-or-vice-versa"></a>是否可以從容器層級輸送量佈建，切換成資料庫層級輸送量佈建？ 或者反過來？

容器和資料庫層級輸送量佈建是不同的供應項目，在其間切換需要將資料從來源移轉到目的地。 這表示您需要建立新資料庫或新集合，然後藉由使用[大量執行程式程式庫](bulk-executor-overview.md)或 [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) 來移轉資料。

### <a name="how-do-i-create-fixed-collection-with-partition-key"></a>如何使用分割區索引鍵建立固定的集合

目前，您可以使用 .Net SDK 的 [CreatePartitionedCollection](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/CollectionManagement/Program.cs#L118) 方法或使用 [Azure CLI](https://docs.microsoft.com/cli/azure/cosmosdb/collection?view=azure-cli-latest#az-cosmosdb-collection-create)，透過分割區索引鍵的輸送量來建立集合。 目前不支援使用 Azure 入口網站建立固定集合。  

### <a name="does-azure-cosmosdb-support-time-series-analysis"></a>Azure CosmosDB 是否支援時間序列分析？ 
是的，Azure CosmosDB 支援時間序列分析，請參考這裡的[時間序列模式範例](https://github.com/Azure/azure-cosmosdb-dotnet/tree/master/samples/Patterns)。 此範例會顯示如何使用變更摘要來建置經過一段時間的資料彙總檢視。 您可以使用 Spark 串流或其他串流資料處理器來延伸方法的使用範圍。


## <a name="sql-api"></a>SQL API

### <a name="how-do-i-start-developing-against-the-sql-api"></a>如何開始針對 SQL API 進行開發？
首先，您必須註冊 Azure 訂用帳戶。 註冊 Azure 訂用帳戶之後，您可以將 SQL API 容器新增至您的 Azure 訂用帳戶。 如需加入 Azure Cosmos DB 帳戶的相關指示，請參閱[建立 Azure Cosmos DB 資料庫帳戶](create-sql-api-dotnet.md#create-account)。 

[SDK](sql-api-sdk-dotnet.md) 適用於 .NET、Python、Node.js、JavaScript 和 Java。 開發人員也可以使用 [RESTful HTTP API](/rest/api/cosmos-db/)，經由各種平台和語言來與 Azure Cosmos DB 資源互動。

### <a name="can-i-access-some-ready-made-samples-to-get-a-head-start"></a>我可以存取一些現有的範例作為有利的開端嗎？
GitHub 上提供 SQL API [.NET](sql-api-dotnet-samples.md)、[Java](https://github.com/Azure/azure-documentdb-java)、[Node.js](sql-api-nodejs-samples.md) 和 [Python](sql-api-python-samples.md) SDK 的範例。


### <a name="does-the-sql-api-database-support-schema-free-data"></a>SQL API 資料庫支援無結構描述的資料嗎？
是，SQL API 可讓應用程式儲存任意的 JSON 文件，而不需要結構描述定義或提示。 透過 Azure Cosmos DB SQL 查詢介面，就可立即查詢資料。  

### <a name="does-the-sql-api-support-acid-transactions"></a>SQL API 支援 ACID 交易嗎？
是，SQL API 支援以 JavaScript 預存程序和觸發程序表示的跨文件交易。 交易以每個容器的單一分割為範圍，且以 ACID 語意執行，也就是「全有或全無」，與其他並行執行的程式碼和使用者要求隔離。 如果伺服器端執行 JavaScript 應用程式碼期間擲回例外狀況，則會回復整個交易。 如需交易的詳細資訊，請參閱 [資料庫程式交易](programming.md#database-program-transactions)。

### <a name="what-is-a-container"></a>容器是什麼？
容器是一組文件及其相關聯的 JavaScript 應用程式邏輯。 容器是計費實體，其[成本](performance-levels.md)是由輸送量和使用的儲存體所決定。 容器可以跨越一或多個磁碟分割或伺服器，也可以進行調整以處理幾乎無限量的儲存體或輸送量。 

* 針對 SQL 和 MongoDB API 帳戶，容器會對應至集合。 
* 針對 Cassandra 和資料表 API 帳戶，容器會對應至資料表。 
* 針對 Gremlin API 帳戶，容器會對應至圖表。 

容器也是 Azure Cosmos DB 的計費實體。 每個容器根據佈建的輸送量和使用的儲存體空間，以小時為單位計費。 如需詳細資訊，請參閱 [Azure Cosmos DB 價格](https://azure.microsoft.com/pricing/details/cosmos-db/)。 

### <a name="how-do-i-create-a-database"></a>我如何建立資料庫？
您可以使用 [Azure 入口網站](https://portal.azure.com) (如[新增集合](create-sql-api-dotnet.md#create-collection)所述)、其中一個 [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) 或 [REST API](/rest/api/cosmos-db/) 來建立資料庫。 

### <a name="how-do-i-set-up-users-and-permissions"></a>我如何設定使用者和權限？
您可以使用其中一個 [Cosmos DB API SDK](sql-api-sdk-dotnet.md) 或 [REST API](/rest/api/cosmos-db/) 來建立使用者和權限。  

### <a name="does-the-sql-api-support-sql"></a>SQL API 支援 SQL 嗎？
SQL API 帳戶所支援的 SQL 查詢語言是 SQL Server 所支援的查詢功能增強子集。 Azure Cosmos DB 的 SQL 查詢語言提供豐富的階層式和關聯式運算子，且透過 JavaScript 型使用者定義函式 (UDF) 支援擴充性。 JSON 文法允許將 JSON 文件模型化為節點加上標籤的樹狀結構，這是由 Azure Cosmos DB 的自動編製索引技術及 Azure Cosmos DB 的 SQL 查詢方言所使用。 如需使用 SQL 文法的詳細資訊，請參閱 [SQL 查詢][query]一文。

### <a name="does-the-sql-api-support-sql-aggregation-functions"></a>SQL API 支援 SQL 彙總函式嗎？
SQL API 透過 SQL 文法，藉由彙總函式 `COUNT`、`MIN`、`MAX`、`AVG` 和 `SUM` 來支援任何規模的低延遲彙總。 如需詳細資訊，請參閱[彙總函式](sql-api-sql-query.md#Aggregates)。

### <a name="how-does-the-sql-api-provide-concurrency"></a>SQL API 如何提供並行存取？
SQL API 透過 HTTP 實體標記或 ETag，支援開放式並行存取控制 (OCC)。 每個 SQL API 資源都有一個 ETag，並且會在每次更新文件時，於伺服器上設定此 ETag。 ETag 標頭和目前的值會包含於所有回應訊息中。 Etag 可與 If-Match 標頭搭配使用，讓伺服器能夠決定是否應該更新資源。 If-Match 值是要經過檢查的 ETag 值。 如果 ETag 值符合伺服器的 ETag 值，就會更新資源。 如果 ETag 不再是最新狀態，則伺服器會拒絕該作業，並提供「HTTP 412 前置條件失敗」回應碼。 用戶端接著會重新擷取資源，以取得該資源目前的 ETag 值。 此外，ETag 可以與 If-None-Match 標頭搭配使用，來判斷是否需要重新擷取資源。

若要在 .NET 中使用開放式並行存取，可使用 [AccessCondition](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.accesscondition.aspx) 類別。 如需 .NET 範例，請參閱 GitHub 上 DocumentManagement 範例中的 [Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/DocumentManagement/Program.cs)。

### <a name="how-do-i-perform-transactions-in-the-sql-api"></a>我如何在 SQL API 中執行交易？
SQL API 透過 JavaScript 預存程序和觸發程序，支援語言整合式交易。 指令碼內的所有資料庫作業都會在快照隔離的情況下執行。 如果是單一資料分割集合，執行範圍將限制為該集合。 如果集合已經過資料分割，執行範圍將限制為集合內具有相同資料分割索引鍵值的文件。 文件版本 (ETag) 的快照是在交易開始時取得，且只有當指令碼成功執行時才會認可。 如果 JavaScript 擲回錯誤，則會回復交易。 如需詳細資訊，請參閱 [Azure Cosmos DB 的伺服器端 JavaScript 程式設計](programming.md)。

### <a name="how-can-i-bulk-insert-documents-into-cosmos-db"></a>如何將大量文件插入 Cosmos DB？
您可以利用下列任一方式將大量文件插入 Azure Cosmos DB：

* 大量執行程式工具，如[使用大量執行程式工具 .NET 程式庫](bulk-executor-dot-net.md)和[使用大量執行程式工具 Java 程式庫](bulk-executor-java.md)中所述
* 資料移轉工具，如 [Azure Cosmos DB 的資料庫移轉工具](import-data.md)中所述。
* 預存程序，如 [Azure Cosmos DB 的伺服器端 JavaScript 程式設計](programming.md)中所述。

### <a name="i-have-setup-my-container-to-use-lazy-indexing-i-see-that-my-queries-do-not-return-expected-results"></a>我已將容器設定為使用延遲索引，但我發現查詢不會傳回預期的結果。 
如＜索引＞一節所述，延遲索引可能會導致此行為。 請一律對所有應用程式使用一致的索引。 


### <a name="does-the-sql-api-support-resource-link-caching"></a>SQL API 支援資源連結快取嗎？
是，因為 Azure Cosmos DB 是一項 RESTful 服務，資源連結是固定不變且可快取的。 SQL API 用戶端可以指定「If-None-Match」標頭來讀取任何類似資源的文件或集合，然後在伺服器版本變更後更新其本機複本。

### <a name="is-a-local-instance-of-sql-api-available"></a>SQL API 的本機執行個體可供使用嗎？
是。 [Azure Cosmos DB 模擬器](local-emulator.md)提供 Cosmos DB 服務的高逼真度模擬。 它支援與 Azure Cosmos DB 完全相同的功能，包括支援建立和查詢 JSON 文件、佈建和擴充集合，以及執行預存程序和觸發程序。 您可以使用 Azure Cosmos DB 模擬器來開發及測試應用程式，並且只需對 Azure Cosmos DB 的連接端點進行單一組態變更，就能將它們部署至全球規模的 Azure。

### <a name="why-are-long-floating-point-values-in-a-document-rounded-when-viewed-from-data-explorer-in-the-portal"></a>在入口網站中從資料總管檢視文件中的長浮點值時，這些值為何會四捨五入。 
這是 JavaScript 的限制。 JavaScript 會使用 IEEE 754 中指定的雙精確度浮點格式數值，且只能夠安全地表示 -(253 - 1) 與 253-1 (即 9007199254740991) 之間的數值。

### <a name="where-are-permissions-allowed-in-the-object-hierarchy"></a>物件階層中允許哪些權限？

您可以在容器層級和其子系 (例如文件、附件) 使用 ResourceTokens 來建立權限。 這意味著目前不允許在資料庫或帳戶層級嘗試建立權限。


## <a name="mongodb-api"></a>MongoDB API
### <a name="what-is-the-azure-cosmos-db-api-for-mongodb"></a>什麼是適用於 MongoDB 的 Azure Cosmos DB API？
適用於 MongoDB 的 Azure Cosmos DB API 是一個相容性層級，可讓應用程式使用社群支援的現有 Apache MongoDB API 和驅動程式，輕鬆且透明地與原生 Azure Cosmos DB 資料庫引擎通訊。 開發人員現在可以使用現有的 MongoDB 工具鏈結和技術，建置充分利用 Azure Cosmos DB 的應用程式。 開發人員能從 Azure Cosmos DB 的獨特功能受益，包括自動編製索引、備份維護、有金錢補償的服務等級協定 (SLA) 等。

### <a name="how-do-i-connect-to-my-api-for-mongodb-database"></a>如何連線我的 API for MongoDB 資料庫？
直接前往 [Azure 入口網站](https://portal.azure.com)，是連接適用於 MongoDB 之 Azure Cosmos DB API 最快的方式。 移至您的帳戶，然後在左側瀏覽功能表中按一下 [快速入門]。 [快速入門] 是取得程式碼片段以連接資料庫的最佳方式。 

Azure Cosmos DB 會強制執行嚴格的安全性需求和標準。 Azure Cosmos DB 帳戶需要驗證和透過 SSL 的安全通訊，因此請務必使用 TLSv1.2。

如需詳細資訊，請參閱[連線到 API for MongoDB 資料庫](connect-mongodb-account.md)。

### <a name="are-there-additional-error-codes-for-an-api-for-mongodb-database"></a>API for MongoDB 資料庫是否有其他錯誤碼？
除了常見的 MongoDB 錯誤碼，MongoDB API 有自己專用的錯誤碼：


| Error               | 代碼  | 說明  | 解決方法  |
|---------------------|-------|--------------|-----------|
| TooManyRequests     | 16500 | 取用的要求單位總數已超過針對集合佈建的要求單位率並已進行節流。 | 請考慮從 Azure 入口網站調整指派給容器或容器集的輸送量，或重試一次。 |
| ExceededMemoryLimit | 16501 | 做為多租用戶服務，作業已超出用戶端的記憶體配額。 | 透過更嚴格的查詢準則來縮小作業的範圍，或經由 [Azure 入口網站](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)連絡支援人員。 <br><br>範例：*&nbsp;&nbsp;&nbsp;&nbsp;db.getCollection('users').aggregate([<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$match: {name: "Andy"}}, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$sort: {age: -1}}<br>&nbsp;&nbsp;&nbsp;&nbsp;])*) |

### <a name="is-the-simba-driver-for-mongodb-supported-for-use-with-azure-cosmosdb-mongodb-api"></a>適用於 MongoDB 的 Simba 驅動程式是否支援與 Azure CosmosDB MongoDB API 搭配使用？
是的，您可以使用 Simba 的 Mongo ODBC 驅動程式搭配 Azure CosmosDB MongoDB API


## <a id="table"></a>資料表 API

### <a name="how-can-i-use-the-table-api-offering"></a>如何使用資料表 API 供應項目？ 
[Azure 入口網站][azure-portal]中已提供 Azure Cosmos DB 資料表 API。 首先，您必須註冊 Azure 訂用帳戶。 註冊之後，您可以將 Azure Cosmos DB 資料表 API 帳戶加入 Azure 訂用帳戶，接著再將資料表加入帳戶。 

您可以在 [Azure Cosmos DB 資料表 API 簡介](table-introduction.md)中找到支援的語言和相關快速入門。

### <a name="do-i-need-a-new-sdk-to-use-the-table-api"></a>我需要新的 SDK，才能使用資料表 API 嗎？ 
不必，現有的儲存體 SDK 應該還是有效。 不過，建議您隨時取得最新的 SDK 以獲得最佳支援，以及在諸多情況下獲得更優異的效能。 請參閱 [Azure Cosmos DB 資料表 API 簡介](table-introduction.md)中的可用語言清單。

### <a name="where-is-table-api-not-identical-with-azure-table-storage-behavior"></a>資料表 API 與 Azure 資料表儲存體的行為有哪些不同？
來自 Azure 資料表儲存體而想要使用 Azure Cosmos DB 資料表 API 來建立資料表的使用者，應該注意以下的部分行為差異：

* Azure Cosmos DB 資料表 API 使用保留容量模型來確保效能獲得保證，但這也表示，使用者一建立資料表，就必須支付其容量費用，即使該容量並未使用也是如此。 而在使用 Azure 資料表儲存體時，使用者只需要就實際使用的容量付費。 此一行為有助於解釋，為何資料表 API 可於第 99 個百分位數的水準提供 10 毫秒的讀取和 15 毫秒的寫入 SLA，而 Azure 資料表儲存體則是提供 10 秒的 SLA。 但也因為這樣，資料表 API 的資料表 (甚至是沒有任何要求的空白資料表) 必須付出金錢，才能確保有容量可用來處理其所收到的任何要求，並以 Azure Cosmos DB 所提供的 SLA 來進行。
* 資料表 API 所傳回的查詢結果，不會和 Azure 資料表儲存體一樣使用資料分割索引鍵/資料列索引鍵順序來排序。
* 資料列索引鍵最多只能是 255 個位元組
* 批次最多只可包含 2 MB
* 目前不支援 CORS
* 資料表名稱在 Azure 資料表儲存體中不區分大小寫，但在 Azure Cosmos DB 資料表 API 中則有區分
* 部分 Azure Cosmos DB 用於編碼資訊 (例如二進位欄位) 的內部格式，目前不如預期地有效率。 因此這會對資料大小造成非預期的限制。 例如，目前無法使用資料表實體的完整 1 Meg 來儲存二進位資料，因為編碼方式增加了資料的大小。
* 目前不支援實體屬性名稱 'Id'
* TableQuery TakeCount 未限制為 1000

就 REST API 來說，Azure Cosmos DB 資料表 API 並不支援某些端點/查詢選項：
| REST 方法 | REST 端點/查詢選項 | 文件 URL | 說明 |
| ------------| ------------- | ---------- | ----------- |
| GET、PUT | /?restype=service@comp=properties| [設定資料表服務屬性](https://docs.microsoft.com/rest/api/storageservices/set-table-service-properties)和[取得資料表服務屬性](https://docs.microsoft.com/rest/api/storageservices/get-table-service-properties) | 此端點可用來設定 CORS 規則、儲存體分析組態以及記錄設定。 CORS 目前不受支援，而且分析和記錄在 Azure Cosmos DB 和 Azure 儲存體資料表中的處理方式不同 |
| OPTIONS | /<table-resource-name> | [事前 CORS 資料表要求](https://docs.microsoft.com/rest/api/storageservices/preflight-table-request) | 這是 CORS 的一部分，但目前不受 Azure Cosmos DB 支援。 |
| GET | /?restype=service@comp=stats | [取得資料表服務統計資料](https://docs.microsoft.com/rest/api/storageservices/get-table-service-stats) | 提供主要與次要資料庫之間的資料複寫速度有多快的資訊。 因為複寫是寫入的一部分，因此這不是 Cosmos DB 中的必須作業。 |
| GET、PUT | /mytable?comp=acl | [取得資料表 ACL](https://docs.microsoft.com/rest/api/storageservices/get-table-acl) 和[設定資料表 ACL](https://docs.microsoft.com/rest/api/storageservices/set-table-acl) | 這會取得並設定用來管理共用存取簽章 (SAS) 的預存存取原則。 雖然 SAS 有受到支援，但其設定和管理方式並不相同。 |

此外，Azure Cosmos DB 資料表 API 只支援 JSON 格式，而不支援 ATOM。

雖然 Azure Cosmos DB 支援共用存取簽章 (SAS)，但它也有某些不支援的原則，具體地說，它不支援與管理作業有關的原則，例如用以建立新資料表的權限。

.NET SDK 尤其如此，裡面會有一些 Azure Cosmos DB 目前不支援的類別和方法。

| 類別 | 不支援的方法 |
|-------|-------- |
| CloudTableClient | \*ServiceProperties* |
|                  | \*ServiceStats* |
| CloudTable | SetPermissions* |
|            | GetPermissions* |
| TableServiceContext | * (此類別實際上已被取代) |
| TableServiceEntity | " " |
| TableServiceExtensions | " " |
| TableServiceQuery | " " |

如果上述差異為您的專案帶來問題，請連絡 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com)，並讓我們知道。

### <a name="how-do-i-provide-feedback-about-the-sdk-or-bugs"></a>如何提供關於 SDK 和 Bug 的意見反應？
您可以使用以下任何方式來分享意見反應：

* [Uservoice](https://feedback.azure.com/forums/599062-azure-cosmos-db-table-api)
* [MSDN 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurecosmosdb)
* [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-cosmosdb)。 Stack Overflow 最適合用於詢問程式設計問題。 確保您的問題[契合主題](https://stackoverflow.com/help/on-topic)且[盡可能提供愈多的詳細資料，讓問題更加清楚並可得到答覆](https://stackoverflow.com/help/how-to-ask)。

### <a name="what-is-the-connection-string-that-i-need-to-use-to-connect-to-the-table-api"></a>我需要哪一個連接字串，才能連接資料表 API？
連接字串為：
```
DefaultEndpointsProtocol=https;AccountName=<AccountNamefromCosmos DB;AccountKey=<FromKeysPaneofCosmosDB>;TableEndpoint=https://<AccountName>.table.cosmosdb.azure.com
```
您可以從 Azure 入口網站的 [連接字串] 頁面取得連接字串。 

### <a name="how-do-i-override-the-config-settings-for-the-request-options-in-the-net-sdk-for-the-table-api"></a>如何在資料表 API 的 .NET SDK 中覆寫要求選項的組態設定？
如需組態設定的相關資訊，請參閱 [Azure Cosmos DB 功能](../cosmos-db/tutorial-develop-table-dotnet.md#azure-cosmos-db-capabilities)。 有些設定是在 CreateCloudTableClient 方法上處理，有些則是透過用戶端應用程式中 appSettings 區段的 app.config 進行處理。

### <a name="are-there-any-changes-for-customers-who-are-using-the-existing-azure-table-storage-sdks"></a>對使用現有「Azure 資料表」儲存體 SDK 的客戶來說，是否有任何變更？
無。 對使用現有「Azure 資料表」儲存體 SDK 的現有客戶或新客戶來說，並無任何變更。 

### <a name="how-do-i-view-table-data-that-is-stored-in-azure-cosmos-db-for-use-with-the-table-api"></a>如何檢視儲存在 Azure Cosmos DB 中的資料表 資料，以便與資料表 API 搭配使用？ 
您可以使用 Azure 入口網站來瀏覽資料。 您也可以使用資料表 API 程式碼或下一個回答所述的工具。 

### <a name="which-tools-work-with-the-table-api"></a>哪些工具可與資料表 API 搭配使用？ 
您可以使用 [Azure 儲存體總管](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer)。

能夠彈性地以先前指定之格式採用連接字串的工具，均可支援新的資料表 API。 [Azure 儲存體用戶端工具](../storage/common/storage-explorers.md)頁面上提供一份資料表工具清單。 

### <a name="is-the-concurrency-on-operations-controlled"></a>可以在作業中控制並行存取嗎？
是，開放式並行存取會透過使用 Etag 機制來提供。 

### <a name="is-the-odata-query-model-supported-for-entities"></a>是否針對實體支援 OData 查詢模型？ 
是，資料表 API 支援 OData 查詢和 LINQ 查詢。 

### <a name="can-i-connect-to-azure-table-storage-and-azure-cosmos-db-table-api-side-by-side-in-the-same-application"></a>我可以在同一個應用程式中並行連線至「Azure 資料表」儲存體和「Azure Cosmos DB 資料表 API」嗎？ 
是，您可以建立兩個不同的 CloudTableClient 執行個體，並透過連接字串讓它們分別指向自己的 URI，藉此進行連線。

### <a name="how-do-i-migrate-an-existing-azure-table-storage-application-to-this-offering"></a>如何將現有的 Azure 資料表儲存體應用程式移轉到這個供應項目？
[AzCopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy) 和 [Azure Cosmos DB 資料移轉工具](import-data.md)皆有受到支援。

### <a name="how-is-expansion-of-the-storage-size-done-for-this-service-if-for-example-i-start-with-n-gb-of-data-and-my-data-will-grow-to-1-tb-over-time"></a>假設我一開始就有 n GB 的資料，而我的資料將隨著時間過去成長為 1 TB，我該如何針對此服務完成儲存體大小的擴充？ 
Azure Cosmos DB 的設計乃藉由使用水平調整提供無限制的儲存體。 這項服務能監視並有效地增加您的儲存體。 

### <a name="how-do-i-monitor-the-table-api-offering"></a>如何監視資料表 API 供應項目？
您可以使用資料表 API 的 [計量] 窗格，來監視要求和儲存體使用量。 

### <a name="how-do-i-calculate-the-throughput-i-require"></a>如何計算我需要的輸送量？
您可以使用的容量估計工具來計算作業所需的 TableThroughput。 如需詳細資訊，請參閱[估算要求單位和資料儲存體 (英文)](https://www.documentdb.com/capacityplanner)。 通常，您可以將實體表示為 JSON，並為您的作業提供編號。 

### <a name="can-i-use-the-table-api-sdk-locally-with-the-emulator"></a>我可以在本機搭配使用資料表 API SDK 與模擬器嗎？
目前沒有。

### <a name="can-my-existing-application-work-with-the-table-api"></a>我現有的應用程式可以與資料表 API 搭配運作嗎？ 
是，可支援相同的 API。

### <a name="do-i-need-to-migrate-my-existing-azure-table-storage-applications-to-the-sdk-if-i-do-not-want-to-use-the-table-api-features"></a>如果我不想要使用「資料表 API」功能，是否需要將現有的「Azure 資料表」儲存體應用程式移轉至 SDK？
否，您可以建立和使用現有的「Azure 資料表」儲存體資產，而不會有任何形式的服務中斷。 不過，如果您不使用資料表 API ，就無法從自動索引、額外的一致性選項或全域散發中獲益。 

### <a name="how-do-i-add-replication-of-the-data-in-the-table-api-across-multiple-regions-of-azure"></a>如何在「資料表 API」中新增跨多個 Azure 區域的資料複寫？
您可以使用 Azure Cosmos DB 入口網站的[全域複寫設定](tutorial-global-distribution-sql-api.md#portal)，加入適合應用程式的區域。 若要開發全域散發的應用程式，您還應該使用設為本地區域的 PreferredLocation 資訊來加入您的應用程式，以提供低讀取延遲。 

### <a name="how-do-i-change-the-primary-write-region-for-the-account-in-the-table-api"></a>如何在「資料表 API」中變更帳戶的主要寫入區域？
您可以使用 Azure Cosmos DB 的全球複寫入口網站窗格來加入一個區域，然後容錯移轉至所需的區域。 如需指示，請參閱[使用多區域 Azure Cosmos DB 帳戶進行開發](regional-failover.md)。 

### <a name="how-do-i-configure-my-preferred-read-regions-for-low-latency-when-i-distribute-my-data"></a>當我散發資料時，如何設定慣用的讀取區域以取得低延遲？ 
若要協助讀取本機位置，請使用 app.config 檔案中的 PreferredLocation 金鑰 。 對於現有的應用程式，如果您已設定 LocationMode，資料表 API 會擲回錯誤。 請移除該程式碼，因為資料表 API 會從 app.config 檔案取用這項資訊。 如需詳細資訊，請參閱 [Azure Cosmos DB 功能](../cosmos-db/tutorial-develop-table-dotnet.md#azure-cosmos-db-capabilities)。

### <a name="how-should-i-think-about-consistency-levels-in-the-table-api"></a>如何看待資料表 API 中的一致性層級？ 
Azure Cosmos DB 可在一致性、高可用性及延遲之間提供完全合乎邏輯的取捨。 Azure Cosmos DB 資料表 API 提供開發人員五種一致性層級，所以您可以在資料表層級選擇合適的一致性模型，以及在查詢資料時進行個別要求。 當用戶端連線時，它可以指定一致性層級。 您可以透過 CreateCloudTableClient 的 consistencyLevel 引數來變更層級。 

資料表 API 會利用「讀取您自己的寫入」來提供低延遲的讀取，並採用限定過期一致性的預設值。 如需詳細資訊，請參閱[一致性層級](consistency-levels.md)。 

根據預設，Azure 資料表儲存體提供區域內的強式一致性，以及次要位置中最終的一致性。 

### <a name="does-azure-cosmos-db-table-api-offer-more-consistency-levels-than-azure-table-storage"></a>「Azure Cosmos DB 資料表 API」是否提供比「Azure 資料表」更多的一致性層級？
是，如需如何從 Azure Cosmos DB 分散式本質獲益的相關資訊，請參閱[一致性層級](consistency-levels.md)。 由於我們提供一致性層級保證，因此您可以放心使用。 如需詳細資訊，請參閱 [Azure Cosmos DB 功能](../cosmos-db/tutorial-develop-table-dotnet.md#azure-cosmos-db-capabilities)。

### <a name="when-global-distribution-is-enabled-how-long-does-it-take-to-replicate-the-data"></a>啟用全域散發時，複寫資料需要多久的時間？
Azure Cosmos DB 會在當地區域持久認可資料，並在幾毫秒內立即將資料推送到其他區域。 這項複寫操作只取決於資料中心的來回時間 (RTT)。 若要深入了解 Azure Cosmos DB 的全域散發能力，請參閱 [Azure Cosmos DB︰Azure 上分散在世界各地的資料庫服務](distribute-data-globally.md)。

### <a name="can-the-read-request-consistency-level-be-changed"></a>可以變更讀取要求一致性層級嗎？
有了 Azure Cosmos DB，您可以設定容器層級的一致性層級 (在資料表中)。 透過 .NET SDK，您可以在 app.config 檔案中提供 TableConsistencyLevel 索引鍵的值，藉以變更層級。 可能的值包括：強式、限定過期、工作階段、一致前置詞或最終。 如需詳細資訊，請參閱 [Azure Cosmos DB 中的 Tunable 資料一致性層級](consistency-levels.md)。 主要概念在於，您無法將要求一致性層級設定為高於資料表設定的值。 例如，您無法將資料表的一致性層級設定為「最終」，而將要求一致性層級設定為「強式」。 

### <a name="how-does-the-table-api-handle-failover-if-a-region-goes-down"></a>當區域發生服務中斷時，「資料表 API」如何處理容錯移轉？ 
資料表 API 會利用 Azure Cosmos DB 的全域散發平台。 若要確保您的應用程式可以容許資料中心停機，請在 Azure Cosmos DB 入口網站中針對該帳戶至少啟用一個以上的區域，[使用多區域 Azure Cosmos DB 帳戶進行開發](regional-failover.md)。 您可以使用入口網站來設定區域的優先順序，[使用多區域 Azure Cosmos DB 帳戶進行開發](regional-failover.md)。 

您可以針對帳戶加入任意數目的區域，並藉由提供容錯移轉優先順序來控制容錯移轉的目標位置。 當然，若要使用的資料庫，您還需要在目標位置提供應用程式。 如果您這樣做，您的客戶就不會經歷停機時間。 [最新的 .NET 用戶端 SDK](table-sdk-dotnet.md) 會自動連接，但其他 SDK 則不會。 也就是說，它可以偵測到區域已關閉，並自動容錯移轉至新的區域。

### <a name="is-the-table-api-enabled-for-backups"></a>是否已啟用「資料表 API」的備份功能？
是，資料表 API 會利用 Azure Cosmos DB 的平台來進行備份。 備份會自動執行。 如需詳細資訊，請參閱[使用 Azure Cosmos DB 進行線上備份及還原](online-backup-and-restore.md)。

 
### <a name="does-the-table-api-index-all-attributes-of-an-entity-by-default"></a>資料表 API 預設會為實體的所有屬性編製索引嗎？
是，根據預設，它會為實體的所有屬性編製索引。 如需詳細資訊，請參閱 [Azure Cosmos DB：索引編製原則](indexing-policies.md)。 

### <a name="does-this-mean-i-do-not-have-to-create-multiple-indexes-to-satisfy-the-queries"></a>這表示我不需要建立多個索引來滿足查詢嗎？ 
是，Azure Cosmos DB 資料表 API 會為所有屬性提供自動編製索引，而不需任何結構描述定義。 這項資訊能讓開發人員專注於應用程式，而不需擔心索引的建立和管理。 如需詳細資訊，請參閱 [Azure Cosmos DB：索引編製原則](indexing-policies.md)。

### <a name="can-i-change-the-indexing-policy"></a>我可以變更索引編制原則嗎？
是，您可以提供索引定義來變更索引編制原則。 如需詳細資訊，請參閱 [Azure Cosmos DB 功能](../cosmos-db/tutorial-develop-table-dotnet.md#azure-cosmos-db-capabilities)。 您需要適當地進行編碼和逸出這些設定。 

針對非 .NET SDK，索引編製原則只能在入口網站的**資料總管**中進行設定，請瀏覽至您想要變更的特定資料表，然後移至 [規模與設定] -> [索引編製原則]，進行所需的變更，然後 [儲存]。

透過 .NET SDK，您可在 app.config 檔案中提交原則：
```
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/somepath",
      "indexes": [
        {
          "kind": "Range",
          "dataType": "Number",
          "precision": -1
        },
        {
          "kind": "Range",
          "dataType": "String",
          "precision": -1
        } 
      ]
    }
  ],
  "excludedPaths": 
[
 {
      "path": "/anotherpath"
 }
]
}
```

### <a name="azure-cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-such-as-sorting-aggregates-hierarchy-and-other-functionality-will-you-be-adding-these-capabilities-to-the-table-api"></a>Azure Cosmos DB 做為平台時似乎還有很多功能，例如排序、彙總、階層及其他功能。 你們會將這些功能加入資料表 API 嗎？ 
資料表 API 提供的查詢功能與 Azure 資料表儲存體相同。 Azure Cosmos DB 也支援排序、彙總、地理空間查詢、階層及各種不同的內建函式。 在未來的服務更新中，我們會在資料表 API 內提供額外功能。 如需詳細資訊，請參閱 [SQL 查詢](sql-api-sql-query.md)。
 
### <a name="when-should-i-change-tablethroughput-for-the-table-api"></a>我何時應該變更資料表 API 的 TableThroughput？
滿足以下任一條件時，您應該變更 TableThroughput：
* 您即將要執行資料擷取、轉換及載入 (ETL)，或想要在短時間內上傳大量資料。 
* 您需要更多來自後端容器或容器集的輸送量。 例如，您發現使用的輸送量比佈建的輸送量還多，而您正要開始進行節流。 如需詳細資訊，請參閱[設定 Azure Cosmos DB 容器輸送量](set-throughput.md)。

### <a name="can-i-scale-up-or-scale-down-the-throughput-of-my-table-api-table"></a>我可以相應增加或減少資料表 API 資料表的輸送量嗎？ 
是，您可以使用 Azure Cosmos DB 入口網站的調整窗格來調整輸送量。 如需詳細資訊，請參閱[設定輸送量](set-throughput.md)。

### <a name="is-a-default-tablethroughput-set-for-newly-provisioned-tables"></a>新佈建的資料表是否有預設的 TableThroughput 設定？
是，如果您不透過 app.config 覆寫 TableThroughput，也不在 Azure Cosmos DB 中使用預先建立的容器，服務會建立輸送量為 400 的資料表。
 
### <a name="is-there-any-change-of-pricing-for-existing-customers-of-the-azure-table-storage-service"></a>對現有的「Azure 資料表」儲存體服務客戶來說，是否有任何定價變更？
無。 對現有的「Azure 資料表」儲存體客戶來說，並沒有任何價格上的變更。 

### <a name="how-is-the-price-calculated-for-the-table-api"></a>如何針對資料表 API 計算價格？ 
價格取決於配置的 TableThroughput。 

### <a name="how-do-i-handle-any-rate-limiting-on-the-tables-in-table-api-offering"></a>如何在資料表 API 供應項目中處理資料表上的任何速率限制？ 
如果要求速率超過為基礎容器或容器集佈建的輸送量容量，您就會收到錯誤，而 SDK 會套用重試原則來重試呼叫。

### <a name="why-do-i-need-to-choose-a-throughput-apart-from-partitionkey-and-rowkey-to-take-advantage-of-the-table-api-offering-of-azure-cosmos-db"></a>為何我需要選擇 PartitionKey 和 RowKey 以外的輸送量，才能運用 Azure Cosmos DB 的資料表 API 供應項目？
如果您未在 app.config 或透過入口網站提供輸送量，Azure Cosmos DB 將為您的容器設定預設輸送量。 

Azure Cosmos DB 能提供效能和延遲上的保證，限定作業時的上限。 當引擎能針對租用戶作業強制執行控管時，這項保證是可行的。 設定 TableThroughput 能確保您享有保證的輸送量和延遲，因為平台會保留產能，保證作業成功。 

透過輸送量規格，您可以彈性地變更它來從應用程式季節性當中獲得好處、滿足輸送量需求和節省成本。

### <a name="azure-table-storage-has-been-very-inexpensive-for-me-because-i-pay-only-to-store-the-data-and-i-rarely-query-the-azure-cosmos-db-table-api-offering-seems-to-be-charging-me-even-though-i-have-not-performed-a-single-transaction-or-stored-anything-can-you-please-explain"></a>Azure 資料表儲存體對我而言非常便宜，因為我只需要支付資料的儲存費用，而且我很少查詢。 即使我尚未執行單一交易或儲存任何項目，Azure Cosmos DB 資料表 API 供應項目似乎還是會向我收取費用。 可以請您說明嗎？

Azure Cosmos DB 已設計為全域散發的 SLA 型系統，並提供可用性、延遲和輸送量的保證。 當您在 Azure Cosmos DB 中保留輸送量時，與其他系統輸送量不同的是，它會提供保證。 Azure Cosmos DB 會提供客戶要求的其他功能，如次要索引和全域散發。  

### <a name="i-never-get-a-quota-full-notification-indicating-that-a-partition-is-full-when-i-ingest-data-into-azure-table-storage-with-the-table-api-i-do-get-this-message-is-this-offering-limiting-me-and-forcing-me-to-change-my-existing-application"></a>將資料內嵌至「Azure 資料表」儲存體時，我從未收到「配額已滿」通知 (表示分割區已滿)。 當我使用資料表 API 時，卻收到了這則訊息。 這個供應項目會限制我，並強迫我變更我現有的應用程式嗎？

Azure Cosmos DB 是 SLA 型系統，它提供無限制的調整和延遲、輸送量、可用性、一致性保證。 若要獲得保證的進階效能，您必須確定資料大小和索引是可管理且可調整的。 每個資料分割索引鍵的實體或項目數目限制為 10 GB，這是為了確保我們能夠提供絕佳的查閱和查詢效能。 為了確保您的應用程式能夠針對 Azure 儲存體進行更佳的調整，建議您「避免」建立常用資料分割 (將所有資訊儲存在一個資料分割，然後進行查詢)。 

### <a name="so-partitionkey-and-rowkey-are-still-required-with-the-table-api"></a>這樣說來，資料表 API 仍然需要 PartitionKey 和 RowKey 嗎？ 
是。 因為「資料表 API」的介面區與「Azure 資料表」儲存體 SDK 的介面區類似，所以分割區索引鍵能提供有效率的資料散發方式。 資料列索引鍵在該資料分割內是唯一的。 資料列索引鍵必須存在且不能為 Null，如同在標準 SDK 中。 RowKey 的長度是 255 個位元組，而 PartitionKey 為 1 KB。 

### <a name="what-are-the-error-messages-for-the-table-api"></a>資料表 API 的錯誤訊息將有哪些？
Azure 資料表儲存體和 Azure Cosmos DB 資料表 API 使用相同的 SDK，因此大部分的錯誤會相同。

### <a name="why-do-i-get-throttled-when-i-try-to-create-lot-of-tables-one-after-another-in-the-table-api"></a>當我嘗試在資料表 API 中逐一建立許多資料表時，為什麼要進行節流？
Azure Cosmos DB 是 SLA 型系統，可提供延遲、輸送量、可用性及一致性的保證。 由於它是佈建的系統，因此會保留資源以保證這些需求。 系統會偵測到快速建立的資料表，並加以節流。 我們建議您查看資料表的建立速率，並降低為低於每分鐘 5 個。 請記住，資料表 API 是佈建的系統。 只要您一佈建，就必須開始支付費用。 

## <a name="gremlin-api"></a>Gremlin API

### <a name="for-cnet-development-should-i-use-the-microsoftazuregraphs-package-or-gremlinnet"></a>若是使用 C#/.NET 開發，應該使用 Microsoft.Azure.Graphs 套件或 Gremlin.NET？ 

Azure Cosmos DB Gremlin API 利用開放原始碼驅動程式作為服務的主要連接器。 因此，建議使用受 [ Apache Tinkerpop 支援的驅動程式](http://tinkerpop.apache.org/)。

### <a name="how-are-rus-charged-when-running-queries-on-a-graph-database"></a>在圖形資料庫中執行查詢時，每秒的 RU 如何計費？ 

包括頂點及邊線的所有圖形物件，都會以 JSON 文件形式於後端呈現。 由於一個 Gremlin 查詢一次可以修正一或多個圖形物件，因此與其相關的成本會與這些由查詢處理的物件及邊線有直接關係。 Azure Cosmos DB 使用相同流程處理其他所有的 API。 如需詳細資訊，請參閱 [Azure Cosmos DB 中的要求單位](request-units.md)。

RU 計費是根據周遊的運作資料庫計算，並不是根據結果集計算。 比方說，若查詢的目標是要獲得單一頂點作為結果，但需要同時周遊其他多個物件，則會依計算單一結果頂點所需的所有圖形物件來計算成本。

### <a name="whats-the-maximum-scale-that-a-graph-database-can-have-in-azure-cosmos-db-gremlin-api"></a>Azure Cosmos DB Gremlin API 中圖形資料庫所能容納的級別上限為何？ 

Azure Cosmos DB 使用[水平資料分割](partition-data.md)，自動處理儲存體和輸送量需要增加的問題。 工作負載的輸送量和儲存體容量上限根據給定集合相關的資料分割數量而定。 不過，Gremlin API 集合具備一組特定的指導方針，確保能提供良好的大規模效能體驗。 如需詳資訊和最佳做法，請參閱[資料分割最佳做法](partition-data.md#best-practices-when-choosing-a-partition-key)文件。 

### <a name="how-can-i-protect-against-injection-attacks-using-gremlin-drivers"></a>如何防止透過 Gremlin 驅動程式發動的資料隱碼攻擊？ 

多數原生 Tinkerpop Gremlin 驅動程式均允許提供參數字典執行查詢。 這裡提供使用 [Gremlin.Net](http://tinkerpop.apache.org/docs/3.2.7/reference/#gremlin-DotNet) 和 [Gremlin-Javascript](https://github.com/Azure-Samples/azure-cosmos-db-graph-nodejs-getting-started/blob/master/app.js) 兩種方法的執行範例。

### <a name="why-am-i-getting-the-gremlin-query-compilation-error-unable-to-find-any-method-error"></a>為什麼我會收到「Gremlin 查詢編譯錯誤：找不到任何方法」錯誤？

Azure Cosmos DB Gremlin API 會實作一個在 Gremlin 介面區定義的功能子集。 如需支援的步驟和詳細資訊，請參閱 [Gremlin 支援](gremlin-support.md)一文。

最好的解決方法，是以支援的功能重新寫入必要的 Gremlin 步驟，因為 Azure Cosmos DB 支援所有必要的 Gremlin 步驟。

### <a name="why-am-i-getting-the-websocketexception-the-server-returned-status-code-200-when-status-code-101-was-expected-error"></a>為什麼我會收到「WebSocketException：伺服器在預期傳回狀態碼 ’101’ 的情況下傳回了狀態碼 '200'」錯誤？

系統會在使用了錯誤的端點時擲回此錯誤。 產生此錯誤的端點模式如下：

`https:// YOUR_DATABASE_ACCOUNT.documents.azure.com:443/` 

這是供您圖形資料庫使用的文件端點。  正確的端點是 Gremlin Endpoint，其格式如下： 

`https://YOUR_DATABASE_ACCOUNT.gremlin.cosmosdb.azure.com:443/`

### <a name="why-am-i-getting-the-requestrateistoolarge-error"></a>為什麼我會收到「RequestRateIsTooLarge」錯誤？

此錯誤表示，已配置的每秒要求單位不足以提供查詢之所用。 通常您會在執行會獲得所有頂點的查詢時看到此錯誤：

```
// Query example:
g.V()
```

此查詢會嘗試從圖形中擷取所有頂點。 因此，此查詢的成本至少等於 RU 的頂點數目。 您應該調整每秒 RU 的設定，才能處理此查詢。

### <a name="why-do-my-gremlin-driver-connections-get-dropped-eventually"></a>為什麼我的 Gremlin 驅動程式連線最後還是失敗？

Gremlin 連線是透過 WebSocket 連線而產生的。 雖然 WebSocket 連線並沒有特定的存留時間，Azure Cosmos DB Gremlin API 仍然會在 30 分鐘的非使用狀態之後終止閒置的連線。 

### <a name="why-cant-i-use-fluent-api-calls-in-the-native-gremlin-drivers"></a>為什麼不能在原生 Gremlin 驅動程式中使用 Fluent API 呼叫？

Azure Cosmos DB Gremlin API 尚未支援 Fluent API 呼叫。 Fluent API 呼叫需要稱為位元組程式碼支援的內部格式化功能，目前 Azure Cosmos DB Gremlin API 不支援此功能。 由於相同的原因，最新的 Gremlin-JavaScript 驅動程式目前也不受支援。 

### <a name="how-can-i-evaluate-the-efficiency-of-my-gremlin-queries"></a>如何評估 Gremlin 查詢的效率？

**executionProfile()** 預覽步驟可提供查詢執行計畫的分析。 您需要將此步驟新增至任何 Gremlin 查詢的結尾，如以下範例所示：

**查詢範例**

```
g.V('mary').out('knows').executionProfile()
```

**範例輸出**

```json
[
  {
    "gremlin": "g.V('mary').out('knows').executionProfile()",
    "totalTime": 8,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 3,
        "annotations": {
          "percentTime": 37.5
        },
        "counts": {
          "resultCount": 1
        }
      },
      {
        "name": "GetEdges",
        "time": 5,
        "annotations": {
          "percentTime": 62.5
        },
        "counts": {
          "resultCount": 0
        },
        "storeOps": [
          {
            "partitionsAccessed": 1,
            "count": 0,
            "size": 0,
            "time": 0.6
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      }
    ]
  }
]
```

以上設定檔的輸出顯示了獲得 Vertex 和 Edge 物件需要花費的時間，及為此運作的資料集大小。 這與 Azure Cosmos DB 查詢的標準成本量值相關。

## <a id="cassandra"></a> Cassandra API

### <a name="what-is-the-protocol-version-supported-in-the-private-preview-is-there-a-plan-to-support-other-protocols"></a>私人預覽版所支援的通訊協定版本為何？ 是否打算支援其他通訊協定？
Azure Cosmos DB 的 Apache Cassandra API 目前支援 CQL 第 4 版。 如果您有關於支援其他通訊協定的意見反應，請透過 [UserVoice 意見反應](https://feedback.azure.com/forums/263030-azure-cosmos-db)讓我們知道，或傳送電子郵件到 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)。 

### <a name="why-is-choosing-a-throughput-for-a-table-a-requirement"></a>為什麼必須要為資料表選擇輸送量？
Azure Cosmos DB 會根據您是透過入口網站還是 CQL 建立資料表，為您的容器設定預設輸送量。 Azure Cosmos DB 能提供效能和延遲上的保證，限定作業時的上限。 當引擎能針對租用戶作業強制執行控管時，這項保證是可行的。 設定輸送量能確保您享有保證的輸送量和延遲，因為平台會保留產能，保證作業成功。 您可以彈性地變更輸送量，以從應用程式的季節因素獲益，並節省成本。

輸送量概念的說明請參閱 [Azure Cosmos DB 中的要求單位](request-units.md)一文。 資料表的輸送量會平均分散到基礎的實體分割區。  

### <a name="what-is-the-default-rus-of-table-when-created-through-cql-what-if-i-need-to-change-it"></a>透過 CQL 建立的資料表，其預設「RU/秒」為何？ 如果我需要變更此輸送量，該怎麼做？
Azure Cosmos DB 會使用每秒的要求單位數 (RU/秒) 做為用來提供輸送量的貨幣。 透過 CQL 所建立的資料表有 400 RU。 您可以從入口網站變更 RU。 

CQL
```
CREATE TABLE keyspaceName.tablename (user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=1200
```

.NET
```csharp
int provisionedThroughput = 400;
var simpleStatement = new SimpleStatement($"CREATE TABLE {keyspaceName}.{tableName} (user_id int PRIMARY KEY, lastname text)");
var outgoingPayload = new Dictionary<string, byte[]>();
outgoingPayload["cosmosdb_provisioned_throughput"] = Encoding.UTF8.GetBytes(provisionedThroughput.ToString());
simpleStatement.SetOutgoingPayload(outgoingPayload); 
``` 

### <a name="what-happens-when-throughput-is-exceeded"></a>超過輸送量會發生什麼事？ 
Azure Cosmos DB 能提供效能和延遲上的保證，限定作業時的上限。 當引擎能針對租用戶作業強制執行控管時，這項保證是可行的。 透過設定輸送量即可獲得保證，確保您享有保證的輸送量和延遲，因為平台會保留產能，保證作業成功。 當您超過此容量時，將會收到多載錯誤訊息，指出您已超過使用容量。 0x1001 多載：無法處理要求，因為「要求速率太大」。 在此時，請務必查看是哪些作業和其磁碟區導致此問題。 您可以透過入口網站上的計量，了解超過所佈建容量的已用容量。 然後，您必須確保您會在所有基礎分割區以近乎平均的方式取用容量。 如果您看到輸送量大多是取自某個分割區，則表示您的工作負載有所傾斜。 

我們提供的計量可讓您知道所有分割區在幾小時、幾天以及每七天的輸送量使用情形，或其彙總。 如需詳細資訊，請參閱[使用 Azure Cosmos DB 中的計量監控及偵錯](use-metrics.md)。

診斷記錄的說明請見 [Azure Cosmos DB 診斷記錄](logging.md)一文。

### <a name="does-the-primary-key-map-to-the-partition-key-concept-of-azure-cosmos-db"></a>主要索引鍵是否會對應至 Azure Cosmos DB 的資料分割索引鍵概念？
是，資料分割索引鍵可用來將實體放在正確的位置。 在 Azure Cosmos DB 中，我們會用它來尋找實體資料分割上所儲存的正確邏輯資料分割。 資料分割概念的詳細說明請見[在 Azure Cosmos DB 中進行資料分割和調整](partition-data.md)一文。 這裡的重點是，邏輯資料分割目前不應超過 10 GB 的限制。 

### <a name="what-happens-when-i-get-a-quota-full-notification-indicating-that-a-partition-is-full"></a>如果我收到「配額已滿」通知，內容指出資料分割已滿，會發生什麼事？
Azure Cosmos DB 是 SLA 型系統，它提供無限制的調整和延遲、輸送量、可用性、一致性保證。 其 Cassandra API 也允許無限制地儲存資料。 此無限制的儲存是根據使用資料分割做為重要概念的水平方式資料相應放大。 資料分割概念的詳細說明請見[在 Azure Cosmos DB 中進行資料分割和調整](partition-data.md)一文。

您應該遵守實體數目或每個邏輯資料分割之項目數的 10 GB 限制。 為了確保您的應用程式能夠進行適當的調整，建議您「避免」建立常用資料分割 (將所有資訊儲存在一個資料分割，然後進行查詢)。 當您的資料分佈有所傾斜 (也就是說，如果您在一個資料分割索引鍵放了大量資料，亦即超過 10 GB) 時才會發生這個錯誤。 您可以使用儲存體入口網站來找到資料的分佈。 若要修正此錯誤，方法是重新建立資料表，然後選擇細微的主要資料分割索引鍵，以讓資料分佈情形更加適當。

### <a name="is-it-possible-to-use-cassandra-api-as-key-value-store-with-millions-or-billions-of-individual-partition-keys"></a>是否能將 Cassandra API 做為索引鍵值存放區，以存放數百萬或數十億個個別的資料分割索引鍵？
Azure Cosmos DB 可以透過相應放大儲存體以儲存無限量的資料。 這與輸送量無關。 是，您一律可以只使用 Cassandra API，透過指定正確的主要/資料分割索引鍵來儲存和擷取索引鍵/值。 這些個別的索引鍵都會有自己的邏輯資料分割，並且是以實體資料分割為基礎的，因此不會有問題。 

### <a name="is-it-possible-to-create-multiple-tables-with-apache-cassandra-api-of-azure-cosmos-db"></a>是否能夠使用 Azure Cosmos DB 的 Apache Cassandra API 建立多個資料表？
是，您可以使用 Apache Cassandra API 建立多個資料表。 系統會將這些資料表各視為輸送量和儲存體的單位。 

### <a name="is-it-possible-to-create-multiple-tables-in-succession"></a>是否能夠連續建立多個資料表？
Azure Cosmos DB 是適用於資料和控制平面活動的資源所管理的系統。 集合和資料表等容器是針對給定的輸送量容量所佈建的執行階段實體。 快速地連續建立這些容器並非預料中的活動，因此會受到節流。 如果您要進行會立即卸除/建立資料表的測試，請嘗試將這些資料表分開。

### <a name="what-is-maximum-number-of-tables-which-can-be-created"></a>可建立的資料表數目上限為何？
資料表數目並無實際限制，如果您有非常大量的資料表 (穩定大小總計超過 10 TB 的資料)，且一般必須以 10 個或 100 個為單位來建立，請傳送電子郵件給 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)。 

### <a name="what-is-the-maximum--of-keyspace-which-we-can-create"></a>可建立的 keyspace 數目上限為何？ 
keyspace 是中繼資料容器，因此 keyspace 數目並無實際限制，如果您因為某些原因而擁有非常大量的 keyspace，請傳送電子郵件到 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)。 

### <a name="is-it-possible-to-bring-in-lot-of-data-after-starting-from-normal-table"></a>從一般資料表開始後，有可能在其中放入大量資料嗎？ 
儲存體容量是由系統自動管理，並且會隨著您放入更多的資料而增加。 因此，您可以放心地匯入您所需的資料量，而不必管理和佈建節點等。 

### <a name="is-it-possible-to-supply-yaml-file-settings-to-configure-apache-casssandra-api-of-azure-cosmos-db-behavior"></a>是否能夠提供 yaml 檔案設定，以設定「Azure Cosmos DB 的 Apache Cassandra API」的行為？
Azure Cosmos DB 的 Apache Cassandra API 是一種平台服務。 它可提供通訊協定層級的相容性，以供執行作業。 它會讓管理、監視和設定作業變得不再複雜。 身為開發人員/使用者，您不必擔心可用性、標記、金鑰快取、資料列快取、Bloom 篩選和諸多其他設定。 Azure Cosmos DB 的 Apache Cassandra API 著重在為您提供所需的讀取和寫入效能，而不會有設定和管理的額外負荷。

### <a name="will-apache-cassandra-api-for-azure-cosmos-db-support-node-additioncluster-statusnode-status-commands"></a>Azure Cosmos DB 的 Apache Cassandra API 是否支援節點新增/叢集狀態/節點狀態命令？
Apache Cassandra API 是一種平台服務，可讓您輕鬆地規劃容量並回應輸送量和儲存體的彈性要求。 藉由 Azure Cosmos DB，您可以佈建所需的輸送量。 然後，您可以在一天之中不限次數地將此輸送量相應增加和減少，而不必擔心如何新增/刪除節點或進行管理。 這也表示您不需要使用節點、叢集管理工具。 

### <a name="what-happens-with-respect-to-various-config-settings-for-keyspace-creation-like-simplenetwork"></a>各種組態設定會對 keyspace 的建立 (例如簡單/網路) 造成什麼影響？
基於可用性和低延遲的考量，Azure Cosmos DB 預設會提供全域發佈。 您不需要設定複本等。在您的資料所寫入的區域中，所有寫入一律會永久地認可仲裁，並提供效能保證。  

### <a name="what-happens-with-respect-to-various-settings-for-table-metadata-like-bloom-filter-caching-read-repair-change-gcgrace-compression-memtableflushperiod-etc"></a>各種設定會對資料表中繼資料 (例如 Bloom 篩選、快取、讀取修復變更、gc_grace 和 compression memtable_flush_period 等) 造成什麼影響？
Azure Cosmos DB 會為讀取/寫入和輸送量提供效能，讓您不必接觸任何組態設定，也不會意外地操作到它們。  

### <a name="is-time-to-live-ttl-supported-for-cassandra-tables"></a>Cassandra 資料表是否支援存留時間 (TTL)？ 
是，有支援 TTL。 

### <a name="is-it-possible-to-monitor-node-status-replica-status-gc-and-os-parameters-earlier-with-various-tools-what-needs-to-be-monitored-now"></a>是否有辦法透過各種工具及早監視節點狀態、複本狀態、gc 和 OS 參數？ 現在需要監視哪些項目？
Azure Cosmos DB 是一種平台服務，可協助您提高產能，而不必擔心如何管理及監視基礎結構。 您只需要處理輸送量，如果您遇到節流情形，您可以在入口網站的計量中找到輸送量，並將該輸送量予以增加或減少。 監視 [SLA](monitor-accounts.md)。
使用[計量](use-metrics.md)。使用[診斷記錄](logging.md)。

### <a name="which-client-sdks-can-work-with-apache-cassandra-api-of-azure-cosmos-db"></a>哪些用戶端 SDK 可以搭配 Azure Cosmos DB 的 Apache Cassandra API 來運作？
在私人預覽版中，使用 CQLv3 的 Apache Cassandra SDK 用戶端驅動程式會用於用戶端程式。 如果您使用其他驅動程式，或如果您遇到問題，請傳送電子郵件到 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)。 

### <a name="is-composite-partition-key-supported"></a>是否支援複合資料分割索引鍵？
是，您可以使用一般語法來建立複合的資料分割索引鍵。 

### <a name="can-i-use-sstable-loader-for-data-loading"></a>是否可以使用 sstable loader 來載入資料？
否，預覽期間不支援 sstable loader。 

### <a name="can-an-on-premises-cassandra-cluster-be-paired-with-azure-cosmos-dbs-apache-cassandra-api"></a>內部部署 Cassandra 叢集是否可以與 Azure Cosmos DB 的 Apache Cassandra API 配對使用？
目前，Azure Cosmos DB 具有最佳化的雲端環境體驗，而不會產生作業負荷。 如果您需要配對，請傳送電子郵件到 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)，並說明您的案例。

### <a name="does-cassandra-api-provide-full-backups"></a>Cassandra API 是否會提供完整備份？ 
Azure Cosmos DB 會提供兩個免費的完整備份，並於當天以四個小時為間隔針對所有 API 來擷取。 這可確保您不需要設定備份排程等。如果您想要修改保留時間和頻率，請傳送電子郵件到 [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com)，或提出支援案例。 [使用 Azure Cosmos DB 進行自動線上備份及還原](online-backup-and-restore.md)一文會提供備份功能的相關資訊。 

### <a name="how-does-the-cassandra-api-account-handle-failover-if-a-region-goes-down"></a>當區域發生服務中斷時，Cassandra API 帳戶如何處理容錯移轉？ 
Azure Cosmos DB Cassandra API 會藉助 Azure Cosmos DB 的全域分散式平台。 若要確保您的應用程式可以容許資料中心停機，請在 Azure Cosmos DB 入口網站中針對該帳戶至少啟用一個以上的區域，[使用多區域 Azure Cosmos DB 帳戶進行開發](regional-failover.md)。 您可以使用入口網站來設定區域的優先順序，[使用多區域 Azure Cosmos DB 帳戶進行開發](regional-failover.md)。 

您可以針對帳戶加入任意數目的區域，並藉由提供容錯移轉優先順序來控制容錯移轉的目標位置。 若要使用的資料庫，您還需要在目標位置提供應用程式。 如果您這樣做，您的客戶就不會經歷停機時間。 

### <a name="does-the-apache-cassandra-api-index-all-attributes-of-an-entity-by-default"></a>Apache Cassandra API 預設會為實體的所有屬性編製索引嗎？
是，Azure Cosmos DB 預設會為實體的所有屬性編製索引。 如需詳細資訊，請參閱 [Azure Cosmos DB：索引編製原則](indexing-policies.md)。 您可以透過一致的索引編製和一律永久認可仲裁的寫入，獲得保證具有效能的優勢。 

### <a name="does-this-mean-i-do-not-have-to-create-multiple-indexes-to-satisfy-the-queries"></a>這表示我不需要建立多個索引來滿足查詢嗎？ 
是，Azure Cosmos DB 會為所有屬性提供自動編製索引，而不需任何結構描述定義。 這項資訊能讓開發人員專注於應用程式，而不需擔心索引的建立和管理。 如需詳細資訊，請參閱 [Azure Cosmos DB：索引編製原則](indexing-policies.md)。

### <a name="can-i-use-the-new-cassandra-api-sdk-locally-with-the-emulator"></a>我可以在本機搭配使用新的 Cassandra API SDK 與模擬器嗎？
我們計劃在未來支援這項功能。 

### <a name="azure-cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-such-as-changefeed-and-other-functionality-will-these-capabilities-be-added-to-the-cassandra-api"></a>Azure Cosmos DB 做為平台時似乎還有很多功能，例如變更摘要及其他功能。 是否會將這些功能新增到 Cassandra API？ 
Apache Cassandra API 會提供和 Apache Cassandra 相同的 CQL 功能。 我們的確有計劃要了解在未來支援各種功能的可行性。

### <a name="feature-x-of-regular-cassandra-api-is-not-working-as-today-where-can-the-feedback-be-provided"></a>一般 Cassandra API 的某項功能運作情形和今天不同，我可以在哪裡提供意見反應？
請透過 [UserVoice 意見反應](https://feedback.azure.com/forums/263030-azure-cosmos-db)提供意見反應。

[azure-portal]: https://portal.azure.com
[query]: sql-api-sql-query.md
