---
title: 監視 Azure Analysis Services 伺服器計量 | Microsoft Docs
description: 了解如何在 Azure 入口網站中監視 Analysis Services 伺服器計量。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 608323c467e0106af816c3432dec24090a9a9599
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442928"
---
# <a name="monitor-server-metrics"></a>監視伺服器計量

Analysis Services 提供計量來協助您監視伺服器的效能和健康情況。 例如，監視記憶體和 CPU 使用率、用戶端連接數目，以及查詢資源消耗。 Analysis Services 和其他大部分的 Azure 服務一樣，採用相同的監視架構。 若要深入了解，請參閱 [Microsoft Azure 中的計量](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md)。

若要在資源群組或訂用帳戶中跨多個服務資源執行更多深入的診斷、追蹤效能及識別趨勢，請使用 [Azure 監視器](https://azure.microsoft.com/services/monitor/)。 使用 Azure 監視器 (服務) 可能需要付費。


## <a name="to-monitor-metrics-for-an-analysis-services-server"></a>監視 Analysis Services 伺服器的計量

1. 在 Azure 入口網站中，選取 [計量]。

    ![在 Azure 入口網站中監視](./media/analysis-services-monitor/aas-monitor-portal.png)

2. 在 [可用的計量] 中，選取要併入圖表中的計量。 

    ![監視器圖表](./media/analysis-services-monitor/aas-monitor-chart.png)

<a id="#server-metrics"></a>
## <a name="server-metrics"></a>伺服器計量
請根據下表來決定哪些計量最適合您的監視情節。 同一圖表上只能顯示相同單位的計量。

|計量|計量顯示名稱|單位|彙總類型|說明|
|---|---|---|---|---|
|CommandPoolJobQueueLength|命令集區作業佇列長度|Count|平均值|命令執行緒集區佇列中的作業數目。|
|CurrentConnections|連接：目前的連接|Count|平均值|目前已建立的用戶端連接數目。|
|CurrentUserSessions|目前的使用者工作階段|Count|平均值|目前建立的使用者工作階段數目。|
|mashup_engine_memory_metric|M 引擎記憶體|位元組|平均值|混搭引擎處理序的記憶體使用量|
|mashup_engine_qpu_metric|M 引擎 QPU|Count|平均值|混搭引擎處理序的 QPU 使用量|
|memory_metric|記憶體|位元組|平均值|記憶體。 範圍 0-25 GB (S1)、0-50 GB (S2) 和 0-100 GB (S4)|
|memory_thrashing_metric|記憶體猛移|百分比|平均值|記憶體猛移的平均值。|
|CleanerCurrentPrice|記憶體︰清除工具目前價格|Count|平均值|記憶體目前的價格 ($/位元組/時間)，並正規化為 1000。|
|CleanerMemoryNonshrinkable|記憶體︰不可壓縮的清除器記憶體|位元組|平均值|背景清除器將不會清除的記憶體數量，以 KB 為單位。|
|CleanerMemoryShrinkable|記憶體︰可壓縮的清除器記憶體|位元組|平均值|背景清除器將清除的記憶體數量，以 KB 為單位。|
|MemoryLimitHard|記憶體︰固定記憶體限制|位元組|平均值|組態檔中的固定記憶體限制。|
|MemoryLimitHigh|記憶體︰記憶體上限|位元組|平均值|來自組態檔的記憶體上限。|
|MemoryLimitLow|記憶體︰記憶體下限|位元組|平均值|來自組態檔的記憶體下限。|
|MemoryLimitVertiPaq|記憶體︰記憶體限制 VertiPaq|位元組|平均值|來自組態檔的記憶體內部限制。|
|MemoryUsage|記憶體：記憶體使用量|位元組|平均值|伺服器處理序用於計算清除器記憶體價格的記憶體使用量。 等於計數器 Process\PrivateBytes 加上記憶體對應的資料大小，忽略記憶體內部分析引擎 (VertiPaq) 在超出引擎記憶體限制時所對應或配置的任何記憶體。|
|Quota|記憶體︰配額|位元組|平均值|目前的記憶體配額，以位元組為單位。 記憶體配額也就是指授與使用的記憶體，或是保留的記憶體。|
|QuotaBlocked|記憶體︰封鎖的配額|Count|平均值|在釋放其他記憶體配額之前，目前已封鎖的配額要求數目。|
|VertiPaqNonpaged|記憶體︰未分頁的 VertiPaq|位元組|平均值|工作集中已封鎖來供記憶體內部引擎使用的記憶體位元組。|
|VertiPaqPaged|記憶體︰分頁的 VertiPaq|位元組|平均值|可供記憶體內部資料使用的分頁記憶體位元組。|
|ProcessingPoolJobQueueLength|處理集區作業佇列長度|Count|平均值|處理執行緒集區佇列中的非 I/O 作業數目。|
|RowsConvertedPerSec|處理︰每秒轉換的資料列|每秒計數|平均值|處理期間資料列轉換的速率。|
|RowsReadPerSec|處理︰每秒讀取的資料列|每秒計數|平均值|從所有關聯式資料庫讀取資料列的速率。|
|RowsWrittenPerSec|處理︰每秒寫入的資料列|每秒計數|平均值|處理期間資料列寫入的速率。|
|qpu_metric|QPU|Count|平均值|QPU。 範圍 0-100 (S1)、0-200 (S2) 和 0-400 (S4)|
|QueryPoolBusyThreads|查詢集區忙碌執行緒|Count|平均值|查詢執行緒集區中的忙碌執行緒數目。|
|SuccessfullConnectionsPerSec|每秒連線成功的次數|每秒計數|平均值|成功完成連線的速率。|
|CommandPoolBusyThreads|執行緒︰命令集區的忙碌執行緒數|Count|平均值|命令執行緒集區中的忙碌執行緒數。|
|CommandPoolIdleThreads|執行緒︰命令集區的閒置執行緒數|Count|平均值|命令執行緒集區中的閒置執行緒數。|
|LongParsingBusyThreads|執行緒︰完整剖析的忙碌執行緒數|Count|平均值|完整剖析執行緒集區中的忙碌執行緒數目。|
|LongParsingIdleThreads|執行緒︰完整剖析的閒置執行緒數|Count|平均值|完整剖析執行緒集區中的閒置執行緒數目。|
|LongParsingJobQueueLength|執行緒︰完整剖析的作業佇列長度|Count|平均值|完整剖析執行緒集區佇列中的作業數目。|
|ProcessingPoolIOJobQueueLength|執行緒：處理集區 I/O 作業佇列長度|Count|平均值|處理執行緒集區佇列中的 I/O 作業數目。|
|ProcessingPoolBusyIOJobThreads|執行緒︰處理集區的忙碌 I/O 作業執行緒數|Count|平均值|處理執行緒集區中執行 I/O 作業的執行緒數目。|
|ProcessingPoolBusyNonIOThreads|執行緒︰處理集區的忙碌非 I/O 執行緒數|Count|平均值|處理執行緒集區中執行非 I/O 作業的執行緒數目。|
|ProcessingPoolIdleIOJobThreads|執行緒︰處理集區的閒置 I/O 作業執行緒數|Count|平均值|處理執行緒集區中 I/O 作業的閒置執行緒數目。|
|ProcessingPoolIdleNonIOThreads|執行緒︰處理集區的閒置非 I/O 執行緒數|Count|平均值|處理執行緒集區中專供非 I/O 作業使用的閒置執行緒數目。|
|QueryPoolIdleThreads|執行緒︰查詢集區的閒置執行緒數|Count|平均值|處理執行緒集區中 I/O 作業的閒置執行緒數目。|
|QueryPoolJobQueueLength|執行緒︰查詢集區的作業佇列長度|Count|平均值|查詢執行緒集區佇列中的作業數目。|
|ShortParsingBusyThreads|執行緒︰簡短剖析的忙碌執行緒數|Count|平均值|簡短剖析執行緒集區中的忙碌執行緒數目。|
|ShortParsingIdleThreads|執行緒︰簡短剖析的閒置執行緒數|Count|平均值|簡短剖析執行緒集區中的閒置執行緒數目。|
|ShortParsingJobQueueLength|執行緒︰簡短剖析的作業佇列長度|Count|平均值|簡短剖析執行緒集區佇列中的作業數目。|
|TotalConnectionFailures|連線失敗的總計|Count|平均值|連線嘗試失敗的總計。|
|TotalConnectionRequests|連線要求的總計|Count|平均值|連線要求的總計。 |

## <a name="next-steps"></a>後續步驟
[在 Microsoft Azure 中監視](../monitoring-and-diagnostics/monitoring-overview.md)   
[Microsoft Azure 中的計量](../monitoring-and-diagnostics/insights-how-to-customize-monitoring.md)   
[Azure 監視器 REST API 中的計量](https://msdn.microsoft.com/library/azure/dn931930.aspx)