---
title: 利用存留時間讓 Azure Cosmos DB 中的資料過期 | Microsoft Docs
description: Microsoft Azure Cosmos DB 可讓您利用 TTL 在一段時間後自動從系統清除文件。
services: cosmos-db
keywords: 存留時間
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 08/29/2017
ms.author: sngun
ms.openlocfilehash: 2cae74224a9d59939175ac7e43d4d6b183ca3933
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44050731"
---
# <a name="expire-data-in-azure-cosmos-db-collections-automatically-with-time-to-live"></a>利用存留時間讓 Azure Cosmos DB 集合中的資料自動過期
應用程式可以產生並儲存大量資料。 其中有些資料，例如電腦產生的事件資料、記錄檔和使用者工作階段資訊只能在有限的期間內使用。 一旦資料超過應用程式的需求，即可放心清除此資料並減少應用程式的儲存體需求。

Microsoft Azure Cosmos DB 可讓您利用「存留時間」(或稱 TTL) 在一段時間後自動從資料庫清除文件。 預設存留時間可以在集合層級設定，並且在每份文件上覆寫。 設定 TTL (不論是作為集合預設值或是在文件層級設定) 之後，Cosmos DB 就會自動移除自上次修改後存在時間達該指定時間 (以秒為單位) 的文件。

Azure Cosmos DB 中的存留時間會使用上次修改文件時的對照時間位移。 為了這麼做，它會使用每份文件上都有的 `_ts` 欄位。 _ts 欄位是 unix 樣式的 Epoch 時間戳記，表示日期和時間。 每次修改文件時都會更新 `_ts` 欄位。 

## <a name="ttl-behavior"></a>TTL 行為
TTL 功能是由兩個層級 (集合層級和文件層級) 的 TTL 屬性所控制。 這些值會以秒為單位來設定，而且會視為與上次修改文件時的 `_ts` 的差異。

1. 集合的 DefaultTTL
   
   * 如果遺失 (或設為 null)，便不會自動刪除文件。
   * 如果存在且值設為 "-1" = 無限 – 文件預設不會過期
   * 如果存在且值設為某個數字 ("n") – 文件會在上次修改後的 "n" 秒到期
2. 文件的 TTL︰ 
   
   * 僅在父集合有 DefaultTTL 時，才適用屬性。
   * 覆寫父集合的 DefaultTTL 值。

文件一過期 (`ttl` + `_ts` <= 目前的伺服器時間)，就會標示為「已過期」。 在此時間之後，不允許在這些文件上執行任何作業，而且將會從任何執行的查詢結果中排除。 這些文件會在系統中遭到實體刪除，而稍後在背景中伺機刪除。 這不會耗用集合預算中的任何 [要求單位 (RU)](request-units.md) 。

上述邏輯可以顯示在下列矩陣中︰

|  | 集合上遺漏/未設定 DefaultTTL | 集合上的 DefaultTTL = -1 | 集合上的 DefaultTTL = n' |
| --- |:--- |:--- |:--- |
| 集合上遺漏 TTL |文件層級沒有可覆寫的項目，因為文件和集合沒有 TTL 的概念。 |此集合中的所有文件都不會過期。 |此集合中的文件將會在間隔 n' 過去時到期。 |
| 文件上的 TTL = -1 |文件層級沒有可覆寫的項目，因為集合未定義文件可以覆寫的 DefaultTTL 屬性。 系統無法解譯文件上的 TTL。 |此集合中的所有文件都不會過期。 |此集合中 TTL=-1 的文件永遠不會過期。 所有其他文件將在 n' 間隔之後過期。 |
| 文件上的 TTL = n |文件層級沒有可覆寫的項目。 系統無法解譯文件上的 TTL。 |TTL = n 的文件將在間隔 n (以秒為單位) 之後到期。 其他文件會繼承間隔 -1 且永遠不會過期。 |TTL = n 的文件將在間隔 n (以秒為單位) 之後到期。 其他文件會繼承集合的間隔 n'。 |

