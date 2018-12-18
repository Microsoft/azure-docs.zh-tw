---
title: Azure SQL 資料倉儲中的 CREATE TABLE AS SELECT (CTAS) | Microsoft Docs
description: 在 Azure SQL 資料倉儲中利用 CREATE TABLE AS SELECT (CTAS) 陳述式撰寫程式碼作為開發解決方案的提示。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: dad0b1570f54cde1b1d474d8ebfc78f793724ef4
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247748"
---
# <a name="using-create-table-as-select-ctas-in-azure-sql-data-warehouse"></a>在 Azure SQL 資料倉儲中使用 CREATE TABLE AS SELECT (CTAS)
在 Azure SQL 資料倉儲中利用 CREATE TABLE AS SELECT (CTAS) T-SQL 陳述式撰寫程式碼作為開發解決方案的提示。

## <a name="what-is-create-table-as-select-ctas"></a>什麼是 CREATE TABLE AS SELECT (CTAS)？

[CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 或 CTAS 陳述式是最重要的可用 T-SQL 功能之一。 該作業是根據 SELECT 陳述式的輸出來建立新資料表的平行作業。 CTASD 是建立資料表複本的最簡單且最快速方式。 

## <a name="selectinto-vs-ctas"></a>比較 SELECT..INTO 與CTAS
您可以將 CTAS 是為加強版的 [SELECT...INTO](/sql/t-sql/queries/select-into-clause-transact-sql) 陳述式。

以下範例是簡單的 SELECT..INTO：

```sql
SELECT *
INTO    [dbo].[FactInternetSales_new]
FROM    [dbo].[FactInternetSales]
```

在上述範例中，`[dbo].[FactInternetSales_new]` 會建立為包含 CLUSTERED COLUMNSTORE INDEX 的 ROUND_ROBIN 分散式資料表，因為這是 Azure SQL 資料倉儲的資料表預設值。

但是 SELECT..INTO 不允許在作業期間變更分散方法或索引類型。 此時便適合使用 CTAS。

要將前一個範例轉換成 CTAS 相當簡單：

```sql
CREATE TABLE [dbo].[FactInternetSales_new]
WITH
(
    DISTRIBUTION = ROUND_ROBIN
,   CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
;
```

您可以使用 CTAS，變更資料表資料的分散方法，以及資料表類型。 

> [!NOTE]
> 如果您嘗試在 `CTAS` 作業中變更索引，且來源資料表是雜湊分散，則維持相同的分散資料行和資料類型會使 `CTAS` 作業有最佳效能。 這樣可避免在作業期間有跨越分散資料移動，所以會有較佳效能。
> 
> 

## <a name="using-ctas-to-copy-a-table"></a>使用 CTAS 複製資料表
`CTAS` 最常見的用途之一，就是建立資料表複本，讓您可以變更 DDL。 例如，您原本將資料表建立為 `ROUND_ROBIN`，而現在想變更為在資料行上散發的資料表，`CTAS` 就是您將變更散發資料行的方式。 `CTAS` 也可以用來變更分割、索引或資料行類型。

假設您因為在 `ROUND_ROBIN` 中沒有指定散發資料行，而使用 `CREATE TABLE` 散發的預設散發類型建立資料表。

```sql
CREATE TABLE FactInternetSales
(
    ProductKey int NOT NULL,
    OrderDateKey int NOT NULL,
    DueDateKey int NOT NULL,
    ShipDateKey int NOT NULL,
    CustomerKey int NOT NULL,
    PromotionKey int NOT NULL,
    CurrencyKey int NOT NULL,
    SalesTerritoryKey int NOT NULL,
    SalesOrderNumber nvarchar(20) NOT NULL,
    SalesOrderLineNumber tinyint NOT NULL,
    RevisionNumber tinyint NOT NULL,
    OrderQuantity smallint NOT NULL,
    UnitPrice money NOT NULL,
    ExtendedAmount money NOT NULL,
    UnitPriceDiscountPct float NOT NULL,
    DiscountAmount float NOT NULL,
    ProductStandardCost money NOT NULL,
    TotalProductCost money NOT NULL,
    SalesAmount money NOT NULL,
    TaxAmt money NOT NULL,
    Freight money NOT NULL,
    CarrierTrackingNumber nvarchar(25),
    CustomerPONumber nvarchar(25)
);
```

現在您想建立此資料表的新複本搭配叢集的資料行存放區索引，讓您可以利用叢集的資料行存放區資料表的效能。 您也會想要在 ProductKey 上散發此資料表，因為您預期會聯結此資料行，並在聯結 ProductKey 期間避免資料移動。 最後您也會想在 OrderDateKey 上加入資料分割，如此就可以透過卸除舊的資料分割來快速刪除舊資料。 以下是 CTAS 陳述式，會將您的舊資料表複製到新資料表。

```sql
CREATE TABLE FactInternetSales_new
WITH
(
    CLUSTERED COLUMNSTORE INDEX,
    DISTRIBUTION = HASH(ProductKey),
    PARTITION
    (
        OrderDateKey RANGE RIGHT FOR VALUES
        (
        20000101,20010101,20020101,20030101,20040101,20050101,20060101,20070101,20080101,20090101,
        20100101,20110101,20120101,20130101,20140101,20150101,20160101,20170101,20180101,20190101,
        20200101,20210101,20220101,20230101,20240101,20250101,20260101,20270101,20280101,20290101
        )
    )
)
AS SELECT * FROM FactInternetSales;
```

最後可以重新命名您的資料表以切換到新的資料表，然後卸除舊資料表。

```sql
RENAME OBJECT FactInternetSales TO FactInternetSales_old;
RENAME OBJECT FactInternetSales_new TO FactInternetSales;

DROP TABLE FactInternetSales_old;
```

> [!NOTE]
> Azure 資料倉儲尚未支援自動建立或自動更新統計資料。  為了獲得查詢的最佳效能，在首次載入資料，或是資料中發生重大變更之後，建立所有資料表的所有資料行統計資料非常重要。  如需統計資料的詳細說明，請參閱「開發」主題群組中的[統計資料][統計資料]主題。
> 
> 

## <a name="using-ctas-to-work-around-unsupported-features"></a>使用 CTAS 解決不支援的功能
CTAS 也可以用來暫時解決以下幾個不支援的功能。 這通常可以證明是雙贏的情況，因為您的程式碼不但能夠相容，而且通常可以在 SQL 資料倉儲上更快速執行。 這是完全平行化設計的結果。 可以使用 CTAS 解決的案例包括：

* ANSI JOINS on UPDATEs
* ANSI JOINs on DELETEs
* MERGE 陳述式

> [!NOTE]
> 嘗試考慮「CTAS 優先」。 如果您認為您可以使用 `CTAS` 解決問題，即使您正在撰寫更多資料做為結果，這通常是最佳的解決方法。
> 
> 

## <a name="ansi-join-replacement-for-update-statements"></a>update 陳述式的 ANSI 聯結取代
您可能會發現有複雜的更新，使用 ANSI 聯結語法執行 UPDATE 或 DELETE 以將兩個以上的資料表聯結在一起。

假設您必須更新此資料表：

```sql
CREATE TABLE [dbo].[AnnualCategorySales]
(    [EnglishProductCategoryName]    NVARCHAR(50)    NOT NULL
,    [CalendarYear]                    SMALLINT        NOT NULL
,    [TotalSalesAmount]                MONEY            NOT NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN
)
;
```

原始的查詢看起來可能像這樣：

```sql
UPDATE    acs
SET        [TotalSalesAmount] = [fis].[TotalSalesAmount]
FROM    [dbo].[AnnualCategorySales]     AS acs
JOIN    (
        SELECT    [EnglishProductCategoryName]
        ,        [CalendarYear]
        ,        SUM([SalesAmount])                AS [TotalSalesAmount]
        FROM    [dbo].[FactInternetSales]        AS s
        JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
        JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
        JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
        JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
        WHERE     [CalendarYear] = 2004
        GROUP BY
                [EnglishProductCategoryName]
        ,        [CalendarYear]
        ) AS fis
ON    [acs].[EnglishProductCategoryName]    = [fis].[EnglishProductCategoryName]
AND    [acs].[CalendarYear]                = [fis].[CalendarYear]
;
```

由於 SQL 資料倉儲不支援在 `UPDATE` 陳述式的 `FROM` 子句中使用 ANSI 聯結，因此您無法在毫不變更此程式碼的情況下將它複製過去。

您可以使用 `CTAS` 和隱含聯結的組合來取代此程式碼：

```sql
-- Create an interim table
CREATE TABLE CTAS_acs
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT    ISNULL(CAST([EnglishProductCategoryName] AS NVARCHAR(50)),0)    AS [EnglishProductCategoryName]
,        ISNULL(CAST([CalendarYear] AS SMALLINT),0)                         AS [CalendarYear]
,        ISNULL(CAST(SUM([SalesAmount]) AS MONEY),0)                        AS [TotalSalesAmount]
FROM    [dbo].[FactInternetSales]        AS s
JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
WHERE     [CalendarYear] = 2004
GROUP BY
        [EnglishProductCategoryName]
,        [CalendarYear]
;

-- Use an implicit join to perform the update
UPDATE  AnnualCategorySales
SET     AnnualCategorySales.TotalSalesAmount = CTAS_ACS.TotalSalesAmount
FROM    CTAS_acs
WHERE   CTAS_acs.[EnglishProductCategoryName] = AnnualCategorySales.[EnglishProductCategoryName]
AND     CTAS_acs.[CalendarYear]               = AnnualCategorySales.[CalendarYear]
;

--Drop the interim table
DROP TABLE CTAS_acs
;
```

## <a name="ansi-join-replacement-for-delete-statements"></a>delete 陳述式的 ANSI 聯結取代
有時候刪除資料的最佳方法是使用 `CTAS`。 除了刪除資料以外，可以只選取您想要保留的資料。 這對於使用 ANSI 聯結語法的 `DELETE` 陳述式尤其適用，因為 SQL 資料倉儲不支援在 `DELETE` 陳述式的 `FROM` 子句中使用 ANSI 聯結。

已轉換之 DELETE 陳述式的範例如下所示：

```sql
CREATE TABLE dbo.DimProduct_upsert
WITH
(   Distribution=HASH(ProductKey)
,   CLUSTERED INDEX (ProductKey)
)
AS -- Select Data you wish to keep
SELECT     p.ProductKey
,          p.EnglishProductName
,          p.Color
FROM       dbo.DimProduct p
RIGHT JOIN dbo.stg_DimProduct s
ON         p.ProductKey = s.ProductKey
;

RENAME OBJECT dbo.DimProduct        TO DimProduct_old;
RENAME OBJECT dbo.DimProduct_upsert TO DimProduct;
```

## <a name="replace-merge-statements"></a>取代 merge 陳述式
Merge 陳述式可以取代，至少有部分可使用 CTAS 取代。 您可以將 INSERT 和 UPDATE 合併成單一陳述式。 任何已刪除的記錄都必須在第二個陳述式中關閉。

以下是 UPSERT 的範例：

```sql
CREATE TABLE dbo.[DimProduct_upsert]
WITH
(   DISTRIBUTION = HASH([ProductKey])
,   CLUSTERED INDEX ([ProductKey])
)
AS
-- New rows and new versions of rows
SELECT      s.[ProductKey]
,           s.[EnglishProductName]
,           s.[Color]
FROM      dbo.[stg_DimProduct] AS s
UNION ALL  
-- Keep rows that are not being touched
SELECT      p.[ProductKey]
,           p.[EnglishProductName]
,           p.[Color]
FROM      dbo.[DimProduct] AS p
WHERE NOT EXISTS
(   SELECT  *
    FROM    [dbo].[stg_DimProduct] s
    WHERE   s.[ProductKey] = p.[ProductKey]
)
;

RENAME OBJECT dbo.[DimProduct]          TO [DimProduct_old];
RENAME OBJECT dbo.[DimpProduct_upsert]  TO [DimProduct];

```

## <a name="ctas-recommendation-explicitly-state-data-type-and-nullability-of-output"></a>CTAS 建議：明確陳述資料類型和輸出可為 null
移轉程式碼時，您可能會發現自己正在跨這種類型的程式碼撰寫模式執行：

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455

CREATE TABLE result
(result DECIMAL(7,2) NOT NULL
)
WITH (DISTRIBUTION = ROUND_ROBIN)

INSERT INTO result
SELECT @d*@f
;
```

您可能會自動認為您應該將此程式碼移轉到 CTAS，就會是正確的。 不過，還是會有隱藏的問題。

下列程式碼不會產生相同的結果：

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455
;

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT @d*@f as result
;
```

請注意，資料行 "result" 沿用運算式的資料類型和可為 null 值。 如果不小心處理，這可能會導致細微差異值。

請嘗試下列內容做為範例：

```sql
SELECT result,result*@d
from result
;

SELECT result,result*@d
from ctas_r
;
```

儲存的結果值是不同的。 因為結果資料行中保留的值用於其他運算式，錯誤變得更加顯著。

![CTAS 結果](media/sql-data-warehouse-develop-ctas/ctas-results.png)

這對資料移轉特別重要。 即使第二個查詢已經更精確，仍然有一個問題。 相較於來源系統，此資料有所不同，這會在移轉中產生完整性的問題。 這是「錯誤」答案其實是正確答案的少數原因之一！

我們會看到這兩個結果之間有差異，追根究柢的原因是隱含的類型轉型。 在第一個範例中，資料表會定義資料行。 插入資料列時，就會發生隱含類型轉換。 在第二個範例中，沒有隱含類型轉換，因為此運算式會定義資料行的資料類型。 請注意，第二個範例中的資料行已定義為可為 Null 的資料行，而在第一個範例中還沒有定義。 在第一個範例中建立資料表時，尚未明確定義資料行可為 null。 在第二個範例中，它只留給了運算式，根據預設，這會導致 NULL 定義。  

若要解決這些問題，您必須在 CTAS 陳述式的 SELECT 部分中明確設定類型轉換和可為 null 屬性。 您無法在建立資料表的部分中設定這些屬性。

下列範例示範如何修正程式碼：

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT ISNULL(CAST(@d*@f AS DECIMAL(7,2)),0) as result
```

請注意：

* 可能已經使用 CAST 或 CONVERT
* ISNULL 是用來強制 NULLability，而非 COALESCE
* ISNULL 是最外層的函式
* ISNULL 的第二個部分是常數，也就是 0

> [!NOTE]
> 若要正確地設定可為 null 屬性，必須使用 ISNULL 而不是 COALESCE。 COALESCE 不是具決定性的函數，因此運算式的結果一律可為 Null。 但 ISNULL 不同。 它是具決定性的。 因此當 ISNULL 函數的第二個部分是常數或常值，則結果值將會是 NOT NULL。
> 
> 

本秘訣不只適用於確保計算的完整性。 它對資料表分割切換也很重要。 假設您根據事實定義此資料表：

```sql
CREATE TABLE [dbo].[Sales]
(
    [date]      INT     NOT NULL
,   [product]   INT     NOT NULL
,   [store]     INT     NOT NULL
,   [quantity]  INT     NOT NULL
,   [price]     MONEY   NOT NULL
,   [amount]    MONEY   NOT NULL
)
WITH
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

不過，[值] 欄位是已計算的運算式，它不是來源資料的一部分。

若要建立資料分割資料集，您可能會想要這麼做：

```sql
CREATE TABLE [dbo].[Sales_in]
WITH    
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
    [date]    
,   [product]
,   [store]
,   [quantity]
,   [price]   
,   [quantity]*[price]  AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create')
;
```

執行最適合的查詢。 當您嘗試執行資料分割切換時會出現問題。 資料表定義不相符。 若要讓資料表定義相符，必須修改 CTAS。

```sql
CREATE TABLE [dbo].[Sales_in]
WITH    
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
    [date]    
,   [product]
,   [store]
,   [quantity]
,   [price]   
,   ISNULL(CAST([quantity]*[price] AS MONEY),0) AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create');
```

因此，您可以查看類型一致性，且維護 CTAS 上的可為 null 屬性是很好的工程最佳作法。 它有助於維護計算的完整性，並且也可確保資料分割切換的可能性。

請參閱 [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 文件。 它是 Azure SQL 資料倉儲中最重要的陳述式之一。 請確定您已徹底了解。

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱[開發概觀](sql-data-warehouse-overview-develop.md)。

