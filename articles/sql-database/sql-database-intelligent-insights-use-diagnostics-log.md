---
title: Intelligent Insights 效能診斷記錄 - Azure SQL Database | Microsoft Docs
description: Intelligent Insights 會提供 Azure SQL Database 效能問題的診斷記錄
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: v-daljep
ms.reviewer: carlrab
manager: craigg
ms.date: 04/04/2018
ms.openlocfilehash: 70096c8f3a5c07fa757b68494c04519b63435dcd
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166889"
---
# <a name="use-the-intelligent-insights-azure-sql-database-performance-diagnostics-log"></a>使用 Intelligent Insights Azure SQL Database 效能診斷記錄

此頁面提供有關如何使用 [Intelligent Insights](sql-database-intelligent-insights.md) 所產生 Azure SQL Database 效能診斷記錄的使用方式、其格式及其所含資料的資訊，來因應您的自訂開發需求。 您可以將此診斷記錄傳送給 [Azure Log Analytics](../log-analytics/log-analytics-azure-sql.md)、[Azure 事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md)、[Azure 儲存體](sql-database-metrics-diag-logging.md#stream-into-storage)或協力廠商解決方案，以取得自訂的 DevOps 警示和報告功能。

## <a name="log-header"></a>記錄標頭

診斷記錄使用 JSON 標準格式來輸出 Intelligent insights 結果。 用於存取 Intelligent Insights 記錄的確切類別屬性是一個固定值 "SQLInsights"。

記錄的標頭是通用的，而且包含在建立項目時顯示的時間戳記 (TimeGenerated)。 它也會包含資源識別碼 (ResourceId)，該識別碼參考與項目與相關的特定 SQL Database。 類別 (Category)、 層級 (Level) 和作業名稱 (OperationName) 是值不會變更的固定屬性。 它們表示記錄項目為參考資訊，來自 Intelligent Insights (SQLInsights)。

```json
"TimeGenerated" : "2017-9-25 11:00:00", // time stamp of the log entry
"ResourceId" : "database identifier", // value points to a database resource
"Category": "SQLInsights", // fixed property
"Level" : "Informational", // fixed property
"OperationName" : "Insight", // fixed property
```

## <a name="issue-id-and-database-affected"></a>問題識別碼和受影響的資料庫

問題識別屬性 (issueId_d) 提供一種以獨特方式追蹤效能問題直到解決為止的方法。 Intelligent Insights 會將每個問題生命週期觀察為：「作用中」、「正在驗證」或「完成」。 透過這每一個狀態階段，Intelligent insights 便可在記錄中記錄多個事件記錄。 這每一個項目的問題識別碼編號都會維持唯一。 Intelligent Insights 會在問題整個生命週期全程追蹤問題，並且每隔 15 分鐘就在診斷記錄中產生一次深入解析。

在偵測到效能問題之後，只要其持續存在，系統就會在狀態 (status_s) 屬性底下將該問題回報為「作用中」。 在偵測到問題已緩和之後，系統就會驗證問題並在狀態 (status_s) 屬性底下回報為「正在驗證」。 如果問題已不存在，狀態 (status_s) 屬性就會將此問題回報為「完成」。

除了「問題識別碼」之外，診斷記錄還會回報與診斷記錄中所回報問題相關之特定事件的開始 (intervalStartTime_t) 和結束 (intervalEndTme_t) 時間戳記。

彈性集區 (elasticPoolName_s) 屬性會指出發生問題之資料庫所屬的彈性集區。 如果資料庫不屬於彈性集區，此屬性就不會有值。 資料庫名稱 (databaseName_s) 屬性中會顯示所偵測到有問題的資料庫。

```json
"intervalStartTime_t": "2017-9-25 11:00", // start of the issue reported time stamp
"intervalEndTme_t":"2017-9-25 12:00", // end of the issue reported time stamp
"elasticPoolName_s" : "", // resource elastic pool (if applicable) 
"databaseName_s" : "db_name",  // database name
"issueId_d" : 1525, // unique ID of the issue detected
"status_s" : "Active" // status of the issue – possible values: "Active", "Verifying", and "Complete"
```

## <a name="detected-issues"></a>偵測到的問題

Intelligent Insights 效能記錄的下一個區段包含透過內建的人工智慧偵測到的效能問題。 JSON 診斷記錄內的屬性會顯示偵測。 這些偵測包括問題類別、問題的影響、受影響的查詢和計量。 偵測屬性可能包含多個偵測到的效能問題。

偵測到的效能問題會藉由下列偵測屬性結構回報：

```json
"detections_s" : [{
"impact" : 1 to 3, // impact of the issue detected, possible values 1-3 (1 low, 2 moderate, 3 high impact)
"category" : "Detectable performance pattern", // performance issue detected, see the table
"details": <Details outputted> // details of an issue (see the table)
}] 
```

下表提供可偵測的效能模式，以及輸出到診斷記錄中的詳細資料。

### <a name="detection-category"></a>偵測類別

類別 (category) 屬性會說明可偵測之效能模式的類別。 如需可偵測之效能模式的所有可能類別，請參閱下表。 如需詳細資訊，請參閱[使用 Intelligent Insights 針對資料庫效能問題進行疑難排解](sql-database-intelligent-insights-troubleshoot-performance.md)。

依據偵測到的效能問題之不同，診斷記錄檔中輸出的詳細資料也會有所不同。

| 可偵測的效能模式 | 輸出的詳細資料 |
| :------------------- | ------------------- |
| 達到資源限制 | <li>受影響的資源</li><li>查詢雜湊</li><li>資源耗用量百分比</li> |
| 工作負載增加 | <li>執行時間增加的查詢數目</li><li>對工作負載增加影響最大之查詢的查詢雜湊</li> |
| 記憶體壓力 | <li>記憶體 Clerk</li> |
| 鎖定 | <li>受影響的查詢雜湊</li><li>封鎖查詢雜湊</li> |
| MAXDOP 增加 | <li>查詢雜湊</li><li>CXP 等候時間</li><li>等候時間</li> |
| 頁面閂鎖爭用 | <li>造成爭用之查詢的查詢雜湊</li> |
| 遺漏索引 | <li>查詢雜湊</li> |
| 新查詢 | <li>新查詢的查詢雜湊</li> |
| 不尋常的等候統計資料 | <li>不尋常的等候類型</li><li>查詢雜湊</li><li>查詢等候時間</li> |
| TempDB 爭用 | <li>造成爭用之查詢的查詢雜湊</li><li>整體資料庫頁面閂鎖爭用等候時間的查詢歸屬 [%]</li> |
| 彈性集區 DTU 不足 | <li>彈性集區</li><li>DTU 取用量最高的資料庫</li><li>取用量最高之取用者所使用的不良 DTU 百分比</li> |
| 計畫迴歸 | <li>查詢雜湊</li><li>良好計畫識別碼</li><li>不良計畫識別碼</li> |
| 資料庫範圍設定值變更 | <li>與預設值相比的資料庫範圍組態變更</li> |
| 用戶端執行速度太慢 | <li>查詢雜湊</li><li>等候時間</li> |
| 定價層降級 | <li>文字通知</li> |

### <a name="impact"></a>影響

影響 (impact) 屬性會說明偵測行為會對資料庫具有的問題造成多少影響。 影響範圍從 1 到 3，3 具有最高的比重、2 為中等，1 的比重最低。 視您的特定需求而定，可以使用影響值來作為自訂警示自動化的輸入。 受影響的屬性查詢 (QueryHashes) 會提供受特定偵測影響的查詢雜湊清單。

### <a name="impacted-queries"></a>受影響的查詢

Intelligent Insights 記錄的下一個區段提供受所偵測到之效能問題影響的特定查詢相關資訊。 此資訊會以內嵌在 impact_s 屬性中的物件陣列形式顯示。 Impact 屬性是由實體和計量所組成。 實體是指特定的查詢 (類型：查詢)。 唯一的查詢雜湊會在值 (Value) 屬性底下顯示。 此外，所顯示的每個查詢後面都會接著一個計量和值，用來指出所偵測到的效能問題。

在下列記錄範例中，使用雜湊 0x9102EXZ4 的查詢會被偵測到有增加的執行持續時間 (計量：DurationIncreaseSeconds)。 110 秒的值指出此特定查詢會再花費 110 秒的時間執行。 因為可以偵測到多個查詢，因此這個特定記錄區段可以包含多個查詢項目。

```json
"impact" : [{
"entity" : { 
"Type" : "Query", // type of entity - query
"Value" : "query hash value", // for example "0x9102EXZ4" query hash value },
"Metric" : "DurationIncreaseSeconds", // measured metric and the measurement unit (in this case seconds)
"Value" : 110 // value of the measured metric (in this case seconds)
}]
```

### <a name="metrics"></a>度量

計量 (metric) 屬性底下會提供所回報之每個計量的度量單位，可能的值為：秒、數字及百分比。 值 (value) 屬性中會回報所測量計量的值。

DurationIncreaseSeconds 屬性提供以秒為單位的度量單位。 CriticalErrorCount 度量單位是數字，代表錯誤計數。

```json
"metric" : "DurationIncreaseSeconds", // issue metric type – possible values: DurationIncreaseSeconds, CriticalErrorCount, WaitingSeconds
"value" : 102 // value of the measured metric (in this case seconds)
```

## <a name="root-cause-analysis-and-improvement-recommendations"></a>根本原因分析與改進建議

Intelligent Insights 效能記錄的最後部分是關於所識別之效能降低問題的自動化根本原因分析。 此資訊會透過根本原因分析 (rootCauseAnalysis_s) 屬性以人類易讀的用語顯示。 改進的建議包含在可能的記錄中。

```json
// example of reported root cause analysis of the detected performance issue, in a human-readable format

"rootCauseAnalysis_s" : "High data IO caused performance to degrade. It seems that this database is missing some indexes that could help."
```

您可以搭配 [Azure Log Analytics]( https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-sql) 或協力廠商解決方案來使用 Intelligent Insights 效能記錄，以提供自訂的 DevOps 警示和報告功能。

## <a name="next-steps"></a>後續步驟
- 了解 [Intelligent Insights](sql-database-intelligent-insights.md) 概念。
- 了解如何[使用 Intelligent Insights 針對 Azure SQL Database 效能問題進行疑難排解](sql-database-intelligent-insights-troubleshoot-performance.md)。
- 了解如何[使用 Azure SQL 分析來監視 Azure SQL Database](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-sql)。
- 了解如何[收集並取用來自 Azure 資源的記錄資料](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)。



