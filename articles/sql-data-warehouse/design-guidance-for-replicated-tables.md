---
title: 複寫資料表設計指引 - Azure SQL 資料倉儲 | Microsoft Docs
description: 針對在「Azure SQL 資料倉儲」結構描述中設計複寫資料表提供建議。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/23/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: dfbfc61b9088535d6b50a9897b908572d88d6676
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302757"
---
# <a name="design-guidance-for-using-replicated-tables-in-azure-sql-data-warehouse"></a>在 Azure SQL 資料倉儲中使用複寫資料表的設計指引
本文針對在「SQL 資料倉儲」結構描述中設計複寫資料表提供建議。 您可以使用這些建議來降低資料移動和查詢的複雜性，以提升查詢效能。

> [!VIDEO https://www.youtube.com/embed/1VS_F37GI9U]

## <a name="prerequisites"></a>必要條件
本文假設您已熟悉「SQL 資料倉儲」中的資料散發和資料移動概念。  如需詳細資訊，請參閱[架構](massively-parallel-processing-mpp-architecture.md)文章。 

在資料表設計過程中，請儘可能了解您的資料及查詢資料的方式。  例如，請思考一下下列問題：

- 資料表的大小為何？   
- 資料表的重新整理頻率為何？   
- 我是否在資料倉儲中有事實資料表和維度資料表？   

## <a name="what-is-a-replicated-table"></a>什麼是複寫資料表？
複寫資料表在每個計算節點上都有一份可存取的完整資料表複本。 複寫資料表可使在進行聯結或彙總之前，不需要在計算節點之間傳輸資料。 由於資料表有多個複本，因此當資料表大小在壓縮後小於 2 GB 時，複寫資料表的運作效能最佳。

下圖顯示每個計算節點上可存取的複寫資料表。 在「SQL 資料倉儲」中，會將複寫資料表完整複製到每個計算節點上的散發資料庫。 

![複寫的資料表](media/guidance-for-using-replicated-tables/replicated-table.png "複寫的資料表")  

複寫資料表適用於星型結構描述中的小型維度資料表。 維度資料表的大小通常是適合儲存及維護多個複本的大小。 維度會儲存變更緩慢的描述性資料，例如客戶名稱和地址，以及產品詳細資料。 資料的變更緩慢特性導致較少發生重建複寫資料表的情況。 

在下列情況下，請考慮使用複寫資料表：

- 磁碟上的資料表大小小於 2 GB，不論它有幾個資料列。 若要了解資料表的大小，您可以使用 [DBCC PDW_SHOWSPACEUSED](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql) 命令：`DBCC PDW_SHOWSPACEUSED('ReplTableCandidate')`。 
- 資料表用於需要進行資料移動的聯結中。 加入未分佈在同一資料行的資料表時 (例如雜湊分散式資料表至循環配置資源資料表)，需要移動資料才能完成查詢。  如果其中一個資料表是小型資料表，請考慮使用複寫資料表。 建議您在大多數情況下都使用複寫資料表，而不要使用循環配置資源資料表。 若要檢視查詢計劃中的資料移動作業，請使用 [sys.dm_pdw_request_steps](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql)。  BroadcastMoveOperation 是典型的資料移動作業，可以使用複寫資料表刪除。  
 
在下列情況下，複寫資料表可能無法產生最佳查詢效能：

- 資料表有頻繁的插入、更新及刪除作業。 這些資料操作語言 (DML) 作業需要重建複寫資料表。 經常重建會導致效能變差。
- 經常調整資料倉儲。 調整資料倉儲會變更計算節點數目，進而導致重建。
- 資料表有大量資料行，但資料作業通常只存取少數資料行。 在此情況下散發資料表，然後針對經常存取的資料行建立索引，可能會比複寫整個資料表還要有效。 當查詢需要進行資料移動時，「SQL 資料倉儲」只會移動所要求資料行中的資料。 

## <a name="use-replicated-tables-with-simple-query-predicates"></a>使用複寫資料表搭配簡單查詢述詞
在您選擇是要散發還是複寫資料表之前，請先思考您打算對資料表執行的查詢類型。 請儘可能

- 針對含有簡單查詢述詞 (例如相等比較或不等比較) 的查詢使用複寫資料表。
- 針對含有複雜查詢述詞 (例如相似或不相似) 的查詢使用分散式資料表。

需要大量 CPU 的查詢在工作分散於所有計算節點時效能最佳。 例如，以在資料表的每個資料列上執行計算的查詢來說，在複寫資料表上執行會比在分散式資料表上執行效能更佳。 由於複寫資料表是完整儲存在每個計算節點上，因此對複寫資料表執行需要大量 CPU 的查詢時，執行對象會是每個計算節點上的整個資料表。 額外的計算會降低查詢效能。

例如，此查詢含有複雜的述詞。  當供應者是分散式資料表而不是複寫資料表時，它的執行速度較快。 在此範例中，供應者可以是循環配置資源分散式資料表。

```sql

SELECT EnglishProductName 
FROM DimProduct 
WHERE EnglishDescription LIKE '%frame%comfortable%'

```

## <a name="convert-existing-round-robin-tables-to-replicated-tables"></a>將現有的循環配置資源資料表轉換成分散式資料表
如果您已經有循環配置資源資料表，而且它們符合本文中所述的準則，建議您將它們轉換成複寫資料表。 複寫資料表可提升循環配置資源資料表的效能，因為它們不需進行資料移動。  循環配置資源資料表針對聯結一律需要進行資料移動。 

此範例使用 [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 將 DimSalesTerritory 資料表變更為複寫資料表。 不論 DimSalesTerritory 是雜湊分散式還是循環配置資源資料表，此範例都有效。

```sql
CREATE TABLE [dbo].[DimSalesTerritory_REPLICATE]   
WITH   
  (   
    CLUSTERED COLUMNSTORE INDEX,  
    DISTRIBUTION = REPLICATE  
  )  
AS SELECT * FROM [dbo].[DimSalesTerritory]
OPTION  (LABEL  = 'CTAS : DimSalesTerritory_REPLICATE') 

--Create statistics on new table
CREATE STATISTICS [SalesTerritoryKey] ON [DimSalesTerritory_REPLICATE] ([SalesTerritoryKey]);
CREATE STATISTICS [SalesTerritoryAlternateKey] ON [DimSalesTerritory_REPLICATE] ([SalesTerritoryAlternateKey]);
CREATE STATISTICS [SalesTerritoryRegion] ON [DimSalesTerritory_REPLICATE] ([SalesTerritoryRegion]);
CREATE STATISTICS [SalesTerritoryCountry] ON [DimSalesTerritory_REPLICATE] ([SalesTerritoryCountry]);
CREATE STATISTICS [SalesTerritoryGroup] ON [DimSalesTerritory_REPLICATE] ([SalesTerritoryGroup]);

-- Switch table names
RENAME OBJECT [dbo].[DimSalesTerritory] to [DimSalesTerritory_old];
RENAME OBJECT [dbo].[DimSalesTerritory_REPLICATE] TO [DimSalesTerritory];

DROP TABLE [dbo].[DimSalesTerritory_old];
```  

### <a name="query-performance-example-for-round-robin-versus-replicated"></a>循環配置資源與複寫之查詢效能比較範例 

複寫資料表針對聯結不需要進行任何資料移動，因為整個資料表已經存在於每個計算節點上。 如果維度資料表是循環配置資源分散式資料表，聯結就會將整個維度資料表複製到每個計算節點。 為了移動資料，查詢計劃會包含一個名為 BroadcastMoveOperation 的作業。 這類資料移動作業會降低查詢效能，而使用複寫資料表則可免除這類作業。 若要檢視查詢計劃步驟，請使用 [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql) 系統目錄檢視。 

例如，在以下針對 AdventureWorks 結構描述執行的查詢中，` FactInternetSales` 資料表是雜湊分散式資料表。 `DimDate` 和 `DimSalesTerritory` 資料表是較小型的維度資料表。 此查詢會傳回 2004 會計年度北美的總銷售額：
 
```sql
SELECT [TotalSalesAmount] = SUM(SalesAmount)
FROM dbo.FactInternetSales s
INNER JOIN dbo.DimDate d
  ON d.DateKey = s.OrderDateKey
INNER JOIN dbo.DimSalesTerritory t
  ON t.SalesTerritoryKey = s.SalesTerritoryKey
WHERE d.FiscalYear = 2004
  AND t.SalesTerritoryGroup = 'North America'
```
我們已將 `DimDate` 和 `DimSalesTerritory` 重新建立成循環配置資源資料表。 因此，此查詢顯示了以下查詢計劃，其中包含多個廣播移動作業： 
 
![循環配置資源的查詢計劃](media/design-guidance-for-replicated-tables/round-robin-tables-query-plan.jpg) 

我們已將 `DimDate` 和 `DimSalesTerritory` 重新建立成複寫資料表，並已重新執行查詢。 產生的查詢計劃長度變短許多，且沒有任何廣播移動。

![複寫的查詢計劃](media/design-guidance-for-replicated-tables/replicated-tables-query-plan.jpg) 


## <a name="performance-considerations-for-modifying-replicated-tables"></a>修改複寫資料表時的效能考量
「SQL 資料倉儲」是透過維護資料表的主要版本來實作複寫資料表。 它會將主要版本複製到每個計算節點上的一個散發資料庫。 當發生變更時，「SQL 資料倉儲」會先更新主資料表。 接著，它會重建每個計算節點上的資料表。 重建複寫資料表包括將資料表複製到每個計算節點，然後建立索引。  例如，DW400 上的複寫資料表有 5 份資料。  主要複本以及每個計算節點上的完整複本。  所有資料都會儲存在散發資料庫中。 SQL 資料倉儲會使用這個模型支援更快的資料修改陳述式和彈性調整作業。 

在執行下列動作之後，必須進行重建：
- 載入或修改資料
- 將資料倉儲調整成不同的等級
- 更新資料表定義

在執行下列動作之後，不須進行重建：
- 暫停作業
- 繼續作業

在修改資料之後，不會立即進行重建。 取而代之的是，會在查詢從資料表選取資料時觸發重建。  以非同步方式將資料複製到每個計算節點時，觸發重建的查詢會立即讀取主要版本的資料表。 資料複製完成之前，後續的查詢將會繼續使用主要版本的資料表。  如果對強制執行另一次重建的複寫資料表發生任何活動，則資料複本將失效，且下一個 Select 陳述式將會再觸發一次資料複製。 

### <a name="use-indexes-conservatively"></a>謹慎地使用索引
標準索引編製做法適用於複寫資料表。 「SQL 資料倉儲」會在重建時，一併重建每個複寫資料表索引。 請只有在效能的提升超過重建索引的代價時，才使用索引。  
 
### <a name="batch-data-loads"></a>批次資料載入
將資料載入複寫資料表時，請嘗試一起批次載入，以將重建次數降到最低。 請在執行 select 陳述式之前，先執行所有批次處理的載入。

例如，以下載入模式會從四個來源載入資料，然後叫用四次重建。 

- 從來源 1 載入。
- Select 陳述式觸發重建 1。
- 從來源 2 載入。
- Select 陳述式觸發重建 2。
- 從來源 3 載入。
- Select 陳述式觸發重建 3。
- 從來源 4 載入。
- Select 陳述式觸發重建 4。

例如，以下載入模式會從四個來源載入資料，但只會叫用一次重建。

- 從來源 1 載入。
- 從來源 2 載入。
- 從來源 3 載入。
- 從來源 4 載入。
- Select 陳述式觸發重建。


### <a name="rebuild-a-replicated-table-after-a-batch-load"></a>在批次載入後重建複寫資料表
為了確保查詢執行時間一致，請在批次載入後，考慮強制建立複寫資料表。 否則，第一個查詢仍然會使用資料移動方式來完成查詢。 

此查詢使用 [sys.pdw_replicated_table_cache_state](/sql/relational-databases/system-catalog-views/sys-pdw-replicated-table-cache-state-transact-sql) DMV 來列出已修改但未重建的複寫資料表。

```sql 
SELECT [ReplicatedTable] = t.[name]
  FROM sys.tables t  
  JOIN sys.pdw_replicated_table_cache_state c  
    ON c.object_id = t.object_id 
  JOIN sys.pdw_table_distribution_properties p 
    ON p.object_id = t.object_id 
  WHERE c.[state] = 'NotReady'
    AND p.[distribution_policy_desc] = 'REPLICATE'
```
 
若要觸發重建，請針對上面輸出中的每個資料表執行下列陳述式。 

```sql
SELECT TOP 1 * FROM [ReplicatedTable]
``` 
 
## <a name="next-steps"></a>後續步驟 
若要建立複寫資料表，請使用下列其中一個陳述式：

- [CREATE TABLE (Azure SQL 資料倉儲)](/sql/t-sql/statements/create-table-azure-sql-data-warehouse)
- [CREATE TABLE AS SELECT (Azure SQL 資料倉儲)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse)

如需分散式資料表的概觀，請參閱[分散式資料表](sql-data-warehouse-tables-distribute.md)。



