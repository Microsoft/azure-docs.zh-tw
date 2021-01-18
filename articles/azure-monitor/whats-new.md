---
title: Azure 監視器文件的最新動向
description: 每月更新的 Azure 監視器文件重大更新。
ms.subservice: ''
ms.topic: overview
author: bwren
ms.author: bwren
ms.date: 01/11/2021
ms.openlocfilehash: 6347bd3710a4727b9d3f820072ffff21cb57752d
ms.sourcegitcommit: 3af12dc5b0b3833acb5d591d0d5a398c926919c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2021
ms.locfileid: "98071862"
---
# <a name="whats-new-in-azure-monitor-documentation"></a>Azure 監視器文件的最新動向

本文提供新增或已大幅更新的 Azure 監視器文章清單。 該清單會在每個月的第一週重新整理，以納入上個月的文章更新。

## <a name="december-2020"></a>2020 年 12 月

### <a name="general"></a>一般
- [Azure 監視器客戶自控金鑰](platform/customer-managed-keys.md) - 新增了錯誤訊息。
- [與 Azure 監視器整合的合作夥伴](platform/partners.md) - 在事件中樞整合上新增了區段。

### <a name="agents"></a>代理程式
- [使用 Azure 監視器跨資源查詢 Azure 資料總管](platform/azure-monitor-data-explorer-proxy.md) - 新文章。
- [Azure 監視代理程式概觀](platform/agents-overview.md) - 新增了 Oracle 8 支援。

### <a name="alerts"></a>警示
- [針對 Azure 計量警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新增了動態閾值的疑難排解。
- [Log Analytics 中的 IT 服務管理連接器](platform/itsmc-definition.md) - 新文章。
- [IT 服務管理連接器概觀](platform/itsmc-overview.md) - 已重構疑難排解資訊。
- [將 Cherwell 與 IT 服務管理連接器連線](platform/itsmc-connections-cherwell.md) - 新文章。
- [將 Provance 與 IT 服務管理連接器連線](platform/itsmc-connections-provance.md) - 新文章。
- [將 SCSM 與 IT 服務管理連接器連線](platform/itsmc-connections-scsm.md) - 新文章。
- [將 ServiceNow 與 IT 服務管理連接器連線](platform/itsmc-connections-servicenow.md) - 新文章。
- [如何手動修正 ServiceNow 同步問題](platform/itsmc-resync-servicenow.md) - 已重構疑難排解資訊。




### <a name="application-insights"></a>Application Insights
- [適用於 JavaScript Web 應用程式的 Azure Application Insights](app/javascript.md) - 新增了連接字串設定。
- [Azure Application Insights 標準計量](app/standard-metrics.md) - 新文章。
- [Azure 監視器 Application Insights Java](app/java-in-process-agent.md) - 從您的應用程式傳送自訂遙測的其他資訊。
- [從 Application Insights 連續匯出遙測](app/export-telemetry.md) - 新增了以診斷設定為基礎的匯出。
- [在 Azure Functions 中啟用適用於 .NET 和 .NET Core 應用程式的快照偵錯工具](app/snapshot-debugger-function-app.md) - 新文章。
- [Application Insights 和 Log Analytics 所使用的 IP 位址](app/ip-addresses.md) - 新增了 Azure Government 的 IP 位址。
- [針對 Azure Application Insights Profiler 問題進行疑難排解](app/profiler-troubleshooting.md) - 在診斷服務網站擴充功能的狀態頁面上新增了資訊。
- [針對您的 Azure Application Insights 可用性測試進行疑難排解](app/troubleshoot-availability.md) - 針對 ping 測試進行疑難排解的更新。
- [針對適用於 Java 的 Azure 監視器 Application Insights 進行疑難排解](app/java-standalone-troubleshoot.md) - 新文章。

### <a name="containers"></a>容器
- [Azure 監視器中適用於容器的報告](insights/container-insights-reports.md) - 新文章。

### <a name="logs"></a>記錄
- [Azure 監視器記錄專用叢集](log-query/logs-dedicated-clusters.md) - 新增了自動化命令、取消連結和移除的方法，以及疑難排解。
- [Azure 監視器與 Azure 資料總管之間的跨服務查詢 (預覽)](platform/azure-data-explorer-monitor-cross-service-query.md) - 新文章。
- [Azure 監視器中的 Log Analytics 工作區資料匯出 (預覽)](platform/logs-data-export.md) - 新增了 ARM 範本。

### <a name="metrics"></a>計量
- [Azure 計量瀏覽器的進階功能](platform/metrics-charts.md) - 新增了資源範圍選擇器的資訊。
- [在計量瀏覽器中檢視多個資源](platform/metrics-dynamic-scope.md) - 新文章。

### <a name="networks"></a>網路
- [Azure 監視器中的 Azure 網路分析解決方案](insights/azure-networking-analytics.md) - 新增了 Network Insights 活頁簿的資訊。

### <a name="virtual-machines"></a>虛擬機器
- [針對混合式環境啟用 Azure 監視器](insights/vminsights-enable-hybrid.md) - 新版的相依性代理程式。


### <a name="visualizations"></a>視覺效果
- [Azure 監視器活頁簿地圖視覺效果](platform/workbooks-map-visualizations.md) - 新文章。
- [Azure 監視器活頁簿自備儲存體](platform/workbooks-bring-your-own-storage.md) - 新文章。


## <a name="november-2020"></a>2020 年 11 月

### <a name="general"></a>一般
- [Azure 監視器服務限制](service-limits.md) - 已更新並支援 Azure Arc。

### <a name="agents"></a>代理程式
- [Azure 監視代理程式概觀](platform/agents-overview.md) - 已更新並支援 Azure Arc。
- [安裝 Azure 監視器代理程式](platform/azure-monitor-agent-install.md) - 新文章。
- [Azure 監視器代理程式概觀](platform/azure-monitor-agent-overview.md) - 已更新並支援 Azure Arc。
- [適用於代理程式的 Resource Manager 範本範例](samples/resource-manager-agent.md) - 已更新並支援 Azure Arc。

### <a name="alerts"></a>警示
- [在 Azure 入口網站中建立和管理動作群組](platform/action-groups.md) - 已新增 Webhook 的來源 IP 位址。

### <a name="application-insights"></a>Application Insights
- [Java 無程式碼應用程式監視 Azure 監視器 Application Insights](app/java-in-process-agent.md) - 已新增組態範例。
- [適用於 Application Insights JavaScript SDK 的 React 外掛程式](app/javascript-react-plugin.md) - 已新增使用 React 勾點一節。
- [從 Application Insights Java 2.x SDK 升級](app/java-standalone-upgrade-from-2x.md) - 新文章。
- [Microsoft.ApplicationInsights.SnapshotCollector 的版本資訊](app/snapshot-collector-release-notes.md) - 新文章。

### <a name="autoscale"></a>Autoscale
- [開始使用 Azure 中的自動調整](platform/autoscale-get-started.md) - 新增有關如何將自動調整移至不同區域的區段。

### <a name="data-collection"></a>資料集合
- [設定 Azure 監視器代理程式 (預覽) 的資料收集](platform/data-collection-rule-azure-monitor-agent.md) - 已更新並支援 Azure Arc。
- [Azure 監視器 (預覽) 中的資料收集規則](platform/data-collection-rule-overview.md) - 已更新並支援 Azure Arc。
- [資料收集規則的 Resource Manager 範本範例](samples/resource-manager-data-collection-rules.md) - 新文章。

### <a name="insights-and-solutions"></a>見解和解決方案
- [使用安全匯出將 Azure 連線到 ITSM 工具](platform/it-service-management-connector-secure-webhook-connections.md) - 新增連線到 ServiceNow 一節。

### <a name="logs"></a>記錄
- [整合 Log Analytics 與 Excel](log-query/log-excel.md) - 新文章。
- [Log Analytics 資料安全性](platform/data-security.md) - 已新增其他安全性功能的章節。
- [Log Analytics 與 Power BI 整合](log-query/log-powerbi.md) - 新文章。
- [Azure 監視器記錄檔中的標準資料行](platform/log-standard-columns.md) - 已新增 _SubscriptionId 資料行。

來自重新建構記錄查詢內容的新文章和更新文章。

- [Log Analytics 教學課程](log-query/log-analytics-tutorial.md)
- [Azure 監視器中的記錄查詢](log-query/log-query-overview.md)
- [Azure 監視器中的 Log Analytics 概觀](log-query/log-analytics-overview.md)
- [Azure 資料總管和 Azure 監視器的查詢範例](/azure/data-explorer/kusto/query/samples?pivots=azuremonitor)
- [教學課程：在 Azure 資料總管和 Azure 監視器中使用 Kusto 查詢](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor)



### <a name="virtual-machines"></a>虛擬機器

- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md) - 新增支援區域。

