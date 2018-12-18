---
title: ApplicationInsights.config 參考 - Azure | Microsoft Docs
description: 啟用或停用資料收集模組，以及加入效能計數器和其他參數。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 6e397752-c086-46e9-8648-a1196e8078c2
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 09/19/2018
ms.reviewer: olegan
ms.author: mbullwin
ms.openlocfilehash: f3bc64bd010bed9e177fd18cc6cb238b94669248
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990226"
---
# <a name="configuring-the-application-insights-sdk-with-applicationinsightsconfig-or-xml"></a>使用 ApplicationInsights.config 或 .xml 設定 Application Insights SDK
Application Insights .NET SDK 是由數個 NuGet 封裝所組成。 [核心封裝](http://www.nuget.org/packages/Microsoft.ApplicationInsights) 提供 API，用於傳送遙測至 Application Insights。 [其他套件](http://www.nuget.org/packages?q=Microsoft.ApplicationInsights)提供遙測*模組*和*初始設定式*，用於自動從您的應用程式和其內容追蹤遙測。 您可以藉由調整組態檔，來啟用或停用遙測模組和初始設定式，並為其設定一些參數。

組態檔的名稱為 `ApplicationInsights.config` 或 `ApplicationInsights.xml`，端視您的應用程式類型而定。 當您[安裝大部分版本的 SDK][start] 時，系統會自動將組態檔加入至您的專案。 [IIS 伺服器上的狀態監視器][redfield]，或是當您選取 [Azure 網站或 VM 的 Application Insights 延伸模組](app-insights-azure-web-apps.md)時，也會將組態檔加入至 Web 應用程式。

沒有同等的檔案可以控制[網頁中的 SDK][client]。

本文件說明您在組態檔中看到的內容、控制 SDK 元件的方式，以及哪些 NuGet 封裝載入這些元件。

> [!NOTE]
> ApplicationInsights.config 和 .xml 指示不適用 .NET Core SDK。 對於 .NET Core 應用程式的變更，我們通常會使用 appsettings.json 檔案。 如需這類範例，請參閱[快照偵錯工具文件](https://docs.microsoft.com/azure/application-insights/app-insights-snapshot-debugger#configure-snapshot-collection-for-aspnet-core-20-applications)。

## <a name="telemetry-modules-aspnet"></a>遙測模組 (ASP.NET)
每個遙測模組收集特定類型的資料，以及使用核心 API 來傳送資料。 模組由不同的 NuGet 封裝安裝，也會將必要的行加入 .config 檔案。

組態檔中的每個模組都有一個節點。 若要停用模組，請刪除節點或將其註解化。

### <a name="dependency-tracking"></a>相依性追蹤
[相依性追蹤](app-insights-asp-net-dependencies.md) 會收集有關您的 app 對資料庫和外部服務和資料庫呼叫的遙測。 若要允許此模組用於 IIS 伺服器，您必須[安裝狀態監視器][redfield]。 若要在 Azure Web 應用程式或 VM 中使用此模組， [請選取 Application Insights 延伸模組](app-insights-azure-web-apps.md)。

您也可以使用 [TrackDependency API](app-insights-api-custom-events-metrics.md#trackdependency)撰寫您自己的相依性追蹤程式碼。

* `Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.DependencyCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) NuGet 封裝。

### <a name="performance-collector"></a>效能收集器
[收集系統效能計數器](app-insights-performance-counters.md)，例如 IIS 安裝的 CPU、記憶體和網路負載。 您可以指定要收集哪些計數器，包括您自己所設定的效能計數器。

* `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule`
* [Microsoft.ApplicationInsights.PerfCounterCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector) NuGet 封裝。

### <a name="application-insights-diagnostics-telemetry"></a>Application Insights 診斷遙測
`DiagnosticsTelemetryModule` 報告 Application Insights 檢測程式碼本身中的錯誤。 例如，如果程式碼無法存取效能計數器，或 `ITelemetryInitializer` 擲回例外狀況。 此模組所追蹤的追蹤遙測會出現在[診斷搜尋][diagnostic]中。 將診斷資料傳送至 dc.services.vsallin.net。

* `Microsoft.ApplicationInsights.Extensibility.Implementation.Tracing.DiagnosticsTelemetryModule`
* [Microsoft.ApplicationInsights](http://www.nuget.org/packages/Microsoft.ApplicationInsights) NuGet 封裝。 如果您只安裝這個封裝，不會自動建立 ApplicationInsights.config 檔案。

### <a name="developer-mode"></a>開發人員模式
`DeveloperModeWithDebuggerAttachedTelemetryModule` 會強制 Application Insights `TelemetryChannel` 立即傳送資料，在偵錯工具附加至應用程式程序時一次傳送一個遙測項目。 這會減少您的應用程式追蹤遙測時，與當遙測出現在 Application Insights 入口網站時之間的時間量。 但是會造成 CPU 和網路頻寬明顯的負擔。

* `Microsoft.ApplicationInsights.WindowsServer.DeveloperModeWithDebuggerAttachedTelemetryModule`
* [Application Insights Windows Server](http://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/) NuGet 封裝

### <a name="web-request-tracking"></a>Web 要求追蹤
報告 HTTP 要求的 [回應時間和結果碼](app-insights-asp-net.md) 。

* `Microsoft.ApplicationInsights.Web.RequestTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](http://www.nuget.org/packages/Microsoft.ApplicationInsights.Web) NuGet 封裝。

### <a name="exception-tracking"></a>例外狀況追蹤
`ExceptionTrackingTelemetryModule` 追蹤 Web 應用程式中未處理的例外狀況。 請參閱[失敗和例外狀況][exceptions]。

* `Microsoft.ApplicationInsights.Web.ExceptionTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](http://www.nuget.org/packages/Microsoft.ApplicationInsights.Web) NuGet 封裝。
* `Microsoft.ApplicationInsights.WindowsServer.UnobservedExceptionTelemetryModule` - 追蹤 [未觀察到的工作例外狀況](http://blogs.msdn.com/b/pfxteam/archive/2011/09/28/task-exception-handling-in-net-4-5.aspx)。
* `Microsoft.ApplicationInsights.WindowsServer.UnhandledExceptionTelemetryModule` - 追蹤背景工作角色、Windows 服務和主控台應用程式的未處理例外狀況。
* [Application Insights Windows Server](http://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/) NuGet 封裝。

### <a name="eventsource-tracking"></a>EventSource 追蹤
`EventSourceTelemetryModule` 可讓您設定要傳送至 Application Insights 作為追蹤的 EventSource 事件。 如需追蹤 EventSource 事件的資訊，請參閱[使用 EventSource 事件](app-insights-asp-net-trace-logs.md#using-eventsource-events)。

* `Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule`
* [Microsoft.ApplicationInsights.EventSourceListener](http://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener) 

### <a name="etw-event-tracking"></a>ETW 事件追蹤
`EtwCollectorTelemetryModule` 可讓您設定要傳送至 Application Insights 作為追蹤的 ETW 提供者事件。 如需追蹤 ETW 事件的資訊，請參閱[使用 ETW 事件](app-insights-asp-net-trace-logs.md#using-etw-events)。

* `Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule`
* [Microsoft.ApplicationInsights.EtwCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector) 

### <a name="microsoftapplicationinsights"></a>Microsoft.ApplicationInsights
Microsoft.ApplicationInsights 封裝提供 SDK 的 [核心 API](https://msdn.microsoft.com/library/mt420197.aspx) 。 其他遙測模組使用此功能，您也可以 [使用它來定義您自己的遙測](app-insights-api-custom-events-metrics.md)。

* ApplicationInsights.config 中沒有項目。
* [Microsoft.ApplicationInsights](http://www.nuget.org/packages/Microsoft.ApplicationInsights) NuGet 封裝。 如果您只安裝此 NuGet，不會產生任何 .config 檔案。

## <a name="telemetry-channel"></a>遙測通道
遙測通道管理遙測的緩衝處理和到 Application Insights 服務的傳輸。

* `Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel` 是服務的預設通道。 它會在記憶體中緩衝資料。
* `Microsoft.ApplicationInsights.PersistenceChannel` 是主控台應用程式的替代通道。 它可以在您的 app 關閉時將任何未清除的資料儲存到永續性儲存體，並在 app 重新啟動時再次傳送資料。

## <a name="telemetry-initializers-aspnet"></a>遙測初始設定式 (ASP.NET)
遙測初始設定式設定與遙測的每個項目一起傳送的內容屬性。

您可以 [撰寫您自己的初始設定式](app-insights-api-filtering-sampling.md#add-properties) 設定內容屬性。

標準的初始設定式已由 Web 或 WindowsServer NuGet 封裝設定：

* `AccountIdTelemetryInitializer` 設定 AccountId 屬性。
* `AuthenticatedUserIdTelemetryInitializer` 如 JavaScript SDK 設定般設定 AuthenticatedUserId 屬性。
* 針對具有從 Azure 執行階段環境擷取之資訊的所有遙測項目，`AzureRoleEnvironmentTelemetryInitializer` 會更新 `Device` 內容的 `RoleName` 和 `RoleInstance` 屬性。
* 針對具有從 MS 組建所產生之 `BuildInfo.config` 檔案擷取值的所有遙測項目，`BuildInfoConfigComponentVersionTelemetryInitializer` 會更新 `Component` 內容的 `Version` 屬性。
* `ClientIpHeaderTelemetryInitializer` 會根據要求的 `X-Forwarded-For` HTTP 標頭來更新所有遙測項目之 `Location` 內容的 `Ip` 屬性。
* `DeviceTelemetryInitializer` 會更新所有遙測項目 `Device` 內容的下列屬性。
  * `Type` 設定為 "PC"
  * `Id` 設定為 Web 應用程式執行所在電腦的網域名稱。
  * `OemName` 設定為使用 WMI 從 `Win32_ComputerSystem.Manufacturer` 欄位擷取的值。
  * `Model` 設定為使用 WMI 從 `Win32_ComputerSystem.Model` 欄位擷取的值。
  * `NetworkType` 設定為從 `NetworkInterface` 擷取的值。
  * `Language` 設定為 `CurrentCulture` 的名稱。
* 針對具有 Web 應用程式執行所在電腦之網域名稱的所有遙測項目，`DomainNameRoleInstanceTelemetryInitializer` 會更新 `Device` 內容的 `RoleInstance` 屬性。
* `OperationNameTelemetryInitializer` 會根據 HTTP 方法，以及 ASP.NET MVC 控制器的名稱和叫用來處理要求的動作，更新所有遙測項目 `RequestTelemetry` 之 `Name` 屬性和 `Operation` 內容的 `Name` 屬性。
* `OperationIdTelemetryInitializer` 或 `OperationCorrelationTelemetryInitializer` 在處理具有自動產生的 `RequestTelemetry.Id` 的要求時，會更新追蹤的所有遙測項目的 `Operation.Id` 內容屬性。
* 針對具有從使用者瀏覽器中執行的 Application Insights JavaScript 檢測程式碼所產生之 `ai_session` Cookie 擷取值的所有遙測項目，`SessionTelemetryInitializer` 會更新 `Session` 內容的 `Id` 屬性。
* `SyntheticTelemetryInitializer` 或 `SyntheticUserAgentTelemetryInitializer` 在處理來自綜合來源 (例如可用性測試或搜尋引擎 Bot) 的要求時，會更新追蹤的所有遙測項目的 `User`、`Session` 和 `Operation` 內容屬性。 根據預設， [計量瀏覽器](app-insights-metrics-explorer.md) 不會顯示綜合的遙測。

    `<Filters>` 會設定要求的識別屬性。
* 針對具有從使用者瀏覽器中執行之 Application Insights JavaScript 檢測程式碼所產生的 `ai_user` Cookie 擷取值的所有遙測項目，`UserTelemetryInitializer` 會更新 `User` 內容的 `Id` 和 `AcquisitionDate` 屬性。
* `WebTestTelemetryInitializer` 會設定使用者識別碼、工作階段識別碼，以及來自 [可用性測試](app-insights-monitor-web-app-availability.md)的 HTTP 要求的綜合來源屬性。
  `<Filters>` 會設定要求的識別屬性。

針對 Service Fabric 中執行的 .NET 應用程式，您可以包含 `Microsoft.ApplicationInsights.ServiceFabric` NuGet 套件。 此套件包含的 `FabricTelemetryInitializer` 會將 Service Fabric 屬性新增至遙測項目。 如需詳細資訊，請參閱 [GitHub 頁面](https://github.com/Microsoft/ApplicationInsights-ServiceFabric/blob/master/README.md)了解這個 NuGet 套件所新增之屬性的相關資訊。

## <a name="telemetry-processors-aspnet"></a>遙測處理器 (ASP.NET)
遙測處理器可以在遙測從 SDK 傳送至入口網站之前篩選並修改每個遙測項目。

您可以 [撰寫您自己的遙測處理器](app-insights-api-filtering-sampling.md#filtering)。

#### <a name="adaptive-sampling-telemetry-processor-from-200-beta3"></a>自適性取樣遙測處理器 (自 2.0.0-beta3 起)
此選項預設為啟用狀態。 如果您的應用程式傳送許多遙測，此處理器會移除其中一些遙測。

```xml

    <TelemetryProcessors>
      <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
        <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
      </Add>
    </TelemetryProcessors>

```

參數會提供演算法嘗試要達成的目標。 每個 SDK 執行個體都獨立運作，因此如果您的伺服器是數個機器的叢集，遙測的實際數量會隨之加乘。

[深入了解取樣](app-insights-sampling.md)。

#### <a name="fixed-rate-sampling-telemetry-processor-from-200-beta1"></a>固定取樣遙測處理器 (自 2.0.0-beta1 起)
還有一種標準的 [取樣遙測處理器](app-insights-api-filtering-sampling.md) (自 2.0.1 起)：

```XML

    <TelemetryProcessors>
     <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">

     <!-- Set a percentage close to 100/N where N is an integer. -->
     <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
     <SamplingPercentage>10</SamplingPercentage>
     </Add>
   </TelemetryProcessors>

```



## <a name="channel-parameters-java"></a>通道參數 (Java)
這些參數會影響 Java SDK 應該儲存和排清它所收集之遙測資料的方式。

#### <a name="maxtelemetrybuffercapacity"></a>MaxTelemetryBufferCapacity
可以儲存在 SDK 記憶體內儲存空間中的遙測項目數。 達到這個數目時，會排清遙測緩衝區，也就是將遙測項目傳送至 Application Insights 伺服器。

* 最小值：1
* 最大值：1000
* 預設值：500

```

  <ApplicationInsights>
      ...
      <Channel>
       <MaxTelemetryBufferCapacity>100</MaxTelemetryBufferCapacity>
      </Channel>
      ...
  </ApplicationInsights>
```

#### <a name="flushintervalinseconds"></a>FlushIntervalInSeconds
決定應該以何種頻率排清儲存在記憶體內儲存空間的資料 (並傳送至 Application Insights)。

* 最小值：1
* 最大值：300
* 預設值：5

```

    <ApplicationInsights>
      ...
      <Channel>
        <FlushIntervalInSeconds>100</FlushIntervalInSeconds>
      </Channel>
      ...
    </ApplicationInsights>
```

#### <a name="maxtransmissionstoragecapacityinmb"></a>MaxTransmissionStorageCapacityInMB
決定分配給本機磁碟上永續性存放裝置的大小上限 (MB)。 此存放裝置會用於保存無法傳送至 Application Insights 端點的遙測項目。 存放裝置大小達到上限時，會捨棄新的遙測項目。

* 最小值：1
* 最大值：100
* 預設值：10

```

   <ApplicationInsights>
      ...
      <Channel>
        <MaxTransmissionStorageCapacityInMB>50</MaxTransmissionStorageCapacityInMB>
      </Channel>
      ...
   </ApplicationInsights>
```

#### <a name="local-forwarder"></a>本機轉送工具

[本機轉送工具](https://docs.microsoft.com/azure/application-insights/local-forwarder)是會從各種 SDK 和架構收集 Application Insights 或 [OpenCensus](https://opencensus.io/) 遙測資料的代理程式，並且會將這些資料路由至 Application Insights。 此工具能夠在 Windows 和 Linux 下執行。 

```xml
<Channel type="com.microsoft.applicationinsights.channel.concrete.localforwarder.LocalForwarderTelemetryChannel">
<DeveloperMode>false</DeveloperMode>
<EndpointAddress><!-- put the hostname:port of your LocalForwarder instance here --></EndpointAddress>
<!-- The properties below are optional. The values shown are the defaults for each property -->
<FlushIntervalInSeconds>5</FlushIntervalInSeconds><!-- must be between [1, 500]. values outside the bound will be rounded to nearest bound -->
<MaxTelemetryBufferCapacity>500</MaxTelemetryBufferCapacity><!-- units=number of telemetry items; must be between [1, 1000] -->
</Channel>
```

如果您使用 SpringBoot 入門版，請將下列項目新增至組態檔 (application.properies)：

```yml
azure.application-insights.channel.local-forwarder.endpoint-address=<!--put the hostname:port of your LocalForwarder instance here-->
azure.application-insights.channel.local-forwarder.flush-interval-in-seconds=<!--optional-->
azure.application-insights.channel.local-forwarder.max-telemetry-buffer-capacity=<!--optional-->
```

SpringBoot application.properties 和 applicationinsights.xml 組態的預設值相同。

## <a name="instrumentationkey"></a>InstrumentationKey
這會決定顯示您資料的 Application Insights 資源。 通常您會針對每個應用程式，用個別的金鑰建立個別資源。

如果想要以動態方式設定金鑰 (例如，想要將應用程式的結果傳送到不同的資源)，您可以在組態檔中省略金鑰，並將金鑰設定在程式碼中。

若要針對 TelemetryClient 的所有執行個體 (包括標準遙測模組) 設定金鑰，請在 TelemetryConfiguration.Active 中設定金鑰。 請在初始化方法中這麼做，例如 ASP.NET 服務中的 global.aspx.cs：

```csharp

    protected void Application_Start()
    {
      Microsoft.ApplicationInsights.Extensibility.
        TelemetryConfiguration.Active.InstrumentationKey =
          // - for example -
          WebConfigurationManager.Settings["ikey"];
      //...
```

如果您只想將特定的一組事件傳送至不同的資源，可以針對特定的 TelemetryClient 設定金鑰：

```csharp

    var tc = new TelemetryClient();
    tc.Context.InstrumentationKey = "----- my key ----";
    tc.TrackEvent("myEvent");
    // ...

```

若要取得新的金鑰，請[在 Application Insights 入口網站中建立新的資源][new]。



## <a name="applicationid-provider"></a>ApplicationId 提供者

_從 v2.6.0 開始可供使用_

提供者的用途是要查閱以檢測金鑰為基礎的應用程式識別碼。 應用程式識別碼包含在 RequestTelemetry 和 DependencyTelemetry 中，並且用來判斷入口網站中的相互關聯。

可藉由在程式碼或組態中設定 `TelemetryConfiguration.ApplicationIdProvider` 來使用此功能。

### <a name="interface-iapplicationidprovider"></a>介面：IApplicationIdProvider

```csharp
public interface IApplicationIdProvider
{
    bool TryGetApplicationId(string instrumentationKey, out string applicationId);
}
```


我們在 [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) SDK 中提供兩個實作：`ApplicationInsightsApplicationIdProvider` 和 `DictionaryApplicationIdProvider`。

### <a name="applicationinsightsapplicationidprovider"></a>ApplicationInsightsApplicationIdProvider

這是我們設定檔 API 的包裝函式。 它會對要求進行節流處理並快取結果。

當您安裝 [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) 或 [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) 時，此提供者會新增至組態檔

此類別具有選擇性屬性 `ProfileQueryEndpoint`。
依預設，這會設定為 `https://dc.services.visualstudio.com/api/profiles/{0}/appId`。
如果您需要為此組態設定 Proxy，我們建議對基底位址 (Base Address) 使用 Proxy 並包括 "/api/profiles/{0}/appId"。 請注意，在每個要求的執行階段上，'{0}' 會替代為檢測金鑰。

#### <a name="example-configuration-via-applicationinsightsconfig"></a>透過 ApplicationInsights.config 的範例組態：
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
        <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>透過程式碼的範例組態：
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new ApplicationInsightsApplicationIdProvider();
```

### <a name="dictionaryapplicationidprovider"></a>DictionaryApplicationIdProvider

這是靜態提供者，將依賴您設定的檢測金鑰/應用程式識別碼配對。

此類別具有 `Defined` 屬性，也就是檢測金鑰與應用程式識別碼配對的「字典<string,string>」。

此類別具有選擇性的 `Next` 屬性，可在您組態中不存在要求的檢測金鑰時，用來設定其他提供者。

#### <a name="example-configuration-via-applicationinsightsconfig"></a>透過 ApplicationInsights.config 的範例組態：
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.DictionaryApplicationIdProvider, Microsoft.ApplicationInsights">
        <Defined>
            <Type key="InstrumentationKey_1" value="ApplicationId_1"/>
            <Type key="InstrumentationKey_2" value="ApplicationId_2"/>
        </Defined>
        <Next Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights" />
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>透過程式碼的範例組態：
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new DictionaryApplicationIdProvider{
 Defined = new Dictionary<string, string>
    {
        {"InstrumentationKey_1", "ApplicationId_1"},
        {"InstrumentationKey_2", "ApplicationId_2"}
    }
};
```




## <a name="next-steps"></a>後續步驟
[深入了解 API][api]。

<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[client]: app-insights-javascript.md
[diagnostic]: app-insights-diagnostic-search.md
[exceptions]: app-insights-asp-net-exceptions.md
[netlogs]: app-insights-asp-net-trace-logs.md
[new]: app-insights-create-new-resource.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[start]: app-insights-overview.md
