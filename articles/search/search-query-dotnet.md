---
title: 查詢索引 (.NET API - Azure 搜尋服務) | Microsoft Docs
description: 在 Azure 搜尋服務中建立搜尋查詢，並使用搜尋參數來篩選及排序搜尋結果。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/19/2017
ms.openlocfilehash: 83f7f29471d49c4dbbf1630b4e8e9006144638cc
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31797781"
---
# <a name="query-your-azure-search-index-using-the-net-sdk"></a>使用 .NET SDK 查詢 Azure 搜尋服務索引
> [!div class="op_single_selector"]
> * [概觀](search-query-overview.md)
> * [入口網站](search-explorer.md)
> * [.NET](search-query-dotnet.md)
> * [REST](search-query-rest-api.md)
> 
> 

本文將說明如何使用 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk)查詢索引。

在開始閱讀本逐步解說前，請先[建立好 Azure 搜尋服務索引](search-what-is-an-index.md)，並[在索引中填入資料](search-what-is-data-import.md)。

> [!NOTE]
> 本文中的所有範例程式碼均以 C# 撰寫。 您可以 [在 GitHub](http://aka.ms/search-dotnet-howto)找到完整的原始程式碼。 您也可以閱讀 [Azure 搜尋服務 .NET SDK](search-howto-dotnet-sdk.md)，以取得更詳細的範例程式碼逐步說明。

## <a name="identify-your-azure-search-services-query-api-key"></a>識別 Azure 搜尋服務的查詢 API 金鑰
現在您已建立 Azure 搜尋服務索引，便差不多可以使用 .NET SDK 發出查詢。 首先，必須取得一個為您佈建的搜尋服務所產生的查詢 API 金鑰。 .NET SDK 將會在每個要求上將此 API 金鑰傳送給您的服務。 擁有有效的金鑰就能為每個要求在傳送要求之應用程式與處理要求之服務間建立信任。

1. 若要尋找服務的 API 金鑰，您可以登入 [Azure 入口網站](https://portal.azure.com/)
2. 前往 Azure 搜尋服務的刀鋒視窗。
3. 按一下 [金鑰] 圖示。

服務會有系統管理金鑰和查詢金鑰。

* 主要和次要系統管理金鑰  會授與所有作業的完整權限，包括管理服務以及建立和刪除索引、索引子與資料來源的能力。 由於有兩個金鑰，因此如果您決定重新產生主要金鑰，您可以繼續使用次要金鑰，反之亦然。
* 查詢金鑰  會授與索引和文件的唯讀存取權，且通常會分派給發出搜尋要求的用戶端應用程式。

若要查詢索引，您可以使用其中一個查詢金鑰。 系統管理金鑰也可以用於進行查詢，但是您應該在應用程式的程式碼中使用查詢金鑰，因為查詢金鑰更加符合 [最低權限準則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)。

## <a name="create-an-instance-of-the-searchindexclient-class"></a>建立 SearchIndexClient 類別的執行個體
若要使用 Azure 搜尋服務 .NET SDK 來發出查詢，您必須建立 `SearchIndexClient` 類別的執行個體。 這個類別有數個建構函式。 您需要的建構函式會取得您的搜尋服務名稱和 `SearchCredentials` 物件作為參數。 `SearchCredentials` 會包裝您的 API 金鑰。

下方程式碼會使用搜尋服務名稱的值，以及儲存於應用程式設定檔中的 API 金鑰 (在[範例應用程式](http://aka.ms/search-dotnet-howto)的情況下為 `appsettings.json`)，為 "hotels" 索引 (建立於[使用 .NET SDK 建立 Azure 搜尋服務索引](search-create-index-dotnet.md)) 建立新的 `SearchIndexClient`：

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "hotels", new SearchCredentials(queryApiKey));
    return indexClient;
}
```

`SearchIndexClient` 具有 `Documents` 屬性。 此屬性會提供您查詢 Azure 搜尋服務索引所需的所有方法。

## <a name="query-your-index"></a>查詢您的索引
使用 .NET SDK 進行搜尋就和在您的 `SearchIndexClient` 呼叫 `Documents.Search` 方法一樣簡單。 此方法會採用一些參數，包括搜尋文字，以及可進一步縮小查詢範圍的 `SearchParameters` 物件。

#### <a name="types-of-queries"></a>查詢類型
您將會使用的兩個主要[查詢類型](search-query-overview.md#types-of-queries)是 `search` 和 `filter`。 `search` 查詢會搜尋索引中所有可搜尋欄位的一或多個字詞。 `filter` 查詢可跨索引的所有可篩選欄位評估布林運算式。 您可以同時或個別使用搜尋和篩選。

搜尋和篩選均使用 `Documents.Search` 方法執行。 搜尋查詢可在 `searchText` 參數中傳遞，而篩選運算式可在 `SearchParameters` 類別的 `Filter` 屬性中傳遞。 若要篩選而不進行搜尋，只要為 `searchText` 參數傳遞 `"*"` 即可。 若要在不進行篩選的情況下搜尋，則只要將 `Filter` 屬性保留在未設定狀態，或完全不要傳入 `SearchParameters` 執行個體。

#### <a name="example-queries"></a>查詢範例
下列範例程式碼示範幾個不同方式，來查詢 [使用.NET SDK 建立 Azure 搜尋服務索引](search-create-index-dotnet.md#DefineIndex)中定義的 "hotels" 索引。 請注意，隨著搜尋結果傳回的文件是 `Hotel` 類別的執行個體，其定義於 [在 Azure 搜尋服務中使用 .NET SDK 匯入資料](search-import-data-dotnet.md#HotelClass)。 範例程式碼運用 `WriteDocuments` 方法將搜尋結果輸出到主控台。 下一節將說明此方法。

```csharp
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
```

## <a name="handle-search-results"></a>處理搜尋結果
`Documents.Search` 方法會傳回包含查詢結果的 `DocumentSearchResult` 物件。 前一章節中的範例使用稱為 `WriteDocuments` 的方法將搜尋結果輸出到主控台：

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

以下是前一節查詢結果的可能樣貌，此處假設 "hotels" 索引是以 [在 Azure 搜尋服務中使用 .NET SDK 匯入資料](search-import-data-dotnet.md)中的範例資料填入：

```
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
```

上方的範例程式碼使用主控台來輸出搜尋結果。 您同樣需要在自己的應用程式中顯示搜尋結果。 如需範例以了解如何在 ASP.NET MVC 架構的 Web 應用程式中轉譯搜尋結果，請參閱 [GitHub 上的此範例](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetSample) 。

