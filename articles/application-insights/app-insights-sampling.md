---
title: Azure Application Insights 中的遙測取樣 | Microsoft Docs
description: 如何讓遙測量保持在控制下。
services: application-insights
documentationcenter: windows
author: mrbullwinkle
manager: carmonm
ms.assetid: 015ab744-d514-42c0-8553-8410eef00368
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/24/2017
ms.reviewer: vitalyg
ms.author: mbullwin
ms.openlocfilehash: 893495a5c37495438d9620595719ca6e733df012
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958113"
---
# <a name="sampling-in-application-insights"></a>Application Insights 中的取樣


取樣是 [Azure Application Insights](app-insights-overview.md) 中的一個功能。 若要既能減少遙測流量和儲存空間，又能保有應用程式資料在統計上的正確分析，便建議使用此方法。 篩選器會選取相關的項目，以便您在進行診斷調查時瀏覽各個項目。
在入口網站中呈現度量計數時，就會重新正規化以考慮取樣，以將對統計資料帶來的任何影響降至最低。

取樣可減少流量與資料成本，而且可以協助您避免節流。

## <a name="in-brief"></a>簡單地說︰
* 取樣會保留「n」  筆記錄的其中 1 筆，並捨棄其餘部分。 比方說，它可能會保留 5 個事件的其中 1 個，取樣率為 20%。 
* 如果您的應用程式傳送大量遙測，ASP.NET Web 伺服器應用程式便會自動進行取樣。
* 您也可以手動設定取樣 (不論是透過入口網站的 [使用量和估計成本] 頁面；或是在 ASP.NET SDK 的 .config 檔案中；或是在 Java SDK 的 ApplicationInsights.xml 檔案中)，以便同時降低網路流量。
* 如果您有記錄自訂事件，而且想要確定某組事件已一起保留下來還是遭到捨棄，請確定它們有相同的 OperationId 值。
* 每個記錄的 `itemCount` 屬性中都會回報取樣除數 *n*，此屬性在 [搜尋] 中會出現在「要求計數」或「事件計數」等易記名稱之下。 當取樣不在作業中，則 `itemCount==1`。
* 如果您要撰寫分析查詢，請 [考慮到取樣](../log-analytics/query-language/aggregations.md)。 特別是，您應該使用 `summarize sum(itemCount)`，而非只計算記錄。

## <a name="types-of-sampling"></a>取樣類型
有三個替代的取樣方法：

* **調適性取樣** 會自動調整從您 ASP.NET 應用程式中 SDK 所傳送的遙測量。 從 SDK v 2.0.0-beta3 起，此為預設取樣方法。 調適型取樣目前僅供 ASP.NET 伺服器端遙測使用。 以完整架構作為目標的 Asp.NET Core 應用程式可從 1.0.0 版的 Microsoft.ApplicationInsights.AspNetCore SDK 使用調適型取樣。 以 NetCore 作為目標的 Asp.NET Core 應用程式可從 2.2.0-beta1 版的 Microsoft.ApplicationInsights.AspNetCore SDK 使用調適型取樣。

* **固定速率取樣**可減少從您 ASP.NET 或 Java 伺服器和使用者的瀏覽器所傳送的遙測量。 而比例則由您設定。 用戶端和伺服器會同步處理它們的取樣，讓您可以在 [搜尋] 終於相關的頁面檢視和要求之間瀏覽。
* 「擷取取樣」可在 Azure 入口網站中運作。 它會根據您設定的取樣比例，捨棄來自您應用程式的一些遙測。 這不會減少從應用程式傳送的遙測流量，但可協助您讓流量不要超過每月配額。 擷取取樣的主要優點是您不需重新部署應用程式即可設定取樣比例，並且它對所有伺服器和用戶端的運作方式都一致。 

如果自適性或固定速率取樣正在作業中，則會停用擷取取樣。

## <a name="ingestion-sampling"></a>擷取取樣
這種形式的取樣會在來自您 Web 伺服器、瀏覽器及裝置的遙測抵達 Application Insights 服務端點時開始執行。 雖然它不會減少來自您應用程式的遙測流量，但會減少 Application Insights 所處理及保留 (並收費) 的遙測量。

如果您的應用程式通常會超過每月配額，且您無法選擇使用任何一種 SDK 式的取樣類型，請使用這種類型的取樣。 

