---
title: 監視工作負載-Azure 入口網站
description: 使用 Azure 入口網站監視 Synapse SQL
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.openlocfilehash: 3334c5ca300b6dbeb0ad055c7dbd2c15e1c5481a
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98677116"
---
# <a name="monitor-workload---azure-portal"></a>監視工作負載-Azure 入口網站

本文說明如何使用 Azure 入口網站來監視您的工作負載。 這包括使用 log analytics for [SYNAPSE SQL](https://azure.microsoft.com/blog/workload-insights-with-sql-data-warehouse-delivered-through-azure-monitor-diagnostic-logs-pass/)來設定 Azure 監視器記錄，以調查查詢執行和工作負載趨勢。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
- SQL 集區：我們將會收集 SQL 集區的記錄檔。 如果您尚未布建 SQL 集區，請參閱 [建立 sql 集](./load-data-from-azure-blob-storage-using-copy.md)區中的指示。

## <a name="create-a-log-analytics-workspace"></a>建立 Log Analytics 工作區

流覽至 Log Analytics 工作區的流覽分頁，並建立工作區

![Log Analytics 工作區](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspaces.png)

![顯示 Log Analytics 工作區的螢幕擷取畫面，您可以在其中選取 [新增]。](./media/sql-data-warehouse-monitor-workload-portal/add_analytics_workspace.png)

![顯示 Log Analytics 工作區的螢幕擷取畫面，您可以在其中輸入值。](./media/sql-data-warehouse-monitor-workload-portal/add_analytics_workspace_2.png)

如需工作區的詳細資訊，請參閱下列 [檔](../../azure-monitor/learn/quick-create-workspace.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.jsond#create-a-workspace)。

## <a name="turn-on-resource-logs"></a>開啟資源記錄

設定診斷設定，以從您的 SQL 集區發出記錄。 記錄包含相當於最常使用的效能疑難排解 Dmv 的遙測資料檢視。 目前支援下列視圖：

- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

![啟用資源記錄](./media/sql-data-warehouse-monitor-workload-portal/enable_diagnostic_logs.png)

您可以將記錄發出至 Azure 儲存體、串流分析或 Log Analytics。 在本教學課程中，請選取 Log Analytics。

![指定記錄檔](./media/sql-data-warehouse-monitor-workload-portal/specify_logs.png)

## <a name="run-queries-against-log-analytics"></a>對 Log Analytics 執行查詢

流覽至您的 Log Analytics 工作區，您可以在其中執行下列作業：

- 使用記錄查詢來分析記錄，並儲存查詢以重複使用
- 儲存查詢以重複使用
- 建立記錄警示
- 將查詢結果釘選到儀表板

如需記錄查詢功能的詳細資訊，請參閱下列 [檔](/azure/data-explorer/kusto/query/?bc=%2fazure%2fsynapse-analytics%2fsql-data-warehouse%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2fsql-data-warehouse%2ftoc.json)。

![Log Analytics 工作區編輯器](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspace_editor.png)

![Log Analytics 工作區查詢](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspace_queries.png)

## <a name="sample-log-queries"></a>範例記錄查詢

```Kusto
//List all queries
AzureDiagnostics
| where Category contains "ExecRequests"
| project TimeGenerated, StartTime_t, EndTime_t, Status_s, Command_s, ResourceClass_s, duration=datetime_diff('millisecond',EndTime_t, StartTime_t)
```

```Kusto
//Chart the most active resource classes
AzureDiagnostics
| where Category contains "ExecRequests"
| where Status_s == "Completed"
| summarize totalQueries = dcount(RequestId_s) by ResourceClass_s
| render barchart
```

```Kusto
//Count of all queued queries
AzureDiagnostics
| where Category contains "waits"
| where Type_s == "UserConcurrencyResourceType"
| summarize totalQueuedQueries = dcount(RequestId_s)
```

## <a name="next-steps"></a>下一步

現在您已設定並設定 Azure 監視器記錄，請 [自訂 azure 儀表板](../../azure-portal/azure-portal-dashboards.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) ，以在您的小組間共用。