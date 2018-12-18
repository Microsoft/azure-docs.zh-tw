---
title: Azure 搜尋服務中的篩選條件 | Microsoft Docs
description: 依使用者安全性身分識別、語言、地理位置或數值進行篩選，以縮減在 Azure 搜尋服務 (Microsoft Azure 上裝載的雲端搜尋服務) 中查詢的搜尋結果。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: heidist
ms.openlocfilehash: 9f891dbe3f051f2fb5bfd242830f3c30abede487
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32191361"
---
# <a name="filters-in-azure-search"></a>Azure 搜尋服務中的篩選條件 

「篩選條件」提供準則來選取 Azure 搜尋服務查詢中所使用的文件。 未篩選的搜尋會包含索引中的所有文件。 篩選條件會將搜尋查詢的範圍設定為文件子集。 例如，篩選條件可以限制只對具有特定品牌或色彩且超過特定閾值之價格點的產品進行全文檢索搜尋。

某些搜尋體驗會強制實行篩選條件需求以作為實作的一部分，但是，當您想要使用「以值為基礎」的準則 (針對 "Simon & Schuster" 所發行的類別 "non-fiction"，將搜尋範圍設定為產品類型 "books") 來限制搜尋時，隨時都可使用篩選條件。

反之，如果您的目標是在特定資料「結構」上進行搜尋 (將搜尋範圍設定為客戶評論欄位)，則有替代方法，如下所述。

## <a name="when-to-use-a-filter"></a>使用篩選條件的時機

篩選條件是數個搜尋體驗的基礎，包括 "find near me"、多面向導覽和安全性篩選，以便只顯示允許使用者查看的文件。 如果您實作這其中任一個體驗，則篩選條件是必要的。 它是搜尋查詢附加的篩選條件，這類查詢可提供地理位置座標、使用者所選取的 Facet 類別或要求者的安全性識別碼。

範例案例包含下列項目：

1. 使用篩選條件，根據索引中的資料值來為索引配量。 假設有一個具有城市、房屋類型及便利設施的結構描述，您可能會建立一個篩選條件，明確選取滿足您準則 (位於西雅圖、公寓、濱水區) 的文件。 

  具備相同輸入的全文檢索搜尋通常會產生類似的結果，但篩選條件更為精確，因為它需要針對索引中的內容取得與篩選條件完全相符的項目。 

2. 如果搜尋體驗伴隨著篩選條件需求，請使用篩選條件：

 * [多面向導覽](search-faceted-navigation.md)會使用篩選條件，傳回使用者所選取的 Facet 類別。
 * 地理搜尋會使用篩選條件，在 "find near me" 應用程式中傳遞目前位置的座標。 
 * 安全性篩選條件會傳遞安全性識別碼作為篩選條件準則，索引中的相符項目則會作為用以存取文件權限的 Proxy。

3. 如果您想要數值欄位上的搜尋準則，請使用篩選條件。 

  數值欄位在文件中是可擷取且可出現在搜尋結果中，但不可單獨搜尋它們 (受限於全文檢索搜尋)。 如果您需要以數值資料為基礎的選取準則，請使用篩選條件。

### <a name="alternative-methods-for-reducing-scope"></a>縮減範圍的替代方法

如果您希望在搜尋結果中產生縮減效果，則篩選條件並非唯一的選擇。 視您的目標而定，這些替代選項可能更適合：

 + `searchFields` 查詢參數會將搜尋限制為特定欄位。 例如，如果您的索引分別提供適用於英文和西班牙文描述的欄位，您可以使用 searchFields，將目標設定為要用來進行全文檢索搜尋的欄位。 

+ `$select` 參數可用來指定要包含在結果集中的欄位，先有效地調整回應，然後將它傳送至呼叫應用程式。 這個參數不會精簡查詢或縮減文件集合，但是，如果您的目標是細微的回應，則這個參數是要考慮的選項。 

