---
title: Azure SQL Database 連線架構 | Microsoft Docs
description: 本文件說明 Azure 內外部的 Azure SQLDB 連線架構。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: DhruvMsft
ms.author: dhruv
ms.reviewer: carlrab
manager: craigg
ms.date: 01/24/2018
ms.openlocfilehash: 66f558db713ab951864fe694f27f2e60d52e875a
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064125"
---
# <a name="azure-sql-database-connectivity-architecture"></a>Azure SQL Database 連線架構 

本文說明 Azure SQL Database 連線架構並說明不同的元件函數如何將流量導向至您的 Azure SQL Database 執行個體。 這些 Azure SQL Database 連線元件函數可使用從 Azure 內部連線的用戶端和從 Azure 外部連線的用戶端，將網路流量導向至 Azure 資料庫。 本文也提供可變更連線方式的指令碼範例，以及和變更預設連線設定相關的考量。 

## <a name="connectivity-architecture"></a>連線架構

下圖提供 Azure SQL Database 連線架構的高階概觀。

![架構概觀](./media/sql-database-connectivity-architecture/architecture-overview.png)


下列步驟說明如何透過 Azure SQL Database 軟體負載平衡器 (SLB) 和 Azure SQL Database 閘道，和 Azure SQL Database 建立連線。

- Azure 內部或 Azure 外部的用戶端會連線到其有公用 IP 位址且接聽連接埠 1433 的 SLB。
- SLB 會將流量導向至 Azure SQL Database 閘道。
- 閘道會將流量重新導向至正確的 Proxy 中介軟體。
- Proxy 中介軟體會將流量重新導向到適當的 Azure SQL Database。

> [!IMPORTANT]
> 這些元件各個都在網路和應用程式層內建分散式阻斷服務 (DDoS) 保護。
>

## <a name="connectivity-from-within-azure"></a>從 Azure 內部連線

如果您從 Azure 內部連線，該連線預設的連線原則為 [重新導向]。 [重新導向] 原則代表連線在和 Azure SQL Database 建立 TCP 工作階段之後，會將其 Azure SQL Database 閘道的目的地虛擬 IP 變更為 Proxy 中介軟體的目的地虛擬 IP，將該用戶端工作階段重新導向至 Proxy 中介軟體。 此後，所有後續封包會略過 Azure SQL Database 閘道，直接流經 Proxy 中介軟體。 下圖說明此流量。

![架構概觀](./media/sql-database-connectivity-architecture/connectivity-from-within-azure.png)

## <a name="connectivity-from-outside-of-azure"></a>從 Azure 外部連線

如果您從 Azure 外部連線，該連線預設的連線原則為 [Proxy]。 [Proxy] 原則代表會透過 Azure SQL Database 閘道建立 TCP 工作階段，且所有後續封包都會流經閘道。 下圖說明此流量。

![架構概觀](./media/sql-database-connectivity-architecture/connectivity-from-outside-azure.png)

> [!IMPORTANT]
> 搭配 Azure SQL Database 使用服務端點時，您的預設原則會是 **Proxy**。 若要從 VNet 內進行連線，您必須允許對以下清單中指定的 Azure SQL Database 閘道 IP 位址進行連出連線。 使用服務端點時，強烈建議將您的連線原則變更為 [重新導向] 以達到更佳的效能。 如果您將連線原則變更為 [重新導向]，則不足以允許在您的 NSG 上輸出至下面所列的 Azure SQLDB 閘道 IP，您就必須允許輸出至所有 Azure SQLDB IP。 透過 NSG (網路安全性群組) 服務標籤可以完成此作業。 如需詳細資訊，請參閱[服務標籤](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)。

## <a name="azure-sql-database-gateway-ip-addresses"></a>Azure SQL Database 閘道 IP 位址

若要從內部部署資源連線到 Azure SQL Database，您必須允許輸出網路流量流到 Azure 區域的 Azure SQL Database 閘道。 在從內部部署資源連線時的預設 Proxy 模式中連線時，您只能透過閘道連線。

下表列出所有資料區之 Azure SQL Database 閘道的主要和次要 IP。 對於某些區域，會有兩個 IP 位址。 在這些區域中，主要 IP 位址是閘道目前的 IP 位址，而次要 IP 位址為容錯移轉 IP 位址。 容錯移轉位址是指可能會移動伺服器以讓服務保持高可用性的位址。 對於這些區域，建議您針對這兩個 IP 位址允許輸出。 次要 IP 位址為 Microsoft 所有，在由 Azure SQL Database 啟動以接受連線之前，並不會接聽任何服務。

> [!IMPORTANT]
> 如果從 Azure 內部連線，則根據預設連線原則將為 [重新導向] (除非您使用的是服務端點)。 它不足以允許下列 IP。 您必須允許所有 Azure SQL Database IP。 如果您是從 VNet 內進行連線，透過 NSG (網路安全性群組) 服務標籤可以完成此作業。 如需詳細資訊，請參閱[服務標籤](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)。

