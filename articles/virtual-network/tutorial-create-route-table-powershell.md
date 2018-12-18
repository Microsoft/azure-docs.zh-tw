---
title: 路由傳送網路流量 Azure PowerShell | Microsoft Docs
description: 在本文中，了解如何使用 PowerShell 以路由表路由傳送網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 2aca1de567dbd4d37daf7f9dd7c407b669396a47
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38630311"
---
# <a name="route-network-traffic-with-a-route-table-using-powershell"></a>使用 PowerShell 以路由表路由傳送網路流量

根據預設，Azure 會自動路由傳送虛擬網路內所有子網路之間的流量。 您可以建立您自己的路由，以覆寫 Azure 的預設路由。 舉例來說，如果您想要通過網路虛擬設備 (NVA) 路由傳送子網路之間的流量，則建立自訂路由的能力很有幫助。 在本文中，您將了解：

* 建立路由表
* 建立路由
* 建立有多個子網路的虛擬網路
* 建立路由表與子網路的關聯
* 建立會路由傳送流量的 NVA
* 將虛擬機器 (VM) 部署到不同子網路
* 透過 NVA 從一個子網路將流量路由傳送到另一個子網路

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本文會要求使用 Azure PowerShell 模組版本 5.4.1 或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。 

## <a name="create-a-route-table"></a>建立路由表

建立路由表之前，請先使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 來建立資源群組。 下列範例會針對在本文中建立的所有資源，建立名為 myResourceGroup 的資源群組。 

```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

使用 [New-AzureRmRouteTable](/powershell/module/azurerm.network/new-azurermroutetable) 建立路由表。 下列範例會建立名為 myRouteTablePublic 的路由表。

```azurepowershell-interactive
$routeTablePublic = New-AzureRmRouteTable `
  -Name 'myRouteTablePublic' `
  -ResourceGroupName myResourceGroup `
  -location EastUS
```

## <a name="create-a-route"></a>建立路由

藉由使用 [Get-AzureRmRouteTable](/powershell/module/azurerm.network/get-azurermroutetable) 擷取路由表物件來建立路由，使用 [Add-AzureRmRouteConfig](/powershell/module/azurerm.network/add-azurermrouteconfig) 建立路由，然後使用 [Set-AzureRmRouteTable](/powershell/module/azurerm.network/set-azurermroutetable) 將路由組態寫入路由表。 

```azurepowershell-interactive
Get-AzureRmRouteTable `
  -ResourceGroupName "myResourceGroup" `
  -Name "myRouteTablePublic" `
  | Add-AzureRmRouteConfig `
  -Name "ToPrivateSubnet" `
  -AddressPrefix 10.0.1.0/24 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress 10.0.2.4 `
 | Set-AzureRmRouteTable
```

## <a name="associate-a-route-table-to-a-subnet"></a>建立路由表與子網路的關聯

您必須先建立虛擬網路和子網路，才能讓路由表與子網路產生關聯。 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 下列範例會建立名為 myVirtualNetwork 的虛擬網路，位址首碼為 10.0.0.0/16。

```azurepowershell-interactive
$virtualNetwork = New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

藉由使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 建立三個子網路組態，建立三個子網路。 下列範例會針對「公用」、「私人」和「DMZ」子網路建立三個子網路組態：

```azurepowershell-interactive
$subnetConfigPublic = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigPrivate = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigDmz = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name DMZ `
  -AddressPrefix 10.0.2.0/24 `
  -VirtualNetwork $virtualNetwork
```

請使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork) 將子網路組態寫入至虛擬網路，這樣會在虛擬網路中建立子網路：

```azurepowershell-interactive
$virtualNetwork | Set-AzureRmVirtualNetwork
```

使用 [Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig) 讓 myRouteTablePublic 路由表與「公用」子網路產生關聯，然後使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/set-azurermvirtualnetwork) 將子網路組態寫入虛擬網路。

```azurepowershell-interactive
Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $virtualNetwork `
  -Name 'Public' `
  -AddressPrefix 10.0.0.0/24 `
  -RouteTable $routeTablePublic | `
