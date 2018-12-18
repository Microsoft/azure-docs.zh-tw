---
title: Azure Cosmos DB 資源模型和概念 | Microsoft Docs
description: 深入了解 Azure Cosmos DB 資料庫的階層式模型、集合、使用者定義函數 (UDF)、文件、權限，以便管理資源等。
keywords: 階層式模型, Hierarchical model, cosmosdb, azure, Microsoft azure
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: rafats
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0869881ace689d12272affb38d3689965e107e8f
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44050927"
---
# <a name="azure-cosmos-db-hierarchical-resource-model-and-core-concepts"></a>Azure Cosmos DB 階層式資源模型和核心概念

Azure Cosmos DB 管理的資料庫實體稱為「資源」。 每個資源可透過邏輯 URI 唯一識別。 您可以使用標準 HTTP 動詞命令、要求/回應標頭和狀態碼來與資源互動。 

本文會回答下列問題：

* 什麼是 Azure Cosmos DB 的資源模型？
* 什麼是系統定義的資源？與使用者定義的資源有何差異？
* 如何處理資源？
* 如何使用集合？
* 如何使用預存程序、觸發程序和使用者定義函式 (UDF)？

## <a name="hierarchical-resource-model"></a>階層式資源模型
如下圖所示，Azure Cosmos DB 的階層式「資源模型」包含某個資料庫帳戶下的多組資源，而每組資源都可透過邏輯和穩定 URI 加以定址。 本文中有一組資源稱為**摘要**。 

> [!NOTE]
> Azure Cosmos DB 提供高效率的 TCP 通訊協定，此 TCP 通訊協定在通訊模型中也符合 REST 限制，並且可以透過 [SQL .NET 用戶端 API](sql-api-sdk-dotnet.md) 取得。
> 
> 

![Azure Cosmos DB 階層式資源模型][1]  
**階層式資源模型**   

若要開始使用資源，您必須使用 Azure 訂用帳戶[建立資料庫帳戶](create-sql-api-dotnet.md)。 資料庫帳戶包含一組**資料庫**，而每個資料庫都包含多個**集合**，且各集合因此包含**預存程序、觸發程序、UDF、文件和相關附件**。 資料庫也有相關聯的**使用者**，其中每位使用者都有一組可存取集合、預存程序、觸發程序、UDF、文件或附件的**權限**。 雖然資料庫、使用者、權限和集合都是具有已知結構描述的系統定義資源，但是文件和附件包含任意使用者定義 JSON 內容。  

| 資源 | 說明 |
| --- | --- |
| 資料庫帳戶 |資料庫帳戶會與一組資料庫及附件之固定數目的 Blob 儲存體相關聯。 您可以使用 Azure 訂用帳戶建立一或多個資料庫帳戶。 如需詳細資訊，請瀏覽[定價頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)。 |
| 資料庫 |資料庫是分割給多個集合之文件儲存體的邏輯容器。 同時也是使用者容器。 |
| 使用者 |範圍權限的邏輯命名空間。 |
| 權限 |與使用者相關聯的授權權杖，可讓使用者用於存取特定資源。 |
| 集合 |集合是 JSON 文件和相關聯 JavaScript 應用程式邏輯的容器。 集合可以跨越一或多個資料分割/伺服器，也可以進行調整以處理幾乎無限量的儲存體或輸送量。 |
| 預存程序 |以 JavaScript 撰寫的應用程式邏輯，會向集合註冊並透過交易方式在資料庫引擎內執行。 |
| 觸發程序 |以 JavaScript 撰寫的應用程式邏輯，會在插入、取代或刪除作業之前或之後執行。 |
| UDF |以 JavaScript 撰寫的應用程式邏輯。 UDF 可讓您建立自訂查詢運算子的模型，進而擴充核心 SQL API 查詢語言。 |
| 文件 |使用者定義的 (任意) JSON 內容。 依照預設，不是不需要定義任何結構描述，就是需要提供所有新增至集合之文件的次要索引。 |
| 附件 |附件是含有外部 Blob/媒體之參考資料和相關聯中繼資料的特殊文件。 開發人員可以選擇讓 Cosmos DB 管理 Blob，或使用外部 Blob 服務提供者 (例如 OneDrive、Dropbox 等) 儲存 Blob。 |

## <a name="system-vs-user-defined-resources"></a>系統與使用者定義的資源
資料庫帳戶、資料庫、集合、使用者、權限、預存程序、觸發程序和 UDF 等資源的結構描述都是固定不變的，因此稱為「系統資源」。 相反地，文件和附件等資源則是「使用者定義的資源」的範例，因為這些資源的結構描述沒有任何限制。 在 Cosmos DB 中，系統和使用者定義資源都會呈現和管理為標準相符 JSON。 所有資源 (不論是系統定義的還是使用者定義的) 都具有下列共同屬性：

