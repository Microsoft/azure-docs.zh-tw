---
title: Azure Application Insights 常見問題集 | Microsoft Docs
description: 關於 Application Insights 的常見問題集。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 0e3b103c-6e2a-4634-9e8c-8b85cf5e9c84
ms.service: application-insights
ms.workload: mobile
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 04/12/2017
ms.author: mbullwin
ms.openlocfilehash: 8ef0ea20256807650f19d25aa77276cb8cbcb7d7
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "35632971"
---
# <a name="application-insights-frequently-asked-questions"></a>Application Insights：常見問題集

## <a name="configuration-problems"></a>組態問題
*我在設定下列項目時有問題：*

* [.NET 應用程式](app-insights-asp-net-troubleshoot-no-data.md)
* [監視已在執行的應用程式](app-insights-monitor-performance-live-website-now.md#troubleshooting-runtime-configuration-of-application-insights)
* [Azure 診斷](app-insights-azure-diagnostics.md)
* [Java Web 應用程式](app-insights-java-troubleshoot.md)

我的伺服器沒有傳回資料

* [設定防火牆例外狀況](app-insights-ip-addresses.md)
* [設定 ASP.NET 伺服器](app-insights-monitor-performance-live-website-now.md)
* [設定 Java 伺服器](app-insights-java-agent.md)

## <a name="can-i-use-application-insights-with-"></a>我是否可以搭配 ... 來使用 Application Insights 嗎？

* [IIS 伺服器 (內部部署或 VM 中) 上的 Web 應用程式](app-insights-asp-net.md)
* [Java Web 應用程式](app-insights-java-get-started.md)
* [Node.js 應用程式](app-insights-nodejs.md)
* [Azure 上的 Web 應用程式](app-insights-azure-web-apps.md)
* [Azure 上的雲端服務](app-insights-cloudservices.md)
* [在 Docker 中執行的應用程式伺服器](app-insights-docker.md)
* [單一頁面 Web 應用程式](app-insights-javascript.md)
* [Sharepoint](app-insights-sharepoint.md)
* [Windows 傳統型應用程式](app-insights-windows-desktop.md)
* [其他平台](app-insights-platforms.md)

## <a name="is-it-free"></a>它是免費的嗎？

是，可作為實驗用途。 在基本定價方案中，您的應用程式每個月可以免費傳送一定額度的資料。 免費額度足夠因應開發用途和為少量使用者發佈應用程式。 您可以設定上限以防止處理超過指定的資料量。

較大量的遙測資料是以 Gb 計費。 我們提供一些有關如何[限制費用](app-insights-pricing.md)的秘訣。

「企業」方案會針對每個 Web 伺服器節點傳送遙測資料的每一天計費。 如果您想要大規模使用「連續匯出」，便適用此方案。

[請參閱定價方案](https://azure.microsoft.com/pricing/details/application-insights/)。

## <a name="how-much-is-it-costing"></a>費用是多少？

* 在 Application Insights 資源中開啟 [使用量和估計成本] 頁面。 系統會顯示一張最近使用量的圖表。 您可以視需要設定資料量上限。
* 開啟 [Azure 帳單刀鋒視窗](https://portal.azure.com/#blade/Microsoft_Azure_Billing/BillingBlade/Overview)以查看您所有資源的帳單。

## <a name="q14"></a>Application Insights 在我的專案中修改什麼？
詳細資料視專案類型而定。 若是 Web 應用程式：

* 將這些檔案新增至您的專案：

  * ApplicationInsights.config。
  * ai.js
* 安裝這些 NuGet 套件：

  * *Application Insights API* - 核心 API
  * *Application Insights API for Web Applications* - 用來從伺服器傳送遙測
  * *Application Insights API for JavaScript Applications* - 用來從用戶端傳送遙測

    套件包含這些組件：
  * Microsoft.ApplicationInsights
  * Microsoft.ApplicationInsights.Platform
* 將項目插入至：

  * Web.config
  * packages.config
* (僅限新專案 - 如果您[將 Application Insights 新增至現有的專案][start]，則必須手動執行。)將程式碼片段插入至用戶端和伺服器程式碼中，以使用 Application Insights 資源識別碼將這些片段初始化。 例如，在 MVC 應用程式中，程式碼會插入至主版頁面 Views/Shared/_Layout.cshtml

## <a name="how-do-i-upgrade-from-older-sdk-versions"></a>如何從舊版 SDK 升級？
請參閱您的應用程式類型所適用之 SDK 的[版本資訊](app-insights-release-notes.md)。

## <a name="update"></a>如何變更我的專案將資料傳送到哪一個 Azure 資源？
在 [方案總管] 中，以滑鼠右鍵按一下 `ApplicationInsights.config`，然後選擇 [更新 Application Insights]。 您可以將資料傳送至 Azure 中的現有資源或新資源。 更新精靈會變更 ApplicationInsights.config 中的檢測金鑰，這決定伺服器 SDK 將您的資料送往何處。 除非您取消選取 [全部更新]，否則也會變更金鑰出現在您網頁中的位置。

## <a name="what-is-status-monitor"></a>什麼是狀態監視器？

一個傳統型應用程式，您可以在 IIS Web 伺服器中使用它，以協助您在 Web 應用程式中設定 Application Insights。 它不會收集遙測資料：當您沒有在設定應用程式時，可以將它停止。 

[深入了解](app-insights-monitor-performance-live-website-now.md#questions)。

## <a name="what-telemetry-is-collected-by-application-insights"></a>Application Insights 會收集什麼遙測資料？

從伺服器 Web 應用程式：

* HTTP 要求
* [相依項目](app-insights-asp-net-dependencies.md)。 對下列各項的呼叫：SQL Database、對外部服務的 HTTP 呼叫、Azure Cosmos DB、資料表、Blob 儲存體及佇列。 
* [例外狀況](app-insights-asp-net-exceptions.md)和堆疊追蹤。
* [效能計數器](app-insights-performance-counters.md) - 如果您使用[狀態監視器](app-insights-monitor-performance-live-website-now.md)、[Azure 監視](app-insights-azure-web-apps.md)或 [Application Insights collectd 寫入器](app-insights-java-collectd.md)。
* 您以程式碼撰寫的[自訂事件和計量](app-insights-api-custom-events-metrics.md)。
* [追蹤記錄檔](app-insights-asp-net-trace-logs.md) - 如果您設定適當的收集器。

從[用戶端網頁](app-insights-javascript.md)：

* [頁面檢視計數](app-insights-web-track-usage.md)
* [AJAX 呼叫](app-insights-asp-net-dependencies.md) - 從執行中指令碼發出的要求。
* 頁面檢視載入資料
* 使用者和工作階段計數
* [已驗證的使用者識別碼](app-insights-api-custom-events-metrics.md#authenticated-users)

從其他來源 (如果您設定它們的話)：

* [Azure 診斷](app-insights-azure-diagnostics.md)
* [Docker 容器](app-insights-docker.md)
* [將資料表匯入到分析](app-insights-analytics-import.md)
* [Log Analytics](https://azure.microsoft.com/blog/omssolutionforappinsightspublicpreview/)
* [Logstash](app-insights-analytics-import.md)

## <a name="can-i-filter-out-or-modify-some-telemetry"></a>我是否可以篩選掉或修改某些遙測？

是，您可以在伺服器中撰寫：

* 「遙測處理器」以在從您的應用程式傳送選取的遙測項目之前，先篩選它們的屬性或為它們新增屬性。
* 「初始設定式」以為所有遙測項目新增屬性。

深入了解 [ASP.NET](app-insights-api-filtering-sampling.md) 或 [Java](app-insights-java-filter-telemetry.md)。

## <a name="how-are-city-country-and-other-geo-location-data-calculated"></a>如何計算城市、國家/地區及其他地理位置的資料？

我們會使用 [GeoLite2](http://dev.maxmind.com/geoip/geoip2/geolite2/) 來查詢 Web 用戶端的 IP 位址 (IPv4 或 IPv6)。

* 瀏覽器遙測：我們會收集傳送者的 IP 位址。
* 伺服器遙測：Application Insights 模組會收集用戶端 IP 位址。 如果已設定 `X-Forwarded-For`，則不會收集該位址。

您可以設定 `ClientIpHeaderTelemetryInitializer` 以從不同的標頭取得 IP 位址。 例如，在某些系統中，Proxy、負載平衡器或 CDN 會將它移至 `X-Originating-IP`。 [深入了解](http://apmtips.com/blog/2016/07/05/client-ip-address/)。

您可以[使用 Power BI](app-insights-export-power-bi.md) 在地圖上顯示您的要求遙測資料。


## <a name="data"></a>資料會保留在入口網站多久？ 是否安全？
請參閱[資料保留和隱私權][data]。

## <a name="might-personally-identifiable-information-pii-be-sent-in-the-telemetry"></a>是否可能在遙測資料中傳送個人識別資訊 (PII)？

如果您的程式碼會傳送這類資料，就有可能。 如果堆疊追蹤中的變數包含 PII，也可能發生這種情況。 您的開發小組應該進行風險評估，以確保正確處理 PII。 [深入了解資料保留和隱私權](app-insights-data-retention-privacy.md)。

在查閱地理位置屬性之後，用戶端 Web 位址的**所有**八位元資料一律會設定為 0。

## <a name="my-ikey-is-visible-in-my-web-page-source"></a>在我的網頁原始碼中可以看見我的 iKey。 

* 這在監視解決方案中是常見的做法。
* 無法使用它來竊取您的資料。
* 可以使用它來扭曲您的資料或觸發警示。
* 我們尚未聽到有任何客戶有這類問題。

您可以：

* 針對用戶端和伺服器資料使用兩個個別的 iKey (個別的 Application Insights 資源)。 或
* 撰寫一個在您伺服器中執行的 Proxy，並讓 Web 用戶端透過該 Proxy 傳送資料。

## <a name="post"></a>如何在診斷搜尋中查看 POST 資料？
我們不會自動記錄 POST 資料，但您可以使用 TrackTrace 呼叫：將資料放置到訊息參數中。 相較於字串屬性的限制，此呼叫可容許較長的大小，不過您無法篩選。

## <a name="should-i-use-single-or-multiple-application-insights-resources"></a>我應該使用單一還是多個 Application Insights 資源？

對於單一商務系統中的所有元件或角色，請使用單一資源。 對於開發、測試和發行版本以及獨立的應用程式，則請使用不同的資源。

* [請參閱這裡的討論](app-insights-separate-resources.md)
* [範例 - 具有背景工作角色和 Web 角色的雲端服務](app-insights-cloudservices.md)

## <a name="how-do-i-dynamically-change-the-instrumentation-key"></a>我如何動態變更檢測金鑰？

* [這裡的討論](app-insights-separate-resources.md)
* [範例 - 具有背景工作角色和 Web 角色的雲端服務](app-insights-cloudservices.md)

## <a name="what-are-the-user-and-session-counts"></a>什麼是使用者和和工作階段計數？

* JavaScript SDK 會在 Web 用戶端上設定使用者 Cookie 來識別傳回使用者，以及設定一個工作階段 Cookie 來將活動分組。
* 如果沒有任何用戶端指令碼，則您可以[在伺服器設定 Cookie](http://apmtips.com/blog/2016/07/09/tracking-users-in-api-apps/)。
* 如果有一個真實的使用者以不同的瀏覽器使用您的站台，或是使用 InPrivate/Incognito 瀏覽，或透過不同的電腦，則系統會將其計算多次。
* 若要跨電腦和瀏覽器識別登入的使用者，請新增對 [setAuthenticatedUserContext()](app-insights-api-custom-events-metrics.md#authenticated-users) 的呼叫。

## <a name="q17"></a> 我是否已啟用 Application Insights 中的所有項目？
| 您應該會看到 | 如何取得 | 取得原因 |
| --- | --- | --- |
| 可用性圖表 |[Web 測試](app-insights-monitor-web-app-availability.md) |知道您的 Web 應用程式已啟動 |
| 伺服器應用程式效能：回應時間... |[將 Application Insights 新增至您的專案](app-insights-asp-net.md)或[在伺服器上安裝 AI 狀態監視器](app-insights-monitor-performance-live-website-now.md) (或撰寫您自己的程式碼來[追蹤相依性](app-insights-api-custom-events-metrics.md#trackdependency)) |偵測效能問題 |
| 相依性遙測 |[在伺服器上安裝 AI 狀態監視器](app-insights-monitor-performance-live-website-now.md) |診斷資料庫或其他外部元件的問題 |
| 取得例外狀況的堆疊追蹤 |[在程式碼中插入 TrackException 呼叫](app-insights-asp-net-exceptions.md) (但部分會自動報告) |偵測並診斷例外狀況 |
| 搜尋記錄追蹤 |[新增記錄配接器](app-insights-asp-net-trace-logs.md) |診斷例外狀況、效能問題 |
| 用戶端使用基本概念：頁面檢視、工作階段... |[網頁中的 JavaScript 初始設定式](app-insights-javascript.md) |流量分析 |
| 用戶端自訂度量 |[追蹤網頁中的呼叫](app-insights-api-custom-events-metrics.md) |增強使用者經驗 |
| 伺服器自訂度量 |[追蹤伺服器中的呼叫](app-insights-api-custom-events-metrics.md) |商業智慧 |

## <a name="why-are-the-counts-in-search-and-metrics-charts-unequal"></a>為什麼 [搜尋] 和 [計量] 圖表中的計數不相等？

[取樣](app-insights-sampling.md)會減少從您應用程式實際傳送到入口網站的遙測項目 (要求、自訂事件等等) 數目。 在「搜尋」中，您會看到實際收到的項目數目。 在顯示事件計數的計量圖表中，您會看到發生的原始事件數目。 

每個傳輸的項目都帶有一個 `itemCount` 屬性，此屬性會顯示該項目代表的原始事件數目。 若要觀察取樣的運作情況，您可以在 [分析] 中執行下列查詢：

```
    requests | summarize original_events = sum(itemCount), transmitted_events = count()
```


## <a name="automation"></a>自動化

### <a name="configuring-application-insights"></a>設定 Application Insights

您可以[撰寫 PowerShell 指令碼](app-insights-powershell.md)以使用「Azure 資源監視器」來：

* 建立和更新 Application Insights 資源。
* 設定定價方案。
* 取得檢測金鑰。
* 新增計量警示。
* 新增可用性測試。

您無法設定「計量瀏覽器」報告或設定連續匯出。

### <a name="querying-the-telemetry"></a>查詢遙測

請使用 [REST API](https://dev.applicationinsights.io/) 來執行[分析](app-insights-analytics.md)查詢。

## <a name="how-can-i-set-an-alert-on-an-event"></a>我要如何在事件上設定警示？

Azure 警示僅針對計量。 請建立一個會在每次事件發生時超出值臨界值的自訂計量。 然後在該計量上設定警示。 請注意：每次該計量超出任一方向的臨界值時，您都會收到通知；不論初始值是高還是低，您都必須等到第一次超出時，才會收到通知；一律都會有幾分鐘的延遲。

## <a name="are-there-data-transfer-charges-between-an-azure-web-app-and-application-insights"></a>Azure Web 應用程式與 Application Insights 之間的資料傳輸需要收費嗎？

* 如果您的 Azure Web 應用程式是裝載在具有 Application Insights 集合端點的資料中心內，就不會有費用。 
* 如果您的裝載資料中心內沒有任何集合端點，您應用程式的遙測資料就會產生 [Azure 傳出費用](https://azure.microsoft.com/pricing/details/bandwidth/)。

這並不取決於您 Application Insights 資源的裝載位置。 而是取決於我們端點的分佈。

## <a name="can-i-send-telemetry-to-the-application-insights-portal"></a>我是否可以將遙測資料傳送給 Application Insights 入口網站？

建議您使用我們的 SDK 和使用 [SDK API](app-insights-api-custom-events-metrics.md)。 SDK 有適用於各種[平台](app-insights-platforms.md)的各種變體。 這些 SDK 可處理緩衝、壓縮、節流、重試等。 不過，[擷取結構描述](https://github.com/Microsoft/ApplicationInsights-dotnet/tree/develop/Schema/PublicSchema)和[端點通訊協定](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/EndpointSpecs/ENDPOINT-PROTOCOL.md)是公用的。

## <a name="can-i-monitor-an-intranet-web-server"></a>我是否可以監視內部網路 Web 伺服器？

有兩種方式：

### <a name="firewall-door"></a>防火牆門

允許您的 Web 伺服器將遙測傳送至我們的端點 https://dc.services.visualstudio.com:443 和 https://rt.services.visualstudio.com:443。 

### <a name="proxy"></a>Proxy

透過覆寫範例 ApplicationInsights.config 中的這些設定，即可將流量從您的伺服器路由到內部網路上的閘道。如果這些 "Endpoint" 屬性並未出現在您的設定中，則這些類別將會使用下列範例中顯示的預設值。

#### <a name="example-applicationinsightsconfig"></a>範例 ApplicationInsights.config：
```xml
<ApplicationInsights>
    ...
    <TelemetryChannel>
         <EndpointAddress>https://dc.services.visualstudio.com/v2/track</EndpointAddress>
    </TelemetryChannel>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
        <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

請注意，ApplicationIdProvider 從 v2.6.0 才開始提供

您的閘道應該將流量路由到 https://dc.services.visualstudio.com:443

用下列的內容取代上述的值：`http://<your.gateway.address>/<relative path>`
 
範例： 
```
http://<your.gateway.endpoint>/v2/track 
http://<your.gateway.endpoint>/api/profiles/{0}/apiId
```




## <a name="can-i-run-availability-web-tests-on-an-intranet-server"></a>我是否可以在內部網路伺服器上執行可用性 Web 測試？

我們的 [Web 測試](app-insights-monitor-web-app-availability.md)是在分散於全球各地的網路節點上執行。 有兩個解決方案：

* 防火牆門 - 允許從[這個又長又可變更的 Web 測試代理程式清單](app-insights-ip-addresses.md)傳送要求給您的伺服器。
* 撰寫您自己的程式碼，以從內部網路內傳送定期要求給您的伺服器。 您可以針對這個目的執行 Visual Studio Web 測試。 測試者可以使用 TrackAvailability() API 將結果傳送給 Application Insights。

## <a name="more-answers"></a>更多解答
* [Application Insights 論壇](https://social.msdn.microsoft.com/Forums/vstudio/en-US/home?forum=ApplicationInsights)

<!--Link references-->

[data]: app-insights-data-retention-privacy.md
[platforms]: app-insights-platforms.md
[start]: app-insights-overview.md
[windows]: app-insights-windows-get-started.md