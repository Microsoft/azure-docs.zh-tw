---
title: Azure 搜尋服務中的完整文字搜尋引擎 (Lucene) 架構 | Microsoft Docs
description: 與 Azure 搜尋服務相關之全文檢索搜尋的 Lucene 查詢處理和文件擷取概念說明。
manager: jlembicz
author: yahnoosh
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: jlembicz
ms.openlocfilehash: 55d361e90dbc5fe48bc118088a6f859d096048ff
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036865"
---
# <a name="how-full-text-search-works-in-azure-search"></a>全文檢索搜尋如何在 Azure 搜尋服務中運作

本文適用於需要更深入了解 Lucene 全文檢索搜尋在 Azure 搜尋服務中運作方式的開發人員。 針對文字查詢，Azure 搜尋服務在大部分情況下會順暢地提供預期的結果，但您偶爾可能會收到看起來像以某種方式「關閉」的結果。 在這些情況下，Lucene 查詢執行之四個階段 (查詢剖析、語彙分析、文件相符、評分) 中的背景，可協助您識別查詢參數的特定變更或要提供所需結果的索引組態。 

> [!Note] 
> Azure 搜尋服務會使用 Lucene 來進行全文檢索搜尋，但 Lucene 整合並不詳盡。 我們可以選擇性地公開和擴充 Lucene 功能，以啟用對於 Azure 搜尋服務很重要的案例。 

## <a name="architecture-overview-and-diagram"></a>架構概觀和圖表

從剖析查詢文字來擷取搜尋詞彙開始處理全文檢索搜尋查詢。 搜尋引擎會使用索引將具有相符詞彙的文件進行擷取。 個別的查詢詞彙有時會予以細分，並重新組成新形式，以更廣泛搜尋視為可能的相符項目。 然後結果集會依指派給每個個別相符文件的相關性分數進行排序。 這些排名清單頂端的項目會回到呼叫應用程式。

重新敘述的查詢執行有四個階段︰ 

1. 查詢剖析 
2. 語彙分析 
3. 擷取文件 
4. 評分 

下圖說明用來處理搜尋要求的元件。 

 ![Azure 搜尋服務中的 Lucene 查詢架構圖表][1]


| 重要元件 | 功能描述 | 
|----------------|------------------------|
|**查詢剖析器** | 將查詢詞彙與查詢運算子分隔開來，並建立要傳送到搜尋引擎的查詢結構 (查詢樹狀結構)。 |
|**分析器** | 執行查詢詞彙的語彙分析。 此程序可能包含轉換、移除或展開查詢詞彙。 |
|**Index** | 用來儲存和組織可搜尋詞彙的有效資料結構是擷取自編製索引的文件。 |
|**搜尋引擎** | 根據反向索引的內容將比對文件進行擷取和評分。 |

## <a name="anatomy-of-a-search-request"></a>搜尋要求的剖析

搜尋要求是結果集所需傳回的完整規格。 最簡單的形式中，它是不設任何類型之條件的空查詢。 更寫實的範例包括參數、數個查詢詞彙 (或許只限於某些欄位)，可能還包括篩選條件運算式和排序規則。  

