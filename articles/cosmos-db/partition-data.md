---
title: Azure Cosmos DB 中的資料分割和水平調整 | Microsoft Docs
description: 了解資料分割在 Azure Cosmos DB 中的運作方式、如何設定資料分割和資料分割索引鍵，以及如何為您的應用程式挑選合適的資料分割索引鍵。
services: cosmos-db
author: aliuy
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 07/26/2018
ms.author: andrl
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 3cc2794105eff196c3e1db02d664a89c9b37e318
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286980"
---
# <a name="partition-and-scale-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中進行資料分割和調整

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 是全域分散的多重模型資料庫服務，其設計目的是要協助您達成快速且可預測的效能。 它會順暢地隨著應用程式調整規模。 本文概述 Azure Cosmos DB 中所有資料模型的資料分割運作方式的概觀， 它也描述可如何設定 Azure Cosmos DB 容器以有效地調整應用程式規模。

## <a name="partitioning-in-azure-cosmos-db"></a>Azure Cosmos DB 中的資料分割
Azure Cosmos DB 提供存放資料的容器，稱為「集合」(適用於文件)、「圖形」或「資料表」。 容器是邏輯資源，可以跨一或多個實體分割或伺服器。 分割區數目由 Azure Cosmos DB 根據為容器或一組容器佈建的儲存體大小和輸送量所決定。 


### <a name="physical-partition"></a>實體分割區

「實體」分割區是固定數量的保留 SSD 型儲存體，與可變數量的計算資源 (CPU 和記憶體) 的結合。 系統會複寫每個實體分割區以獲得高可用性。 每一組容器可共用一或多個實體分割區。 實體分割區管理可完全由 Azure Cosmos DB 管理，您不需要撰寫複雜的程式碼或管理分割區。 Azure Cosmos DB 容器在儲存體和輸送量方面並無限制。 實體分割區是 Azure Cosmos DB 的內部概念，而且是暫時性的。 Azure Cosmos DB 將根據您的工作負載，自動調整實體分割區的數目。 因此您不應該讓資料庫的設計與實體分割區的數目相互關聯，而是要選擇能決定邏輯分割區的正確分割區索引鍵。 

### <a name="logical-partition"></a>邏輯分割區

「邏輯」分割區是一種實體分割區內的分割區，當中儲存與某個單一分割區索引鍵值相關的所有資料。 多個邏輯分割區可以最終都在同一個實體分割區內。 在下圖中，單一容器有三個邏輯分割區。 每個邏輯分割區都會個別儲存一個分割區索引鍵、LAX、AMS 和 MEL 的資料。 每個 LAX、AMS 和 MEL 邏輯分割區無法成長超過 10 GB 的邏輯分割區上限。 

![資源的資料分割](./media/introduction/azure-cosmos-db-partitioning.png) 

