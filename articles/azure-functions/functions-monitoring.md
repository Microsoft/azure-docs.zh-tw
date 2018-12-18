---
title: 監視 Azure Functions
description: 了解如何使用 Azure Application Insights 搭配 Azure Functions 來監視函式執行。
services: functions
author: ggailey777
manager: jeconnoc
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構
ms.assetid: 501722c3-f2f7-4224-a220-6d59da08a320
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/15/2017
ms.author: glenga
ms.openlocfilehash: fb9de98a80d348c3ba1e84ae19551c7ca080628b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966838"
---
# <a name="monitor-azure-functions"></a>監視 Azure Functions

## <a name="overview"></a>概觀 

[Azure Functions](functions-overview.md) 提供與 [Azure Application Insights](../application-insights/app-insights-overview.md) 的內建整合來監視函式。 本文示範如何設定 Functions，以將遙測資料傳送至 Application Insights。

![Application Insights 計量瀏覽器](media/functions-monitoring/metrics-explorer.png)

Functions 也有[未使用 Application Insights 的內建監視](#monitoring-without-application-insights)。 我們建議使用 Application Insights，因為它提供更多資料和更好的方法來分析資料。

## <a name="application-insights-pricing-and-limits"></a>Application Insights 定價和限制

您可以免費嘗試 Azure Application Insights 與 Function Apps 整合。 不過，還有可以免費處理多少資料的每日限制，而且您可能會在測試期間達到該限制。 當您趨近每日限制時，Azure 會提供入口網站及電子郵件通知。  但是，如果您錯過這些警示並達到該限制，Application Insights 查詢中就不會出現新記錄。 因此請留意限制，以避免不必要的疑難排解時間。 如需詳細資訊，請參閱[管理 Application Insights 中的價格和資料磁碟區](../application-insights/app-insights-pricing.md)。

## <a name="enable-app-insights-integration"></a>啟用 Application Insights 整合

針對將資料傳送至 Application Insights 的函式應用程式，它需要知道 Application Insights 資源的檢測金鑰。 必須在名為 APPINSIGHTS_INSTRUMENTATIONKEY 的應用程式設定中提供此金鑰。

您無法在 [Azure 入口網站](https://portal.azure.com)中建立此連線：

* [自動新的函式應用程式](#new-function-app)
* [手動連線 App Insights 資源](#manually-connect-an-app-insights-resource)

### <a name="new-function-app"></a>新的函式應用程式

1. 移至函式應用程式 [建立] 頁面。

1. 將 [Application Insights] 參數設為 [開啟]。

2. 選取 **Application Insights 位置**。

   在您想要儲存資料的 [Azure 地理位置](https://azure.microsoft.com/global-infrastructure/geographies/)中，選擇最接近函式應用程式區域的區域。

   ![建立函式應用程式時啟用 Application Insights](media/functions-monitoring/enable-ai-new-function-app.png)

3. 輸入其他必要資訊。

1. 選取 [建立] 。

下一個步驟式[停用內建記錄功能](#disable-built-in-logging)。

### <a name="manually-connect-an-app-insights-resource"></a>手動連線 App Insights 資源 

1. 建立 Application Insights 資源。 將應用程式類型設為 [一般]。

   ![建立 Application Insights 資源，輸入 [一般]](media/functions-monitoring/ai-general.png)

2. 從 Application Insights 資源的 [程式集] 頁面複製檢測金鑰。 將滑鼠停留在所顯示金鑰值的結尾處，以取得 [按一下以複製] 按鈕。

   ![複製 Application Insights 檢測金鑰](media/functions-monitoring/copy-ai-key.png)

1. 在函式應用程式的 [應用程式設定] 頁面上，按一下 [新增設定] 以[新增應用程式設定](functions-how-to-use-azure-function-app-settings.md#settings)。 將新的設定命名為 APPINSIGHTS_INSTRUMENTATIONKEY，並貼上所複製的檢測金鑰。

   ![將檢測金鑰新增至應用程式設定](media/functions-monitoring/add-ai-key.png)

1. 按一下 [檔案] 。

## <a name="disable-built-in-logging"></a>停用內建記錄

如果您啟用 Application Insights，我們建議您停用[使用 Azure 儲存體的內建記錄](#logging-to-storage)。 內建記錄適合用來測試少量的工作負載，但不適用於負載繁重的生產環境。 若要監控生產環境，建議您使用 Application Insights。 如果將內建記錄用於生產環境，記錄內容可能會因為 Azure 儲存體的節流而有所佚失。

若要停用內建記錄，請刪除 `AzureWebJobsDashboard` 應用程式設定。 如需在 Azure 入口網站中刪除應用程式設定的相關資訊，請參閱[如何管理函式應用程式](functions-how-to-use-azure-function-app-settings.md#settings)的**應用程式設定**。 在刪除應用程式設定之前，請確定相同的函式應用程式中沒有現有函式將它使用於 Azure 儲存體觸發程序或繫結。

## <a name="view-telemetry-in-monitor-tab"></a>在監視索引標籤中檢視遙測

在您設定如上一節所示的 Application Insights 整合之後，您可以在 [監視器] 索引標籤中檢視遙測資料。

1. 在函式應用程式頁面中，選取在 Application Insights 設定之後至少執行一次的函式，然後選取 [監視] 索引標籤。

   ![選取監視索引標籤](media/functions-monitoring/monitor-tab.png)

2. 定期選取 [重新整理]，直到函式引動過程清單出現為止。

   由於遙測用戶端將資料分批傳輸到伺服器的方式，此清單可能需要 5 分鐘的時間才會出現。 (此延遲不適用於[即時計量資料流](../application-insights/app-insights-live-stream.md)。 當您載入頁面時，該服務會連線到函式主機，讓記錄直接串流處理至頁面。)

   ![引動過程清單](media/functions-monitoring/monitor-tab-ai-invocations.png)

2. 若要查看特定函式引動過程的記錄，請選取該引動過程的 [日期] 資料行連結。

   ![引動過程詳細資料連結](media/functions-monitoring/invocation-details-link-ai.png)

   該引動過程的記錄輸出會顯示在新頁面中。

   ![引動過程詳細資料](media/functions-monitoring/invocation-details-ai.png)

這兩個頁面 (引動過程清單和詳細資料) 都會連結至 Application Insights 分析查詢以擷取資料：

![在 Application Insights 中執行](media/functions-monitoring/run-in-ai.png)

![Application Insights 分析引動過程清單](media/functions-monitoring/ai-analytics-invocation-list.png)

從這些查詢中，您可以看到引動過程清單受限於最近 30 天，不能超過 20 個資料列 (`where timestamp > ago(30d) | take 20`)，而引動過程詳細資料清單則為過去 30 天且沒有限制。

如需詳細資訊，請參閱本文稍後的[查詢遙測資料](#query-telemetry-data)。

## <a name="view-telemetry-in-app-insights"></a>在 Application Insights 中檢視遙測

若要在 Azure 入口網站中從函式應用程式開啟 Application Insights，請選取函式應用程式 [概觀] 頁面的 [已設定的功能] 區段中的 [Application Insights] 連結。

![概觀頁面上的 Application Insights 連結](media/functions-monitoring/ai-link.png)


如需如何使用 Application Insights 的相關資訊，請參閱 [Application Insights 文件](https://docs.microsoft.com/azure/application-insights/)。 本節示範一些如何在 Application Insights 中檢視資料的範例。 如果您已經熟悉 Application Insights，就可以直接前往[關於設定和自訂遙測資料的小節](#configure-categories-and-log-levels)。

在[計量瀏覽器](../application-insights/app-insights-metrics-explorer.md)中，您可以根據計量 (例如函式引動過程的數量、執行時間及成功率) 建立圖表和警示。

![計量瀏覽器](media/functions-monitoring/metrics-explorer.png)

在 [[失敗](../application-insights/app-insights-asp-net-exceptions.md)] 索引標籤上，您可以根據函式失敗和伺服器例外狀況建立圖表和警示。 **作業名稱**是函式名稱。 除非您實作[自訂遙測](#custom-telemetry-in-c-functions)來取得相依性，否則不會顯示相依性中的失敗。

![失敗](media/functions-monitoring/failures.png)

在 [[效能](../application-insights/app-insights-performance-counters.md)] 索引標籤上，您可以分析效能問題。

![效能](media/functions-monitoring/performance.png)

[伺服器] 索引標籤會顯示每一部伺服器的資源使用量和輸送量。 如果要對函式拖累基礎資源的案例進行偵錯，此資料非常有用。 伺服器會作為「雲端角色執行個體」來參考。

![伺服器](media/functions-monitoring/servers.png)

[[即時計量資料流](../application-insights/app-insights-live-stream.md)] 索引標籤會在即時建立時顯示計量資料。

![即時資料流](media/functions-monitoring/live-stream.png)

## <a name="query-telemetry-data"></a>查詢遙測資料

[Application Insights 分析](../application-insights/app-insights-analytics.md)可讓您存取資料庫中資料表形式的所有遙測資料。 分析會提供用於擷取、操作和視覺化資料的查詢語言。

![選取 [分析]](media/functions-monitoring/select-analytics.png)

![分析範例](media/functions-monitoring/analytics-traces.png)

以下是查詢範例。 這一個會顯示過去 30 分鐘內每個背景工作的要求分佈。

```
requests
| where timestamp > ago(30m) 
| summarize count() by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
```

可用的資料表顯示於左窗格的 [結構描述] 索引標籤中。 您可以找到下表中函式引動過程所產生的資料：

* **traces**：由執行階段和函式程式碼所建立的記錄。
* **requests**：每個函式引動過程一個。
* **exceptions**：執行階段擲回的任何例外狀況。
* **customMetrics**：成功和失敗的引動過程計數、成功率、持續時間。
* **customEvents**：執行階段所追蹤的事件，例如：觸發函式的 HTTP 要求。
* **performanceCounters**：函式執行所在的伺服器效能相關資訊。

其他資料表適用於可用性測試和用戶端/瀏覽器遙測。 您可以實作自訂遙測，將資料新增至它們。

在每個資料表內，一些 Functions 特有的資料會位於 `customDimensions` 欄位中。  例如，下列查詢會擷取記錄層級為 `Error` 的所有追蹤。

```
traces 
| where customDimensions.LogLevel == "Error"
```

執行階段提供 `customDimensions.LogLevel` 和 `customDimensions.Category`。 您可以在函式程式碼內撰寫的記錄中提供額外的欄位。 請參閱本文稍後的[結構化記錄](#structured-logging)。

## <a name="configure-categories-and-log-levels"></a>設定類別和記錄層級

您可以使用 Application Insights 而不需任何自訂設定，但預設的組態可能會導致大量資料。 如果您使用 Visual Studio Azure 訂用帳戶，可能就會達到 Application Insights 適用的資料上限。 本文的其餘部分示範如何設定及自訂函式傳送至 Application Insights 的資料。

### <a name="categories"></a>類別

Azure Functions 記錄器包括每個記錄的「類別」。 類別指出寫入記錄的是哪個部分的執行階段程式碼或函式程式碼。 

Functions 執行階段會建立含有以 "Host" 開頭之類別的記錄。 例如，「函式已啟動」、「函式已執行」及「函式已完成」記錄的類別為 "Host.Executor"。 

如果您在函式程式碼中寫入記錄，則其類別為 "Function"。

### <a name="log-levels"></a>記錄層級

Azure Functions 記錄器也包含具有每個記錄的「記錄層級」。 [LogLevel](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel#Microsoft_Extensions_Logging_LogLevel) 為一個列舉，而整數代碼表示相對的重要性：

|LogLevel    |代碼|
|------------|---|
|追蹤       | 0 |
|偵錯       | 1 |
|資訊 | 2 |
|警告     | 3 |
|Error       | 4 |
|重要    | 5 |
|None        | 6 |

下一節會說明記錄層級 `None`。 

### <a name="configure-logging-in-hostjson"></a>在 Host.json 中設定記錄

*Host.json* 檔案會設定函式應用程式傳送到 Application Insights 的記錄數量。 針對每個類別，您可以指出要傳送的最小記錄層級。 以下是範例：

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host.Results": "Error",
        "Function": "Error",
        "Host.Aggregator": "Trace"
      }
    }
  }
}
```

此範例會設定下列規則：

1. 針對類別為 "Host.Results" 或 "Function" 的記錄，只會將 `Error` 層級和以上層級傳送至 Application Insights。 `Warning` 層級和以下層級的記錄均會被忽略。
2. 針對類別為 Host.Aggregator 的記錄，將所有記錄傳送至 Application Insights。 `Trace` 記錄層級和某些記錄器稱為 `Verbose` 的記錄層級相同，但會在 *host.json* 檔案中使用 `Trace`。
3. 針對所有其他記錄，只會將 `Information` 層級和以上層級傳送至 Application Insights。

*Host.json* 中的類別值會控制以相同值開頭之所有類別的記錄。 例如，*host.json* 中的 "Host" 會控制 "Host.General"、"Host.Executor"、"Host.Results" 等的記錄。

如果 *host.json* 包含多個以相同字串開頭的類別，則會先比對較長的類別。 例如，假設您想要取得執行階段在 `Error` 層級上記錄的所有項目 ("Host.Aggregator" 除外)，但您想要 "Host.Aggregator" 記錄於 `Information` 層級：

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host": "Error",
        "Function": "Error",
        "Host.Aggregator": "Information"
      }
    }
  }
}
```

若要隱藏類別的所有記錄，您可以使用記錄層級 `None`。 不會使用該類別寫入任何記錄，而且沒有任何高於該類別的記錄層級。

下列各節說明執行階段建立之記錄的主要類別。 

### <a name="category-hostresults"></a>類別 Host.Results

這些記錄會在 Application Insights 中顯示為 "requests"。 它們表示函式的成功或失敗。

![要求圖表](media/functions-monitoring/requests-chart.png)

這些記錄全都寫入於 `Information` 層級，因此，如果您在 `Warning` 或以上層級進行篩選，將不會看到任何這類資料。

### <a name="category-hostaggregator"></a>類別 Host.Aggregator

這些記錄會透過[可設定](#configure-the-aggregator)的期間來提供函式引動過程的計數與平均。 預設期間為 30 秒或 1,000 個結果，視何者較早達到而定。 

您可在 Application Insights 中的 **customMetrics** 資料表取得記錄。 範例包括執行個數、成功率和持續時間。

![customMetrics 查詢](media/functions-monitoring/custom-metrics-query.png)

這些記錄全都寫入於 `Information` 層級，因此，如果您在 `Warning` 或以上層級進行篩選，將不會看到任何這類資料。

### <a name="other-categories"></a>其他類別

除了已經列出的類別之外，其他類別的所有記錄均可在 Application Insights 中的 **traces** 資料表內取得。

![追蹤查詢](media/functions-monitoring/analytics-traces.png)

以 "Host" 開頭之類別的所有記錄都是由 Functions 執行個體寫入的。 「函式已啟動」和「函式已完成」記錄的類別為 "Host.Executor"。 針對成功的執行，這些記錄是 `Information` 層級；例外狀況會記錄於 `Error` 層級。 執行階段也會建立 `Warning` 層級的記錄，例如：傳送至有害佇列的佇列訊息。

您函式程式碼所寫入的記錄類別為 "Function"，而且可能是任何記錄層級。

## <a name="configure-the-aggregator"></a>設定彙總工具

如前一節所述，執行階段經過一段時間就會彙總有關函式執行的資料。 預設期間為 30 秒或 1,000 個執行，視何者較早達到而定。 您可以在 *host.json* 檔案中設定此設定。  以下是範例：

```json
{
    "aggregator": {
      "batchSize": 1000,
      "flushTimeout": "00:00:30"
    }
}
```

## <a name="configure-sampling"></a>設定取樣

Application Insights 具有[取樣](../application-insights/app-insights-sampling.md)功能，可以提供保護，避免在尖峰負載的情況下產生過多的遙測資料。 當內送遙測速率超過指定的閾值時，Application Insights 就會開始隨機忽略部分傳入的項目。 每秒的項目數上限的預設值為 5。 您可以在 *host.json* 中設定取樣。  以下是範例：

```json
{
  "applicationInsights": {
    "sampling": {
      "isEnabled": true,
      "maxTelemetryItemsPerSecond" : 5
    }
  }
}
```

## <a name="write-logs-in-c-functions"></a>在 C# 函式中寫入記錄

您可以在函式程式碼中寫入記錄，其會在 Application Insights 中顯示為追蹤。

### <a name="ilogger"></a>ILogger

在您的函式中使用 [ILogger](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger) 參數而不是 `TraceWriter` 參數。 使用 `TraceWriter` 建立的記錄確實會移至 Application Insights，但 `ILogger` 可讓您執行[結構化記錄](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging) \(英文\)。

利用 `ILogger` 物件，您可以呼叫 `Log<level>` [擴充方法 (位於 ILogger 上)](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loggerextensions#Methods_) \(英文\) 來建立記錄。 例如，下列程式碼會寫入 `Information` 記錄且類別為 "Function"。

```cs
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger logger)
{
    logger.LogInformation("Request for item with key={itemKey}.", id);
```

### <a name="structured-logging"></a>結構化記錄

預留位置的順序 (而不是名稱) 會決定要在記錄訊息中使用的參數。 例如，假設您有下列程式碼：

```csharp
string partitionKey = "partitionKey";
string rowKey = "rowKey";
logger.LogInformation("partitionKey={partitionKey}, rowKey={rowKey}", partitionKey, rowKey);
```

如果您保留相同的訊息字串並反轉參數的順序，則產生的訊息文字值會處於錯誤的位置上。

您可以使用這種方式來處理預留位置，讓您能夠執行結構化記錄。 除了訊息字串，Application Insights 還會儲存參數名稱/值組。 結果就是訊息引數會變成您可以查詢的欄位。

例如，如果記錄器方法呼叫看起來像上述範例，則您可以查詢欄位 `customDimensions.prop__rowKey`。 `prop__` 前置詞已新增以確保執行階段新增的欄位與您函式程式碼新增的欄位之間沒有衝突。

您也可以藉由參考欄位 `customDimensions.prop__{OriginalFormat}`，在原始訊息字串上進行查詢。  

以下是 `customDimensions` 資料的範例 JSON 表示法：

```json
{
  customDimensions: {
    "prop__{OriginalFormat}":"C# Queue trigger function processed: {message}",
    "Category":"Function",
    "LogLevel":"Information",
    "prop__message":"c9519cbf-b1e6-4b9b-bf24-cb7d10b1bb89"
  }
}
```

### <a name="logging-custom-metrics"></a>記錄自訂計量  

在 C# 指令碼函式中，您可以使用 `ILogger` 上的 `LogMetric` 擴充方法，在 Application Insights 中建立自訂計量。 以下是範例方法呼叫：

```csharp
logger.LogMetric("TestMetric", 1234); 
```

此程式碼是使用[適用於 .NET 的 Application Insights API](#custom-telemetry-in-c-functions) 呼叫 `TrackMetric` 的替代方法。

## <a name="write-logs-in-javascript-functions"></a>在 JavaScript 函式中寫入記錄

在 Node.js 函式，使用 `context.log` 寫入記錄。 不啟用結構化記錄。

```
context.log('JavaScript HTTP trigger function processed a request.' + context.invocationId);
```

### <a name="logging-custom-metrics"></a>記錄自訂計量  

在 Node.js 函式中，您可以使用 `context.log.metric` 方法，在 Application Insights 中建立自訂計量。 以下是範例方法呼叫：

```javascript
context.log.metric("TestMetric", 1234); 
```

此程式碼是使用[適用於 Application Insights 的 Node.js SDK](#custom-telemetry-in-javascript-functions) 呼叫 `trackMetric` 的替代方法。

## <a name="custom-telemetry-in-c-functions"></a>C# 函式中的自訂遙測

您可以使用 [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) \(英文\) NuGet 封裝，將自訂遙測資料傳送至 Application Insights。

以下是使用[自訂遙測 API](../application-insights/app-insights-api-custom-events-metrics.md) 的 C# 程式碼範例。 此範例適用於 .NET 類別庫，但 Application Insights 程式碼同樣適用於 C# 指令碼。

```cs
using System;
using System.Net;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Azure.WebJobs;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Linq;

namespace functionapp0915
{
    public static class HttpTrigger2
    {
        private static string key = TelemetryConfiguration.Active.InstrumentationKey = 
            System.Environment.GetEnvironmentVariable(
                "APPINSIGHTS_INSTRUMENTATIONKEY", EnvironmentVariableTarget.Process);

        private static TelemetryClient telemetryClient = 
            new TelemetryClient() { InstrumentationKey = key };

        [FunctionName("HttpTrigger2")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
            HttpRequestMessage req, ExecutionContext context, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            DateTime start = DateTime.UtcNow;

            // parse query parameter
            string name = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
                .Value;

            // Get request body
            dynamic data = await req.Content.ReadAsAsync<object>();

            // Set name to query string or body data
            name = name ?? data?.name;
         
            // Track an Event
            var evt = new EventTelemetry("Function called");
            UpdateTelemetryContext(evt.Context, context, name);
            telemetryClient.TrackEvent(evt);
            
            // Track a Metric
            var metric = new MetricTelemetry("Test Metric", DateTime.Now.Millisecond);
            UpdateTelemetryContext(metric.Context, context, name);
            telemetryClient.TrackMetric(metric);
            
            // Track a Dependency
            var dependency = new DependencyTelemetry
                {
                    Name = "GET api/planets/1/",
                    Target = "swapi.co",
                    Data = "https://swapi.co/api/planets/1/",
                    Timestamp = start,
                    Duration = DateTime.UtcNow - start,
                    Success = true
                };
            UpdateTelemetryContext(dependency.Context, context, name);
            telemetryClient.TrackDependency(dependency);
        }
        
        // This correllates all telemetry with the current Function invocation
        private static void UpdateTelemetryContext(TelemetryContext context, ExecutionContext functionContext, string userName)
        {
            context.Operation.Id = functionContext.InvocationId.ToString();
            context.Operation.ParentId = functionContext.InvocationId.ToString();
            context.Operation.Name = functionContext.FunctionName;
            context.User.Id = userName;
        }
    }    
}
```

不要呼叫 `TrackRequest` 或 `StartOperation<RequestTelemetry>`，因為您將會看到對函式引動過程的重複要求。  Functions 執行階段會自動追蹤要求。

請勿設定 `telemetryClient.Context.Operation.Id`。 這是一項全域設定，且在許多函式同時執行時，會導致不正確的相互關聯。 請改為建立新的遙測執行個體 (`DependencyTelemetry`、`EventTelemetry`)，並修改其 `Context` 屬性。 接著，將遙測執行個體傳入至 `TelemetryClient` 上的對應 `Track` 方法 (`TrackDependency()`、`TrackEvent()`)。 這可確保遙測具有目前函式引動過程的正確相互關聯詳細資料。

## <a name="custom-telemetry-in-javascript-functions"></a>JavaScript 函式中的自訂遙測

[Application Insights Node.js SDK](https://www.npmjs.com/package/applicationinsights) 目前處於為搶鮮版 (Beta)。 以下是一些將自訂遙測傳送至 Application Insights 的範例程式碼：

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    client.trackEvent({name: "my custom event", tagOverrides:{"ai.operation.id": context.invocationId}, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:{"ai.operation.id": context.invocationId}});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:{"ai.operation.id": context.invocationId}});
    client.trackTrace({message: "trace message", tagOverrides:{"ai.operation.id": context.invocationId}});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:{"ai.operation.id": context.invocationId}});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:{"ai.operation.id": context.invocationId}});

    context.done();
};
```

`tagOverrides` 參數會將 `operation_Id` 設為函式的引動過程識別碼。 此設定能夠讓指定的函式引動過程中所有自動產生和自訂的遙測相互關聯。

## <a name="known-issues"></a>已知問題

### <a name="dependencies"></a>相依性

函式對其他服務的相依性不會自動顯示，但您可以撰寫自訂程式碼來顯示相依性。 [C# 自訂遙測區段](#custom-telemetry-in-c-functions)中的範例程式碼會顯示作法。 範例程式碼會在 Application Insights 中產生如下的*應用程式對應*：

![應用程式對應](media/functions-monitoring/app-map.png)

### <a name="report-issues"></a>報告問題

若要回報關於 Functions 中 Application Insights 整合的問題，或是提出建議或要求，請[在 GitHub 中建立問題](https://github.com/Azure/Azure-Functions/issues/new) \(英文\)。

## <a name="monitoring-without-application-insights"></a>不使用 Application Insights 來監視

我們建議使用 Application Insights 來監視函式，因為它提供更多資料和更好的方法來分析資料。 但如果您偏好使用 Azure 儲存體的內建記錄系統，您可以繼續使用它。

### <a name="logging-to-storage"></a>記錄到儲存體

內建記錄使用 `AzureWebJobsDashboard` 應用程式設定中由連接字串所指定的儲存體帳戶。 在函式應用程式頁面中，依序選取函式和 [監視] 索引標籤，然後選擇讓它保留在傳統檢視中。

![切換至傳統檢視](media/functions-monitoring/switch-to-classic-view.png)

 您會取得函式執行清單。 選取函式執行，以檢閱持續時間、輸入資料、錯誤和相關聯的記錄檔。

如果您先前已啟用 Application Insights，但現在想要移至內建記錄功能，請手動停用 Application Insights，然後選取 [監視] 索引標籤。若要停用 Application Insights 整合，請刪除 APPINSIGHTS_INSTRUMENTATIONKEY 應用程式設定。

即使 [監視] 索引標籤顯示 Application Insights 資料，只要您尚未[停用內建記錄功能](#disable-built-in-logging)，就可以檢視檔案系統中的記錄資料。 在儲存體資源中，移至 [檔案]、選取適用於函式的檔案服務，然後移至 `LogFiles > Application > Functions > Function > your_function`，即可查看記錄檔。

### <a name="real-time-monitoring"></a>即時監視

您可以使用 [Azure 命令列介面 (CLI)](/cli/azure/install-azure-cli) 或 [Azure PowerShell](/powershell/azure/overview)，在本機工作站上將記錄檔串流處理至命令列工作階段。  

針對 Azure CLI，請使用下列命令來登入、選擇訂用帳戶及串流處理記錄檔：

```
az login
az account list
az account set <subscriptionNameOrId>
az webapp log tail --resource-group <resource group name> --name <function app name>
```

如果您使用 Azure PowerShell，請使用下列命令來新增 Azure 帳戶、選擇訂用帳戶及串流處理記錄檔：

```
PS C:\> Add-AzureAccount
PS C:\> Get-AzureSubscription
PS C:\> Get-AzureSubscription -SubscriptionName "<subscription name>" | Select-AzureSubscription
PS C:\> Get-AzureWebSiteLog -Name <function app name> -Tail
```

如需詳細資訊，請參閱[如何串流處理記錄檔](../app-service/web-sites-enable-diagnostic-log.md#streamlogs)。

### <a name="viewing-log-files-locally"></a>在本機檢視記錄檔

[!INCLUDE [functions-local-logs-location](../../includes/functions-local-logs-location.md)]

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列資源：

* [Application Insights](/azure/application-insights/)
* [ASP.NET Core 記錄](/aspnet/core/fundamentals/logging/)
