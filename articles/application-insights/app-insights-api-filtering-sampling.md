---
title: 在 Application Insights SDK 中篩選及前置處理 | Microsoft Docs
description: 撰寫 SDK 的遙測處理器和遙測初始設定式來篩選屬性或將屬性新增至資料，再將遙測傳送至 Application Insights 入口網站。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 38a9e454-43d5-4dba-a0f0-bd7cd75fb97b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: article
ms.date: 11/23/2016
ms.author: mbullwin; borooji
ms.openlocfilehash: 987ae184a0812f24df99a0b6e6543c8be55a9e79
ms.sourcegitcommit: 909469bf17211be40ea24a981c3e0331ea182996
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
---
# <a name="filtering-and-preprocessing-telemetry-in-the-application-insights-sdk"></a>在 Application Insights SDK 中篩選及前置處理遙測


您可以針對 Application Insights SDK 撰寫與設定外掛程式，以自訂在遙測傳送至 Application Insights 服務之前，擷取與處理它的方式。

* [取樣](app-insights-sampling.md) 可減少遙測的量而不會影響統計資料。 它可將相關資料點寶持放在一起，因此您診斷問題時，能夠在資料點之間瀏覽。 在入口網站中將乘以總計數，以補償取樣。
* 使用 [ASP.NET](#filtering) 或 [Java](app-insights-java-filter-telemetry.md) 適用的遙測處理器進行篩選，可讓您先在 SDK 中選取或修改遙測，再將遙測傳送到伺服器。 例如，您可以從傀儡程式中排除要求來減少遙測量。 但和取樣相比，篩選是減少流量更基本的方法。 它可讓您更充分掌握傳輸內容，但是您必須注意，它會影響統計資料 (例如，若您要篩選所有成功的要求)。
* [遙測初始設定式會新增屬性](#add-properties) 至任何從應用程式傳送出來的遙測，包括從標準模組傳送出來的遙測。 例如，您可以新增計算好的值，或是用來在入口網站中篩選資料的版本號碼。
* [SDK API](app-insights-api-custom-events-metrics.md) 可用來傳送自訂事件和計量。

開始之前：

* 在應用程式中安裝 Application Insights [SDK for ASP.NET](app-insights-asp-net.md) 或 [SDK for Java](app-insights-java-get-started.md)。

<a name="filtering"></a>

## <a name="filtering-itelemetryprocessor"></a>篩選︰ITelemetryProcessor
這項技術可讓您更直接地控制包含在遙測串流中或排除於遙測串流外的內容。 您可以將它與取樣搭配使用或分開使用。

若要篩選遙測，請撰寫遙測處理器，並將它向 SDK 註冊。 所有遙測都會通過處理器，您可以選擇從串流中卸除或加入屬性。 這包括來自標準模組 (例如 HTTP 要求收集器和相依性收集器) 的遙測，以及您自己撰寫的遙測。 比方說，您可以篩選出有關來自傀儡程式要求或成功的相依性呼叫的遙測。

> [!WARNING]
> 篩選傳送自使用處理器的 SDK 的遙測可能會曲解您在入口網站中看到的統計資料，並且難以追蹤相關的項目。
>
> 請考慮改用 [取樣](app-insights-sampling.md)。
>
>

### <a name="create-a-telemetry-processor-c"></a>建立遙測處理器 (C#)
1. 確認專案中的 Application Insights SDK 為 2.0.0 版或更新版本。 在 Visual Studio 方案總管中以滑鼠右鍵按一下專案，然後選擇 [管理 NuGet 封裝]。 檢查 NuGet 封裝管理員中的 Microsoft.ApplicationInsights.Web。
2. 若要建立篩選器，請實作 ITelemetryProcessor。 這是遙測模組、遙測初始設定式和遙測通道之類的另一個擴充點。

    請注意，遙測處理器建構一連串的處理。 當您具現化遙測處理器時，您會傳遞連結至鏈結中的下一個處理器。 遙測資料點傳遞至處理序方法時，它會完成其工作並接著呼叫鏈結中的下一個遙測處理器。

```csharp

using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

public class SuccessfulDependencyFilter : ITelemetryProcessor
    {

    private ITelemetryProcessor Next { get; set; }

    // You can pass values from .config
    public string MyParamFromConfigFile { get; set; }

    // Link processors to each other in a chain.
    public SuccessfulDependencyFilter(ITelemetryProcessor next)
    {
        this.Next = next;
    }
    public void Process(ITelemetry item)
    {
        // To filter out an item, just return
        if (!OKtoSend(item)) { return; }
        // Modify the item if required
        ModifyItem(item);

        this.Next.Process(item);
    }

    // Example: replace with your own criteria.
    private bool OKtoSend (ITelemetry item)
    {
        var dependency = item as DependencyTelemetry;
        if (dependency == null) return true;

        return dependency.Success != true;
    }

    // Example: replace with your own modifiers.
    private void ModifyItem (ITelemetry item)
    {
        item.Context.Properties.Add("app-version", "1." + MyParamFromConfigFile);
    }
}

```
1. 在 ApplicationInsights.config 中插入：

```xml
<TelemetryProcessors>
    <Add Type="WebApplication9.SuccessfulDependencyFilter, WebApplication9">
        <!-- Set public property -->
        <MyParamFromConfigFile>2-beta</MyParamFromConfigFile>
    </Add>
</TelemetryProcessors>

```

(這是您用來初始化取樣篩選器的相同區段)。

您可以在類別中提供公開具名屬性，以從 .config 檔案傳遞字串值。

> [!WARNING]
> 仔細地將 .config 檔案中的類型名稱和任何屬性名稱與程式碼中的類別和屬性名稱做比對。 如果 .config 檔案參考不存在的類型或屬性，SDK 可能無法傳送任何遙測，而且不會產生任何訊息。
>
>

，您也可以在程式碼中初始化篩選。 在適當的初始化類別中 - 例如在 Global.asax.cs 中的 AppStart - 插入您的處理器至鏈結：

```csharp

var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;
builder.Use((next) => new SuccessfulDependencyFilter(next));

// If you have more processors:
builder.Use((next) => new AnotherProcessor(next));

builder.Build();

```

在這個點之後建立的 TelemetryClients 會使用您的處理器。

### <a name="example-filters"></a>範例篩選器
#### <a name="synthetic-requests"></a>綜合要求
篩選出 bot 和 Web 測試。 雖然計量瀏覽器可讓您篩選出綜合來源，此選項會藉由在 SDK 篩選它們以降低流量。

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
        // To filter out an item, just terminate the chain:
        return;
    }
    // Send everything else:
    this.Next.Process(item);
}

