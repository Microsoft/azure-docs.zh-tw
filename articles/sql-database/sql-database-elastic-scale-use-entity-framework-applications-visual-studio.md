---
title: 搭配使用彈性資料庫用戶端程式庫與 Entity Framework | Microsoft Docs
description: 使用彈性資料庫用戶端程式庫與和 Entity Framework 來編寫資料庫
services: sql-database
manager: craigg
author: stevestein
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: 2eafd4b23da8f21f1a4b3ffcf29e50b65882d6c0
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646757"
---
# <a name="elastic-database-client-library-with-entity-framework"></a>搭配使用彈性資料庫用戶端程式庫與 Entity Framework
這份文件說明 Entity Framework 應用程式為了要與 [彈性資料庫工具](sql-database-elastic-scale-introduction.md)整合所需做的變更。 重點將著重於使用 Entity Framework **Code First** 方法來編寫[分區對應管理](sql-database-elastic-scale-shard-map-management.md)和[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)。 這整份文件是以 EF 的 [Code First - 新的資料庫](http://msdn.microsoft.com/data/jj193542.aspx)教學課程作為執行範例。 本文所附的範例程式碼取自於 Visual Studio 程式碼範例中的彈性資料庫工具範例集。

## <a name="downloading-and-running-the-sample-code"></a>下載並執行範例程式碼
若要下載本文的程式碼：

* 需要 visual Studio 2012 或更新版本。 
* 從 MSDN 下載 [Elastic DB Tools for Azure SQL - Entity Framework 整合範例](https://code.msdn.microsoft.com/windowsapps/Elastic-Scale-with-Azure-bae904ba)。 將範例解壓縮至您選擇的位置。
* 啟動 Visual Studio。 
* 在 Visual Studio 中，選取 [檔案] -> [開啟專案/方案]。 
* 在 [開啟專案] 對話方塊中，瀏覽至您下載的範例，然後選取 **EntityFrameworkCodeFirst.sln** 以開啟範例。 

若要執行範例，您在 Azure SQL Database 中需要建立三個空的資料庫：

* 分區對應管理員資料庫
* 分區 1 資料庫
* 分區 2 資料庫

一旦建立這些資料庫後，在 **Program.cs** 的預留位置中，填入您的 Azure SQL DB 伺服器名稱、資料庫名稱及用來連線到資料庫的認證。 在 Visual Studio 中建置方案。 在建置過程中，Visual Studio 會下載彈性資料庫用戶端程式庫、Entity Framework 和暫時性錯誤處理所需的 NuGet 套件。 請確定您的解決方案已啟用還原 NuGet 封裝。 您可以用滑鼠右鍵按一下 Visual Studio [方案總管] 中的方案檔來啟用這個設定。 

## <a name="entity-framework-workflows"></a>Entity Framework 工作流程
Entity Framework 開發人員依賴下列四種工作流程來建置應用程式，以及確保應用程式物件的持續性： 

* **Code First (新的資料庫)**：EF 開發人員在應用程式碼中建立模型，然後 EF 從它產生資料庫。 
* **Code First (現有的資料庫)**：開發人員可讓 EF 從現有資料庫產生模型的應用程式碼。
* **Model First**：開發人員在 EF 設計工具中建立模型，然後 EF 從模型建立資料庫。
* **Database First**：開發人員使用 EF 工具從現有的資料庫推斷模型。 

這些方法都依賴 DbContext 類別來自動管理應用程式的資料庫連接和資料庫結構描述。 DbContext 基底類別的不同建構函式允許在不同程度上控制連線建立、資料庫啟動載入和結構描述建立。 問題主要起因於 EF 所提供的資料庫連線管理，與彈性資料庫用戶端程式庫所提供的資料相依路由介面的連線管理功能，兩者交錯。 

## <a name="elastic-database-tools-assumptions"></a>彈性資料庫工具假設
關於詞彙定義，請參閱 [彈性資料庫工具字彙](sql-database-elastic-scale-glossary.md)。

在彈性資料庫用戶端程式庫中，您會定義應用程式資料的資料分割，稱為 Shardlet。 Shardlet 以分區化索引鍵來識別，而且會對應到特定的資料庫。 根據目前的商務需求，應用程式可依需要而有無數個資料庫，並散發 Shardlet 來提供足夠的容量或效能。 分區化索引鍵值到資料庫的對應，由彈性資料庫用戶端 API 所提供的分區對應來儲存。 這項功能稱為**分區對應管理**，簡稱為 SMM。 對於攜帶分區化索引鍵的要求，分區對應也充當資料庫連接的代理人。 這項功能稱為**資料相依路由**。 

分區對應管理員可防止使用者檢視 Shardlet 資料時出現不一致，這種情況發生在並行 Shardlet 管理作業中 (例如將資料從一個分區重新放置到另一個分區)。 在作法上，用戶端程式庫所管理的分區對應會代理應用程式的資料庫連接。 這可讓分區對應功能在分區管理作業可能影響已建立連接的 Shardlet 時，自動終止資料庫連接。 這種方法需要與一些 EF 功能整合，例如從現有連接建立新的連接以檢查資料庫是否存在。 一般而言，我們觀察是只有在已關閉的資料庫連接上 (EF 工作可安心複製)，標準 DbContext 建構函式才能可靠地運作。 彈性資料庫的設計原則只是代理已開啟的連接。 有人可能會認為先關閉用戶端程式庫所代理的連接，再交給 EF DbContext，就可以解決這個問題。 不過，若關閉連線並依賴 EF 來重新開啟它，就等於放棄程式庫所執行的驗證和一致性檢查。 不過，EF 的移轉功能會使用這些連接，在應用程式不知情的情況下管理基礎資料庫結構描述。 在理想的情況下，您要在相同的應用程式中保留並結合彈性資料庫用戶端程式庫和 EF 提供的所有這些功能。 下一節詳細討論這些屬性和需求。 

## <a name="requirements"></a>需求
當使用彈性資料庫用戶端程式庫和 Entity Framework API 時，您需要保留下列屬性： 

* **相應放大**：我們想要依需要在分區化應用程式資料層中新增或移除資料庫，以滿足應用程式的容量需求。 這表示控制資料庫的建立和刪除，以及使用彈性資料庫分區對應管理員 API 來管理資料庫，還有 Shardlet 的對應。 
* **一致性**：應用程式採用分區化，並且使用用戶端程式庫的資料相依路由功能。 為了避免查詢結果損毀或錯誤，由分區對應管理員來代理連接。 這也會保留驗證和一致性。
* **Code First**：保留 EF code first 典範的便利性。 在 Code First 中，應用程式中的類別直接對應至基礎資料庫結構。 應用程式碼與 DbSets 互動，DbSets 將基礎資料庫處理所需的大部分工作隱藏起來。
* **結構描述**：Entity Framework 會處理建立初始資料庫結構描述，以及結構描述後續透過移轉的演進。 只要保留這些功能，就可輕鬆隨著資料演進來調整您的應用程式。 

下列指導方針指示如何使用彈性資料庫工具，以滿足 Code First 應用程式的這些需求。 

## <a name="data-dependent-routing-using-ef-dbcontext"></a>使用 EF DbContext 的資料相依路由
資料庫與 Entity Framework 的連接通常是透過 **DbContext**的子類別管理。 從 **DbContext**衍生來建立這些子類別。 您在此定義 **DbSets** ，為您的應用程式實作 CLR 物件的資料庫支援集合。 就資料相依路由來說，您可以識別其他 EF Code First 應用程式案例中不一定支援的幾個實用的屬性： 

* 資料庫已經存在，而且已在彈性資料庫分區對應中註冊。 
* 應用程式的結構描述已經部署到資料庫 (如下所述)。 
* 由分區對應代理資料庫的資料相依路由連接。 

整合 **DbContexts** 與資料相依路由以相應放大：

1. 透過分區對應管理員的彈性資料庫用戶端介面，建立實體資料庫連接， 
2. 使用 **DbContext** 子類別來包裝連接
3. 將連接向下傳入 **DbContext** 基底類別，以確保也在 EF 端進行所有處理。 

下列程式碼範例說明此方法。 (此程式碼也在隨附的 Visual Studio 專案中)

    public class ElasticScaleContext<T> : DbContext
    {
    public DbSet<Blog> Blogs { get; set; }
    …

        // C'tor for data-dependent routing. This call opens a validated connection 
        // routed to the proper shard by the shard map manager. 
        // Note that the base class c'tor call fails for an open connection
        // if migrations need to be done and SQL credentials are used. This is the reason for the 
        // separation of c'tors into the data-dependent routing case (this c'tor) and the internal c'tor for new shards.
        public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
            : base(CreateDDRConnection(shardMap, shardingKey, connectionStr), 
            true /* contextOwnsConnection */)
        {
        }

        // Only static methods are allowed in calls into base class c'tors.
        private static DbConnection CreateDDRConnection(
        ShardMap shardMap, 
        T shardingKey, 
        string connectionStr)
        {
            // No initialization
            Database.SetInitializer<ElasticScaleContext<T>>(null);

            // Ask shard map to broker a validated connection for the given key
            SqlConnection conn = shardMap.OpenConnectionForKey<T>
                                (shardingKey, connectionStr, ConnectionOptions.Validate);
            return conn;
        }    

## <a name="main-points"></a>重點
* 新的建構函式取代 DbContext 子類別的預設建構函式 
* 新的建構函式接受透過彈性資料庫用戶端程式庫的資料相依路由所需的引數：
  
  * 用來存取資料相依路由介面的分區對應，
  * 用來識別 Shardlet 的分區化索引鍵，
  * 分區的資料相依路由連接所需的連接字串和認證。 
* 基底類別建構函式的呼叫會繞道至靜態方法，以執行資料相依路由所需的所有步驟。 
  
  * 它會在分區對應上使用彈性資料庫用戶端介面的 OpenConnectionForKey 呼叫，以建立開啟的連接。
  * 分區對應會對分區建立開啟的連接，此分區保留給定分區化索引鍵的 Shardlet。
  * 這個開啟的連接會傳回給 DbContext 的基底類別建構函式，以指出此連接要給 EF 使用，而不要讓 EF 自動建立新的連接。 如此一來，連接已具有彈性資料庫用戶端 API 的性質，將可以保證分區對應管理作業時的一致性。

在您的程式碼中使用 DbContext 子類別的新建構函式，而不是預設建構函式。 下列是一個範例： 

    // Create and save a new blog.

    Console.Write("Enter a name for a new blog: "); 
    var name = Console.ReadLine(); 

    using (var db = new ElasticScaleContext<int>( 
                            sharding.ShardMap,  
                            tenantId1,  
                            connStrBldr.ConnectionString)) 
    { 
        var blog = new Blog { Name = name }; 
        db.Blogs.Add(blog); 
        db.SaveChanges(); 

        // Display all Blogs for tenant 1 
        var query = from b in db.Blogs 
                    orderby b.Name 
                    select b; 
     … 
    }

新的建構函式會對分區開啟連接，此分區保留 **tenantid1**值所識別的分區的資料。 **using** 區塊中的程式碼維持不變，可在 **tenantid1** 的分區上使用 EF 存取部落格的 **DbSet**。 這會變更 using 區塊中的程式碼語意，使得所有資料庫作業現在以一個保留 **tenantid1** 的分區為範圍。 例如，在部落格 **DbSet** 上的 LINQ 查詢只傳回目前分區上儲存的部落格，而不是儲存在其他分區的部落格。  

#### <a name="transient-faults-handling"></a>暫時性錯誤處理
Microsoft 模式和作法小組已發佈[暫時性錯誤處理應用程式區塊](https://msdn.microsoft.com/library/dn440719.aspx)。 此程式庫搭配 Elastic Scale 用戶端程式庫並結合 EF 一起使用。 不過，請確定任何暫時性例外狀況都傳回至某處，讓您可以確定暫時性失敗後會使用新的建構函式，以便使用您已調整的建構函式來進行任何新的連線嘗試。 否則，無法保證連接至正確的分區，也不能確保分區對應變更發生時會保持連接。 

下列程式碼範例說明如何在新的 **DbContext** 子類別建構函式周圍使用 SQL 重試原則： 

    SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() => 
    { 
        using (var db = new ElasticScaleContext<int>( 
                                sharding.ShardMap,  
                                tenantId1,  
                                connStrBldr.ConnectionString)) 
            { 
                    var blog = new Blog { Name = name }; 
                    db.Blogs.Add(blog); 
                    db.SaveChanges(); 
            … 
            } 
        }); 

上述程式碼中的 **SqlDatabaseUtils.SqlRetryPolicy** 定義為 **SqlDatabaseTransientErrorDetectionStrategy**，並指定重試計數 10，重試之間等待時間為 5 秒。 這種方式類似於 EF 和使用者起始交易的指引 (請參閱 [重試執行策略的限制 (從 EF6 開始)](http://msdn.microsoft.com/data/dn307226))。 這兩種情況都需要應用程式控制暫時性例外狀況傳回的範圍：重新開啟交易，或 (如下所示) 從使用彈性資料庫用戶端程式庫的適當建構函式重新建立內容。

控制暫時性例外狀況把我們帶回範圍中何處的需求，也會妨礙使用 EF 隨附的內建 **SqlAzureExecutionStrategy** 。 **SqlAzureExecutionStrategy** 會重新開啟連接，但不使用 **OpenConnectionForKey**，因此會略過 **OpenConnectionForKey** 呼叫過程中執行的所有驗證。 然而，此程式碼範例會使用同樣是 EF 隨附的內建 **DefaultExecutionStrategy** 。 相對於 **SqlAzureExecutionStrategy**，它會結合暫時性錯誤處理的重試原則一起正確運作。 執行原則設定於 **ElasticScaleDbConfiguration** 類別中。 請注意，我們決定不使用 **DefaultSqlExecutionStrategy**，因為它會在發生暫時性例外狀況時建議使用 **SqlAzureExecutionStrategy**，而這會導致所述的錯誤行為。 如需不同重試原則與 EF 的詳細資訊，請參閱 [EF 的連線恢復功能](http://msdn.microsoft.com/data/dn456835.aspx)。     

#### <a name="constructor-rewrites"></a>建構函式重寫
上述程式碼範例說明應用程式所需的預設建構函式重寫，以便使用資料相依路由與 Entity Framework。 下表將這個方法擴及其他建構函式。 

| 目前的建構函式 | 重寫的資料建構函式 | 基底建構函式 | 注意 |
| --- | --- | --- | --- |
| MyContext() |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |連接必須是分區對應和資料相依路由索引鍵的函數。 您需要略過由 EF 自動建立連接，改用分區對應來代理連接。 |
| MyContext(string) |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |連接是分區對應和資料相依路由索引鍵的函數。 固定的資料庫名稱或連接字串無法運作，因為它們會略過分區對應所執行的驗證。 |
| MyContext(DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |將使用提供的模型建立給定分區對應和分區化索引鍵的連線。 編譯的模型將會傳遞給基底 c'tor。 |
| MyContext(DbConnection, bool) |ElasticScaleContext(ShardMap, TKey, bool) |DbContext(DbConnection, bool) |連接必須是從分區對應和索引鍵推斷的函數。 無法提供它做為輸入 (除非該輸入已經使用分區對應和索引鍵)。 將傳遞布林值。 |
| MyContext(string, DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |連接必須是從分區對應和索引鍵推斷的函數。 無法提供它做為輸入 (除非該輸入使用分區對應和索引鍵)。 將傳遞編譯的模型。 |
| MyContext(ObjectContext, bool) |ElasticScaleContext(ShardMap, TKey, ObjectContext, bool) |DbContext(ObjectContext, bool) |新的建構函式必須確定在 ObjectContext 中做為輸入傳遞的任何連接，都會重新路由至 Elastic Scale 所管理的連接。 ObjectContexts 類別的詳細討論已超出本文的範圍。 |
| MyContext(DbConnection, DbCompiledModel, bool) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel, bool) |DbContext(DbConnection, DbCompiledModel, bool); |連接必須是從分區對應和索引鍵推斷的函數。 無法提供連接做為輸入 (除非該輸入已經使用分區對應和索引鍵)。 模型和布林值會傳遞至基底類別建構函式。 |

## <a name="shard-schema-deployment-through-ef-migrations"></a>透過 EF 移轉的分區結構描述部署
自動結構描述管理是 Entity Framework 所提供的便利性。 就使用彈性資料庫工具的應用程式而言，您需要保留這項功能，以便在資料庫新增至分區化應用程式時，自動將結構描述佈建至新建立的分區。 主要使用案例是針對使用 EF 的分區化應用程式，在資料層增加容量。 依賴 EF 的功能來管理結構描述，可減輕以 EF 建置的分區化應用程式所需的資料庫管理工作。 

透過 EF 移轉的結構描述部署在 **未開啟的連接**上表現最好。 這與資料相依路由的情況相反，它依賴彈性資料庫用戶端 API 所提供的已開啟連線。 另一個差異是一致性需求：雖然希望確保所有資料相依路由連接的一致性，以防止並行分區對應操作，但這無關於最初將結構描述部署到尚未在分區對應中註冊且尚未配置來保留 Shardlet 的新資料庫。 因此，您在此案例中可依賴一般資料庫連線，而不是資料相依路由。  

這會形成一種方法，使得透過 EF 移轉的結構描述部署，與新的資料庫在應用程式的分區對應中註冊為分區緊密相關。 這依賴下列必要條件： 

* 已建立資料庫。 
* 資料庫是空的 - 不含任何使用者結構描述與任何使用者資料。
* 還不能透過資料相依路由的彈性資料庫用戶端 API 存取資料庫。 

符合這些必要條件後，您就可以建立一般未開啟的 **SqlConnection** ，以開始進行結構描述部署的 EF 移轉。 下列程式碼範例說明此方法。 

        // Enter a new shard - i.e. an empty database - to the shard map, allocate a first tenant to it  
        // and kick off EF intialization of the database to deploy schema 

        public void RegisterNewShard(string server, string database, string connStr, int key) 
        { 

            Shard shard = this.ShardMap.CreateShard(new ShardLocation(server, database)); 

            SqlConnectionStringBuilder connStrBldr = new SqlConnectionStringBuilder(connStr); 
            connStrBldr.DataSource = server; 
            connStrBldr.InitialCatalog = database; 

            // Go into a DbContext to trigger migrations and schema deployment for the new shard. 
            // This requires an un-opened connection. 
            using (var db = new ElasticScaleContext<int>(connStrBldr.ConnectionString)) 
            { 
                // Run a query to engage EF migrations 
                (from b in db.Blogs 
                    select b).Count(); 
            } 

            // Register the mapping of the tenant to the shard in the shard map. 
            // After this step, data-dependent routing on the shard map can be used 

            this.ShardMap.CreatePointMapping(key, shard); 
        } 


這個範例示範 **RegisterNewShard** 方法，此方法會在分區對應中註冊分區、透過 EF 移轉部署結構描述，並將分區化索引鍵的對應儲存至分區。 它依賴 **DbContext** 子類別 (範例中的 **ElasticScaleContext**) 的建構函式 (接受 SQL 連接字串做為輸入)。 這個建構函式的程式碼簡單易懂，如下列範例所示： 

        // C'tor to deploy schema and migrations to a new shard 
        protected internal ElasticScaleContext(string connectionString) 
            : base(SetInitializerForConnection(connectionString)) 
        { 
        } 

        // Only static methods are allowed in calls into base class c'tors 
        private static string SetInitializerForConnection(string connnectionString) 
        { 
            // You want existence checks so that the schema can get deployed 
            Database.SetInitializer<ElasticScaleContext<T>>( 
        new CreateDatabaseIfNotExists<ElasticScaleContext<T>>()); 

            return connnectionString; 
        } 

您可能使用繼承自基底類別的建構函式版本。 但是，程式碼必須確定連接時使用 EF 的預設初始設定式。 因此，在以連接字串呼叫基底類別建構函式之前，先稍微繞道至靜態方法。 請注意，分區的註冊應該在不同的應用程式定義域或程序中執行，以確保 EF 的初始設定式設定不會發生衝突。 

## <a name="limitations"></a>限制
這份文件中所述的方法有幾個限制： 

* 使用 **LocalDb** 的 EF 應用程式必須先移轉至一般 SQL Server 資料庫，才能使用彈性資料庫用戶端程式庫。 使用 **LocalDb**時，無法透過分區化與 Elastic Scale 來相應放大應用程式。 請注意，仍可使用 **LocalDb**來開發。 
* 應用程式中涉及資料庫結構描述變更的任何變更，都必須在所有分區上經過 EF 移轉。 這份文件的範例程式碼不示範此作法。 請考慮使用 Update-Database 搭配 ConnectionString 參數，以逐一查看所有分區；或使用 Update-Database 搭配 -Script 選項，以擷取擱置中移轉的 T-SQL 指令碼，然後將 T-SQL 指令碼套用至您的分區。  
* 提出要求時，將會假定所有資料庫處理都包含在要求所提供的分區化索引鍵所識別的單一分區內。 不過，這項假設不一定永遠成立。 例如，有時無法提供分區化索引鍵。 為了解決這個問題，用戶端程式庫提供 **MultiShardQuery** 類別，此類別實作連接抽象來查詢數個分區。 學習搭配 EF 來使用 **MultiShardQuery** 已超出本文的範圍

## <a name="conclusion"></a>結論
透過本文件中所述的步驟，EF 應用程式可以使用彈性資料庫用戶端程式庫的資料相依路由功能，重構 EF 應用程式中使用的 **DbContext** 子類別的建構函式。 已存在 **DbContext** 類別的地方不需要做太多變更。 此外，EF 應用程式可以結合叫用必要 EF 移轉的步驟，以及將新的分區和對應註冊在分區對應中的步驟，以繼續受益於自動結構描述部署。 

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/sample.png
