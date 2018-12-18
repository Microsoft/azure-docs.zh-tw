---
title: 使用 Intelligent Insights 監視資料庫使用情況 - Azure SQL Database | Microsoft Docs
description: Azure SQL Database Intelligent Insights 使用內建的智慧功能透過人工智慧持續監視資料庫使用情況，並偵測導致效能不佳的干擾性事件。
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
ms.date: 09/20/2018
ms.openlocfilehash: 6ccb29c9c464c21f09b1e547903a283201c52618
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159614"
---
# <a name="intelligent-insights"></a>Intelligent Insights

Azure SQL Database Intelligent Insights 可讓您知道 SQL Database 和受控執行個體資料庫效能發生了什麼。

Intelligent Insights 使用內建的智慧功能透過人工智慧持續監視資料庫使用情況，並偵測導致效能不佳的干擾性事件。 一旦偵測到，它就會執行詳細的分析來產生含有該問題智慧型評估的診斷記錄。 此評估包含資料庫效能問題的根本原因分析，並在可能的情況下提供效能改進建議。 

## <a name="what-can-intelligent-insights-do-for-you"></a>Intelligent Insights 可以為您做什麼？

Intelligent Insights 是 Azure 內建智慧功能的一項獨特功能，可提供下列價值：

- 主動監視
- 量身打造的效能深入解析
- 提早偵測到資料庫效能降低
- 針對所偵測到問題的根本原因分析
- 效能改進建議
- 以數十萬個資料庫向外延展的功能
- 為 DevOps 資源及擁有權總成本帶來正面影響

## <a name="how-does-intelligent-insights-work"></a>Intelligent Insights 如何運作？

Intelligent Insights 會藉由將最後一小時的資料庫工作負載與過去七天的基準工作負載做比較，來分析資料庫效能。 資料庫工作負載是由被判斷為對資料庫效能最具影響的查詢 (例如重複次數最多和最大型的查詢) 所組成。 由於每個資料庫皆根據其結構、資料、使用方式及應用程式而有所不同，因此所產生的每個工作負載基準對個別執行個體來說都是特定且唯一的。 獨立於工作負載基準之外，Intelligent Insights 也會監視絕對作業臨界值，並偵測有關等候時間過長的問題、重大例外狀況，以及有關可能影響效能之查詢參數化的問題。

在使用人工智慧透過多個受觀察計量偵測到效能降低問題之後，系統便會執行分析。 系統會產生診斷記錄，以及針對您資料庫所發生之情況的智慧型深入解析。 Intelligent Insights 可讓您從資料庫效能問題一開始出現到解決為止，都能輕鬆追蹤情況。 系統會針對每個偵測到的問題追蹤其整個生命週期 (從初始問題偵測、驗證效能改進，到其完成為止)。 系統會每隔 15 分鐘在診斷記錄中提供一次更新。 

![資料庫效能分析工作流程](./media/sql-database-intelligent-insights/intelligent-insights-concept.png)

用來測量和偵測資料庫效能問題的計量，是以查詢持續時間、逾時要求、過長的等候時間及發生錯誤的要求作為基礎。 如需計量的詳細資訊，請參閱本文件的[偵測計量](sql-database-intelligent-insights.md#detection-metrics)一節。

已識別的 SQL Database 效能降低會記錄在診斷記錄中，其中會包含由下列屬性組成的智慧項目：

| 屬性             | 詳細資料              |
| :------------------- | ------------------- |
| 資料庫資訊 | 偵測到有深入解析之資料庫的相關中繼資料，例如資源 URI。 |
| 觀察的時間範圍 | 偵測到之深入解析的期間開始和結束時間。 |
| 受影響的計量 | 導致產生深入解析的計量： <ul><li>查詢持續時間增加 [秒]。</li><li>過長的等候時間 [秒]。</li><li>逾時的要求 [百分比]。</li><li>發生錯誤的要求 [百分比]。</li></ul>|
| 影響值 | 所測量計量的值。 |
| 受影響的查詢和錯誤碼 | 查詢雜湊或錯誤碼。 這些可用來與受影響的查詢輕鬆相互關聯。 系統會提供由查詢持續時間增加、等候時間、逾時計數或錯誤碼所組成的計量。 |
| 偵測 | 發生事件時在資料庫識別出的偵測。 一共有 15 個偵測模式。 如需詳細資訊，請參閱[使用 Intelligent Insights 針對資料庫效能問題進行疑難排解](sql-database-intelligent-insights-troubleshoot-performance.md)。 |
| 根本原因分析 | 所識別之問題的根原因分析，此分析會以人類看得懂的格式顯示。 有些深入解析可能會在可能的情況下包含效能改進建議。 |
|||

如需搭配使用 Intelligent Insights 與 Azure SQL 分析以及一般使用方式情節的實際操作概觀，請觀看內嵌影片：


> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Get-Intelligent-Insights-for-Improving-Azure-SQL-Database-Performance/player]
>

