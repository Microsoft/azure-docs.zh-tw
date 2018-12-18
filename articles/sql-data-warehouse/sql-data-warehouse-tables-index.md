---
title: 在 Azure SQL 資料倉儲中編製資料表的索引 | Microsoft Azure
description: 在 Azure SQL 資料倉儲中編製資料表索引的建議與範例。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: d709acfe378583a21b72971f465e4b5d73818bcd
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307723"
---
# <a name="indexing-tables-in-sql-data-warehouse"></a>在 SQL 資料倉儲中編製資料表的索引
在 Azure SQL 資料倉儲中編製資料表索引的建議與範例。

## <a name="what-are-index-choices"></a>索引選項有哪些？

SQL 資料倉儲提供數個索引選項，包括[叢集資料行存放區索引](/sql/relational-databases/indexes/columnstore-indexes-overview)、[叢集索引和非叢集索引](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described)，以及稱為[堆積](/sql/relational-databases/indexes/heaps-tables-without-clustered-indexes)的非索引選項。  

若要以索引建立資料表，請參閱 [CREATE TABLE (Azure SQL 資料倉儲)](/sql/t-sql/statements/create-table-azure-sql-data-warehouse) 文件。

## <a name="clustered-columnstore-indexes"></a>叢集資料行存放區索引
根據預設，若未在資料表上指定任何索引選項，則 SQL 資料倉儲會建立叢集資料行存放區索引。 叢集資料行存放區資料表提供最高層級的資料壓縮，以及最佳的整體查詢效能。  叢集資料行存放區資料表通常勝過叢集索引或堆積資料表，而且通常是大型資料表的最佳選擇。  基於這些理由，叢集資料行存放區是您不確定如何編製資料表索引時的最佳起點。  

若要建立叢集資料行存放區資料表，只要在 WITH 子句中指定 CLUSTERED COLUMNSTORE INDEX，或省略 WITH 子句︰

```SQL
CREATE TABLE myTable   
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( CLUSTERED COLUMNSTORE INDEX );
```

在有些情況下，叢集資料行存放區可能不是很好的選擇︰

- 資料行存放區資料表不支援 varchar(max)、nvarchar(max) 和 varbinary(max)。 請改為考慮堆積或叢集索引。
- 資料行存放區資料表可能比暫時性資料沒有效率。 請考慮堆積，甚至是暫時性資料表。
- 具有少於 1 億個資料列的小型資料表。 請考慮堆積資料表。

## <a name="heap-tables"></a>堆積資料表
當您讓資料暫時登陸於 SQL 資料倉儲時，可能會發現使用堆積資料表會讓整個程序更快速。 這是因為堆積的載入速度比索引資料表還要快，而在某些情況下，可以從快取進行後續的讀取。  如果您載入資料只是在做執行更多轉換之前的預備，將資料表載入堆積資料表會遠快於將資料載入叢集資料行存放區資料表。 此外，將資料載入[暫存資料表](sql-data-warehouse-tables-temporary.md)會比將資料表載入永久儲存體快速。  

若為小於 1 億個資料列的小型查閱資料表，堆積資料表通常比較適合。  一旦超過 1 億個資料列，叢集資料行存放區資料表就會開始達到最佳的壓縮。

若要建立堆積資料表，只需在 WITH 子句中指定 HEAP︰

```SQL
CREATE TABLE myTable   
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( HEAP );
```

## <a name="clustered-and-nonclustered-indexes"></a>叢集與非叢集索引
需要快速擷取單一資料列時，叢集索引可能會優於叢集資料行存放區資料表。 對於需要單一或極少數資料列查閱才能疾速執行的查詢，請考慮使用叢集索引或非叢集次要索引。 使用叢集索引的缺點是受益的只有在叢集索引資料行上使用高度選擇性篩選的查詢。 若要改善其他資料行的篩選，可以將非叢集索引加入至其他資料行。 不過，每個加入資料表的索引皆會增加載入的空間和處理時間。

若要建立叢集索引資料表，只要在 WITH 子句中指定 CLUSTERED INDEX︰

