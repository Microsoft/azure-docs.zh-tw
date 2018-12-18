---
title: 使用 Azure SQL 資料倉儲中的分組方式選項 | Microsoft Docs
description: 根據 Azure SQL 資料倉儲中的選項實作群組以便開發解決方案的秘訣。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: 1f5723bd160abc164779062f213762751e5875c8
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43303373"
---
# <a name="group-by-options-in-sql-data-warehouse"></a>根據 SQL 資料倉儲中的選項分組
根據 Azure SQL 資料倉儲中的選項實作群組以便開發解決方案的秘訣。

## <a name="what-does-group-by-do"></a>GROUP BY 有什麼用途？

[GROUP BY](/sql/t-sql/queries/select-group-by-transact-sql) T-SQL 子句可將資料彙總至摘要的一組資料列。 GROUP BY 有一些 SQL 資料倉儲不支援的選項。 這些選項有因應措施。

可用選項包括

* GROUP BY 搭配 ROLLUP
* GROUPING SETS
* GROUP BY 搭配 CUBE

## <a name="rollup-and-grouping-sets-options"></a>Rollup 和 grouping sets 選項
此處最簡單的選項是改為使用 UNION ALL 來執行彙總，而不是依賴明確的語法。 應該會出現幾乎相同的結果

下列範例使用 GROUP BY 陳述式搭配 ROLLUP 選項：
```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount)             AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t       ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY ROLLUP (
                        [SalesTerritoryCountry]
                ,       [SalesTerritoryRegion]
                )
;
```

藉由使用 ROLLUP，上述範例會要求下列彙總：

* 國家及區域
* 國家 (地區)
* 總計

若要取代 ROLLUP，並傳回相同的結果，您可以使用 UNION ALL，並明確指定所需的彙總：

```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
UNION ALL
SELECT [SalesTerritoryCountry]
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
UNION ALL
SELECT NULL
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey;
```

若要取代 GROUPING SETS，則適用範例原則。 您只需要針對想要查看的彙總層級建立 UNION ALL 區段。

## <a name="cube-options"></a>Cube 選項
可以使用 UNION ALL 方法建立 GROUP BY WITH CUBE。 問題是程式碼可能很快就會很麻煩且不易處理。 若要避免此情形，您可以使用這項更進階的方法。

使用上述範例。

第一個步驟是定義 'cube'，其定義我們想要建立的所有彙總層級。 請務必記下兩個衍生資料表的交叉聯結。 這樣可以為我們產生所有層級。 其餘程式碼真的有格式化。

```sql
CREATE TABLE #Cube
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
AS
WITH GrpCube AS
(SELECT    CAST(ISNULL(Country,'NULL')+','+ISNULL(Region,'NULL') AS NVARCHAR(50)) as 'Cols'
,          CAST(ISNULL(Country+',','')+ISNULL(Region,'') AS NVARCHAR(50))  as 'GroupBy'
,          ROW_NUMBER() OVER (ORDER BY Country) as 'Seq'
FROM       ( SELECT 'SalesTerritoryCountry' as Country
             UNION ALL
             SELECT NULL
           ) c
CROSS JOIN ( SELECT 'SalesTerritoryRegion' as Region
             UNION ALL
             SELECT NULL
           ) r
)
SELECT Cols
,      CASE WHEN SUBSTRING(GroupBy,LEN(GroupBy),1) = ','
            THEN SUBSTRING(GroupBy,1,LEN(GroupBy)-1)
            ELSE GroupBy
       END AS GroupBy  --Remove Trailing Comma
,Seq
FROM GrpCube;
```

以下顯示 CTAS 的結果：

![依 Cube 分組](media/sql-data-warehouse-develop-group-by-options/sql-data-warehouse-develop-group-by-cube.png)

第二個步驟是指定目標資料表來儲存過渡結果：

```sql
DECLARE
 @SQL NVARCHAR(4000)
,@Columns NVARCHAR(4000)
,@GroupBy NVARCHAR(4000)
,@i INT = 1
,@nbr INT = 0
;
CREATE TABLE #Results
(
 [SalesTerritoryCountry] NVARCHAR(50)
,[SalesTerritoryRegion]  NVARCHAR(50)
,[TotalSalesAmount]      MONEY
)
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
;
```

第三個步驟是對執行彙總的資料行 cube 執行迴圈。 查詢會為 #Cube 暫存資料表中的每個資料列執行一次，並將結果儲存在 #Results 暫存資料表中

```sql
SET @nbr =(SELECT MAX(Seq) FROM #Cube);

WHILE @i<=@nbr
BEGIN
    SET @Columns = (SELECT Cols    FROM #Cube where seq = @i);
    SET @GroupBy = (SELECT GroupBy FROM #Cube where seq = @i);

    SET @SQL ='INSERT INTO #Results
              SELECT '+@Columns+'
              ,      SUM(SalesAmount) AS TotalSalesAmount
              FROM  dbo.factInternetSales s
              JOIN  dbo.DimSalesTerritory t  
              ON s.SalesTerritoryKey = t.SalesTerritoryKey
              '+CASE WHEN @GroupBy <>''
                     THEN 'GROUP BY '+@GroupBy ELSE '' END

    EXEC sp_executesql @SQL;
    SET @i +=1;
END
```

最後，您只需要從 #Results 暫存資料表讀取，就可以傳回結果

```sql
SELECT *
FROM #Results
ORDER BY 1,2,3
;
```

將程式碼分成區段，並產生迴圈建構，程式碼就會變得更容易管理及維護。

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱[開發概觀](sql-data-warehouse-overview-develop.md)。

