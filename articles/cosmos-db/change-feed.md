---
title: 使用 Azure Cosmos DB 中的變更摘要支援 | Microsoft Docs
description: 使用 Azure Cosmos DB 的變更摘要支援來追蹤文件中的變更，並執行以事件為基礎的處理 (例如觸發程序)，以及讓快取和分析系統保持最新狀態。
keywords: 變更摘要
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: rafats
ms.openlocfilehash: 3170ee1b48aa332a8730ba835396761ca5ef44c7
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287320"
---
# <a name="working-with-the-change-feed-support-in-azure-cosmos-db"></a>使用 Azure Cosmos DB 中的變更摘要支援

[Azure Cosmos DB](../cosmos-db/introduction.md) 是彈性、快速的全域複寫資料庫，適合用於 IoT、遊戲、零售，以及操作記錄應用程式。 這類應用程式常用的設計模式，是利用資料的變更啟動其他動作的進行。 其他的動作可能是下列其中一項： 

* 於插入或修改文件時觸發 API 通知或呼叫。
* 進行 IoT 資料流處理或執行分析。
* 藉由與快取、搜尋引擎、或資料倉儲同步處理，或將資料封存到冷儲存體，進行額外的資料移動。

Azure Cosmos DB 中的**變更摘要支援**允許您針對每一個模式建置有效率且可調整的解決方案，如下圖所示：

![使用 Azure Cosmos DB 變更摘要來提供即時分析和事件導向的計算案例](./media/change-feed/changefeedoverview.png)

> [!NOTE]
> Azure Cosmos DB 中的所有資料模型和容器均可使用變更摘要支援。 不過，變更摘要是利用 SQL 用戶端來讀取，並可將項目序列化為 JSON 格式。 因為是 JSON 格式，MongoDB 用戶端會遇到 BSON 格式文件與 JSON 格式變更摘要互不相符的情況。

## <a name="how-does-change-feed-work"></a>變更摘要如何運作？

Azure Cosmos DB 中的變更摘要支援是靠接聽 Azure Cosmos DB 集合的任何變更而運作。 然後變更摘要會輸出已排序的文件清單，這些文件已依其修改的順序變更過。 變更會保存，可進行非同步累加處理，而輸出可配送給一或多個取用者進行平行處理。 

後文會說明，您可以三個不同的方式讀取變更摘要：