當容器符合[資料分割必要條件](#prerequisites)時，資料分割對應用程式而言完全是背景作業。 Azure Cosmos DB 會跨實體和邏輯分割區散發資料，以及將查詢要求路由傳送至正確的分割區。 

## <a name="how-does-partitioning-work"></a>資料分割如何運作

每個文件都必須具備可唯一識別它的「分割區索引鍵」和「資料列索引鍵」。 您的分割區索引鍵是存放資料的邏輯分割區，並為 Azure Cosmos DB 提供將資料分散到實體分割區的自然界限。 **單一邏輯分割區的資料必須位在單一實體分割區內，而實體分割區則是受 Azure Cosmos DB 管理**。 

簡單地說，Azure Cosmos DB 中的資料分割運作方式如下︰

* 您佈建一組輸送量為 **T** RU/s (每秒要求數目) 的 Azure Cosmos DB 容器。  
* Azure Cosmos DB 會在幕後佈建服務 **T** 每秒要求所需的實體分割區。 如果 **T** 超過每一實體分割區的最大輸送量 **t**，Azure Cosmos DB 就會佈建 **N = T/t** 的實體分割區。 每一分割區的最大輸送量值由 Azure Cosmos DB 所設定，此值會根據佈建的總輸送量和使用的硬體組態來指派。  
* Azure Cosmos DB 會在 **N** 個實體分割區平均配置分割區索引鍵雜湊的索引鍵空間。 因此，每個實體分割區裝載的邏輯分割區數目是 **1/N** * 分割區索引鍵值的數目。  
* 當實體分割區 **p** 達到儲存體限制時，Azure Cosmos DB 會以無縫方式將 **p** 分割成兩個新的實體分割區 **p1** 和 **p2**。 它會將相當於約索引鍵一半的值分配給每個新的實體分割區。 此分割作業對應用程式來說完全是隱形的。 如果實體分割區觸達其儲存空間限制，且實體分割區中的所有資料屬於相同的邏輯分割區索引鍵，就不會進行分割作業。 這是因為單一邏輯分割區索引鍵的所有資料必須位於相同的實體分割區中。 在此情況下，應該採用不同的分割區索引鍵策略。  
* 當您佈建超過 **t*N** 的輸送量時，Azure Cosmos DB 會分割您的一或多個實體分割區，以支援更高的輸送量。

資料分割索引鍵的語意會稍有不同，以符合各 API 的語意，如下表所示︰

| API | 資料分割索引鍵 | 列索引鍵 |
| --- | --- | --- |
| SQL | 自訂資料分割索引鍵路徑 | 固定 `id` | 
| MongoDB | 自訂共用索引鍵  | 固定 `_id` | 
| Gremlin | 自訂資料分割索引鍵屬性 | 固定 `id` | 
| 資料表 | 固定 `PartitionKey` | 固定 `RowKey` | 

Azure Cosmos DB 使用雜湊型資料分割。 當您寫入項目時，Azure Cosmos DB 會將資料分割索引鍵值進行雜湊處理，然後使用雜湊的結果來判斷要在其中儲存項目的分割區。 

> [!NOTE]
> Azure Cosmos DB 會將資料分割索引鍵相同的所有項目儲存在相同的實體分割區中。 

## <a name="best-practices-when-choosing-a-partition-key"></a>選擇分割區索引鍵時的最佳做法

選擇資料分割索引鍵是在設計階段必須進行的一項重要決策。 請選擇具有各種不同的值、且有平均存取模式的屬性名稱。 分割區索引鍵最好是有大量的相異值 (例如數百或數千個)。 這可讓您將工作負載平均分散到這些值。 理想的分割區索引鍵是經常在您的查詢中出現作為篩選條件的索引鍵，並且還要具有足夠的基數，以確保您的解決方案靈活有彈性。

如果實體分割區達到其儲存體限制，而分割區中的所有資料都具有相同的分割區索引鍵，Azure Cosmos DB 就會傳回「分割區索引鍵已達 10 GB 大小上限」訊息，且不會將分割區進行分割。 選擇良好的分割區索引鍵是非常重要的決策。 

選擇分割區索引鍵，以便：

* 儲存體均勻分佈在所有索引鍵中。  
* 指定時間點的要求磁碟區均勻分佈在所有索引鍵中。  

  建議您檢查資料在分割區之間的分佈情況。 若要在入口網站中檢查資料分布，請移至 Azure Cosmos DB 帳戶，並按一下 [監視] 區段中的 [計量]，然後按一下 [儲存體] 索引標籤，以查看資料在不同實體分割區之間的分割情況。

  ![資源的資料分割](./media/partition-data/partitionkey-example.png)

  上面左圖顯示不良分割區索引鍵的結果，右圖則顯示選擇良好分割區索引鍵時的結果。 在左圖中，您可以看到資料並未平均分佈在各個分割區。 您應該盡量選擇一個可將資料分散成像右圖那樣的分割區索引鍵。

* 可藉由在篩選器述詞中包含分割區索引鍵，有效地路由傳送以高基數叫用的查詢。  
* 選擇具有較高基數的分割區索引鍵通常是偏好選項，因為它通常會產生較佳的分佈和延展性。 例如，可以串連多個屬性的值來增加基數，藉以形成綜合索引鍵。  

當您根據上述考量選擇分割區索引鍵時，您不必擔心分割區的數目或每個實體分割區配置了多少的輸送量，因為 Azure Cosmos DB 能相應放大實體分割區的數目，也可以視需要調整個別的分割區。

<a name="prerequisites"></a>
## <a name="prerequisites-for-partitioning"></a>資料分割的必要條件

Azure Cosmos DB 容器可以在 Azure 入口網站中建立為「固定」或「無限制」。 固定大小的容器具有上限為 10 GB 和 10,000 RU/秒的輸送量。 若要建立無限制的容器，您必須指定一個分割區索引鍵，以及 1,000 RU/秒的最小輸送量。 Azure DB Cosmos 容器也可設定為在一組容器之間共用輸送量；在此情況下，每個容器都必須指定一個分割區索引鍵，且可以無限制成長。 下列為針對資料分割與調整規模應考慮的必要條件：

* 在 Azure 入口網站中建立容器 (例如集合、圖表或資料表) 時，選取 [無限制] 儲存容量選項，即可利用無限制調整功能。 對於要自動分割為 **p1** 和 **p2** 的實體分割區，如[資料分割如何運作](#how-does-partitioning-work)中所述，必須使用 1,000 RU/s 以上的輸送量來建立容器 (或在一組容器間共用輸送量)，且必須提供分割區索引鍵。 

* 如果您已在 Azure 入口網站中或透過程式設計方式建立容器，而初始傳輸量為 1,000 RU/秒以上，並且已提供分割區索引鍵，便可以利用無限制調整功能，而無須支付容器費用。 這包括**固定**容器，只要初始容器建立時所搭配的傳輸量至少為 1,000 RU/秒，且已指定分割區索引鍵即可。

* 所有設定為在一組容器間共用輸送量的容器，都會視為**無限制的**容器。

如果您已建立**固定**容器，但沒有分割區索引鍵或輸送量低於 1,000 RU/秒，該容器就不會自動調整。 若要將資料從固定容器移轉到無限制的容器，您必須使用[資料移轉工具](import-data.md)或[變更摘要庫](change-feed.md)。 

## <a name="partitioning-and-provisioned-throughput"></a>資料分割與佈建的輸送量
Azure Cosmos DB 設計用來取得可預測的效能。 當您建立容器或一組容器時，須以「每秒[要求單位](request-units.md) (RU) 數目」保留輸送量。 每個要求都會造成一筆 RU 費用，此費用與作業所耗用的系統資源 (例如 CPU、記憶體及 IO) 數量成正比。 讀取 1 KB 具有工作階段一致性的文件，會使用 1 RU。 不論儲存的項目數或同時執行的並行要求數，讀取一次都是 1 個 RU。 根據大小之不同，較大的項目需要較高的 RU。 如果您知道實體大小及針對應用程式所需支援的讀取次數，則可以佈建應用程式需求所需的確切輸送量。 

> [!NOTE]
> 為了充分利用為容器或一組容器佈建的輸送量，您必須選擇一個可讓您將要求平均分散到所有相異分割區索引鍵值的分割區索引鍵。
> 
> 

<a name="designing-for-partitioning"></a>
## <a name="create-partition-key"></a>建立分割區索引鍵 
您可以使用 Azure 入口網站或 Azure CLI 建立容器，並在任何時間進行調整。 本節說明如何使用每個 API 來建立容器，以及指定佈建的輸送量和分割區索引鍵。


### <a name="sql-api"></a>SQL API
下列範例示範如何使用 SQL API 來建立容器 (集合)。 

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey);
await client.CreateDatabaseAsync(new Database { Id = "db" });

DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 20000 });
```

您可以使用 REST API 中的 `GET` 方法或使用其中一個 SDK 中的 `ReadDocumentAsync` 來讀取項目 (文件)。

```csharp
// Read document. Needs the partition key and the ID to be specified
DeviceReading document = await client.ReadDocumentAsync<DeviceReading>(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"), 
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });
```

如需詳細資訊，請參閱[使用 SQL API 在 Azure Cosmos DB 進行資料分割](sql-api-partition-data.md)。

### <a name="mongodb-api"></a>MongoDB API
您可以使用 MongoDB API，透過您喜愛的工具、驅動程式或 SDK 建立分區化集合。 在此範例中，我們使用 Mongo 殼層來建立集合。

在 Mongo 殼層中︰

```
db.runCommand( { shardCollection: "admin.people", key: { region: "hashed" } } )
```
    
結果：

```JSON
{
    "_t" : "ShardCollectionResponse",
    "ok" : 1,
    "collectionsharded" : "admin.people"
}
```

### <a name="table-api"></a>資料表 API

若要使用「資料表 API」來建立資料表，請使用 `CreateIfNotExists`方法。 

```csharp
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

