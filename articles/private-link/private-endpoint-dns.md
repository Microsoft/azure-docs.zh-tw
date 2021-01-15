---
title: Azure 私人端點 DNS 設定
description: 瞭解 Azure 私人端點 DNS 設定
services: private-link
author: allensu
ms.service: private-link
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: allensu
ms.openlocfilehash: 49e1b45ca3953d008542c2ed508537d1a3ea0bf3
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98218972"
---
# <a name="azure-private-endpoint-dns-configuration"></a>Azure 私人端點 DNS 設定

請務必正確地設定您的 DNS 設定，以將私人端點 IP 位址解析為連接字串 (FQDN) 的完整功能變數名稱。

現有的 Microsoft Azure 服務可能已經有公用端點的 DNS 設定。 必須覆寫此設定，才能使用您的私人端點進行連接。 
 
與私人端點相關聯的網路介面包含設定 DNS 的資訊。 網路介面資訊包含私人連結資源的 FQDN 和私人 IP 位址。 
 
您可以使用下列選項來設定私人端點的 DNS 設定： 
- **使用主機檔案 (只建議用於測試)**。 您可以使用虛擬機器上的主機檔案來覆寫 DNS。  
- **使用私人 DNS 區域** 您可以使用 [私人 DNS 區域](../dns/private-dns-privatednszone.md) 覆寫私人端點的 DNS 解析。 私人 DNS 區域可以連結至您的虛擬網路，以解析特定網域。
- **使用您的 DNS 轉寄站 (選擇性)**。 您可以使用 DNS 轉寄站來覆寫私人連結資源的 DNS 解析。 建立 DNS 轉送規則，以在虛擬網路中託管的 [dns 伺服器](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) 上使用私人 DNS 區域。

> [!IMPORTANT]
> 不建議覆寫主動使用的區域來解析公用端點。 若未將 DNS 轉送至公用 DNS，將無法正確解析資源的連線。 為避免發生問題，請建立不同的網域名稱，或遵循下列每個服務的建議名稱。 

## <a name="azure-services-dns-zone-configuration"></a>Azure 服務 DNS 區域設定
Azure 會在公用 DNS 上建立標準名稱 DNS 記錄 (CNAME) 。 CNAME 記錄會將解析重新導向至私用功能變數名稱。 您可以使用私人端點的私人 IP 位址來覆寫解析。 
 
您的應用程式不需要變更連線 URL。 當解析為公用 DNS 服務時，DNS 伺服器會解析為您的私人端點。 此程式不會影響您現有的應用程式。 

> [!IMPORTANT]
> 私人網路已針對指定的類型使用私人 DNS 區域，只有在沒有任何私人端點連線時，才能連線至公用資源，否則私人 DNS 區域上需要有對應的 DNS 設定，才能完成 DNS 解析順序。 

針對 Azure 服務，請使用下表中所述的建議區域名稱：

