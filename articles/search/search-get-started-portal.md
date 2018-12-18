---
title: 使用入口網站在 Azure 搜尋服務中進行編製索引、查詢和篩選的教學課程 | Microsoft Docs
description: 在本教學課程中，會使用 Azure 入口網站和預先定義的範例資料在 Azure 搜尋服務中產生索引。 探索全文檢索搜尋、篩選器、面向、模糊搜尋、地理搜尋功能等等。
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.topic: tutorial
ms.date: 07/10/2018
ms.author: heidist
ms.openlocfilehash: aac579da3aaf6ab1507bbc12d79a5b183a82d665
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39592750"
---
# <a name="tutorial-use-built-in-tools-for-azure-search-indexing-and-queries"></a>教學課程：使用內建工具進行 Azure 搜尋服務的編製索引和查詢

如需快速檢閱和進一步了解 Azure 搜尋服務的概念，您可以使用 Azure 入口網站中的 [Azure 搜尋服務] 頁面中，所提供的內建工具。 這些工具可能不會提供 .NET 和 REST API 的完整功能。 但是，其中的精靈和編輯器會提供免程式碼的 Azure 搜尋服務簡介，讓您能夠立即針對資料集範例撰寫有趣的查詢。

> [!div class="checklist"]
> * 從使用公用的範例資料開始，並使用 [匯入資料] 精靈自動產生 Azure 搜尋服務索引。
> * 針對任何發佈至 Azure 搜尋服務的索引，檢視索引結構描述和屬性。
> * 使用**搜尋總管**探索全文檢索搜尋、篩選器、面向、模糊搜尋和地理搜尋功能。  

