---
title: SQL 資料倉儲中的暫存資料表 | Microsoft Docs
description: 使用暫存資料表的基本指引，並強調說明工作階段層級暫存資料表的原則。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: f8eb401d8bc4f348be3c84390d3422571bebe444
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307320"
---
# <a name="temporary-tables-in-sql-data-warehouse"></a>SQL 資料倉儲中的暫存資料表
本文包含使用暫存資料表的基本指引，並強調說明工作階段層級暫存資料表的原則。 使用這份文件中的資訊可協助您將程式碼模組化，以提高程式碼的重複使用性，維護起來更簡單。

## <a name="what-are-temporary-tables"></a>什麼是暫存資料表？
暫存資料表在處理資料時很有用 - 尤其是具有暫時性中繼結果的轉換期間。 在 SQL 資料倉儲中，暫存資料表存在於工作階段層級。  它們只出現在建立它們的工作階段中，工作階段登出時就會自動卸除它們。  暫存資料表的結果會寫入至本機，而不是遠端儲存體，這是它的效能優點。  Azure SQL 資料倉儲中的暫存資料表稍微不同於 Azure SQL Database，因為從工作階段內的任何地方都可存取它們，包括在預存程序的內部和外部。

## <a name="create-a-temporary-table"></a>建立暫存資料表
建立暫存資料表時會在資料表名稱前面加上 `#`。  例如︰

```sql
CREATE TABLE #stats_ddl
(
    [schema_name]        NVARCHAR(128) NOT NULL
,    [table_name]            NVARCHAR(128) NOT NULL
,    [stats_name]            NVARCHAR(128) NOT NULL
,    [stats_is_filtered]     BIT           NOT NULL
,    [seq_nmbr]              BIGINT        NOT NULL
,    [two_part_name]         NVARCHAR(260) NOT NULL
,    [three_part_name]       NVARCHAR(400) NOT NULL
)
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
```

`CTAS` 也可用來建立暫存資料表，方法完全相同：

```sql
CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
``` 

> [!NOTE]
> `CTAS` 是一個強大的命令，可有效率地使用交易記錄空間是它額外的好處。 
> 
> 

## <a name="dropping-temporary-tables"></a>捨棄暫存資料表
建立新的工作階段時，不應該存在任何暫存資料表。  不過，如果您呼叫同一個預存程序來建立具有相同名稱的暫存資料表，為了確保 `CREATE TABLE` 陳述式成功執行，可使用 `DROP` 進行簡單的預先存在性檢查，如下列範例所示︰

```sql
IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END
```

為了維持編寫程式碼的一致性，資料表和暫存資料表最好都採用此模式。  當您在程式碼中完成使用暫存資料表之後，使用 `DROP TABLE` 加以移除也是一個很好的做法。  在預存程序開發期間，在程序結尾一併搭配 drop 命令以確保會清除這些物件，也是常見的做法。

```sql
DROP TABLE #stats_ddl
```

## <a name="modularizing-code"></a>模組化程式碼
因為在使用者工作階段中的任何位置均可看見暫存資料表，這可用於協助您將應用程式程式碼模組化。  例如，下列預存程序會產生 DDL，根據統計資料名稱來更新資料庫中的所有統計資料。

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_update_stats]
(   @update_type    tinyint -- 1 default 2 fullscan 3 sample 4 resample
    ,@sample_pct     tinyint
)
AS

IF @update_type NOT IN (1,2,3,4)
BEGIN;
    THROW 151000,'Invalid value for @update_type parameter. Valid range 1 (default), 2 (fullscan), 3 (sample) or 4 (resample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END

CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
GO
```

在這個階段中，唯一進行的動作是建立預存程序，以 DDL 陳述式產生暫存資料表 #stats_ddl。  如果 #stats_ddl 已經存在，這個預存程序會卸除它，以確保在工作階段中執行一次以上時不會失敗。  不過，因為預存程序結尾沒有任何 `DROP TABLE`，當預存程序完成時，它會保留建立的資料表，以便能夠從預存程序之外讀取。  不同於其他 SQL Server 資料庫，在 SQL 資料倉儲中，從建立暫存資料表的程序之外能夠使用此暫存資料表。  工作階段內的 **任何位置** 都可以使用 SQL 資料倉儲暫存資料表。 這可以產生更具模組化和更易於管理的程式碼，如下列範例所示：

```sql
EXEC [dbo].[prc_sqldw_update_stats] @update_type = 1, @sample_pct = NULL;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''

WHILE @i <= @t
BEGIN
    SET @s=(SELECT update_stats_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

## <a name="temporary-table-limitations"></a>暫存資料表限制
SQL 資料倉儲在實作暫存資料表時的確有一些限制。  目前，僅支援工作階段範圍內的暫存資料表。  不支援全域暫存資料表。  此外，無法在暫存資料表上建立檢視。

## <a name="next-steps"></a>後續步驟
若要深入了解如何開發資料表，請參閱[資料表概觀](sql-data-warehouse-tables-overview.md)。

