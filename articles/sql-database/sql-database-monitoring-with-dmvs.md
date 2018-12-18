---
title: 使用動態管理檢視監視 Azure SQL Database | Microsoft Docs
description: 了解如何使用動態管理檢視監視 Microsoft Azure SQL Database 來偵測和診斷常見的效能問題。
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 08/08/2018
ms.openlocfilehash: 97907eee9982fdf6a804bc13edbf8c14efa4ce42
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161381"
---
# <a name="monitoring-azure-sql-database-using-dynamic-management-views"></a>使用動態管理檢視監視 Azure SQL Database
Microsoft Azure SQL Database 可使用動態管理檢視的子集來診斷可能因為封鎖或長時間執行的查詢、資源瓶頸、不佳的查詢計畫等等所造成的效能問題。 本主題提供有關如何使用動態管理檢視來偵測常見效能問題的資訊。

SQL Database 部分支援動態管理檢視的三個類別目錄：

* 資料庫相關的動態管理檢視。
* 執行相關的動態管理檢視。
* 交易相關的動態管理檢視。

如需動態管理檢視的詳細資訊，請參閱《SQL Server 線上叢書》中的 [動態管理檢視和函數 (Transact-SQL)](https://msdn.microsoft.com/library/ms188754.aspx) 。

## <a name="permissions"></a>權限
在 SQL Database 中，查詢動態管理檢視需要 **VIEW DATABASE STATE** 權限。 **VIEW DATABASE STATE** 權限會傳回目前資料庫中所有物件的相關資訊。
若要授與 **VIEW DATABASE STATE** 權限給特定的資料庫使用者，請執行下列查詢：

```GRANT VIEW DATABASE STATE TO database_user; ```

在內部部署 SQL Server 的執行個體中，動態管理檢視會傳回伺服器狀態資訊。 在 SQL Database 中，僅會傳回與您目前的邏輯資料庫相關的資訊。

## <a name="calculating-database-size"></a>正在計算資料庫大小
下列查詢會傳回資料庫的大小 (以 MB 為單位)：

```
-- Calculates the size of the database.
SELECT SUM(CAST(FILEPROPERTY(name, 'SpaceUsed') AS bigint) * 8192.) / 1024 / 1024 AS DatabaseSizeInMB
FROM sys.database_files
WHERE type_desc = 'ROWS';
GO
```

下列查詢會傳回您資料庫中個別物件的大小 (以 MB 為單位)：

```
-- Calculates the size of individual database objects.
SELECT sys.objects.name, SUM(reserved_page_count) * 8.0 / 1024
FROM sys.dm_db_partition_stats, sys.objects
WHERE sys.dm_db_partition_stats.object_id = sys.objects.object_id
GROUP BY sys.objects.name;
GO
```

## <a name="monitoring-connections"></a>監視連線
您可以使用 [sys.dm_exec_connections](https://msdn.microsoft.com/library/ms181509.aspx) 檢視，擷取與特定 Azure SQL Database 伺服器建立之連接和每個連接之詳細資料的相關資訊。 此外，[sys.dm_exec_sessions](https://msdn.microsoft.com/library/ms176013.aspx) 檢視有助於擷取所有作用中使用者的連接資訊和內部工作。
下列查詢可擷取目前的連接資訊：

```
SELECT
    c.session_id, c.net_transport, c.encrypt_option,
    c.auth_scheme, s.host_name, s.program_name,
    s.client_interface_name, s.login_name, s.nt_domain,
    s.nt_user_name, s.original_login_name, c.connect_time,
    s.login_time
FROM sys.dm_exec_connections AS c
JOIN sys.dm_exec_sessions AS s
    ON c.session_id = s.session_id
WHERE c.session_id = @@SPID;
```

> [!NOTE]
> 執行 **sys.dm_exec_requests** 和 **sys.dm_exec_sessions views** 時，如果您具備資料庫的 **VIEW DATABASE STATE** 權限，您就會看到該資料庫上所有正在執行的工作階段；否則您只會看到目前的工作階段。
> 
> 

## <a name="monitor-resource-use"></a>監視資源使用量

您可以使用 [SQL Database 查詢效能深入解析](sql-database-query-performance.md)和[查詢存放區](https://msdn.microsoft.com/library/dn817826.aspx)來監視資源用量。

您也可以使用以下兩種檢視來監視用量：

* [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx)
* [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx)

### <a name="sysdmdbresourcestats"></a>sys.dm_db_resource_stats
您可以在每一個 SQL Database 中使用 [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx)檢視。 **sys.dm_db_resource_stats** 檢視可顯示相對於服務層的最新資源使用量資料。 每隔 15 秒鐘就會記錄一次 CPU、資料 IO、記錄檔寫入和記憶體的平均百分比，並且會維持 1 小時。

因為此檢視會提供更細微的資源使用量資訊，請先使用 **sys.dm_db_resource_stats** 來進行任何現狀分析或疑難排解。 例如，下列查詢會顯示目前的資料庫在過去一小時的平均和最大資源使用量：

    SELECT  
        AVG(avg_cpu_percent) AS 'Average CPU use in percent',
        MAX(avg_cpu_percent) AS 'Maximum CPU use in percent',
        AVG(avg_data_io_percent) AS 'Average data IO in percent',
        MAX(avg_data_io_percent) AS 'Maximum data IO in percent',
        AVG(avg_log_write_percent) AS 'Average log write use in percent',
        MAX(avg_log_write_percent) AS 'Maximum log write use in percent',
        AVG(avg_memory_usage_percent) AS 'Average memory use in percent',
        MAX(avg_memory_usage_percent) AS 'Maximum memory use in percent'
    FROM sys.dm_db_resource_stats;  

如需其他查詢的資訊，請參閱 [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx) 中的範例。

### <a name="sysresourcestats"></a>sys.resource_stats
**master** 資料庫中的 [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx) 檢視有其他資訊可協助您監視 SQL Database 在其特定服務層和計算大小的效能。 這項資料每隔 5 分鐘就會收集一次，並且會維持大約 14 天。 這個檢視適合用於進行 SQL Database 如何使用資源的長期歷史分析。

下圖顯示在一週中 P2 計算大小之高階資料庫每小時的 CPU 資源使用量。 此圖從星期一開始，顯示 5 個工作天，然後顯示較少發生在應用程式的週末。

![SQL Database 的資源使用量](./media/sql-database-performance-guidance/sql_db_resource_utilization.png)

從資料中，這個資料庫目前相對於 P2 計算大小的尖峰 CPU 負載剛好超過 50% 的 CPU 使用量 (星期二中午)。 如果 CPU 是應用程式的資源設定檔中的主要因素，您可能會決定 P2 是正確的計算大小，以確保永遠符合工作負載。 如果您預期應用程式會隨著時間成長，最好預留額外的資源緩衝，讓應用程式永遠不會達到效能等級限制。 如果您提升計算大小，則可協助避免資料庫沒有足夠能力來有效處理要求時可能發生的客戶可見錯誤，尤其是在延遲敏感的環境中。 其中一例便是支援可根據資料庫呼叫結果繪製網頁之應用程式的資料庫。

其他應用程式類型可能會以不同方式解譯相同的圖形。 例如，如果應用程式嘗試每天處理薪資資料而且具有同一張圖表，這種「批次作業」模型可能會在 P1 計算大小中正常執行。 P1 計算大小有 100 個 DTU，相較之下，P2 計算大小則有 200 個 DTU。 P1 計算大小提供的效能是 P2 計算大小的一半。 因此，P2 中 50% 的 CPU 使用量等於 P1 中 100% 的 CPU 使用量。 如果應用程式沒有逾時，就算作業花了 2 個小時或 2.5 個小時才能完成也沒關係，只要它在今天內完成即可。 這個類別中的應用程式或許可以使用 P1 計算大小。 一天中有好幾個時段的資源使用量較低，您可以善用這個事實，讓任何「巨量尖峰」可以溢出到當天稍後的一個低谷。 只要作業可以每天及時完成，P1 計算大小可能就很適合這類應用程式 (並節省經費)。

Azure SQL Database 會在每個伺服器 **master** 資料庫的 **sys.resource_stats** 檢視中公開每個作用中資料庫的耗用資源資訊。 資料表中的資料會以 5 分鐘的間隔彙總。 利用基本、標準和進階服務層，資料可能要花 5 分鐘以上的時間才會出現在資料表中，因此這項資料比較適合進行歷程記錄分析而不是近乎即時的分析。 查詢 **sys.resource_stats** 檢視可查看資料庫的近期歷程記錄，並驗證所選的保留是否會在必要時提供所需的效能。

> [!NOTE]
> 您必須連接到邏輯 SQL Database 伺服器的 **master** 資料庫才能在下列範例中查詢 **sys.resource_stats**。
> 
> 

下列範例會示範如何公開此檢視中的資料：

    SELECT TOP 10 *
    FROM sys.resource_stats
    WHERE database_name = 'resource1'
    ORDER BY start_time DESC

![sys.resource_stats 目錄檢視](./media/sql-database-performance-guidance/sys_resource_stats.png)

下列範例示範不同方式，以供您用來使用 **sys.resource_stats** 目錄檢視取得有關 SQL Database 如何使用資源的相關資訊：

1. 若要查看 userdb1 資料庫在過去一週的資源使用量，您可以執行下列查詢：
   
        SELECT *
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND
              start_time > DATEADD(day, -7, GETDATE())
        ORDER BY start_time DESC;
2. 若要評估您的工作負載與計算大小的符合程度，您必須鑽研資源度量的每個層面：CPU、讀取、寫入、背景工作角色數目和工作階段數目。 以下是使用 **sys.resource_stats** 修訂過的查詢，可報告這些資源度量的平均值和最大值：
   
        SELECT
            avg(avg_cpu_percent) AS 'Average CPU use in percent',
            max(avg_cpu_percent) AS 'Maximum CPU use in percent',
            avg(avg_data_io_percent) AS 'Average physical data IO use in percent',
            max(avg_data_io_percent) AS 'Maximum physical data IO use in percent',
            avg(avg_log_write_percent) AS 'Average log write use in percent',
            max(avg_log_write_percent) AS 'Maximum log write use in percent',
            avg(max_session_percent) AS 'Average % of sessions',
            max(max_session_percent) AS 'Maximum % of sessions',
            avg(max_worker_percent) AS 'Average % of workers',
            max(max_worker_percent) AS 'Maximum % of workers'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
3. 利用這項有關各資源度量平均值和最大值的資訊，您可以評估您的工作負載與您所選之計算大小的符合程度。 通常，來自 **sys.resource_stats** 的平均值可提供您對目標大小所使用的理想基準。 它應該是您主要的量尺。 例如，您可能使用標準服務層搭配 S2 計算大小。 CPU 以及 IO 讀取和寫入的平均使用量百分比低於 40%，背景工作角色平均數目低於 50，而且工作階段平均數目低於 200。 您的工作負載可能符合 S1 計算大小。 要看到您的資料庫是否符合背景工作和工作階段限制範圍內非常容易。 若要查看資料庫在 CPU、讀取和寫入方面是否符合較低的計算大小，請將較低計算大小的 DTU 數目除以目前計算大小的 DTU 數目，然後將結果乘以 100：
   
    **S1 DTU / S2 DTU * 100 = 20 / 50 * 100 = 40**
   
    此結果是以百分比表示之兩個計算大小的相對效能差異。 如果您的資源使用量未超過這個數量，您的工作負載可能符合較低的計算大小。 不過，您需要查看所有範圍的資源使用量值，並以百分比判斷資料庫工作負載符合較低計算大小的頻率。 下列查詢會根據我們在此範例中計算的 40% 臨界值，輸出每個資源維度的相符百分比：
   
        SELECT
            (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log Write Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical Data IO Fit Percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
   
    根據資料庫服務層，您可以決定工作負載是否符合較低的計算大小。 如果資料庫工作負載目標是 99.9%，且上述查詢針對三個資源維度傳回的值都大於 99.9，則工作負載可能會符合較低的計算大小。
   
    查看相符百分比也可讓您深入了解是否必須移到下一個較高的計算大小來滿足目標。 例如，userdb1 會顯示過去一週的下列 CPU 使用量：
   
   | 平均 CPU 百分比 | 最大 CPU 百分比 |
   | --- | --- |
   | 24.5 |100.00 |
   
    平均 CPU 大約是計算大小限制的四分之一，完全符合資料庫的計算大小。 不過，此最大值會顯示資料庫達到計算大小的限制。 您需要移至下一個較高的計算大小嗎？ 請查看工作負載達到 100% 的次數，然後將其與您的資料庫工作負載目標做比較。
   
        SELECT
        (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log write fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical data IO fit percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
   
    如果此查詢針對三個資源維度傳回的值小於 99.9%，請考慮移到下一個較高的計算大小或使用應用程式微調技術減少 SQL Database 的負載。
4. 此練習也會考慮您預計的未來工作負載增加量。

若為彈性集區，您可以監視其中的個別資料庫，技巧如本節所述。 但您也可以監視集區整體。 如需詳細資訊，請參閱 [監視和管理彈性集區](sql-database-elastic-pool-manage-portal.md)。


### <a name="maximum-concurrent-requests"></a>並行要求數上限
若要查看並行要求數目，請在 SQL Database 上執行下列 Transact-SQL 查詢：

    SELECT COUNT(*) AS [Concurrent_Requests]
    FROM sys.dm_exec_requests R

若要分析內部部署 SQL Server 資料庫的工作負載，請修改此查詢來篩選您要分析的特定資料庫。 比方說，如果您擁有名為 MyDatabase 的內部部署資料庫，則下列 Transact-SQL 查詢會傳回該資料庫中並行要求的計數：

    SELECT COUNT(*) AS [Concurrent_Requests]
    FROM sys.dm_exec_requests R
    INNER JOIN sys.databases D ON D.database_id = R.database_id
    AND D.name = 'MyDatabase'

這只是單一時間點的快照。 若要進一步了解您的工作負載和並行要求需求，您必須收集一段時間內的許多範例。

### <a name="maximum-concurrent-logins"></a>並行登入數上限
您可以分析您的使用者和應用程式模式以了解登入頻率。 您也可以在測試環境中執行真實世界的負載，藉此確定您不會達到我們在本文中討論的這項限制或其他限制。 沒有任何單一查詢或動態管理檢視 (DMV) 可以向您顯示並行登入計數或歷程記錄。

如果這些用戶端使用相同的連接字串，服務仍會驗證每一個登入。 如果有 10 位使用者同時以相同的使用者名稱和密碼連接到資料庫，將會有 10 個並行登入。 這項限制只適用於登入和驗證期間。 如果相同的 10 位使用者依序連接到資料庫，並行登入數目絕對不會大於 1。

> [!NOTE]
> 這項限制目前不適用於彈性集區中的資料庫。
> 
> 

### <a name="maximum-sessions"></a>工作階段數上限
若要查看目前的作用中工作階段數目，請在 SQL Database 上執行下列 Transact-SQL 查詢：

    SELECT COUNT(*) AS [Sessions]
    FROM sys.dm_exec_connections

如果您要分析內部部署 SQL Server 的工作負載，請修改查詢以專注於特定資料庫。 如果您考慮將資料庫移至 Azure SQL Database，此查詢可協助您判斷該資料庫可能的工作階段需求。

    SELECT COUNT(*)  AS [Sessions]
    FROM sys.dm_exec_connections C
    INNER JOIN sys.dm_exec_sessions S ON (S.session_id = C.session_id)
    INNER JOIN sys.databases D ON (D.database_id = S.database_id)
    WHERE D.name = 'MyDatabase'

同樣地，這些查詢傳回的是某一時間點的計數。 如果您收集一段時間內的多個範例，您就可以充分了解您的工作階段使用量。

若要進行 SQL Database 分析，您可以查詢 [sys.resource_stats](https://msdn.microsoft.com/library/dn269979.aspx)檢視，並檢閱 **active_session_count** 資料行，以取得工作階段的歷史統計資料。 

## <a name="monitoring-query-performance"></a>監視查詢效能
速度慢或長時間執行的查詢可能會耗用大量系統資源。 本節示範如何使用動態管理檢視偵測幾個常見的查詢效能問題。 在 Microsoft TechNet 上的文章 [疑難排解 SQL Server 2008 的效能問題](http://download.microsoft.com/download/D/B/D/DBDE7972-1EB9-470A-BA18-58849DB3EB3B/TShootPerfProbs2008.docx) 雖然較舊，但仍是針對疑難排解相當有用的參考資料。

### <a name="finding-top-n-queries"></a>尋找前 N 個查詢
下列範例會傳回以平均 CPU 時間排名的前五個查詢的相關資訊。 此範例會根據查詢雜湊來彙總查詢，使在邏輯上等同的查詢依其累積資源耗用量進行分組。

```
SELECT TOP 5 query_stats.query_hash AS "Query Hash",
    SUM(query_stats.total_worker_time) / SUM(query_stats.execution_count) AS "Avg CPU Time",
    MIN(query_stats.statement_text) AS "Statement Text"
FROM
    (SELECT QS.*,
    SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
    ((CASE statement_end_offset
        WHEN -1 THEN DATALENGTH(ST.text)
        ELSE QS.statement_end_offset END
            - QS.statement_start_offset)/2) + 1) AS statement_text
     FROM sys.dm_exec_query_stats AS QS
     CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
GROUP BY query_stats.query_hash
ORDER BY 2 DESC;
```

### <a name="monitoring-blocked-queries"></a>監視封鎖的查詢
速度慢或長時間執行的查詢會造成過度的資源耗用，並且會導致封鎖查詢的後果。 導致封鎖的原因可能是不佳的應用程式設計、不良的查詢計畫、缺乏有用的索引等等。 您可以使用 sys.dm_tran_locks 檢視，取得在您的 Azure SQL Database 中目前正鎖定之活動的相關資訊。 如需範例程式碼，請參閱《SQL Server 線上叢書》中的 [sys.dm_tran_locks (Transact-SQL)](https://msdn.microsoft.com/library/ms190345.aspx)。

### <a name="monitoring-query-plans"></a>監視查詢計畫
效率不佳的查詢計畫也可能會增加 CPU 耗用量。 下列範例使用 [sys.dm_exec_query_stats](https://msdn.microsoft.com/library/ms189741.aspx) 檢視來判斷哪一個查詢使用最多的累計 CPU。

```
SELECT
    highest_cpu_queries.plan_handle,
    highest_cpu_queries.total_worker_time,
    q.dbid,
    q.objectid,
    q.number,
    q.encrypted,
    q.[text]
FROM
    (SELECT TOP 50
        qs.plan_handle,
        qs.total_worker_time
    FROM
        sys.dm_exec_query_stats qs
    ORDER BY qs.total_worker_time desc) AS highest_cpu_queries
    CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS q
ORDER BY highest_cpu_queries.total_worker_time DESC;
```

## <a name="see-also"></a>另請參閱
[SQL Database 簡介](sql-database-technical-overview.md)

