---
title: 設計用於查詢的 Azure 儲存體資料表 | Microsoft Docs
description: 設計 Azure 表格儲存體中用於查詢的資料表。
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: 8b4ae066edc1c62c25762b5c6feebce1ecfff5a2
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521535"
---
# <a name="design-for-querying"></a>查詢的設計
資料表服務方案可以是讀取密集、寫入密集或兩者混合的方案。 本文主要說明您在設計表格服務以有效地支援讀取作業時應謹記在心的事項。 一般而言，支援有效讀取作業的設計，也可兼顧寫入作業的效率。 不過，設計支援寫入作業時還有其他考量必須牢記在心，這將在[資料修改的設計](table-storage-design-for-modification.md)一文中說明這些考量。

要設計資料表服務方案，讓您能夠有效率地讀取資料，您首先可以自問：「我的應用程式需要執行何種查詢以從資料表服務中擷取它所需要的資料？」  

> [!NOTE]
> 使用資料表服務時，事先做出正確的設計是很重要的，因為事後要更改不僅困難，也很昂貴。 例如，在關聯式資料庫中，通常只要藉由將索引新增至現有的資料庫，就可以解決效能問題：這對於資料表服務是行不通的。  
> 
> 

本節主要討論您在設計查詢資料表時所須解決的重要問題。 本節所涵蓋的主題包括：

