---
title: 重建 Azure 搜尋服務索引或重新整理可搜尋內容 | Microsoft Docs
description: 藉由完整重建或部分累加索引編製來新增元素、更新現有的元素或文件或刪除過時的文件，以重新整理 Azure 搜尋服務索引。
services: search
author: HeidiSteen
manager: cgronlun
ms.service: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: heidist
ms.openlocfilehash: 374e7601169647f0eb7d3a214cf15567b7b11090
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "34641419"
---
# <a name="how-to-rebuild-an-azure-search-index"></a>如何重建 Azure 搜尋服務索引

重建索引可變更其結構，改變索引在 Azure 搜尋服務中的實際表示法。 相反地，重新整理索引則是僅限於內容的更新，會從參與的外部資料來源收取最新的變更。 本文將指示如何在結構和實質上同時更新索引。

進行索引更新時需要具備服務層級的讀寫權限。 您可以用程式設計方式，使用指定更新選項的參數呼叫 REST 或 .NET API，為內容進行完整重建或累加的索引編製。 

一般而言，索引的更新是隨需的。 不過，對於使用來源特定[索引子](search-indexer-overview.md)進行填入的索引，您可以使用內建的排程器。 排程器可支援每 15 分鐘一次的文件重新整理，而且您可以依需求設定任意間隔和模式。 較快的重新整理速率需要以手動方式推送索引更新，這有可能透過交易的重複寫入來執行，而同時更新外部資料來源和 Azure 搜尋服務索引。

## <a name="full-rebuilds"></a>完整重建

對於許多類型的更新，都必須執行完整重建。 完整重建是指刪除索引 (包括資料和中繼資料)，接著再從外部資料來源重新填入索引。 請以程式設計方式[刪除](https://docs.microsoft.com/rest/api/searchservice/delete-index)、[建立](https://docs.microsoft.com/rest/api/searchservice/create-index)並[重新載入](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)索引，而加以重建。 

重建之後請留意，如果您已測試過查詢模式和評分設定檔，且基礎內容已變更，則可以預期查詢結果將會有變化。

## <a name="when-to-rebuild"></a>重建時機

在作用中的開發期間，如果索引結構描述處於變動狀態，則應規劃頻繁的完整重建。

| 修改 | 重建狀態|
|--------------|---------------|
| 變更欄位名稱、資料類型或其[索引屬性](https://docs.microsoft.com/rest/api/searchservice/create-index) | 變更欄位定義通常會造成重建的負面影響，但下列[索引屬性](https://docs.microsoft.com/rest/api/searchservice/create-index)除外：Retrievable、SearchAnalyzer、SynonymMaps。 您可以將 Retrievable、SearchAnalyzer 和 SynonymMaps 屬性新增至現有的欄位，而無須重建其索引。|
| 新增欄位 | 重建並沒有嚴格的需求。 對於已編製索引的現有文件，會為其新欄位提供 Null 值。 在未來重新編製索引時，來源資料中的值會取代 Azure 搜尋服務所加入的 Null。 |
| 刪除欄位 | 您無法直接從 Azure 搜尋服務索引中刪除欄位。 您應讓應用程式忽略「已刪除的」欄位，以避免使用該欄位。 實際上，在您下次使用省略問題欄位的結構描述重建索引之前，欄位定義和內容都會保留在索引中。|

> [!Note]
> 如果您切換層，也需要重建。 如果您在某個時間點決定提高容量，將無法使用就地升級。 您必須在新的容量點建立新的服務，且必須為新服務從頭建置索引。 

## <a name="partial-or-incremental-indexing"></a>部分或累加式索引編製

在索引進入生產環境後，焦點將會移轉至累加式索引編製，這通常不會沒有明顯的服務中斷。 部分或累加式索引編製是僅限內容的工作負載，會同步處理搜尋索引的內容，以反映參與的資料來源內容所處的狀態。 在來源中新增或刪除的文件，將會在索引中新增或刪除。 在程式碼中，呼叫[新增、更新或刪除文件](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)作業或 .NET 的等同作業。

> [!Note]
> 使用會搜耙外部資料來源的索引子時，會使用來源系統中的變更追蹤機制進行累加式索引編製。 對於 [Azure Blob 儲存體](search-howto-indexing-azure-blob-storage.md#incremental-indexing-and-deletion-detection)，會使用 `lastModified` 欄位。 在 [Azure 資料表儲存體](search-howto-indexing-azure-tables.md#incremental-indexing-and-deletion-detection)中，`timestamp` 具有相同的用途。 同樣地，[Azure SQL Database 索引子](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#capture-new-changed-and-deleted-rows)和 [Azure Cosmos DB 索引子](search-howto-index-cosmosdb.md#indexing-changed-documents)都具有標示資料列更新的欄位。 如需索引子的詳細資訊，請參閱[索引子概觀](search-indexer-overview.md)。


## <a name="see-also"></a>另請參閱

+ [索引子概觀](search-indexer-overview.md)
+ [大規模索引大型資料集](search-howto-large-index.md)
+ [在入口網站中編製索引](search-import-data-portal.md)
+ [Azure SQL Database 索引子](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
+ [Azure Cosmos DB 索引子](search-howto-index-cosmosdb.md)
+ [Azure Blob 儲存體索引子](search-howto-indexing-azure-blob-storage.md)
+ [Azure 資料表儲存體索引子](search-howto-indexing-azure-tables.md)
+ [Azure 搜尋服務中的安全性](search-security-overview.md)