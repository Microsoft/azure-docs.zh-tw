---
title: Azure Cosmos DB 的佈建輸送量 | Microsoft Docs
description: 了解如何設定 Azure Cosmos DB 容器、集合、圖表和資料表的佈建輸送量。
services: cosmos-db
author: aliuy
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: andrl
ms.openlocfilehash: 2da00f700f5cc234455cc686377e5863f1c35bdd
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45734466"
---
# <a name="set-and-get-throughput-for-azure-cosmos-db-containers-and-database"></a>設定及取得 Azure Cosmos DB 容器和資料庫的輸送量

您可以使用 Azure 入口網站或使用用戶端 SDK，設定 Azure Cosmos DB 容器或一組容器的輸送量。 

**佈建個別容器的輸送量：** 在佈建一組容器的輸送量時，這些容器全都會共用所佈建的輸送量。 佈建個別容器的輸送量會保證保留該特定容器的輸送量。 指派個別容器層級的 RU/秒時，容器可以建立為*固定*或*無限制的*形式。 固定大小的容器具有上限為 10 GB 和 10,000 RU/秒的輸送量。 若要建立無限制的容器，您必須指定最小輸送量 1,000 RU/s 和[分割區索引鍵](partition-data.md)。 由於您的資料可能必須分散在多個分割區，因此必須挑選基數高 (100 個到數百萬個相異值) 的分割區索引鍵。 藉由選取具有許多相異值的分割區索引鍵，您便可確保 Azure Cosmos DB 可以一致地調整您的容器/資料表/圖表和要求。 

**佈建一組容器或資料庫的輸送量：** 佈建資料庫的輸送量可讓您在所有屬於該資料庫的容器之間共用輸送量。 在 Azure Cosmos DB 資料庫內，您可以有一組共用輸送量的容器，以及有專用輸送量的容器。 指派跨一組容器的 RU/秒時，屬於這組容器的容器會視為*無限制的*容器，且必須指定分割區索引鍵。

Azure Cosmos DB 會根據所佈建的輸送量，配置實體分割區來裝載您的容器，並隨著輸送量的成長來跨分割區分割/重新平衡資料。 容器和資料庫層級輸送量佈建是不同的供應項目，在其間切換需要將資料從來源移轉到目的地。 這表示您需要建立新資料庫或新集合，然後藉由使用[大量執行程式程式庫](bulk-executor-overview.md)或 [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) 來移轉資料。 下圖說明在不同層級佈建輸送量：

![佈建個別容器和一組容器的要求單位](./media/request-units/provisioning_set_containers.png)

在後續幾節中，您將會了解在不同層級為 Azure Cosmos DB 帳戶設定輸送量所需的步驟。 

## <a name="provision-throughput-by-using-azure-portal"></a>使用 Azure 入口網站佈建輸送量

### <a name="provision-throughput-for-a-container-collectiongraphtable"></a>佈建容器的輸送量 (集合/圖表/資料表)

