---
title: 診斷虛擬機器網路流量篩選問題 - 快速入門 - Azure PowerShell | Microsoft Docs
description: 在本快速入門中，您將了解如何使用 Azure 網路監看員的 IP 流量驗證功能來診斷虛擬機器網路流量篩選問題。
services: network-watcher
documentationcenter: network-watcher
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: d98a804961defc80bebe3e3a838dd229c23044bc
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38704182"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-powershell"></a>快速入門：診斷虛擬機器網路流量篩選問題 - Azure PowerShell

本快速入門中，您可部署虛擬機器 (VM)，然後檢查送至 IP 位址和 URL 以及來自 IP 位址的通訊。 您可判斷通訊失敗的原因及其解決方式。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在進行此快速入門時，您必須使用 AzureRM PowerShell 模組 5.4.1 版或更新版本。 若要尋找已安裝的版本，請執行 ` Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-vm"></a>建立 VM

您必須先建立資源群組來包含 VM，才能建立 VM。 使用 [New-AzureRmResourceGroup](/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup)建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 執行此步驟時，系統會提示您輸入認證。 您輸入的值會設定為 VM 的使用者名稱和密碼。

```azurepowershell-interactive
$vM = New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVm" `
    -Location "East US"
```

建立 VM 需要幾分鐘的時間。 在建立 VM 且 PowerShell 傳回輸出之前，請勿繼續進行其餘步驟。

## <a name="test-network-communication"></a>測試網路通訊

若要測試網路監看員的網路通訊，您必須先在欲測試的 VM 所在地區啟用網路監看員，然後使用網路監看員的 IP 流量驗證功能來測試通訊。

### <a name="enable-network-watcher"></a>啟用網路監看員

如果您已經在美國東部區域啟用網路監看員，請使用 [New-AzureRmNetworkWatcher](/powershell/module/azurerm.network/get-azurermnetworkwatcher) 來擷取網路監看員。 下列範例會擷取名為 *NetworkWatcher_eastus* 的現有網路監看員，其位於 *NetworkWatcherRG* 資源群組中：

```azurepowershell-interactive
$networkWatcher = Get-AzureRmNetworkWatcher `
  -Name NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG
```

如果您尚未在美國東部地區啟用網路監看員，請使用 [New-AzureRmNetworkWatcher](/powershell/module/azurerm.network/new-azurermnetworkwatcher) 在美國東部地區建立網路監看員：

```azurepowershell-interactive
$networkWatcher = New-AzureRmNetworkWatcher `
  -Name "NetworkWatcher_eastus" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "East US"
```

### <a name="use-ip-flow-verify"></a>使用 IP 流量驗證

根據預設，當您建立 VM 時，Azure 會允許及拒絕 VM 的雙向網路流量。 您稍後可能會覆寫 Azure 的預設值，並允許或拒絕其他類型的流量。 若要測試允許或拒絕送至不同目的地的流量和來自來源 IP 位址的流量，請使用 [Test-AzureRmNetworkWatcherIPFlow](/powershell/module/azurerm.network/test-azurermnetworkwatcheripflow) 命令。

測試從 VM 輸出至 www.bing.com 的其中一個 IP 位址的通訊：

```azurepowershell-interactive
Test-AzureRmNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 60000 `
  -RemoteIPAddress 13.107.21.200 `
  -RemotePort 80
```

幾秒之後，傳回的結果會通知您，名為 **AllowInternetOutbound** 的安全性規則允許存取。

測試從 VM 輸出至 172.31.0.100 的通訊：

```azurepowershell-interactive
Test-AzureRmNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 60000 `
  -RemoteIPAddress 172.31.0.100 `
  -RemotePort 80
```

傳回的結果會通知您，名為 **DefaultOutboundDenyAll** 的安全性規則拒絕存取。

測試從 172.31.0.100 輸入至 VM 的通訊：

```azurepowershell-interactive
Test-AzureRmNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Inbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 80 `
  -RemoteIPAddress 172.31.0.100 `
  -RemotePort 60000
```

傳回的結果會通知您，因為名為 **DefaultInboundDenyAll** 的安全性規則而拒絕存取。 既然您已知道哪些安全性規則允許或拒絕 VM 的流量，即可判斷如何解決問題。

## <a name="view-details-of-a-security-rule"></a>檢視安全性規則的詳細資料

若要判斷為何[測試網路通訊](#test-network-communication)中的規則允許或禁止通訊，請使用 [Get-AzureRmEffectiveNetworkSecurityGroup](/powershell/module/azurerm.network/get-azurermeffectivenetworksecuritygroup) 檢閱網路介面的有效安全性規則：

```azurepowershell-interactive
Get-AzureRmEffectiveNetworkSecurityGroup `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup
```