在 [使用量和估計成本] 頁面中設定取樣率：

![在 [應用程式概觀] 刀鋒視窗中，依序按一下 [設定]、[配額]、[範例]，然後選取某個取樣率，並按一下 [更新]。](./media/app-insights-sampling/04.png)

就跟其他取樣類型一樣，演算法會保留相關的遙測項目。 舉例來說，當您在 [搜尋] 中檢查遙測時，將能夠尋找與特定例外狀況相關的要求。 度量計量 (例如要求率及例外狀況率) 會正確地保留。

遭到取樣捨棄的資料點將無法在任何 Application Insights 功能中使用，例如 [連續匯出](app-insights-export-telemetry.md)。

進行 SDK 自適性或固定速率取樣時，不執行擷取取樣。 請注意，當 Visual Studio 中已啟用 ASP.NET SDK 時或是使用「狀態監視器」時，預設會啟用調適型取樣，而擷取取樣則為停用。 如果 SDK 的取樣率小於 100%，則忽略您設定的擷取取樣率。

> [!WARNING]
> 圖格上顯示的值會指出您要為擷取取樣的值。 如果 SDK 取樣在運作中，這並不代表實際的取樣率。
> 
> 

## <a name="adaptive-sampling-at-your-web-server"></a>在您 Web 伺服器上的調適性取樣
Application Insights SDK for ASP.NET v 2.0.0-beta3 及更新版本提供調適性取樣功能，且預設為啟用。 

調適性取樣會影響從您的 Web 伺服器應用程式傳送給 Application Insights 服務端點的遙測量。 系統會自動調整遙測量，以便讓流量速率不會超出指定的上限。

它無法在低遙測量的情況下運作，因此偵錯中的應用程式或使用率低的網站不會受到影響。

為了要讓遙測量達到目標，系統會捨棄部分已產生的遙測。 但就跟其他取樣類型一樣，演算法會保留相關的遙測項目。 舉例來說，當您在 [搜尋] 中檢查遙測時，將能夠尋找與特定例外狀況相關的要求。 

度量計量 (例如要求率及例外狀況率) 會受到調整來補償取樣率，讓它們能在計量瀏覽器中顯示大致上正確的值。

### <a name="update-nuget-packages"></a>更新 NuGet 套件 ###

將您專案的 NuGet 封裝更新至 Application Insights 的最新*發行前*版本。 在 Visual Studio 中，以滑鼠右鍵按一下方案總管中的專案，選擇 [管理 NuGet 封裝]，然後核取 [包含發行前版本]，並搜尋 Microsoft.ApplicationInsights.Web。 

### <a name="configuring-adaptive-sampling"></a>使用調適性取樣 ###

在 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 中，您可以調整 `AdaptiveSamplingTelemetryProcessor` 節點中的數個參數。 顯示的數字是預設值：

* `<MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>`
  
    調整演算法對於 **每部伺服器主機**的目標速率。 如果 Web 應用程式在許多主機上執行，請減少此值，以保持在您的 Application Insights 入口網站的流量目標速率內。
* `<EvaluationInterval>00:00:15</EvaluationInterval>` 
  
    目前遙測速率的間隔已重新評估。 評估是以移動平均來執行。 如果您的遙測會突然暴增，您可能想要縮短此間隔。
* `<SamplingPercentageDecreaseTimeout>00:02:00</SamplingPercentageDecreaseTimeout>`
  
    當取樣百分比值變更時，多久之後我們可以降低取樣百分比，以擷取較少的資料。
* `<SamplingPercentageIncreaseTimeout>00:15:00</SamplingPercentageIncreaseTimeout>`
  
    當取樣百分比值變更時，多久之後我們可以增加取樣百分比，以擷取較多的資料。
* `<MinSamplingPercentage>0.1</MinSamplingPercentage>`
  
    隨著取樣百分比改變，我們可以設定的最小值是多少。
* `<MaxSamplingPercentage>100.0</MaxSamplingPercentage>`
  
    隨著取樣百分比改變，我們可以設定的最大值是多少。
* `<MovingAverageRatio>0.25</MovingAverageRatio>` 
  
    在計算移動平均時，指派給最新的值的權數。 使用等於或小於 1 的值。 較小的值會讓演算法不易受突然的變更影響。