CloudTable table = tableClient.GetTableReference("people");
table.CreateIfNotExists(throughput: 800);
```

佈建的輸送量會以 `CreateIfNotExists` 的引數形式來設定。 資料分割索引鍵會隱含地建立為 `PartitionKey` 值。 

您可以使用下列程式碼來擷取單一實體：

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = table.Execute(retrieveOperation);
```
如需詳細資訊，請參閱[使用資料表 API 進行開發](tutorial-develop-table-dotnet.md)。

### <a name="gremlin-api"></a>Gremlin API

使用 Gremlin API 時，您可以使用 Azure 入口網站或 Azure CLI 來建立代表圖表的容器。 或者，由於 Azure Cosmos DB 是多模型資料庫的緣故，因此您也可以使用其中一個其他 API 來建立和調整您的圖表容器。

您可以在 Gremlin 中使用資料分割索引鍵和識別碼來判讀任何頂點或邊緣。 例如，對於使用區域 (美國) 作為資料分割索引鍵和使用「西雅圖」作為資料列索引鍵的圖形，您可以使用下列語法找到頂點︰

```
g.V(['USA', 'Seattle'])
```

您可以使用資料分割索引鍵和資料列索引鍵來參考某個邊緣。

```
g.E(['USA', 'I5'])
```