```SQL
CREATE TABLE myTable   
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( CLUSTERED INDEX (id) );
```

若要在資料表上新增非叢集索引，只要使用下列語法：

```SQL
CREATE INDEX zipCodeIndex ON myTable (zipCode);
```

## <a name="optimizing-clustered-columnstore-indexes"></a>最佳化叢集資料行存放區索引
叢集資料行存放區資料表會將資料組織成不同區段。  擁有高區段品質是在資料行存放區資料表上達到最佳查詢效能的關鍵。  壓縮的資料列群組中的資料列數目可以測量區段品質。  每個壓縮的資料列群組至少有 10 萬個資料列時的區段品質最佳，而隨著每個資料列群組的資料列數趨近 1,048,576 個資料列 (這是資料列群組可以包含的最大資料列數)，效能會跟著提升。

以下檢視可以建立於您的系統上並用來計算每個資料列群組的平均資料列數，以及識別任何次佳的叢集資料行存放區索引。  此檢視上的最後一個資料行會產生 SQL 陳述式，以便用來重建索引。

```sql
CREATE VIEW dbo.vColumnstoreDensity
AS
SELECT
        GETDATE()                                                               AS [execution_date]
,       DB_Name()                                                               AS [database_name]
,       s.name                                                                  AS [schema_name]
,       t.name                                                                  AS [table_name]
,    COUNT(DISTINCT rg.[partition_number])                    AS [table_partition_count]
,       SUM(rg.[total_rows])                                                    AS [row_count_total]
,       SUM(rg.[total_rows])/COUNT(DISTINCT rg.[distribution_id])               AS [row_count_per_distribution_MAX]
,    CEILING    ((SUM(rg.[total_rows])*1.0/COUNT(DISTINCT rg.[distribution_id]))/1048576) AS [rowgroup_per_distribution_MAX]
,       SUM(CASE WHEN rg.[State] = 0 THEN 1                   ELSE 0    END)    AS [INVISIBLE_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE 0    END)    AS [INVISIBLE_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 1 THEN 1                   ELSE 0    END)    AS [OPEN_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE 0    END)    AS [OPEN_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 2 THEN 1                   ELSE 0    END)    AS [CLOSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE 0    END)    AS [CLOSED_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 3 THEN 1                   ELSE 0    END)    AS [COMPRESSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE 0    END)    AS [COMPRESSED_rowgroup_rows]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[deleted_rows]   ELSE 0    END)    AS [COMPRESSED_rowgroup_rows_DELETED]
,       MIN(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_AVG]
,       'ALTER INDEX ALL ON ' + s.name + '.' + t.NAME + ' REBUILD;'             AS [Rebuild_Index_SQL]
FROM    sys.[pdw_nodes_column_store_row_groups] rg
JOIN    sys.[pdw_nodes_tables] nt                   ON  rg.[object_id]          = nt.[object_id]
                                                    AND rg.[pdw_node_id]        = nt.[pdw_node_id]
                                                    AND rg.[distribution_id]    = nt.[distribution_id]
JOIN    sys.[pdw_table_mappings] mp                 ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[tables] t                              ON  mp.[object_id]          = t.[object_id]
JOIN    sys.[schemas] s                             ON t.[schema_id]            = s.[schema_id]
GROUP BY
        s.[name]
,       t.[name]
;
```

現在您已建立檢視，請執行此查詢來識別哪些資料表的資料列群組中的資料列少於 10 萬個。 當然，如果您要尋求更理想的區段品質，您可能想要提高 10 萬的臨界值。 

```sql
SELECT    *
FROM    [dbo].[vColumnstoreDensity]
WHERE    COMPRESSED_rowgroup_rows_AVG < 100000
        OR INVISIBLE_rowgroup_rows_AVG < 100000
```

一旦您執行查詢，就可以開始查看資料，並分析您的結果。 此表格會說明在您的資料列群組分析中要尋找的項目。

