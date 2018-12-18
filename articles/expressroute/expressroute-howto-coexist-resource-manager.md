---
title: 設定可以並存的 ExpressRoute 和站對站 VPN 連線：PowerShell：Azure | Microsoft Docs
description: 本文會引導您使用 PowerShell 為 Resource Manager 模型設定可以並存的 ExpressRoute 和站對站 VPN 連線
services: expressroute
author: charwen
manager: rossort
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/07/2018
ms.author: charwen
ms.openlocfilehash: c267e5002fbd603e4bb749550c19e8d022ce4d54
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44162337"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-using-powershell"></a>使用 PowerShell 設定 ExpressRoute 和站對站並存連線
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell - 傳統](expressroute-howto-coexist-classic.md)
> 
> 

設定站對站 VPN 和 ExpressRoute 並存連線有諸多好處：

* 您可以設定站對站 VPN 作為 ExpressRoute 的安全容錯移轉路徑。 
* 或者，您可以使用站對站 VPN 來連線到未透過 ExpressRoute 連線的網站。 

本文涵蓋設定這兩個案例的步驟。 本文適用於 Resource Manager 部署模型並使用 PowerShell。 您也可以使用 Azure 入口網站來設定這些案例，但目前無法提供相關文件。

>[!NOTE]
>如果您想要建立透過 ExpressRoute 線路的站對站 VPN，請參閱[本文](site-to-site-vpn-over-microsoft-peering.md)。
>

## <a name="limits-and-limitations"></a>限制
* **不支援傳輸路由。** 您無法在透過站對站 VPN 連線的區域網路與透過 ExpressRoute 連線的區域網路之間，進行路由傳送 (透過 Azure)。
* **不支援基本 SKU 閘道。** 您必須對 [ExpressRoute 閘道](expressroute-about-virtual-network-gateways.md)和 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)使用非基本 SKU 閘道。
* **僅支援路由式 VPN 閘道。** 您必須使用路由式 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)。
* **應該為 VPN 閘道設定靜態路由。** 如果您的區域網路連線到 ExpressRoute 和站對站 VPN，您必須在區域網路中設定靜態路由，才能將站對站 VPN 連線路由傳送到公用網際網路。

## <a name="configuration-designs"></a>組態設計
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>設定站對站 VPN 作為 ExpressRoute 的容錯移轉路徑
您可以設定站對站 VPN 連線作為 ExpressRoute 的備用連線。 此連線僅適用於連結至 Azure 私用對等路徑的虛擬網路。 對於可透過 Microsoft Azure 對等互連存取的服務，沒有以 VPN 為基礎的容錯移轉解決方案。 ExpressRoute 線路一律為主要連結。 只有在 ExpressRoute 線路故障時，資料才能流經站對站 VPN 路徑。 為了避免對稱路由，您的本機網路組態應該也偏好透過站對站 VPN 的 ExpressRoute 線路。 您可以針對接收 ExpressRoute 的路由設定較高的本機喜好設定，即表示偏好使用 ExpressRoute 路徑。 

> [!NOTE]
> 雖然當兩個路由相同時，ExpressRoute 線路偏好透過站對站 VPN，但 Azure 會使用最長的相符前置詞來選擇朝向封包目的地的路由。
> 
> 