如果這些工具過於限制，您可以考慮[使用 .NET 進行 Azure 搜尋服務程式設計的程式碼型簡介](search-howto-dotnet-sdk.md)或是[發出 REST API 呼叫的 Web 測試工具](search-fiddler.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 您也可以觀看本教學課程中 6 分鐘的步驟示範，大約在此 [Azure 搜尋服務概觀影片](https://channel9.msdn.com/Events/Connect/2016/138)中的第三分鐘處開始。

## <a name="prerequisites"></a>必要條件

[建立 Azure 搜尋服務](search-create-service-portal.md)，或在您目前的訂用帳戶下尋找現有服務。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟 Azure 搜尋服務的服務儀表板。 如果您並未將服務圖格釘選至儀表板，可以用以下方式找到您的服務︰

   * 在導向列中，按一下左導覽窗格上的 [所有服務]。
   * 在搜尋方塊中，鍵入 search 以取得訂用帳戶的搜尋相關服務清單。 按一下 [搜尋服務]。 您的服務應該會出現在清單中。

### <a name="check-for-space"></a>檢查空間

許多客戶開始使用此免費服務。 此版本的限制為三個索引、三個資料來源，以及三個索引子。 開始之前，請先確定您有空間可容納額外的項目。 本教學課程會建立各一個物件。

> [!TIP]
> 服務儀表板上的圖格會顯示您已經有多少個索引、索引子和資料來源。 [索引子] 圖格可顯示成功與失敗指標。 按一下此圖格以檢視索引子計數。
>
> ![索引子和資料來源的圖格][1]
>

## <a name="create-index"></a> 建立索引和載入資料

搜尋查詢會逐一查看[索引](search-what-is-an-index.md)，其中包含會將特定搜尋行為最佳化的可搜尋資料、中繼資料及其他建構。

在本教學課程中，我們使用可以透過 [匯入資料] 精靈使用[索引子](search-indexer-overview.md)編目的內建資料集範例。 索引子是來源特定的編目程式，可以從支援的 Azure 資料來源讀取中繼資料和內容。 在入口網站中，您可以透過 [匯入資料] 精靈看到這些索引子。 稍候，您可以透過程式設計方式將索引子當作獨立資源來加以建立及管理。

### <a name="step-1-start-the-import-data-wizard"></a>步驟 1︰啟動匯入資料精靈

1. 從 Azure 搜尋服務儀表板，按一下命令列上的 [匯入資料]，以啟動精靈。 此精靈會協助您建立並填入搜尋索引。

    ![匯入資料命令][2]

2. 在精靈中，按一下 [連線到您的資料] > [範例] > [realestate-us-sample]。 此資料來源已預先設定名稱、類型和連線資訊。 一旦建立，就會變成可在其他匯入作業中重複使用的「現有資料來源」。

    ![選取範例資料集][9]

3. 按一下 [確定] 加以使用。

### <a name="skip-cognitive-skills"></a>跳過認知技術

[匯入資料] 會提供選擇性的認知技能步驟，讓您能對編製索引程序新增自訂的 AI 演算法。 目前請先略過此步驟，並繼續進行**自訂目標索引**。

> [!TIP]
> 您可以從[認知搜尋快速入門](cognitive-search-quickstart-blob.md)或是[教學課程](cognitive-search-tutorial-blob.md)，嘗試 Azure 搜尋服務的新認知搜尋預覽功能。

   ![跳過認知技術步驟][11]

### <a name="step-2-define-the-index"></a>步驟 2：定義索引

一般而言，您會使用程式碼來完成建立索引的手做練習。 在本教學課程中，精靈可以為其可搜耙的任何資料來源產生索引。 索引至少需要有名稱和欄位的集合；其中一個欄位應標示為文件索引鍵，以便唯一識別每份文件。

欄位具有資料類型和屬性。 上方的核取方塊為「索引屬性」，可控制欄位的使用方式。

*  表示它會出現在搜尋結果清單中。 例如當欄位僅使用於篩選運算式時，您可以清除此核取方塊，將個別欄位標記為關閉搜尋結果的限制。
* [可篩選]、[可排序] 和 [可面向化] 判斷欄位是否可以用於篩選、排序或多面向導覽結構。
*  表示欄位包含在全文檢索搜尋中。 字串可以搜尋。 數字欄位和布林值欄位通常會標示為不可搜尋。

根據預設，精靈會掃描資料來源中的唯一識別碼做為索引鍵欄位的基礎。 字串具有可擷取和可搜尋的特性。 整數具有可擷取、可篩選、可排序和可 Fact 處理的特性。

  ![產生的 realestate 索引][3]

按一下 [確定] 以建立索引。

### <a name="step-3-define-the-indexer"></a>步驟 3：定義索引子

同樣在 [匯入資料] 精靈中，按一下 [索引子] > [名稱]，並且鍵入索引子的名稱。

此物件定義可執行的程序。 您就可以將其放入週期性排程，但現在請先按一下 [確定]，立即使用預設選項執行索引子一次。  

  ![realestate 索引子][8]

### <a name="check-progress"></a>檢查進度

若要監視資料匯入，請回到服務儀表板，向下捲動，然後按兩下 [索引子] 圖格以開啟索引子清單。 您應該會在清單中看到新建立的索引子，其狀態指出「進行中」或成功，以及已編製索引的文件數目。

   ![索引子進度訊息][4]

### <a name="step-4-view-the-index"></a>步驟 4：檢視索引

服務儀表板中的圖格會同時提供資源中各種物件的摘要資訊以及詳細資訊的存取權。 [索引] 圖格會列出現有索引，包括您剛才在上一個步驟中建立的 realestate-us-sample 索引。

現在按一下 [realestate-us-sample] 索引即可檢視其定義的入口網站選項。 [新增/編輯欄位] 選項可讓您建立並完整編輯新欄位的屬性。 現有的欄位在 Azure 搜尋服務中具有實體表示法，因此不可修改，甚至是在程式碼中也不可修改。 若要對現有欄位進行重大變更，請建立一個新欄位並捨棄原始欄位。

   ![範例索引定義][10]

您可以隨時新增其他建構，例如評分設定檔和 CORS 選項。

若要清楚地了解您在索引設計期間可以編輯和無法編輯的項目，請花點時間檢視索引定義選項。 灰色的選項表示無法修改或刪除選項值。 同樣地，目前請先略過「分析器」及「建議工具」的核取方塊。

## <a name="query-index"></a> 查詢索引

接下來，您現在應該有搜尋索引，準備好要使用內建的 [[搜尋總管]](search-explorer.md) 查詢頁面進行查詢。 它會提供搜尋方塊，讓您能夠測試任意的查詢字串。

> [!TIP]
> 下列步驟會在 [Azure 搜尋服務概觀影片](https://channel9.msdn.com/Events/Connect/2016/138)的 6 分 08 秒處示範。
>

1. 按一下命令列上的 [搜尋總管]  。

   ![搜尋總管命令][5]

2. 按一下命令列上的 [變更索引] 以切換到 realestate-us-sample。 按一下命令列上的 [設定 API 版本] 以查看可用的 REST API。 對於下列查詢，請使用正式運作版本 (2017-11-11)。

   ![索引和 API 命令][6]

3. 在搜尋列中，輸入下列查詢字串，然後按一下 [搜尋]。

    > [!NOTE]
    > [搜尋總管] 只能用於處理 [REST API 要求](https://docs.microsoft.com/rest/api/searchservice/search-documents)。 它可接受[簡單查詢語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)和[完整 Lucene 查詢剖析器](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)的語法，以及[搜尋文件](https://docs.microsoft.com/rest/api/searchservice/search-documents)作業中可用的所有搜尋參數。
    >

### <a name="simple-query-with-top-n-results"></a>含有前 N 個結果的簡單查詢

#### <a name="example-string-searchseattle"></a>範例 (字串)：`search=seattle`

* **search** 參數用來輸入可供全文檢索搜尋的關鍵字搜尋，在此案例中，會傳回華盛頓州金郡的清單，並且在文件任何可搜尋的欄位中包含 Seattle。

* **搜尋總管**會以 JSON 傳回結果，這是詳細資訊，而如果文件的結構很密集則難以閱讀。 這是刻意設計；整份文件的可見性對開發目的而言很重要，特別是在測試期間。 為了獲得更好的使用者體驗，您必須撰寫程式碼，以[處理搜尋結果](search-pagination-page-layout.md)來找出重要的元素。

* 文件是由索引中標記為「可擷取」的所有欄位組成。 若要在入口網站中檢視索引屬性，請按一下 [索引] 圖格中的 realestate-us-sample。

#### <a name="example-parameterized-searchseattlecounttruetop100"></a>範例 (參數化)：`search=seattle&$count=true&$top=100`

* **&** 符號用來附加搜尋參數 (可依任何順序指定)。

* **$count=true** 參數會傳回所有已傳回文件的總計數。 這個值會出現在搜尋結果的頂端附近。 您可以藉由監視 **$count=true** 所報告的變更來驗證篩選查詢。 較小的計數表示正在處理您的篩選條件。

* **$top=100** 會傳回所有文件中最高順位的 100 份文件。 根據預設，Azure 搜尋服務會傳回前 50 個最相符項目。 您可以透過 **$top** 來增加或減少數量。

### <a name="filter-query"></a> 篩選查詢

當您附加 **$filter** 參數時，篩選器會包含在搜尋要求中。 

#### <a name="example-filtered-searchseattlefilterbeds-gt-3"></a>範例 (已篩選)：`search=seattle&$filter=beds gt 3`

* **$filter** 參數會傳回符合您提供之準則的結果。 在此情況下，房間大於 3 間。

* 篩選語法是 OData 建構。 如需詳細資訊，請參閱[篩選 OData 語法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)。

### <a name="facet-query"></a> 將查詢面向化

Facet 篩選器會包含在搜尋要求中。 您可以使用 facet 參數，傳回符合您所提供之 facet 值的文件彙總計數。

#### <a name="example-faceted-with-scope-reduction-searchfacetcitytop2"></a>範例 (多面向且範圍已縮減)：`search=*&facet=city&$top=2`

* **search=*** 是空的搜尋。 空的搜尋會搜尋所有一切。 提交空查詢的其中一個原因是為了篩選一組完整的文件或使其面向化。 例如，您希望多面向導覽結構包含索引中的所有城市。

* **facet** 會傳回您可傳遞至 UI 控制項的導覽結構。 它會傳回一些類別和一個計數。 在此情況下，類別是以城市數目為基礎。 在 Azure 搜尋服務中沒有彙總功能，但您可以透過 `facet` 模擬彙總，其可提供各類別中的文件計數。

* **$top=2** 會傳回兩份文件，這表示您可以使用 `top` 來減少或增加結果。

#### <a name="example-facet-on-numeric-values-searchseattlefacetbeds"></a>範例 (數值的多面向)：`search=seattle&facet=beds`**

* 此查詢是 beds 的面向，以 Seattle 的文字搜尋為基礎。 可以將 beds 字詞指定為一個面向，因為該欄位標示為可在索引中擷取、篩選及面向化 (1 到 5 的數值)，適合用於將清單分類 (包含 3 個房間、4 個房間的清單)。

* 只有可篩選的欄位可以面向化。 只有可擷取的欄位可以在結果中傳回。

### <a name="highlight-query"></a> 醒目提示搜尋結果

命中項目醒目提示是指關鍵字文字比對的格式設定，前提是在特定欄位中找到相符項目。 如果您的搜尋字詞深藏在描述中，您可以新增命中項目醒目提示功能，更輕鬆地發現所搜尋的字詞。

#### <a name="example-highlighter-searchgranite-countertopshighlightdescription"></a>範例 (螢光筆)：`search=granite countertops&highlight=description`

* 在此範例中，格式化片語 granite countertops 可在描述欄位中更容易找出。

#### <a name="example-linguistic-analysis-searchmicehighlightdescription"></a>範例 (語言分析)：`search=mice&highlight=description`

* 全文檢索搜尋會尋找具有類似語意的文字形式。 在受老鼠侵擾家庭的這個案例中，搜尋結果包含 "mouse" 的醒目提示文字，以回應 "mice" 關鍵字搜尋。 因為語言分析的緣故，結果中可能會出現同一個字的不同形式。

* Azure 搜尋服務支援 Microsoft 和 Lucene 所提供的 56 個分析器。 Azure 搜尋服務預設使用標準 Lucene 分析器。

### <a name="fuzzy-search"></a> 嘗試模糊搜尋

根據預設，拼錯的查詢字詞 (如 samamish 是指 Seattle 地區的 Samammish 高原) 無法在一般搜尋中傳回相符項目。 下列範例不會傳回任何結果。

#### <a name="example-misspelled-term-unhandled-searchsamamish"></a>範例 (拼錯的字詞，未處理)：`search=samamish`

若要處理拼字錯誤，您可以使用模糊搜尋。 當您使用完整的 Lucene 查詢語法時，會在下列兩種情況下啟用模糊搜尋：在查詢中設定 **queryType=full** 時，以及將 **~** 附加至搜尋字串時。

#### <a name="example-misspelled-term-handled-searchsamamishquerytypefull"></a>範例 (拼字錯誤的字詞，已處理)：`search=samamish~&queryType=full`

此範例現在會傳回文件，當中包含 "Sammamish" 上的相符項目。

若未指定 **queryType**，則會使用預設的簡單查詢剖析器。 簡單查詢剖析器的速度較快，但如果您需要模糊搜尋、規則運算式、鄰近搜尋或其他進階的查詢類型，則需要完整的語法。

模糊搜尋和萬用字元搜尋會影響搜尋輸出。 不會以這些查詢格式執行語言分析。 在使用模糊及萬用字元搜尋之前，請檢閱[全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md#stage-2-lexical-analysis)，並尋找語彙分析例外狀況的相關章節。

如需完整查詢剖析器所啟用查詢案例的詳細資訊，請參閱 [Azure 搜尋服務中的 Lucene 查詢語法](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)。

### <a name="geo-search"></a> 嘗試地理空間搜尋

地理空間搜尋是透過含有座標之欄位上的 [edm.GeographyPoint 資料類型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)提供支援。 地理搜尋是在[篩選 OData 語法](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search)中指定的一種篩選器。

#### <a name="example-geo-coordinate-filters-searchcounttruefiltergeodistancelocationgeographypoint-122121513-47673988-le-5"></a>範例 (地理座標篩選)：`search=*&$count=true&$filter=geo.distance(location,geography'POINT(-122.121513 47.673988)') le 5`

範例查詢會篩選所有結果中的位置資料，而結果是小於距離指定點 (以緯度和經度座標指定) 5 公里。 藉由新增 **$count**，您可以看到當您變更距離或座標時傳回多少筆結果。

如果搜尋應用程式有「尋找附近地點」功能或使用地圖導航功能，地理空間搜尋便很實用。 不過，並不是全文檢索搜尋。 如果使用者有需要依照名稱搜尋城市或國家/地區，則除了座標以外，請新增包含城市或國家/地區名稱的欄位。

## <a name="takeaways"></a>重要心得

本教學課程提供了從 Azure 入口網站使用 Azure 搜尋服務的快速簡介。

您已了解如何使用**匯入資料**精靈建立搜尋索引。 您已了解[索引子](search-indexer-overview.md)，以及索引設計的基本工作流程，包括[對已發佈索引支援的修改](https://docs.microsoft.com/rest/api/searchservice/update-index)。

藉由在 Azure 入口網站中使用**搜尋總管**，您已透過實際操作的範例了解某些基本查詢語法，這些範例示範了篩選、搜尋結果醒目提示、模糊搜尋和地理搜尋等主要功能。

您也已經了解如何使用入口網站儀表板中適用於搜尋索引、索引子和資料來源的圖格。 日後若有任何新的資料來源，您就可以使用入口網站以最省力的方式快速檢查其定義或欄位集合。

## <a name="clean-up"></a>清除

如果本教學課程是您第一次使用 Azure 搜尋服務，請刪除包含 Azure 搜尋服務的資源群組。 如果不是，請從服務清單找到正確的資源群組名稱，並刪除適當的群組。

## <a name="next-steps"></a>後續步驟

您可以使用程式設計工具深入探索 Azure 搜尋服務：

* [使用 .NET SDK 建立索引](https://docs.microsoft.com/azure/search/search-create-index-dotnet)
* [使用 REST API 建立索引](https://docs.microsoft.com/azure/search/search-create-index-rest-api)
* 使用 [Postman 或 Fiddler 等 Web 測試工具來呼叫 Azure 搜尋服務 REST API](search-fiddler.md)

<!--Image references-->
[1]: ./media/search-get-started-portal/tiles-indexers-datasources2.png
[2]: ./media/search-get-started-portal/import-data-cmd2.png
[3]: ./media/search-get-started-portal/realestateindex2.png
[4]: ./media/search-get-started-portal/indexers-inprogress2.png
[5]: ./media/search-get-started-portal/search-explorer-cmd2.png
[6]: ./media/search-get-started-portal/search-explorer-changeindex-se2.png
[7]: ./media/search-get-started-portal/search-explorer-query2.png
[8]: ./media/search-get-started-portal/realestate-indexer2.png
[9]: ./media/search-get-started-portal/import-datasource-sample2.png
[10]: ./media/search-get-started-portal/sample-index-def.png
[11]: ./media/search-get-started-portal/skip-cog-skill-step.png