---
title: 將您的結構描述移轉至 SQL 資料倉儲 | Microsoft Docs
description: 將您的結構描述移轉至 Azure SQL 資料倉儲來開發解決方案的秘訣。
services: sql-data-warehouse
author: jrowlandjones
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: jrj
ms.reviewer: igorstan
ms.openlocfilehash: 51ad7eed0bf37194b1e5ff2c605b39246e9a1191
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301177"
---
# <a name="migrate-your-schemas-to-sql-data-warehouse"></a>將您的結構描述移轉至 SQL 資料倉儲
將您的 SQL 結構描述移轉至 SQL 資料倉儲的指引。 

## <a name="plan-your-schema-migration"></a>規劃結構描述移轉

規劃結構描述移轉時，請參閱[資料表概觀][table overview]以熟悉各種資料表設計考量，例如統計資料、發佈、分割及編製索引。  它也會列出一些[不支援的資料表功能][unsupported table features]和其因應措施。

## <a name="use-user-defined-schemas-to-consolidate-databases"></a>使用使用者定義的結構描述來合併資料庫

您現有的工作負載可能有多個資料庫。 例如，SQL Server 資料倉儲可能包含臨時資料庫、資料倉儲資料庫和某些資料超市資料庫。 在此拓撲中，每個資料庫都會搭配個別的安全性原則，以個別工作負載的形式執行。

相反地，SQL 資料倉儲會在某個資料庫中執行整個資料倉儲工作負載。 不允許跨資料庫聯結。 因此，SQL 資料倉儲預期由資料倉儲使用的所有資料表都會儲存在單一資料庫內。

建議您透過使用者定義的結構描述，將現有的工作負載合併成單一資料庫。 如需範例，請參閱[使用者定義的結構描述](sql-data-warehouse-develop-user-defined-schemas.md)

## <a name="use-compatible-data-types"></a>使用相容的資料類型
修改資料類型以與 SQL 資料倉儲相容。 如需所有支援和不支援資料類型的清單，請參閱[資料類型][data types]。 該主題提供不支援類型的因應措施。 它也會提供查詢來識別 SQL 資料倉儲中不支援的現有類型。

## <a name="minimize-row-size"></a>將資料列大小最小化
若要達到最佳效能，請將資料表的資料列長度最小化。 由於較短的資料列長度會有更好的效能，因此請使用適合您資料的最小資料類型。 

針對資料表的資料列寬度，PolyBase 具有 1 MB 的限制。  如果您打算使用 PolyBase 將資料載入 SQL 資料倉儲，請更新您的資料表以使最大資料列寬度小於 1 MB。 

<!--
- For example, this table uses variable length data but the largest possible size of the row is still less than 1 MB. PolyBase will load data into this table.

- This table uses variable length data and the defined row width is less than one MB. When loading rows, PolyBase allocates the full length of the variable-length data. The full length of this row is greater than one MB.  PolyBase will not load data into this table.  

-->

## <a name="specify-the-distribution-option"></a>指定發佈選項
SQL 資料倉儲是一種分散式資料庫系統。 每個資料表都會分散或複寫至多個計算節點。 有一個資料表選項，可讓您指定散發資料的方式。 選擇包括循環配置資源、複寫或雜湊分散。 各有其優缺點。 如果您未指定發佈選項，SQL 資料倉儲預設會使用循環配置資源。

- 預設值為循環配置資源。 它的使用方式很簡單，並能以最快的速度載入資料，但聯結作業將會需要移動資料，因而會使查詢效能變慢。
- 複寫會在每個計算節點上儲存資料表的複本。 由於複寫資料表的聯結和彙總作業並不需要移動資料，所以很有效率。 相對的，它們需要額外的儲存體，因此比較適合較小的資料表使用。
- 雜湊分散會透過雜湊函式將資料列散發至所有節點上。 雜湊分散式資料表是 SQL 資料倉儲的核心，因為它們是針對在大型資料表上提供高度的查詢效能所設計。 此選項需要稍加規劃，以選出最適合在其上方散發資料的資料行。 不過，如果一開始並沒有選擇最適合的資料行，您可以輕鬆地在不同的資料行上重新散發資料。 

若要針對每個資料表選擇最佳發佈選項，請參閱[分散式資料表](sql-data-warehouse-tables-distribute.md)。


## <a name="next-steps"></a>後續步驟
將資料庫結構描述成功移轉到「SQL 資料倉儲」之後，您就可以繼續閱讀下列其中一篇文章：

* [移轉資料][Migrate your data]
* [移轉程式碼][Migrate your code]

若要深入了解 SQL 資料倉儲最佳做法，請參閱[最佳做法][best practices]一文。

<!--Image references-->

<!--Article references-->
[Migrate your code]: ./sql-data-warehouse-migrate-code.md
[Migrate your data]: ./sql-data-warehouse-migrate-data.md
[best practices]: ./sql-data-warehouse-best-practices.md
[table overview]: ./sql-data-warehouse-tables-overview.md
[unsupported table features]: ./sql-data-warehouse-tables-overview.md#unsupported-table-features
[data types]: ./sql-data-warehouse-tables-data-types.md
[unsupported data types]: ./sql-data-warehouse-tables-data-types.md#unsupported-data-types

<!--MSDN references-->


<!--Other Web references-->
