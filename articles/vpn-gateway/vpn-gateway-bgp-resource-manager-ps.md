---
title: '在 Azure VPN 閘道上設定 BGP：Resource Manager: PowerShell | Microsoft Docs'
description: 本文將逐步引導您使用 Azure Resource Manager 和 PowerShell 來設定 BGP 與 Azure VPN 閘道。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: 905b11a7-1333-482c-820b-0fd0f44238e5
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/12/2017
ms.author: yushwang
ms.openlocfilehash: fc9337188fd439082c4aa34f0cbebe3eb2da5d99
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31603212"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways-using-powershell"></a>如何使用 PowerShell 在 Azure VPN 閘道上設定 BGP
本文將逐步引導您進行使用 Resource Manager 部署模型和 PowerShell 在跨單位網站間 (S2S) VPN 連線和 VNet 對 VNet 連線上啟用 BGP 的步驟。

## <a name="about-bgp"></a>關於 BGP
BGP 是常用於網際網路的標準路由通訊協定，可交換兩個或多個網路之間的路由和可執行性資訊。 BGP 會啟用 Azure VPN 閘道，以及內部部署 VPN 裝置 (稱為 BGP 對等或鄰近項目) 來交換「路由」，其會通知這兩個閘道對要通過閘道的首碼或所涉及之路由器的可用性和可執行性。 BGP 也可以傳播從一個 BGP 對等互連到所有其他 BGP 對等所識別的 BGP 閘道，來啟用多個網路之間的傳輸路由。

如需 BGP 優點的討論並了解使用 BGP 的技術需求和考量，請參閱 [BGP 與 Azure VPN 閘道的概觀](vpn-gateway-bgp-overview.md)。

## <a name="getting-started-with-bgp-on-azure-vpn-gateways"></a>開始在 Azure VPN 閘道上使用 BGP

本文將逐步引導您完成執行下列作業的步驟：

