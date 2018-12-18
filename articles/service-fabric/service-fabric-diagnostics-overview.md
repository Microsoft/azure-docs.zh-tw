---
title: Azure Service Fabric 監視和診斷概觀 | Microsoft Docs
description: 了解如何對 Azure Service Fabric 叢集、應用程式和服務進行監視和診斷。
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/25/2018
ms.author: srrengar
ms.openlocfilehash: f7fe07500f877cf34626e53361c9c68dd459a5e4
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34643170"
---
# <a name="monitoring-and-diagnostics-for-azure-service-fabric"></a>對 Azure Service Fabric 進行監視和診斷

本文提供監視和診斷 Azure Service Fabric 的概觀。 不論任何雲端環境，針對工作負載的開發、測試及部署進行監視和診斷都極為重要。 監視可讓您追蹤應用程式的使用方式、資源使用率，以及叢集的整體健康情況。 您可以使用這些資訊來診斷並修正任何問題，以及防止未來發生問題。 

## <a name="application-monitoring"></a>應用程式監視
應用程式監視會追蹤您應用程式的功能與元件使用情況。 您可以監視應用程式，以確定找出影響使用者的問題。 在下列情況，監視應用程式可能非常有用：
* 判斷應用程式負載和使用者流量 - 您需要調整服務以符合使用者需求，或解決應用程式中的潛在瓶頸嗎？
* 識別叢集中服務通訊與遠端的問題
* 找出您的使用者使用應用程式的行為 - 在您的應用程式中收集遙測可協助引導未來功能的開發，也更能診斷出應用程式錯誤
* 監視執行中容器內所發生的情況

Service Fabric 支援許多選項，以使用適當的追蹤和遙測來檢測您的應用程式程式碼。 建議您使用 Application Insights (AI)。 AI 與 Service Fabric 的整合包括 Visual Studio 和 Azure 入口網站的工具體驗，以及 Service Fabric 特有計量，可提供完整而立即可用的記錄體驗。 AI 雖然會自動為您建立和收集許多記錄，但我們建議您為應用程式新增進一步的自訂記錄，以打造更豐富的診斷體驗。 在[使用 Application Insights 進行事件分析](service-fabric-diagnostics-event-analysis-appinsights.md)中可深入了解如何開始使用 Application Insights 搭配 Service Fabric。

## <a name="platform-cluster-monitoring"></a>平台 (叢集) 監視
如果要確保平台和所有工作負載都如預期般執行，監視 Service Fabric 叢集就非常重要。 Service Fabric 的其中一個目標是讓應用程式在硬體失敗時能夠復原。 這個目標可以透過 平台的系統服務偵測基礎結構問題，並快速地將工作負載容錯移轉到叢集中的其他節點來達成。 但是在這個特殊情況下，如果是系統服務本身有問題，會發生什麼情況？ 或如果在嘗試移動工作負載時違反了設置服務的規則，會發生麼情況？ 監視叢集可讓您隨時掌握叢集中正在進行的活動，有助於診斷問題並有效地修正。 您要設法找出的一些重點包括：
* 以設置應用程式和平衡叢集的工作而言，Service Fabric 是否依照您預期的方式運作？ 
* 使用者對您的叢集採取的動作是否已認可並如預期般執行？ 這在調整叢集規模時尤其重要。
* Service Fabric 是否在叢集內正確處理您的資料和服務間通訊？

Service Fabric 提供一組完整的現成事件。 這些 [Service Fabric 事件](service-fabric-diagnostics-events.md)可以透過 EventStore API 或作業通道 (平台公開的事件通道) 來存取。 
* EventStore - EventStore (Windows 6.2 版和更新版本提供；Linux 在本文章最新更新時仍在進行中) 能透過一組 API 公開這些事件 (透過 REST 端點或用戶端程式庫存取)。 若要進一步了解 EventStore，請閱讀 [EventStore 概觀](service-fabric-diagnostics-eventstore.md)。
* Service Fabric 事件通道 - 在 Windows 上，透過一組用來挑選「作業和資料」與「傳訊」通道的相關 `logLevelKeywordFilters`，就能從單一 ETW 提供者取得 Service Fabric 事件 - 這是我們在需要時區分出待篩選傳出 Service Fabric 事件的方法。 在 Linux 上，Service Fabric 事件會經過 LTTng 並放入一個儲存體資料表，您可以視需要從這個資料表篩選事件。 這些通道包含經過策劃、結構化的事件，可用來進一步了解您的叢集狀態。 叢集建立時預設會啟用診斷，這會建立一個 Azure 儲存體表格，來自這些通道的事件會傳送到這個表格，供您將來查詢之用。 

我們建議您使用 EventStore 來進行快速分析，概略了解叢集的運作情況，以及各項功能是否如預期般正常進行。 若要收集叢集所產生的記錄和事件，我們通常建議使用 [Azure 診斷擴充功能](service-fabric-diagnostics-event-aggregation-wad.md)。 這項擴充功能會與 OMS Log Analytics 的 Service Fabric 專屬解決方案「Service Fabric 分析」充分整合，該解決方案提供一個監視 Service Fabric 叢集的自訂儀表板，可讓您查詢叢集的事件和設定警示。 您可以在[使用 OMS 進行事件分析](service-fabric-diagnostics-event-analysis-oms.md)閱讀更多資訊。 

 您可以在[平台層級事件和記錄產生](service-fabric-diagnostics-event-generation-infra.md)閱讀更多監視叢集的詳細資訊。

