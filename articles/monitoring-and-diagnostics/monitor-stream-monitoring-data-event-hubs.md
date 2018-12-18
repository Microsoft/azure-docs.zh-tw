---
title: 將 Azure 監視資料串流至事件中樞
description: 了解如何將所有的 Azure 監視資料串流至事件中樞，以將資料匯入夥伴 SIEM 或分析工具。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 8/21/2018
ms.author: johnkem
ms.component: ''
ms.openlocfilehash: 18c0f8176a85eef79000fff8ed717ad7e57f20d8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954835"
---
# <a name="stream-azure-monitoring-data-to-an-event-hub-for-consumption-by-an-external-tool"></a>將 Azure 監視資料串流至事件中樞以供外部工具取用

Azure 監視器有提供單一管道以存取 Azure 環境的所有監視資料，讓您輕鬆設定夥伴 SIEM 和監視工具來取用該資料。 本文會逐步解說如何設定 Azure 環境的不同資料層，使其傳送至單一事件中樞命名空間或事件中樞，再由外部工具收集。

> [!VIDEO https://www.youtube.com/embed/SPHxCgbcvSw]

## <a name="what-data-can-i-send-into-an-event-hub"></a>我可以將哪些資料傳送至事件中樞？ 

Azure 環境內有數個「層級」的監視資料，而存取每一層資料的方法都略有不同。 一般而言，針對各層的描述如下：

- **應用程式監視資料：** 您所撰寫並在 Azure 上執行之程式碼的效能和功能相關資料。 應用程式監視資料的範例包括效能追蹤、應用程式記錄，以及使用者遙測。 應用程式監視資料通常會以下列其中一種方式收集：
  - 透過使用 [Application Insights SDK](../application-insights/app-insights-overview.md) 之類的 SDK 來檢測程式碼。
  - 透過執行會在執行應用程式的電腦上接聽新應用程式記錄的監視代理程式，例如 [Windows Azure 診斷代理程式](./azure-diagnostics.md)或 [Linux Azure 診斷代理程式](../virtual-machines/linux/diagnostic-extension.md)。
- **客體 OS 監視資料：** 有關執行應用程式之作業系統的資料。 客體 OS 監視資料的範例為 Linux syslog 或 Windows 系統事件。 若要收集這類資料，您必須安裝像是 [Windows Azure 診斷代理程式](./azure-diagnostics.md)或 [Linux Azure 診斷代理程式](../virtual-machines/linux/diagnostic-extension.md) 這類的代理程式。
- **Azure 資源監視資料：** 有關 Azure 資源之作業的資料。 針對某些 Azure 資源類型 (例如虛擬機器)，在該 Azure 服務內會有要監視的客體 OS 與應用程式。 針對其他 Azure 資源 (例如網路安全性群組)，資源監視資料是最高層級的可用資料 (因為在那些資源中沒有執行任何客體 OS 或應用程式)。 這類資料可使用[資源診斷設定](./monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)來收集。
- **Azure 訂用帳戶監視資料：** 有關 Azure 訂用帳戶作業和管理的資料，以及有關 Azure 本身健康情況和作業的資料。 [活動記錄](./monitoring-overview-activity-logs.md)包含大部分的訂用帳戶監視資料，例如服務健康情況事件和 Azure Resource Manager 稽核。 您可以使用記錄設定檔收集此資料。
- **Azure 租用戶監視資料：** 租用戶層級 Azure 服務的作業相關資料，例如 Azure Active Directory。 Azure Active Directory 稽核和登入是租用戶監視資料的範例。 您也可以使用租用戶診斷設定來收集此資料。

來自任何層的資料都能傳送至事件中樞，然後再提取至夥伴工具。 以下幾節說明如何設定來自每一層的資料，以串流至事件中樞。 當中的步驟均假設您在要監視的層級皆已具有資產。

## <a name="set-up-an-event-hubs-namespace"></a>設定事件中樞命名空間

在開始之前，您必須[建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。 此命名空間和事件中樞是所有監視資料的目的地。 「事件中樞」命名空間是共用相同存取原則之事件中樞的邏輯群組，非常類似於儲存體帳戶在該儲存體帳戶內有個別的 Blob。 請注意幾個與您所建立之事件中樞命名空間和事件中樞相關的詳細資料：
* 建議您使用「標準事件中樞」命名空間。
* 一般而言，只需要一個輸送量單位。 如果隨著記錄檔使用量的增加，您需要擴大規模，您一律可以稍後手動增加命名空間的輸送量單位數，或是啟用自動擴大。
* 輸送量單位數可讓您擴大事件中樞的輸送量規模。 分割區數可讓您跨眾多客戶平行處理取用量。 單一分割區每秒最多可以處理 20 MBps 或大約 20,000 則訊息。 視取用資料的工具而定，可能支援也可能不知支援從多個分割區取用。 如果您不確定要設定多少個分割區，建議您從 4 個分割區開始。
* 建議您將事件中樞上的訊息保留期設定為 7 天。 如果您的取用工具停止運作的時間超過一天，這可確保該工具能夠在其停止的地方接著執行 (最長可達 7 天的事件)。
* 建議您使用事件中樞的預設取用者群組。 您不需要建立其他取用者群組或使用個別的取用者群組，除非您打算讓兩個不同的工具從相同的事件中樞取用相同的資料。
* 就「Azure 活動記錄」而言，您需挑選「事件中樞」命名空間，而「Azure 監視器」就會在該命名空間內建立一個名為 'insights-logs-operationallogs' 的事件中樞。 就其他記錄類型而言，您可以選擇現有的事件中樞 (可讓您重複使用相同的 insights-logs-operationallogs 事件中樞)，或讓「Azure 監視器」為每個記錄分類建立一個事件中樞。
* 一般而言，必須在從事件中樞取用資料的電腦上開啟連接埠 5671 和 5672。

另請參閱 [Azure 事件中樞常見問題集](../event-hubs/event-hubs-faq.md)。

## <a name="azure-tenant-monitoring-data"></a>Azure 租用戶監視資料

Azure 租用戶監視資料目前只適用於 Azure Active Directory。 您可以使用 [Azure Active Directory 報告](../active-directory/reports-monitoring/overview-reports.md)中的資料，其中包含在特定租用戶內所進行登入活動的歷程記錄，和所做變更的稽核記錄。

### <a name="azure-active-directory-data"></a>Azure Active Directory 資料

若要將資料從 Azure Active Directory 記錄傳送到事件中樞命名空間，您要設定 AAD 租用戶上的租用戶診斷設定。 [遵循本指南](../active-directory/reports-monitoring/quickstart-azure-monitor-stream-logs-to-event-hub.md)來設定租用戶診斷設定。

## <a name="azure-subscription-monitoring-data"></a>Azure 訂用帳戶監視資料

Azure 訂用帳戶監視資料可用於 [Azure 活動記錄](./monitoring-overview-activity-logs.md)。 這包含來自 Resource Manager 的建立、更新及刪除作業、可能會影響訂用帳戶中資源的 [Azure 服務健康情況](../service-health/service-health-overview.md)變更、[資源健康情況](../service-health/resource-health-overview.md)在狀態上的轉換，以及數種其他類型的訂用帳戶層級事件。 [本文詳細說明 Azure 活動記錄中所出現事件的所有類別](./monitoring-activity-log-schema.md)。

### <a name="activity-log-data"></a>活動記錄資料

若要將 Azure 活動記錄的資料傳送至事件中樞命名空間，您必須在訂用帳戶上設定記錄設定檔。 [遵循此指南](./monitoring-stream-activity-logs-event-hubs.md)以在訂用帳戶上設定記錄設定檔。 請為每個要監視的訂用帳戶執行這項操作一次。

> [!TIP]
> 記錄設定檔目前僅允許您選取事件中樞命名空間，其中會以 'insights-operational-logs' 的名稱建立事件中樞。 目前還不能在記錄設定檔中自行指定事件中樞名稱。

## <a name="azure-resource-metrics-and-diagnostics-logs"></a>Azure 資源計量和診斷記錄

Azure 資源會發出兩種監視資料：
1. [資源診斷記錄](./monitoring-overview-of-diagnostic-logs.md)
2. [計量](monitoring-overview-metrics.md)

這兩種資料都會使用資源診斷設定來傳送至事件中樞。 [遵循此指南](./monitoring-stream-diagnostic-logs-to-event-hubs.md)以在特定資源上設定資源診斷設定。 為每個要收集記錄的資源進行資源診斷設定。

> [!TIP]
> 您可以使用 Azure 原則以透過[在原則規則中使用 DeployIfNotExists 效果](../governance/policy/concepts/definition-structure.md#policy-rule)，來確保特定範圍內的每個資源一律會搭配診斷設定進行設定。

## <a name="guest-os-data"></a>客體 OS 資料

您必須安裝代理程式，才能將客體 OS 監視資料傳送至事件中樞。 不論是使用 Windows 或 Linux，您都要在設定檔中指定想傳送至事件中樞的資料，以及要傳送資料的目標事件中樞，並將該設定檔傳遞至 VM 上執行的代理程式。

### <a name="linux-data"></a>Linux 資料

[Linux Azure 診斷代理程式](../virtual-machines/extensions/diagnostics-linux.md)可用來將 Linux 電腦的監視資料傳送至事件中樞。 若要執行此動作，請在 LAD 設定檔保護的設定 JSON 中，將事件中樞新增為接收裝置。 [請參閱此文章以深入了解將事件中樞接收新增至 Linux Azure 診斷代理程式](../virtual-machines/extensions/diagnostics-linux.md#protected-settings)。

> [!NOTE]
> 您無法在入口網站中進行將客體 OS 監視資料串流至事件中樞的設定。 相反地，您必須手動編輯設定檔。

### <a name="windows-data"></a>Windows 資料

[Windows Azure 診斷代理程式](./azure-diagnostics.md)可用來將 Windows 電腦的監視資料傳送至事件中樞。 若要執行此動作，請在 WAD 設定檔的 privateConfig 區段中，將事件中樞新增為接收裝置。 [請參閱此文章以深入了解如何將事件中樞接收裝置新增至 Windows Azure 診斷代理程式](./azure-diagnostics-streaming-event-hubs.md)。

> [!NOTE]
> 您無法在入口網站中進行將客體 OS 監視資料串流至事件中樞的設定。 相反地，您必須手動編輯設定檔。

## <a name="application-monitoring-data"></a>應用程式監視資料

應用程式監視資料要求您的程式碼需搭配 SDK 進行檢測，因此沒有一般的解決方案可將應用程式監視資料路由至 Azure 中的事件中樞。 不過，[Azure Application Insights](../application-insights/app-insights-overview.md) 是可以用來收集 Azure 應用程式層級資料的其中一個服務。 如果您是使用 Application Insights，便執行下列動作來將監視資料串流至事件中樞：

1. 針對儲存體帳戶[設定連續匯出](../application-insights/app-insights-export-telemetry.md) Application Insights 資料。

2. 設定以計時器觸發的邏輯應用程式，使它能[從 Blob 儲存體提取資料](../connectors/connectors-create-api-azureblobstorage.md#add-action)，並[以訊息的形式將它推送至事件中樞](../connectors/connectors-create-api-azure-event-hubs.md#add-action)。

## <a name="what-can-i-do-with-the-monitoring-data-being-sent-to-my-event-hub"></a>傳送至事件中樞的監視資料有何功用？

使用 Azure 監視器將監視資料路由至事件中樞，可讓您輕鬆與夥伴 SIEM 和監視工具整合。 大部分工具皆需要事件中樞連接字串和 Azure 訂用帳戶的特定權限，才能讀取事件中樞的資料。 可與 Azure 監視器整合的部分工具清單如下：

* **IBM QRadar** - Microsoft Azure DSM 與 Microsoft Azure 事件中樞通訊協定均可從 [IBM 支援網站](http://www.ibm.com/support) \(英文\) 下載。 您可以[從這裡深入了解與 Azure 的整合](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/c_dsm_guide_microsoft_azure_overview.html?cp=SS42VS_7.3.0) \(英文\)。
* **Splunk** -根據您的 Splunk 安裝程式而定，共有兩種方法：
    1. [適用於 Splunk 的 Azure 監視器附加元件](https://splunkbase.splunk.com/app/3534/) \(英文\) 可從 Splunkbase 和開放原始碼專案中取得。 [文件在此](https://github.com/Microsoft/AzureMonitorAddonForSplunk/wiki/Azure-Monitor-Addon-For-Splunk) \(英文\)。
    2. 如果您無法在 Splunk 執行個體中安裝附加元件 (例如， 如果使用 Proxy 或在 Splunk Cloud 上執行)，您可以使用[由事件中樞中的新訊息觸發的這個功能](https://github.com/Microsoft/AzureFunctionforSplunkVS)，將這些事件轉寄給 Splunk HTTP Event Collector。
* **SumoLogic** - 設定 SumoLogic 以從事件中樞取用資料的指示[可從這裡取得](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure-Audit/02Collect-Logs-for-Azure-Audit-from-Event-Hub) \(英文\)
* **Syslog 伺服器** - 若要直接將 Azure 監視器資料串流到 syslog 伺服器，您可以查看[此 github 存放庫](https://github.com/miguelangelopereira/azuremonitor2syslog/) \(英文\)。

## <a name="next-steps"></a>後續步驟
* [將活動記錄檔封存至儲存體帳戶](monitoring-archive-activity-log.md)
* [閱讀 Azure 活動記錄檔的概觀](monitoring-overview-activity-logs.md)
* [根據活動記錄檔事件設定警示](insights-auditlog-to-webhook-email.md)

