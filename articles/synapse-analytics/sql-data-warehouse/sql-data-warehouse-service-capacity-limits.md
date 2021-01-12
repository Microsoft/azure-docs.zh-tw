---
title: 專用 SQL 集區的容量限制
description: Azure Synapse Analytics 中專用 SQL 集區的各種元件允許的最大值。
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 2/19/2020
ms.author: martinle
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: d778844fee8cad9359532ffa23e177bf7b13c4b8
ms.sourcegitcommit: aacbf77e4e40266e497b6073679642d97d110cda
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98117685"
---
# <a name="capacity-limits-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Azure Synapse Analytics 中專用 SQL 集區的容量限制

Azure Synapse Analytics 中專用 SQL 集區的各種元件允許的最大值。

## <a name="workload-management"></a>工作負載管理

| 類別 | 描述 | 最大值 |
|:--- |:--- |:--- |
| [資料倉儲單位 (DWU)](what-is-a-data-warehouse-unit-dwu-cdwu.md) |單一專用 SQL 集區的 DWU 上限  | Gen1：DW6000<br></br>Gen2：DW30000c |
| [資料倉儲單位 (DWU)](what-is-a-data-warehouse-unit-dwu-cdwu.md) |每一部伺服器的預設 DTU |54,000<br></br>每個 SQL Server (例如 myserver.database.windows.net) 的 DTU 配額為 54,000，最多允許 DW6000c。 此配額僅是安全限制。 您可以藉由 [建立支援票證](sql-data-warehouse-get-started-create-support-ticket.md) ，並選取 [ *配額* ] 作為要求類型來增加配額。  若要計算 DTU 需求，請將7.5 乘以所需的總 DWU，或將9乘以所需的總 cDWU 數。 例如：<br></br>DW6000 x 7.5 = 45,000 DTU<br></br>DW7500c x 9 = 67500 Dtu。<br></br>您可以在入口網站的 [SQL Server] 選項中檢視目前的 DTU 耗用量。 已暫停和未暫停的資料庫都會計入 DTU 配額。 |
| 資料庫連接 |並行開啟的會話數上限 |1024<br/><br/>並行開啟的會話數目會根據選取的 DWU 而有所不同。 DWU600c 和更新版本最多支援1024個開啟中的會話。 DWU500c 和以下支援最大並行開啟會話限制512。 請注意，可同時執行的查詢數目有所限制。 超過並行存取限制時，要求會進入內部佇列以等待處理。 |
| 資料庫連接 |準備陳述式的最大記憶體 |20 MB |
| [工作負載管理](resource-classes-for-workload-management.md) |並行查詢數目上限 |128<br/><br/>  最多會執行128個並行查詢，其餘查詢將會排入佇列。<br/><br/>當使用者指派給較高的資源類別，或當 [資料倉儲單位](memory-concurrency-limits.md) 設定降低時，並行查詢的數目可能會降低。 系統總是會允許某些查詢 (例如 DMV 查詢) 執行，而不影響並行查詢限制。 如需並行查詢執行的詳細資訊，請參閱 [並行](memory-concurrency-limits.md) 存取最大的文章。 |
| [tempdb](sql-data-warehouse-tables-temporary.md) |GB 上限 |每個 DW100c 399 GB。 在 DWU1000c 中，tempdb 的大小為 3.99 TB。 |
||||

## <a name="database-objects"></a>資料庫物件

| 類別 | 描述 | 最大值 |
|:--- |:--- |:--- |
| 資料庫 |大小上限 | Gen1：在磁碟上壓縮後 240 TB。 此空間與 tempdb 或記錄檔空間無關，因此此空間為供永久資料表專用。  叢集資料行存放區壓縮估計為 5 X。  當所有資料表都是叢集資料行存放區 (預設的資料表類型) 時，這個壓縮可讓資料庫成長約 1 PB。 <br/><br/> Gen2：資料行存放區資料表的無限制儲存體。  資料庫的 Rowstore 部分仍受限於在磁片上壓縮的 240 TB。 |
| Table |大小上限 |資料行存放區資料表的大小不限。 <br>針對壓縮在磁片上的 rowstore 資料表，則為 60 TB。 |
| Table |每個資料庫的資料表數 | 100,000 |
| Table |資料表的資料行數 |1024 個資料行 |
| Table |每個資料行的位元組 |相依於資料行[資料類型](sql-data-warehouse-tables-data-types.md)。 char 資料類型的限制為 8000、nvarchar 為 4000 或 MAX 資料類型為 2 GB。 |
| Table |每個資料列的位元組，已定義的大小 |8060 個位元組<br/><br/>每個資料列的位元組數目計算方式和使用頁面壓縮的 SQL Server 所使用的方式相同。 和 SQL Server 一樣，支援資料列溢位儲存體，可讓 **可變長度** 資料行以非資料列方式推送。 可變長度的資料列會發送至超出資料列，只有 24 位元組的根會儲存在主要記錄中。 如需詳細資訊，請參閱 [超過 8 KB 的資料列溢位資料](/previous-versions/sql/sql-server-2008-r2/ms186981(v=sql.105))。 |
| Table |每個資料表的資料分割 |15,000<br/><br/>為了獲得高效能，建議在仍能支援業務需求的情況下，將您需要的資料分割數目降至最低。 隨著資料分割數目增加，資料定義語言 (DDL) 和資料操作語言 (DML) 作業的負荷會加重，導致效能變慢。 |
| Table |每個資料分割界限值的字元。 |4000 |
| 索引 |每個資料表的非叢集索引。 |50<br/><br/>僅適用於資料列存放區資料表。 |
| 索引 |每個資料表的叢集索引。 |1<br><br/>適用於資料列存放區資料表和資料行存放區資料表。 |
| 索引 |索引鍵的大小。 |900 個位元組。<br/><br/>僅適用於資料列存放區索引。<br/><br/>建立索引時，如果資料行中的現有資料沒有超過 900 個位元組，就可以在 varchar 資料行上建立大小上限超過 900 個位元組的索引。 不過，後續在資料行上執行 INSERT 或 UPDATE 動作時，如果總計大小超過 900 個位元組，將會失敗。 |
| 索引 |每個索引的索引鍵資料行。 |16<br/><br/>僅適用於資料列存放區索引。 叢集資料行存放區索引包含所有資料行。 |
| 統計資料 |結合資料行值的大小。 |900 個位元組。 |
| 統計資料 |每個統計資料物件的資料行。 |32 |
| 統計資料 |每個資料表的資料行上建立的統計資料。 |30,000 |
| 預存程序 |最大巢狀層級。 |8 |
| 檢視 |每個檢視的資料行數 |1,024 |
||||

