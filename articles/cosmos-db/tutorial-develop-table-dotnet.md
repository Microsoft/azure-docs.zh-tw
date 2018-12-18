---
title: Azure Cosmos DB：使用 .NET 搭配資料表 API 進行開發 | Microsoft Docs
description: 了解如何使用 .NET 搭配 Azure Cosmos DB 的「資料表 API」進行開發
services: cosmos-db
author: SnehaGunda
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/18/2017
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: e6511b9511d2598b58fd3afee34803ceb09ac5ce
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38582847"
---
# <a name="azure-cosmos-db-develop-with-the-table-api-in-net"></a>Azure Cosmos DB：使用 .NET 搭配資料表 API 進行開發

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。

本教學課程涵蓋下列工作： 

> [!div class="checklist"] 
> * 建立 Azure Cosmos DB 帳戶 
> * 在 app.config 檔案中啟用功能 
> * 使用[資料表 API](table-introduction.md) 建立資料表
> * 將實體新增至資料表 
> * 插入一批實體 
> * 擷取單一實體 
> * 使用自動次要索引來查詢實體 
> * 取代實體 
> * 刪除實體 
> * 刪除資料表
 
## <a name="tables-in-azure-cosmos-db"></a>Azure Cosmos DB 中的資料表 

Azure Cosmos DB 針對需要無結構描述設計之索引鍵-值存放區的應用程式，提供[資料表 API](table-introduction.md)。 Azure Cosmos DB 資料表 API 和 [Azure 表格儲存體](../storage/common/storage-introduction.md)現在都支援相同的 SDK 和 REST API。 您可以使用 Azure Cosmos DB 來建立具有高輸送量需求的資料表。

本教學課程的對象是熟悉「Azure 資料表」儲存體 SDK 且想要使用 Azure Cosmos DB 所提供之進階功能的開發人員。 本教學課程是以[以 .NET 開始使用 Azure 表格儲存體](table-storage-how-to-use-dotnet.md)為基礎，並說明如何利用額外的功能，例如次要索引、佈建的輸送量及多路連接。 本教學課程說明如何使用 Azure 入口網站來建立 Azure Cosmos DB 帳戶，然後建置及部署「資料表 API」應用程式。 此外，也逐步解說 .NET 範例，這些範例可用來建立和刪除資料表，以及插入、更新、刪除和查詢資料表資料。 

如果您目前是使用 Azure 資料表儲存體，改用 Azure Cosmos DB 資料表 API 可獲得下列好處：