適用於 VM 的 Azure 監視器來賓健康情況的新文章 (預覽)

- [適用於 VM 的 Azure 監視器來賓健康情況 (預覽)](insights/vminsights-health-overview.md)
- [適用於 VM 的 Azure 監視器來賓健康情況警示 (預覽)](insights/vminsights-health-alerts.md)
- [在適用於 VM 的 Azure 監視器設定來賓健康情況監視 (預覽)](insights/vminsights-health-configure.md)
- [使用資料收集規則在適用於 VM 的 Azure 監視器設定來賓健康情況監視 (預覽)](insights/vminsights-health-configure-dcr.md)
- [啟用適用於 VM 的 Azure 監視器來賓健康情況 (預覽)](insights/vminsights-health-enable.md)
- [針對適用於 VM 的 Azure 監視器來賓健康情況進行疑難排解 (預覽)](insights/vminsights-health-troubleshoot.md)





## <a name="october-2020"></a>2020 年 10 月

### <a name="general"></a>一般
- [Azure 監視器 API 淘汰](platform/operationalinsights-api-retirement.md) - 新文章。

### <a name="agents"></a>代理程式
- [Azure 監視器監視的項目為何](monitor-reference.md) - 新增有關代理程式的區段。

### <a name="alerts"></a>警示
- [在 Azure 入口網站中建立和管理動作群組](platform/action-groups.md) - 新增有關服務標籤的區段。
- [計量警示的 Resource Manager 範本範例](samples/resource-manager-alerts-metric.md) - 新增內容比對參數和測試位置。
- [針對 Azure 計量警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新增規則設定的最佳做法。

### <a name="application-insights"></a>Application Insights
- [適用於 Application Insights JavaScript SDK 的 Angular 外掛程式](app/javascript-angular-plugin.md) - 新文章。
- [適用於 ASP.NET Core 應用程式的 Azure Application Insights](app/asp-net-core.md) - 新增關於 ILogger 記錄的常見問題集。
- [使用 Azure Application Insights 設定 ASP.NET 的監視功能](app/asp-net.md) - 已重新編寫文章。
- [Azure Application Insights 中記錄型和預先彙總的計量](app/pre-aggregated-metrics-log-metrics.md) - 新增包含預先彙總計量的資料表。
- [監視任何網站的可用性和回應性](app/monitor-web-app-availability.md) - 新增有關區域母體標籤的區段。
- [監視任何位置的 Java 應用程式 - Azure 監視器 Application Insights](app/java-standalone-config.md) - 新增設定範例。
- [監視任何位置的 Java 應用程式 - Azure 監視器 Application Insights](app/java-standalone-telemetry-processors.md) - 新文章。
- [在 Azure 監視器中使用應用程式變更分析來尋找 Web 應用程式問題](app/change-analysis.md) - 新增有關虛擬機器和活動記錄的區段。
  
### <a name="autoscale"></a>Autoscale
- [開始使用 Azure 中的自動調整](platform/autoscale-get-started.md) - 新增有關如何將自動調整移至不同區域的區段。

### <a name="containers"></a>容器
- [使用適用於容器的 Azure 監視器來設定 PV 監視](insights/container-insights-persistent-volumes.md) - 新文章。
- [如何管理適用於容器代理程式的 Azure 監視器](insights/container-insights-manage-agent.md) - 新增已啟用 Azure Arc 的 Kubernetes 叢集支援。
- [Azure 監視器中適用於容器的計量警示](insights/container-insights-metric-alerts.md) - 新增已啟用 Azure Arc 的 Kubernetes 叢集支援。

### <a name="insights-and-solutions"></a>見解和解決方案
- [IT 服務管理連接器 - Azure 監視器中的安全匯出](platform/it-service-management-connector-secure-webhook-connections.md) - 新增有關 ServiceNow 的區段。

### <a name="logs"></a>記錄
- [使用邏輯應用程式將資料從 Log Analytics 工作區封存至 Azure 儲存體](platform/logs-export-logic-app.md) - 新文章。
- [Azure 監視器中的 Log Analytics 工作區資料匯出 (預覽)](platform/logs-data-export.md) - 新增事件中樞 REST 要求的範例主體。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 新增 Azure 監視器記錄與 Azure 資訊安全中心計費之間的關聯性資訊。 新增節點計數的查詢 (如果使用「每個節點」定價層)。 
- [在 Azure 監視器中監視 Log Analytics 工作區的健康情況](platform/monitor-workspace.md) - 新文章。
- [使用 Azure 資料總管來查詢 Azure 監視器中的資料 (預覽)](platform/azure-data-explorer-monitor-proxy.md) - 新文章。
- [使用 Azure 資料總管來查詢從 Azure 監視器匯出的資料 (預覽)](platform/azure-data-explorer-query-storage.md) - 新文章。

### <a name="networks"></a>網路
- [Azure 網路監視器預覽版](insights/network-insights-overview.md) - 新增疑難排解區段。 新增有關連線的區段。

### <a name="platform-logs"></a>平台記錄
- [Azure 活動記錄事件結構描述](platform/activity-log-schema.md) - 新增嚴重性層級的描述。

### <a name="virtual-machines"></a>虛擬機器
- [適用於 VM 的 Azure 監視器中的變更分析](insights/vminsights-change-analysis.md) - 新文章。
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md) - 新增支援區域。
- [如何更新適用於容器的 Azure 監視器以取得計量](insights/container-insights-update-metrics.md) - 新增已啟用 Azure Arc 的 Kubernetes 叢集支援。



## <a name="september-2020"></a>2020 年 9 月

### <a name="general"></a>一般
- [Azure 監視器常見問題集](faq.md) - 新增了 OpenTelemetry 相關章節。

### <a name="agents"></a>代理程式
- [Azure 監視器代理程式概觀](platform/azure-monitor-agent-overview.md) - 新增了切換至新代理程式的決策因素。
- [Azure 監視代理程式概觀](platform/agents-overview.md) - 新增了 Windows 10 的支援。

