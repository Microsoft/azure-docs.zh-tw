---
title: Azure Cosmos DB 的資料分割與調整規模 | Microsoft Docs
description: 了解資料分割在 Azure Cosmos DB 中的運作方式、如何設定資料分割和資料分割索引鍵，以及如何為您的應用程式挑選合適的資料分割索引鍵。
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: conceptual
ms.date: 05/24/2017
ms.author: rafats
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7e3c15ce201cee88d400b9d5fb9272b584293a2f
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34797426"
---
# <a name="partitioning-in-azure-cosmos-db-using-the-sql-api"></a>使用 SQL API 在 Azure Cosmos DB 進行資料分割

[Microsoft Azure Cosmos DB](../cosmos-db/introduction.md) 是全域分散的多重模型資料庫服務，其設計可協助您達成快速且可預測的效能，並順暢地隨著應用程式的成長而調整規模。 

本文提供如何透過 SQL API 使用 Cosmos DB 容器資料分割的概觀。 如需使用任一 Azure Cosmos DB API 進行資料分割之概念和最佳做法的概觀，請參閱[資料分割與水平調整](../cosmos-db/partition-data.md)。 

若要開始使用程式碼，請從 [Github](https://github.com/Azure/azure-documentdb-dotnet/tree/a2d61ddb53f8ab2a23d3ce323c77afcf5a608f52/samples/documentdb-benchmark) 下載專案。 

閱讀本文後，您將能夠回答下列問題：   

* Azure Cosmos DB 中資料分割的運作方式為何？
* 如何在 Azure Cosmos DB 中設定資料分割
* 什麼是資料分割索引鍵，以及如何為我的應用程式選擇正確的資料分割索引鍵？

若要開始使用程式碼，請從 [Azure Cosmos DB 效能測試驅動程式範例](https://github.com/Azure/azure-documentdb-dotnet/tree/a2d61ddb53f8ab2a23d3ce323c77afcf5a608f52/samples/documentdb-benchmark)下載專案。 

<!-- placeholder until we have a permanent solution-->
<a name="partition-keys"></a>
<a name="single-partition-and-partitioned-collections"></a>
<a name="migrating-from-single-partition"></a>

## <a name="partition-keys"></a>資料分割索引鍵

在 SQL API 中，您要以 JSON 路徑的形式來指定分割區索引鍵定義。 下表顯示資料分割索引鍵定義及各個對應值的範例。 資料分割索引鍵會以路徑的形式指定，例如 `/department` 代表 department 屬性。 

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td valign="top"><p><strong>資料分割索引鍵</strong></p></td>
            <td valign="top"><p><strong>說明</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>/department</p></td>
            <td valign="top"><p>對應至 doc.department 的值，其中 doc 是指項目。</p></td>
        </tr>
        <tr>
            <td valign="top"><p>/properties/name</p></td>
            <td valign="top"><p>對應至 doc.properties.name 的值，其中 doc 是指項目 (巢狀屬性)。</p></td>
        </tr>
        <tr>
            <td valign="top"><p>/id</p></td>
            <td valign="top"><p>對應至 doc.id 的值 (識別碼和資料分割索引鍵是相同屬性)。</p></td>
        </tr>
        <tr>
            <td valign="top"><p>/"department name"</p></td>
            <td valign="top"><p>對應至 doc["department name"] 的值，其中 doc 是指項目。</p></td>
        </tr>
    </tbody>
</table>

> [!NOTE]
> 資料分割索引鍵的語法類似於指定路徑以為原則路徑編製索引，但主要的差別在於路徑會對應至屬性而不是值，也就是結尾沒有萬用字元。 例如，您可以指定 /department/? 以為 department 下的值編製索引，但您會將 /department 指定為資料分割索引鍵定義。 資料分割索引鍵會以隱含方式編製索引，而且無法使用索引原則覆寫來排除編製索引。
> 
> 

讓我們看看資料分割索引鍵選擇對於應用程式效能的影響。

## <a name="working-with-the-azure-cosmos-db-sdks"></a>使用 Azure Cosmos DB SDK
Azure Cosmos DB 已透過 [REST API 版本 2015-12-16](/rest/api/cosmos-db/) 新增對自動資料分割的支援。 若要建立資料分割的容器，必須下載其中一個支援的 SDK 平台 (.NET、Node.js、Java、Python、MongoDB) 中的 SDK 1.6.0 版或更新版本。 

### <a name="creating-containers"></a>建立容器
下列範例顯示建立容器的 .NET 程式碼片段，可儲存輸送量每秒 20,000 個要求單位的裝置遙測資料。 SDK 會設定 OfferThroughput 值 (這會接著設定 REST API 中的 `x-ms-offer-throughput` 要求標頭)。 在此您會將 `/deviceId` 設定為分割區索引鍵。 選擇的資料分割索引鍵會與其餘的容器中繼資料 (例如名稱與編製索引原則) 一併儲存。

在此範例中，您挑選了 `deviceId`，因為您知道 (a) 由於有大量的裝置，因此可以將寫入平均分散到所有資料分割，讓您能夠調整資料庫規模以內嵌大量資料，以及 (b) 許多要求 (例如，提取裝置的最新讀取) 會限定在單一 deviceId，而可以從單一資料分割擷取。

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey);
await client.CreateDatabaseAsync(new Database { Id = "db" });

// Container for device telemetry. Here the property deviceId will be used as the partition key to 
// spread across partitions. Configured for 10K RU/s throughput and an indexing policy that supports 
// sorting against any number or string property.
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 20000 });
```

此方法會對 Cosmos DB 進行 REST API 呼叫，且服務會根據要求的輸送量來佈建許多資料分割。 您可以隨著效能需求的發展變更容器或一組容器的輸送量。 

### <a name="reading-and-writing-items"></a>讀取和寫入項目
現在，我們將資料插入 Cosmos DB 中。 以下是包含裝置讀取的範例類別，以及呼叫 CreateDocumentAsync 將新的裝置讀取插入容器中。 以下是運用 SQL API 的範例程式碼區塊：

```csharp
public class DeviceReading
{
    [JsonProperty("id")]
    public string Id;