- 具有多路連接及[自動和手動容錯移轉](regional-failover.md)的完備[全域散發](distribute-data-globally.md)
- 支援所有屬性 (次要索引) 之無從驗證結構描述的自動索引編製，以及快速查詢 
- 支援在任意數目的區域[獨立調整儲存體和輸送量](partition-data.md)
- 支援[每一資料表有專用輸送量](request-units.md) (可從每秒數百個要求調整到數百萬個要求)
- 支援[五個可調整的一致性層級](consistency-levels.md)，可根據您應用程式的需求，進行可用性、延遲及一致性的取捨
- 單一區域內可達 99.99% 的可用性，並且能夠新增更多區域來提高可用性，以及在一般可用性上達到[業界頂尖的全面性 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
- 可與現有的 Azure 儲存體 .NET SDK 搭配運作，不需對您的應用程式進行任何程式碼變更

本教學課程涵蓋使用 .NET SDK 的 Azure Cosmos DB 資料表 API。 您可以從 NuGet 下載 [Azure Cosmos DB 表格 API .NET SDK](https://aka.ms/tableapinuget)。

若要深入了解複雜的 Azure 資料表儲存體工作，請參閱：

* [Azure Cosmos DB 資料表 API 簡介](table-introduction.md)
* 如需資料表服務參考文件中有關可用 API 的完整詳細資訊，請參閱 [Azure Cosmos DB 資料表 API .NET SDK](https://docs.microsoft.com/dotnet/api/overview/azure/cosmosdb/client?view=azure-dotnet)

### <a name="about-this-tutorial"></a>關於本教學課程
本教學課程的對象是熟悉 Azure 資料表儲存體 SDK 且想要使用 Azure Cosmos DB 所提供之進階功能的開發人員。 本教學課程是以[以 .NET 開始使用 Azure 表格儲存體](table-storage-how-to-use-dotnet.md)為基礎，並說明如何利用額外的功能，例如次要索引、佈建的輸送量及多路連接。 我們涵蓋了如何使用 Azure 入口網站來建立 Azure Cosmos DB 帳戶，然後建置和部署「資料表」應用程式。 此外，也逐步解說 .NET 範例，這些範例可用來建立和刪除資料表，以及插入、更新、刪除和查詢資料表資料。 

如果尚未安裝 Visual Studio 2017，您可以下載並使用「免費的」[Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)。 務必在 Visual Studio 設定期間啟用 **Azure 開發**。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-database-account"></a>建立資料庫帳戶

我們將從在 Azure 入口網站中建立 Azure Cosmos DB 帳戶開始著手。  
 
> [!IMPORTANT]  
> 您需要建立新的資料表 API 帳戶，以與正式推出的資料表 API SDK 搭配使用。 正式推出的 SDK 不支援在預覽期間建立的資料表 API 帳戶。 
>

[!INCLUDE [cosmosdb-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)] 

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在，我們將從 Github 複製「資料表」應用程式、設定連接字串，然後執行它。 您會看到，以程式設計方式來處理資料有多麼的容易。 

1. 開啟 git 終端機視窗 (例如 git bash)，並使用 `cd` 命令變更至要安裝範例應用程式的資料夾。 

    ```bash
    cd "C:\git-samples"
    ```

2. 執行下列命令來複製範例存放庫。 此命令會在您的電腦上建立範例應用程式副本。 

    ```bash
    git clone https://github.com/Azure-Samples/storage-table-dotnet-getting-started.git
    ```

3. 然後在 Visual Studio 中開啟方案檔案。 

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。 這可讓您的應用程式與託管資料庫進行通訊。 

1. 在 [Azure 入口網站](http://portal.azure.com/)中，按一下 [連接字串]。 

    使用畫面右方的複製按鈕來複製主要連接字串。

    ![在 [連接字串] 窗格中檢視及複製「連接字串」。](./media/create-table-dotnet/connection-string.png)

2. 在 Visual Studio 中，開啟 app.config 檔案。 

3. 本教學課程不會使用儲存體模擬器，因此請取消註解第 8 行的 StorageConnectionString，並將第 7 行的 StorageConnectionString 註解化。 第 7 行和第 8 行現在看起來應該像這樣：

    ```
    <!--key="StorageConnectionString" value="UseDevelopmentStorage=true;" />-->
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]" />
    ```

4. 將來自入口網站的主要連接字串貼到第 8 行上的 StorageConnectionString 值中。 在引號內貼上字串。
   
    > [!IMPORTANT]
    > 如果端點使用 documents.azure.com，這表示您擁有預覽帳戶，因此您需要建立一個[新的資料表 API 帳戶](#create-a-database-account)與正式推出的資料表 API SDK 搭配使用。 
    >

    第 8 行現在看起來應該類似：

    ```
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=txZACN9f...==;TableEndpoint=https://<account name>.table.cosmosdb.azure.com;" />
    ```

5. 儲存 app.config 檔案。

您現已更新應用程式，使其具有與 Azure Cosmos DB 通訊所需的所有資訊。 

## <a name="azure-cosmos-db-capabilities"></a>Azure Cosmos DB 功能
Azure Cosmos DB 支援一些 Azure 資料表儲存體 API 中未提供的功能。 

某些功能要透過新的多載 CreateCloudTableClient 才能使用；CreateCloudTableClient 可用來指定連線原則和一致性層級。

| 資料表連線設定 | 說明 |
| --- | --- |
| 連線模式  | Azure Cosmos DB 支援兩種連線模式。 在 `Gateway` 模式下，一律是對 Azure Cosmos DB 閘道提出要求，閘道會將要求轉送到對應的資料分割區。 在 `Direct` 連線模式下，用戶端會擷取資料表與分割的對應，而提出要求時，則是直接對資料分割區提出。 我們建議使用預設的 `Direct`。  |
| 連線通訊協定 | Azure Cosmos DB 支援兩種連線通訊協定 - `Https` 和 `Tcp`。 `Tcp` 是預設值，也是建議使用的通訊協定，因為它較精簡。 |
| 慣用位置 | 供讀取使用的慣用 (多路連接) 位置逗號分隔清單。 每個 Azure Cosmos DB 帳戶都可以與 1-30 個以上的區域建立關聯。 每個用戶端執行個體可以依慣用順序指定這些區域的子集，供低延遲讀取使用。 指定這些區域時，必須使用其[顯示名稱](https://msdn.microsoft.com/library/azure/gg441293.aspx)，例如 `West US`。 另請參閱[多路連接 API](tutorial-global-distribution-table.md)。 |
| 一致性層級 | 您可以在五個定義完善的一致性層級之間做選擇，以在延遲、一致性及可用性之間做取捨：`Strong`、`Session`、`Bounded-Staleness`、`ConsistentPrefix` 及 `Eventual`。 預設值為 `Session`。 一致性層級的選擇會在多區域設定中造成明顯的效能差異。 如需詳細資料，請參閱[一致性層級](consistency-levels.md)。 |

透過下列 `appSettings` 設定值可啟用其他的功能。

| Key | 說明 |
| --- | --- |
| TableQueryMaxItemCount | 設定單一來回行程中每一資料表查詢所傳回的項目數上限。 預設值為 `-1`，這會讓 Azure Cosmos DB 在執行階段動態決定值。 |
| TableQueryEnableScan | 如果查詢無法針對任何篩選使用索引，則無論如何還是透過掃描執行它。 預設值為 `false`。|
| TableQueryMaxDegreeOfParallelism | 用於執行跨分割區查詢的平行處理程度。 `0` 為循序且不預先擷取，`1` 為循序並預先擷取，而值越高代表平行程度也越高。 預設值為 `-1`，這會讓 Azure Cosmos DB 在執行階段動態決定值。 |

若要變更預設值，請從 Visual Studio 中的 [方案總管] 開啟 `app.config` 檔案。 如下所示，加入 `<appSettings>` 元素的內容。 以您的儲存體帳戶名稱取代 `account-name`，並以您的帳戶存取金鑰取代 `account-key`。 

```xml
<configuration>
    ...
    <appSettings>
      <!-- Client options -->
      <add key="CosmosDBStorageConnectionString" 
        value="DefaultEndpointsProtocol=https;AccountName=MYSTORAGEACCOUNT;AccountKey=AUTHKEY;TableEndpoint=https://account-name.table.cosmosdb.azure.com" />
      <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key; TableEndpoint=https://account-name.documents.azure.com" />

      <!-- Table query options -->
      <add key="TableQueryMaxItemCount" value="-1"/>
      <add key="TableQueryEnableScan" value="false"/>
      <add key="TableQueryMaxDegreeOfParallelism" value="-1"/>
      <add key="TableQueryContinuationTokenLimitInKb" value="16"/>
            
    </appSettings>
</configuration>
```

讓我們快速檢閱應用程式中發生了什麼。 開啟 `Program.cs` 檔案，您會發現這些程式碼建立了「資料表」資源。 

## <a name="create-the-table-client"></a>建立資料表用戶端
您需將 `CloudTableClient` 初始化以連線到資料表帳戶。

```csharp
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
```
將此用戶端初始化時，會使用 `TableConnectionMode`、`TableConnectionProtocol`、`TableConsistencyLevel` 及 `TablePreferredLocations` 組態值 (如果在應用程式設定中指定)。

## <a name="create-a-table"></a>建立資料表
然後，您需使用 `CloudTable` 來建立資料表。 Azure Cosmos DB 中的「資料表」可以就儲存體和輸送量方面獨立調整，而資料分割則是由服務自動處理。 Azure Cosmos DB 同時支援固定大小和無限制的資料表。 如需詳細資料，請參閱 [Azure Cosmos DB 中的資料分割](partition-data.md)。 

```csharp
CloudTable table = tableClient.GetTableReference("people");
400
table.CreateIfNotExists(throughput: 800);
```

在資料表的建立方式方面有重大的差異。 Azure Cosmos DB 會保留輸送量，而不像 Azure 儲存體針對交易是採用以耗用量為基礎的模型。 您的輸送量是專用/已保留的，因此當您的要求率正好在或低於所佈建的輸送量時，一律不會進行節流。

您可以藉由將預設輸送量納入 CreateIfNotExists 的參數，來對其進行設定。

讀值為 1-KB 的實體會標準化為 1 RU，其他作業則會根據其 CPU、記憶體及 IOPS 耗用量來標準化為固定的 RU 值。 深入了解 [Azure Cosmos DB 中的要求單位](request-units.md)，尤其是[索引鍵值存放區](key-value-store-cost.md)。

接著，我們將逐步解說如何使用 Azure 資料表儲存體 SDK 來進行簡單的讀寫 (CRUD) 作業。 本教學課程會示範 Azure Cosmos DB 所提供之可預測的低個位數毫秒延遲和快速查詢。

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表
Azure 資料表儲存體中的實體會從 `TableEntity` 類別延伸，並且必須具有 `PartitionKey` 和 `RowKey` 屬性。 以下是一個客戶實體的範例定義。

```csharp
public class CustomerEntity : TableEntity
{
    public CustomerEntity(string lastName, string firstName)
    {
        this.PartitionKey = lastName;
        this.RowKey = firstName;
    }

    public CustomerEntity() { }

    public string Email { get; set; }

    public string PhoneNumber { get; set; }
}
```

下列程式碼片段說明如何使用 Azure 儲存體 SDK 來插入實體。 Azure Cosmos DB 的設計目的是要為世界各地、不論任何規模都確保低延遲。

針對執行區域在與 Azure Cosmos DB 帳戶相同區域的應用程式，寫入在 P99 是 15 毫秒內完成，在 P50 則是大約 6 毫秒完成。 而這個持續時間說明了一個事實，就是寫入只有在同步複寫、永久認可且所有內容都已編製索引之後，才會認可回用戶端。


```csharp
// Create a new customer entity.
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
customer1.Email = "Walter@contoso.com";
customer1.PhoneNumber = "425-555-0101";

// Create the TableOperation object that inserts the customer entity.
TableOperation insertOperation = TableOperation.Insert(customer1);

// Execute the insert operation.
table.Execute(insertOperation);
```

## <a name="insert-a-batch-of-entities"></a>插入一批實體
Azure 資料表儲存體支援批次作業 API，可讓您將更新、刪除及插入結合在相同批次作業中。

```csharp
// Create the batch operation.
TableBatchOperation batchOperation = new TableBatchOperation();

// Create a customer entity and add it to the table.
CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
customer1.Email = "Jeff@contoso.com";
customer1.PhoneNumber = "425-555-0104";

// Create another customer entity and add it to the table.
CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
customer2.Email = "Ben@contoso.com";
customer2.PhoneNumber = "425-555-0102";

// Add both customer entities to the batch insert operation.
batchOperation.Insert(customer1);
batchOperation.Insert(customer2);

// Execute the batch operation.
table.ExecuteBatch(batchOperation);
```
## <a name="retrieve-a-single-entity"></a>擷取單一實體
相同 Azure 區域內 Azure Cosmos DB 中的 擷取 (GET) 在 P99 是 10 毫秒內完成，在 P50 則是大約 1 毫秒完成。 您可以視所需的區域數量，將多個區域新增到您的帳戶來提供低延遲的讀取，並透過設定 `TablePreferredLocations` 將應用程式部署成從其本機區域 (多路連接) 讀取。 

您可以使用下列程式碼片段來擷取單一實體：

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = table.Execute(retrieveOperation);
```
> [!TIP]
> 若要了解多路連接 API，請參閱[使用多個區域進行開發](tutorial-global-distribution-table.md)
>

## <a name="query-entities-using-automatic-secondary-indexes"></a>使用自動次要索引來查詢實體
您可以使用 `TableQuery` 類別來查詢資料表。 Azure Cosmos DB 具有寫入最佳化資料庫引擎，可自動為您資料表中的所有資料行編製索引。 Azure Cosmos DB 中的索引編製是不驗證結構描述的。 因此，即使資料列之間的結構描述不同，或結構描述隨著時間演變，仍然會自動編製索引。 由於 Azure Cosmos DB 支援自動次要索引，因此對任何屬性的查詢都可使用該索引並有效率地獲得服務。

```csharp
CloudTable table = tableClient.GetTableReference("people");

// Filter against a property that's not partition key or row key
TableQuery<CustomerEntity> emailQuery = new TableQuery<CustomerEntity>().Where(
    TableQuery.GenerateFilterCondition("Email", QueryComparisons.Equal, "Ben@contoso.com"));

foreach (CustomerEntity entity in table.ExecuteQuery(emailQuery))
{
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
}
```

Azure Cosmos DB 支援的「資料表 API」查詢功能與 Azure 資料表儲存體相同。 Azure Cosmos DB 也支援排序、彙總、地理空間查詢、階層及各種不同的內建函式。 在未來的服務更新中，將會在資料表 API 中提供額外的功能。 如需這些功能的概觀，請參閱 [Azure Cosmos DB 查詢](sql-api-sql-query.md)。 

## <a name="replace-an-entity"></a>取代實體
若要更新實體，從資料表服務擷取實體、修改實體物件，然後將變更儲存回資料表服務。 下列程式碼會變更現有客戶的電話號碼。 

```csharp
TableOperation updateOperation = TableOperation.Replace(updateEntity);
table.Execute(updateOperation);
```
同樣地，您也可以執行 `InsertOrMerge` 或 `Merge` 作業。  

## <a name="delete-an-entity"></a>刪除實體
使用更新實體時所展示的相同方法，輕鬆地在擷取實體後將其刪除。 下列程式碼會擷取並刪除客戶實體。

```csharp
TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
table.Execute(deleteOperation);
```

## <a name="delete-a-table"></a>刪除資料表
最後，下列程式碼範例會從儲存體帳戶刪除資料表。 您可使用 Azure Cosmos DB 來立即刪除並重新建立資料表。

```csharp
CloudTable table = tableClient.GetTableReference("people");
table.DeleteIfExists();
```

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，我們涵蓋了如何開始使用 Azure Cosmos DB 搭配「資料表 API」，而您已經完成下列操作： 

> [!div class="checklist"] 
> * 建立 Azure Cosmos DB 帳戶 
> * 在 app.config 檔案中啟用功能 
> * 建立資料表 
> * 將實體新增到資料表 
> * 插入一批實體 
> * 擷取單一實體 
> * 使用自動次要索引來查詢實體 
> * 取代實體 
> * 刪除實體 
> * 刪除資料表  

您現在可以繼續進行到下一個教學課程，以進一步了解如何查詢資料表資料。 

> [!div class="nextstepaction"]
> [使用資料表 API 進行查詢](tutorial-query-table.md)
