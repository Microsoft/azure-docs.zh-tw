---
title: 分析工作負載 - Azure SQL 資料倉儲 | Microsoft Docs
description: 在 Azure SQL 資料倉儲中用來分析工作負載查詢優先順序的技術。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 4ce84e9714b580bcc243285dc1da5ae24a27e8e5
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248088"
---
# <a name="analyze-your-workload-in-azure-sql-data-warehouse"></a>分析 Azure SQL 資料倉儲的工作負載 | Microsoft Docs
在 Azure SQL 資料倉儲中用來分析工作負載查詢優先順序的技術。

## <a name="workload-groups"></a>工作負載群組 
SQL 資料倉儲使用工作負載群組來實作資源類別。 共有八個工作負載群組，它們可跨各種 DWU 大小控制資源類別的行為。 對於任何 DWU，SQL 資料倉儲只會使用這八個工作負載群組的其中四個。 此方法很合理，因為每個工作負載群組都會指派給四個資源類別之一：smallrc、mediumrc、largerc 或 xlargerc。 了解這些工作負載群組的重要性在於其中某些工作負載群組會設定為較高的「重要性」 。 重要性可用於 CPU 排程。 以高重要性執行的查詢會取得比中度重要性的查詢高三倍的 CPU 週期。 因此，並行存取插槽對應也會判斷 CPU 優先順序。 當查詢取用 16 個以上的插槽時，就會以高重要性執行。

下表是每個工作負載群組的重要性對應。

### <a name="workload-group-mappings-to-concurrency-slots-and-importance"></a>工作負載群組與並行存取插槽數目和重要性的對應

| 工作負載群組 | 並行存取插槽對應 | MB / 散發 (彈性) | MB / 散發 (計算) | 重要性對應 |
|:---------------:|:------------------------:|:------------------------------:|:---------------------------:|:------------------:|
| SloDWGroupC00   | 1                        |    100                         | 250                         | 中             |
| SloDWGroupC01   | 2                        |    200                         | 500                         | 中             |
| SloDWGroupC02   | 4                        |    400                         | 1000                        | 中             |
| SloDWGroupC03   | 8                        |    800                         | 2000                        | 中             |
| SloDWGroupC04   | 16                       |  1,600                         | 4000                        | 高               |
| SloDWGroupC05   | 32                       |  3,200                         | 8000                        | 高               |
| SloDWGroupC06   | 64                       |  6,400                         | 16,000                      | 高               |
| SloDWGroupC07   | 128                      | 12,800                         | 32,000                      | 高               |
| SloDWGroupC08   | 256                      | 25,600                         | 64,000                      | 高               |

<!-- where are the allocation and consumption of concurrency slots charts? -->**並行存取插槽的配置和耗用量**圖表中，顯示 DW500 針對 smallrc、mediumrc、largerc 和 xlargerc，分別使用了 1、4、8 或 16 個並行存取插槽。 若要了解每個資源類別的重要性，您可以在上述圖表中查詢這些值。

### <a name="dw500-mapping-of-resource-classes-to-importance"></a>重要性與資源類別的 DW500 對應
| 資源類別 | 工作負載群組 | 已使用的並行存取插槽 | MB / 散發套件 | 重要性 |
|:-------------- |:-------------- |:----------------------:|:-----------------:|:---------- |
| smallrc        | SloDWGroupC00  | 1                      | 100               | 中     |
| mediumrc       | SloDWGroupC02  | 4                      | 400               | 中     |
| largerc        | SloDWGroupC03  | 8                      | 800               | 中     |
| xlargerc       | SloDWGroupC04  | 16                     | 1,600             | 高       |
| staticrc10     | SloDWGroupC00  | 1                      | 100               | 中     |
| staticrc20     | SloDWGroupC01  | 2                      | 200               | 中     |
| staticrc30     | SloDWGroupC02  | 4                      | 400               | 中     |
| staticrc40     | SloDWGroupC03  | 8                      | 800               | 中     |
| staticrc50     | SloDWGroupC03  | 16                     | 1,600             | 高       |
| staticrc60     | SloDWGroupC03  | 16                     | 1,600             | 高       |
| staticrc70     | SloDWGroupC03  | 16                     | 1,600             | 高       |
| staticrc80     | SloDWGroupC03  | 16                     | 1,600             | 高       |

