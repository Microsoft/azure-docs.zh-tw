---
title: Azure Service Fabric 診斷常見案例 | Microsoft Docs
description: 了解如何針對 Azure Service Fabric 的常見案例進行疑難排解
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/16/2018
ms.author: srrengar
ms.openlocfilehash: bd7a7e0288ced0219a0600034b273d1acba6b09b
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34659183"
---
# <a name="diagnose-common-scenarios-with-service-fabric"></a>針對 Service Fabric 的常見案例進行診斷

本文說明使用者在 Service Fabric 的監視和診斷領域所經常遇到的案例。 所提供的案例涵蓋了服務網狀架構的所有 (共 3 個) 層級：應用程式、叢集和基礎結構。 每個解決方案會使用 Application Insights 和 Log Analytics (Azure 的監視工具) 來完成每個案例。 每個解決方案中的步驟都會為使用者介紹該如何在 Service Fabric 的內容中使用 Application Insights 和 Log Analytics。

## <a name="prerequisites-and-recommendations"></a>先決條件和建議

本文中的解決方案會使用下列工具。 建議您安裝並設定好這些工具：

* [Application Insights 搭配 Service Fabric](service-fabric-tutorial-monitoring-aspnet.md)
* [在叢集上啟用 Azure 診斷](service-fabric-diagnostics-event-aggregation-wad.md)
* [設定 OMS Log Analytics 工作區](service-fabric-diagnostics-oms-setup.md)
* [用來追蹤效能計數器的 OMS 代理程式](service-fabric-diagnostics-oms-agent.md)

## <a name="how-can-i-see-unhandled-exceptions-in-my-application"></a>如何查看應用程式中的未處理例外狀況？

1. 瀏覽至用來設定應用程式的 Application Insights 資源。
2. 按一下左上方的 [搜尋]。 然後在下一個面板上按一下篩選條件。

    ![AI 概觀](media/service-fabric-diagnostics-common-scenarios/ai-search-filter.png)

3. 您會看到許多類型的事件 (追蹤、要求、自訂事件)。 選擇 [例外狀況] 作為您的篩選條件。

    ![AI 篩選條件清單](media/service-fabric-diagnostics-common-scenarios/ai-filter-list.png)

    按一下清單中的例外狀況，即可查看更多詳細資料，包括服務內容 (如果您使用 Service Fabric Application Insights SDK 的話)。

    ![AI 例外狀況](media/service-fabric-diagnostics-common-scenarios/ai-exception.png)

## <a name="how-do-i-view-which-http-calls-are-used-in-my-services"></a>如何檢視服務中所使用的 HTTP 呼叫？

1. 在同一個 Application Insights 資源中，您可以篩選「要求」而非例外狀況，然後檢視所提出的所有要求
2. 如果您使用 Service Fabric Application Insights SDK，您會看到服務彼此連線的視覺呈現，以及已成功和已失敗的要求數目。 在左側按一下 [應用程式對應]

    ![AI 應用程式對應刀鋒視窗](media/service-fabric-diagnostics-common-scenarios/app-map-blade.png) ![AI 應用程式對應](media/service-fabric-diagnostics-common-scenarios/app-map-new.png)

    如需應用程式對應的詳細資訊，請瀏覽[應用程式對應文件](../application-insights/app-insights-app-map.md)

## <a name="how-do-i-create-an-alert-when-a-node-goes-down"></a>如何在節點故障時建立警示

1. Service Fabric 叢集會追蹤節點事件。 請瀏覽至名為 **ServiceFabric(NameofResourceGroup)** 的 Service Fabric 分析解決方案資源
2. 按一下刀鋒視窗底部標題為「摘要」的圖形

    ![OMS 解決方案](media/service-fabric-diagnostics-common-scenarios/oms-solution-azure-portal.png)

