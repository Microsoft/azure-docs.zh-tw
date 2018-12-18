---
title: 網路安全性群組事件和規則計數器 Azure 診斷記錄 | Microsoft Docs
description: 了解如何針對 Azure 網路安全性群組啟用事件和規則計數器診斷記錄。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 2e699078-043f-48bd-8aa8-b011a32d98ca
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/04/2018
ms.author: jdial
ms.openlocfilehash: 5635998eb72f08ddc665793e77008890b2cdb05d
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42140903"
---
# <a name="diagnostic-logging-for-a-network-security-group"></a>適用於網路安全性群組的診斷記錄

網路安全性群組 (NSG) 包含允許或拒絕前往虛擬網路子網路、網路介面或兩者流量的規則。 當您針對 NSG 啟用診斷記錄時，可以記錄下列類別的資訊：

* **事件︰** 記錄要根據 MAC 位址，將哪些 NSG 規則套用至 VM 的項目。 每隔 60 秒會收集一次這些規則的狀態。
* **規則計數器：** 包含套用每個 NSG 規則以拒絕或允許流量之次數的項目。

診斷記錄檔僅適用於透過 Azure Resource Manager 部署模型中部署的 NSG。 您無法啟用透過傳統部署模型部署的 NSG 診斷記錄。 若要深入了解這兩個模型，請參閱[了解 Azure 部署模型](../resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

診斷記錄會針對*每個*您想要收集診斷資料的 NSG 分別啟用。 如果您對作業或活動記錄較感興趣，請參閱 Azure [活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="enable-logging"></a>啟用記錄

您可以使用 [Azure 入口網站](#azure-portal)、[PowerShell](#powershell) 或 [Azure CLI](#azure-cli) 來啟用診斷記錄。

### <a name="azure-portal"></a>Azure 入口網站

1. 登入[入口網站](https://portal.azure.com)。
2. 選取 [所有服務]，然後輸入*網路安全性群組*。 當 [網路安全性群組] 出現在搜尋結果中時，請選取它。
3. 選取您想要啟用記錄功能的 NSG。
4. 在 [監視] 下方，選取 [診斷記錄]，然後選取 [開啟診斷]，如下圖所示：
 
    ![開啟診斷](./media/virtual-network-nsg-manage-log/turn-on-diagnostics.png)

5. 在 [診斷設定] 下方，輸入或選取下列資訊，然後選取 [儲存]：

    | 設定                                                                                     | 值                                                          |
    | ---------                                                                                   |---------                                                       |
    | 名稱                                                                                        | 您選擇的名稱。  例如：*myNsgDiagnostics*      |
    | **封存至儲存體帳戶**，**串流至事件中樞**，以及**傳送至 Log Analytics** | 您可以任意選取多個目的地。 若要深入了解每個目的地，請參閱[記錄目的地](#log-destinations)。                                                                                                                                           |
    | 記錄                                                                                         | 選取任一或兩個記錄類別。 若要深入了解針對每個類別所記錄的資料，請參閱[記錄類別](#log-categories)。                                                                                                                                             |
6. 檢視及分析記錄。 如需詳細資訊，請參閱[檢視及分析記錄檔](#view-and-analyze-logs)。

### <a name="powershell"></a>PowerShell

您可以執行 [Azure Cloud Shell](https://shell.azure.com/powershell) 中採用的命令，或從您的電腦執行 PowerShell。 Azure Cloud Shell 是免費的互動式殼層。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 如果您從電腦執行 PowerShell，就需要 *AzureRM* PowerShell 模組 6.1.1 版或更新版本。 請在您的電腦上執行 `Get-Module -ListAvailable AzureRM`，以尋找已安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則還需要執行 `Login-AzureRmAccount` 以使用具有[必要權限](virtual-network-network-interface.md#permissions)的帳戶來登入 Azure。

若要啟用診斷記錄，您需要現有 NSG 的識別碼。 如果您目前沒有 NSG，可以使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) 建立一個。

使用 [Get-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/get-azurermnetworksecuritygroup) 來擷取您想要啟用診斷記錄的網路安全性群組。 例如，若要擷取名為 *myNsg* 的 NSG 且該 NSG 存在於名為 *myResourceGroup* 的資源群組中，請輸入下列命令：

```azurepowershell-interactive
$Nsg=Get-AzureRmNetworkSecurityGroup `
  -Name myNsg `
  -ResourceGroupName myResourceGroup
```

您可以將診斷記錄寫入至三種目的地類型。 如需詳細資訊，請參閱[記錄目的地](#log-destinations)。 舉例來說，本文中的內容會將記錄傳送到 *Log Analytics* 目的地。 使用 [Get-AzureRmOperationalInsightsWorkspace](/powershell/module/azurerm.operationalinsights/get-azurermoperationalinsightsworkspace) 來擷取現有的 Log Analytics 工作區。 例如，若要在名為 myWorkspaces 的資源群組中擷取名為 myWorkspace 的現有工作區，請輸入下列命令：

```azurepowershell-interactive
$Oms=Get-AzureRmOperationalInsightsWorkspace `
  -ResourceGroupName myWorkspaces `
  -Name myWorkspace
```

如果您目前沒有工作區，可以使用 [New-AzureRmOperationalInsightsWorkspace](/powershell/module/azurerm.operationalinsights/new-azurermoperationalinsightsworkspace) 建立一個。

您可以啟用記錄的記錄類別有兩種。 如需詳細資訊，請參閱[記錄類別](#log-categories)。 使用 [Set-AzureRmDiagnosticSetting](/powershell/module/azurerm.insights/set-azurermdiagnosticsetting) 來針對 NSG 啟用診斷記錄。 下列範例會使用您先前所擷取 NSG 和工作區的識別碼，將事件和計數器類別資料記錄到 NSG 的工作區：

```azurepowershell-interactive
Set-AzureRmDiagnosticSetting `
  -ResourceId $Nsg.Id `
  -WorkspaceId $Oms.ResourceId `
  -Enabled $true
```

如果您只想針對其中一個類別 (而非兩者) 記錄資料，請將 `-Categories` 選項加入至先前的命令，後面接著輸入 *NetworkSecurityGroupEvent* 或 *NetworkSecurityGroupRuleCounter*。 如果您想要記錄到 Log Analytics 工作區以外的不同[目的地](#log-destinations)，請使用適用於 Azure [儲存體帳戶](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或[事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的適當參數。

檢視及分析記錄。 如需詳細資訊，請參閱[檢視及分析記錄檔](#view-and-analyze-logs)。

### <a name="azure-cli"></a>Azure CLI

您可以執行 [Azure Cloud Shell](https://shell.azure.com/bash) 中採用的命令，或從您的電腦執行 Azure CLI。 Azure Cloud Shell 是免費的互動式殼層。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 如果您是從電腦執行 CLI，您需要版本 2.0.38 或更新版本。 請在您的電腦上執行 `az --version`，以尋找已安裝的版本。 如果您需要升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest)。 如果您在本機執行 CLI，則還需要執行 `az login` 以使用具有[必要權限](virtual-network-network-interface.md#permissions)的帳戶來登入 Azure。

若要啟用診斷記錄，您需要現有 NSG 的識別碼。 如果您目前沒有 NSG，可以使用 [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) 來建立。

使用 [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show) 來擷取您想要啟用診斷記錄的網路安全性群組。 例如，若要擷取名為 *myNsg* 的 NSG 且該 NSG 存在於名為 *myResourceGroup* 的資源群組中，請輸入下列命令：

```azurecli-interactive
nsgId=$(az network nsg show \
  --name myNsg \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)
```

您可以將診斷記錄寫入至三種目的地類型。 如需詳細資訊，請參閱[記錄目的地](#log-destinations)。 舉例來說，本文中的內容會將記錄傳送到 *Log Analytics* 目的地。 如需詳細資訊，請參閱[記錄類別](#log-categories)。 

使用 [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) 來針對 NSG 啟用診斷記錄。 下列範例會將事件和計數器類別資料都記錄到名為 myWorkspace 的現有工作區，該工作區存在於名為 myWorkspaces 的資源群組中，NSG 識別碼則是您先前所擷取的：

```azurecli-interactive
az monitor diagnostic-settings create \
  --name myNsgDiagnostics \
  --resource $nsgId \
  --logs '[ { "category": "NetworkSecurityGroupEvent", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "NetworkSecurityGroupRuleCounter", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --workspace myWorkspace \
  --resource-group myWorkspaces
```

如果您目前沒有工作區，可以使用 [Azure 入口網站](../log-analytics/log-analytics-quick-create-workspace.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或 [PowerShell](/powershell/module/azurerm.operationalinsights/new-azurermoperationalinsightsworkspace) 來建立一個工作區。 您可以啟用記錄的記錄類別有兩種。 

如果您只想要記錄某個類別或其他類別的資料，請在上一個命令中移除您不想要記錄資料的類別。 如果您想要記錄到 Log Analytics 工作區以外的不同[目的地](#log-destinations)，請使用適用於 Azure [儲存體帳戶](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或[事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的適當參數。

檢視及分析記錄。 如需詳細資訊，請參閱[檢視及分析記錄檔](#view-and-analyze-logs)。

## <a name="log-destinations"></a>記錄目的地

診斷資料可以：
- [寫入至 Azure 儲存體帳戶](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以利稽核或手動檢查。 您可以使用資源診斷設定來指定保留時間 (以天為單位)。
- [串流至事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，以供第三方服務或自訂的分析解決方案 (如 PowerBI) 擷取。
- [寫入至 Azure Log Analytics](../log-analytics/log-analytics-azure-storage.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-diagnostics-direct-to-log-analytics)。

## <a name="log-categories"></a>記錄類別

系統針對下列記錄類別會寫入 JSON 格式的資料：

### <a name="event"></a>Event

事件記錄包含要根據 MAC 位址，將哪些 NSG 規則套用至 VM 的相關資訊。 每個事件會記錄下列資料。 在下列範例中，會為 IP 位址為 192.168.1.4 且 MAC 為 00-0D-3A-92-6A-7C 的虛擬機器記錄資料：

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupEvent",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupEvents",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "priority":[PRIORITY-SPECIFIED-IN-RULE],
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "conditions":{
            "protocols":"[PROTOCOLS-SPECIFIED-IN-RULE]",
            "destinationPortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourcePortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourceIP":"[SOURCE-IP-OR-RANGE-SPECIFIED-IN-RULE]",
            "destinationIP":"[DESTINATION-IP-OR-RANGE-SPECIFIED-IN-RULE]"
            }
        }
}
```

### <a name="rule-counter"></a>規則計數器

規則計數器記錄中針對每個套用至資源的規則，包含其相關資訊。 每次套用規則時會記錄下列範例資料。 在下列範例中，會為 IP 位址為 192.168.1.4 且 MAC 為 00-0D-3A-92-6A-7C 的虛擬機器記錄資料：

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupRuleCounter",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupCounters",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "matchedConnections":125
        }
}
```

> [!NOTE]
> 不會記錄用於通訊的來源 IP 位址。 但是，您可以針對 NSG 啟用 [NSG 流程記錄](../network-watcher/network-watcher-nsg-flow-logging-portal.md)，它會記錄所有規則計數器資訊，以及起始通訊的來源 IP 位址。 NSG 流量記錄資料會寫入至 Azure 儲存體帳戶。 您可以使用 Azure 網路監看員的[流量分析](../network-watcher/traffic-analytics.md)功能來分析資料。

## <a name="view-and-analyze-logs"></a>檢視及分析記錄檔

若要了解如何檢視診斷記錄資料，請參閱 [Azure 診斷記錄概觀](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 如果您將診斷資料傳送到下列位置：
- **Log Analytics**：您可以使用[網路安全性群組分析](../log-analytics/log-analytics-azure-networking-analytics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-network-security-group-analytics-solution-in-log-analytics
)解決方案來取得增強的見解。 此解決方案能提供 NSG 規則的視覺效果，以根據 MAC 位址允許或拒絕虛擬機器中網路介面的流量。
- **Azure 儲存體帳戶**：資料會寫入至 PT1H.json 檔案。 您可以：
    - 在下列路徑中找到事件記錄：`insights-logs-networksecuritygroupevent/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`
    - 在下列路徑中找到規則計數器記錄：`insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`

## <a name="next-steps"></a>後續步驟

- 深入了解[活動記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (先前稱為稽核或作業記錄)。 預設會針對透過任何一個 Azure 部署模型所建立的 NSG 啟用活動記錄。 若要判斷在 NSG 上已完成哪些作業，請在活動記錄中尋找包含下列資源類型的項目：
    - Microsoft.ClassicNetwork/networkSecurityGroups
    - Microsoft.ClassicNetwork/networkSecurityGroups/securityRules
    - Microsoft.Network/networkSecurityGroups
    - Microsoft.Network/networkSecurityGroups/securityRules
- 若要了解如何記錄診斷資訊以包含每個流程的來源 IP 位址，請參閱 [NSG 流程記錄](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