| Private link 資源類型/Subresource |私人 DNS 區功能變數名稱稱 | 公用 DNS 區域轉寄站 |
|---|---|---|
| Azure 自動化/ (Microsoft. Automation/automationAccounts) /Webhook，DSCAndHybridWorker | privatelink.azure-automation.net | azure-automation.net |
| Azure SQL Database (的 Microsoft .Sql/伺服器) /SQL Server | privatelink.database.windows.net | database.windows.net |
| Azure Synapse Analytics (的 Microsoft .Sql/伺服器) /SQL Server  | privatelink.database.windows.net | database.windows.net |
| 儲存體帳戶 (儲存/storageAccounts) /Blob (blob，blob_secondary)  | privatelink.blob.core.windows.net | blob.core.windows.net |
| 儲存體帳戶 (Microsoft. 儲存體/storageAccounts) /資料表 (資料表、table_secondary)  | privatelink.table.core.windows.net | table.core.windows.net |
| 儲存體帳戶 (儲存/storageAccounts) /佇列 (佇列，queue_secondary)  | privatelink.queue.core.windows.net | queue.core.windows.net |
| 儲存體帳戶 (Microsoft. Storage/storageAccounts) /File (檔案，file_secondary)  | privatelink.file.core.windows.net | file.core.windows.net |
| 儲存體帳戶 (Microsoft Storage/storageAccounts) /Web (web，web_secondary)  | privatelink.web.core.windows.net | web.core.windows.net |
| Azure Data Lake 檔案系統 Gen2 (Microsoft. Storage/storageAccounts) /Data Lake 檔案系統 Gen2 (dfs，dfs_secondary)  | privatelink.dfs.core.windows.net | dfs.core.windows.net |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.com | documents.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.com | mongo.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.com | cassandra.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.com | gremlin.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Table | privatelink.table.cosmos.azure.com | table.cosmos.azure.com |
| 適用於 PostgreSQL 的 Azure 資料庫 - 單一伺服器 (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.azure.com | postgres.database.azure.com |
| 適用於 MySQL 的 Azure 資料庫 (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.azure.com | mysql.database.azure.com |
| 適用於 MariaDB 的 Azure 資料庫 (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.azure.com | mariadb.database.azure.com |
| Azure Key Vault (Microsoft.KeyVault/vaults) / 保存庫 | privatelink.vaultcore.azure.net | vault.azure.net <br> vaultcore.azure.net |
| Azure Kubernetes Service-Kubernetes API (Microsoft. >microsoft.containerservice/managedClusters) /管理 | privatelink.{region}.azmk8s.io | {region}.azmk8s.io |
| Azure 搜尋服務 (Microsoft.Search/searchServices) / searchService | privatelink.search.windows.net | search.windows.net |
| Azure Container Registry (Microsoft.ContainerRegistry/registries) / 登錄 | privatelink.azurecr.io | azurecr.io |
| Azure 應用程式組態 (Microsoft.AppConfiguration/configurationStores) / configurationStore | privatelink.azconfig.io | azconfig.io |
| Azure 備份 (Microsoft.RecoveryServices/vaults) / 保存庫 | privatelink.{region}.backup.windowsazure.com | {region}.backup.windowsazure.com |
| Azure 事件中樞 (Microsoft. EventHub/namespace) /namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure 服務匯流排 (Microsoft.ServiceBus/namespaces) / 命名空間 | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure IoT 中樞 (IotHubs) /iotHub | privatelink.azure-devices.net<br/>privatelink.servicebus.windows.net<sup>1</sup> | azure-devices.net<br/>servicebus.windows.net |
| Azure 轉送 (Microsoft.Relay/namespaces) / 命名空間 | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure 事件方格 (Microsoft.EventGrid/topics) / 主題 | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure 事件方格 (Microsoft.EventGrid/domains) / 網域 | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure Web Apps (的 Microsoft 網站/網站) /網站 | privatelink.azurewebsites.net | azurewebsites.net |
| Azure Machine Learning (MachineLearningServices/工作區) /工作區 | privatelink.api.azureml.ms | api.azureml.ms |
| IoT 中樞 (Microsoft. Devices/IotHubs) /IotHub | privatelink.azure-devices.net | azure-devices.net |
| SignalR (Microsoft. Microsoft.signalrservice/SignalR) /signalR | privatelink.service.signalr.net | service.signalr.net |
| Azure 監視器 (privateLinkScopes) /azuremonitor | privatelink.monitor.azure.com<br/> privatelink.oms.opinsights.azure.com <br/> privatelink.ods.opinsights.azure.com <br/> privatelink.agentsvc.azure-automation.net | monitor.azure.com<br/> oms.opinsights.azure.com<br/> ods.opinsights.azure.com<br/> agentsvc.azure-automation.net |
|  (Microsoft CognitiveServices/帳戶) /帳戶的認知服務 | privatelink.cognitiveservices.azure.com  | cognitiveservices.azure.com  |
| Azure 檔案同步 (Microsoft.storagesync/storageSyncServices) /afs |  privatelink.afs.azure.net  |  afs.azure.net  |
| Azure Data Factory (DataFactory/工廠) /dataFactory |  privatelink.datafactory.azure.net  |  datafactory.azure.net  |
| Azure Data Factory (DataFactory/工廠) /入口網站 |  privatelink.azure.com  |  azure.com  |
| Azure Cache for Redis (Redis) /redisCache | privatelink.redis.cache.windows.net | redis.cache.windows.net |

<sup>1</sup>使用 IoT 中樞內建的事件中樞相容端點。 若要深入瞭解，請參閱 [IoT 中樞內建端點的 private link 支援](../iot-hub/virtual-network-support.md#built-in-event-hub-compatible-endpoint)

### <a name="china"></a>中國

| Private link 資源類型/Subresource |私人 DNS 區功能變數名稱稱 | 公用 DNS 區域轉寄站 |
|---|---|---|
| Azure SQL Database (的 Microsoft .Sql/伺服器) /SQL Server | privatelink.database.chinacloudapi.cn | database.chinacloudapi.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.cn | documents.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.cn | mongo.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.cn | cassandra.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.cn | gremlin.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Table | privatelink.table.cosmos.azure.cn | table.cosmos.azure.cn |
| 適用於 PostgreSQL 的 Azure 資料庫 - 單一伺服器 (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.chinacloudapi.cn | postgres.database.chinacloudapi.cn |
| 適用於 MySQL 的 Azure 資料庫 (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.chinacloudapi.cn  | mysql.database.chinacloudapi.cn  |
| 適用於 MariaDB 的 Azure 資料庫 (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.chinacloudapi.cn | mariadb.database.chinacloudapi.cn |


## <a name="dns-configuration-scenarios"></a>DNS 設定案例

服務的 FQDN 會自動解析為公用 IP 位址。 若要解析為私人端點的私人 IP 位址，請變更您的 DNS 設定。

DNS 是一個重要元件，可成功解析私人端點 IP 位址，讓應用程式正常運作。

根據您的喜好設定，下列案例可用於整合 DNS 解析：

- [沒有自訂 DNS 伺服器的虛擬網路工作負載](#virtual-network-workloads-without-custom-dns-server)
- [使用 DNS 轉寄站的內部部署工作負載](#on-premises-workloads-using-a-dns-forwarder)
- [使用 DNS 轉寄站的虛擬網路和內部部署工作負載](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder)

> [!NOTE]
> 您可以使用[Azure 防火牆 dns proxy](../firewall/dns-settings.md#dns-proxy) ，作為[內部部署工作負載](#on-premises-workloads-using-a-dns-forwarder)的 dns 轉寄站，以及使用 Dns 轉寄站的[虛擬網路工作負載](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder)。

## <a name="virtual-network-workloads-without-custom-dns-server"></a>沒有自訂 DNS 伺服器的虛擬網路工作負載

這項設定適用于沒有自訂 DNS 伺服器的虛擬網路工作負載。 在此案例中，用戶端會向 Azure 提供的 DNS 服務 [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md)查詢私人端點 IP 位址。 Azure DNS 將負責私人 DNS 區域的 DNS 解析。

> [!NOTE]
> 此案例會使用 Azure SQL Database 建議的私人 DNS 區域。 對於其他服務，您可以使用下列參考來調整模型： [Azure 服務 DNS 區域](#azure-services-dns-zone-configuration)設定。

若要正確設定，您需要下列資源：

- 用戶端虛擬網路

- 具有[類型 A 記錄的](../dns/dns-zones-records.md#record-types)私人 DNS 區域[privatelink.database.windows.net](../dns/private-dns-privatednszone.md)

- 私人端點資訊 (FQDN 記錄名稱和私人 IP 位址) 

下列螢幕擷取畫面說明使用私人 DNS 區域的虛擬網路工作負載中的 DNS 解析順序：

:::image type="content" source="media/private-endpoint-dns/single-vnet-azure-dns.png" alt-text="單一虛擬網路與 Azure 提供的 DNS":::

您可以擴充此模型，以對等互連與相同私人端點相關聯的虛擬網路。 [將新的虛擬網路連結新增](../dns/private-dns-virtual-network-links.md) 至所有對等互連虛擬網路的私人 DNS 區域。

> [!IMPORTANT]
> 此設定需要單一私人 DNS 區域。 針對不同的虛擬網路建立多個具有相同名稱的區域，需要手動操作來合併 DNS 記錄。

> [!IMPORTANT]
> 如果您是在不同訂用帳戶的中樞和輪輻模型中使用私人端點，請在中樞上重複使用相同的私人 DNS 區域。

在此案例中，有一個 [中樞和輪輻](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) 網路拓撲。 輪輻網路會共用私人端點。 輪輻虛擬網路會連結到相同的私人 DNS 區域。 

:::image type="content" source="media/private-endpoint-dns/hub-and-spoke-azure-dns.png" alt-text="具有 Azure 提供之 DNS 的中樞與輪輻":::

## <a name="on-premises-workloads-using-a-dns-forwarder"></a>使用 DNS 轉寄站的內部部署工作負載

若要讓內部部署工作負載解析私人端點的 FQDN，請使用 DNS 轉寄站來解析 Azure 中的 Azure 服務 [公用 DNS 區域](#azure-services-dns-zone-configuration) 。

下列案例適用于在 Azure 中具有 DNS 轉寄站的內部部署網路。 此轉寄站會透過伺服器層級轉寄站，將 DNS 查詢解析成 Azure 提供的 DNS [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md)。 

> [!NOTE]
> 此案例會使用 Azure SQL Database 建議的私人 DNS 區域。 對於其他服務，您可以使用下列參考來調整模型： [Azure 服務 DNS 區域](#azure-services-dns-zone-configuration)設定。

若要正確設定，您需要下列資源：

- 內部部署網路
- [連線到內部部署的](/azure/architecture/reference-architectures/hybrid-networking/)虛擬網路
- 部署在 Azure 中的 DNS 轉寄站 
- 具有[類型 A 記錄的](../dns/dns-zones-records.md#record-types)私人 DNS 區域[privatelink.database.windows.net](../dns/private-dns-privatednszone.md)
- 私人端點資訊 (FQDN 記錄名稱和私人 IP 位址) 

下圖說明內部部署網路中的 DNS 解析順序。 設定會使用部署在 Azure 中的 DNS 轉寄站。 解析是由 [連結至虛擬網路](../dns/private-dns-virtual-network-links.md)的私人 DNS 區域所建立：

:::image type="content" source="media/private-endpoint-dns/on-premises-using-azure-dns.png" alt-text="使用 Azure DNS 的內部部署":::

這項設定可以針對已經有 DNS 解決方案的內部部署網路進行擴充。 內部部署 DNS 解決方案設定為透過 [條件](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)轉寄站將 DNS 流量轉送至 Azure DNS。 條件轉寄站會參考 Azure 中部署的 DNS 轉寄站。

> [!NOTE]
> 此案例會使用 Azure SQL Database 建議的私人 DNS 區域。 針對其他服務，您可以使用下列參考來調整模型： [Azure 服務 DNS 區域](#azure-services-dns-zone-configuration)設定

若要正確設定，您需要下列資源：

- 具備自訂 DNS 解決方案的內部部署網路 
- [連線到內部部署的](/azure/architecture/reference-architectures/hybrid-networking/)虛擬網路
- 部署在 Azure 中的 DNS 轉寄站
- 具有[類型 A 記錄的](../dns/dns-zones-records.md#record-types)私人 DNS 區域[privatelink.database.windows.net](../dns/private-dns-privatednszone.md)
- 私人端點資訊 (FQDN 記錄名稱和私人 IP 位址) 

下圖說明內部部署網路中的 DNS 解析。 DNS 解析會有條件地轉送至 Azure。 解析是由 [連結至虛擬網路](../dns/private-dns-virtual-network-links.md)的私人 DNS 區域所建立。

> [!IMPORTANT]
> 條件式轉送必須對建議的 [公用 DNS 區域](#azure-services-dns-zone-configuration)轉寄站進行。 例如： `database.windows.net` 而不是 **privatelink**. database.windows.net。

:::image type="content" source="media/private-endpoint-dns/on-premises-forwarding-to-azure.png" alt-text="內部部署轉送至 Azure DNS":::

## <a name="virtual-network-and-on-premises-workloads-using-a-dns-forwarder"></a>使用 DNS 轉寄站的虛擬網路和內部部署工作負載

針對從虛擬和內部部署網路存取私人端點的工作負載，請使用 DNS 轉寄站來解析部署在 Azure 中的 Azure 服務 [公用 DNS 區域](#azure-services-dns-zone-configuration) 。

下列案例適用于在 Azure 中使用虛擬網路的內部部署網路。 這兩個網路都會存取位於共用中樞網路中的私人端點。

此 DNS 轉寄站負責透過伺服器層級轉寄站，將所有 DNS 查詢解析到 Azure 提供的 DNS 服務 [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md)。

> [!IMPORTANT]
> 此設定需要單一私人 DNS 區域。 從內部部署和 [對等互連虛擬網路](../virtual-network/virtual-network-peering-overview.md) 建立的所有用戶端連線也都必須使用相同的私人 DNS 區域。

> [!NOTE]
> 此案例會使用 Azure SQL Database 建議的私人 DNS 區域。 對於其他服務，您可以使用下列參考來調整模型： [Azure 服務 DNS 區域](#azure-services-dns-zone-configuration)設定。

若要正確設定，您需要下列資源：

- 內部部署網路
- [連線到內部部署的](/azure/architecture/reference-architectures/hybrid-networking/)虛擬網路
- [對等互連虛擬網路](../virtual-network/virtual-network-peering-overview.md) 
- 部署在 Azure 中的 DNS 轉寄站
- 具有[類型 A 記錄的](../dns/dns-zones-records.md#record-types)私人 DNS 區域[privatelink.database.windows.net](../dns/private-dns-privatednszone.md)
- 私人端點資訊 (FQDN 記錄名稱和私人 IP 位址) 

下圖顯示網路、內部部署和虛擬網路的 DNS 解析。 解決方法是使用 DNS 轉寄站。 解析是由 [連結至虛擬網路](../dns/private-dns-virtual-network-links.md)的私人 DNS 區域所建立：

:::image type="content" source="media/private-endpoint-dns/hybrid-scenario.png" alt-text="混合式案例":::

## <a name="next-steps"></a>後續步驟
- [深入瞭解私人端點](private-endpoint-overview.md)
