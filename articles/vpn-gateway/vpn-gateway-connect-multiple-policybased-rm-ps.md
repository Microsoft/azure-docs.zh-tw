---
title: 將 Azure VPN 閘道連線至多個內部部署以原則為基礎的 VPN 裝置：Azure Resource Manager：PowerShell | Microsoft Docs
description: 使用 Azure Resource Manager 和 PowerShell，將以 Azure 路由為基礎的 VPN 閘道設定為多個以原則為基礎的 VPN 裝置。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: yushwang
ms.openlocfilehash: dc2dc660262cec892270f8d6e70691fdd169a5c4
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31601928"
---
# <a name="connect-azure-vpn-gateways-to-multiple-on-premises-policy-based-vpn-devices-using-powershell"></a>使用 PowerShell 將 Azure VPN 閘道連線至多個內部部署以原則為基礎的 VPN 裝置

本文將協助您運用 S2S VPN 連線上的自訂 IPsec/IKE 原則，設定以 Azure 路由為基礎的 VPN 閘道連線至多個內部部署以原則為基礎的 VPN 裝置。

## <a name="about-policy-based-and-route-based-vpn-gateways"></a>關於以原則為基礎的 VPN 閘道和以路由為基礎的 VPN 閘道

以原則為基礎的 VPN 裝置「與」以路由為基礎的 VPN 裝置，其差異在於如何於連線上設定 IPsec 流量選取器：

* **以原則為基礎的** VPN 裝置使用這兩個網路的前置詞組合，定義如何透過 IPsec 通道來加密/解密流量。 它通常內建在執行封包篩選的防火牆裝置上。 IPsec 通道加密和解密會新增至封包篩選和處理引擎。
* **以路由為基礎的** VPN 裝置使用任何對任何 (萬用字元) 流量選取器，並讓路由/轉接資料表將流量導向到不同 IPsec 通道。 它通常內建在路由器平台上，其中，每個 IPsec 通道都會建模為網路介面或 VTI (虛擬通道介面)。

下列各圖反白顯示兩個模型：

### <a name="policy-based-vpn-example"></a>以原則為基礎的 VPN 範例
![以原則為基礎](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedmultisite.png)

### <a name="route-based-vpn-example"></a>以路由為基礎的 VPN 範例
![以路由為基礎](./media/vpn-gateway-connect-multiple-policybased-rm-ps/routebasedmultisite.png)

### <a name="azure-support-for-policy-based-vpn"></a>以原則為基礎的 VPN 的 Azure 支援
Azure 目前支援兩種 VPN 閘道模式：以路由為基礎的 VPN 閘道和以原則為基礎的 VPN 閘道。 它們內建在不同內部平台上，因而導致不同的規格：

|                          | **PolicyBased VPN 閘道** | **RouteBased VPN 閘道**               |
| ---                      | ---                         | ---                                      |
| **Azure 閘道 SKU**    | 基本                       | Basic、Standard、HighPerformance、VpnGw1、VpnGw2、VpnGw3 |
| **IKE 版本**          | IKEv1                       | IKEv2                                    |
| **最大S2S 連線** | **1**                       | 基本/標準：10<br> 高效能：30 |
|                          |                             |                                          |

您現在可以使用自訂 IPsec/IKE 原則，設定以 Azure 路由為基礎的 VPN 閘道搭配使用以前置詞為基礎的流量選取器與 "**PolicyBasedTrafficSelectors**" 選項，以連線至內部部署以原則為基礎的 VPN 裝置。 這項功能可讓您從 Azure 虛擬網路和 VPN 閘道連線至多個內部部署以原則為基礎的 VPN/防火牆裝置，並從目前以 Azure 原則為基礎的 VPN 閘道移除單一連線限制。

> [!IMPORTANT]
> 1. 若要啟用這個連線，內部部署以原則為基礎的 VPN 裝置必須支援 **IKEv2** 連線至以 Azure 路由為基礎的 VPN 閘道。 確認您的 VPN 裝置規格。
> 2. 使用這種機制透過以原則為基礎的 VPN 裝置所連線的內部部署網路，只能連線至 Azure 虛擬網路；**其無法透過相同的 Azure VPN 閘道轉換到其他內部部署網路或虛擬網路**。
> 3. 設定選項是自訂 IPsec/IKE 連線原則的一部分。 如果您啟用以原則為基礎的流量選取器選項，則必須指定完整原則 (IPsec/IKE 加密及完整性演算法、金鑰長度和 SA 存留期)。

下圖顯示透過 Azure VPN 閘道的轉換路由為何不適用於以原則為基礎的選項：

![以原則為基礎的傳輸](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedtransit.png)

如圖所示，Azure VPN 閘道會有從虛擬網路到每個內部部署網路前置詞的流量選取器，但不是跨連線前置詞。 例如，內部部署網站 2、網站 3 和網站 4 分別可以與 VNet1 通訊，但無法透過 Azure VPN 閘道彼此連線。 此圖顯示此設定下不適用於 Azure VPN 閘道的跨連線流量選取器。

## <a name="configure-policy-based-traffic-selectors-on-a-connection"></a>在連線上設定以原則為基礎的流量選取器