Set-AzureRmVirtualNetwork
```

## <a name="create-an-nva"></a>建立 NVA

NVA 是會執行網路功能的虛擬機器，例如路由、防火牆或 WAN 最佳化。

建立虛擬機器之前，請建立網路介面。

### <a name="create-a-network-interface"></a>建立網路介面

在建立網路介面之前，您必須使用 [Get-AzureRmVirtualNetwork](/powershell/module/azurerm.network/get-azurermvirtualnetwork) 擷取虛擬網路識別碼，然後使用 [Get-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/get-azurermvirtualnetworksubnetconfig) 擷取子網路識別碼。 在已啟用 IP 轉送的 DMZ 子網路中，使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立網路介面：

```azurepowershell-interactive
# Retrieve the virtual network object into a variable.
$virtualNetwork=Get-AzureRmVirtualNetwork `
  -Name myVirtualNetwork `
  -ResourceGroupName myResourceGroup

# Retrieve the subnet configuration into a variable.
$subnetConfigDmz = Get-AzureRmVirtualNetworkSubnetConfig `
  -Name DMZ `
  -VirtualNetwork $virtualNetwork

# Create the network interface.
$nic = New-AzureRmNetworkInterface `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name 'myVmNva' `
  -SubnetId $subnetConfigDmz.Id `
  -EnableIPForwarding
```

### <a name="create-a-vm"></a>建立 VM

若要建立虛擬機器並將現有網路介面與它連結，您必須先使用 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) 建立虛擬機器組態。 組態包含在先前步驟中建立的網路介面。 出現提供使用者名稱和密碼的提示時，選取您想要用來登入虛擬機器的使用者名稱和密碼。 

```azurepowershell-interactive
# Create a credential object.
$cred = Get-Credential -Message "Enter a username and password for the VM."

# Create a VM configuration.
$vmConfig = New-AzureRmVMConfig `
  -VMName 'myVmNva' `
  -VMSize Standard_DS2 | `
  Set-AzureRmVMOperatingSystem -Windows `
    -ComputerName 'myVmNva' `
    -Credential $cred | `
  Set-AzureRmVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
  Add-AzureRmVMNetworkInterface -Id $nic.Id
```

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm)，以虛擬機器組態建立虛擬機器。 下列範例會建立名為 myVmNva 的虛擬機器。 

```azurepowershell-interactive
$vmNva = New-AzureRmVM `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -VM $vmConfig `
  -AsJob
```

`-AsJob` 選項會在背景建立虛擬機器，以便您繼續進行下一步。

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路中建立兩部虛擬機器，您就可以在後續步驟中驗證來自「公用」子網路的流量是否透過網路虛擬設備路由傳送至「私人」子網路。 

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 在「公用」子網路中建立虛擬機器。 下列範例會在 myVirtualNetwork 虛擬網路的「公用」子網路中建立名為 myVmPublic 的虛擬機器。 

```azurepowershell-interactive
New-AzureRmVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Public" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPublic" `
  -AsJob
```

在「私人」子網路中建立虛擬機器。

```azurepowershell-interactive
New-AzureRmVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Private" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPrivate"
```

建立 VM 需要幾分鐘的時間。 在建立虛擬機器且 Azure 將輸出傳回 PowerShell 之前，請勿繼續進行下一個步驟。

## <a name="route-traffic-through-an-nva"></a>透過 NVA 路由傳送流量

請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 來傳回 myVmPrivate 虛擬機器的公用 IP 位址。 以下範例會傳回 myVmPrivate 虛擬機器的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

請從您的本機電腦使用下列命令，來建立與 myVmPrivate 虛擬機器的遠端桌面工作階段。 以上一個命令傳回的 IP 位址取代 `<publicIpAddress>`。

```
mstsc /v:<publicIpAddress>
```

開啟所下載的 RDP 檔案。 如果出現提示，請選取 [連接]。

輸入您在建立虛擬機器時指定的使用者名稱和密碼 (您可能需要選取 [更多選擇]，然後選取 [使用不同的帳戶] 以指定您在建立虛擬機器時輸入的認證)，然後選取 [確定]。 您可能會在登入過程中收到憑證警告。 選取 [是] 以繼續進行連線。 

在稍後步驟中，tracert.exe 命令用於測試路由。 Tracert 會使用網際網路控制訊息通訊協定 (ICMP)，它在通過 Windows 防火牆時會遭到拒絕。 從 myVmPrivate VM 上的 PowerShell 中輸入下列命令，讓 ICMP 通過 Windows 防火牆：

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4
```

