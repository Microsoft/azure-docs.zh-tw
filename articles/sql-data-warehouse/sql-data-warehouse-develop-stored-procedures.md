---
title: 在 Azure SQL 資料倉儲中使用預存程序 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中實作預存程序以便開發解決方案的秘訣。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: 4cd8394104c72e8df53fa0c44e60037b4dc05976
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301608"
---
# <a name="using-stored-procedures-in-sql-data-warehouse"></a>在 SQL 資料倉儲中使用預存程序
在 Azure SQL 資料倉儲中實作預存程序以便開發解決方案的秘訣。

## <a name="what-to-expect"></a>未來展望

SQL 資料倉儲支援許多 SQL Server 中使用的 T-SQL 功能。 更重要的是有相應放大的特定功能，您可用來將解決方案效能最大化。

不過，為了維護 SQL 資料倉儲的規模和效能，還有一些具有行為差異的功能以及其他不支援的功能。


## <a name="introducing-stored-procedures"></a>預存程序簡介
預存程序很適合用來封裝您的 SQL 程式碼；將它儲存在資料倉儲中您的資料附近。 藉由將程式碼封裝成可管理的單位，預存程序協助開發人員將其解決方案模組化；促使程式碼有更大的可重複使用性。 每個預存程序也可接受參數，使其更具彈性。

SQL 資料倉儲提供簡化且更簡化的預存程序實作。 相較於 SQL Server，最大差異是預存程序不是預先編譯的程式碼。 在資料倉儲中，相對於針對大型資料磁碟區執行查詢所花費的時間，編譯時間很少。 更重要的是，務必要針對大型查詢最佳化預存程序程式碼。 目標是要節省時數、分鐘數和秒數，而不是毫秒數。 因此，將預存程序視為 SQL 邏輯的容器更有幫助。     

當 SQL 資料倉儲執行預存程序時，SQL 陳述式會在執行階段進行剖析、轉譯和最佳化。 在此過程中，每個陳述式都會轉換為分散式查詢。 針對資料執行的 SQL 程式碼與提交的查詢不同。

## <a name="nesting-stored-procedures"></a>巢狀預存程序
當預存程序呼叫其他預存程序或執行動態 SQL 時，內部預存程序或程式碼叫用據稱就是巢狀。

SQL 資料倉儲最多支援八個巢狀層級。 這與 SQL Server 稍有不同。 SQL Server 中的巢狀層級為 32。

最上層預存程序呼叫等同於巢狀層級 1。

```sql
EXEC prc_nesting
```
如果預存程序也會進行另一個 EXEC 呼叫，這會將巢狀層級提高到二。

```sql
CREATE PROCEDURE prc_nesting
AS
EXEC prc_nesting_2  -- This call is nest level 2
GO
EXEC prc_nesting
```
如果第二個程序接著會執行一些動態 SQL，巢狀層級會提高到三。

```sql
CREATE PROCEDURE prc_nesting_2
AS
EXEC sp_executesql 'SELECT 'another nest level'  -- This call is nest level 2
GO
EXEC prc_nesting
```

請注意，SQL 資料倉儲目前不支援 [@@NESTLEVEL](/sql/t-sql/functions/nestlevel-transact-sql)。 您需要追蹤巢狀層級。 您不太可能會超過八個巢狀層級限制，但如果會，則需要修改您的程式碼，以讓巢狀層級符合這項限制。

## <a name="insertexecute"></a>INSERT..EXECUTE
SQL 資料倉儲不允許您透過 INSERT 陳述式取用預存程序的結果集。 不過，您可以使用另一個方法。 如需範例，請參閱[暫存資料表](sql-data-warehouse-tables-temporary.md)上的文章。 

## <a name="limitations"></a>限制
在 SQL 資料倉儲中不會實作 TRANSACT-SQL 預存程序的有些層面。

如下：

* 暫存預存程序
* 編號預存程序
* 擴充預存程序
* CLR 預存程序
* 加密選項
* 複寫選項
* 資料表值參數
* 唯讀參數
* 預設參數
* 執行內容
* return 陳述式

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱[開發概觀](sql-data-warehouse-overview-develop.md)。

