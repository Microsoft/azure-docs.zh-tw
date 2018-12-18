---
title: Log Analytics 中的 Azure SQL Analytics 解決方案 | Microsoft Docs
description: Azure SQL 分析解決方案可協助您管理 Azure SQL 資料庫
services: log-analytics
documentationcenter: ''
author: danimir
manager: carmonm
ms.reviewer: carlrab
ms.assetid: b2712749-1ded-40c4-b211-abc51cc65171
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/03/2018
ms.author: v-daljep
ms.component: na
ms.openlocfilehash: b7a7e2787128c74cd7d016c01b751d15628fb4b2
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181986"
---
# <a name="monitor-azure-sql-database-using-azure-sql-analytics-preview"></a>使用 Azure SQL 分析來監視 Azure SQL Database (預覽)

![Azure SQL 分析符號](./media/log-analytics-azure-sql/azure-sql-symbol.png)

Azure SQL 分析是一個雲端監視解決方案，可以跨多個訂用帳戶大規模監視 Azure SQL Database、彈性集區和受控執行個體的效能。 它會收集重要的 Azure SQL Database 效能計量，並且以視覺效果方式呈現，具有內建智慧可以執行效能疑難排解。

藉由使用您以解決方案收集的計量，您可以建立自訂的監視規則和警示。 解決方案可協助您找出應用程式堆疊中每個層級的問題。 它會使用 Azure 診斷計量與 Log Analytics 檢視，來呈現單一 Log Analytics 工作區中所有 Azure SQL 資料庫、彈性集區和受控執行個體中各個資料庫的相關資料。 Log Analytics 可協助您收集、相互關聯，並以視覺化方式檢視結構化和非結構化資料。

如需使用 Azure SQL Analytics 解決方案，以及一般使用案例的實際操作概觀，請觀看內嵌影片：


> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Get-Intelligent-Insights-for-Improving-Azure-SQL-Database-Performance/player]
>

## <a name="connected-sources"></a>連接的來源

Azure SQL 分析是僅限雲端的監視解決方案，支援 Azure SQL Database、受控執行個體資料庫和彈性集區診斷遙測的串流。

由於該解決方案不使用代理程式連線至 Log Analytics 服務，因此不支援對 SQL Server 裝載的內部部署環境或在 VM 中進行監視，請參閱下方的相容性資料表。

| 連接的來源 | 支援 | 說明 |
| --- | --- | --- |
| **[Azure 診斷](log-analytics-azure-storage.md)** | **是** | Azure 會將 Azure 計量與記錄資料直接傳送至 Log Analytics。 |
| [Azure 儲存體帳戶](log-analytics-azure-storage.md) | 否 | Log Analytics 不會從儲存體帳戶讀取資料。 |
| [Windows 代理程式](log-analytics-windows-agent.md) | 否 | 解決方案不使用直接 Windows 代理程式。 |
| [Linux 代理程式](log-analytics-linux-agents.md) | 否 | 解決方案不使用直接 Linux 代理程式。 |
| [SCOM 管理群組](log-analytics-om-agents.md) | 否 | 解決方案不使用從 SCOM 代理程式直接連線到 Log Analytics。 |

## <a name="configuration"></a>組態

執行下列步驟將 Azure SQL 分析解決方案新增至 Azure 儀表板。

1. 從 [Azure 市集](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureSQLAnalyticsOMS?tab=Overview)將 Azure SQL 分析解決方案新增至您的工作區。
2. 在 Azure 入口網站中，按一下 [+ 建立資源]，然後搜尋 **Azure SQL 分析**。  
    ![監視 + 管理](./media/log-analytics-azure-sql/monitoring-management.png)
3. 從清單中選取 [Azure SQL 分析 (預覽)]
4. 在 [Azure SQL 分析 (預覽)] 區域中，按一下 [建立]。  
    ![建立](./media/log-analytics-azure-sql/portal-create.png)
5. 在 [建立新解決方案] 區域中，建立您想要將解決方案新增至其中的新工作區，或選取現有工作區，然後按一下 [建立]。

    ![新增到工作區](./media/log-analytics-azure-sql/add-to-workspace.png)

