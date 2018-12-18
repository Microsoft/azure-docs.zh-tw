---
title: 設定 S2S VPN 或 VNet 對 VNet 連線的 IPsec/IKE 原則：Azure Resource Manager：PowerShell | Microsoft Docs
description: 使用 Azure Resource Manager 和 PowerShell，設定與 Azure VPN 閘道之 S2S 或 VNet 對 VNet 連線的 IPsec/IKE 原則。
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
ms.date: 02/14/2018
ms.author: yushwang
ms.openlocfilehash: fa1aed76f63e500a6c2849fb9b62a918e85c9fb0
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/20/2018
ms.locfileid: "31601146"
---
# <a name="configure-ipsecike-policy-for-s2s-vpn-or-vnet-to-vnet-connections"></a>設定 S2S VPN 或 VNet 對 VNet 連線的 IPsec/IKE 原則

本文將逐步引導您使用 Resource Manager 部署模型和 PowerShell，設定站對站 VPN 或 VNet 對 VNet 連線的 IPsec/IKE 原則。

## <a name="about"></a>關於 Azure VPN 閘道的 IPsec 和 IKE 原則參數
IPsec 和 IKE 通訊協定標準支援各種不同的密碼編譯演算法的各種組合。 請參閱[關於密碼編譯需求和 Azure VPN 閘道](vpn-gateway-about-compliance-crypto.md)，以查看這如何協助確保跨單位和 VNet 對 VNet 連線滿足合規性或安全性需求。

本文提供指示來建立和設定 IPsec/IKE 原則並套用至新的或現有連線：

