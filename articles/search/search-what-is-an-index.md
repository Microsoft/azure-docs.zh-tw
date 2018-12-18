---
title: Azure 搜尋服務中的索引 | Microsoft Docs
description: 深入了解 Azure 搜尋服務中的索引概念以及索引的使用方式。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.topic: conceptual
ms.date: 11/08/2017
ms.openlocfilehash: cc2c56d4463203aa2561e820540acdfff651d80a
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31793397"
---
# <a name="indexes-in-azure-search"></a>Azure 搜尋服務中的索引
> [!div class="op_single_selector"]
> * [概觀](search-what-is-an-index.md)
> * [入口網站](search-create-index-portal.md)
> * [.NET](search-create-index-dotnet.md)
> * [REST](search-create-index-rest-api.md)
> 
> 

在 Azure 搜尋服務中，索引是對 Azure 搜尋服務所使用的文件和其他建構所做的持續性儲存。 文件是索引中單一單位的可搜尋資料。 例如，電子商務零售商可能會有儲存了每個銷售項目的文件、新聞組織可能會有儲存了每篇文章的文件，類似情況不一而足。 對應這些概念到更熟悉的資料庫同等項目：索引在概念上類似於資料表，而文件大致上相當於資料表中的資料列。

當您新增/上傳文件並提交搜尋查詢到 Azure 搜尋服務時，您是將要求提交到搜尋服務中的特定索引。

## <a name="field-types-and-attributes-in-an-azure-search-index"></a>Azure 搜尋服務索引中的欄位類型和屬性
您在定義結構描述時必須指定索引中每個欄位的名稱、類型和屬性。 欄位類型可分類該欄位中儲存的資料。 個別欄位上設定的屬性可指定使用欄位的方式。 下列幾個資料表列舉您可以指定的類型和屬性。

### <a name="field-types"></a>欄位類型
| 類型 | 說明 |
| --- | --- |
| *Edm.String* |可選擇性予以 Token 化以供進行全文檢索搜尋 (斷字、詞幹分析等) 的文字。 |
| *Collection(Edm.String)* |可選擇性予以 Token 化以供進行全文檢索搜尋的字串清單。 理論上，集合中的項目數沒有上限，但集合的承載大小有 16 MB 的上限。 |
| *Edm.Boolean* |包含 true/false 值。 |
| *Edm.Int32* |32 位元整數值。 |
| *Edm.Int64* |64 位元整數值。 |
| *Edm.Double* |雙精度數值資料。 |
| *Edm.DateTimeOffset* |以 OData V4 格式 (例如 `yyyy-MM-ddTHH:mm:ss.fffZ` 或 `yyyy-MM-ddTHH:mm:ss.fff[+/-]HH:mm`) 表示的日期時間值。 |
| *Edm.GeographyPoint* |代表地球上地理位置的一點。 |

您可以在這裡找到有關 Azure 搜尋服務 [支援的資料類型](https://docs.microsoft.com/rest/api/searchservice/Supported-data-types)的詳細資訊。

### <a name="field-attributes"></a>欄位屬性
| 屬性 | 說明 |
| --- | --- |
| *Key* |字串，提供每一份文件的唯一識別碼，用於查閱文件。 每個索引必須有一個索引鍵。 只有一個欄位可以做為索引鍵，而且其類型必須設定為 Edm.String。 |
| *Retrievable* |指定搜尋結果中是否可傳回某欄位。 |
| *Filterable* |允許欄位用於篩選查詢。 |
| *Sortable* |允許查詢使用此欄位排序搜尋結果。 |
| *Facetable* |允許欄位用於使用者自我引導篩選的 [多面向導覽](search-faceted-navigation.md) 結構中。 通常，欄位若包含您可以用來將多份文件群組在一起的重複值 (例如，落在單一品牌或服務類別目錄下的多份文件)，最適合做為 Facet。 |
| *Searchable* |將欄位標記為可供全文檢索。 |

您可以在這裡找到有關 Azure 搜尋服務 [索引屬性](https://docs.microsoft.com/rest/api/searchservice/Create-Index)的詳細資訊。

## <a name="guidance-for-defining-an-index-schema"></a>適用於定義索引結構描述的指引
當設計索引時，請在規劃階段花時間仔細考量每個決策。 由於必須為每個欄位指派 [適當屬性](https://docs.microsoft.com/rest/api/searchservice/Create-Index)，因此在設計索引時，請務必牢記搜尋服務使用者體驗和商務需求。 在索引部署後若要加以變更，將牽涉到重建和重新載入資料。

如果資料儲存體需求在之後有所改變，您可以藉由新增或移除資料分割來增加或減少容量。 如需詳細資料，請參閱[在 Azure 中管理您的搜尋服務](search-manage.md)或[服務限制](search-limits-quotas-capacity.md)。

