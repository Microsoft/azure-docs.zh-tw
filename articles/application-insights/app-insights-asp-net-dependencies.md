---
title: Azure Application Insights 中的相依性追蹤 | Microsoft Docs
description: 使用 Application Insights 分析內部部署或 Microsoft Azure Web 應用程式的使用情況、可用性和效能。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: d15c4ca8-4c1a-47ab-a03d-c322b4bb2a9e
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/08/2018
ms.author: mbullwin
ms.openlocfilehash: f1a1c0bd759a88b2e84584f1d52458ac6f56d97f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999684"
---
# <a name="set-up-application-insights-dependency-tracking"></a>設定 Application Insights：追蹤相依性
「相依性」  是由應用程式呼叫的外部元件。 這通常是使用 HTTP 呼叫的服務，或資料庫，或檔案系統。 [Application Insights](app-insights-overview.md) 會測量您應用程式等待相依性所花費的時間，以及相依性呼叫失敗的頻率。 您可以調查特定的呼叫，然後將它們與要求和例外狀況建立關聯。

![範例圖表](./media/app-insights-asp-net-dependencies/10-intro.png)

預設的相依性監視目前會回報這些相依性類型的呼叫：

* 伺服器
  * SQL DATABASE
  * 使用 HTTP 式繫結的 ASP.NET Web 和 WCF 服務
  * 本機或遠端 HTTP 呼叫
  * Azure Cosmos DB、資料表、Blob 儲存體和佇列
* 網頁
  * AJAX 呼叫