    [JsonProperty("deviceId")]
    public string DeviceId;

    [JsonConverter(typeof(IsoDateTimeConverter))]
    [JsonProperty("readingTime")]
    public DateTime ReadingTime;

    [JsonProperty("metricType")]
    public string MetricType;

    [JsonProperty("unit")]
    public string Unit;

    [JsonProperty("metricValue")]
    public double MetricValue;
  }

// Create a document. Here the partition key is extracted as "XMS-0001" based on the collection definition
await client.CreateDocumentAsync(
    UriFactory.CreateDocumentCollectionUri("db", "coll"),
    new DeviceReading
    {
        Id = "XMS-001-FE24C",
        DeviceId = "XMS-0001",
        MetricType = "Temperature",
        MetricValue = 105.00,
        Unit = "Fahrenheit",
        ReadingTime = DateTime.UtcNow
    });
```

請依資料分割索引鍵和識別碼來讀取項目，並加以更新，然後最後一個步驟是依資料分割索引鍵和識別碼刪除項目。讀取包括 PartitionKey 值 (對應至 REST API 中的 `x-ms-documentdb-partitionkey` 要求標頭)。

```csharp
// Read document. Needs the partition key and the ID to be specified
Document result = await client.ReadDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"), 
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });

DeviceReading reading = (DeviceReading)(dynamic)result;

// Update the document. Partition key is not required, again extracted from the document
reading.MetricValue = 104;
reading.ReadingTime = DateTime.UtcNow;

await client.ReplaceDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"), 
  reading);

// Delete document. Needs partition key
await client.DeleteDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"), 
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });
```

### <a name="querying-partitioned-containers"></a>查詢資料分割的容器
當您查詢資料分割容器中的資料時，Cosmos DB 會自動將查詢路由傳送到與篩選中所指定之資料分割索引鍵值 (如果有的話) 對應的資料分割。 例如，此查詢只會路由傳送至包含資料分割索引鍵 "XMS-0001" 的資料分割。

```csharp
// Query using partition key
IQueryable<DeviceReading> query = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"))
    .Where(m => m.MetricType == "Temperature" && m.DeviceId == "XMS-0001");
