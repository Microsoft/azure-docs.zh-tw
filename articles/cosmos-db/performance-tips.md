---
title: Azure Cosmos DB 的 .NET 效能祕訣 | Microsoft Docs
description: 了解用以改善 Azure Cosmos DB 資料庫效能的用戶端設定選項
keywords: 如何改善資料庫效能
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 01/24/2018
ms.author: sngun
ms.openlocfilehash: a805294ecb416d18f3ce13981d26a7d25cd5a204
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432846"
---
# <a name="performance-tips-for-azure-cosmos-db-and-net"></a>Azure Cosmos DB 和 .NET 的效能祕訣

> [!div class="op_single_selector"]
> * [非同步 Java](performance-tips-async-java.md)
> * [Java](performance-tips-java.md)
> * [.NET](performance-tips.md)
> 

Azure Cosmos DB 是一個既快速又彈性的分散式資料庫，可在獲得延遲與輸送量保證的情況下順暢地調整。 使用 Azure Cosmos DB 時，您不須進行主要的架構變更，或是撰寫複雜的程式碼來調整您的資料庫。 相應增加和減少就像進行單一 API 呼叫或 [SDK 方法呼叫](set-throughput.md#set-throughput-sdk)一樣簡單。 不過，由於 Azure Cosmos DB 是透過網路呼叫存取，所以您可以在使用 [SQL .NET SDK](documentdb-sdk-dotnet.md) 時進行用戶端最佳化以達到最高效能。

如果您詢問「如何改善我的資料庫效能？ 」，請考慮下列選項：

## <a name="networking"></a>網路功能
<a id="direct-connection"></a>

1. **原則︰使用直接連接模式**

    用戶端連線到 Azure Cosmos DB 的方式，對於效能有重大影響 (尤其對觀察到的用戶端延遲而言)。 有兩個重要組態設定可用來設定用戶端連接原則 - 連接模式和連接[*通訊協定*](#connection-protocol)。  兩個可用的模式︰

   * 閘道模式 (預設值)
      
     所有 SDK 平台都支援閘道模式並設為預設值。 如果您的應用程式在有嚴格防火牆限制的公司網路中執行，則閘道模式會是最佳的選擇，因為它會使用標準 HTTPS 連接埠與單一端點。 不過，對於效能的影響是每次讀取或寫入 Azure Cosmos DB 資料時，閘道模式都會涉及額外的網路躍點。 因此，直接模式因為網路躍點較少，所以可提供較佳的效能。 此外，當您在通訊端連線數目有限的環境中執行應用程式時，也建議使用閘道連線模式，例如，使用 Azure Functions 時，或當您處於使用情況方案時。 

   * 直接模式

     直接模式支援透過 TCP 和 HTTPS 通訊協定連線。 目前，只有 .NET Standard 2.0 支援直接模式。 使用直接模式時，有兩個可用的通訊協定選項：

    * TCP
    * HTTPS

    使用閘道模式時，Azure Cosmos DB 會使用連接埠 443，而 MongoDB API 則會使用 10250、10255 和 10256 連接埠。 10250 連接埠會直接對應至預設 Mongodb 執行個體而不使用異地複寫，10255/10256 連接埠則會使用異地複寫功能對應至 Mongodb 執行個體。 在直接模式下使用 TCP 時，除了「閘道」連接埠之外，您還務必要開啟 10000 到 20000 之間的連接埠範圍，因為 Azure Cosmos DB 使用動態 TCP 連接埠。 如果未開啟這些連接埠而您嘗試使用 TCP，您就會收到「503 服務無法使用」錯誤。 下表顯示不同 API 的可用連線模式，以及每個 API 的服務連接埠使用者：

    |連線模式  |支援的通訊協定  |支援的 SDK  |API/服務連接埠  |
    |---------|---------|---------|---------|
    |閘道器  |   HTTPS    |  所有 SDK    |   SQL(443)、Mongo(10250, 10255, 10256)、Table(443)、Cassandra(443)、Graph(443)    |
    |直接    |    HTTPS     |  .Net 和 Java SDK    |    SQL(443)   |
    |直接    |     TCP    |  .Net SDK    | 10,000-20,000 範圍內的連接埠 |

    Azure Cosmos DB 提供透過 HTTPS 的簡單且開放 RESTful 程式設計模型。 此外，它可提供有效率的 TCP 通訊協定，此 TCP 通訊協定在通訊模型中也符合 REST 限制，並且可以透過 .NET 用戶端 SDK 取得。 直接 TCP 和 HTTPS 皆使用 SSL 來進行初始驗證和加密流量。 為了達到最佳效能，儘可能使用 TCP 通訊協定。

    連接模式設定於使用 ConnectionPolicy 參數建構 DocumentClient 執行個體期間。 如果使用直接模式，也可以在 ConnectionPolicy 參數內設定 Protocol。

    ```csharp
    var serviceEndpoint = new Uri("https://contoso.documents.net");
    var authKey = new "your authKey from the Azure portal";
    DocumentClient client = new DocumentClient(serviceEndpoint, authKey,
    new ConnectionPolicy
    {
        ConnectionMode = ConnectionMode.Direct,
        ConnectionProtocol = Protocol.Tcp
    });
    ```

    因為只有直接模式支援 TCP，所以如果使用閘道模式，則 HTTPS 通訊協定一律用來與閘道通訊，並忽略 ConnectionPolicy 中的 Protocol 值。

    ![Azure Cosmos DB 連接原則的圖例](./media/performance-tips/connection-policy.png)

2. **呼叫 OpenAsync 以避免第一次要求的啟動延遲**

    根據預設，第一個要求會因為必須擷取位址路由表而有較高的延遲。 若要避免此第一次要求的啟動延遲，您應該在初始化期間呼叫 OpenAsync() 一次，如下所示。

        await client.OpenAsync();
   <a id="same-region"></a>
3. **為了效能在相同 Azure 區域中共置用戶端**

    可能的話，請將任何呼叫 Azure Cosmos DB 的應用程式放在與 Azure Cosmos DB 資料庫相同的區域中。 以約略的比較來說，在相同區域內對 Azure Cosmos DB 進行的呼叫會在 1-2 毫秒內完成，但美國西岸和美國東岸之間的延遲則會大於 50 毫秒。 視要求所採用的路由而定，各項要求從用戶端傳遞至 Azure 資料中心界限時的這類延遲可能有所不同。 確保呼叫端應用程式與佈建的 Azure Cosmos DB 端點位於相同的 Azure 區域中，將可能達到最低的延遲。 如需可用區域的清單，請參閱 [Azure 區域](https://azure.microsoft.com/regions/#services)。

    ![Azure Cosmos DB 連接原則的圖例](./media/performance-tips/same-region.png)
   <a id="increase-threads"></a>
4. **增加執行緒/工作數目**

    由於對 Azure Cosmos DB 的呼叫要透過網路進行，因此您可能需要改變要求的平行處理原則程度，以便讓用戶端應用程式在不同要求之間只需等待很短的時間。 例如，如果您使用 .NET 的[工作平行程式庫](https://msdn.microsoft.com//library/dd460717.aspx)，請建立大約數百個讀取或寫入 Azure Cosmos DB 的工作。

## <a name="sdk-usage"></a>SDK 的使用方式
1. **安裝最新的 SDK**

    Azure Cosmos DB SDK 會持續改善以提供最佳效能。 請參閱 [Azure Cosmos DB SDK](documentdb-sdk-dotnet.md) 頁面來判斷最新的 SDK 並檢閱改善項目。
2. **在應用程式存留期內使用單一 Azure Cosmos DB 用戶端**

    每個 DocumentClient 執行個體都是安全執行緒，並且在直接模式中運作時執行有效率的連線管理和位址快取。 若要藉由 DocumentClient 獲得有效率的連接管理和更佳的效能，建議在應用程式存留期內對每個 AppDomain 使用單一 DocumentClient 執行個體。

   <a id="max-connection"></a>
3. **增加使用閘道模式時每部主機的 System.Net MaxConnections**

    使用閘道模式時，Azure Cosmos DB 要求是透過 HTTPS/REST 發出，並受制於每個主機名稱或 IP 位址的預設連線限制。 您可能必須將 MaxConnections 設定成較高的值 (100-1000)，以便用戶端程式庫利用多個同時連到 Azure Cosmos DB 的連線。 在 .NET SDK 1.8.0 和更新版本中，[ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit.aspx) 的預設值為 50，而若要變更此值，您可以將 [Documents.Client.ConnectionPolicy.MaxConnectionLimit](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.maxconnectionlimit.aspx) 設定為更高的值。   
4. **微調分割之集合的平行查詢**

     SQL .NET SDK 1.9.0 版和更新版本支援平行查詢，可讓您平行查詢分割的集合 (詳細資訊請參閱[使用 SDK](sql-api-partition-data.md#working-with-the-azure-cosmos-db-sdks) 和相關的[程式碼範例](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Queries/Program.cs))。 平行查詢的設計目的是要改善其連續對應項目的查詢延遲和輸送量。 平行查詢提供兩個可供使用者微調以符合其需求的參數：(a) MaxDegreeOfParallelism：用來控制可平行查詢的分割數目上限，以及 (b) MaxBufferedItemCount：用來控制預先擷取的結果數目。

    (a) ***微調 MaxDegreeOfParallelism\:*** 平行查詢的運作方式是以平行方式查詢多個分割。 不過，對於查詢會以循序方式擷取來自個別分割集合的資料。 因此，將 MaxDegreeOfParallelism 設定為分割數目會最有機會達到最高效能的查詢，但前提是其他所有系統條件皆維持不變。 如果您不知道分割數目，您可以將 MaxDegreeOfParallelism 設定為較高的數字，然後系統會選擇最小值 (資料分割數目、使用者提供的輸入值) 做為 MaxDegreeOfParallelism。

    請務必注意，若對於查詢是以平均方式將資料分佈於所有分割，平行查詢便會產生最佳效益。 如果分割之集合的分割方式是查詢所傳回的所有或大多數資料集中在少數幾個分割中 (最差的情況是集中在一個分割)，則這些分割會成為查詢效能的瓶頸。

    (b) ***微調 MaxBufferedItemCount\:*** 平行查詢是設計成可以在用戶端處理目前的結果批次時預先擷取結果。 預先擷取有助於改善查詢的整體延遲。 MaxBufferedItemCount 是用來限制預先擷取之結果數量的參數。 將 MaxBufferedItemCount 設定為預期傳回的結果數目 (或更高的數目)，可讓查詢透過預先擷取獲得最大效益。

    預先擷取會以相同方式運作，不受 MaxDegreeOfParallelism 的影響，而且來自所有分割的資料會有單一緩衝區。  
5. **開啟伺服器端 GC**

    在某些情況下，降低廢棄項目收集頻率可能會有幫助。 在 .NET 中，請將 [gcServer](https://msdn.microsoft.com/library/ms229357.aspx) 設定為 true。
6. **在 RetryAfter 間隔實作降速**

    在進行效能測試期間，您應該增加負載，直到系統對小部分要求進行節流處理為止。 如果進行節流處理，用戶端應用程式應該在節流時降速，且持續時間達伺服器指定的重試間隔。 採用降速可確保您在重試之間花費最少的等待時間。 重試原則支援包含於 SQL [.NET](sql-api-sdk-dotnet.md) 和 [Java](sql-api-sdk-java.md) 的版本 1.8.0 和以上版本中，[Node.js](sql-api-sdk-node.md) 和 [Python](sql-api-sdk-python.md) 的版本 1.9.0 或以上版本中，以及 [.NET 核心](sql-api-sdk-dotnet-core.md) SDK 所有支援的版本。 如需詳細資訊，請參閱[超過保留的輸送量限制](request-units.md#RequestRateTooLarge)和 [RetryAfter](https://msdn.microsoft.com/library/microsoft.azure.documents.documentclientexception.retryafter.aspx)。
    
    使用 .NET SDK 1.19 版和更新版本時，有一個機制可以記錄診斷資訊，並且針對延遲問題進行移難排解，如以下範例所示。 您可以針對具有較高讀取延遲的要求記錄診斷字串。 所擷取的診斷字串有助於您了解針對指定要求觀察 429s 的次數。
    ```csharp
    ResourceResponse<Document> readDocument = await this.readClient.ReadDocumentAsync(oldDocuments[i].SelfLink);
    readDocument.RequestDiagnosticsString 
    ```
    
7. **相應放大用戶端工作負載**

    如果您是以高輸送量層級 (> 50,000 RU/秒) 進行測試，用戶端應用程式可能會成為瓶頸，因為電腦對 CPU 或網路的使用率將達到上限。 如果到了這一刻，您可以將用戶端應用程式向外延展至多部伺服器，以繼續將 Azure Cosmos DB 帳戶再往前推進一步。
8. **快取較低讀取延遲的文件 URI**

    盡可能快取文件 URI 以達到最佳讀取效能。 您必須在建立資源時，定義快取 resourceid 的邏輯。 以 Resourceid 為基礎的查閱比以名稱為基礎的查閱更快，因此快取這些值可改善效能。 

   <a id="tune-page-size"></a>
1. **調整查詢/讀取摘要的頁面大小以獲得更好的效能**

    使用讀取摘要功能 (例如 ReadDocumentFeedAsync) 執行大量文件讀取時，或發出 SQL 查詢時，如果結果集太大，則會以分段方式傳回結果。 根據預設，會以 100 個項目或 1 MB 的區塊傳回結果 (以先達到的限制為準)。

    若要減少擷取所有適用結果所需的網路來回行程次數，您可以使用 [x-ms-max-item-count](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) 要求標頭將頁面大小最多增加至 1000。 在您只需要顯示幾個結果的情況下 (例如，您的使用者介面或應用程式 API 一次只傳回 10 筆結果)，您也可以將頁面大小縮小為 10，以降低讀取和查詢所耗用的輸送量。

    您也可以使用可用的 Azure Cosmos DB SDK 設定頁面大小。  例如︰

        IQueryable<dynamic> authorResults = client.CreateDocumentQuery(documentCollection.SelfLink, "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", new FeedOptions { MaxItemCount = 1000 });
10. **增加執行緒/工作數目**

    請參閱＜網路＞一節中的[增加執行緒/工作數目](#increase-threads)。

11. **使用 64 位元主機處理序**

    當您使用 SQL .NET SDK 版本 1.11.4 和更新版本時，SQL SDK 會在 32 位元主機處理序中運作。 不過，若使用跨分割區查詢，建議您使用 64 位元主機處理以獲得改進的效能。 下列的應用程式類型預設使用 32 位元主機處理序，若要將其變更為 64 位元，請根據您的應用程式類型依照下列步驟執行：

    - 針對「可執行檔」應用程式，做法是在 [專案屬性] 視窗中的 [建置] 索引標籤上取消選取 [建議使用 32 位元] 選項。

    - 針對 VSTest 型的測試專案，可以從 [Visual Studio 測試] 功能表選項，選取 [測試]->[測試設定]->[以 X64 做為預設處理器架構] 來完成。

    - 針對本機部署的 ASP.NET Web 應用程式，可以在 [工具]->[選項]->[專案和方案]->[Web 專案] 之下，選取 [將 64 位元版本的 IIS Express 用於網站和專案] 來完成。

    - 針對部署於 Azure 上的 ASP.NET Web 應用程式，可以在 Azure 入口網站上的 [應用程式設定] 中選擇 [以 64 位元做為平台] 來完成。

## <a name="indexing-policy"></a>索引原則
 
1. **從索引編製中排除未使用的路徑以加快寫入速度**

    Cosmos DB 的索引編製原則也可讓您利用檢索路徑 (IndexingPolicy.IncludedPaths 和 IndexingPolicy.ExcludedPaths)，指定要在索引編製中包含或排除的文件路徑。 在事先知道查詢模式的案例中，使用檢索路徑可改善寫入效能並降低索引儲存空間，因為檢索成本與檢索的唯一路徑數目直接相互關聯。  例如，以下程式碼示範如何將文件的整個區段 (也稱為 樹狀子目錄) 自索引編製作業中排除 (透過使用 "*" 萬用字元)。

    ```csharp
    var collection = new DocumentCollection { Id = "excludedPathCollection" };
    collection.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
    collection.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*");
    collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), excluded);
    ```

    如需詳細資訊，請參閱 [Azure Cosmos DB 索引編製原則](indexing-policies.md)。

## <a name="throughput"></a>Throughput
<a id="measure-rus"></a>

1. **測量和調整較低的要求單位/秒使用量**

    Azure Cosmos DB 提供許多的資料庫作業，包括使用 UDF、預存程序和觸發程序進行關聯式和階層式查詢，而這些作業全都是對資料庫集合內的文件來進行。 與上述各項作業相關聯的成本，會因為完成作業所需的 CPU、IO 和記憶體而不同。 您不需要考慮和管理硬體資源，您可以將要求單位 (RU) 想成是執行各種資料庫作業以及服務應用程式要求時所需的資源數量。

    輸送量是根據為每個容器所設定的[要求單位](request-units.md)數量來佈建。 要求單位消耗量是以每秒的速率來計算。 如果應用程式的速率超過為其容器佈建的要求單位速率，便會受到限制，直到該速率降到容器的佈建層級以下。 如果您的應用程式需要較高的輸送量，您可以藉由佈建其他的要求單位來增加輸送量。 

    查詢的複雜性會影響針對作業所耗用的要求單位數量。 述詞數目、述詞性質、UDF 數目，以及來源資料集的大小，全都會影響查詢作業的成本。

    若要測量任何作業 (建立、更新或刪除) 的負荷，請檢查 [x-ms-request-charge](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) 標頭 (或 .NET SDK 的 ResourceResponse<T> 或 FeedResponse<T> 中同等的 RequestCharge 屬性) 來測量這些作業所耗用的要求單位數量。

    ```csharp
    // Measure the performance (request units) of writes
    ResourceResponse<Document> response = await client.CreateDocumentAsync(collectionSelfLink, myDocument);
    Console.WriteLine("Insert of document consumed {0} request units", response.RequestCharge);
    // Measure the performance (request units) of queries
    IDocumentQuery<dynamic> queryable = client.CreateDocumentQuery(collectionSelfLink, queryString).AsDocumentQuery();
    while (queryable.HasMoreResults)
         {
              FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>();
              Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge);
         }
    ```             

    在此標頭中傳回的要求費用是佈建輸送量的一小部分 (也就是 2000 RU / 秒)。 例如，如果前述查詢傳回 1000 份 1KB 文件，則作業成本會是 1000。 因此在一秒內，伺服器在對後續要求進行速率限制前，只會接受兩個這類要求。 如需詳細資訊，請參閱[要求單位](request-units.md)和[要求單位計算機](https://www.documentdb.com/capacityplanner)。
<a id="429"></a>
2. **處理速率限制/要求速率太大**

    當用戶端嘗試超過帳戶保留的輸送量時，伺服器的效能不會降低，而且不會使用超過保留層級的輸送量容量。 伺服器將預先使用 RequestRateTooLarge (HTTP 狀態碼 429) 來結束要求，並傳回 [x-ms-retry-after-ms](https://docs.microsoft.com/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) 標頭，以指出使用者重試要求之前必須等候的時間量 (毫秒)。

        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK 全都隱含地攔截這個回應，採用伺服器指定的 retry-after 標頭，並重試此要求。 除非有多個用戶端同時存取您的帳戶，否則下次重試將會成功。

    如果您有多個用戶端都以高於要求速率的方式累積運作，則用戶端在內部設定為 9 的預設重試計數可能會不敷使用；在此情況下，用戶端會擲回 DocumentClientException (狀態碼 429) 到應用程式。 在 ConnectionPolicy 執行個體上設定 RetryOptions，即可變更預設重試次數。 根據預設，如果要求繼續以高於要求速率的方式運作，則會在 30 秒的累計等候時間後傳回 DocumentClientException (狀態碼 429)。 即使目前的重試計數小於最大重試計數 (預設值 9 或使用者定義的值)，也會發生這種情況。

    雖然自動重試行為有助於改善大部分應用程式的恢復功能和可用性，但是在進行效能基準測試時可能會有所歧異 (尤其是在測量延遲時)。  如果實驗達到伺服器節流並導致用戶端 SDK 以無訊息模式重試，則用戶端觀察到的延遲將會突然增加。 若要避免效能實驗期間的延遲尖峰，測量每個作業所傳回的費用，並確保要求是以低於保留要求速率的方式運作。 如需詳細資訊，請參閱 [要求單位](request-units.md)。
3. **輸送量較高之少量文件的設計**

    指定之作業的要求費用 (也就是要求處理成本) 與文件大小直接相互關聯。 大型文件的作業成本高於小型文件的作業成本。

## <a name="next-steps"></a>後續步驟
如需用來評估 Azure Cosmos DB 在少數用戶端電腦上達到高效能的範例應用程式，請參閱 [Azure Cosmos DB 的效能和規模測試](performance-testing.md)。

此外，若要深入了解如何針對規模和高效能設計您的應用程式，請參閱 [Azure Cosmos DB 的資料分割與調整規模](partition-data.md)。