如需詳細資訊，請參閱[使用 Azure Cosmos DB 中的資料分割圖表](graph-partitioning.md)。

## <a name="form-partition-key-by-concatenating-multiple-fields"></a>透過串連多個欄位來形成分割區索引鍵

您也可以透過將多個屬性值串連並填補為該項目的單一人工 “partitionKey” 屬性來形成分割區索引鍵。 這些索引鍵被稱為綜合索引鍵。

例如，假設您有一個看起來如下的文件：

```json
{
"deviceId": "abc-123",
"date": 2018
}
```

其中一個選項為針對 /deviceId 或 /date 設定 partitionKey。 如果想要針對裝置識別碼和日期形成分割區索引鍵。 將這兩個值串連為人工的 "partitionKey" 屬性，並將分割區索引鍵設定至 /partitionKey。

```json
{
"deviceId": "abc-123",
"date": 2018,
"partitionKey": "abc-123-2018"
}
```

在即時案例中，您可能會有數千個文件，因此您應該定義用戶端邏輯來將值串連為綜合索引鍵，將該綜合索引鍵插入至文件中，然後用它來指定分割區索引鍵。

<a name="designing-for-scale"></a>
## <a name="design-for-scale"></a>專為調整設計
若要使用 Azure Cosmos DB 進行有效調整，您必須在建立容器時挑選適當的資料分割索引鍵。 選擇良好的分割區索引鍵時，有兩個主要的考量：

* **查詢界限與交易**。 您選擇的分割區索引鍵應在使用交易的需求與將實體分散到多個分割區索引鍵的需求之間取得平衡，以確保解決方案具有可調整的靈活彈性。 以一個極端的情況來說，您可以針對所有項目設定相同的資料分割索引鍵，但這種做法可能會限制解決方案的延展性。 反過來，您可以為每個項目指派唯一的資料分割索引鍵。 這個做法具有高延展性，但是會讓您無法透過預存程序和觸發程序使用跨文件交易。 理想的資料分割索引鍵可讓您使用有效率的查詢，而且具有足夠的基數，可確保您的解決方案能加以調整。 
* **沒有儲存體和效能瓶頸**。 請務必選取允許寫入分散到不同相異值的屬性。 對相同分割區索引鍵提出的要求不能超過配置給分割區的已佈建輸送量，並且會受到速率限制。 因此，請務必挑選不會在應用程式內導致「作用點」的資料分割索引鍵。 因為單一資料分割索引鍵的所有資料皆必須儲存在資料分割中，您應該在遇到有大量相同值的資料時，避免資料分割索引鍵。 