* [第 1 部分 - 建立和設定 IPsec/IKE 原則的工作流程](#workflow)
* [第 2 部分 - 支援的密碼編譯演算法和金鑰長度](#params)
* [第 3 部分 - 使用 IPsec/IKE 原則建立新的 S2S VPN 連線](#crossprem)
* [第 4 部分 - 使用 IPsec/IKE 原則建立新的 VNet 對 VNet 連線](#vnet2vnet)
* [第 5 部分 - 管理 (建立、新增、移除) 連線的 IPsec/IKE 原則](#managepolicy)

> [!IMPORTANT]
> 1. 請注意，IPsec/IKE 原則僅適用於下列閘道 SKU：
>    * ***VpnGw1、VpnGw2、VpnGw3*** (路由式)
>    * ***Standard*** 和 ***HighPerformance*** (路由式)
> 2. 每個給定的連線只能指定***一個***原則組合。
> 3. 您必須同時對 IKE (主要模式) 和 IPsec (快速模式) 指定所有的演算法和參數。 系統不允許只指定一部分原則。
> 4. 請確認 VPN 裝置廠商規格，確保內部部署 VPN 裝置支援原則。 如果原則不相容，則無法建立 S2S 或 VNet 對 VNet 連線。

## <a name ="workflow"></a>第 1 部分 - 建立和設定 IPsec/IKE 原則的工作流程
本節所概述的工作流程可在 S2S VPN 或 VNet 對 VNet 連線上建立和更新 IPsec/IKE 原則：
1. 建立虛擬網路和 VPN 閘道
2. 建立區域網路閘道以進行跨單位連線，或建立另一個虛擬網路和閘道以進行 VNet 對 VNet 連線。
3. 使用選取的演算法和參數建立 IPsec/IKE 原則
4. 使用 IPsec/IKE 原則建立連線 (IPsec 或 VNet2VNet)
5. 新增/更新/移除現有連線的 IPsec/IKE 原則

本文中的指示將會協助您安裝並設定 IPsec/IKE 原則，如圖所示：

![ipsec-ike-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)

## <a name ="params"></a>第 2 部分 - 支援的密碼編譯演算法和金鑰長度

下表列出可供客戶設定的支援密碼編譯演算法和金鑰強度：

| **IPsec/IKEv2**  | **選項**    |
| ---  | --- 
| IKEv2 加密 | AES256、AES192、AES128、DES3、DES  
| IKEv2 完整性  | SHA384、SHA256、SHA1、MD5  |
| DH 群組         | DHGroup24、ECP384、ECP256、DHGroup14、DHGroup2048、DHGroup2、DHGroup1、無 |
| IPsec 加密 | GCMAES256、GCMAES192、GCMAES128、AES256、AES192、AES128、DES3、DES、無    |
| IPsec 完整性  | GCMASE256、GCMAES192、GCMAES128、SHA256、SHA1、MD5 |
| PFS 群組        | PFS24、ECP384、ECP256、PFS2048、PFS2、PFS1、無 
| QM SA 存留期   | (**選擇性**：如果未指定，即會使用預設值)<br>秒 (整數；**最小 300**/預設值 27000 秒)<br>KB 數 (整數；**最小 1024**/預設值 102400000 KB 數)   |
| 流量選取器 | UsePolicyBasedTrafficSelectors** ($True/$False；**選擇性**，如果未指定，即為預設的 $False)    |
|  |  |

> [!IMPORTANT]
> 1. **內部部署 VPN 裝置組態必須符合或包含您在 Azure IPsec/IKE 原則中指定的下列演算法和參數︰**
>    * IKE 加密演算法 (主要模式/第 1 階段)
>    * IKE 完整性演算法 (主要模式/第 1 階段)
>    * DH 群組 (主要模式/第 1 階段)
>    * IPsec 加密演算法 (快速模式/第 2 階段)
>    * IPsec 完整性演算法 (快速模式/第 2 階段)
>    * PFS 群組 (快速模式/第 2 階段)
>    * 流量選取器 (如果使用 UsePolicyBasedTrafficSelectors)
>    * SA 存留期只需要在本機指定，不需要相符。
>
> 2. **如果 GCMAES 會用於 IPsec 加密演算法，您必須基於 IPsec 完整性選取相同的 GCMAES 演算法和金鑰長度；例如，針對這兩者使用 GCMAES128**
> 3. 在上表中：
>    * IKEv2 會對應到主要模式或第 1 階段
>    * IPsec 會對應到快速模式或第 2 階段
>    * DH 群組會指定在主要模式或第 1 階段中使用的 Diffie-Hellmen 群組
>    * PFS 群組會指定在快速模式或第 2 階段中使用的 Diffie-Hellmen 群組
> 4. Azure VPN 閘道的 IKEv2 主要模式 SA 存留期會固定為 28,800 秒
> 5. 在連線上將 "UsePolicyBasedTrafficSelectors" 設定為 $True，將設定 Azure VPN 閘道連線至內部部署的原則式 VPN 防火牆。 如果您啟用 PolicyBasedTrafficSelectors，則必須確定 VPN 裝置的流量選取器已定義內部部署網路 (區域網路閘道) 前置詞往/返 Azure 虛擬網路前置詞的所有組合，而不是任意對任意的組合。 例如，如果內部部署網路的前置詞為 10.1.0.0/16 和 10.2.0.0/16，而虛擬網路的前置詞為 192.168.0.0/16 和 172.16.0.0/16，則需要指定下列流量選取器︰
>    * 10.1.0.0/16 <====> 192.168.0.0/16
>    * 10.1.0.0/16 <====> 172.16.0.0/16
>    * 10.2.0.0/16 <====> 192.168.0.0/16
>    * 10.2.0.0/16 <====> 172.16.0.0/16

如需原則式流量選取器的詳細資訊，請參閱[連線多個內部部署原則式 VPN 裝置](vpn-gateway-connect-multiple-policybased-rm-ps.md)。

下表列出自訂原則所支援的對應 Diffie-Hellman 群組：

| **Diffie-Hellman 群組**  | **DHGroup**              | **PFSGroup** | **金鑰長度** |
| --- | --- | --- | --- |
| 1                         | DHGroup1                 | PFS1         | 768 位元 MODP   |
| 2                         | DHGroup2                 | PFS2         | 1024 位元 MODP  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | 2048 位元 MODP  |
| 19                        | ECP256                   | ECP256       | 256 位元 ECP    |
| 20                        | ECP384                   | ECP284       | 384 位元 ECP    |
| 24                        | DHGroup24                | PFS24        | 2048 位元 MODP  |

如需詳細資訊，請參閱 [RFC3526](https://tools.ietf.org/html/rfc3526) 和 [RFC5114](https://tools.ietf.org/html/rfc5114)。

## <a name ="crossprem"></a>第 3 部分 - 使用 IPsec/IKE 原則建立新的 S2S VPN 連線

本節將逐步引導您使用 IPsec/IKE 原則建立 S2S VPN 連線。 下列步驟將建立連線，如圖所示：

![s2s-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/s2spolicy.png)

如需建立 S2S VPN 連線的詳細逐步指示，請參閱[建立 S2S VPN 連線](vpn-gateway-create-site-to-site-rm-powershell.md)。

### <a name="before"></a>開始之前

* 請確認您有 Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* 您必須安裝 Azure Resource Manager PowerShell Cmdlet。 如需安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

### <a name="createvnet1"></a>步驟1 - 建立虛擬網路、VPN 閘道和區域網路閘道

#### <a name="1-declare-your-variables"></a>1.宣告變數

對於此練習，我們一開始先宣告我們的變數。 請務必在設定生產環境時，使用您自己的值來取代該值。

```powershell
$Sub1          = "<YourSubscriptionName>"
$RG1           = "TestPolicyRG1"
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
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GW1IPName1    = "VNet1GWIP1"
$GW1IPconf1    = "gw1ipconf1"
$Connection16  = "VNet1toSite6"

$LNGName6      = "Site6"
$LNGPrefix61   = "10.61.0.0/16"
$LNGPrefix62   = "10.62.0.0/16"
$LNGIP6        = "131.107.72.22"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2.連接至您的訂用帳戶並建立新的資源群組

請確定您切換為 PowerShell 模式以使用資源管理員 Cmdlet。 如需詳細資訊，請參閱 [搭配使用 Windows PowerShell 與 Resource Manager](../powershell-azure-resource-manager.md)。

開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $Sub1
New-AzureRmResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>3.建立虛擬網路、VPN 閘道和區域網路閘道

下列範例會建立有三個子網路的虛擬網路 TestVNet1 和 VPN 閘道。 替代值時，務必一律將您的閘道子網路特定命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

```powershell
$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

$gw1pip1    = New-AzureRmPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1      = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1    = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1

New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1

New-AzureRmLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
```

### <a name="s2sconnection"></a>步驟 2 - 使用 IPsec/IKE 原則建立 S2S VPN 連線

#### <a name="1-create-an-ipsecike-policy"></a>1.建立 IPsec/IKE 原則

下列範例指令碼會使用下列演算法和參數來建立 IPsec/IKE 原則：

* IKEv2：AES256、SHA384、DHGroup24
* IPsec：AES256、SHA256、PFS 無、SA 存留期 14400 秒和 102400000 KB

```powershell
$ipsecpolicy6 = New-AzureRmIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

如果您針對 IPsec 使用 GCMAES，就必須針對 IPsec 加密和完整性使用相同的 GCMAES 演算法和金鑰長度。 就上述範例而言，當使用 GCMAES256 時，對應參數將會是"-IpsecEncryption GCMAES256 -IpsecIntegrity GCMAES256"。

#### <a name="2-create-the-s2s-vpn-connection-with-the-ipsecike-policy"></a>2.使用 IPsec/IKE 原則建立 S2S VPN 連線

建立 S2S VPN 連線，並套用稍早建立的 IPsec/IKE 原則。

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng6 = Get-AzureRmLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
```

您可以選擇性地將 "-UsePolicyBasedTrafficSelectors $True" 新增至建立連線 Cmdlet，讓 Azure VPN 閘道連線至內部部署以原則為基礎的 VPN 裝置，如上所述。

> [!IMPORTANT]
> 在連線上指定 IPsec/IKE 原則之後，Azure VPN 閘道只會傳送或接受該特定連線上的 IPsec/IKE 提案，而提案具有所指定的密碼編譯演算法和金鑰長度。 確定連線的內部部署 VPN 裝置使用或接受確切原則組合，否則不會建立 S2S VPN 通道。


## <a name ="vnet2vnet"></a>第 4 部分 - 使用 IPsec/IKE 原則建立新的 VNet 對 VNet 連線

使用 IPsec/IKE 原則建立 VNet 對 VNet 連線的步驟，與使用 IPsec/IKE 原則建立 S2S VPN 連線的步驟相同。 下列範例指令碼將建立連線，如圖所示：

![v2v-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/v2vpolicy.png)

如需建立 VNet 對 VNet 連線的詳細步驟，請參閱[建立 VNet 對 VNet 連線](vpn-gateway-vnet-vnet-rm-ps.md)。 您必須完成[第 3 部分](#crossprem)，才能建立和設定 TestVNet1 及 VPN 閘道。

### <a name="createvnet2"></a>步驟 1 - 建立第二個虛擬網路和 VPN 閘道

#### <a name="1-declare-your-variables"></a>1.宣告變數

請務必使用您想用於設定的值來取代該值。

```powershell
$RG2          = "TestPolicyRG2"
$Location2    = "East US 2"
$VNetName2    = "TestVNet2"
$FESubName2   = "FrontEnd"
$BESubName2   = "Backend"
$GWSubName2   = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$DNS2         = "8.8.8.8"
$GWName2      = "VNet2GW"
$GW2IPName1   = "VNet2GWIP1"
$GW2IPconf1   = "gw2ipconf1"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-the-second-virtual-network-and-vpn-gateway-in-the-new-resource-group"></a>2.在新的資源群組中建立第二個虛擬網路和 VPN 閘道

```powershell
New-AzureRmResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzureRmVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2

$gw2pip1    = New-AzureRmPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$vnet2      = Get-AzureRmVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2    = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1

New-AzureRmVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
```

### <a name="step-2---create-a-vnet-tovnet-connection-with-the-ipsecike-policy"></a>步驟 2 - 使用 IPsec/IKE 原則建立 VNet 對 VNet 連線

與 S2S VPN 連線類似，請建立 IPsec/IKE 原則，然後將原則套用至新的連線。

#### <a name="1-create-an-ipsecike-policy"></a>1.建立 IPsec/IKE 原則

下列範例指令碼會使用下列演算法和參數來建立不同的 IPsec/IKE 原則：
* IKEv2：AES128、SHA1、DHGroup14
* IPsec：GCMAES128、GCMAES128、PFS14、SA 存留期 14400 秒和 102400000KB

```powershell
$ipsecpolicy2 = New-AzureRmIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption GCMAES128 -IpsecIntegrity GCMAES128 -PfsGroup PFS14 -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

#### <a name="2-create-vnet-to-vnet-connections-with-the-ipsecike-policy"></a>2.使用 IPsec/IKE 原則建立 VNet 對 VNet 連線

建立 VNet 對 VNet 連線，並套用您建立的 IPsec/IKE 原則。 在此範例中，兩個閘道位於相同的訂用帳戶。 因此，可以在相同的 PowerShell 工作階段中建立和設定兩個具有相同 IPsec/IKE 原則的連線。

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$vnet2gw = Get-AzureRmVirtualNetworkGateway -Name $GWName2  -ResourceGroupName $RG2

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'
```

> [!IMPORTANT]
> 在連線上指定 IPsec/IKE 原則之後，Azure VPN 閘道只會傳送或接受該特定連線上的 IPsec/IKE 提案，而提案具有所指定的密碼編譯演算法和金鑰長度。 確定這兩個連線的 IPsec 原則相同，否則不會建立 VNet 對 VNet 連線。

完成這些步驟之後，就會在幾分鐘內建立連線，而且您會有下列網路拓撲，如開頭所示：

![ipsec-ike-policy](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)


## <a name ="managepolicy"></a>第 5 部分 - 更新連線的 IPsec/IKE 原則

最後一節會說明如何管理現有 S2S 或 VNet 對 VNet 連線的 IPsec/IKE 原則。 下列練習會引導您在連線上進行下列作業：

1. 顯示連線的 IPsec/IKE 原則
2. 新增或更新連線的 IPsec/IKE 原則
3. 移除連線的 IPsec/IKE 原則

相同的步驟適用於 S2S 和 VNet 對 VNet 連線。

> [!IMPORTANT]
> 只有 Standard 和 HighPerformance 以路由為基礎的 VPN 閘道才支援 IPsec/IKE 原則。 它不適用於 Basic 閘道 SKU 或以原則為基礎的 VPN 閘道。

#### <a name="1-show-the-ipsecike-policy-of-a-connection"></a>1.顯示連線的 IPsec/IKE 原則

下列範例示範如何取得連線上所設定的 IPsec/IKE 原則。 指令碼也會從上述練習繼續。

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

最後一個命令會列出連線上所設定的目前 IPsec/IKE 原則 (如果有的話)。 以下是連線的範例輸出：

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES256
IkeIntegrity        : SHA384
DhGroup             : DHGroup24
PfsGroup            : PFS24
```

如果未設定任何 IPsec/IKE 原則，命令 (PS> $connection6.policy) 會收到空白傳回。 這並不表示連線上未設定 IPsec/IKE，而是沒有自訂 IPsec/IKE 原則。 實際連線會使用內部部署 VPN 裝置與 Azure VPN 閘道之間交涉的預設原則。

#### <a name="2-add-or-update-an-ipsecike-policy-for-a-connection"></a>2.新增或更新連線的 IPsec/IKE 原則

在連線上新增原則或更新現有原則的步驟相同：建立新的原則，然後將新的原則套用至連線。

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$newpolicy6   = New-AzureRmIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000

Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6
```

若要在連線至內部部署以原則為基礎的 VPN 裝置時啟用 "UsePolicyBasedTrafficSelectors"，請將 "-UsePolicyBaseTrafficSelectors" 參數新增至 Cmdlet，或將它設定為 $False 以停用此選項：

```powershell
Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6 -UsePolicyBasedTrafficSelectors $True
```

您可以再次取得連線，以檢查是否已更新原則。

```powershell
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

您應該會在最後一行看到輸出，如下列範例所示：

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES128
IkeIntegrity        : SHA1
DhGroup             : DHGroup14
PfsGroup            : None
```

#### <a name="3-remove-an-ipsecike-policy-from-a-connection"></a>3.移除連線的 IPsec/IKE 原則

移除連線的自訂原則後，Azure VPN 閘道會回復為使用[預設的 IPsec/IKE 提案清單](vpn-gateway-about-vpn-devices.md)，並與內部部署 VPN 裝置重新進行交涉。

```powershell
$RG1           = "TestPolicyRG1"
$Connection16  = "VNet1toSite6"
$connection6   = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$currentpolicy = $connection6.IpsecPolicies[0]
$connection6.IpsecPolicies.Remove($currentpolicy)

Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6
```

您可以使用相同的指令碼，以檢查是否已從連線中移除原則。

## <a name="next-steps"></a>後續步驟

如需以原則為基礎的流量選取器的詳細資訊，請參閱[連線多個內部部署以原則為基礎的 VPN 裝置](vpn-gateway-connect-multiple-policybased-rm-ps.md)。

一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。
