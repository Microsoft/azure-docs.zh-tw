---
title: 將內部部署網路連接至 Azure 虛擬網路：站對站 VPN：PowerShell | Microsoft Docs
description: 透過公用網際網路建立從內部部署網路至 Azure 虛擬網路之 IPsec 連線的步驟。 這些步驟可協助您使用 PowerShell 建立跨單位的站對站 VPN 閘道連線。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: fcc2fda5-4493-4c15-9436-84d35adbda8e
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/28/2018
ms.author: cherylmc
ms.openlocfilehash: ee4e200fa57a38249b0be2997de7e3268a9302cb
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38712918"
---
# <a name="create-a-vnet-with-a-site-to-site-vpn-connection-using-powershell"></a>使用 PowerShell 建立具有站對站 VPN 連接的 VNet

本文說明如何使用 PowerShell 來建立從內部部署網路到 VNet 的站對站 VPN 閘道連線。 本文中的步驟適用於 Resource Manager 部署模型。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-site-to-site-classic-portal.md)
> 
>

站對站 VPN 閘道連線可用來透過 IPsec/IKE (IKEv1 或 IKEv2) VPN 通道，將內部部署網路連線到 Azure 虛擬網路。 此類型的連線需要位於內部部署的 VPN 裝置，且您已對該裝置指派對外開放的公用 IP 位址。 如需 VPN 閘道的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。

![站對站 VPN 閘道跨單位連線圖表](./media/vpn-gateway-create-site-to-site-rm-powershell/site-to-site-diagram.png)

## <a name="before"></a>開始之前

在開始設定之前，請確認您已符合下列條件：

* 確定您有相容的 VPN 裝置以及能夠對其進行設定的人員。 如需相容 VPN 裝置和裝置組態的詳細資訊，請參閱[關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)。
* 確認您的 VPN 裝置有對外開放的公用 IPv4 位址。 此 IP 位址不能位於 NAT 後方。
* 如果您不熟悉位於內部部署網路組態的 IP 位址範圍，您需要與能夠提供那些詳細資料的人協調。 當您建立此組態時，您必須指定 IP 位址範圍的首碼，以供 Azure 路由傳送至您的內部部署位置。 內部部署網路的子網路皆不得與您所要連線的虛擬網路子網路重疊。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

### <a name="running-powershell-locally"></a>在本機執行 PowerShell

如果您選擇在本機安裝及使用 PowerShell，請安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 PowerShell Cmdlet 會經常更新，您通常會需要更新您的 PowerShell Cmdlet 才能取得最新的功能。 如果您未更新 PowerShell Cmdlet，指定的值可能會失敗。 

若要尋找您所使用的版本，請執行 'Get-Module -ListAvailable AzureRM'。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。
如果您在本機執行 PowerShell，則也需要執行 'Connect-AzureRmAccount' 以建立與 Azure 的連線。


### <a name="example"></a>範例值

本文的範例使用下列值。 您可以使用這些值來建立測試環境，或參考這些值，進一步了解本文中的範例。

```
#Example values

VnetName                = VNet1
ResourceGroup           = TestRG1
Location                = East US 
AddressSpace            = 10.1.0.0/16 
SubnetName              = Frontend 
Subnet                  = 10.1.0.0/24 
GatewaySubnet           = 10.1.255.0/27
LocalNetworkGatewayName = Site1
LNG Public IP           = <On-premises VPN device IP address> 
Local Address Prefixes  = 10.101.0.0/24, 10.101.1.0/24
Gateway Name            = VNet1GW
PublicIP                = VNet1GWPIP
Gateway IP Config       = gwipconfig1 
VPNType                 = RouteBased 
GatewayType             = Vpn 
ConnectionName          = VNet1toSite1

```

## <a name="VNet"></a>1.建立虛擬網路和閘道器子網路

如果您還沒有虛擬網路，請建立一個。 在建立虛擬網路時，請確定您指定的位址空間沒有與您在內部部署網路上所擁有的任何位址空間重疊。 

>[!NOTE]
>為了讓此 VNet 連線到內部部署位置，您需要與內部部署網路系統管理員協調，以切割出此虛擬網路專用的 IP 位址範圍。 如果 VPN 連線的兩端存在重複的位址範圍，流量就不會如預期的方式進行路由。 此外，如果您想要將此 VNet 連線到另一個 VNet，則位址空間不能與其他 VNet 重疊。 因此，請謹慎規劃您的網路組態。
>
>

