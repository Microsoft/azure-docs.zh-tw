---
title: Azure 搜尋服務中的 Facet 篩選條件 | Microsoft Docs
description: 依使用者安全性身分識別、語言、地理位置或數值來篩選準則，以縮減在 Azure 搜尋服務 (Microsoft Azure 上裝載的雲端搜尋服務) 中查詢的搜尋結果。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 10/13/2017
ms.author: heidist
ms.openlocfilehash: 3f2cfea52d3c3f4bfc75364d0662a4218219152d
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31792404"
---
# <a name="how-to-build-a-facet-filter-in-azure-search"></a>如何在 Azure 搜尋服務中建置 Facet 篩選條件 

多面向導覽適用於在搜尋應用程式中對查詢結果進行自我導向的篩選，您的應用程式可以在其中提供 UI 控制項來設定文件群組 (例如，類別或品牌) 的搜尋範圍，而 Azure 搜尋服務會提供資料結構來支持該體驗。 本文將快速檢閱用於建立多面向導覽結構以支持您想要提供之搜尋體驗的基本步驟。 

> [!div class="checklist"]
> * 選擇用以篩選和設定 Facet 的欄位
> * 在欄位上設定屬性
> * 建置索引和載入資料
> * 將 Facet 篩選條件新增至查詢
> * 處理結果

Facet 是動態且會在查詢中傳回。 搜尋回應會為它們提供用來瀏覽結果的 Facet 類別。 如果您不熟悉 Facet，下列範例是多面向導覽結構的說明。

  ![](./media/search-filters-facets/facet-nav.png)

多面向導覽的新手，想要更詳細說明嗎？ 請參閱[如何在 Azure 搜尋服務中實作多面向導覽](search-faceted-navigation.md)。

## <a name="choose-fields"></a>選擇欄位

Facet 可透過單一值欄位以及集合來計算。 最適合在多面向導覽中使用的欄位具有較低的基數：在您的搜尋主體 (例如，色彩、國家/地區或品牌名稱的清單) 內重複出現於文件各處的少數相異值。 

當您建立索引時，可藉由將下列屬性設為 TRUE，逐欄位啟用 Facet：`filterable`、`facetable`。 只有可篩選的欄位可以面向化。

任何可能用於多面向導覽的[欄位類型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types)都會標記為 "facetable"：

+ Edm.String
+ Edm.DateTimeOffset
+ Edm.Boolean
+ Edm.Collections
+ 數值欄位類型：Edm.Int32、Edm.Int64、Edm.Double

您無法在多面向導覽中使用 Edm.GeographyPoint。 Facet 是從人類可讀取的文字或數字建構的。 因此，不支援針對地理座標使用 Facet。 您必須依位置來為城市或區域欄位設定 Facet。

## <a name="set-attributes"></a>設定屬性

控制如何使用欄位的索引屬性會新增至索引中的個別欄位定義。 在下列範例中，具有較低基數且適用於 Facet 的欄位包括：類別 (hotel、motel、hostel)、便利設施及評等。 

在 .NET API 中，必須明確設定篩選屬性。 在 REST API 中，預設會啟用 Facet 和篩選，這表示當您想要關閉屬性時，只需明確地設定它們即可。 雖然這在技術上並非必要，但我們會基於教學目的在下列 REST 範例中顯示屬性。 

> [!Tip]
> 適用於效能和儲存體最佳化的最佳做法是，針對永遠不應作為 Facet 使用的欄位關閉 Facet。 尤其是適用於單一值的字串欄位 (例如識別碼或產品名稱)，應該將它們設定為 "Facetable": false，以防止意外 (且不應) 被用於多面向導覽。


