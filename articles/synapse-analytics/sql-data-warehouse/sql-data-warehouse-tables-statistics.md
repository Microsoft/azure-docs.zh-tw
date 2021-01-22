---
title: 在資料表上建立和更新統計資料
description: 針對專用 SQL 集區中的資料表建立和更新查詢優化統計資料的建議和範例。
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/09/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 3ade41c51cbb8065734e8957cfc8b9f0c22b2df3
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98673361"
---
# <a name="table-statistics-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Azure Synapse Analytics 中專用 SQL 集區的資料表統計資料

在本文中，您可以找到針對專用 SQL 集區中的資料表建立和更新查詢優化統計資料的建議和範例。

## <a name="why-use-statistics"></a>為何使用統計資料

更專用的 SQL 集區知道您的資料，它對其執行查詢的速度會更快。 將資料載入專用的 SQL 集區之後，收集資料的統計資料是優化查詢最重要的一件事。

專用的 SQL 集區查詢最佳化工具是以成本為基礎的優化工具。 它會比較各種查詢方案的成本，然後選擇成本最低的方案。 在大部分的情況下，它會選擇執行最快的方案。

例如，如果最佳化工具評估您的查詢篩選的日期會傳回一個資料列，則會選擇一個方案。 如果最佳化工具評估所選取的日期將傳回 100 萬個資料列，則會傳回不同的方案。

## <a name="automatic-creation-of-statistic"></a>自動建立統計資料

當資料庫 AUTO_CREATE_STATISTICS 選項為 on 時，專用的 SQL 集區會分析傳入的使用者查詢是否有遺漏的統計資料。

如果缺少統計資料，查詢最佳化工具會在查詢述詞或聯結條件中的個別資料行上建立統計資料，以改善查詢計劃的基數估計值。

> [!NOTE]
> 自動建立統計資料目前依預設開啟。

您可以藉由執行下列命令來檢查專用的 SQL 集區是否 AUTO_CREATE_STATISTICS 設定：

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

如果您專用的 SQL 集區未設定 AUTO_CREATE_STATISTICS，建議您執行下列命令來啟用此屬性：

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

這些語句將會觸發自動建立統計資料：

- SELECT
- INSERT-SELECT
- CTAS
- UPDATE
- 刪除
- EXPLAIN，包含聯結或偵測到存在述詞時

> [!NOTE]
> 不會在暫存或外部資料表上建立自動建立統計資料。

自動建立統計資料是同步進行的，因此，如果您的資料行缺少統計資料，查詢效能可能會略為降低。 為單一資料行建立統計資料的時間，取決於資料表的大小。

若要避免測量效能降低，請在分析系統前，先執行基準測試的工作負載，以確定會先建立統計資料。

> [!NOTE]
> 建立統計資料時，會以不同的使用者內容登入 [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) 。

當自動統計資料建立完成時，會採用以下格式：_WA_Sys_<8 digit column id in Hex>_<8 digit table id in Hex>。 您可以藉由執行 [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) 命令來查看已建立的統計資料：

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

Table_name 是包含要顯示之統計資料的資料表名稱。 此資料表不可以是外部資料表。 目標是用來顯示統計資料資訊的目標索引、統計資料或資料行名稱。

## <a name="update-statistics"></a>更新統計資料

其中一個最佳做法，是隨著新增新的日期，每天在日期資料行上更新統計資料。 每次將新的資料列載入專用的 SQL 集區時，就會加入新的載入日期或交易日期。 這些增加項目會造成資料散發變更，並使統計資料過時。

客戶資料表中的國家/地區資料行的統計資料可能永遠不需要更新，因為值散發通常不會變更。 假設客戶間的散發固定不變，將新資料列加入至資料表變化並不會改變資料散發情況。

但是，如果您專用的 SQL 集區只包含一個國家/地區，而您從新的國家/地區導入資料，而導致儲存多個國家/地區的資料，則您需要更新 [國家/地區] 資料行的統計資料。

以下為更新統計資料的相關建議：

|||
|-|-|
| **統計資料更新的頻率**  | 保守：每日 </br> 載入或轉換資料之後 |
| **取樣** |  小於 10 億個資料列，使用預設取樣 (20%)。 </br> 大於 10 億個資料列，使用 2% 的取樣。 |

