---
title: 2018 年 7 月 Azure SQL 資料倉儲版本資訊 | Microsoft Docs
description: Azure SQL 資料倉儲的版本資訊。
services: sql-data-warehouse
author: twounder
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/06/2018
ms.author: twounder
ms.reviewer: twounder
ms.openlocfilehash: c7d2211ca69fcd18588ea1b20b638b2970b8439c
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318835"
---
# <a name="whats-new-in-azure-sql-data-warehouse-july-2018"></a>Azure SQL 資料倉儲有哪些最新功能？ 2018 年 7 月
Azure SQL 資料倉儲會持續改進。 本文說明 2018 年 7 月導入的新功能和變更。

## <a name="lightning-fast-query-performance"></a>快速查詢效能
[Azure SQL 資料倉儲](https://aka.ms/sqldw)引進可改善壓縮作業的「即時資料存取」工具，確立新的效能基準。 透過使用直接 SQL Server 對 SQL Server 原生資料作業，「即時資料存取」可減少資料移動作業的額外負荷。 與 SQL Server 引擎直接整合進行資料移動，表示使用衍生自公認的業界標準 [TPC Benchmark™ H (TPC-H)](http://www.tpc.org/tpch/) 的工作負載時，SQL 資料倉儲現在的**速度比 Amazon Redshift 快 67%**。

![和 Amazon Redshift 相比，Azure SQL 資料倉儲更快速且更便宜](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/eb3b908a-464d-4847-b384-9f296083a737.png)
<sub>來源：[Gigaom 研究分析報告：雲端資料倉儲基準](https://gigaom.com/report/data-warehouse-in-the-cloud-benchmark/)</sub> \(英文\)

除了執行階段效能，[Gigaom 研究](https://gigaom.com/report/data-warehouse-in-the-cloud-benchmark/)報告也測量性價比，以量化特定工作負載的美金成本。 和 Redshift 相比，同樣處理 30 TB 的工作負載，SQL 資料倉儲**至少便宜 23%**。 透過 SQL 資料倉儲可彈性調整計算並能暫停和繼續工作負載的能力，客戶只須支付服務使用期間的費用，進而降低其成本。
![和 Amazon Redshift 相比，Azure SQL 資料倉儲更快速且更便宜](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/cb76447e-621e-414b-861e-732ffee5345a.png)
<sub>來源：[Gigaom 研究分析報告：雲端資料倉儲基準](https://gigaom.com/report/data-warehouse-in-the-cloud-benchmark/)</sub> \(英文\)

### <a name="query-concurrency"></a>查詢並行存取
SQL 資料倉儲也可確保資料能夠跨組織存取。 Microsoft 已經增強服務以支援 128 個並行查詢，讓更多使用者能查詢相同的資料庫，而不會被其他要求封鎖。 相較之下，Amazon Redshift 的並行查詢數上限為 50 個，限制了組織內的資料存取。

SQL 資料倉儲可提升這些查詢效能與查詢並行存取，而價格不變，並以低耦合的儲存體與計算建置其獨特的架構。

## <a name="finer-granularity-for-cross-region-and-server-restores"></a>更精細的跨區域和跨伺服器還原
您現在可以使用任何還原點進行跨區域和跨伺服器的還原，而不是選取每隔 24 小時執行一次的異地備援備份。 使用者定義或自動還原點皆可支援跨區域和跨伺服器還原，藉以為資料提供更精細的額外保護。 透過更多可用的還原點，即可確保在進行跨區域還原時，您的資料倉儲會根據邏輯來保持一致。

![還原命令 - Azure SQL 資料倉儲](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/6ac23972-9ec0-4502-ab10-7b6bc1a3d947.png)
![還原選項 - Azure SQL 資料倉儲](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/6c63bd0e-9c52-414d-b4be-d3bd3774ee08.png)

如需詳細資訊，請參閱部落格文章：[加速且具彈性的還原點](https://azure.microsoft.com/blog/accelerated-and-flexible-restore-points-with-sql-data-warehouse/)。

## <a name="20-minute-restorations"></a>20 分鐘還原
SQL 資料倉儲現在可讓您在 **20 分鐘內**還原相同區域內的任何資料倉儲 (不論任何資料庫大小)。 還原時間會與還原到相同區域內的相同或不同邏輯伺服器有關。 此外，快照集處理程序已經過改良，可減少建立還原點的時間。 在較低效能層級 (較低的 DWU 設定) 中，改善的幅度是將建立快照集的時間「減少 2 倍」。

如需詳細資訊，請參閱部落格文章：[加速且具彈性的還原點](https://azure.microsoft.com/blog/accelerated-and-flexible-restore-points-with-sql-data-warehouse/)。

## <a name="custom-restoration-configurations"></a>自訂還原組態
現在，當您在 Azure 入口網站中進行還原時，可以變更效能層級 (DWU)。 您也可以還原到已升級的 Gen2 資料倉儲。 您現在可以藉由還原至 Gen 2 執行個體來評估 Gen2 的影響，然後再[升級您的 Gen1 資料倉儲](https://docs.microsoft.com/azure/sql-data-warehouse/upgrade-to-latest-generation)。

![自訂的還原組態 - Azure SQL 資料倉儲](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/f4c410c7-8515-409c-a983-0976792b8628.png)

## <a name="spdescribeundeclaredparameters"></a>SP_DESCRIBE_UNDECLARED_PARAMETERS
工具常會使用 [sp_describe_undeclared_parameters](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-describe-undeclared-parameters-transact-sql) 預存程序來取得 Transact-SQL 批次中的參數中繼資料。 此程序會針對批次中每個參數傳回一個資料列，其中包含為該參數推算的類型資訊。 

```sql
EXEC sp_describe_undeclared_parameters
    @tsql = 
    N'SELECT
        object_id, name, type_desc
      FROM
        sys.indexes
      WHERE
        object_id = @id;'
```

結果集會包含 `@id` 參數的中繼資料 (顯示部分結果)
```
parameter_ordinal | name | suggested_system_type_id | suggested_system_type_name
--------------------------------------------------------------------------------
1                 | @id  | 56                       | int
```
## <a name="sprefreshsqlmodule"></a>SP_REFRESHSQLMODULE
如果因為底層物件有所變更，而使得底層中繼資料過期時，[Sp_refreshsqlmodule](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-refreshsqlmodule-transact-sql) 預存程序會更新資料庫物件的中繼資料。 如果變更檢視的基底資料表，卻未重新建立檢視，也可能發生這種情況。 這可讓您不必進行卸除和重新建立相依物件的步驟。

下列範例顯示因為基礎資料表變更而過時的檢視。 您會發現第一個資料行的資料正確 (1 變成 Mollie)，但資料行名稱無效且第二個資料行不存在。 
```sql
CREATE TABLE base_table (Id INT);
GO

INSERT INTO base_table (Id) VALUES (1);
GO

CREATE VIEW base_view AS SELECT * FROM base_table;
GO

SELECT * FROM base_view;
GO

-- Id
-- ----
-- 1

DROP TABLE base_table;
GO

CREATE TABLE base_table (fname VARCHAR(10), lname VARCHAR(10));
GO

INSERT INTO base_table (fname, lname) VALUES ('Mollie', 'Gallegos');
GO

SELECT * FROM base_view;
GO

-- Id
-- ----------
-- Mollie

EXEC sp_refreshsqlmodule @Name = 'base_view';
GO

SELECT * FROM base_view;
GO

-- fname     | lname
-- ---------- ----------
-- Mollie    | Gallegos
```

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
