---
title: 在 Application Insights SDK 中進行篩選和前置處理 |Microsoft Docs
description: 針對 SDK 撰寫遙測處理器和遙測初始化運算式，以便在遙測傳送至 Application Insights 入口網站之前，篩選或將屬性新增至資料。
ms.topic: conceptual
ms.date: 11/23/2016
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: cb9159b98b219c6fb04beb7bbbaade64fc72a30b
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98927757"
---
# <a name="filter-and-preprocess-telemetry-in-the-application-insights-sdk"></a>在 Application Insights SDK 中篩選及前置處理遙測

您可以撰寫並設定 Application Insights SDK 的外掛程式，以自訂如何擴充和處理遙測，然後再將其傳送到 Application Insights 服務。

* [取樣](sampling.md) 可減少遙測的量而不會影響統計資料。 它會將相關的資料點保持在一起，以便您在診斷問題時，可以在這些資料點之間進行導覽。 在入口網站中將乘以總計數，以補償取樣。
* 使用遙測處理器進行篩選，可讓您在 SDK 傳送到伺服器之前，先篩選出 SDK 中的遙測。 例如，您可以從傀儡程式中排除要求來減少遙測量。 篩選是減少流量比取樣更基本的方法。 它可讓您更充分掌控所傳輸的內容，但它會影響您的統計資料。 例如，您可以篩選出所有成功的要求。
* [遙測初始化運算式會將屬性新增或修改](#add-properties) 至從您的應用程式傳送的任何遙測資料，其中包括來自標準模組的遙測。 例如，您可以在入口網站中加入用來篩選資料的計算值或版本號碼。
* [SDK API](./api-custom-events-metrics.md) 可用來傳送自訂事件和計量。

在開始之前：

* 為您的應用程式安裝適當的 SDK： [ASP.NET](asp-net.md)、 [ASP.NET Core](asp-net-core.md)、 [適用于 .Net/.net Core 的非 HTTP/Worker](worker-service.md)或 [JavaScript](javascript.md)。

<a name="filtering"></a>

## <a name="filtering"></a>篩選

這項技術可讓您直接掌控遙測串流中包含或排除的內容。 您可以使用篩選，將遙測專案傳送至 Application Insights。 您可以搭配取樣或個別使用篩選。

若要篩選遙測，您可以撰寫遙測處理器並向註冊 `TelemetryConfiguration` 。 所有遙測都會通過您的處理器。 您可以選擇從資料流程卸載，或將它提供給鏈中的下一個處理器。 系統會包含標準模組（例如 HTTP 要求收集器和相依性收集器）的遙測，以及您自行追蹤的遙測。 例如，您可以篩選出來自機器人的要求或成功的相依性呼叫的遙測。

> [!WARNING]
> 使用處理器篩選從 SDK 傳送的遙測可以扭曲您在入口網站中看到的統計資料，並使其難以遵循相關專案。
>
> 請考慮改用 [取樣](./sampling.md)。
>
>

### <a name="create-a-telemetry-processor-c"></a>建立遙測處理器 (C#)

1. 若要建立篩選準則，請執行 `ITelemetryProcessor` 。

    遙測處理器會建造一連串的處理。 當您具現化遙測處理器時，系統會參考鏈中的下一個處理器。 當遙測資料點傳遞至 process 方法時，它會執行其工作，然後呼叫 (或不會) 鏈中的下一個遙測處理器呼叫。

    ```csharp
    using Microsoft.ApplicationInsights.Channel;
    using Microsoft.ApplicationInsights.Extensibility;

    public class SuccessfulDependencyFilter : ITelemetryProcessor
    {
        private ITelemetryProcessor Next { get; set; }

        // next will point to the next TelemetryProcessor in the chain.
        public SuccessfulDependencyFilter(ITelemetryProcessor next)
        {
            this.Next = next;
        }

        public void Process(ITelemetry item)
        {
            // To filter out an item, return without calling the next processor.
            if (!OKtoSend(item)) { return; }

            this.Next.Process(item);
        }

        // Example: replace with your own criteria.
        private bool OKtoSend (ITelemetry item)
        {
            var dependency = item as DependencyTelemetry;
            if (dependency == null) return true;

            return dependency.Success != true;
        }
    }
    ```

2. 新增您的處理器。

ASP.NET **應用程式**

將此程式碼片段插入 ApplicationInsights.config：

```xml
<TelemetryProcessors>
  <Add Type="WebApplication9.SuccessfulDependencyFilter, WebApplication9">
     <!-- Set public property -->
     <MyParamFromConfigFile>2-beta</MyParamFromConfigFile>
  </Add>
</TelemetryProcessors>
```

您可以在類別中提供公開具名屬性，以從 .config 檔案傳遞字串值。

> [!WARNING]
> 仔細地將 .config 檔案中的類型名稱和任何屬性名稱與程式碼中的類別和屬性名稱做比對。 如果 .config 檔案參考不存在的類型或屬性，SDK 可能會以無訊息方式傳送任何遙測。
>

或者， 您也可以在程式碼中初始化篩選。 在適當的初始化類別中（例如，在中 AppStart `Global.asax.cs` ），將您的處理器插入鏈中：

```csharp
var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
builder.Use((next) => new SuccessfulDependencyFilter(next));

// If you have more processors:
builder.Use((next) => new AnotherProcessor(next));

builder.Build();
```

在此時間點之後建立的遙測用戶端將會使用您的處理器。

ASP.NET **核心/背景工作服務應用程式**

> [!NOTE]
> 使用或加入處理器 `ApplicationInsights.config` `TelemetryConfiguration.Active` 對 ASP.NET Core 的應用程式而言是不正確，或者如果您使用的是 ApplicationInsights. WorkerService SDK。

針對使用 [ASP.NET Core](asp-net-core.md#adding-telemetry-processors) 或 [WorkerService](worker-service.md#adding-telemetry-processors)撰寫的應用程式，您可以在上使用擴充方法來加入新的遙測處理器 `AddApplicationInsightsTelemetryProcessor` `IServiceCollection` ，如下所示。 這個方法會在 `ConfigureServices` 您類別的方法中呼叫 `Startup.cs` 。

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...
        services.AddApplicationInsightsTelemetry();
        services.AddApplicationInsightsTelemetryProcessor<SuccessfulDependencyFilter>();

        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<AnotherProcessor>();
    }
```

### <a name="example-filters"></a>範例篩選器

#### <a name="synthetic-requests"></a>綜合要求

篩選出 bot 和 Web 測試。 雖然計量瀏覽器可提供您篩選出綜合來源的選項，但此選項會在 SDK 本身進行篩選，以減少流量和內嵌大小。

```csharp
public void Process(ITelemetry item)
{
  if (!string.IsNullOrEmpty(item.Context.Operation.SyntheticSource)) {return;}

  // Send everything else:
  this.Next.Process(item);
}
```

#### <a name="failed-authentication"></a>驗證失敗

篩選出具有 "401" 回應的要求。

```csharp
public void Process(ITelemetry item)
{
    var request = item as RequestTelemetry;

    if (request != null &&
    request.ResponseCode.Equals("401", StringComparison.OrdinalIgnoreCase))
    {
        // To filter out an item, return without calling the next processor.
        return;
    }

    // Send everything else
    this.Next.Process(item);
}
```

#### <a name="filter-out-fast-remote-dependency-calls"></a>篩選出快速遠端相依性呼叫

如果您只想要診斷速度很慢的呼叫，請篩選出快速的呼叫。

> [!NOTE]
> 這種篩選會扭曲您在入口網站上看到的統計資料。
>
>

```csharp
public void Process(ITelemetry item)
{
    var request = item as DependencyTelemetry;

    if (request != null && request.Duration.TotalMilliseconds < 100)
    {
        return;
    }
    this.Next.Process(item);
}
```

#### <a name="diagnose-dependency-issues"></a>診斷相依性問題

[這篇部落格文章](https://azure.microsoft.com/blog/implement-an-application-insights-telemetry-processor/) 描述可自動傳送定期的 Ping 給相依項目，藉以診斷相依性問題的專案。

<a name="add-properties"></a>

### <a name="javascript-web-applications"></a>JavaScript web 應用程式

**使用 ITelemetryInitializer 進行篩選**

1. 建立遙測初始化運算式回呼函數。 回呼函式會採用 `ITelemetryItem` 做為參數，也就是正在處理的事件。 `false`從這個回呼傳回會導致要篩選掉的遙測專案。

   ```JS
   var filteringFunction = (envelope) => {
     if (envelope.data.someField === 'tobefilteredout') {
         return false;
     }
  
     return true;
   };
   ```

2. 新增遙測初始化運算式回呼：

   ```JS
   appInsights.addTelemetryInitializer(filteringFunction);
   ```

## <a name="addmodify-properties-itelemetryinitializer"></a>新增/修改屬性： ITelemetryInitializer

使用遙測初始化運算式以額外資訊擴充遙測，或覆寫標準遙測模組所設定的遙測屬性。

例如，web 套件的 Application Insights 會收集有關 HTTP 要求的遙測。 依預設，它會將回應碼為「失敗」的任何要求標示為「失敗」 >= 400。 但是，如果您想要將400視為成功，您可以提供設定 success 屬性的遙測初始化運算式。

如果您提供遙測初始化運算式，則會在呼叫任何 Track * ( # A1 方法時呼叫它。 這包括 `Track()` 標準遙測模組所呼叫的方法。 依照慣例，這些模組不會設定任何已由初始化運算式設定的屬性。 遙測初始化運算式會在呼叫遙測處理器之前呼叫。 因此處理器可以看到初始化運算式所完成的任何擴充。

**定義您的初始設定式**

*C#*

```csharp
using System;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MvcWebRole.Telemetry
{
  /*
   * Custom TelemetryInitializer that overrides the default SDK
   * behavior of treating response codes >= 400 as failed requests
   *
   */
  public class MyTelemetryInitializer : ITelemetryInitializer
  {
    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        // Is this a TrackRequest() ?
        if (requestTelemetry == null) return;
        int code;
        bool parsed = Int32.TryParse(requestTelemetry.ResponseCode, out code);
        if (!parsed) return;
        if (code >= 400 && code < 500)
        {
            // If we set the Success property, the SDK won't change it:
            requestTelemetry.Success = true;

            // Allow us to filter these requests in the portal:
            requestTelemetry.Properties["Overridden400s"] = "true";
        }
        // else leave the SDK to set the Success property
    }
  }
}
```

ASP.NET **apps：載入您的初始化運算式**

在 ApplicationInsights.config 中：

```xml
<ApplicationInsights>
  <TelemetryInitializers>
    <!-- Fully qualified type name, assembly name: -->
    <Add Type="MvcWebRole.Telemetry.MyTelemetryInitializer, MvcWebRole"/>
    ...
  </TelemetryInitializers>