## <a name="view-workload-groups"></a>檢視工作負載群組
下列查詢會顯示從資源管理員的觀點顯示記憶體資源配置的詳細資料。 這有助於在進行疑難排解時，分析工作負載群組的作用中和歷史使用量。

```sql
WITH rg
AS
(   SELECT  
     pn.name                                AS node_name
    ,pn.[type]                              AS node_type
    ,pn.pdw_node_id                         AS node_id
    ,rp.name                                AS pool_name
    ,rp.max_memory_kb*1.0/1024              AS pool_max_mem_MB
    ,wg.name                                AS group_name
    ,wg.importance                          AS group_importance
    ,wg.request_max_memory_grant_percent    AS group_request_max_memory_grant_pcnt
    ,wg.max_dop                             AS group_max_dop
    ,wg.effective_max_dop                   AS group_effective_max_dop
    ,wg.total_request_count                 AS group_total_request_count
    ,wg.total_queued_request_count          AS group_total_queued_request_count
    ,wg.active_request_count                AS group_active_request_count
    ,wg.queued_request_count                AS group_queued_request_count
    FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
    JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp    
            ON  wg.pdw_node_id  = rp.pdw_node_id
            AND wg.pool_id      = rp.pool_id
    JOIN    sys.dm_pdw_nodes pn
            ON    wg.pdw_node_id    = pn.pdw_node_id
    WHERE   wg.name like 'SloDWGroup%'
    AND     rp.name    = 'SloDWPool'
)
SELECT  pool_name
,       pool_max_mem_MB
,       group_name
,       group_importance
,       (pool_max_mem_MB/100)*group_request_max_memory_grant_pcnt AS max_memory_grant_MB
,       node_name
,       node_type
,       group_total_request_count
,       group_total_queued_request_count
,       group_active_request_count
,       group_queued_request_count
FROM    rg
ORDER BY
        node_name
,       group_request_max_memory_grant_pcnt
,       group_importance
;
```

## <a name="queued-query-detection-and-other-dmvs"></a>已排入佇列的查詢偵測和其他 DMV
您可以使用 `sys.dm_pdw_exec_requests` DMV，來識別正在並行存取佇列中等候的查詢。 正在等待並行存取插槽的查詢狀態為**暫止**。

```sql
SELECT  r.[request_id]                           AS Request_ID
,       r.[status]                               AS Request_Status
,       r.[submit_time]                          AS Request_SubmitTime
,       r.[start_time]                           AS Request_StartTime
,       DATEDIFF(ms,[submit_time],[start_time])  AS Request_InitiateDuration_ms
,       r.resource_class                         AS Request_resource_class
FROM    sys.dm_pdw_exec_requests r
;
```

工作負載管理角色可以使用 `sys.database_principals`來檢視。

```sql
SELECT  ro.[name]           AS [db_role_name]
FROM    sys.database_principals ro
WHERE   ro.[type_desc]      = 'DATABASE_ROLE'
AND     ro.[is_fixed_role]  = 0
;
```

下列查詢會顯示每位使用者指派給哪些角色。

```sql
SELECT  r.name AS role_principal_name
,       m.name AS member_principal_name
FROM    sys.database_role_members rm
JOIN    sys.database_principals AS r            ON rm.role_principal_id      = r.principal_id
JOIN    sys.database_principals AS m            ON rm.member_principal_id    = m.principal_id
WHERE   r.name IN ('mediumrc','largerc','xlargerc')
;
```

SQL 資料倉儲具有下列等候類型：

