---
title: '設定 VPN 閘道的主動-主動 S2S VPN 連線：Azure Resource Manager: PowerShell | Microsoft Docs'
description: 本文將逐步引導您使用 Azure Resource Manager 和 PowerShell 來設定與 Azure VPN 閘道的主動-主動連線。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: 238cd9b3-f1ce-4341-b18e-7390935604fa
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/24/2018
ms.author: yushwang, cherylmc
ms.openlocfilehash: 01f25df117eddaaf640a8bd2ef184fe685c5bc75
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39258988"
---
# <a name="configure-active-active-s2s-vpn-connections-with-azure-vpn-gateways"></a>設定 Azure VPN 閘道的主動-主動 S2S VPN 連線

本文將逐步引導您進行使用 Resource Manager 部署模型和 PowerShell 建立主動-主動跨單位和 VNet 對 VNet 連線。

## <a name="about-highly-available-cross-premises-connections"></a>關於高可用性跨單位連線
若要達到跨單位和 VNet 對 VNet 連線能力的高可用性，您應該部署多個 VPN 閘道，並在您的網路和 Azure 之間建立多個平行連線。 如需連線能力選項和拓撲的概觀，請參閱[高可用性跨單位和 VNet 對 VNet 連線能力](vpn-gateway-highlyavailable.md)。

本文提供用來設定主動-主動跨單位 VPN 連線，以及兩個虛擬網路間之主動-主動連線的指示。