## <a name="performance-monitoring"></a>效能監視
監視根本的基礎結構是了解叢集狀態和資源使用率的關鍵部分。 測量系統效能取決於許多因素，每個因素通常都可透過關鍵效能指標 (KPI) 測量。 Service Fabric 相關的 KPI 可以對應至可從叢集中節點收集的計量，作為效能計數器。
這些 KPI 可協助：
* 了解資源使用率和負載 - 作為調整叢集規模，或最佳化服務流程之用。
* 預測基礎結構問題 - 許多問題在發生之前，效能會有突然的變化 (降低)，因此您可以使用如網路 I/O 和 CPU 使用率等 KPI 來預測和診斷基礎結構的問題。

您可以在[效能計量](service-fabric-diagnostics-event-generation-perf.md)找到在基礎結構層級應該收集的效能計數器清單。 

Service Fabric 會提供一組效能計數器，以供 Reliable Services 和動作項目程式設計模型使用。 如果您使用上述其中一種模型，這些效能計數器可以提供一些 KPI，協助確保您的動作項目正確向上和向下微調，或者您的可靠服務要求處理的速度夠快。 如需詳細資訊，請參閱[可靠服務遠端的監視](service-fabric-reliable-serviceremoting-diagnostics.md#performance-counters)和 [Reliable Actors 的效能監視](service-fabric-reliable-actors-diagnostics.md#performance-counters)。 除此之外，如果您的應用程式經過設定，Application Insights 也會收集一組效能計量。

請使用 [OMS 代理程式](service-fabric-diagnostics-oms-agent.md)收集適當的效能計數器，並在 OMS Log Analytics 中檢視這些 KPI。

![診斷概觀圖表](media/service-fabric-diagnostics-overview/diagnostics-overview.png)

## <a name="health-monitoring"></a>健康狀況監視
Service Fabric 平台包括健康情況模型，針對叢集中的實體狀態提供可延伸的健康情況報告。 每個節點、應用程式、服務、分割區、複本或執行個體，都有可持續更新的健康情況狀態。 健康情況狀態可以是「良好」、「警告」或「錯誤」。 健康情況狀態會根據叢集中的問題，透過針對每個實體所發出的健康情況報告變更。 實體的健康情況狀態可以隨時在 Service Fabric Explorer (SFX) 中加以檢查 (如下所示)，或透過平台的健康情況 API 查詢。 您也可以新增您自己的健康情況報告或使用健康情況 API，來自訂健康情況報告和修改實體的健康情況狀態。 在 [Service Fabric 健康情況監視簡介](service-fabric-health-introduction.md)可以找到健康情況模型的更多詳細資料。

![SFX 健康情況儀表板](media/service-fabric-diagnostics-overview/sfx-healthstatus.png)

在 SFX 中除了能看到最新的健康情況報告，每個報告也可當成一個事件。 健康情況事件可透過操作通道收集 (請參閱[使用 Azure 診斷進行事件彙總](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations))，並儲存在 Log Analytics 中供未來警示和查詢之用。 這有助於偵測出可能會影響您應用程式可用性的問題，所以我們建議您針對適當的失敗案例設定警示 (透過 Log Analytics 設定自訂警示)。

## <a name="other-logging-solutions"></a>其他記錄解決方案

雖然我們建議兩個解決方案，但是 [OMS](service-fabric-diagnostics-event-analysis-oms.md) 和 [Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md) 已內建置在 Service Fabric 的整合中，所以許多事件會透過 etw 提供者寫出來並可利用其他記錄解決方案延伸。 您也應該研究 [Elastic Stack](https://www.elastic.co/products) (尤其是當您考慮在離線環境中執行叢集時)、[Splunk](https://www.splunk.com/)、[Dynatrace](https://www.dynatrace.com/)，或您偏好的任何其他平台。 

不論選擇何種平台，重點在於您是否喜歡其中的使用者介面和查詢選項、視覺化資料與建立一目了然之儀表板的功能，以及是否提供可強化監視的其他工具，例如自動化警示。

## <a name="next-steps"></a>後續步驟

* 若要開始檢測您的應用程式，請參閱[應用程式層級事件和記錄產生](service-fabric-diagnostics-event-generation-app.md)。
* 在[平台層級事件和記錄產生](service-fabric-diagnostics-event-generation-infra.md)深入了解監視平台，以及監視 Service Fabric 提供給您的事件。
* 透過[監視和診斷 Service Fabric 上的 ASP.NET Core 應用程式](service-fabric-tutorial-monitoring-aspnet.md)，針對您的應用程式逐步執行設定 AI 的步驟。
* 了解如何設定 OMS Log Analytics 以監視容器：[監視和診斷 Azure Service Fabric 中的 Windows 容器](service-fabric-tutorial-monitoring-wincontainers.md)。
* 了解 Azure 資源的一般監視建議：[最佳做法 - 監視和診斷](https://docs.microsoft.com/azure/architecture/best-practices/monitoring)。 