為查詢疑難排解時，首先要詢問的問題之一就是「統計資料是最新的嗎？」

這個問題不是可依資料存留期回答的問題。 如果基礎資料並沒有任何實質變更，最新的統計資料物件可能會是舊的。 當資料列數目已顯著變更，或資料行的值散發有實質變更時，就應該更新統計資料。 

沒有動態管理檢視可判斷資料表內的資料自從上次更新統計資料之後是否有所變更。  下列兩個查詢可協助您判斷統計資料是否已過時。

**查詢1：**  找出 [統計資料] (**stats_row_count**) 的資料列計數，以及 (**actual_row_count**) 的實際資料列計數之間的差異。 

```sql
select 
objIdsWithStats.[object_id], 
actualRowCounts.[schema], 
actualRowCounts.logical_table_name, 
statsRowCounts.stats_row_count, 
actualRowCounts.actual_row_count,
row_count_difference = CASE
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN actualRowCounts.actual_row_count - statsRowCounts.stats_row_count
    ELSE statsRowCounts.stats_row_count - actualRowCounts.actual_row_count
END,
percent_deviation_from_actual = CASE
    WHEN actualRowCounts.actual_row_count = 0 THEN statsRowCounts.stats_row_count
    WHEN statsRowCounts.stats_row_count = 0 THEN actualRowCounts.actual_row_count
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (actualRowCounts.actual_row_count - statsRowCounts.stats_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
    ELSE CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (statsRowCounts.stats_row_count - actualRowCounts.actual_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
END
from
(
    select distinct object_id from sys.stats where stats_id > 1
) objIdsWithStats
left join
(
    select object_id, sum(rows) as stats_row_count from sys.partitions group by object_id
) statsRowCounts
on objIdsWithStats.object_id = statsRowCounts.object_id 
left join
(
    SELECT sm.name [schema] ,
    tb.name logical_table_name ,
    tb.object_id object_id ,
    SUM(rg.row_count) actual_row_count
    FROM sys.schemas sm
    INNER JOIN sys.tables tb ON sm.schema_id = tb.schema_id
    INNER JOIN sys.pdw_table_mappings mp ON tb.object_id = mp.object_id
    INNER JOIN sys.pdw_nodes_tables nt ON nt.name = mp.physical_name
    INNER JOIN sys.dm_pdw_nodes_db_partition_stats rg
    ON rg.object_id = nt.object_id
    AND rg.pdw_node_id = nt.pdw_node_id
    AND rg.distribution_id = nt.distribution_id
    WHERE 1 = 1
    GROUP BY sm.name, tb.name, tb.object_id
) actualRowCounts
on objIdsWithStats.object_id = actualRowCounts.object_id

```

**查詢2：** 藉由檢查每個資料表上上次更新統計資料的時間，找出統計資料的存在時間。 

> [!NOTE]
> 如果資料行的值散發有實質變更，無論統計資料上一次更新的時間為何，您都應該更新統計資料。

```sql
SELECT
    sm.[name] AS [schema_name],
    tb.[name] AS [table_name],
    co.[name] AS [stats_column_name],
    st.[name] AS [stats_name],
    STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
FROM
    sys.objects ob
    JOIN sys.stats st
        ON  ob.[object_id] = st.[object_id]
    JOIN sys.stats_columns sc
        ON  st.[stats_id] = sc.[stats_id]
        AND st.[object_id] = sc.[object_id]
    JOIN sys.columns co
        ON  sc.[column_id] = co.[column_id]
        AND sc.[object_id] = co.[object_id]
    JOIN sys.types  ty
        ON  co.[user_type_id] = ty.[user_type_id]
    JOIN sys.tables tb
        ON  co.[object_id] = tb.[object_id]
    JOIN sys.schemas sm
        ON  tb.[schema_id] = sm.[schema_id]
WHERE
    st.[user_created] = 1;
```

例如，專用 SQL 集區中的 **日期資料行** 通常需要經常更新統計資料。 每次將新的資料列載入專用的 SQL 集區時，就會加入新的載入日期或交易日期。 這些增加項目會造成資料散發變更，並使統計資料過時。

相反地，客戶資料表上性別資料行的統計資料可能永遠不需要更新。 假設客戶間的散發固定不變，將新資料列加入至資料表變化並不會改變資料散發情況。