## <a name="loads"></a>負載

| 類別 | 描述 | 最大值 |
|:--- |:--- |:--- |
| PolyBase 載入 |每列 MB 數 |1<br/><br/>Polybase 會載入小於 1 MB 的資料列。 不支援將 LOB 資料類型載入具有叢集資料行存放區索引的資料表 (CCI) 。<br/> |
|PolyBase 載入|檔案總數|1,000,000<br/><br/>Polybase 載入不能超過1M 個以上的檔案。 您可能會遇到下列錯誤： **因為分割計數超過1000000的上限**，所以作業失敗。|

## <a name="queries"></a>查詢

| 類別 | 描述 | 最大值 |
|:--- |:--- |:--- |
| 查詢 |使用者資料表上已排入佇列的查詢。 |1000 |
| 查詢 |系統檢視表上的並行查詢。 |100 |
| 查詢 |系統檢視表上已排入佇列的查詢 |1000 |
| 查詢 |參數個數上限 |2098 |
| Batch |大小上限 |65,536*4096 |
| SELECT 結果 |每個資料列的資料行 |4096<br/><br/>在 SELECT 結果中，每個資料列一律不超過 4096 個資料行。 不保證一定可以有 4096 個。 如果查詢計畫需要暫存資料表，可能會限定每個資料表最多 1024 個資料行。 |
| SELECT |巢狀子查詢 |32<br/><br/>SELECT 陳述式中一律不超過 32 個巢狀子查詢。 不保證一定可以有 32 個。 例如，JOIN 可以將子查詢加入查詢計畫中。 子查詢的數目也受限於可用記憶體。 |
| SELECT |每個 JOIN 的資料行 |1024 個資料行<br/><br/>JOIN 中一律不超過 1024 個資料行。 不保證一定可以有 1024 個。 如果 JOIN 計畫需要比 JOIN 結果更多資料行的暫存資料表，暫存資料表會受限於 1024 的限制。 |
| SELECT |每個 GROUP BY 資料行的位元組。 |8060<br/><br/>GROUP BY 子句中的資料行最多可以有 8060 個位元組。 |
| SELECT |每個 ORDER BY 資料行的位元組 |8060 個位元組<br/><br/>ORDER BY 子句中的資料行最多可以有 8060 個位元組 |
| 每個陳述式的識別項 |參考的識別項個數 |65,535<br/><br/> 查詢的單一運算式中可包含的識別碼數目有限。 超過此數字會導致 SQL Server 錯誤 8632。 如需詳細資訊，請參閱[內部錯誤：到達運算式服務的限制](https://support.microsoft.com/help/913050/error-message-when-you-run-a-query-in-sql-server-2005-internal-error-a)。 |
| 字串常值 | 陳述式中的字串常值數目 | 20,000 <br/><br/>查詢的單一運算式中的字串常數數目有限。 超過此數字會導致 SQL Server 錯誤 8632。|
||||

## <a name="metadata"></a>中繼資料

| 系統檢視表 | 最大資料列數 |
|:--- |:--- |
| sys.dm_pdw_component_health_alerts |10,000 |
| sys.dm_pdw_dms_cores |100 |
| sys.dm_pdw_dms_workers |最近 1000 個 SQL 要求的 DMS 背景工作角色總數。 |
| sys.dm_pdw_errors |10,000 |
| sys.dm_pdw_exec_requests |10,000 |
| sys.dm_pdw_exec_sessions |10,000 |
| sys.dm_pdw_request_steps |儲存在 sys.dm_pdw_exec_requests 中的最近 1000 個 SQL 要求的步驟總數。 |
| sys.dm_pdw_os_event_logs |10,000 |
| sys.dm_pdw_sql_requests |儲存在 sys.dm_pdw_exec_requests 中的最近 1000 個 SQL 要求。 |
|||

## <a name="next-steps"></a>後續步驟

如需使用 Azure [Synapse 的建議](cheat-sheet.md)，請參閱功能提要。