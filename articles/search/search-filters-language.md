---
title: Azure 搜尋服務中的語言篩選條件 | Microsoft Docs
description: 依使用者安全性身分識別、語言、地理位置或數值來篩選準則，以縮減在 Azure 搜尋服務 (Microsoft Azure 上裝載的雲端搜尋服務) 中查詢的搜尋結果。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.workload: search
ms.date: 10/23/2017
ms.author: heidist
ms.openlocfilehash: 6d7fa7ab6db1fe9f8e2d1530c2917f4716a38079
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31790622"
---
# <a name="how-to-filter-by-language-in-azure-search"></a>如何在 Azure 搜尋服務中依語言進行篩選 

在多國語言的搜尋應用程式中，有一個關鍵需求是能夠以使用者自己的語言搜尋和擷取結果。 在 Azure 搜尋服務中，符合多國語言應用程式之語言需求的一種方式是，建立一系列專門用來以特定語言儲存字串的欄位，然後在查詢期間限制只對那些欄位進行全文檢索搜尋。

要求上的查詢參數可用來設定搜尋作業的範圍，接著如果欄位未提供與您想要傳遞之搜尋體驗相容的內容，則可調整所有這類欄位的結果。

| 參數 | 目的 |
|-----------|--------------|
| **searchFields** | 限制只對具名欄位清單進行全文檢索搜尋。 |
| **$select** | 調整回應，只包含您指定的欄位。 預設會傳回所有可擷取的欄位。 **$Select** 參數可讓您選擇要傳回哪些欄位。 |

這個技術的成功與否，取決於欄位內容的完整性。 Azure 搜尋服務不會翻譯字串或執行語言偵測。 必須由您來確定欄位包含預期的字串。

## <a name="define-fields-for-content-in-different-languages"></a>定義適用於不同語言內容的欄位

在 Azure 搜尋服務中，查詢是以單一索引為目標。 想要在單一搜尋體驗中提供特定語言之字串的開發人員，通常會定義專用欄位來儲存值：一個適用於英文字串的欄位、一個適用於法文，依此類推。 

在我們的範例中 (包括如下所示的[不動產範例](search-get-started-portal.md))，您可能會看到類似下列螢幕擷取畫面的欄位定義。 請注意此範例如何在這個索引中顯示欄位的語言分析器指派。 在與制訂來處理目標語言之語言規則的分析器配對時，包含字串的欄位在全文檢索搜尋中的執行效能較佳。

  ![](./media/search-filters-language/lang-fields.png)

> [!Note]
> 如需利用語言分析器顯示欄位定義的程式碼範例，請參閱[定義索引 (.NET)](https://docs.microsoft.com/azure/search/search-create-index-dotnet#define-your-azure-search-index) 和[定義索引 (REST)](https://docs.microsoft.com/azure/search/search-create-index-rest-api#define-your-azure-search-index-using-well-formed-json)。

## <a name="build-and-load-an-index"></a>建置和載入索引

有個中繼 (且可能明顯) 的步驟是，您必須先[建立並填入索引](https://docs.microsoft.com/azure/search/search-create-index-dotnet#create-the-index)，之後才能編寫查詢。 我們基於完整性而在這裡提及此步驟。 判斷索引是否可用的方法之一是在[入口網站](https://portal.azure.com)中檢查索引清單。

## <a name="constrain-the-query-and-trim-results"></a>限制查詢與調整結果

查詢上的參數可用來限制搜尋特定欄位，然後調整任何對您案例不是很有助益之欄位的結果。 假設目標為限制搜尋包含法文字串的欄位，則您會使用 **searchFields**，將目標設為查詢包含該語言之字串的欄位。 

根據預設，搜尋會傳回所有標記為可擷取的欄位。 因此，針對您想要提供的特定語言搜尋體驗，您可以排除不符合的欄位。 具體來說，如果您限制搜尋具法文字串的欄位，則可能想要從結果中排除具英文字串的欄位。 使用 **$select** 查詢參數，可讓您控制要將哪些欄位傳回呼叫應用程式。

```csharp
parameters =
    new SearchParameters()
    {
        searchFields = "description_fr" 
        Select = new[] { "description_fr"  }
    };
```
> [!Note]
> 雖然查詢中沒有 $filter 引數，但這個使用案例與篩選概念具有強烈的關聯性，因此，我們將它作為篩選案例。

## <a name="see-also"></a>另請參閱

+ [Azure 搜尋服務中的篩選條件](search-filters.md)
+ [語言分析器](https://docs.microsoft.com/rest/api/searchservice/language-support)
+ [全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md)
+ [搜尋文件 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents)

