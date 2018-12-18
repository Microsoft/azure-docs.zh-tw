---
title: 在 Azure 搜尋服務中編製多種語言的索引 | Microsoft Docs
description: Azure 搜尋服務支援 56 種語言，運用來自 Lucene 的語言分析器和來自 Microsoft 的自然語言處理技術。
author: yahnoosh
manager: jlembicz
services: search
ms.service: search
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: jlembicz
ms.openlocfilehash: 38f93f5415282d2f976d9f3acc2b0a7aeead6c3d
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35767803"
---
# <a name="create-an-index-for-documents-in-multiple-languages-in-azure-search"></a>在 Azure 搜尋服務中建立多種語言的文件索引
> [!div class="op_single_selector"]
>
> * [入口網站](search-language-support.md)
> * [REST](https://msdn.microsoft.com/library/azure/dn879793.aspx)
> * [.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.search.models.analyzername.aspx)
>
>

揭開語言分析器的強大功能，就如同在索引定義中的可搜尋欄位上設定一個屬性一樣簡單。 您現在可以在入口網站中執行這個步驟。

以下是 Azure 搜尋服務的 Azure 入口網站刀鋒視窗螢幕擷取畫面，可供使用者定義索引結構描述。 在這個刀鋒視窗中，使用者可以建立所有欄位並設定各欄位的分析器屬性。

> [!IMPORTANT]
> 如同在從頭建立新索引時，或將新欄位加入至現有索引時，您只能在欄位定義期間設定語言分析器。 確保您建立欄位時完全指定所有的屬性 (包括分析器)。 儲存變更後，您將無法編輯屬性或變更分析器類型。
>
>

## <a name="define-a-new-field-definition"></a>定義新的欄位定義
1. 登入 [Azure 入口網站](https://portal.azure.com) 並開啟您的搜尋服務的服務刀鋒視窗。
2. 在服務儀表板頂端的命令列中按一下 [新增索引]  即可開始新的索引，或開啟現有索引以在您加入至現有索引的新欄位上設定分析器。
3. [欄位] 刀鋒視窗隨即出現，顯示可供您定義索引結構描述的選項，包括用於選擇語言分析器的 [分析器] 索引標籤。
4. 在 [欄位] 中，藉由提供名稱、選擇資料類型，以及設定屬性來開始欄位定義，進而將欄位標示為可全文檢索搜尋、可在搜尋結果中擷取，可用於 facet 導覽結構中、可排序等等。
5. 在移到下一個欄位之前，開啟 [分析器]  索引標籤。

![][1]
*若要選取分析器，請按一下 [欄位] 刀鋒視窗上的 [分析器] 索引標籤*

## <a name="choose-an-analyzer"></a>選擇分析器
1. 捲動以尋找您要定義的欄位。
2. 如果您沒有將此欄位標示為搜尋，請立即按一下此核取方塊將其標示為 **Searchable**。
3. 按一下 [分析器] 區域以顯示可用的分析器清單。
4. 選擇您要使用的分析器。

![][2]
*為每個欄位選取其中一個支援的分析器*

根據預設，所有可搜尋的欄位都可使用 [標準 Lucene 分析器](http://lucene.apache.org/core/4_10_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) (語言中立)。 若要檢視支援分析器的完整清單，請參閱 [Azure 搜尋服務的語言支援](https://msdn.microsoft.com/library/azure/dn879793.aspx)。

一旦針對某個欄位選取語言分析器，它將用於該欄位的每個索引和搜尋要求。 使用不同的分析器針對多個欄位發出查詢時，查詢將由每個欄位的右分析器獨立處理。

許多 Web 和行動應用程式使用不同的語言來服務世界各地的使用者。 有可能藉由為每種支援語言建立一個欄位，以定義這類案例的索引。

![][3]
*每種支援語言都有一個描述欄位的索引定義*

如果已知發出查詢之代理程式的語言，則可使用 **searchFields** 查詢參數將搜尋要求的範圍限制為特定欄位。 下列查詢只會針對波蘭文描述發出：

`https://[service name].search.windows.net/indexes/[index name]/docs?search=darmowy&searchFields=description_pl&api-version=2017-11-11`

您可以使用 [搜尋總管]  貼上類似上述的查詢，便可從入口網站查詢您的索引。 從服務刀鋒視窗的命令列可以存取搜尋總管。 如需詳細資料，請參閱 [在入口網站查詢 Azure 搜尋服務索引](search-explorer.md) 。

有時候不知道發出查詢之代理程式的語言，在此情況下，可以針對所有欄位同時發出查詢。 如有需要，可以使用 [評分設定檔](https://msdn.microsoft.com/library/azure/dn798928.aspx)來定義特定語言之結果的喜好設定。 在下面範例中，相對於波蘭文和法文的相符項目，在英文描述中找到的相符項目會有較高的評分：

    "scoringProfiles": [
      {
        "name": "englishFirst",
        "text": {
          "weights": { "description_en": 2 }
        }
      }
    ]

`https://[service name].search.windows.net/indexes/[index name]/docs?search=Microsoft&scoringProfile=englishFirst&api-version=2017-11-11`

如果您是 .NET 開發人員，請注意您可以使用 [Azure 搜尋服務 .NET SDK](http://www.nuget.org/packages/Microsoft.Azure.Search)來設定語言分析器。 最新版本包含對 Microsoft 語言分析器的支援。

<!-- Image References -->
[1]: ./media/search-language-support/AnalyzerTab.png
[2]: ./media/search-language-support/SelectAnalyzer.png
[3]: ./media/search-language-support/IndexDefinition.png
