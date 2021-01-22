---
title: 改善專用 SQL 集區的資料行存放區索引效能
description: 減少記憶體需求或增加可用記憶體，以最大化專用 SQL 集區中每個資料列群組內的資料列數目。
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/22/2019
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: d668c3e505d6849d3cde52d52698a95c1c5647d9
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98676157"
---
# <a name="maximizing-rowgroup-quality-for-columnstore-indexes-in-dedicated-sql-pool"></a>最大化專用 SQL 集區中資料行存放區索引的資料列群組品質 

資料列群組品質取決於資料列群組中的資料列數目。 增加可用的記憶體可將資料行存放區索引壓縮成每個資料列群組的資料列數目最大化。  使用這些方法來改善壓縮率和查詢資料行存放區索引的效能。

## <a name="why-the-rowgroup-size-matters"></a>為什麼資料列群組很重要

因為資料行存放區索引會藉由掃描個別資料列群組的資料行區段來掃描資料表，最大化每個資料列群組的資料列數目可以提升查詢效能。

當資料列群組會有大量的資料列時，可改善資料壓縮，這表示從磁碟讀取的資料比較少。

如需關於資料列群組的詳細資訊，請參閱[資料行存放區索引指南](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。

## <a name="target-size-for-rowgroups"></a>資料列群組的目標大小

為了達到最佳查詢效能，目標是最大化存放區索引中每個資料行的資料列數目。 一個資料列群組最多可有 1,048,576 個資料列。

每個資料列群組沒有資料列數目上限也沒關係。 當資料列群組擁有至少 100,000 個資料列時，資料行存放區索引會獲得良好效能。

## <a name="rowgroups-can-get-trimmed-during-compression"></a>資料列群組可以在壓縮期間進行修剪

在大量載入或資料行存放區索引重建期間，有時可用的記憶體不足，無法壓縮指定給每個資料列群組的所有資料列。 當記憶體壓力存在時，資料行存放區索引會修剪資料列群組大小，因此壓縮至資料行存放區將會成功。

如果記憶體不足，無法將至少10000個數據列壓縮到每個資料列群組中，則會產生錯誤。

如需有關大量載入的詳細資訊，請參閱[大量載入叢集資料行存放區索引](/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。

## <a name="how-to-monitor-rowgroup-quality"></a>如何監視資料列群組品質

DMV sys.dm_pdw_nodes_db_column_store_row_group_physical_stats ([sys.dm_db_column_store_row_group_physical_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-column-store-row-group-physical-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) 包含符合 SQL db) 的視圖定義，可公開有用的資訊，例如資料列群組中的資料列數目，以及修剪的原因（如果有修剪的話）。

您可以建立下列檢視，並將其作為查詢這個 DMV 以取得有關資料列群組修剪資訊的便利方法。

```sql
create view dbo.vCS_rg_physical_stats
as
with cte
as
(
select   tb.[name]                    AS [logical_table_name]
,        rg.[row_group_id]            AS [row_group_id]
,        rg.[state]                   AS [state]
,        rg.[state_desc]              AS [state_desc]
,        rg.[total_rows]              AS [total_rows]
,        rg.[trim_reason_desc]        AS trim_reason_desc
,        mp.[physical_name]           AS physical_name
FROM    sys.[schemas] sm
JOIN    sys.[tables] tb               ON  sm.[schema_id]          = tb.[schema_id]
JOIN    sys.[pdw_table_mappings] mp   ON  tb.[object_id]          = mp.[object_id]
JOIN    sys.[pdw_nodes_tables] nt     ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[dm_pdw_nodes_db_column_store_row_group_physical_stats] rg      ON  rg.[object_id]     = nt.[object_id]
                                                                            AND rg.[pdw_node_id]   = nt.[pdw_node_id]
                                        AND rg.[distribution_id]    = nt.[distribution_id]
)
select *
from cte;
```

trim_reason_desc 會告知是否已修剪資料列群組 (trim_reason_desc = NO_TRIM 表示沒有修剪，且資料列群組屬於最佳品質)。 下列修剪原因表示過早修剪了資料列群組：

- BULKLOAD：當傳入的負載資料列批次具有少於 1 百萬個的資料列時，會使用這個修剪原因。 如果插入了多於 100,000 個資料列 (而不是插入到差異存放區)，則引擎會建立壓縮的資料列群組，但是會將修剪原因設定為大量載入。 在此案例中，請考慮增加您的批次負載以包含更多資料列。 此外，請重新評估資料分割配置，以確保資料列群組無法跨越資料分割界限時，不太細微。
- MEMORY_LIMITATION：若要建立包含 1 百萬個資料列的資料列群組，引擎會需要特定數量的工作記憶體。 當載入工作階段的可用記憶體小於所需的工作記憶體時，會提前修剪資料列群組。 下列各節說明如何估計所需的記憶體並配置更多記憶體。
- DICTIONARY_SIZE：這個修剪原因表示因為至少有一個字串資料行具有寬/或高基數字串而發生資料列群組修剪。 記憶體中的字典大小限制為 16 MB，且一旦達到此限制，便會壓縮資料列群組。 如果您遇到這種情況，請考慮將問題資料行隔離到單獨的資料表中。

## <a name="how-to-estimate-memory-requirements"></a>如何估計記憶體需求

<!--
To view an estimate of the memory requirements to compress a rowgroup of maximum size into a columnstore index, download and run the view [dbo.vCS_mon_mem_grant](). This view shows the size of the memory grant that a rowgroup requires for compression in to the columnstore.
-->

要壓縮一個資料列群組所需的最大記憶體大約是

- 72 MB +
- \#資料列資料行 \* \# \* 8 個位元組 +
- \#資料 \* \# 列短字串資料行 \* 32 位元組 +
- \#長字串資料行 \* 壓縮字典 16 MB

> [!NOTE]
> 字串資料行使用 <= 32 位元組和長字串資料行的字串資料類型，會使用 > 32 位元組的字串資料類型。

會使用專為壓縮文字的壓縮方法來壓縮長字串。 這個壓縮方法會使用字典來儲存文字模式。 字典的大小上限為 16 MB。 資料列群組中的每一個長字串資料行只有一個字典。

如需資料行存放區記憶體需求的深入討論，請參閱影片 [專用的 SQL 集區調整：設定和指引](https://channel9.msdn.com/Events/Ignite/2016/BRK3291)。

## <a name="ways-to-reduce-memory-requirements"></a>減少記憶體需求的方式

使用下列技巧來減少記憶體需求，以將資料列群組壓縮到資料行存放區索引。

### <a name="use-fewer-columns"></a>使用較少的資料行

可能的話，設計較少資料行的資料表。 當資料列群組壓縮至資料行存放區內時，資料行存放區索引會個別壓縮每個資料行區段。

因此，壓縮資料列群組的記憶體需求會隨著資料行數目的增加而增加。

### <a name="use-fewer-string-columns"></a>使用較少的字串資料行

字串資料類型的資料行比數字和日期資料類型需要更多的記憶體。 若要減少記憶體需求，請考慮從事實資料表中移除字串資料行，並將其放在較小的維度資料表中。

字串壓縮的其他記憶體需求︰

- 最多 32 個字元的字串資料類型可能每個值需要 32 個額外的位元組。
- 使用超過 32 個字元的字串資料類型會使用字典方法進行壓縮。  資料列群組中的每個資料行可能需要建置額外的 16 MB 建置字典。

### <a name="avoid-over-partitioning"></a>避免過度分割

資料行存放區索引針對每個資料分割會建立一個或多個資料列群組。 針對 Azure Synapse Analytics 中的專用 SQL 集區，資料分割數目會快速成長，因為資料已散發，而且每個散發都已分割。

如果資料表有太多資料分割，則可能沒有足夠的資料列可填滿資料列群組。 缺少資料列並不會在壓縮期間建立記憶體壓力。 但是，它會導致資料列群組無法達到最佳資料行存放區查詢效能。

要避免過度磁碟分割的另一個原因，是有額外負荷的記憶體將資料列載入分割資料表上的資料行存放區索引。

在載入期間，許多資料分割可能會收到內送資料列並保留在記憶體中，直到每個分割區有足夠的資料列可壓縮為止。 具有太多資料分割會建立額外的記憶體不足壓力。

### <a name="simplify-the-load-query"></a>簡化載入查詢

資料庫會共用在查詢中所有運算子之間的查詢記憶體授權。 當載入查詢具有複雜的排序和聯結時，可供壓縮的記憶體便會減少。

設計負載查詢以僅著重於載入查詢。 如果您需要在資料上執行轉換，從載入查詢個別執行它們。 例如，接移堆積資料表中的資料、執行轉換，然後將暫存資料表載入資料行存放區索引。

> [!TIP]
> 您可以也先載入資料，然後使用 MPP 系統來轉換資料。

### <a name="adjust-maxdop"></a>調整 MAXDOP

當有多個 CPU 核心可供每個散發使用時，每個散發會將資料列群組平行壓縮到資料行存放區。

平行處理原則需要額外的記憶體資源，可能會導致記憶體不足的壓力和調整資料列群組。

若要減少記憶體不足的壓力，您可以使用 MAXDOP 查詢提示來強制載入作業，以便在每個散發內的序列模式中執行。

```sql
CREATE TABLE MyFactSalesQuota
WITH (DISTRIBUTION = ROUND_ROBIN)
AS SELECT * FROM FactSalesQuota
OPTION (MAXDOP 1);
```

## <a name="ways-to-allocate-more-memory"></a>配置更多記憶體的方式

DWU 大小和使用者資源類別會共同判斷有多少記憶體可供使用者查詢。

若要增加負載查詢的記憶體授權，您可以增加 DWU 數目或增加資源類別。

- 若要增加 DWU，請參閱[如何調整效能？](quickstart-scale-compute-portal.md)
- 若要變更查詢的資源類別，請參閱[變更使用者資源類別的範例](resource-classes-for-workload-management.md#change-a-users-resource-class)。

## <a name="next-steps"></a>下一步

若要尋找更多改善專用 SQL 集區效能的方法，請參閱 [效能總覽](cheat-sheet.md)。
