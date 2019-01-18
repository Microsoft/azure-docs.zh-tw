---
title: Azure 監視器中資料的來源 | Microsoft Docs
description: 描述可用於監視 Azure 資源 (以及在資源上執行的應用程式) 健康情況和效能的資料。
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/13/2018
ms.author: bwren
ms.openlocfilehash: 39229bbd120ca52f31a0bd54446e49990d952c9d
ms.sourcegitcommit: 25936232821e1e5a88843136044eb71e28911928
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2019
ms.locfileid: "54020127"
---
# <a name="sources-of-data-in-azure-monitor"></a>Azure 監視器中資料的來源
本文描述 Azure 監視器所收集的資料來源，這些資料可用來監視資源 (以及在資源上執行的應用程式) 健康情況和效能。 這些資源可能在 Azure、另一個雲端或內部部署中。  請參閱 [Azure 監視器所收集的資料](data-collection.md)，以取得這項資料儲存方式以及如何檢視它的詳細資訊。

Azure 中的監視資料來自各種來源，可分為多層，最高層是您的應用程式和任何作業系統，而下面幾層是 Azure 平台的元件。 在下圖中會加以說明，另外在下列章節會詳細說明各層。

![用以監視資料的層](media/data-sources/monitoring-tiers.png)

## <a name="azure-tenant"></a>Azure 租用戶
與您 Azure 租用戶相關的遙測，會從整個租用戶服務 (例如 Azure Active Directory) 收集。

![Azure 租用戶集合](media/data-sources/tenant-collection.png)

### <a name="azure-active-directory-audit-logs"></a>Azure Active Directory 稽核記錄
[Azure Active Directory 報告](../../active-directory/reports-monitoring/overview-reports.md)包含在特定租用戶內所進行登入活動的歷程記錄，以及所做變更的稽核線索。 這些稽核記錄可以寫入 Azure 監視器記錄，以和其他記錄資料一起進行分析。


## <a name="azure-platform"></a>Azure 平台
與 Azure 本身健康情況和作業相關的遙測，包含 Azure 訂用帳戶作業和管理的相關資料。 該遙測包含 Azure 活動記錄中儲存的服務健康狀態資料，以及來自 Azure Active Directory 的稽核記錄。

![Azure 訂用帳戶集合](media/data-sources/azure-collection.png)

### <a name="azure-service-health"></a>Azure 服務健康狀態
[Azure 服務健康狀態](../../azure-monitor/platform/service-notifications.md)提供您有關應用程式和資源依賴的訂用帳戶中，Azure 服務健康狀態的資訊。 您可以建立警示，針對可能會影響應用程式的目前和預期重大問題進行通知。 服務健康狀態記錄是儲存在 [Azure 活動記錄](../../azure-monitor/platform/activity-logs-overview.md)中，因此您可以在 [活動記錄總管] 中檢視記錄，以及將記錄複製到 Azure 監視器記錄。

### <a name="azure-activity-log"></a>Azure 活動記錄檔
[Azure 活動記錄](../../azure-monitor/platform/activity-logs-overview.md)包含服務健康狀態記錄，以及針對 Azure 資源所做的任何組態變更記錄。 所有 Azure 資源都會有活動記錄，也代表了資源的 _外部_ 檢視。 活動記錄中的特定記錄類型會在 [Azure 活動記錄事件結構描述](../../azure-monitor/platform/activity-log-schema.md)中說明。

您可以在 Azure 入口網站中特定資源的頁面上檢視該資源的活動記錄，或是從[活動記錄總管](../../azure-monitor/platform/activity-logs-overview.md)中檢視多個資源的記錄。 這在將記錄項目複製到 Azure 監視器，以便與其他監視資料合併時特別有用。 您也可以使用[事件中樞](../../azure-monitor/platform/activity-logs-stream-event-hubs.md)將它們傳送到其他位置。



## <a name="azure-services"></a>Azure 服務
計量和資源層級診斷記錄會提供 Azure 資源 _內部_ 作業的相關資訊。 這些記錄適用於大部分 Azure 服務，而管理解決方案會為特定服務提供額外的深入解析。

![Azure 資源集合](media/data-sources/azure-resource-collection.png)