| 區域名稱 | 主要 IP 位址 | 次要 IP 位址 |
| --- | --- |--- |
| 澳洲東部 | 191.238.66.109 | 13.75.149.87 |
| 澳大利亞東南部 | 191.239.192.109 | 13.73.109.251 |
| 巴西南部 | 104.41.11.5 | |
| 加拿大中部 | 40.85.224.249 | |
| 加拿大東部 | 40.86.226.166 | |
| 美國中部 | 23.99.160.139 | 13.67.215.62 |
| 東亞 | 191.234.2.139 | 52.175.33.150 |
| 美國東部 1 | 191.238.6.43 | 40.121.158.30 |
| 美國東部 2 | 191.239.224.107 | 40.79.84.180 * |
| 印度中部 | 104.211.96.159  | |
| 印度南部 | 104.211.224.146  | |
| 印度西部 | 104.211.160.80 | |
| 日本東部 | 191.237.240.43 | 13.78.61.196 |
| 日本西部 | 191.238.68.11 | 104.214.148.156 |
| 南韓中部 | 52.231.32.42 | |
| 南韓南部 | 52.231.200.86 |  |
| 美國中北部 | 23.98.55.75 | 23.96.178.199 |
| 北歐 | 191.235.193.75 | 40.113.93.91 |
| 美國中南部 | 23.98.162.75 | 13.66.62.124 |
| 東南亞 | 23.100.117.95 | 104.43.15.0 |
| 英國北部 | 13.87.97.210 | |
| 英國南部 1 | 51.140.184.11 | |
| 英國南部 2 | 13.87.34.7 | |
| 英國西部 | 51.141.8.11  | |
| 美國中西部 | 13.78.145.25 | |
| 西歐 | 191.237.232.75 | 40.68.37.158 |
| 美國西部 1 | 23.99.34.75 | 104.42.238.205 |
| 美國西部 2 | 13.66.226.202  | |
||||

\* **注意：**[美國東部 2] 也有第三 IP 位址 `52.167.104.0`。

## <a name="change-azure-sql-database-connection-policy"></a>變更 Azure SQL Database 連線原則

若要變更 Azure SQL Database 伺服器的 Azure SQL Database 連線原則，請使用 [conn-policy](https://docs.microsoft.com/cli/azure/sql/server/conn-policy) \(英文\) 命令。

- 如果您的連線原則設定為 [Proxy]，所有網路封包都會流經 Azure SQL Database 閘道。 對於此設定，您必須只允許輸出至 Azure SQL Database 閘道 IP。 和 [重新導向] 設定相比，使用 [Proxy] 設定會較為延遲。
- 如果您的連線原則設定為 [重新導向]，所有網路封包都會直接流到中介軟體 Proxy。 對於此設定，您需要允許輸出至多個 IP。

## <a name="script-to-change-connection-settings-via-powershell"></a>透過 PowerShell 變更連線設定的指令碼

> [!IMPORTANT]
> 此指令碼需要 [Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。
>

下列 PowerShell 指令碼會示範如何變更連線原則。

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName <Subscription Name>

# Azure Active Directory ID
$tenantId = "<Azure Active Directory GUID>"
$authUrl = "https://login.microsoftonline.com/$tenantId"

# Subscription ID
$subscriptionId = "<Subscription GUID>"

# Create an App Registration in Azure Active Directory.  Ensure the application type is set to NATIVE
# Under Required Permissions, add the API:  Windows Azure Service Management API

# Specify the redirect URL for the app registration
$uri = "<NATIVE APP - REDIRECT URI>"

# Specify the application id for the app registration
$clientId = "<NATIVE APP - APPLICATION ID>"

# Logical SQL Server Name
$serverName = "<LOGICAL DATABASE SERVER - NAME>"

# Resource Group where the SQL Server is located
$resourceGroupName= "<LOGICAL DATABASE SERVER - RESOURCE GROUP NAME>"


# Login and acquire a bearer token
$AuthContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]$authUrl
$result = $AuthContext.AcquireToken(
"https://management.core.windows.net/",
$clientId,
[Uri]$uri,
[Microsoft.IdentityModel.Clients.ActiveDirectory.PromptBehavior]::Auto
)

$authHeader = @{
'Content-Type'='application\json; '
'Authorization'=$result.CreateAuthorizationHeader()
}

#Get current connection Policy
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/servers/$serverName/connectionPolicies/Default?api-version=2014-04-01-preview" -Method GET -Headers $authHeader

#Set connection policy to Proxy
$connectionType="Proxy" <#Redirect / Default are other options#>
$body = @{properties=@{connectionType=$connectionType}} | ConvertTo-Json

# Apply Changes
Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Sql/servers/$serverName/connectionPolicies/Default?api-version=2014-04-01-preview" -Method PUT -Headers $authHeader -Body $body -ContentType "application/json"
```

## <a name="script-to-change-connection-settings-via-azure-cli"></a>透過 Azure CLI 變更連線設定的指令碼

> [!IMPORTANT]
> 此指令碼需要 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。
>

下列 CLI 指令碼會示範如何變更連線原則。

<pre>
# Get SQL Server ID
sqlserverid=$(az sql server show -n <b>sql-server-name</b> -g <b>sql-server-group</b> --query 'id' -o tsv)

# Set URI
id="$sqlserverid/connectionPolicies/Default"

# Get current connection policy 
az resource show --ids $id

# Update connection policy 
az resource update --ids $id --set properties.connectionType=Proxy

</pre>

## <a name="next-steps"></a>後續步驟

- 如需如何變更 Azure SQL Database 伺服器的 Azure SQL Database 連線原則相關資訊，請參閱 [conn-policy](https://docs.microsoft.com/cli/azure/sql/server/conn-policy) \(英文\)。
- 如需使用 ADO.NET 4.5 或更新版本用戶端之 Azure SQL Database 連接行為的詳細資訊，請參閱 [ADO.NET 4.5 超過 1433以外的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)。
- 如需一般應用程式開發概觀的資訊，請參閱 [SQL Database 應用程式開發概觀](sql-database-develop-overview.md)。
