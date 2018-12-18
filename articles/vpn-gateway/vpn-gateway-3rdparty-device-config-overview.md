---
title: 連線到 Azure VPN 閘道的合作夥伴 VPN 裝置設定 | Microsoft Docs
description: 本文提供連線到 Azure VPN 閘道之合作夥伴 VPN 裝置設定的概觀。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: ''
ms.assetid: a8bfc955-de49-4172-95ac-5257e262d7ea
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/20/2017
ms.author: yushwang
ms.openlocfilehash: dd9ca3937d688170798c42fce45dbcd7711773d1
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31599606"
---
# <a name="overview-of-partner-vpn-device-configurations"></a>合作夥伴 VPN 裝置設定的概觀
本文提供設定連線到 Azure VPN 閘道之內部部署 VPN 裝置的概觀。 範例 Azure 虛擬網路和 VPN 閘道安裝程式是用來以相同參數連線到不同的內部部署 VPN 裝置設定。

## <a name="device-requirements"></a>裝置需求
Azure VPN 閘道會針對站對站 (S2S) VPN 通道使用標準的 IPsec/IKE 通訊協定組合。 如需 IPsec/IKE 參數的清單，以及 Azure VPN 閘道的密碼編譯演算法，請參閱[關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)。 您也可以針對特定連線指定確切的演算法和金鑰長度，如[關於密碼編譯需求](vpn-gateway-about-compliance-crypto.md)中所述。

## <a name ="singletunnel"></a>單一 VPN 通道
範例中的第一個設定是由 Azure VPN 閘道與內部部署 VPN 裝置之間的單一 S2S VPN 通道所組成。 您可以選擇[透過 VPN 通道設定邊界閘道協定 (BGP)](#bgp)。

![單一 S2S VPN 通道的圖表](./media/vpn-gateway-3rdparty-device-config-overview/singletunnel.png)

如需設定單一 VPN 通道的逐步指示，請參閱[設定站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)。 下列章節指定範例設定的連線參數，並且提供 PowerShell 指令碼以協助您開始使用。

### <a name="connection-parameters"></a>連線參數
本節列出先前章節所述之範例的參數。

| **參數**                | **值**                    |
| ---                          | ---                          |
| 虛擬網路位址首碼        | 10.11.0.0/16<br>10.12.0.0/16 |
| Azure VPN 閘道 IP         | Azure VPN 閘道 IP         |
| 內部部署位址首碼 | 10.51.0.0/16<br>10.52.0.0/16 |
| 內部部署 VPN 裝置 IP    | 內部部署 VPN 裝置 IP    |
| * 虛擬網路 BGP ASN                | 65010                        |
| * Azure BGP 對等 IP           | 10.12.255.30                 |
| * 內部部署 BGP ASN         | 65050                        |
| * 內部部署 BGP 對等 IP     | 10.52.255.254                |

\* 僅適用於 BGP 的選擇性參數。

### <a name="sample-powershell-script"></a>範例 PowerShell 指令碼
本節提供可協助您開始的範例指令碼。 如需詳細指示，請參閱[使用 PowerShell 建立 S2S VPN 連線](vpn-gateway-create-site-to-site-rm-powershell.md)。

```powershell
# Declare your variables

$Sub1          = "Replace_With_Your_Subcription_Name"
$RG1           = "TestRG1"
$Location1     = "East US 2"
$VNetName1     = "TestVNet1"
$FESubName1    = "FrontEnd"
$BESubName1    = "Backend"
$GWSubName1    = "GatewaySubnet"
$VNetPrefix11  = "10.11.0.0/16"
$VNetPrefix12  = "10.12.0.0/16"
$FESubPrefix1  = "10.11.0.0/24"
$BESubPrefix1  = "10.12.0.0/24"
$GWSubPrefix1  = "10.12.255.0/27"
$VNet1ASN      = 65010
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GWIPName1     = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection15  = "VNet1toSite5"
$LNGName5      = "Site5"
$LNGPrefix50   = "10.52.255.254/32"
$LNGPrefix51   = "10.51.0.0/16"
$LNGPrefix52   = "10.52.0.0/16"
$LNGIP5        = "Your_VPN_Device_IP"
$LNGASN5       = 65050
$BGPPeerIP5    = "10.52.255.254"

# Connect to your subscription and create a new resource group

Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $Sub1
New-AzureRmResourceGroup -Name $RG1 -Location $Location1

# Create virtual network

$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 $besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

# Create VPN gateway

$gwpip1    = New-AzureRmPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1     = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1   = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1

New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN

# Create local network gateway

New-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix51,$LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5

# Create the S2S VPN connection

$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False
```

### <a name ="policybased"></a>(選擇性) 搭配 UsePolicyBasedTrafficSelectors 使用自訂的 IPsec/IKE 原則
如果您的 VPN 裝置不支援任意-到-任意的流量選取器 (例如，路由式或 VTI 式設定)，請搭配 [UsePolicyBasedTrafficSelectors](vpn-gateway-connect-multiple-policybased-rm-ps.md) 選項建立自訂的 IPsec/IKE 原則。

> [!IMPORTANT]
> 您必須建立 IPsec/IKE 原則，才能在連線上啟用 **UsePolicyBasedTrafficSelectors** 選項。


範例指令碼會使用下列演算法和參數來建立 IPsec/IKE 原則：
* IKEv2：AES256、SHA384、DHGroup24
* IPsec：AES256、SHA1、PFS24、SA 存留期 7200 秒和 20480000KB (20GB)

指令碼會套用 IPsec/IKE 原則，並且在連線上啟用 **UsePolicyBasedTrafficSelectors** 選項。

```powershell
$ipsecpolicy5 = New-AzureRmIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA1 -PfsGroup PFS24 -SALifeTimeSeconds 7200 -SADataSizeKilobytes 20480000

$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False -IpsecPolicies $ipsecpolicy5 -UsePolicyBasedTrafficSelectors $True
```

### <a name ="bgp"></a>(選擇性) 在 S2S VPN 連線使用 BGP
當您建立 S2S VPN 連線時，您可以選擇性地使用[適用於 VPN 閘道的 BGP](vpn-gateway-bgp-resource-manager-ps.md)。 這個方法有兩個差異：

* 內部部署位址首碼可以是單一主機位址。 內部部署 BGP 對等 IP 位址指定如下：

    ```powershell
    New-AzureRmLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
    ```

* 當您建立連線時，您必須將 **-EnableBGP** 選項設定為 $True：

    ```powershell
    New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
    ```

## <a name="next-steps"></a>後續步驟
如需設定主動-主動 VPN 閘道的逐步指示，請參閱[設定跨單位和 VNet 對 VNet 連線的主動-主動 VPN 閘道](vpn-gateway-activeactive-rm-powershell.md)。

