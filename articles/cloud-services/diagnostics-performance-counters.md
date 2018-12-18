---
title: 在 Azure 雲端服務中收集效能計數器 | Microsoft Docs
description: 了解如何在雲端服務中使用 Azure 診斷和 Application Insights 來探索、使用及建立效能計數器。
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: ''
ms.service: cloud-services
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/18
ms.author: jeconnoc
ms.openlocfilehash: d3aeb930dcb325aebc8c6b0a9dfde3602312618b
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001458"
---
# <a name="collect-performance-counters-for-your-azure-cloud-service"></a>為您的 Azure 雲端服務收集效能計數器

效能計數器提供一個方法，讓您追蹤您的應用程式和主機的執行程度。 Windows Server 提供與硬體、應用程式、作業系統等等相關的許多不同效能計數器。 藉由收集效能計數器並傳送至 Azure，您可以分析此資訊來協助進行更好的決策。 

## <a name="discover-available-counters"></a>探索可用的計數器

效能計數器是由兩個部分組成，集合名稱 (也稱為分類) 和一或多個計數器。 您可以使用 PowerShell 以取得可用效能計數器的清單：

```PowerShell
Get-Counter -ListSet * | Select-Object CounterSetName, Paths | Sort-Object CounterSetName

CounterSetName                                  Paths
--------------                                  -----
.NET CLR Data                                   {\.NET CLR Data(*)\SqlClient...
.NET CLR Exceptions                             {\.NET CLR Exceptions(*)\# o...
.NET CLR Interop                                {\.NET CLR Interop(*)\# of C...
.NET CLR Jit                                    {\.NET CLR Jit(*)\# of Metho...
.NET Data Provider for Oracle                   {\.NET Data Provider for Ora...
.NET Data Provider for SqlServer                {\.NET Data Provider for Sql...
.NET Memory Cache 4.0                           {\.NET Memory Cache 4.0(*)\C...
AppV Client Streamed Data Percentage            {\AppV Client Streamed Data ...
ASP.NET                                         {\ASP.NET\Application Restar...
ASP.NET Apps v4.0.30319                         {\ASP.NET Apps v4.0.30319(*)...
ASP.NET State Service                           {\ASP.NET State Service\Stat...
ASP.NET v2.0.50727                              {\ASP.NET v2.0.50727\Applica...
ASP.NET v4.0.30319                              {\ASP.NET v4.0.30319\Applica...
Authorization Manager Applications              {\Authorization Manager Appl...

#... results cut to save space ...
```

`CounterSetName` 屬性代表集合 (或分類)，而且是效能計數器與什麼相關的良好指標。 `Paths` 屬性代表集合的計數器收集。 您也可以取得 `Description` 屬性以取得計數器集合的詳細資訊。

若要取得集合的所有計數器，請使用 `CounterSetName` 值並展開 `Paths` 集合。 每個路徑項目都是您可以查詢的計數器。 例如，若要取得與 `Processor` 集合相關的可用計數器，請展開 `Paths` 集合：

```PowerShell
Get-Counter -ListSet * | Where-Object CounterSetName -eq "Processor" | Select -ExpandProperty Paths

\Processor(*)\% Processor Time
\Processor(*)\% User Time
\Processor(*)\% Privileged Time
\Processor(*)\Interrupts/sec
\Processor(*)\% DPC Time
\Processor(*)\% Interrupt Time
\Processor(*)\DPCs Queued/sec
\Processor(*)\DPC Rate
\Processor(*)\% Idle Time
\Processor(*)\% C1 Time
\Processor(*)\% C2 Time
\Processor(*)\% C3 Time
\Processor(*)\C1 Transitions/sec
\Processor(*)\C2 Transitions/sec
\Processor(*)\C3 Transitions/sec
```