### <a name="alerts"></a>警示
- [使用 Azure Resource Manager 範本建立記錄警示](platform/alerts-log-create-templates.md) - 新文章。
- [針對 Azure 計量警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新增了針對計量警示規則匯出 ARM 範本相關章節。

### <a name="application-insights"></a>Application Insights
- [建立新的 Azure 監視器 Application Insights 以工作區為基礎的資源](app/create-workspace-resource.md) - 移除了預覽指定。
- [Azure Application Insights 中的資料保留和儲存](app/data-retention-privacy.md) - 針對新支援的 Mac 和 Linux 資料遺失保護新增了詳細資訊。
- [Application Insights 中的事件計數器](app/eventcounters.md) - 在預設收集的計數器上新增了附註。
- [Azure Application Insights 中記錄型和預先彙總的計量](app/pre-aggregated-metrics-log-metrics.md) - 移除了預覽指定。
- [將 Azure 監視器 Application Insights 傳統資源遷移至以工作區為基礎的資源](app/convert-classic-resource.md) - 新文章。
- [監視任何環境上的 Java 應用程式 - Azure 監視器 Application Insights](app/java-in-process-agent.md) - 已針對代理程式的新預覽版更新。
- [使用 Azure Application Insights 設定 ASP.NET 的 Web 應用程式分析](app/asp-net.md) - 已重新編寫文章。
- [Azure Application Insights 中的遙測通道](app/telemetry-channels.md) - 針對新支援的 Mac 和 Linux 資料遺失保護新增了詳細資訊。
- [針對 Azure Application Insights 快照偵錯工具進行疑難排解](app/snapshot-debugger-troubleshoot.md) - 新增了快照偵錯工具疑難排解的 SSL 章節。
- [在 Azure 監視器中使用應用程式變更分析來尋找 Web 應用程式問題](app/change-analysis.md) - 新增了虛擬機器和活動記錄。


### <a name="containers"></a>容器
- [使用適用於容器的 Azure 監視器來設定已啟用 Azure Arc 的 Kubernetes 叢集](insights/container-insights-enable-arc-enabled-clusters.md) - 新增了使用服務主體來啟用監視的指引。
- [適用於容器的 Azure 監視器的部署和 HPA 計量](insights/container-insights-deployment-hpa-metrics.md) - 新文章。

### <a name="insights-and-solutions"></a>見解和解決方案
- [適用於 Azure Cache for Redis 的 Azure 監視器](insights/redis-cache-insights-overview.md) - 移除了預覽指定。
- [Azure 網路監視器 (預覽)](insights/network-insights-overview.md) -新增了連線和流量章節。
- [IT 服務管理連接器 - Azure 監視器中的安全匯出](platform/it-service-management-connector-secure-webhook-connections.md) - 新文章。
- [Azure 監視器中的 IT 服務管理連接器](platform/itsmc-connections.md) - Cherwell 和 Provance ITSM 整合的相關附註。
- [使用適用於 Key Vault 的 Azure 監視器來監視 Key Vault](insights/key-vault-insights-overview.md) - 移除了預覽指定。

### <a name="logs"></a>記錄
- [稽核 Azure 監視器記錄查詢中的查詢](log-query/query-audit.md) - 新文章。
- [Azure 監視器客戶自控金鑰](platform/customer-managed-keys.md) - 新增了客戶加密箱。
- [Azure 監視器記錄專用叢集](log-query/logs-dedicated-clusters.md) - 新文章。
- [設計您的 Azure 監視器記錄部署](platform/design-logs-deployment.md) - 更新了縮放和擷取量速率限制章節。
- [Azure 監視器 Log Analytics 中的記錄查詢範圍](log-query/scope.md) - 更新為包含以工作區為基礎的應用程式。
- [Azure 監視器中的記錄](platform/data-platform-logs.md) -更新為包含以工作區為基礎的應用程式。
- [Azure 監視器記錄檔記錄中的標準資料行](platform/log-standard-columns.md) - 更新為包含以工作區為基礎的應用程式。
- [Azure 監視器服務限制](service-limits.md) - 更新了使用者查詢節流的限制。
- [在 Azure 監視器 Log Analytics 中使用客戶管理的儲存體帳戶](platform/private-storage.md) - 已重新編寫文章。
- [在 Azure Log Analytics 中檢視和分析資料](./platform/data-platform-logs.md) - 更新為包含以工作區為基礎的應用程式。


### <a name="platform-logs"></a>平台記錄
- [Azure 活動記錄事件結構描述 - Azure 監視器](platform/activity-log-schema.md) 新增了嚴重性層級。
- [診斷設定的 Resource Manager 範本範例](samples/resource-manager-diagnostic-settings.md) - 新增了 Azure 儲存體帳戶的範例。

### <a name="visualizations"></a>視覺效果
- [Azure 監視器活頁簿圖表視覺效果](platform/workbooks-chart-visualizations.md) - 新文章。
- [Azure 監視器活頁簿複合橫條轉譯器](platform/workbooks-composite-bar.md) - 新文章。
- [Azure 監視器活頁簿圖形視覺效果](platform/workbooks-graph-visualizations.md) - 新文章。
- [Azure 監視器活頁簿格線視覺效果](platform/workbooks-grid-visualizations.md) - 新文章。
- [Azure 監視器活頁簿蜂巢視覺效果](platform/workbooks-honey-comb.md) - 新文章。
- [Azure 監視器活頁簿文字視覺效果](platform/workbooks-text-visualizations.md) - 新文章。
- [Azure 監視器活頁簿並排顯示視覺效果](platform/workbooks-tile-visualizations.md) - 新文章。
- [Azure 監視器活頁簿樹狀視覺效果](platform/workbooks-tree-visualizations.md) - 新文章。




## <a name="august-2020"></a>2020 年 8 月

### <a name="general"></a>一般

- [Azure 監視器所監視的內容](monitor-reference.md) - 已更新為包含 Azure 監視器代理程式。


### <a name="agents"></a>代理程式
- [Azure 監視器代理程式概觀](platform/azure-monitor-agent-overview.md) - 新文章。
- [針對混合式環境啟用 Azure 監視器](insights/vminsights-enable-hybrid.md) - 已更新相依性代理程式版本。
- [Azure 監視代理程式概觀](platform/agents-overview.md) - 已新增 Azure 監視器代理程式並合併 OS 支援資料表。


#### <a name="new-and-updated-articles-from-restructure-of-agent-content"></a>來自重新建構代理程式內容的新文章和更新文章
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md)
- [在 Linux 電腦上安裝 Log Analytics 代理程式](platform/agent-linux.md)
- [在 Windows 電腦上安裝 Log Analytics 代理程式](platform/agent-windows.md)
- [Log Analytics 代理程式概觀](platform/log-analytics-agent.md)

### <a name="application-insights"></a>Application Insights
- [Azure Application Insights for JavaScript Web 應用程式](app/javascript.md) - 已新增一節，釐清用戶端伺服器相互關聯，以及 CORS 相互關聯的組態。
- [建立新的 Azure 監視器 Application Insights 工作區型資源](app/create-workspace-resource.md) - 已新增工作區型應用程式所提供的功能。
- [Application Insights 和 Log Analytics 所使用的 IP 位址](app/ip-addresses.md) - 已更新即時計量資料流的 IP 位址。
- [監視任何環境上的 Java 應用程式 - Azure 監視器 Application Insights](app/java-in-process-agent.md) - 已為支援的自訂遙測新增資料表。
- [適用於 Application Insights JavaScript SDK 的 Native React 外掛程式](app/javascript-react-native-plugin.md) - 新文章。
- [適用於 Application Insights JavaScript SDK 的 React 外掛程式](app/javascript-react-plugin.md) - 新文章。
- [建立 Azure 函式應用程式與 Application Insights 資源監視功能的 Resource Manager 範本範例](samples/resource-manager-function-app.md) - 新文章。
- [建立 Azure App Service Web 應用程式與 Application Insights 資源監視功能的 Resource Manager 範本範例](samples/resource-manager-web-app.md) - 新文章。
- [使用 Azure Application Insights 進行使用量分析](app/usage-overview.md) - 新增的影片。

### <a name="autoscale"></a>Autoscale
- [開始使用 Azure 中的自動調整](platform/autoscale-get-started.md) - 已新增一節，說明如何路由傳送至 App Service 的狀況良好執行個體。

### <a name="data-collection"></a>資料集合
- [設定 Azure 監視器代理程式 (預覽) 的資料收集](platform/data-collection-rule-azure-monitor-agent.md) - 新文章。
- [Azure 監視器 (預覽) 中的資料收集規則](platform/data-collection-rule-overview.md) - 新文章。


### <a name="containers"></a>容器
- [適用於容器的 Azure 監視器的部署和 HPA 計量](insights/container-insights-deployment-hpa-metrics.md) - 新文章。

### <a name="insights"></a>深入解析
- [Azure 監視器中的監視解決方案](insights/solutions.md) - 已針對新 UI 進行更新。
- [Azure 中的網路效能監視器解決方案](insights/network-performance-monitor.md) - 已新增支援的工作區區域。


### <a name="logs"></a>記錄
- [Azure 監視器常見問題集](faq.md) - 已新增從工作區刪除資料的項目。 已在 502 和 503 回應上新增項目。
  - [設計您的 Azure 監視器記錄部署](platform/design-logs-deployment.md) - [擷取量速率限制] 區段的更新。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 已將使用量查詢更新為更有效率的查詢格式。
- [最佳化 Azure 監視器中的記錄查詢](log-query/query-optimization.md) - 已將特定值新增至效能指標。
- [診斷設定適用的 Resource Manager 範本範例](samples/resource-manager-diagnostic-settings.md) - 已新增記錄查詢稽核記錄的範例。


### <a name="platform-logs"></a>平台記錄
- [建立診斷設定以將平台記錄和計量傳送至不同目的地](platform/diagnostic-settings.md) - 已新增診斷設定的區域需求。

### <a name="visualizations"></a>視覺效果
- [Azure 監視器活頁簿概觀](platform/workbooks-overview.md) - 新增的影片。
- [將 Azure 活頁簿範本移至另一個區域](platform/workbook-templates-move-region.md) - 新文章。
- [將 Azure 活頁簿移至另一個區域](platform/workbooks-move-region.md) - 新文章。



## <a name="july-2020"></a>2020 年 7 月

### <a name="general"></a>一般
- [部署 Azure 監視器](deploy-scale.md) - 重新建構適用於 VM 的 Azure 監視器上線內容。
- [使用 Azure 私人連結將網路安全地連線到 Azure 監視器](platform/private-link-security.md) - 新增了關於限制的章節。