* [第 1 部分 - 建立 Azure VPN 閘道並將它設定為主動-主動模式](#aagateway)
* [第 2 部分 - 建立主動-主動跨單位連線](#aacrossprem)
* [第 3 部分 - 建立主動-主動 VNet 對 VNet 連線](#aav2v)

如果您已經有 VPN 閘道，您可以：
* [將現有的 VPN 閘道從作用中-待命更新為主動-主動，或反之亦然](#aaupdate)

您可以將這些結合起來，建立更複雜、高可用性網路拓撲以符合您的需求。

> [!IMPORTANT]
> 主動-主動模式僅使用下列 SKU： 
  * VpnGw1、VpnGw2、VpnGw3
  * HighPerformance (對於舊版 SKU)
> 
> 

## <a name ="aagateway"></a>第 1 部分 - 建立並設定主動-主動 VPN 閘道
以下步驟會將您的 Azure VPN 閘道設定為主動-主動模式。 主動-主動和作用中-待命閘道之間的重要差異：

* 您需要使用兩個公用 IP 位址建立兩個閘道 IP 組態
* 您需要設定 EnableActiveActiveFeature 旗標
* 閘道 SKU 必須是 VpnGw1、VpnGw2、VpnGw3 或 HighPerformance (舊版 SKU)。

其他屬性都與非主動-主動閘道相同。 

### <a name="before-you-begin"></a>開始之前
* 請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* 您必須安裝 Azure Resource Manager PowerShell Cmdlet。 如需安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

### <a name="step-1---create-and-configure-vnet1"></a>步驟 1 - 建立及設定 VNet1
#### <a name="1-declare-your-variables"></a>1.宣告變數
對於此練習，我們一開始先宣告我們的變數。 下列範例會使用此練習中的值來宣告變數。 請務必在設定生產環境時，使用您自己的值來取代該值。 若您執行這些步驟是為了熟悉此類型的設定，則可以使用這些變數。 修改變數，然後將其複製並貼到您的 PowerShell 主控台中。

```powershell
$Sub1 = "Ross"
$RG1 = "TestAARG1"
$Location1 = "West US"
$VNetName1 = "TestVNet1"
$FESubName1 = "FrontEnd"
$BESubName1 = "Backend"
$GWSubName1 = "GatewaySubnet"
$VNetPrefix11 = "10.11.0.0/16"
$VNetPrefix12 = "10.12.0.0/16"
$FESubPrefix1 = "10.11.0.0/24"
$BESubPrefix1 = "10.12.0.0/24"
$GWSubPrefix1 = "10.12.255.0/27"
$VNet1ASN = 65010
$DNS1 = "8.8.8.8"
$GWName1 = "VNet1GW"
$GW1IPName1 = "VNet1GWIP1"
$GW1IPName2 = "VNet1GWIP2"
$GW1IPconf1 = "gw1ipconf1"
$GW1IPconf2 = "gw1ipconf2"
$Connection12 = "VNet1toVNet2"
$Connection151 = "VNet1toSite5_1"
$Connection152 = "VNet1toSite5_2"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2.連接至您的訂用帳戶並建立新的資源群組
請確定您切換為 PowerShell 模式以使用資源管理員 Cmdlet。 如需詳細資訊，請參閱 [搭配使用 Windows PowerShell 與 Resource Manager](../powershell-azure-resource-manager.md)。

開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $Sub1
New-AzureRmResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3.建立 TestVNet1
下列範例會建立一個名為 TestVNet1 的虛擬網路和三個子網路：一個名為 GatewaySubnet、一個名為 FrontEnd，另一個名為 Backend。 替代值時，務必一律將您的閘道子網路特定命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

```powershell
$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-active-active-mode"></a>步驟 2 - 以主動-主動模式建立 TestVNet1 的 VPN 閘道
#### <a name="1-create-the-public-ip-addresses-and-gateway-ip-configurations"></a>1.建立公用 IP 位址和閘道 IP 組態
需要將兩個公用 IP 位址配置給您將為 VNet 建立的閘道。 您也會定義所需的子網路和 IP 組態。

```powershell
$gw1pip1 = New-AzureRmPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$gw1pip2 = New-AzureRmPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
$gw1ipconf2 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW1IPconf2 -Subnet $subnet1 -PublicIpAddress $gw1pip2
```

#### <a name="2-create-the-vpn-gateway-with-active-active-configuration"></a>2.以主動-主動組態建立 VPN 閘道
建立 TestVNet1 的虛擬網路閘道。 請注意有兩個 GatewayIpConfig 項目，且已設定 EnableActiveActiveFeature 旗標。 建立閘道可能需要花費一段時間 (45 分鐘或更久)。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1,$gw1ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN -EnableActiveActiveFeature -Debug
```

#### <a name="3-obtain-the-gateway-public-ip-addresses-and-the-bgp-peer-ip-address"></a>3.取得閘道公用 IP 位址和 BGP 對等 IP 位址
建立閘道後，您必須取得 Azure VPN 閘道上的 BGP 對等 IP 位址。 需要有此位址，才能將 Azure VPN 閘道設定為您的內部部署 VPN 裝置的 BGP 對等。

```powershell
$gw1pip1 = Get-AzureRmPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1
$gw1pip2 = Get-AzureRmPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
```

使用以下 Cmdlet 來顯示為 VPN 閘道配置的兩個公用 IP 位址，以及和每個閘道執行個體相對應的 BGP 對等 IP 位址：

```powershell
PS D:\> $gw1pip1.IpAddress
40.112.190.5

PS D:\> $gw1pip2.IpAddress
138.91.156.129

PS D:\> $vnet1gw.BgpSettingsText
{
  "Asn": 65010,
  "BgpPeeringAddress": "10.12.255.4,10.12.255.5",
  "PeerWeight": 0
}
```

閘道執行個體的公用 IP 位址順序與相對應的 BGP 對等連線位址相同。 在此範例中，公用 IP 為 40.112.190.5 的閘道 VM 會使用 10.12.255.4 作為其 BGP 對等連線位址，而公用 IP 為 138.91.156.129 的閘道會使用 10.12.255.5。 當您在內部部署 VPN 裝置上設定連線到主動-主動閘道時會需要此資訊。 下面圖表中顯示閘道及所有位址：

![主動-主動閘道](./media/vpn-gateway-activeactive-rm-powershell/active-active-gw.png)

一旦建立閘道，您就可以使用此閘道建立主動-主動跨單位或 VNet 對 VNet 連線。 下列各節將逐步說明完成練習的步驟。

## <a name ="aacrossprem"></a>第 2 部分 - 建立主動-主動跨單位連線
若要建立跨單位連線，您需要建立區域網路閘道來代表您的內部部署 VPN 裝置，以及建立一個連線來連接 Azure VPN 閘道與區域網路閘道。 在此範例中，Azure VPN 閘道是處於主動-主動模式。 因此，即使只有一個內部部署 VPN 裝置 (區域網路閘道) 和一個連線資源，兩個 Azure VPN 閘道執行個體都會建立與該內部部署裝置的 S2S VPN 通道。

在繼續之前，請確定您已完成本練習的 [第 1 部分](#aagateway) 。

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>步驟 1 - 建立及設定區域網路閘道
#### <a name="1-declare-your-variables"></a>1.宣告變數
本練習將繼續建置圖中所示的組態。 請務必使用您想用於設定的值來取代該值。

```powershell
$RG5 = "TestAARG5"
$Location5 = "West US"
$LNGName51 = "Site5_1"
$LNGPrefix51 = "10.52.255.253/32"
$LNGIP51 = "131.107.72.22"
$LNGASN5 = 65050
$BGPPeerIP51 = "10.52.255.253"
```

關於區域網路閘道參數，有幾件事要注意︰

* 區域網路閘道可以位於與 VPN 閘道相同或不同的位置和資源群組中。 此範例會顯示它們位在不同的資源群組，但是位於相同的 Azure 位置。
* 如果如上面顯示只有一個內部部署 VPN 裝置，則無論有無 BGP 通訊協定，主動-主動連線皆可運作。 此範例針對跨單位連線使用 BGP。
* 如果已啟用 BGP，您需要針對區域網路閘道宣告的前置詞就是您 VPN 裝置上 BGP 對等 IP 位址的主機位址。 在此情況下是 "10.52.255.253/32" 的前置詞 /32。
* 提醒您，您必須在內部部署網路與 Azure VNet 之間使用不同的 BGP ASN。 在兩者相同的情況下，如果內部部署 VPN 裝置已經使用 ASN 來與其他 BGP 芳鄰建立對等連線，您就需要變更您的 VNet ASN。

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2.建立 Site5 的區域網路閘道
請確定您仍然與訂用帳戶 1 保持連線，然後再繼續。 建立資源群組 (若尚未建立)。

```powershell
New-AzureRmResourceGroup -Name $RG5 -Location $Location5
New-AzureRmLocalNetworkGateway -Name $LNGName51 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP51 -AddressPrefix $LNGPrefix51 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP51
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>步驟 2 - 連接 VNet 閘道與區域網路閘道
#### <a name="1-get-the-two-gateways"></a>1.取得兩個閘道

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw1 = Get-AzureRmLocalNetworkGateway  -Name $LNGName51 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2.建立 TestVNet1 至 Site5 的連線
在此步驟中，您會建立從 TestVNet1 到 Site5_1 的連線，並將 "EnableBGP" 設為 $True。

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name $Connection151 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw1 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-on-premises-vpn-device"></a>3.內部部署 VPN 裝置的 VPN 和 BGP 參數
下列範例列出您將會針對此練習中，在內部部署 VPN 裝置的 BGP 組態區段中輸入的參數︰

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.253
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

幾分鐘後應會建立連線，而一旦建立 IPsec 連線，BGP 對等工作階段就會啟動。 此範例中到目前為止只有設定一個內部部署 VPN 裝置，結果如下面圖表所示︰

![active-active-crossprem](./media/vpn-gateway-activeactive-rm-powershell/active-active.png)

### <a name="step-3---connect-two-on-premises-vpn-devices-to-the-active-active-vpn-gateway"></a>步驟 3 - 將兩個內部部署 VPN 裝置連線到主動-主動 VPN 閘道
如果您在同一個內部部署網路上有兩個 VPN 裝置，您可以透過將 Azure VPN 閘道連接到第二個 VPN 裝置來達成雙重備援。

#### <a name="1-create-the-second-local-network-gateway-for-site5"></a>1.針對 Site5 建立第二個區域網路閘道
第二個區域網路閘道的閘道 IP位址、位址首碼及 BGP 對等互連位址，不得與同一個內部部署網路的前一個區域網路閘道重疊。

```powershell
$LNGName52 = "Site5_2"
$LNGPrefix52 = "10.52.255.254/32"
$LNGIP52 = "131.107.72.23"
$BGPPeerIP52 = "10.52.255.254"
```

```powershell
New-AzureRmLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP52 -AddressPrefix $LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP52
```

#### <a name="2-connect-the-vnet-gateway-and-the-second-local-network-gateway"></a>2.將 VNet 閘道與第二個區域網路閘道連線
建立從 TestVNet1 至 Site5_2 的連線且 "EnableBGP" 設為 $True

```powershell
$lng5gw2 = Get-AzureRmLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5
```

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name $Connection152 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw2 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-second-on-premises-vpn-device"></a>3.第二個內部部署 VPN 裝置的 VPN 和 BGP 參數
同樣地，以下列出您將輸入到第二個 VPN 裝置的參數︰

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

一旦建立連線 (通道)，您就會有連線到您內部部署網路和 Azure 的雙重備援 VPN 裝置和通道：

![dual-redundancy-crossprem](./media/vpn-gateway-activeactive-rm-powershell/dual-redundancy.png)

## <a name ="aav2v"></a>第 3 部分 - 建立主動-主動 VNet 對 VNet 連線
本節會使用 BGP 建立主動-主動 VNet 對 VNet 連線。 

下面的指示是從上方所列的前一個步驟繼續執行。 您必須完成 [第 1 部分](#aagateway) 以使用 BGP 建立和設定 TestVNet1 及 VPN 閘道。 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>步驟 1 - 建立 TestVNet2 和 VPN 閘道
請務必確定新虛擬網路的 IP 位址空間 TestVNet2 不會與任何 VNet 範圍重疊。

在此範例中，虛擬網路屬於相同的訂用帳戶。 您可以在不同訂用帳戶之間設定 VNet 對 VNet 連線，若要深入了解詳細資料，請參閱 [設定 VNet 對 VNet 連線](vpn-gateway-vnet-vnet-rm-ps.md) 。 請務必在建立連線時新增 "-EnableBgp $True"，才能啟用 BGP。

#### <a name="1-declare-your-variables"></a>1.宣告變數
請務必使用您想用於設定的值來取代該值。

```powershell
$RG2 = "TestAARG2"
$Location2 = "East US"
$VNetName2 = "TestVNet2"
$FESubName2 = "FrontEnd"
$BESubName2 = "Backend"
$GWSubName2 = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$VNet2ASN = 65020
$DNS2 = "8.8.8.8"
$GWName2 = "VNet2GW"
$GW2IPName1 = "VNet2GWIP1"
$GW2IPconf1 = "gw2ipconf1"
$GW2IPName2 = "VNet2GWIP2"
$GW2IPconf2 = "gw2ipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2.在新的資源群組中建立 TestVNet2

```powershell
New-AzureRmResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzureRmVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-active-active-vpn-gateway-for-testvnet2"></a>3.建立 TestVNet2 的主動-主動 VPN 閘道
需要將兩個公用 IP 位址配置給您將為 VNet 建立的閘道。 您也會定義所需的子網路和 IP 組態。

```powershell
$gw2pip1 = New-AzureRmPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$gw2pip2 = New-AzureRmPublicIpAddress -Name $GW2IPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2 = Get-AzureRmVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1
$gw2ipconf2 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW2IPconf2 -Subnet $subnet2 -PublicIpAddress $gw2pip2
```

使用 AS 編號和 "EnableActiveActiveFeature" 旗標建立 VPN 閘道。 請注意，您必須覆寫您的 Azure VPN 閘道上的預設 ASN。 已連接 VNet 的 ASN 必須不同，才能啟用 BGP 與傳輸路由。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1,$gw2ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN -EnableActiveActiveFeature
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>步驟 2 - 連接 TestVNet1 與 TestVNet2 閘道
在此範例中，兩個閘道位於相同的訂用帳戶。 您可以在相同的 PowerShell 工作階段中完成此步驟。

#### <a name="1-get-both-gateways"></a>1.取得兩個閘道
請確定您已登入並連接到訂用帳戶 1。

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzureRmVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2.建立兩個連線
在此步驟中，您將建立從 TestVNet1 到 TestVNet2 的連線，以及從 TestVNet2 到 TestVNet1 的連線。

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> 請務必對這兩個連線啟用 BGP。
> 
> 

完成這些步驟後，將會在幾分鐘內建立連線，而一旦完成具備雙重備援的 VNet 對 VNet 連線，BGP 對等工作階段就會啟動：

![active-active-v2v](./media/vpn-gateway-activeactive-rm-powershell/vnet-to-vnet.png)

## <a name ="aaupdate"></a>更新現有的 VPN 閘道

本節會協助您將現有的 Azure VPN 閘道從作用中-待命變更為主動-主動模式，或反之亦然。

### <a name="change-an-active-standby-gateway-to-an-active-active-gateway"></a>將作用中-待命閘道變更為主動-主動閘道

以下範例會將作用中-待命閘道轉換成主動-主動閘道。 當您將作用中-待命閘道變更為主動-主動時，會建立另一個公用 IP 位址，然後新增第二個閘道 IP 設定。

#### <a name="1-declare-your-variables"></a>1.宣告變數

使用您自己的組態所需的設定來取代下列針對範例所使用的參數，然後宣告這些變數。

```powershell
$GWName = "TestVNetAA1GW"
$VNetName = "TestVNetAA1"
$RG = "TestVPNActiveActive01"
$GWIPName2 = "gwpip2"
$GWIPconf2 = "gw1ipconf2"
```

宣告變數之後，您可以複製此範例並貼上至您的 PowerShell 主控台。

```powershell
$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gw = Get-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$location = $gw.Location
```

#### <a name="2-create-the-public-ip-address-then-add-the-second-gateway-ip-configuration"></a>2.建立公用 IP 位址，然後新增第二個閘道 IP 組態

```powershell
$gwpip2 = New-AzureRmPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG -Location $location -AllocationMethod Dynamic
Add-AzureRmVirtualNetworkGatewayIpConfig -VirtualNetworkGateway $gw -Name $GWIPconf2 -Subnet $subnet -PublicIpAddress $gwpip2
```

#### <a name="3-enable-active-active-mode-and-update-the-gateway"></a>3.啟用主動-主動模式並更新閘道

您會在這個步驟中啟用主動-主動模式並更新閘道。 在範例中，VPN 閘道目前使用舊版的標準 SKU。 不過，主動-主動不支援標準 SKU。 若要將舊版 SKU 調整為支援的 SKU (在此案例中為 HighPerformance)，只需指定想要使用之支援的舊版 SKU 即可。

* 您無法使用此步驟將舊版 SKU 變更為其中一個新的 SKU。 您只能將舊版 SKU 調整為另一個支援的舊版 SKU。 例如，您無法將 SKU 從 Standard 變更為 VpnGw1 (即使主動-主動支援 VpnGw1 也一樣)，因為 Standard 是舊版 SKU，而 VpnGw1 則是目前的 SKU。 如需調整與移轉 SKU 的詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpngateways.md#gwsku)。

* 如果您想要調整目前的 SKU (例如，從 VpnGw1 到 VpnGw3)，您可以使用此步驟來達成，因為 SKU 都屬於同一個 SKU 系列。 若要這樣做，您會使用值：```-GatewaySku VpnGw3```

當您在環境中使用此項時，如果您不需要調整閘道，將不需指定 -GatewaySku。 請注意，在此步驟中，您必須在 PowerShell 中設定閘道物件以觸發實際的更新。 即使您不調整閘道，這項更新可能還是需要 30 到 45 分鐘。

```powershell
Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -EnableActiveActiveFeature -GatewaySku HighPerformance
```

### <a name="change-an-active-active-gateway-to-an-active-standby-gateway"></a>將主動-主動閘道變更為作用中-待命閘道
#### <a name="1-declare-your-variables"></a>1.宣告變數

使用您自己的組態所需的設定來取代下列針對範例所使用的參數，然後宣告這些變數。

```powershell
$GWName = "TestVNetAA1GW"
$RG = "TestVPNActiveActive01"
```

宣告變數之後，取得您要移除之 IP 設定的名稱。

```powershell
$gw = Get-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$ipconfname = $gw.IpConfigurations[1].Name
```

#### <a name="2-remove-the-gateway-ip-configuration-and-disable-the-active-active-mode"></a>2.移除閘道 IP 組態並停用主動-主動模式

使用此範例，來移除閘道 IP 設定並停用主動-主動模式。 請注意，您必須在 PowerShell 中設定閘道物件以觸發實際的更新。

```powershell
Remove-AzureRmVirtualNetworkGatewayIpConfig -Name $ipconfname -VirtualNetworkGateway $gw
Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -DisableActiveActiveFeature
```

此更新最多可能需要 30 到 45 分鐘。

## <a name="next-steps"></a>後續步驟
一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。