1. 登入 [Azure 入口網站](https://portal.azure.com)。  
2. 從左側導覽選取 [所有資源]，然後尋找 Azure Cosmos DB 帳戶。  
3. 您可以在建立容器 (集合/圖表/資料表) 時設定輸送量，也可以更新現有容器的輸送量。  
4. 若要在建立容器時指派輸送量，請開啟 [資料總管] 刀鋒視窗，然後針對其他 API 選取 [新增集合] ([新增圖表]、[新增資料表])  
5. 填寫 [新增集合] 刀鋒視窗中的表單。 下表會說明此刀鋒視窗中的欄位：  

   |**設定**  |**說明**  |
   |---------|---------|
   |資料庫識別碼  |  提供可識別資料庫的唯一名稱。 資料庫是一或多個集合的邏輯容器。 資料庫名稱必須包含從 1 到 255 個字元，且不能包含 /、\\、#、? 或尾端空格。 |
   |集合識別碼  | 提供可識別集合的唯一名稱。 集合識別碼與資料庫名稱具有相同的字元需求。 |
   |儲存體容量   | 此值代表資料庫的儲存體容量。 在佈建個別集合的輸送量時，儲存體容量可以**固定 (10 GB)**，也可以**無限量**。 若要使用無限制的儲存體容量，您需要為資料設定資料分割索引鍵。  |
   |Throughput   | 每個集合和資料庫都會有以「每秒要求單位數」表示的輸送量。  如果是固定的儲存體容量，最小輸送量為每秒 400 個要求單位 (RU/秒)，如果是無限制的儲存體容量，最小輸送量則會設為 1000 RU/秒。|

6. 在輸入這些欄位的值之後，選取 [確定] 以儲存設定。  

   ![設定集合的輸送量](./media/set-throughput/set-throughput-for-container.png)

7. 若要更新現有容器的輸送量，請展開資料庫和容器，然後按一下 [設定]。 在新視窗中輸入新的輸送量值，然後選取 [儲存]。  

   ![更新集合的輸送量](./media/set-throughput/update-throughput-for-container.png)

### <a name="provision-throughput-for-a-set-of-containers-or-at-the-database-level"></a>針對一組容器或在資料庫層級佈建輸送量

1. 登入 [Azure 入口網站](https://portal.azure.com)。  
2. 從左側導覽選取 [所有資源]，然後尋找 Azure Cosmos DB 帳戶。  
3. 您可以在建立資料庫時設定輸送量，也可以更新現有資料庫的輸送量。  
4. 若要在建立資料庫時指派輸送量，請開啟 [資料總管] 刀鋒視窗，然後選取 [新增資料庫]  
5. 填寫 [資料庫識別碼] 值，勾選 [佈建輸送量] 選項，然後設定輸送量值。  

   ![使用新增資料庫選項設定輸送量](./media/set-throughput/set-throughput-with-new-database-option.png)

6. 若要更新現有資料庫的輸送量，請展開資料庫和容器，然後按一下 [調整]。 在新視窗中輸入新的輸送量值，然後選取 [儲存]。  

   ![更新資料庫的輸送量](./media/set-throughput/update-throughput-for-database.png)

### <a name="provision-throughput-for-a-set-of-containers-as-well-as-for-an-individual-container-in-a-database"></a>針對一組容器以及資料庫中的個別容器佈建輸送量

1. 登入 [Azure 入口網站](https://portal.azure.com)。  
2. 從左側導覽選取 [所有資源]，然後尋找 Azure Cosmos DB 帳戶。  
3. 建立資料庫，並對其指派輸送量。 開啟 [資料總管] 刀鋒視窗，然後選取 [新增資料庫]  
4. 填寫 [資料庫識別碼] 值，勾選 [佈建輸送量] 選項，然後設定輸送量值。  

   ![使用新增資料庫選項設定輸送量](./media/set-throughput/set-throughput-with-new-database-option.png)

5. 接下來，在您於上述步驟所建立的資料庫中建立集合。 若要建立集合，請以滑鼠右鍵按一下資料庫，然後選取 [新增集合]。  

6. 在 [新增集合] 刀鋒視窗中，輸入集合的名稱和資料分割索引鍵。 (選擇性) 如果您選擇不指派輸送量值，您可以針對該特定容器佈建輸送量，指派給資料庫的輸送量會與集合共用。  

   ![(選擇性) 設定容器的輸送量](./media/set-throughput/optionally-set-throughput-for-the-container.png)

## <a name="considerations-when-provisioning-throughput"></a>佈建輸送量時的考量

以下是一些可協助您決定輸送量保留策略的考量。

### <a name="considerations-when-provisioning-throughput-at-the-database-level"></a>在資料庫層級佈建輸送量時的考量

有下列情況時，請考慮在資料庫層級 (也就是針對一組容器) 佈建輸送量：

* 如果您有為數十多個以上的容器，並可在其中一部分或全部一起共用輸送量。  

* 在從設計目的是要在 IaaS 所裝載的 VM 上或在內部部署環境上執行的單一租用戶資料庫 (例如，NoSQL 或關聯式資料庫) 移轉至 Azure Cosmos DB，並擁有許多容器時。  

* 如果您想要考慮在資料庫層級使用集區輸送量進行非計劃性的工作負載提升。  

* 您不想設定個別容器的輸送量，而是要跨資料庫內的一組容器取得彙總輸送量。

### <a name="considerations-when-provisioning-throughput-at-the-container-level"></a>在容器層級佈建輸送量時的考量

有下列情況時，請考慮在個別容器佈建輸送量：

* 如果您的 Azure Cosmos DB 容器數量不多。  

* 如果您想要在獲得 SLA 支援的指定容器上取得保證的輸送量。

<a id="set-throughput-sdk"></a>

## <a name="set-throughput-by-using-sql-api-for-net"></a>使用 SQL API for .NET 來設定輸送量

### <a name="set-throughput-at-the-container-level"></a>設定容器層級的輸送量
以下程式碼片段會使用 SQL API 的 .NET SDK，為個別容器建立一個具有每秒 3,000 個要求單位的容器：

```csharp
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 3000 });
```

### <a name="set-throughput-for-a-set-of-containers-at-the-database-level"></a>在資料庫層級為一組容器設定輸送量

以下程式碼片段會使用 SQL API 的 .NET SDK，在一組容器間佈建每秒 100,000 個要求單位：

```csharp
// Provision 100,000 RU/sec at the database level. 
// sharedCollection1 and sharedCollection2 will share the 100,000 RU/sec from the parent database
// dedicatedCollection will have its own dedicated 4,000 RU/sec, independant of the 100,000 RU/sec provisioned from the parent database
Database database = await client.CreateDatabaseAsync(new Database { Id = "myDb" }, new RequestOptions { OfferThroughput = 100000 });

DocumentCollection sharedCollection1 = new DocumentCollection();
sharedCollection1.Id = "sharedCollection1";
sharedCollection1.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(database.SelfLink, sharedCollection1, new RequestOptions())

DocumentCollection sharedCollection2 = new DocumentCollection();
sharedCollection2.Id = "sharedCollection2";
sharedCollection2.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(database.SelfLink, sharedCollection2, new RequestOptions())

DocumentCollection dedicatedCollection = new DocumentCollection();
dedicatedCollection.Id = "dedicatedCollection";
dedicatedCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(database.SelfLink, dedicatedCollection, new RequestOptions { OfferThroughput = 4000 )
```

Azure Cosmos DB 針對輸送量會以保留模型的方式運作。 也就是說，您需要針對所「保留」的輸送量支付費用，而不論該輸送量中有多少數量是主動「使用」的。 當您應用程式的負載、資料及使用方式模式變更時，您可以透過 SDK 或使用 [Azure 入口網站](https://portal.azure.com)，輕鬆地相應增加和減少保留 RU 的數量。

每個容器或一組容器會對應至 Azure Cosmos DB 中的一個 `Offer` 資源，其中有所佈建輸送量的相關中繼資料。 藉由查閱容器的對應 offer 資源，然後使用新的輸送量值加以更新，即可變更已配置的輸送量。 以下程式碼片段會使用 .NET SDK，將容器的輸送量變更為每秒 5,000 個要求單位。 在變更輸送量之後，您應重新整理任何現有的 Azure 入口網站視窗，以顯示變更的輸送量。 

```csharp
// Fetch the resource to be updated
// For a updating throughput for a set of containers, replace the collection's self link with the database's self link
Offer offer = client.CreateOfferQuery()
                .Where(r => r.ResourceLink == collection.SelfLink)    
                .AsEnumerable()
                .SingleOrDefault();

// Set the throughput to 5000 request units per second
offer = new OfferV2(offer, 5000);

// Now persist these changes to the database by replacing the original resource
await client.ReplaceOfferAsync(offer);
```

當您變更輸送量時，不會影響容器或一組容器的可用性。 新保留的輸送量通常會在套用新輸送量的數秒內於生效。

<a id="set-throughput-java"></a>

## <a name="to-set-the-throughput-by-using-the-sql-api-for-java"></a>使用 SQL API for Java 設定輸送量

下列程式碼片段會取出目前的輸送量，並將其更改為 500 RU/秒。 如需完整的程式碼範例，請參閱 GitHub 上的 [OfferCrudSamples.java](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/OfferCrudSamples.java) 檔案。 

```Java
// find offer associated with this collection
// To change the throughput for a set of containers, use the database's resource id instead of the collection's resource id
Iterator < Offer > it = client.queryOffers(
    String.format("SELECT * FROM r where r.offerResourceId = '%s'", collectionResourceId), null).getQueryIterator();
assertThat(it.hasNext(), equalTo(true));

Offer offer = it.next();
assertThat(offer.getString("offerResourceId"), equalTo(collectionResourceId));
assertThat(offer.getContent().getInt("offerThroughput"), equalTo(throughput));

// update the offer
int newThroughput = 500;
offer.getContent().put("offerThroughput", newThroughput);
client.replaceOffer(offer);
```

## <a name="get-throughput-by-using-mongodb-api-portal-metrics"></a>使用 MongoDB API 入口網站計量取得輸送量

若要準確估計 MongoDB API 資料庫的要求單位費用，最簡單的方法就是使用 [Azure 入口網站](https://portal.azure.com)計量。 利用 [要求數目] 和 [要求費用] 圖表，您可以估計每個作業耗用多少要求單位，以及它們彼此之間相對耗用多少要求單位。

![MongoDB API 入口網站計量][1]

### <a id="RequestRateTooLargeAPIforMongoDB"></a> 超過 MongoDB API 中保留的輸送量限制
超過為容器或一組容器佈建之輸送量的應用程式會受到速率限制，直取用速率降到佈建的輸送量速率以下為止。 發生速率限制時，後端會結束要求，並傳回 `16500` 錯誤碼 - `Too Many Requests`。 根據預設，在傳回 `Too Many Requests` 錯誤碼之前，MongoDB API 會自動重試最多 10 次。 如果您收到很多 `Too Many Requests` 錯誤碼，您可以考慮在應用程式的錯誤處理常式中新增重試行為，或[提高容器的佈建輸送量](set-throughput.md)。

## <a id="GetLastRequestStatistics"></a>使用 MongoDB API 的 GetLastRequestStatistics 命令取得要求費用

MongoDB API 支援自訂命令 getLastRequestStatistics，可擷取指定作業的要求費用。

例如，在 Mongo 殼層中，執行您想要驗證要求費用的作業。
```
> db.sample.find()
```

接著，執行命令 *getLastRequestStatistics*。
```
> db.runCommand({getLastRequestStatistics: 1})
{
    "_t": "GetRequestStatisticsResponse",
    "ok": 1,
    "CommandName": "OP_QUERY",
    "RequestCharge": 2.48,
    "RequestDurationInMilliSeconds" : 4.0048
}
```

若要估計您的應用程式所需的保留輸送量，其中一個方法為對照應用程式使用的代表性項目，記錄與執行一般作業相關聯的要求單位費用，然後估計您預期每秒會執行的作業數目。

> [!NOTE]
> 如果您的項目類型與已編製索引之屬性的大小與數目截然不同，則請記錄與每個一般項目「類型」相關聯的適用作業要求單位費用。
> 
> 

## <a name="throughput-faq"></a>輸送量常見問題集

**我可以將輸送量設定為小於 400 RU/s 嗎？**

400 RU/s 是 Cosmos DB 單一資料分割容器上可用的最小輸送量 (1000 RU/s 是資料分割容器的最小值)。 要求單位是設定為 100 RU/s 時間間隔，但不能將輸送量設定為 100 RU/s 或任何小於 400 RU/s 的值。 如果您正在尋找符合成本效益的方法來開發和測試 Cosmos DB，您可以使用免費的 [Cosmos DB 模擬器](local-emulator.md)，於本機免費部署此模擬器。 

**如何使用 MongoDB API 設定輸送量？**

MongoDB API 沒有可以設定輸送量的擴充功能。 建議使用 SQL API，如[使用 SQL API for .NET 設定輸送量](#set-throughput-sdk)中所示。

## <a name="next-steps"></a>後續步驟

* 若要了解如何估計輸送量和要求單位，請參閱 [Azure Cosmos DB 中的要求單位和估計輸送量](request-units.md)

* 若要深入了解 Cosmos DB 的佈建和全球規模化，請參閱 [Cosmos DB 的資料分割和規模調整](partition-data.md)。

[1]: ./media/set-throughput/api-for-mongodb-metrics.png