* **LocalQueriesConcurrencyResourceType**：位於並行存取插槽架構外部的查詢。 DMV 查詢及 `SELECT @@VERSION` 這類的系統函數是本機查詢的範例。
* **UserConcurrencyResourceType**：位於並行存取插槽架構內部的查詢。 針對使用者資料表的查詢代表會使用此資源類型的範例。
* **DmsConcurrencyResourceType**：資料移動作業所產生的等候。
* **BackupConcurrencyResourceType**：此等候指出正在備份資料庫。 此資源類型的最大值為 1。 如果在同一時間要求多個備份，其他備份會排入佇列。

`sys.dm_pdw_waits` DMV 可用來查看要求正在等待哪些資源。

```sql
SELECT  w.[wait_id]
,       w.[session_id]
,       w.[type]                                           AS Wait_type
,       w.[object_type]
,       w.[object_name]
,       w.[request_id]
,       w.[request_time]
,       w.[acquire_time]
,       w.[state]
,       w.[priority]
,       SESSION_ID()                                       AS Current_session
,       s.[status]                                         AS Session_status
,       s.[login_name]
,       s.[query_count]
,       s.[client_id]
,       s.[sql_spid]
,       r.[command]                                        AS Request_command
,       r.[label]
,       r.[status]                                         AS Request_status
,       r.[submit_time]
,       r.[start_time]
,       r.[end_compile_time]
,       r.[end_time]
,       DATEDIFF(ms,r.[submit_time],r.[start_time])        AS Request_queue_time_ms
,       DATEDIFF(ms,r.[start_time],r.[end_compile_time])   AS Request_compile_time_ms
,       DATEDIFF(ms,r.[end_compile_time],r.[end_time])     AS Request_execution_time_ms
,       r.[total_elapsed_time]
FROM    sys.dm_pdw_waits w
JOIN    sys.dm_pdw_exec_sessions s  ON w.[session_id] = s.[session_id]
JOIN    sys.dm_pdw_exec_requests r  ON w.[request_id] = r.[request_id]
WHERE    w.[session_id] <> SESSION_ID()
;
```

`sys.dm_pdw_resource_waits` DMV 只會顯示等待指定查詢取用的資源。 資源等候時間只會測量等候提供資源的時間，與訊號等候時間相反，後者是基礎 SQL Server 將查詢排程到 CPU 所需的時間。

```sql
SELECT  [session_id]
,       [type]
,       [object_type]
,       [object_name]
,       [request_id]
,       [request_time]
,       [acquire_time]
,       DATEDIFF(ms,[request_time],[acquire_time])  AS acquire_duration_ms
,       [concurrency_slots_used]                    AS concurrency_slots_reserved
,       [resource_class]
,       [wait_id]                                   AS queue_position
FROM    sys.dm_pdw_resource_waits
WHERE    [session_id] <> SESSION_ID()
;
```
您也可以使用 `sys.dm_pdw_resource_waits` DMV 來計算已授與的並行位置數量。

```sql
SELECT  SUM([concurrency_slots_used]) as total_granted_slots 
FROM    sys.[dm_pdw_resource_waits] 
WHERE   [state]           = 'Granted' 
AND     [resource_class] is not null
AND     [session_id]     <> session_id()
;
```

`sys.dm_pdw_wait_stats` DMV 可用於針對等候項目進行歷史趨勢分析。

```sql
SELECT   w.[pdw_node_id]
,        w.[wait_name]
,        w.[max_wait_time]
,        w.[request_count]
,        w.[signal_time]
,        w.[completed_count]
,        w.[wait_time]
FROM    sys.dm_pdw_wait_stats w
;
```

## <a name="next-steps"></a>後續步驟
如需管理資料庫使用者和安全性的詳細資訊，請參閱[保護 SQL 資料倉儲中的資料庫](sql-data-warehouse-overview-manage-security.md)。 如需較大資源類別如何改善叢集資料行存放區索引品質的詳細資訊，請參閱 [重建索引以提升區段品質](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality)。


