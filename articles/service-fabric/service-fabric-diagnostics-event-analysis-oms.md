---
title: 使用 Log Analytics 進行 Azure Service Fabric 事件分析 | Microsoft Docs
description: 了解如何使用 Log Analytics 視覺化及分析事件，以監視和分析 Azure Service Fabric 叢集。
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
ms.date: 05/29/2018
ms.author: srrengar
ms.openlocfilehash: 49d9b5306a0fcf51cc0de036c725fca8345cd0ec
ms.sourcegitcommit: ea5193f0729e85e2ddb11bb6d4516958510fd14c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36302177"
---
# <a name="event-analysis-and-visualization-with-log-analytics"></a>使用 Log Analytics 進行事件分析與視覺化
Log Analytics 會從裝載於雲端的應用程式和服務收集和分析遙測，並提供分析工具來協助您將其可用性和效能最大化。 本文將概述如何在 Log Analytics 中執行查詢，以便深入了解叢集發生什麼狀況並進行疑難排解。 我們將討論下列常見的問題：

* 如何針對健康情況事件進行疑難排解？
* 如何知道節點發生故障？
* 如何知道我的應用程式服務已經啟動或停止？

## <a name="log-analytics-workspace"></a>Log Analytics 工作區

Log Analytics 會從受控資源 (包括 Azure 儲存體資料表或代理程式) 收集資料，並在中央存放庫中維護資料。 此資料可接著用於分析、警示和視覺效果，或進一步匯出。 Log Analytics 支援事件、效能資料或任何其他自訂資料。 請查看[如何設定分析延伸模組來彙總事件](service-fabric-diagnostics-event-aggregation-wad.md)以及[如何建立 Log Analytics 工作區來讀取儲存體中的事件](service-fabric-diagnostics-oms-setup.md)，以確定資料會流入 Log Analytics。

Log Analytics 收到資料之後，Azure 會針對數個案例自訂數個*管理解決方案*，這些預先封裝的解決方案可監視傳入資料。 其中包括 *Service Fabric 分析*解決方案和*容器*解決方案，這是與使用 Service Fabric 叢集進行診斷和監視最相關的兩個解決方案。 本文說明如何使用 Service Fabric 分析解決方案，它是用工作區建立的。

## <a name="access-the-service-fabric-analytics-solution"></a>存取 Service Fabric 分析解決方案

1. 在 Azure 入口網站中，移至您建立 Service Fabric 分析解決方案所在的資源群組。

2. 選取資源 **ServiceFabric\<nameOfOMSWorkspace\>**。

2. 在 [摘要] 中，您會看到磚以圖形形式來代表每一個啟用的解決方案，其中有一個是 Service Fabric 的解決方案。 按一下 **Service Fabric** 的圖形 (第一個圖像下方) 以繼續前往 Service Fabric 分析解決方案 (第二個圖像下方)。

    ![Service Fabric 解決方案](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_summary.PNG)

    ![Service Fabric 解決方案](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_solution.PNG)

上面的圖像是 Service Fabric 分析解決方案的首頁。 這是叢集目前情況的快照集檢視。 如果您建立叢集的時候啟用了分析功能，就可以看到以下各項的事件 

* [操作通道](service-fabric-diagnostics-event-generation-operational.md)：Service Fabric 平台 (系統服務的集合) 執行的較高層級操作。
* [Reliable Actors 程式設計模型事件](service-fabric-reliable-actors-diagnostics.md)
* [Reliable Services 程式設計模型事件](service-fabric-reliable-services-diagnostics.md)

>[!NOTE]
>除了操作通道外，[更新分析延伸模組的設定](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations)可以收集更詳細的系統事件。

### <a name="view-service-fabric-events-including-actions-on-nodes"></a>檢視 Service Fabric 事件，包括節點上的動作

1. 在 Service Fabric 分析頁面上，按一下 **Service Fabric 事件**的圖形。

    ![Service Fabric 解決方案操作通道](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events_selection.png)

2. 按一下 [清單] 來檢視清單中的事件。 這裡您會看到所有收集的系統事件。 為了方便參考，這些是來自 Azure 儲存體帳戶中的 WADServiceFabricSystemEventsTable，而且同樣地，您接下來看到的 Reliable Service 和 Actor 事件是來自這些個別的表格。
    
    ![查詢操作通道](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events.png)

或者您可以按一下左邊的放大鏡，然後使用 Kusto 查詢語言來尋找您想要的資料。 例如，如果想尋找針對叢集節點所採取的動作，您可以使用下列查詢。 下面使用的事件識別碼，可以在[操作通道事件參考](service-fabric-diagnostics-event-generation-operational.md)中找到。

```kusto
ServiceFabricOperationalEvent
| where EventId < 25627 and EventId > 25619 
```

您可以查詢很多欄位，例如特定的節點 (電腦)、系統服務 (TaskName) 等。

### <a name="view-service-fabric-reliable-service-and-actor-events"></a>檢視 Service Fabric Reliable Service 和 Actor 事件

1. 在 Service Fabric 分析頁面上，按一下 **Reliable Services** 的圖形。

    ![Service Fabric 解決方案 Reliable Services](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_services_events_selection.png)

2. 按一下 [清單] 來檢視清單中的事件。 這裡您會看到來自 Reliable Services 的事件。 啟動及完成 runasync 服務時，您可以看到不同的事件，它們通常是在部署和升級發生的。 

    ![查詢 Reliable Services](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_service_events.png)

以類似的方式，就可以檢視 Reliable Actor 事件。 若要為 Reliable Actor 設定更詳細的事件，您需要到分析延伸模組的設定中去變更 `scheduledTransferKeywordFilter` (如下所示)。 [Reliable Actor 事件參考](service-fabric-reliable-actors-diagnostics.md#keywords)會有這些事件值的詳細資料。

```json
"EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
```

Kusto 查詢語言功能很強大。 您可以執行的另一個重要查詢是找出哪些節點會產生最多事件。 下列螢幕擷取畫面中的查詢會顯示如何使用特定服務和節點來彙總 Service Fabric 操作事件。

![每個節點的查詢事件](media/service-fabric-diagnostics-event-analysis-oms/oms_kusto_query.png)

## <a name="next-steps"></a>後續步驟

* 若要啟用基礎結構監視 (也就是監視效能計數器)，請前往[新增 Log Analytics 代理程式](service-fabric-diagnostics-oms-agent.md)。 這個代理程式會收集效能計數器，並將它們新增至現有的工作區。
* 針對內部部署叢集，Log Analytics 提供可用於將資料傳送至 Log Analytics 的閘道 (HTTP 正向 Proxy)。 如需詳細資訊，請參閱[在無網際網路存取下使用 OMS 閘道將電腦連線到 Log Analytics](../log-analytics/log-analytics-oms-gateway.md)
* 設定[自動化警示](../log-analytics/log-analytics-alerts.md)，以協助偵測與診斷
* 熟悉 Log Analytics 的[記錄搜尋和查詢](../log-analytics/log-analytics-log-searches.md)功能
* 若要深入了解 Log Analytics 及其提供的功能，請參閱 [Log Analytics 是什麼？](../operations-management-suite/operations-management-suite-overview.md)