本文中的指示遵循[設定 S2S 或 VNet 對 VNet 連線的 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)中所述的相同範例，以建立 S2S VPN 連線。 如下圖所示：

![s2s-policy](./media/vpn-gateway-connect-multiple-policybased-rm-ps/s2spolicypb.png)

啟用這個連線的工作流程：
1. 建立跨單位連線的虛擬網路、VPN 閘道和區域網路閘道
2. 建立 IPsec/IKE 原則
3. 如果您建立 S2S 或 VNet 對 VNet 連線，並在連線上**啟用以原則為基礎的流量選取器**，請套用原則。
4. 如果已經建立連線，您可以套用原則，或將其更新為現有連線。

## <a name="enable-policy-based-traffic-selectors-on-a-connection"></a>在連線上啟用以原則為基礎的流量選取器

請確定您已針對本節完成[設定 IPsec/IKE 原則文章的第 3 部分](vpn-gateway-ipsecikepolicy-rm-powershell.md)。 下列範例會使用相同的參數和步驟：

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>步驟1 - 建立虛擬網路、VPN 閘道和區域網路閘道

#### <a name="1-declare-your-variables--connect-to-your-subscription"></a>1.宣告變數，並連線到您的訂用帳戶
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
請確定您切換為 PowerShell 模式以使用 Resource Manager Cmdlet。 如需詳細資訊，請參閱 [搭配使用 Windows PowerShell 與 Resource Manager](../powershell-azure-resource-manager.md)。

開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $Sub1
New-AzureRmResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="2-create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>2.建立虛擬網路、VPN 閘道和區域網路閘道
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

New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance

New-AzureRmLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
```

### <a name="step-2---create-a-s2s-vpn-connection-with-an-ipsecike-policy"></a>步驟 2 - 使用 IPsec/IKE 原則建立 S2S VPN 連線

#### <a name="1-create-an-ipsecike-policy"></a>1.建立 IPsec/IKE 原則

> [!IMPORTANT]
> 您需要建立 IPsec/IKE 原則，才能在連線上啟用 "UsePolicyBasedTrafficSelectors" 選項。

下列範例會使用這些演算法和參數來建立 IPsec/IKE 原則：
* IKEv2：AES256、SHA384、DHGroup24
* IPsec：AES256、SHA256、PFS24、SA 存留期 3600 秒和 2048KB

```powershell
$ipsecpolicy6 = New-AzureRmIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup PFS24 -SALifeTimeSeconds 3600 -SADataSizeKilobytes 2048
```

#### <a name="2-create-the-s2s-vpn-connection-with-policy-based-traffic-selectors-and-ipsecike-policy"></a>2.使用以原則為基礎的流量選取器和 IPsec/IKE 原則來建立 S2S VPN 連線
建立 S2S VPN 連線，並套用前面步驟所建立的 IPsec/IKE 原則。 請注意，使用額外參數 "-UsePolicyBasedTrafficSelectors $True"，可在連線上啟用以原則為基礎的流量選取器。

```powershell
$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng6 = Get-AzureRmLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

New-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -UsePolicyBasedTrafficSelectors $True -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
```

完成步驟後，S2S VPN 連線將會使用所定義的 IPsec/IKE 原則，並在連線上啟用以原則為基礎的流量選取器。 您可以重複相同的步驟，以從相同的 Azure VPN 閘道將更多的連線新增至其他內部部署以原則為基礎的 VPN 裝置。

## <a name="update-policy-based-traffic-selectors-for-a-connection"></a>更新連線的以原則為基礎的流量選取器
最後一節將示範如何更新現有 S2S VPN 連線的以原則為基礎的流量選取器選項。

### <a name="1-get-the-connection"></a>1.取得連線
取得連線資源。

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
```

### <a name="2-check-the-policy-based-traffic-selectors-option"></a>2.檢查以原則為基礎的流量選取器選項
下行將說明是否將以原則為基礎的流量選取器用於連線：

```powershell
$connection6.UsePolicyBasedTrafficSelectors
```

如果該行傳回 "**True**"，則已在連線上設定以原則為基礎的流量選取器；否則，它會傳回 "**False**"。

### <a name="3-update-the-policy-based-traffic-selectors-on-a-connection"></a>3.更新連線上的以原則為基礎的流量選取器
在您取得連線資源之後，即可啟用或停用此選項。

#### <a name="disable-usepolicybasedtrafficselectors"></a>停用 UsePolicyBasedTrafficSelectors
下列範例會停用以原則為基礎的流量選取器選項，但 IPsec/IKE 原則保留不變：

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $False
```

#### <a name="enable-usepolicybasedtrafficselectors"></a>啟用 UsePolicyBasedTrafficSelectors
下列範例會啟用以原則為基礎的流量選取器選項，但 IPsec/IKE 原則保留不變：

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzureRmVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $True
```

## <a name="next-steps"></a>後續步驟
一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。

如需自訂 IPsec/IKE 原則的詳細資訊，也請檢閱[設定 S2S VPN 或 VNet 對 VNet 連線的 IPsec/IKE 原則](vpn-gateway-ipsecikepolicy-rm-powershell.md)。
