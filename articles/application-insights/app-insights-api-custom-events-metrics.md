---
title: 自訂事件和度量的 Application Insights API | Microsoft Docs
description: 在您的裝置或桌面應用程式、網頁或服務中插入幾行程式碼，來追蹤使用狀況及診斷問題。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 80400495-c67b-4468-a92e-abf49793a54d
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/16/2018
ms.author: mbullwin
ms.openlocfilehash: 3de3a7b15f2ebef549f925f6604956ca16f4551b
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388087"
---
# <a name="application-insights-api-for-custom-events-and-metrics"></a>自訂事件和度量的 Application Insights API

在您的應用程式中插入幾行程式碼，以了解使用者對它進行的動作或協助診斷問題。 您可以從裝置和桌面應用程式、Web 用戶端以及 Web 伺服器傳送遙測。 使用 [Azure Application Insights](app-insights-overview.md) 核心遙測 API 來傳送自訂的事件和度量，以及您自己的標準遙測版本。 這個 API 與標準 Application Insights 資料收集器所使用的 API 相同。

> [!NOTE]
> `TrackMetric()` 不再是針對 .NET 型應用程式傳送自訂計量的慣用方法。 [2.60-beta 3 版](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/develop/CHANGELOG.md#version-260-beta3)的 Application Insights .NET SDK 中引進了新方法 [`TelemetryClient.GetMetric()`](https://docs.microsoft.com/dotnet/api/microsoft.applicationinsights.telemetryclient.getmetric?view=azure-dotnet)。 從 Application Insights .NET SDK [2.72 版](https://docs.microsoft.com/dotnet/api/microsoft.applicationinsights.telemetryclient.getmetric?view=azure-dotnet)起，這項功能現在屬於穩定的版本。

## <a name="api-summary"></a>API summary

核心 API 是跨所有平台統一的，除了 `GetMetric` (僅限 .NET) 之類的一些變化形式。

| 方法 | 用於 |
| --- | --- |
| [`TrackPageView`](#page-views) |頁面、畫面、刀鋒視窗或表單。 |
| [`TrackEvent`](#trackevent) |使用者動作和其他事件。 用來追蹤使用者行為，或監視效能。 |
| [`GetMetric`](#getmetric) |零和多維度計量、集中設定的彙總 (僅限 C#)。 |
| [`TrackMetric`](#trackmetric) |效能度量，例如與特定事件不相關的佇列長度。 |
| [`TrackException`](#trackexception) |記錄例外狀況以供診斷。 追蹤與其他事件的發生相對位置，並且檢查堆疊追蹤。 |
| [`TrackRequest`](#trackrequest) |記錄伺服器要求的頻率和持續時間以進行效能分析。 |
| [`TrackTrace`](#tracktrace) |診斷記錄訊息。 您也可以擷取第三方記錄檔。 |
| [`TrackDependency`](#trackdependency) |記錄應用程式所依賴之外部元件呼叫的持續時間及頻率。 |

您可以 [附加屬性和度量](#properties) 至這裡大部分的遙測呼叫。

## <a name="prep"></a>開始之前

如果您還沒有 Application Insights SDK 的參考：

* 將 Application Insights SDK 加入至專案：

  * [ASP.NET 專案](app-insights-asp-net.md)
  * [Java 專案](app-insights-java-get-started.md)
  * [Node.js 專案](app-insights-nodejs.md)
  * [每個網頁中的 JavaScript](app-insights-javascript.md) 
* 在裝置或 Web 伺服器程式碼中，加入：

    *C#:* `using Microsoft.ApplicationInsights;`

    *Visual Basic：*`Imports Microsoft.ApplicationInsights`

    *Java：* `import com.microsoft.applicationinsights.TelemetryClient;`

    *Node.js：*`var applicationInsights = require("applicationinsights");`

## <a name="get-a-telemetryclient-instance"></a>取得 TelemetryClient 執行個體

取得 `TelemetryClient` 的執行個體 (除非是在網頁的 JavaScript 中)：

*C#*

```csharp
private TelemetryClient telemetry = new TelemetryClient();
```

*Visual Basic*

```vb
Private Dim telemetry As New TelemetryClient
```

*Java*

```java
private TelemetryClient telemetry = new TelemetryClient();
``` 

*Node.js*

```javascript
var telemetry = applicationInsights.defaultClient;
```

TelemetryClient 具備執行緒安全。

針對 ASP.NET 和 Java 專案，系統會自動擷取傳入 HTTP 要求。 您可以為應用程式的其他模組建立額外的 TelemetryClient 執行個體。 比方說，您可以在中介軟體類別中配置一個 TelemetryClient 執行個體，用來報告商業邏輯事件。 您可以設定 UserId 和 DeviceId 等屬性，藉此辨識機器。 這項資訊會附加至執行個體所傳送的所有事件。 

*C#*

```csharp
TelemetryClient.Context.User.Id = "...";
TelemetryClient.Context.Device.Id = "...";
```

*Java*

```java
telemetry.getContext().getUser().setId("...");
telemetry.getContext().getDevice().setId("...");
```

在 Node.js 專案中，您可以使用 `new applicationInsights.TelemetryClient(instrumentationKey?)` 建立新的執行個體，但只有在需要從單次 `defaultClient` 隔離設定的情況下，才建議這樣做。

## <a name="trackevent"></a>TrackEvent

在 Application Insights 中，「自訂事件」是您可以在[計量瀏覽器](app-insights-metrics-explorer.md)顯示為彙總計數，以及在[診斷搜尋](app-insights-diagnostic-search.md)中顯示為個別發生點的資料點。 (它與 MVC 或其他架構的「事件」不相關。)

在您的程式碼中插入 `TrackEvent` 呼叫，以計算各種事件。 使用者選擇特定功能的頻率、達成特定目標的頻率，或他們犯特定類型錯誤的頻率。

例如，在遊戲應用程式中，每當使用者贏得遊戲時傳送事件：

*JavaScript*

```javascript
appInsights.trackEvent("WinGame");
```

*C#*

```csharp
telemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
telemetry.TrackEvent("WinGame")
```

*Java*

```java
telemetry.trackEvent("WinGame");
```

*Node.js*

```javascript
telemetry.trackEvent({name: "WinGame"});
```

### <a name="custom-events-in-analytics"></a>分析中的自訂事件

[Application Insights 分析](app-insights-analytics.md)的 `customEvents` 資料表中有提供遙測資料。 每個資料列各代表應用程式中的一個 `trackEvent(..)` 呼叫。

如果[取樣](app-insights-sampling.md)運作中，itemCount 屬性會顯示大於 1 的值。 例如，itemCount==10 表示在 trackEvent() 的 10 個呼叫中，取樣處理序只會傳輸其中一個。 若要取得正確的自訂事件計數，您應該使用程式碼，例如 `customEvent | summarize sum(itemCount)`。

## <a name="getmetric"></a>GetMetric

### <a name="examples"></a>範例

*C#*

```csharp
#pragma warning disable CA1716  // Namespace naming

namespace User.Namespace.Example01
{
    using System;
    using Microsoft.ApplicationInsights;
    using TraceSeveretyLevel = Microsoft.ApplicationInsights.DataContracts.SeverityLevel;

    /// <summary>
    /// Most simple cases are one-liners.
    /// This is all possible without even importing an additional namespace.
    /// </summary>

    public class Sample01
    {
        /// <summary />
        public static void Exec()
        {
            // *** SENDING METRICS ***

            // Recall how you send custom telemetry with Application Insights in other cases, e.g. Events.
            // The following will result in an EventTelemetry object to be sent to the cloud right away.
            TelemetryClient client = new TelemetryClient();
            client.TrackEvent("SomethingInterestingHappened");

            // Metrics work very similar. However, the value is not sent right away.
            // It is aggregated with other values for the same metric, and the resulting summary (aka "aggregate" is sent automatically every minute.
            // To mark this difference, we use a pattern that is similar, but different from the established TrackXxx(..) pattern that sends telemetry right away:

            client.GetMetric("CowsSold").TrackValue(42);

            // *** MULTI-DIMENSIONAL METRICS ***

            // The above example shows a zero-dimensional metric.
            // Metrics can also be multi-dimensional.
            // In the initial version we are supporting up to 2 dimensions, and we will add support for more in the future as needed.
            // Here is an example for a one-dimensional metric:

            Metric animalsSold = client.GetMetric("AnimalsSold", "Species");

            animalsSold.TrackValue(42, "Pigs");
            animalsSold.TrackValue(24, "Horses");

            // The values for Pigs and Horses will be aggregated separately from each other and will result in two distinct aggregates.
            // You can control the maximum number of number data series per metric (and thus your resource usage and cost).
            // The default limits are no more than 1000 total data series per metric, and no more than 100 different values per dimension.
            // We discuss elsewhere how to change them.
            // We use a common .Net pattern: TryXxx(..) to make sure that the limits are observed.
            // If the limits are already reached, Metric.TrackValue(..) will return False and the value will not be tracked. Otherwise it will return True.
            // This is particularly useful if the data for a metric originates from user input, e.g. a file:

            Tuple<int, string> countAndSpecies = ReadSpeciesFromUserInput();
            int count = countAndSpecies.Item1;
            string species = countAndSpecies.Item2;

            if (!animalsSold.TrackValue(count, species))

            {
                client.TrackTrace($"Data series or dimension cap was reached for metric {animalsSold.Identifier.MetricId}.", TraceSeveretyLevel.Error);
            }

            // You can inspect a metric object to reason about its current state. For example:
            int currentNumberOfSpecies = animalsSold.GetDimensionValues(1).Count;
        }

        private static void ResetDataStructure()
        {
            // Do stuff
        }

        private static Tuple<int, string> ReadSpeciesFromUserInput()
        {
            return Tuple.Create(18, "Cows");
        }

        private static int AddItemsToDataStructure()
        {
            // Do stuff
            return 5;
        }
    }
}
```

## <a name="trackmetric"></a>TrackMetric

> [!NOTE]
> Microsoft.ApplicationInsights.TelemetryClient.TrackMetric 已在 .NET SDK 中淘汰。 您應該一律預先彙總一段時間的計量，再加以傳送。 使用其中一個 GetMetric(..) 多載來取得可供存取 SDK 預先彙總功能的計量物件。 如果您要實作自己的預先彙總邏輯，則可使用追蹤 (ITelemetry metricTelemetry) 方法來傳送所產生的彙總。 如果您的應用程式需要每次傳送個別遙測項目 (未隨時間彙總)，則可能有事件遙測的使用案例；請參閱 TelemetryClient.TrackEvent (Microsoft.Applicationlnsights.DataContracts.EventTelemetry)。

Application Insights 可以將未附加至特定事件的計量繪製成圖表。 例如，您可以定期監視佇列長度。 當您使用計量時，個別測量的重要性就不如變化和趨勢，因此統計圖表很有用。

為了將計量傳送至 Application Insights，您可以使用 `TrackMetric(..)` API。 您有兩種方式可以傳送計量：

* 單一值： 每次在應用程式中執行一個測量，都會將對應值傳送至 Application Insights。 例如，假設您有一個描述容器中項目數的計量。 在特定期間內，您先將 3 個項目放入容器中，再移除 2 個項目。 因此，您會呼叫 `TrackMetric` 兩次：先傳遞值 `3`，再傳遞值 `-2`。 Application Insights 會代替您儲存這兩個值。

* 彙總： 使用計量時，每個單一測量並不重要。 相反地，在特定期間內發生的狀況摘要才重要。 這類摘要稱為_彙總_。 在上述範例中，該期間的彙總計量總和為 `1`，而計量值的計數為 `2`。 使用彙總方法時，您只會在每段期間叫用 `TrackMetric` 一次，並傳送彙總值。 這是建議的方法，因為它可以藉由傳送較少資料點至 Application Insights，同時仍收集所有相關資訊，來大幅降低成本和效能負擔。

### <a name="examples"></a>範例

#### <a name="single-values"></a>單一值

若要傳送單一計量值︰

*JavaScript*

 ```javascript
appInsights.trackMetric("queueLength", 42.0);
 ```

*C#*

```csharp
var sample = new MetricTelemetry();
sample.Name = "metric name";
sample.Value = 42.3;
telemetryClient.TrackMetric(sample);
```

*Java*

```java
telemetry.trackMetric("queueLength", 42.0);
```

*Node.js*

 ```javascript
telemetry.trackMetric({name: "queueLength", value: 42.0});
 ```

### <a name="custom-metrics-in-analytics"></a>Analytics 中的自訂計量

[Application Insights 分析](app-insights-analytics.md)的 `customMetrics` 資料表中有提供遙測資料。 每個資料列各代表應用程式中的一個 `trackMetric(..)` 呼叫。

* `valueSum` - 這是測量結果的總和。 若要取得平均值，請將它除以 `valueCount`。
* `valueCount` - 彙總到這個 `trackMetric(..)` 呼叫的測量數目。

## <a name="page-views"></a>頁面檢視

在裝置或網頁應用程式中，每個畫面或頁面載入時預設會傳送頁面檢視遙測。 但是，您可以變更為在其他或不同的時間追蹤頁面檢視。 例如，在顯示索引標籤或刀鋒視窗的應用程式中，您可能想要在使用者每次開啟新的刀鋒視窗時追蹤頁面。

使用者和工作階段資料會與頁面檢視一起傳送為屬性，當有頁面檢視遙測時，讓使用者與工作階段圖表顯現。

### <a name="custom-page-views"></a>自訂頁面檢視

*JavaScript*

```javascript
appInsights.trackPageView("tab1");
```

*C#*

```csharp
telemetry.TrackPageView("GameReviewPage");
```

*Visual Basic*

```vb
telemetry.TrackPageView("GameReviewPage")
```

*Java*

```java
telemetry.trackPageView("GameReviewPage");
```

如果您在不同的 HTML 網頁內有數個索引標籤，您也可以指定 URL：

```javascript
appInsights.trackPageView("tab1", "http://fabrikam.com/page1.htm");
```

### <a name="timing-page-views"></a>計時頁面檢視

根據預設，回報為**頁面檢視載入時間**的時間是測量從瀏覽器傳送要求開始，直到呼叫瀏覽器的頁面載入事件為止的時間。

相反地，您可以：

* 在 [trackPageView](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md#trackpageview) 呼叫中設定明確的持續時間：`appInsights.trackPageView("tab1", null, null, null, durationInMilliseconds);`。
* 使用頁面檢視計時呼叫 `startTrackPage` 和 `stopTrackPage`。

*JavaScript*

```javascript
// To start timing a page:
appInsights.startTrackPage("Page1");

...

// To stop timing and log the page:
appInsights.stopTrackPage("Page1", url, properties, measurements);
```

做為第一個參數的名稱會將開始和停止呼叫相關聯。 預設為目前的頁面名稱。

產生後顯示在計量瀏覽器中的頁面載入持續時間是衍生自開始和停止呼叫之間的間隔。 取決於您實際計時的間隔。

### <a name="page-telemetry-in-analytics"></a>分析中的頁面遙測

在[分析](app-insights-analytics.md)中，有兩個資料表顯示瀏覽器作業的資料：

* `pageViews` 資料表包含 URL 和網頁標題的相關資料
* `browserTimings` 資料表包含用戶端效能的相關資料，例如處理傳入資料所花費的時間

若要了解瀏覽器處理不同頁面所花費的時間：

```kusto
browserTimings
| summarize avg(networkDuration), avg(processingDuration), avg(totalDuration) by name
```

若要探索不同瀏覽器的熱門程度：

```kusto
pageViews
| summarize count() by client_Browser
```

若要將頁面檢視與 AJAX 呼叫產生關聯，請聯結相依性：

```kusto
pageViews
| join (dependencies) on operation_Id 
```

## <a name="trackrequest"></a>TrackRequest

伺服器 SDK 會使用 TrackRequest 來記錄 HTTP 要求。

如果您想要在沒有 Web 服務模組執行的內容中模擬要求，您也可以自行呼叫。

不過，建議用來傳送要求遙測的方式是在要求作為<a href="#operation-context">作業內容</a>的地方。

## <a name="operation-context"></a>作業內容

您可以使遙測項目和作業內容產生關聯，藉此讓遙測項目相互關聯。 標準的要求追蹤模組會針對在處理 HTTP 要求時傳送的例外狀況和其他事件執行此動作。 在[搜尋](app-insights-diagnostic-search.md)和[分析](app-insights-analytics.md)中，您可以使用要求的作業識別碼，輕易地找出任何與要求相關聯的事件。

如需相互關聯的詳細資訊，請參閱 [Application Insights 中的遙測相互關聯](application-insights-correlation.md)。

以手動方式追蹤遙測時，確保遙測相互關聯最簡單的方式是使用此模式：

*C#*

```csharp
// Establish an operation context and associated telemetry item:
using (var operation = telemetryClient.StartOperation<RequestTelemetry>("operationName"))
{
    // Telemetry sent in here will use the same operation ID.
    ...
    telemetryClient.TrackTrace(...); // or other Track* calls
    ...

    // Set properties of containing telemetry item--for example:
    operation.Telemetry.ResponseCode = "200";

    // Optional: explicitly send telemetry item:
    telemetryClient.StopOperation(operation);

} // When operation is disposed, telemetry item is sent.
```

在設定作業內容時，`StartOperation` 會建立所指定類型的遙測項目。 它會在您處置作業時或您明確地呼叫 `StopOperation` 時傳送遙測項目。 如果您使用 `RequestTelemetry` 做為遙測類型，則其持續時間會設定為開始與停止之間的時間間隔。

在作業範圍內回報的遙測項目會變成這類作業的「子項目」。 作業內容可以是巢狀。

在搜尋中，會使用作業內容來建立 [相關項目] 清單：

![相關項目](./media/app-insights-api-custom-events-metrics/21.png)

如需有關自訂作業追蹤的詳細資訊，請參閱[使用 Application Insights .NET SDK 追蹤自訂作業](application-insights-custom-operations-tracking.md)。

### <a name="requests-in-analytics"></a>分析中的要求

在 [Application Insights 分析](app-insights-analytics.md)中，要求會顯示在 `requests` 資料表中。

如果[取樣](app-insights-sampling.md)運作中，itemCount 屬性將會顯示大於 1 的值。 例如，itemCount==10 表示在 trackRequest() 的 10 個呼叫中，取樣處理序只會傳輸其中一個。 若要取得依要求名稱分割的正確要求計數和平均持續時間，請使用類似如下的程式碼：

```kusto
requests
| summarize count = sum(itemCount), avgduration = avg(duration) by name
```

## <a name="trackexception"></a>TrackException

傳送例外狀況至 Application Insights︰

* 以[計算數目](app-insights-metrics-explorer.md)，來指出問題的頻率。
* 以[檢查個別出現次數](app-insights-diagnostic-search.md)。

報告包含堆疊追蹤。

*C#*

```csharp
try
{
    ...
}
catch (Exception ex)
{
    telemetry.TrackException(ex);
}
```

*Java*

```java
try {
    ...
} catch (Exception ex) {
    telemetry.trackException(ex);
}
```

*JavaScript*

```javascript
try
{
    ...
}
catch (ex)
{
    appInsights.trackException(ex);
}
```

*Node.js*

```javascript
try
{
    ...
}
catch (ex)
{
    telemetry.trackException({exception: ex});
}
```

SDK 將自動攔截許多例外狀況，所以您不一定需要明確呼叫 TrackException。

* ASP.NET：[撰寫程式碼以攔截例外狀況](app-insights-asp-net-exceptions.md)。
* J2EE：[自動攔截例外狀況](app-insights-java-get-started.md#exceptions-and-request-failures)。
* JavaScript：自動攔截例外狀況。 如果您想要停用自動收集，請在您插入網頁的程式碼片段中加入一行：

```javascript
({
    instrumentationKey: "your key",
    disableExceptionTracking: true
})
```

### <a name="exceptions-in-analytics"></a>分析中的例外狀況

在 [Application Insights 分析](app-insights-analytics.md)中，例外狀況會顯示在 `exceptions` 資料表中。

如果[取樣](app-insights-sampling.md)運作中，`itemCount` 屬性會顯示大於 1 的值。 例如，itemCount==10 表示在 trackException() 的 10 個呼叫中，取樣處理序只會傳輸其中一個。 若要取得依例外狀況類型分割的正確例外狀況計數，請使用類似如下的程式碼：

```kusto
exceptions
| summarize sum(itemCount) by type
```

大多數重要堆疊資訊已擷取到不同的變數中，但您可以拉開 `details` 結構以取得更多資訊。 由於這是動態結構，因此您應該將結果轉換成預期的類型。 例如︰

```kusto
exceptions
| extend method2 = tostring(details[0].parsedStack[1].method)
```

若要將例外狀況與其相關要求產生關聯，請使用聯結：

```kusto
exceptions
| join (requests) on operation_Id
```

## <a name="tracktrace"></a>TrackTrace

使用 TrackTrace 可協助您藉由將 "breadcrumb trail" 傳送至 Application Insights 來診斷問題。 您可以傳送診斷資料區塊，並且在[診斷搜尋](app-insights-diagnostic-search.md)中檢查。

在 .NET 中，[記錄配接器](app-insights-asp-net-trace-logs.md)使用此 API 將第三方記錄傳送至入口網站。

在 Java 中，[Log4J、Logback 等標準記錄器](app-insights-java-trace-logs.md)使用 Application Insights Log4j 或 Logback 附加器將第三方記錄傳送至入口網站。

*C#*

```csharp
telemetry.TrackTrace(message, SeverityLevel.Warning, properties);
```

*Java*

```java
telemetry.trackTrace(message, SeverityLevel.Warning, properties);
```

*Node.js*

```javascript
telemetry.trackTrace({
    message: message,
    severity: applicationInsights.Contracts.SeverityLevel.Warning,
    properties: properties
});
```

您可以搜尋訊息內容，但是 (不同於屬性值) 您無法在其中進行篩選。

`message` 上的大小限制比屬性上的限制高得多。
TrackTrace 的優點在於您可以將較長的資料放在訊息中。 例如，您可以在該處編碼 POST 資料。  

此外，您可以在訊息中新增嚴重性層級。 就像其他遙測一樣，您可以新增屬性值以供協助篩選或搜尋不同的追蹤集。 例如︰

*C#*

```csharp
var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
telemetry.TrackTrace("Slow database response",
                SeverityLevel.Warning,
                new Dictionary<string,string> { {"database", db.ID} });
```

*Java*

```java
Map<String, Integer> properties = new HashMap<>();
properties.put("Database", db.ID);
telemetry.trackTrace("Slow Database response", SeverityLevel.Warning, properties);
```

在[搜尋](app-insights-diagnostic-search.md)中，您便可以輕鬆地篩選出與特定資料庫相關且具有特定嚴重性層級的所有訊息。

### <a name="traces-in-analytics"></a>分析中的追蹤

在 [Application Insights 分析](app-insights-analytics.md)中，TrackTrace 的呼叫會顯示在 `traces` 資料表中。

如果[取樣](app-insights-sampling.md)運作中，itemCount 屬性會顯示大於 1 的值。 例如，itemCount==10 表示在 `trackTrace()` 的 10 個呼叫中，取樣處理序只會傳輸其中一個。 若要取得正確的追蹤呼叫計數，您應該使用程式碼，例如 `traces | summarize sum(itemCount)`。

## <a name="trackdependency"></a>TrackDependency

您可以使用 TrackDependency 呼叫來追蹤回應時間以及呼叫外部程式碼片段的成功率。 結果會出現在入口網站中的相依性圖表中。

*C#*

```csharp
var success = false;
var startTime = DateTime.UtcNow;
var timer = System.Diagnostics.Stopwatch.StartNew();
try
{
    success = dependency.Call();
}
finally
{
    timer.Stop();
    telemetry.TrackDependency("myDependency", "myCall", startTime, timer.Elapsed, success);
     // The call above has been made obsolete in the latest SDK. The updated call follows this format:
     // TrackDependency (string dependencyTypeName, string dependencyName, string data, DateTimeOffset startTime, TimeSpan duration, bool success);
}
```

*Java*

```java
boolean success = false;
long startTime = System.currentTimeMillis();
try {
    success = dependency.call();
}
finally {
    long endTime = System.currentTimeMillis();
    long delta = endTime - startTime;
    RemoteDependencyTelemetry dependencyTelemetry = new RemoteDependencyTelemetry("My Dependency", "myCall", delta, success);
    telemetry.setTimeStamp(startTime);
    telemetry.trackDependency(dependencyTelemetry);
}
```

*JavaScript*

```javascript
var success = false;
var startTime = new Date().getTime();
try
{
    success = dependency.Call();
}
finally
{
    var elapsed = new Date() - startTime;
    telemetry.trackDependency({
        dependencyTypeName: "myDependency",
        name: "myCall",
        duration: elapsed,
        success: success
    });
}
```

請記住，伺服器 SDK 包含[相依性模組](app-insights-asp-net-dependencies.md)，可用來自動探索和追蹤特定相依性呼叫 (例如資料庫和 REST API)。 您必須在伺服器上安裝代理程式才能讓模組正常運作。 

在 Java 中，您可以使用 [Java 代理程式](app-insights-java-agent.md)追蹤某些相依性呼叫。

如果您想要追蹤自動化追蹤不會攔截的呼叫，或不想安裝代理程式，您可以使用這個呼叫。

若要在 C# 中關閉標準相依性追蹤模組，請編輯 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 並刪除 `DependencyCollector.DependencyTrackingTelemetryModule` 的參考。 在 Java 中，如果您不想要自動收集標準相依性，請勿安裝 Java 用戶端。

### <a name="dependencies-in-analytics"></a>分析中的相依性

在 [Application Insights 分析](app-insights-analytics.md)中，trackDependency 呼叫會顯示在 `dependencies` 資料表中。

如果[取樣](app-insights-sampling.md)運作中，itemCount 屬性會顯示大於 1 的值。 例如，itemCount==10 表示在 trackDependency() 的 10 個呼叫中，取樣處理序只會傳輸其中一個。 若要取得依目標元件分割的正確相依性計數，請使用類似如下的程式碼：

```kusto
dependencies
| summarize sum(itemCount) by target
```

若要將相依性與其相關要求產生關聯，請使用聯結：

```kusto
dependencies
| join (requests) on operation_Id
```

## <a name="flushing-data"></a>排清資料

通常 SDK 會在選擇的時間傳送資料以將對使用者的影響降到最低。 不過，在某些情況下您可能想要排清緩衝區，例如，如果您在會關閉的應用程式中使用 SDK。

*C#*

 ```csharp
telemetry.Flush();
// Allow some time for flushing before shutdown.
System.Threading.Thread.Sleep(5000);
```

*Java*

```java
telemetry.flush();
//Allow some time for flushing before shutting down
Thread.sleep(5000);
```

*Node.js*

```javascript
telemetry.flush();
```

請注意，此函式對[伺服器遙測通道](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/)而言是非同步的。

在理想情況下，flush() 方法應該用於應用程式的關閉活動。

## <a name="authenticated-users"></a>驗證的使用者

在 Web 應用程式中，預設是透過 Cookie 來識別使用者。 如果使用者從不同的電腦或瀏覽器存取您的 app 或刪除 Cookie，就可能多次計算他們。

如果使用者登入您的 app，您可以藉由在瀏覽器程式碼中設定經過驗證的使用者識別碼，來取得更正確的計數：

*JavaScript*

```javascript
// Called when my app has identified the user.
function Authenticated(signInId) {
    var validatedId = signInId.replace(/[,;=| ]+/g, "_");
    appInsights.setAuthenticatedUserContext(validatedId);
    ...
}
```

在 ASP.NET Web MVC 應用程式，例如：

*Razor*

```cshtml
@if (Request.IsAuthenticated)
{
    <script>
        appInsights.setAuthenticatedUserContext("@User.Identity.Name
            .Replace("\\", "\\\\")"
            .replace(/[,;=| ]+/g, "_"));
    </script>
}
```

這不需要用到使用者的實際登入名稱。 只需是使用者的唯一識別碼。 其中不能包含空格或任何 `,;=|` 字元。

使用者識別碼也會設定於工作階段 Cookie 中，並傳送到伺服器。 如果安裝了伺服器 SDK，則會傳送經過驗證的使用者識別碼以作為用戶端和伺服器遙測的內容屬性一部分。 您可以接著對它進行篩選和搜尋。

如果您的 app 會將使用者群組為帳戶，您也可以傳遞該帳戶的識別碼 (具有相同的字元限制)。

```javascript
appInsights.setAuthenticatedUserContext(validatedId, accountId);
```

在[計量瀏覽器](app-insights-metrics-explorer.md)中，您可建立可計算 [已驗證的使用者] 和 [使用者帳戶] 的圖表。

您也可以[搜尋](app-insights-diagnostic-search.md)具有特定使用者名稱和帳戶的用戶端資料點。

## <a name="properties"></a>使用屬性篩選、搜尋和分割資料

您可以將屬性和測量結果附加至您的事件 (同時還有度量，頁面檢視、例外狀況和其他的遙測資料)。

*屬性* 是可在使用情況報告中用來篩選遙測的字串值。 例如，如果您的應用程式提供數個遊戲，則您可以將遊戲的名稱附加至每個事件，以了解哪些遊戲較受歡迎。

字串長度限制為 8192 個。 (如果您想要傳送大量的資料區塊，請使用訊息參數 [TrackTrace](#track-trace)。)

*度量* 是可以用圖表方式呈現的數值。 例如，您可能想要查看玩家達到的分數是否逐漸增加。 圖表可以依據隨事件傳送的屬性分割，讓您可以針對不同遊戲取得個別或堆疊圖表。

若要讓計量值正確顯示，這些值應該大於或等於 0。

有一些 [屬性、屬性值和度量的數目限制](#limits) 可供您使用。

*JavaScript*

```javascript
appInsights.trackEvent
    ("WinGame",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );

appInsights.trackPageView
    ("page name", "http://fabrikam.com/pageurl.html",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );
```

*C#*

```csharp
// Set up some properties and metrics:
var properties = new Dictionary <string, string>
    {{"game", currentGame.Name}, {"difficulty", currentGame.Difficulty}};
var metrics = new Dictionary <string, double>
    {{"Score", currentGame.Score}, {"Opponents", currentGame.OpponentCount}};

// Send the event:
telemetry.TrackEvent("WinGame", properties, metrics);
```

*Node.js*

```javascript
// Set up some properties and metrics:
var properties = {"game": currentGame.Name, "difficulty": currentGame.Difficulty};
var metrics = {"Score": currentGame.Score, "Opponents": currentGame.OpponentCount};

// Send the event:
telemetry.trackEvent({name: "WinGame", properties: properties, measurements: metrics});
```

*Visual Basic*

```vb
' Set up some properties:
Dim properties = New Dictionary (Of String, String)
properties.Add("game", currentGame.Name)
properties.Add("difficulty", currentGame.Difficulty)

Dim metrics = New Dictionary (Of String, Double)
metrics.Add("Score", currentGame.Score)
metrics.Add("Opponents", currentGame.OpponentCount)

' Send the event:
telemetry.TrackEvent("WinGame", properties, metrics)
```

*Java*

```java
Map<String, String> properties = new HashMap<String, String>();
properties.put("game", currentGame.getName());
properties.put("difficulty", currentGame.getDifficulty());

Map<String, Double> metrics = new HashMap<String, Double>();
metrics.put("Score", currentGame.getScore());
metrics.put("Opponents", currentGame.getOpponentCount());

telemetry.trackEvent("WinGame", properties, metrics);
```

> [!NOTE]
> 切勿在屬性中記錄個人識別資訊。
>
>

### <a name="alternative-way-to-set-properties-and-metrics"></a>設定屬性和度量的替代方式

如果更加方便，您可以收集個別物件中事件的參數：

```csharp
var event = new EventTelemetry();

event.Name = "WinGame";
event.Metrics["processingTime"] = stopwatch.Elapsed.TotalMilliseconds;
event.Properties["game"] = currentGame.Name;
event.Properties["difficulty"] = currentGame.Difficulty;
event.Metrics["Score"] = currentGame.Score;
event.Metrics["Opponents"] = currentGame.Opponents.Length;

telemetry.TrackEvent(event);
```

> [!WARNING]
> 不要重複使用相同的遙測項目執行個體 (此範例中的 `event`) 來呼叫 Track*() 多次。 這可能會讓遙測隨著不正確的組態傳送。
>
>

### <a name="custom-measurements-and-properties-in-analytics"></a>分析中的自訂測量和屬性

在[分析](app-insights-analytics.md)中，自訂計量和屬性會顯示在每個遙測記錄的 `customMeasurements` 和 `customDimensions` 屬性中。

例如，如果您將一個名為 "game" 的屬性新增至您的要求遙測，此查詢將會計算不同 "game" 值出現的次數，並顯示自訂計量 "score" 的平均值：

```kusto
requests
| summarize sum(itemCount), avg(todouble(customMeasurements.score)) by tostring(customDimensions.game)
```

請注意：

* 當您從 customDimensions 或 customMeasurements JSON 中擷取值時，其類型為動態，因此您必須將它轉換成 `tostring` 或 `todouble`。
* 考量到[取樣](app-insights-sampling.md)的可能性，您應該使用 `sum(itemCount)`，而不是 `count()`。

## <a name="timed"></a> 計時事件

有時候您想要繪製執行某些動作耗費多少時間的圖表。 例如，您可能想要知道使用者在遊戲中思考選項時花費多少時間。 您可以對此使用測量參數。

*C#*

```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();

// ... perform the timed action ...

stopwatch.Stop();

var metrics = new Dictionary <string, double>
    {{"processingTime", stopwatch.Elapsed.TotalMilliseconds}};

// Set up some properties:
var properties = new Dictionary <string, string>
    {{"signalSource", currentSignalSource.Name}};

// Send the event:
telemetry.TrackEvent("SignalProcessed", properties, metrics);
```

*Java*

```java
long startTime = System.currentTimeMillis();

// Perform timed action

long endTime = System.currentTimeMillis();
Map<String, Double> metrics = new HashMap<>();
metrics.put("ProcessingTime", endTime-startTime);

// Setup some properties
Map<String, String> properties = new HashMap<>();
properties.put("signalSource", currentSignalSource.getName());

// Send the event
telemetry.trackEvent("SignalProcessed", properties, metrics);
```

## <a name="defaults"></a>自訂遙測資料的預設屬性

如果您想為您撰寫的一些自訂事件設定預設屬性值，您可以在 TelemetryClient 執行個體中設定它們。 它們會附加至從該用戶端傳送的每個遙測項目。

*C#*

```csharp
using Microsoft.ApplicationInsights.DataContracts;

var gameTelemetry = new TelemetryClient();
gameTelemetry.Context.Properties["Game"] = currentGame.Name;
// Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
Dim gameTelemetry = New TelemetryClient()
gameTelemetry.Context.Properties("Game") = currentGame.Name
' Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame")
```

*Java*

```java
import com.microsoft.applicationinsights.TelemetryClient;
import com.microsoft.applicationinsights.TelemetryContext;
...


TelemetryClient gameTelemetry = new TelemetryClient();
TelemetryContext context = gameTelemetry.getContext();
context.getProperties().put("Game", currentGame.Name);

gameTelemetry.TrackEvent("WinGame");
```

*Node.js*

```javascript
var gameTelemetry = new applicationInsights.TelemetryClient();
gameTelemetry.commonProperties["Game"] = currentGame.Name;

gameTelemetry.TrackEvent({name: "WinGame"});
```

個別遙測呼叫可以覆寫其屬性字典中的預設值。

*針對 JavaScript Web 用戶端*， [請使用 JavaScript 遙測初始設定式](#js-initializer)。

*若要將屬性新增至所有遙測*，並包括來自標準集合模組的資料，請[實作 `ITelemetryInitializer`](app-insights-api-filtering-sampling.md#add-properties)。

## <a name="sampling-filtering-and-processing-telemetry"></a>取樣、篩選及處理遙測資料

您可以撰寫程式碼，在從 SDK 傳送遙測資料前加以處理。 處理包括從標準遙測模組 (如 HTTP 要求收集和相依性收集) 的資料。

實作 `ITelemetryInitializer` 以[屬性](app-insights-api-filtering-sampling.md#add-properties)至遙測資料。 例如，您可以新增版本號碼或從其他屬性計算得出的值。

[篩選](app-insights-api-filtering-sampling.md#filtering)可以先修改或捨棄遙測，再藉由實作 `ITelemetryProcesor` 從 SDK 傳送遙測。 您可控制要傳送或捨棄的項目，但是您必須考量這對您的度量的影響。 視您捨棄項目的方式而定，您可能會喪失在相關項目之間瀏覽的能力。

[取樣](app-insights-api-filtering-sampling.md)是減少從應用程式傳送至入口網站的資料量的套件方案。 它在這麼做時並不會影響顯示的度量。 而且它在這麼做時可藉由在相關項目 (如例外狀況、要求和頁面檢視) 之間瀏覽，而不會影響您診斷問題的能力。

[深入了解](app-insights-api-filtering-sampling.md)。

## <a name="disabling-telemetry"></a>停用遙測

*動態停止和開始* 收集及傳輸遙測資料：

*C#*

```csharp
using  Microsoft.ApplicationInsights.Extensibility;

TelemetryConfiguration.Active.DisableTelemetry = true;
```

*Java*

```java
telemetry.getConfiguration().setTrackingDisabled(true);
```

若要*停用選取的標準收集器* (例如效能計數器、HTTP 要求或相依性)，請刪除或註解化 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 中的相關行。例如，如果您想要傳送自己的 TrackRequest 資料，可以這麼做。

*Node.js*

```javascript
telemetry.config.disableAppInsights = true;
```

若要在初始化時「停用選取的標準收集器」(例如，效能計數器、HTTP 要求或相依性)，請將設定方法鏈結至您的 SDK 初始化程式碼：

```javascript
applicationInsights.setup()
    .setAutoCollectRequests(false)
    .setAutoCollectPerformance(false)
    .setAutoCollectExceptions(false)
    .setAutoCollectDependencies(false)
    .setAutoCollectConsole(false)
    .start();
```

若要在初始化之後停用這些收集器，請使用 Configuration 物件：`applicationInsights.Configuration.setAutoCollectRequests(false)`

## <a name="debug"></a>開發人員模式

偵錯期間，讓您的遙測透過管線加速很有用，如此您就可以立即看到結果。 您也會取得額外的訊息，協助您追蹤任何遙測的問題。 在生產環境中將它關閉，因為它可能會拖慢您的應用程式。

*C#*

```csharp
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = true;
```

*Visual Basic*

```vb
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = True
```

## <a name="ikey"></a>設定已選取自訂遙測的檢測金鑰

*C#*

```csharp
var telemetry = new TelemetryClient();
telemetry.InstrumentationKey = "---my key---";
// ...
```

## <a name="dynamic-ikey"></a> 動態檢測金鑰

若要避免混合來自開發、測試和實際執行環境的遙測，您可以[建立個別的 Application Insights 資源](app-insights-create-new-resource.md)，並且依據環境變更其金鑰。

而不是從組態檔取得檢測金鑰，您可以在程式碼中設定。 在初始化方法中設定金鑰，例如 ASP.NET 服務中的 global.aspx.cs：

*C#*

```csharp
protected void Application_Start()
{
    Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey =
        // - for example -
        WebConfigurationManager.Settings["ikey"];
    ...
}
```

*JavaScript*

```javascript
appInsights.config.instrumentationKey = myKey;
```

在網頁中，您可能想要從 Web 伺服器的狀態設定，而不是按其原義編碼至指令碼。 例如，在 ASP.NET 應用程式中產生的網頁：

*Razor 中的 JavaScript*

```cshtml
<script type="text/javascript">
// Standard Application Insights webpage script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
    // Generate from server property:
    @Microsoft.ApplicationInsights.Extensibility.
        TelemetryConfiguration.Active.InstrumentationKey;
}) // ...
```

## <a name="telemetrycontext"></a>TelemetryContext

TelemetryClient 具有內容屬性，其中包含與所有遙測資料一起傳送的值。 它們通常由標準遙測模組設定，但是您也可以自行設定它們。 例如︰

```csharp
telemetry.Context.Operation.Name = "MyOperationName";
```

如果您自行設定這些值，請考慮從 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 移除相關的程式碼行，讓您的值和標準值不致混淆。

* **元件**：應用程式及其版本。
* **裝置**︰應用程式執行所在裝置的相關資料  (在 Web 應用程式中，這是傳送遙測的伺服器或用戶端裝置)。
* **InstrumentationKey**：Azure 中遙測顯示之位置的 Application Insights 資源。 通常會揀選自 ApplicationInsights.config。
* **位置**：裝置的地理位置。
* **作業**：在 Web 應用程式中，目前的 HTTP 要求。 在其他應用程式類型中，您可以設定以將事件群組在一起。
  * **識別碼**：產生的值，與不同事件相互關聯，如此當您在診斷搜尋中檢查任何事件時，您可以發現相關項目。
  * **名稱**：識別碼，通常是 HTTP 要求的 URL。
  * **SyntheticSource**：如果不為 null 或空白，這個字串表示要求的來源已被識別為傀儡程式或 Web 測試。 根據預設，會從計量瀏覽器的計算中排除。
* **屬性**：與所有遙測資料一起傳送的屬性。 可以在個別 Track* 呼叫中覆寫。
* **工作階段**︰使用者的工作階段。 識別碼會設為產生的值，當使用者一段時間沒有作用時會變更。
* **使用者**：使用者資訊。

## <a name="limits"></a>限制

[!INCLUDE [application-insights-limits](../../includes/application-insights-limits.md)]

若要避免達到資料速率限制，請使用[取樣](app-insights-sampling.md)。

若要決定資料的保留時間，請參閱[資料保留和隱私權](app-insights-data-retention-privacy.md)。

## <a name="reference-docs"></a>參考文件

* [ASP.NET 參考](https://msdn.microsoft.com/library/dn817570.aspx)
* [Java 參考](http://dl.windowsazure.com/applicationinsights/javadoc/)
* [JavaScript 參考](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md)
* [Android SDK](https://github.com/Microsoft/ApplicationInsights-Android)
* [iOS SDK](https://github.com/Microsoft/ApplicationInsights-iOS)

## <a name="sdk-code"></a>SDK 程式碼

* [ASP.NET 核心 SDK](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [ASP.NET 5](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Windows Server 套件](https://github.com/Microsoft/applicationInsights-dotnet-server)
* [Java SDK](https://github.com/Microsoft/ApplicationInsights-Java)
* [Node.js SDK](https://github.com/Microsoft/ApplicationInsights-Node.js)
* [JavaScript SDK](https://github.com/Microsoft/ApplicationInsights-JS)
* [所有平台](https://github.com/Microsoft?utf8=%E2%9C%93&query=applicationInsights)

## <a name="questions"></a>問題

* *Track_() 呼叫會擲回什麼例外狀況？*

    無。 您不需要將它們包裝在 try-catch 子句中。 如果 SDK 發生問題，它會在偵錯主控台輸出以及 (若訊息得以傳輸過去) 診斷搜尋中記錄訊息。
* *是否有 REST API 可供從入口網站中取得資料？*

    是，[資料存取 API](https://dev.applicationinsights.io/)。 其他擷取資料的方法包括[從分析匯出至 Power BI](app-insights-export-power-bi.md) 和[連續匯出](app-insights-export-telemetry.md)。

## <a name="next"></a>接續步驟

* [搜尋事件和記錄](app-insights-diagnostic-search.md)
* [疑難排解](app-insights-troubleshoot-faq.md)
