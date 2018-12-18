---
title: Azure 搜尋服務中的分析器 | Microsoft Docs
description: 將分析器指派給索引中的可搜尋文字欄位，可將預設的標準 Lucene 取代為自訂、預先定義或特定語言的替代項目。
services: search
ms.service: search
ms.topic: conceptual
ms.date: 09/11/2017
ms.author: heidist
manager: cgronlun
author: HeidiSteen
ms.openlocfilehash: 68ce4fa5536f21d6d66245a9383a4b58c42febff
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49116365"
---
# <a name="analyzers-in-azure-search"></a>Azure 搜尋服務中的分析器

*分析器*是一種[全文搜尋](search-lucene-query-architecture.md)元件，負責查詢字串和已編製索引文件中的文字處理。 在分析期間，會有下列典型轉換：

+ 會移除非必要字組 (停用字詞) 和標點符號。
+ 片語和有連字號的字組會被細分為組件。
+ 大寫字組會被改為小寫。
+ 字組會縮減為根表單，如此不論時態就可以找到相符項目。

語言分析器會將文字輸入轉換為基本表單或根表單，如此可以有效率地儲存和檢索資訊。 在編製索引期間、索引建立之時、以及讀取索引進行搜尋的時候，皆會進行轉換。 若兩方作業均使用相同的文字分析器，更有可能獲得想要的搜尋結果。