如需其中任一個參數的詳細資訊，請參閱[搜尋文件 > 要求 > 查詢參數](https://docs.microsoft.com/rest/api/searchservice/search-documents#request) \(英文\)。


## <a name="filters-in-the-query-pipeline"></a>查詢管線中的篩選條件

查詢時，篩選條件剖析器會接受準則作為輸入、將運算式轉換為不可部分完成的布林運算式，並建置篩選條件樹狀結構，然後在索引中針對可篩選欄位進行評估。  

篩選會在搜尋之前發生，限定下游處理程序中要包含哪些文件來進行文件擷取及相關的評分。 與搜尋字串配對時，篩選條件會有效地縮減後續搜尋作業的介面區。 單獨使用時 (例如，當查詢字串空白時，其中 `search=*`)，篩選條件準則會是唯一的輸入。 

## <a name="filter-definition"></a>篩選條件定義

篩選條件是 OData 運算式，會使用 [Azure 搜尋服務中支援的 OData V4 語法子集](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) \(英文\) 來聯結。 

您可以針對每個**搜尋**作業指定一個篩選條件，但篩選條件本身可以包含多個欄位、多個條件，而且如果您使用 **ismatch** 函式，則可包含多個運算式。 在多部分的篩選條件運算式中，您能以任意順序指定述詞。 如果您嘗試以特定順序重新整理述詞，則不會有任何顯著的效能改善。

篩選條件運算式的硬限制是要求上的最大限制。 整個要求 (包含篩選條件) 針對 POST 最多可有 16 MB，針對 GET 則是 8 KB。 軟限制與篩選條件運算式中的子句數目相互關聯。 最佳經驗法則是，如果您有數百個子句，則必須承擔達到限制的風險。 我們建議以這類不會產生無大小限制之篩選條件的方式來設計您的應用程式。

下列範例會呈現數個 API 中的典型篩選條件定義。

```http
# Option 1:  Use $filter for GET
GET https://[service name].search.windows.net/indexes/hotels/docs?search=*&$filter=baseRate lt 150&$select=hotelId,description&api-version=2017-11-11

# Option 2: Use filter for POST and pass it in the header
POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2017-11-11
{
    "search": "*",
    "filter": "baseRate lt 150",
    "select": "hotelId,description"
}
```

```csharp
    parameters =
        new SearchParameters()
        {
            Filter = "baseRate lt 150",
            Select = new[] { "hotelId", "description" }
        };

```

## <a name="filter-design-patterns"></a>篩選條件設計模式

下列範例說明適用於篩選條件案例的數個設計模式。 如需更多概念，請參閱 [OData 運算式語法 > 範例](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#bkmk_examples) \(英文\) 。

+ 獨立的 **$filter** (不含查詢字串)，在篩選條件運算式能夠完全限定為感興趣的文件時很有用。 沒有查詢字串，就沒有語彙或語言分析、沒有計分且沒有排名。 請注意，搜尋字串是空的。

   ```
   search=*&$filter=(baseRate ge 60 and baseRate lt 300) and accommodation eq 'Hotel' and city eq 'Nogales'
   ```

+ 查詢字串和 **$filter** 的組合，其中的篩選條件會建立子集，而查詢字串會提供字詞輸入，以便在篩選的子集上進行全文檢索搜尋。 搭配查詢字串使用篩選條件是最常見的程式碼模式。

   ```
   search=hotels ocean$filter=(baseRate ge 60 and baseRate lt 300) and city eq 'Los Angeles'
   ```

+ 複合式查詢 (以 "or" 分隔)，每個均有自己的篩選條件準則 (例如，'dog' 中的 'beagles' 或 'cat' 中的 'siamese')。 OR 運算式會個別評估，並將來自每一個的回應組合成一個回應後傳回給呼叫應用程式。 此設計模式會透過 search.ismatch 函式來達成。 您可以使用非計分版本 (search.ismatch) 或計分版本 (search.ismatchscoring)。

   ```
   # Match on hostels rated higher than 4 OR 5-star motels.
   $filter=search.ismatchscoring('hostel') and rating ge 4 or search.ismatchscoring('motel') and rating eq 5

   # Match on 'luxury' or 'high-end' in the description field OR on category exactly equal to 'Luxury'.
   $filter=search.ismatchscoring('luxury | high-end', 'description') or category eq 'Luxury'
   ```

繼續閱讀下列文章，以取得特定使用案例的完整指導方針：

+ [Facet 篩選條件](search-filters-facets.md)
+ [語言篩選條件](search-filters-language.md)
+ [安全性調整](search-security-trimming-for-azure-search.md) 

## <a name="field-requirements-for-filtering"></a>進行篩選的欄位需求

在 REST API 中，可篩選預設為 *on*。 可篩選的欄位會增加索引大小；請務必針對您不打算在篩選條件中實際使用的欄位設定 `filterable=FALSE`。 如需適用於欄位定義之設定的詳細資訊，請參閱[建立索引](https://docs.microsoft.com/rest/api/searchservice/create-index) \(英文\)。

在 .NET SDK 中，可篩選預設為 *off*。 用以設定可篩選項屬性的 API 是 [IsFilterable](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.isfilterableattribute)。 在下列範例中，它是在 BaseRate 欄位定義上所設定。

```csharp
    [IsFilterable, IsSortable, IsFacetable]
    public double? BaseRate { get; set; }
```

### <a name="reindexing-requirements"></a>重新編製索引需求

如果欄位為不可篩選的欄位，而您想要將它設定為可篩選，則必須新增一個欄位，或重建現有的欄位。 變更欄位定義會改變索引的實體結構。 在 Azure 搜尋服務中，所有允許的存取路徑均已編製索引以加快查詢速度，其會在欄位定義變更時，強制執行資料結構的重建。 

重建個別欄位可以是影響很小的作業，只需一個合併作業來將現有的文件索引鍵和相關聯的值傳送至索引，並原封不動地留下每份文件的其餘部分。 如果您遇到重建需求，請參閱下列連結以取得指示：

 + [使用 .NET SDK 的索引編製動作](https://docs.microsoft.com/azure/search/search-import-data-dotnet#decide-which-indexing-action-to-use)
 + [使用 REST API 的索引編製動作](https://docs.microsoft.com/azure/search/search-import-data-rest-api#decide-which-indexing-action-to-use)

## <a name="text-filter-fundamentals"></a>文字篩選條件基本概念

文字篩選條件適用於字串欄位，您會想要根據搜尋主體內的值，從其中提取一些任意的文件集合。

針對由字串組成的文字篩選條件，不會有任何語彙分析或斷詞，因此，比較僅限於完全相符的項目。 例如，假設欄位 *f* 包含 "sunny day"，則 `$filter=f eq 'Sunny'` 不相符，但 `$filter=f eq 'Sunny day'` 將會符合。 

文字字串會區分大小寫。 沒有大寫字的小寫寫法：`$filter=f eq 'Sunny day'` 將找不到 'sunny day'`。


| 方法 | 說明 | 
|----------|-------------|
| [search.in()](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) | 針對指定的欄位提供以逗號分隔之字串清單的函式。 字串由篩選條件準則所組成，其會套用至查詢範圍中的每個欄位。 <br/><br/>`search.in(f, ‘a, b, c’)` 語意上相當於 `f eq ‘a’ or f eq ‘b’ or f eq ‘c’`，只不過它在值清單太大時執行的速度更快。<br/><br/>我們建議針對[安全性篩選條件](search-security-trimming-for-azure-search.md)使用 **search.in** 函式，以及針對由要符合指定欄位之值的未經處理文字所組成的任何篩選條件使用該函式。 這個方法是基於速度而設計的。 傳回數百甚至數千個值只需要不到一秒的時間。 雖然對於您可以傳遞給函式的項目數並無明確的限制，但延遲會根據您提供的字串數目按比例增加。 | 
| [search.ismatch()](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) | 此函式可讓您在相同的篩選條件運算式中，混用全文檢索搜尋作業以及純布林值篩選作業。 它可以讓您在一個要求中使用多個查詢篩選條件組合。 您也可以將它用於 *contains* 篩選條件，以篩選較大字串內的部分字串。 |  
| [$filter=field operator string](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) | 使用者定義的運算式，由欄位、運算子和值所組成。 | 

## <a name="numeric-filter-fundamentals"></a>數值篩選條件基本概念

數值欄位在全文檢索搜尋的內容中不是 `searchable`。 只有字串才能進行全文檢索搜尋。 例如，如果您輸入 99.99 作為搜尋字詞，則將不會得到價格為 $ 99.99 美元的項目。 相反地，您會看到在文件的字串欄位中有數字 99 的項目。 因此，如果您有數值資料，則很可能會用於篩選條件，包括範圍、Facet、群組等。 

包含數值欄位 (價格、大小、SKU、識別碼) 的文件會在搜尋結果中提供那些值，但前提是已將該欄位標記為 `retrievable`。 此處的重點是全文檢索搜尋本身不適用於數值欄位類型。

## <a name="next-steps"></a>後續步驟

首先，嘗試使用入口網站中的 [搜尋總管]，提交含有 **$filter** 參數的查詢。 [不動產範例索引](search-get-started-portal.md)會在您將下列篩選查詢貼至搜尋列時提供它們的有趣結果：

```
# Geo-filter returning documents within 5 kilometers of Redmond, Washington state
# Use $count=true to get a number of hits returned by the query
# Use $select to trim results, showing values for named fields only
# Use search=* for an empty query string. The filter is the sole input

search=*&$count=true&$select=description,city,postCode&$filter=geo.distance(location,geography'POINT(-122.121513 47.673988)') le 5

# Numeric filters use comparison like greater than (gt), less than (lt), not equal (ne)
# Include "and" to filter on multiple fields (baths and bed)
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=baths gt 3 and beds gt 4

# Text filters can also use comparison operators
# Wrap text in single or double quotes and use the correct case
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=city gt 'Seattle'
```

若要使用更多範例，請參閱 [OData 篩選條件運算式語法 > 範例](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#bkmk_examples) \(英文\)。

## <a name="see-also"></a>另請參閱

+ [全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md)
+ [搜尋文件 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents)
+ [簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [支援的資料類型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)
