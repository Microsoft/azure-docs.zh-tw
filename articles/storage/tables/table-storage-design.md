---
title: 在 Azure 表格儲存體中設計可擴充且高效能的資料表。 | Microsoft Docs
description: 在 Azure 表格儲存體中設計可擴充且高效能的資料表。
services: storage
author: SnehaGunda
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: 783522997a752c4eac575316983bc6ef853c3f43
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526907"
---
# <a name="design-scalable-and-performant-tables"></a>設計可擴充且具效能的資料表

[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-tip-include.md)]

若要設計可擴充且高效能的資料表，您必須考慮許多因素，例如效能、延展性和成本。 如果您先前曾設計關聯式資料庫的結構描述，您對這些考量會很熟悉，但在部分 Azure 表格服務儲存體模型和關聯式模型之間相似時，將會有些重大差異。 這些差異常會導致不同的設計，看起來可能違反直覺性，或讓熟悉關聯式資料庫的人覺得不對勁，但如果您設計的是 NoSQL 索引鍵/值存放區 (例如 Azure 表格服務)，這就顯得合理了。 許多設計差異將會反映一個事實，也就是表格服務的設計目的是，專門用來支援可能包含數十億個資料實體 (或關聯式資料庫詞彙中的資料列) 的雲端規模應用程式，或是支援必須支援高交易量的資料集。 因此，您需要從不同角度思考如何儲存資料，並了解表格服務的運作方式。 相較於使用關聯式資料庫的方案，設計良好的 NoSQL 資料存放區可以讓您更進一步 (並以較低的成本) 進行調整。 本指南針對這些主題為您提供協助。  

## <a name="about-the-azure-table-service"></a>關於 Azure 資料表服務
本節強調資料表服務的某些重要功能，特別是關於效能和延展性的設計。 如果您是第一次使用 Azure 儲存體和表格服務，請先閱讀 [Microsoft Azure 儲存體簡介](../../storage/common/storage-introduction.md)和[以 .NET 開始使用 Azure 表格儲存體](../../cosmos-db/table-storage-how-to-use-dotnet.md)，再閱讀本文的其餘部分。 雖然本指南的焦點是表格服務，但仍會討論 Azure 佇列和 Blob 服務，以及如何將它們與表格服務搭配使用。  

什麼是資料表服務？ 顧名思義，資料表服務會使用表格格式來儲存資料。 在標準術語中，資料表的每個資料列都代表一個實體，而資料行儲存該實體的各種屬性。 每個實體都有一組可唯一識別該實體的索引鍵，以及表格服務用來追蹤實體最後更新時間的時間戳記資料行。 時間戳記會自動套用，而且您無法手動使用任意值來覆寫時間戳記。 資料表服務會使用這個上次修改的時間戳記 (LMT) 來管理開放式並行存取。  

> [!NOTE]
> 表格服務 REST API 作業也會傳回它從 LMT 衍生的 **ETag** 值。 本文將交互使用 ETag 和 LMT 詞彙，因為它們參考相同的基礎資料。  
> 
> 

下列範例說明簡單的資料表設計，用以儲存員工和部門的實體。 本指南稍後說明的許多範例皆以這個簡單的設計為基礎。  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td>行銷</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>行銷</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>Jun</td>
<td>Cao</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>行銷</td>
<td>系所</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>行銷</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>銷售</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


