---
title: 定義資料類型 - Azure SQL 資料倉儲 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中定義資料表資料類型的建議。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: eb469e6a654414b0411f8c45b73658f99a383751
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306578"
---
# <a name="table-data-types-in-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲中的資料表資料類型
在 Azure SQL 資料倉儲中定義資料表資料類型的建議。 

## <a name="what-are-the-data-types"></a>資料類型是什麼？

SQL 資料倉儲支援最常用的資料類型。 如需支援的資料類型清單，請參閱 CREATE TABLE 陳述式中的[資料類型](/sql/t-sql/statements/create-table-azure-sql-data-warehouse#DataTypes)。 

## <a name="minimize-row-length"></a>將資料列長度最小化
將資料類型的大小最小化可縮短資料列長度，這樣會提升查詢效能。 使用您的資料適用之最小資料類型。 

- 避免使用較大的預設長度來定義字元資料行。 例如，如果最長的值是 25 個字元，請將您的資料行定義為 VARCHAR(25)。 
- 當您僅需要 VARCHAR 時，請避免使用 [NVARCHAR][NVARCHAR]。
- 儘可能使用 NVARCHAR(4000) 或 VARCHAR(8000)，而非 NVARCHAR(MAX) 或 VARCHAR(MAX)。

如果您使用 PolyBase 外部資料表來載入資料表，定義的資料表資料列長度不得超過 1 MB。 當資料列的變數長度資料超過 1 MB 時，您可以使用 BCP 但不可使用 PolyBase 載入資料列。

## <a name="identify-unsupported-data-types"></a>識別不支援的資料類型
如果您從另一個 SQL Database 移轉您的資料庫，可能會遇到 SQL 資料倉儲不支援的資料類型。 使用此查詢可找出您現有 SQL 結構描述中不支援的資料類型。

```sql
SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
FROM sys.tables  t
JOIN sys.columns c on t.[object_id]    = c.[object_id]
JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','xml')
 AND  y.[is_user_defined] = 1;
```


## <a name="unsupported-data-types"></a>不支援的資料類型可用的因應措施

下列清單會顯示 SQL 資料倉儲不支援的資料類型，並提供您可以用來取代不支援資料類型的替代項目。

| 不支援的資料類型 | 因應措施 |
| --- | --- |
| [geometry](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql) |
| [geography](/sql/t-sql/spatial-geography/spatial-types-geography) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql) |
| [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql)(4000) |
| [映像](/sql/t-sql/data-types/ntext-text-and-image-transact-sql) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql) |
| [text](/sql/t-sql/data-types/ntext-text-and-image-transact-sql) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql) |
| [ntext](/sql/t-sql/data-types/ntext-text-and-image-transact-sql) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql) |
| [sql_variant](/sql/t-sql/data-types/sql-variant-transact-sql) |將資料行分割成數個強型別資料行。 |
| [資料表](/sql/t-sql/data-types/table-transact-sql) |轉換成暫存資料表。 |
| [timestamp](/sql/t-sql/data-types/date-and-time-types) |修改程式碼來使用 [datetime2](/sql/t-sql/data-types/datetime2-transact-sql) 和 [CURRENT_TIMESTAMP](/sql/t-sql/functions/current-timestamp-transact-sql) 函式。 僅支援常數做為預設值，因此 current_timestamp 不可定義為預設條件約束。 如果您需要從 timestamp 類型資料行移轉資料列版本值，請對 NOT NULL 或 NULL 資料列版本值使用 [BINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql)(8) 或 [VARBINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql)(8)。 |
| [xml](/sql/t-sql/xml/xml-transact-sql) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql) |
| [使用者定義型別](/sql/relational-databases/native-client/features/using-user-defined-types) |可能時，請轉換回原生資料類型。 |
| 預設值 | 預設值僅支援常值和常數。 |


## <a name="next-steps"></a>後續步驟
如需開發資料表的詳細資訊，請參閱[資料表概觀](sql-data-warehouse-tables-overview.md)。
