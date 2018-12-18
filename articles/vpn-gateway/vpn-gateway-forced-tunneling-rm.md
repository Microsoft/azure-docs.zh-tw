---
title: 設定 Azure 站對站連線的強制通道：Resource Manager | Microsoft Docs
description: 如何重新導向或「強制」所有網際網路繫結流量回到內部部署位置。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: cbe58db8-b598-4c9f-ac88-62c865eb8721
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2018
ms.author: cherylmc
ms.openlocfilehash: 00330f49d4acc9bd2d720a60b743b78c86b08f86
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38308147"
---
# <a name="configure-forced-tunneling-using-the-azure-resource-manager-deployment-model"></a>使用 Azure Resource Manager 部署模型設定強制通道

強制通道可讓您透過站對站 VPN 通道，重新導向或「強制」所有網際網路繫結流量傳回內部部署位置，以便進行檢查和稽核。 這是多數企業 IT 原則的重要安全性需求。 若不使用強制通道處理，則來自您 Azure 中 VM 的網際網路繫結流量一律會從 Azure 網路基礎結構直接向外周遊到網際網路，而您無法選擇檢查或稽核流量。 未經授權的網際網路存取可能會導致資訊洩漏或其他類型的安全性漏洞。

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)] 

本文會引導您為使用 Resource Manager 部署模型建立的虛擬網路設定強制通道。 強制通道可使用 PowerShell 設定，而非透過入口網站。 如果想要設定傳統部署模型的強制通道，請從下列下拉式清單選取傳統文章：

> [!div class="op_single_selector"]
> * [PowerShell - 傳統](vpn-gateway-about-forced-tunneling.md)
> * [PowerShell - Resource Manager](vpn-gateway-forced-tunneling-rm.md)
> 
> 

## <a name="about-forced-tunneling"></a>有關強制通道

下圖說明如何使強制通道正常運作。 

![強制通道](./media/vpn-gateway-forced-tunneling-rm/forced-tunnel.png)

在上述範例中，前端子網路不會使用強制通道。 前端子網路中的工作負載可以直接從網際網路繼續接受並回應客戶要求。 中間層和後端的子網路會使用強制通道。 任何從這兩個子網路到網際網路的輸出連接會強制或重新導向回 S2S VPN 通道的其中一個內部部署網站。

這可讓您在 Azure 中限制並檢查來自虛擬機器或雲端服務的網際網路存取，同時繼續啟用您所需的多層式服務架構。 如果虛擬網路中沒有任何網際網路對向工作負載，您也可以將強制通道處理套用至整個虛擬網路。

## <a name="requirements-and-considerations"></a>需求和考量

Azure 中的強制通道處理會透過虛擬網路使用者定義的路由進行設定。 將流量重新導向至在內部部署網站時，會表示為至 Azure VPN 閘道的「預設路由」。 如需有關使用者定義路由和虛擬網路的詳細資訊，請參閱[使用者定義路由和 IP 轉送](../virtual-network/virtual-networks-udr-overview.md)。

* 每個虛擬網路的子網路皆有內建的系統路由表。 系統路由表具有下列 3 個路由群組：
  
  * **本機 VNet 路由：** 直接連線到相同虛擬網路中的目的地 VM。
  * **內部部署路由：** 連線到 Azure VPN 閘道。
  * **預設路由：** 直接連線到網際網路。 系統將會捨棄尚未由前兩個路由涵蓋之私人 IP 位址目的地的封包。
