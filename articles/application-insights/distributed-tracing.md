---
title: Azure Application Insights 中的分散式追蹤 | Microsoft Docs
description: 提供有關 Microsoft 對於透過我們的本機轉送工具和 OpenCensus 專案中的合作關係進行分散式追蹤的支援資訊
services: application-insights
keywords: ''
author: nikmd23
ms.author: nimolnar
ms.reviewer: mbullwin
ms.date: 09/17/2018
ms.service: application-insights
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 47ea4fadba50fdbbd6d83531bd4eb40fc581d2ad
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950194"
---
# <a name="what-is-distributed-tracing"></a>什麼是分散式追蹤？

新式雲端和微服務架構的問世造就了可獨立部署的簡單服務，協助您降低成本，同時提高可用性和輸送量。 這些變動雖然整體上讓個別服務更容易了解，但是讓整體系統更難以理解和偵錯。

在單體式架構中，我們已習慣透過呼叫堆疊進行偵錯。 呼叫堆疊是很出色的工具，可用來顯示執行的流程 (方法 A 呼叫方法 B，方法 B 呼叫方法 C)，以及有關每個呼叫的詳細資料和參數。 非常適合於在單一程序中執行的單體或服務，但我們如何在呼叫跨越程序界限 (不只限於本機堆疊上的參考) 時進行偵錯？ 

這就是分散式追蹤的由來。  

分散式追蹤相當於新式雲端和微服務架構的呼叫堆疊，但增添了非常簡單的效能分析工具。 在 Azure 監視器中，我們提供兩種取用分散式追蹤資料的體驗。 第一種是[交易診斷](https://docs.microsoft.com/azure/application-insights/app-insights-transaction-diagnostics)檢視，就像是新增了時間維度的呼叫堆疊。 交易診斷檢視可顯示單一交易/要求，並可協助您找出可靠性問題的根本原因及每個要求的效能瓶頸。

Azure 監視器也提供可彙總許多交易的[應用程式對應](https://docs.microsoft.com/azure/application-insights/app-insights-app-map)檢視，以顯示系統互動方式的拓撲檢視，以及平均效能和錯誤率。 

## <a name="how-to-enable-distributed-tracing"></a>如何啟用分散式追蹤

根據導入服務的語言，在應用程式中啟用跨服務的分散式追蹤，就如同將適當的 SDK 或程式庫新增至每個服務一樣簡單。

## <a name="enabling-via-application-insights-sdks"></a>透過 Application Insights SDK 啟用

適用於 .NET、.NET Core、Java、Node.js 和 JavaScript 的 Application Insights SDK 全都提供分散式追蹤的原生支援。 以下可取得安裝和設定每個 Application Insights SDK 的指示：

* [.NET](https://docs.microsoft.com/azure/application-insights/quick-monitor-portal)
* [.NET Core](https://docs.microsoft.com/azure/application-insights/app-insights-dotnetcore-quick-start)
* [Java](https://docs.microsoft.com/azure/application-insights/app-insights-java-get-started)
* [Node.js](https://docs.microsoft.com/azure/application-insights/app-insights-nodejs-quick-start)
* [JavaScript](https://docs.microsoft.com/azure/application-insights/app-insights-javascript)

安裝和設定適當的 Application Insights SDK 後，SDK 相依性自動收集器就會針對熱門的架構、程式庫和技術自動收集追蹤資訊。 完整的支援技術清單位於[相依性自動收集文件](https://docs.microsoft.com/azure/application-insights/auto-collect-dependencies)中。

 此外，在 [TeleletryClient](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics) 上呼叫 [TrackDependency](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics)，即可手動追蹤任何技術。

## <a name="enable-via-opencensus"></a>透過 OpenCensus 啟用

除了 Application Insights SDK，Application Insights 也支援透過 [OpenCensus](https://opencensus.io/) 的分散式追蹤。 OpenCensus 是開放原始碼、廠商中立的單一程式庫散發，可提供服務的計量收集和分散式追蹤。 它也可讓開放原始碼社群，使用熱門的技術 (例如 Redis、Memcached 或 MongoDB) 進行分散式追蹤。 [Microsoft 在 OpenCensus 上與其他數個監視和雲端夥伴共同作業](https://open.microsoft.com/2018/06/13/microsoft-joins-the-opencensus-project/)。

若要將分散式追蹤功能新增至採用 OpenCensus 的應用程式，請先[安裝和設定 Application Insights 本機轉送工具](./opencensus-local-forwarder.md)。 從該處，將 OpenCensus 設定為透過本機轉送工具路由傳送分散式追蹤資料。 支援 [Python](./opencensus-python.md) 與 [Go](./opencensus-go.md)。

OpenCensus 網站會維護 [Python](https://opencensus.io/api/python/trace/usage.html) 與 [Go](https://godoc.org/go.opencensus.io) 的 API 參考文件，以及可供使用 OpenCensus 的各種不同指南。 

## <a name="next-steps"></a>後續步驟

* [OpenCensus Python 使用指南](https://opencensus.io/api/python/trace/usage.html)
* [應用程式對應](./app-insights-app-map.md)
* [端對端效能監視](./app-insights-tutorial-performance.md)