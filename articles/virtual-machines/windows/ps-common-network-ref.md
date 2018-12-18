---
title: Azure 虛擬網路的常用 PowerShell 命令 | Microsoft Docs
description: 常用 PowerShell 命令，可讓您開始為 VM 建立虛擬網路與其相關聯的資源。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 56e1a73c-8299-4996-bd03-f74585caa1dc
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: cynthn
ms.openlocfilehash: a5b3f84c27a0a5f6458808940b16a9001097b30b
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31526697"
---
# <a name="common-powershell-commands-for-azure-virtual-networks"></a>Azure 虛擬網路的常用 PowerShell 命令

如果您想要建立虛擬機器，您需要建立 [虛擬網路](../../virtual-network/virtual-networks-overview.md) 或了解可以加入 VM 的現有虛擬網路。 一般而言，當您建立 VM 時，您也需要考慮建立本文中所述的資源。

如需如何安裝最新版 Azure PowerShell、選取訂用帳戶，以及登入帳戶的相關資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

如果您要執行本文中的多個命令，有些變數可能會相當實用：

- $location - 網路資源的位置。 您可以使用 [Get-AzureRmLocation](https://docs.microsoft.com/powershell/module/azurerm.resources/get-azurermlocation) 來尋找適合您的[地理區域](https://azure.microsoft.com/regions/)。
- $myResourceGroup - 網路資源所在的資源群組名稱。

## <a name="create-network-resources"></a>建立網路資源

| Task | 命令 |
| ---- | ------- |
| 建立子網路組態 |$subnet1 = [New-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) -Name "mySubnet1" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet2" -AddressPrefix XX.X.X.X/XX<BR><BR>一般網路可能有[網際網路面向負載平衡器](../../load-balancer/load-balancer-internet-overview.md)的子網路和[內部負載平衡器](../../load-balancer/load-balancer-internal-overview.md)的不同子網路。 |
| 建立虛擬網路 |$vnet = [New-AzureRmVirtualNetwork](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup -Location $location -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2 |
| 唯一網域名稱的測試 |[Test-AzureRmDnsAvailability](https://docs.microsoft.com/powershell/module/azurerm.network/test-azurermdnsavailability) -DomainNameLabel "myDNS" -Location $location<BR><BR>您可以指定[公用 IP 資源](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)的 DNS 網域名稱，以建立 domainname.location.cloudapp.azure.com 與 Azure 受控 DNS 伺服器中的公用 IP 位址的對應。 名稱只能包含字母、數字和連字號。 第一個和最後一個字元必須是字母或數字，而網域名稱必須是其 Azure 位置內唯一的。 如果傳回 **True** ，表示您提出的名稱是全域唯一的。 |
| 建立公用 IP 位址 |$pip = [New-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermpublicipaddress) -Name "myPublicIp" -ResourceGroupName $myResourceGroup -DomainNameLabel "myDNS" -Location $location -AllocationMethod Dynamic<BR><BR>公用 IP 位址會使用您先前測試並由負載平衡器前端組態使用的網域名稱。 |
| 建立前端 IP 組態 |$frontendIP = [New-AzureRmLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancerfrontendipconfig) -Name "myFrontendIP" -PublicIpAddress $pip<BR><BR>前端組態包含您先前針對連入網路流量建立的公用 IP 位址。 |
| 建立後端位址集區 |$beAddressPool = [New-AzureRmLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig) -Name "myBackendAddressPool"<BR><BR>提供可透過網路介面存取的負載平衡器後端內部位址。 |
| 建立探查 |$healthProbe = [New-AzureRmLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancerprobeconfig) -Name "myProbe" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>包含健康情況探查，可用來檢查後端位址集區中虛擬機器執行個體的可用性。 |
| 建立負載平衡規則 |$lbRule = [New-AzureRmLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancerruleconfig) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>包含將負載平衡器上的公用連接埠指派至後端位址集區中的連接埠的規則。 |
| 建立輸入 NAT 規則 |$inboundNATRule = [New-AzureRmLoadBalancerInboundNatRuleConfig](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancerinboundnatruleconfig) -Name "myInboundRule1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>包含將負載平衡器上的公用連接埠對應至後端位址集區中特定虛擬機器的連接埠的規則。 |
| 建立負載平衡器 |$loadBalancer = [New-AzureRmLoadBalancer](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermloadbalancer) -ResourceGroupName $myResourceGroup -Name "myLoadBalancer" -Location $location -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe |
| 建立網路介面 |$nic1= [New-AzureRmNetworkInterface](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermnetworkinterface) -ResourceGroupName $myResourceGroup -Name "myNIC" -Location $location -PrivateIpAddress XX.X.X.X -Subnet $subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>建立一個網路介面，其使用您先前建立的公用 IP 位址和虛擬網路子網路。 |

## <a name="get-information-about-network-resources"></a>取得關於網路資源的資訊

| Task | 命令 |
| ---- | ------- |
| 列出虛擬網路 |[Get-AzureRmVirtualNetwork](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermvirtualnetwork) -ResourceGroupName $myResourceGroup<BR><BR>列出資源群組中的所有虛擬網路。 |
| 取得虛擬網路的相關資訊 |Get-AzureRmVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup |
| 列出虛擬網路中的子網路 |Get-AzureRmVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup &#124; Select Subnets |
| 取得子網路的相關資訊 |[Get-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermvirtualnetworksubnetconfig) -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>取得指定虛擬網路中的子網路相關資訊。 $Vnet 值代表 Get-AzureRmVirtualNetwork 所傳回的物件。 |
| 列出 IP 位址 |[Get-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermpublicipaddress) -ResourceGroupName $myResourceGroup<BR><BR>列出資源群組中的公用 IP 位址。 |
| 列出負載平衡器 |[Get-AzureRmLoadBalancer](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermloadbalancer) -ResourceGroupName $myResourceGroup<BR><BR>列出資源群組中的所有負載平衡器。 |
| 列出網路介面 |[Get-AzureRmNetworkInterface](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermnetworkinterface) -ResourceGroupName $myResourceGroup<BR><BR>列出資源群組中的所有網路介面。 |
| 取得網路介面的相關資訊 |Get-AzureRmNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>取得特定網路介面的相關資訊。 |
| 取得網路介面的 IP 組態 |[Get-AzureRmNetworkInterfaceIPConfig](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermnetworkinterfaceipconfig) -Name "myNICIP" -NetworkInterface $nic<BR><BR>取得指定網路介面的 IP 組態相關資訊。 $nic 值代表 Get-AzureRmNetworkInterface 所傳回的物件。 |

## <a name="manage-network-resources"></a>管理網路資源

| Task | 命令 |
| ---- | ------- |
| 將子網路新增至虛擬網路 |[Add-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/azurerm.network/add-azurermvirtualnetworksubnetconfig) -AddressPrefix XX.X.X.X/XX -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>可將子網路新增現有虛擬網路。 $Vnet 值代表 Get-AzureRmVirtualNetwork 所傳回的物件。 |
| 刪除虛擬網路 |[Remove-AzureRmVirtualNetwork](https://docs.microsoft.com/powershell/module/azurerm.network/remove-azurermvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup<BR><BR>從資源群組移除指定的虛擬網路。 |
| 刪除網路介面 |[Remove-AzureRmNetworkInterface](https://docs.microsoft.com/powershell/module/azurerm.network/remove-azurermnetworkinterface) -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>從資源群組移除指定的網路介面。 |
| 刪除負載平衡器 |[Remove-AzureRmLoadBalancer](https://docs.microsoft.com/powershell/module/azurerm.network/remove-azurermloadbalancer) -Name "myLoadBalancer" -ResourceGroupName $myResourceGroup<BR><BR>從資源群組移除指定的負載平衡器。 |
| 刪除公用 IP 位址 |[Remove-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/module/azurerm.network/remove-azurermpublicipaddress)-Name "myIPAddress" -ResourceGroupName $myResourceGroup<BR><BR>從資源群組移除指定的公用 IP 位址。 |

## <a name="next-steps"></a>後續步驟
* 使用您剛才[建立 VM](../virtual-machines-windows-ps-create.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 時所建立的網路介面。
* 深入了解如何 [建立具有多個網路介面的 VM](../../virtual-network/virtual-network-deploy-multinic-classic-ps.md)。

