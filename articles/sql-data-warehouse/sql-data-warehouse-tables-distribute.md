---
title: 分散式資料表設計指引 - Azure SQL 資料倉儲 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中設計雜湊分散式資料表和循環配置資源分散式資料表的建議。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: 36db91cd7c4dad3c28c0c110ee837ca6d1284959
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45575371"
---
# <a name="guidance-for-designing-distributed-tables-in-azure-sql-data-warehouse"></a>在 Azure SQL 資料倉儲中設計分散式資料表的指引
在 Azure SQL 資料倉儲中設計雜湊分散式資料表和循環配置資源分散式資料表的建議。

本文假設您已熟悉「SQL 資料倉儲」中的資料散發和資料移動概念。  如需詳細資訊，請參閱[Azure SQL 資料倉儲 - 大量平行處理 (MPP) 架構](massively-parallel-processing-mpp-architecture.md)。 

## <a name="what-is-a-distributed-table"></a>什麼是分散式資料表？
分散式資料表會顯示為單一資料表，但資料列實際上會儲存在 60 個散發。 這些資料列是透過雜湊或循環配置資源演算法來散發。  

**雜湊分散式資料表**是本文的重點，其可改善大型事實資料表的查詢效能。 **循環配置資源資料表**可用於改善載入速度。 這些設計選擇對於改善查詢和載入效能有顯著的影響。

另一個資料表儲存體選項是將小型資料表複寫到所有計算節點。 如需詳細資訊，請參閱[複寫資料表的設計指引](design-guidance-for-replicated-tables.md)。 若要在三個選項中快速做選擇，請參閱[資料表概觀](sql-data-warehouse-tables-overview.md)中的分散式資料表。 

在資料表設計過程中，請儘可能了解您的資料及查詢資料的方式。  例如，請思考一下下列問題：

- 資料表的大小為何？   
- 資料表的重新整理頻率為何？   
- 我是否在資料倉儲中有事實資料表和維度資料表？   