## <a name="configuring-ttl"></a>設定 TTL
根據預設，在所有 Cosmos DB 集合中及所有文件上都會停用存留時間。 您可以程式設計方式或使用 Azure 入口網站來設定 TTL。 請使用下列步驟從 Azure 入口網站設定 TTL：

1. 登入 [Azure 入口網站](https://portal.azure.com/)並瀏覽至 Azure Cosmos DB 帳戶。  

2. 瀏覽至您想要設定 TTL 值的集合，開啟 [級別與設定] 窗格。 您可以看見 [存留時間] 預設會設為 [關閉]。 您可以將它變更為 [開啟 (無預設)] 或 [開啟]。

   **關閉** - 不會自動刪除文件。  
   **開啟 (無預設)** - 此選項會將 TTL 值設定為 "-1" (無限)，這表示文件預設不會過期。  
   **開啟** - 文件會在上次修改後的 "n" 秒到期。  

   ![設定存留時間](./media/time-to-live/set-ttl-in-portal.png)

## <a name="enabling-ttl"></a>啟用 TTL
若要在集合上或集合內的文件上啟用 TTL，您需要將集合的 DefaultTTL 屬性設定為 -1 或非零的正數。 將 DefaultTTL 設定為 -1，表示集合中的所有文件都預設為永遠存留，但 Cosmos DB 服務應監視此集合中已覆寫這個預設值的文件。

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = "orders";
    collectionDefinition.PartitionKey.Paths.Add("/customerId");
    collectionDefinition.DefaultTimeToLive =-1; //never expire by default

    DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
        UriFactory.CreateDatabaseUri(databaseName),
        collectionDefinition,
        new RequestOptions { OfferThroughput = 20000 });

## <a name="configuring-default-ttl-on-a-collection"></a>在集合上設定預設 TTL
您可以在集合層級設定預設存留時間。 若要在集合上設定 TTL，您必須提供一個非零的正數，指出集合中的所有文件在上次修改的時間戳記 (`_ts`) 之後要到期的期間 (以秒為單位)。 或者，您可以將預設值設定為 -1，表示插入集合中的所有文件預設會無限期存留。

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = "orders";
    collectionDefinition.PartitionKey.Paths.Add("/customerId");
    collectionDefinition.DefaultTimeToLive = 90 * 60 * 60 * 24; // expire all documents after 90 days
    
    DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
        "/dbs/salesdb",
        collectionDefinition,
        new RequestOptions { OfferThroughput = 20000 });


## <a name="setting-ttl-on-a-document"></a>在文件上設定 TTL
除了在集合上設定預設 TTL，您可以在文件層級設定特定的 TTL。 這麼做將會覆寫集合的預設值。

* 若要在文件上設定 TTL，您必須提供一個非零的正數，指出文件在上次修改的時間戳記 (`_ts`) 之後要到期的期間 (以秒為單位)。
* 如果文件沒有 TTL 欄位，則會套用集合的預設值。
* 如果已在集合層級停用 TTL，則會忽略文件上的 TTL 欄位，直到在集合上重新啟用 TTL 為止。

以下是說明如何在文件上設定 TTL 到期時間的程式碼片段：

    // Include a property that serializes to "ttl" in JSON
    public class SalesOrder
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }
        
        [JsonProperty(PropertyName="cid")]
        public string CustomerId { get; set; }
        
        // used to set expiration policy
        [JsonProperty(PropertyName = "ttl", NullValueHandling = NullValueHandling.Ignore)]
        public int? TimeToLive { get; set; }
        
        //...
    }
    
    // Set the value to the expiration in seconds
    SalesOrder salesOrder = new SalesOrder
    {
        Id = "SO05",
        CustomerId = "CO18009186470",
        TimeToLive = 60 * 60 * 24 * 30;  // Expire sales orders in 30 days 
    };