* [您選擇的 PartitionKey 和 RowKey 對查詢效能有何影響](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
* [選擇適當的 PartitionKey](#choosing-an-appropriate-partitionkey)
* [最佳化資料表服務的查詢](#optimizing-queries-for-the-table-service)
* [在表格服務中排序資料](#sorting-data-in-the-table-service)

## <a name="how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance"></a>您選擇的 PartitionKey 和 RowKey 對查詢效能有何影響
下列範例假設表格服務會以下列結構儲存員工實體 (為求簡潔，大部分的範例皆省略 **Timestamp** 屬性)：  

| *資料行名稱* | *資料類型* |
| --- | --- |
| **PartitionKey** (部門名稱) |字串 |
| **RowKey** (員工識別碼) |字串 |
| **名字** |字串 |
| **姓氏** |字串 |
| **年齡** |整數  |
| **EmailAddress** |字串 |

＜[Azure 表格儲存體概觀](table-storage-overview.md) ＞一文將說明某些對查詢設計有直接影響的重要 Azure 表格服務功能。 這些功能產生了設計資料表服務查詢的一般指導方針。 請注意，下列範例中使用的篩選語法來自於表格服務 REST API，如需詳細資訊，請參閱 [查詢實體](https://docs.microsoft.com/rest/api/storageservices/Query-Entities)。  

* ***點查詢***是使用上最有效率的查閱，建議用於高容量查閱或只能容許最低延遲的查閱。 這類查詢使用索引尋找個別實體的效率極高，方法是同時指定 **PartitionKey** 和 **RowKey** 值。 例如：$filter=(PartitionKey eq 'Sales') and (RowKey eq '2')  
* 次佳的是***範圍查詢***，它使用 **PartitionKey**，並篩選特定範圍的 **RowKey** 值，以傳回多個實體。 **PartitionKey** 值會識別特定的分割，而 **RowKey** 值會識別該分割中實體的子集。 例如：$filter=PartitionKey eq 'Sales' and RowKey ge 'S' and RowKey lt 'T'  
* 再其次是***分割掃描***，它使用 **PartitionKey**，並篩選另一個非索引鍵的，可傳回多個實體。 **PartitionKey** 值會識別特定的分割，而屬性值會選取該分割中實體的子集。 例如：$filter=PartitionKey eq 'Sales' and LastName eq 'Smith'  
* ***資料表掃描***不包含 **PartitionKey**，且效率極差，因為它會依序在所有組成資料表的分割中搜尋是否有任何相符的實體。 無論您的篩選是否使用 **RowKey**，它都會執行資料表掃描。 例如：$filter=LastName eq 'Jones'  
* 傳回多個實體的查詢，在傳回時會以 **PartitionKey** 和 **RowKey** 順序排序。 若要避免重新排序用戶端中的實體，請選擇定義最常見的排序次序的 **RowKey** 。  

請注意，使用 "**or**" 指定以 **RowKey** 值為基礎的篩選條件，會產生資料分割掃描且不被當作範圍查詢。 因此，您應該避免會使用下列篩選條件的搜尋：例如 $filter=PartitionKey eq 'Sales' and (RowKey eq '121' or RowKey eq '322')  

如需使用儲存體用戶端程式庫執行有效率查詢的用戶端程式碼範例，請參閱：  

* [使用儲存體用戶端程式庫執行點查詢](table-storage-design-patterns.md#executing-a-point-query-using-the-storage-client-library)
* [使用 LINQ 擷取多個實體](table-storage-design-patterns.md#retrieving-multiple-entities-using-linq)
* [伺服器端預測](table-storage-design-patterns.md#server-side-projection)  

如需可對儲存在相同資料表中的多個實體類型進行處理的用戶端程式碼範例，請參閱：  

* [使用異質性實體類型](table-storage-design-patterns.md#working-with-heterogeneous-entity-types)  

## <a name="choosing-an-appropriate-partitionkey"></a>選擇適當的 PartitionKey
您選擇的 **PartitionKey** 應兼顧 EGT 的啟用 (以確保一致性) 和將實體分散到多個分割區 (以確保可調整的解決方案) 的需求。  

就一個極端而言，您可以在單一磁碟分割儲存您的實體，但這可能會限制方案的延展性，且會妨礙資料表服務的負載平衡要求。 就另一個極端而言，您可以為每個資料分割儲存一個實體以達到高擴充性，並且讓資料表服務可進行負載平衡要求，但這會讓您無法使用實體群組交易。  

理想的 **PartitionKey** 應可讓您使用有效率的查詢，而且具有足夠的資料分割，以確保您的解決方案可進行調整。 一般而言，您會發現您的實體將具有適當的屬性，可將您的實體分散到足夠的磁碟分割。

> [!NOTE]
> 例如，在儲存使用者或員工相關資訊的系統中，UserID 可以是一個良好的 PartitionKey。 您可能需要數個使用指定的 UserID 做為資料分割索引鍵的實體。 儲存使用者相關資料的每個實體會分組到單一資料分割，因此這些實體可透過實體群組交易來存取，同時仍具有高擴充性。
> 
> 

在選擇 **PartitionKey** 時還有其他考量，這牽涉到您如何插入、更新和刪除實體。 如需詳細資訊，請參閱[設計用於資料修改的表格](table-storage-design-for-modification.md)。  

## <a name="optimizing-queries-for-the-table-service"></a>最佳化資料表服務的查詢
資料表服務會使用 **PartitionKey** 和 **RowKey** 值在單一叢集化索引中自動為您的實體編製索引，因此，點查詢在使用上最有效率。 不過除此以外，在 **PartitionKey** 和 **RowKey** 的叢集化索引上並沒有其他索引。

許多設計必須符合需求，才能讓您根據多個準則查閱實體。 比方說，根據電子郵件、員工識別碼或姓氏找出員工實體。 [資料表設計模式](table-storage-design-patterns.md) 中所述的模式可因應這些類型的需求，並說明處理表格服務不提供次要索引的方式：  

* [內部資料分割次要索引模式](table-storage-design-patterns.md#intra-partition-secondary-index-pattern) - 為每個實體儲存多個複本且使用不同 **RowKey** 值 (在相同的資料分割內)，透過使用不同的 **RowKey** 值，就能快速且有效率的查閱和替代排序次序。  
* [間資料分割次要索引模式](table-storage-design-patterns.md#inter-partition-secondary-index-pattern) - 在個別資料分割或個別資料表中為每個實體儲存多個複本且使用不同 **RowKey** 值，透過使用不同的 **RowKey** 值，就能快速有效率地查閱和替代排序次序。  
* [索引實體模式](table-storage-design-patterns.md#index-entities-pattern) - 維護索引實體，啟用有效的搜尋以傳回實體清單。  

## <a name="sorting-data-in-the-table-service"></a>在表格服務中排序資料
資料表服務會先根據 **PartitionKey**、再根據 **RowKey** 傳回以遞增方式排序的實體。 這些索引鍵是字串值，若要確保能正確排序數字值，您應該將它們轉換成固定長度，並以零填補它們。 例如，如果您用來做為 **RowKey** 的員工識別碼值是整數值，您應將員工識別碼 **123** 轉換為 **00000123**。  

許多應用程式都需要使用以不同順序排序的資料：例如，依名稱或加入日期為員工排序。 下列模式可因應如何為您的實體取代排序次序：  

* [內部資料分割次要索引模式](table-storage-design-patterns.md#intra-partition-secondary-index-pattern) - 為每個實體儲存多個複本且使用不同 RowKey 值 (在相同的資料分割內)，透過使用不同的 RowKey 值，就能快速且有效率的查閱和替代排序次序。  
* [間資料分割次要索引模式](table-storage-design-patterns.md#inter-partition-secondary-index-pattern) - 在個別資料分割或個別資料表中為每個實體儲存多個複本且使用不同 RowKey 值，透過使用不同的 RowKey 值，就能快速且有效率的查閱和替代排序次序。
* [記錄結尾模式](table-storage-design-patterns.md#log-tail-pattern) - 使用以反向的日期和時間順序排序的 *RowKey* 值，擷取最近加入資料分割的 **n** 個實體。  

## <a name="next-steps"></a>後續步驟

- [資料表設計模式](table-storage-design-patterns.md)
- [將關聯性模型化](table-storage-design-modeling.md)
- [加密資料表資料](table-storage-design-encrypt-data.md)
- [資料修改的設計](table-storage-design-for-modification.md)