> [!NOTE]
> 所有系統產生的資源屬性在以 JSON 形式表示時，前面都會加上底線 (_)。
> 
> 

<table>
    <tbody>
        <tr>
            <td valign="top"><p><strong>屬性</strong></p></td>
            <td valign="top"><p><strong>可由使用者設定或由系統產生？</strong></p></td>
            <td valign="top"><p><strong>用途</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>_rid</p></td>
            <td valign="top"><p>由系統產生</p></td>
            <td valign="top"><p>系統產生的唯一階層式資源識別碼</p></td>
        </tr>
        <tr>
            <td valign="top"><p>_etag</p></td>
            <td valign="top"><p>由系統產生</p></td>
            <td valign="top"><p>要控制開放式並行存取所需之資源的 etag</p></td>
        </tr>
        <tr>
            <td valign="top"><p>_ts</p></td>
            <td valign="top"><p>由系統產生</p></td>
            <td valign="top"><p>資源上次更新之時間的時間戳記</p></td>
        </tr>
        <tr>
            <td valign="top"><p>_self</p></td>
            <td valign="top"><p>由系統產生</p></td>
            <td valign="top"><p>資源的唯一可定址 URI</p></td>
        </tr>
        <tr>
            <td valign="top"><p>id</p></td>
            <td valign="top"><p>無論是</p></td>
            <td valign="top"><p>使用者定義的資源唯一名稱 (具有相同的分割索引鍵值)。 如果使用者未指定 id，id 會由系統產生</p></td>
        </tr>
    </tbody>
</table>

### <a name="wire-representation-of-resources"></a>以線路表示資源
Cosmos DB 不會要求您提供用於 JSON 標準或特殊編碼的專屬延伸模組；Cosmos DB 本身就能處理符合 JSON 標準的文件。  

### <a name="addressing-a-resource"></a>資源定址
所有資源都能以 URI 定址。 資源的 **_self** 屬性值代表資源的相對 URI。 URI 的格式是由 /\<feed\>/{_rid} 路徑片段所組成：  

| _self 的值 | 說明 |
| --- | --- |
| /dbs |資料庫帳戶下的資料庫摘要 |
| /dbs/{dbName} |具有符合值 {dbName} 的識別碼的資料庫 |
| /dbs/{dbName}/colls/ |在資料庫底下的集合摘要 |
| /dbs/{dbName}/colls/{collName} |具有符合值 {collName} 的識別碼的集合 |
| /dbs/{dbName}/colls/{collName}/docs |在集合底下的文件摘要 |
| /dbs/{dbName}/colls/{collName}/docs/{docId} |具有符合值 {doc} 的識別碼的文件 |
| /dbs/{dbName}/users/ |在資料庫底下的使用者摘要 |
| /dbs/{dbName}/users/{userId} |具有符合值 {user} 的識別碼的使用者 |
| /dbs/{dbName}/users/{userId}/permissions |在使用者底下的權限摘要 |
| /dbs/{dbName}/users/{userId}/permissions/{permissionId} |具有符合值 {permission} 的識別碼的權限 |

每項資源都具有一個使用者所定義的不重複名稱，並會透過 id 屬性公開。 注意：對於文件，如果使用者未指定識別碼，則 SDK 會自動產生文件的唯一識別碼。 id 是使用者定義的字串，最多 256 個字元，且在特定父系資源的內容中會是唯一的。 

每個資源還會有系統產生的階層式資源識別碼 (也稱為 RID)，其可透過 _rid 屬性取得。 RID 會對指定資源的整個階層進行編碼，此內部表示法十分方便，可透過分散方式強制執行參考完整性。 RID 在資料庫帳戶內不會重複。Cosmos DB 會在內部使用 RID，完全無須跨資料分割進行查閱，就能有效率地進行路由。 _self 和 _rid 屬性的值都是資源的替代及標準表示法。 

REST API 可透過識別碼與 _rid 屬性，支援資源的定址和要求的路由。

## <a name="database-accounts"></a>資料庫帳戶
您可以使用 Azure 訂用帳戶佈建一或多個 Cosmos DB 資料庫帳戶。

