---
title: Azure 搜尋服務中的查詢類型和組合 | Microsoft Docs
description: 在 Azure 搜尋服務中建置搜尋查詢，並使用參數來篩選、選取及排序結果的基本概念。
author: HeidiSteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.topic: conceptual
ms.date: 08/03/2018
ms.openlocfilehash: a1dad30148da9f6b322c75fd40dc01098c4d6b63
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42139800"
---
# <a name="query-types-and-composition-in-azure-search"></a>Azure 搜尋服務中的查詢類型和組合

在 Azure 搜尋服務中，查詢是往返作業的完整規格。 參數會提供比對準則 (用於尋找索引中文件)、引擎的執行指示和用於形成回應的指示詞。 更精確地說，您可以指定哪些欄位在搜尋範圍內、搜尋方式、要傳回哪些欄位，以及是否要排序或篩選等等。 未指定時，將會以全文檢索搜尋作業的形式對所有可搜尋的欄位執行查詢，並以任意順序傳回未計分的結果集。

## <a name="a-first-look-at-query-requests"></a>初窺查詢要求

範例適合用來介紹新的概念。 作為 [REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 中所建構的代表性查詢，下列範例鎖定[不動產示範索引](search-get-started-portal.md)，並包含常見參數。

```
{  
    "queryType": "simple" 
    "search": "seattle townhouse* +\"lake\"", 
    "searchFields": "description, city",  
    "count": "true", 
    "select": "listingId, street, status, daysOnMarket, description",
    "top": "10",
    "orderby": "daysOnMarket"
 } 
```

+ **`queryType`** 會設定剖析器，在 Azure 搜尋服務中，這既可以是[預設的簡單查詢剖析器](search-query-simple-examples.md) (最適合全文檢索搜尋)，也可以是[完整的 Lucene 查詢剖析器](search-query-lucene-examples.md) (用於進階的查詢建構，例如規則運算式、鄰近搜尋、模糊和萬用字元搜尋等等)。

+ **`search`** 會提供比對準則，這通常是文字，但往往會伴隨布林運算子。 單一的獨立字詞是「字詞」查詢。 用引號括住、有多個部分的查詢則是「關鍵片語」查詢。 搜尋可能是未定義的 (如 **`search=*`** 中)，但更可能會包含字詞、片語和運算子，如下列範例所示。

+ **`searchFields`** 是選擇性的，可用來將查詢執行限制在特定欄位。

回應也會由查詢中所包含的參數來形成。 在範例中，結果集包含 **`select`** 陳述式中所列的欄位。 此查詢只會傳回前 10 個相符項目，但 **`count`** 會告訴您總共有多少份文件相符。 在此查詢中，資料列會依 daysOnMarket 來排序。

在 Azure 搜尋服務中，查詢一律會針對單一索引來執行，並使用要求中提供 API 金鑰進行驗證。 在 REST 中，則會於要求標頭中同時提供這兩者。

### <a name="how-to-run-this-query"></a>如何執行這項查詢

若要執行這項查詢，請使用[搜尋總管和不動產示範索引](search-get-started-portal.md)。 

您可以將這個查詢字串貼到總管中的搜尋列：`search=seattle townhouse +lake&searchFields=description, city&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket`

### <a name="how-query-operations-are-enabled-by-the-index"></a>索引如何啟用查詢作業

Azure 搜尋服務緊密結合了索引設計和查詢設計。 事先要知道的重要事實是，「索引結構描述」與每個欄位上的屬性會決定您可以建置的查詢類型。 

欄位上的索引屬性會設定允許的作業 - 欄位在索引中是否「可搜尋」、在結果中是否「可擷取」、是否「可排序」、是否「可篩選」，不一而足。 在查詢字串範例中，只有當 daysOnMarket 欄位在索引結構描述中標記為「可排序」時，`"$orderby": "daysOnMarket"` 才會有作用。 

![不動產範例的索引定義](./media/search-query-overview/realestate-sample-index-definition.png "不動產範例的索引定義")

上述螢幕擷取畫面列出了一部分不動產範例的索引屬性。 您可以在入口網站中檢視整個索引結構描述。 如需索引屬性的詳細資訊，請參閱[建立索引 REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)。

> [!Note]
> 某些查詢功能會在整個索引 (而非個別欄位) 啟用。 這些功能包括：[同義字地圖](https://docs.microsoft.com/rest/api/searchservice/synonym-map-operations)、[自訂分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)、[建議工具建構 (用於自動完成和自動建議)](https://docs.microsoft.com/rest/api/searchservice/suggesters)、[結果排名的評分邏輯](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。

## <a name="elements-of-a-query-request"></a>查詢要求的元素

查詢一律會導向單一索引。 您無法加入索引，或建立自訂或暫存資料結構作為查詢目標。 

查詢要求的必要元素包含下列元件：

+ 服務端點和索引文件集合，以包含固定元件和使用者定義元件的 URL 來表示：**`https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`**
+ **`api-version`** (僅限 REST) 是必要項目，原因是 API 隨時都有多個版本。 
+ **`api-key`** (查詢或系統管理 api-key)，會向服務驗證要求。
+ **`queryType`** (簡單或完整)，如果您使用內建的預設簡單語法，則可省略。
+ **`search`** 或 **`filter`** 會提供比對準則，如果您要執行空白搜尋，則可以不指定。 這兩種查詢類型會以簡單的剖析器來進行討論，但更進階的查詢則需要搜尋參數以便傳遞複雜的查詢運算式。

其他所有搜尋參數都是選用的。 如需屬性的完整清單，請參閱[建立索引 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-index)。 若要深入了解處理期間要如何使用參數，請參閱[全文檢索搜尋在 Azure 搜尋服務中的運作方式](search-lucene-query-architecture.md)。

## <a name="choose-a-parser-simple--full"></a>選擇剖析器：簡單 | 完整

Azure 搜尋服務位於 Apache Lucene 之上，並可讓您從兩種查詢剖析器中選擇其一來處理一般和特製化查詢。 使用簡單剖析器的要求會使用[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/Simple-query-syntax-in-Azure-Search)來制訂，可選取作為預設值，以透過自由格式的文字查詢來獲得其速度和效能。 此語法支援許多常見的搜尋運算子，包括 AND、OR、NOT、片語、尾碼和優先順序運算子。

[完整的 Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_syntax)會在您將 `queryType=full` 新增至要求時啟用，可公開廣為採用、且開發作為 [Apache Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) 一部分的表達式查詢語言。 完整語法可延伸簡單語法。 您為簡單語法所撰寫的查詢，均可在完整的 Lucene 剖析器下執行。 

下列範例可說明這點：相同的查詢，但有不同的 queryType 設定，會產生不同的結果。 在第一個查詢中，系統會將 `^3` 視為搜尋字詞的一部分。

```
queryType=simple&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

使用完整 Lucene 剖析器的相同查詢會解譯關於 "ranch" 的欄位內提升，這可提升包含該特定字詞的結果搜尋順位。

```
queryType=simple&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

<a name="types-of-queries"></a>

## <a name="types-of-queries"></a>查詢類型

Azure 搜尋服務支援廣泛的查詢類型。 

| 查詢類型 | 使用量 | 範例和詳細資訊 |
|------------|--------|-------------------------------|
| 自由格式文字檢索 | 搜尋參數和任一剖析器| 全文檢索搜尋會掃描索引中所有「可搜尋」欄位內的一或多個字詞，而且運作方式就如同您對 Google 或 Bing 等搜尋引擎的期待。 簡介中的範例便是全文檢索搜尋。<br/><br/>全文檢索搜尋會使用標準 Lucene 分析器 (預設值) 將所有字詞轉換為小寫，並移除停用字詞 (例如 "the")，以進行文字分析。 若要覆寫預設值，您可以使用[非英文分析器](https://docs.microsoft.com/rest/api/searchservice/language-support#analyzer-list)或[特製化分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search#AnalyzerTable) (以修改文字分析)。 其中一個範例是[關鍵字](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)，其會將欄位的整個內容視為單一語彙基元。 這適合用於郵遞區號、識別碼和產品名稱等資料。 | 
| 經過篩選的搜尋 | [OData 篩選條件運算式](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search)和任一剖析器 | 篩選查詢會對索引中的所有「可篩選」欄位評估布林運算式。 不同於搜尋，篩選查詢會比對欄位的確切內容，包括字串欄位的大小寫。 另一個差異是篩選查詢會以 OData 語法來表示。 <br/>[篩選條件運算式範例](search-query-simple-examples.md#example-3-filter-queries) |
| 地區搜尋 | 欄位上的 [Edm.GeographyPoint 類型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)、篩選條件運算式和任一剖析器 | 座標若儲存在具有 Edm.GeographyPoint 的欄位中，可用於 "find near me" 或以地圖為基礎的搜尋控制項。 <br/>[地理搜尋範例](search-query-simple-examples.md#example-5-geo-search)|
| 範圍搜尋 | 篩選條件運算式和簡單的剖析器 | Azure 搜尋服務會使用篩選參數來建置範圍查詢。 <br/>[範圍篩選範例](search-query-simple-examples.md#example-4-range-filters) | 
| [內部欄位篩選](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_fields) | 搜尋參數和完整剖析器 | 建置以單一欄位為目標的複合查詢運算式。 <br/>[內部欄位篩選範例](search-query-lucene-examples.md#example-2-intra-field-filtering) |
| [模糊搜尋](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_fuzzy) | 搜尋參數和完整剖析器 | 比對具有類似建構或拼法的字詞。 <br/>[模糊搜尋範例](search-query-lucene-examples.md#example-3-fuzzy-search) |
| [鄰近搜尋](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_proximity) | 搜尋參數和完整剖析器 | 尋找文件中彼此相近的字詞。 <br/>[鄰近搜尋範例](search-query-lucene-examples.md#example-4-proximity-search) |
| [詞彙提升](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost) | 搜尋參數和完整剖析器 | 如果文件包含已提升的字詞 (相較於其他未提升的字詞)，則將文件的順位提高。 <br/>[字詞提升範例](search-query-lucene-examples.md#example-5-term-boosting) |
| [規則運算式搜尋](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_regex) | 搜尋參數和完整剖析器 | 根據規則運算式的內容來比對。 <br/>[規則運算式範例](search-query-lucene-examples.md#example-6-regex) |
|  [萬用字元或首碼搜尋](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_wildcard) | 搜尋參數和完整剖析器 | 根據首碼和波狀符號 (`~`) 或單一字元 (`?`) 來比對。 <br/>[萬用字元搜尋範例](search-query-lucene-examples.md#example-7-wildcard-search) |

## <a name="manage-search-results"></a>管理搜尋結果 

即使您使用 .NET API 而內建序列化，查詢結果仍會在 REST API 中以 JSON 文件的形式串流處理。 您可以設定查詢的參數，並為回應選取特定欄位，以形成結果。

查詢的參數可透過下列方式用來建構結果集：

+ 限制或批次處理結果中的文件數目 (預設為 50 個)
+ 選取要包含在結果中的欄位
+ 設定排序次序
+ 新增搜尋結果醒目提示，以凸顯搜尋結果本文中的相符字詞

### <a name="tips-for-unexpected-results"></a>未預期結果的提示

有時候，結果的本質和結構會不符預期的。 當查詢結果不符合您想看到的結果時，您可以嘗試進行下列查詢修改，看看結果是否有所改善：

+ 將 **`searchMode=any`** (預設值) 變更為 **`searchMode=all`**，以要求須符合所有準則，而非任何準則。 此做法在查詢中包含布林運算子時特別有用。

+ 如果文字或語彙分析是必要的，但查詢類型無法進行語言處理，請變更查詢技術。 在全文檢索搜尋中，文字或語彙分析會自動更正拼字錯誤、單數複數文字形式，甚至不規則的動詞或名詞。 在像是模糊或萬用字元搜尋等某些查詢中，文字分析並不是查詢剖析管線的一部分。 在某些情況下，規則運算式是既有的因應措施。 

### <a name="paging-results"></a>分頁結果
Azure 搜尋服務可讓您輕鬆地對搜尋結果分頁。 透過使用 **`top`** 和 **`skip`** 參數，您就可以順利地發出搜尋要求，將所收到的整組搜尋結果分拆成方便管理且經過排序的子集，輕鬆實現良好的搜尋 UI 做法。 在收到這些分拆成較小子集的結果時，您也會收到整組搜尋結果中所含文件的計數。

您可以在 [如何在 Azure 搜尋服務中對搜尋結果分頁](search-pagination-page-layout.md)一文中深入了解如何對搜尋結果分頁。

### <a name="ordering-results"></a>排序結果
收到搜尋查詢的結果時，您可以要求 Azure 搜尋服務提供依特定欄位值排序的結果。 根據預設，Azure 搜尋服務會以每份文件的搜尋分數排名 (衍生自 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)) 來排序搜尋結果。

如果您想要讓 Azure 搜尋服務在傳回結果時是以搜尋分數以外的值來排序，您可以使用 **`orderby`** 搜尋參數。 您可以指定 **`orderby`** 參數的值，納入欄位名稱和 [**`geo.distance()` 函式**](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search)的呼叫以獲得地理空間值。 每個運算式後面都可以加上 `asc` 來表示要以遞增順序要求結果，而加上 **`desc`** 則表示要以遞減順序要求結果。 預設排名為遞增順序。


### <a name="hit-highlighting"></a>搜尋結果醒目提示
在 Azure 搜尋服務中，只要使用 **`highlight`**、**`highlightPreTag`** 和 **`highlightPostTag`** 參數就可強調提示搜尋結果中符合搜尋查詢的確切部分。 您可以指定哪些可搜尋欄位應該強調其相符的文字，以及指定要在 Azure 搜尋服務傳回的相符文字開頭和結尾附加的確切字串標記。

## <a name="apis-and-tools-for-testing"></a>測試用的 API 和工具

下表列出使用 API 和工具來提交查詢的方法。

| 方法 | 說明 |
|-------------|-------------|
| [SearchIndexClient (.NET)](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient?view=azure-dotnet) | 可用來查詢 Azure 搜尋服務索引的用戶端。  <br/>[深入了解。](search-howto-dotnet-sdk.md#core-scenarios)  |
| [搜尋文件 (REST API)](https://docs.microsoft.com/rest/api/searchservice/search-documents) | 索引的 GET 或 POST 方法，使用查詢參數作為額外輸入。  |
| [Fiddler、Postman 或其他 HTTP 測試工具](search-fiddler.md) | 說明如何設定將查詢傳送至 Azure 搜尋服務的要求標頭和本文。  |
| [Azure 入口網站中的搜尋總管](search-explorer.md) | 提供搜尋列以及選取索引和 API 版本的選項。 結果會以 JSON 文件的形式傳回。 <br/>[深入了解。](search-get-started-portal.md#query-index) | 

## <a name="see-also"></a>另請參閱

+ [全文檢索搜尋如何在 Azure 搜尋服務中運作 (查詢剖析架構)](search-lucene-query-architecture.md)
+ [搜尋總管](search-explorer.md)
+ [如何在 .NET 中進行查詢](search-query-dotnet.md)
+ [如何在 REST 中進行查詢](search-query-rest-api.md)
