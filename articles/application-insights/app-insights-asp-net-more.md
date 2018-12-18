---
title: 充分利用 Azure Application Insights | Microsoft Docs
description: 開始使用 Application Insights 之後，以下是您可以瀏覽的功能摘要。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 7ec10a2d-c669-448d-8d45-b486ee32c8db
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 02/03/2017
ms.author: mbullwin
ms.openlocfilehash: 2f03083367de4e818bdc953ab76c28ff687f0a48
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35294331"
---
# <a name="more-telemetry-from-application-insights"></a>更多來自 Application Insights 的遙測
在您已經 [將 Application Insights 加入到您的 ASP.NET 程式碼](app-insights-asp-net.md)之後，您可以採取幾個動作來取得更多遙測。 

| 動作 | 得到什麼結果|
|---|---|
|(IIS 伺服器) 在每一部伺服器電腦上[安裝狀態監視器](http://go.microsoft.com/fwlink/?LinkId=506648)。<br/>(Azure Web 應用程式) 在 Web 應用程式的 Azure 控制台中，開啟 Application Insights 刀鋒視窗。| [**效能計數器**](app-insights-performance-counters.md)<br/>[**例外狀況**](app-insights-asp-net-exceptions.md) - 包括堆疊追蹤<br/>[**相依項目**](app-insights-asp-net-dependencies.md)|
|[將 JavaScript 程式碼片段新增至您的網頁](app-insights-javascript.md)|[頁面效能](app-insights-web-track-usage.md)、瀏覽器例外狀況、AJAX 效能。 自訂用戶端遙測。|
|[建立可用性 Web 測試](app-insights-monitor-web-app-availability.md)|如果網站無法使用，將收到警示|
|[確定 buildinfo.config](https://msdn.microsoft.com/library/dn449058.aspx) 由 MSBuild 產生|[建置計量圖表中的註解](https://blogs.msdn.microsoft.com/visualstudioalm/2013/11/14/implementing-deployment-markers-in-application-insights/) \(英文\)
|[寫入自訂事件和度量](app-insights-api-custom-events-metrics.md)|計算商務事件和度量、追蹤詳細使用方式等等。|
|[剖析您的即時網站](https://aka.ms/AIProfilerPreview)|從作用中的 Web 應用程式取得詳細的函式計時|