</ApplicationInsights>
```

或者，您可以在程式碼中具現化初始化運算式，例如，在 Global.aspx.cs 中：

```csharp
protected void Application_Start()
{
    // ...
    TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
}
```

請參閱 [此範例](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole)的詳細資訊。

ASP.NET **核心/背景工作服務應用程式：載入您的初始化運算式**

> [!NOTE]
> 使用或加入初始化運算式 `ApplicationInsights.config` `TelemetryConfiguration.Active` 對於 ASP.NET Core 應用程式或使用 ApplicationInsights. WorkerService SDK 是不正確。

針對使用 [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers) 或 [WorkerService](worker-service.md#adding-telemetryinitializers)撰寫的應用程式，新增遙測初始化運算式的方式是將它新增至相依性插入容器，如下所示。 這是在方法中完成 `Startup.ConfigureServices` 。

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```
### <a name="javascript-telemetry-initializers"></a>JavaScript 遙測初始設定式
*JavaScript*

在您從入口網站取得的初始化程式碼之後立即插入遙測初始設定式：

```JS
<script type="text/javascript">
    // ... initialization code
    ...({
        instrumentationKey: "your instrumentation key"
    });
    window.appInsights = appInsights;


    // Adding telemetry initializer.
    // This is called whenever a new telemetry item
    // is created.

    appInsights.addTelemetryInitializer(function (envelope) {
        var telemetryItem = envelope.data.baseData;

        // To check the telemetry items type - for example PageView:
        if (envelope.name == Microsoft.ApplicationInsights.Telemetry.PageView.envelopeType) {
            // this statement removes url from all page view documents
            telemetryItem.url = "URL CENSORED";
        }

        // To set custom properties:
        telemetryItem.properties = telemetryItem.properties || {};
        telemetryItem.properties["globalProperty"] = "boo";
        
        // To set cloud role name / instance
        envelope.tags["ai.cloud.role"] = "your role name";
        envelope.tags["ai.cloud.roleInstance"] = "your role instance";
    });

    // End of inserted code.

    appInsights.trackPageView();
</script>
```