雖然本文使用追蹤路由來測試路由，但不建議在生產環境部署中允許 ICMP 通過 Windows 防火牆。

您已在[啟用 IP 轉送](#enable-ip-forwarding)中針對虛擬機器的網路介面啟用在 Azure 內 IP 轉送。 在虛擬機器內，作業系統或在虛擬機器內執行的應用程式也必須能夠轉送網路流量。 在 myVmNva 的作業系統內啟用 IP 轉送。

從 myVmPrivate 虛擬機器的命令提示字元中，使用遠端桌面連線到 myVmNva：

``` 
mstsc /v:myvmnva
```
    
若要在作業系統內啟用 IP 轉送，請從 myVmNva 虛擬機器在 PowerShell 中輸入下列命令：

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
```
    
重新啟動 myVmNva VM，也會中斷與遠端桌面工作階段的連線。

在連線至 myVmPrivate VM 的狀態下，在 myVmNva VM 重新啟動後建立 myVmPublic VM 的遠端桌面工作階段：

``` 
mstsc /v:myVmPublic
```
    
從 myVmPublic VM 上的 PowerShell 中輸入下列命令，讓 ICMP 通過 Windows 防火牆：

```powershell
New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
```

若要測試從 myVmPublic VM 到 myVmPrivate VM 的網路流量路由，請在 myVmPublic VM 上的 PowerShell 中輸入下列命令：

```
tracert myVmPrivate
```

回應如下列範例所示：
    
```
Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
over a maximum of 30 hops:
        
1    <1 ms     *        1 ms  10.0.2.4
2     1 ms     1 ms     1 ms  10.0.1.4
        
Trace complete.
```
      
您可以看到第一個躍點是 10.0.2.4，也就是 NVA 的私人 IP 位址。 第二躍點是 10.0.1.4，也就是 myVmPrivate 虛擬機器的私人 IP 位址。 新增至 myRouteTablePublic 路由表且與「公用」子網路產生關聯的路由，會導致 Azure 透過 NVA 路由傳送流量，而不是直接路由傳送到「私人」子網路。

關閉 myVmPublic 虛擬機器的遠端桌面工作階段，但您仍然與 myVmPrivate 虛擬機器連線。

若要測試從 myVmPrivate VM 到 myVmPublic VM 的網路流量路由，請在 myVmPrivate VM 上的命令提示字元中輸入下列命令：

```
tracert myVmPublic
```

回應如下列範例所示：

```
Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
over a maximum of 30 hops:
    
1     1 ms     1 ms     1 ms  10.0.0.4
   
Trace complete.
```

您可以看到流量是直接從 myVmPrivate 虛擬機器直接路由傳送到 myVmPublic 虛擬機器。 根據預設，Azure 會直接路由傳送子網路之間的流量。

關閉 myVmPrivate 虛擬機器的遠端桌面工作階段。

## <a name="clean-up-resources"></a>清除資源

請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本文中，您已建立路由表並將其與子網路產生關聯。 您已建立簡單的網路虛擬設備，它會將來自公用子網路的流量路由傳送至私人子網路。 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) 部署各種預先設定的網路虛擬設備，這些設備會執行例如防火牆和 WAN 最佳化的網路功能。 若要深入了解路由，請參閱[路由概觀](virtual-networks-udr-overview.md)和[管理路由表](manage-route-table.md)。

雖然您可以在虛擬網路內部署許多 Azure 資源，但是某些 Azure PaaS 服務的資源無法部署到虛擬網路中。 您仍可將某些 Azure PaaS 服務的資源存取，限制為僅來自虛擬網路子網路的流量。 若要深入了解，請參閱[限制對 PaaS 資源的網路存取](tutorial-restrict-network-access-to-resources-powershell.md)。