如果您專用的 SQL 集區只包含一個性別，而新的需求導致多個性別，則您需要更新性別資料行的統計資料。

如需詳細資訊，請參閱[統計資料](/sql/relational-databases/statistics/statistics?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)的一般指引。

## <a name="implementing-statistics-management"></a>實作統計資料管理

擴充您的資料載入程式通常是個不錯的主意，可確保在載入結束時更新統計資料，以避免/減少並行查詢之間的封鎖或資源爭用。  

當資料表變更其大小和/或其值散發時，資料載入最為頻繁。 資料載入是執行某些管理進程的邏輯位置。

以下提供指導原則，以便更新您的統計資料：

- 確保每個載入的資料表至少都更新一個統計資料物件。 這會在統計資料更新過程中更新資料表大小 (資料列計數和頁面計數) 資訊。
- 將焦點放在參與 JOIN、GROUP BY、ORDER BY 和 DISTINCT 子句的資料行。
- 考慮較頻繁更新「遞增索引鍵」資料行 (例如交易日期)，因為這些值不會包含在統計資料長條圖中。
- 考慮較不常更新靜態散發資料行。
- 請記得，每個統計資料物件會依序更新。 僅只實作 `UPDATE STATISTICS <TABLE_NAME>` 不一定理想，尤其是對具有許多統計資料物件的寬型資料表而言。