### <a name="about-the-gateway-subnet"></a>關於閘道子網路

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-include.md)]

[!INCLUDE [No NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="vnet"></a>建立虛擬網路和閘道器子網路

此範例會建立虛擬網路和閘道子網路。 如果您已經有虛擬網路，並需要對它新增閘道子網路，請參閱[將閘道子網路新增至您已建立的虛擬網路](#gatewaysubnet)。

建立資源群組：

```azurepowershell-interactive
New-AzureRmResourceGroup -Name TestRG1 -Location 'East US'
```

建立虛擬網路。

1. 設定變數。

  ```azurepowershell-interactive
  $subnet1 = New-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27
  $subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name 'Frontend' -AddressPrefix 10.1.0.0/24
  ```
2. 建立 VNet。

  ```azurepowershell-interactive
  New-AzureRmVirtualNetwork -Name VNet1 -ResourceGroupName TestRG1 `
  -Location 'East US' -AddressPrefix 10.1.0.0/16 -Subnet $subnet1, $subnet2
  ```

### <a name="gatewaysubnet"></a>將閘道器子網路加入至您已建立的虛擬網路

如果您已經有虛擬網路，但需要新增閘道子網路，請使用本節中的步驟。

1. 設定變數。

  ```azurepowershell-interactive
  $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG1 -Name TestVet1
  ```
2. 建立閘道子網路。

  ```azurepowershell-interactive
  Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $vnet
  ```
3. 設定組態。

  ```azurepowershell-interactive
  Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```

## 2.<a name="localnet"></a>建立區域網路閘道

區域網路閘道通常是指您的內部部署位置。 請賦予網站可供 Azure 參考的名稱，然後指定您想要與其建立連線之內部部署 VPN 裝置的 IP 位址。 也請指定 IP 位址首碼，以供系統透過 VPN 閘道路由至 VPN 裝置。 您指定的位址首碼是位於內部部署網路上的首碼。 如果您的內部部署網路有所變更，您可以輕鬆地更新首碼。

輸入下列值：

* *GatewayIPAddress* 是內部部署 VPN 裝置的 IP 位址。 VPN 裝置不能位於 NAT 後方。
* *AddressPrefix* 是您的內部部署位址空間。

若要新增具有單一位址前置詞的區域網路閘道：

  ```azurepowershell-interactive
  New-AzureRmLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1 `
  -Location 'East US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.101.0.0/24'
  ```

若要新增具有多個位址前置詞的區域網路閘道：

  ```azurepowershell-interactive
  New-AzureRmLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1 `
  -Location 'East US' -GatewayIpAddress '23.99.221.164' -AddressPrefix @('10.101.0.0/24','10.101.1.0/24')
  ```

修改區域網路閘道的 IP 位址首碼：<br>
有時候您的區域網路閘道首碼會有所變更。 若要修改您的 IP 位址前置詞，採取的步驟取決於您是否已建立 VPN 閘道連線。 請參閱本文的 [修改區域網路閘道的 IP 位址首碼](#modify) 一節。

## <a name="PublicIP"></a>3.要求公用 IP 位址

VPN 閘道必須具有公用 IP 位址。 您會先要求 IP 位址資源，然後在建立虛擬網路閘道時參考它。 建立 VPN 閘道時，系統會將 IP 位址動態指派給此資源。 VPN 閘道目前僅支援*動態*公用 IP 位址配置。 您無法要求靜態公用 IP 位址指派。 不過，這不表示 IP 位址變更之後已被指派至您的 VPN 閘道。 公用 IP 位址只會在刪除或重新建立閘道時變更。 它不會因為重新調整、重設或 VPN 閘道的其他內部維護/升級而變更。

請要求公用 IP 位址，以指派給虛擬網路 VPN 閘道。

```azurepowershell-interactive
$gwpip= New-AzureRmPublicIpAddress -Name VNet1GWPIP -ResourceGroupName TestRG1 -Location 'East US' -AllocationMethod Dynamic
```

## <a name="GatewayIPConfig"></a>4.建立閘道器 IP 位址組態

閘道器組態定義要使用的子網路和公用 IP 位址。 使用下列範例來建立閘道組態：

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork -Name VNet1 -ResourceGroupName TestRG1
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
```

## <a name="CreateGateway"></a>5.建立 VPN 閘道

建立虛擬網路 VPN 閘道。

輸入下列值：

* 網站間組態的 -GatewayType 是 Vpn。 閘道器類型永遠是您實作的組態的特定類型。 例如，其他閘道器組態可能需要 -GatewayType ExpressRoute。
* -VpnType 可以是 RouteBased (在某些文件中稱為動態閘道器)，或 PolicyBased (在某些文件中稱為靜態閘道器)。 如需 VPN 閘道類型的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。
* 選取您想要使用的閘道 SKU。 某些 SKU 有組態限制。 如需詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。 如果您在建立有關 -GatewaySku 的 VPN 閘道時發生錯誤，請確認您已安裝最新版的 PowerShell Cmdlet。

```azurepowershell-interactive
New-AzureRmVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'East US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased -GatewaySku VpnGw1
```

執行此命令後，可能需要 45 分鐘的時間才能完成閘道設定。

## <a name="ConfigureVPNDevice"></a>6.設定 VPN 裝置

內部部署網路的站對站連線需要 VPN 裝置。 在此步驟中，設定 VPN 裝置。 在設定 VPN 裝置時，您需要下列項目：

- 共用金鑰。 這個共同金鑰與您建立站對站 VPN 連線時指定的共用金鑰相同。 在我們的範例中，我們會使用基本的共用金鑰。 我們建議您產生更複雜的金鑰以供使用。
- 虛擬網路閘道的公用 IP 位址。 您可以使用 Azure 入口網站、PowerShell 或 CLI 來檢視公用 IP 位址。 若要使用 PowerShell 尋找虛擬網路閘道的公用 IP 位址，請使用下面範例：

  ```azurepowershell-interactive
  Get-AzureRmPublicIpAddress -Name GW1PublicIP -ResourceGroupName TestRG1
  ```

[!INCLUDE [Configure VPN device](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]


## <a name="CreateConnection"></a>7.建立 VPN 連線

接下來，在虛擬網路閘道與 VPN 裝置之間建立網站間 VPN 連線。 請務必將值取代為您自己的值。 共用的金鑰必須符合您用於 VPN 裝置設定的值。 請注意，網站間的「-ConnectionType」為 IPsec。

1. 設定變數。
  ```azurepowershell-interactive
  $gateway1 = Get-AzureRmVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1
  $local = Get-AzureRmLocalNetworkGateway -Name Site1 -ResourceGroupName TestRG1
  ```

2. 建立連線。
  ```azurepowershell-interactive
  New-AzureRmVirtualNetworkGatewayConnection -Name VNet1toSite1 -ResourceGroupName TestRG1 `
  -Location 'East US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
  -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
  ```

過一會兒，連接將會建立。

## <a name="toverify"></a>8.驗證 VPN 連線

VPN 連線有幾種不同的驗證方式。

[!INCLUDE [Verify connection](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="connectVM"></a>連線至虛擬機器

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-s2s-include.md)]


## <a name="modify"></a>修改區域網路閘道的 IP 位址首碼

如果您要路由傳送到內部部署位置的 IP 位址首碼變更，您可以修改區域網路閘道。 所提供的指示有兩組。 要選擇哪組指示取決於您是否已建立閘道連線。

[!INCLUDE [Modify prefixes](../../includes/vpn-gateway-modify-ip-prefix-rm-include.md)]

## <a name="modifygwipaddress"></a>修改區域網路閘道的閘道 IP 位址

[!INCLUDE [Modify gateway IP address](../../includes/vpn-gateway-modify-lng-gateway-ip-rm-include.md)]

## <a name="next-steps"></a>後續步驟

*  一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。
* 如需 BGP 的相關資訊，請參閱 [BGP 概觀](vpn-gateway-bgp-overview.md)和[如何設定 BGP](vpn-gateway-bgp-resource-manager-ps.md)。
* 如需使用 Azure Resource Manager 範本建立站對站 VPN 連線的相關資訊，請參閱[建立站對站 VPN 連線](https://azure.microsoft.com/resources/templates/101-site-to-site-vpn-create/)。
* 如需使用 Azure Resource Manager 範本建立 vnet 對 vnet VPN 連線的相關資訊，請參閱[部署 HB 異地複寫](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-replication-geo/)。