| 欄 | 如何使用這項資料 |
| --- | --- |
| [table_partition_count] |如果資料表已分割，您可能會預期看到較高的開放資料列群組計數。 散發套件中的每個分割在理論上有與其相關聯的開放資料列群組。 將這個因素納入您的分析。 已分割的小型資料表可以藉由移除分割進行最佳化，因為這樣會改善壓縮。 |
| [row_count_total] |資料表的資料列計數。 例如，您可以使用此值來計算資料列的百分比 (壓縮的狀態)。 |
| [row_count_per_distribution_MAX] |如果所有資料列平均分配，這個值會是每個散發的目標資料列數目。 比較此值與 compressed_rowgroup_count。 |
| [COMPRESSED_rowgroup_rows] |資料表的資料行存放區格式中的資料列總數。 |
| [COMPRESSED_rowgroup_rows_AVG] |如果平均資料列數目遠小於資料群組最大的資料列數目，則可考慮使用 CTAS 或 ALTER INDEX REBUILD 重新壓縮資料 |
| [COMPRESSED_rowgroup_count] |資料行存放區格式中的資料列群組數目。 如果相對於資料表的這個數目很高，表示資料行存放區密度很低。 |
| [COMPRESSED_rowgroup_rows_DELETED] |資料行存放區格式中的資料列會以邏輯方式刪除。 如果相對於資料表大小的這個數目很高，請考慮重新建立分割或重建索引，因為這樣會將其實際移除。 |
| [COMPRESSED_rowgroup_rows_MIN] |將它與 AVG 和 MAX 資料行搭配使用，以了解資料行存放區中資料列群組的值範圍。 載入臨界值上較低的數目 (每個分割對齊散發套件 102,400) 表示資料載入可進行最佳化 |
| [COMPRESSED_rowgroup_rows_MAX] |同上。 |
| [OPEN_rowgroup_count] |開放資料列群組都正常。 每個資料表散發都應該有一個開放資料列群組 (60)。 過多的數目表示資料跨分割載入。 重複檢查分割策略並確定它是正確的 |
| [OPEN_rowgroup_rows] |每個資料列群組可以有最多 1,048,576 個資料列。 使用這個值查看開放資料列群組目前的飽和度 |
| [OPEN_rowgroup_rows_MIN] |開放群組會指出資料是緩慢移動載入資料表，或是先前的載入將剩餘的資料列溢出到此資料列群組。 使用 MIN、MAX、AVG 欄位查看多少資料位於開放資料列群組。 對於小型資料表，可能是所有資料的 100%！ 在此情況下，ALTER INDEX REBUILD 來強制資料進入資料行存放區。 |
| [OPEN_rowgroup_rows_MAX] |同上。 |
| [OPEN_rowgroup_rows_AVG] |同上。 |
| [CLOSED_rowgroup_rows] |查看關閉的資料列群組資料列做為例行性檢查。 |
| [CLOSED_rowgroup_count] |如果發現任何關閉資料列群組，其數目應該很小。 關閉資料列群組可以使用 ALTER INDEX 轉換成壓縮資料列群組...REORGANIZE 命令。 不過，通常並不需要。 關閉群組會透過背景 "tuple mover" 程序自動轉換成資料行存放區的資料列群組。 |
| [CLOSED_rowgroup_rows_MIN] |關閉資料列群組應該具有極高的填滿率。 如果關閉資料列群組的填滿率很低，就需要進一步分析資料行存放區。 |
| [CLOSED_rowgroup_rows_MAX] |同上。 |
| [CLOSED_rowgroup_rows_AVG] |同上。 |
| [Rebuild_Index_SQL] |用來重建資料表的資料行存放區索引的 SQL |

## <a name="causes-of-poor-columnstore-index-quality"></a>資料行存放區索引品質不佳的原因
如果您已識別區段品質不佳的資料表，便會想要找出根本原因。  以下是區段品質不佳的一些其他常見原因︰

1. 建立索引時的記憶體壓力
2. 大量的 DML 作業
3. 小型或緩慢移動的載入作業
4. 太多資料分割

這些因素可能會導致資料行存放區索引在每個資料列群組中的資料列大幅少於最佳的 100 萬個。 它們也會造成資料列移至差異資料列群組，而不是壓縮的資料列群組。 

