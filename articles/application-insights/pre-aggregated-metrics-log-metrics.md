---
title: Azure Application Insights 中記錄型和預先彙總的計量 | Microsoft Docs
description: 為何在 Azure Application Insights 中使用記錄型和預先彙總的計量
services: application-insights
keywords: ''
author: vgorbenko
ms.author: vitaly.gorbenko
ms.reviewer: mbullwin
ms.date: 09/18/2018
ms.service: application-insights
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 136279a94d64b846247c436eb7a5fb487e05b41a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946896"
---
# <a name="log-based-and-pre-aggregated-metrics-in-application-insights"></a>Application Insights 中記錄型和預先彙總的計量

本文說明以記錄為基礎的「傳統」Application Insights 計量，與目前處於公開預覽狀態的預先彙總計量之間的差異。 這兩種類型的計量均可供 Application Insights 使用者使用，而且各自在監視應用程式健康情況、診斷和分析方面帶來獨特的價值。 檢測應用程式的開發人員可以根據應用程式的大小、預期的遙測數量，以及計量精確度和警示的商務需求，決定哪種類型的計量最適合於特定案例。

## <a name="log-based-metrics"></a>記錄型計量

直到最近，在 Application Insights 中監視遙測資料模型的應用程式才完全以少數預先定義的事件類型為基礎，例如要求、例外狀況、相依性呼叫、頁面檢視等等。開發人員可以使用 SDK 手動發出這些事件 (藉由撰寫可明確叫用 SDK 的程式碼)，也可以依賴從自動檢測自動收集事件。 在任一情況下，Application Insights 後端會將所有收集的事件儲存為記錄，而 Azure 入口網站中的 Application Insights 刀鋒視窗會作為分析與診斷工具，以視覺化方式呈現記錄中以事件為基礎的資料。

使用記錄來保留一組完整的事件，可以帶來絕佳的分析與診斷價值。 例如，您可以取得特定 URL 的確切要求計數，以及進行這些呼叫的不同使用者數目。 您也可以取得詳細的診斷追蹤，包括任何使用者工作階段的例外狀況和相依性呼叫。 擁有此類型的資訊可大幅改善應用程式健康狀態和使用量的可見性，進而縮短診斷應用程式問題所需的時間。 

