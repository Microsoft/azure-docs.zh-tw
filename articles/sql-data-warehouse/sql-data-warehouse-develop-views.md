---
title: 在 Azure SQL 資料倉儲中使用 T-SQL 檢視 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中使用 T-SQL 檢視開發解決方案的秘訣。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: 796a549c3a01c235422667dc63b729244b1917bf
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306498"
---
# <a name="views-in-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲中的檢視
在 Azure SQL 資料倉儲中使用 T-SQL 檢視開發解決方案的秘訣。 

## <a name="why-use-views"></a>為何要使用檢視？
檢視能以許多不同的方式使用，提升您的方案品質。  本文特別強調幾個範例，說明如何以檢視來豐富您的解決方案，以及需要考量的限制。

> [!NOTE]
> 本文中不會討論 CREATE VIEW 的語法。 如需詳細資訊，請參閱 [CREATE VIEW](/sql/t-sql/statements/create-view-transact-sql) 文件。
> 
> 

## <a name="architectural-abstraction"></a>架構抽象概念
常見的應用程式模式就是在載入資料時，使用後面接著物件重新命名模式的 CREATE TABLE AS SELECT (CTAS) 來重建資料表。

下列範例會將新的日期記錄加入至日期維度。 請注意新的資料表 DimDate_New 最初是如何建立，然後重新命名，以取代原始版本的資料表。

```sql
CREATE TABLE dbo.DimDate_New
WITH (DISTRIBUTION = ROUND_ROBIN
, CLUSTERED INDEX (DateKey ASC)
)
AS
SELECT *
FROM   dbo.DimDate  AS prod
UNION ALL
SELECT *
FROM   dbo.DimDate_stg AS stg
;

RENAME OBJECT DimDate TO DimDate_Old;
RENAME OBJECT DimDate_New TO DimDate;

```

不過，此方法可能導致資料表在使用者的檢視中出現和消失，還有「資料表不存在」錯誤訊息。 儘管基礎物件已重新命名，檢視可為使用者提供一致的展示層。 透過檢視存取資料，意味著使用者不需要看見基礎資料表。 這一層提供一致的使用者體驗，同時確保資料倉儲設計人員能持續改進資料模型。 能夠改進基礎資料表，意味著設計人員可在資料載入過程中使用 CTAS 充分發揮效能。   

## <a name="performance-optimization"></a>效能最佳化
檢視也可用來強化資料表之間的效能最佳化聯結。 例如，檢視可以納入備援散發金鑰作為聯結準則的一部分，以將資料移動最小化。 檢視的另一項好處是強制執行特定查詢或聯結提示。 以這種方式使用檢視，可確保一律以最佳方式執行聯結，使用者不需要記住其聯結的正確建構。

## <a name="limitations"></a>限制
SQL 資料倉儲中的檢視只會儲存為中繼資料。 因此無法使用下列選項︰

* 沒有結構描述繫結選項
* 無法透過檢視更新基底資料表
* 無法在暫存資料表上建立檢視
* 不支援 EXPAND / NOEXPAND 提示
* SQL 資料倉儲中沒有索引檢視表

## <a name="next-steps"></a>後續步驟
如需更多開發秘訣，請參閱 [SQL 資料倉儲開發概觀](sql-data-warehouse-overview-develop.md)。


