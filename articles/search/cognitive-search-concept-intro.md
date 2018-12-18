---
title: Azure 搜尋服務中的資料擷取和自然語言 AI 處理的認知搜尋 | Microsoft Docs
description: 可使用認知技能和 AI 演算法在 Azure 搜尋服務索引中建立可搜尋內容的內容擷取、自然語言處理 (NLP) 和影像處理
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 08/07/2018
ms.author: heidist
ms.openlocfilehash: 72d1630ecaeada3acf8b49952a31ccd3ae8634aa
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39617953"
---
# <a name="what-is-cognitive-search"></a>什麼是認知搜尋？

認知搜尋可藉由將 AI 演算法附加至索引管線，從不可搜尋的內容建立可搜尋的資訊。 AI 整合是透過*認知技能*執行的，可在使用搜尋索引的過程中擴充來源文件。 

**自然語言處理**技能包括[實體辨識](cognitive-search-skill-named-entity-recognition.md)、語言偵測、[關鍵片語擷取](cognitive-search-skill-keyphrases.md)、文字操作和情感偵測。 透過這些技能，非結構化的文字會變成結構化文字，並對應至索引中可搜尋和可篩選的欄位。

**影像處理**包括 [OCR](cognitive-search-skill-ocr.md) 和[視覺功能](cognitive-search-skill-image-analysis.md)的識別，例如臉部偵測、影像轉譯、影像辨識 (名人和地標)，或是色彩或影像方向之類的屬性。 您可以為影像內容建立可使用 Azure 搜尋服務的各種查詢功能來搜尋的文字表示法。

![認知搜尋管線圖](./media/cognitive-search-intro/cogsearch-architecture.png "認知搜尋管線概觀")

Azure 搜尋服務中的認知技能以認知服務 API 中使用的相同 AI 演算法為基礎：[具名實體辨識 API](cognitive-search-skill-named-entity-recognition.md)、[關鍵片語擷取 API](cognitive-search-skill-keyphrases.md) 和 [OCR API](cognitive-search-skill-ocr.md) 只是其中幾項。 

在資料擷取階段中會套用自然語言和影像處理，且其結果會在 Azure 搜尋服務的可搜尋索引中成為文件撰寫的一部分。 資料會作為 Azure 資料集的來源，然後使用您所需的[內建技能](cognitive-search-predefined-skills.md)透過索引管線推送出去。 其架構是可延伸的，因此如果內建技能不敷使用，您可以建立及附加[自訂技能](cognitive-search-create-custom-skill-example.md)，以整合自訂處理。 其範例包括以特定領域為目標 (例如金融、科學出版品或醫藥) 的自訂實體模組或文件分類器。

> [!NOTE]
> 認知搜尋為公開預覽，且目前免費提供技能集執行。 我們將在不久後宣布此功能的定價。 

## <a name="components-of-cognitive-search"></a>認知搜尋的元件

認知搜尋是 [Azure 搜尋服務](search-what-is-azure-search.md)的預覽功能，適用於美國中南部和西歐區域的每一層。 

認知搜尋管線以會搜耙資料來源並提供端對端索引處理的 [Azure 搜尋服務*索引子*](search-indexer-overview.md)為基礎。 技能現在已連結至索引子，會根據您所定義的技能集攔截及擴充文件。 完成索引編製後，您即可使用 [Azure 搜尋服務所支援的所有查詢類型](search-query-overview.md)，透過搜尋要求來存取內容。  如果您不熟悉索引子，本節將引導您逐步完成相關步驟。

### <a name="source-data-and-document-cracking-phase"></a>來源資料和文件萃取階段

在一開始執行管線時，您會有非結構化的文字或非文字內容 (例如影像和掃描的文件 JPEG 檔案)。 資料必須存在於可由索引子存取的 Azure 資料儲存體服務中。 索引子可以「萃取」來源文件，以從來源資料中擷取文字。

