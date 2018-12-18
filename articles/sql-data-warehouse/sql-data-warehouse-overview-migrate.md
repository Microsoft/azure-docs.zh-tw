---
title: 將您的解決方案移轉至 SQL 資料倉儲 | Microsoft Docs
description: 將您的解決方案帶入 Azure SQL 資料倉儲平台的移轉指導。
services: sql-data-warehouse
author: jrowlandjones
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: jrj
ms.reviewer: igorstan
ms.openlocfilehash: b3e01968c74060bd0dc366609275d63753ad62dd
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306701"
---
# <a name="migrate-your-solution-to-azure-sql-data-warehouse"></a>將您的解決方案移轉至 Azure SQL 資料倉儲
查看將現有資料庫解決方案移轉至 Azure SQL 資料倉儲所需採取的步驟。 

## <a name="profile-your-workload"></a>分析工作負載
在移轉之前，您應該要確定 SQL 資料倉儲是您工作負載的正確解決方案。 SQL 資料倉儲是分散式系統，是為了針對大型資料執行分析而設計。  移轉至 SQL 資料倉儲需要一些設計變更，這些變更並不會太難了解，但可能需要一些時間來實作。 但如果您的業務需要企業級的資料倉儲，它所帶來的優點將值得您投入時間和精力。 不過，如果您不需要 SQL 資料倉儲的功能，使用 SQL Server 或 Azure SQL Database 將會比較符合成本效益。

可考慮使用 SQL 資料倉儲的時機如下：
- 具有超過 1 TB 的資料
- 規劃對大量資料執行分析
- 需要調整計算和儲存體的功能 
- 想要在不需要時暫停計算資源來節省成本。

不要使用 SQL 資料倉儲進行具備以下條件的操作 (OLTP) 工作負載：
- 高頻率的讀取和寫入
- 大量的單一項目選取
- 大量的單一資料列插入
- 逐一處理資料列的需求
- 不相容的格式 (JSON、XML)


## <a name="plan-the-migration"></a>規劃移轉

一旦您決定將現有解決方案移轉至 SQL 資料倉儲，在開始移轉之前，請務必事先規劃。 

規劃的目標之一是為了確保您的資料、資料表結構描述，以及程式碼皆能與 SQL 資料倉儲相容。 您目前的系統與 SQL 資料倉儲之間有一些需要解決的相容性差異。 此外，將大量資料移轉至 Azure 需花費大量的時間。 謹慎規劃可加快將資料移轉至 Azure 的速度。 

規劃的另一個目標是做出設計上的調整，以確保您的解決方案能利用 SQL 資料倉儲的設計所提供的高查詢效能。 由於針對延展性設計資料倉儲帶來了不同的設計模式，因此傳統的方法不一定最適合。 雖然您可以在移轉之後做出一些設計上的調整，在程序中盡快做出變更將可節省日後時間。

若要成功執行移轉，您必須移轉資料表結構描述、程式碼及資料。 如需這些移轉主題的指引，請參閱：

-  [移轉結構描述](sql-data-warehouse-migrate-schema.md)
-  [遷移程式碼](sql-data-warehouse-migrate-code.md)
-  [移轉資料](sql-data-warehouse-migrate-data.md)。 

<!--
## Perform the migration


## Deploy the solution


## Validate the migration

-->

## <a name="next-steps"></a>後續步驟
CAT (客戶諮詢小組) 也會透過部落格發佈一些實用的 SQL 資料倉儲指引。  請瀏覽他們的[將資料移轉到 Azure SQL 資料倉儲的實際作法][Migrating data to Azure SQL Data Warehouse in practice]一文，以取得關於移轉的額外指導方針。

<!--Image references-->

<!--Article references-->

<!--MSDN references-->

<!--Other Web references-->
[Migrating data to Azure SQL Data Warehouse in practice]: https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/
