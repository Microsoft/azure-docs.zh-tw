---
title: 使用 DMV 監視工作負載 | Microsoft Docs
description: 了解如何使用 DMV 監視工作負載。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: fe989a1693d73dbbea7ed0e3e91ed7aaf6fc37c4
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301077"
---
# <a name="monitor-your-workload-using-dmvs"></a>使用 DMV 監視工作負載
本文說明如何使用動態管理檢視 (DMV) 來監視您的工作負載。 這包括調查 Azure SQL 資料倉儲中的查詢執行。

## <a name="permissions"></a>權限
若要查詢此文章中的 DMV，您需要「檢視資料庫狀態」或「控制」權限。 通常授與「檢視資料庫狀態」是慣用的權限，因為它較具限制性。

```sql
GRANT VIEW DATABASE STATE TO myuser;
```

## <a name="monitor-connections"></a>監視連接
所有針對 SQL 資料倉儲的登入都會記錄到 [sys.dm_pdw_exec_sessions][sys.dm_pdw_exec_sessions]。  這個 DMV 會包含最後 10,000 筆登入。  Session_id 是主索引鍵，並依序指派給每個新的登入。

```sql
-- Other Active Connections
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed' and session_id <> session_id();
```

## <a name="monitor-query-execution"></a>監視查詢執行
SQL 資料倉儲上所執行的所有查詢會都記錄到 [sys.dm_pdw_exec_requests][sys.dm_pdw_exec_requests]。  這個 DMV 會包含最後 10,000 筆執行的查詢。  Request_id 可唯一識別每筆查詢，而且是此 DMV 的主索引鍵。  Request_id 會依序指派給每筆新查詢，並加上 QID 代表查詢識別碼。  查詢此 DMV 來尋找指定的 session_id，即會顯示指定登入的所有查詢。

> [!NOTE]
> 預存程序會使用多個要求 ID。  要求 ID 是依序指派。 
> 
> 

請遵循以下步驟來調查特定查詢的查詢執行計畫和時間。

### <a name="step-1-identify-the-query-you-wish-to-investigate"></a>步驟 1︰識別您想要調查的查詢
```sql
-- Monitor active queries
SELECT * 
FROM sys.dm_pdw_exec_requests 
WHERE status not in ('Completed','Failed','Cancelled')
  AND session_id <> session_id()
ORDER BY submit_time DESC;

-- Find top 10 queries longest running queries
SELECT TOP 10 * 
FROM sys.dm_pdw_exec_requests 
ORDER BY total_elapsed_time DESC;

-- Find a query with the Label 'My Query'
-- Use brackets when querying the label column, as it it a key word
SELECT  *
FROM    sys.dm_pdw_exec_requests
WHERE   [label] = 'My Query';
```

從前述的查詢結果中，記下您想要調查之查詢的 **要求 ID** 。

 狀態的查詢會因為並行限制而進入佇列。 這些查詢也會顯示在 sys.dm_pdw_waits 等候查詢中，類型為 UserConcurrencyResourceType。 如需有關並行限制的資訊，請參閱[效能層級](performance-tiers.md)或[適用於工作負載管理的資源類別](resource-classes-for-workload-management.md)。 查詢也會因其他原因 (例如物件鎖定) 而等候。  如果您的查詢正在等候資源，請參閱本文稍後的[檢查正在等候資源的查詢][Investigating queries waiting for resources]。

若要簡化在 sys.dm_pdw_exec_requests 資料表中查閱查詢的方式，請使用 [LABEL][LABEL] 來將可在 sys.dm_pdw_exec_requests 檢視中查閱的註解指派給您的查詢。

```sql
-- Query with Label
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query')
;
```

### <a name="step-2-investigate-the-query-plan"></a>步驟 2︰ 調查查詢計劃
使用要求識別碼，從 [sys.dm_pdw_request_steps][sys.dm_pdw_request_steps] 擷取查詢的分散式 SQL (DSQL) 計畫。

```sql
-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID####'
ORDER BY step_index;
```

當 DSQL 計劃所花的時間超出預期時，有可能是含有許多 DSQL 步驟的複雜計劃所導致，或只是某個步驟需要長時間處理。  如果計劃是含有數個移動作業的許多步驟，請考慮最佳化您的資料表散發以減少資料移動。 [資料表散發][Table distribution]一文說明為何需要移動資料才能解決查詢，並說明最小化資料移動的一些散發策略。

進一步調查單一步驟 (長時間執行查詢步驟的 *operation_type* 資料行) 的詳細資料，並且記下**步驟索引**：

* 針對下列 **SQL 作業**繼續執行步驟 3a：OnOperation、RemoteOperation、ReturnOperation。
* 針對下列 **資料移動作業**繼續執行步驟 3b：ShuffleMoveOperation、BroadcastMoveOperation、TrimMoveOperation、PartitionMoveOperation、MoveOperation、CopyOperation。