然而，對於可產生大量遙測的應用程式而言，收集一組完整的事件可能不切實際 (或甚至不可能的)。 在事件數量太高的情況下，Application Insights 會實作數種遙測數量縮減技術，例如[取樣](https://docs.microsoft.com/azure/application-insights/app-insights-sampling)和[篩選](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling)，以減少所收集和預存的事件數目。 不幸的是，降低的預存事件數目也會降低計量的精確度，其在幕後必須執行記錄中所儲事件的查詢階段彙總。

> [!NOTE]
> 在 Application Insights 中，以事件的查詢階段彙總為基礎的計量和以記錄中儲存的量測為基礎的計量都稱為記錄型計量。 這些計量通常有許多來自事件屬性的維度，使其更適合用於分析，但是取樣和篩選會對這些計量的精確度造成負面影響。

## <a name="pre-aggregated-metrics"></a>預先彙總的計量

除了記錄型計量，Application Insights 團隊在 2018 年秋季推出計量公開預覽版，而這些計量會儲存在針對時間序列最佳化的特製化存放庫中。 新的計量不再保存為具有許多屬性的個別事件。 相反地，它們會儲存為預先彙總的時間序列，而且只有重要維度。 這可讓新的計量在查詢階段表現優異：擷取資料更快速地發生，而且需要較少的計算能力。 因此造就新的案例，例如[近乎即時的計量維度警示](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-near-real-time-metric-alerts)、更有回應的[儀表板](https://docs.microsoft.com/azure/application-insights/app-insights-dashboards)等。

> [!IMPORTANT]
> 記錄型和預先彙總的計量兩者共存於 Application Insights 中。 為了在 Application Insights UX 中區分這兩者，預先彙總的計量現在稱為「標準計量 (預覽) 」，而事件中的傳統計量已重新命名為「記錄型計量」。

在遙測數量縮減技術開始運作之前，較新的 SDK (適用於 .NET 的[Application Insights 2.7](https://www.nuget.org/packages/Microsoft.ApplicationInsights/2.7.2) SDK 或更新版本) 會預先彙總收集期間的計量。 這表示使用最新的 Application Insights SDK 時，新計量的精確度不受取樣和篩選影響。

對於未實作預先彙總的 SDK (也就是舊版的 Application Insights SDK 或適用於瀏覽器檢測)，Application Insights 後端仍會藉由彙總 Application Insights 事件收集端點所收到的事件，填入新的計量。 這表示您雖然並未受惠於透過網路傳輸的資料量減少，但您仍可使用預先彙總的計量，並利用在收集期間並未預先彙總計量的 SDK，體驗更佳的效能和幾近即時的維度警示支援。

值得一提的是收集端點會在擷取取樣前預先彙總事件，這表示不論您使用哪個版本的 SDK 搭配您的應用程式，[擷取取樣](https://docs.microsoft.com/azure/application-insights/app-insights-sampling)決不會影響預先彙總的計量精確度。  

## <a name="using-pre-aggregation-with-application-insights-custom-metrics"></a>使用預先彙總搭配 Application Insights 自訂計量

您可以使用預先彙總搭配自訂計量。 有兩個主要優勢：能夠設定自訂計量的維度和發出警示，以及減少從 SDK 傳送至 Application Insights 收集端點的資料量。

[從 Application Insights SDK 傳送自訂計量的方法](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics)有好幾種。 如果您的 SDK 版本提供 [GetMetric 和 TrackValue](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#getmetric) 方法，這是傳送自訂計量慣用的方式，因為在此情況下，預先彙總會發生於 SDK 內部，不僅可減少 Azure 中儲存的資料量，也可減少從 SDK 傳輸至 Application Insights 的資料量。 否則，使用 [trackMetric](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackmetric) 方法，該方法會在資料擷取期間預先彙總計量事件。

## <a name="custom-metrics-dimensions-and-pre-aggregation"></a>自訂計量維度和預先彙總

您使用 [trackMetric](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackmetric) 或 [GetMetric 和 TrackValue](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#getmetric) API 呼叫傳送的所有計量，都會自動儲存在記錄和計量存放區中。 不過，雖然記錄型自訂計量版本一律會保留所有維度，但是預先彙總的計量版本儲存時預設不含任何維度。 您可以藉由勾選 [啟用自訂計量維度警示]，在[使用量和估計成本](https://docs.microsoft.com/azure/application-insights/app-insights-pricing)索引標籤上開啟自訂計量的維度收集： 

![使用量和估計成本](.\media\pre-aggregated-metrics-log-metrics\001-cost.png)

## <a name="why-is-collection-of-custom-metrics-dimensions-turned-off-by-default"></a>自訂計量維度的收集為何預設會關閉？

自訂計量維度的集合預設會關閉，因為未來儲存具有維度的自訂計量將與 Application Insights 分開計費，而儲存無維度的自訂計量仍為免費 (達到配額前)。 您可以在我們的官方[定價頁面](https://azure.microsoft.com/pricing/details/monitor/)上了解即將推出的定價模型變更。

## <a name="creating-charts-and-exploring-log-based-and-standard-pre-aggregated-metrics"></a>建立圖表並探索記錄型和標準預先彙總的計量

使用 Azure 監視器計量瀏覽器，根據預先彙總和記錄型計量繪製圖表，以及編寫具有圖表的儀表板。 選取所需的 Application Insights 資源之後，使用命名空間選擇器在標準 (預覽) 與記錄型計量之間切換，或選取自訂計量命名空間：

![計量命名空間](.\media\pre-aggregated-metrics-log-metrics\002-metric-namespace.png)

## <a name="next-steps"></a>後續步驟

* [近乎即時警示](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-near-real-time-metric-alerts)
* [GetMetric 和 TrackValue](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#getmetric)