### <a name="metrics"></a>度量
大部分 Azure 服務都會產生[平台計量](data-collection.md#metrics)，可反映其效能和作業。 特定[計量會因為各種類型的資源而異](../../azure-monitor/platform/metrics-supported.md)。  這些計量可以從 [計量瀏覽器] 存取，並且可以複製到 Log Analytics 以進行趨勢和其他分析。


### <a name="resource-diagnostic-logs"></a>資源診斷記錄
活動記錄提供在 Azure 資源上執行作業的相關資訊，而資源層級[診斷記錄](../../azure-monitor/platform/diagnostic-logs-overview.md)則提供對於資源作業本身的深入解析。   這些記錄的組態需求與內容[因資源類型而異](../../azure-monitor/platform/tutorial-dashboards.md)。

您無法在 Azure 入口網站中直接檢視診斷記錄，但是您可以[將記錄傳送到 Azure 儲存體以進行封存](../../azure-monitor/platform/archive-diagnostic-logs.md)，以及將它們匯出至[事件中樞](../../event-hubs/event-hubs-about.md)以便重新導向至其他服務，或者匯出至 [Log Analytics](../../azure-monitor/platform/diagnostic-logs-stream-log-store.md) 以進行分析。 部分資源可以直接寫入到 Log Analytics，但是其他資源則需要先寫入到儲存體帳戶，然後再[匯入到 Log Analytics](../../azure-monitor/platform/azure-storage-iis-table.md#use-the-azure-portal-to-collect-logs-from-azure-storage)。

### <a name="monitoring-solutions"></a>監視解決方案
 [監視解決方案](../insights/solutions.md)會收集資料，針對特定服務或應用程式的作業提供額外深入解析。 解決方案會將資料收集到 Azure 監視器記錄，您可在該處使用[查詢語言](../log-query/log-query-overview.md)，或者解決方案中通常會包含的[檢視](view-designer.md)來分析資料。

## <a name="guest-operating-system"></a>客體作業系統
Azure、其他雲端和內部部署中的計算資源具有要監視的客體作業系統。 若安裝了一或多個代理程式，您就可以將客體遙測收集到與 Azure 服務本身相同的監視工具中。

![Azure 計算資源集合](media/data-sources/compute-resource-collection.png)

### <a name="azure-diagnostic-extension"></a>Azure 診斷擴充功能
Azure 診斷擴充功能可藉由從 Azure 計算資源的用戶端作業系統收集記錄和效能資料，來提供基本的監視層級。   

### <a name="log-analytics-agent"></a>Log Analytics 代理程式
Log Analytics 代理程式隨附用於 Windows 虛擬機器、Linux 虛擬機器或實體電腦的完整監視及管理功能。 虛擬機器可以在 Azure、另一個雲端或內部部署環境中執行，而代理程式會直接連線到 Azure 監視器，或者透過 System Center Operations Manager 連線；此外，還可讓您從所設定的[資料來源](agent-data-sources.md)或[監視解決方案](../insights/solutions.md)收集資料，後者會針對在虛擬機器上執行的應用程式提供額外深入解析。

### <a name="dependency-agent"></a>相依性代理程式
[服務對應](../insights/service-map.md)和[適用於 VM 的 Azure 監視器](../insights/vminsights-overview.md)需要 Windows 和 Linux 虛擬機器上的 Dependency Agent。 這會與 Log Analytics 代理程式整合，以收集虛擬機器上所執行程序的相關探索資料，以及外部程序相依性。 Dependency Agent 會將此資料儲存在 Azure 監視器中，並以視覺化方式呈現探索到的互連元件。  

若要進一步了解代理程式間的差異，以及適用您監視需求的是哪一種代理程式，請參閱[監視代理程式概觀](agents-overview.md)。

## <a name="applications"></a>[應用程式]
除了您的應用程式會寫入到客體作業系統的遙測之外，系統會透過 [Application Insights](https://docs.microsoft.com/azure/application-insights/) 來完成詳細應用程式監視。 Application Insights 可以從各種平台上所執行的應用程式收集資料。 應用程式可以在 Azure、其他雲端或內部部署中執行。

![應用程式資料收集](media/data-sources/application-collection.png)


### <a name="application-data"></a>應用程式資料
當您藉由安裝檢測套件，針對應用程式啟用 Application Insights 時，它會收集與應用程式效能及作業相關的計量和記錄。 其中包含頁面檢視、應用程式要求和例外狀況的詳細資訊。 Application Insights 會將收集到的資料儲存在 Azure 監視器中。 它包含廣泛的工具，可用於分析此資料，但是您也可以使用例如計量分析和記錄分析等工具，利用來自其他來源的資料進行分析。

您也可以使用 Application Insights 來[建立自訂計量](../../azure-monitor/app/api-custom-events-metrics.md)。  這樣可讓您定義自己的邏輯，以便計算數值，然後用可從 [計量瀏覽器] 存取的其他計量儲存該值，並且用於[自動調整](../../azure-monitor/platform/autoscale-custom-metric.md)和計量警示。

### <a name="dependencies"></a>相依性
若要監視應用程式的不同邏輯作業，您必須[跨多個元件收集遙測](../../azure-monitor/app/transaction-diagnostics.md)。 Application Insights 支援[分散式遙測相互關聯](../../azure-monitor/app/correlation.md)，這個功能可識別元件之間的相依性，讓您對它們一起進行分析。

### <a name="availability-tests"></a>可用性集合
Application Insights 中的[可用性測試](../../azure-monitor/app/monitor-web-app-availability.md)可讓您從公用網際網路上的不同位置，測試應用程式的可用性和回應能力。 您可以執行簡單的 ping 測試來確認應用程式是否運作，或使用 Visual Studio 來建立 Web 測試，以模擬使用者案例。  可用性測試不需要在應用程式中進行任何檢測。

## <a name="custom-sources"></a>自訂來源
除了應用程式的標準層，您可能需要監視其他資源，這些資源具有無法與其他資料來源一起收集的遙測。 針對這些資源，您需要使用 Azure 監視器 API 來寫入此資料。

![自訂資料收集](media/data-sources/custom-collection.png)

### <a name="data-collector-api"></a>資料收集器 API
Azure 監視器可以使用[資料收集器 API](data-collector-api.md)，從任何 REST 用戶端收集記錄資料。 這可讓您建立自訂監視案例，並且將監視延伸到不會透過其他來源公開遙測的資源。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure 監視器所收集的監視資料類型](data-collection.md)，以及如何檢視及分析這項資料。
