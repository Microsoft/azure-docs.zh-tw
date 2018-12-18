---
title: 如何從 .NET 應用程式使用 Azure 搜尋服務 | Microsoft Docs
description: 如何從 .NET 應用程式使用 Azure 搜尋服務
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: brjohnst
ms.openlocfilehash: 19913f9c30992e833e5435af7066611d4662ba56
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525632"
---
# <a name="how-to-use-azure-search-from-a-net-application"></a>如何從 .NET 應用程式使用 Azure 搜尋服務
本文會逐步指引您學會如何使用 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk)。 您可以透過 .NET SDK，利用 Azure 搜尋服務在應用程式中實作豐富的搜尋經驗。

## <a name="whats-in-the-azure-search-sdk"></a>Azure 搜尋服務 SDK 中有哪些內容
SDK 包含數個用戶端程式庫，可讓您管理索引、資料來源、索引子及同義字地圖，以及上傳和管理文件，還可以執行查詢，而且一律不需要處理 HTTP 和 JSON 的細節。 這些用戶端程式庫都以 NuGet 套件來散發。

主要的 NuGet 套件是 `Microsoft.Azure.Search`，這是包含所有其他相依套件的中繼套件。 如果您剛開始使用，或知道應用程式將需要 Azure 搜尋服務的所有功能，則請使用此套件。

SDK 中還有其他 NuGet 套件：
 
  - `Microsoft.Azure.Search.Data`：如果您正在開發使用 Azure 搜尋服務的 .NET 應用程式，而且只需要查詢或更新索引中的文件，則請使用此套件。 如果您也需要建立或更新索引、同義字地圖或其他服務層級資源，請改用 `Microsoft.Azure.Search` 套件。
  - `Microsoft.Azure.Search.Service`：如果您正在開發 .NET 自動化來管理 Azure 搜尋服務的索引、同義字地圖、索引子、資料來源或其他服務層級資源，請使用此套件。 如果您只需要查詢或更新索引中的文件，請改用 `Microsoft.Azure.Search.Data` 套件。 如果您需要 Azure 搜尋服務的所有功能，請改用 `Microsoft.Azure.Search` 套件。
  - `Microsoft.Azure.Search.Common`：Azure 搜尋服務 .NET 程式庫所需的常用類型。 您不需要在應用程式中直接使用此套件；此套件乃是做為相依性之用。

各種用戶端程式庫會定義類別，例如 `Index`、`Field` 及 `Document`，以及定義作業，例如 `SearchServiceClient` 和 `SearchIndexClient` 類別上的 `Indexes.Create` 和 `Documents.Search`。 這些類別可編成以下命名空間：