### <a name="alerts"></a>警示
- [Azure 監視器警示的動作規則](platform/alerts-action-rules.md) - 新增了 CLI 程序。
- [在 Azure 入口網站中建立和管理動作群組](platform/action-groups.md) - 有所更新以反映 UI 中的變更。
- [Azure 監視器 Log Analytics 中的範例查詢](log-query/example-queries.md) - 新文章。
- [針對 Azure 監視器中的記錄警示進行疑難排解](platform/alerts-troubleshoot-log.md) - 新增了關於警示規則配額的章節。
- [針對 Azure 計量警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新增了關於尚未發出的自訂計量上警示規則的章節。
- [了解計量警示在 Azure 監視器中的運作方式。](platform/alerts-metric-overview.md) - 新增了彙總細微性的選取建議。

### <a name="application-insights"></a>Application Insights
- [Azure Web 應用程式擴充功能的版本資訊 - Application Insights](app/web-app-extension-release-notes.md) - 新文章。
- [Application Insights 資源的 Resource Manager 範本範例](samples/resource-manager-app-resource.md) - 新文章。
- [針對 Azure Application Insights Profiler 的問題進行疑難排解](app/profiler-troubleshooting.md) - 針對在 Azure App Service 上執行 ASP.NET Core 應用程式分析工具時所發生的錯誤新增了附註。 

### <a name="containers"></a>容器
- [Azure 監視器針對容器所發出的記錄警示](insights/container-insights-log-alerts.md) - 新文章。
- [Azure 監視器針對容器所發出的計量警示](insights/container-insights-metric-alerts.md) - 新文章。

### <a name="logs"></a>記錄
- [Azure 監視器客戶自控金鑰](platform/customer-managed-keys.md) - 新增了錯誤訊息和查詢的區段 CMK 設定。
- [Azure 監視器 HTTP 資料收集器 API](platform/data-collector-api.md) - 新增了 Python 3 範例。
- [將 Azure 監視器中的記錄查詢最佳化](log-query/query-optimization.md) - 新增了關於在使用子查詢時避免多個資料掃描的章節。
- [教學課程：開始使用 Log Analytics 查詢](./log-query/log-analytics-tutorial.md) - 新增了影片。

### <a name="platform-logs"></a>平台記錄
- [建立診斷設定以將平台記錄和計量傳送至不同目的地](platform/diagnostic-settings.md) - 新增了影片。
- [Azure 監視器的 Resource Manager 範本範例](samples/resource-manager-samples.md) - 新增了使用記錄目的地類型的 ARM 範例。 

### <a name="solutions"></a>方案
- [Azure 監視器中的監視解決方案](insights/solutions.md) - 新增了 CLI 程序。
- [Azure 中的 Office 365 管理解決方案](insights/solution-office-365.md) - 變更了停用日期。

### <a name="virtual-machines"></a>虛擬機器

來自重新建構「適用於 VM 的 Azure 監視器」內容的新文章和更新文章

- [什麼是適用於 VM 的 Azure 監視器？](insights/vminsights-overview.md)
- [針對適用於 VM 的 Azure 監視器來設定 Log Analytics 工作區](insights/vminsights-configure-workspace.md)
- [將 Linux 電腦連線至 Azure 監視器](platform/agent-linux.md)
- [針對混合式環境啟用 Azure 監視器](insights/vminsights-enable-hybrid.md)
- [針對 Azure 入口網站中的單一虛擬機器或虛擬機器擴展集啟用 Azure 監視器](insights/vminsights-enable-portal.md)
- [使用 Azure 原則啟用適用於 VM 的 Azure 監視器](./insights/vminsights-enable-policy.md)
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md)
- [使用 PowerShell 啟用適用於 VM 的 Azure 監視器](insights/vminsights-enable-powershell.md)
- [使用 Resource Manager 範本啟用適用於 VM 的 Azure 監視器](insights/vminsights-enable-resource-manager.md)
- [使用 PowerShell 或範本啟用適用於 VM 的 Azure 監視器](./insights/vminsights-enable-powershell.md)


### <a name="visualizations"></a>視覺效果
- [升級您的 Log Analytics 儀表板視覺效果](log-query/dashboard-upgrade.md) - 更新了重新整理的頻率。
- [將來自 Azure 監視器的資料視覺化](visualizations.md) - 新增了影片。


## <a name="june-2020"></a>2020 年 6 月

### <a name="general"></a>一般
- [部署 Azure 監視器](deploy-scale.md) - 新文章。
- [Azure 監視器客戶管理的金鑰](platform/customer-managed-keys.md) - 更新的 billingtype 屬性。 已新增 PowerShell 命令。

### <a name="agents"></a>代理程式
- [Log Analytics 代理程式概觀](platform/log-analytics-agent.md) - 已新增 Python 2 需求。

