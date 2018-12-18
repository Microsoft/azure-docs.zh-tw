---
title: 篩選網路流量 - Azure PowerShell | Microsoft Docs
description: 在本文中，您將了解如何使用 PowerShell 透過網路安全性群組篩選傳至子網路的網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: a37ed044275abfca23c83e7d17012b9afdc78f6c
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37906454"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-powershell"></a>使用 PowerShell 透過網路安全性群組篩選網路流量

您可以透過網路安全性群組篩選輸入虛擬網路子網路和從中輸出的網路流量。 網路安全性群組包含可依 IP 位址、連接埠和通訊協定篩選網路流量的安全性規則。 安全性規則會套用至子網路中部署的資源。 在本文中，您將了解：

* 建立網路安全性群組和安全性規則
* 建立虛擬網路，並將網路安全性群組與子網路產生關聯
* 將虛擬機器 (VM) 部署至子網路中
* 測試流量篩選

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本文會要求使用 Azure PowerShell 模組版本 6.2.1 或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-network-security-group"></a>建立網路安全性群組

網路安全性群組包含安全性規則。 安全性規則可指定來源和目的地。 來源和目的地可以是應用程式安全性群組。

### <a name="create-application-security-groups"></a>建立應用程式安全性群組

首先，請使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 為本文中建立的所有資源建立資源群組。 下列範例會在 *eastus* 位置建立資源群組： 


```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

使用 [New-AzureRmApplicationSecurityGroup](/powershell/module/azurerm.network/new-azurermapplicationsecuritygroup) 建立應用程式安全性群組。 應用程式安全性群組可讓您將具有類似連接埠篩選需求的伺服器分組在一起。 下列範例會建立兩個應用程式安全性群組。

```azurepowershell-interactive
$webAsg = New-AzureRmApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgWebServers `
  -Location eastus

$mgmtAsg = New-AzureRmApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgMgmtServers `
  -Location eastus
```

### <a name="create-security-rules"></a>建立安全性規則

使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) 建立安全性規則。 下列範例會建立允許透過連接埠 80 和 443 從網際網路將流量輸入 *myWebServers* 應用程式安全性群組的規則：

```azurepowershell-interactive
$webRule = New-AzureRmNetworkSecurityRuleConfig `
  -Name "Allow-Web-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $webAsg.id `
  -DestinationPortRange 80,443

The following example creates a rule that allows traffic inbound from the internet to the *myMgmtServers* application security group over port 3389:

$mgmtRule = New-AzureRmNetworkSecurityRuleConfig `
  -Name "Allow-RDP-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 110 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $mgmtAsg.id `
  -DestinationPortRange 3389
```

在本文中，會針對 *myAsgMgmtServers* VM 將 RDP (連接埠 3389) 公開至網際網路。 在生產環境中則不應將連接埠 3389 公開至網際網路，而是建議您使用 [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或[私人](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)網路連線連接到您想要管理的 Azure 資源。

### <a name="create-a-network-security-group"></a>建立網路安全性群組

使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) 建立網路安全性群組。 下列範例會建立名為 myNsg 的網路安全性群組： 

```powershell-interactive
$nsg = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myNsg `
  -SecurityRules $webRule,$mgmtRule
```

## <a name="create-a-virtual-network"></a>建立虛擬網路

使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 下列範例會建立名為 myVirtualNetwork 的虛擬網路：

```azurepowershell-interactive
$virtualNetwork = New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 建立子網路組態，然後使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork) 將子網路組態寫入至虛擬網路。 下列範例會將名為 mySubnet 的子網路新增至虛擬網路，並將其與 myNsg 網路安全性群組產生關聯：

```azurepowershell-interactive
Add-AzureRmVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -VirtualNetwork $virtualNetwork `
  -AddressPrefix "10.0.2.0/24" `
  -NetworkSecurityGroup $nsg
$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="create-virtual-machines"></a>建立虛擬機器

建立 VM 之前，請使用 [Get-AzureRmVirtualNetwork](/powershell/module/azurerm.network/get-azurermvirtualnetwork) 擷取具有子網路的虛擬網路物件：

```powershell-interactive
$virtualNetwork = Get-AzureRmVirtualNetwork `
 -Name myVirtualNetwork `
 -Resourcegroupname myResourceGroup
```
使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 建立每個 VM 的公用 IP 位址：

```powershell-interactive
$publicIpWeb = New-AzureRmPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmWeb

$publicIpMgmt = New-AzureRmPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmMgmt
```

使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立兩個網路介面，並將公用 IP 位址指派給網路介面。 下列範例會建立網路介面、將其與 myVmWeb 公用 IP 位址產生關聯，並使其成為 myAsgWebServers 應用程式安全性群組的成員：

```powershell-interactive
$webNic = New-AzureRmNetworkInterface `
  -Location eastus `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $webAsg.Id `
  -PublicIpAddressId $publicIpWeb.Id