## <a name="extending-ttl-on-an-existing-document"></a>在現有文件上延長 TTL
您可以在文件上進行任何寫入作業，以重設文件上的 TTL。 這麼做會將 `_ts` 設定為目前的時間，並重新開始倒數文件的到期時間 (如 `ttl` 所設定)。 如果您想要變更文件的 `ttl`，您可以如同處理任何其他可設定的欄位一樣更新此欄位。

    response = await client.ReadDocumentAsync(
        "/dbs/salesdb/colls/orders/docs/SO05"), 
        new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });
    
    Document readDocument = response.Resource;
    readDocument.TimeToLive = 60 * 30 * 30; // update time to live
    
    response = await client.ReplaceDocumentAsync(readDocument);

## <a name="removing-ttl-from-a-document"></a>從文件移除 TTL
如果文件上已設定 TTL，而您不再希望文件會到期，您可以擷取此文件、移除 TTL 欄位，然後取代伺服器上的文件。 移除文件的 TTL 欄位後，將會套用集合的預設值。 若要阻止文件到期且不要繼承集合的值，您必須將 TTL 值設定為 -1。

    response = await client.ReadDocumentAsync(
        "/dbs/salesdb/colls/orders/docs/SO05"), 
        new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });
    
    Document readDocument = response.Resource;
    readDocument.TimeToLive = null; // inherit the default TTL of the collection
    
    response = await client.ReplaceDocumentAsync(readDocument);

## <a name="disabling-ttl"></a>停用 TTL
若要在集合上完全停用 TTL 並阻止背景處理程序尋找過期的文件，則應刪除集合上的 DefaultTTL 屬性。 刪除此屬性與將它設定為 -1 不同。 設定為 -1，表示加入至集合的新文件會永遠存留，但是您可以在集合中的特定文件上覆寫此值。 從集合中完全移除此屬性，表示即使有已明確覆寫先前預設值的文件，所有文件也都不會過期。

    DocumentCollection collection = await client.ReadDocumentCollectionAsync("/dbs/salesdb/colls/orders");
    
    // Disable TTL
    collection.DefaultTimeToLive = null;
    
    await client.ReplaceDocumentCollectionAsync(collection);

<a id="ttl-and-index-interaction"></a> 
## <a name="ttl-and-index-interaction"></a>TTL 和索引互動
新增或變更集合上的 TTL 設定會變更基礎索引。 當 TTL 值從 Off 變更為 On，集合會重新建立索引。 在索引模式保持一致的情況下對索引原則進行變更時，使用者不會注意到索引的變更。 當索引模式設定為緩慢時，索引永遠能夠跟上，如果 TTL 值變更，則會從頭開始重新建立索引。 當 TTL 值變更且索引模式設定為緩慢時，在索引重建期間所執行的查詢不會傳回完整或正確的結果。

如果您需要傳回精確的資料，則請勿在索引模式設定為緩慢時變更 TTL 值。 在理想情況下，請選擇一致的索引以確保查詢結果保持一致。 

## <a name="faq"></a>常見問題集
**TTL 的費用為何？**

在文件上設定 TTL 不會產生額外的費用。

**TTL 一旦到了，刪除我的文件要花多少時間？**

TTL 一旦到了，文件會立即到期，並將無法透過 CRUD 或查詢 API 存取。 

**文件上的 TTL 是否會對 RU 費用造成任何影響？**

否，在 Cosmos DB 中透過 TTL 刪除過期的文件將不會影響 RU 費用。

**TTL 功能只會套用至整個文件，或者我可以讓個別的文件屬性值過期？**

TTL 會套用到整份文件。 如果您只想要讓文件的一部分過期，建議您從主要文件中將該部分擷取到個別的「已連結」文件，然後在該擷取文件上使用 TTL。

**TTL 功能是否有任何特定的編製索索引需求？**

是。 集合的[編製索引原則](indexing-policies.md)必須設定為 [延遲] 或 [一致]。 嘗試在編製索引設為 [無] 的集合上設定 DefaultTTL 會造成錯誤，而嘗試在已設定 DefaultTTL 的集合上關閉索引編製也會造成錯誤。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure Cosmos DB，請參閱服務的[*文件 (英文)*](https://azure.microsoft.com/documentation/services/cosmos-db/) 頁面。