### <a name="alerts"></a>警示
- [如何在警示規則或動作規則的目標資源移至不同 Azure 區域時進行更新](platform/alerts-resource-move.md) - 新文章。
- [針對 Azure 計量警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新文章。
- [針對 Azure 監視器中的記錄警示進行疑難排解](platform/alerts-troubleshoot-metric.md) - 新文章。
  
### <a name="application-insights"></a>Application Insights
- [適用於 JavaScript Web 應用程式的 Azure Application Insights](app/javascript.md) - 更新至 JavaScript SDK 區段。 更新程式碼片段以報告載入失敗。
- [針對 Profiler 和快照偵錯工具設定 BYOS (自備儲存體)](app/profiler-bring-your-own-storage.md) - 新文章。
- [使用 OpenCensus Python 進行 Azure Application Insights 中的傳入要求追蹤](app/opencensus-python-request.md) - 更新 OpenCensus 的記錄和設定。
- [使用 Azure Application Insights 監視即時的 ASP.NET Web 應用程式](app/monitor-performance-live-website-now.md) - 更新狀態監視器 v1 的淘汰日期。
- [使用 Azure Application Insights 監視 Node.js 服務](app/nodejs.md) - 多個更新，包括從舊版本進行遷移和 SDK 設定
- [使用 Azure 監視器監視 Python 應用程式 (預覽)](app/opencensus-python.md) - 新增設定 Azure 監視器匯出工具的區段。
- [監視您的應用程式而不需要變更程式碼 - 自動檢測 Azure 監視器 Application Insights](app/codeless-overview.md) - 新文章。
- [針對 JavaScript Web 應用程式的 SDK 載入失敗問題進行疑難排解](app/javascript-sdk-load-failure.md) - 新文章。

### <a name="containers"></a>容器
- [如何停止監視混合式 Kubernetes 叢集](insights/container-insights-optout-hybrid.md) - 已針對啟用 Arc 的 Kubernetes 新增區段。
- [使用適用於容器的 Azure 監視器來設定啟用 Azure Arc 的 Kubernetes 叢集](insights/container-insights-enable-arc-enabled-clusters.md) - 新文章。
- [使用適用於容器的 Azure 監視器設定 Azure Red Hat OpenShift v4.x](insights/container-insights-azure-redhat4-setup.md) - 更新必要條件。
- [為適用於容器的 Azure 監視器設定即時資料 (Live Data) (預覽)](insights/container-insights-livedata-setup.md) - 已針對 Azure 美國政府中未提供的功能移除相關注意事項。

### <a name="insights"></a>深入解析
- [常見問題 - Azure 中的網路效能監控解決方案](insights/network-performance-monitor-faq.md) - 新增 ExpressRoute 監視器的常見問題。

### <a name="logs"></a>記錄
- [刪除和復原 Azure Log Analytics 工作區](platform/delete-workspace.md) - 已新增 PowerShell 命令。 已更新疑難排解。
- [在 Azure 監視器中管理 Log Analytics 工作區](platform/manage-access.md) - 針對 Azure RBAC 區段中不允許的資料表新增範例。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 資料大小計算的其他詳細資訊。 已更新資料量警示的設定。 Azure Sentinel 所收集的安全性資料詳細資訊。 資料上限的說明。
- [搭配 Azure Logic Apps 和 Power Automate 使用 Azure 監視器記錄](platform/logicapp-flow-connector.md) - 新增連接器限制。

### <a name="metrics"></a>計量
- [Azure 監視器支援的資源類型計量](platform/metrics-supported.md) - 已更新 SQL Server 計量。


### <a name="platform-logs"></a>平台記錄

- [診斷設定的 Resource Manager 範本範例](samples/resource-manager-diagnostic-settings.md) - 修正活動記錄診斷設定。
- [使用 Azure 入口網站將 Azure 活動記錄傳送至 Log Analytics 工作區](learn/quick-collect-activity-log-portal.md) - 新文章
- [使用 Azure Resource Manager 範本將 Azure 活動記錄傳送至 Log Analytics 工作區](learn/quick-collect-activity-log-arm.md) - 新文章。

平台記錄內容重構和彙總的最新和更新文章

- [將 Azure 資源記錄封存到儲存體帳戶](./platform/resource-logs.md#send-to-azure-storage)
- [Azure 活動記錄事件結構描述](platform/activity-log-schema.md)
- [Azure 活動記錄](platform/activity-log.md)
- [Azure 監視器 CLI 範例](samples/cli-samples.md)
- [Azure 監視器 PowerShell 範例](samples/powershell-samples.md)
- [Azure 監視 REST API 逐步解說](platform/rest-api-walkthrough.md)
- [Azure Resource 記錄支援的服務和結構描述](./platform/resource-logs-schema.md)
- [Azure 資源記錄](platform/resource-logs.md)
- [收集並分析 Azure 監視器中的 Azure 活動記錄](./platform/activity-log.md)
- [在 Log Analytics 工作區中收集 Azure 資源記錄](./platform/resource-logs.md#send-to-log-analytics-workspace)
- [建立診斷設定以將平台記錄和計量傳送至不同目的地](platform/diagnostic-settings.md)
- [匯出 Azure 活動記錄](./platform/activity-log.md#legacy-collection-methods)
- [Azure 平台記錄概觀](platform/platform-logs-overview.md)
- [將 Azure 平台記錄串流至事件中樞](./platform/resource-logs.md#send-to-azure-event-hubs)
- [在 Azure 監視器中檢視 Azure 活動記錄](./platform/activity-log.md#view-the-activity-log)

### <a name="virtual-machines"></a>虛擬機器
- [在 Azure 入口網站中啟用適用於 VM 的 Azure 監視器](./insights/vminsights-enable-portal.md) - 已更新為包含 Azure Arc。
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md) - 已更新為包含 Azure Arc。
- [什麼是適用於 VM 的 Azure 監視器？](insights/vminsights-overview.md) - 已更新為包含 Azure Arc。


### <a name="visualizations"></a>視覺效果
- [Azure 監視器活頁簿資料來源](platform/workbooks-data-sources.md) - 新增警示和自訂端點區段。
- [針對 Azure 監視器的活頁簿型深入解析進行疑難排解](insights/troubleshoot-workbooks.md) - 新文章。
- [升級您的 Log Analytics 儀表板視覺效果](log-query/dashboard-upgrade.md) - 新文章。



## <a name="may-2020"></a>2020 年 5 月

### <a name="general"></a>一般

- [Azure 監視器常見問題集](faq.md) - 新增計量的區段。
- [Azure 監視器客戶管理的金鑰](platform/customer-managed-keys.md) - 準備正式發行的各種變更。
- [Azure 監視器的內建原則定義](./samples/policy-reference.md) - 新文章。
- [適用於記錄內嵌之客戶擁有的儲存體帳戶](platform/private-storage.md) - 新文章。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 新增叢集的比例計費。
- [使用 Azure 私人連結將網路安全地連線到 Azure 監視器](platform/private-link-security.md) - 新文章。


#### <a name="new-resource-manager-template-samples"></a>新的 Resource Manager 範本範例 
- [Azure 監視器的 Resource Manager 範本範例](samples/resource-manager-samples.md)
- [動作群組的 Resource Manager 範本範例](samples/resource-manager-action-groups.md)
- [代理程式的 Resource Manager 範本範例](samples/resource-manager-agent.md)
- [適用於容器之 Azure 監視器的 Resource Manager 範本範例](samples/resource-manager-container-insights.md)
- [適用於 VM 之 Azure 監視器的 Resource Manager 範本範例](samples/resource-manager-vminsights.md)
- [診斷設定的 Resource Manager 範本範例](samples/resource-manager-diagnostic-settings.md)
- [Log Analytics 工作區的 Resource Manager 範本範例](samples/resource-manager-workspace.md)
- [記錄查詢的 Resource Manager 範本範例](samples/resource-manager-log-queries.md)
- [記錄查詢警示規則的 Resource Manager 範本範例](samples/resource-manager-alerts-log.md)
- [度量警示規則的 Resource Manager 範本範例](samples/resource-manager-alerts-metric.md)
- [活頁簿的 Resource Manager 範本範例](samples/resource-manager-workbooks.md)

### <a name="agents"></a>代理程式
- [安裝和設定 Windows Azure 診斷擴充功能 (WAD)](platform/diagnostics-extension-windows-install.md) - 新增設定診斷的詳細資料。
- [Log Analytics 代理程式概觀](platform/log-analytics-agent.md) - 新增支援的 Linux 版本。

### <a name="application-insights"></a>Application Insights

- [藉由 Application Insights 監視在 Azure Functions 上執行的應用程式 - Azure 監視器](app/monitor-functions.md) - 新文章。
- [藉由 Azure Application Insights 監視 Node.js 服務](app/nodejs.md) - 一般更新，包含從舊版遷移的新章節。
- [Application Insights 和 Log Analytics 所使用的 IP 位址](app/ip-addresses.md) - 新增 webhook 和美國政府的 IP 位址。
- [藉由 Application Insights 在 Azure Kubernetes Service (AKS) 監視器應用程式 - Azure 監視器](app/kubernetes-codeless.md) - 新文章。
- [沒有要進行疑難排解的資料 - 適用於 .NET 的 Application Insights](app/asp-net-troubleshoot-no-data.md) - 新增藉由 dotnet 追蹤的收集記錄的章節。
- [在 Azure 監視器中使用應用程式變更分析來尋找 web 應用程式問題](app/change-analysis.md) - 變更分析功能中的多項更新。

#### <a name="new-and-updated-articles-for-preview-of-workspace-based-applications"></a>適用於以工作區為基礎之應用程式預覽的新增和更新文章
- [Azure 監視器 Application Insights 以工作區為基礎的資源結構描述](app/apm-tables.md)
- [建立新的 Azure 監視器 Application Insights 以工作區為基礎的資源](app/create-workspace-resource.md)
- [Azure 監視器記錄檔查詢中的 app() 運算式](log-query/app-expression.md)
- [Azure 監視器 Log Analytics 中的記錄查詢範圍](log-query/scope.md)
- [藉由 Azure 監視器進行跨資源查詢](log-query/cross-workspace-query.md)
- [Azure 監視器記錄檔記錄中的標準屬性](./platform/log-standard-columns.md)
- [Azure 監視器記錄的結構](./platform/data-platform-logs.md)





### <a name="containers"></a>容器
- [如何啟用容器的 Azure 監視器](insights/container-insights-onboard.md) - 更新的防火牆設定資料表。
- [如何針對計量的容器更新 Azure 監視器](insights/container-insights-update-metrics.md) - 更新以使用受控識別來收集計量。
- [適用於容器之 Azure 監視器的監視成本](insights/container-insights-cost.md) - 新文章。
- [設定適用於容器之 Azure 監視器的即時資料 (預覽)](insights/container-insights-livedata-setup.md) -支援新的叢集角色繫結。

### <a name="insights"></a>深入解析
- [適用於 Azure Cache for Redis 的 Azure 監視器 (預覽)](insights/redis-cache-insights-overview.md) - 新文章。
- [藉由適用於 Key Vault 的 Azure 監視器來監視 Key Vault (預覽)](./insights/key-vault-insights-overview.md) - 新文章。

### <a name="logs"></a>記錄
- [藉由 PowerShell 建立並設定 Log Analytics](platform/powershell-workspace-configuration.md) - 新增疑難排解一節。
- [在 Azure 入口網站建立 Log Analytics 工作區](learn/quick-create-workspace.md) - 新增疑難排解一節。
- [使用 Azure CLI](learn/quick-create-workspace-cli.md) 新增的疑難排解區段來建立 Log Analytics 工作區。
- [刪除並復原 Azure Log Analytics 工作區](platform/delete-workspace.md) - 更新復原已刪除工作區的相關資訊。
- [Azure 監視器記錄檔查詢中的函式](log-query/functions.md) - 移除不包含其他函式之函式的相關注意事項。
- [Azure 監視器記錄的結構](./platform/data-platform-logs.md) - 釐清 Application Insights 資料表的屬性描述。
- [搭配 Azure Logic Apps 和 Power Automate 使用 Azure 監視器記錄](platform/logicapp-flow-connector.md) - 新增限制一節。
- [使用 PowerShell 建立和設定 Log Analytics 工作區](platform/powershell-workspace-configuration.md) - 新增疑難排解一節。


### <a name="metrics"></a>計量
- [依資源類型區分 Azure 監視器支援的計量](platform/metrics-supported.md) - 釐清客體計量和計量路由。 

### <a name="solutions"></a>方案
- [藉由 Azure 監視器最佳化您的 Active Directory 環境](insights/ad-assessment.md) - 將 Windows Server 2019 新增至支援的版本。
- [藉由 Azure 監視器最佳化您的 SQL Server 環境](insights/sql-assessment.md) - 新增至支援的 SQL Server 版本。


### <a name="virtual-machines"></a>虛擬機器
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md) - 新增至支援的 Ubuntu Server 版本。 新增 Log Analytics 工作區支援的區域。
- [藉由適用於 VM 之 Azure 監視器將效能以圖表方式呈現](insights/vminsights-performance.md) - 新增無法使用的計量的限制一節。

### <a name="visualizations"></a>視覺效果
- [Azure 監視器活頁簿和 Azure Resource Manager 的範本](platform/workbooks-automate.md) - 新增部署活頁簿範本的 Resource Manager 更新。
- [Azure 監視器活頁簿群組](platform/workbooks-groups.md) - 新文章。
- [Azure 監視器活頁簿 - 藉由 JSONPath 轉換 JSON 資料](platform/workbooks-jsonpath.md) - 新文章。


## <a name="april-2020"></a>2020 年 4 月

### <a name="general"></a>一般

- [Azure 監視器客戶受控金鑰](platform/customer-managed-keys.md) - 非同步作業的新增區段
- [在 Azure 監視器中管理 Log Analytics 工作區](platform/manage-access.md) - 更新的自訂記錄區段。

### <a name="alerts"></a>警示

- [Azure 監視器警示的動作規則](platform/alerts-action-rules.md) - 新增的影片。
- [Azure 中的警示與通知監視概觀](platform/alerts-overview.md) - 新增的影片。

### <a name="application-insights"></a>Application Insights

- [Azure Application Insights 中的應用程式對應](app/app-map.md) - 針對 JAVA 代理程式新增的雲端角色名稱設定。
- [Azure Application Insights .NET 代理程式 API 參考](app/status-monitor-v2-api-reference.md) - 匯總 API 參考。
- [Application Insights 和 Log Analytics 使用的 IP 位址](app/ip-addresses.md) - App Insights 和 Log Analytics API、動作群組 Webhook、Azure 美國政府的更新 IP 位址。
- [隨時隨地監視 JAVA 應用程式](app/java-standalone-config.md) - 新文章。
- [在任何環境中監視 JAVA 應用程式](app/java-in-process-agent.md) - 新文章。
- [監視在任何環境中執行的 JAVA 應用程式](app/java-standalone-arguments.md) - 新文章。
- [監視內部部署執行的 JAVA 應用程式](app/java-on-premises.md) - 新文章。
- [移除 Visual Studio 中的 Application Insights](app/remove-application-insights.md) - 新文章。
- [Azure Application Insights 中的遙測取樣](app/sampling.md) - 修正 Python 中的固定取樣率。

### <a name="containers"></a>容器

- [設定適用於容器之 Azure 監視器的 Azure Red Hat OpenShift v4](insights/container-insights-azure-redhat4-setup.md) - 新文章。
- [如何手動修正 ServiceNow 同步問題](platform/itsmc-resync-servicenow.md) - 新文章。
- [如何停止監視您的 Azure 和 Red Hat OpenShift v4 叢集](insights/container-insights-optout-openshift-v4.md) - 新文章。
- [如何停止監視您的 Azure Red Hat OpenShift v3 叢集](insights/container-insights-optout-openshift-v3.md) - 新文章。
- [如何停止監視混合式 Kubernetes 叢集](insights/container-insights-optout-hybrid.md) - 新文章。

### <a name="insights"></a>深入解析

- [使用適用於Key Vaults 的 Azure 監視器來監視 Azure Key Vault (預覽)](insights/key-vault-insights-overview.md) - 新文章

### <a name="logs"></a>記錄

- [Azure 監視器服務限制](service-limits.md) - 新增使用者查詢節流。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 新增記錄叢集的計費。 已新增 Kusto 查詢，讓客戶能夠使用舊版的每個節點定價層來判斷是否應該移至每 GB 或容量保留層。

### <a name="metrics"></a>計量

- [Azure 計量瀏覽器的進階功能](platform/metrics-charts.md) - 新增彙總區段。

### <a name="workbooks"></a>活頁簿

- [Azure 監視器活頁簿和 Azure Resource Manager 範本](platform/workbooks-automate.md) - 新增 Resource Manager 範本以部署活頁簿範本。

## <a name="march-2020"></a>2020 年 3 月

### <a name="general"></a>一般

- [Azure 監視器概觀](overview.md) - 新增 Azure 監視器概觀影片。
- [Azure 監視器客戶管理的金鑰組態](platform/customer-managed-keys.md) - 一般更新。
- [Azure 監視器資料參考](/azure/azure-monitor/reference/) - 新增網站。

### <a name="alerts"></a>警示

- [在 Azure 監視器中建立、檢視和管理活動記錄警示](platform/alerts-activity-log.md) - Resource Manager 範本的其他說明。
- [了解計量警示在 Azure 監視器中的運作方式。](platform/alerts-metric-overview.md) - 已更新以提供政府支援。
- [Azure 監視器警示和通知的疑難排解](platform/alerts-troubleshoot.md) - 新文章。

### <a name="application-insights"></a>Application Insights

- [使用 PowerShell 將 Azure Application Insights 自動化](app/powershell.md) - 新增了 ARMClient 範例。
- [從 Application Insights 連續匯出遙測](app/export-telemetry.md) - 新增包含匯出結構詳細資料的資料表。
- [在 Azure App Service 中啟用 .NET 應用程式的快照偵錯工具](app/snapshot-debugger-appservice.md) - 新增了 Resource Manager 範本範例。
- [管理 Azure Application Insights 的使用量和成本](app/pricing.md) - 新增了資料容量警示的資訊。
- [使用 Azure 監視器 (預覽) 來監視 Python 應用程式](app/opencensus-python.md) - 新增了標準計量。
- [JavaScript 應用程式的來源對應支援 - Azure 監視器 Application Insights](app/source-map-support.md) - 新文章。

### <a name="containers"></a>容器

- [Azure 監視器常見問題集](faq.md) - 適用於容器的 Azure 監視器更新。
- [使用適用於容器的 Azure 監視器來設定 GPU 監視](insights/container-insights-gpu-monitoring.md) - 新文章。

### <a name="insights"></a>深入解析

- [Azure 中的 Office 365 管理解決方案](insights/solution-office-365.md) - 淘汰日期更新。

### <a name="logs"></a>記錄

- [將 Azure 監視器中的記錄查詢最佳化](log-query/query-optimization.md) - 新增了 XML 和 JSON 剖析的 CPU 條件。
- [刪除和復原 Azure Log Analytics 工作區](platform/delete-workspace.md) - 新增了疑難排解方法。
- [搭配 Azure Logic Apps 和 Power Automate 使用 Azure 監視器記錄](platform/logicapp-flow-connector.md) - 針對新的 Azure 監視器連接器更新。

### <a name="metrics"></a>計量

- [Azure 入口網站中的磁碟計量淘汰](platform/portal-disk-metrics-deprecation.md) - 新文章。
- [教學課程 - 在 Azure 監視器中建立計量圖表](learn/tutorial-metrics-explorer.md) - 新增了影片。

### <a name="platform-logs"></a>平台記錄

- [在 Azure 監視器中收集並分析 Azure 活動記錄](./platform/activity-log.md) - 內容重寫，更完整的說明如何使用診斷設定收集活動記錄。

### <a name="virtual-machines"></a>虛擬機器

- [使用 Azure 監視器監視 Azure 虛擬機器](insights/monitor-vm-azure.md) - 新文章。
- [快速入門：使用 Azure 監視器來監視 Azure 虛擬機器](learn/quick-monitor-azure-vm.md) - 更新內容以新增適用於 VM 的 Azure 監視器。
- [來自適用於 VM 的 Azure 監視器的警示](insights/vminsights-alerts.md) - 新文章。
- [啟用適用於 VM 的 Azure 監視器概觀](insights/vminsights-enable-overview.md) - 更新了代理程式下載連結。

適用於 VM 的 Azure 監視器正式發行的一般更新

- [什麼是適用於 VM 的 Azure 監視器？](insights/vminsights-overview.md)
- [適用於 VM 的 Azure 監視器 (GA) 常見問題集](insights/vminsights-ga-release-faq.md) 
- [使用 Azure 原則啟用適用於 VM 的 Azure 監視器](./insights/vminsights-enable-policy.md) 
- [如何使用適用於 VM 的 Azure 監視器來繪製效能圖表](insights/vminsights-performance.md)
- [如何從適用於 VM 的 Azure 監視器查詢記錄](insights/vminsights-log-search.md)
- [使用適用於 VM 的 Azure 監視器檢視應用程式相依性](insights/vminsights-maps.md) 

### <a name="visualizations"></a>視覺效果

- [從 Azure 監視器將資料視覺化](visualizations.md) - 更新說明檢視設計工具的淘汰規劃。

## <a name="february-2020"></a>2020 年 2 月

### <a name="agents"></a>代理程式

診斷擴充功能內容改寫多次更新。

- [Azure 監視代理程式概觀](platform/agents-overview.md) - 調整過的資料表，以更清楚地說明每個代理程式的獨特功能。
- [Azure 診斷擴充功能概觀](platform/diagnostics-extension-overview.md) - 完成改寫。
- [在 Azure 監視器中，對 IIS 使用 Blob 儲存體並對事件使用資料表儲存體](platform/diagnostics-extension-logs.md) - 一般改寫以更新和釐清資訊。
- [安裝和設定 Windows Azure 診斷擴充功能 (WAD)](platform/diagnostics-extension-windows-install.md) - 新增文章。 
- [Windows 診斷擴充功能結構描述](platform/diagnostics-extension-schema-windows.md) - 重新整理。
- [將資料從 Windows Azure 診斷擴充功能傳送至 Azure 事件中樞](platform/diagnostics-extension-stream-event-hubs.md) - 完全改重寫並更新。
- [儲存和檢視 Azure 儲存體中的診斷資料](../cloud-services/diagnostics-extension-to-storage.md) - 完全改重寫並更新。
- [適用於 Windows 的 Log Analytics 虛擬機器擴充功能](../virtual-machines/extensions/oms-windows.md) - 更清楚說明與 Log Analytics 代理程式的關係。
- [適用於 Linux 的 Azure 監視器虛擬機器擴充功能](../virtual-machines/extensions/oms-linux.md) - 更清楚說明與 Log Analytics 代理程式的關係。

### <a name="application-insights"></a>Application Insights

- [Azure Application Insights 中的連接字串](app/sdk-connection-string.md) - 新文章。

### <a name="insights-and-solutions"></a>見解和解決方案

#### <a name="azure-monitor-for-containers"></a>適用於容器的 Azure 監視器

- [整合 Azure Active Directory 與 Azure Kubernetes Service](../aks/azure-ad-integration-cli.md) - 新增的備註，用來建立用戶端應用程式以支援已啟用 Kubernetes RBAC 的叢集，進而支援容器的 Azure 監視器。

#### <a name="azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器

- [適用於 VM 的 Azure 監視器 (GA) 常見問題集](insights/vminsights-ga-release-faq.md) - 儲存效能資料的方式有所變更。

#### <a name="office-365"></a>Office 365

- [Azure 中的 Office 365 管理解決方案](insights/solution-office-365.md) - 淘汰日期更新。


### <a name="logs"></a>記錄

- [將 Azure 監視器中的記錄查詢最佳化](log-query/query-optimization.md) - 新文章。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 改善的範例查詢，協助您了解使用量。

### <a name="metrics"></a>計量

- [可透過診斷設定匯出 Azure 監視器平台計量](platform/metrics-supported-export-diagnostic-settings.md) - 新增了 Null 和零值的行為變更一節。

### <a name="visualizations"></a>視覺效果

活頁簿轉換指南中有關檢視設計工具的多篇新文章。

- [將 Azure 監視器檢視表設計師轉換成活頁簿的指南](platform/view-designer-conversion-overview.md) - 新文章。
- [將 Azure 監視器檢視表設計師轉換成活頁簿的指南](platform/view-designer-conversion-options.md) - 新文章。
- [將 Azure 監視器檢視表設計師轉換成活頁簿圖格](platform/view-designer-conversion-tiles.md) - 新文章。
- [將 Azure 監視器檢視表設計師轉換成活頁簿的摘要和存取](platform/view-designer-conversion-access.md) - 新文章。
- [將 Azure 監視器檢視表設計師轉換成活頁簿的常見工作](platform/view-designer-conversion-tasks.md) - 新文章。
- [將 Azure 監視器檢視表設計師轉換成活頁簿的常見工作](platform/view-designer-conversion-examples.md) - 新文章。

## <a name="january-2020"></a>2020 年 1 月

### <a name="general"></a>一般

- [Azure 監視器監視的項目為何？](monitor-reference.md) - 新文章。

### <a name="agents"></a>代理程式

- [使用 Azure Log Analytics 代理程式收集記錄資料](platform/log-analytics-agent.md) - 已更新網路防火牆需求表。

### <a name="alerts"></a>警示

- [在 Azure 入口網站中建立和管理動作群組](platform/action-groups.md) - 針對不再需要的 v2 功能移除了設定。
- [使用 Resource Manager 範本建立計量警示](platform/alerts-metric-create-templates.md) - 已新增 *ignoreDataBefore* 參數的範例。  已新增多準則規則相關條件約束。
- [使用 Log Analytics 警示 REST API](platform/api-alerts.md) - 已更正 JSON 範例。

### <a name="application-insights"></a>Application Insights

- [Application Insights 和 Log Analytics 所使用的 IP 位址](app/ip-addresses.md) - 已更新「可用性測試」一節，其中包含如何新增輸入連接埠規則，以允許使用 Azure 網路安全性群組的流量。
- [針對 Azure Application Insights Profiler 的問題進行疑難排解](app/profiler-troubleshooting.md) - 已更新一般疑難排解。
- [Azure Application Insights 中的遙測取樣](app/sampling.md) - 已更新並重新建構，以根據客戶的意見反應改善可讀性。

### <a name="data-security"></a>資料安全性

- [Azure 監視器客戶管理的金鑰設定](platform/customer-managed-keys.md) - 新文章。

### <a name="insights-and-solutions"></a>見解和解決方案

#### <a name="azure-monitor-for-containers"></a>適用於容器的 Azure 監視器

- [設定適用於容器的 Azure 監視器代理程式資料收集](insights/container-insights-agent-config.md) - 已新增在 Azure Red Hat OpenShift 上升級代理程式的詳細資訊，並新增了其他資訊來區分升級代理程式的方法。
- [為適用於容器的 Azure 監視器建立效能警示](./insights/container-insights-log-alerts.md) - 已修訂資訊及更新步驟，以便使用工作區內容警示，對工作區中儲存的效能資料建立警示。
- [使用適用於容器的 Azure 監視器進行 Kubernetes 監視](insights/container-insights-analyze.md) - 已更新有關 Windows Kubernetes 叢集支援的概觀文章和分析文章。
- [使用適用於容器的 Azure 監視器來設定 Azure Red Hat OpenShift 叢集](insights/container-insights-azure-redhat-setup.md) - 已新增在 Azure Red Hat OpenShift 上升級代理程式的詳細資訊，並新增了其他資訊來區分升級代理程式的方法。
- [使用適用於容器的 Azure 監視器來設定混合式 Kubernetes 叢集](insights/container-insights-hybrid-setup.md) - 已更新內容，以反映 Kubelet 的 cAdvisor 新增了安全連接埠 10250 支援。
- [如何管理適用於容器的 Azure 監視器代理程式](insights/container-insights-manage-agent.md) - 相較於其他類型的 Kubernetes 叢集，已更新透過 Azure Red Hat OpenShift 抓取計量的行為和設定相關詳細資料。
- [如何管理適用於容器的 Azure 監視器代理程式](insights/container-insights-prometheus-integration.md) - 已更新透過 Azure Red Hat OpenShift (相較於其他類型的 Kubernetes 叢集) 抓取計量的行為和設定相關詳細資訊。
- [如何針對計量更新適用於容器的 Azure 監視器](insights/container-insights-update-metrics.md) - 相較於其他類型的 Kubernetes 叢集， Azure Red Hat OpenShift 的計量抓取的行為和設定相關的更新詳細資料。

#### <a name="azure-monitor-for-vms"></a>適用於 VM 的 Azure 監視器

- [適用於 VM 的 Azure 監視器 (GA) 常見問題集](insights/vminsights-ga-release-faq.md) - 已新增有關將工作區和代理程式升級至新版本的資訊。

#### <a name="office-365"></a>Office 365

- [Azure 中的 office 365 管理解決方案](insights/solution-office-365.md) - 已新增在 Azure Sentinel 中遷移至 Office 365 解決方案的詳細資訊和常見問題集。 已移除上線區段。

### <a name="logs"></a>記錄

- [在 Azure 監視器中管理 Log Analytics 工作區](platform/manage-access.md)- 更新無法執行的動作。
- [管理 Azure 監視器記錄的使用量和成本](platform/manage-cost-storage.md) - 在「計價模式」一節中新增了資料量計算的詳細說明。
- [使用 Azure Resource Manager 範本建立和設定 Log Analytics 工作區](./samples/resource-manager-workspace.md) - 已使用新定價層來更新範例。

### <a name="platform-logs"></a>平台記錄

- [使用診斷設定收集 Azure 活動記錄 - Azure 監視器](./platform/activity-log.md) - 已變更屬性的其他資訊。
- [匯出 Azure 活動記錄](./platform/activity-log.md#legacy-collection-methods) - 已針對 UI 變更進行更新。 

## <a name="december-2019"></a>2019 年 12 月

### <a name="agents"></a>代理程式

- [將 Linux 電腦連線至 Azure 監視器](platform/agent-linux.md) - 新文章。

### <a name="alerts"></a>警示

- [使用 Resource Manager 範本建立計量警示](platform/alerts-metric-create-templates.md) - 已新增自訂計量的範例。
- [在 Azure 監視器中建立具有動態閾值的警示](platform/alerts-dynamic-thresholds.md) - 已新增有關解讀動態閾值圖表的章節。
- [Azure 中的警示與通知監視概觀](platform/alerts-overview.md) - 已更新 Resource Graph 查詢。
- [Azure 監視器中計量警示支援的資源](platform/alerts-metric-near-real-time.md) - 更新支援的計量和維度。
- [從舊版 Log Analytics 警示 API 切換至新的 Azure 警示 API](platform/alerts-log-api-switch.md) - 新增了已修改警示名稱的附註。
- [了解計量警示在 Azure 監視器中的運作方式。](platform/alerts-metric-overview.md) - 已新增支援大規模監視的資源類型。

### <a name="application-insights"></a>Application Insights

- [適用於背景工作角色服務應用程式 (非 HTTP 應用程式) 的 Application Insights](app/worker-service.md) - 已新增 C# 程式碼的預設記錄層級。 已更新套件參考版本。
- [ApplicationInsights.config 參考 - Azure](app/configuration-with-applicationinsights-config.md)- 已更新範例程式碼。
- [使用 PowerShell 將 Azure Application Insights 自動化](app/powershell.md) - 更新 Resource Manager 範本。
- [建立新的 Azure Application Insights 資源](app/create-new-resource.md) - 新增了全域唯一名稱的附註。
- [使用即時計量資料流進行診斷](app/live-stream.md) - 已更新 ASP.NET Core SDK 版本需求。
- [Application Insights 中的事件計數器](app/eventcounters.md) - 已更新 customMetrics 的類別和資料表。
- [探索 Azure Application Insights 中的 Java 追蹤記錄](app/java-trace-logs.md) - 已新增 Java 代理程式記錄閾值的設定。
- [Application Insights 和 Log Analytics 所使用的 IP 位址](app/ip-addresses.md) - 已更新即時計量資料流的 IP 位址。
- [監視 Azure 應用程式服務效能](app/azure-web-apps.md) - 已新增 ASP.NET Core 3.0 支援。 
- [使用 Azure 監視器來監視 Python 應用程式 (預覽)](app/opencensus-python.md) - 已新增 OpenCensus Python 結構描述對應至 Azure 監視器結構描述的詳細說明。
- [Azure Application Insights 的版本資訊](app/release-notes.md) - 新增了舊版的附註。
- [Azure Application Insights 中的遙測通道](app/telemetry-channels.md) - 已經更在長期中斷連線期間內已捨棄資料的持續時間。
- [Azure Application Insights 中的遙測取樣](app/sampling.md) - 已修正自訂 TelemetryInitializer 的程式碼片段。
- [針對 Java Web 專案中的 Application Insights 進行疑難排解](app/java-troubleshoot.md) - 已移除 JDK 9 中有關不支援相依性集合的陳述。

### <a name="insights-and-solutions"></a>見解和解決方案

- [適用於容器的 Azure 監視器常見問題集](./faq.md) - 已新增有關 [映像] 和 [名稱] 欄位的問題。
- [Azure 監視器中的 Azure SQL 分析解決方案](insights/azure-sql.md) - 已更新資料庫等候受控執行個體支援。
- [設定適用於容器的 Azure 監視器代理程式資料集合](insights/container-insights-agent-config.md) - 已新增 enrich_container_logs 的設定。
- [使用適用於容器的 Azure 監視器來設定混合式 Kubernetes 叢集](insights/container-insights-hybrid-setup.md) - 已新增疑難排解章節。
- [使用 Azure 監視器來監視 Active Directory 複寫狀態](insights/ad-replication-status.md) - 已更新 .NET Framework 必要條件。
- [Azure 中的網路效能監視器解決方案](insights/network-performance-monitor.md)- 已新增支援的區域。
- [使用 Azure 監視器將 Active Directory 環境最佳化](insights/ad-assessment.md) - 已更新 .NET Framework 必要條件。
- [使用 Azure 監視器將 SQL Server 環境最佳化](insights/sql-assessment.md) - 已更新 .NET Framework 必要條件。
- [使用 Azure Log Analytics 將 System Center Operations Manager 環境最佳化](insights/scom-assessment.md) - 已更新 .NET Framework 必要條件。
- [支援與 Azure Log Analytics 中的 IT Service Management Connector 連線](platform/itsmc-connections.md) - 已將 New York 新增至必要用戶端識別碼和用戶端密碼。

### <a name="logs"></a>記錄

- [刪除和復原 Azure Log Analytics 工作區](platform/delete-workspace.md) - 已新增 PowerShell 方法。
- [設計您的 Azure 監視器記錄部署](platform/design-logs-deployment.md) - 已提高工作區的擷取率。

### <a name="metrics"></a>計量

- [可透過診斷設定匯出的 Azure 監視器平台計量](platform/metrics-supported-export-diagnostic-settings.md) - 新文章。

### <a name="platform-logs"></a>平台記錄

根據使用診斷設定來設定活動記錄的新功能，在平台記錄內容的重新建構過程中更新了多篇文章。

- [將 Azure 資源記錄封存到儲存體帳戶](./platform/resource-logs.md#send-to-azure-storage)
- [Azure 活動記錄事件結構描述](platform/activity-log-schema.md)
- [Azure 監視器服務限制](service-limits.md)
- [在 Log Analytics 工作區中收集和分析 Azure 活動記錄](./platform/activity-log.md)
- [使用診斷設定收集 Azure 活動記錄 (預覽) - Azure 監視器](./platform/activity-log.md)
- [跨 Azure 租用戶將 Azure 活動記錄收集到 Log Analytics 工作區中](./platform/activity-log.md)
- [在 Log Analytics 工作區中收集 Azure 資源記錄](./platform/resource-logs.md#send-to-log-analytics-workspace)
- [使用 Resource Manager 範本在 Azure 中建立診斷設定](./samples/resource-manager-diagnostic-settings.md)
- [建立診斷設定以收集 Azure 中的記錄和計量](platform/diagnostic-settings.md)
- [匯出 Azure 活動記錄](./platform/activity-log.md#legacy-collection-methods)
- [Azure 平台記錄概觀](platform/platform-logs-overview.md)
- [將 Azure 監視資料串流至事件中樞](platform/stream-monitoring-data-event-hubs.md)
- [將 Azure 平台記錄串流至事件中樞](./platform/resource-logs.md#send-to-azure-event-hubs)

### <a name="quickstarts-and-tutorials"></a>快速入門與教學課程

- [在 Azure 監視器中建立計量圖表](learn/tutorial-metrics-explorer.md) - 新文章。
- [從 Azure 資源收集資源記錄，並使用 Azure 監視器進行分析](learn/tutorial-resource-logs.md) - 新文章。
- [使用 Azure 監視器來監視 Azure 資源](learn/quick-monitor-azure-resource.md) - 新文章。
   
## <a name="next-steps"></a>後續步驟

- 如果您想要參與 Azure 監視器文件，請參閱[文件參與者指南](/contribute/)。