* `<InitialSamplingPercentage>100</InitialSamplingPercentage>`
  
    當應用程式剛開始時指派的值。 不要在偵錯時減少此值。 

* `<ExcludedTypes>Trace;Exception</ExcludedTypes>`
  
    不要進行取樣的分號分隔類型清單。 可辨識的類型為：相依性、事件、例外狀況、頁面檢視、要求、追蹤。 會傳送所指定類型的所有執行個體；會針對未指定的類型進行取樣。

* `<IncludedTypes>Request;Dependency</IncludedTypes>`
  
    要進行取樣的分號分隔類型清單。 可辨識的類型為：相依性、事件、例外狀況、頁面檢視、要求、追蹤。 會針對指定的類型進行取樣；將一律會傳輸其他類型的所有執行個體。


**若要關閉**調適型取樣，請將 AdaptiveSamplingTelemetryProcessor 節點從 applicationinsights-config 移除。

### <a name="alternative-configure-adaptive-sampling-in-code"></a>替代方法：在程式碼中設定調適性取樣
除了在 .config 檔中設定取樣參數之外，您還可以透過程式設計的方式來設定這些值。 這可讓您指定在每次取樣率重新評估時叫用的回呼函式。 例如，您可以使用這個方法來找出使用中的取樣率。

移除 .config 檔案中的 `AdaptiveSamplingTelemetryProcessor` 節點。

*C#*

```csharp

    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.Channel.Implementation;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    ...

    var adaptiveSamplingSettings = new SamplingPercentageEstimatorSettings();

    // Optional: here you can adjust the settings from their defaults.

    var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    builder.UseAdaptiveSampling(
         adaptiveSamplingSettings,

        // Callback on rate re-evaluation:
        (double afterSamplingTelemetryItemRatePerSecond,
         double currentSamplingPercentage,
         double newSamplingPercentage,
         bool isSamplingPercentageChanged,
         SamplingPercentageEstimatorSettings s
        ) =>
        {
          if (isSamplingPercentageChanged)
          {
             // Report the sampling rate.
             telemetryClient.TrackMetric("samplingPercentage", newSamplingPercentage);
          }
      });

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();

```

([深入了解遙測處理器](app-insights-api-filtering-sampling.md#filtering))。

<a name="other-web-pages"></a>

## <a name="sampling-for-web-pages-with-javascript"></a>具有 JavaScript 的網頁的取樣
您可以從任何伺服器設定固定取樣率的網頁。 

當您[設定 Application Insights 的網頁](app-insights-javascript.md)時，請修改您從 Application Insights 入口網站取得的 JavaScript 片段。 (在 ASP.NET 應用程式中，程式碼片段通常會出現在 _Layout.cshtml。)在檢測金鑰之前插入類似 `samplingPercentage: 10,` 的一行：

    <script>
    var appInsights= ... 
    }({ 


    // Value must be 100/N where N is an integer.
    // Valid examples: 50, 25, 20, 10, 5, 1, 0.1, ...
    samplingPercentage: 10, 

    instrumentationKey:...
    }); 

    window.appInsights=appInsights; 
    appInsights.trackPageView(); 
    </script> 

針對取樣百分比，選擇接近 100/N 的百分比，其中 N 是整數。  目前取樣並不支援其他值。

如果您也在伺服器啟用固定比例取樣，用戶端和伺服器就會同步，讓您可以在 [搜尋] 中的相關頁面檢視與要求之間瀏覽。

## <a name="fixed-rate-sampling-for-aspnet-and-java-web-sites"></a>適用於 ASP.NET 和 Java 網站的固定速率取樣
固定取樣率會減少從您的 Web 伺服器及網頁瀏覽器所傳送的流量。 但它會依照您設定的速率來降低遙測，這與調適性取樣不同。 它也會同步處理用戶端及伺服器取樣，讓相關項目能夠保留；舉例來說，當您在 [搜尋] 中查看頁面檢視時，可以尋找其相關要求。

取樣演算法會保留相關項目。 對於每個 HTTP 要求事件，該要求及其相關事件會一併遭到捨棄或傳輸。 

在計量瀏覽器中，速率 (例如要求及例外狀況數) 會乘以某個係數來補償取樣率，讓它們能大致上正確。

### <a name="configuring-fixed-rate-sampling-in-aspnet"></a>在 ASP.NET 中設定固定速率取樣 ###

1. **將您專案的 NuGet 套件更新**至 Application Insights 的最新「發行前」版本。 在 Visual Studio 中，以滑鼠右鍵按一下方案總管中的專案，選擇 [管理 NuGet 封裝]，然後核取 [包含發行前版本]，並搜尋 Microsoft.ApplicationInsights.Web。 
2. **停用調適性取樣**：在 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 中，將 `AdaptiveSamplingTelemetryProcessor` 節點移除或設成註解。
   
    ```xml
   
    <TelemetryProcessors>
   
    <!-- Disabled adaptive sampling:
      <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
        <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
      </Add>
    -->

    ```

3. **啟用固定取樣率模組。** 將此程式碼片段新增至 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md)：
   
    ```XML
   
    <TelemetryProcessors>
     <Add  Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
   
      <!-- Set a percentage close to 100/N where N is an integer. -->
     <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
      <SamplingPercentage>10</SamplingPercentage>
      </Add>
    </TelemetryProcessors>
   
    ```