在選取的方法周圍使用[位元組程式碼檢測](https://msdn.microsoft.com/library/z9z62c29.aspx)來監視工作。 效能額外負荷非常小。

您也可以使用 [TrackDependency API](app-insights-api-custom-events-metrics.md#trackdependency)，同時在用戶端和伺服器程式碼中撰寫自己的 SDK 呼叫，來監視其他相依性。

## <a name="set-up-dependency-monitoring"></a>設定相依性監視
[Application Insights SDK](app-insights-asp-net.md) 會自動收集部分相依性資訊。 若要取得完整資料，請為主機伺服器安裝適當的代理程式。

| 平台 | Install |
| --- | --- |
| IIS 伺服器 |[在您的伺服器上安裝狀態監視器](app-insights-monitor-performance-live-website-now.md)或[將您的應用程式升級到 .NET Framework 4.6 或更新版本](http://go.microsoft.com/fwlink/?LinkId=528259)，然後在應用程式中安裝 [Application Insights SDK](app-insights-asp-net.md)。 |
| Azure Web 應用程式 |在您的 Web 應用程式控制台中，[開啟 Application Insights 刀鋒視窗](app-insights-azure-web-apps.md)，然後在出現提示時選擇 [安裝]。 |
| Azure 雲端服務 |[使用啟動工作](app-insights-cloudservices.md)或[安裝 .NET Framework 4.6+](../cloud-services/cloud-services-dotnet-install-dotnet.md) |

## <a name="where-to-find-dependency-data"></a>哪裡可以找到相依性資料
* [應用程式對應](#application-map)會以視覺化方式顯示您應用程式與相鄰元件之間的相依性。
* [效能、瀏覽器及失敗刀鋒視窗](#performance-and-blades)會顯示伺服器相依性資料。
* [瀏覽器刀鋒視窗](#ajax-calls)會顯示來自您使用者瀏覽器的 AJAX 呼叫。
* [從速度緩慢或失敗的要求逐一點選](#diagnose-slow-requests)來檢查它們的相依性呼叫。
* [分析](#analytics)可用來查詢相依性資料。

## <a name="application-map"></a>應用程式對應
應用程式對應可做為探索應用程式元件之間相依性的視覺輔助工具。 它會從來自您應用程式的遙測自動產生。 此範例顯示來自瀏覽器指令碼的 AJAX 呼叫，以及從伺服器應用程式到兩個外部服務的 REST 呼叫。

![應用程式對應](./media/app-insights-asp-net-dependencies/08.png)

* **從方塊中瀏覽**至相關的相依性及其他圖表。
* **將對應釘選**至[儀表板](app-insights-dashboards.md) (對應將可在其中完整運作)。

[深入了解](app-insights-app-map.md)。

## <a name="performance-and-failure-blades"></a>效能和失敗刀鋒視窗
[效能] 刀鋒視窗會顯示伺服器應用程式所發出之相依性呼叫的持續時間。 其中會顯示摘要圖表和依呼叫劃分的表格。

![效能刀鋒視窗相依性圖表](./media/app-insights-asp-net-dependencies/dependencies-in-performance-blade.png)

在摘要圖表或表格項目上逐一點選來搜尋這些呼叫的原始發生項目。

![相依性呼叫執行個體](./media/app-insights-asp-net-dependencies/dependency-call-instance.png)

[失敗計數] 會顯示在 [失敗] 刀鋒視窗上。 失敗是指任何範圍不在 200-399 內或是不明的傳回碼。

> [!NOTE]
> **100% 失敗？** - 這可能是指您取得的只是部分相依性資料。 您必須[設定適合您平台的相依性監視](#set-up-dependency-monitoring)。
>
>

## <a name="ajax-calls"></a>AJAX 呼叫
[瀏覽器] 刀鋒視窗會顯示來自[您網頁中 JavaScript](app-insights-javascript.md) 之 AJAX 呼叫的持續時間和失敗率。 它們會顯示為「相依性」。

## <a name="diagnosis"></a> 診斷速度緩慢的要求
每個要求事件都與相依性呼叫、例外狀況及您應用程式處理要求時所追蹤的其他事件相關聯。 因此，如果某些要求執行效能很差，您可以了解是否是因為某個相依性的回應太慢。

讓我們逐步解說一個該情況的範例。

### <a name="tracing-from-requests-to-dependencies"></a>進行從要求到相依性的追蹤
開啟 [效能] 刀鋒視窗，然後查看要求方格：

![含有平均和計數的要求清單](./media/app-insights-asp-net-dependencies/02-reqs.png)

最上方的要求花了很長的時間。 來看看我們是否可以查明時間花費在何處。

按一下該列，以查看個別的要求事件：

![要求發生次數的清單](./media/app-insights-asp-net-dependencies/03-instances.png)

按一下任何長時間執行的執行個體來進一步檢查，然後向下捲動至與此要求相關的遠端相依性呼叫：

![尋找遠端相依性的呼叫，識別不尋常的持續時間](./media/app-insights-asp-net-dependencies/04-dependencies.png)

看起來此要求的大部分時間似乎都花費在呼叫本機服務上。

選取該列，以取得詳細資訊：

![點選該遠端相依性來找出問題原因](./media/app-insights-asp-net-dependencies/05-detail.png)

看起來這就是問題所在。 我們已經指出問題，因此現在只需要了解為何該呼叫花費那麼長的時間。

### <a name="request-timeline"></a>要求時間軸
在一個不同的案例中，並沒有任何特別長的相依性呼叫。 但藉由切換到時間軸檢視，我們即可看到在內部處理中發生延遲的地方：

![尋找遠端相依性的呼叫，識別不尋常的持續時間](./media/app-insights-asp-net-dependencies/04-1.png)

在第一次相依性呼叫之後似乎有一個很大的間隔，因此我們應該查看程式碼來找出原因。

### <a name="profile-your-live-site"></a>剖析您的即時網站

不清楚時間花在哪裡嗎？ [Application Insights 分析工具](app-insights-profiler.md)會追蹤對您即時站台發出的 HTTP 呼叫，並顯示您程式碼中哪些函式花費的時間最長。

## <a name="failed-requests"></a>失敗的要求
失敗的要求可能也會與失敗的相依性呼叫關聯。 同樣地，我們可以逐一點選來追蹤問題。

![按一下失敗要求的圖表](./media/app-insights-asp-net-dependencies/06-fail.png)

逐一點選至失敗要求的某個發生項目，然後查看其相關事件。

![按一下要求類型，按一下執行個體以取得同一個執行個體的不同檢視，按一下執行個體以取得例外狀況的詳細資料。](./media/app-insights-asp-net-dependencies/07-faildetail.png)

## <a name="analytics"></a>分析
您可以在 [Log Analytics 查詢語言](https://aka.ms/LogAnalyticsLanguage)中追蹤相依性。 以下是一些範例。

* 尋找任何失敗的相依性呼叫：

```

    dependencies | where success != "True" | take 10
```

* 尋找 AJAX 呼叫︰

```

    dependencies | where client_Type == "Browser" | take 10
```

* 尋找與要求關聯的相依性呼叫：

```

    dependencies
    | where timestamp > ago(1d) and  client_Type != "Browser"
    | join (requests | where timestamp > ago(1d))
      on operation_Id  
```


* 尋找與頁面檢視關聯的 AJAX 呼叫：

```

    dependencies
    | where timestamp > ago(1d) and  client_Type == "Browser"
    | join (browserTimings | where timestamp > ago(1d))
      on operation_Id
```



## <a name="custom-dependency-tracking"></a>自訂相依性追蹤
標準的相依性追蹤模組會自動探索外部相依性，例如資料庫和 REST API。 但是您可能想以相同的方式對待一些其他元件。

您可以使用標準模組所使用的相同 [TrackDependency API](app-insights-api-custom-events-metrics.md#trackdependency) 來撰寫傳送相依性資訊的程式碼。

例如，如果您建置程式碼的組件不是您自己撰寫的，您可以計算對組件的所有呼叫，以找出它佔回應時間的比例。 若要在 Application Insights 中的相依性圖表中顯示此資料，請使用 `TrackDependency`傳送。

```csharp

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

如果您想要關閉標準的相依性追蹤模組，請移除 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md)中 DependencyTrackingTelemetryModule 的參考。

## <a name="troubleshooting"></a>疑難排解
*相依性成功旗標一律顯示 true 或 false。*

*SQL 查詢未完整顯示。*

請參閱下表，確保您已選擇正確的設定來啟用應用程式的相依性監視。

| 平台 | Install |
| --- | --- |
| IIS 伺服器 |[在您的伺服器上安裝狀態監視器](app-insights-monitor-performance-live-website-now.md)， 或[將您的應用程式升級到 .NET Framework 4.6 或更新版本](http://go.microsoft.com/fwlink/?LinkId=528259)，然後在應用程式中安裝 [Application Insights SDK](app-insights-asp-net.md)。 |
| Azure Web 應用程式 |在您的 Web 應用程式控制台中，[開啟 Application Insights 刀鋒視窗](app-insights-azure-web-apps.md)，然後在出現提示時選擇 [安裝]。 |
| Azure 雲端服務 |[使用啟動工作](app-insights-cloudservices.md)或[安裝 .NET Framework 4.6+](../cloud-services/cloud-services-dotnet-install-dotnet.md) |

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟
* [例外狀況](app-insights-asp-net-exceptions.md)
* [使用者和頁面資料](app-insights-javascript.md)
* [Availability](app-insights-monitor-web-app-availability.md)