* [第 1 部份 - 在 Azure VPN 閘道上啟用 BGP](#enablebgp)
* [第 2 部份 – 建立與 BGP 的跨單位連線](#crossprembgp)
* [第 3 部份 – 建立與 BGP 的 VNet 對 VNet 連線](#v2vbgp)

指示的每個部分均構成基本建置組塊，可供在您的網路連線中啟用 BGP。 如果您完成上述三個部分，將建置如下圖所示的拓撲︰

![BGP 拓樸](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

您可以將多個部分結合起來，依您的需求建立更複雜的多重躍點傳輸網路。

## <a name ="enablebgp"></a>第 1 部分 - 在 Azure VPN 閘道上設定 BGP
設定步驟將會設定 Azure VPN 閘道的 BGP 參數，如下圖所示︰

![BGP 閘道](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>開始之前
* 請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* 您必須安裝 Azure Resource Manager PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。 

### <a name="step-1---create-and-configure-vnet1"></a>步驟 1 - 建立及設定 VNet1
#### <a name="1-declare-your-variables"></a>1.宣告變數
對於此練習，我們一開始先宣告我們的變數。 下列範例會使用此練習中的值來宣告變數。 請務必在設定生產環境時，使用您自己的值來取代該值。 若您執行這些步驟是為了熟悉此類型的設定，則可以使用這些變數。 修改變數，然後將其複製並貼到您的 PowerShell 主控台中。

```powershell
$Sub1 = "Replace_With_Your_Subcription_Name"
$RG1 = "TestBGPRG1"
$Location1 = "East US"
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
$GWIPName1 = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection12 = "VNet1toVNet2"
$Connection15 = "VNet1toSite5"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2.連接至您的訂用帳戶並建立新的資源群組
請確定您切換為 PowerShell 模式，以使用 Resource Manager Cmdlet。 如需詳細資訊，請參閱 [搭配使用 Windows PowerShell 與 Resource Manager](../powershell-azure-resource-manager.md)。

開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $Sub1
New-AzureRmResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3.建立 TestVNet1
下列範例會建立一個名為 TestVNet1 的虛擬網路和三個子網路：一個名為 GatewaySubnet、一個名為 FrontEnd，另一個名為 Backend。 替代值時，務必一律將您的閘道子網路特定命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

```powershell
$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 $besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>步驟 2 - 使用 BGP 參數建立 TestVNet1 的 VPN 閘道
#### <a name="1-create-the-ip-and-subnet-configurations"></a>1.建立 IP 和子網路組態
要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的閘道。 您也會定義必要的子網路和 IP 組態。

```powershell
$gwpip1 = New-AzureRmPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2.透過 AS 號碼建立 VPN 閘道
建立 TestVNet1 的虛擬網路閘道。 BGP 需要路由式 VPN 閘道，以及額外參數 (-Asn) 才能設定 TestVNet1 的 ASN (AS 號碼)。 如果您沒有設定 ASN 參數，系統會指派 ASN 65515。 建立閘道可能需要花費一段時間 (30 分鐘或更久)。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance -Asn $VNet1ASN
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3.取得 Azure BGP 對等 IP 位址
建立閘道後，您必須取得 Azure VPN 閘道上的 BGP 對等 IP 位址。 需要有此位址，才能將 Azure VPN 閘道設定為您的內部部署 VPN 裝置的 BGP 對等。

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet1gw.BgpSettingsText
```

最後一個命令會顯示 Azure VPN 閘道上對應的 BGP 組態；例如︰

```powershell
$vnet1gw.BgpSettingsText
{
    "Asn": 65010,
    "BgpPeeringAddress": "10.12.255.30",
    "PeerWeight": 0
}
```

建立閘道後，您可以使用此閘道來建立跨單位連線或與 BGP 的 VNet 對 VNet 連線。 下列各節將逐步說明完成練習的步驟。

## <a name ="crossprembbgp"></a>第 2 部份 – 建立與 BGP 的跨單位連線

若要建立跨單位連線，您需要建立區域網路閘道來代表您的內部部署 VPN 裝置，以及建立一個連線來連線 VPN 閘道與區域網路閘道。 許多文章會引導您完成這些步驟，本文則會包含指定 BGP 組態參數所需的其他屬性。

![跨單位的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)

在繼續之前，請確定您已完成本練習的[第 1 部分](#enablebgp)。

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>步驟 1 - 建立及設定區域網路閘道

#### <a name="1-declare-your-variables"></a>1.宣告變數

本練習將繼續建置圖中所示的組態。 請務必使用您想用於設定的值來取代該值。

```powershell
$RG5 = "TestBGPRG5"
$Location5 = "East US 2"
$LNGName5 = "Site5"
$LNGPrefix50 = "10.52.255.254/32"
$LNGIP5 = "Your_VPN_Device_IP"
$LNGASN5 = 65050
$BGPPeerIP5 = "10.52.255.254"
```

關於區域網路閘道參數，有幾件事要注意︰

* 區域網路閘道可以位於與 VPN 閘道相同或不同的位置和資源群組中。 此範例會顯示它們位於不同位置的不同資源群組中。
* 您需要針對區域網路閘道宣告的最小前置詞是 VPN 裝置上 BGP 對等 IP 位址的主機位址。 在此情況下是 "10.52.255.254/32" 的前置詞 /32。
* 提醒您，您必須在內部部署網路與 Azure VNet 之間使用不同的 BGP ASN。 在兩者相同的情況下，如果內部部署 VPN 裝置已經使用 ASN 來與其他 BGP 芳鄰建立對等連線，您就需要變更您的 VNet ASN。

繼續之前，請先確定您仍然與訂用帳戶 1 保持連線。

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2.建立 Site5 的區域網路閘道

如果尚未建立資源群組，請務必加以建立，才能建立區域網路閘道。 請注意，區域網路閘道的兩個額外參數︰Asn 與 BgpPeerAddress。

```powershell
New-AzureRmResourceGroup -Name $RG5 -Location $Location5

New-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>步驟 2 - 連接 VNet 閘道與區域網路閘道

#### <a name="1-get-the-two-gateways"></a>1.取得兩個閘道

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2.建立 TestVNet1 至 Site5 的連線

在此步驟中，您將建立從 TestVNet1 至 Site5 的連線。 您必須指定 "-EnableBGP $True" 才能為此連線啟用 BGP。 如先前所討論，相同的 Azure VPN 閘道可以同時有 BGP 和非 BGP 連線。 除非已在連接屬性中啟用 BGP，否則即使已在兩個閘道上設定 BGP 參數，Azure 也不會為此連線啟用 BGP。

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

下列範例所列出的參數，是您要針對此練習，在內部部署 VPN 裝置的 BGP 組態區段中加以輸入︰

```

- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP    : 10.12.255.30
- Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

幾分鐘後就會建立連線，而一旦建立 IPsec 連線後，BGP 對等工作階段就會啟動。

## <a name ="v2vbgp"></a>第 3 部份 – 建立與 BGP 的 VNet 對 VNet 連線

本節新增與 BGP 的 VNet 對 VNet 連線，如下圖所示：

![VNet 對 VNet 的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

以下指示延續自從上面所列的先前步驟。 您必須完成 [I 部分](#enablebgp) ，以建立並設定 TestVNet1 和採用 BGP 的 VPN 閘道。 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>步驟 1 - 建立 TestVNet2 和 VPN 閘道

請務必確定新虛擬網路的 IP 位址空間 TestVNet2 不會與任何 VNet 範圍重疊。

在此範例中，虛擬網路屬於相同的訂用帳戶。 您可以設定不同訂用帳戶之間的 VNet 對 VNet 連線。 如需詳細資訊，請參閱[設定 VNet 對 VNet 連線](vpn-gateway-vnet-vnet-rm-ps.md)。 請務必在建立連線時新增 "-EnableBgp $True"，才能啟用 BGP。

#### <a name="1-declare-your-variables"></a>1.宣告變數

請務必使用您想用於設定的值來取代該值。

```powershell
$RG2 = "TestBGPRG2"
$Location2 = "West US"
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
$GWIPName2 = "VNet2GWIP"
$GWIPconfName2 = "gwipconf2"
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

#### <a name="3-create-the-vpn-gateway-for-testvnet2-with-bgp-parameters"></a>3.使用 BGP 參數建立 TestVNet2 的 VPN 閘道

要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的閘道，並定義必要的子網路和 IP 組態。

```powershell
$gwpip2    = New-AzureRmPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2     = Get-AzureRmVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2   = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName2 -Subnet $subnet2 -PublicIpAddress $gwpip2
```

透過 AS 號碼建立 VPN 閘道。 您必須覆寫您 Azure VPN 閘道上的預設 ASN。 已連接 VNet 的 ASN 必須不同，才能啟用 BGP 與傳輸路由。

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gwipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku Standard -Asn $VNet2ASN
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

在此步驟中，您要建立從 TestVNet1 到 TestVNet2 的連線，以及從 TestVNet2 到 TestVNet1 的連線。

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> 請務必對這兩個連線啟用 BGP。
> 
> 

完成這些步驟之後，會在幾分鐘後建立連線。 一旦 VNet 對 VNet 連線完成後，就會啟動 BGP 對等互連工作階段。

如果您完成此練習的上述三個部分，就已建立下列網路拓撲︰

![VNet 對 VNet 的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

## <a name="next-steps"></a>後續步驟

一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。
