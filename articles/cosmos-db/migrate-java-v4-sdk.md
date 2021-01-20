---
title: 遷移您的應用程式，以使用 Azure Cosmos DB Java SDK v4 (com.azure.cosmos)
description: 瞭解如何使用舊版的 Azure Cosmos DB JAVA SDK，將您現有的 JAVA 應用程式升級為適用於 Core (SQL) API 的較新 JAVA SDK 4.0 (com.azure.cosmos 套件)。
author: anfeldma-ms
ms.custom: devx-track-java
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/11/2020
ms.reviewer: sngun
ms.openlocfilehash: e537c964d6063b76df63b3d80c5ef72b1ea56c92
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98600245"
---
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-java-sdk-v4"></a>遷移您的應用程式，以使用 Azure Cosmos DB Java SDK v4
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]  
> 如需此 SDK 的詳細資訊，請檢視 Azure Cosmos DB Java SDK v4 [版本資訊](sql-api-sdk-java-v4.md)、[Maven 存放庫](https://mvnrepository.com/artifact/com.azure/azure-cosmos)、Azure Cosmos DB Java SDK v4 [效能秘訣](performance-tips-java-sdk-v4-sql.md)和 Azure Cosmos DB Java SDK v4 [疑難排解指南](troubleshoot-java-sdk-v4-sql.md)。
>

本文說明如何將使用舊版 Azure Cosmos DB JAVA SDK 的現有 JAVA 應用程式升級至較新的 Azure Cosmos DB Java SDK 4.0 for Core (SQL) API。 Azure Cosmos DB JAVA SDK v4 會對應到 `com.azure.cosmos` 套件。 如果您要從下列任何一個 Azure Cosmos DB JAVA SDK 移轉應用程式，您可以使用本文件中的指示： 

* 同步 Java SDK 2.x.x
* 非同步 Java SDK 2.x.x
* Java SDK 3.x.x

## <a name="azure-cosmos-db-java-sdks-and-package-mappings"></a>Azure Cosmos DB JAVA SDK 和封裝對應

下表列出不同的 Azure Cosmos DB JAVA SDK、套件名稱和版本資訊：

| Java SDK| 發行日期 | 搭售方案 API   | Maven Jar  | Java 封裝名稱  |API 參考   | 版本資訊  |
|-------|------|-----------|-----------|--------------|-------------|---------------------------|
| 非同步 2.x.x  | 2018 年 6 月    | 非同步 (RxJAVA)  | `com.microsoft.azure::azure-cosmosdb` | `com.microsoft.azure.cosmosdb.rx` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [版本資訊](sql-api-sdk-async-java.md) |
| 同步 2.x.x     | 2018 年 9 月    | Sync   | `com.microsoft.azure::azure-documentdb` | `com.microsoft.azure.cosmosdb` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [版本資訊](sql-api-sdk-java.md)  |
| 3.x.x    | 2019 年 7 月    | Async(Reactor)/Sync  | `com.microsoft.azure::azure-cosmos`  | `com.azure.data.cosmos` | [API](https://azure.github.io/azure-cosmosdb-java/3.0.0/) | - |
| 4.0   | 2020 年 6 月   | Async(Reactor)/Sync  | `com.azure::azure-cosmos` | `com.azure.cosmos`   | -  | [API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-cosmos/4.0.1/index.html)  |

## <a name="sdk-level-implementation-changes"></a>SDK 層級的執行變更

以下是不同 SDK 之間的主要執行差異：

### <a name="rxjava-is-replaced-with-reactor-in-azure-cosmos-db-java-sdk-versions-3xx-and-40"></a>RxJAVA 會在 Azure Cosmos DB JAVA SDK 3.x.x 和 4.0 版中取代為 Reactor

如果您不熟悉非同步程式設計或回應式程式設計，請參閱 [Reactor 模式指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md)，以取得非同步程式設計和專案 Reactor 的簡介。 如果您一直在使用 Azure Cosmos DB 同步 JAVA SDK 2.x.x 或 Azure Cosmos DB JAVA SDK 3.x.x 同步 API，本指南可能會很有用。

如果您一直在使用 Azure Cosmos DB 非同步 JAVA SDK 2.x.x，並打算移轉至 4.0 SDK，請參閱 [Reactor vs RxJAVA 指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md)，以取得將 RxJAVA 程式碼轉換為使用 Reactor 的指引。

### <a name="azure-cosmos-db-java-sdk-v4-has-direct-connectivity-mode-in-both-async-and-sync-apis"></a>Azure Cosmos DB JAVA SDK v4 在非同步和同步 API 中都有直接連線模式

如果您已使用 Azure Cosmos DB 同步 JAVA SDK 2.x.x，請注意以 TCP 為基礎的直接連線模式 (相對於 HTTP)，會在適用於非同步和同步 API 的 Azure Cosmos DB JAVA SDK 4.0 中執行。

## <a name="api-level-changes"></a>API 層級變更

相較於先前的 SDK，以下是 Azure Cosmos DB JAVA SDK 4.x 中的 API 層級變更 (JAVA SDK 3.x.x、Async JAVA SDK 2.x.x 和 Sync JAVA SDK 2.x.x)：

:::image type="content" source="./media/migrate-java-v4-sdk/java-sdk-naming-conventions.png" alt-text="Azure Cosmos DB JAVA SDK 命名慣例":::

* Azure Cosmos DB JAVA SDK 3.x.x 和4.0 會將用戶端資源稱為 `Cosmos<resourceName>`。 例如，`CosmosClient`、`CosmosDatabase`、`CosmosContainer`。 而在2.x.x 版中，Azure Cosmos DB JAVA SDK 並沒有統一的命名配置。

* Azure Cosmos DB JAVA SDK 3.x.x 和4.0 提供同步和非同步 API。

  * **Java SDK 4.0**：所有類別都屬於同步 API，除非類別名稱在 `Cosmos` 之後附加 `Async`。

  * **Java SDK 3.x.x**：所有的類別都屬於非同步 API，除非類別名稱在 `Cosmos` 之後附加 `Async`。

  * **非同步 Java SDK 2.x.x**：類別名稱類似於同步 JAVA SDK 2.x.x，但名稱開頭為 *非同步*。

### <a name="hierarchical-api-structure"></a>階層式 API 結構

Azure Cosmos DB JAVA SDK 4.0 和 3.x.x 引進階層式 API 結構，以嵌套的方式組織用戶端、資料庫和容器，如下列 4.0 SDK 程式碼片段所示：

```java
CosmosContainer container = client.getDatabase("MyDatabaseName").getContainer("MyContainerName");

```

在2.x.x 版的 Azure Cosmos DB JAVA SDK 中，資源和文件上的所有作業都是透過用戶端執行個體執行。

### <a name="representing-documents"></a>代表文件

在 Azure Cosmos DB JAVA SDK 4.0 中，自訂 POJO 的和 `JsonNodes` 是從 Azure Cosmos DB 讀取和寫入文件的兩個選項。

在 Azure Cosmos DB JAVA SDK 3.x.x 中，`CosmosItemProperties` 物件是由公用 API 公開，並以文件標記法的形式提供。 此類別不再公開於4.0 版。

### <a name="imports"></a>匯入

* Azure Cosmos DB 的 JAVA SDK 4.0 套件開頭為 `com.azure.cosmos`
* Azure Cosmos DB JAVA SDK 3.x.x 套件開頭為 `com.azure.data.cosmos`
* Azure Cosmos DB JAVA SDK 2.x. x 同步 API 套件開頭為 `com.microsoft.azure.documentdb`

* Azure Cosmos DB JAVA SDK 4.0 會將數個類別放在嵌套封裝 `com.azure.cosmos.models`中。 這些封裝有些包括：

  * `CosmosContainerResponse`
  * `CosmosDatabaseResponse`
  * `CosmosItemResponse`
  * 所有上述封裝的非同步 API 類比
  * `CosmosContainerProperties`
  * `FeedOptions`
  * `PartitionKey`
  * `IndexingPolicy`
  * `IndexingMode` ...等事項

### <a name="accessors"></a>存取子

Azure Cosmos DB JAVA SDK 4.0 會公開用來存取執行個體成員的 `get` 和 `set` 方法。 例如，`CosmosContainer` 執行個體具有 `container.getId()` 和 `container.setId()` 方法。

這不同於會公開流暢介面的 Azure Cosmos DB JAVA SDK 3.x.x。 例如，`CosmosSyncContainer` 執行個體具有多載的 `container.id()`，以取得或設定 `id` 值。

## <a name="code-snippet-comparisons"></a>程式碼片段比較

### <a name="create-resources"></a>建立資源

下列程式碼片段顯示如何在4.0、3.x、Async Api 和2.x 同步 Api 之間建立資源的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateJavaSDKv4ResourceAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
ConnectionPolicy defaultPolicy = ConnectionPolicy.defaultPolicy();
//  Setting the preferred location to Cosmos DB Account region
defaultPolicy.preferredLocations(Lists.newArrayList("Your Account Location"));

//  Create async client
//  <CreateAsyncClient>
client = new CosmosClientBuilder()
        .endpoint("your.hostname")
        .key("yourmasterkey")
        .connectionPolicy(defaultPolicy)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .build();

// Create database with specified name
client.createDatabaseIfNotExists("YourDatabaseName")
      .flatMap(databaseResponse -> {
        database = databaseResponse.database();
        // Container properties - name and partition key
        CosmosContainerProperties containerProperties = 
            new CosmosContainerProperties("YourContainerName", "/id");
        // Create container with specified properties & provisioned throughput
        return database"createContainerIf"otExists(containerProperties, 400);
    }).flatMap(containerResponse -> {
        container = containerResponse.container();
        return Mono.empty();
}).subscribe();
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
ConnectionPolicy defaultPolicy = ConnectionPolicy.GetDefault();
//  Setting the preferred location to Cosmos DB Account region
defaultPolicy.setPreferredLocations(Lists.newArrayList("Your Account Location"));

//  Create document client
//  <CreateDocumentClient>
client = new DocumentClient("your.hostname", "your.masterkey", defaultPolicy, ConsistencyLevel.Eventual)

// Create database with specified name
Database databaseDefinition = new Database();
databaseDefinition.setId("YourDatabaseName");
ResourceResponse<Database> databaseResourceResponse = client.createDatabase(databaseDefinition, new RequestOptions());

// Read database with specified name
String databaseLink = "dbs/YourDatabaseName";
databaseResourceResponse = client.readDatabase(databaseLink, new RequestOptions());
Database database = databaseResourceResponse.getResource();

// Create container with specified name
DocumentCollection documentCollection = new DocumentCollection();
documentCollection.setId("YourContainerName");
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="item-operations"></a>項目操作

下列程式碼片段顯示如何在4.0、3.x、Async Api 和2.x 同步 Api 之間執行專案作業的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemOpsAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
// Container is created. Generate many docs to insert.
int number_of_docs = 50000;
ArrayList<JsonNode> docs = generateManyDocs(number_of_docs);

// Insert many docs into container...
Flux.fromIterable(docs)
    .flatMap(doc -> container.createItem(doc))
    .subscribe(); // ...Subscribing triggers stream execution.
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
//  Container is created. Generate documents to insert.
Document document = new Document();
document.setId("YourDocumentId");
ResourceResponse<Document> documentResourceResponse = client.createDocument(documentCollection.getSelfLink(), document,
    new RequestOptions(), true);
Document responseDocument = documentResourceResponse.getResource();
```
---

### <a name="indexing"></a>編製索引

下列程式碼片段顯示如何在4.0、3.x、Async Api 和2.x 同步 Api 之間建立索引的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateIndexingAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties(containerName, "/lastName");

// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT); //To turn indexing off set IndexingMode.NONE

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.path("/*");
includedPaths.add(includedPath);
indexingPolicy.includedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.path("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.excludedPaths(excludedPaths);

containerProperties.indexingPolicy(indexingPolicy);

CosmosContainer containerIfNotExists = database.createContainerIfNotExists(containerProperties, 400)
                                               .block()
                                               .container();
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.Consistent); //To turn indexing off set IndexingMode.NONE

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Create container with specified name and indexing policy
DocumentCollection documentCollection = new DocumentCollection();
documentCollection.setId("YourContainerName");
documentCollection.setIndexingPolicy(indexingPolicy);
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="stored-procedures"></a>預存程序

下列程式碼片段顯示如何在4.0、3.x、Async Api 和2.x 同步 Api 之間建立預存程式的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateSprocAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
logger.info("Creating stored procedure...\n");

sprocId = "createMyDocument";
String sprocBody = "function createMyDocument() {\n" +
        "var documentToCreate = {\"id\":\"test_doc\"}\n" +
        "var context = getContext();\n" +
        "var collection = context.getCollection();\n" +
        "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
        "    function (err, documentCreated) {\n" +
        "if (err) throw new Error('Error' + err.message);\n" +
        "context.getResponse().setBody(documentCreated.id)\n" +
        "});\n" +
        "if (!accepted) return;\n" +
        "}";
CosmosStoredProcedureProperties storedProcedureDef = new CosmosStoredProcedureProperties(sprocId, sprocBody);
container.getScripts()
        .createStoredProcedure(storedProcedureDef,
                new CosmosStoredProcedureRequestOptions()).block();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

CosmosStoredProcedureRequestOptions options = new CosmosStoredProcedureRequestOptions();
options.partitionKey(new PartitionKey("test_doc"));

container.getScripts()
        .getStoredProcedure(sprocId)
        .execute(null, options)
        .flatMap(executeResponse -> {
            logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
                    sprocId,
                    executeResponse.responseAsString(),
                    executeResponse.statusCode(),
                    executeResponse.requestCharge()));
            return Mono.empty();
        }).block();
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
logger.info("Creating stored procedure...\n");

String sprocId = "createMyDocument";
String sprocBody = "function createMyDocument() {\n" +
    "var documentToCreate = {\"id\":\"test_doc\"}\n" +
    "var context = getContext();\n" +
    "var collection = context.getCollection();\n" +
    "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
    "    function (err, documentCreated) {\n" +
    "if (err) throw new Error('Error' + err.message);\n" +
    "context.getResponse().setBody(documentCreated.id)\n" +
    "});\n" +
    "if (!accepted) return;\n" +
    "}";
StoredProcedure storedProcedureDef = new StoredProcedure();
storedProcedureDef.setId(sprocId);
storedProcedureDef.setBody(sprocBody);
StoredProcedure storedProcedure = client.createStoredProcedure(documentCollection.getSelfLink(), storedProcedureDef, new RequestOptions())
                                        .getResource();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

RequestOptions options = new RequestOptions();
options.setPartitionKey(new PartitionKey("test_doc"));

StoredProcedureResponse storedProcedureResponse =
    client.executeStoredProcedure(storedProcedure.getSelfLink(), options, null);
logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
    sprocId,
    storedProcedureResponse.getResponseAsString(),
    storedProcedureResponse.getStatusCode(),
    storedProcedureResponse.getRequestCharge()));
```
---

### <a name="change-feed"></a>變更摘要

下列程式碼片段顯示如何在4.0 和 3.x.x 非同步 API 之間執行變更摘要作業的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateCFAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
ChangeFeedProcessor changeFeedProcessorInstance = 
ChangeFeedProcessor.Builder()
        .hostName(hostName)
        .feedContainer(feedContainer)
        .leaseContainer(leaseContainer)
        .handleChanges((List<CosmosItemProperties> docs) -> {
            logger.info("--->setHandleChanges() START");

            for (CosmosItemProperties document : docs) {
                try {

                    // You are given the document as a CosmosItemProperties instance which you may
                    // cast to the desired type.
                    CustomPOJO pojo_doc = document.getObject(CustomPOJO.class);
                    logger.info("----=>id: " + pojo_doc.id());

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            logger.info("--->handleChanges() END");

        })
        .build();

// ...

 changeFeedProcessorInstance.start()
                            .subscribeOn(Schedulers.elastic())
                            .subscribe();
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

* JAVA SDK v2 同步處理不支援這項功能。 
---

### <a name="container-level-time-to-livettl"></a>容器層級存留時間 (TTL)

下列程式碼片段顯示如何使用4.0、3.x、Async Api 和2.x 同步 Api 來為容器中的資料建立存留時間的差異：

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateContainerTTLAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
CosmosContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
DocumentCollection documentCollection;

// Create a new container with TTL enabled with default expiration value
documentCollection.setDefaultTimeToLive(90 * 60 * 60 * 24);
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="item-level-time-to-livettl"></a>項目層級存留時間 (TTL)

下列程式碼片段顯示如何使用4.0、3.x、Async Api 和2.x 同步 Api 來為專案建立存留時間的差異： < 2. x

# <a name="java-sdk-40-async-api"></a>[JAVA SDK 4.0 非同步 API](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemTTLClassAsync)]

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemTTLAsync)]

# <a name="java-sdk-3xx-async-api"></a>[JAVA SDK 3.x.x 非同步 API](#tab/java-v3-async)

```java
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String id() {return this.id;}
    public SalesOrder id(String new_id) {this.id = new_id; return this;}
    public String customerId() {return this.customerId; return this;}
    public SalesOrder customerId(String new_cid) {this.customerId = new_cid;}
    public Integer ttl() {return this.ttl;}
    public SalesOrder ttl(Integer new_ttl) {this.ttl = new_ttl; return this;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);
```

# <a name="java-sdk-2xx-sync-api"></a>[JAVA SDK 2.x 同步 API](#tab/java-v2-sync)

```java
Document document = new Document();
document.setId("YourDocumentId");
document.setTimeToLive(60 * 60 * 24 * 30 ); // Expire document in 30 days
ResourceResponse<Document> documentResourceResponse = client.createDocument(documentCollection.getSelfLink(), document,
    new RequestOptions(), true);
Document responseDocument = documentResourceResponse.getResource();
```
---

## <a name="next-steps"></a>後續步驟

* [建置 JAVA 應用程式](create-sql-api-java.md)來使用 V4 SDK 管理 Azure Cosmos DB SQL API 資料
* 瞭解[以 Reactor 為基礎的 JAVA SDK](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md)
* 瞭解如何使用 [Reactor vs RxJAVA 指南](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md)將 RxJAVA 非同步程式碼轉換成 Reactor 非同步程式碼