![並存](media/expressroute-howto-coexist-resource-manager/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>設定站對站 VPN 來連線到未透過 ExpressRoute 連接的網站
您可以將網路設定成有些網站透過站對站 VPN 直接連線到 Azure，而有些網站透過 ExpressRoute 來連線。 

![並存](media/expressroute-howto-coexist-resource-manager/scenario2.jpg)

> [!NOTE]
> 您無法將虛擬網路設定為轉送路由器。
> 
> 

## <a name="selecting-the-steps-to-use"></a>選取要使用的步驟
有兩組不同的程序可供選擇。 您所選取的設定程序取決於您是已經有想要連線的現有虛擬網路，還是想要建立新的虛擬網路。

* 我沒有 VNet 且需要建立一個。
  
    如果您還沒有虛擬網路，這個程序將引導您使用 Resource Manager 部署模型來建立新的虛擬網路，並建立新的 ExpressRoute 和站對站 VPN 連線。 若要設定虛擬網路，請依照[建立新的虛擬網路和並存的連線](#new)中的步驟進行。
* 我已經有一個 Resource Manager 部署模型 VNet。
  
    您可能已經有虛擬網路，而且使用現有的站對站 VPN 連線或 ExpressRoute 連線。 在這個案例中，如果閘道子網路遮罩是 /28 或更小 (/28、/29 等等)，您就必須刪除現有的閘道。 [為已經存在的 VNet 設定並存的連線](#add)一節會引導您刪除閘道，然後建立新的 ExpressRoute 和站對站 VPN 連線。
  
    如果您刪除後重建閘道，您的跨單位連線將會停止。 不過，您的 VM 和服務仍能透過負載平衡器對外通訊，而您能夠在這兩者設定為這樣做時進行閘道器設定。

## <a name="new"></a>建立新的虛擬網路和並存的連線
此程序會引導您建立會並存的 VNet、站對站和 ExpressRoute 連線。

1. 安裝最新版的 Azure PowerShell Cmdlet。 如需如何安裝 Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。 您針對此組態使用的 Cmdlet 可能與您熟悉的 Cmdlet 有些微不同。 請務必使用這些指示中指定的 Cmdlet。

2. 登入您的帳戶並設定環境。

  ```powershell
  Connect-AzureRmAccount
  Select-AzureRmSubscription -SubscriptionName 'yoursubscription'
  $location = "Central US"
  $resgrp = New-AzureRmResourceGroup -Name "ErVpnCoex" -Location $location
  $VNetASN = 65515
  ```
3. 建立包含閘道子網路的虛擬網路。 如需有關建立虛擬網路的詳細資訊，請參閱[建立虛擬網路](../virtual-network/manage-virtual-network.md#create-a-virtual-network)。 如需有關建立子網路的詳細資訊，請參閱[建立子網路](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet)。
   
   > [!IMPORTANT]
   > 閘道子網路必須是 /27 或更短的首碼 (例如 /26 或 /25)。
   > 
   > 
   
    建立新的 VNet。

  ```powershell
  $vnet = New-AzureRmVirtualNetwork -Name "CoexVnet" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AddressPrefix "10.200.0.0/16"
  ```
   
    新增子網路。

  ```powershell
  Add-AzureRmVirtualNetworkSubnetConfig -Name "App" -VirtualNetwork $vnet -AddressPrefix "10.200.1.0/24"
  Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
  ```
   
    儲存 VNet 組態。

  ```powershell
  $vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```
4. <a name="vpngw"></a>接下來，建立站對站 VPN 閘道。 如需 VPN 閘道組態的詳細資訊，請參閱[利用站對站連線設定 VNet](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)。 只有 VpnGw1、VpnGw2、VpnGw3、Standard 和 HighPerformance VPN 閘道支援 GatewaySku。 基本 SKU 不支援 ExpressRoute-VPN 閘道共存組態。 VpnType 必須是 RouteBased。

  ```powershell
  $gwSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
  $gwIP = New-AzureRmPublicIpAddress -Name "VPNGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
  $gwConfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name "VPNGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
  New-AzureRmVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1"
  ```
   
    Azure VPN 閘道支援 BGP 路由通訊協定。 您可以為該虛擬網路指定 ASN (AS 號碼)，方法是在下列命令中新增 -Asn 參數。 未指定該參數，則會預設為 AS 號碼 65515。

  ```powershell
  $azureVpn = New-AzureRmVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "Vpn" -VpnType "RouteBased" -GatewaySku "VpnGw1" -Asn $VNetASN
  ```
   
    您可以在 $azureVpn.BgpSettings.BgpPeeringAddress 和 $azureVpn.BgpSettings.Asn 中找到 Azure 用於 VPN 閘道的 BGP 對等互連 IP 和 AS 號碼。 如需詳細資訊，請參閱針對 Azure VPN 閘道[設定 BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md)。
5. 建立本機的站台 VPN 閘道實體。 此命令不會設定內部部署 VPN 閘道。 相反地，它可讓您提供本機閘道器設定 (例如公用 IP 與內部位址空間)，使 Azure VPN 閘道能夠與其連線。
   
    如果您的本機 VPN 裝置只支援靜態路由，您可以利用下列方式設定靜態路由：

  ```powershell
  $MyLocalNetworkAddress = @("10.100.0.0/16","10.101.0.0/16","10.102.0.0/16")
  $localVpn = New-AzureRmLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress *<Public IP>* -AddressPrefix $MyLocalNetworkAddress
  ```
   
    如果您的本機 VPN 裝置支援 BGP，而且您想要啟用動態路由，您必須本機 VPN 裝置所用的知道 BGP 對等互連 IP 和 AS 號碼。

  ```powershell
  $localVPNPublicIP = "<Public IP>"
  $localBGPPeeringIP = "<Private IP for the BGP session>"
  $localBGPASN = "<ASN>"
  $localAddressPrefix = $localBGPPeeringIP + "/32"
  $localVpn = New-AzureRmLocalNetworkGateway -Name "LocalVPNGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -GatewayIpAddress $localVPNPublicIP -AddressPrefix $localAddressPrefix -BgpPeeringAddress $localBGPPeeringIP -Asn $localBGPASN
  ```
6. 設定本機 VPN 裝置以連接到新的 Azure VPN 閘道。 如需關於 VPN 裝置組態的詳細資訊，請參閱 [VPN 裝置組態](../vpn-gateway/vpn-gateway-about-vpn-devices.md)。

7. 將 Azure 上的站對站 VPN 閘道連結至本機閘道器。

  ```powershell
  $azureVpn = Get-AzureRmVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
  New-AzureRmVirtualNetworkGatewayConnection -Name "VPNConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $azureVpn -LocalNetworkGateway2 $localVpn -ConnectionType IPsec -SharedKey <yourkey>
  ```
 

8. 如果您要連線到現有的 ExpressRoute 線路，請略過步驟 8 和 9 並跳至步驟 10。 設定 ExpressRoute 線路。 如需詳細資訊，請參閱[建立 ExpressRoute 路線](expressroute-howto-circuit-arm.md)。


9. 設定透過 ExpressRoute 線路的 Azure 私人對等互連。 如需設定透過 ExpressRoute 線路的 Azure 私人對等互連詳細資訊，請參閱[設定對等互連](expressroute-howto-routing-arm.md)

10. <a name="gw"></a>建立 ExpressRoute 閘道。 如需 ExpressRoute 閘道組態的詳細資訊，請參閱 [ExpressRoute 閘道組態](expressroute-howto-add-gateway-resource-manager.md)。 GatewaySKU 必須是 *Standard*、*HighPerformance* 或 *UltraPerformance*。

  ```powershell
  $gwSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
  $gwIP = New-AzureRmPublicIpAddress -Name "ERGatewayIP" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -AllocationMethod Dynamic
  $gwConfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name "ERGatewayIpConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id
  $gw = New-AzureRmVirtualNetworkGateway -Name "ERGateway" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -IpConfigurations $gwConfig -GatewayType "ExpressRoute" -GatewaySku Standard
  ```
11. 將 ExpressRoute 閘道器連結到 ExpressRoute 電路。 完成這個步驟之後，內部部署網路和 Azure 之間的連線 (透過 ExpressRoute ) 便會建立。 如需連結作業的詳細資訊，請參閱 [將 Vnet 連結到 ExpressRoute](expressroute-howto-linkvnet-arm.md)。

  ```powershell
  $ckt = Get-AzureRmExpressRouteCircuit -Name "YourCircuit" -ResourceGroupName "YourCircuitResourceGroup"
  New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName $resgrp.ResourceGroupName -Location $location -VirtualNetworkGateway1 $gw -PeerId $ckt.Id -ConnectionType ExpressRoute
  ```

## <a name="add"></a>為已經存在的 VNet 設定並存的連線
如果您的某個虛擬網路只有一個虛擬網路閘道 (比方說，站對站 VPN 閘道)，而您想要新增不同類型的另一個閘道 (比方說，ExpressRoute 閘道)，請檢查閘道子網路大小。 如果閘道器子網路是 /27 以上，您可以略過下列步驟，並依照上一節中的步驟新增站對站 VPN 閘道或 ExpressRoute 閘道。 如果閘道器子網路是/28 或/29，您必須先刪除虛擬網路閘道器，並增加閘道器子網路大小。 本節中的步驟示範如何執行該作業。

1. 您必須安裝最新版的 Azure PowerShell Cmdlet。 如需如何安裝 Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 您針對此組態使用的 Cmdlet 可能與您熟悉的 Cmdlet 有些微不同。 請務必使用這些指示中指定的 Cmdlet。 
2. 刪除現有的 ExpressRoute 或站對站 VPN 閘道。

  ```powershell 
  Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
  ```
3. 刪除閘道子網路。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup> Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
  ```
4. 新增 /27 或更大的閘道子網路。
   
   > [!NOTE]
   > 如果您的虛擬網路中沒有足夠的 IP 位址可以增加閘道器子網路大小，您必須新增更多 IP 位址空間。
   > 
   > 

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
  Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.0/24"
  ```
   
    儲存 VNet 組態。

  ```powershell
  $vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```
5. 此時，您會有不具閘道的虛擬網路。 若要建立新的閘道並設定連線，請依照上一節中的步驟操作。

## <a name="to-add-point-to-site-configuration-to-the-vpn-gateway"></a>將點對站組態新增至 VPN 閘道
您可以在並存設定中，依照下列步驟將點對站組態新增至您的 VPN 閘道。

1. 新增 VPN 用戶端位址集區。

  ```powershell
  $azureVpn = Get-AzureRmVirtualNetworkGateway -Name "VPNGateway" -ResourceGroupName $resgrp.ResourceGroupName
  Set-AzureRmVirtualNetworkGatewayVpnClientConfig -VirtualNetworkGateway $azureVpn -VpnClientAddressPool "10.251.251.0/24"
  ```
2. 針對您的 VPN 閘道將 VPN 根憑證上傳至 Azure。 在此範例中，假設根憑證是儲存於下列 PowerShell Cmdlet 執行所在的本機電腦。

  ```powershell
  $p2sCertFullName = "RootErVpnCoexP2S.cer" 
  $p2sCertMatchName = "RootErVpnCoexP2S" 
  $p2sCertToUpload=get-childitem Cert:\CurrentUser\My | Where-Object {$_.Subject -match $p2sCertMatchName} 
  if ($p2sCertToUpload.count -eq 1){write-host "cert found"} else {write-host "cert not found" exit} 
  $p2sCertData = [System.Convert]::ToBase64String($p2sCertToUpload.RawData) Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $p2sCertFullName -VirtualNetworkGatewayname $azureVpn.Name -ResourceGroupName $resgrp.ResourceGroupName -PublicCertData $p2sCertData
  ```

如需點對站 VPN 的詳細資訊，請參閱 [設定點對站連線](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)。

## <a name="next-steps"></a>後續步驟
如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。
