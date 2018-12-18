---
title: 2018 年 5 月 Azure SQL 資料倉儲版本資訊 | Microsoft Docs
description: Azure SQL 資料倉儲的版本資訊。
services: sql-data-warehouse
author: twounder
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 07/23/2018
ms.author: twounder
ms.reviewer: twounder
ms.openlocfilehash: d26ae755b658ce39df04788b7edeee6da7e20a47
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43288087"
---
# <a name="whats-new-in-azure-sql-data-warehouse-may-2018"></a>Azure SQL 資料倉儲有哪些最新功能？ 2018 年 5 月 
Azure SQL 資料倉儲會持續改進。 本文說明 2018 年 5 月導入的新功能和變更。 

## <a name="gen-2-instances"></a>Gen 2 執行個體
![alt](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/2528b41b-f09f-45b1-aa65-fc60d562d3bd.png) Azure SQL 資料倉儲計算最佳化 Gen2 層級，為雲端資料倉儲樹立了新的效能標準。 比起目前這一代的表現，客戶現在可得到的查詢效能提升了多達五倍、並行存取數量多出四倍，而運算能力則高出五倍。 它現在可從單一叢集中進行 128 個並行查詢，是所有雲端資料倉儲服務中最多的。

請參閱 Azure Data 公司副總裁 Rohan Kumar 所帶來的[使用 Azure SQL 資料倉儲大幅加速雲端分析](https://azure.microsoft.com/blog/turbocharge-cloud-analytics-with-azure-sql-data-warehouse/)部落格通知。

## <a name="auto-statistics"></a>自動統計資料
在現代化的成本型最佳化工具 (例如 SQL 資料倉儲中的引擎) 中，統計資料對於將查詢計劃產生最佳化而言至關緊要。 若能事先知道所有的查詢，就能決定需要建立的統計資料物件。 不過，當系統面臨通常用於資料倉儲工作負載的臨機操作和隨機查詢時，系統管理員可能難以預測需要建立哪些統計資料，這可能會導致查詢執行計劃無法最佳化，且查詢回應時間變長。 減輕這個問題的其中一種方式，是在所有資料表的資料行上事先建立統計資料物件。 然而，該程序會帶來負面影響，因為統計資料物件在資料表載入程序期間需要進行維護，而導致載入時間變長。

SQL 資料倉儲現在支援自動建立統計資料物件，以提供更大彈性、生產力，並讓系統管理員和開發人員更輕鬆使用，同時確保系統持續提供高品質的執行計劃和最佳的回應時間。

若要在 SQL 資料倉儲中啟用或停用自動建立統計資料，請執行下列陳述式：
```sql
ALTER DATABASE { database_name } SET { AUTO_CREATE_STATISTICS { OFF | ON } } [;]
```

建議您依照最佳做法和指南，將 `AUTO_CREATE_STATISTICS` 選項設定為 `ON`。

> [!NOTE]
> 對於所有新資料倉儲，自動建立統計資料「預設為啟用」。
>  

如需其他詳細資料，請參閱 [ALTER DATABASE SET 選項](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options)文件。

## <a name="rejected-row-support"></a>拒絕的資料列支援
客戶通常會因為資料載入的高效能、平行本質，而使用 [PolyBase (外部資料表) 將資料載入](design-elt-data-loading.md) SQL 資料倉儲。 透過 [Azure Data Factory](http://azure.com/adf) 載入資料時，PolyBase 也是預設的載入模型。 

SQL 資料倉儲新增了功能，可透過 `REJECTED_ROW_LOCATION` 參數搭配 [CREATE EXTERNAL TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql) 陳述式，定義遭拒絕的資料列位置。 從外部資料表中執行 [CREATE TABLE AS SELECT (CTAS)](https://docs.microsoft.com/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 之後，任何無法載入的資料列都會儲存在來源附近的檔案中，以供進一步調查。 

如需關於遭拒絕資料列行為的詳細資訊，請參閱[使用 SQL 資料倉儲 PolyBase 遭拒絕的資料列位置來安心載入](https://azure.microsoft.com/blog/load-confidently-with-sql-data-warehouse-polybase-rejected-row-location/)部落格。

下列範例示範指定「拒絕資料列」的新語法。

```sql
CREATE EXTERNAL TABLE [dbo].[Reject_Example]
(
    ...
)
WITH
(
    ...
    ,REJECTED_ROW_LOCATION=‘/Reject_Directory'
)
```

## <a name="alter-view"></a>ALTER VIEW
[ALTER VIEW](https://docs.microsoft.com/sql/t-sql/statements/alter-view-transact-sql) 可讓使用者修改先前建立的檢視，而不必刪除/建立檢視，並重新套用權限。 

下列範例會修改先前建立的檢視。
```sql
ALTER VIEW test_view AS SELECT 1 [data];
```

## <a name="concatws"></a>CONCAT_WS
[CONCAT_WS()](https://docs.microsoft.com/sql/t-sql/functions/concat-ws-transact-sql) 函式會傳回字串，該字串會以端對端方式將兩個或多個值串連。 該函式會使用第一個引數中指定的分隔符號，將串連的值分隔。 `CONCAT_WS` 函式很適合用於產生以逗號分隔值 (CSV) 輸出。

下列範例會示範以逗點串連一組 int 值。
```sql
SELECT CONCAT_WS(',', 1, 2, 3) [result];
```
陳述式會傳回下列結果：
```
result
---------
1,2,3
```
下列範例會示範以逗點串連一組混合的資料類型值。
```sql
SELECT CONCAT_WS(',', 1, 2, 'String', NEWID()) [result]
```
陳述式會傳回下列結果：
```
result
---------
1,2,String,26E1F74D-5746-44DC-B47F-2FC1DA1B6E49
```

## <a name="spdatatypeinfo"></a>SP_DATATYPE_INFO
[sp_datatype_info](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-datatype-info-transact-sql) 系統預存程序會傳回目前環境所支援資料類型的相關資訊。 許多工具常會使用它來透過資料類型調查的 ODBC 進行連線。

下列範例會擷取 SQL 資料倉儲所支援所有資料類型的詳細資料。

```sql
EXEC sp_datatype_info
```

## <a name="select-into-with-order-by-behavior-change"></a>包含 ORDER BY 的 SELECT INTO 行為變更
SQL 資料倉儲現在將會封鎖包含 `ORDER BY` 子句的 `SELECT INTO` 查詢。 先前，藉由先排序記憶體中的資料，然後插入目標資料表中，並重新排列資料以符合資料表圖形，即可成功進行這項作業。

### <a name="previous-behavior"></a>先前的行為
下列陳述式雖會成功，但會產生額外的處理負荷。
```sql
SELECT * INTO table2 FROM table1 ORDER BY 1;
```

### <a name="current-behavior"></a>目前的行為
下列陳述式將會擲回錯誤，指出 `SELECT INTO` 陳述式中不支援 `ORDER BY` 子句。
```sql
SELECT * INTO table2 FROM table1 ORDER BY 1;
```
傳回的錯誤陳述式：
```
Msg 104381, Level 16, State 1, Line 1
The ORDER BY clause is invalid in views, CREATE TABLE AS SELECT, INSERT SELECT, SELECT INTO, inline functions, derived tables, subqueries, and common table expressions, unless TOP or FOR XML is also specified.
```

## <a name="set-parseonly-on-query-status-behavior-change"></a>SET PARSEONLY ON 查詢狀態 (行為變更)
使用 `SET PARSEONLY ON` 語法可讓使用者利用 SQL 資料倉儲引擎檢查每個 T-SQL 陳述式的語法，並傳回任何錯誤訊息，而無須編譯或執行陳述式。 先前，在 [sys.dm_pdw_exec_requests](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql) 系統檢視中，這些陳述式的狀態會保持在 `Running` 狀態。 `sys.dm_pdw_exec_requests` 檢視現在會傳回 `Complete` 的狀態。

## <a name="next-steps"></a>後續步驟
現在您已稍微了解 SQL 資料倉儲，請了解如何快速[建立 SQL 資料倉儲][create a SQL Data Warehouse]。 如果您不熟悉 Azure，您可能會發現 [Azure 詞彙][Azure glossary]在您遇到新術語時很有幫助。 或者，也可以看一下其中一些其他 SQL 資料倉儲資源。  

* [客戶成功案例]
* [部落格]
* [功能要求]
* [影片]
* [客戶諮詢小組部落格]
* [Stack Overflow 論壇]
* [Twitter]


[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客戶諮詢小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[客戶成功案例]: https://azure.microsoft.com/case-studies/?service=sql-data-warehouse
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[create a SQL Data Warehouse]: ./create-data-warehouse-portal.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md