```

下列範例會建立網路介面、將其與 myVmMgmt 公用 IP 位址產生關聯，並使其成為 myAsgMgmtServers 應用程式安全性群組的成員：

```powershell-interactive
$mgmtNic = New-AzureRmNetworkInterface `
  -Location eastus `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $mgmtAsg.Id `
  -PublicIpAddressId $publicIpMgmt.Id
```

在虛擬網路中建立兩個 VM，以便在後續步驟中驗證流量篩選。 

使用 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) 建立 VM 組態，然後使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 下列範例會建立將作為 Web 伺服器的 VM。 `-AsJob` 選項會在背景建立虛擬機器，以便您繼續進行下一個步驟： 

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

$webVmConfig = New-AzureRmVMConfig `
  -VMName myVmWeb `
  -VMSize Standard_DS1_V2 | `
Set-AzureRmVMOperatingSystem -Windows `
  -ComputerName myVmWeb `
  -Credential $cred | `
Set-AzureRmVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzureRmVMNetworkInterface `
  -Id $webNic.Id
New-AzureRmVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $webVmConfig `
  -AsJob
```

建立作為管理伺服器的 VM：

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create the web server virtual machine configuration and virtual machine.
$mgmtVmConfig = New-AzureRmVMConfig `
  -VMName myVmMgmt `
  -VMSize Standard_DS1_V2 | `
Set-AzureRmVMOperatingSystem -Windows `
  -ComputerName myVmMgmt `
  -Credential $cred | `
Set-AzureRmVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzureRmVMNetworkInterface `
  -Id $mgmtNic.Id
New-AzureRmVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $mgmtVmConfig
```

建立虛擬機器需要幾分鐘的時間。 在 VM 建立完成前，請勿繼續進行下一個步驟。

## <a name="test-traffic-filters"></a>測試流量篩選

請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 來傳回 VM 的公用 IP 位址。 以下範例會傳回 myVmMgmt VM 的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

請從您的本機電腦使用下列命令，建立 myVmMgmt VM 的遠端桌面工作階段。 以上一個命令傳回的 IP 位址取代 `<publicIpAddress>`。

```
mstsc /v:<publicIpAddress>
```

開啟所下載的 RDP 檔案。 如果出現提示，請選取 [連接]。

輸入您在建立虛擬機器時指定的使用者名稱和密碼 (您可能需要選取 [更多選擇]，然後選取 [使用不同的帳戶] 以指定您在建立虛擬機器時輸入的認證)，然後選取 [確定]。 您可能會在登入過程中收到憑證警告。 選取 [是] 以繼續進行連線。 
   
由於允許連接埠 3389 從網際網路將流量輸入連結至 myVmMgmt VM 的網路介面所在的 myAsgMgmtServers 應用程式安全性群組，因此連線會成功。

請從 myVmMgmt VM 使用 PowerShell 的下列命令，建立 myVmWeb VM 的遠端桌面工作階段：

``` 
mstsc /v:myvmWeb
```

由於每個網路安全性群組中的預設安全性規則允許透過所有連接埠在虛擬網路內的所有 IP 位址之間傳輸流量，因此連線會成功。 您無法建立從網際網路到 myVmWeb VM 的遠端桌面連線，因為 myAsgWebServers 的安全性規則不允許從網際網路的連接埠 3389 輸入。

請從 PowerShell 使用下列命令在 myVmWeb VM 上安裝 Microsoft IIS：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成 IIS 安裝之後，請將 myVmWeb VM 中斷連線，以留在 myVmMgmt VM 遠端桌面連線中。 若要檢視 IIS 的歡迎使用畫面，請開啟網際網路瀏覽器，並瀏覽至 http://myVmWeb。

從 myVmMgmt VM 中斷連線。

在您的電腦上，從 PowerShell 輸入下列命令，以擷取 myVmWeb 伺服器的公用 IP 位址的：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

若要確認您可以從 Azure 外部存取 myVmWeb Web 伺服器，請在電腦上開啟網際網路瀏覽器，並瀏覽至 `http://<public-ip-address-from-previous-step>`。 由於允許連接埠 80 從網際網路將流量輸入連結至 myVmWeb VM 的網路介面所在的 myAsgWebServers 應用程式安全性群組，因此連線會成功。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本文中，您已建立網路安全性群組，並將其與虛擬網路子網路產生關聯。 若要深入了解網路安全性群組，請參閱[網路安全性群組概觀](security-overview.md)和[管理網路安全性群組](manage-network-security-group.md)。

Azure 依預設會路由傳送子網路之間的流量。 您可以改採其他方式，例如，透過作為防火牆的 VM 路由傳送子網路之間的流量。 Azure 依預設會路由傳送子網路之間的流量。 您可以改採其他方式，例如，透過作為防火牆的 VM 路由傳送子網路之間的流量。 若要了解操作方式，請參閱[建立路由表](tutorial-create-route-table-powershell.md)。