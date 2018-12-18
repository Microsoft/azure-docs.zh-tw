---
title: 上傳資料 (.NET - Azure 搜尋服務) | Microsoft Docs
description: 了解如何使用 .NET SDK 將資料上傳至 Azure 搜尋服務中的索引。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 01/13/2017
ms.openlocfilehash: fb8fb153c7393af0e084cb96ee2f5e82fba5b220
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
---
# <a name="upload-data-to-azure-search-using-the-net-sdk"></a>使用 .NET SDK 將資料上傳到 Azure 搜尋服務
> [!div class="op_single_selector"]
> * [概觀](search-what-is-data-import.md)
> * [.NET](search-import-data-dotnet.md)
> * [REST](search-import-data-rest-api.md)
> 
> 

本文將說明如何使用 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk) 將資料匯入 Azure 搜尋服務索引。

在開始閱讀本逐步解說前，請先 [建立好 Azure 搜尋服務索引](search-what-is-an-index.md)。 本文也假設您已建立 `SearchServiceClient` 物件，如 [使用 .NET SDK 建立 Azure 搜尋服務索引](search-create-index-dotnet.md#CreateSearchServiceClient)中所示。

> [!NOTE]
> 本文中的所有範例程式碼均以 C# 撰寫。 您可以 [在 GitHub](http://aka.ms/search-dotnet-howto)找到完整的原始程式碼。 您也可以閱讀 [Azure 搜尋服務 .NET SDK](search-howto-dotnet-sdk.md)，以取得更詳細的範例程式碼逐步說明。

若要使用 .NET SDK 將文件發送到您的索引中，您必須：

1. 建立 `SearchIndexClient` 物件以連接到您的搜尋索引。
2. 建立 `IndexBatch`，其中包含要新增、修改或刪除的文件。
3. 呼叫 `SearchIndexClient` 的 `Documents.Index` 方法以將 `IndexBatch` 傳送到您的搜尋索引。

## <a name="create-an-instance-of-the-searchindexclient-class"></a>建立 SearchIndexClient 類別的執行個體
若要使用 Azure 搜尋服務 .NET SDK 將資料匯入索引，您必須建立 `SearchIndexClient` 類別的執行個體。 您可以自己建構此執行個體，但如果您已經有 `SearchServiceClient` 執行個體可呼叫其 `Indexes.GetClient` 方法，會更為輕鬆。 例如，以下是您可從名為 `serviceClient` 的 `SearchServiceClient` 為索引 "hotels" 取得 `SearchIndexClient` 的方法：

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

> [!NOTE]
> 在一般搜尋應用程式中，索引的管理和填入是由搜尋查詢的個別元件所處理。 `Indexes.GetClient` 可以很輕易填入索引，因為它可以節省提供其他 `SearchCredentials` 的麻煩。 執行方法是將用於建立 `SearchServiceClient` 的系統管理金鑰傳遞至新的 `SearchIndexClient`。 但在執行查詢的應用程式中，最好直接建立 `SearchIndexClient` ，如此一來就可以傳遞查詢金鑰，而非系統管理金鑰。 這不僅符合 [最低權限原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege) ，也可以讓您的應用程式更安全。 您可以在 [Azure 搜尋服務 REST API 參考](https://docs.microsoft.com/rest/api/searchservice/)找到系統管理金鑰及查詢金鑰的詳細資訊。
> 
> 

`SearchIndexClient` 具有 `Documents` 屬性。 此屬性提供您新增、修改、刪除或查詢索引中文件所需的所有方法。

## <a name="decide-which-indexing-action-to-use"></a>決定要使用的索引編製動作
若要使用 .NET SDK 匯入資料，您必須將資料封裝到 `IndexBatch` 物件中。 `IndexBatch` 會封裝 `IndexAction` 物件集合，每個物件各包含一份文件和一個屬性，後者會告知 Azure 搜尋服務要對該文件執行什麼動作 (上傳、合併、刪除等)。 依據您在以下動作中所做的選擇，每個文件內只需包含某些欄位：

| 動作 | 說明 | 每個文件的必要欄位 | 注意 |
| --- | --- | --- | --- |
| `Upload` |`Upload` 動作類似「upsert」，如果是新文件，就會插入該文件，如果文件已經存在，就會更新/取代它。 |索引鍵以及其他任何您想要定義的欄位 |在更新/取代現有文件時，要求中未指定的欄位會將其欄位設定為 `null`。 即使先前已將欄位設定為非 null 值也是一樣。 |
| `Merge` |使用指定的欄位更新現有文件。 如果文件不存在於索引中，合併就會失敗。 |索引鍵以及其他任何您想要定義的欄位 |您在合併中指定的任何欄位將取代文件中現有的欄位。 這包括類型 `DataType.Collection(DataType.String)`的欄位。 例如，如果文件包含欄位 `tags` 且值為 `["budget"]`，而您使用值 `["economy", "pool"]` 針對 `tags` 執行合併，則 `tags` 欄位最後的值會是 `["economy", "pool"]`。 而不會是 `["budget", "economy", "pool"]`。 |
| `MergeOrUpload` |如果含有指定索引鍵的文件已經存在於索引中，則此動作的行為會類似 `Merge`。 如果文件不存在，其行為會類似新文件的 `Upload` 。 |索引鍵以及其他任何您想要定義的欄位 |- |
| `Delete` |從索引中移除指定的文件。 |僅索引鍵 |您指定的所有欄位 (索引鍵欄位除外) 都將被忽略。 如果您想要從文件中移除個別欄位，請改用 `Merge` ，而且只需明確地將該欄位設為 null。 |

您可以指定要搭配 `IndexBatch` 不同靜態方法及 `IndexAction` 類別使用的動作，如下一節所示。

## <a name="construct-your-indexbatch"></a>建構您的 IndexBatch
您現在已經知道可對文件執行哪些動作，並準備好建構 `IndexBatch`。 下方範例示範如何使用不同動作建立批次。 請注意，我們的範例使用稱為 `Hotel` 的自訂類別，其對應到 "hotels" 索引中的文件。

```csharp
var actions =
    new IndexAction<Hotel>[]
    {
        IndexAction.Upload(
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
            }),
        IndexAction.Upload(
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
            }),
        IndexAction.MergeOrUpload(
            new Hotel()
            {
                HotelId = "3",
                BaseRate = 129.99,
                Description = "Close to town hall and the river"
            }),
        IndexAction.Delete(new Hotel() { HotelId = "6" })
    };

var batch = IndexBatch.New(actions);
```

在此案例中，我們使用了 `Upload`、`MergeOrUpload`和 `Delete` 作為搜尋動作，如同對 `IndexAction` 類別呼叫的方法所指定。

假設此 "hotels" 索引範例已填入幾份文件。 請留意我們在使用 `MergeOrUpload` 時是如何不必指定所有可能的文件欄位，以及在使用 `Delete` 時如何僅指定文件索引鍵 (`HotelId`)。

另請注意，每個索引編製要求中最多只能包含 1000 份文件。

> [!NOTE]
> 在此範例中，我們在不同文件中套用了不同動作。 如果您要在批次中所有文件執行相同動作，而不要呼叫 `IndexBatch.New`，則可以使用 `IndexBatch` 的其他靜態方法。 例如，您可以藉由呼叫 `IndexBatch.Merge`、`IndexBatch.MergeOrUpload` 或 `IndexBatch.Delete` 來建立批次。 這些方法會採用文件 (在此範例中為類型 `Hotel` 的物件) 集合，而非 `IndexAction` 物件。
> 
> 

## <a name="import-data-to-the-index"></a>將資料匯入索引
您現在已將 `IndexBatch` 物件初始化，便可在 `SearchIndexClient` 物件上呼叫 `Documents.Index`，將其傳送到索引。 下列範例示範如何呼叫 `Index`，以及您必須執行的幾個額外步驟：

```csharp
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
```

請留意對 `Index` 方法的呼叫前後的 `try`/`catch`。 Catch 區塊會處理索引編製的重大錯誤案例。 如果您的 Azure Search 服務無法將 Batch 中的一些文件編制索引，則 `Documents.Index` 會擲回 `IndexBatchException`。 如果您在服務負載過重時編制文件的索引，就會發生此情況。 **我們強烈建議您在程式碼中明確處理此情況。** 您可以延遲，然後重新嘗試將失敗的文件編制索引，或像範例一樣加以記錄並繼續，或是根據您應用程式的資料一致性需求執行其他操作。

最後，上方範例中的程式碼會延遲兩秒。 您的 Azure 搜尋服務中會發生非同步索引編製，因此範例應用程式必須稍待一會，才能確定文件已準備好可供搜尋。 通常只有在示範、測試和範例應用程式中，才需要這類延遲。

<a name="HotelClass"></a>

### <a name="how-the-net-sdk-handles-documents"></a>.NET SDK 如何處理文件
您可能想知道 Azure 搜尋服務 .NET SDK 如何能夠將使用者定義的類別執行個體 (例如 `Hotel` 上傳至索引。 若要協助回答該問題，請查看 `Hotel` 類別，其對應到 [使用 .NET SDK 建立 Azure 搜尋服務索引](search-create-index-dotnet.md#DefineIndex)中定義的索引結構描述：

```csharp
[SerializePropertyNamesAsCamelCase]
public partial class Hotel
{
    [Key]
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

    // ToString() method omitted for brevity...
}
```

首先要注意的是，每個 `Hotel` 的公用屬性會對應索引定義中的欄位，但這之中有一項關鍵的差異：每個欄位的名稱會以小寫字母 (「駝峰式命名法」) 為開頭，而每個 `Hotel` 的公用屬性名稱會以大小字母 (「巴斯卡命名法」) 為開頭。 這在執行資料繫結、而目標結構描述在應用程式開發人員控制範圍之外的 .NET 應用程式中很常見。 與其違反 .NET 命名方針，使屬性名稱為駝峰式命名法，您可以改用 `[SerializePropertyNamesAsCamelCase]` 屬性，告訴 SDK 自動將屬性名稱對應至駝峰式命名法。

> [!NOTE]
> Azure 搜尋服務 .NET SDK 使用 [NewtonSoft JSON.NET](http://www.newtonsoft.com/json/help/html/Introduction.htm) 程式庫來將您的自訂模型物件序列化到 JSON 中，以及將您在 JSON 中的自訂模型物件還原序列化。 如有需要，您可以自訂這個序列化的過程。 您可以在[使用 JSON.NET 自訂序列化](search-howto-dotnet-sdk.md#JsonDotNet)中找到詳細資訊。 其中一個範例就是上方範例程式碼中在 `DescriptionFr` 屬性使用 `[JsonProperty]` 屬性的方式。
> 
> 

第二個要注意的是，`Hotel` 類別為公用屬性的資料類型。 這些屬性的 .NET 類型會對應至索引定義中，與其相當的欄位類型。 例如，`Category` 字串屬性會對應至 `category` 欄位 (此欄位屬於 `DataType.String` 類型)。 `bool?` 與 `DataType.Boolean`、`DateTimeOffset?` 與 `DataType.DateTimeOffset` 等之間也有類似的類型對應。 類型對應的特定規則和 `Documents.Get` 方法已一起記載於 [Azure 搜尋服務 .NET SDK 參考](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations#Microsoft_Azure_Search_IDocumentsOperations_GetWithHttpMessagesAsync__1_System_String_System_Collections_Generic_IEnumerable_System_String__Microsoft_Azure_Search_Models_SearchRequestOptions_System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_)。

將您自己的類別用作文件的功能雙向均適用；您也可以擷取搜尋結果，然後讓 SDK 將結果自動還原序列化為您選擇的類型，如 [下一篇文章](search-query-dotnet.md)所示。

> [!NOTE]
> Azure 搜尋服務 .NET SDK 還支援使用 `Document` 類別的動態類型文件，也就是欄位名稱與欄位值的索引鍵/值對應。 當您在設計階段卻不知道索引的結構描述時，這很實用，否則要繫結到特定模型類別會很麻煩。 SDK 中所有處理文件的方法，都有可搭配 `Document` 類別使用的多載，以及使用泛型類型參數的強類型多載。 本文的範例程式碼只使用了後者。
> 
> 

**為什麼您應該使用 Nullable 資料類型**

當您將自己的模型類別設計為可對應至 Azure 搜尋服務索引時，建議您將 `bool` 和 `int` 等的值類型屬性宣告為可為 Null (例如宣告為 `bool?`，而不是 `bool`)。 如果您使用不可為 Null 的屬性，則必須保證  索引中沒有任何文件的對應欄位包含 Null 值。 SDK 和 Azure 搜尋服務都不會協助您強制執行這項規定。

這不只是假設性的問題：如果您在類型為 `DataType.Int32` 的現有索引中新增欄位， 更新索引定義之後，所有文件對於該新的欄位具有 null 值 (因為所有類型在 Azure 搜尋服務中都可為 null)。 如果您接著對該欄位使用 `int` 屬性不可為 Null 的模型類別，就會在嘗試擷取文件時得到類似這樣的 `JsonSerializationException`：

    Error converting value {null} to type 'System.Int32'. Path 'IntValue'.

因此，我們建議您在模型類別中使用可為 Null 的類型，來做為最佳作法。

## <a name="next-steps"></a>後續步驟
在填入 Azure 搜尋服務索引後，您就可以開始發出查詢來搜尋文件。 如需詳細資料，請參閱 [查詢 Azure 搜尋服務索引](search-query-overview.md) 。