Intelligent Insights 在發現和針對 SQL Database 效能問題進行疑難排解方面表現出色。 若要使用 Intelligent Insights 來針對 SQL Database 和受控執行個體資料庫效能問題進行疑難排解，請參閱[使用 Intelligent Insights 針對 Azure SQL Database 效能問題進行疑難排解](sql-database-intelligent-insights-troubleshoot-performance.md)。

## <a name="configure-intelligent-insights"></a>設定 Intelligent Insights

Intelligent Insights 的輸出是一個智慧效能診斷記錄。 此記錄的使用方式有數種：透過將它串流處理到 Azure SQL 分析、Azure 事件中樞和 Azure 儲存體，或協力廠商產品。 

* 搭配使用產品與 [Azure SQL 分析](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-azure-sql)，以透過 Azure 入口網站使用者介面檢視深入解析。 這是整合式 Azure 解決方案，而且是檢視深入解析的最常見方式。
* 搭配使用產品與 Azure 事件中樞，以開發自訂監視和警示情節
* 搭配使用產品與 Azure 儲存體來開發自訂應用程式，例如自訂報告、長期資料封存等等。

Intelligent Insights 與其他產品 (Azure SQL 分析、Azure 事件中樞、Azure 儲存體或供取用的協力廠商產品) 的整合方式，為先在資料庫的 [診斷設定] 刀鋒視窗中啟用 Intelligent Insights 記錄 ("SQLInsights" 記錄)，然後將 Intelligent Insights 記錄資料設定為串流處理到其中一個產品。

如需如何啟用 Intelligent Insights 記錄，以及將記錄資料設定為串流處理到取用產品的相關詳細資訊，請參閱 [Azure SQL Database 計量和診斷記錄](sql-database-metrics-diag-logging.md)。 

### <a name="set-up-with-azure-sql-analytics"></a>使用 Azure SQL 分析設定 

Azure SQL 分析解決方案提供圖形化使用者介面、資料庫效能的報告和警示功能，以及 Intelligent Insights 診斷記錄資料。

> [!TIP]
> 快速入門：開始使用 Intelligent Insights 的最簡單方式是將它與 Azure SQL 分析搭配使用，以提供資料庫效能問題的圖形化使用者介面。 從市集新增 Azure SQL 分析解決方案，並在此解決方案內建立工作區，然後針對您想要在其上啟用 Intelligent Insights 的每個資料庫，在資料庫的 [診斷設定] 刀鋒視窗中設定將 "SQLInsights" 記錄串流處理到 Azure SQL 分析的工作區。
>