```
    
下列查詢沒有根據資料分割索引鍵 (DeviceId) 的篩選，且已展開至對資料分割索引執行它的所有資料分割。 您必須指定 EnableCrossPartitionQuery (REST API 中的 `x-ms-documentdb-query-enablecrosspartition`)，才能讓 SDK 跨分割區執行查詢。

```csharp
// Query across partition keys
IQueryable<DeviceReading> crossPartitionQuery = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"), 
    new FeedOptions { EnableCrossPartitionQuery = true })
    .Where(m => m.MetricType == "Temperature" && m.MetricValue > 100);
```

Cosmos DB 使用 SQL (包含 SDK 1.12.0 和更新版本) 支援分割容器的[彙總函式](sql-api-sql-query.md#Aggregates) `COUNT`、`MIN`、`MAX` 和 `AVG`。 查詢必須包含單一彙總運算子，而且在投射中必須包含單一值。

### <a name="parallel-query-execution"></a>平行查詢執行
Cosmos DB SDK 1.9.0 和更新版本支援平行查詢執行選項，可讓您對資料分割集合執行低延遲查詢 (即使它們需要涉及大量的資料分割也一樣)。 例如，下列查詢設定為跨資料分割平行執行。

```csharp
// Cross-partition Order By Queries
IQueryable<DeviceReading> crossPartitionQuery = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"), 
    new FeedOptions { EnableCrossPartitionQuery = true, MaxDegreeOfParallelism = 10, MaxBufferedItemCount = 100})
    .Where(m => m.MetricType == "Temperature" && m.MetricValue > 100)
    .OrderBy(m => m.MetricValue);
```
    
若要管理平行執行查詢，您可以調整下列參數︰

* 藉由設定 `MaxDegreeOfParallelism`，您可以控制平行處理原則的程度，亦即同時連往容器分割區的網路連線數上限。 如果您將此屬性設定為 -1，平行處理原則的程度會由 SDK 管理。 如果 `MaxDegreeOfParallelism` 未指定或設為 0 (這是預設值)，將會有連往容器資料分割的單一網路連線。
* 您可藉由設定 `MaxBufferedItemCount`，來權衡取捨查詢延遲和用戶端記憶體使用量。 如果您省略此參數或將此屬性設定為 -1，平行查詢執行期間緩衝處理的項目數會由 SDK 管理。

在相同的集合狀態下，平行查詢會以和序列執行相同的順序傳回結果。 執行包含排序 (ORDER BY 和/或 TOP) 的跨資料分割查詢時，Azure Cosmos DB SDK 會跨資料分割發出平行查詢，並合併用戶端中已部分排序的結果來產生全域排序的結果。

### <a name="executing-stored-procedures"></a>執行預存程序
您也可以對具有相同裝置識別碼的文件執行不可部分完成交易，例如，如果您正在維護彙總或處於單一項目中裝置的最新狀態。 

```csharp
await client.ExecuteStoredProcedureAsync<DeviceReading>(
    UriFactory.CreateStoredProcedureUri("db", "coll", "SetLatestStateAcrossReadings"),
    new RequestOptions { PartitionKey = new PartitionKey("XMS-001") }, 
    "XMS-001-FE24C");
```
   
在下一節中，我們會探討如何從單一分割區容器改為分割容器。

## <a name="next-steps"></a>後續步驟
本文提供如何透過 SQL API 使用 Azure Cosmos DB 容器分割的概觀。 如需使用任一 Azure Cosmos DB API 進行資料分割之概念和最佳做法的概觀，另請參閱[資料分割與水平調整](../cosmos-db/partition-data.md)。 

* 使用 Azure Cosmos DB 執行規模和效能測試。 如需範例，請參閱 [Azure Cosmos DB 的效能和規模測試](performance-testing.md)。
* 使用 [SDK](sql-api-sdk-dotnet.md) 或 [REST API](/rest/api/cosmos-db/) 開始撰寫程式碼
* 了解 [Azure Cosmos DB 中佈建的輸送量](request-units.md)

