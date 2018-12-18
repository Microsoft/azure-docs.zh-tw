---
title: Azure 診斷記錄支援的服務和結構描述
description: 了解 Azure 診斷記錄支援的服務和事件結構描述。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 8/21/2018
ms.author: johnkem
ms.component: logs
ms.openlocfilehash: 06d9fda01a89340eb019b4900c02e321e0b73cf5
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818959"
---
# <a name="supported-services-schemas-and-categories-for-azure-diagnostic-logs"></a>Azure 診斷記錄支援的服務、結構描述和類別

[Azure 監視器診斷記錄](monitoring-overview-of-diagnostic-logs.md)是 Azure 服務所發出的記錄，會描述這些服務或資源的作業。 透過 Azure 監視器提供的所有診斷記錄都會共用通用最上層結構描述，並且具有每個服務發出其專屬事件之唯一屬性的彈性。

資源類型 (適用於 `resourceId` 屬性) 與 `category` 的組合可唯一識別結構描述。 本文描述每個服務結構描述之診斷記錄和連結的最上層結構描述。

## <a name="top-level-diagnostic-logs-schema"></a>最上層診斷記錄結構描述

| 名稱 | 必要/選用 | 說明 |
|---|---|---|
| 分析 | 必要 | 事件的時間戳記 (UTC)。 |
| ResourceId | 必要 | 發出事件之資源的資源識別碼。 對於租用戶服務，這是 /tenants/tenant-id/providers/provider-name 的格式。 |
| tenantId | 租用戶記錄所需的 | 此事件所繫結 Active Directory 租用戶的租用戶識別碼。 這個屬性只能用於租用戶層級記錄，並不會出現在資源層級記錄中。 |
| operationName | 必要 | 此事件所代表的作業名稱。 如果事件代表 RBAC 作業，則這是 RBAC 作業名稱 (例如 Microsoft.Storage/storageAccounts/BlobServices/Blobs/Read)。 通常以 Resource Manager 作業形式建模，即使它們不是實際記載的 Resource Manager 作業也是一樣 (`Microsoft.<providerName>/<resourceType>/<subtype>/<Write/Read/Delete/Action>`) |
| operationVersion | 選用 | 與作業建立關聯的 api-version，如果使用 API 執行 operationName (例如 `http://myservice.windowsazure.net/object?api-version=2016-06-01`)。 如果沒有對應至此作業的 API，則版本代表該作業的版本，以防與作業建立關聯的屬性在未來變更。 |
| category | 必要 | 事件的記錄類別。 類別是您可在特定資源上啟用或停用記錄的資料粒度。 事件之屬性 Blob 內顯示的屬性，會與特定記錄類別和資源類型內的屬性相同。 典型記錄類別是「稽核」、「作業」、「執行」和「要求」。 |
| resultType | 選用 | 事件的狀態。 一般值包括︰「已啟動」、「進行中」、「成功」、「失敗」、「作用中」和「已解決」。 |
| resultSignature | 選用 | 事件的子狀態。 如果此作業對應至 REST API 呼叫，則這是對應 REST 呼叫的 HTTP 狀態碼。 |
| resultDescription | 選用 | 此作業的靜態文字描述，例如 “Get storage file.” |
| durationMs | 選用 | 作業的持續時間 (以毫秒為單位)。 |
| callerIpAddress | 選用 | 呼叫端 IP 位址，如果作業對應至來自具有公開可用 IP 位址之實體的 API 呼叫。 |
| correlationId | 選用 | 用來將一組相關事件群組在一起的 GUID。 一般而言，如果兩個事件具有相同 operationName 但具有兩個不同狀態 (例如 "Started" 和 "Succeeded")，則會共用相同的相互關聯識別碼。 這也可能代表事件之間的其他關聯性。 |
| 身分識別 | 選用 | JSON Blob，描述已執行作業之使用者或應用程式的身分識別。 這通常包括來自 Active Directory 的授權和宣告/JWT 權杖。 |
| Level | 選用 | 事件的嚴重性層級。 必須是「資訊」、「警告」、「錯誤」或「嚴重」中的其中一個。 |
| location | 選用 | 發出事件之資源的區域，例如 「美國東部」或「法國南部」 |
| properties | 選用 | 任何與此特定事件類別相關的擴充屬性。 所有自訂/唯一屬性都必須放在結構描述的這個「部分 B」內。 |

