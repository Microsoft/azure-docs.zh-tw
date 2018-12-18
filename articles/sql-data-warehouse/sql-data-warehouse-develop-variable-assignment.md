---
title: 在 Azure SQL 資料倉儲中指派變數 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中指派 T-SQL 變數以便開發解決方案的秘訣。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: cf6d63c8cf67e42eed2ca52bfd0d0a3f9b0e10b1
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302079"
---
# <a name="assigning-variables-in-azure-sql-data-warehouse"></a>在 Azure SQL 資料倉儲中指派變數
在 Azure SQL 資料倉儲中指派 T-SQL 變數以便開發解決方案的秘訣。

## <a name="setting-variables-with-declare"></a>使用 DECLARE 設定宣告
SQL 資料倉儲中的變數是使用 `DECLARE` 陳述式或 `SET` 陳述式進行設定的。 使用 DECLARE 初始化變數是在 SQL 資料倉儲中設定變數值的其中一種最具彈性的方式。

```sql
DECLARE @v  int = 0
;
```

您也可以使用 DECLARE，一次設定一個以上的變數。 您無法使用 SELECT 或 UPDATE 來執行下列動作：

```sql
DECLARE @v  INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Smith')
,       @v1 INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Jones')
;
```

您無法在相同的 DECLARE 陳述式中初始化並使用變數。 為了說明這點，**不**允許下列範例，因為 @p1 已在相同的 DECLARE 陳述式中初始化和使用。 下列範例會顯示錯誤。

```sql
DECLARE @p1 int = 0
,       @p2 int = (SELECT COUNT (*) FROM sys.types where is_user_defined = @p1 )
;
```

## <a name="setting-values-with-set"></a>使用 SET 設定值
SET 是設定單一變數時常見的方法。

下列陳述式是使用 SET 設定變數所有的有效方法：

```sql
SET     @v = (Select max(database_id) from sys.databases);
SET     @v = 1;
SET     @v = @v+1;
SET     @v +=1;
```

您一次只能使用 SET 設定一個變數。 不過，允許複合運算子。

## <a name="limitations"></a>限制
您無法使用 SELECT 或 UPDATE 進行變數指派。

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱[開發概觀](sql-data-warehouse-overview-develop.md)。