對於在[使用 IP 流量驗證](#use-ip-flow-verify) 中允許對外存取 www.bing.com 的 **AllowInternetOutbound** 規則，傳回的輸出包含下列文字：

```powershell
{
  "Name":
"defaultSecurityRules/AllowInternetOutBound",
  "Protocol": "All",
  "SourcePortRange": [
    "0-65535"
  ],
  "DestinationPortRange": [
    "0-65535"
  ],
  "SourceAddressPrefix": [
    "0.0.0.0/0"
  ],
  "DestinationAddressPrefix": [
    "Internet"
  ],
  "ExpandedSourceAddressPrefix": [],
  "ExpandedDestinationAddressPrefix": [
    "1.0.0.0/8",
    "2.0.0.0/7",
    "4.0.0.0/6",
    "8.0.0.0/7",
    "11.0.0.0/8",
    "12.0.0.0/6",
    ...
    ],
    "Access": "Allow",
    "Priority": 65001,
    "Direction": "Outbound"
  },
```

您可以在輸出中看到 **DestinationAddressPrefix** 為 **Internet**。 雖然並不清楚 13.107.21.200 (您在[使用 IP 流量驗證](#use-ip-flow-verify)中測試的位址) 與 **Internet** 有何關連。 您會看到有幾個位址首碼列在 **ExpandedDestinationAddressPrefix** 底下。 清單中有一個前置詞是 **12.0.0.0/6**，其中包含 IP 位址的 12.0.0.1-15.255.255.254 範圍。 因為 13.107.21.200 在該位址範圍內，所以 **AllowInternetOutBound** 規則允許輸出流量。 此外，`Get-AzureRmEffectiveNetworkSecurityGroup` 所傳回的輸出中並未列出任何會覆寫此規則的較高**優先順序** (數字較小) 規則。 若要拒絕送至 13.107.21.200 的輸出通訊，您可以新增具有較高優先順序的安全性規則，以拒絕連接埠 80 輸出至此 IP 位址。

當您在[使用 IP 流量驗證](#use-ip-flow-verify)中執行 `Test-AzureRmNetworkWatcherIPFlow` 命令來測試輸出至 172.131.0.100 的通訊時，輸出會告知您 **DefaultOutboundDenyAll** 規則拒絕通訊。 **DefaultOutboundDenyAll** 規則等同於 `Get-AzureRmEffectiveNetworkSecurityGroup` 命令的下列輸出中所列的 **DenyAllOutBound** 規則：

```powershell
{
"Name": "defaultSecurityRules/DenyAllOutBound",
"Protocol": "All",
"SourcePortRange": [
  "0-65535"
],
"DestinationPortRange": [
  "0-65535"
],
"SourceAddressPrefix": [
  "0.0.0.0/0"
],
"DestinationAddressPrefix": [
  "0.0.0.0/0"
],
"ExpandedSourceAddressPrefix": [],
"ExpandedDestinationAddressPrefix": [],
"Access": "Deny",
"Priority": 65500,
"Direction": "Outbound"
}
```

此規則列出 **0.0.0.0/0** 作為 **DestinationAddressPrefix**。 此規則會拒絕輸出至 172.131.0.100 的通訊，因為此位址不在 `Get-AzureRmEffectiveNetworkSecurityGroup` 命令輸出中任何其他輸出規則的 **DestinationAddressPrefix** 內。 若要允許輸出通訊，您可以新增較高優先順序的安全性規則，以允許輸出至位於 172.131.0.100 之通訊埠 80 的流量。

當您在[使用 IP 流量驗證](#use-ip-flow-verify)中執行 `Test-AzureRmNetworkWatcherIPFlow` 命令來測試從 172.131.0.100 輸入的通訊時，輸出會告知您 **DefaultInboundDenyAll** 規則拒絕通訊。 **DefaultInboundDenyAll** 規則等同於 `Get-AzureRmEffectiveNetworkSecurityGroup` 命令的下列輸出中所列的 **DenyAllInBound** 規則：

```powershell
{
"Name": "defaultSecurityRules/DenyAllInBound",
"Protocol": "All",
"SourcePortRange": [
  "0-65535"
],
"DestinationPortRange": [
  "0-65535"
],
"SourceAddressPrefix": [
  "0.0.0.0/0"
],
"DestinationAddressPrefix": [
  "0.0.0.0/0"
],
"ExpandedSourceAddressPrefix": [],
"ExpandedDestinationAddressPrefix": [],
"Access": "Deny",
"Priority": 65500,
"Direction": "Inbound"
},
```

如輸出所示，系統會套用 **DenyAllInBound** 規則，因為 `Get-AzureRmEffectiveNetworkSecurityGroup` 命令的輸出中不存在其他較高優先順序的規則，可允許連接埠 80 從 172.131.0.100 輸入至 VM。 若要允許輸入通訊，您可以新增較高優先順序的安全性規則，以允許通訊埠 80 從 172.131.0.100 輸入。

本快速入門中的檢查已測試 Azure 組態。 如果檢查傳回預期的結果，但仍有網路問題，請確定您的 VM 與您正在通訊的端點之間沒有防火牆，而且您的 VM 中的作業系統沒有允許或拒絕通訊的防火牆。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立 VM 並診斷出輸入和輸出網路流量篩選條件。 您已了解網路安全性群組規則允許或拒絕 VM 的雙向流量。 深入了解[安全性規則](../virtual-network/security-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)以及如何[建立安全性規則](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule)。

即使採用適當的網路流量篩選條件，VM 的通訊仍可能因為路由組態而失敗。 若要了解如何診斷 VM 網路路由問題，請參閱[診斷 VM 路由問題](diagnose-vm-network-routing-problem-powershell.md)，或若要透過一項工具來診斷輸出路由、延遲和流量篩選問題，請參閱[連線疑難排解](network-watcher-connectivity-powershell.md)。