```http
{
    "name": "hotels",  
    "fields": [
        {"name": "hotelId", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
        {"name": "baseRate", "type": "Edm.Double"},
        {"name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false},
        {"name": "description_fr", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
        {"name": "hotelName", "type": "Edm.String", "facetable": false},
        {"name": "category", "type": "Edm.String", "filterable": true, "facetable": true},
        {"name": "tags", "type": "Collection(Edm.String)", "filterable": true, "facetable": true},
        {"name": "parkingIncluded", "type": "Edm.Boolean",  "filterable": true, "facetable": true, "sortable": false},
        {"name": "smokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true, "sortable": false},
        {"name": "lastRenovationDate", "type": "Edm.DateTimeOffset"},
        {"name": "rating", "type": "Edm.Int32", "filterable": true, "facetable": true},
        {"name": "location", "type": "Edm.GeographyPoint"}
    ]
}
```

> [!Note]
> 此索引定義是從[使用 REST API 建立 Azure 搜尋服務索引](https://docs.microsoft.com/azure/search/search-create-index-rest-api)複製。 在欄位定義中，除了外表差異之外，其餘的都一樣。 可篩選項和可 Facet 項屬性已明確新增於 category、tags、parkingIncluded、smokingAllowed 及 rating 欄位中。 在實務上，您可以免費取得 Edm.String、Edm.Boolean 與 Edm.Int32 欄位類型上的可篩選項和可 Facet 項。 

## <a name="build-and-load-an-index"></a>建置和載入索引

有個中繼 (且可能明顯) 的步驟是，您必須先[建立並填入索引](https://docs.microsoft.com/azure/search/search-create-index-dotnet#create-the-index)，之後才能編寫查詢。 我們基於完整性而在這裡提及此步驟。 判斷索引是否可用的方法之一是在[入口網站](https://portal.azure.com)中檢查索引清單。

## <a name="add-facet-filters-to-a-query"></a>將 Facet 篩選條件新增至查詢

在應用程式的程式碼中，建構一個查詢，以指定有效查詢的所有部分，包括搜尋運算式、Facet、篩選條件、評分設定檔 (制訂要求使用的任何項目)。 下列範例會建置一個要求，根據住宿、評等及其他便利設施的類型來建立多面向導覽。

```csharp
SearchParameters sp = new SearchParameters()
{
  ...
  // Add facets
  Facets = new List<String>() { "category", "rating", "parkingIncluded", "smokingAllowed" },
};
```

### <a name="return-filtered-results-on-click-events"></a>傳回 Click 事件的篩選結果

篩選條件運算式會處理面向值的點擊事件。 指定一個類別 Facet，按一下 "motel" 類別是透過選取該類型之住宿的 `$filter` 運算式來實作。 當使用者按一下 [motels] 來表示僅應顯示汽車旅館，應用程式傳送的下一個查詢就會包含 $filter=category eq ‘motels’。

如果使用者從類別 Facet 選取值，下列程式碼片段就會將類別 新增至篩選條件。

```csharp
if (categoryFacet != "")
  filter = "category eq '" + categoryFacet + "'";
```
使用 REST API，要求會以 `$filter=category eq 'c1'` 形式來聯結。 若要將類別設定為多重值欄位，請使用下列語法：`$filter=category/any(c: c eq 'c1')`

## <a name="tips-and-workarounds"></a>祕訣和因應措施

### <a name="initialize-a-page-with-facets-in-place"></a>就地將具有 Facet 的頁面初始化

如果您想要就地將具有 Facet 的頁面初始化，則可以在頁面初始化過程中傳送查詢，以植入具有最初 Facet 結構的頁面。

### <a name="preserve-a-facet-navigation-structure-asynchronously-of-filtered-results"></a>以非同步方式保留篩選結果的多面向導覽結構

在 Azure 搜尋服務中使用多面向導覽的其中一個挑戰是 Facet 只存在於目前的結果中。 實務上常會保留一組靜態的 Facet，讓使用者可以反向瀏覽，回溯步驟以透過搜尋內容探索替代路徑。 

儘管這是一個常見使用案例，但並不是多面向導覽目前預設提供的功能。 需要靜態 Facet 的開發人員通常可藉由發出兩個篩選查詢來解決限制：一個將範圍設為結果，另一個則基於導覽目的用來建立靜態的 Facet 清單。

## <a name="see-also"></a>另請參閱

+ [Azure 搜尋服務中的篩選條件](search-filters.md)
+ [建立索引 REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)
+ [搜尋文件 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents)