### <a name="memory-pressure-when-index-was-built"></a>建立索引時的記憶體壓力
每個壓縮資料列群組的資料列數目，直接與資料列寬度以及可用來處理資料列群組的記憶體數量相關。  當資料列在記憶體不足的狀態下寫入資料行存放區資料表時，資料行存放區區段品質可能會降低。  因此，最佳做法是盡可能讓寫入至您的資料行存放區索引資料表的工作階段能存取較多的記憶體。  因為記憶體與並行存取之間有所取捨，正確的記憶體配置指引取決於您的資料表的每個資料列中的資料、配置給系統的資料倉儲單位，以及您可以提供給將資料寫入至資料表的工作階段的並行存取插槽數目。  最佳做法：如果您使用 DW300 或更少，我們建議從 xlargerc 開始，如果您使用 DW400 至 DW600，則從 largerc 開始，而如果您使用 DW1000 和更高，則從 mediumrc 開始。

### <a name="high-volume-of-dml-operations"></a>大量的 DML 作業
更新和刪除資料列的大量 DML 作業，會造成資料行存放區沒有效率。 這在資料列群組中大部分的資料列都已修改時，更是如此。

- 從壓縮的資料列群組刪除資料列僅會以邏輯方式將資料列標示為已刪除。 資料列會保留在壓縮的資料列群組中，直到重建資料分割或資料表為止。
- 插入資料列會將資料列新增至名為差異資料列群組的內部資料列存放區資料表。 在差異資料列群組已滿且標示為已關閉之前，插入的資料列不會轉換成資料行存放區。 一旦達到 1,048,576 個資料列的容量上限，資料列群組就會關閉。 
- 更新資料行存放區格式的資料列會做為邏輯刪除和插入來處理。 插入的資料列可儲存在差異存放區。

針對每個資料分割對齊分佈的 102,400 個資料列大量臨界值，超出臨界值的批次更新和插入作業會直接進入資料行存放區格式。 不過，假設在平均分佈情況下，您將需要在單一作業中修改超過 6.144 百萬個資料列才會發生這種情況。 如果特定資料分割對齊分佈的資料列數目少於 102,400 個，則資料列會移至差異存放區，且在插入足夠的資料列、修改資料列以關閉資料列群組或已重新建立索引之前，都會存放於差異存放區。

### <a name="small-or-trickle-load-operations"></a>小型或緩慢移動的載入作業
流入 SQL 資料倉儲的小型負載，有時也稱為緩慢移動的負載。 它們通常代表系統接近連續擷取的串流。 不過，因為這個串流已接近連續狀態，所以資料列的容量並沒有特別大。 通常資料遠低於直接載入資料行存放區格式所需的閾值。

在這些情況下，最好先將資料登陸到 Azure Blob 儲存體中，並讓它在載入之前累積。 這項技術通常稱為 *微批次處理*。

### <a name="too-many-partitions"></a>太多資料分割
另一個考慮事項是資料分割對於叢集資料行存放區資料表的影響。  資料分割之前，SQL 資料倉儲已將您的資料分成 60 個資料庫。  進一步分割會分割您的資料。  如果您將資料分割，請考慮到**每個**資料分割需要有至少 1 百萬個資料列，使用叢集資料行存放區索引才會有幫助。  如果將您的資料表分割成 100 個資料分割，則您的資料表需要至少有 60 億個資料列，才會受益於叢集資料行存放區索引 (60 個分佈 * 100 個資料分割 * 1 百萬個資料列)。 如果您的 100 個分割資料表沒有 60 億個資料列，請減少資料分割數目，或考慮改用堆積資料表。

您的資料表載入一些資料之後，請依照下列步驟來識別並重建具有次佳叢集資料行存放區索引的資料表。