## <a name="service-specific-schemas-for-resource-diagnostic-logs"></a>資源診斷記錄的服務特定結構描述
資源診斷記錄的結構描述會根據資源和記錄類別而有所不同。 此清單顯示的所有服務都可以製作服務和類別特定結構描述 (如果可用) 的可用診斷記錄和連結。

| 服務 | 結構描述與文件 |
| --- | --- |
| Azure Active Directory | [概觀](../active-directory/reports-monitoring/overview-activity-logs-in-azure-monitor.md)、[稽核記錄結構描述](../active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema.md)和[登入結構描述](../active-directory/reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md) |
| Analysis Services | https://azure.microsoft.com/blog/azure-analysis-services-integration-with-azure-diagnostic-logs/ |
| API 管理 | [API 管理診斷記錄](../api-management/api-management-howto-use-azure-monitor.md#diagnostic-logs) |
| 應用程式閘道 |[應用程式閘道的診斷記錄功能](../application-gateway/application-gateway-diagnostics.md) |
| Azure 自動化 |[Azure 自動化的記錄檔分析](../automation/automation-manage-send-joblogs-log-analytics.md) |
| Azure Batch |[Azure Batch 診斷記錄](../batch/batch-diagnostics.md) |
| 認知服務 | 無法使用結構描述。 |
| 內容傳遞網路 | [CDN 的 Azure 診斷記錄](../cdn/cdn-azure-diagnostic-logs.md) |
| CosmosDB | [Azure Cosmos DB 記錄](../cosmos-db/logging.md) |
| Data Factory | [使用 Azure 監視器來監視 Data Factory](../data-factory/monitor-using-azure-monitor.md) |
| 資料湖分析 |[存取 Azure Data Lake Analytics 的診斷記錄](../data-lake-analytics/data-lake-analytics-diagnostic-logs.md) |
| Data Lake Store |[存取 Azure Data Lake Store 的診斷記錄](../data-lake-store/data-lake-store-diagnostic-logs.md) |
| 適用於 PostgreSQL 的 DB |  無法使用結構描述。 |
| 事件中樞 |[Azure 事件中樞診斷記錄](../event-hubs/event-hubs-diagnostic-logs.md) |
| ExpressRoute | 無法使用結構描述。 |
| Azure 防火牆 | 無法使用結構描述。 |
| IoT 中樞 | [IoT 中樞作業](../iot-hub/iot-hub-monitor-resource-health.md#use-azure-monitor) |
| Key Vault |[Azure 金鑰保存庫記錄](../key-vault/key-vault-logging.md) |
| 負載平衡器 |[Azure 負載平衡器的 Log Analytics](../load-balancer/load-balancer-monitor-log.md) |
| Logic Apps |[Logic Apps B2B 自訂追蹤結構描述](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md) |
| 網路安全性群組 |[網路安全性群組 (NSG) 的記錄檔分析](../virtual-network/virtual-network-nsg-manage-log.md) |
| DDOS 保護 | [管理 Azure DDoS Protection Standard](../virtual-network/manage-ddos-protection.md) |
| PowerBI 專用 | [Azure 中 PowerBI Embedded 的診斷記錄](https://docs.microsoft.com/power-bi/developer/azure-pbie-diag-logs) |
| [復原服務] | [Azure 備份的資料模型](../backup/backup-azure-reports-data-model.md)|
| Search |[啟用和使用搜尋流量分析](../search/search-traffic-analytics.md) |
| 服務匯流排 |[Azure 服務匯流排診斷記錄](../service-bus-messaging/service-bus-diagnostic-logs.md) |
| SQL Database | [Azure SQL Database 診斷記錄](../sql-database/sql-database-metrics-diag-logging.md) |
| 串流分析 |[作業診斷記錄](../stream-analytics/stream-analytics-job-diagnostic-logs.md) |
| 流量管理員 | 無法使用結構描述。 |
| 虛擬網路 | 無法使用結構描述。 |
| 虛擬網路閘道 | 無法使用結構描述。 |

## <a name="supported-log-categories-per-resource-type"></a>每個資源類型支援的記錄檔類別
|資源類型|類別|類別顯示名稱|
|---|---|---|
|Microsoft.AnalysisServices/servers|引擎|引擎|
|Microsoft.AnalysisServices/servers|服務|服務|
|Microsoft.ApiManagement/service|GatewayLogs|ApiManagement 閘道的相關記錄檔|
|Microsoft.Automation/automationAccounts|JobLogs|作業記錄檔|
|Microsoft.Automation/automationAccounts|JobStreams|作業串流|
|Microsoft.Automation/automationAccounts|DscNodeStatus|Dsc 節點狀態|
|Microsoft.Batch/batchAccounts|ServiceLog|服務記錄檔|
|Microsoft.Cdn/profiles/endpoints|CoreAnalytics|取得端點的計量，例如頻寬、輸出等資訊。|
|Microsoft.ClassicNetwork/networksecuritygroups|網路安全性群組規則流程事件|網路安全性群組規則流程事件|
|Microsoft.CognitiveServices/accounts|稽核|稽核|
|Microsoft.ContainerService/managedClusters|kube-apiserver|Kubernetes API 伺服器|
|Microsoft.ContainerService/managedClusters|kube-controller-manager|Kubernetes 控制器管理員|
|Microsoft.ContainerService/managedClusters|kube-scheduler|Kubernetes 排程器|
|Microsoft.ContainerService/managedClusters|成立條件|驗證 Webhook|
|Microsoft.CustomerInsights/hubs|AuditEvents|AuditEvents|
|Microsoft.DataFactory/factories|ActivityRuns|管線活動執行記錄|
|Microsoft.DataFactory/factories|PipelineRuns|管線執行記錄|
|Microsoft.DataFactory/factories|TriggerRuns|觸發程序執行記錄|
|Microsoft.DataLakeAnalytics/accounts|稽核|稽核記錄|
|Microsoft.DataLakeAnalytics/accounts|要求|要求記錄檔|
|Microsoft.DataLakeStore/accounts|稽核|稽核記錄|
|Microsoft.DataLakeStore/accounts|Requests|要求記錄檔|
|Microsoft.DBforPostgreSQL/servers|PostgreSQLLogs|PostgreSQL 伺服器記錄|
|Microsoft.Devices/IotHubs|連線|連線|
|Microsoft.Devices/IotHubs|DeviceTelemetry|裝置遙測|
|Microsoft.Devices/IotHubs|C2DCommands|C2D 命令|
|Microsoft.Devices/IotHubs|DeviceIdentityOperations|裝置身分識別作業|
|Microsoft.Devices/IotHubs|FileUploadOperations|檔案上傳作業|
|Microsoft.Devices/IotHubs|路由|路由|
|Microsoft.Devices/IotHubs|D2C 對應項作業|D2C 對應項作業|
|Microsoft.Devices/IotHubs|C2DTwinOperations|C2D 對應項作業|
|Microsoft.Devices/IotHubs|TwinQueries|對應項查詢|
|Microsoft.Devices/IotHubs|JobsOperations|作業的操作|
|Microsoft.Devices/IotHubs|DirectMethods|直接方法|
|Microsoft.Devices/IotHubs|E2EDiagnostics|E2E 診斷 (預覽)|
|Microsoft.Devices/IotHubs|組態|組態|
|Microsoft.Devices/provisioningServices|DeviceOperations|裝置作業|
|Microsoft.Devices/provisioningServices|ServiceOperations|服務作業|
|Microsoft.DocumentDB/databaseAccounts|DataPlaneRequests|DataPlaneRequests|
|Microsoft.DocumentDB/databaseAccounts|MongoRequests|MongoRequests|
|Microsoft.DocumentDB/databaseAccounts|QueryRuntimeStatistics|QueryRuntimeStatistics|
|Microsoft.EventHub/namespaces|ArchiveLogs|封存記錄檔|
|Microsoft.EventHub/namespaces|OperationalLogs|作業記錄|
|Microsoft.EventHub/namespaces|AutoScaleLogs|自動調整規模記錄檔|
|Microsoft.KeyVault/vaults|AuditEvent|稽核記錄|
|Microsoft.Logic/workflows|WorkflowRuntime|工作流程執行階段診斷事件|
|Microsoft.Logic/integrationAccounts|IntegrationAccountTrackingEvents|整合帳戶追蹤事件|
|Microsoft.Network/networksecuritygroups|NetworkSecurityGroupEvent|網路安全性群組事件|
|Microsoft.Network/networksecuritygroups|NetworkSecurityGroupRuleCounter|網路安全性群組規則計數器|
|Microsoft.Network/loadBalancers|LoadBalancerAlertEvent|負載平衡器警示事件|
|Microsoft.Network/loadBalancers|LoadBalancerProbeHealthStatus|負載平衡器探查健全狀況狀態|
|Microsoft.Network/publicIPAddresses|DDoSProtectionNotifications|DDoS 保護通知|
|Microsoft.Network/virtualNetworks|VMProtectionAlerts|VM 保護警示|
|Microsoft.Network/applicationGateways|ApplicationGatewayAccessLog|應用程式閘道存取記錄檔|
|Microsoft.Network/applicationGateways|ApplicationGatewayPerformanceLog|應用程式閘道效能記錄檔|
|Microsoft.Network/applicationGateways|ApplicationGatewayFirewallLog|應用程式閘道防火牆記錄檔|
|Microsoft.Network/securegateways|AzureFirewallApplicationRule|Azure 防火牆應用程式規則|
|Microsoft.Network/securegateways|AzureFirewallNetworkRule|Azure 防火牆網路規則|
|Microsoft.Network/azurefirewalls|AzureFirewallApplicationRule|Azure 防火牆應用程式規則|
|Microsoft.Network/azurefirewalls|AzureFirewallNetworkRule|Azure 防火牆網路規則|
|Microsoft.Network/virtualNetworkGateways|GatewayDiagnosticLog|閘道診斷記錄|
|Microsoft.Network/virtualNetworkGateways|TunnelDiagnosticLog|通道診斷記錄|
|Microsoft.Network/virtualNetworkGateways|RouteDiagnosticLog|路由診斷記錄|
|Microsoft.Network/virtualNetworkGateways|IKEDiagnosticLog|IKE 診斷記錄|
|Microsoft.Network/virtualNetworkGateways|P2SDiagnosticLog|P2S 診斷記錄|
|Microsoft.Network/trafficManagerProfiles|ProbeHealthStatusEvents|流量管理員探查健康情況結果事件|
|Microsoft.Network/expressRouteCircuits|PeeringRouteLog|對等互連路由表記錄|
|Microsoft.PowerBIDedicated/capacities|引擎|引擎|
|Microsoft.RecoveryServices/Vaults|AzureBackupReport|Azure 備份報表資料|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryJobs|Azure Site Recovery 作業|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryEvents|Azure Site Recovery 事件|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryReplicatedItems|Azure Site Recovery 複寫項目|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryReplicationStats|Azure Site Recovery 複寫統計資料|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryRecoveryPoints|Azure Site Recovery 復原點|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryReplicationDataUploadRate|Azure Site Recovery 複寫資料上傳速率|
|Microsoft.RecoveryServices/Vaults|AzureSiteRecoveryProtectedDiskDataChurn|Azure Site Recovery 受保護的磁碟資料變換|
|Microsoft.Search/searchServices|OperationLogs|作業記錄|
|Microsoft.ServiceBus/namespaces|OperationalLogs|作業記錄|
|Microsoft.Sql/servers/databases|SQLInsights|SQL Insights|
|Microsoft.Sql/servers/databases|AutomaticTuning|自動微調|
|Microsoft.Sql/servers/databases|QueryStoreRuntimeStatistics|查詢存放區執行階段統計資料|
|Microsoft.Sql/servers/databases|QueryStoreWaitStatistics|查詢存放區等候統計資料|
|Microsoft.Sql/servers/databases|Errors|Errors|
|Microsoft.Sql/servers/databases|DatabaseWaitStatistics|資料庫等候統計資料|
|Microsoft.Sql/servers/databases|逾時|逾時|
|Microsoft.Sql/servers/databases|區塊|區塊|
|Microsoft.Sql/servers/databases|死結|死結|
|Microsoft.Sql/servers/databases|稽核|稽核記錄|
|Microsoft.Sql/servers/databases|SQLSecurityAuditEvents|SQL 安全性稽核事件|
|Microsoft.Sql/servers/databases|SqlDw_Requests|SQL DW 要求|
|Microsoft.Sql/servers/databases|SqlDw_RequestSteps|SQL DW 要求步驟|
|Microsoft.StreamAnalytics/streamingjobs|執行|執行|
|Microsoft.StreamAnalytics/streamingjobs|編寫|編寫|

## <a name="next-steps"></a>後續步驟

* [深入了解診斷記錄](monitoring-overview-of-diagnostic-logs.md)
* [將資源診斷記錄串流至**事件中樞**](monitoring-stream-diagnostic-logs-to-event-hubs.md)
* [使用 Azure 監視器 REST API 變更資源診斷設定](https://msdn.microsoft.com/library/azure/dn931931.aspx)
* [使用 Log Analytics 分析來自 Azure 儲存體的記錄](../log-analytics/log-analytics-azure-storage.md)