### <a name="hash-distributed"></a>雜湊分散式
雜湊分散式資料表會使用確定性雜湊函式，將每個資料列指派給一個[散發](massively-parallel-processing-mpp-architecture.md#distributions)，藉此將資料表的資料列散發於計算節點。 

![分散式資料表](media/sql-data-warehouse-distributed-data/hash-distributed-table.png "分散式資料表")  

因為相同的值一律會雜湊到相同的散發，所以資料倉儲具有資料列位置的內建知識。 SQL 資料倉儲會使用此知識來將查詢期間的資料移動降到最低，進而改善查詢效能。 

雜湊分散式資料表適合用於處理星型結構描述中的大型事實資料表。 這類資料表可能有非常大量的資料列，但仍可達到高效能。 當然，也會有一些設計考量可協助您取得分散式系統設計所要提供的效能。 選擇良好的散發資料行是這類考量的其中一項 (敘述於本文中)。 

請在下列時機考慮使用雜湊分散式資料表：

- 磁碟上的資料表大小為 2 GB 以上。
- 資料表有頻繁的插入、更新及刪除作業。 

### <a name="round-robin-distributed"></a>循環配置資源分散式
循環配置資源分散式資料表會將資料表的資料列平均散發於所有散發。 散發的資料列指派是隨機的。 不同於雜湊分散式資料表，具有相等值的資料列不保證會指派給相同的散發。 

因此，系統有時候需要叫用資料移動作業，才能在解析查詢前，更加妥善地組織您的資料。  此額外步驟會使您的查詢變慢。 例如，加入循環配置資源資料表時通常需要重組資料列，而這會影響效能。

在下列情況下，請考慮對資料表使用循環配置資源散發：

- 以簡單的起點開始使用時 (因為這是預設值)
- 如果沒有明顯的聯結索引鍵
- 如果沒有合適的候選資料行可供雜湊散發資料表
- 如果資料表並未與其他資料表共用常見的聯結索引鍵
- 如果此聯結比查詢中的其他聯結較不重要
- 當資料表是暫存預備資料表時

[將紐約計程車資料載入 Azure SQL 資料倉儲](load-data-from-azure-blob-storage-using-polybase.md#load-the-data-into-your-data-warehouse)教學課程會提供範例，示範如何將資料載入循環配置資源的暫存資料表。


## <a name="choosing-a-distribution-column"></a>選擇散發資料行
雜湊分散式資料表中有一個散發資料行，也就是雜湊索引鍵。 例如，下列程式碼會建立以 ProductKey 作為散發資料行的雜湊分散式資料表。

```SQL
CREATE TABLE [dbo].[FactInternetSales]
(   [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,  DISTRIBUTION = HASH([ProductKey])
)
;
``` 

選擇散發資料行是很重要的設計決策，因為此資料行中的值會決定資料列的散發方式。 最佳選擇取決於許多因素，通常需要權衡取捨。 不過，如果您未在第一次就選擇最佳資料行，您可以使用 [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 來重建具有不同散發資料行的資料表。 

### <a name="choose-a-distribution-column-that-does-not-require-updates"></a>選擇不需要更新的散發資料行
除非您刪除資料列並以更新後的值插入新資料列，否則無法更新散發資料行。 因此，請選取具有靜態值的資料行。 

### <a name="choose-a-distribution-column-with-data-that-distributes-evenly"></a>選擇資料平均散發的散發資料行

為了達到最佳效能，所有散發應有大約相同的資料列數。 當一個或多個散發的資料列數目不均衡時，有些散發會比其他散發先完成其平行查詢的部分。 查詢必須等到所有散發都完成處理才能完成，因此每個查詢只能跟最慢的散發一樣快。

- 資料扭曲表示資料不平均地散發於散發
- 處理偏斜表示某些散發需要比其他散發更長的時間來執行平行查詢。 當資料有所扭曲時，就可能發生這種情形。
  
若要使平行處理達到平衡，請選取具有下列條件的散發資料行：

- **有許多唯一值。** 資料行可能有一些重複值。 然而，所有具有相同值的資料列都會指派至相同的散發。 由於有 60 個散發，因此資料行應至少有 60 個唯一值。  唯一值的數目通常會更大。
- **沒有 Null，或只有少數 Null。** 舉一個極端的例子，如果資料行中所有的值都是 NULL，則所有資料列都會指派至相同的散發。 如此一來，查詢處理就會偏斜至某一個散發，因而失去平行處理的好處。 
- **不是日期資料行**。 日期相同的所有資料都會落在同一個散發。 如果以相同的日期上篩選出多位使用者，則 60 個散發中只有 1 個散發會進行所有處理工作。 

### <a name="choose-a-distribution-column-that-minimizes-data-movement"></a>選擇可將資料移動降到最低的散發資料行

為了取得正確的查詢結果，查詢可能會在計算節點間移動資料。 當查詢有分散式資料表的聯結和彙總時，通常會發生資料移動。 選取有助於將資料移動降至最低的散發資料行，是讓 SQL 資料倉儲的效能達到最佳化的最重要策略之一。

若要將資料移動降至最低，請選取具有下列條件的散發資料行：

- 在 `JOIN`、`GROUP BY`、`DISTINCT`、`OVER` 和 `HAVING` 子句中使用。 當兩個大型事實資料表有頻繁的聯結時，如果您在其中一個聯結資料行上散發這兩個資料表，即可改善查詢效能。  當資料表未使用於聯結時，請考慮在經常出現於 `GROUP BY` 子句中的資料行上散發資料表。
- 「不」在 `WHERE` 子句中使用。 這可能會縮小查詢範圍，使其無法在所有散發上執行。 
- 「不」是日期資料行。 WHERE 子句通常會依日期篩選。  在這種情況下，所有處理都只能在少數散發上執行。

### <a name="what-to-do-when-none-of-the-columns-are-a-good-distribution-column"></a>沒有資料行是理想的散發資料行時該怎麼辦

若您的資料行都沒有散發資料行所需的足夠相異值，您可以建立新資料行以作為一或多個值的複合。 為避免查詢執行期間的資料移動，請在查詢中使用複合散發資料行作為聯結資料行。

設計雜湊分散式資料表後，下一個步驟是將資料載入資料表中。  如需載入指引，請參閱[載入概觀](sql-data-warehouse-overview-load.md)。 

## <a name="how-to-tell-if-your-distribution-column-is-a-good-choice"></a>如何分辨您的散發資料行是否為理想的選擇
將資料載入雜湊分散式資料表之後，請查看資料列如何平均散發於 60 個散發。 每個發佈區的資料列最多可有 10% 的變化，而且不會對效能產生顯著影響。 

### <a name="determine-if-the-table-has-data-skew"></a>判斷資料表是否有資料扭曲
快速檢查資料扭曲的方法是使用 [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql)。 下列 SQL 程式碼會傳回在 60 個散發的每個散發中儲存的資料表資料列數目。 為了達到平衡的效能，分散式資料表中的資料列應平均散發於所有散發。

```sql
-- Find data skew for a distributed table
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

若要識別哪些資料表有 10% 以上的資料扭曲：

1. 建立[資料表概觀](sql-data-warehouse-tables-overview.md#table-size-queries)一文中所示的 dbo.vTableSizes 檢視。  
2. 請執行下列查詢：

```sql
select *
from dbo.vTableSizes
where two_part_name in
    (
    select two_part_name
    from dbo.vTableSizes
    where row_count > 0
    group by two_part_name
    having min(row_count * 1.000)/max(row_count * 1.000) > .10
    )
order by two_part_name, row_count
;
```

### <a name="check-query-plans-for-data-movement"></a>檢查資料移動的查詢計劃
理想的散發資料行可讓聯結和彙總具有最低的資料移動。 這會影響聯結的撰寫方式。 若要讓兩個雜湊分散式資料表上的聯結具有最低的資料移動，其中一個聯結資料行必須是散發資料行。  如果有兩個雜湊分散式資料表在相同資料類型的散發資料行上聯結，該聯結不需要移動資料。 聯結可以使用其他資料行，而不會造成資料移動。

若要避免在聯結期間發生資料移動：

- 參與聯結之資料行的相關資料表必須雜湊散發於其中 **一個** 聯結資料行上。
- 兩個資料表間聯結資料行的資料類型必須相符。
- 資料行必須以 equals 運算子聯結。
- 聯結類型可能不是 `CROSS JOIN`。

若要查看查詢是否遇到資料移動，您可以查看查詢計劃。  


## <a name="resolve-a-distribution-column-problem"></a>解決散發資料行問題
您不需要解決所有的資料扭曲情況。 散發資料就是找出將資料扭曲降至最低與將資料移動降至最低兩者之間的適當平衡。 您不一定能夠同時將資料扭曲和資料移動降低最低。 有時候，具有最少資料移動的好處可能勝過具有資料扭曲的影響。

若要決定是否應該解決資料表中的資料扭曲，您應該盡可能了解工作負載中的資料磁區和查詢。 您可以使用[查詢監視](sql-data-warehouse-manage-monitor.md)一文中的步驟，來監視查詢效能偏斜的影響。 具體而言，就是找出在個別散發上完成大型查詢所花費的時間。

由於您無法變更現有資料表上的散發資料行，所以解決資料扭曲的典型方式就是重建具有不同散發資料行的資料表。  

### <a name="re-create-the-table-with-a-new-distribution-column"></a>重建具有新散發資料行的資料表
此範例會使用 [CREATE TABLE AS SELECT](https://docs.microsoft.com/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=aps-pdw-2016-au7) 來重建具有不同雜湊散發資料行的資料表。

```sql
CREATE TABLE [dbo].[FactInternetSales_CustomerKey]
WITH (  CLUSTERED COLUMNSTORE INDEX
     ,  DISTRIBUTION =  HASH([CustomerKey])
     ,  PARTITION       ( [OrderDateKey] RANGE RIGHT FOR VALUES (   20000101, 20010101, 20020101, 20030101
                                                                ,   20040101, 20050101, 20060101, 20070101
                                                                ,   20080101, 20090101, 20100101, 20110101
                                                                ,   20120101, 20130101, 20140101, 20150101
                                                                ,   20160101, 20170101, 20180101, 20190101
                                                                ,   20200101, 20210101, 20220101, 20230101
                                                                ,   20240101, 20250101, 20260101, 20270101
                                                                ,   20280101, 20290101
                                                                )
                        )
    )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
OPTION  (LABEL  = 'CTAS : FactInternetSales_CustomerKey')
;

--Create statistics on new table
CREATE STATISTICS [ProductKey] ON [FactInternetSales_CustomerKey] ([ProductKey]);
CREATE STATISTICS [OrderDateKey] ON [FactInternetSales_CustomerKey] ([OrderDateKey]);
CREATE STATISTICS [CustomerKey] ON [FactInternetSales_CustomerKey] ([CustomerKey]);
CREATE STATISTICS [PromotionKey] ON [FactInternetSales_CustomerKey] ([PromotionKey]);
CREATE STATISTICS [SalesOrderNumber] ON [FactInternetSales_CustomerKey] ([SalesOrderNumber]);
CREATE STATISTICS [OrderQuantity] ON [FactInternetSales_CustomerKey] ([OrderQuantity]);
CREATE STATISTICS [UnitPrice] ON [FactInternetSales_CustomerKey] ([UnitPrice]);
CREATE STATISTICS [SalesAmount] ON [FactInternetSales_CustomerKey] ([SalesAmount]);

--Rename the tables
RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_ProductKey];
RENAME OBJECT [dbo].[FactInternetSales_CustomerKey] TO [FactInternetSales];
```

## <a name="next-steps"></a>後續步驟

若要建立分散式資料表，請使用下列其中一個陳述式：

- [CREATE TABLE (Azure SQL 資料倉儲)](https://docs.microsoft.com/sql/t-sql/statements/create-table-azure-sql-data-warehouse)
- [CREATE TABLE AS SELECT (Azure SQL 資料倉儲)](https://docs.microsoft.com/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse)


