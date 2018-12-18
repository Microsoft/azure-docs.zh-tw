---
title: 我如何在 Azure Application Insights 中... | Microsoft Docs
description: Application Insights 中的常見問題集。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 48b2b644-92e4-44c3-bc14-068f1bbedd22
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 04/04/2017
ms.author: mbullwin
ms.openlocfilehash: 8cee346a45cd20e7dd677fd7f2efed5500175598
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096389"
---
# <a name="how-do-i--in-application-insights"></a>我如何在 Application Insights 中...？
## <a name="get-an-email-when-"></a>... 時收到電子郵件
### <a name="email-if-my-site-goes-down"></a>我的網站當機時寄送電子郵件
設定 [可用性 Web 測試](app-insights-monitor-web-app-availability.md)。

### <a name="email-if-my-site-is-overloaded"></a>我的網站多載時寄送電子郵件
針對 **伺服器回應時間** 設定 [警示](app-insights-alerts.md)。 介於 1 到 2 秒之間的閾值應該會運作。

![](./media/app-insights-how-do-i/030-server.png)

您的 app 也可能會藉由傳回失敗碼顯示資源耗盡的徵兆。 針對 [失敗的要求] 設定警示。

如果您想要針對 [伺服器例外狀況] 設定警示，可能必須進行 [一些其他設定](app-insights-asp-net-exceptions.md) 才能看到資料。

### <a name="email-on-exceptions"></a>傳送電子郵件的例外狀況
1. [設定例外狀況監視](app-insights-asp-net-exceptions.md)
2. [設定警示](app-insights-alerts.md) 

### <a name="email-on-an-event-in-my-app"></a>我的 app 發生事件時寄送電子郵件
我們假設您想要在特定事件發生時收到電子郵件。 Application Insights 並未直接提供此功能，但它可 [在計量超過某個閾值時傳送警示](app-insights-alerts.md)。

