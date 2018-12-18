---
title: Azure 搜尋服務的簡單查詢範例 | Microsoft Docs
description: 全文檢索搜尋、篩選搜尋、地理搜尋、多面向搜尋，和其他用來查詢 Azure 搜尋服務索引的查詢字串適用的簡單查詢範例。
author: HeidiSteen
manager: cgronlun
tags: Simple query analyzer syntax
services: search
ms.service: search
ms.topic: conceptual
ms.date: 08/09/2018
ms.author: heidist
ms.openlocfilehash: 2d9e69a900f6665aa0ee3034cd6f9d7c394e8f0b
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42146161"
---
# <a name="simple-syntax-query-examples-for-building-queries-in-azure-search"></a>在 Azure 搜尋服務中建置查詢的簡單語法查詢範例

[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)會叫用預設查詢剖析器，以對 Azure 搜尋服務索引執行全文檢索搜尋查詢。 簡單查詢分析器速度快，可處理 Azure 搜尋服務中的常見案例，包括全文檢索搜尋、篩選搜尋、多面向搜尋和地理搜尋。 本文中的範例示範在使用簡單語法時可用的查詢作業，請逐步執行這些範例。

替代的查詢語法是[完整 Lucene](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)，可支援如模糊和萬用字元搜尋等較複雜的查詢結構，而其處理時間可能會比較久。 如需詳細資訊和示範完整語法的範例，請參閱 [Lucene 語法查詢範例](search-query-lucene-examples.md)。

## <a name="formulate-requests-in-postman"></a>以 Postman 編寫要求