![文件萃取階段](./media/cognitive-search-intro/document-cracking-phase-blowup.png "文件萃取")

 支援的來源包括 Azure Blob 儲存體、Azure 資料表儲存體、Azure SQL Database 和 Azure Cosmos DB。 以文字為基礎的內容可以從下列檔案類型中擷取：PDF、Word、PowerPoint、CSV 檔案。 如需完整的清單，請參閱[支援的格式](search-howto-indexing-azure-blob-storage.md#supported-document-formats)。

### <a name="cognitive-skills-and-enrichment-phase"></a>認知技能和擴充階段

擴充是經由*認知技能*執行不可部分完成作業而進行的。 例如，當您從 PDF 取得文字內容後，您可以套用實體辨識語言偵測或關鍵片語擷取，在您的索引中產生在來源中原本無法使用的新欄位。 技能的集合會統合用於您的管線中，我們稱之為*技能集*。  

![擴充階段](./media/cognitive-search-intro/enrichment-phase-blowup.png "擴充階段")

技能集以您所提供並連線至技能集的[預先定義的認知技能](cognitive-search-predefined-skills.md)或[自訂技能](cognitive-search-create-custom-skill-example.md)為基礎。 技能集可以是基本或高度複雜的，而且不但會決定處理類型，作業的順序也取決於它。 技能集加上定義為索引子一部分的欄位對應，即可完整指定擴充管線。 如需關於彙整前述各項組件的詳細資訊，請參閱[定義技能集](cognitive-search-defining-skillset.md)。

就內部而言，管線會產生擴充文件的集合。 您可以決定擴充文件的哪些部分應對應至搜尋索引中可編製索引的欄位。 例如，如果您套用關鍵片語擷取和實體辨識技能，則這些新欄位將會成為擴充文件的一部分，並且可對應至索引上的欄位。 若要深入了解輸入/輸出格式，請參閱[註解](cognitive-search-concept-annotations-syntax.md)。

### <a name="search-index-and-query-based-access"></a>搜尋索引和以查詢為基礎的存取

處理完成後，您即具有由擴充文件組成、可在 Azure 搜尋服務中進行全文檢索搜尋的搜尋主體。 [查詢索引](search-query-overview.md)是開發人員和使用者存取管線所產生之擴充內容的具體方式。 

![搜尋的索引圖示](./media/cognitive-search-intro/search-phase-blowup.png "搜尋的索引圖示")

索引就像任何其他您可能為 Azure 搜尋服務建立的項目一樣：您可以使用自訂分析器進行補強、叫用模糊搜尋查詢、新增篩選搜尋，或試用評分設定檔以調整搜尋結果。

索引是從定義連結至特定索引的欄位、屬性和其他建構的索引結構描述產生的，例如評分設定檔和同義字對應。 在定義並填入索引後，而且會擴展，您即可以累加方式編製索引，以收取新的和更新的來源文件。 特定的修改需要完整重建。 在結構描述設計趨於穩定之前，您應使用小型資料集。 如需詳細資訊，請參閱[如何重建索引](search-howto-reindex.md)。

<a name="feature-concepts"></a>

## <a name="key-features-and-concepts"></a>重要功能與概念

| 概念 | 說明| 連結 |
|---------|------------|-------|
| 技能集 | 包含技能集合的最上層具名資源。 技能集是擴充管線。 在索引子編製索引期間，即會叫用技能集。 | [定義技能集](cognitive-search-defining-skillset.md) |
| 認知技能 | 擴充管線中不可部分完成的轉換。 它通常是一種擷取或推斷結構的元件，因此能提高我們對輸入資料的了解。 輸出幾乎都是以文字為基礎，而處理則是自然語言處理，或是從影像輸入擷取或產生的文字的影像處理。 技能的輸出可以對應至索引中的欄位，或作為下游擴充的輸入。 技能可以是 Microsoft 預先定義並提供的，或是自訂的：由您建立和部署。 | [預先定義的技能](cognitive-search-predefined-skills.md) |
| 資料擷取 | 涵蓋多種形式的處理，但與認知搜尋有關，具名實體辨識技能最常用來從原本未提供具體資訊的來源擷取資料 (實體)。 | [具名實體辨識技能](cognitive-search-skill-named-entity-recognition.md)| 
| 影像處理 | 從影像推斷文字 (例如辨識地標的能力)，或從影像擷取文字。 常見的範例包括從掃描的文件 (JPEG) 檔案中挑取字元的 OCR，或是在包含路標的相片中辨識街道名稱。 | [影像分析技能](cognitive-search-skill-image-analysis.md)或 [OCR 技能](cognitive-search-skill-ocr.md)
| 自然語言處理 | 文字輸入的相關深入解析和資訊的文字處理。 語言偵測、情感分析和關鍵片語擷取都是屬於自然語言處理的技能。  | [關鍵片語擷取技能](cognitive-search-skill-keyphrases.md)、[語言偵測技能](cognitive-search-skill-language-detection.md)、[情感分析技能](cognitive-search-skill-sentiment.md) |
| 文件萃取 | 在索引編製期間從非文字來源擷取或建立文字內容的程序。 光學字元辨識 (OCR) 也是範例之一，但它通常指涉索引子核心功能，因為索引子會從應用程式檔案中擷取內容。 提供來源檔案位置的資料來源，與提供欄位對應的索引子定義，都是文件萃取的關鍵因素。 | 請參閱[索引子](search-indexer-overview.md)。 |
| 塑形 | 將文字片段合併成較大的結構，或者反向將較大的文字區塊分解成可管理的大小，以進行進一步的下游處理。 | [塑形器技能](cognitive-search-skill-shaper.md)、[文字合併技能](cognitive-search-skill-textmerger.md)、[文字分割技能](cognitive-search-skill-textsplit.md) |
| 擴充的文件 | 一個暫時性內部結構，無法在程式碼中直接存取。 在處理期間會產生擴充的文件，但只有最終的輸出會保存在搜尋索引中。 欄位對應將決定哪些資料元素會新增至索引。 | 請參閱[存取擴充的文件](cognitive-search-tutorial-blob.md#access-enriched-document)。 |
| 索引器 |  一種編目程式，可從外部資料來源擷取可搜尋的資料和中繼資料，並根據索引和資料來源之間的欄位對欄位對應填入索引，以進行文件萃取。 在進行認知搜尋擴充時，索引子會叫用技能集，並且包含將擴充輸出與索引中的目標欄位產生關聯的欄位對應。 索引子定義中包含管線作業的所有指示和參考，當您執行索引子時，即會叫用管線。 | [索引子](search-indexer-overview.md) |
| 資料來源  | 索引子用來與 Azure 上支援的外部資料來源類型連線的物件。 | 請參閱[索引子](search-indexer-overview.md)。 |
| 索引 | 從定義欄位結構和使用方式的索引結構描述建置，並保存在 Azure 搜尋服務中的搜尋主體。 | [Azure 搜尋服務中的索引](search-what-is-an-index.md) | 


## <a name="where-do-i-start"></a>我該從哪裡開始？

**步驟 1：在提供 API 的區域中建立搜尋服務** 

+ 美國中南部
+ 西歐

**步驟 2：實際操作工作流程加以熟悉**

+ [快速入門 (入口網站)](cognitive-search-quickstart-blob.md)
+ [教學課程 (HTTP 要求)](cognitive-search-tutorial-blob.md)
+ [範例自訂技能 (C#)](cognitive-search-create-custom-skill-example.md)

**步驟 3：檢閱 API (僅限 REST)**

目前僅提供 REST API。 所有要求均應使用 `api-version=2017-11-11-Preview`。 請使用下列 API 來建立認知搜尋解決方案。 針對認知搜尋僅新增或擴充了兩個 API。 其他 API 的語法與公開上市的版本相同。

| REST API | 說明 |
|-----|-------------|
| [建立資料來源](https://docs.microsoft.com/rest/api/searchservice/create-data-source)  | 一項資源，用以識別提供來源資料以建立擴充文件的外部資料來源。  |
| [建立技能集 (api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 在索引編製期間負責對擴充管線中使用的[預先定義的技能](cognitive-search-predefined-skills.md)和[自訂認知技能](cognitive-search-custom-skill-interface.md)進行用法協調的資源。 |
| [建立索引](https://docs.microsoft.com/rest/api/searchservice/create-index)  | 表示 Azure 搜尋服務索引的結構描述。 索引中與來源資料中的欄位或在擴充階段產生的欄位 (例如，實體辨識所建立之組織名稱的欄位) 相對應的欄位。 |
| [建立索引子 (api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)  | 一項資源，用以定義在索引編製期間所使用的元件：包括資料來源、技能集、來源和中繼資料結構與目標索引的欄位關聯性，以及索引本身。 執行索引子是擷取和擴充資料的觸發程序。 輸出是以索引結構描述為基礎、以來源資料填入，並透過技能集進行擴充的搜尋主體。  |

**檢查清單：一般工作流程**

1. 將您的 Azure 來源資料分成代表性範例子集。 索引編製會從小型、具代表性的資料集展開，然後再隨著解決方案的成熟而逐漸補強。

1. 在 Azure 搜尋服務中建立[資料來源物件](https://docs.microsoft.com/rest/api/searchservice/create-data-source)，以提供用來擷取資料的連接字串。

1. 透過擴充步驟建立[技能集](https://docs.microsoft.com/rest/api/searchservice/create-skillset)。

1. 定義[索引結構描述](https://docs.microsoft.com/rest/api/searchservice/create-index)。 *欄位*集合包含來源資料中的欄位。 您也應該清除多餘的欄位，以保存為擴充期間建立的內容產生的值。

1. 定義參考資料來源、技能集和索引的[索引子](https://docs.microsoft.com/rest/api/searchservice/create-skillset)。

1. 在索引子內新增 *outputFieldMappings*。 這部分會將技能集的輸出 (在步驟 3 中) 對應至索引結構描述中的輸入欄位 (在步驟 4 中)。

1. 傳送您剛剛建立的*建立索引子*要求 (在要求本文中具有索引子定義的 POST 要求)，以表示 Azure 搜尋服務中的索引子。 此步驟說明您叫用管線以執行索引子的方式。

1. 執行查詢以評估結果，並修改程式碼以更新技能集、結構描述或索引子組態。

1. 在[重設索引子](search-howto-reindex.md)後重建管線。

如需關於特定問題的詳細資訊，請參閱[疑難排解提示](cognitive-search-concept-troubleshooting.md)。

## <a name="next-steps"></a>後續步驟

+ [認知搜尋文件](cognitive-search-resources-documentation.md)
+ [快速入門：在入口網站中逐步試用認知搜尋](cognitive-search-quickstart-blob.md)
+ [教學課程：了解認知搜尋 API](cognitive-search-tutorial-blob.md)