如需詳細資訊，請參閱[基數估計](/sql/relational-databases/performance/cardinality-estimation-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。

## <a name="examples-create-statistics"></a>範例：建立統計資料

下列範例顯示如何使用各種選項來建立統計資料。 您用於每個資料行的選項取決於您的資料特定以及在查詢中使用資料行的方式。

### <a name="create-single-column-statistics-with-default-options"></a>使用預設選項建立單一資料行統計資料

若要建立資料行的統計資料，請提供統計資料物件的名稱和資料行的名稱。

此語法會使用所有預設選項。 根據預設，建立統計資料時，會取樣資料表的 **20%** 。

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

例如：

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### <a name="create-single-column-statistics-by-examining-every-row"></a>檢查每個資料列以建立單一資料行統計資料

20% 的預設取樣率足以應付大部分的情況。 不過，您可以調整取樣率。

若要取樣整個資料表，請使用此語法：

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

例如：

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>指定取樣大小以建立單一資料行統計資料

或者，您可以以百分比指定取樣大小：

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>只對某些資料列建立單一資料行統計資料

您也可以對資料表中部分的資料列建立統計資料。 這稱為篩選的統計資料。

例如，當您計劃查詢大型分割資料表的特定分割時，可以使用篩選的統計資料。 只對分割值建立統計資料，統計資料的精確度將會改善，並因而改善查詢效能。

這個範例會建立某個值範圍的統計資料。 您可以輕鬆地定義這些值以符合分割中的值範圍。

```sql
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> 若要讓查詢最佳化工具在選擇分散式查詢計劃時考慮使用篩選的統計資料，查詢必須符合統計資料物件的定義。 使用上述範例，查詢的 WHERE 子句需要指定介於 2000101 和 20001231 之間的 col1 值。

### <a name="create-single-column-statistics-with-all-the-options"></a>使用所有選項建立單一資料行統計資料

您也可以將選項結合在一起。 以下範例會使用自訂樣本大小建立篩選的統計資料物件：

```sql
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

如需完整參考，請參閱 [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。

### <a name="create-multi-column-statistics"></a>建立多重資料行統計資料

若要建立多重資料行統計資料物件，請利用上述範例，但要指定更多資料行。

> [!NOTE]
> 用來估計查詢結果中資料列數目的長條圖，只適用於統計資料物件定義中所列的第一個資料行。

在此範例中，長條圖位於 *product\_category*。 跨資料行統計資料會依據 *product\_category* 和 *product\_sub_category* 計算：

```sql
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

因為 product\_category 和 product\_sub\_category 之間有關聯性，所以多重資料行統計資料物件在同時存取這些資料行時相當實用。

### <a name="create-statistics-on-all-columns-in-a-table"></a>對資料表中的所有資料行建立統計資料

建立統計資料的其中一個方法是在建立資料表後發出 CREATE STATISTICS 命令：

```sql
CREATE TABLE dbo.table1
(
   col1 int
,  col2 int
,  col3 int
)
WITH
  (
    CLUSTERED COLUMNSTORE INDEX
  )
;

CREATE STATISTICS stats_col1 on dbo.table1 (col1);
CREATE STATISTICS stats_col2 on dbo.table2 (col2);
CREATE STATISTICS stats_col3 on dbo.table3 (col3);
```

### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-sql-pool"></a>使用預存程式來建立 SQL 集區中所有資料行的統計資料

專用的 SQL 集區沒有相當於 SQL Server 中 sp_create_stats 的系統預存程式。 這個預存程式會在 SQL 集區中的每個資料行上建立單一資料行統計資料物件，而該資料集還沒有統計資料

下列範例將協助您開始使用 SQL 集區設計。 請放心地依照您的需求進行調整。

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
,   @sample_pct     tinyint
)
AS

IF @create_type IS NULL
BEGIN
    SET @create_type = 1;
END;

IF @create_type NOT IN (1,2,3)
BEGIN
    THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN;
    DROP TABLE #stats_ddl;
END;

CREATE TABLE #stats_ddl
WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
        ,   LOCATION        = USER_DB
        )
AS
WITH T
AS
(
SELECT      t.[name]                        AS [table_name]
,           s.[name]                        AS [table_schema_name]
,           c.[name]                        AS [column_name]
,           c.[column_id]                   AS [column_id]
,           t.[object_id]                   AS [object_id]
,           ROW_NUMBER()
            OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
FROM        sys.[tables] t
JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                    AND l.[column_id]       = c.[column_id]
                                    AND l.[stats_column_id] = 1
LEFT JOIN    sys.[external_tables] e    ON    e.[object_id]        = t.[object_id]
WHERE       l.[object_id] IS NULL
AND            e.[object_id] IS NULL -- not an external table
)
SELECT  [table_schema_name]
,       [table_name]
,       [column_name]
,       [column_id]
,       [object_id]
,       [seq_nmbr]
,       CASE @create_type
        WHEN 1
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
        WHEN 2
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
        WHEN 3
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+CONVERT(varchar(4),@sample_pct)+' PERCENT' AS VARCHAR(8000))
        END AS create_stat_ddl
FROM T
;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
;

WHILE @i <= @t
BEGIN
    SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

若要使用預設值為資料表中的所有資料行建立統計資料，請執行預存程序。

```sql
EXEC [dbo].[prc_sqldw_create_stats] 1, NULL;
```

若要使用 fullscan 來建立資料表中所有資料行的統計資料，請呼叫此程式。

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

若要為資料表中的所有資料行建立取樣的統計資料，請輸入 3 和取樣百分比。 此程式使用20% 的取樣率。

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

## <a name="examples-update-statistics"></a>範例：更新統計資料

若要更新統計資料，您可以：

- 更新一個統計資料物件。 指定您要更新的統計資料物件名稱。
- 更新資料表上的所有統計資料物件。 指定資料表名稱，而不是一個特定統計資料物件。

### <a name="update-one-specific-statistics-object"></a>更新一個特定統計資料物件

使用下列語法來更新特定統計資料物件：

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

例如：

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

藉由更新特定統計資料物件，即可減少管理統計資料所需的時間和資源。 這麼做需要考慮選擇要更新的最佳統計資料物件。

### <a name="update-all-statistics-on-a-table"></a>更新資料表的所有統計資料

更新資料表上所有統計資料物件的簡單方法為：

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

例如：

```sql
UPDATE STATISTICS dbo.table1;
```

UPDATE STATISTICS 陳述式易於使用。 只要記住這會更新資料表上的所有統計資料，因此可能會執行超出所需的更多工作。 如果效能不成問題，這是確保統計資料為最新狀態的最簡單且最完整的方式。

> [!NOTE]
> 更新資料表上的所有統計資料時，專用的 SQL 集區會進行掃描，以針對每個統計資料物件來取樣資料表。 如果資料表很大，而且有許多資料行以及許多統計資料，則根據需求來更新個別統計資料可能比較有效率。

如需執行程式的相關 `UPDATE STATISTICS` 步驟，請參閱 [臨時表](sql-data-warehouse-tables-temporary.md)。 實作方法與上述的 `CREATE STATISTICS` 程序有點不同，但結果相同。

如需完整的語法，請參閱 [更新統計資料](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。

## <a name="statistics-metadata"></a>統計資料中繼資料

您可利用數個系統檢視和函式來尋找統計資料相關資訊。 例如，使用 stats-date 函式來查看最後建立或更新統計資料的時間，即可查看統計資料物件是否可能過期。

### <a name="catalog-views-for-statistics"></a>統計資料的目錄檢視

這些系統檢視提供統計資料的相關資訊：

| 目錄檢視 | 描述 |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |每個資料行有一個資料列。 |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |資料庫中每個物件有一個資料列。 |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |資料庫中每個結構描述有一個資料列。 |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |每個統計資料物件有一個資料列。 |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |統計資料物件中每個資料行有一個資料列。 連結回到 sys.columns。 |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |每個資料表 (包括外部資料表) 有一個資料列。 |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |每個資料類型有一個資料列。 |

### <a name="system-functions-for-statistics"></a>統計資料的系統函式

這些系統函式很適合用於處理統計資料：

| 系統函式 | 描述 |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |上次更新統計資料物件的日期。 |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |有關統計資料物件所理解之值散發的摘要層級和詳細資訊。 |

### <a name="combine-statistics-columns-and-functions-into-one-view"></a>將統計資料資料行和函式結合成一個檢視

此檢視會一起顯示與統計資料相關的資料行，以及 STATS_DATE() 函式的結果。

```sql
CREATE VIEW dbo.vstats_columns
AS
SELECT
        sm.[name]                           AS [schema_name]
,       tb.[name]                           AS [table_name]
,       st.[name]                           AS [stats_name]
,       st.[filter_definition]              AS [stats_filter_definition]
,       st.[has_filter]                     AS [stats_is_filtered]
,       STATS_DATE(st.[object_id],st.[stats_id])
                                            AS [stats_last_updated_date]
,       co.[name]                           AS [stats_column_name]
,       ty.[name]                           AS [column_type]
,       co.[max_length]                     AS [column_max_length]
,       co.[precision]                      AS [column_precision]
,       co.[scale]                          AS [column_scale]
,       co.[is_nullable]                    AS [column_is_nullable]
,       co.[collation_name]                 AS [column_collation_name]
,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS two_part_name
,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS three_part_name
FROM    sys.objects                         AS ob
JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                            AND         st.[object_id]      = sc.[object_id]
JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                            AND         sc.[object_id]      = co.[object_id]
JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

## <a name="dbcc-show_statistics-examples"></a>DBCC SHOW_STATISTICS() 範例

DBCC SHOW_STATISTICS() 顯示統計資料物件中保存的資料。 此資料來自三個部分：

- 頁首
- 密度向量
- 長條圖

有關統計資料的標頭中繼資料。 此長條圖會顯示統計資料物件的第一個索引鍵資料行中的值散發。 密度向量可測量跨資料行關聯性。

> [!NOTE]
> 專用的 SQL 集區會使用統計資料物件中的任何資料來計算基數估計值。

### <a name="show-header-density-and-histogram"></a>顯示標頭、密度和長條圖

這個簡單範例顯示統計資料物件的所有三個部分：

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

例如：

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>顯示 DBCC SHOW_STATISTICS() 的一或多個部分

如果您只想要檢視特定部分，請使用 `WITH` 子句並指定您要查看哪些部分：

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

例如：

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## <a name="dbcc-show_statistics-differences"></a>DBCC SHOW_STATISTICS() 差異

相較于 SQL Server，DBCC SHOW_STATISTICS ( # A1 會更嚴格地在專用的 SQL 集區中執行：

- 不支援未記載的功能。
- 無法使用 Stats_stream。
- 無法聯結特定統計資料子集的結果。 例如，STAT_HEADER JOIN DENSITY_VECTOR。
- 無法針對訊息隱藏項目設定 NO_INFOMSGS。
- 無法使用統計資料名稱前後的方括弧。
- 無法使用資料行名稱來識別統計資料物件。
- 不支援自訂錯誤 2767。

## <a name="next-steps"></a>後續步驟

如需進一步改善查詢效能，請參閱[監視工作負載](sql-data-warehouse-manage-monitor.md)
