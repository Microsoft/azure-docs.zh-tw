---
title: 什麼是 Azure 搜尋服務 | Microsoft Docs
description: Azure 搜尋服務是完全受控的託管雲端搜尋服務。 深入了解此功能概觀。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.topic: overview
ms.date: 07/09/2018
ms.author: heidist
ms.openlocfilehash: 4874e42139e277400c866a7fc4c7094faeae136d
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39002199"
---
# <a name="what-is-azure-search"></a>何謂 Azure 搜尋服務？
Azure 搜尋服務是搜尋即服務雲端解決方案，可為開發人員提供 API 和工具，透過 Web、行動和企業應用程式中的私用和異質內容來增添豐富的搜尋體驗。

透過簡單的 [REST API](/rest/api/searchservice/) 或 [.NET SDK](search-howto-dotnet-sdk.md) 展現功能，並同時隱蔽資訊擷取固有之複雜性。 除了 API 之外，Azure 入口網站也提供系統管理和內容管理的支援，以及建立原型和查詢索引的工具。 因為服務在雲端執行，Microsoft 會管理基礎結構和可用性。

<a name="feature-drilldown"></a>

## <a name="feature-summary"></a>功能摘要

| 類別 | 特性 |
|----------|----------|
|全文檢索搜尋和文字分析 | [全文檢索搜尋](search-lucene-query-architecture.md)是大多數以搜尋為基礎之應用程式的主要使用案例。 您可以使用 Azure 搜尋服務支援的語法制訂查詢。 <br/><br/>[**簡單查詢語法**](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)可提供邏輯運算子、片語搜尋運算子、後置運算子、優先順序運算子。<br/><br/>[**Lucene 查詢語法**](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)包含簡單語法中的所有作業，並具有諸如模糊搜尋、鄰近搜尋、詞彙提升和規則運算式等擴充功能。|
|認知搜尋 (預覽) | 用於影像和文字分析的 [AI 演算法](cognitive-search-concept-intro.md)可以套用至索引管線，以從原始內容擷取文字資訊。 [內建技巧](cognitive-search-predefined-skills.md)的範例包括光學字元辨識 (讓掃描過的 JPEG 可搜尋)、實體辨識 (識別組織、名稱或位置)，以及關鍵片語辨識。 您也可以[將自訂技巧編碼](cognitive-search-create-custom-skill-example.md)，以附加至管線。 |
| 資料整合 | Azure 搜尋服務索引接受以 JSON 資料結構格式提交的任何來源。 <br/><br/> 對於在 Azure 中支援的資料來源，您也可以使用[**索引子**來](search-indexer-overview.md)自動編目 [Azure SQL Database](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)、[Azure Cosmos DB](search-howto-index-cosmosdb.md) 或 [Azure Blob 儲存體](search-howto-indexing-azure-blob-storage.md)，以取得主要資料存放區的可搜尋內容。 Azure Blob 索引子可執行「文件破解」以[從主要檔案格式擷取文字](search-howto-indexing-azure-blob-storage.md)，包括 Microsoft Office、PDF 和 HTML 文件。 |
| 語言分析 | 分析器是在編製索引和搜尋作業期間用於文字處理的元件。 有兩種類型。 <br/><br/>[**自訂語彙分析器**](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search) (英文) 用於使用語音比對和規則運算式的複雜搜尋查詢。 <br/><br/>Lucene 或 Microsoft 的[**語言分析器**](https://docs.microsoft.com/rest/api/searchservice/language-support) (英文) 可用來以智慧方式處理特定語言的語言學，包括動詞時態、性別、不規則複數名詞 (例如 ' mouse' 與 'mice')、拆解複合字、斷字 (針對沒有空格的語言) 等。 |
| 地區搜尋 | Azure 搜尋服務可處理、篩選和顯示地理位置。 可讓使用者根據搜尋結果與實體位置的鄰近程度來瀏覽資料。 如需詳細資訊，請[觀看此影片](https://channel9.msdn.com/Shows/Data-Exposed/Azure-Search-and-Geospatial-Data) (英文) 或[檢閱此範例](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) (英文)。 |
| 使用者體驗功能 | 您可以在搜尋列中針對預先輸入的查詢啟用[**自動完成 (預覽)**](search-autocomplete-tutorial.md)。 <br/><br/>[**搜尋建議**](https://docs.microsoft.com/rest/api/searchservice/suggesters)也會在搜尋列中排除部分的文字輸入，但結果是您索引中的實際文件，而不是查詢字詞。 <br/><br/>與對等字詞相關聯的[**同義字**](search-synonyms.md)，讓使用者不必提供替代字詞，就能以隱含方式擴充查詢範圍。 <br/><br/>透過單一查詢參數便可啟用[**多面向導覽**](search-faceted-navigation.md) (英文)。 Azure 搜尋服務會傳回多面向導覽結構，可做為類別清單背後的程式碼，用於自我引導篩選 (例如，依價格範圍或品牌來篩選類別目錄項目)。 <br/><br/> [**篩選**](https://docs.microsoft.com/rest/api/searchservice/odata-expression-syntax-for-azure-search) (英文) 可用於將多面向導覽納入應用程式的 UI、增強查詢編寫，以及根據使用者或開發人員指定的準則進行篩選。 請使用 OData 語法建立篩選。<br/><br/> [**搜尋結果醒目提示**](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)會將文字格式套用至搜尋結果中的相符關鍵字。 您可以選擇哪些欄位傳回醒目提示的文字片段。<br/><br/>我們會透過索引結構描述針對多個欄位提供[**排序**](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (英文)，然後使用單一搜尋參數在查詢階段進行切換。<br/><br/> Azure 搜尋服務針對搜尋結果提供完善的控制項，讓您可輕鬆對搜尋結果進行[**分頁**](search-pagination-page-layout.md)和節流。  
| 相關性 | [**簡單評分**](/rest/api/searchservice/add-scoring-profiles-to-a-search-index)是 Azure 搜尋服務的主要優點。 評分設定檔用於將相關性模型化為文件本身中的值函數。 例如，您可能想要新推出的產品或折扣的產品，出現在搜尋結果中較高的位置。 您也可以根據您所追蹤並個別儲存的客戶搜尋喜好設定，使用標記進行個人化計分來建置計分設定檔。 |
| 監視和報告 | 收集和分析[**搜尋流量分析**](search-traffic-analytics.md)可深入分析使用者在搜尋方塊中輸入的內容。 <br/><br/>自動會擷取每秒查詢次數、延遲和節流的計量，並在入口網站頁面中報告，不需要其他設定。 您也可以輕鬆地監視索引和文件計數，以便視需要調整容量。 如需詳細資訊，請參閱[服務管理](search-manage.md) |
| 用於原型設計和檢查的工具 | 在入口網站中，您可以使用 [[匯入資料]](search-import-data-portal.md) 精靈設定索引子、使用索引設計工具建立索引，以及使用 [[搜尋總管]](search-explorer.md) 測試查詢並修改評分設定檔。 您也可以開啟任何索引，以檢視其結構描述。 |
| 基礎結構 | **高可用性平台**可確保相當可靠的搜尋服務體驗。 經過適當的調整， [Azure 搜尋服務可提供 99.9% SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/)。<br/><br/> Azure 搜尋服務是**完全受控和可調整的**端對端解決方案，完全不需要基礎結構管理。 以兩種方式調整您的服務，即可針對您的需求量身訂做，以處理更多的文件儲存體、更高的查詢負載，或兩者。

## <a name="how-to-use-azure-search"></a>如何使用 Azure 搜尋服務
### <a name="step-1-provision-service"></a>步驟 1：佈建服務
您可以在 [Azure 入口網站](https://portal.azure.com/)中或透過 [Azure 資源管理 API](/rest/api/searchmanagement/) 佈建 Azure 搜尋服務。 您可以選擇與其他訂閱者共用的免費服務，或選擇只為您的服務提供專用資源的[付費層](https://azure.microsoft.com/pricing/details/search/)。 若選擇付費層，您可以從兩方面調整服務︰ 

- 新增複本，以擴大容量來處理繁重的查詢負載。   
- 新增資料分割，以擴充儲存體來存放更多文件。 

將文件儲存體和查詢輸送量分開處理，可讓您根據生產需求而校正資源分配。

### <a name="step-2-create-index"></a>步驟 2：建立索引
您必須先定義 Azure 搜尋服務索引，才能上傳可搜尋的內容。 索引就像是資料庫資料表，其中保存您的資料並可接受搜尋查詢。 您可定義索引結構描述，以反映您要搜尋的文件結構 (類似於資料庫中的欄位)。

您可以在 Azure 入口網站中建立結構描述，或使用 [.NET SDK](search-howto-dotnet-sdk.md) 或 [REST API](/rest/api/searchservice/) 以程式設計方式建立結構描述。

### <a name="step-3-index-data"></a>步驟 3︰索引資料
定義索引之後，您便可以上傳內容。 您可以使用推送或提取模型。

提取模型會從外部資料來源擷取資料。 這可透過*索引子*而達成，索引子可簡化和自動化資料擷取的各層面，例如連線至、讀取和序列化資料。 [索引子](/rest/api/searchservice/Indexer-operations)適用於 Azure Cosmos DB、Azure SQL Database、Azure Blob 儲存體和 Azure VM 中裝載的 SQL Server。 您可以重新整理隨需或排定的資料以設定索引子。

推送模式是透過 SDK 或 REST API 來提供，可用來將已更新的文件傳送到索引。 使用 JSON 格式，您幾乎可以從任何資料集發送資料。 如需載入資料的指引，請參閱[新增、更新或刪除文件](/rest/api/searchservice/addupdate-or-delete-documents)或[如何使用 .NET SDK](search-howto-dotnet-sdk.md)。

### <a name="step-4-search"></a>步驟 4︰搜尋
填入索引後，您可以透過 REST API 或 .NET SDK，使用簡單的 HTTP 要求對服務端點[發出搜尋查詢](/rest/api/searchservice/Search-Documents)。

## <a name="how-azure-search-compares"></a>Azure 搜尋服務的比較方式

客戶通常會詢問 Azure 搜尋服務與其他搜尋相關之解決方案的比較方式。 下表摘要說明主要差異。

| 相較於 | 主要差異 |
|-------------|-----------------|
|Bing | [Bing Web 搜尋 API](https://docs.microsoft.com/azure/cognitive-services/bing-web-search/) 會在 Bing.com 上搜尋索引中符合您送出的項目。 索引是從 HTML、XML 和其他公用網站上的網站內容所建置。 根據相同的基礎，[Bing 自訂搜尋](https://docs.microsoft.com/azure/cognitive-services/bing-custom-search/)會為網站內容類型提供相同的編目程式技術，範圍設定為個別的網站。<br/><br/>Azure 搜尋服務會搜尋您所定義的索引，使用您所擁有的資料與文件填入，通常是來自不同的來源。 Azure 搜尋服務可透過[索引子](search-indexer-overview.md)為一些資料來源提供編目程式功能，但您可以將任何符合您索引結構描述的 JSON 文件推送到單一、彙總的可搜尋資源。 |
|資料庫搜尋 | 許多資料庫平台包括內建的搜尋經驗。 SQL Server 有[全文檢索搜尋](https://docs.microsoft.com/sql/relational-databases/search/full-text-search)。 Cosmos DB 與類似技術有可供查詢的索引。 對結合搜尋和儲存體的產品進行評估時，可能不容易決定要使用哪一個。 許多解決方案會同時使用兩者：具有儲存體的 DBMS 和具有特製化搜尋功能的 Azure 搜尋。<br/><br/>相較於 DBMS 搜尋，Azure 搜尋會儲存異質來源中的內容及提供特製化文字處理功能，例如超過 [55 種語言](https://docs.microsoft.com/rest/api/searchservice/language-support)的語言感知文字處理 (詞幹分析、詞形歸併還原、文字形式)。 並也支援拼字錯誤的自動校正、[同義字](https://docs.microsoft.com/rest/api/searchservice/synonym-map-operations)、[建議](https://docs.microsoft.com/rest/api/searchservice/suggestions)、[計分控制](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)、[Facet](https://docs.microsoft.com/azure/search/search-filters-facets) 和[自訂 Token 化](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search)。 Azure 搜尋服務中的[全文檢索搜尋引擎](search-lucene-query-architecture.md)是建置於 Apache Lucene，這是資訊擷取的業界標準。 Azure 搜尋服務會以反向索引的形式保留資料，而以此代替真實的資料儲存體十分罕見。 如需詳細資訊，請參閱[此論壇文章](https://stackoverflow.com/questions/40101159/can-azure-search-be-used-as-a-primary-database-for-some-data)。 <br/><br/>資源使用率是此類別中的另一項轉折點。 編製索引及某些查詢作業通常會耗用大量運算資源。 將搜尋從 DBMS 卸載至雲端中的專用解決方案，可保留用於交易處理的系統資源。 此外，將搜尋外部化可以輕鬆調整級別以符合查詢量。|
|專用的搜尋解決方案 | 假設您已決定使用具有全面功能的專用搜尋，最後就是要比較內部部署解決方案或雲端服務的範圍。 許多搜尋技術會提供編製索引和查詢管線的控制權、存取更豐富的查詢及篩選語法、控制順位和相關性，以及自我引導和智慧型搜尋的功能。 <br/><br/>如果您想要擁有[最少額外負荷和維護，還可調整級別的周全方案](#cloud-service-advantage)，雲端服務是正確的選擇。 <br/><br/>在雲端範式內，許多提供者提供可比較的基準功能，不但有全文檢索搜尋、地區搜尋，還能夠處理搜尋輸入中一定程度的語意模糊。 是否最適合通常取決於[特殊化功能](#feature-drilldown)，或 API、工具和管理的容易性和整體簡單性。 |

在雲端提供者中，針對主要依賴搜尋來擷取資訊和導覽內容的應用程式來說，Azure 搜尋服務在 Azure 上的內容存放區和資料庫處理全文檢索搜尋工作負載時，功能最強大。 

主要優點包括︰

+ 索引層的 Azure 資料整合 (編目程式)
+ 集中管理的 Azure 入口網站
+ Azure 調整性、可靠性和世界級的可用性
+ 語言和自訂分析，還有分析器提供可靠的全文檢索搜尋，支援 56 種語言
+ [搜尋導向應用程式常用的核心功能包括](#feature-drilldown)：評分、多面向、建議、同義字、地區搜尋等。

> [!Note]
> 雖然完全支援非 Azure 資料來源，但依賴需要更大量程式碼的推送方法，而不是索引子。 透過 API，您可以將任何 JSON 文件集合輸送至 Azure 搜尋服務索引。

在我們的客戶中，可利用 Azure 搜尋服務中最廣泛功能的客戶包括線上型錄、商務營運程式和文件探索應用程式。

## <a name="rest-api--net-sdk"></a>REST API | .Net SDK

雖然在入口網站中可以執行許多工作，但 Azure 搜尋服務主要是提供給想要將搜尋功能整合到現有應用程式的開發人員使用。 有下列程式設計介面可用。

|平台 |說明 |
|-----|------------|
|[REST](/rest/api/searchservice/) | 任何程式設計平台和語言 (包括 Xamarin、Java 和 JavaScript) 支援的 HTTP 指令|
|[.NET SDK](search-howto-dotnet-sdk.md) | REST API 的 .NET 包裝函式支援使用 C# 和以 .NET Framework 為目標的其他受控程式碼語言，有效率地撰寫程式碼 |

## <a name="free-trial"></a>免費試用
Azure 訂閱者可以[在免費層中佈建服務](search-create-service-portal.md)。

如果您不是訂閱者，可以[免費申請 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)。 您會獲得信用額度來試用 Azure 付費服務。 信用額度用完之後，您可以保留帳戶並使用[免費的 Azure 服務](https://azure.microsoft.com/free/)。 除非您明確變更您的設定且同意付費，否則我們絕對不會從您的信用卡收取任何費用。

或者，您也可以[啟用 MSDN 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F)：您的 MSDN 訂用帳戶每月會提供您額度，您可以用在 Azure 付費服務。 

## <a name="how-to-get-started"></a>如何開始使用

1. 在[免費層](search-create-service-portal.md)中建立服務。

2. 逐步執行一個或多個下列教學課程。 

  + [如何使用 .NET SDK](search-howto-dotnet-sdk.md) 會示範受控碼中的主要步驟。  
  + [開始使用 REST API](https://github.com/Azure-Samples/search-rest-api-getting-started) (英文) 會示範使用 REST API 的相同步驟。  
  + 使用內建索引和原型功能，[在入口網站中建立您的第一個索引](search-get-started-portal.md)。   

搜尋引擎是行動裝置應用程式、Web 和公司資料存放區中推動資訊擷取的常用工具。 Azure 搜尋服務提供工具讓您創造類似大型商業網站上的搜尋體驗。

在程式管理員 Liam Cavanagh 主講的這段 9 分鐘影片中，您可以了解應用程式如何經由整合搜尋引擎而受益。 這段簡短的示範涵蓋了 Azure 搜尋服務中的主要功能，以及一般工作流程的樣貌。 

>[!VIDEO https://channel9.msdn.com/Events/Connect/2016/138/player]
 
+ 0-3 分鐘涵蓋主要功能和使用案例。
+ 3-4 分鐘涵蓋服務佈建。 
+ 4-6 分鐘涵蓋使用「匯入資料」精靈，以內建的房地產資料集建立索引。
+ 第 6-9 分鐘涵蓋「搜尋總管」和各種查詢。