### <a name="step-3a-investigate-sql-on-the-distributed-databases"></a>步驟 3a︰調查分散式資料庫上的 SQL
使用要求識別碼及步驟索引，從 [sys.dm_pdw_sql_requests][sys.dm_pdw_sql_requests] 擷取詳細資料，其中包含所有分散式資料庫上查詢步驟的執行資訊。

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID####' AND step_index = 2;
```

當查詢步驟正在執行時，可以使用 [DBCC PDW_SHOWEXECUTIONPLAN][DBCC PDW_SHOWEXECUTIONPLAN] 針對正在特定散發上執行的步驟，從 SQL Server 計畫快取擷取 SQL Server 預估計畫。

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);
```

### <a name="step-3b-investigate-data-movement-on-the-distributed-databases"></a>步驟 3b︰調查分散式資料庫的資料移動
使用要求識別碼和步驟索引，從 [sys.dm_pdw_dms_workers][sys.dm_pdw_dms_workers] 擷取在每個散發上執行之資料移動步驟的相關資訊。

```sql
-- Find the information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID####' AND step_index = 2;
```

* 檢查 *total_elapsed_time* 資料行，查看是否有特定散發，在資料移動上比其他散發用了更多時間。
* 如果是長時間執行的散發，請檢查 *rows_processed* 資料行，查看從該散發移動的資料列數是否遠多過其他散發。 若是如此，這個結果可能表示基礎資料的扭曲。

如果查詢正在執行，可以使用 [DBCC PDW_SHOWEXECUTIONPLAN][DBCC PDW_SHOWEXECUTIONPLAN]，針對特定散發內目前正在執行的 SQL 步驟，從 SQL Server 計畫快取擷取 SQL Server 預估計畫。

```sql
-- Find the SQL Server estimated plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);
```

<a name="waiting"></a>

## <a name="monitor-waiting-queries"></a>監視等候中的查詢
如果您發現您的查詢因為正在等候資源而沒有進度，以下查詢可顯示查詢正在等候的所有資源。

```sql
-- Find queries 
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,
      waits.state,
      waits.object_type,
      waits.object_name
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID####'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

如果查詢正在主動等候另一個查詢的資源，則狀態會是 **AcquireResources**。  如果查詢具有全部的所需資源，則狀態會是 **Granted**。

## <a name="monitor-tempdb"></a>監視 tempdb
高 tempdb 使用量是效能緩慢及記憶體不足問題的根本原因。 如果您在查詢執行期間發現 tempdb 達到其上限，請考慮調整您的資料倉儲。 以下資訊描述如何識別每個節點上每個查詢的 tempdb 使用量。 

建立下列檢視，以針對 sys.dm_pdw_sql_requests 建立適當節點識別碼的關聯。 擁有節點識別碼可讓您使用其他傳遞 DMV，並將這些資料表與 sys.dm_pdw_sql_requests 聯結。

```sql
-- sys.dm_pdw_sql_requests with the correct node id
CREATE VIEW sql_requests AS
(SELECT
       sr.request_id,
       sr.step_index,
       (CASE 
              WHEN (sr.distribution_id = -1 ) THEN 
              (SELECT pdw_node_id FROM sys.dm_pdw_nodes WHERE type = 'CONTROL') 
              ELSE d.pdw_node_id END) AS pdw_node_id,
       sr.distribution_id,
       sr.status,
       sr.error_id,
       sr.start_time,
       sr.end_time,
       sr.total_elapsed_time,
       sr.row_count,
       sr.spid,
       sr.command
FROM sys.pdw_distributions AS d
RIGHT JOIN sys.dm_pdw_sql_requests AS sr ON d.distribution_id = sr.distribution_id)
```
若要監視 tempdb，請執行下列查詢：

```sql
-- Monitor tempdb
SELECT
    sr.request_id,
    ssu.session_id,
    ssu.pdw_node_id,
    sr.command,
    sr.total_elapsed_time,
    es.login_name AS 'LoginName',
    DB_NAME(ssu.database_id) AS 'DatabaseName',
    (es.memory_usage * 8) AS 'MemoryUsage (in KB)',
    (ssu.user_objects_alloc_page_count * 8) AS 'Space Allocated For User Objects (in KB)',
    (ssu.user_objects_dealloc_page_count * 8) AS 'Space Deallocated For User Objects (in KB)',
    (ssu.internal_objects_alloc_page_count * 8) AS 'Space Allocated For Internal Objects (in KB)',
    (ssu.internal_objects_dealloc_page_count * 8) AS 'Space Deallocated For Internal Objects (in KB)',
    CASE es.is_user_process
    WHEN 1 THEN 'User Session'
    WHEN 0 THEN 'System Session'
    END AS 'SessionType',
    es.row_count AS 'RowCount'