您可以透過 Azure 入口網站 (網址：[http://portal.azure.com/](https://portal.azure.com/)) 建立和管理 Cosmos DB 資料庫帳戶。 建立和管理資料庫帳戶都需要管理存取權，而且只有在 Azure 訂用帳戶下才能執行。 

### <a name="database-account-properties"></a>資料庫帳戶屬性
在佈建和管理資料庫帳戶時，您可以設定和讀取下列屬性：  

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td valign="top"><p><strong>屬性名稱</strong></p></td>
            <td valign="top"><p><strong>說明</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>一致性原則</p></td>
            <td valign="top"><p>設定此屬性，以設定您資料庫帳戶下所有集合的預設一致性層級。 您可以使用 [x-ms-consistency-level] 要求標頭，覆寫每個要求的一致性層級。 <p><p>此屬性僅適用於「使用者定義的資源」<i></i>。 所有系統定義資源都是設定成支援具有強式一致性的讀取/查詢。</p></td>
        </tr>
        <tr>
            <td valign="top"><p>授權金鑰</p></td>
            <td valign="top"><p>這些是主要和次要且唯讀金鑰，提供資料庫帳戶下所有資源的管理存取權。</p></td>
        </tr>
    </tbody>
</table>

除了從「Azure 入口網站」佈建、設定及管理資料庫帳戶之外，您也可以透過使用 [Azure Cosmos DB REST API (英文)](/rest/api/cosmos-db/) 以及[用戶端 SDK (英文)](sql-api-sdk-dotnet.md)，以程式設計方式建立和管理 Cosmos DB 資料庫帳戶。  

## <a name="databases"></a>資料庫
Cosmos DB 資料庫是一個或多個集合和使用者的邏輯容器，如下圖所示。 您可以在 Cosmos DB 資料庫帳戶下，根據供應項目限制建立任意數目的資料庫。  

![資料庫帳戶和集合階層式模型][2]  
**資料庫是使用者和集合的邏輯容器**

資料庫可包含集合內分割的文件儲存體，且數量不限。

### <a name="elastic-scale-of-an-azure-cosmos-db-database"></a>Azure Cosmos DB 資料庫靈活的擴充能力
Cosmos DB 資料庫預設相當靈活，範圍從幾 GB 到數 PB 的 SSD 型式文件儲存體和佈建輸送量。 

不同於傳統 RDBMS 中的資料庫，Cosmos DB 中的資料庫不會侷限在單一電腦中。 有了 Cosmos DB 之後，您就能隨著應用程式的規模而相應放大，並能建立更多的集合及 (或) 資料庫。 的確，Microsoft 中的各種第一方應用程式都是依消費者規模來使用 Azure Cosmos DB，建立極大的 Azure Cosmos DB 資料庫，每個資料庫各包含數千個具有好幾 TB 文件儲存體的集合。 您可以新增或移除集合來擴充或縮減資料庫，以配合您的應用程式規模需求。 

您可以依據供應項目，在資料庫中建立任意數目的集合。 視選取的供應項目而定，每個集合 (或資料庫中的一組集合) 都有為您佈建的 SSD 型儲存體和輸送量。

Azure Cosmos DB 資料庫同時也是使用者的容器。 使用者因此是一組權限的邏輯命名空間，可針對集合、文件和附件提供微調的授權和存取權。  

與 Azure Cosmos DB 資源模型中的其他資源相同，不論是使用 [REST API](/rest/api/cosmos-db/) 還是任何[用戶端 SDK](sql-api-sdk-dotnet.md)，都可輕鬆地建立、取代、刪除、讀取或列舉資料庫。 Azure Cosmos DB 保證讀取或查詢資料庫資源之中繼資料的強式一致性。 刪除資料庫會自動確定您無法存取其內所含的任何集合或使用者。   

## <a name="collections"></a>集合
Cosmos DB 集合是 JSON 文件的容器。 

### <a name="elastic-ssd-backed-document-storage"></a>彈性 SSD 支持文件儲存體
集合本質上是彈性的 - 它會隨著您新增或移除文件自動成長和縮減。 集合是邏輯資源，可以跨一或多個實體分割或伺服器。 指派給集合的分割區數目，由 Cosmos DB 根據為集合或一組集合佈建的儲存體大小和輸送量所決定。 Cosmos DB 中的每個分割都有其相關聯的固定 SSD 支援儲存體數量，並且複寫以提供高可用性。 分割管理完全由 Azure Cosmos DB 管理，您不需要撰寫複雜程式碼或管理分割。 Cosmos DB 集合在儲存體和輸送量方面**並無限制**。 

### <a name="automatic-indexing-of-collections"></a>集合的自動編製索引
Azure Cosmos DB 是真正不具結構描述的資料庫系統。 它不會假設或不需要 JSON 文件的任何結構描述。 將文件新增至集合時，Azure Cosmos DB 會自動編製它們的索引，並且可供您進行查詢。 在不需要結構描述或次要索引的情況下自動編製文件索引是 Azure Cosmos DB 的重要功能，並且會啟用以獲得寫入最佳化、無鎖定和記錄結構化索引維護技術。 Azure Cosmos DB 支援極快速持續寫入量，同時仍然提供一致的查詢。 文件和索引儲存體都是用來計算每個集合所使用的儲存體。 您可以設定集合的索引原則，以控制與索引相關聯的儲存體和效能取捨。 

### <a name="configuring-the-indexing-policy-of-a-collection"></a>設定集合的索引原則
每個集合的索引原則都可讓您進行與索引相關聯的效能和儲存體取捨。 下列是可供您在進行索引組態時使用的選項：  

* 選擇集合是否自動編製所有文件的索引。 預設會自動編製所有文件的索引。 您可以選擇關閉自動編製索引，並選擇性地僅將特定文件新增至索引。 相反地，您可以選擇性地選擇僅排除特定文件。 做法是在集合的 indexingPolicy 上將自動屬性設定為 true 或 false，以及在插入、取代或刪除文件時使用 [x-ms-indexingdirective] 要求標頭。  
* 選擇在文件中包括還是排除索引中的特定路徑或模式。 做法是分別在集合的 indexingPolicy 上設定 includedPaths 和 excludedPaths。 您也可以針對特定路徑模式的範圍和雜湊查詢，設定儲存體和效能取捨。 
* 選擇同步 (一致) 與非同步 (緩慢) 索引更新。 每次在集合中插入、取代或刪除文件時，預設會同步更新索引。 這個行為讓查詢能夠使用與文件的讀取相同的一致性層級。 雖然 Azure Cosmos DB 的寫入已經過最佳化處理，並支援持續的文件寫入數量以及同步索引維護和提供一致的查詢，但是您還是可以設定特定集合，讓集合的索引更新速度變慢。 緩慢索引會進一步地促進寫入效能，而且適合在主要進行大量讀取集合時大量擷取案例。

在集合上執行 PUT 即可變更索引原則。 不論是透過[用戶端 SDK](sql-api-sdk-dotnet.md)、[Azure 入口網站](https://portal.azure.com)還是 [REST API](/rest/api/cosmos-db/)，都可達到此目的。

### <a name="querying-a-collection"></a>查詢集合
集合內的文件可以有任意結構描述，而且您可以查詢集合內的文件，而不需要預先提供任何結構描述或次要索引。 您可以使用 [Azure Cosmos DB SQL 語法參考](https://msdn.microsoft.com/library/azure/dn782250.aspx)來查詢集合，這些語法透過 JavaScript 型 UDF 提供豐富階層式與關聯式的空間運算子及擴充性。 JSON 文法允許用於將 JSON 文件建模為標籤做為樹狀節點的樹狀目錄。 這會同時應用 SQL API 的自動編製索引技術與 Azure Cosmos DB 的 SQL 方言。 SQL 查詢語言包含三個主要部分：   

1. 本質上對應至樹狀結構的較小一組查詢作業 (包括階層式查詢和投射)。 
2. 關聯式作業 (包括複合、篩選、投射、彙總和自我聯結) 的子集。 
3. 可與 (1) 和 (2) 搭配使用的純 JavaScript 型 UDF。  

Azure Cosmos DB 查詢模型嘗試打破功能、效率和簡易性之間的平衡。 Azure Cosmos DB 資料庫引擎會原生編譯和執行 SQL 查詢陳述式。 您可以使用 [REST API](/rest/api/cosmos-db/) 或任何[用戶端 SDK](sql-api-sdk-dotnet.md) 來查詢集合。 .NET SDK 隨附於 LINQ 提供者。

> [!TIP]
> 您可以試試 SQL API，並在[查詢園地](https://www.documentdb.com/sql/demo) (英文) 中針對我們的資料集執行 SQL 查詢。
> 
> 

## <a name="multi-document-transactions"></a>多文件交易
資料庫交易提供安全且可預測的程式設計模型來處理同時的資料變更。 在 RDBMS 中，撰寫商務邏輯的傳統方式是撰寫「預存程序」和/或「觸發程序」，並將它傳送至資料庫伺服器以進行交易式執行。 在 RDBMS 中，需要有應用程式設計人員，才能處理兩個不同的程式設計語言： 

* (非交易式) 應用程式程式設計語言 (例如 JavaScript、Python、C#、Java 等等)
* T-SQL 是交易式程式設計語言，專門由資料庫執行

透過直接在資料庫引擎內深入承諾 JavaScript 和 JSON，Azure Cosmos DB 提供直覺式程式設計模型，以透過預存程序和觸發程序直接在集合上執行 JavaScript 型應用程式邏輯。 這會允許下列兩項動作：

* 直接在資料庫引擎中，有效率地實作 JSON 物件圖形的並行存取控制、復原、自動編製索引
* 在 JavaScript 程式設計語言方面，直接自然表達資料庫交易的控制流程、變數範圍、指派和整合例外狀況處理基本項目

在集合層級註冊的 JavaScript 邏輯接著可以對給定集合的文件發出資料庫作業。 Azure Cosmos DB 可以跨集合內的文件，隱含地在具有快照隔離的環境 ACID 交易內包裝 JavaScript 型預存程序和觸發程序。 在執行期間，如果 JavaScript 擲回例外狀況，則會中止整個交易。 產生的程式設計模型非常簡單，但功能強大。 JavaScript 開發人員會取得「持續性」程式設計模型，同時仍然使用其熟悉的語言建構和程式庫基本。   

直接在資料庫引擎 (與緩衝集區位於相同的位址空間) 內執行 JavaScript 的能力，可對集合的文件啟用資料庫作業的具效能和交易式執行。 此外，Cosmos DB 資料庫引擎會進一步對 JSON 和 JavaScript 執行認可，以去除應用程式與資料庫之的類型系統間的阻抗不相符。   

建立集合之後，您即可使用 [REST API](/rest/api/cosmos-db/) 或任何[用戶端 SDK](sql-api-sdk-dotnet.md)，向集合註冊預存程序、觸發程序及 UDF。 註冊之後，您就可以參考和執行它們。 您可以考慮使用下列完全以 JavaScript 撰寫的預存程序。下列程式碼採用兩個引數 (書籍名稱和作者名稱) 建立新文件，接著再查詢文件，然後加以更新。這些作業全都會在隱含的 ACID 交易中進行。 在執行期間的任何時間點，如果擲回 JavaScript 例外狀況，整個交易便會中止。

    function businessLogic(name, author) {
        var context = getContext();
        var collectionManager = context.getCollection();        
        var collectionLink = collectionManager.getSelfLink()

        // create a new document.
        collectionManager.createDocument(collectionLink,
            {id: name, author: author},
            function(err, documentCreated) {
                if(err) throw new Error(err.message);

                // filter documents by author
                var filterQuery = "SELECT * from root r WHERE r.author = 'George R.'";
                collectionManager.queryDocuments(collectionLink,
                    filterQuery,
                    function(err, matchingDocuments) {
                        if(err) throw new Error(err.message);

                        context.getResponse().setBody(matchingDocuments.length);

                        // Replace the author name for all documents that satisfied the query.
                        for (var i = 0; i < matchingDocuments.length; i++) {
                            matchingDocuments[i].author = "George R. R. Martin";
                            // we don’t need to execute a callback because they are in parallel
                            collectionManager.replaceDocument(matchingDocuments[i]._self,
                                matchingDocuments[i]);   
                        }
                    })
            })
    };

用戶端可以將上面的 JavaScript 邏輯「傳送」至資料庫，以透過 HTTP POST 進行交易式執行。 如需關於使用 HTTP 方法的詳細資訊，請參閱[與 Azure Cosmos DB 資源進行 RESTful 互動 (英文)](https://msdn.microsoft.com/library/azure/mt622086.aspx)。 

    client.createStoredProcedureAsync(collection._self, {id: "CRUDProc", body: businessLogic})
       .then(function(createdStoredProcedure) {
            return client.executeStoredProcedureAsync(createdStoredProcedure.resource._self,
                "NoSQL Distilled",
                "Martin Fowler");
        })
        .then(function(result) {
            console.log(result);
        },
        function(error) {
            console.log(error);
        });


請注意，因為資料庫原本就了解 JSON 和 JavaScript，所以不會有類型系統不符、沒有「OR 對應」或不需要程式碼產生魔術。   

預存程序和觸發程序會透過定義良好的物件模型 (可公開目前集合內容)，與集合及集合內的文件互動。  

不論是使用 [REST API](/rest/api/cosmos-db/) 還是任何[用戶端 SDK](sql-api-sdk-dotnet.md)，都可輕鬆地建立、刪除、讀取或列舉 SQL API 中的集合。 SQL API 一律提供讀取或查詢集合之中繼資料的強式一致性。 刪除集合會自動確定您無法存取其內所含的任何文件、附件、預存程序、觸發程序和 UDF。   

## <a name="stored-procedures-triggers-and-user-defined-functions-udf"></a>預存程序、觸發程序和使用者定義函數 (UDF)
如上一節所述，您可以撰寫直接在資料庫引擎的交易內執行的應用程式邏輯。 應用程式邏輯可以完全以 JavaScript 撰寫，也可以建模為預存程序、觸發程序或 UDF。 預存程序或觸發程序內的 JavaScript 程式碼可以在集合內插入、取代、刪除、讀取或查詢文件。 另一方面，UDF 內的 JavaScript 無法插入、取代或刪除文件。 UDF 會列舉查詢結果集的文件並產生另一個結果集。 若是多重租用，Azure Cosmos DB 會強制執行嚴謹的保留型資源管理。 每個預存程序、觸發程序或 UDF 都會取得固定配量的作業系統資源來執行工作。 此外，預存程序、觸發程序或 UDF 無法連結外部 JavaScript 程式庫，因此，當這些項目超出配置的資源預算時，便會將其列入封鎖清單。 您可以使用 REST API 向集合註冊、取消註冊預存程序、觸發程序或 UDF。  註冊時，預存程序、觸發程序或 UDF 會預先編譯並儲存為位元組程式碼，以在稍後執行。 下節說明如何使用 Azure Cosmos DB JavaScript SDK 來註冊、執行和取消註冊預存程序、觸發程序和 UDF。 JavaScript SDK 是一種透過 [REST API](/rest/api/cosmos-db/) 的簡單包裝函式。 

### <a name="registering-a-stored-procedure"></a>註冊預存程序
透過 HTTP POST 在集合上建立新的預存程序資源，即可註冊預存程序。  

    var storedProc = {
        id: "validateAndCreate",
        body: function (documentToCreate) {
            documentToCreate.id = documentToCreate.id.toUpperCase();

            var collectionManager = getContext().getCollection();
            collectionManager.createDocument(collectionManager.getSelfLink(),
                documentToCreate,
                function(err, documentCreated) {
                    if(err) throw new Error('Error while creating document: ' + err.message;
                    getContext().getResponse().setBody('success - created ' + 
                            documentCreated.name);
                });
        }
    };

    client.createStoredProcedureAsync(collection._self, storedProc)
        .then(function (createdStoredProcedure) {
            console.log("Successfully created stored procedure");
        }, function(error) {
            console.log("Error");
        });

### <a name="executing-a-stored-procedure"></a>執行預存程序
透過將參數傳遞給要求本文中的程序，以對現有預存程序資源發出 HTTP POST，即可執行預存程序。

    var inputDocument = {id : "document1", author: "G. G. Marquez"};
    client.executeStoredProcedureAsync(createdStoredProcedure.resource._self, inputDocument)
        .then(function(executionResult) {
            assert.equal(executionResult, "success - created DOCUMENT1");
        }, function(error) {
            console.log("Error");
        });

### <a name="unregistering-a-stored-procedure"></a>取消註冊預存程序
對現有預存程序資源發出 HTTP DELETE，即可取消註冊預存程序。   

    client.deleteStoredProcedureAsync(createdStoredProcedure.resource._self)
        .then(function (response) {
            return;
        }, function(error) {
            console.log("Error");
        });


### <a name="registering-a-pre-trigger"></a>註冊預先觸發程序
透過 HTTP POST 在集合上建立新的觸發程序資源，即可註冊觸發程序。 您可以指定觸發程序是預先觸發程序還是後續觸發程序，以及可與之相關聯的作業類型 (例如，[建立]、[取代]、[刪除] 或 [全部])。   

    var preTrigger = {
        id: "upperCaseId",
        body: function() {
                var item = getContext().getRequest().getBody();
                item.id = item.id.toUpperCase();
                getContext().getRequest().setBody(item);
        },
        triggerType: TriggerType.Pre,
        triggerOperation: TriggerOperation.All
    }

    client.createTriggerAsync(collection._self, preTrigger)
        .then(function (createdPreTrigger) {
            console.log("Successfully created trigger");
        }, function(error) {
            console.log("Error");
        });

### <a name="executing-a-pre-trigger"></a>執行預先觸發程序
透過要求標頭發出文件資源的 POST/PUT/DELETE 要求時指定現有觸發程序名稱，即可執行觸發程序。  

    client.createDocumentAsync(collection._self, { id: "doc1", key: "Love in the Time of Cholera" }, { preTriggerInclude: "upperCaseId" })
        .then(function(createdDocument) {
            assert.equal(createdDocument.resource.id, "DOC1");
        }, function(error) {
            console.log("Error");
        });

### <a name="unregistering-a-pre-trigger"></a>取消註冊預先觸發程序
對現有觸發程序資源發出 HTTP DELETE，即可取消註冊觸發程序。  

    client.deleteTriggerAsync(createdPreTrigger._self);
        .then(function(response) {
            return;
        }, function(error) {
            console.log("Error");
        });

### <a name="registering-a-udf"></a>註冊 UDF
透過 HTTP POST 在集合上建立新的 UDF 資源，即可註冊 UDF。  

    var udf = { 
        id: "mathSqrt",
        body: function(number) {
                return Math.sqrt(number);
        },
    };
    client.createUserDefinedFunctionAsync(collection._self, udf)
        .then(function (createdUdf) {
            console.log("Successfully created stored procedure");
        }, function(error) {
            console.log("Error");
        });

### <a name="executing-a-udf-as-part-of-the-query"></a>將 UDF 執行為查詢的一部分
UDF 可以指定為部分 SQL 查詢，也可作為一種擴充核心 [SQL 查詢語言](sql-api-sql-query-reference.md)的方法。

    var filterQuery = "SELECT udf.mathSqrt(r.Age) AS sqrtAge FROM root r WHERE r.FirstName='John'";
    client.queryDocuments(collection._self, filterQuery).toArrayAsync();
        .then(function(queryResponse) {
            var queryResponseDocuments = queryResponse.feed;
        }, function(error) {
            console.log("Error");
        });

### <a name="unregistering-a-udf"></a>取消註冊 UDF
只要對現有 UDF 資源發出 HTTP DELETE，即可取消註冊 UDF。  

    client.deleteUserDefinedFunctionAsync(createdUdf._self)
        .then(function(response) {
            return;
        }, function(error) {
            console.log("Error");
        });

雖然以上程式碼片段顯示的是透過 [JavaScript SDK](https://github.com/Azure/azure-documentdb-js) 執行的註冊 (POST)、取消註冊 (PUT)、讀取/列出 (GET) 及執行 (POST)，但您也可以使用 [REST API](/rest/api/cosmos-db/) 或其他[用戶端 SDK](sql-api-sdk-dotnet.md)。 

## <a name="documents"></a>文件
您可以在集合中插入、取代、刪除、讀取、列舉和查詢任意 JSON 文件。 Azure Cosmos DB 不會託管任何結構描述，而且不需要次要索引，就支援逐一查詢集合中的文件。 文件的大小上限為 2 MB。   

Azure Cosmos DB 是真正開放的資料庫服務，不會發明 JSON 文件的任何特殊資料類型 (例如日期時間) 或特定編碼。 Azure Cosmos DB 無須遵循任何特殊 JSON 慣例，即可編寫各種文件之間的關聯性。Azure Cosmos DB 的 SQL 語法提供極有效率的階層式和關係查詢運算子，讓您可以用於查詢及保護文件，不僅無須任何特殊註釋，也無須使用不同的屬性來編寫文件之間的關聯性。  

與所有其他資源相同，使用 REST API 或任何[用戶端 SDK](sql-api-sdk-dotnet.md)，即可輕鬆地建立、取代、刪除、讀取、列舉或查詢文件。 刪除文件時會立即清出對應至所有巢狀附件的配額。 文件的讀取一致性層級會遵循資料庫帳戶的一致性原則。 根據您應用程式的資料一致性需求，可以覆寫每個要求的這個原則。 查詢文件時，讀取一致性會遵循集合上所設定的索引模式。 為求「一致」，這會遵循帳戶的一致性原則。 

## <a name="attachments-and-media"></a>附件和媒體
Azure Cosmos DB 可讓您將二進位 Blob/媒體儲存至 Azure Cosmos DB (每個帳戶最多 2 GB) 或您自己的遠端媒體存放區。 它也可讓您透過特殊文件 (稱為附件) 來呈現媒體的中繼資料。 Azure Cosmos DB 中的附件是一種特殊 (JSON) 文件，可參考儲存在其他位置的媒體/Blob。 附件只是特殊文件，可擷取遠端媒體存放裝置中所儲存媒體的中繼資料 (例如位置、作者等)。 

請考慮使用社交閱讀應用程式，它會使用 Azure Cosmos DB 儲存手寫註釋，以及與指定使用者電子書相關聯的中繼資料 (包括註解、醒目提示、書籤、評等、喜歡/不喜歡等)。   

* 書籍本身的內容儲存在媒體存放裝置 (作為 Azure Cosmos DB 資料庫帳戶的一部分) 或遠端媒體存放區中。 
* 應用程式可能會將每個使用者的中繼資料儲存為不同的文件 (例如 Joe 的 book1 中繼資料儲存在 /colls/joe/docs/book1 所參考的文件中)。 
* 指向使用者給定書籍之內容頁面的附件，儲存在對應的文件中 (例如 /colls/joe/docs/book1/chapter1、/colls/joe/docs/book1/chapter2 等)。 

上述範例會使用易記 ID 來傳達資源階層。 資源是透過 REST API 以根據唯一資源 ID 進行存取。 

針對 Azure Cosmos DB 所管理的媒體，附件的 _media 屬性將會依媒體的 URI 來參考媒體。 Azure Cosmos DB 將會在捨棄所有未完成的參考時，確保回收媒體的記憶體。 如果您上傳新的媒體並填入 _media 以指向新增的媒體，則 Azure Cosmos DB 會自動產生附件。 如果您選擇將媒體儲存在您所管理的遠端 Blob 存放區中 (例如 OneDrive、Azure Storage、DropBox 等)，則還是可以使用附件來參考媒體。 在此情況下，您將自行建立附件，並填入其 _media 屬性中。   

與所有其他資源相同，使用 REST API 或任何用戶端 SDK，即可輕鬆地建立、取代、刪除、讀取或列舉附件。 與文件相同，附件的讀取一致性層級會遵循資料庫帳戶的一致性原則。 根據您應用程式的資料一致性需求，可以覆寫每個要求的這個原則。 查詢附件時，讀取一致性會遵循集合上所設定的索引模式。 為求「一致」，這會遵循帳戶的一致性原則。 
 

## <a name="users"></a>使用者
Azure Cosmos DB 使用者代表用於分組權限的邏輯命名空間。 Azure Cosmos DB 使用者可能對應至身分識別管理系統或預先定義應用程式角色中的使用者。 對於 Azure Cosmos DB，使用者只代表將資料庫下的一組權限分組的抽象概念。   

如需在您的應用程式中實作多重租用，您可以在 Azure Cosmos DB 中建立對應至實際使用者或應用程式租用戶的使用者。 您接著可以建立給定使用者的權限，而權限透過各種集合、文件、附件等對應至存取控制。   

應用程式需要隨著使用者成長而調整時，您可以採用各種方式來共用資料。 您可以建立每位使用者的模型，如下所示：   

* 每個使用者都對應至資料庫。
* 每個使用者都對應至集合。 
* 將對應至多位使用者的文件移至專用的集合。 
* 將對應至多位使用者的文件移至一組的集合。   

不論您選擇的特定分區化策略為何，您可以將實際使用者模型化為 Azure Cosmos DB 資料庫中的使用者，並將微調後的權限關聯至每個使用者。  

![使用者集合][3]  
**分區化策略和模型化使用者**

與所有其他資源相同，使用 REST API 或任何用戶端 SDK，即可輕鬆地在 Azure Cosmos DB 中建立、取代、刪除、讀取或列舉使用者。 Azure Cosmos DB 一律提供讀取或查詢使用者資源之中繼資料的強式一致性。 這值得指出刪除使用者時會自動確保您無法存取其內所含的任何權限。 即使 Azure Cosmos DB 在背景回收佈建為所刪除使用者一部分的權限配額，但是所刪除權限還是立即可以再度使用。  

## <a name="permissions"></a>權限
從存取控制觀點來看，系統會將資源 (例如資料庫帳戶、資料庫、使用者和權限) 視為「管理」  資源，因為這些都需要管理權限。 另一方面，會將資源 (包括集合、文件、附件、預存程序、觸發程序和 UDF) 限制至給定資料庫，並將其視為「應用程式資源」 。 授權模型定義兩種「存取金鑰」來對應兩種資源及可存取這些資源的角色 (即系統管理員和使用者)：「主要金鑰」和「資源金鑰」。 主要金鑰是資料庫帳戶的一部分，並且提供給將佈建資料庫帳戶的開發人員 (或系統管理員)。 此主要金鑰具有系統管理員語意，即它可以用來授權對管理和應用程式資源的存取權。 相較之下，資源金鑰是精細的存取金鑰，可允許存取「特定」  應用程式資源。 因此，它會擷取資料庫使用者與使用者具有特定資源 (例如，集合、文件、附件、預存程序、觸發程序或 UDF) 的權限之間的關聯性。   

取得資源金鑰的唯一方式是透過建立給定使用者的權限資源。 若要建立或擷取權限，授權標頭中必須要有主要金鑰。 權限資源會繫結資源、其存取權和使用者。 建立權限資源之後，使用者只需要具有相關聯的資源金鑰，就能存取相關資源。 因此，可能會以權限資源的邏輯和壓縮呈現來檢視資源金鑰。  

與所有其他資源相同，使用 REST API 或任何用戶端 SDK，即可輕鬆地在 Azure Cosmos DB 中建立、取代、刪除、讀取或列舉權限。 Azure Cosmos DB 一律提供讀取或查詢權限之中繼資料的強式一致性。 

## <a name="next-steps"></a>後續步驟
深入了解如何使用 HTTP 命令來使用資源，請參閱[與 Azure Cosmos DB 資源進行 RESTful 互動 (英文)](https://msdn.microsoft.com/library/azure/mt622086.aspx)。

[1]: media/sql-api-resources/resources1.png
[2]: media/sql-api-resources/resources2.png
[3]: media/sql-api-resources/resources3.png

