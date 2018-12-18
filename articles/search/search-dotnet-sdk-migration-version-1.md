---
title: 升級至 Azure 搜尋服務 .NET SDK 版本 1.1 | Microsoft Docs
description: 升級至 Azure 搜尋服務 .NET SDK 版本 1.1
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: brjohnst
ms.openlocfilehash: ccefd21e2aa89a2b46129956b3c4417d548cbf32
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31796739"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-11"></a>升級至 Azure 搜尋服務 .NET SDK 版本 1.1

如果您使用 1.0.2 預覽版或更舊版本的 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk)，本文會協助您將應用程式升級為使用 1.1 版。

如需包括範例的 SDK 一般逐步解說，請參閱 [如何從 .NET 應用程式使用 Azure 搜尋服務](search-howto-dotnet-sdk.md)。

> [!NOTE]
> 一旦您升級到 1.1 版之後，或如果您已使用 1.1 和 2.0 預覽版 (含) 之間的版本，就應該升級至第 3 版。 如需指示，請參閱[升級至 Azure 搜尋服務 .NET SDK 第 3 版](search-dotnet-sdk-migration.md)。
>

首先，更新 `Microsoft.Azure.Search` 的 NuGet 參考，方法是使用 NuGet 封裝管理員主控台，或是在 Visual Studio 中用滑鼠右鍵按一下您的專案參考並選取 [管理 NuGet 封裝]。

一旦 NuGet 下載新的封裝和其相依性，請重建您的專案。

如果您之前使用版本 1.0.0 預覽版、1.0.1 預覽版，或 1.0.2 預覽版，應能夠成功建置且您已經準備就緒！

如果您之前使用版本 0.13.0 預覽版或更舊版本，您應該會看到如下所示的建置錯誤：

    Program.cs(137,56,137,62): error CS0117: 'Microsoft.Azure.Search.Models.IndexBatch' does not contain a definition for 'Create'
    Program.cs(137,99,137,105): error CS0117: 'Microsoft.Azure.Search.Models.IndexAction' does not contain a definition for 'Create'
    Program.cs(146,41,146,54): error CS1061: 'Microsoft.Azure.Search.IndexBatchException' does not contain a definition for 'IndexResponse' and no extension method 'IndexResponse' accepting a first argument of type 'Microsoft.Azure.Search.IndexBatchException' could be found (are you missing a using directive or an assembly reference?)
    Program.cs(163,13,163,42): error CS0246: The type or namespace name 'DocumentSearchResponse' could not be found (are you missing a using directive or an assembly reference?)

