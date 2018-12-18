---
title: 建立索引 (.NET API - Azure 搜尋服務) | Microsoft Docs
description: 使用 Azure 搜尋服務 .NET SDK 在程式碼中建立索引。
author: brjohnstmsft
manager: jlembicz
tags: azure-portal
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/22/2017
ms.author: brjohnst
ms.openlocfilehash: 7e7d1f8110d8470fe7596633563529f397c5551e
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31794747"
---
# <a name="create-an-azure-search-index-using-the-net-sdk"></a>使用 .NET SDK 建立 Azure 搜尋服務索引
> [!div class="op_single_selector"]
> * [概觀](search-what-is-an-index.md)
> * [入口網站](search-create-index-portal.md)
> * [.NET](search-create-index-dotnet.md)
> * [REST](search-create-index-rest-api.md)
> 
> 

本文將逐步引導您完成使用 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk) 建立 Azure 搜尋服務[索引](https://docs.microsoft.com/rest/api/searchservice/Create-Index)的程序。

在按照本指南進行並建立索引錢，請先 [建立好 Azure 搜尋服務](search-create-service-portal.md)。

> [!NOTE]
> 本文中的所有範例程式碼均以 C# 撰寫。 您可以 [在 GitHub](http://aka.ms/search-dotnet-howto)找到完整的原始程式碼。 您也可以閱讀 [Azure 搜尋服務 .NET SDK](search-howto-dotnet-sdk.md)，以取得更詳細的範例程式碼逐步說明。


## <a name="identify-your-azure-search-services-admin-api-key"></a>識別 Azure 搜尋服務的系統管理 API 金鑰
既然您已佈建好 Azure 搜尋服務，您幾乎已經準備好針對使用 .NET SDK 的服務端點發出要求。 首先，必須取得一個針對您佈建的搜尋服務所產生的管理員 API 金鑰。 .NET SDK 將會在每個要求上將此 API 金鑰傳送給您的服務。 擁有有效的金鑰就能為每個要求在傳送要求之應用程式與處理要求之服務間建立信任。

1. 若要尋找服務的 API 金鑰，請登入 [Azure 入口網站](https://portal.azure.com/)
2. 前往 Azure 搜尋服務的刀鋒視窗。
3. 按一下 [金鑰] 圖示。

服務會有系統管理金鑰和查詢金鑰。

* 主要和次要系統管理金鑰  會授與所有作業的完整權限，包括管理服務以及建立和刪除索引、索引子與資料來源的能力。 由於有兩個金鑰，因此如果您決定重新產生主要金鑰，您可以繼續使用次要金鑰，反之亦然。
* 查詢金鑰  會授與索引和文件的唯讀存取權，且通常會分派給發出搜尋要求的用戶端應用程式。

主要或次要系統管理金鑰都可用於建立索引。

<a name="CreateSearchServiceClient"></a>

## <a name="create-an-instance-of-the-searchserviceclient-class"></a>建立 SearchServiceClient 類別的執行個體
若要開始使用 Azure 搜尋服務 .NET SDK，您必須建立 `SearchServiceClient` 類別的執行個體。 這個類別有數個建構函式。 您所需的建構函式會取得您的搜尋服務名稱和 `SearchCredentials` 物件作為參數。 `SearchCredentials` 會包裝您的 API 金鑰。

下列程式碼會使用搜尋服務名稱的值，以及儲存於應用程式組態檔中的 API 金鑰 (在[範例應用程式](http://aka.ms/search-dotnet-howto)的情況下為 `appsettings.json`)，來建立新的 `SearchServiceClient`：

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

`SearchServiceClient` 具有 `Indexes` 屬性。 這個屬性會提供您需要用來建立、列出、更新或刪除 Azure 搜尋服務索引的所有方法。

> [!NOTE]
> `SearchServiceClient` 類別會管理連至搜尋服務的連線。 為了避免開啟太多連線，如果可以，您應該嘗試在應用程式中共用 `SearchServiceClient` 的單一執行個體。 它的方法是可啟用這類共用的安全執行緒。
> 
> 

<a name="DefineIndex"></a>

## <a name="define-your-azure-search-index"></a>定義 Azure 搜尋服務索引
對 `Indexes.Create` 方法的單一呼叫會建立您的索引。 這個方法會將 `Index` 物件作為參數，來定義您的 Azure 搜尋服務索引。 您必須建立 `Index` 物件並將它初始化，如下所示 ︰

1. 將 `Index` 物件的 `Name` 屬性設定為索引的名稱。
2. 將 `Index` 物件的 `Fields` 屬性設定為 `Field` 物件的陣列。 建立 `Field` 物件最簡單的方式是藉由呼叫 `FieldBuilder.BuildForType` 方法，傳遞類型參數的模型類別。 模型類別具有會對應至您索引欄位的屬性。 這可讓您將來自您搜尋索引的文件繫結至模型類別的執行個體。

> [!NOTE]
> 如果您不打算使用模型類別，仍然可以藉由直接建立 `Field` 物件來定義您的索引。 您可以為建構函式提供欄位的名稱，以及資料類型 (或字串欄位的分析器)。 您也可以設定其他屬性，例如 `IsSearchable`、`IsFilterable` 等屬性。
>
>

由於必須為每個欄位指派[適當屬性](https://docs.microsoft.com/rest/api/searchservice/Create-Index)，因此在設計索引時，請務必牢記搜尋服務使用者體驗和商務需求。 這些屬性可控制要對哪些欄位套用哪些搜尋功能 (篩選、面向設定、排序全文檢索搜尋等)。 針對您未明確設定的任何屬性，除非您特別啟用，否則 `Field` 類別預設會停用對應的搜尋功能。

在我們的範例中，我們已將索引命名為 "hotels"，並使用模型類別定義欄位。 每個模型類別的屬性皆具有屬性，會判斷對應索引欄位的搜尋相關行為。 會定義模型類別，如下所示︰

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

我們已根據欄位在應用程式中的可能使用方式仔細選擇每個屬性的屬性。 例如，搜尋旅館的人很可能會對 `description` 欄位上的關鍵字相符項目感興趣，因此我們藉由將 `IsSearchable` 屬性新增至 `Description` 屬性來啟用該欄位的全文檢索搜尋。

請注意，`string` 類型的索引中必須有一個欄位 (且只有一個) 指定為 key 欄位，方法是新增 `Key`屬性 (請參閱上述範例中的 `HotelId`)。

`description_fr` 欄位會用來儲存法文文字，因此上述索引定義會對此欄位使用語言分析器。 如需語言分析器的詳細資訊，請參閱[語言支援主題](https://docs.microsoft.com/rest/api/searchservice/Language-support)以及對應的[部落格文章](https://azure.microsoft.com/blog/language-support-in-azure-search/)。

> [!NOTE]
> 根據預設，模型類別中的每個屬性名稱會用來作為索引中對應欄位的名稱。 如果您想要將所有屬性名稱對應至駝峰式欄位名稱，使用 `SerializePropertyNamesAsCamelCase` 屬性來標示類別。 如果您想要對應至不同的名稱，您可以使用 `JsonProperty` 屬性，類似上述 `DescriptionFr` 屬性。 `JsonProperty` 屬性會優先於 `SerializePropertyNamesAsCamelCase` 屬性。
> 
> 

現在，我們定義了模型類別，可以非常輕鬆地建立索引定義︰

```csharp
var definition = new Index()
{
    Name = "hotels",
    Fields = FieldBuilder.BuildForType<Hotel>()
};
```

## <a name="create-the-index"></a>建立索引
您現在已初始化 `Index` 物件，您只需在 `SearchServiceClient` 物件上呼叫 `Indexes.Create`，就能建立索引：

```csharp
serviceClient.Indexes.Create(definition);
```

若要求成功，方法將會正常傳回。 如果要求發生問題 (例如無效的參數)，方法即會擲回 `CloudException`。

索引已使用完畢而想要將其刪除時，只需在 `SearchServiceClient` 上呼叫 `Indexes.Delete` 方法。 例如，以下是我們刪除 "hotels" 索引的方式：

```csharp
serviceClient.Indexes.Delete("hotels");
```

> [!NOTE]
> 為簡單起見，本文的範例程式碼使用 Azure 搜尋服務 .NET SDK 的同步方法。 我們建議您在應用程式中使用非同步方法，讓應用程式保有可擴充性且回應靈敏。 例如，您可以在上述範例中使用 `CreateAsync` 和 `DeleteAsync`，而非 `Create` 和 `Delete`。
> 
> 

## <a name="next-steps"></a>後續步驟
建立 Azure 搜尋服務索引後，您就可以 [將內容上傳到索引](search-what-is-data-import.md) ，以便開始搜尋資料。

