---
title: 在 Application Insights 中探索 .NET 追蹤記錄
description: 搜尋使用 Trace、NLog 或 Log4Net 產生的記錄檔。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 0c2a084f-6e71-467b-a6aa-4ab222f17153
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 05/03/2017
ms.author: mbullwin
ms.openlocfilehash: b1cd2e8d7649de48f34efb0c7d839e17906a29bf
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39044980"
---
# <a name="explore-net-trace-logs-in-application-insights"></a>在 Application Insights 中探索 .NET 追蹤記錄
如果您使用 NLog、log4Net 或 System.Diagnostics.Trace 在 ASP.NET 應用程式中進行診斷追蹤，您可以將記錄傳送至 [Azure Application Insights][start]，以在其中探索和搜尋這些記錄。 您的記錄檔會與來自應用程式的其他遙測合併，讓您可以識別與服務每個使用者要求相關聯的追蹤，並將它們與其他事件和例外狀況報告相互關聯。

> [!NOTE]
> 您需要記錄擷取模組嗎？ 對於第三方記錄器來說，它是一個有用的配接器，但是如果您還沒使用 NLog、log4Net 或 System.Diagnostics.Trace，請考慮直接呼叫 [Application Insights TrackTrace()](app-insights-api-custom-events-metrics.md#tracktrace) 。
>
>

## <a name="install-logging-on-your-app"></a>在您的 app 上安裝記錄
在專案中安裝您選擇的記錄架構。 這應該會在 app.config 或 web.config 中產生一個項目。

如果您使用 System.Diagnostics.Trace，則必須在 web.config 中加入一個項目：

```XML

    <configuration>
     <system.diagnostics>
       <trace autoflush="false" indentsize="4">
         <listeners>
           <add name="myListener"
             type="System.Diagnostics.TextWriterTraceListener"
             initializeData="TextWriterOutput.log" />
           <remove name="Default" />
         </listeners>
       </trace>
     </system.diagnostics>
   </configuration>
```
## <a name="configure-application-insights-to-collect-logs"></a>設定 Application Insights 收集記錄
如果您尚未這麼做，請**[將 Application Insights 新增至您的專案](app-insights-asp-net.md)**。 您將會看見包含記錄收集器的選項。

或者，在 [方案總管] 中以滑鼠右鍵按一下專案，來 **設定 Application Insights** 。 選取此選項以 [設定追蹤集合]。

*沒有 Application Insights 功能表或記錄收集器選項嗎？* 請嘗試進行[疑難排解](#troubleshooting)。

## <a name="manual-installation"></a>手動安裝
如果 Application Insights 安裝程式不支援您的專案類型 (例如 Windows 傳統型專案)，請使用這個方法。

1. 如果您打算使用 log4Net 或 NLog，請將它安裝在您的專案。
2. 在 [方案總管] 中，以滑鼠右鍵按一下您的專案並選擇 [ **管理 NuGet 封裝**]。
3. 搜尋「Application Insights」
4. 選取適當的套件 - 下列其中一個：

   * Microsoft.ApplicationInsights.TraceListener (擷取 System.Diagnostics.Trace 呼叫)
   * Microsoft.ApplicationInsights.EventSourceListener (若為擷取 EventSource 事件)
   * Microsoft.ApplicationInsights.EtwCollector (若為擷取 ETW 事件)
   * Microsoft.ApplicationInsights.NLogTarget
   * Microsoft.ApplicationInsights.Log4NetAppender

NuGet 封裝會安裝必要的組件，並修改 web.config 或 app.config。

## <a name="insert-diagnostic-log-calls"></a>插入診斷記錄呼叫
如果您使用 System.Diagnostics.Trace，典型的呼叫如下：

    System.Diagnostics.Trace.TraceWarning("Slow response - database01");

如果您偏好 log4net 或 NLog：

    logger.Warn("Slow response - database01");

## <a name="using-eventsource-events"></a>使用 EventSource 事件
您可以將 [System.Diagnostics.Tracing.EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) 設定為要傳送至 Application Insights 作為追蹤的事件。 首先，安裝 `Microsoft.ApplicationInsights.EventSourceListener` NuGet 套件。 然後編輯 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 檔案的 `TelemetryModules` 區段。

```xml
    <Add Type="Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule, Microsoft.ApplicationInsights.EventSourceListener">
      <Sources>
        <Add Name="MyCompany" Level="Verbose" />
      </Sources>
    </Add>
```

針對每個來源，您可以設定下列參數︰
 * `Name` 會指定要收集之 EventSource 的名稱。
 * `Level` 會指定要收集的記錄等級。 可以是 `Critical`、`Error`、`Informational`、`LogAlways`、`Verbose`、`Warning` 其中一個。
 * `Keywords` (選擇性) 會指定要使用的關鍵字組合之整數值。

## <a name="using-diagnosticsource-events"></a>使用 DiagnosticSource 事件
您可以將 [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) 設定為要傳送至 Application Insights 作為追蹤的事件。 首先，安裝 [`Microsoft.ApplicationInsights.DiagnosticSourceListener`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener) NuGet 套件。 然後編輯 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 檔案的 `TelemetryModules` 區段。

```xml
    <Add Type="Microsoft.ApplicationInsights.DiagnosticSourceListener.DiagnosticSourceTelemetryModule, Microsoft.ApplicationInsights.DiagnosticSourceListener">
      <Sources>
        <Add Name="MyDiagnosticSourceName" />
      </Sources>
    </Add>
```

針對您想要追蹤的每個 DiagnosticSource，新增項目並將其 `Name` 屬性設定為 DiagnosticSource 的名稱。

## <a name="using-etw-events"></a>使用 ETW 事件
您可以設定要傳送至 Application Insights 作為追蹤的 ETW 事件。 首先，安裝 `Microsoft.ApplicationInsights.EtwCollector` NuGet 套件。 然後編輯 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 檔案的 `TelemetryModules` 區段。

> [!NOTE] 
> 僅在裝載 SDK 的處理序是以「效能記錄使用者」或系統管理員成員的身分識別執行時，才可收集 ETW 事件。

```xml
    <Add Type="Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule, Microsoft.ApplicationInsights.EtwCollector">
      <Sources>
        <Add ProviderName="MyCompanyEventSourceName" Level="Verbose" />
      </Sources>
    </Add>
```

針對每個來源，您可以設定下列參數︰
 * `ProviderName` 是要收集之 ETW 提供者的名稱。
 * `ProviderGuid` 會指定要收集的 ETW 提供者 GUID，並可取代 `ProviderName` 使用之。
 * `Level` 會設定要收集的記錄等級。 可以是 `Critical`、`Error`、`Informational`、`LogAlways`、`Verbose`、`Warning` 其中一個。
 * `Keywords` (選擇性) 會設定要使用的關鍵字組合之整數值。

## <a name="using-the-trace-api-directly"></a>直接使用追蹤 API
您可以直接呼叫 Application Insights 追蹤 API。 記錄配接器會使用此 API。

例如︰

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow response - database01");

TrackTrace 的優點在於您可以將較長的資料放在訊息中。 例如，您可以在該處編碼 POST 資料。

此外，您可以在訊息中新增嚴重性層級。 就像其他遙測一樣，您可以新增屬性值以供協助篩選或搜尋不同的追蹤集。 例如︰

    var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
    telemetry.TrackTrace("Slow database response",
                   SeverityLevel.Warning,
                   new Dictionary<string,string> { {"database", db.ID} });

在[搜尋][diagnostic]中，這可讓您輕鬆地篩選出與特定資料庫相關且具有特定嚴重性層級的所有訊息。

## <a name="explore-your-logs"></a>探索記錄
在偵錯模式中執行您的 app 或即時部署它。

在 [Application Insights 入口網站][portal]中您的應用程式概觀刀鋒視窗，選擇 [搜尋][diagnostic]。

![在 Application Insights 中，選擇 [搜尋]](./media/app-insights-asp-net-trace-logs/020-diagnostic-search.png)

![Search](./media/app-insights-asp-net-trace-logs/10-diagnostics.png)

例如，您可以：

* 篩選記錄追蹤，或具有特定屬性的項目
* 詳細檢查特定項目。
* 尋找與相同使用者要求相關的其他遙測 (也就是使用相同的 OperationId)
* 將此頁面的組態儲存為我的最愛

> [!NOTE]
> **取樣** 如果您的應用程式傳送大量資料，且您是使用 Application Insights SDK for ASP.NET 版本 2.0.0-beta3 或更新版本，則調適性取樣功能可能會運作，並只傳送一部分的遙測資料。 [深入了解取樣。](app-insights-sampling.md)
>
>

## <a name="next-steps"></a>後續步驟
[在 ASP.NET 中診斷失敗和例外狀況][exceptions]

[深入了解搜尋][diagnostic]。

## <a name="troubleshooting"></a>疑難排解
### <a name="how-do-i-do-this-for-java"></a>如果是 Java，我要怎麼做？
使用 [Java 記錄配接器](app-insights-java-trace-logs.md)。

### <a name="theres-no-application-insights-option-on-the-project-context-menu"></a>專案內容功能表上沒有 Application Insights 選項
* 請檢查已在此開發電腦上安裝 Application Insights 工具。 在 Visual Studio 功能表 [工具、擴充功能和更新] 中，尋找 Application Insights 工具。 如果此工具不在 [已安裝] 索引標籤中，請開啟 [線上] 索引標籤並安裝它。
* 這可能是 Application Insights 工具不支援的一種專案類型。 請使用 [手動安裝](#manual-installation)。

### <a name="no-log-adapter-option-in-the-configuration-tool"></a>組態工具中沒有記錄配接器選項
* 您必須先安裝記錄架構。
* 如果您使用 System.Diagnostics.Trace，請確定您[已在 `web.config`](https://msdn.microsoft.com/library/system.diagnostics.eventlogtracelistener.aspx) 中設定它。
* 您已有最新版的 Application Insights 嗎？ 在 Visual Studio 的 [工具] 功能表中，選擇 [擴充功能和更新]，然後開啟 [更新] 索引標籤。如果發現開發人員分析工具，請按一下該工具以進行更新。

### <a name="emptykey"></a>我收到「檢測金鑰不能是空白」的錯誤
您可能只安裝記錄配接器 Nuget 封裝，但未安裝 Application Insights。

在 [方案總管] 中，以滑鼠右鍵按一下 `ApplicationInsights.config` ，然後選擇 [ **更新 Application Insights**]。 將會出現對話方塊邀請您登入 Azure，並建立 Application Insights 資源或重複使用現有的資源。 這樣應該可以解決。

### <a name="i-can-see-traces-in-diagnostic-search-but-not-the-other-events"></a>我可以在診斷搜尋中看見追蹤，但是看不到其他事件
有時候可能需要一段時間，所有事件和要求才會通過管線。

### <a name="limits"></a>保留多少資料？
有好幾個因素會影響保留的資料量。 如需更多資訊，請參閱客戶事件計量頁面的 [限制](app-insights-api-custom-events-metrics.md#limits) 區段。 

### <a name="im-not-seeing-some-of-the-log-entries-that-i-expect"></a>我沒看到一些預期的記錄項目
如果您的應用程式傳送大量資料，且您是使用 Application Insights SDK for ASP.NET 版本 2.0.0-beta3 或更新版本，則調適性取樣功能可能會運作，並只傳送一部分的遙測資料。 [深入了解取樣。](app-insights-sampling.md)

## <a name="add"></a>後續步驟
* [設定可用性和回應性測試][availability]
* [疑難排解][qna]

<!--Link references-->

[availability]: app-insights-monitor-web-app-availability.md
[diagnostic]: app-insights-diagnostic-search.md
[exceptions]: app-insights-asp-net-exceptions.md
[portal]: https://portal.azure.com/
[qna]: app-insights-troubleshoot-faq.md
[start]: app-insights-overview.md