來看看一些真實案例和每個案例良好的資料分割索引鍵︰
* 如果您要實作使用者設定檔後端，「使用者識別碼」會是不錯的分割區索引鍵選擇。
* 如果您要儲存 IoT 資料 (例如裝置狀態)，「裝置識別碼」會是不錯的分割區索引鍵選擇。
* 如果您要使用 Azure Cosmos DB 來記錄時間序列資料，「主機名稱」或「處理序識別碼」會是不錯的分割區索引鍵選擇。
* 如果您有多租用戶架構，「租用戶識別碼」會是不錯的分割區索引鍵選擇。

在一些使用案例 (例如 IoT 與使用者設定檔) 中，分割區索引鍵可能與您的「識別碼」 (文件索引鍵) 相同。 在其他使用案例 (例如時間序列資料) 中，分割區索引鍵則可能與「識別碼」不同。

### <a name="partitioning-and-loggingtime-series-data"></a>資料分割和記錄/時間序列資料
Azure Cosmos DB 中的其中一個常見使用案例是記錄與遙測。 在此案例中，挑選一個良好的分割區索引鍵相當重要，因為您可能需要讀取/寫入大量資料。 分割區索引鍵的選擇取決於您的讀寫速率，以及您預期要執行的查詢類型。 如何選擇適當資料分割索引鍵的一些秘訣如下：

* 如果您的使用案例牽涉到已累積一段長時間的較小速率寫入，而您需要依時間戳記範圍搭配其他篩選條件進行查詢，請使用彙總的時間戳記。 例如，以日期作為資料分割索引鍵是不錯的方法。 使用此方法時，您可以查詢單一分割區中某個指定日期的所有資料。 
* 如果您的工作負載是大量寫入 (在此案例中非常常見)，請使用不是以時間戳記為基礎的分割區索引鍵。 如此，Azure Cosmos DB 才能在各個分割區之間平均分配及調整寫入。 在這個案例中，「主機名稱」、「處理序識別碼」、「活動識別碼」或其他具有高基數的屬性會是不錯的選擇。 
* 另一個方法是混合式方法，其中您會擁有多個容器，每天/每月各有一個容器，而分割區索引鍵會是更細微的屬性，例如「主機名稱」。 此方法的優點是，您可以根據時間範圍及規模和效能需求，為每個容器或一組容器設定不同的輸送量。 例如，可以為當前月份的容器佈建較高的輸送量，因為它要為讀取和寫入提供服務。 針對先前月份，可以佈建較低的輸送量，因為它們只為讀取提供服務。

### <a name="partitioning-and-multitenancy"></a>資料分割與多組織用戶管理
如果您要使用 Azure Cosmos DB 來實作多租用戶應用程式，可以考量兩個常用的設計：「每一租用戶一個分割區索引鍵」和「每一租用戶一個容器」。 以下是每項方式的優缺點︰

* **一個租用戶一個資料分割索引鍵**。 在此模型中，租用戶會共置於單一容器內。 您可以針對單一分割區，執行單一租用戶的查詢與插入。 您也可以跨屬於某個租用戶的所有項目實作交易邏輯。 由於多個租用戶共用一個容器，因此您可以將所有租用戶的資源集中置於單一容器內，而不針對每個租用戶個別進行佈建，來更充分運用儲存體和已佈建的輸送量。 缺點是無法隔離每個租用戶的效能。 提高輸送量來確保效能將會套用至含有所有租用戶的整個容器，而不會只針對個別租用戶提高輸送量。
* **一個租用戶一個容器**。 在此模型中，每個租用戶都有自己的容器，而您則可以依每一租用戶保留保證效能的輸送量。 對於只有一些租用戶的多租用戶應用程式來說，此模型較具成本效益。

您也可以使用將小型租用戶共置在一起而將較大型租用戶隔離至其專屬容器的混合式方法。

## <a name="next-steps"></a>後續步驟
在本文中，我們概述了在 Azure Cosmos DB 中進行調整和資料分割的概念及最佳做法。 

* 了解 [Azure Cosmos DB 中佈建的輸送量](request-units.md)。
* 了解 [Azure Cosmos DB 中的全域分散](distribute-data-globally.md)。