### <a name="configuring-fixed-rate-sampling-in-java"></a>在 JAVA 中設定固定速率取樣 ###

1. 使用最新的 [Application Insights Java SDK](app-insights-java-get-started.md) 下載及設定您的 Web 應用程式

2. 將下列程式碼片段加入 ApplicationInsights.xml 檔案以**啟用固定速率取樣模組**。

```XML
    <TelemetryProcessors>
        <BuiltInProcessors>
            <Processor type = "FixedRateSamplingTelemetryProcessor">
                <!-- Set a percentage close to 100/N where N is an integer. -->
                <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
                <Add name = "SamplingPercentage" value = "50" />
            </Processor>
        </BuilrInProcessors>
    <TelemetryProcessors/>
```

3. 您可以在 Processor 標籤 "FixedRateSamplingTelemetryProcessor" 中使用下列標籤，在取樣中包括 (Include) 或排除 (Exclude) 特定的遙測類型
```XML
    <ExcludedTypes>
        <ExcludedType>Request</ExcludedType>
    </ExcludedTypes>

    <IncludedTypes>
        <IncludedType>Exception</IncludedType>
    </IncludedTypes>
```
可以在取樣中包括或排除的遙測類型為：Dependency、Event、Exception、PageView、Request 和 Trace。

> [!NOTE]
> 針對取樣百分比，選擇接近 100/N 的百分比，其中 N 是整數。  目前取樣並不支援其他值。
> 
> 

### <a name="alternative-enable-fixed-rate-sampling-in-your-server-code"></a>替代方法：在伺服器程式碼中啟用固定取樣率
除了在 .config 檔中設定取樣參數之外，您還可以透過程式設計的方式來設定這些值。 

*C#*

```csharp

    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    ...

    var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;
    builder.UseSampling(10.0); // percentage

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();

```