下列範例會根據 [紐約市 OpenData](https://nycopendata.socrata.com/) 計劃所提供的資料集，利用由可用工作組成的 NYC 工作搜尋索引。 這項資料不應視為目前的或已完成。 此索引屬於 Microsoft 所提供的沙箱服務，這表示您不需要 Azure 訂用帳戶或 Azure 搜尋服務，即可嘗試執行這些查詢。

您的需要是 Postman，或可對 GET 發出 HTTP 要求的對等工具。 如需詳細資訊，請參閱[使用 REST 用戶端瀏覽](search-fiddler.md)。

### <a name="set-the-request-header"></a>設定要求標頭

1. 在要求標頭中，將 **Content-Type** 設定為 `application/json`。

2. 新增 **api-key**，並將其設定為此字串：`252044BE3886FE4A8E3BAA4F595114BB`。 這是裝載 NYC 工作索引的沙箱搜尋服務所適用的查詢金鑰。

在指定要求標頭後，您可以將其重複用於本文中的所有查詢，只要替換掉 **search=** 字串即可。 

  ![Postman 要求標頭](media/search-query-lucene-examples/postman-header.png)

### <a name="set-the-request-url"></a>設定要求 URL

要求是與包含 Azure 搜尋服務端點和搜尋字串的 URL 成對的 GET 命令。

  ![Postman 要求標頭](media/search-query-lucene-examples/postman-basic-url-request-elements.png)

URL 組合具有下列元素：

+ **`https://azs-playground.search.windows.net/`** 是由 Azure 搜尋服務開發小組維護的沙箱搜尋服務。 
+ **`indexes/nycjobs/`** 是該服務的索引集合中包含的 NYC 工作索引。 要求上必須同時有服務名稱和索引。
+ **`docs`** 是包含所有可搜尋內容的文件集合。 要求標頭中提供的查詢 API 金鑰僅適用於以文件集合為目標的讀取作業。
+ **`api-version=2017-11-11`** 會設定 API 版本，這是每個要求的必要參數。
+ **`search=*`** 是查詢字串，在初始查詢中設為 Null，會傳回前 50 個結果 (根據預設)。

## <a name="send-your-first-query"></a>傳送第一個查詢

在驗證步驟中，將下列要求貼到 GET 中，然後按一下 [傳送]。 結果會以詳細 JSON 文件的形式傳回。 您可以將此 URL 複製並貼到下方的第一個範例中。

  ```http
  https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&search=*
  ```

查詢字串 **`search=*`** 是未指定的搜尋，等同於 Null 或空的搜尋。 其功用並不高，卻是最方便執行的搜尋。

(選擇性) 您可以將 **`$count=true`** 新增至 URL，以傳回符合搜尋準則的文件計數。 在空的搜尋字串上，這是索引中的所有文件 (在 NYC 作業的案例中大約 2800)。

## <a name="how-to-invoke-simple-query-parsing"></a>如何叫用簡單查詢剖析

針對互動式查詢，您不需要指定任何項目：簡單是預設值。 在程式碼中，如果您先前叫用 **queryType=full** 而使用完整查詢語法，您可以透過 **queryType=simple** 重設預設值。

## <a name="example-1-field-scoped-query"></a>範例 1：欄位範圍查詢

第一個範例不是剖析器專屬的，但我們會介紹第一個基本查詢概念：內含項目。 此範例會將查詢執行和回應範圍限定為少數特定欄位。 當您的工具是 Postman 或搜尋總管時，了解如何建構可讀取的 JSON 回應很重要。 

為求簡潔，查詢僅以 *business_title* 欄位為目標，且指定僅傳回公司職稱。 語法為 **searchFields** 可將執行查詢限制為只有 business_title 欄位，而 **select** 可指定要包含在回應中的欄位。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&searchFields=business_title&$select=business_title&search=*
```

此查詢的回應會如下列螢幕擷取畫面所示。

  ![Postman 範例回應](media/search-query-lucene-examples/postman-sample-results.png)

您可能已注意到在回應中的搜尋分數。 沒有排名時，分數一律為 1，這是因為搜尋不是全文檢索搜尋，或是未套用任何準則。 若是未套用任何準則的 Null 搜尋，資料列會以任意順序傳回。 當您包含實際準則時，您會發現搜尋分數逐漸具有其實質意義。

## <a name="example-2-look-up-by-id"></a>範例 2︰依識別碼查閱

此範例有點不規則，但在評估搜尋行為時，您可能想要查看特定文件的完整內容，以了解結果為何會包含或排除該文件。 若要傳回一份完整的文件，請使用[查閱作業](https://docs.microsoft.com/rest/api/searchservice/lookup-document)傳入文件識別碼。

所有文件都有唯一識別碼。 若要嘗試使用查閱查詢的語法，請先傳回文件識別碼清單，以尋找您要使用的文件。 NYC 工作的識別碼會儲存在 `id` 欄位中。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&searchFields=id&$select=id&search=*
 ```

下一個範例是根據 `id` "9E1E3AF9-0660-4E00-AF51-9B654925A2D5" (出現在上述回應中的最上方) 傳回特定文件的查閱查詢。 下列查詢會傳回整份文件，而不只是選取的欄位。 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs/9E1E3AF9-0660-4E00-AF51-9B654925A2D5?api-version=2017-11-11&$count=true&search=*
 ```

## <a name="example-3-filter-queries"></a>範例 3：篩選查詢

[篩選語法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)是您可搭配**搜尋**使用或單獨使用的 OData 運算式。 獨立的篩選條件 (不含搜尋參數) 在篩選運算式能夠完全限定相關文件時，將有其效用。 沒有查詢字串，就沒有語彙或語言分析、沒有計分 (所有分數均為 1)，也沒有排名。 請注意，搜尋字串是空的。

```http
POST /indexes/nycjobs/docs/search?api-version=2017-11-11  
    {  
      "search": "",
      "filter": "salary_frequency eq 'Annual' and salary_range_from gt 90000",
      "select": "select=job_id, business_title, agency, salary_range_from",
      "count": "true"
    }
```

搭配使用時，會先將篩選套用到整個索引，再對篩選結果執行搜尋。 由於篩選能夠減少搜尋查詢需要處理的資料集合，因此對於提升查詢效能方面是很實用的技術。

  ![篩選查詢回應](media/search-query-simple-examples/filtered-query.png)

如果您想要使用 GET 在 Postman 中試用看看，您可以貼入此字串：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,business_title,agency,salary_range_from&search=&$filter=salary_frequency eq 'Annual' and salary_range_from gt 90000
 ```

合併篩選和搜尋的另一種強大方式就是透過篩選運算式中的 **`search.ismatch*()`**，您可以在其中的篩選內使用搜尋查詢。 這個篩選運算式在 *plan* 上使用萬用字元來選取包含 term plan、planner、planning 等等的 business_title。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,business_title,agency&search=&$filter=search.ismatch('plan*', 'business_title', 'full', 'any')
 ```

如需此函式的詳細資訊，請參閱[「篩選範例」中的 search.ismatch](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)。

## <a name="example-4-range-filters"></a>範例 4︰範圍篩選條件

範圍篩選是透過任何資料類型的 **`$filter`** 運算式來支援。 下列範例會搜尋數值和字串欄位。 

資料類型在範圍篩選條件中很重要，而當數值資料位於數值欄位且字串資料位於字串欄位時效果最好。 字串欄位中的數值資料不適合用於範圍，因為數值字串在 Azure 搜尋服務中無法比較。 

下列範例採用 POST 格式，以方便閱讀 (數值範圍，後面接著文字範圍)：

```http
POST /indexes/nycjobs/docs/search?api-version=2017-11-11  
    {  
      "search": "",
      "filter": "num_of_positions ge 5 and num_of_positions lt 10",
      "select": "job_id, business_title, num_of_positions, agency",
      "orderby": "agency",
      "count": "true"
    }
```
  ![數值範圍的範圍篩選條件](media/search-query-simple-examples/rangefilternumeric.png)


```http
POST /indexes/nycjobs/docs/search?api-version=2017-11-11  
    {  
      "search": "",
      "filter": "business_title ge 'A*' and business_title lt 'C*'",
      "select": "job_id, business_title, agency",
      "orderby": "business_title",
      "count": "true"
    }
```

  ![文字範圍的範圍篩選條件](media/search-query-simple-examples/rangefiltertext.png)

您也可以使用 GET 在 Postman 中試用這些：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&search=&$filter=num_of_positions ge 5 and num_of_positions lt 10&$select=job_id, business_title, num_of_positions, agency&$orderby=agency&$count=true
```

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&search=&$filter=business_title ge 'A*' and business_title lt 'C*'&$select=job_id, business_title, agency&$orderby=business_title&$count=true
 ```

> [!NOTE]
> 透過值範圍進行面向化是常見的搜尋應用程式需求。 如需為 Facet 導覽結構建立篩選條件的詳細資訊和範例，請參閱[「如何實作多面向導覽」中的「根據範圍篩選」](search-faceted-navigation.md#filter-based-on-a-range)。

## <a name="example-5-geo-search"></a>範例 5：異地搜尋

此範例索引包含具有經度和緯度座標的 geo_location 欄位。 這個範例會使用 [geo.distance 函式](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search#filter-examples)，以篩選起始點周圍以至您提供的任意距離 (以公里為單位) 內的文件。 您可以調整查詢 (4) 中的最後一個值，以縮小或放大查詢的介面區。

下列範例採用 POST 格式以便閱讀：

```http
POST /indexes/nycjobs/docs/search?api-version=2017-11-11  
    {  
      "search": "",
      "filter": "geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4",
      "select": "job_id, business_title, work_location",
      "count": "true"
    }
```
為了讓結果更容易閱讀，搜尋結果會經過修剪，以包含作業識別碼、職稱及工作地點。 起始座標是從索引中的隨機文件取得 (在此例中為史泰登島上的工作地點)。

您也可以使用 GET 在 Postman 中試用看看：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&search=&$select=job_id, business_title, work_location&$filter=geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4
```

## <a name="example-6-search-precision"></a>範例 6：搜尋精準度

字詞查詢是個別評估的單一字詞 (可能有許多個)。 片語查詢會以引號括住，並以逐字字串的形式評估。 比對的精確度由運算子和 searchMode 所控制。

範例 1︰**`&search=fire`** 傳回 150 個結果，其中，所有相符項目皆在文件中的某處包含 word 一字。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&search=fire
```

範例 2︰**`&search=fire department`** 傳回 2002 個結果。 系統會針對包含 fire 或 department 的文件傳回相符項目。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&search=fire department
```

範例 3︰**`&search="fire department"`** 傳回 82 個結果。 以引號括住字串時會對這兩個字詞進行逐字搜尋，並從索引中包含此組合字詞的權杖化字詞尋找相符項目。 這解釋了為何 **`search=+fire +department`** 這樣的搜尋並不相同。 這兩個字詞都必須存在，但兩者的掃描會個別執行。 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&search="fire department"
```

## <a name="example-7-booleans-with-searchmode"></a>範例 7︰使用 searchMode 的布林值

簡單語法支援字元格式 (`+, -, |`) 的布林運算子。 SearchMode 參數會指出精確度與召回率之間的取捨；`searchMode=any` 會以召回率優先 (符合任何準則即可將文件納入結果集內)，而 `searchMode=all` 則以精確度優先 (必須符合所有準則)。 預設值為 `searchMode=any`，這在以多個運算子堆疊出查詢而取得範圍較廣 (而非範圍較窄) 的結果時，可能會令人混淆。 在使用 NOT 更是如此，因為其結果會包含所有「不含」特定字詞的文件。

使用預設 searchMode (any) 會傳回 2800 份文件：包含複合字詞 "fire department" 的文件，以及所有不含 "Metrotech Center" 一詞的文件。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&searchMode=any&search="fire department"  -"Metrotech Center"
```

  ![搜尋模式：任何](media/search-query-simple-examples/searchmodeany.png)

將 searchMode 變更為 `all` 時，會對準則強制執行累加效果，而傳回較小的結果集 (21 份文件)，由包含完整片語 "fire department"、但工作地點不是 Metrotech Center 的文件所組成。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&searchMode=all&search="fire department"  -"Metrotech Center"
```
  ![搜尋模式：全部](media/search-query-simple-examples/searchmodeall.png)

## <a name="example-8-structuring-results"></a>範例 8︰建構結果

有數個參數會控制要在搜尋結果中包含的欄位、每個批次中傳回的文件數目，以及排序次序。 此範例將從前述幾個範例延伸，使用 **$select** 陳述式和逐字搜尋準則將結果限定於特定欄位，而傳回 82 個相符項目 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"
```
接續先前的範例，您可以依職稱排序。 之所以可進行此排序，是因為 civil_service_title 在索引中是*可排序的*。

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title
```

分頁結果會使用 **$top** 參數來實作，在此範例中會傳回前 5 份文件：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=0
```

若要取得後續的 5 份文件，請略過第一個批次：

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2017-11-11&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=5
```

## <a name="next-steps"></a>後續步驟
嘗試在您的程式碼中指定查詢。 下列連結說明如何使用預設的簡單語法設定 .NET 和 REST API 的搜尋查詢。

* [使用 .NET SDK 查詢 Azure 搜尋服務索引](search-query-dotnet.md)
* [使用 REST API 查詢 Azure 搜尋服務索引](search-query-rest-api.md)

您可以在下列連結中找到其他語法參考、查詢架構和範例：

+ [建置進階查詢的 Lucene 語法查詢範例](search-query-lucene-examples.md)
+ [全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md)
+ [簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [完整 Lucene 查詢](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [篩選和 Order 語法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)