* 此程序使用「使用者定義的路由 (UDR)」建立路由表以新增預設路由，然後將路由表關聯至您的 VNet 子網路，以啟用那些子網路上的強制通道處理。
* 強制通道必須與具有路由型 VPN 閘道的 VNet 相關聯。 您需要在連接到虛擬網路的內部部署本機網站間設定「預設網站」。 此外，內部部署 VPN 裝置必須使用 0.0.0.0/0 設定為流量選取器。 
* ExpressRoute 強制通道不會透過這項機制進行設定，相反地，將由透過 ExpressRoute BGP 對等互連工作階段的廣告預設路由進行啟用。 如需詳細資訊，請參閱 [ExpressRoute 文件](https://azure.microsoft.com/documentation/services/expressroute/)。

## <a name="configuration-overview"></a>組態概觀

下列程序可協助您建立資源群組和 VNet。 然後您將建立 VPN 閘道，並設定強制通道。 在此程序中，'MultiTier-VNet' 虛擬網路具有三個子網路：「前端」、「中層」和「後端」，包含四個跨單位連線：DefaultSiteHQ 和三個「分支」。

程序步驟會將 'DefaultSiteHQ' 設定為強制通道處理的預設網站連線，並設定「中層」和「後端」子網路以使用強制通道處理。

## <a name="before"></a>開始之前

安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

> [!IMPORTANT]
> 您必須安裝最新版的 PowerShell Cmdlet。 否則，在執行某些 Cmdlet 時可能會收到驗證錯誤。
>
>

### <a name="to-log-in"></a>登入

[!INCLUDE [To log in](../../includes/vpn-gateway-ps-login-include.md)]

## <a name="configure-forced-tunneling"></a>設定強制通道

> [!NOTE]
> 您可能會看到指出「此 Cmdlet 的輸出物件類型將會在未來版本中修改」的警告。 這是預期的行為，您可以放心地忽略這些警告。
>
>


1. 建立資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name 'ForcedTunneling' -Location 'North Europe'
  ```
2. 建立虛擬網路並指定子網路。

  ```powershell 
  $s1 = New-AzureRmVirtualNetworkSubnetConfig -Name "Frontend" -AddressPrefix "10.1.0.0/24"
  $s2 = New-AzureRmVirtualNetworkSubnetConfig -Name "Midtier" -AddressPrefix "10.1.1.0/24"
  $s3 = New-AzureRmVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.1.2.0/24"
  $s4 = New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "10.1.200.0/28"
  $vnet = New-AzureRmVirtualNetwork -Name "MultiTier-VNet" -Location "North Europe" -ResourceGroupName "ForcedTunneling" -AddressPrefix "10.1.0.0/16" -Subnet $s1,$s2,$s3,$s4
  ```
3. 建立區域網路閘道。

  ```powershell
  $lng1 = New-AzureRmLocalNetworkGateway -Name "DefaultSiteHQ" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -GatewayIpAddress "111.111.111.111" -AddressPrefix "192.168.1.0/24"
  $lng2 = New-AzureRmLocalNetworkGateway -Name "Branch1" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -GatewayIpAddress "111.111.111.112" -AddressPrefix "192.168.2.0/24"
  $lng3 = New-AzureRmLocalNetworkGateway -Name "Branch2" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -GatewayIpAddress "111.111.111.113" -AddressPrefix "192.168.3.0/24"
  $lng4 = New-AzureRmLocalNetworkGateway -Name "Branch3" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -GatewayIpAddress "111.111.111.114" -AddressPrefix "192.168.4.0/24"
  ```
4. 建立路由表和路由規則。

  ```powershell
  New-AzureRmRouteTable –Name "MyRouteTable" -ResourceGroupName "ForcedTunneling" –Location "North Europe"
  $rt = Get-AzureRmRouteTable –Name "MyRouteTable" -ResourceGroupName "ForcedTunneling" 
  Add-AzureRmRouteConfig -Name "DefaultRoute" -AddressPrefix "0.0.0.0/0" -NextHopType VirtualNetworkGateway -RouteTable $rt
  Set-AzureRmRouteTable -RouteTable $rt
  ```
5. 建立路由表與中間層和後端子網路的關聯。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name "MultiTier-Vnet" -ResourceGroupName "ForcedTunneling"
  Set-AzureRmVirtualNetworkSubnetConfig -Name "MidTier" -VirtualNetwork $vnet -AddressPrefix "10.1.1.0/24" -RouteTable $rt
  Set-AzureRmVirtualNetworkSubnetConfig -Name "Backend" -VirtualNetwork $vnet -AddressPrefix "10.1.2.0/24" -RouteTable $rt
  Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```
6. 建立虛擬網路閘道。 此步驟需要一些時間才能完成，有時需要 45 分鐘或更久，因為您將建立及設定閘道。 如果您看到有關 GatewaySKU 值的 ValidateSet 錯誤，請確認您已安裝[最新版的 PowerShell Cmdlet](#before)。 最新版的 PowerShell Cmdlet 包含最新閘道 SKU 已經過驗證的新值。

  ```powershell
  $pip = New-AzureRmPublicIpAddress -Name "GatewayIP" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -AllocationMethod Dynamic
  $gwsubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
  $ipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name "gwIpConfig" -SubnetId $gwsubnet.Id -PublicIpAddressId $pip.Id
  New-AzureRmVirtualNetworkGateway -Name "Gateway1" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -IpConfigurations $ipconfig -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $false
  ```
7. 將預設網站指派給虛擬網路閘道。 **-GatewayDefaultSite** 是可讓強制路由組態得以運作的 Cmdlet 參數，因此請務必正確地進行此設定。 

  ```powershell
  $LocalGateway = Get-AzureRmLocalNetworkGateway -Name "DefaultSiteHQ" -ResourceGroupName "ForcedTunneling"
  $VirtualGateway = Get-AzureRmVirtualNetworkGateway -Name "Gateway1" -ResourceGroupName "ForcedTunneling"
  Set-AzureRmVirtualNetworkGatewayDefaultSite -GatewayDefaultSite $LocalGateway -VirtualNetworkGateway $VirtualGateway
  ```
8. 建立站對站 VPN 連線。

  ```powershell
  $gateway = Get-AzureRmVirtualNetworkGateway -Name "Gateway1" -ResourceGroupName "ForcedTunneling"
  $lng1 = Get-AzureRmLocalNetworkGateway -Name "DefaultSiteHQ" -ResourceGroupName "ForcedTunneling" 
  $lng2 = Get-AzureRmLocalNetworkGateway -Name "Branch1" -ResourceGroupName "ForcedTunneling" 
  $lng3 = Get-AzureRmLocalNetworkGateway -Name "Branch2" -ResourceGroupName "ForcedTunneling" 
  $lng4 = Get-AzureRmLocalNetworkGateway -Name "Branch3" -ResourceGroupName "ForcedTunneling" 
    
  New-AzureRmVirtualNetworkGatewayConnection -Name "Connection1" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng1 -ConnectionType IPsec -SharedKey "preSharedKey"
  New-AzureRmVirtualNetworkGatewayConnection -Name "Connection2" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng2 -ConnectionType IPsec -SharedKey "preSharedKey"
  New-AzureRmVirtualNetworkGatewayConnection -Name "Connection3" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng3 -ConnectionType IPsec -SharedKey "preSharedKey"
  New-AzureRmVirtualNetworkGatewayConnection -Name "Connection4" -ResourceGroupName "ForcedTunneling" -Location "North Europe" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng4 -ConnectionType IPsec -SharedKey "preSharedKey"
    
  Get-AzureRmVirtualNetworkGatewayConnection -Name "Connection1" -ResourceGroupName "ForcedTunneling"
  ```