([深入了解遙測處理器](app-insights-api-filtering-sampling.md#filtering))。

## <a name="when-to-use-sampling"></a>何時使用取樣？
如果您使用 ASP.NET SDK 版本 2.0.0-beta3 或更新版本，調適性取樣會自動啟用。 無論您使用哪個版本的 SDK，都可以啟用擷取取樣，允許 Application Insights 對收集的資料取樣。

預設 Java SDK 中不會啟用任何取樣。 目前，它只支援固定速率取樣。 Java SDK 中不支援調適型取樣。

通常，對於大多數小型和中型的應用程式，您不需要取樣。 最有用的診斷資訊和最準確的統計資料會是透過收集所有使用者活動的資料取得。 

取樣的主要優點如下：

* 當您的應用程式在短時間間隔傳送非常高比率的遙測時，Application Insights 服務會將資料點卸除 (「節流」)。 
* 保持在定價層的資料點 [配額](app-insights-pricing.md) 內。 
* 若要從收集的遙測降低網路流量。 

### <a name="which-type-of-sampling-should-i-use"></a>我應該使用哪種類型的取樣？
**如果是下列情形，請使用擷取取樣：**

* 您的遙測經常會超出每月配額。
* 您使用不支援取樣的 SDK 版本，例如比 ASP.NET 版本 2 早的版本。
* 您收到大量來自使用者網頁瀏覽器的遙測。

**如果是下列情形，則使用固定取樣率：**

* 您使用 Application Insights SDK for ASP.NET Web 服務版本 2.0.0 或更新版本或 Java SDK v2.0.1 或更新版本，且
* 您想要同步處理用戶端與伺服器之間的取樣，因此，當您在 [搜尋](app-insights-diagnostic-search.md)中調查事件時，您可以在用戶端與伺服器的相關事件之間調查，例如頁面檢視和 HTTP 要求。
* 您對於您的應用程式的適當取樣百分比有信心。 應該夠高以取得精確的度量，但是低於超過價格配額和節流限制的取樣率。 

**使用調適性取樣：**

如果欲使用其他形式取樣但條件不適用，建議使用適應型取樣。 在 ASP.NET 伺服器 SDK 版本 2.0.0-beta3 或更新版本中，此功能為預設啟用。 在達到一定的比率下限之前，不會減少流量，因此低使用率的網站不會受到影響。

## <a name="how-do-i-know-whether-sampling-is-in-operation"></a>如何得知取樣是否正在運作中？
若要找出實際的取樣率 (不論是否已套用)，請使用如下所示的 [分析查詢](app-insights-analytics.md) ︰

```
union * 
| where timestamp > ago(1d)
| summarize 100/avg(itemCount) by bin(timestamp, 1h), itemType
| render timechart 
```

在每筆保留的記錄中， `itemCount` 表示它所代表的原始記錄筆數，其等於 1 + 先前捨棄的記錄筆數。 

## <a name="how-does-sampling-work"></a>取樣運作方式？
固定速率取樣是從 ASP.NET 版本 2.0.0 的 SDK 和 Java SDK 版本 2.0.1 及更新版本開始有的功能。 調適型取樣是 ASP.NET 版本 (從 2.0.0 及更新版本開始) 中 SDK 的一個功能。 擷取取樣是 Application Insights 服務的一項功能，而且可在 SDK 未執行取樣時運作。 

採樣演算法會決定要捨棄哪些遙測項目，以及要保留哪些遙測項目 (其是否位於 SDK 或 Application Insights 服務中)。 取樣決策會根據數個規則，目標是要保留相關的資料點不變，在 Application Insights 中保有可採取動作而且即使有縮減資料集仍可靠的診斷經驗。 比方說，如果是失敗的要求，您的應用程式會傳送其他遙測項目 (例如從此要求記錄的例外狀況和追蹤)，取樣將不會分割此要求和其他遙測。 它會一起保留或卸除。 如此一來，當您在 Application Insights 中查看要求詳細資料時，您一律可以看到要求和其相關聯的遙測項目。 

取樣決策是根據要求的作業識別碼，這表示屬於特定作業的所有遙測項目不是保留下來，就是已捨棄。 對於未設定作業識別碼的遙測項目 (例如非同步執行緒報告、不具使用任何 http 內容的遙測項目)，取樣只會擷取每種類型的遙測項目的某個百分比。 在 2.5.0-beta2 版的 .NET SDK 和 2.2.0-beta3 版的 ASP.NET Core SDK 之前，取樣決策是根據定義「使用者」的應用程式 (也就是最常見的 Web 應用程式) 的使用者識別碼雜湊。 針對沒有定義使用者的應用程式類型 (例如 Web 服務)，取樣決策係根據要求的作業識別碼。

呈現遙測回來給您時，Application Insights 服務會以收集時使用的相同取樣百分比調整度量，來彌補遺漏的資料點。 因此，在 Application Insights 中查看遙測時，使用者會看到統計正確也非常接近實際數的近似值。

近似值的精確度絕大部分取決於設定的取樣百分比。 此外，對於處理大量使用者的通常類似要求的應用程式，其精確度會增加。 相反地，對於不處理大量負載的應用程式，就不需要取樣，因為這些應用程式通常可以傳送遙測同時保持在配額內，而不會因節流造成資料遺失。 

> [!WARNING]
> Application Insights 不會對計量和工作階段遙測類型進行取樣。 這些遙測類型非常不想要降低精確度。
> 

### <a name="adaptive-sampling"></a>調適性取樣
調適性取樣會新增元件，該元件會監視 SDK 的目前傳輸速率，並調整取樣百分比，以嘗試保持在目標最大速率。 調整會定期重新計算，並且根據外寄傳輸速率的移動平均。

## <a name="sampling-and-the-javascript-sdk"></a>取樣與 JavaScript SDK
用戶端 (JavaScript) SDK 與伺服器端 SDK 一同參與固定取樣率。 已檢測的頁面只會從伺服器端決定「納入取樣」的相同使用者傳送用戶端遙測。 此邏輯的設計是為了在用戶端和伺服器端之間保有使用者工作階段的完整性。 如此一來，您可以從 Application Insights 中的任何特定遙測項目找到這個使用者或工作階段的所有其他遙測項目。 

*我的用戶端和伺服器端遙測未顯示您上方描述的協調範例。*

* 請確認您在伺服器及用戶端上都已啟用固定取樣率。
* 確定 SDK 版本為 2.0 或更新版本。
* 請檢查您的用戶端和伺服器中設定相同的取樣百分比。

## <a name="frequently-asked-questions"></a>常見問題集
*為什麼不取樣簡單的「收集每個遙測類型百分之 X」？*

* 雖然這個取樣方法會提供具有極高精確度的度量近似值，它會破壞根據每個使用者、工作階段和要求相互關聯資料的能力，而這對於診斷是非常重要。 因此，對於「收集應用程式使用者百分之 X 的所有遙測項目」或「收集應用程式要求百分之 X 的所有遙測」邏輯，取樣的效果更佳。 對於與要求無關聯的遙測項目 (例如背景非同步處理)，改為「收集每個遙測類型百分之 X 的所有項目」。 

*取樣百分比會隨著時間變更嗎？*

* 是的，調適性取樣會根據目前觀察到的遙測量，逐漸變更取樣百分比。

*如果我使用固定取樣率，如何知道哪個取樣百分比最適合我的應用程式？*

* 開始使用調適性取樣的其中一個方法，就是找出它選擇的取樣率 (請參閱上一個問題)，然後再切換為使用該取樣率的固定取樣率。 
  
    否則，您就必須猜測。 分析 Application Insights 中您目前的遙測使用量、觀察目前的節流，並估計所收集之遙測的量。 這三項輸入與所選定價層，可對您可能想要減少收集的遙測量提出建議。 不過，使用者數目的增加或遙測量的其他某些變化可能會讓您的評估失效。

*如果將取樣百分比設定成太低會發生什麼事？*

* 當 Application Insights 嘗試補償減少資料量縮減的資料視覺效果時，過度低的取樣百分比 (過度積極取樣) 會降低近似值的精確度。 此外，診斷經驗可能會有負面影響，因為可能會出取樣出某些不常失敗或緩慢的要求。

*如果將取樣百分比設定成太高會發生什麼事？*

* 設定太高的取樣百分比 (不夠積極) 會導致收集的遙測量減少不足。 您可能仍會遇到與節流相關的遙測資料遺失，而使用 Application Insights 的成本由於超額費可能高於您的計劃。

*我可以在何種平台上使用取樣？*

* 如果 SDK 未執行取樣，則擷取取樣會在任何遙測超過特定數量時自動運作。 例如，如果您使用舊版 ASP.NET SDK 或舊版 Java SDK (1.0.10 或更舊版本)，這應該可行。
* 如果您使用 ASP.NET SDK 版本 2.0.0 和更新版本 (裝載於 Azure 或您自己的伺服器上)，您預設會得到調適性取樣，但您可以切換到固定取樣率 (如上所述)。 使用固定取樣率，瀏覽器 SDK 會自動同步至取樣相關的事件。 
* 如果您使用 Java SDK 版本 2.0.1 或更新版本，就可以設定 ApplicationInsights.xml 來開啟固定速率取樣。 根據預設取樣功能為關閉。 使用固定取樣率，瀏覽器 SDK 會自動同步至取樣相關的事件。

*我一律想要看見特定罕見的事件。我要如何讓它們通過取樣模組？*

* 使用新的 TelemetryConfiguration (非預設使用中的組態) 初始化個別的 TelemetryClient 執行個體。 使用該執行個體來傳送您的罕見的事件。

## <a name="next-steps"></a>後續步驟
* [篩選](app-insights-api-filtering-sampling.md) 可以對您的 SDK 所傳送的內容，提供更嚴格的控制。

