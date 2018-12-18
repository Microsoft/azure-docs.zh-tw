---
title: 升級至 Azure 搜尋服務 .NET SDK 第 3 版 | Microsoft Docs
description: 升級至 Azure 搜尋服務 .NET SDK 版本 3
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: brjohnst
ms.openlocfilehash: 161d22e0ff4ec4ab28107919a80ecc48cd027967
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31793472"
---
# <a name="upgrading-to-the-azure-search-net-sdk-version-3"></a>升級至 Azure 搜尋服務 .NET SDK 版本 3
如果您使用版本 2.0 預覽版或更舊版本的 [Azure 搜尋服務 .NET SDK](https://aka.ms/search-sdk)，本文可協助您將應用程式升級至版本 3。

如需包括範例的 SDK 一般逐步解說，請參閱 [如何從 .NET 應用程式使用 Azure 搜尋服務](search-howto-dotnet-sdk.md)。

Azure 搜尋服務 .NET SDK 版本 3 包含一些從舊版所做的變更。 這些變更大部分是次要變更，所以變更程式碼應該只需要最少的工作。 請參閱 [升級步驟](#UpgradeSteps) 以取得如何變更您的程式碼以使用新的 SDK 版本的指示。

> [!NOTE]
> 如果您使用 1.0.2 預覽版或更舊版本，請先升級至 1.1 版，再升級至版本 3。 如需指示，請參閱[升級至 Azure 搜尋服務 .NET SDK 1.1 版](search-dotnet-sdk-migration-version-1.md)。
>
> 您的「Azure 搜尋服務」執行個體支援數個 REST API 版本，包括最新版本。 當一個版本不再是最新版本時，您仍可繼續使用該版本，但建議您將程式碼移轉成使用最新版本。 使用 REST API 時，您必須每個要求中透過 api-version 參數指定 API 版本。 使用 .NET SDK 時，您使用的 SDK 版本會決定對應的 REST API 版本。 如果您使用的是舊版 SDK，則即使服務已升級成支援新版 API 版本，您仍可繼續執行該程式碼而無須變更。

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-3"></a>版本 3 的新功能
Azure 搜尋服務 .NET SDK 版本 3 以最新推出的 Azure 搜尋服務 REST API 版本為目標，特別是 2016-09-01。 這樣可讓您從 .NET 應用程式中使用 Azure 搜尋服務的許多新功能，包括：

* [自訂分析器](https://aka.ms/customanalyzers)
* [Azure Blob 儲存體](search-howto-indexing-azure-blob-storage.md)和 [Azure 表格儲存體](search-howto-indexing-azure-tables.md)索引子支援
* 透過 [欄位對應](search-indexer-field-mappings.md)
* 可安全地並行更新索引定義、索引子和資料來源的 ETag 支援
* 支援修飾您的模型類別和使用新的 `FieldBuilder` 類別，以宣告方式建置索引欄位定義。
* 對 .NET Core 和 .NET Portable Profile 111 的支援

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>升級步驟
首先，更新 `Microsoft.Azure.Search` 的 NuGet 參考，方法是使用 NuGet 封裝管理員主控台，或是在 Visual Studio 中用滑鼠右鍵按一下您的專案參考並選取 [管理 NuGet 封裝]。

一旦 NuGet 下載新的封裝和其相依性，請重建您的專案。 根據您的程式碼結構情況，它可能會順利重新建置。 如果是這樣，代表您已準備就緒！

如果建置失敗，您應該會看到如下所示的建置錯誤：

    Program.cs(31,45,31,86): error CS0266: Cannot implicitly convert type 'Microsoft.Azure.Search.ISearchIndexClient' to 'Microsoft.Azure.Search.SearchIndexClient'. An explicit conversion exists (are you missing a cast?)

下一個步驟是修正此建置錯誤。 如需有關造成錯誤的原因及修正方式的詳細資料，請參閱[版本 3 的重大變更](#ListOfChanges)。

您可能會看到與過時的方法或屬性相關的其他建置警告。 警告將會包含要使用哪些內容取代過時功能的指示。 例如，如果您的應用程式使用 `IndexingParameters.Base64EncodeKeys` 屬性，您應該會收到指出 `"This property is obsolete. Please create a field mapping using 'FieldMapping.Base64Encode' instead."`

一旦您已修正任何建置錯誤，您就可以變更您的應用程式以視需要利用新的功能。 [版本 3 的新功能](#WhatsNew)中詳述 SDK 的新功能。

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-3"></a>版本 3 的重大變更
除了重新建置您的應用程式，版本 3 有少數幾項重大變更可能需要變更程式碼。

### <a name="indexesgetclient-return-type"></a>Indexes.GetClient 傳回類型
`Indexes.GetClient` 方法有新的傳回類型。 以往它會傳回 `SearchIndexClient`，但這在 2.0 預覽版中已變更為 `ISearchIndexClient`，該變更也延續至版本 3。 這是為了支援想要藉由傳回 `ISearchIndexClient` 的模擬實作，針對單元測試模擬 `GetClient` 方法的客戶。

#### <a name="example"></a>範例
如果您的程式碼如下所示：

```csharp
SearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

您可以將其變更如下以修正任何建置錯誤：

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

### <a name="analyzername-datatype-and-others-are-no-longer-implicitly-convertible-to-strings"></a>AnalyzerName、DataType 及其他類型已無法再隱含地轉換成字串
Azure 搜尋服務 .NET SDK 中有許多類型衍生自 `ExtensibleEnum`。 以往這些類型全都可隱含地轉換成 `string` 類型。 不過，在這些類別的 `Object.Equals` 實作中已發現錯誤，為了修正錯誤，需要停用這項隱含轉換。 仍然允許明確轉換為 `string`。

#### <a name="example"></a>範例
如果您的程式碼如下所示：

```csharp
var customTokenizerName = TokenizerName.Create("my_tokenizer"); 
var customTokenFilterName = TokenFilterName.Create("my_tokenfilter"); 
var customCharFilterName = CharFilterName.Create("my_charfilter"); 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        customTokenizerName,  
        new[] { customTokenFilterName },  
        new[] { customCharFilterName }), 
}; 
```

您可以將其變更如下以修正任何建置錯誤：

```csharp
const string CustomTokenizerName = "my_tokenizer"; 
const string CustomTokenFilterName = "my_tokenfilter"; 
const string CustomCharFilterName = "my_charfilter"; 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        CustomTokenizerName,  
        new TokenFilterName[] { CustomTokenFilterName },  
        new CharFilterName[] { CustomCharFilterName })
}; 
```

### <a name="removed-obsolete-members"></a>移除過時的成員

您可能會看到關於方法或屬性的建置錯誤，這些已在 2.0 預覽版中標示為過時，且隨後於版本 3 中移除。 如果您遇到這類錯誤，解決方法如下︰

- 如果您使用這個建構函式︰`ScoringParameter(string name, string value)`，請改用︰`ScoringParameter(string name, IEnumerable<string> values)`
- 如果您原本使用 `ScoringParameter.Value` 屬性，請改用 `ScoringParameter.Values` 屬性或`ToString` 方法。
- 如果您原本使用 `SearchRequestOptions.RequestId` 屬性，請改用 `ClientRequestId` 屬性。

### <a name="removed-preview-features"></a>已移除的預覽功能

如果您從 2.0 預覽版升級至版本 3，請注意已移除 Blob 索引子的 JSON 和 CSV 剖析支援，因為這些功能仍處於預覽階段。 具體而言，`IndexingParametersExtensions` 類別中已移除下列方法︰

- `ParseJson`
- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

如果您的應用程式緊密相依於這些功能，則您無法升級至 Azure 搜尋服務 .NET SDK 版本 3。 您可以繼續使用 2.0 預覽版。 但請記住，**我們不建議在實際執行應用程式中使用預覽 SDK**。 預覽功能僅供評估，可能會變更。

## <a name="conclusion"></a>結論
如需更多有關使用 Azure 搜尋服務 .NET SDK 的詳細資料，請參閱 [.NET 做法](search-howto-dotnet-sdk.md)。

歡迎您提供 SDK 的意見反應。 如果您遇到問題，歡迎在 [Azure 搜尋服務 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azuresearch)上尋求協助。 如果您發現錯誤，您可以在 [Azure .NET SDK GitHub 儲存機制](https://github.com/Azure/azure-sdk-for-net/issues)中提出問題。 請務必在問題的標題前面加上「[Azure 搜尋服務]」前置詞。

感謝您使用 Azure 搜尋服務！