下一步是逐一修正建置錯誤。 大部分錯誤需要變更 SDK 中已重新命名的某些類別和方法名稱。 [版本 1.1 的重大變更清單](#ListOfChangesV1) 一節包含這些名稱變更的清單。

如果您使用自訂類別來建立您文件的模型，且這些類別有不可為 Null 的基本類型屬性 (例如 C# 中的 `int` 或 `bool`)，則您應該要注意 SDK 1.1 版本中的某個錯誤修正。 請參閱 [版本 1.1 的錯誤修正](#BugFixesV1) 一節以取得更多詳細資料。

最後，一旦您已修正任何建置錯誤，您可以變更您的應用程式以視需要利用新的功能。

<a name="ListOfChangesV1"></a>

### <a name="list-of-breaking-changes-in-version-11"></a>版本 1.1 的重大變更清單
下列清單會根據變更影響應用程式程式碼的可能性排序。

#### <a name="indexbatch-and-indexaction-changes"></a>IndexBatch 和 IndexAction 變更
`IndexBatch.Create` 已重新命名為 `IndexBatch.New` 且不再具有 `params` 引數。 您可以針對混合不同類型動作 (合併、刪除等等) 的批次使用 `IndexBatch.New` 。 此外，還有新的靜態方法可以用來建立批次，其中所有動作都相同：`Delete`、`Merge`、`MergeOrUpload` 和 `Upload`。

`IndexAction` 不再具有公用建構函式且其屬性現在固定不變。 您應該使用新的靜態方法來針對不同用途建立動作：`Delete`、`Merge`、`MergeOrUpload` 和 `Upload`。 `IndexAction.Create` 已移除。 如果您使用僅採用文件的多載，請務必改為使用 `Upload` 。

##### <a name="example"></a>範例
如果您的程式碼如下所示：

    var batch = IndexBatch.Create(documents.Select(doc => IndexAction.Create(doc)));
    indexClient.Documents.Index(batch);

您可以將其變更如下以修正任何建置錯誤：

    var batch = IndexBatch.New(documents.Select(doc => IndexAction.Upload(doc)));
    indexClient.Documents.Index(batch);

如果您想要的話，可以進一步加以簡化如下：

    var batch = IndexBatch.Upload(documents);
    indexClient.Documents.Index(batch);

#### <a name="indexbatchexception-changes"></a>IndexBatchException 變更
`IndexBatchException.IndexResponse` 屬性已重新命名為 `IndexingResults`，而其類型現在是 `IList<IndexingResult>`。

##### <a name="example"></a>範例
如果您的程式碼如下所示：

    catch (IndexBatchException e)
    {
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexResponse.Results.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

您可以將其變更如下以修正任何建置錯誤：

    catch (IndexBatchException e)
    {
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

<a name="OperationMethodChanges"></a>

#### <a name="operation-method-changes"></a>作業方法變更
Azure 搜尋服務 .NET SDK 中的每項作業都針對同步和非同步呼叫端公開為一組方法多載。 在版本 1.1 中，這些方法多載的簽章和分解已經變更。

例如，較舊版本的 SDK 中的「取得索引統計資料」作業會公開這些簽章：

在 `IIndexOperations`中：

    // Asynchronous operation with all parameters
    Task<IndexGetStatisticsResponse> GetStatisticsAsync(
        string indexName,
        CancellationToken cancellationToken);

在 `IndexOperationsExtensions`中：

    // Asynchronous operation with only required parameters
    public static Task<IndexGetStatisticsResponse> GetStatisticsAsync(
        this IIndexOperations operations,
        string indexName);

    // Synchronous operation with only required parameters
    public static IndexGetStatisticsResponse GetStatistics(
        this IIndexOperations operations,
        string indexName);

在版本 1.1 中，相同作業的方法簽章看起來像這樣：

在 `IIndexesOperations`中：

    // Asynchronous operation with lower-level HTTP features exposed
    Task<AzureOperationResponse<IndexGetStatisticsResult>> GetStatisticsWithHttpMessagesAsync(
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
        Dictionary<string, List<string>> customHeaders = null,
        CancellationToken cancellationToken = default(CancellationToken));

在 `IndexesOperationsExtensions`中：

    // Simplified asynchronous operation
    public static Task<IndexGetStatisticsResult> GetStatisticsAsync(
        this IIndexesOperations operations,
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
        CancellationToken cancellationToken = default(CancellationToken));

    // Simplified synchronous operation
    public static IndexGetStatisticsResult GetStatistics(
        this IIndexesOperations operations,
        string indexName,
        SearchRequestOptions searchRequestOptions = default(SearchRequestOptions));

從版本 1.1 開始，Azure 搜尋服務 .NET SDK 會採用不同的作業方法組織方式：

* 選擇性參數現在模型化為預設參數，而不是其他方法多載。 這樣可以減少方法多載的數目，有時候減少的數量相當大。
* 擴充方法現在會隱藏許多來自呼叫端的無關 HTTP 詳細資料。 例如，較舊版本的 SDK 會傳回具有 HTTP 狀態碼的回應物件，您通常不需要檢查，因為作業方法會針對表示發生錯誤的任何狀態碼擲回 `CloudException` 。 新的擴充方法只會傳回模型物件，省卻您必須在程式碼中將它們解除包裝的麻煩。
* 相反地，核心介面現在會公開方法，在您需要時為您提供 HTTP 層級更多的控制權。 您現在可以傳入自訂 HTTP 標頭以包含在要求中，新的 `AzureOperationResponse<T>` 傳回類型給予您針對作業對於 `HttpRequestMessage` 和 `HttpResponseMessage` 的直接存取。 `AzureOperationResponse` 是在 `Microsoft.Rest.Azure` 命名空間中定義，並且取代 `Hyak.Common.OperationResponse`。

#### <a name="scoringparameters-changes"></a>ScoringParameters 變更
名為 `ScoringParameter` 的新類別已在最新的 SDK 中加入，讓您更輕鬆地提供參數給搜尋查詢中的評分設定檔。 先前 `SearchParameters` 類別的 `ScoringProfiles` 屬性的類型是 `IList<string>`；現在它的類型是 `IList<ScoringParameter>`。

##### <a name="example"></a>範例
如果您的程式碼如下所示：

    var sp = new SearchParameters();
    sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
    sp.ScoringParameters = new[] { "featuredParam-featured", "mapCenterParam-" + lon + "," + lat };

您可以將其變更如下以修正任何建置錯誤： 

    var sp = new SearchParameters();
    sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
    sp.ScoringParameters =
        new[]
        {
            new ScoringParameter("featuredParam", new[] { "featured" }),
            new ScoringParameter("mapCenterParam", GeographyPoint.Create(lat, lon))
        };

#### <a name="model-class-changes"></a>模型類別變更
由於[作業方法變更](#OperationMethodChanges)中所述的簽章變更，`Microsoft.Azure.Search.Models` 命名空間中的許多類別都已重新命名或移除。 例如︰

* `IndexDefinitionResponse` 已由 `AzureOperationResponse<Index>` 取代
* `DocumentSearchResponse` 已重新命名為 `DocumentSearchResult`
* `IndexResult` 已重新命名為 `IndexingResult`
* `Documents.Count()` 現在會傳回 `long`，具有文件計數而不是 `DocumentCountResponse`
* `IndexGetStatisticsResponse` 已重新命名為 `IndexGetStatisticsResult`
* `IndexListResponse` 已重新命名為 `IndexListResult`

總而言之，已存在的 `OperationResponse`衍生類別只是用來包裝已移除的模型物件。 其餘的類別已將其尾碼從 `Response` 變更為 `Result`。

##### <a name="example"></a>範例
如果您的程式碼如下所示：

    IndexerGetStatusResponse statusResponse = null;

    try
    {
        statusResponse = _searchClient.Indexers.GetStatus(indexer.Name);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
        return;
    }

    IndexerExecutionResult lastResult = statusResponse.ExecutionInfo.LastResult;

您可以將其變更如下以修正任何建置錯誤：

    IndexerExecutionInfo status = null;

    try
    {
        status = _searchClient.Indexers.GetStatus(indexer.Name);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
        return;
    }

    IndexerExecutionResult lastResult = status.LastResult;

##### <a name="response-classes-and-ienumerable"></a>回應類別和 IEnumerable
可能會影響您程式碼的其他變更，就是讓集合不再實作 `IEnumerable<T>`的回應類別。 您可以改為直接存取集合屬性。 例如，如果您的程式碼如下所示：

    DocumentSearchResponse<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
    foreach (SearchResult<Hotel> result in response)
    {
        Console.WriteLine(result.Document);
    }

您可以將其變更如下以修正任何建置錯誤：

    DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
    foreach (SearchResult<Hotel> result in response.Results)
    {
        Console.WriteLine(result.Document);
    }

##### <a name="special-case-for-web-applications"></a>Web 應用程式的特殊案例
如果您有 Web 應用程式會直接將 `DocumentSearchResponse` 序列化，以將搜尋結果傳送至瀏覽器，您將需要變更程式碼，否則結果不會正確地序列化。 例如，如果您的程式碼如下所示：

    public ActionResult Search(string q = "")
    {
        // If blank search, assume they want to search everything
        if (string.IsNullOrWhiteSpace(q))
            q = "*";

        return new JsonResult
        {
            JsonRequestBehavior = JsonRequestBehavior.AllowGet,
            Data = _featuresSearch.Search(q)
        };
    }

您可以藉由取得搜尋回應的 `.Results` 屬性以修正搜尋結果轉譯，來加以變更：

    public ActionResult Search(string q = "")
    {
        // If blank search, assume they want to search everything
        if (string.IsNullOrWhiteSpace(q))
            q = "*";

        return new JsonResult
        {
            JsonRequestBehavior = JsonRequestBehavior.AllowGet,
            Data = _featuresSearch.Search(q).Results
        };
    }

您必須自行在您的程式碼中尋找這種情況；**編譯器不會警告您**，因為 `JsonResult.Data` 的類型是 `object`。

#### <a name="cloudexception-changes"></a>CloudException 變更
`CloudException` 類別已經從 `Hyak.Common` 命名空間移至 `Microsoft.Rest.Azure` 命名空間。 此外，`Error` 屬性已重新命名為 `Body`。

#### <a name="searchserviceclient-and-searchindexclient-changes"></a>SearchServiceClient 和 SearchIndexClient 變更
`Credentials` 屬性的類型已經從 `SearchCredentials` 變更為其基底類別，`ServiceClientCredentials`。 如果您需要存取 `SearchIndexClient` 或 `SearchServiceClient` 的 `SearchCredentials`，請使用新的 `SearchCredentials` 屬性。

在舊版的 SDK 中，`SearchServiceClient` 和 `SearchIndexClient` 有建構函式，會採用 `HttpClient` 參數。 這些建構函式已由使用 `HttpClientHandler` 的建構函式和 `DelegatingHandler` 物件的陣列取代。 如此可讓在需要時將自訂處理常式安裝至前置處理 HTTP 要求更加簡易。

最後，採用 `Uri` 和 `SearchCredentials` 的建構函式已變更。 例如，如果您的程式碼如下所示：

    var client =
        new SearchServiceClient(
            new SearchCredentials("abc123"),
            new Uri("http://myservice.search.windows.net"));

您可以將其變更如下以修正任何建置錯誤：

    var client =
        new SearchServiceClient(
            new Uri("http://myservice.search.windows.net"),
            new SearchCredentials("abc123"));

另外請注意，認證參數的類型已變更為 `ServiceClientCredentials`。 這不會影響您的程式碼，因為 `SearchCredentials` 衍生自 `ServiceClientCredentials`。

#### <a name="passing-a-request-id"></a>傳送要求 ID
在較舊版本的 SDK 中，您可以在 `SearchServiceClient` 或 `SearchIndexClient` 上設定要求 ID，且它會納入 REST API 的每個要求中。 如果您需要連絡支援人員，疑難排解搜尋服務的問題時，相當有用。 不過，為每個作業設定唯一要求 ID 更加有用，而不是對所有作業使用相同 ID。 基於這個原因，`SearchServiceClient` 和 `SearchIndexClient` 的 `SetClientRequestId` 方法已移除。 您可以改為透過選擇性 `SearchRequestOptions` 參數將要求 ID 傳送給每個作業方法。

> [!NOTE]
> 在未來的 SDK 版本中，我們會新增新機制，在用戶端物件上全域設定要求 ID，與其他 Azure SDK 所使用的方法一致。
> 
> 

#### <a name="example"></a>範例
如果您的程式碼如下所示：

    client.SetClientRequestId(Guid.NewGuid());
    ...
    long count = client.Documents.Count();

您可以將其變更如下以修正任何建置錯誤：

    long count = client.Documents.Count(new SearchRequestOptions(requestId: Guid.NewGuid()));

#### <a name="interface-name-changes"></a>介面名稱變更
作業群組介面名稱皆已變更，與其對應的屬性名稱一致：

* `ISearchServiceClient.Indexes` 的類型已從 `IIndexOperations` 重新命名為 `IIndexesOperations`。
* `ISearchServiceClient.Indexers` 的類型已從 `IIndexerOperations` 重新命名為 `IIndexersOperations`。
* `ISearchServiceClient.DataSources` 的類型已從 `IDataSourceOperations` 重新命名為 `IDataSourcesOperations`。
* `ISearchIndexClient.Documents` 的類型已從 `IDocumentOperations` 重新命名為 `IDocumentsOperations`。

這項變更不會影響您的程式碼，除非您針對測試目的建立這些介面的模擬。

<a name="BugFixesV1"></a>

### <a name="bug-fixes-in-version-11"></a>版本 1.1 的錯誤修正
在舊版的 Azure 搜尋服務 .NET SDK 中，有與自訂模型類別的序列化相關的錯誤。 如果您建立具有不可為 null 值類型的屬性的自訂模型類別，可能會發生錯誤。

#### <a name="steps-to-reproduce"></a>重現的步驟
建立具有不可為 null 值類型的屬性的自訂模型類別。 例如，加入類型 `int` 而不是 `int?` 的公用 `UnitCount` 屬性。

如果要使用該類型的預設值編製文件的索引 (例如， `int`的 0)，欄位在 Azure 搜尋服務中將會是 null。 如果您後續搜尋該文件，`Search` 呼叫將會擲回 `JsonSerializationException`，提報它無法將 `null` 轉換為 `int`。

此外，篩選可能無法如預期般執行，因為 null 被寫入索引，而不是預期的值。

#### <a name="fix-details"></a>修正詳細資料
我們已在 SDK 的版本 1.1 中修正此問題。 現在，如果您有一個如下的模型類別：

    public class Model
    {
        public string Key { get; set; }

        public int IntValue { get; set; }
    }

而且您將 `IntValue` 設定為 0，該值現在會在線路上正確序列化為 0，並且在索引中儲存為 0。 來回行程也如預期般運作。

這種方法有一個需要注意的可能問題：如果您使用具有不可為 null 屬性的模型類型，您必須保證  索引中沒有任何文件對於對應欄位包含 null 值。 SDK 和 Azure 搜尋服務 REST API 都不會協助您強制執行。

這不只是假設性的問題：如果您在類型為 `Edm.Int32` 的現有索引中新增欄位， 更新索引定義之後，所有文件對於該新的欄位具有 null 值 (因為所有類型在 Azure 搜尋服務中都可為 null)。 如果您接著對該欄位使用 `int` 屬性不可為 Null 的模型類別，就會在嘗試擷取文件時得到類似這樣的 `JsonSerializationException`：

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

基於這個原因，我們仍然建議您在模型類別中使用可為 null 的類型做為最佳作法。

如需有關此錯誤和修正的詳細資訊，請參閱 [GitHub 上的這個問題](https://github.com/Azure/azure-sdk-for-net/issues/1063)。