您可以針對 [自訂計量](app-insights-api-custom-events-metrics.md#trackmetric)設定警示，而不是自訂事件。 撰寫一些程式碼以在事件發生時增加計量：

    telemetry.TrackMetric("Alarm", 10);

或：

    var measurements = new Dictionary<string,double>();
    measurements ["Alarm"] = 10;
    telemetry.TrackEvent("status", null, measurements);

因為警示有兩個狀態，所以當您考慮讓警示結束時必須傳送較低的值：

    telemetry.TrackMetric("Alarm", 0.5);

在 [計量總管](app-insights-metrics-explorer.md) 中建立圖表來查看您的警示：

![](./media/app-insights-how-do-i/010-alarm.png)

現在設定警示在計量一段短時間高於中間值時引發：

![](./media/app-insights-how-do-i/020-threshold.png)

將平均期間設定為最小值。

您就會在計量高於和低於閾值時收到電子郵件。

考慮事項：

* 警示有兩種狀態 (「警示」和「良好」)。 只有在收到計量時才會評估狀態。
* 只有在狀態變更時才會傳送電子郵件。 這是為什麼您必須同時傳送高值和低值的計量。
* 若要評估警示，會採用收到的值除以先前的期間來獲得平均值。 因為每次收到計量時都會進行計算，所以傳送電子郵件的頻率會比您設定的期間還要高。
* 由於「警示」和「良好」都會傳送電子郵件，所以您可能要以兩個狀態的條件來重新思考單次發生的事件。 例如，不要採用「作業完成」事件，而採用「作業進行中」的條件，您就可以在工作開始和結束時收到電子郵件。

### <a name="set-up-alerts-automatically"></a>自動設定警示
[使用 PowerShell 建立新的警示](app-insights-alerts.md#automation)

## <a name="use-powershell-to-manage-application-insights"></a>使用 PowerShell 管理 Application Insights
* [建立新的資源](app-insights-powershell-script-create-resource.md)
* [建立新的警示](app-insights-alerts.md#automation)

## <a name="separate-telemetry-from-different-versions"></a>區分不同版本的遙測

* 應用程式中的多個角色︰使用單一 Application Insights 資源，並依據 cloud_Rolename 進行篩選。 [深入了解](app-insights-monitor-multi-role-apps.md)
* 區分開發、測試和發行版本︰使用不同的 Application Insights 資源。 從 web.config 挑選檢測金鑰。[深入了解](app-insights-separate-resources.md)
* 報告組建版本︰使用遙測初始設定式新增屬性。 [深入了解](app-insights-separate-resources.md)

## <a name="monitor-backend-servers-and-desktop-apps"></a>監視後端伺服器與桌面應用程式
[使用 Windows Server SDK 模組](app-insights-windows-desktop.md)。

## <a name="visualize-data"></a>顯現資料
#### <a name="dashboard-with-metrics-from-multiple-apps"></a>具有來自多個 App 之計量的儀表板
* 在 [計量總管](app-insights-metrics-explorer.md)中，自訂圖表並將它儲存為我的最愛。 將它釘選到 Azure 儀表板。

#### <a name="dashboard-with-data-from-other-sources-and-application-insights"></a>資料來自其他來源和 Application Insights 的儀表板
* [將遙測匯出至 Power Bi](app-insights-export-power-bi.md)。

或

* 使用 SharePoint 做為您的儀表板，在 SharePoint web 組件中顯示資料。 [使用連續的匯出和資料流分析來匯出至 SQL](app-insights-code-sample-export-sql-stream-analytics.md)。  使用 PowerView 來檢查資料庫，並建立適用於 PowerView 的 SharePoint web 組件。

<a name="search-specific-users"></a>

### <a name="filter-out-anonymous-or-authenticated-users"></a>篩選出匿名或已驗證的使用者
如果您的使用者登入，您可以設定[驗證使用者識別碼](app-insights-api-custom-events-metrics.md#authenticated-users)。(它不會自動重新整理)。

接著，您可以：

* 搜尋特定的使用者識別碼

![](./media/app-insights-how-do-i/110-search.png)

* 篩選匿名或已驗證使用者的計量

![](./media/app-insights-how-do-i/115-metrics.png)

## <a name="modify-property-names-or-values"></a>修改屬性名稱或值
建立 [篩選器](app-insights-api-filtering-sampling.md#filtering)。 這可讓您先修改或篩選遙測，然後再將它從您的應用程式傳送至 Application Insights。

## <a name="list-specific-users-and-their-usage"></a>列出特定使用者和其使用方式
如果您只想要[搜尋特定使用者](#search-specific-users)，就可以設定[驗證使用者識別碼](app-insights-api-custom-events-metrics.md#authenticated-users)。

如果您想要使用者清單以及像是他們查看過哪些頁面或登入頻率等資料，則有兩個選項：

* [設定驗證使用者識別碼](app-insights-api-custom-events-metrics.md#authenticated-users)、[匯出到資料庫](app-insights-code-sample-export-sql-stream-analytics.md)，然後使用適當的工具來分析使用者資料。
* 如果您只有少數的使用者，則可傳送自訂事件或計量、使用感興趣的資料做為計量值或事件名稱，然後設定使用者識別碼做為屬性。 若要分析頁面檢視，可取代標準的 JavaScript trackPageView 呼叫。 若要分析伺服器端遙測，可使用遙測初始設定式，將使用者識別碼新增至所有伺服器遙測。 然後您可以篩選與分割關於使用者識別碼的計量資訊和搜尋。

## <a name="reduce-traffic-from-my-app-to-application-insights"></a>降低從我的 App 到 Application Insights 的流量
* 在 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md)中，停用任何您不需要的模組，例如效能計數器收集器。
* 使用 SDK 中的 [取樣和篩選](app-insights-api-filtering-sampling.md) 。
* 您在網頁中，限制針對每個頁面檢視回報的 Ajax 呼叫次數。 在 `instrumentationKey:...` 之後的指令碼片段中，插入：`,maxAjaxCallsPerView:3` (或適當的數字)。
* 如果您使用的是 [TrackMetric](app-insights-api-custom-events-metrics.md#trackmetric)，請在傳送結果之前，先計算計量值批次的彙總。 有一個 TrackMetric() 的多載是針對該動作所提供。

深入了解 [價格和配額](app-insights-pricing.md)。

## <a name="disable-telemetry"></a>停用遙測
若要從伺服器 **動態停止和開始** 收集及傳輸遙測資料：

```

    using  Microsoft.ApplicationInsights.Extensibility;

    TelemetryConfiguration.Active.DisableTelemetry = true;
```



若要 **停用選取的標準收集器** (例如效能計數器、HTTP 要求或相依性)，請刪除或註解化 [ApplicationInsights.config](app-insights-api-custom-events-metrics.md)中的相關行。例如，如果您想要傳送自己的 TrackRequest 資料，可以這麼做。

## <a name="view-system-performance-counters"></a>檢視系統效能計數器
您可以在計量總管中顯示的計量資訊是一組系統效能計數器。 有一個預先定義且標題為 **伺服器** 的刀鋒視窗會顯示它們其中幾個。

![開啟 Application Insights 資源並按一下伺服器](./media/app-insights-how-do-i/121-servers.png)

### <a name="if-you-see-no-performance-counter-data"></a>如果您看不到效能計數器資料
* **IIS 伺服器** 。 [安裝狀態監視器](app-insights-monitor-performance-live-website-now.md)。
* **Azure 網站** - 我們尚未支援效能計數器。 您可以取得數個計量來做為 Azure 網站控制台的標準部分。
* **Unix 伺服器** - [安裝 collectd](app-insights-java-collectd.md)

### <a name="to-display-more-performance-counters"></a>顯示更多效能計數器
* 首先，[新增圖表](app-insights-metrics-explorer.md) ，並查看計數器是否位於我們提供的基本組合中。
* 如果沒有，請[將計數器加入效能計數器模組所收集的組合中](app-insights-performance-counters.md)。