3. 這裡有許多圖形和圖格，並顯示各種計量。 對其中一個圖形按一下，您就會前往 [記錄搜尋]。 您可以在此查詢任何叢集事件或效能計數器。
4. 輸入下列查詢。 這些事件識別碼位於[節點事件參考](service-fabric-diagnostics-event-generation-operational.md#application-events)中

    ```kusto
    ServiceFabricOperationalEvent
    | where EventId >= 25623 or EventId <= 25626
    ```

5. 按一下頂端的 [新增警示規則]，現在，每當有根據這項查詢的事件抵達時，您就會在所選擇的通訊方法中收到警示。

    ![OMS 的新警示](media/service-fabric-diagnostics-common-scenarios/oms-create-alert.png)

## <a name="how-can-i-be-alerted-of-application-upgrade-rollbacks"></a>如何收到應用程式升級復原的警示？

1. 在和前面相同的 [記錄搜尋] 視窗上，輸入下列升級復原查詢。 這些事件識別碼位於[應用程式事件參考](service-fabric-diagnostics-event-generation-operational.md#application-events)中

    ```kusto
    ServiceFabricOperationalEvent
    | where EventId == 29623 or EventId == 29624
    ```

2. 按一下頂端的 [新增警示規則]，現在，每當有根據這項查詢的事件抵達時，您就會收到警示。

## <a name="how-do-i-see-container-metrics"></a>如何查看容器計量？

在具有所有圖形的相同檢視中，您會看到一些顯示出容器效能的圖格。 您必須有 OMS 代理程式和[容器監視解決方案](service-fabric-diagnostics-oms-containers.md)，這些圖格才會填入資料。

![OMS 容器計量](media/service-fabric-diagnostics-common-scenarios/containermetrics.png)

>[!NOTE]
>若要從容器**內部**檢測遙測，您必須新增[適用於容器的 Application Insights Nuget 套件](https://github.com/Microsoft/ApplicationInsights-servicefabric#microsoftapplicationinsightsservicefabric--for-service-fabric-lift-and-shift-scenarios)。

## <a name="how-can-i-monitor-performance-counters"></a>如何監視效能計數器？

1. 在叢集中新增 OMS 代理程式後，您必須新增所要追蹤的特定效能計數器。請瀏覽至入口網站中的 [OMS 工作區] 頁面 – 從解決方案的頁面來看，[工作區] 索引標籤位於左功能表中。

    ![[OMS 工作區] 索引標籤](media/service-fabric-diagnostics-common-scenarios/workspacetab.png)

2. 到達工作區頁面後，在同一個左側功能表中按一下 [進階設定]。

    ![OMS 進階設定](media/service-fabric-diagnostics-common-scenarios/advancedsettingsoms.png)

3. 按一下 [資料] > [Windows 效能計數器] (如果是 Linux 機器，則按一下 [資料] > [Linux 效能計數器])，以透過 OMS 代理程式開始收集節點的特定計數器。 要新增的計數器格式範例如下

    * `.NET CLR Memory(<ProcessNameHere>)\\# Total committed Bytes`
    * `Processor(_Total)\\% Processor Time`
    * `Service Fabric Service(*)\\Average milliseconds per request`

    在本快速入門中，所使用的程序名稱為 VotingData 和 VotingWeb，因此追蹤這些計數器看起來會像

    * `.NET CLR Memory(VotingData)\\# Total committed Bytes`
    * `.NET CLR Memory(VotingWeb)\\# Total committed Bytes`

    ![OMS 效能計數器](media/service-fabric-diagnostics-common-scenarios/omsperfcounters.png)

4. 這能讓您看到基礎結構處理工作負載的情形，並設定根據資源使用量的相關警示。 例如，您可以設定「如果處理器總使用率高於 90% 或低於 5%」的警示。 對此，您所使用的計數器名稱會是「% Processor Time」。 若要這麼做，您可以對下列查詢建立警示規則：

    ```kusto
    Perf | where CounterName == "% Processor Time" and InstanceName == "_Total" | where CounterValue >= 90 or CounterValue <= 5.
    ```

## <a name="how-do-i-track-performance-of-my-reliable-services-and-actors"></a>如何追蹤 Reliable Services 和 Reliable Actors 的效能？

若要追蹤應用程式中的 Reliable Services 或 Reliable Actors 效能，也請新增「Service Fabric 動作項目」、「動作項目方法」、「服務」和「服務方法」等計數器。 您可以使用和上述案例類似的方式新增這些計數器，以下是要在 OMS 中新增的可靠服務和動作項目效能計數器範例

* `Service Fabric Service(*)\\Average milliseconds per request`
* `Service Fabric Service Method(*)\\Invocations/Sec`
* `Service Fabric Actor(*)\\Average milliseconds per request`
* `Service Fabric Actor Method(*)\\Invocations/Sec`

如需 Reliable [Services](service-fabric-reliable-serviceremoting-diagnostics.md) 和 [Actors](service-fabric-reliable-actors-diagnostics.md) 效能計數器的完整清單，請查看這些連結

## <a name="next-steps"></a>後續步驟

* [在 AI 中設定警示](../application-insights/app-insights-alerts.md)以收到效能或使用方式的變更通知
* [Application Insights 的智慧偵測](../application-insights/app-insights-proactive-diagnostics.md)會對傳送至 AI 的遙測資料執行主動式分析，對可能的效能問題提出警告。
* 深入了解 OMS Log Analytics [警示](../log-analytics/log-analytics-alerts.md)，以協助偵測和診斷。
* 針對內部部署叢集，OMS 提供可用來傳送資料至 OMS 的閘道 (HTTP 正向 Proxy)。 如需詳細資訊，請參閱[在無網際網路存取下使用 OMS 閘道將電腦連線到 OMS](../log-analytics/log-analytics-oms-gateway.md)
* 熟悉 Log Analytics 的[記錄搜尋和查詢](../log-analytics/log-analytics-log-searches.md)功能
* 若要深入了解 Log Analytics 及其提供的功能，請參閱 [Log Analytics 是什麼？](../operations-management-suite/operations-management-suite-overview.md)