FROM sys.dm_pdw_nodes_db_session_space_usage AS ssu
    INNER JOIN sys.dm_pdw_nodes_exec_sessions AS es ON ssu.session_id = es.session_id AND ssu.pdw_node_id = es.pdw_node_id
    INNER JOIN sys.dm_pdw_nodes_exec_connections AS er ON ssu.session_id = er.session_id AND ssu.pdw_node_id = er.pdw_node_id
    INNER JOIN sql_requests AS sr ON ssu.session_id = sr.spid AND ssu.pdw_node_id = sr.pdw_node_id
WHERE DB_NAME(ssu.database_id) = 'tempdb'
    AND es.session_id <> @@SPID
    AND es.login_name <> 'sa' 
ORDER BY sr.request_id;
```
## <a name="monitor-memory"></a>監視記憶體

記憶體是效能緩慢及記憶體不足問題的根本原因。 如果您在查詢執行期間發現 SQL Server 記憶體使用量達到其上限，請考慮調整您的資料倉儲。

下列查詢會傳回每個節點的 SQL Server 記憶體使用量和記憶體不足壓力：   
```sql
-- Memory consumption
SELECT
  pc1.cntr_value as Curr_Mem_KB, 
  pc1.cntr_value/1024.0 as Curr_Mem_MB,
  (pc1.cntr_value/1048576.0) as Curr_Mem_GB,
  pc2.cntr_value as Max_Mem_KB,
  pc2.cntr_value/1024.0 as Max_Mem_MB,
  (pc2.cntr_value/1048576.0) as Max_Mem_GB,
  pc1.cntr_value * 100.0/pc2.cntr_value AS Memory_Utilization_Percentage,
  pc1.pdw_node_id
FROM
-- pc1: current memory
sys.dm_pdw_nodes_os_performance_counters AS pc1
-- pc2: total memory allowed for this SQL instance
JOIN sys.dm_pdw_nodes_os_performance_counters AS pc2 
ON pc1.object_name = pc2.object_name AND pc1.pdw_node_id = pc2.pdw_node_id
WHERE
pc1.counter_name = 'Total Server Memory (KB)'
AND pc2.counter_name = 'Target Server Memory (KB)'
```
## <a name="monitor-transaction-log-size"></a>監視交易記錄大小
下列查詢會傳回每個發佈上的交易記錄大小。 如果其中一個記錄檔達到 160 GB，您應該考慮將您的執行個體相應放大或限制您交易的大小。 
```sql
-- Transaction log size
SELECT
  instance_name as distribution_db,
  cntr_value*1.0/1048576 as log_file_size_used_GB,
  pdw_node_id 
FROM sys.dm_pdw_nodes_os_performance_counters 
WHERE 
instance_name like 'Distribution_%' 
AND counter_name = 'Log File(s) Used Size (KB)'
```
## <a name="monitor-transaction-log-rollback"></a>監視交易記錄復原
如果您的查詢失敗或需要長時間才能繼續，您可以檢查及監視是否有任何交易復原。
```sql
-- Monitor rollback
SELECT 
    SUM(CASE WHEN t.database_transaction_next_undo_lsn IS NOT NULL THEN 1 ELSE 0 END),
    t.pdw_node_id,
    nod.[type]
FROM sys.dm_pdw_nodes_tran_database_transactions t
JOIN sys.dm_pdw_nodes nod ON t.pdw_node_id = nod.pdw_node_id
GROUP BY t.pdw_node_id, nod.[type]
```

## <a name="next-steps"></a>後續步驟
如需 DMV 的詳細資訊，請參閱[系統檢視][System views]。


<!--Image references-->

<!--Article references-->
[SQL Data Warehouse best practices]: ./sql-data-warehouse-best-practices.md
[System views]: ./sql-data-warehouse-reference-tsql-system-views.md
[Table distribution]: ./sql-data-warehouse-tables-distribute.md
[Investigating queries waiting for resources]: ./sql-data-warehouse-manage-monitor.md#waiting

<!--MSDN references-->
[sys.dm_pdw_dms_workers]: http://msdn.microsoft.com/library/mt203878.aspx
[sys.dm_pdw_exec_requests]: http://msdn.microsoft.com/library/mt203887.aspx
[sys.dm_pdw_exec_sessions]: http://msdn.microsoft.com/library/mt203883.aspx
[sys.dm_pdw_request_steps]: http://msdn.microsoft.com/library/mt203913.aspx
[sys.dm_pdw_sql_requests]: http://msdn.microsoft.com/library/mt203889.aspx
[DBCC PDW_SHOWEXECUTIONPLAN]: http://msdn.microsoft.com/library/mt204017.aspx
[DBCC PDW_SHOWSPACEUSED]: http://msdn.microsoft.com/library/mt204028.aspx
[LABEL]: https://msdn.microsoft.com/library/ms190322.aspx