這些個別計數器路徑可以新增至您的雲端服務使用的診斷架構。 如需有關效能計數器路徑建構方式的詳細資訊，請參閱[指定計數器路徑](https://msdn.microsoft.com/library/windows/desktop/aa373193(v=vs.85))。

## <a name="collect-a-performance-counter"></a>收集效能計數器

效能計數器可以針對 Azure 診斷或 Application Insights 新增至您的雲端服務。

### <a name="application-insights"></a>Application Insights

雲端服務的 Azure Application Insights 可讓您指定想要收集哪些計數器。 在您[將 Application Insights 新增至您的專案](../application-insights/app-insights-cloudservices.md#sdk)之後，名為 **ApplicationInsights.config** 的設定檔會新增至您的 Visual Studio 專案。 此設定檔會定義 Application Insights 收集何種類型的資訊並且傳送至 Azure。

開啟 **ApplicationInsights.config** 檔案並且尋找 **ApplicationInsights** > **TelemetryModules** 元素。 每個 `<Add>` 子元素會定義要收集的遙測類型，以及其設定。 效能計數器遙測模組類型是 `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector`。 如果已定義這個元素，請勿第二次新增。 每個要收集的效能計數器會定義在名為 `<Counters>` 的節點下。 以下是收集磁碟機效能計數器的範例：

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Write Bytes/sec" ReportAs="Disk write (C:)" />
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Read Bytes/sec" ReportAs="Disk read (C:)" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

每個效能計數器表示為 `<Counters>` 底下的 `<Add>` 元素。 `PerformanceCounter` 屬性會定義要收集哪個效能計數器。 `ReportAs` 屬性是要在 Azure 入口網站中針對效能計數器顯示的標題。 您收集的任何效能計數器都會放入入口網站中名為「自訂」的分類。 不同於 Azure 診斷，您無法設定收集這些效能計數器並傳送至 Azure 的間隔。 使用 Application Insights，會每分鐘收集效能計數器並傳送。 

Application Insights 會自動收集下列效能計數器：

* \Process(??APP_WIN32_PROC??)\%Processor Time
* \Memory\Available Bytes
* \.NET CLR Exceptions(??APP_CLR_PROC??)\# of Exceps Thrown / sec
* \Process(??APP_WIN32_PROC??)\Private Bytes
* \Process(??APP_WIN32_PROC??)\IO Data Bytes/sec
* \Processor(_Total)\% Processor Time

如需詳細資訊，請參閱 [Application Insights 中的系統效能計數器](../application-insights/app-insights-performance-counters.md)和 [Azure 雲端服務的 Application Insights](../application-insights/app-insights-cloudservices.md#performance-counters)。

### <a name="azure-diagnostics"></a>Azure 診斷

> [!IMPORTANT]
> 雖然此資料全都會彙總至儲存體帳戶，但是入口網站**不會**提供製作資料圖表的原生方法。 強烈建議您將其他像是 Application Insights 的診斷服務整合到您的應用程式。

雲端服務的 Azure 診斷延伸模組可讓您指定想要收集哪些計數器。 若要設定 Azure 診斷，請參閱[雲端服務監視概觀](cloud-services-how-to-monitor.md#setup-diagnostics-extension)。

您想要收集的效能計數器會在 **diagnostics.wadcfgx** 檔案中定義。 在 Visual Studio 中開啟此檔案 (針對每個角色定義) 並且尋找 **DiagnosticsConfiguration** > **PublicConfig** > **WadCfg** > **DiagnosticMonitorConfiguration** > **PerformanceCounters** 元素。 新增新的 **PerformanceCounterConfiguration** 元素作為子項目。 這個元素具有兩個屬性：`counterSpecifier` 和 `sampleRate`。 `counterSpecifier` 屬性會定義要收集哪個系統效能計數器集合 (在上一節中概述)。 `sampleRate` 值表示該值輪詢的頻率。 整體而言，所有效能計數器都會根據父代 `PerformanceCounters` 元素的 `scheduledTransferPeriod` 屬性值，傳送到 Azure。

如需有關 `PerformanceCounters` 結構描述元素的詳細資訊，請參閱 [Azure 診斷結構描述](../monitoring-and-diagnostics/azure-diagnostics-schema-1dot3-and-later.md#performancecounters-element)。

`sampleRate` 屬性定義的期間會使用 XML 持續時間資料類型來表示效能計數器輪詢的頻率。 在下列範例中，速率設為 `PT3M`，表示 `[P]eriod[T]ime[3][M]inutes`：每隔三分鐘。

如需有關 `sampleRate` 和 `scheduledTransferPeriod` 如何定義的詳細資訊，請參閱 [W3 XML 日期和時間日期類型](https://www.w3schools.com/XML/schema_dtypes_date.asp)教學課程中的**持續時間資料類型**一節。

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />

          <!-- This is a new perf counter which will track the C: disk read activity in bytes per second, every minute -->
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(C:)\Disk Read Bytes/sec" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="create-a-new-perf-counter"></a>建立新的效能計數器

可以建立新的效能計數器，供您的程式碼使用。 建立新效能計數器的程式碼必須以提升的權限執行，否則它將會失敗。 您的雲端服務 `OnStart` 啟動程式碼可以建立效能計數器，需要您在提升權限的內容中執行角色。 或者，您可以建立以提升權限執行的啟動工作，並且建立效能計數器。 如需啟動工作的詳細資訊，請參閱[如何設定和執行雲端服務的啟動工作](cloud-services-startup-tasks.md)。

若要設定您的角色以提升的權限執行，將 `<Runtime>` 元素新增至 [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) 檔案。

```xml
<ServiceDefinition name="CloudServiceLoadTesting" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Large">

    <!-- ... cut to save space ... -->

    <Runtime executionContext="elevated">
      
    </Runtime>

    <!-- ... cut to save space ... -->

  </WorkerRole>
</ServiceDefinition>
```

您可以使用少數幾行程式碼建立及註冊新的效能計數器。 使用 `System.Diagnostics.PerformanceCounterCategory.Create` 方法多載，建立分類和計數器。 下列程式碼會先檢查是否有分類存在，如果遺失，就會同時建立分類和計數器。

```csharp
using System.Diagnostics;
using Microsoft.WindowsAzure;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRoleWithSBQueue1
{
    public class WorkerRole : RoleEntryPoint
    {
        // Perf counter variable representing times service was used.
        private PerformanceCounter counterServiceUsed;

        public override bool OnStart()
        {
            // ... Other startup code here ...

            // Define the cateogry and counter names.
            string perfCounterCatName = "MyService";
            string perfCounterName = "Times Used";

            // Create the counter if needed. Our counter category only has a single counter.
            // Both the category and counter are created in the same method call.
            if (!PerformanceCounterCategory.Exists(perfCounterCatName))
            {
                PerformanceCounterCategory.Create(perfCounterCatName, "Collects information about the cloud service.", 
                                                  PerformanceCounterCategoryType.SingleInstance, 
                                                  perfCounterName, "How many times the cloud service was used.");
            }

            // Get reference to our counter
            counterServiceUsed = new PerformanceCounter(perfCounterCatName, perfCounterName);
            counterServiceUsed.ReadOnly = false;
            
            return base.OnStart();
        }

        // ... cut class code to save space
    }
}
```

當您想要使用計數器時，呼叫 `Increment` 或 `IncrementBy` 方法。

```csharp
// Increase the counter by 1
counterServiceUsed.Increment();
```

現在，您的應用程式會使用您的自訂計數器，您需要設定 Azure 診斷或 Application Insights 以追蹤計數器。


### <a name="application-insights"></a>Application Insights

如先前所述，Application Insights 的效能計數器是在 **ApplicationInsights.config** 檔案中定義。 開啟 **ApplicationInsights.config** 並尋找 **ApplicationInsights** > **TelemetryModules** > **Add** > **Counters** 元素。 建立 `<Add>` 子元素，並且將 `PerformanceCounter` 屬性設為您在程式碼中建立之效能計數器的分類和名稱。 將 `ReportAs` 屬性設為您想要在入口網站中看到的易記名稱。

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <!-- ... cut other perf counters to save space ... -->

        <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
        <Add PerformanceCounter="\MyService\Times Used" ReportAs="Service used counter" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

### <a name="azure-diagnostics"></a>Azure 診斷

如先前所述，您想要收集的效能計數器會在 **diagnostics.wadcfgx** 檔案中定義。 在 Visual Studio 中開啟此檔案 (針對每個角色定義) 並且尋找 **DiagnosticsConfiguration** > **PublicConfig** > **WadCfg** > **DiagnosticMonitorConfiguration** > **PerformanceCounters** 元素。 新增新的 **PerformanceCounterConfiguration** 元素作為子項目。 將 `counterSpecifier` 屬性設為您在程式碼中建立之效能計數器的分類和名稱。 

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <!-- ... cut other perf counters to save space ... -->
          
          <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
          <PerformanceCounterConfiguration counterSpecifier="\MyService\Times Used" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="more-information"></a>詳細資訊

- [Azure 雲端服務的 Application Insights](../application-insights/app-insights-cloudservices.md#performance-counters)
- [Application Insights 中的系統效能計數器](../application-insights/app-insights-performance-counters.md)
- [指定計數器路徑](https://msdn.microsoft.com/library/windows/desktop/aa373193(v=vs.85))
- [Azure 診斷結構描述 - 效能計數器](../monitoring-and-diagnostics/azure-diagnostics-schema-1dot3-and-later.md#performancecounters-element)