下列範例是您可能會使用 [REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 傳送至 Azure 搜尋服務的搜尋要求。  

~~~~
POST /indexes/hotels/docs/search?api-version=2017-11-11 
{  
    "search": "Spacious, air-condition* +\"Ocean view\"",  
    "searchFields": "description, title",  
    "searchMode": "any",
    "filter": "price ge 60 and price lt 300",  
    "orderby": "geo.distance(location, geography'POINT(-159.476235 22.227659)')", 
    "queryType": "full" 
 } 
~~~~

針對此要求，搜尋引擎會執行下列作業︰

1. 篩選出價格為至少 $60 且小於 $300 的文件。
2. 執行查詢。 在此範例中，搜尋查詢是由片語和詞彙所組成︰`"Spacious, air-condition* +\"Ocean view\""` (使用者通常不會輸入標點符號，但將它包含在範例中，可讓我們解釋分析器如何處理它)。 對於此查詢，搜尋引擎會掃描 `searchFields` 中指定的描述和標題欄位，以找出包含 "Ocean view"的文件，並額外掃描 "spacious" 這個詞彙或以 "air-condition" 這個前置詞開頭的詞彙。 `searchMode` 參數是在並非明確需要詞彙的案例 (`+`) 中，用來比對任何詞彙 (預設值) 或所有這些詞彙。
3. 依特定地理位置之鄰近性將旅館的結果集進行排序，然後傳回給呼叫應用程式。 

本文大部分內容是關於處理「搜尋查詢」：`"Spacious, air-condition* +\"Ocean view\""`。 篩選和排序不在範圍內。 如需詳細資訊，請參閱[搜尋服務 API 參考文件](https://docs.microsoft.com/rest/api/searchservice/search-documents)。

<a name="stage1"></a>
## <a name="stage-1-query-parsing"></a>第 1 階段︰查詢剖析 

如前所述，查詢字串是要求的第一行︰ 

~~~~
 "search": "Spacious, air-condition* +\"Ocean view\"", 
~~~~

查詢剖析器會分隔運算子 (例如範例中的 `*` 和 `+`) 與搜尋詞彙，並將搜尋查詢解構為支援類型的子查詢︰ 

+ 詞彙查詢適用於獨立詞彙 (例如 spacious)
+ 片語查詢適用於引號括住的詞彙 (例如 ocean view)
+ 前置詞查詢適用於前置詞運算子後面的詞彙 `*` (例如 air-condition)

如需支援查詢類型的完整清單，請參閱 [Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)

為使文件視為相符項目，與子查詢相關聯的運算子會決定查詢「必須」還是「應該」符合。 例如，由於 `+` 運算子，`+"Ocean view"` 是「必須」。 

查詢剖析器會將子查詢重新建立成它傳遞給搜尋引擎的查詢樹狀結構 (表示查詢的內部結構)。 在查詢剖析的第一個階段中，查詢樹狀結構看起來像這樣。  

 ![Boolean query searchmode any][2]

### <a name="supported-parsers-simple-and-full-lucene"></a>支援的剖析器︰簡單和完整的 Lucene 

 Azure 搜尋服務會公開兩種不同的查詢語言，`simple` (預設值) 和 `full`。 藉由使用搜尋要求來設定 `queryType` 參數，您告知查詢剖析器所選擇的查詢語言，讓它知道如何解譯運算子和語法。 [簡單的查詢語言](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)是直覺式且強固的，通常適用於以現況解譯使用者輸入而無用戶端處理。 它支援 web 搜尋引擎熟悉的查詢運算子。 [完整的 Lucene 查詢語言](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)是您藉由設定 `queryType=full` 所取得，會透過新增更多運算子與查詢類型 (例如萬用字元、模糊、Regex 和欄位範圍查詢) 的支援，來擴充預設的簡單查詢語言。 例如，簡單查詢語法傳入的規則運算式會解譯為查詢字串而非運算式。 本文中的範例要求會使用完整的 Lucene 查詢語言。

### <a name="impact-of-searchmode-on-the-parser"></a>SearchMode 在剖析器上的影響 

會影響剖析的另一個搜尋要求參數是 `searchMode` 參數。 它會控制布林值查詢的預設運算子︰any (預設值) 或 all。  

當 `searchMode=any` (這是預設值) 時，spacious 和 air-condition 之間的空白分隔符號為 OR (`||`)，會使範例查詢文字相當於︰ 

~~~~
Spacious,||air-condition*+"Ocean view" 
~~~~

明確運算子 (例如 `+"Ocean view"` 中的 `+`) 在布林值查詢建構中是很明確的 (詞彙必須相符)。 較不明顯的是如何解譯其餘詞彙︰spacious 和 air-condition。 搜尋引擎應該尋找 ocean view 和  spacious 和 air-condition 的相符項目嗎？ 還是它應該尋找 ocean view 加上其中一項？ 

根據預設 (`searchMode=any`)，搜尋引擎會假設較廣泛的解譯。 任一個欄位應該相符，反映「或」語意。 先前所述使用這兩個 "should" 作業的初始查詢樹狀結構會顯示預設值。  

假設我們現在設定 `searchMode=all`。 在此情況下，空白會解譯為 "and" 作業。 每個其餘詞彙都必須存在於文件中，才可認定為相符。 產生的範例查詢會如下解譯︰ 

~~~~
+Spacious,+air-condition*+"Ocean view"  
~~~~

此查詢的經修改查詢樹狀結構會如下所示，其中相符文件是所有三個子查詢的交集︰ 

 ![Boolean query searchmode all][3]

> [!Note] 
> 透過 `searchMode=all` 選擇 `searchMode=any` 是藉由執行具代表性之查詢所得出的最佳決策。 如果 `searchMode=all` 通知布林值查詢建構，則可能包括運算子的使用者 (搜尋文件存放區時很常見) 應該會覺得結果較具直覺性。 如需 `searchMode` 和運算子之間所發生之相互作用的詳細資訊，請參閱[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)。

<a name="stage2"></a>
## <a name="stage-2-lexical-analysis"></a>第 2 階段：語彙分析 

在將查詢樹狀結構進行結構化之後，語彙分析器會處理詞彙查詢和片語查詢。 分析器會接受剖析器所提供給它的文字輸入、處理文字，然後將要併入查詢樹狀結構的權杖化詞彙傳回。 

最常見的語彙分析形式是語言分析它會根據給定語言的特定規則將查詢詞彙進行轉換︰ 

* 將查詢詞彙減少為文字的根形式 
* 移除不必要的字 (停用字詞，例如英文的 "the" 或 "and") 
* 將複合字分成多個組件元件 
* 將大寫的字改成小寫 

這些作業通常全都會將使用者所提供之文字輸入和索引中儲存的詞彙之間的差異清除。 這類作業超出文字處理，而且需要深入了解語言本身。 若要新增此語言感知圖層，Azure 搜尋服務支援一系列 Lucene 和 Microsoft 提供的[語言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

> [!Note]
> 取決於您的案例，分析需求的範圍可從最小到詳細。 您可以控制語彙分析的複雜性，方法為選取其中一個預先定義的分析器，或建立您自己的[自訂分析器](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search)。 分析器的範圍設定為可搜尋的欄位，並指定為欄位定義的一部分。 這可讓您依每個欄位更改語彙分析。 若未指定，則會使用標準 Lucene 分析器。

在範例中，分析之前的初始查詢樹狀結構會有 "Spacious," 的詞彙，包含大寫的 "S" 以及查詢剖析器解譯為查詢詞彙 (逗號不會視為查詢語言運算子) 一部分的逗號。  

當預設分析器處理詞彙時，它會將 "ocean view" 和 "spacious" 小寫，並將逗號字元移除。 修改過的查詢樹狀結構看起來如下︰ 

 ![包含分析過詞彙的布林值查詢][4]

### <a name="testing-analyzer-behaviors"></a>測試分析器的行為 

可以使用[分析 API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer)來測試分析器的行為。 提供您想要分析的詞彙，查看指定分析器將會產生的文字。 例如，若要查看標準分析器會如何處理 "air-condition" 文字，您可以發出下列要求︰

~~~~
{ 
    "text": "air-condition",
    "analyzer": "standard"
}
~~~~

標準分析器會將輸入文字分成下列兩個權杖，並以開始和結束位移 (用於點閱數醒目提示) 等屬性以及它們的位置 (用於相符片語) 加以註解︰

~~~~
{  
  "tokens": [
    {
      "token": "air",
      "startOffset": 0,
      "endOffset": 3,
      "position": 0
    },
    {
      "token": "condition",
      "startOffset": 4,
      "endOffset": 13,
      "position": 1
    }
  ]
}
~~~~

<a name="exceptions"></a>

### <a name="exceptions-to-lexical-analysis"></a>語彙分析的例外狀況 

語彙分析只適用於需要完整詞彙 (詞彙查詢或片語查詢) 的查詢類型。 它不會套用至詞彙不完整的查詢類型 – 前置詞查詢、萬用字元查詢、Regex 查詢 – 或不會套用至模糊查詢。 這些查詢類型 (包括範例中具有 `air-condition*` 詞彙的前置詞查詢) 會直接新增至查詢樹狀結構，並略過分析階段。 針對這些類型之查詢詞彙所執行的唯一轉換就是小寫。

<a name="stage3"></a>

## <a name="stage-3-document-retrieval"></a>第 3 階段：擷取文件 

擷取文件是指在索引中尋找包含相符詞彙的文件。 了解這個階段的最佳方式就是範例。 讓我們從具有下列簡單結構描述的旅館索引開始︰ 

~~~~
{   
    "name": "hotels",     
    "fields": [     
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },     
        { "name": "title", "type": "Edm.String", "searchable": true },     
        { "name": "description", "type": "Edm.String", "searchable": true }
    ] 
} 
~~~~

進一步假設這個索引包含下列四個文件︰ 

~~~~
{ 
    "value": [
        {         
            "id": "1",         
            "title": "Hotel Atman",         
            "description": "Spacious rooms, ocean view, walking distance to the beach."   
        },       
        {         
            "id": "2",         
            "title": "Beach Resort",        
            "description": "Located on the north shore of the island of Kauaʻi. Ocean view."     
        },       
        {         
            "id": "3",         
            "title": "Playa Hotel",         
            "description": "Comfortable, air-conditioned rooms with ocean view."
        },       
        {         
            "id": "4",         
            "title": "Ocean Retreat",         
            "description": "Quiet and secluded"
        }    
    ]
}
~~~~

**詞彙編製索引的方式**

若要了解擷取，那麼了解有關編製索引的幾個基本概念會有所幫助。 儲存體的單位是反向索引，每個可搜尋的欄位會有一個。 反向索引內會有所有文件中所有詞彙的排序清單。 每個詞彙會對應到它在其中出現的文件清單，如下列範例中所示。

若要在反向索引中產生詞彙，搜尋引擎會透過文件的內容執行語彙分析，類似於查詢處理期間所發生的動作：

1. *文字輸入*會傳遞至分析器、小寫、除去標點符號等等，視分析器組態而定。 
2. *權杖*是文字分析的輸出。
3. *字詞*會新增至索引。

使用相同的分析器進行搜尋和編製索引作業很常見但並非必要，可讓查詢詞彙看起來比較像索引內的詞彙。

> [!Note]
> Azure 搜尋服務可讓您指定不同的分析器來編製索引，並透過其他的 `indexAnalyzer` 和 `searchAnalyzer` 欄位參數進行搜尋。 如果未指定，以 `analyzer` 屬性設定的分析器可用於編製索引和搜尋。  

**範例文件的反向索引**

回到我們的範例，針對 [標題] 欄位，反向索引看起來像這樣︰

| 詞彙 | 文件清單 |
|------|---------------|
| atman | 1 |
| beach | 2 |
| hotel | 1, 3 |
| ocean | 4  |
| playa | 3 |
| resort | 3 |
| retreat | 4 |

在 [標題] 欄位中，只有旅館會出現在兩個文件中︰1、3。

針對 [描述] 欄位，索引如下所示︰

| 詞彙 | 文件清單 |
|------|---------------|
| air | 3
| 和 | 4
| beach | 1
| conditioned | 3
| comfortable | 3
| distance | 1
| island | 2
| kauaʻi | 2
| located | 2
| north | 2
| ocean | 1, 2, 3
| of | 2
| on |2
| quiet | 4
| rooms  | 1, 3
| secluded | 4
| shore | 2
| spacious | 1
| the | 1、2
| to | 1
| view | 1, 2, 3
| walking | 1
| 取代為 | 3


**比對查詢詞彙與索引的詞彙**

假設上述的反向索引，讓我們回到範例查詢，並查看如何找到我們範例查詢的相符文件。 回想一下，最後一個查詢樹狀結構看起來像這樣︰ 

 ![包含分析過詞彙的布林值查詢][4]

在執行查詢期間，會針對可搜尋的欄位單獨執行個別查詢。 

+ "spacious" 這個 TermQuery 符合文件 1 (Hotel Atman)。 

+ "air-condition *" 這個 PrefixQuery 不符合任何文件。 

  這個行為有時候會混淆開發人員。 雖然 air-conditioned 這個詞彙存在於文件中，它還是可由分析器劃分為兩個詞彙。 還記得包含部分詞彙的前置詞查詢不會進行分析。 因此，會在反向索引中查閱包含 "air-condition" 前置詞的詞彙，但找不到。

+ "ocean view" 這個 PhraseQuery 會查閱 "ocean" 和 "view" 這兩個詞彙，並檢查原始文件中的詞彙鄰近性。 文件 1、2 和 3 在描述欄位中符合這項查詢。 請注意，文件 4 在標題中有 ocean 這個詞彙，但不被認為相符，因為我們是要尋找 "ocean view" 片語而非個別單字。 

> [!Note]
> 搜尋查詢會針對 Azure 搜尋服務索引中所有可搜尋的欄位獨立執行，除非您使用 `searchFields` 參數來限制欄位設定，如範例搜尋要求中所示。 會傳回任何選取欄位中相符的文件。 

整體來說，針對問題中的查詢，相符的文件是 1、2、3。 

## <a name="stage-4-scoring"></a>第 4 階段︰評分  

會將相關性分數指派給搜尋結果集中的每個文件。 相關性分數的功能，是將使用者問題之最佳解答的這些文件排名在較高順位，如搜尋查詢所表示。 分數會根據相符詞彙的統計屬性來計算。 評分公式的核心是 [TF/IDF (詞彙頻率-反向文件頻率)](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)。 在包含罕見和一般詞彙的查詢中，TF/IDF 會提升包含罕見詞彙的結果。 例如，在所有包含 Wikipedia 文章的假設索引中，從比對 the president 查詢的文件，比對 president 的文件會視為較比對 the 的文件更具有相關性。


### <a name="scoring-example"></a>評分範例

還記得與我們範例查詢相符的三份文件︰
~~~~
search=Spacious, air-condition* +"Ocean view"  
~~~~
~~~~
{  
  "value": [
    {
      "@search.score": 0.25610128,
      "id": "1",
      "title": "Hotel Atman",
      "description": "Spacious rooms, ocean view, walking distance to the beach."
    },
    {
      "@search.score": 0.08951007,
      "id": "3",
      "title": "Playa Hotel",
      "description": "Comfortable, air-conditioned rooms with ocean view."
    },
    {
      "@search.score": 0.05967338,
      "id": "2",
      "title": "Ocean Resort",
      "description": "Located on a cliff on the north shore of the island of Kauai. Ocean view."
    }
  ]
}
~~~~

文件 1 與查詢最相符，因為 spacious 詞彙和必要的 ocean view片語都出現在 [描述] 欄位中。 接下來兩個文件只符合 *ocean view* 片語。 您可能會很意外，文件 2 和 3 的相關性分數不同，即使它們符合查詢的方式相同。 這是因為相較於只有 TF/IDF，評分公式具有更多個元件。 在此情況下，已將較高的分數指派給文件 3，因為它的說明較短。 深入了解 [Lucene 的實際評分公式](https://lucene.apache.org/core/4_0_0/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html)，以了解欄位長度和其他因素會如何影響相關性分數。

某些查詢類型 (萬用字元、前置詞、Regex) 一律會對文件的整體分數提供常數分數。 這可讓透過查詢擴充找到的相符項目包含在結果中，但不會影響排名。 

範例會說明這很重要的原因。 萬用字元搜尋 (包括前置詞搜尋) 根據定義是模稜兩可的，因為輸入是部分字串，包含非常大量不同詞彙的可能相符項目 (請考慮 "tour*" 的輸入，包含 “tours”、“tourettes” 和 “tourmaline” 找到的相符項目)。 考慮到這些結果的本質，便無法合理推斷哪些詞彙較其他詞彙更有價值。 基於這個理由，當評分導致萬用字元、前置詞和 Rregex 的查詢類型時，我們會忽略詞彙頻率。 在包含部分和完整詞彙的多部分搜尋要求中，部分輸入中的結果會納入常數的分數，以避免可能無法預期的相符項目偏差。

### <a name="score-tuning"></a>微調分數

有兩種方式可以調整 Azure 搜尋服務中的相關性分數︰

1. **評分設定檔**會根據一組規則，依結果的排名清單來升級文件。 在範例中，我們可以認為 [標題] 欄位中相符的文件比 [描述] 欄位中相符的文件更為相關。 此外，如果索引有每家旅館的價格欄位，我們便可以提升包含較低價格的文件。 深入了解如何[將評分設定檔新增至搜尋索引。](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)
2. **詞彙提升**(僅適用於完整的 Lucene 查詢語法) 提供 `^` 這個提升運算子，可以套用到查詢樹狀結構的任何部分。 在範例中，使用者並非搜尋 air-condition\* 這個前置詞，而是可以搜尋精確詞彙 air-condition 或前置詞，但是符合精確詞彙的文件可藉由將提升套用至詞彙查詢 *air-condition^2||air-condition** 來提高排名。 深入了解[詞彙提升](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search#bkmk_termboost)。


### <a name="scoring-in-a-distributed-index"></a>分散式索引中的評分

Azure 搜尋服務中的所有索引會自動分成多個分區，讓我們可以在服務相應增加或相應減少期間，快速地在多個節點之間將索引進行散發。 當發出搜尋要求時，它會針對每個分區獨立發行。 然後每個分區的結果會進行合併，並依分數排序 (如果未定義其他的順序)。 請務必了解，評分函式會針對其在分區內所有文件中的反向文件頻率 (而非針對所有分區)，將查詢詞彙頻率進行加權！

這表示如果相同的文件位於不同的分區，則相同文件的相關性分數可能會有所不同。 幸運的是，這類差異通常會隨著索引中的文件數目增加而消失，因為更甚至隨著詞彙散發。 無法假設會將任何指定的文件放置在哪一個分區。 不過，假設文件金鑰並不會變更，則一律會將它指派至相同的分區。

一般情況下，如果順序穩定性很重要，則文件分數便不是排序文件的最佳屬性。 例如，假設兩個文件具有完全相同的分數，則不保證在相同查詢的後續執行中會先出現哪一個。 相對於結果集內的其他文件，文件分數應該只提供廣義的文件相關性。

## <a name="conclusion"></a>結論

網際網路搜尋引擎的成功，使得透過私人資料進行全文檢索搜尋的期望大幅提高。 針對幾乎所有種類的搜尋經驗，我們現在預期引擎可以了解我們的目的，即使是詞彙拼錯或不完整。 我們甚至可能會根據接近對等的詞彙，或從來不會實際指定的同義字來預期相符項目。

從技術觀點來看，全文檢索搜尋相當複雜，需要精密的語言分析及系統化的方法，透過將查詢詞彙進行摘錄、展開並轉換來傳遞相關結果的方式進行處理。 假設固有的複雜性，則有許多因素可能會影響查詢的結果。 基於這個理由，嘗試進行非預期的結果時，投入時間了解全文檢索搜尋的機制能夠獲益良多。  

本文探討 Azure 搜尋服務內容中的全文檢索搜尋。 我們希望它能提供足夠的背景，讓您可以找出可能的原因和解決方式，來解決常見的查詢問題。 

## <a name="next-steps"></a>後續步驟

+ 建立範例索引、嘗試不同的查詢和檢視結果。 如需指示，請參閱[在入口網站中建置和查詢索引](search-get-started-portal.md#query-index)。

+ 請從[搜尋文件](https://docs.microsoft.com/rest/api/searchservice/search-documents#examples)範例章節，或從入口網站的搜尋總管中[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)，嘗試其他查詢語法。

+ 如果您想要調整搜尋應用程式中的排名，請檢閱[評分設定檔](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)。

+ 了解如何套用[特定語言的語彙分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

+ [設定自訂分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)以進行最少的處理，或是在特定欄位上進行特殊的處理。

+ 在這個示範網站上同時[比較標準和英文分析器](http://alice.unearth.ai/)。 

## <a name="see-also"></a>另請參閱

[搜尋文件 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 

[完整的 Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 

[處理搜尋結果](https://docs.microsoft.com/azure/search/search-pagination-page-layout)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
[2]: ./media/search-lucene-query-architecture/azSearch-queryparsing-should2.png
[3]: ./media/search-lucene-query-architecture/azSearch-queryparsing-must2.png
[4]: ./media/search-lucene-query-architecture/azSearch-queryparsing-spacious2.png