到目前為止，此資料看起來非常類似於關聯式資料庫中的資料表，主要差異在於必要資料行，並在相同資料表中儲存多個實體類型的能力。 此外，每個使用者定義屬性 (例如**名字**或**年齡**) 皆具有資料類型，例如整數或字串，就像關聯式資料庫中的資料行。 雖然不同於關聯式資料庫，但資料表服務的無結構描述本質意味著一個屬性在每個實體上不需要相同的資料類型。 若要將複雜資料類型儲存在單一屬性中，您必須使用序列化格式，例如 JSON 或 XML。 如需表格服務 (例如支援的資料類型、支援的日期範圍、命名規則和大小限制) 的詳細資訊，請參閱 [了解表格服務資料模型](http://msdn.microsoft.com/library/azure/dd179338.aspx)。

您所選擇的 **PartitionKey** 和 **RowKey** 是良好資料表設計不可或缺的元素。 儲存在資料表中的每個實體都必須有一個獨一無二的 **PartitionKey** 和 **RowKey** 組合。 如同關聯式資料庫資料表中的索引鍵，**PartitionKey** 和 **RowKey** 值會進行索引編製，以建立可啟用快速查閱的叢集索引。 不過，表格服務不會建立任何次要索引，因此 **PartitionKey** 和 **RowKey** 是唯一已編製索引的屬性。 [資料表設計模式](table-storage-design-patterns.md)中描述的某些模式會說明您可以如何解決此明顯限制。  

資料表由一或多個分割所組成，您所做的許多設計決策都牽涉到必須選擇適合的 **PartitionKey** 和 **RowKey** 以最佳化解決方案。 方案可以由單一資料表組成 (其中組織成資料分割的所有實體)，但方案通常會有多個資料表。 資料表可幫助您以邏輯方式組織您的實體，幫助您使用存取控制清單管理資料的存取，而且您可以使用單一儲存體作業放置整個資料表。  

## <a name="table-partitions"></a>資料表的資料分割
帳戶名稱、資料表名稱和 **PartitionKey** 這三個項目可共同識別儲存體服務中的分割，而儲存體服務即是表格服務儲存實體的地方。 分割也同時是實體的定址配置的一部分，可定義交易的範圍 (請參閱下方的 [實體群組交易](#entity-group-transactions) )，並打造表格服務調整方式的基礎。 如需分割的詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](../../storage/common/storage-scalability-targets.md)。  

在表格服務中，個別節點可為一或多個完整資料分割提供服務，且服務會透過動態的負載平衡資料分割在節點間進行調整。 如果某節點的負載過大，表格服務可將由該節點提供服務的分割範圍*分割*到不同的節點；當流量變小時，服務可將分割範圍從靜止節點*合併*回單一節點。  

如需表格服務之內部詳細資料的詳細資訊 (特別是服務管理分割的方式)，請參閱文件 [Microsoft Azure 儲存體：具有高度一致性的高可用性雲端儲存體服務](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)。  

## <a name="entity-group-transactions"></a>實體群組交易
在資料表服務中，實體群組交易 (EGT) 是唯一的內建機制，可跨越多個實體執行不可部分完成的更新。 EGT 有時也稱為「批次交易」。 EGT 僅能對相同分割 (亦即，共用指定表格中的相同分割索引鍵) 中儲存的實體進行作業。 因此每當您需要跨多個實體執行不可部分完成的交易行為時，您必須確定這些實體位於相同的分割中。 通常就是基於此原因，才需要將多個實體類型放在相同的資料表 (和資料分割) 中，而不讓不同的實體類型使用多個資料表。 單一 EGT 最多可以操作 100 個實體。  如果您送出多個並行 EGT 進行處理，請務必確保這些 EGT 不會在 EGT 的通用實體上運作，否則處理可能會延遲。

EGT 也會帶來需在您設計中評估的潛在取捨。 那就是，使用較多分割會增加應用程式的延展性，因為 Azure 有更多機會可在多個節點間執行負載平衡要求。 但是，使用多個分割可能會在應用程式執行不可部分完成的交易時，及維護資料的嚴格一致性時造成限制。 此外，還有些在分割層級上的特定延展性目標，可能會限制單一節點的預期交易輸送量。 如需深入了解 Azure 儲存體帳戶和表格服務的延展性目標，請參閱 [Azure 儲存體延展性和效能目標](../../storage/common/storage-scalability-targets.md)。   

## <a name="capacity-considerations"></a>容量考量
下表說明您在設計表格服務方案時所需注意的某些索引鍵值：  

| Azure 儲存體帳戶的容量總計 | 500 TB |
| --- | --- |
| Azure 儲存體帳戶中的資料表數目 |僅受限於儲存體帳戶的容量 |
| 資料表中的資料分割數目 |僅受限於儲存體帳戶的容量 |
| 資料分割中的實體數目 |僅受限於儲存體帳戶的容量 |
| 個別實體的大小 |最多 1 MB 和 255 個屬性 (包括 **PartitionKey**、**RowKey** 和 **Timestamp**) |
| **PartitionKey** |大小最多 1 KB 的字串 |
| **RowKey** |大小最多 1 KB 的字串 |
| 實體群組交易的大小 |交易最多可以包含 100 個實體，而承載大小必須小於 4 MB。 一個 EGT 只能更新實體一次。 |

如需詳細資訊，請參閱 [了解表格服務資料模型](http://msdn.microsoft.com/library/azure/dd179338.aspx)。  

## <a name="cost-considerations"></a>成本考量
表格儲存體比較便宜，但您在評估任何表格服務方案時，均應將容量使用量和交易數目的成本預估同時納入考量。 不過，在許多情況下，儲存反正規化或重複的資料以改善方案的效能或延展性，也是有效措施。 如需定價的詳細資訊，請參閱 [Azure 儲存體定價](https://azure.microsoft.com/pricing/details/storage/)。  

## <a name="next-steps"></a>後續步驟

- [資料表設計模式](table-storage-design-patterns.md)
- [將關聯性模型化](table-storage-design-modeling.md)
- [查詢的設計](table-storage-design-for-query.md)
- [加密資料表的資料](table-storage-design-encrypt-data.md)
- [資料修改的設計](table-storage-design-for-modification.md)