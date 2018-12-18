---
title: Azure 監視器概觀 | Microsoft Docs
description: 構成 Azure 服務和應用程式之完整監視策略的各種 Microsoft 服務和功能的概觀。
author: bwren
manager: carmonm
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.service: azure-monitor
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/26/2018
ms.author: bwren
ms.openlocfilehash: 0f17cf7f2ef655f482b08b5c7163c0d85be77ce5
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49068687"
---
# <a name="azure-monitor-overview"></a>Azure 監視器概觀

Azure 監視器可藉由提供全方位的解決方案，以便收集、分析及處理來自雲端和內部部署環境的遙測資料，進而將應用程式的可用性和效能最大化。 它可協助您了解您的應用程式表現如何，並主動識別影響它們的問題以及它們所依賴的資源。

> [!VIDEO https://www.youtube.com/embed/_hGff5bVtkM]

## <a name="overview"></a>概觀
下圖提供 Azure 監視器的高階檢視。 圖表的中間是計量和記錄的資料存放區，而這兩者是 Azure 監視器所用資料的兩個基本類型。 左邊是[從不同受監視資源收集遙測的來源](../monitoring/monitoring-data-sources.md)，而且會填入[資料存放區](../monitoring/monitoring-data-collection.md)。 右邊是 Azure 監視器對此收集資料執行的不同功能，例如分析、警示及串流處理至外部系統。


![Azure 監視器概觀](media/overview/overview.png)


## <a name="monitoring-data-platform"></a>監視資料平台
Azure 監視器所收集的所有資料均符合下列兩個基本類型之一：[計量和記錄](../monitoring/monitoring-data-collection.md)。 [計量](../monitoring/monitoring-data-collection.md#metrics)為數值，可描述系統在特定時間點的某個方面。 它們屬於輕量型，而且能夠支援近乎即時的案例。 [記錄](../monitoring/monitoring-data-collection.md#logs)包含不同類型的資料，而資料會針對每個類型組織成不同的屬性集。 除了效能資料，還會將事件和追蹤之類的遙測資料儲存為記錄，讓它能夠全部組合在一起進行分析。

對於許多 Azure 資源，您會在 Azure 入口網站的 [概觀] 頁面當中看到 Azure 監視器所收集的資料。 比方說，查看一下任何虛擬機器，您會看到數個顯示效能計量的圖表。 按一下任何圖表，以在 Azure 入口網站的[計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)中開啟資料，可讓您將一段時間的多個計量值繪製成圖表。  您可以互動方式檢視圖表，或將其釘選到儀表板，利用其他視覺效果進行檢視。

![度量](media/overview/metrics.png)

Azure 監視器所收集的記錄資料會儲存在 Log Analytics 中，其中包含的[豐富查詢語言](../log-analytics/log-analytics-queries.md)可快速擷取、彙總及分析所收集的資料。  您可以在 Azure 入口網站中使用 [Log Analytics 頁面](../log-analytics/log-analytics-log-search-portals.md)來建立和測試查詢，然後使用這些工具直接分析資料，或儲存查詢以便搭配[視覺效果](visualizations.md)或[警示規則](../monitoring-and-diagnostics/monitoring-overview-alerts.md)使用。

Log Analytics 查詢語言適合用於簡單的記錄查詢，但也包含進階的功能，例如彙總、聯結和智慧分析。 您可以使用可用的[多個課程](../log-analytics/query-language/get-started-queries.md)，快速了解查詢語言。  我們會為已經熟悉 [SQL](../log-analytics/query-language/sql-cheatsheet.md) 和 [Splunk](../log-analytics/query-language/splunk-cheatsheet.md) 的使用者提供特別指引。

![記錄檔](media/overview/logs.png)

## <a name="what-data-does-azure-monitor-collect"></a>Azure 監視器會收集哪些資料？
Azure 監視器可以從各種來源收集資料。 您可以考慮為各層中的應用程式監視資料，範圍從您的應用程式、它所倚賴的任何作業系統和服務，以至平台本身。 Azure 監視器會從下列各層收集資料：

- **應用程式監視資料：** 您所撰寫程式碼的效能和功能相關資料 (不論其平台為何)。
- **客體 OS 監視資料：** 有關應用程式執行所在作業系統的資料。 這可以在 Azure、其他雲端或內部部署中執行。 
- **Azure 資源監視資料：** 有關 Azure 資源作業的資料。
- **Azure 訂用帳戶監視資料：** 有關 Azure 訂用帳戶作業和管理的資料，以及有關 Azure 本身健康情況和作業的資料。 
- **Azure 租用戶監視資料：** 租用戶層級 Azure 服務的作業相關資料，例如 Azure Active Directory。

當您建立 Azure 訂用帳戶並開始新增資源 (例如虛擬機器和 Web 應用程式) 時，Azure 監視器就會開始收集資料。  [活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)會記錄資源的建立或修改時間。 [計量](../monitoring-and-diagnostics/monitoring-overview-metrics.md)會告訴您如何執行資源以及它所取用的資源。 

[啟用診斷](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)並[新增代理程式](../log-analytics/log-analytics-agent-windows.md)來計算資源，以將您所收集的資料擴展至實際資源運作。 這會收集資源內部作業的遙測資料，並可讓您設定不同的[資料來源](../log-analytics/log-analytics-data-sources.md)，以從 Windows 和 Linux 客體作業系統收集記錄和計量。 

[將檢測套件新增至您的應用程式](../application-insights/app-insights-azure-web-apps.md)，讓 Application Insights 能夠收集您應用程式的詳細資訊，包括頁面檢視、應用程式要求和例外狀況。 設定[可用性測試](../application-insights/app-insights-monitor-web-app-availability.md)來模擬使用者流量，進一步確認您應用程式的可用性。

### <a name="custom-sources"></a>自訂來源
Azure 監視器可以使用[資料收集器 API](../log-analytics/log-analytics-data-collector-api.md)，從任何 REST 用戶端收集記錄資料。 這可讓您建立自訂監視案例，並且將監視延伸到不會透過其他來源公開遙測的資源。



## <a name="insights"></a>深入解析
如果監視資料可以提高您對於運算環境作業的可見性，監視資料才有用處。 Azure 監視器包含數個功能和工具，可對您的應用程式及其相依的其他資源提供寶貴的深入解析。 [監視解決方案](../monitoring/monitoring-solutions.md)和功能 (例如 [Application Insights](../application-insights/app-insights-overview.md) 和 Container Insights) 可為您的應用程式和特定 Azure 服務的不同層面提供深入解析。 

### <a name="application-insights"></a>Application Insights
[Application Insights](../application-insights/app-insights-overview.md) 可監視 Web 應用程式的可用性、效能及使用情況 (不論應用程式是裝載在雲端還是內部部署環境)。 它會利用 Log Analytics 中強大的資料分析平台，為您提供應用程式作業的深入解析以及診斷錯誤，毋需等待使用者回報錯誤。 Application Insights 包含各種開發工具的連接點，並與 Visual Studio 整合以支援您的 DevOps 程序。

![應用程式情資](media/overview/app-insights.png)

### <a name="azure-monitor-for-containers"></a>適用於容器的 Azure 監視器
適用於容器的 Azure 監視器是一項功能，其設計訴求是要監視部署至 Azure Kubernetes Service (AKS) 上所裝載受控 Kubernetes 叢集的容器工作負載效能。 它可透過計量 API 從 Kubernetes 中提供的控制器、節點與容器收集記憶體與處理器計量，為您提供效能可見性。 容器記錄也會一併收集。  在您能夠從 Kubernetes 叢集監視之後，系統會透過適用於 Linux 的 Log Analytics 代理程式容器化版本自動收集這些計量和記錄，並儲存在 Log Analytics 中。

![容器健康情況](media/overview/container-insights.png)

### <a name="azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器
Azure 監視器的 VM Insights 可藉由分析 Windows 和 Linux VM 的效能和健康情況 (包括其不同的程序以及其他資源和外部程序的互連相依性)，大規模監視 Azure 虛擬機器 (VM)。 此解決方案包含針對在內部部署環境或其他雲端提供者上裝載的 VM，監視效能和應用程式相依性的支援。  


![VM Insights](media/overview/vm-insights.png)

### <a name="monitoring-solutions"></a>監視解決方案
Azure 監視器中的[監視解決方案](../monitoring/monitoring-solutions.md)是幾組封裝的邏輯，可提供特定應用程式或服務的深入解析。 它們會將資料連同其他監視資料一起收集到 Log Analytics 中，並使用[查詢](../log-analytics/log-analytics-queries.md)進行分析，以及使用[檢視](../log-analytics/log-analytics-view-designer.md)呈現視覺效果。 您可以從 [Microsoft](../monitoring/monitoring-solutions-inventory.md) 及合作夥伴取得監視解決方案，為各種 Azure 服務和其他應用程式提供監視功能。

![監視解決方案](../monitoring/media/monitoring-solutions/overview.png)

## <a name="responding-to-critical-situations"></a>回應重大情況
有效的監視解決方案除了可讓您以互動方式分析監視資料，還必須能夠主動回應在它所收集資料中找到的重大情況。 可以是傳送文字或郵件給負責調查問題的系統管理員。 或者，您可啟動自動化程序，嘗試更正錯誤情況。


### <a name="alerts"></a>警示
[Azure 監視器中的警示](../monitoring-and-diagnostics/monitoring-overview-alerts.md)會主動通知您重大情況，並可能嘗試採取矯正措施。 以計量為基礎的警示規則會根據數值提供近乎即時的警示，而以記錄為基礎的規則則允許使用橫跨多個來源資料的複雜邏輯。

Azure 監視器中的警示規則會使用[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，其中包含幾組獨特的收件人以及多個規則可共用的動作。 根據您的需求，動作群組可以執行一些動作，例如使用 Webhook 讓警示啟動外部動作，或與 ITSM 工具整合。

![警示](media/overview/alerts.png)

### <a name="autoscale"></a>Autoscale
自動調整可讓您執行適當數量的資源來處理應用程式的負載。 它可讓您建立一些規則，這些規則會使用 Azure 監視器所收集的計量來決定何時自動新增資源來處理增加的負載，並可藉由移除閒置資源來節省成本。 您可指定執行個體的數目上限和下限，以及何時要增加或減少資源的邏輯。

![Autoscale](media/overview/autoscale.png)

## <a name="visualizing-monitoring-data"></a>將監視資料視覺化
[視覺效果](visualizations.md) (例如圖表和資料表) 是摘要監視資料以及向不同對象呈現資料的有效工具。 Azure 監視器有自己的功能可將監視資料視覺化，並運用其他 Azure 服務向不同的對象發佈資料。

### <a name="dashboards"></a>儀表板
[Azure 儀表板](../azure-portal/azure-portal-dashboards.md)可讓您將不同種類的資料 (包括計量和記錄) 結合至 [Azure 入口網站](https://portal.azure.com)的單一窗格中。 您可以選擇性地與其他 Azure 使用者共用儀表板。 您可以將 Azure 監視器的元素新增至 Azure 儀表板 (任何記錄查詢或計量圖表的輸出除外)。 例如，您可以建立一個儀表板，將顯示計量圖表、活動記錄表、來自 Application Insights 的使用情況圖表，以及來自 Log Analytics 的查詢輸出的圖格結合在一起。

![儀表板](media/overview/dashboard.png)

### <a name="views"></a>Views
[Azure 監視器中的檢視](../log-analytics/log-analytics-view-designer.md)通常會以視覺化方式呈現 Log Analytics 中的記錄資料。  每個檢視都包含單一圖格，該圖格可向下切入至視覺效果的組合，例如長條圖和折線圖 (摘要重要資料的清單除外)。  監視解決方案包含可摘要特定應用程式資料的檢視，而且您可以建立自己的檢視，以呈現來自任何 Log Analytics 記錄搜尋的資料。 如何 Azure 監視器中的其他元素，可以將檢視新增至 Azure 儀表板。

![Log Analytics 檢視](media/overview/view.png)

### <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com) 是一項商務分析服務，可提供橫跨各種資料來源的互動式視覺效果，而且是將資料提供給貴組織內外的其他人使用的有效方法。 您可以將 Power BI 設定為[從 Azure 監視器自動匯入記錄資料](../log-analytics/log-analytics-powerbi.md)，以便利用這些額外的視覺效果。


![Power BI](media/overview/power-bi.png)


## <a name="integrate-and-export-data"></a>整合並匯出資料
您通常需要整合 Azure 監視器與其他系統，以及建置自訂解決方案來使用您的監視資料。 其他 Azure 服務會與 Azure 監視器搭配運作來提供這項整合。

### <a name="event-hub"></a>事件中樞
[Azure 事件中樞](https://docs.microsoft.com/azure/event-hubs)是一個串流平台和事件擷取服務，可使用任何即時分析提供者或批次/儲存配接器來轉換和儲存資料。 使用事件中樞[將來自 Azure 監視器的記錄資料串流處理](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)至夥伴 SIEM 和監視工具。

> [!VIDEO https://www.youtube.com/embed/SPHxCgbcvSw]

### <a name="logic-apps"></a>Logic Apps
[Logic Apps](https://azure.microsoft.com/services/logic-apps) 是一項服務，可讓您使用可與不同系統和服務整合的工作流程，自動執行工作和商務程序。 可用於讀取和寫入 Azure 監視器中計量和記錄的活動，可讓您建置可與其他各種系統整合的工作流程。

![邏輯應用程式](../log-analytics/media/log-analytics-activity-logs-subscriptions/log-analytics-logic-apps-activity-log-overview.png)

### <a name="api"></a>API
除了存取所產生的警示，還有多個 API 可供讀取和寫入 Azure 監視器中的計量和記錄。 您也可以設定和擷取警示。 這可為您提供基本上無限制的可能性，來建置可與 Azure 監視器整合的自訂解決方案。

## <a name="next-steps"></a>後續步驟
深入了解：

* Azure 監視器所收集資料的[計量和記錄](../monitoring/monitoring-data-collection.md)。
* [資料來源](../monitoring/monitoring-data-sources.md)，以了解應用程式的不同元件如何傳送遙測。
* [Log Analytics](../log-analytics/log-analytics-queries.md)，以分析所收集的資料。