預先需求是將 Azure SQL 分析從市集新增至 Azure 入口網站儀表板，以及建立工作區，請參閱[設定 Azure SQL 分析](../log-analytics/log-analytics-azure-sql.md#configuration) 

若要搭配使用 Intelligent Insights 與 Azure SQL 分析，請將 Intelligent Insights 記錄資料設定為串流處理到您在上個步驟中建立的 Azure SQL 分析工作區，請參閱 [Azure SQL Database 計量和診斷記錄](sql-database-metrics-diag-logging.md)。 

下列範例顯示透過 Azure SQL 分析所檢視的 Intelligent Insights：

![Intelligent Insights 報表](./media/sql-database-intelligent-insights/intelligent-insights-azure-sql-analytics.png)

### <a name="set-up-with-event-hubs"></a>使用事件中樞設定

若要使用 Intelligent Insights 搭配事件中樞，請將 Intelligent Insights 記錄資料設定為串流處理到事件中樞，請參閱[將 Azure 診斷記錄串流處理至事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md)。

若要使用事件中樞設定自訂監視和警示，請參閱[如何在事件中樞處理計量和診斷記錄](sql-database-metrics-diag-logging.md#what-to-do-with-metrics-and-diagnostics-logs-in-event-hubs)。 

### <a name="set-up-with-azure-storage"></a>使用 Azure 儲存體設定

若要使用 Intelligent Insights 搭配儲存體，請將 Intelligent Insights 記錄資料設定為串流處理到儲存體，請參閱[串流處理到 Azure 儲存體](sql-database-metrics-diag-logging.md#stream-into-storage)。

### <a name="custom-integrations-of-intelligent-insights-log"></a>自訂的 Intelligent Insights 記錄整合

若要使用 Intelligent Insights 搭配協力廠商工具，或自訂警示與監視開發，請參閱[使用 Intelligent Insights 資料庫效能診斷記錄](sql-database-intelligent-insights-use-diagnostics-log.md)。

## <a name="detection-metrics"></a>偵測計量

用於產生 Intelligent Insights 之偵測模型的計量是以監視下列各項為基礎：

- 查詢持續時間
- 逾時要求
- 過長的等候時間
- 發生錯誤的要求

查詢持續時間和逾時要求會作為偵測資料庫工作負載效能相關問題的主要模型。 之所以會使用它們，是因為它們會直接測量工作負載發生什麼情況。 為了偵測所有可能的工作負載效能降低情況，會使用過長的等候時間和發生錯誤的要求作為額外的模型，以指出影響工作負載效能的問題。

系統會自動將對工作負載所做的變更及對資料庫發出的查詢要求數所做的變更納入考量，以動態判斷正常和異常的資料庫效能臨界值。

將會透過以科學方式衍生並將所偵測到之每個問題分類的資料模型，將所有計量以各種關聯性一起納入考量。 透過智慧型深入解析所提供的資訊包括：

* 所偵測到問題的詳細資料。 
* 針對所偵測到問題的根本原因分析。 
* 在可能的情況下改善被監視 SQL 資料庫效能之方法的建議。

## <a name="query-duration"></a>查詢持續時間

查詢持續時間效能降低模型會分析個別的查詢，並偵測與效能基準相比，花費在編譯和執行查詢上的時間是否增加。

當 SQL Database 內建的智慧功能偵測到查詢編譯或查詢執行時間大幅增加而影響到工作負載效能時，就會將這些查詢標示為查詢持續時間效能降低問題。 

Intelligent Insights 診斷記錄會輸出效能降低之查詢的查詢雜湊。 查詢雜湊會表示效能降低是否與會增加查詢持續時間的查詢編譯或執行時間增加有關。

## <a name="timeout-requests"></a>逾時要求

逾時要求效能降低模型會分析個別的查詢，並偵測與效能基準期間相比，查詢執行層級的逾時及資料庫層級的整體要求逾時是否有任何增加。

某些查詢甚至可能會在抵達執行階段之前便逾時。 透過比較已中止背景工作角色和做出的要求，SQL Database 的內建智慧功能會測量並分析所有抵達資料庫的查詢，無論它們是否抵達執行階段。 

在所執行之查詢的逾時數目或已中止之要求背景工作角色的數目增加到超出系統所管理的臨界值之後，診斷記錄中就會填入智慧型深入解析。

產生的深入解析包括逾時要求的數目，以及逾時查詢的數目。 效能降低的跡象會與執行階段的逾時提升相關聯，否則便會提供整體的資料庫層級。 當系統認為逾時增加對資料庫效能有重大影響時，就會將這些查詢標示為逾時效能降低問題。 

## <a name="excessive-wait-times"></a>過長的等候時間

過長的等候時間模型會監視個別的資料庫查詢。 它會偵測超出系統所管理之絕對臨界值的特高查詢等候統計資料。 您可以使用新的 SQL Server 功能「查詢存放區等候統計資料」(sys.query_store_wait_stats)，來觀察下列查詢過長等候時間計量：

- 達到資源限制
- 達到彈性集區資源限制
- 過多的背景工作或工作階段執行緒數目
- 過多的資料庫鎖定
- 記憶體壓力
- 其他等候統計資料

當達到資源限制或彈性集區資源限制時，即表示訂用帳戶上或彈性集區中的可用資源耗用量已超出絕對臨界值。 這些統計資料表示工作負載效能降低。 當背景工作或工作階段執行緒數目過多時，即表示所起始的背景工作執行緒或工作階段數目已超出絕對臨界值。 這些統計資料表示工作負載效能降低。

當資料庫鎖定過多時，即表示資料庫上的鎖定計數已超出絕對臨界值。 此統計資料表示工作負載效能降低。 當發生記憶體壓力時，即表示要求記憶體授與的執行緒數目已超出絕對臨界值。 此統計資料表示工作負載效能降低。

其他等候統計資料偵測所表示的情況則是，透過「查詢存放區等候統計資料」測量的其他計量已超出絕對臨界值。 這些統計資料表示工作負載效能降低。

偵測到過長的等候時間之後，根據可用的資料，Intelligent Insights 診斷記錄會輸出效能降低之影響和受影響查詢的雜湊、導致查詢在執行中等候的計量詳細資料，以及測量到的等候時間。

## <a name="errored-requests"></a>發生錯誤的要求

發生錯誤的要求效能降低模型會監視個別的查詢，並偵測與基準期間相比，發生錯誤的查詢數目是否增加。 此模型也會監視已超出 SQL Database 內建智慧功能所管理之絕對臨界值的重大例外狀況。 系統會自動將在受監視期間內，對資料庫和帳戶發出以進行任何工作負載變更的查詢要求數目納入考量。

當系統認為發生錯誤之要求的提升，相對於所發出的整體要求數目來說，對工作負載效能有重大影響時，就會將受影響的查詢標示為發生錯誤的要求效能降低問題。

Intelligent Insights 記錄會輸出發生錯誤之要求的計數。 它會指出效能降低是否和發生錯誤之要求的提升，或是和超出被監視的重大例外狀況臨界值相關聯，以及測量到的效能降低時間。 

如果任何一個受監視的重大例外狀況超出系統所管理的絕對臨界值，就會產生含有重大例外狀況詳細資料的智慧型深入解析。

## <a name="next-steps"></a>後續步驟
* 了解如何[使用 Intelligent Insights 針對 SQL Database 效能問題進行疑難排解](sql-database-intelligent-insights-troubleshoot-performance.md)。
* 使用 [Intelligent Insights SQL Database 效能診斷記錄](sql-database-intelligent-insights-use-diagnostics-log.md)。
* 了解如何[使用 SQL 分析來監視 SQL Database](../log-analytics/log-analytics-azure-sql.md)。
* 了解如何[收集並取用來自 Azure 資源的記錄資料](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。