```

#### <a name="filter-out-fast-remote-dependency-calls"></a>篩選出快速遠端相依性呼叫
如果您只想要診斷速度很慢的呼叫，請篩選出快的項目。

> [!NOTE]
> 這樣會曲解您在入口網站上看到的統計資料。 相依性圖表將看起來好像相依性呼叫均失敗。
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

## <a name="add-properties-itelemetryinitializer"></a>新增屬性︰ITelemetryInitializer
使用遙測初始設定式來定義與所有遙測一起傳送的全域屬性；並覆寫選取的標準遙測模組行為。

例如，Web 封裝的 Application Insights 會收集有關 HTTP 要求的遙測。 根據預設，它會將所有含 >= 400 回應碼的要求標記為失敗。 但如果您想將 400 視為成功，您可以提供設定 Success 屬性的遙測初始設定式。

如果您提供遙測初始設定式，則會在呼叫任何的 Track*() 方法時呼叫它。 這包括由標準遙測模組呼叫的方法。 依照慣例，這些模組不會設定任何已由初始設定式設定的屬性。

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
            requestTelemetry.Context.Properties["Overridden400s"] = "true";
        }
        // else leave the SDK to set the Success property      
    }
    }
}
```

**載入您的初始設定式**

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

*或者* ，您也可以在程式碼 (如 Global.aspx.cs) 中具現化初始設定式：

```csharp
protected void Application_Start()
{
    // ...
    TelemetryConfiguration.Active.TelemetryInitializers
    .Add(new MyTelemetryInitializer());
}
```


[詳細查看此範例。](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole)

<a name="js-initializer"></a>

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

    appInsights.queue.push(function () {
        appInsights.context.addTelemetryInitializer(function (envelope) {
            var telemetryItem = envelope.data.baseData;

            // To check the telemetry item’s type - for example PageView:
            if (envelope.name == Microsoft.ApplicationInsights.Telemetry.PageView.envelopeType) {
                // this statement removes url from all page view documents
                telemetryItem.url = "URL CENSORED";
            }

            // To set custom properties:
            telemetryItem.properties = telemetryItem.properties || {};
            telemetryItem.properties["globalProperty"] = "boo";

            // To set custom metrics:
            telemetryItem.measurements = telemetryItem.measurements || {};
            telemetryItem.measurements["globalMetric"] = 100;
        });
    });

    // End of inserted code.

    appInsights.trackPageView();
</script>
```

如需 telemetryItem 上可用的非自訂屬性摘要，請參閱 [Application Insights 匯出資料模型](app-insights-export-data-model.md)。

您可以依需要加入多個初始設定式。

## <a name="itelemetryprocessor-and-itelemetryinitializer"></a>ITelemetryProcessor 和 ITelemetryInitializer
遙測處理器與遙測初始設定式之間有何差異？

* 它們的用途有部分重疊︰兩者都可以用來在遙測中新增屬性。
* TelemetryInitializers 一律會在 TelemetryProcessors 之前執行。
* TelemetryProcessors 可讓您完全取代或捨棄遙測項目。
* TelemetryProcessors 不會處理效能計數器遙測。

## <a name="troubleshooting-applicationinsightsconfig"></a>為 ApplicationInsights.config 疑難排解
* 確認完整格式的類型名稱和組件名稱均正確。
* 確認 applicationinsights.config 檔案在您的輸出目錄中，並且包含任何最近的變更。

## <a name="reference-docs"></a>參考文件
* [API 概觀](app-insights-api-custom-events-metrics.md)
* [ASP.NET 參考](https://msdn.microsoft.com/library/dn817570.aspx)

## <a name="sdk-code"></a>SDK 程式碼
* [ASP.NET Core SDK](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [ASP.NET SDK](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [JavaScript SDK](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="next"></a>後續步驟
* [搜尋事件和記錄檔](app-insights-diagnostic-search.md)
* [取樣](app-insights-sampling.md)
* [疑難排解](app-insights-troubleshoot-faq.md)
