---
title: Azure 監視器中的記錄警示
description: 當您指定的分析查詢條件符合 Azure 警示時，觸發電子郵件、通知、呼叫網站 URL (Webhook) 或自動化。
author: msvijayn
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: 2e2db54f4c356a754144e17b11cf25fdf3f12d9f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993998"
---
# <a name="log-alerts-in-azure-monitor"></a>Azure 監視器中的記錄警示
此文章提供記錄警示的詳細資料，記錄警示是 [Azure 警示](monitoring-overview-unified-alerts.md) 內所支援的其中一種警示類型，可讓使用者使用 Azure 的分析平台來作為基礎警示。

記錄警示包含針對 [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 或 [Application Insights](../application-insights/app-insights-cloudservices.md#view-azure-diagnostic-events) 所建立的記錄搜尋規則。 若要深入了解其使用方式，請參閱[在 Azure 中建立記錄警示](alert-log.md)

> [!NOTE]
> [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 中常用的記錄資料現在也會提供在 Azure 監視器的計量平台上。 如需詳細資料檢視，請參閱[記錄的計量警示](monitoring-metric-alerts-logs.md)


## <a name="log-search-alert-rule---definition-and-types"></a>記錄搜尋警示規則 - 定義和類型

Azure 警示會建立記錄搜尋規則，以自動定期執行指定的記錄查詢。  如果記錄查詢的結果符合特定準則，則會建立警示的記錄。 規則接著可自動使用[動作群組](monitoring-action-groups.md)來執行一或多個動作。 

記錄搜尋規則會由下列詳細資料定義：
- **記錄查詢**。  每次引發警示規則都會執行的查詢。  此查詢所傳回的記錄將是用來判斷是否要建立警示。 Analytics 查詢也可以包含[跨應用程式呼叫](https://dev.applicationinsights.io/ai/documentation/2-Using-the-API/CrossResourceQuery)跨工作區呼叫，以及[跨資源呼叫](../log-analytics/log-analytics-cross-workspace-search.md)但前提是使用者具有外部應用程式的存取權限。 

    > [!IMPORTANT]
    > 使用者必須擁有[Azure 監視參與者](monitoring-roles-permissions-security.md)角色以便在 Azure 監視器中來建立、修改和更新記錄警示；還要加上在警示規則或警示查詢中存取及查詢分析目標的執行權限。 如果執行建立的使用者沒有警示規則或警示查詢中所有分析目標的存取權 - 則規則建立會失敗，或者會以部分結果來執行記錄警示規則。

- **時間週期**。  指定查詢的時間範圍。 查詢只會傳回在此目前時間範圍內建立的記錄。 時間週期會限制為了查詢記錄所能擷取的資料以防濫用，並可規避記錄查詢中所使用的任何時間命令 (例如 ago)。 <br>*例如，如果時間週期設定為 60 分鐘，且查詢會在下午 1:15 執行，則只會傳回在下午 12:15 與下午 1:15 之間所建立的記錄以執行記錄查詢。現在，如果記錄查詢使用 ago (7d) 之類的時間命令，則只會對下午 12:15 與下午 1:15 之間的資料執行記錄查詢 - 彷彿只有過去 60 分鐘有資料。而不是記錄查詢中所指定的七天資料。*
- **頻率**。  指定應執行查詢的頻率。 可以是介於 5 分鐘與 24 小時之間的任何值。 應等於或小於此時間週期。  如果值大於時間週期，則您可能有遺漏記錄的風險。<br>*例如，請考慮 30 分鐘的時間週期，以及 60 分鐘的頻率。如果在 1:00 執行查詢，它會傳回 12:30 到下午 1:00 之間的記錄。下一次執行查詢就是 2:00 時，它會傳回 1:30 至 2:00 之間的記錄。1:00 和 1:30 之間建立的任何記錄一律不會評估。*
- **閾值**。  系統會評估記錄搜尋的結果，以判斷是否應該建立警示。  不同類型的記錄搜尋警示規則會有不同的閾值。

不論是 [Azure Log Analytics](../log-analytics/log-analytics-tutorial-viewdata.md) 還是 [Application Insights](../application-insights/app-insights-cloudservices.md#view-azure-diagnostic-events) 的記錄搜尋規則都可以屬於兩個類型。 下列各節會詳細說明這兩個類型。

- **[結果數目](#number-of-results-alert-rules)**。 當記錄搜尋所傳回的數目記錄超過指定數目時，系統會建立單一警示。
- **[計量測量](#metric-measurement-alert-rules)**。  針對記錄搜尋結果中的每個物件，若其值超過指定的閾值，系統就會為其建立警示。

警示規則類型之間的差異如下所示。

- 「結果數目」警示規則一律會建立單一警示，而「計量測量」警示規則會為每個超過閾值的物件建立警示。
- *結果數目*警示規則會在閾值超過一次時建立警示。 *計量測量*警示規則會在閾值於特定時間間隔內超過一定次數時建立警示。

### <a name="number-of-results-alert-rules"></a>結果數目警示規則
**結果數目**警示規則會在搜尋查詢所傳回的記錄數目超過指定閾值時建立單一警示。 這種類型的警示規則適用於處理例如 Windows 事件記錄、Syslog、WebApp 回應和自訂記錄的事件。  您可能希望在建立特定的錯誤事件時，或是在特定時間週期內建立多個錯誤事件時建立警示。

**閾值**：結果數目警示規則的閾值會大於或小於特定值。  如果記錄搜尋所傳回的記錄數目符合此準則，則會建立警示。

若要針對單一事件警示，請將結果數目設定為大於 0，並檢查在上次執行查詢後是否發生所建立的單一事件。 有些應用程式可能會記錄不一定會產生警示的偶發錯誤。  例如，應用程式可能會重試造成錯誤事件的處理序，而下一次就會成功。  在此情況下，除非在特定時間週期內建立多個事件，否則您不會想建立警示。  

在某些情況下，您可能想在某個事件不存在時建立警示。  例如，處理序可能會記錄一般事件，表示運作正常。  如果它不在特定的時間週期內記錄一個事件，則應建立警示。  在此情況下，您會將閾值設定為**小於 1**。

#### <a name="example-of-number-of-records-type-log-alert"></a>記錄數目類型記錄警示的範例
請考慮一個情境，您想要知道您的 Web 架構應用程式何時向使用者回應程式碼 500 (亦即) 內部伺服器錯誤。 您可以建立詳細資料如下的警示規則：  
- **查詢：** 要求 | 其中 resultCode =="500"<br>
- **時間週期：** 30 分鐘<br>
- **警示頻率：** 5 分鐘<br>
- **閾值：** 大於 0<br>

然後警示會以 30 分鐘的資料，每隔 5 分鐘執行一次查詢，以尋找結果碼為 500 的任何記錄。 如果找到一筆這樣的記錄，它就會引發警示並觸發設定的動作。

### <a name="metric-measurement-alert-rules"></a>公制度量單位的警示規則

- **計量測量**警示規則會針對查詢中其值超過指定閾值的每個物件建立警示。  這些警示規則與**結果數目**警示規則具有下列明顯差異。
- **彙總函式**：決定要執行的計算，並可能決定要彙總的數值欄位。  例如，**count()** 會在查詢中傳回記錄數目，**avg(CounterValue)** 則會傳回 CounterValue 欄位在一段間隔內的平均值。 查詢中的彙總函式必須是名為/稱為：AggregatedValue，並提供數值。 
- **群組欄位**：系統會為這個欄位中的每個執行個體建立帶有彙總值的記錄，而且每個執行個體都可產生警示。  例如，如果您想要為每部電腦產生警示，您可以使用**依電腦** 

    > [!NOTE]
    > 對於以 Application Insights 為基礎的計量測量警示規則，您可以指定用來分組資料的欄位。 若要這樣做，請使用規則定義中的 [彙總依據] 選項。   
    
- **間隔**：定義用來彙總資料的時間間隔。  例如，如果您指定 **5 分鐘**，則系統會為群組欄位的每個執行個體建立記錄，而這些執行個體是在針對警示所指定的時間週期內以每 5 分鐘為間隔進行彙總的。

    > [!NOTE]
    > 查詢中必須使用 Bin 函式來指定間隔。 由於 bin() 可能會導致時間間隔不相等，警示會在執行階段自動使用適當時間將 bin 命令轉換為 bin_at 命令，以確保結果有固定時間點。 警示記錄的計量測量類型的設計宗旨是搭配具有單一 bin() 命令的查詢使用
    
- **閾值**：計量測量警示規則的閾值是由彙總值與若干違規所定義。  如果記錄搜尋中的任何資料點超過此值，系統就會將其視為違規。  如果結果中任何物件的違規數目超過指定值，系統舊會為該物件建立警示。

#### <a name="example-of-metric-measurement-type-log-alert"></a>計量測量類型記錄警示的範例
假設您想要在任何電腦於過去 30 分鐘內發生三次處理器使用率超過 90% 時收到警示。  您可以建立詳細資料如下的警示規則：  

- **查詢：** Perf | where ObjectName == "Processor" and CounterName == "% Processor Time" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 5m), Computer<br>
- **時間週期：** 30 分鐘<br>
- **警示頻率：** 5 分鐘<br>
- **彙總值︰** 大於 90<br>
- **警示觸發依據︰** 違規總數大於 2<br>

查詢會以 5 分鐘為間隔為每部電腦建立平均值。  此查詢會每隔 5 分鐘針對在先前 30 分鐘內所收集的資料來執行。  三部電腦的資料範例如下所示。

![查詢結果範例](./media/monitor-alerts-unified/metrics-measurement-sample-graph.png)

在此範例中，系統會為 srv02 和 srv03 建立個別警示，因為它們在時間週期內違反 90% 閾值三次。  如果**警示觸發依據︰** 變更為**連續**，則系統只會為 srv03 建立警示，因為它違反了 3 個連續取樣的閾值。

## <a name="log-search-alert-rule---firing-and-state"></a>記錄搜尋警示規則 - 引發與狀態
記錄搜尋警示規則適用於使用者根據組態和所使用的自訂分析查詢所決定的邏輯。 由於應該觸發警示規則的確切條件或原因的邏輯是封裝在 Analytics 查詢中 - 因此每個記錄警示規則可能各不相同。 當符合或超過記錄搜索警示規則的閾值條件時，Azure 警示幾乎沒有關於記錄結果內特定基礎根本原因的資訊。 因此，記錄警示稱為無狀態，而且每當記錄搜尋結果足以超過*結果數目*或*計量測量*類型的條件時，將會引發記錄警示。 而且，只要提供的自訂分析查詢結果符合警示條件，記錄警示規則就會持續引發；沒有一個警示獲得解決。 由於監視失敗的確切根本原因邏輯在使用者提供的分析查詢中受到遮掩；因此 Azure 警示無法明確推論不符合閾值的記錄搜尋結果是否表示問題已解決。

現在假設我們有一個名為 *Contoso-Log-Alert* 的記錄警示規則，根據的是[結果數目類型記錄警示提供的範例](#example-of-number-of-records-type-log-alert)中的組態。 
- 下午 1:05，在 Azure 警示執行 Contoso-Log-Alert 時，記錄搜尋結果產生 0 個記錄；低於閾值，因此不引發警示。 
- 下午 1:10 的下一個反覆項目，在 Azure 警示執行 Contoso-Log-Alert 時，記錄搜尋結果提供了 5 個記錄；超過閾值並在觸發相關聯的[動作群組](monitoring-action-groups.md)之後不久引發警示。 
- 下午 1:15，在 Azure 警示執行 Contoso-Log-Alert 時，記錄搜尋結果提供了 2 個記錄；超過閾值並在觸發相關聯的[動作群組](monitoring-action-groups.md)之後不久引發警示。
- 現在，下午 1:20 的下一個反覆項目，在 Azure 警示執行 Contoso-Log-Alert 時，記錄搜尋結果再次提供 0 個記錄；低於閾值，因此不引發警示。

但在上述所列的情況中，下午 1:15 - Azure 警示無法判斷下午 1:10 發現的基本問題是否仍存在，以及是否有完全新的失敗；因為使用者提供的查詢可能會算在較早的記錄中 - 這是 Azure 警示可以確定的。 因此，為了謹慎起見，Contoso-Log-Alert 會在下午 1:15 透過設定[動作群組](monitoring-action-groups.md)再引發一次。 現在，下午 1:20 沒有看到任何記錄 - Azure 警示無法確定記錄的原因是否已經解決；因此，Contoso-Log-Alert 不會在 Azure 儀表板中和/或已發出表示解決警示的通知中變更為已解決。


## <a name="pricing-and-billing-of-log-alerts"></a>記錄警示的價格和計費
在 [Azure 監視器價格](https://azure.microsoft.com/en-us/pricing/details/monitor/)頁面有說明適用於記錄警示的價格。 在 Azure 帳單中，記錄警示是以類型 `microsoft.insights/scheduledqueryrules` 搭配下列項目來表示：
- 隨著警示名稱與資源群組警示內容顯示的 Application Insights 上的記錄警示
- 隨著警示名稱 (`<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>`) 與資源群組和警示內容顯示的 Log Analytics 上的記錄警示

    > [!NOTE]
    > Log Analytics API 所建立並儲存的所有搜尋、排程和動作，都必須使用小寫名稱。 若使用無效的字元 (例如 `<, >, %, &, \, ?, /`) - 它們在帳單中將會被取代為 `_`。

## <a name="next-steps"></a>後續步驟
* 了解[在 Azure 警示中的記錄警示中建立](alert-log.md)。
* 了解 [Azure 中記錄警示中的 Webhook](monitor-alerts-unified-log-webhook.md)。
* 了解 [Azure 警示](monitoring-overview-unified-alerts.md)。
* 深入了解 [Application Insights](../application-insights/app-insights-analytics.md)。
* 深入了解 [Log Analytics](../log-analytics/log-analytics-overview.md)。    
