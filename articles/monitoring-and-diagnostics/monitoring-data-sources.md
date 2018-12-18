---
title: 監視 Azure 資料的來源
description: 了解目前 Azure 所提供的所有監視資料來源。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 06/12/2018
ms.author: johnkem
ms.component: ''
ms.openlocfilehash: 55dc43f238b487392be48047b3d608f84e1e3ee2
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579305"
---
# <a name="consume-monitoring-data-from-azure"></a>取用來自 Azure 的監視資料

我們正在利用 Azure 監視器管線，將橫跨整個 Azure 平台的所有監視資料集中在單一位置，但我們也承認該管線目前尚未提供所有監視資料。 在本文中，我們將摘要說明您可以從 Azure 服務以程式設計方式存取監視資料的各種方式。

## <a name="options-for-data-consumption"></a>資料取用的選項

| 資料類型 | 類別 | 支援的服務 | 存取方法 |
| --- | --- | --- | --- |
| Azure 監視器平台層級計量 | 度量 | [請參閱此處的清單](monitoring-supported-metrics.md) | <ul><li>**REST API：**[Azure 監視器計量 API (英文)](https://docs.microsoft.com/rest/api/monitor/metrics)</li><li>**儲存體 Blob 或事件中樞：**[診斷設定](monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)</li></ul> |
| 計算客體 OS 計量 (例如 效能計數器) | 度量 | [Windows](../virtual-machines-dotnet-diagnostics.md) 和 Linux 虛擬機器 (v2)、[雲端服務](../cloud-services/cloud-services-dotnet-diagnostics-trace-flow.md)、[Service Fabric](../service-fabric/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md) | <ul><li>**儲存體資料表或 Blob：**[Windows 或 Linux Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics-storage.md)</li><li>**事件中樞：**[Windows Azure 診斷](../event-hubs/event-hubs-streaming-azure-diags-data.md)</li></ul> |
| 自訂或應用程式計量 | 度量 | 使用 Application Insights 檢測的任何應用程式 | <ul><li>**REST API：**[Application Insights REST API (英文)](https://dev.applicationinsights.io/reference)</li></ul> |
| 儲存體度量 | 度量 | Azure 儲存體 | <ul><li>**儲存體資料表：**[儲存體分析 (英文)](https://docs.microsoft.com/rest/api/storageservices/storage-analytics)</li></ul> |
| 帳單資料 | 度量 | 所有 Azure 服務 | <ul><li>**REST API：** [Azure 資源使用情況與 RateCard API](../billing/billing-usage-rate-card-overview.md)</li></ul> |
| 活動記錄檔 | 活動 | 所有 Azure 服務 | <ul><li>**REST API：**[Azure 監視器事件 API (英文)](https://docs.microsoft.com/rest/api/monitor/eventcategories)</li><li>**儲存體 Blob 或事件中樞︰**[記錄設定檔](monitoring-overview-activity-logs.md#export-the-activity-log-with-a-log-profile)</li></ul> |
| Azure 監視器診斷記錄檔 | 活動 | [請參閱此處的清單](monitoring-diagnostic-logs-schema.md) | <ul><li>**儲存體 Blob 或事件中樞：**[診斷設定](monitoring-overview-of-diagnostic-logs.md#diagnostic-settings)</li></ul> |
| 計算客體 OS 記錄檔 (例如 IIS、ETW、syslog) | 活動 | [Windows](../virtual-machines-dotnet-diagnostics.md) 和 Linux 虛擬機器 (v2)、[雲端服務](../cloud-services/cloud-services-dotnet-diagnostics-trace-flow.md)、[Service Fabric](../service-fabric/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md) | <ul><li>**儲存體資料表或 Blob：**[Windows 或 Linux Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics-storage.md)</li><li>**事件中樞：**[Windows Azure 診斷](../event-hubs/event-hubs-streaming-azure-diags-data.md)</li></ul> |
| App Service 記錄檔 | 活動 | 應用程式服務 | <ul><li>**檔案、資料表或 Blob 儲存體：**[Web 應用程式診斷](../app-service/web-sites-enable-diagnostic-log.md)</li></ul> |
| 儲存體記錄 | 活動 | Azure 儲存體 | <ul><li>**儲存體資料表：**[儲存體分析 (英文)](https://docs.microsoft.com/rest/api/storageservices/storage-analytics)</li></ul> |
| 資訊安全中心警示 | 活動 | Azure 資訊安全中心 | <ul><li>**REST API：**[安全性警示 (英文)](https://msdn.microsoft.com/library/mt704050.aspx)</li></ul> |
| Active Directory 報告 | 活動 | Azure Active Directory | <ul><li>**REST API：**[Azure Active Directory 圖形 API](../active-directory/active-directory-reporting-api-getting-started.md)</li></ul> |
| 資訊安全中心資源狀態 | 狀態 | [所有支援的資源](https://msdn.microsoft.com/library/mt704041.aspx#Anchor_1) | <ul><li>**REST API：**[安全性狀態 (英文)](https://msdn.microsoft.com/library/mt704041.aspx)</li></ul> |
| 資源健全狀況 | 狀態 | 支援的服務 | <ul><li>**REST API：**[資源健康狀態 REST API (英文)](https://azure.microsoft.com/blog/reduce-troubleshooting-time-with-azure-resource-health/)</li></ul> |
| Azure 監視器計量警示 | 通知 | [請參閱此處的清單](monitoring-supported-metrics.md) | <ul><li>**Webhook：**[Azure 計量警示](insights-webhooks-alerts.md)</li></ul> |
| Azure 監視器活動記錄警示 | 通知 | 所有 Azure 服務 | <ul><li>**Webhook：** Azure 活動記錄警示</li></ul> |
| 自動調整規模通知 | 通知 | [請參閱此處的清單](monitoring-overview-autoscale.md#supported-services-for-autoscale) | <ul><li>**Webhook：**[自動調整規模通知 Webhook 承載結構描述](insights-autoscale-to-webhook-email.md#autoscale-notification-webhook-payload-schema)</li></ul> |
| 記錄搜尋查詢警示 | 通知 | Log Analytics | <ul><li>**Webhook：**[Webhook 動作記錄警示規則](../monitoring-and-diagnostics/monitor-alerts-unified-log-webhook.md)</li></ul> |
| Application Insights 計量警示 | 通知 | Application Insights | <ul><li>**Webhook：**[Application Insights 警示](../application-insights/app-insights-alerts.md)</li></ul> |
| Application Insights Web 測試 | 通知 | Application Insights | <ul><li>**Webhook：**[Application Insights 警示](../application-insights/app-insights-alerts.md)</li></ul> |

## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure 監視器計量](monitoring-overview-metrics.md)
- 深入了解 [Azure 活動記錄](monitoring-overview-activity-logs.md)
- 深入了解 [Azure 診斷記錄檔](monitoring-overview-of-diagnostic-logs.md)