* [Microsoft.Azure.Search](https://docs.microsoft.com/dotnet/api/microsoft.azure.search)
* [Microsoft.Azure.Search.Models](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models)

目前的 Azure 搜尋服務 .NET SDK 版本現在正式推出。 如果您想提供意見反應給我們，讓我們可以將您的意見併入下一個版本中，請瀏覽我們的 [意見回應頁面](https://feedback.azure.com/forums/263029-azure-search/)。

.NET SDK 支援 `2017-11-11` 版的 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/)。 此版本現在支援同義字，以及索引子的累加增強功能。 不屬於此版本的預覽功能 (例如 JSON 陣列和 CSV 檔案編製索引的支援) 處於[預覽](search-api-2016-09-01-preview.md)狀態，並可透過 [4.0-preview 版 .NET SDK](https://aka.ms/search-sdk-preview) 取得。

此 SDK 不支援[管理作業](https://docs.microsoft.com/rest/api/searchmanagement/)，例如建立及調整搜尋服務和管理 API 金鑰。 如果您需要從 .NET 應用程式管理搜尋資源，您可以使用 [Azure 搜尋服務 .NET 管理 SDK](https://aka.ms/search-mgmt-sdk)。

## <a name="upgrading-to-the-latest-version-of-the-sdk"></a>升級到最新版本的 SDK
如果您已經在使用舊版的 Azure 搜尋服務 .NET SDK，而您想要升級至新的正式推出版本， [這篇文章](search-dotnet-sdk-migration-version-5.md) 會說明如何進行。

## <a name="requirements-for-the-sdk"></a>SDK 的需求
1. Visual Studio 2017。
2. 擁有 Azure Search 服務。 為了使用 SDK，您需要為服務命名，還需要一或多個 API 金鑰。 [在入口網站建立服務](search-create-service-portal.md) 可協助您執行這些步驟。
3. 使用 Visual Studio 中的 [管理 NuGet 封裝] 下載 Azure 搜尋服務 .NET SDK [NuGet 封裝](http://www.nuget.org/packages/Microsoft.Azure.Search) 。 只要在 NuGet.org 搜尋套件名稱 `Microsoft.Azure.Search` (或者，如果您只需要某個功能子集，則搜尋上述其中一個其他套件名稱)。

Azure 搜尋服務 .NET SDK 支援以 .NET Framework 4.5.2 (含) 以上版本，以及 .NET Core 為目標的應用程式。

## <a name="core-scenarios"></a>核心案例
您必須在搜尋應用程式中執行一些作業。 本教學課程中涵蓋了這些主要情節：

* 建立索引
* 透過文件填入索引
* 使用全文檢索搜尋和篩選來搜尋文件

這些情節的說明都隨附範例程式碼， 歡迎在您的應用程式中使用這些程式碼片段。

### <a name="overview"></a>概觀
我們將探索的範例應用程式，會建立名為 "hotels" 的新索引，並透過一些文件填入索引，然後執行一些搜尋查詢。 以下是主要程式，該程式顯示了整體流程：

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteHotelsIndexIfExists(serviceClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateHotelsIndex(serviceClient);

    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);

    ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

> [!NOTE]
> 您可以在 [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) 上尋找本逐步解說中所用範例應用程式的完整原始程式碼。
> 
>

我們會逐步說明， 首先必須建立新的 `SearchServiceClient`。 此物件可讓您管理索引。 若要建構此物件，必須提供 Azure Search 服務名稱和系統管理 API 金鑰。 您可以在[範例應用程式](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)的 `appsettings.json` 檔案中輸入這項資訊。

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

> [!NOTE]
> 如果提供不正確的金鑰 (例如，需要系統管理金鑰卻提供查詢金鑰)，則 `SearchServiceClient` 會在您第一次用它呼叫作業方法時 (例如 `Indexes.Create`) 擲回 `CloudException`，並附上「禁止」的錯誤訊息。 如果遇到此情況，請按兩下我們的 API 金鑰。
> 
> 

以下幾行程式碼會呼叫建立名為 "hotels" 索引的方法，如果索引已存在，請加以刪除。 稍後我們會逐項執行這些方法。

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteHotelsIndexIfExists(serviceClient);

Console.WriteLine("{0}", "Creating index...\n");
CreateHotelsIndex(serviceClient);
```

接著必須填入索引。 若要這麼做，必須要有 `SearchIndexClient`。 取得的方式有兩種：建構該項目，或用 `SearchServiceClient` 呼叫 `Indexes.GetClient`。 為方便起見，我們使用後者。

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

> [!NOTE]
> 在一般搜尋應用程式中，索引的管理和填入是由搜尋查詢的個別元件所處理。 `Indexes.GetClient` 可以很輕易填入索引，因為它可以節省提供其他 `SearchCredentials` 的麻煩。 執行方法是將用於建立 `SearchServiceClient` 的系統管理金鑰傳遞至新的 `SearchIndexClient`。 但在執行查詢的應用程式中，最好直接建立 `SearchIndexClient` ，如此一來就可以傳遞查詢金鑰，而非系統管理金鑰。 這不但符合最低權限的準則，也可以讓您的應用程式更安全。 您可以 [在這裡](https://docs.microsoft.com/rest/api/searchservice/#authentication-and-authorization)深入了解系統管理金鑰和查詢金鑰。
> 
> 

現在我們有 `SearchIndexClient`，可以開始填入索引。 這要使用另一個方法來執行，稍後我們會逐項說明。

```csharp
Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(indexClient);
```

最後，我們會執行一些搜尋查詢並顯示結果。 這次我們使用不同的 `SearchIndexClient`：

```csharp
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

RunQueries(indexClientForQueries);
```

我們稍後會仔細查看 `RunQueries` 方法。 以下是可建立新 `SearchIndexClient` 的程式碼：

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "hotels", new SearchCredentials(queryApiKey));
    return indexClient;
}
```

這次我們使用查詢金鑰，因為我們不需要索引的寫入存取權。 您可以在[範例應用程式](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)的 `appsettings.json` 檔案中輸入這項資訊。

如果使用有效的服務名稱和 API 金鑰執行此應用程式，則輸出應該會看起來如下：

    Deleting index...
    
    Creating index...
    
    Uploading documents...
    
    Waiting for documents to be indexed...
    
    Search the entire index for the term 'budget' and return only the hotelName field:
    
    Name: Roach Motel
    
    Apply a filter to the index to find hotels cheaper than $150 per night, and return the hotelId and description:
    
    ID: 2   Description: Cheapest hotel in town
    ID: 3   Description: Close to town hall and the river
    
    Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:
    
    Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
    Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00
    
    Search the entire index for the term 'motel':
    
    ID: 2   Base rate: 79.99        Description: Cheapest hotel in town     Description (French): Hôtel le moins cher en ville      Name: Roach Motel       Category: Budget        Tags: [motel, budget]   Parking included: yes   Smoking allowed: yes    Last renovated on: 4/28/1982 12:00:00 AM +00:00 Rating: 1/5     Location: Latitude 49.678581, longitude -122.131577
    
    Complete.  Press any key to end application...

本文結尾會提供完整的應用程式原始程式碼。

接著，我們要進一步了解 `Main`所呼叫的每個方法。

### <a name="creating-an-index"></a>建立索引
建立 `SearchServiceClient`後，`Main` 會接著刪除 "hotels" 索引 (如果已經存在)。 刪除方法如下：

```csharp
private static void DeleteHotelsIndexIfExists(SearchServiceClient serviceClient)
{
    if (serviceClient.Indexes.Exists("hotels"))
    {
        serviceClient.Indexes.Delete("hotels");
    }
}
```

此方法使用指定的 `SearchServiceClient` 查看索引是否存在，如果存在，就加以刪除。

> [!NOTE]
> 為簡單起見，本文的範例程式碼使用 Azure 搜尋服務 .NET SDK 的同步方法。 我們建議您在應用程式中使用非同步方法，讓應用程式保有可擴充性且回應靈敏。 例如，您可以在以上方法中使用 `ExistsAsync` 和 `DeleteAsync`，而非 `Exists` 和 `Delete`。
> 
> 

接著，`Main` 會呼叫此方法建立新的 "hotels" 索引：

```csharp
private static void CreateHotelsIndex(SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = "hotels",
        Fields = FieldBuilder.BuildForType<Hotel>()
    };

    serviceClient.Indexes.Create(definition);
}
```

此方法會以定義新索引結構描述的 `Field` 物件清單，建立新的 `Index` 物件。 每個欄位均有一個名稱、資料類型和一些屬性，以用於定義欄位的搜尋行為。 `FieldBuilder` 類別會檢查指定 `Hotel` 模型類別的公用屬性 (property) 和屬性 (attribute)，進而使用反映來建立索引的 `Field` 物件清單。 我們稍後會仔細查看 `Hotel` 類別。

> [!NOTE]
> 如有需要，您永遠可以直接建立 `Field` 物件清單，而不是使用`FieldBuilder`。 例如，您可能不想使用模型類別，或您可能需要使用不想藉由新增屬性來修改的現有模型類別。
>
> 

除了欄位之外，您還可以新增評分設定檔、建議工具或 CORS 選項到 Index (基於簡化目的，範例已省略這些選項)。 如需 Index 物件以及其組成部分的詳細資訊，請參閱 [SDK 參考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index#microsoft_azure_search_models_index)和 [Azure 搜尋服務 REST API 參考](https://docs.microsoft.com/rest/api/searchservice/)。

### <a name="populating-the-index"></a>填入索引
`Main` 的下一步是填入新建立的索引。 執行方法如下：

```csharp
private static void UploadDocuments(ISearchIndexClient indexClient)
{
    var hotels = new Hotel[]
    {
        new Hotel()
        { 
            HotelId = "1", 
            BaseRate = 199.0, 
            Description = "Best hotel in town",
            DescriptionFr = "Meilleur hôtel en ville",
            HotelName = "Fancy Stay",
            Category = "Luxury", 
            Tags = new[] { "pool", "view", "wifi", "concierge" },
            ParkingIncluded = false, 
            SmokingAllowed = false,
            LastRenovationDate = new DateTimeOffset(2010, 6, 27, 0, 0, 0, TimeSpan.Zero), 
            Rating = 5, 
            Location = GeographyPoint.Create(47.678581, -122.131577)
        },
        new Hotel()
        { 
            HotelId = "2", 
            BaseRate = 79.99,
            Description = "Cheapest hotel in town",
            DescriptionFr = "Hôtel le moins cher en ville",
            HotelName = "Roach Motel",
            Category = "Budget",
            Tags = new[] { "motel", "budget" },
            ParkingIncluded = true,
            SmokingAllowed = true,
            LastRenovationDate = new DateTimeOffset(1982, 4, 28, 0, 0, 0, TimeSpan.Zero),
            Rating = 1,
            Location = GeographyPoint.Create(49.678581, -122.131577)
        },
        new Hotel() 
        { 
            HotelId = "3", 
            BaseRate = 129.99,
            Description = "Close to town hall and the river"
        }
    };

    var batch = IndexBatch.Upload(hotels);

    try
    {
        indexClient.Documents.Index(batch);
    }
    catch (IndexBatchException e)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
}
```

此方法分四個部分。 第一個部分會建立 `Hotel` 物件的陣列，做為我們上傳到索引的輸入資料。 為簡單起見，此資料採硬式編碼。 在您的應用程式中，資料可能會來自像是 SQL 資料庫的外部資料來源。

第二個部分會建立包含文件的 `IndexBatch` 。 您在建立批次時 (在此案例中，是藉由呼叫 `IndexBatch.Upload`)，指定要套用至該批次的作業。 接著以 `Documents.Index` 方法將該 Batch 上傳至 Azure 搜尋服務索引。

> [!NOTE]
> 在此範例中，我們只上傳文件。 如果您想要將變更合併至現有的文件，或是刪除文件，您可以改為呼叫 `IndexBatch.Merge`、`IndexBatch.MergeOrUpload` 或 `IndexBatch.Delete` 來建立批次。 您也可以在單一批次中混合不同的作業，方法是呼叫 `IndexBatch.New`，它會採用一組 `IndexAction` 物件，其中每個物件都會要求 Azure 搜尋服務針對文件執行特定的作業。 您可以建立每個擁有自己作業的 `IndexAction`，方法是呼叫對應的方法，例如 `IndexAction.Merge`、`IndexAction.Upload` 等等。
> 
> 

此方法的第三部分是擷取區塊，該區塊會為編制索引處理重要錯誤情況。 如果您的 Azure Search 服務無法將 Batch 中的一些文件編制索引，則 `Documents.Index` 會擲回 `IndexBatchException`。 如果您在服務負載過重時編制文件的索引，就會發生此情況。 **我們強烈建議您在程式碼中明確處理此情況。** 您可以延遲，然後重新嘗試將失敗的文件編制索引，或像範例一樣加以記錄並繼續，或是根據您應用程式的資料一致性需求執行其他操作。

> [!NOTE]
> 您可以使用 `FindFailedActionsToRetry` 方法來建構新的批次，該批次僅包含先前呼叫 `Index` 時失敗的動作。 此方法記載於[這裡](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.indexbatchexception#Microsoft_Azure_Search_IndexBatchException_FindFailedActionsToRetry_Microsoft_Azure_Search_Models_IndexBatch_System_String_)，而且有如何[在 StackOverflow 上](http://stackoverflow.com/questions/40012885/azure-search-net-sdk-how-to-use-findfailedactionstoretry)正確使用它的討論。
>
>

最後，`UploadDocuments` 方法會延遲兩秒。 您的 Azure 搜尋服務中會發生非同步索引編製，因此範例應用程式必須稍待一會，才能確定文件已準備好可供搜尋。 通常只有在示範、測試和範例應用程式中，才需要這類延遲。

#### <a name="how-the-net-sdk-handles-documents"></a>.NET SDK 如何處理文件
您可能想知道 Azure 搜尋服務 .NET SDK 如何能夠將使用者定義的類別執行個體 (例如 `Hotel` 上傳至索引。 為了回答這問題，我們來看一下 `Hotel` 類別：

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;
using Newtonsoft.Json;

// The SerializePropertyNamesAsCamelCase attribute is defined in the Azure Search .NET SDK.
// It ensures that Pascal-case property names in the model class are mapped to camel-case
// field names in the index.
[SerializePropertyNamesAsCamelCase]
public partial class Hotel
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsFilterable]
    public string HotelId { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public double? BaseRate { get; set; }

    [IsSearchable]
    public string Description { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.FrLucene)]
    [JsonProperty("description_fr")]
    public string DescriptionFr { get; set; }

    [IsSearchable, IsFilterable, IsSortable]
    public string HotelName { get; set; }

    [IsSearchable, IsFilterable, IsSortable, IsFacetable]
    public string Category { get; set; }

    [IsSearchable, IsFilterable, IsFacetable]
    public string[] Tags { get; set; }

    [IsFilterable, IsFacetable]
    public bool? ParkingIncluded { get; set; }

    [IsFilterable, IsFacetable]
    public bool? SmokingAllowed { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public DateTimeOffset? LastRenovationDate { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public int? Rating { get; set; }

    [IsFilterable, IsSortable]
    public GeographyPoint Location { get; set; }
}
```

首先要注意的是，每個 `Hotel` 的公用屬性會對應索引定義中的欄位，但這之中有一項關鍵的差異：每個欄位的名稱會以小寫字母 (「駝峰式命名法」) 為開頭，而每個 `Hotel` 的公用屬性名稱會以大小字母 (「巴斯卡命名法」) 為開頭。 這在執行資料繫結、而目標結構描述在應用程式開發人員控制範圍之外的 .NET 應用程式中很常見。 與其違反 .NET 命名方針，使屬性名稱為駝峰式命名法，您可以改用 `[SerializePropertyNamesAsCamelCase]` 屬性，告訴 SDK 自動將屬性名稱對應至駝峰式命名法。

> [!NOTE]
> Azure 搜尋服務 .NET SDK 使用 [NewtonSoft JSON.NET](http://www.newtonsoft.com/json/help/html/Introduction.htm) 程式庫來將您的自訂模型物件序列化到 JSON 中，以及將您在 JSON 中的自訂模型物件還原序列化。 如有需要，您可以自訂這個序列化的過程。 如需詳細資訊，請參閱[使用 JSON.NET 自訂序列化](#JsonDotNet)。
> 
> 

第二個要注意的是裝飾每個公用屬性 (property) 的屬性 (attribute)，例如 `IsFilterable`、`IsSearchable`、`Key` 和 `Analyzer`。 這些屬性直接對應至 [Azure 搜尋服務索引的對應屬性](https://docs.microsoft.com/rest/api/searchservice/create-index#request)。 `FieldBuilder` 類別會使用這些屬性來建構索引的欄位定義。

第三個要注意的是，`Hotel` 類別為公用屬性的資料類型。 這些屬性的 .NET 類型會對應至索引定義中，與其相當的欄位類型。 例如，`Category` 字串屬性會對應至 `category` 欄位 (此欄位屬於 `Edm.String` 類型)。 `bool?` 與 `Edm.Boolean`、`DateTimeOffset?` 與 `Edm.DateTimeOffset` 等，它們之間也有類似的類型對應。類型對應的特定規則和 `Documents.Get` 方法已一起記載於 [Azure 搜尋服務 .NET SDK 參考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations#Microsoft_Azure_Search_IDocumentsOperations_GetWithHttpMessagesAsync__1_System_String_System_Collections_Generic_IEnumerable_System_String__Microsoft_Azure_Search_Models_SearchRequestOptions_System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_)。 `FieldBuilder` 類別會為您處理這項對應，但它仍有助您了解您需要對任何序列化問題進行疑難排解的情況。

這讓使用您的類別做為文件可雙向有效；您也可以擷取搜尋結果，然後讓 SDK 將結果自動還原序列化為您選擇的類型，我們會在下一節中看到這部分。

> [!NOTE]
> Azure 搜尋服務 .NET SDK 還支援使用 `Document` 類別的動態類型文件，也就是欄位名稱與欄位值的索引鍵/值對應。 當您在設計階段卻不知道索引的結構描述時，這很實用，否則要繫結到特定模型類別會很麻煩。 SDK 中所有處理文件的方法，都有可搭配 `Document` 類別使用的多載，以及使用泛型類型參數的強類型多載。 本教學課程的範例程式碼只使用了後者。 `Document` 類別繼承自 `Dictionary<string, object>`。 如需其他詳細資訊，請前往[這裡](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.document#microsoft_azure_search_models_document)。
> 
> 

**為什麼您應該使用 Nullable 資料類型**

當您將自己的模型類別設計為可對應至 Azure 搜尋服務索引時，建議您將 `bool` 和 `int` 等的值類型屬性宣告為可為 Null (例如宣告為 `bool?`，而不是 `bool`)。 如果您使用不可為 Null 的屬性，則必須保證  索引中沒有任何文件的對應欄位包含 Null 值。 SDK 和 Azure 搜尋服務都不會協助您強制執行這項規定。

這不只是假設性的問題：如果您在類型為 `Edm.Int32` 的現有索引中新增欄位， 更新索引定義之後，所有文件對於該新的欄位具有 null 值 (因為所有類型在 Azure 搜尋服務中都可為 null)。 如果您接著對該欄位使用 `int` 屬性不可為 Null 的模型類別，就會在嘗試擷取文件時得到類似這樣的 `JsonSerializationException`：

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

因此，我們建議您在模型類別中使用可為 Null 的類型，來做為最佳作法。

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-jsonnet"></a>使用 JSON.NET 自訂序列化
SDK 會使用 JSON.NET 序列化和還原序列化文件。 您可以視需要定義您自己的 `JsonConverter` 或 `IContractResolver` 來自訂序列化和還原序列化 (如需詳細資訊，請參閱 [JSON.NET 文件](http://www.newtonsoft.com/json/help/html/Introduction.htm))。 當您想要調整您的應用程式的現有模型類別以搭配使用 Azure 搜尋服務以及其他更進階的案例時，這非常有用。 例如，使用自訂序列化，您可以：

* 包含或排除模型類別的特定屬性儲存為文件欄位。
* 在程式碼的屬性名稱和索引的欄位名稱之間進行對應。
* 建立可用來將屬性對應至文件欄位的自訂屬性。

您可以在 GitHub 上的 Azure 搜尋服務 .NET SDK 的單元測試中找到實作自訂序列化的範例。 [這個資料夾](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Search/Search.Tests/Tests/Models)是好的起點。 它包含自訂序列化測試使用的類別。

### <a name="searching-for-documents-in-the-index"></a>搜尋索引中的文件
此範例應用程式的最後一個步驟是搜尋索引中的一些文件。 執行方法如下：

```csharp
private static void RunQueries(ISearchIndexClient indexClient)
{
    SearchParameters parameters;
    DocumentSearchResult<Hotel> results;

    Console.WriteLine("Search the entire index for the term 'budget' and return only the hotelName field:\n");

    parameters =
        new SearchParameters()
        {
            Select = new[] { "hotelName" }
        };

    results = indexClient.Documents.Search<Hotel>("budget", parameters);

    WriteDocuments(results);

    Console.Write("Apply a filter to the index to find hotels cheaper than $150 per night, ");
    Console.WriteLine("and return the hotelId and description:\n");

    parameters =
        new SearchParameters()
        {
            Filter = "baseRate lt 150",
            Select = new[] { "hotelId", "description" }
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    parameters =
        new SearchParameters()
        {
            OrderBy = new[] { "lastRenovationDate desc" },
            Select = new[] { "hotelName", "lastRenovationDate" },
            Top = 2
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.WriteLine("Search the entire index for the term 'motel':\n");

    parameters = new SearchParameters();
    results = indexClient.Documents.Search<Hotel>("motel", parameters);

    WriteDocuments(results);
}
```

每次執行查詢時，這個方法都會先建立新的 `SearchParameters` 物件。 該物件用於指定查詢的其他選項，例如排序、篩選、分頁及 Facet。 在此方法中，我們會針對不同查詢設定 `Filter`、`Select`、`OrderBy` 和 `Top` 屬性。 所有 `SearchParameters` 屬性都記載於[這裡](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.searchparameters)。

下一步是實際執行搜尋查詢， 我們透過 `Documents.Search` 方法完成此步驟。 針對每次查詢，我們會傳遞搜尋文字做為字串 (如果沒有搜尋文字，則傳遞 `"*"`)，並再加上先前建立的搜尋參數。 我們也指定了 `Hotel` 做為 `Documents.Search` 的類型參數，藉此告訴 SDK 將搜尋結果中的文件還原序列化為 `Hotel` 類型的物件。

> [!NOTE]
> 如需搜尋查詢運算式語法的詳細資訊，請參閱 [這裡](https://docs.microsoft.com/rest/api/searchservice/Simple-query-syntax-in-Azure-Search)。
> 
> 

最後，在每次查詢之後，此方法會反覆查詢搜尋結果中的所有相符項，然後將每個文件列印至主控台：

```csharp
private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.Results)
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

讓我們依序仔細查看每個查詢。 以下是執行第一個查詢的程式碼︰

```csharp
parameters =
    new SearchParameters()
    {
        Select = new[] { "hotelName" }
    };

results = indexClient.Documents.Search<Hotel>("budget", parameters);

WriteDocuments(results);
```

在此情況下，我們會搜尋符合 "budget" 這個字的旅館，而且我們只想要傳回 `Select` 參數所指定的旅館名稱。 結果如下︰

    Name: Roach Motel

接下來，我們想要尋找每晚費率小於 $150 的旅館，並且只傳回旅館識別碼和描述︰

```csharp
parameters =
    new SearchParameters()
    {
        Filter = "baseRate lt 150",
        Select = new[] { "hotelId", "description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

此查詢會使用 OData `$filter` 運算式 (`baseRate lt 150`)，以篩選索引中的文件。 您可以在 [這裡](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search)找到更多 Azure 搜尋服務支援的 OData 語法。

查詢的結果如下：

    ID: 2   Description: Cheapest hotel in town
    ID: 3   Description: Close to town hall and the river

接下來，我們要尋找最近重新裝修的前兩間旅館，並顯示旅館名稱和上次重新裝修日期。 程式碼如下： 

```csharp
parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "lastRenovationDate desc" },
        Select = new[] { "hotelName", "lastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

在此情況下，我們再次使用 OData 語法，將 `OrderBy` 參數指定為 `lastRenovationDate desc`。 我們也會將 `Top` 設定為 2，以確保只取得前兩份文件。 如同前面，我們會設定 `Select` 來指定應傳回哪些欄位。

結果如下︰

    Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
    Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00

最後，我們想要尋找符合 "motel" 這個字的所有旅館︰

```csharp
parameters = new SearchParameters();
results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

結果如下，其中包含所有欄位，因為我們並未指定 `Select` 屬性︰

    ID: 2   Base rate: 79.99        Description: Cheapest hotel in town     Description (French): Hôtel le moins cher en ville      Name: Roach Motel       Category: Budget        Tags: [motel, budget]   Parking included: yes   Smoking allowed: yes    Last renovated on: 4/28/1982 12:00:00 AM +00:00 Rating: 1/5     Location: Latitude 49.678581, longitude -122.131577

此步驟已完成本教學課程，但別就此結束。 **後續步驟** 會提供可深入了解 Azure 搜尋服務的其他資源。

## <a name="next-steps"></a>後續步驟
* 瀏覽 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search) 及 [REST API](https://docs.microsoft.com/rest/api/searchservice/) 參考文章。
* 檢閱 [命名規則](https://docs.microsoft.com/rest/api/searchservice/Naming-rules) ，了解命名各種物件的規則。
* 檢閱 Azure 搜尋服務 [支援的資料類型](https://docs.microsoft.com/rest/api/searchservice/Supported-data-types) 。