## <a name="rebuilding-indexes-to-improve-segment-quality"></a>重建索引以提升區段品質
### <a name="step-1-identify-or-create-user-which-uses-the-right-resource-class"></a>步驟 1︰識別或建立會使用適當資源類別的使用者
立即提升區段品質的快速方法就是重建索引。  上述檢視所傳回的 SQL 會傳回可用來重建索引的 ALTER INDEX REBUILD 陳述式。 重建索引時，請確定配置足夠的記憶體給重建索引的工作階段。  若要這樣做，請增加使用者的資源類別，該使用者有權將此資料表上的索引重建為建議的最小值。 您無法變更資料庫擁有者使用者的資源類別，所以如果尚未在系統上建立使用者，請先建立一個使用者。 如果您使用 DW300 或更少，建議使用的最小資源類別為 xlargerc；如果使用 DW400 至 DW600，則為 largerc；而如果使用 DW1000 和更高，則為 mediumrc。

以下範例示範如何藉由增加資源類別，配置更多記憶體給使用者。 若要使用資源類別，請參閱[適用於工作負載管理的資源類別](resource-classes-for-workload-management.md)。

```sql
EXEC sp_addrolemember 'xlargerc', 'LoadUser'
```

### <a name="step-2-rebuild-clustered-columnstore-indexes-with-higher-resource-class-user"></a>步驟 2︰使用較高的資源類別使用者重建叢集資料行存放區索引
以步驟 1 的使用者身分登入 (例如 LoadUser)，該使用者現在使用較高的資源類別，並執行 ALTER INDEX 陳述式。 請確定這個使用者對於重建索引的資料表擁有 ALTER 權限。 這些範例示範如何重建整個資料行存放區索引或如何重建單一資料分割。 在大型資料表上，比較適合一次重建單一資料分割的索引。

或者，您可以[使用 CTAS](sql-data-warehouse-develop-ctas.md) 將資料表複製到新的資料表，而非重建索引。 哪一種方式最好？ 針對大量的資料，CTAS 的速度通常比 [ALTER INDEX](/sql/t-sql/statements/alter-index-transact-sql) 來得快。 針對較小量的資料，ALTER INDEX 較為容易使用，且您不需要交換出資料表。 如需有關如何使用 CTAS 重建索引的詳細資訊，請參閱 **使用 CTAS 和分割切換重建索引** 。

```sql
-- Rebuild the entire clustered index
ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD
```

```sql
-- Rebuild a single partition
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5
```

```sql
-- Rebuild a single partition with archival compression
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)
```

```sql
-- Rebuild a single partition with columnstore compression
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE)
```

在 SQL 資料倉儲中重建索引是一項離線作業。  如需重建索引的詳細資訊，請參閱[資料行存放區索引重組](/sql/relational-databases/indexes/columnstore-indexes-defragmentation) 中的 ALTER INDEX REBUILD 小節和 [ALTER INDEX](/sql/t-sql/statements/alter-index-transact-sql)。

### <a name="step-3-verify-clustered-columnstore-segment-quality-has-improved"></a>步驟 3︰確認已改善叢集資料行存放區區段品質
請重新執行已識別區段品質不佳之資料表的查詢，並驗證區段品質是否已改善。  如果區段品質並未改善，可能是您的資料表中的資料列過寬。  請考慮在重建索引時使用較高的資源類別或 DWU。

## <a name="rebuilding-indexes-with-ctas-and-partition-switching"></a>使用 CTAS 和分割切換重建索引
此範例會使用 [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 陳述式與資料分割切換來重建資料表資料分割。 

```sql
-- Step 1: Select the partition of data and write it out to a new table using CTAS
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

-- Step 2: Create a SWITCH out table
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    [dbo].[FactInternetSales]
WHERE   1=2 -- Note this table will be empty

-- Step 3: Switch OUT the data 
ALTER TABLE [dbo].[FactInternetSales] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales_20000101] PARTITION 2;

-- Step 4: Switch IN the rebuilt data
ALTER TABLE [dbo].[FactInternetSales_20000101_20010101] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales] PARTITION 2;
```

如需使用 CTAS 重新建立資料分割的詳細資訊，請參閱[在 SQL 資料倉儲中使用資料分割](sql-data-warehouse-tables-partition.md)。

## <a name="next-steps"></a>後續步驟
如需開發資料表的詳細資訊，請參閱[開發資料表](sql-data-warehouse-tables-overview.md)。