*   [使用 Azure Functions](#azure-functions)
*   [使用 Azure Cosmos DB SDK](#sql-sdk)
*   [使用 Azure Cosmos DB 變更摘要處理器程式庫](#change-feed-processor)

文件集合內每個分割區索引鍵範圍都可使用變更摘要，因此可以配送給一或多個取用者以進行平行處理，如下圖所示。

![Azure Cosmos DB 變更摘要的分散式處理](./media/change-feed/changefeedvisual.png)

其他詳細資料：
* 所有帳戶預設都會啟用 變更摘要。
* 和任何其他 Azure Cosmos DB 作業一樣，您可以在寫入區域或任何[讀取區域](distribute-data-globally.md)中使用[佈建輸送量](request-units.md)來讀取變更摘要。
* 變更摘要包含對集合內文件進行的插入與更新作業。 您可以擷取刪除項目，方法是在您的文件內設定「虛刪除」旗標來取代刪除動作。 或者，您可以透過 [TTL 功能](time-to-live.md)針對您的文件設定有限的逾期期限 (例如 24 小時)，並使用該屬性的值擷取刪除項目。 使用此解決方案，您必須在比 TTL 逾期期限更短的時間間隔內處理變更。
* 對文件所做的每項變更皆會在變更摘要中出現一次，且用戶端會管理其檢查點邏輯。 變更摘要的處理器程式庫提供自動檢查點和「至少一次」語意。
* 變更記錄檔中只會包含指定文件的最新變更。 中繼變更可能無法使用。
* 變更摘要會依照各個資料分割索引鍵值內的修改順序排序。 跨資料分割索引鍵值順序不會是固定的。
* 變更可以從任何時間點同步處理，也就是說，可用變更沒有固定的資料保留期限。
* 變更會以資料分割索引鍵區塊為單位提供。 這項功能可讓大型集合的變更由多個取用者/伺服器平行處理。
* 應用程式可以同時要求同一個集合的多個變更摘要。
* ChangeFeedOptions.StartTime 可用來提供初始的起點，例如，用來尋找對應到指定時鐘時間的接續權杖。 ContinuationToken 如經指定，會優先於 StartTime 和 StartFromBeginning 值。 ChangeFeedOptions.StartTime 的精確度為 5 秒內。 

## <a name="use-cases-and-scenarios"></a>使用個案和案例

變更摘要利用大量寫入讓大型資料集的處理更有效率，是查詢整個資料集以識別已變更之項目的另一種做法。 

例如，您可以利用變更摘要有效率地執行下列工作︰

* 透過儲存在 Azure Cosmos DB 中的資料，來更新快取、搜尋索引或資料倉儲。
* 實作應用程式層級的資料階層處理和封存，也就是在 Azure Cosmos DB 中儲存「熱資料」，並將「冷資料」淘汰到 [Azure Blob 儲存體](../storage/common/storage-introduction.md)或 [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)。
* 透過不同的分割配置，執行零停機時間移轉至另一個 Azure Cosmos DB 帳戶。
* 透過 Azure Cosmos DB [在 Azure 上實作 Lambda 管線 (英文)](https://blogs.technet.microsoft.com/msuspartner/2016/01/27/azure-partner-community-big-data-advanced-analytics-and-lambda-architecture/)。 Azure Cosmos DB 提供一個可調整的資料庫解決方案，可以處理擷取和查詢，並實作具有低 TCO 的 Lambda 架構。 
* 接收與儲存來自裝置、感應器、基礎結構和應用程式的事件資料，並透過 [Azure 串流分析](../stream-analytics/stream-analytics-documentdb-output.md)、[Apache Storm](../hdinsight/storm/apache-storm-overview.md) 或 [Apache Spark](../hdinsight/spark/apache-spark-overview.md) 即時處理這些事件。 

下圖顯示利用 Azure Cosmos DB 進行擷取和查詢的 lambda 管線，如何使用變更摘要支援： 

![運用 Azure Cosmos DB 進行擷取和查詢的 lambda 管線](./media/change-feed/lambda.png)

此外，在您的 [serverless](http://azure.com/serverless) Web 和行動應用程式內，您可以追蹤例如變更客戶設定檔、喜好設定或位置等的事件，以觸發像是使用 [Azure Functions](#azure-functions) 傳送推播通知到客戶裝置的特定動作。 例如，如果您使用 Azure Cosmos DB 來建置遊戲，就可以根據完成遊戲的分數，使用變更摘要來實作即時排行榜。

<a id="azure-functions"></a>
## <a name="using-azure-functions"></a>使用 Azure Functions 

如果您使用 Azure Functions，要與 Azure Cosmos DB 變更摘要連線最簡單的方式，是在您的 Azure Functions 應用程式新增 Azure Cosmos DB 觸發程序。 當您在 Azure Functions 應用程式中 建立 Azure Cosmos DB 觸發程序時，需選取要連線的 Azure Cosmos DB 集合，以及集合每次變更時要觸發的函式。 

您可以在 Azure Functions 入口網站中、在 Azure Cosmos DB 入口網站中建立觸發程序，或以程式設計方式建立。 如需詳細資訊，請參閱 [Azure Cosmos DB：使用 Azure Functions 的無伺服器資料庫計算](serverless-computing-database.md)。

<a id="sql-sdk"></a>
## <a name="using-the-sdk"></a>使用 SDK

Azure Cosmos DB 使用的 [SQL SDK](sql-api-sdk-dotnet.md) 提供讀取和管理變更摘要的所有功能。 但是，能力愈強責任愈多。 如果您想要管理檢查點、處理文件序號，並能夠更精確地控制分割區索引鍵，那麼，使用 SDK 可能是正確的方法。

本節逐步解說如何使用 SQL SDK 運用變更摘要。

1. 首先，讀取 appconfig 中的下列資源。 擷取端點和授權金鑰的指示，可於[更新連接字串](create-sql-api-dotnet.md#update-your-connection-string)中找到。

    ``` csharp
    DocumentClient client;
    string DatabaseName = ConfigurationManager.AppSettings["database"];
    string CollectionName = ConfigurationManager.AppSettings["collection"];
    string endpointUrl = ConfigurationManager.AppSettings["endpoint"];
    string authorizationKey = ConfigurationManager.AppSettings["authKey"];
    ```

2. 遵循下列方式建立用戶端：

    ```csharp
    using (client = new DocumentClient(new Uri(endpointUrl), authorizationKey,
    new ConnectionPolicy { ConnectionMode = ConnectionMode.Direct, ConnectionProtocol = Protocol.Tcp }))
    {
    }
    ```

3. 取得分割區索引鍵範圍：

    ```csharp
    FeedResponse pkRangesResponse = await client.ReadPartitionKeyRangeFeedAsync(
        collectionUri,
        new FeedOptions
            {RequestContinuation = pkRangesResponseContinuation });
     
    partitionKeyRanges.AddRange(pkRangesResponse);
    pkRangesResponseContinuation = pkRangesResponse.ResponseContinuation;
    ```

4. 為每個分割區索引鍵範圍呼叫 ExecuteNextAsync：

    ```csharp
    foreach (PartitionKeyRange pkRange in partitionKeyRanges){
        string continuation = null;
        checkpoints.TryGetValue(pkRange.Id, out continuation);
        IDocumentQuery<Document> query = client.CreateDocumentChangeFeedQuery(
            collectionUri,
            new ChangeFeedOptions
            {
                PartitionKeyRangeId = pkRange.Id,
                StartFromBeginning = true,
                RequestContinuation = continuation,
                MaxItemCount = -1,
                // Set reading time: only show change feed results modified since StartTime
                StartTime = DateTime.Now - TimeSpan.FromSeconds(30)
            });
        while (query.HasMoreResults)
            {
                FeedResponse<dynamic> readChangesResponse = query.ExecuteNextAsync<dynamic>().Result;
    
                foreach (dynamic changedDocument in readChangesResponse)
                    {
                         Console.WriteLine("document: {0}", changedDocument);
                    }
                checkpoints[pkRange.Id] = readChangesResponse.ResponseContinuation;
            }
    }
    ```

> [!NOTE]
> 並非 `ChangeFeedOptions.PartitionKeyRangeId`，您可以使用 `ChangeFeedOptions.PartitionKey` 來指定要取得變更摘要的單一分割區索引鍵。 例如： `PartitionKey = new PartitionKey("D8CFA2FD-486A-4F3E-8EA6-F3AA94E5BD44")`。
> 
>

如果您有多個讀取器，可以使用 **ChangeFeedOptions** 將讀取負載分配至不同的執行緒或不同的用戶端。

就這麼簡單，只要這幾行程式碼，您便可以開始讀取變更摘要。 您可以從 [GitHub 儲存機制](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples/ChangeFeed)取得本文中使用的完整程式碼。

在上述的步驟 4 程式碼中，最後一行的 **ResponseContinuation** 中有最後一個邏輯序號 (LSN)，下一次您讀取新文件時會用到這個序號 (新文件在這個序號之後)。 利用 **ChangeFeedOption** 的 **StartTime**，可以加大取得文件的範圍。 因此，如果您的 **ResponseContinuation** 是 null，但 **StartTime** 是過去的時間，則會取得自 **StartTime** 開始變更過的所有文件。 但是，如果 **ResponseContinuation** 有值，則系統會取得自 LSN 開始的所有文件。

因此，您的檢查點陣列只保留每個分割區的 LSN。 但是，如果您不想處理分割區、檢查點、LSN、開始時間等，更簡單的做法是使用變更摘要處理器程式庫。

<a id="change-feed-processor"></a>
## <a name="using-the-change-feed-processor-library"></a>使用變更摘要處理器程式庫 

[Azure Cosmos DB 變更摘要處理器程式庫](https://docs.microsoft.com/azure/cosmos-db/sql-api-sdk-dotnet-changefeed)可幫助您輕鬆地將事件處理分散給多個取用者。 此程式庫會簡化跨分割區和多個平行運作執行緒上的變更讀取。

變更摘要處理器程式庫的主要優點是您不需要管理每個分割區和接續權杖，也不需要手動輪詢每個集合。

變更摘要處理器程式庫可以簡化跨分割區和多個平行運作執行緒上的變更讀取。  它會使用租用機制自動管理所有分割區上的變更讀取。 如您在下圖中所見，如果您啟動了使用變更摘要處理器程式庫的兩個用戶端，它們會在彼此之中分割工作。 當您繼續增加用戶端，它們會繼續在彼此之中分割工作。

![Azure Cosmos DB 變更摘要的分散式處理](./media/change-feed/change-feed-output.png)

先啟動左邊的用戶端，它開始並啟監視所有分割區，接著啟動第二個用戶端，然後第一個將部份放租用釋出給第二個用戶端。 如您所見，這是在不同電腦與用戶端之間分散工作的好方法。

請注意，如果您有兩個無伺服器的 Azure 函式使用相同的租用在監視相同的集合，則這兩個函式可能會得到不同的文件，取決於處理器程式庫決定如何處理分割區。

<a id="understand-cf"></a>
### <a name="understanding-the-change-feed-processor-library"></a>了解變更摘要處理器程式庫

用來實作變更摘要處理器程式庫的主要元件有四個：受監視的集合、租用集合、處理器主機和取用者。 

> [!WARNING]
> 建立集合會牽涉到定價，因為您會將輸送量保留供應用程式與 Azure Cosmos DB 通訊使用。 如需詳細資訊，請瀏覽[定價頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)
> 
> 

**受監視的集合：** 受監視的集合是將會產生變更摘要的資料。 對於受監視集合所進行的任何插入和變更都會反映在集合的變更摘要中。 

**租用集合：** 租用集合會協調如何處理多個背景工作角色的變更摘要。 另外會有一個集合用來儲存租用 (每個分割區一個租用)。 在不同帳戶儲存此租用集合，並讓其寫入區域更靠近變更摘要處理器的執行所在位置會有好處。 租用物件包含下列屬性： 
* 擁有者：指定擁有租用的主機
* 接續：指定特定分割區在變更摘要中的位置 (接續權杖)
* 時間戳記：上次更新租用的時間；時間戳記可用來檢查是否將租用視為過期 

**處理器主機：** 每一部主機都會根據主機的其他執行個體中有多少個執行個體擁有作用中租用，來決定要處理的分割區數量。 
1.  主機在啟動時會取得租用，以平衡分配所有主機的工作負載。 主機會定期更新租用，以便讓租用保持作用中狀態。 
2.  主機會對其每個讀取的租用設置最後一個接續權杖的檢查點。 為確保能夠安全地進行並行存取，主機會檢查每個租用更新的 ETag。 主機也支援其他檢查點策略。  
3.  關機後，主機會將所有租用釋出，但會保留接續資訊，以便之後能夠繼續讀取預存的檢查點。 

目前，主機數目不能大於分割區 (租用) 數目。

**取用者：** 取用者 (或背景工作角色) 是負責執行每個主機所起始之變更摘要處理活動的執行緒。 每個處理器主機都可以有多個取用者。 每個取用者會從其獲派到的分割區讀取變更摘要，並向其主機通知所發生的變更和已過期的租用。

為了進一步了解變更摘要處理器的這四個元素是如何一起運作的，我們來看看下圖中的範例。 受監視的集合會儲存文件，並使用 "city" 來作為分割區索引鍵。 我們可以看到，藍色的分割區包含 "city" 欄位為 "A 到 E" 的文件，依此類推。 主機有兩部，每一部都有兩個取用者從四個分割區進行平行讀取。 箭號顯示從變更摘要中的特定地點進行讀取的取用者。 在第一個分割區中，較深的藍色代表未讀取的變更，較淺的藍色則代表變更摘要上已讀取的變更。 主機會使用租用集合來儲存「接續」值，以追蹤每個取用者目前的讀取位置。 

![使用 Azure Cosmos DB 變更摘要處理器主機](./media/change-feed/changefeedprocessornew.png)

### <a name="working-with-the-change-feed-processor-library"></a>使用變更摘要處理器程式庫

在安裝變更摘要處理器 NuGet 套件之前，請先安裝： 

* Microsoft.Azure.DocumentD (最新版本)。
* Newtonsoft.Json (最新版本)

然後安裝 [Microsoft.Azure.DocumentDB.ChangeFeedProcessor Nuget 套件](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.ChangeFeedProcessor/)並將它加入參考。

若要實作變更摘要處理器程式庫，您必須要：

1. 實作 **DocumentFeedObserver** 物件，它會實作 **IChangeFeedObserver**。
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Documents;
    using Microsoft.Azure.Documents.ChangeFeedProcessor.FeedProcessing;
    using Microsoft.Azure.Documents.Client;

    /// <summary>
    /// This class implements the IChangeFeedObserver interface and is used to observe 
    /// changes on change feed. ChangeFeedEventHost will create as many instances of 
    /// this class as needed. 
    /// </summary>
    public class DocumentFeedObserver : IChangeFeedObserver
    {
    private static int totalDocs = 0;

        /// <summary>
        /// Initializes a new instance of the <see cref="DocumentFeedObserver" /> class.
        /// Saves input DocumentClient and DocumentCollectionInfo parameters to class fields
        /// </summary>
        /// <param name="client"> Client connected to destination collection </param>
        /// <param name="destCollInfo"> Destination collection information </param>
        public DocumentFeedObserver()
        {
            
        }

        /// <summary>
        /// Called when change feed observer is opened; 
        /// this function prints out observer partition key id. 
        /// </summary>
        /// <param name="context">The context specifying partition for this observer, etc.</param>
        /// <returns>A Task to allow asynchronous execution</returns>
        public Task OpenAsync(IChangeFeedObserverContext context)
        {
            Console.ForegroundColor = ConsoleColor.Magenta;
            Console.WriteLine("Observer opened for partition Key Range: {0}", context.PartitionKeyRangeId);
            return Task.CompletedTask;
        }

        /// <summary>
        /// Called when change feed observer is closed; 
        /// this function prints out observer partition key id and reason for shut down. 
        /// </summary>
        /// <param name="context">The context specifying partition for this observer, etc.</param>
        /// <param name="reason">Specifies the reason the observer is closed.</param>
        /// <returns>A Task to allow asynchronous execution</returns>
        public Task CloseAsync(IChangeFeedObserverContext context, ChangeFeedObserverCloseReason reason)
        {
            Console.ForegroundColor = ConsoleColor.Cyan;
            Console.WriteLine("Observer closed, {0}", context.PartitionKeyRangeId);
            Console.WriteLine("Reason for shutdown, {0}", reason);
            return Task.CompletedTask;
        }

        public Task ProcessChangesAsync(IChangeFeedObserverContext context, IReadOnlyList<Document> docs, CancellationToken cancellationToken)
        {
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("Change feed: PartitionId {0} total {1} doc(s)", context.PartitionKeyRangeId, Interlocked.Add(ref totalDocs, docs.Count));
            foreach (Document doc in docs)
            {
                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine(doc.Id.ToString());
            }

            return Task.CompletedTask;
        }
    }
    ```

2. 實作 **DocumentFeedObserverFactory**，它會實作 **IChangeFeedObserverFactory**。
    ```csharp
     using Microsoft.Azure.Documents.ChangeFeedProcessor.FeedProcessing;

    /// <summary>
    /// Factory class to create instance of document feed observer. 
    /// </summary>
    public class DocumentFeedObserverFactory : IChangeFeedObserverFactory
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="DocumentFeedObserverFactory" /> class.
        /// Saves input DocumentClient and DocumentCollectionInfo parameters to class fields
        /// </summary>
        public DocumentFeedObserverFactory()
        {
        }

        /// <summary>
        /// Creates document observer instance with client and destination collection information
        /// </summary>
        /// <returns>DocumentFeedObserver with client and destination collection information</returns>
        public IChangeFeedObserver CreateObserver()
        {
            DocumentFeedObserver newObserver = new DocumentFeedObserver();
            return newObserver as IChangeFeedObserver;
        }
    }
    ```

3. 定義 *CancellationTokenSource* 和 *ChangeFeedProcessorBuilder*

    ```csharp
    private readonly CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();
    private readonly ChangeFeedProcessorBuilder builder = new ChangeFeedProcessorBuilder();
    ```

5. 在定義相關物件之後建置 **ChangeFeedProcessorBuilder** 

    ```csharp
            string hostName = Guid.NewGuid().ToString();
      
            // monitored collection info 
            DocumentCollectionInfo documentCollectionInfo = new DocumentCollectionInfo
            {
                Uri = new Uri(this.monitoredUri),
                MasterKey = this.monitoredSecretKey,
                DatabaseName = this.monitoredDbName,
                CollectionName = this.monitoredCollectionName
            };
            
            DocumentCollectionInfo leaseCollectionInfo = new DocumentCollectionInfo
                {
                    Uri = new Uri(this.leaseUri),
                    MasterKey = this.leaseSecretKey,
                    DatabaseName = this.leaseDbName,
                    CollectionName = this.leaseCollectionName
                };
            DocumentFeedObserverFactory docObserverFactory = new DocumentFeedObserverFactory();
            ChangeFeedOptions feedOptions = new ChangeFeedOptions();

            /* ie customize StartFromBeginning so change feed reads from beginning
                can customize MaxItemCount, PartitonKeyRangeId, RequestContinuation, SessionToken and StartFromBeginning
            */

            feedOptions.StartFromBeginning = true;
        
            ChangeFeedProcessorOptions feedProcessorOptions = new ChangeFeedProcessorOptions();

            // ie. customizing lease renewal interval to 15 seconds
            // can customize LeaseRenewInterval, LeaseAcquireInterval, LeaseExpirationInterval, FeedPollDelay 
            feedProcessorOptions.LeaseRenewInterval = TimeSpan.FromSeconds(15);

            this.builder
                .WithHostName(hostName)
                .WithFeedCollection(documentCollectionInfo)
                .WithLeaseCollection(leaseCollectionInfo)
                .WithProcessorOptions (feedProcessorOptions)
                .WithObserverFactory(new DocumentFeedObserverFactory());               
                //.WithObserver<DocumentFeedObserver>();  If no factory then just pass an observer

            var result =  await this.builder.BuildAsync();
            await result.StartAsync();
            Console.Read();
            await result.StopAsync();    
    ```

就這麼簡單！ 執行這幾個步驟之後，文件會開始顯示於 **DocumentFeedObserver ProcessChangesAsync** 方法中。

上述程式碼為說明之用，以顯示不同類型的物件及其互動。 您必須定義適當的變數，並且以正確的值啟動這些變數。 您可以從 [GitHub 儲存機制](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples/ChangeFeedProcessorV2)取得本文中使用的完整程式碼。

> [!NOTE]
> 如以上的程式碼所示，在您的程式碼或組態檔中，永遠都不應該有主要金鑰。 請參閱[如何使用 Key Vault 擷取金鑰](https://sarosh.wordpress.com/2017/11/23/cosmos-db-and-key-vault/) (英文)。


## <a name="faq"></a>常見問題集

### <a name="what-are-the-different-ways-you-can-read-change-feed-and-when-to-use-each-method"></a>有哪些不同的方法可以用來讀取變更摘要？各種方法的使用時機在什麼時候？

若要讀取變更摘要，有三個選項可供選擇：

* **[使用 Azure Cosmos DB SQL API .NET SDK](#sql-sdk)**
   
   使用此方法，可以對變更摘要進行較低層級的控制。 您可以管理檢查點，也可以存取特定磁碟分割區索引鍵等。如果您有多個讀取器，可以使用 [ChangeFeedOptions](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.changefeedoptions?view=azure-dotnet) 將讀取負載分配至不同的執行緒或不同的用戶端。 .

* **[使用 Azure Cosmos DB 變更摘要處理器程式庫](#change-feed-processor)**

   如果您想外包更加複雜的變更摘要，則可使用變更摘要處理器程式庫。 此程式庫會將大量的複雜作業隱藏起來，但仍然可讓您完全控制變更摘要。 此程式庫依循的是[觀察者模式](https://en.wikipedia.org/wiki/Observer_pattern)，SDK 會呼叫您的處理函式。 

   如果您有高輸送量的變更摘要，則可將多個用戶端具現化，以讀取變更摘要。 因為您正使用「變更摘要處理器程式庫」，它會自動將負載分散到不同用戶端。 您不需要執行任何動作。 SDK 會處理所有的複雜作業。 不過，如果您想要擁有自己的負載平衡器，您可以實作 IParitionLoadBalancingStrategy，自訂磁碟分割的策略。 實作 IPartitionProcessor - 用於分割區上的自訂處理變更。 不過，您可以使用 SDK 處理磁碟分割範圍，但是，如果您想要處理特定磁碟分割區索引鍵，則必須使用適用於 SQL API 的 SDK。

* **[使用 Azure Functions](#azure-functions)** 
   
   最後一個選項 Azure Functions 是最簡單的， 建議您使用此選項。 當您在 Azure Functions 應用程式中建立 Azure Cosmos DB 觸發程序時，需選取要連線的 Azure Cosmos DB 集合，以及集合每次變更時要觸發的函式。 觀看使用 Azure 函式與變更摘要的[螢幕錄製影片](https://www.youtube.com/watch?v=Mnq0O91i-0s&t=14s)

   您可以在 Azure Functions 入口網站中、在 Azure Cosmos DB 入口網站中建立觸發程序，或以程式設計方式建立。 Visual Studio 與 VS Code 對於撰寫 Azure Function 提供了良好的支援。 您可以在電腦上撰寫和偵錯程式碼，然後只需按一下即可部署該功能。 如需詳細資訊，請參閱 [Azure Cosmos DB：使用 Azure Functions 的無伺服器資料庫計算](serverless-computing-database.md)一文。

### <a name="what-is-the-sort-order-of-documents-in-change-feed"></a>變更摘要中的文件排序順序為何？

變更摘要文件是按修改時間順序排列。 僅在每個分割區內可保障會按此順序排列。

### <a name="for-a-multi-region-account-what-happens-to-the-change-feed-when-the-write-region-fails-over-does-the-change-feed-also-failover-would-the-change-feed-still-appear-contiguous-or-would-the-fail-over-cause-change-feed-to-reset"></a>對於多區域帳戶，當寫入區域容錯移轉時，變更摘要會發生什麼事？ 變更摘要也會跟著容錯移轉嗎？ 變更摘要是否仍會顯示為連續的，還是容錯移轉會導致變更摘要重設？

變更摘要會在手動容錯移轉作業時持續運作，而且保持連續。

### <a name="how-long-change-feed-persist-the-changed-data-if-i-set-the-ttl-time-to-live-property-for-the-document-to--1"></a>如果我將文件的 TTL (存留時間) 設為 -1，變更摘要會存留變更資料多久時間？

變更摘要會永久存留。 如果未刪除資料，即會一直保留在變更摘要中。

### <a name="how-can-i-configure-azure-functions-to-read-from-a-particular-region-as-change-feed-is-available-in-all-the-read-regions-by-default"></a>按照預設，變更摘要在所有讀取區域中皆可使用，該如何設定 Azure 函式從特定區域讀取？

目前無法將 Azure Functions 設定為從特定區域讀取。 設定任何 Azure Cosmos DB 繫結與觸發程序的慣用區域時，Azure Functions 存放庫中有個 GitHub 問題。

Azure Functions 會使用預設的連線原則。 您可以在 Azure Functions 中設定連線模式，依預設會從寫入區域讀取，因此，最好將 Azure Functions 共置在相同區域中。

### <a name="what-is-the-default-size-of-batches-in-azure-functions"></a>Azure Functions 中預設的批次大小是多少？

每次引動 Azure Functions 時可有 100 個文件。 不過，此數量可在 function.json 檔案中設定。 以下是完整的[設定選項清單](../azure-functions/functions-run-local.md)。 如果是在本機開發，請更新 [local.settings.json](../azure-functions/functions-run-local.md) 檔案中的應用程式設定。

### <a name="i-am-monitoring-a-collection-and-reading-its-change-feed-however-i-see-i-am-not-getting-all-the-inserted-document-some-documents-are-missing-what-is-going-on-here"></a>我正在監視一個集合，並讀取其變更摘要，但是我發現無法取得所有插入的文件，缺少某些文件。 發生了什麼狀況？

請確定沒有其他函式使用相同的租用集合在讀取相同的集合。 我也碰過這種狀抗，後來我發現缺少的文件是由其他 Azure 函式處理的，這些函式也在使用相同的租用集合。

因此，如果您要建立多個 Azure Functions 以讀取相同的變更摘要，這些函式必須使用不同的租用集合，或使用「leasePrefix」設定以共用相同的集合。 不過，在使用變更摘要處理器程式庫時，您可以啟動函式的多個執行個體，SDK 會自動把文件分配到不同的執行個體上。

### <a name="my-document-is-updated-every-second-and-i-am-not-getting-all-the-changes-in-azure-functions-listening-to-change-feed"></a>我的文件每秒更新一次，而我沒有收到接聽變更摘要之 Azure Functions 中的所有變更。

Azure Functions 每 5 秒輪詢變更摘要一次，因此，5 秒內所做的任何變更都將遺失。 Azure Cosmos DB 每 5 秒僅儲存一次單一版本，因此您會得到該文件第五次變更的版本。 不過，如果您想要時間少於於 5 秒，而且希望每秒輪詢變更摘要一次，則可設定輪詢時間「feedPollTime」，請參閱 [Azure Cosmos DB 繫結](../azure-functions/functions-bindings-cosmosdb.md#trigger---configuration)。 它以毫秒為單位定義，預設值為 5000。 低於 1 秒是可行的 (但不建議)，因為您會耗用更多 CPU。

### <a name="i-inserted-a-document-in-the-mongo-api-collection-but-when-i-get-the-document-in-change-feed-it-shows-a-different-id-value-what-is-wrong-here"></a>我在 Mongo API 集合中已插入文件，但是當我在變更摘要中取得文件時，其顯示的是不同的 ID 值。 這其中發生什麼狀況？

您的集合是 Mongo API 集合。 請記住，變更摘要是利用 SQL 用戶端來讀取，並可將項目序列化為 JSON 格式。 因為是 JSON 格式，MongoDB 用戶端會遇到 BSON 格式文件與 JSON 格式變更摘要互不相符的情況。 您會看見以 JSON 格式表示的 BSON 文件。 如果您使用 Mongo 帳戶中的二進位屬性，它們會轉換為 JSON。

### <a name="is-there-a-way-to-control-change-feed-for-updates-only-and-not-inserts"></a>是否可僅針對更新項目 (而非針對插入項目) 控制變更摘要？

現在無法，但已在規劃此功能。 目前，您可以為更新項目在文件上新增軟標記。

### <a name="is-there-a-way-to-get-deletes-in-change-feed"></a>是否可取得變更摘要中的刪除項目？

目前變更摘要不會記錄刪除項目。 變更摘要會不斷改善，已在規劃此功能。 目前，您可以為刪除項目在文件上新增軟標記。 在名為「已刪除」的文件上新增屬性，將其設定為「true」，並在文件上設定 TTL，以便可以自動刪除。

### <a name="can-i-read-change-feed-for-historic-documentsfor-example-documents-that-were-added-5-years-back-"></a>我是否可以讀取歷史文件 (例如，5 年前新增的文件) 的變更摘要？

可以，如果文件未刪除，您可以讀取至原始收集的變更摘要。

### <a name="can-i-read-change-feed-using-javascript"></a>我是否可以使用 JavaScript 讀取變更摘要？

可以，最近已新增對變更摘要的 Node.js SDK 初始支援。 它可以依照下列範例所示使用，請先將 documentdb 模組更新為目前版本，然後再執行程式碼：

```js

var DocumentDBClient = require('documentdb').DocumentClient;
const host = "https://your_host:443/";
const masterKey = "your_master_key==";
const databaseId = "db";
const collectionId = "c1";
const dbLink = 'dbs/' + databaseId;
const collLink = dbLink + '/colls/' + collectionId;
var client = new DocumentDBClient(host, { masterKey: masterKey });
let options = {
    a_im: "Incremental feed",
    accessCondition: {
        type: "IfNoneMatch",        // Use: - empty condition (or remove accessCondition entirely) to start from beginning.
        //      - '*' to start from current.
        //      - specific etag value to start from specific continuation.
        condition: ""
    }
};
 
var query = client.readDocuments(collLink, options);
query.executeNext((err, results, headers) =&gt; {
    // Now we have headers.etag, which can be used in next readDocuments in accessCondition option.
    console.log(results);
    console.log(headers.etag);
    console.log(results.length);
    options.accessCondition = { type: "IfNoneMatch", condition: headers.etag };
    var query = client.readDocuments(collLink, options);
    query.executeNext((err, results, headers) =&gt; {
        console.log("next one:", results[0]);
    });
});<span id="mce_SELREST_start" style="overflow:hidden;line-height:0;"></span>

```

### <a name="can-i-read-change-feed-using-java"></a>我是否可以使用 Java 讀取變更摘要？

讀取變更摘要的 Java 程式庫可在 [Github 存放庫](https://github.com/Azure/azure-documentdb-changefeedprocessor-java)中找到。 但是，目前 Java 程式庫是 .NET 程式庫之後的幾個版本。 不久將會同步這兩個程式庫。

### <a name="can-i-use-etag-lsn-or-ts-for-internal-bookkeeping-which-i-get-in-response"></a>我是否可以使用 _etag、_lsn 或 _ts 進行回應中取得的內部簿記？

_etag 格式是作為內部之用，因為該格式可以隨時變更，請別以它為依歸 (請勿解析)。
_ts 是用於修改或建立時間戳記。 您可以使用 _ts，依時間順序進行比較。
_lsn 是僅為變更摘要新增的批次 ID，其代表存放區中的交易 ID。 許多文件可能有相同的 _lsn。
還有一點需要注意，FeedResponse 上的 ETag 與您在文件中看到的 _etag 不同。 _etag 是內部識別碼，而且用於並行，其指示文件版本，以及已使用 ETag 來排序摘要。

### <a name="does-reading-change-feed-add-any-additional-cost-"></a>讀取變更摘要是否會增加任何額外成本？

您需要支付耗用 RU 的費用，也就是資料移入 Azure Cosmos DB 集合一律會耗用 RU。 將會向使用者收取租用集合的 RU 費用。

### <a name="can-multiple-azure-functions-read-one-collections-change-feed"></a>多個 Azure Functions 是否可讀取一個集合變更摘要？

是。 多個 Azure Functions 可讀取相同的集合變更摘要。 但是，Azure Functions 需要定義一個單獨的 leaseCollectionPrefix。

### <a name="should-the-lease-collection-be-partitioned"></a>是否應該分割租用集合？

不，租用集合可以是固定的。 不需要分割的租用集合，而且目前並未支援。

### <a name="can-i-read-change-feed-from-spark"></a>我是否可以從 Spark 讀取變更摘要？

是，您可以這麼做。 請參閱 [Azure Cosmos DB Spark 連接器](spark-connector.md)。 以下是[螢幕錄製影片](https://www.youtube.com/watch?v=P9Qz4pwKm_0&t=1519s)，示範了如何以結構化串流處理變更摘要。

### <a name="if-i-am-processing-change-feed-by-using-azure-functions-say-a-batch-of-10-documents-and-i-get-an-error-at-7th-document-in-that-case-the-last-three-documents-are-not-processed-how-can-i-start-processing-from-the-failed-documentie-7th-document-in-my-next-feed"></a>如果我使用 Azure Functions 處理變更摘要，假設一個批次有 10 個文件，在第 7 個文件中出現錯誤。 在此情況下，不會處理最後三個文件，我要如何在後續的摘要中，從失敗的文件 (例如， 第 7 個文件) 啟動處理？

若要處理錯誤，建議的模式是使用 try-catch 區塊包裝您的程式碼。 攔截錯誤並將該文件放在佇列中 (無效信件)，然後定義邏輯以處理產生錯誤的文件。 如果您有一個包含 200 個文件的批次，而且只有一個文件失敗，請使用此方法，不需要丟棄整個批次。

如果出現錯誤，請勿將檢查點倒轉回開始位置，您才可以繼續從變更摘要中取得這些文件。 請記住，變更摘要會保留文件的最後一個快照集，因此，您可能會遺失文件中的上一個快照集。 變更摘要僅保留最後一個版本的文件，而且可在其他處理程序之間顯示，及變更文件。

隨著不斷修正程式碼，不久之後，您在無效信件佇列中無法找到任何文件。
變更摘要系統會自動呼叫 Azure Functions，而且 Azure Functions 會內部維護檢查點等。 如果您想復原檢查點並控制其每個層面，則應考慮使用變更摘要處理器 SDK。


## <a name="next-steps"></a>後續步驟

如需有關使用 Azure Cosmos DB 與 Azure Functions 的詳細資訊，請參閱 [Azure Cosmos DB：使用 Azure Functions 的無伺服器資料庫計算](serverless-computing-database.md)。

如需有關使用變更摘要處理器程式庫的詳細資訊，請利用下列資源：

* [資訊頁面](sql-api-sdk-dotnet-changefeed.md) 
* [Nuget 套件](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.ChangeFeedProcessor/)
* [展示上述步驟 1-6 的範例程式碼](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples/ChangeFeedProcessor)
* [GitHub 上的其他範例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/ChangeFeedProcessor)

如需有關透過 SDK 使用變更摘要的詳細資訊，請看下列資源：

* [SDK 資訊頁面](sql-api-sdk-dotnet.md)
