---
title: Azure SQL 資料倉儲 - MPP 架構 | Microsoft Docs
description: 了解 Azure SQL 資料倉儲如何將大量平行處理 (MPP) 與 Azure 儲存體結合，以達到高效能和延展性。
services: sql-data-warehouse
author: ronortloff
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: design
ms.date: 04/17/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: 34b908ef79b0a2479c420675272f7d3f3bf0ff15
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286787"
---
# <a name="azure-sql-data-warehouse---massively-parallel-processing-mpp-architecture"></a>Azure SQL 資料倉儲 - 大量平行處理 (MPP) 架構
了解 Azure SQL 資料倉儲如何將大量平行處理 (MPP) 與 Azure 儲存體結合，以達到高效能和延展性。 

## <a name="mpp-architecture-components"></a>MPP 架構元件
SQL 資料倉儲會利用相應放大架構，將資料的計算處理散發到多個節點。 縮放單位是抽象的計算能力，即為資料倉儲單位。 SQL 資料倉儲會將計算與儲存體分隔開來，讓您可以在系統中單獨調整資料的計算。

![SQL 資料倉儲架構](media/massively-parallel-processing-mpp-architecture/massively-parallel-processing-mpp-architecture.png)

SQL 資料倉儲使用以節點為基礎的架構。 應用程式會連接並發出 T-SQL 命令到控制節點，此是資料倉儲的單一進入點。 控制節點會執行 MPP 引擎，將查詢最佳化以進行平行處理，然後將作業傳遞到計算節點，以平行方式執行其工作。 計算節點會在 Azure 儲存體中儲存所有使用者資料，並執行平行查詢。 資料移動服務 (DMS) 是系統層級的內部服務，其會視需要在節點之間移動資料，以平行方式執行查詢並傳回精確的結果。 

使用減少的儲存體和計算，SQL 資料倉儲可以︰

* 獨立估算計算能力，而不論您的儲存體需求為何。
* 擴大或縮小計算能力，而不移動資料。
* 暫停計算容量，同時讓資料保持不變，因此您只需支付儲存體的費用。
* 在營運時間期間繼續計算容量。

### <a name="azure-storage"></a>Azure 儲存體
SQL 資料倉儲會使用 Azure 儲存體來保護您使用者資料的安全。  因為您的資料是由 Azure 儲存體儲存及管理的，所以 SQL 資料倉儲會與您的儲存體使用情況分開計費。 資料本身會分區化到**散發**中，以將系統效能最佳化。 當您定義資料表時，可以選擇要用來散發資料的分區化模式。 SQL 資料倉儲支援這些分區化模式：

* 雜湊
* 循環配置資源
* 複寫

### <a name="control-node"></a>控制節點

控制節點是資料倉儲的大腦。 它是與所有應用程式與連接互動的前端。 MPP 引擎會在控制節點上執行，以便將平行查詢最佳化並加以協調。 當您將 T-SQL 查詢提交到 SQL 資料倉儲時，控制節點會將其轉換為要根據每個散發平行執行的查詢。

### <a name="compute-nodes"></a>計算節點數量

計算節點提供計算能力。 散發會對應到計算節點以進行處理。 當您需要支付更多計算資源的費用時，SQL 資料倉儲會將散發重新對應到可用的計算節點。 計算節點數目範圍是從 1 到 60，取決於資料倉儲的服務等級。

每個計算節點都有會在系統檢視中顯示的節點識別碼。 您可以在名稱開頭為 sys.pdw_nodes 的系統檢視中尋找 node_id 資料行，以查看計算節點識別碼。 如需這些系統檢視的清單，請參閱 [MPP 系統檢視](sql-data-warehouse-reference-tsql-statements.md)。

### <a name="data-movement-service"></a>資料移動服務
資料移動服務 (DMS) 是資料傳輸技術，可協調計算節點之間的資料移動。 某些查詢需要資料移動，以確保平行查詢會傳回精確的結果。 若資料移動是必要的，DMS 確保正確的資料會到達正確的位置。 

## <a name="distributions"></a>散發

散發是儲存體的基本單位，可處理在分散式資料上執行的平行查詢。 當 SQL 資料倉儲執行查詢時，會將工作分成 60 個平行執行的較小查詢。 這 60 個較小查詢中的每一個都會在其中一個資料散發中執行。 每個計算節點都會管理這 60 個散發中的一或多個。 具有最大計算資源的資料倉儲，在每個計算節點上都有一個散發。 具有最小計算資源的資料倉儲，其所有散發都會在一個計算節點上。  

## <a name="hash-distributed-tables"></a>雜湊分散式資料表
雜湊分散式資料表可以針對大型資料表上的聯結和彙總提供最高查詢效能。 

為了將資料分區化到雜湊分散式資料表，SQL 資料倉儲會使用雜湊函式，決定性地將每個資料列指派給一個散發。 在資料表定義中，其中一個資料行會指定為發佈資料行。 雜湊函式會使用散發資料行中的值，將每個資料列指派給一個散發。

下圖說明完整 (非分散式資料表) 會如何儲存為雜湊分散式資料表。 

![分散式資料表](media/sql-data-warehouse-distributed-data/hash-distributed-table.png "分散式資料表")  

* 每個資料列屬於一種發佈。  
* 具決定性的雜湊演算法會將每個資料列指派給一個發佈。  
* 每個發佈的資料表資料列數目會隨著顯示不同資料表大小而有所不同。

選取散發資料行有效能考量，例如區分、資料扭曲，以及在系統上執行的查詢類型。

## <a name="round-robin-distributed-tables"></a>循環配置資源分散式資料表
循環配置資源資料表是要建立的最簡單資料表，可以在用來作為暫存資料表以供載入使用時提供快速效能。

循環配置資源分散式資料表會在整個資料表中平均散發資料，但不需任何進一步最佳化。 系統會先隨機選擇一個散發，然後將資料列的緩衝區循序指派給散發。 將資料快速載入循環配置資源資料表，但搭配雜湊分散式資料表通常會有較佳的查詢效能。 循環配置資源資料表上的聯結需要重新輪換資料，而這需要額外時間。


## <a name="replicated-tables"></a>複寫的資料表
複寫的資料表會為小型資料表提供最快速的查詢效能。

複寫的資料表會在每個計算節點上快取一份完整的資料表複本。 因此，複寫資料表就不需在進行聯結或彙總之前，於計算節點之間傳輸資料。 複寫的資料表最好能與小型資料表搭配使用。 需要額外的儲存體，而且，會在寫入資料而導致大型資料表不適用時產生額外的負擔。  

下圖顯示複寫的資料表。 對於 SQL 資料倉儲，會在每個計算節點中的第一個散發上快取複寫的資料表。  

![複寫的資料表](media/sql-data-warehouse-distributed-data/replicated-table.png "複寫的資料表") 

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
[MSDN 論壇]: https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureSQLDataWarehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[SLA for SQL Data Warehouse]: https://azure.microsoft.com/support/legal/sla/sql-data-warehouse/v1_0/
[Volume Licensing]: http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=37
[Service Level Agreements]: https://azure.microsoft.com/support/legal/sla/