Azure 搜尋服務會使用[標準 Lucene 分析器](https://lucene.apache.org/core/4_0_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) \( 英文\) 作為預設值。 您可以逐欄覆寫預設值。 本文描述選項範圍，並提供自訂分析的最佳作法。 文中也會提供重要案例的範例設定。

## <a name="supported-analyzers"></a>支援的分析器

下列清單說明 Azure 搜尋服務中支援的分析器。

| 類別 | 說明 |
|----------|-------------|
| [標準 Lucene 分析器](https://lucene.apache.org/core/4_0_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) | 預設值。 不需要任何規格或設定。 這個一般用途的分析器對於大部分的語言和情節都能順利執行。|
| 預先定義的分析器 | 提供作為完成的產品旨在依現狀使用，並包含有限的自訂。 <br/>共有兩種類型：特製化和語言。 使它們成為「預先定義」的條件是依名稱參考，而無任何自訂。 <br/><br/>[特製化 (語言無從驗證) 分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search#AnalyzerTable)適用於文字輸入需要特殊處理或最少處理時。 非語言預先定義的分析器包含 **Asciifolding**、**金鑰**、**模式**、**簡單**、**停止**、**空白**。<br/><br/>[語言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)適用於當您需要為個別語言提供豐富的語言支援時。 Azure 搜尋服務支援 35 個 Lucene 語言分析器和 50 個 Microsoft 自然語言處理分析器。 |
|[自訂分析器](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search) | 結合現有元素的使用者定義組態，包括一個權杖化工具 (必要) 和選擇性篩選條件 (Char 或權杖)。|

您可以自訂預先定義的分析器，例如**模式**或**停止**，從而使用記載於[預先定義分析器參考](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search#AnalyzerTable)的替代選項。 只有幾個預先定義的分析器具有可供您設定的選項。 如同任何自訂所敘述，為您的新組態提供名稱，例如 myPatternAnalyzer，以便與 Lucene 分析器有所區別。

## <a name="how-to-specify-analyzers"></a>如何指定分析器

1. (僅適用於自訂分析器) 在索引定義中建立 **analyzer** 區段。 如需詳細資訊，請參閱[建立索引](https://docs.microsoft.com/rest/api/searchservice/create-index)以及[自訂分析器 > 建立](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search#create-a-custom-analyzer)。

2. 在索引的[欄位定義](https://docs.microsoft.com/rest/api/searchservice/create-index)上，將 **analyzer** 屬性設為目標分析器的名稱 (例如 `"analyzer" = "keyword"`)。 有效值包括預先定義的分析器名稱、語言分析器名稱，或者也會定義於索引結構描述中的自訂分析器名稱。

3. 您可以選擇性地使用 **indexAnalyzer** 和 **searchAnalyzer`** 欄位參數，而非一個 **analyzer** 屬性，設定不同的分析器來編製索引和查詢。 

3. 將分析器新增至欄位定義會在索引上產生寫入作業。 如果您將**分析器**新增至現有索引，請注意下列步驟：
 
 | 案例 | 影響 | 步驟 |
 |----------|--------|-------|
 | 新增欄位 | 最低限度 | 若在結構描述中尚未具有欄位，則由於在索引中尚未實際存有欄位，因此不會執行欄位修訂。 使用[更新索引](https://docs.microsoft.com/rest/api/searchservice/update-index)和[mergeOrUpload](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents) 進行這項工作。|
 | 將分析器新增至現有已編製索引的欄位。 | 重建 | 該欄位的反向索引必須從頭重新編製，且必須重新編製這些欄位的內容索引。 <br/> <br/>針對正在開發中的索引，[刪除](https://docs.microsoft.com/rest/api/searchservice/delete-index)和[建立](https://docs.microsoft.com/rest/api/searchservice/create-index)索引以挑選新的欄位定義。 <br/> <br/>針對生產環境中的索引，您應該建立新的欄位來提供修改過的定義，然後開始使用它。 使用[更新索引](https://docs.microsoft.com/rest/api/searchservice/update-index)和 [mergeOrUpload](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents) 合併新欄位。 稍後，您可以清除索引以移除過時的欄位，當作規劃索引服務的一部分。 |

## <a name="tips-and-best-practices"></a>秘訣和最佳作法

本節提供如何使用分析器的建議。

### <a name="one-analyzer-for-read-write-unless-you-have-specific-requirements"></a>一個用於讀寫的分析器，除非您有特定的需求

Azure 搜尋服務可讓您指定不同的分析器來編製索引，並透過其他的 `indexAnalyzer` 和 `searchAnalyzer` 欄位參數進行搜尋。 如果未指定，以 `analyzer` 屬性設定的分析器可用於編製索引和搜尋。 若未指定 `analyzer`，就會使用標準 Lucene 分析器。

一般規則是使用索引和查詢的相同分析器，除非特定需求另有指示。 請務必徹底測試。 當搜尋和索引時的文字處理不同，會有查詢字詞和索引字詞不符的風險，因為搜尋和索引分析器的設定不一致。

### <a name="test-during-active-development"></a>在開發期間進行測試

覆寫標準分析器需要重建索引。 可能的話，請先決定要在開發期間使用的作用中分析器，然後才將索引實際執行。

### <a name="inspect-tokenized-terms"></a>檢查權杖化字詞

若搜尋作業無法傳回預期中的結果，則查詢的字詞輸入以及索引中的權杖化字詞之間，極有可能發生權杖不相符的狀況。 若權杖不相同，則比對作業無法實行。 若要檢查權杖化工具輸出，建議使用[分析 API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer) 做為調查工具。 回應由權杖組成，一如特定分析器所產生的權杖。

### <a name="compare-english-analyzers"></a>比較英文分析器

[搜尋分析器示範](http://alice.unearth.ai/)是一種協力廠商示範應用程式，其會顯示標準 Lucene 分析器、Lucene 的英文語言分析器，以及 Microsoft 的英文版自然語言處理器之間的並排比較。 索引已固定；內含熱門故事中的文字。 您提供的每項搜尋輸入，會在相鄰窗格中顯示來自每個分析器的結果，協助您瞭解每個分析器處理相同字串的方式。 

## <a name="examples"></a>範例

下列範例會顯示幾個重要情節的分析器定義。

<a name="Example1"></a>
### <a name="example-1-custom-options"></a>範例 1：自訂選項

此範例會使用自訂選項來說明分析器定義。 Char 篩選、安全性權杖和權杖篩選條件會個別指定為具名的建構，然後在分析器定義中加以參考。 預先定義的元素會依現狀使用，而且只依名稱加以參考。

逐步解說這個範例：

* 分析器是可搜尋欄位的欄位類別屬性。
* 自訂分析器是索引定義的一部分。 它可能是小幅自訂 (例如，在一個篩選條件中自訂單一選項) 或在多個位置中加以自訂。
* 在此情況下，自訂分析器是 "my_analyzer"，會依次使用自訂的標準權杖化工具 "my_standard_tokenizer" 和兩個權杖篩選條件：小寫和自訂的 asciifolding 篩選條件 "my_asciifolding"。
* 它也會定義自訂 "map_dash" Char 篩選條件，在 Token 化 (標準權杖化工具是在連字號而非在底線中斷) 之間，將所有連字號取代為底線。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"my_analyzer"
        }
     ],
     "analyzers":[
        {
           "name":"my_analyzer",
           "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
           "charFilters":[
              "map_dash"
           ],
           "tokenizer":"my_standard_tokenizer",
           "tokenFilters":[
              "my_asciifolding",
              "lowercase"
           ]
        }
     ],
     "charFilters":[
        {
           "name":"map_dash",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["-=>_"]
        }
     ],
     "tokenizers":[
        {
           "name":"my_standard_tokenizer",
           "@odata.type":"#Microsoft.Azure.Search.StandardTokenizerV2",
           "maxTokenLength":20
        }
     ],
     "tokenFilters":[
        {
           "name":"my_asciifolding",
           "@odata.type":"#Microsoft.Azure.Search.AsciiFoldingTokenFilter",
           "preserveOriginal":true
        }
     ]
  }
~~~~

<a name="Example2"></a>
### <a name="example-2-override-the-default-analyzer"></a>範例 2：覆寫預設分析器

預設值為標準分析器。 假設您需要將預設值取代為不同的預先定義分析器，例如模式分析器。 如果您未設定自訂選項，就只需要在欄位定義中依名稱加以指定。

「分析器」元素會以各欄位方式來覆寫標準分析器。 沒有通用的覆寫。 在此範例中，`text1` 會使用模式分析器和 `text2` (其中並未指定分析器) 來使用預設值。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text1",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"pattern"
        },
        {
           "name":"text2",
           "type":"Edm.String",
           "searchable":true
        }
     ]
  }
~~~~

<a name="Example3"></a>
### <a name="example-3-different-analyzers-for-indexing-and-search-operations"></a>範例 3：用來索引和搜尋作業的不同分析器

API 包含其他的索引屬性，可針對索引和搜尋指定不同的分析器。 必須將 `searchAnalyzer` 和 `indexAnalyzer` 屬性指定為一組，從而取代單一 `analyzer` 屬性。


~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
     ],
  }
~~~~

<a name="Example4"></a>
### <a name="example-4-language-analyzer"></a>範例 4：語言分析器

包含不同語言之字串的欄位可以使用語言分析器，而其他欄位則是保留預設值 (或使用一些其他的預先定義或自訂分析器)。 如果您是使用語言分析器，就必須將它用於索引和搜尋作業。 使用語言分析器的欄位不能具有索引和搜尋的不同分析器。

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
        {
           "name":"text_fr",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"fr.lucene"
        }
     ],
  }
~~~~

## <a name="next-steps"></a>後續步驟

+ 請檢閱[全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md)的完整說明。 本文使用範例來說明表面上看似違反直覺的行為。

+ 請從[搜尋文件](https://docs.microsoft.com/rest/api/searchservice/search-documents#examples)範例章節，或從入口網站的搜尋總管中[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)，嘗試其他查詢語法。

+ 了解如何套用[特定語言的語彙分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)。

+ [設定自訂分析器](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)以進行最少的處理，或是在個別欄位上進行特殊的處理。

+ 在這個示範網站上的相鄰窗格中[比較標準和英文分析器](http://alice.unearth.ai/)。 

## <a name="see-also"></a>另請參閱

 [搜尋文件 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

 [簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 

 [完整的 Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 
 
 [處理搜尋結果](https://docs.microsoft.com/azure/search/search-pagination-page-layout)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
