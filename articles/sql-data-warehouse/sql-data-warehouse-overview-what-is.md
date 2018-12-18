---
title: 什麼是 Azure SQL 資料倉儲？ | Microsoft Docs
description: 企業級分散式資料庫，可處理資料量高達 PB 的關聯式與非關聯式資料。 它是業界首見能在幾秒內增加、縮減和暫停的雲端資料倉儲。
services: sql-data-warehouse
author: igorstanko
manager: craigg
ms.service: sql-data-warehouse
ms.topic: overview
ms.component: design
ms.date: 04/17/2018
ms.author: igorstan
ms.reviewer: igorstan
ms.openlocfilehash: 7ba1cb75b7e7c5a93438ea4f17bb563477837099
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301928"
---
# <a name="what-is-azure-sql-data-warehouse"></a>什麼是 Azure SQL 資料倉儲？

SQL 資料倉儲是雲端式企業資料倉儲 (EDW)，利用大量平行處理 (MPP) 以快速地在數以 PB 計的資料中執行複雜的查詢。 使用 SQL 資料倉儲作為巨量資料解決方案的重要元件。 使用簡單的 PolyBase T-SQL 查詢將巨量資料匯入 SQL 資料倉儲，然後使用 MPP 功能來執行高效能分析。 整合及分析時，資料倉儲將會變成您的企業可以信賴的真正單一版本。  


## <a name="key-component-of-big-data-solution"></a>巨量資料解決方案的重要元件
SQL 資料倉儲是雲端中端對端巨量資料解決方案的重要元件。

![資料倉儲解決方案](media/sql-data-warehouse-overview-what-is/data-warehouse-solution.png) 

在雲端資料解決方案中，資料會內嵌至來自各種來源的巨量資料存放區。 一旦位於巨量資料存放區中，Hadoop、Spark 和機器學習演算法就會準備及定型資料。 當資料就緒進行複雜分析時，SQL 資料倉儲會使用 PolyBase 來查詢巨量資料存放區。 PolyBase 會使用標準 T-SQL 查詢來將資料帶入 SQL 資料倉儲。
 
SQL 資料倉儲會將資料儲存到具有單欄式儲存體的關聯式資料表。 這種格式會大幅減少資料儲存體成本，而且可以改善查詢效能。 一旦資料儲存在 SQL 資料倉儲，您可以大規模執行分析。 相較於傳統資料庫系統，分析查詢在數秒鐘的時間內就可完成，不用到數分鐘，或者在數小時內而不用數天。 

分析結果可以移至全球報告資料庫或應用程式。 商務分析師便可充分了解，以進行靈活的商務決策。


## <a name="next-steps"></a>後續步驟
現在您已稍微了解 SQL 資料倉儲，請了解如何快速[建立 SQL 資料倉儲][create a SQL Data Warehouse]和[載入範例資料][load sample data]。 如果您不熟悉 Azure，您可能會發現 [Azure 詞彙][Azure glossary]在您遇到新術語時很有幫助。 或者，也可以看一下其中一些其他 SQL 資料倉儲資源。  

* [客戶成功案例]
* [部落格]
* [功能要求]
* [影片]
* [客戶諮詢小組部落格]
* [建立支援票證]
* [MSDN 論壇]
* [Stack Overflow 論壇]
* [Twitter]

<!--Image references-->
[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->
[建立支援票證]: ./sql-data-warehouse-get-started-create-support-ticket.md
[load sample data]: ./sql-data-warehouse-load-sample-databases.md
[create a SQL Data Warehouse]: ./sql-data-warehouse-get-started-provision.md
[Migration documentation]: ./sql-data-warehouse-overview-migrate.md
[SQL Data Warehouse solution partners]: ./sql-data-warehouse-partner-business-intelligence.md
[Integrated tools overview]: ./sql-data-warehouse-overview-integrate.md
[Backup and restore overview]: ./sql-data-warehouse-restore-database-overview.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md

<!--MSDN references-->

<!--Other Web references-->
[客戶成功案例]: https://azure.microsoft.com/case-studies/?service=sql-data-warehouse
[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客戶諮詢小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[MSDN 論壇]: https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=AzureSQLDataWarehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[SLA for SQL Data Warehouse]: https://azure.microsoft.com/support/legal/sla/sql-data-warehouse/v1_0/
[Volume Licensing]: http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=37
[Service Level Agreements]: https://azure.microsoft.com/support/legal/sla/