如需遙測專案上可用之 noncustom 屬性的摘要，請參閱 [Application Insights 匯出資料模型](./export-data-model.md)。

您可以依需要加入多個初始設定式。 它們是依其新增的順序來呼叫。

### <a name="opencensus-python-telemetry-processors"></a>OpenCensus Python 遙測處理器

OpenCensus Python 中的遙測處理器是稱為的回呼函式，可在匯出之前先處理遙測。 回呼函數必須接受 [信封](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py#L86) 資料類型做為其參數。 若要篩選出無法匯出的遙測，請確定回呼函數傳回 `False` 。 您可以在 [GitHub 上](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py)的信封中看到 Azure 監視器資料類型的架構。

> [!NOTE]
> 您可以藉 `cloud_RoleName` 由變更 `ai.cloud.role` 欄位中的屬性來進行修改 `tags` 。

```python
def callback_function(envelope):
    envelope.tags['ai.cloud.role'] = 'new_role_name'
```

```python
# Example for log exporter
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)

# Callback function to append '_hello' to each log message telemetry
def callback_function(envelope):
    envelope.data.baseData.message += '_hello'
    return True

handler = AzureLogHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>')
handler.add_telemetry_processor(callback_function)
logger.addHandler(handler)
logger.warning('Hello, World!')
```
```python
# Example for trace exporter
import requests

from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['requests'])

# Callback function to add os_type: linux to span properties
def callback_function(envelope):
    envelope.data.baseData.properties['os_type'] = 'linux'
    return True

exporter = AzureExporter(
    connection_string='InstrumentationKey=<your-instrumentation-key-here>'
)
exporter.add_telemetry_processor(callback_function)
tracer = Tracer(exporter=exporter, sampler=ProbabilitySampler(1.0))
with tracer.span(name='parent'):
response = requests.get(url='https://www.wikipedia.org/wiki/Rabbit')
```

```python
# Example for metrics exporter
import time

from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module

stats = stats_module.stats
view_manager = stats.view_manager
stats_recorder = stats.stats_recorder

CARROTS_MEASURE = measure_module.MeasureInt("carrots",
                                            "number of carrots",
                                            "carrots")
CARROTS_VIEW = view_module.View("carrots_view",
                                "number of carrots",
                                [],
                                CARROTS_MEASURE,
                                aggregation_module.CountAggregation())

# Callback function to only export the metric if value is greater than 0
def callback_function(envelope):
    return envelope.data.baseData.metrics[0].value > 0

def main():
    # Enable metrics
    # Set the interval in seconds in which you want to send metrics
    exporter = metrics_exporter.new_metrics_exporter(connection_string='InstrumentationKey=<your-instrumentation-key-here>')
    exporter.add_telemetry_processor(callback_function)
    view_manager.register_exporter(exporter)

    view_manager.register_view(CARROTS_VIEW)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    mmap.measure_int_put(CARROTS_MEASURE, 1000)
    mmap.record(tmap)
    # Default export interval is every 15.0s
    # Your application should run for at least this amount
    # of time so the exporter will meet this interval
    # Sleep can fulfill this
    time.sleep(60)

    print("Done recording metrics")

if __name__ == "__main__":
    main()
```
您可以根據自己的需要新增多個處理器。 它們是依其新增的順序來呼叫。 如果有一個處理器擲回例外狀況，則不會影響下列處理器。

### <a name="example-telemetryinitializers"></a>範例 TelemetryInitializers

#### <a name="add-a-custom-property"></a>新增自訂屬性

下列範例初始化運算式會將自訂屬性新增至每個追蹤的遙測。

```csharp
public void Initialize(ITelemetry item)
{
  var itemProperties = item as ISupportProperties;
  if(itemProperties != null && !itemProperties.Properties.ContainsKey("customProp"))
    {
        itemProperties.Properties["customProp"] = "customValue";
    }
}
```

#### <a name="add-a-cloud-role-name"></a>新增雲端角色名稱

下列範例初始化運算式會將雲端角色名稱設定為每個追蹤的遙測。

```csharp
public void Initialize(ITelemetry telemetry)
{
    if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
    {
        telemetry.Context.Cloud.RoleName = "MyCloudRoleName";
    }
}
```

#### <a name="add-information-from-httpcontext"></a>從 HttpCoNtext 新增資訊

下列範例初始化運算式會從讀取資料 [`HttpContext`](/aspnet/core/fundamentals/http-context) ，並將其附加至 `RequestTelemetry` 實例。 會透過「函式相依性 `IHttpContextAccessor` 插入」自動提供。

```csharp
public class HttpContextRequestTelemetryInitializer : ITelemetryInitializer
{
    private readonly IHttpContextAccessor httpContextAccessor;

    public HttpContextRequestTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        this.httpContextAccessor =
            httpContextAccessor ??
            throw new ArgumentNullException(nameof(httpContextAccessor));
    }

    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        if (requestTelemetry == null) return;

        var claims = this.httpContextAccessor.HttpContext.User.Claims;
        Claim oidClaim = claims.FirstOrDefault(claim => claim.Type == "oid");
        requestTelemetry.Properties.Add("UserOid", oidClaim?.Value);
    }
}
```

## <a name="itelemetryprocessor-and-itelemetryinitializer"></a>ITelemetryProcessor 和 ITelemetryInitializer

遙測處理器與遙測初始設定式之間有何差異？

* 您可以使用它們來做一些重迭。 兩者都可以用來新增或修改遙測的屬性，不過我們建議您針對該目的使用初始化運算式。
* 遙測初始化運算式一律會在遙測處理器之前執行。
* 遙測初始化運算式可能會呼叫一次以上。 依照慣例，它們並不會設定任何已設定的屬性。
* 遙測處理器可讓您完全取代或捨棄遙測專案。
* 保證會針對每個遙測專案呼叫所有已註冊的遙測初始化運算式。 針對遙測處理器，SDK 保證會呼叫第一個遙測處理器。 先前的遙測處理器會決定是否要呼叫其餘的處理器。
* 使用遙測初始化運算式以其他屬性擴充遙測，或覆寫現有的屬性。 使用遙測處理器來篩選出遙測。

## <a name="troubleshoot-applicationinsightsconfig"></a>針對 ApplicationInsights.config 進行疑難排解

* 確認完整格式的類型名稱和組件名稱均正確。
* 確認 applicationinsights.config 檔案在您的輸出目錄中，並且包含任何最近的變更。

## <a name="reference-docs"></a>參考文件

* [API 概觀](./api-custom-events-metrics.md)
* [ASP.NET 參考](/previous-versions/azure/dn817570(v=azure.100))

## <a name="sdk-code"></a>SDK 程式碼

* [ASP.NET Core SDK](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [ASP.NET SDK](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [JavaScript SDK](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="next-steps"></a><a name="next"></a>後續步驟
* [搜尋事件和記錄](./diagnostic-search.md)
* [採樣](./sampling.md)
* [疑難排解](../faq.md)