### <a name="configure-azure-sql-databases-elastic-pools-and-managed-instances-to-stream-diagnostics-telemetry"></a>設定 Azure SQL Database、彈性集區 和受控執行個體以串流診斷遙測

在工作區建立了 Azure SQL 分析解決方案後，為了監視 Azure SQL Database、受控執行個體資料庫和彈性集區的效能，您必須**設定每個**想要監視的資源，以將其診斷遙測串流到解決方案。

- 為 Azure SQL Database、受控執行個體資料庫和彈性集區啟用 Azure 診斷，以[將診斷遙測串流至 Azure SQL 分析](../sql-database/sql-database-metrics-diag-logging.md)。

### <a name="to-configure-multiple-azure-subscriptions"></a>若要設定多個 Azure 訂用帳戶
 
若要支援多個訂用帳戶，從[使用 PowerShell 啟用 Azure 資源計量記錄](https://blogs.technet.microsoft.com/msoms/2017/01/17/enable-azure-resource-metrics-logging-using-powershell/)使用 PowerShell 指令碼。 當執行指令碼以將診斷資料從一個 Azure 訂用帳戶中的資源傳送至另一個 Azure 訂用帳戶中的工作區時，提供工作區資源識別碼做為參數。

**範例**

```
PS C:\> $WSID = "/subscriptions/<subID>/resourcegroups/oms/providers/microsoft.operationalinsights/workspaces/omsws"
```

```
PS C:\> .\Enable-AzureRMDiagnostics.ps1 -WSID $WSID
```

## <a name="using-the-solution"></a>使用解決方案

當您將解決方案新增至您的工作區時，Azure SQL 分析圖格會新增至您的工作區，而且會顯示在 [概觀] 中。 此圖格會顯示解決方案從中接收診斷遙測的 Azure SQL Database、彈性集區、受控執行個體和受控執行個體資料庫各有多少數量。

![Azure SQL 分析圖格](./media/log-analytics-azure-sql/azure-sql-sol-tile.png)

解決方案可提供兩個不同的檢視，一個用於監視 Azure SQL Database 和彈性集區，另一個檢視用於監視受控執行個體和受控執行個體中的資料庫。

若要檢視適用於 Azure SQL Database 和彈性集區的 Azure SQL 分析監視儀表板，請按一下圖格的上半部。 若要檢視適用於受控執行個體和受控執行個體資料庫的 Azure SQL 分析監視儀表板，請按一下圖格的下半部。

### <a name="viewing-azure-sql-analytics-data"></a>檢視 Azure SQL 分析資料

儀表板包含透過不同檢視方塊監視之所有資料庫的概觀。 若要讓不同的檢視方塊運作，您必須在要串流處理至 Azure Log Analytics 工作區的 SQL 資源上，啟用適當的計量或記錄。

請注意，如果某些計量或記錄未串流到 Azure Log Analytics，解決方案中的圖格就不會填入監視資訊。

### <a name="azure-sql-database-and-elastic-pool-view"></a>Azure SQL Database 和彈性集區檢視

選取了適用於 Azure SQL Database 和彈性集區的 [Azure SQL 分析] 圖格後，便會顯示監視儀表板。

![Azure SQL 分析概觀](./media/log-analytics-azure-sql/azure-sql-sol-overview.png)

選取任何磚，以便在特定的檢視方塊中開啟向下鑽研報表。 一旦選取檢視方塊，向下鑽研報表隨即開啟。

![Azure SQL 分析逾時](./media/log-analytics-azure-sql/azure-sql-sol-metrics.png)

此檢視中的每個檢視方塊都會提供訂用帳戶、伺服器、彈性集區和資料庫層級的摘要。 此外，每個檢視方塊都會在右側顯示檢視方塊專屬的報表。 從清單中選取訂用帳戶、伺服器、集區或資料庫可繼續往下鑽研。

### <a name="managed-instance-and-databases-in-managed-instance-view"></a>受控執行個體和受控執行個體中的資料庫檢視

選取了適用於受控執行個體和受控執行個體資料庫的 [Azure SQL 分析] 圖格後，便會顯示監視儀表板。

![Azure SQL 分析概觀](./media/log-analytics-azure-sql/azure-sql-sol-overview-mi.png)

選取任何磚，以便在特定的檢視方塊中開啟向下鑽研報表。 一旦選取檢視方塊，向下鑽研報表隨即開啟。

選取 [受控執行個體] 檢視會顯示受控執行個體使用量的詳細資料、其所包含的資料庫，以及跨執行個體所執行的查詢遙測。

![Azure SQL 分析逾時](./media/log-analytics-azure-sql/azure-sql-sol-metrics-mi.png)

### <a name="perspectives"></a>檢視方塊

下表概述兩個儀表板版本所支援的檢視方塊，一個適用於 Azure SQL Database 和彈性集區，另一個適用於受控執行個體。

| 檢視方塊 | 說明 | SQL Database 和彈性集區支援 | 受控執行個體支援 |
| --- | ------- | ----- | ----- |
| 資源 (依類型) | 可計算所有受監視資源的檢視方塊。 | 是 | 是 | 
| 深入解析 | 可透過階層的方式，向下鑽研至 Intelligent Insights 乃至效能。 | 是 | 是 |
| Errors | 可透過階層的方式，向下鑽研至資料庫上發生的 SQL 錯誤。 | 是 | 是 |
| 逾時 | 可透過階層的方式，向下鑽研至資料庫上發生的 SQL 逾時。 | 是 | 否 |
| 封鎖 | 可透過階層的方式，向下鑽研至資料庫上發生的 SQL 封鎖。 | 是 | 否 |
| 資料庫等候 | 可透過階層的方式，向下鑽研至資料庫層級的 SQL 等候統計資料。 包含總等候時間及每種等候類型等候時間的摘要。 |是 | 是 |
| 查詢持續時間 | 可透過階層的方式，向下鑽研至查詢執行統計資料，例如查詢持續時間、CPU 使用量、資料 IO 使用量、記錄 IO 使用量。 | 是 | 是 |
| 查詢等候 | 可透過階層的方式，依等候類別，向下鑽研至查詢等候統計資料。 | 是 | 是 |

### <a name="intelligent-insights-report"></a>Intelligent Insights 報表

Azure SQL Database [Intelligent Insights](../sql-database/sql-database-intelligent-insights.md) 可讓您知道 Azure SQL Database 和受控執行個體資料庫的效能發生什麼情況。 收集的所有 Intelligent Insights 都可以透過 Insights 檢視方塊視覺化及存取。

![Azure SQL 分析見解](./media/log-analytics-azure-sql/azure-sql-sol-insights.png)

### <a name="elastic-pool-and-database-reports"></a>彈性集區和資料庫報表

彈性集區和 SQL Database 都有自己特定的報表，可顯示在指定的時間，針對資源收集的所有資料。

![Azure SQL 分析資料庫](./media/log-analytics-azure-sql/azure-sql-sol-database.png)

![Azure SQL 彈性集區](./media/log-analytics-azure-sql/azure-sql-sol-pool.png)

### <a name="query-reports"></a>查詢報表

您可以透過查詢持續時間和查詢等候檢視方塊，將任何查詢的效能透過查詢報表相互關聯。 此報表會比較不同資料庫上的查詢效能，並可讓您輕鬆地找出所選查詢執行速度良好與緩慢的資料庫。

![Azure SQL 分析查詢](./media/log-analytics-azure-sql/azure-sql-sol-queries.png)

### <a name="analyze-data-and-create-alerts"></a>分析資料並建立警示

### <a name="creating-alerts-for-azure-sql-database"></a>建立 Azure SQL Database 的警示

您可以使用來自 Azure SQL Database 資源的資料，輕鬆[建立警示](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)。 以下是您可以搭配記錄警示使用的一些實用[記錄搜尋](log-analytics-log-searches.md)查詢：

Azure SQL Database 上的高 CPU

```
AzureMetrics 
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/DATABASES/"
| where MetricName=="cpu_percent" 
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
> - 若要設定此警示，先決條件是受監視的資料庫會將診斷計量 (「所有計量」選項) 串流至解決方案。
> - 請將 MetricName 值 cpu_percent 更換為 dtu_consumption_percent，即可改為取得高 DTU 結果。

Azure SQL Database 彈性集區上的高 CPU

```
AzureMetrics 
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/ELASTICPOOLS/"
| where MetricName=="cpu_percent" 
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
> - 若要設定此警示，先決條件是受監視的資料庫會將診斷計量 (「所有計量」選項) 串流至解決方案。
> - 請將 MetricName 值 cpu_percent 更換為 dtu_consumption_percent，即可改為取得高 DTU 結果。

*過去 1 小時的平均 Azure SQL Database 儲存體高於 95%*

```
let time_range = 1h;
let storage_threshold = 95;
AzureMetrics
| where ResourceId contains "/DATABASES/"
| where MetricName == "storage_percent"
| summarize max_storage = max(Average) by ResourceId, bin(TimeGenerated, time_range)
| where max_storage > storage_threshold
| distinct ResourceId
```

> [!NOTE]
> - 若要設定此警示，先決條件是受監視的資料庫會將診斷計量 (「所有計量」選項) 串流至解決方案。
> - 這項查詢需要將警示規則設定為會在有查詢結果 (> 0 個結果) 時引發警示，這表示某些資料庫上有此情況。 其輸出中會列出所定義 time_range 內高於 storage_threshold 的資料庫資源。
> - 其輸出中會列出所定義 time_range 內高於 storage_threshold 的資料庫資源。

針對 Intelligent insights 的警示

```
let alert_run_interval = 1h;
let insights_string = "hitting its CPU limits";
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active" 
| where TimeGenerated > ago(alert_run_interval)
| where rootCauseAnalysis_s contains insights_string
| distinct ResourceId
```

> [!NOTE]
> - 若要設定此警示，先決條件是受監視的資料庫會將 SQLInsights 診斷記錄串流至解決方案。
> - 這項查詢需要將警示規則設定為利用和 alert_run_interval 相同的頻率來執行，以避免產生重複結果。 請將此規則設定為會在有查詢結果 (> 0 個結果) 時引發警示。
> - 自訂 alert_run_interval 來指定時間範圍，以檢查設定為會將 SQLInsights 記錄串流至解決方案的資料庫上是否發生此情況。
> - 自訂 insights_string 以擷取深入解析根本原因分析文字的輸出。 在可從現有深入解析使用的解決方案中，其 UI 中會顯示此相同文字。 或者，您也可以使用下列查詢來查看訂用帳戶所產生的所有深入解析文字。 請使用查詢的輸出來搜集可供對深入解析設定警示的不同字串。

```
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active" 
| distinct rootCauseAnalysis_s
```

### <a name="creating-alerts-for-managed-instance"></a>建立受控執行個體的警示

*受控執行個體儲存體高於 90%

```
let storage_percentage_treshold = 90;
AzureDiagnostics
| where Category =="ResourceUsageStats"
| summarize (TimeGenerated, calculated_storage_percentage) = arg_max(TimeGenerated, todouble(storage_space_used_mb_s) *100 / todouble (reserved_storage_mb_s))
   by ResourceId
| where calculated_storage_percentage > storage_percentage_treshold
```

> [!NOTE]
> - 若要設定此警示，預先需求是所監視的受控執行個體已對解決方案啟用 ResourceUsageStats 記錄的串流。
> - 這項查詢需要將警示規則設定為會在有查詢結果 (> 0 個結果) 時引發警示，這表示受控執行個體上有此情況。 輸出是受控執行個體上的儲存體耗用量百分比。

## <a name="next-steps"></a>後續步驟

- 使用 Log Analytics 中的[記錄搜尋](log-analytics-log-searches.md)來檢視詳細的 Azure SQL 資料。
- [建立您自己的儀表板](log-analytics-dashboards.md)來顯示 Azure SQL 資料。
- 在特定的 Azure SQL 事件發生時[建立警示](log-analytics-alerts.md)。
