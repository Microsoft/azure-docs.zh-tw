---
title: 監視 Azure 監視器所收集的資料 | Microsoft Docs
description: 監視 Azure 監視器所收集的資料，會分為輕量型且能夠支援近乎即時之案例的計量，以及儲存於 Log Analytics 以供進階分析使用的記錄。
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.service: monitoring
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/27/2018
ms.author: bwren
ms.openlocfilehash: 19a611ca88310f06503bea2b8606699fe3e1c709
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406035"
---
# <a name="monitoring-data-collected-by-azure-monitor"></a>監視 Azure 監視器所收集的資料
[Azure 監視器](../azure-monitor/overview.md)是一項服務，可協助您監視應用程式及其所依賴的資源。 此功能的核心是儲存來自受監視資源的遙測和其他資料。 本文提供如何透過 Azure 監視器來儲存和使用此資料的完整說明。

Azure 監視器所收集的所有資料均符合下列兩個基本類型之一：[計量](#metrics)和[記錄](#logs)。 計量為數值，可描述系統在特定時間點的某個方面。 它們屬於輕量型，而且能夠支援近乎即時的案例。 記錄包含不同類型的資料，而資料會針對每個類型組織成不同的屬性集。 除了效能資料，還會將事件和追蹤之類的遙測資料儲存為記錄，讓它能夠全部組合在一起進行分析。

![Azure 監視器概觀](../azure-monitor/media/overview/overview.png)

## <a name="metrics"></a>度量
計量為數值，描述系統在特定時間的某個方面。 它們屬於輕量型，而且能夠支援近乎即時的案例。 不論值是否變更，系統都會定期收集計量。 它們適合用於警示，因為它們可頻繁地進行取樣，而且可使用相對較簡單的邏輯快速引發警示。 

例如，您可能會每分鐘從虛擬機器收集處理器使用率，或每隔 10 分鐘收集已登入您應用程式的使用者數目。 當所收集之值的任一個值或甚至兩個值之間的差異超過所定義的閾值時，您可以引發警示。

Azure 中計量的特定屬性包括下列項目：

* 以每隔一分鐘的頻率來收集 (除非在計量定義中另有指定)。
* 依計量名稱和用來作為分類的命名空間進行唯一識別。
* 儲存 93 天。 您可以將計量複製到 Log Analytics，以進行長期趨勢分析。

每個計量值都具有下列屬性：
* 收集到值的時間。
* 值所代表的量值類型。
* 與值相關聯的資源。
* 值本身。
* 某些計量可能具有多個維度，如下一節所述。 自訂計量最多可以有 10 個維度。

### <a name="multi-dimensional-metrics"></a>多維度計量
計量的維度是成對的名稱和數值，其中含有額外的資料來說明計量值。 例如，「可用磁碟空間」計量可以有名為「磁碟機」且值為 _C:_、_D:_ 的維度，讓您能夠檢視所有磁碟機或每個磁碟機的可用磁碟空間。 

下列範例說明適用於名為「網路輸送量」之假設性計量的兩個資料集。 第一個資料集沒有維度。 第二個資料集會顯示具有兩個維度 (I_P Address_ 與「方向」) 的值：

### <a name="network-throughput"></a>網路輸送量

 |Timestamp        | 計量值 | 
   | ------------- |:-------------| 
   | 8/9/2017 8:14 | 1,331.8 Kbps | 
   | 8/9/2017 8:15 | 1,141.4 Kbps |
   | 8/9/2017 8:16 | 1,110.2 Kbps |

這個無維度計量只能回答像是「我的網路輸送量在指定時間為何」這種基本問題。

### <a name="network-throughput--two-dimensions-ip-and-direction"></a>網路輸送量 + 兩個維度 (「IP」與「方向」)

| Timestamp          | 維度「IP」 | 維度「方向」 | 計量值| 
   | ------------- |:-----------------|:------------------- |:-----------|  
   | 8/9/2017 8:14 | IP="192.168.5.2" | 方向="傳送"    | 646.5 Kbps |
   | 8/9/2017 8:14 | IP="192.168.5.2" | 方向="接收" | 420.1 Kbps |
   | 8/9/2017 8:14 | IP="10.24.2.15"  | 方向="傳送"    | 150.0 Kbps | 
   | 8/9/2017 8:14 | IP="10.24.2.15"  | 方向="接收" | 115.2 Kbps |
   | 8/9/2017 8:15 | IP="192.168.5.2" | 方向="傳送"    | 515.2 Kbps |
   | 8/9/2017 8:15 | IP="192.168.5.2" | 方向="接收" | 371.1 Kbps |
   | 8/9/2017 8:15 | IP="10.24.2.15"  | 方向="傳送"    | 155.0 Kbps |
   | 8/9/2017 8:15 | IP="10.24.2.15"  | 方向="接收" | 100.1 Kbps |

此計量可以回答像是「每個 IP 位址的網路輸送量為何」與「資料的傳送與接收比為何」這種問題。 相較於無維度計量，多維度計量具有分析與診斷值。

### <a name="value-of-metrics"></a>計量值
個別計量通常自行提供很少的見解。 它們提供單一值，而沒有任何可與簡單閾值比擬的內容。 個別計量在下列情況很實用：與其他計量結合時以識別模式和趨勢，或與可提供以特定值為主之內容的記錄結合時。 

例如，您的應用程式在特定時間的特定使用者數目可能難以告訴您應用程式的健康情況。 但是以相同計量的多個值所表示的使用者數突然下降，可能表示有問題。 由應用程式擲回並以不同計量表示的過度例外狀況，可能可以識別導致下降的應用程式問題。 應用程式在其元件中建立來識別失敗的事件可協助您識別根本原因。

### <a name="sources-of-metric-data"></a>計量資料的來源
Azure 監視器所收集的計量有三個基本來源。 這些計量全都適用於計量存放區，不論其來源為何，都可在該存放區中一起進行評估。

**平台計量**是由 Azure 資源所建立，讓您能夠掌握其健康情況和效能。 每個類型的資源都會建立[一組不同的計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md)，而不需進行任何必要的設定。 

**應用程式計量**是由 Application Insights 為您受監視的應用程式所建立，可協助您偵測效能問題，以及追蹤應用程式的使用趨勢。 這包括像是「伺服器回應時間」和「瀏覽器例外狀況」之類的值。

**自訂計量**是除了自動可供使用的標準計量之外您所定義的計量。 您必須針對與該資源相同區域中的單一資源建立自訂計量。 您可以使用下列方法來建立自訂計量：
    - [在應用程式中定義自訂計量](../application-insights/app-insights-api-custom-events-metrics.md)，透過 Application Insights 進行監視。 這些是除了標準的應用程式計量組合以外的計量。
    - 使用 [Windows 診斷延伸模組 (WAD)](../monitoring-and-diagnostics/azure-diagnostics.md)，從您的 Windows 虛擬機器發佈自訂計量。
    - 使用 [InfluxData Telegraf 代理程式](https://www.influxdata.com/time-series-platform/telegraf/)，從您的 Linux 虛擬機器發佈自訂計量。
    - 使用自訂計量 API，從 Azure 服務撰寫自訂計量。
    
![計量概觀](media/monitoring-data-collection/metrics-overview.png)

### <a name="what-can-you-do-with-metrics"></a>度量能讓您做什麼？
您可以使用計量來執行的工作包括下列各項：

- 使用[計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)來分析收集到的計量，並將它們繪製於圖表上。 追蹤資源 (例如 VM、網站或邏輯應用程式) 的效能，只要將該圖表釘選到 [Azure 儀表板](../azure-portal/azure-portal-dashboards.md)即可。
- 設定[計量警示規則](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)，在計量超出閾值時，傳送通知或採取[自動化動作](../monitoring-and-diagnostics/monitoring-action-groups.md)。
- 使用[自動調整規模](../monitoring-and-diagnostics/monitoring-overview-autoscale.md)，根據超出閾值的計量來增加或減少資源。
- 將計量路由傳送到 Log Analytics 來分析計量資料與記錄資料，並儲存計量值超過 93 天。 
- 將計量串流處理到[事件中樞](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)，以將它們路由傳送到 [Azure 串流分析](../stream-analytics/stream-analytics-introduction.md)或外部系統。
- [封存](../monitoring-and-diagnostics/monitor-tutorial-archive-monitoring-data.md)資源的效能或健全狀況歷程記錄，以用於相容性、稽核或離線報告。
- 使用 [PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.insights/?view=azurermps-6.7.0) 或 [REST API](../monitoring-and-diagnostics/monitoring-rest-api-walkthrough.md)，從命令列或自訂應用程式中存取計量值。



### <a name="viewing-metrics"></a>檢視計量
Azure 監視器中的計量均會收集於 Azure 監視器計量存放區中。 這是已最佳化來進行快速存取的時間序列資料庫，並會儲存計量值 93 天。 將計量複製到 Log Analytics，以用於長期分析與趨勢。

您可以透過上述各種方法來使用計量資料。 使用[計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)，直接分析計量存放區中的資料，並隨著時間繪製多個計量值的圖表。 您可以互動方式檢視圖表，或將其釘選到儀表板，利用其他視覺效果進行檢視。 您也可以使用 [Azure 監視 REST API](../monitoring-and-diagnostics/monitoring-rest-api-walkthrough.md) 來擷取計量。

![計量瀏覽器](media/monitoring-data-collection/metrics-explorer.png)





## <a name="logs"></a>記錄檔
記錄包含不同類型的資料，而資料會針對每個類型組織成不同的屬性集。 記錄可以包含數值 (例如計量)，但通常會包含具有詳細說明的文字資料。 其在結構方面進一步與計量不同，而且通常不會定期收集。

常見的記錄項目類型是偶爾會收集的事件。 事件是由應用程式或服務所建立，而且通常包含足夠的資訊來自行提供完整的內容。 例如，事件可以表示已建立或修改特定資源、為了回應增加的流量而啟動新主機，或在應用程式中偵測到錯誤。

記錄特別適合用於結合來自各種來源的資料，以進行複雜分析及隨著時間變化的趨勢分析。 資料的格式可能有所不同，應用程式可以使用其所需的結構建立自訂記錄。 計量甚至可在記錄中進行複寫，將它們與其他監視資料結合，以進行趨勢和其他資料分析。



### <a name="log-analytics"></a>Log Analytics
Azure 監視器所收集的記錄都會儲存於 Log Analytics，其會從各種來源收集遙測和其他資料。 它提供豐富的查詢語言和分析引擎，可讓您深入解析應用程式和資源的作業。 像是 [Azure 資訊安全中心](../security-center/security-center-intro.md)的其他 Azure 服務均會將資料儲存於 Log Analytics，以便跨 Azure 管理提供常見的資料平台。

> [!IMPORTANT]
> 來自 Application Insights 的資料會像其他記錄資料一樣儲存於 Log Analytics，不同之處在於它會儲存於不同的磁碟分割。 這支援與其他 Log Analytics 資料相同的功能，但您必須使用 [Application Insights 主控台](/application-insights/app-insights-analytics.md)或 [Application Insights API](https://dev.applicationinsights.io/) 來存取此資料。 您可以使用[跨資源查詢](../log-analytics/log-analytics-cross-workspace-search.md)來分析應用程式資料和其他記錄資料。


### <a name="sources-of-log-data"></a>記錄資料的來源
Log Analytics 可以在 Azure 中及內部部署資源的各種來源收集資料。 寫入到 Log Analytics 的資料來源包含下列各項：

- 來自 Azure 資源的[活動記錄](../log-analytics/log-analytics-activity.md) (包括其設定和健康情況的相關資訊) 與[診斷記錄](../monitoring-and-diagnostics/monitor-stream-diagnostic-logs-log-analytics.md) (可深入解析它們的作業)。
- [Windows](../log-analytics/log-analytics-windows-agent.md) 和 [Linux](../log-analytics/log-analytics-linux-agents.md) 虛擬機器上的代理程式會根據您設定的[資料來源](../log-analytics/log-analytics-data-sources.md)，將遙測從客體作業系統和應用程式傳送到 Log Analytics。
- [Application Insights](https://docs.microsoft.com/azure/application-insights/) 所收集的應用程式資料。
- 可從[監視解決方案](../monitoring/monitoring-solutions.md)或功能 (例如，容器深入解析、VM Insights 或資源群組深入解析) 來深入解析特定應用程式或服務的資料。
- [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/)所收集的安全性資料。
- 來自 Azure 資源的[計量](#metrics)。 這可讓您儲存計量超過 93 天，並利用其他記錄資料進行分析。
- 寫入到 [Azure 儲存體](../log-analytics/log-analytics-azure-storage-iis-table.md)的遙測資料。
- 使用 [HTTP 資料收集器 API](../log-analytics/log-analytics-data-collector-api.md) 用戶端，從任何 REST API 用戶端收集而來的自訂資料，或來自 [Azure Logic App](https://docs.microsoft.com/azure/logic-apps/) 工作流程的自訂資料。

![Log Analytics 元件](media/monitoring-data-collection/logs-overview.png)




### <a name="what-can-you-do-with-logs"></a>記錄能讓您做什麼？
您可以使用記錄來執行的工作包括下列各項：

- 在 Azure 入口網站中，使用 [Log Analytics 頁面](../log-analytics/query-language/get-started-analytics-portal.md)來撰寫分析記錄資料的查詢。  將轉譯為表格或圖表的結果釘選到 [Azure 儀表板](../azure-portal/azure-portal-dashboards.md)。
- 設定[記錄警示規則](../monitoring-and-diagnostics/alert-log.md)，在查詢結果符合特定結果時，傳送通知或採取[自動化動作](../monitoring-and-diagnostics/monitoring-action-groups.md)。
- 使用 [Logic Apps]()，根據 Log Analytics 中的資料建置工作流程。
- 將查詢的結果匯出到 [Power BI](../log-analytics/log-analytics-powerbi.md) 以使用不同的視覺效果，並與 Azure 外部的使用者共用。
- 使用 [PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.operationalinsights/?view=azurermps-6.8.1) 或 [REST API](https://dev.loganalytics.io/)，從命令列或自訂應用程式中存取計量值。

### <a name="viewing-log-data"></a>檢視記錄資料
使用[記錄查詢](../log-analytics/log-analytics-queries.md)擷取 Log Analytics 的所有資料，該查詢會指定一組特定的資料。 使用 [Log Analytics 查詢語言](../log-analytics/query-language/get-started-queries.md)來撰寫查詢，該語言為豐富的查詢語言，可用於快速擷取、彙總及分析收集到的資料。 使用 Azure 入口網站中的 [Log Analytics 頁面](../log-analytics/log-analytics-log-search-portals.md)，直接分析計量存放區中的資料，並隨著時間繪製多個計量值的圖表。 您可以互動方式檢視圖表，或將其釘選到儀表板，利用其他視覺效果進行檢視。 您也可以使用 [Azure 監視 REST API](../monitoring-and-diagnostics/monitoring-rest-api-walkthrough.md) 來擷取計量。

![記錄檔](media/monitoring-data-collection/logs.png)

## <a name="convert-monitoring-data"></a>轉換監視資料

### <a name="metrics-to-logs"></a>計量至記錄
您可以將計量複製到 Log Analytics，以使用其豐富的查詢語言來執行其他資料類型的複雜分析。 您也可以將記錄資料保留得比計量久，這可讓您執行一段時間的趨勢分析。 當計量或任何其他效能資料儲存在 Log Analytics 時，該資料可當作記錄。 使用記錄進行其他資料的趨勢分析時，使用計量來支援近乎即時的分析和警示。

您可以在[收集 Azure 服務記錄和計量以便使用於 Log Analytics](../log-analytics/log-analytics-azure-storage.md)，取得從 Azure 資源收集計量的指引。 在[使用 Log Analytics 設定 Azure PaaS 資源計量的集合](../log-analytics/log-analytics-collect-azurepass-posh.md)，可取得從 Azure PaaS 資源收集資源計量的指引。

### <a name="logs-to-metrics"></a>計量至記錄
如上所述，計量比記錄更有回應，因此您能以較低成本建立延遲較低的警示。 Log Analytics 會收集大量數值資料，這些資料適合用於計量，但不會儲存於 Azure 計量存放區。  常見範例是從代理程式和管理解決方案收集的效能資料。 您可以將這其中一些值複製到計量存放區，以便用於警示以及透過計量瀏覽器進行分析。

這項功能的說明位於[為 Azure 監視器中的記錄建立計量警示](../monitoring-and-diagnostics/monitoring-metric-alerts-logs.md)。 值的支援清單位於[支援 Azure 監視器的計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftoperationalinsightsworkspaces)。

## <a name="stream-data-to-external-systems"></a>將資料串流處理到外部系統
除了使用 Azure 中的工具來分析監視資料，您可能也有將監視資料轉送到外部工具的需求，例如安全性資訊和事件管理 (SIEM) 產品。 此轉送通常是透過 [Azure 事件中樞](https://docs.microsoft.com/azure/event-hubs/)，直接從受監視的資源來完成。 

您可以在[將 Azure 監視資料串流至事件中樞以供外部工具取用](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)取得各種監視資料的指引。

## <a name="next-steps"></a>後續步驟

- 深入了解可用於 Azure 中不同資源的[監視資料](monitoring-data-sources.md)。