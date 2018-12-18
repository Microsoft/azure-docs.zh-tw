---
title: Application Insights 中的效能計數器 | Microsoft Docs
description: 監視 Application Insights 中的系統和自訂 .NET 效能計數器。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 5b816f4c-a77a-4674-ae36-802ee3a2f56d
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 10/11/2016
ms.author: mbullwin
ms.openlocfilehash: bb1b1e2934de052479f39d40bfe143345e91bc26
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094078"
---
# <a name="system-performance-counters-in-application-insights"></a>Application Insights 中的系統效能計數器
Windows 提供多種[效能計數器](http://www.codeproject.com/Articles/8590/An-Introduction-To-Performance-Counters) (例如 CPU 使用、記憶體、磁碟和網路使用量)。 您也可以自行定義。 如果應用程式是在您具有系統管理存取權的內部部署主機或虛擬機器上於 IIS 下執行，則 [Application Insights](app-insights-overview.md) 可以顯示這些效能計數器。 這些圖表指出即時應用程式可用的資源，而且有助於識別伺服器執行個體之間的不平衡負載。

效能計數器會出現在 [伺服器] 刀鋒視窗中，其包括由伺服器執行個體所分段的資料表。

![Application Insights 中所報告的效能計數器](./media/app-insights-performance-counters/counters-by-server-instance.png)

(效能計數器不適用於 Azure Web Apps。 但是，您可以[將 Azure 診斷傳送至 Application Insights](app-insights-azure-diagnostics.md))。

## <a name="view-counters"></a>檢視計數器
[伺服器] 刀鋒視窗會顯示一組預設效能計數器。 

若要查看其他計數器，請編輯 [伺服器] 刀鋒視窗上的圖表，或開啟新的[計量瀏覽器](app-insights-metrics-explorer.md)刀鋒視窗，並新增新的圖表。 

當您編輯圖表時，可用的計數器會列為計量。

![Application Insights 中所報告的效能計數器](./media/app-insights-performance-counters/choose-performance-counters.png)

若要在一個地方查看所有最常用的圖表，請建立[儀表板](app-insights-dashboards.md)，並在其中固定這些圖表。

## <a name="add-counters"></a>新增計數器
如果您想要的效能計數器未顯示在計量清單中，則原因是 Application Insights SDK 未在 Web 伺服器中收集它。 您可以設定它執行這項作業。

1. 在伺服器上使用這個 PowerShell 命令，以找出伺服器中可用的計數器：
   
    `Get-Counter -ListSet *`
   
    (請參閱 [`Get-Counter`](https://technet.microsoft.com/library/hh849685.aspx)。)
2. 開啟 ApplicationInsights.config。
   
   * 如果您已在開發期間將 Application Insights 新增至應用程式，請編輯專案中的 ApplicationInsights.config，然後將它重新部署至伺服器。
   * 如果您已在執行階段使用狀態監視器檢測 Web 應用程式，請在 IIS 的應用程式根目錄中找到 ApplicationInsights.config。 在每個伺服器執行個體中於該處對其進行更新。
3. 編輯效能收集器指示詞：
   
```XML
   
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\Objects\Processes"/>
        <Add PerformanceCounter="\Sales(photo)\# Items Sold" ReportAs="Photo sales"/>
      </Counters>
    </Add>

```

您可以擷取標準計數器以及您自行實作的計數器。 `\Objects\Processes` 是所有 Windows 系統上都具有的標準計數器範例。 `\Sales(photo)\# Items Sold` 是可能在 Web 服務中實作的自訂計數器範例。 

格式為 `\Category(instance)\Counter"`，若是沒有執行個體的類別，則為 `\Category\Counter`。

不符合 `[a-zA-Z()/-_ \.]+` 的計數器名稱需要有 `ReportAs`；亦即，它們包含不在下列集合中的字元：字母、圓括號、正斜線、連字號、底線、空格、點。

如果您指定執行個體，系統會收集它做為報告度量的 "CounterInstanceName" 維度。

### <a name="collecting-performance-counters-in-code"></a>在程式碼中收集效能計數器
若要收集系統效能計數器並將其傳送至 Application Insights，可以採用下列程式碼片段：


``` C#

    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\.NET CLR Memory([replace-with-application-process-name])\# GC Handles", "GC Handles")));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```
或者，您可以透過您建立的自訂度量執行相同的作業︰

``` C#
    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Sales(photo)\# Items Sold", "Photo sales"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```

## <a name="performance-counters-in-analytics"></a>Analytics 中的效能計數器
您可以搜尋並顯示 [Analytics](app-insights-analytics.md) 中的效能計數器報告。

**PerformanceCounters** 結構描述會公開每個效能計數器的 `category`、`counter` 名稱和 `instance` 名稱。  在每個應用程式的遙測中，您只會看到該應用程式的計數器。 例如，若要查看哪些計數器可用︰ 

![Application Insights 分析中的效能計數器](./media/app-insights-performance-counters/analytics-performance-counters.png)

(這裡的 'Instance' 指的是效能計數器執行個體，而不是角色或伺服器電腦執行個體。 效能計數器執行個體名稱一般會將計數器分段，例如依處理序或應用程式名稱的處理器時間。)

若要取得可用記憶體在最近一段時間的圖表︰ 

![Application Insights 分析中的記憶體時間圖表](./media/app-insights-performance-counters/analytics-available-memory.png)

與其他遙測一樣，**performanceCounters** 也有 `cloud_RoleInstance` 資料行可指出應用程式執行所在主機伺服器執行個體的身分識別。 例如，若要比較不同機器上的應用程式效能︰ 

![Application Insights 分析中依角色執行個體分割的效能](./media/app-insights-performance-counters/analytics-metrics-role-instance.png)

## <a name="aspnet-and-application-insights-counts"></a>ASP.NET 和 Application Insights 計數
*例外狀況率和例外狀況計量之間的差異為何？*

*  是系統效能計數器。 CLR 會計算所有擲回之已處理和未處理的例外狀況，並依據間隔的長度將總數分割為取樣間隔。 Application Insights SDK 會收集此結果並將它傳送至入口網站。
*  是在圖表的取樣間隔中由入口網站接收之 TrackException 報告的計數。 它只包含您程式碼中撰寫 TrackException 呼叫所在位置的已處理例外狀況，並且不包含所有的 [未處理例外狀況](app-insights-asp-net-exceptions.md)。 

## <a name="performance-counters-in-aspnet-core-applications"></a>Asp.Net Core 應用程式中的效能計數器
只有在應用程式以完整的 .NET Framework 為目標時，方可支援效能計數器。 無法為 .Net Core 應用程式收集效能計數器。

## <a name="alerts"></a>警示
與其他計量一樣，您可以[設定警示](app-insights-alerts.md)，在效能計數器超出您指定的界限時提出警告。 開啟 [警示] 刀鋒視窗，然後按一下 [新增警示]。

## <a name="next"></a>後續步驟
* [相依性追蹤](app-insights-asp-net-dependencies.md)
* [例外狀況追蹤](app-insights-asp-net-exceptions.md)

