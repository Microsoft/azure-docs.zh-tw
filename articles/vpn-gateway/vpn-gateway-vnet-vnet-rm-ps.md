---
title: 使用 VNet 對 VNet 連線將 Azure 虛擬網路連線至另一個 VNet︰PowerShell | Microsoft Docs
description: 使用 VNet 對 VNet 連線和 PowerShell，將虛擬網路連在一起。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: 0683c664-9c03-40a4-b198-a6529bf1ce8b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: cherylmc
ms.openlocfilehash: a96326e23a2ed913ccab8a23666a4f408157d3da
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31603975"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-powershell"></a>使用 PowerShell 設定 VNet 對 VNet 的 VPN 閘道連線

本文協助您使用 VNet 對 VNet 連線類型來連線虛擬網路。 虛擬網路可位於相同或不同的區域，以及來自相同或不同的訂用帳戶。 連線來自不同訂用帳戶的 VNet 時，訂用帳戶不需與相同的 Active Directory 租用戶相關聯。

本文中的步驟適用於 Resource Manager 部署模型並使用 PowerShell。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [連線不同的部署模型 - Azure 入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [連線不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about"></a>關於連線 VNet

VNet 的連線方法有很多種。 下列各節說明不同的虛擬網路連線方式。

### <a name="vnet-to-vnet"></a>VNet 對 VNet

設定 VNet 對 VNet 連線是輕鬆連線 VNet 的好方法。 使用 VNet 對 VNet 連線類型 (VNet2VNet) 將虛擬網路連線到另一個虛擬網路，類似於建立內部部署位置的站對站 IPsec 連線。  這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道，且兩者在通訊時的運作方式相同。 連線類型之間的差異在於區域網路閘道的設定方式。 當您建立 VNet 對 VNet 連線時，會看不到區域網路閘道的位址空間。 系統會自動建立並填入該位址空間。 如果您更新一個 VNet 的位址空間，另一個 VNet 就會自動得知要路由到已更新的位址空間。 相較於在 VNet 之間建立站對站連線，建立 VNet 對 VNet 連線通常比較快速而容易。

### <a name="site-to-site-ipsec"></a>站對站 (IPsec)

如果您使用複雜的網路組態，您可能偏好使用[站對站](vpn-gateway-create-site-to-site-rm-powershell.md)步驟 (而不是 VNet 對 VNet 步驟) 來連線 VNet。 當您使用站對站步驟時，您會以手動方式建立及設定區域網路閘道。 每個 VNet 的區域網路閘道都會將其他 VNet 視為本機網站。 這可讓您指定區域網路閘道的其他位址空間，以便路由傳送流量。 如果 VNet 的位址空間變更，您需要更新對應的區域網路閘道，才會反映變更。 它並不會自動更新。

### <a name="vnet-peering"></a>VNet 對等互連

建議您使用 VNet 對等互連來進行 VNet 連線。 VNet 對等互連不會使用 VPN 閘道，且具有不同的條件約束。 此外，[VNet 對等互連價格](https://azure.microsoft.com/pricing/details/virtual-network)與 [VNet 對 VNet VPN 閘道價格](https://azure.microsoft.com/pricing/details/vpn-gateway)的計算方式不同。 如需詳細資訊，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)。

## <a name="why"></a>為何要建立 VNet 對 VNet 連線？

基於下列原因，建議您使用 VNet 對 VNet 連線來進行虛擬網路連線：

* **跨區域的異地備援和異地目前狀態**

  * 您可以使用安全連線設定自己的異地複寫或同步處理，而不用查看網際網路對向端點。
  * 您可以使用 Azure 流量管理員和負載平衡器，利用異地備援跨多個 Azure 區域設定高度可用的工作負載。 其中一個重要的範例就是使用分散在多個 Azure 區域的可用性群組來設定 SQL Always On。
* **具有隔離或管理界限的區域性多層式應用程式**

  * 在相同區域中，您可以因為隔離或管理需求，設定將多層式應用程式與多個虛擬網路連線在一起。

您可以將 VNet 對 VNet 通訊與多站台組態結合。 這可讓您建立結合了跨單位連線與內部虛擬網路連線的網路拓撲。

## <a name="steps"></a>我應該使用哪些 VNet 對 VNet 步驟？

在本文中，您會看到兩組不同的步驟。 一組步驟適用於[位於相同訂用帳戶中的 VNet](#samesub)，一組步驟適用於[位於不同訂用帳戶中的 VNet](#difsub)。
主要差異在於設定位於不同訂用帳戶之 VNet 的連線時，您必須使用個別的 PowerShell 工作階段。 

在此練習中，您可以合併組態，或只選擇您需要使用的一個組態。 所有組態都會使用 VNet 對 VNet 連線類型。 網路流量會在彼此直接連線的 VNet 之間流動。 在此練習中，來自 TestVNet4 的流量不會路由傳送至 TestVNet5。

* [位於相同訂用帳戶中的 VNet：](#samesub)此組態的步驟會使用 TestVNet1 和 TestVNet4。

  ![v2v 圖表](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)

* [位於不同訂用帳戶中的 VNet：](#difsub)此組態的步驟會使用 TestVNet1 和 TestVNet5。

  ![v2v 圖表](./media/vpn-gateway-vnet-vnet-rm-ps/v2vdiffsub.png)

## <a name="samesub"></a>如何連接相同訂用帳戶中的 VNet

### <a name="before-you-begin"></a>開始之前

開始之前，您必須安裝最新版的 Azure Resource Manager PowerShell Cmdlet (至少 4.0 或更新版本)。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。

### <a name="Step1"></a>步驟 1 - 規劃 IP 位址範圍

在下列步驟中，您會建立兩個虛擬網路，以及它們各自的閘道子網路和組態。 接著建立這兩個 VNet 之間的 VPN 連線。 請務必規劃您的網路組態的 IP 位址範圍。 請記住，您必須先確定您的 VNet 範圍或區域網路範圍沒有以任何方式重疊。 在這些範例中，我們不會包含 DNS 伺服器。 如果您想要了解虛擬網路的名稱解析，請參閱[名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

我們會在範例中使用下列值：

**TestVNet1 的值︰**

* VNet 名稱︰TestVNet1
* 資源群組︰TestRG1
* 位置：美國東部
* TestVNet1：10.11.0.0/16 和 10.12.0.0/16
* FrontEnd：10.11.0.0/24
* BackEnd：10.12.0.0/24
* GatewaySubnet：10.12.255.0/27
* GatewayName：VNet1GW
* 公用 IP: VNet1GWIP
* VPNType：RouteBased
* Connection(1to4)：VNet1toVNet4
* Connection(1to5)：VNet1toVNet5 (適用於不同訂用帳戶中的 Vnet)
* ConnectionType：VNet2VNet

**TestVNet4 的值︰**

* VNet 名稱︰TestVNet4
* TestVNet2：10.41.0.0/16 和 10.42.0.0/16
* FrontEnd：10.41.0.0/24
* BackEnd：10.42.0.0/24
* GatewaySubnet：10.42.255.0/27
* 資源群組：TestRG4
* 位置：美國西部
* GatewayName：VNet4GW
* 公用 IP：VNet4GWIP
* VPNType：RouteBased
* 連線︰VNet4toVNet1
* ConnectionType：VNet2VNet


### <a name="Step2"></a>步驟 2 - 建立及設定 TestVNet1

1. 宣告變數。 下列範例會使用此練習中的值來宣告變數。 在大部分的情況下，您應將值取代為您自己的值。 不過，若您執行這些步驟是為了熟悉此類型的設定，則可以使用這些變數。 視需要修改變數，然後將其複製並貼到您的 PowerShell 主控台中。

  ```powershell
  $Sub1 = "Replace_With_Your_Subcription_Name"
  $RG1 = "TestRG1"
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
  $GWName1 = "VNet1GW"
  $GWIPName1 = "VNet1GWIP"
  $GWIPconfName1 = "gwipconf1"
  $Connection14 = "VNet1toVNet4"
  $Connection15 = "VNet1toVNet5"
  ```

2. 連線至您的帳戶。 使用下列範例來協助您連接：

  ```powershell
  Connect-AzureRmAccount
  ```

  檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```

  指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName $Sub1
  ```
3. 建立新的資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name $RG1 -Location $Location1
  ```
4. 建立 TestVNet1 的子網路設定。 此範例會建立一個名為 TestVNet1 的虛擬網路和三個子網路：一個名為 GatewaySubnet、一個名為 FrontEnd，另一個名為 Backend。 替代值時，務必一律將您的閘道子網路特定命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

  下列範例會使用您先前設定的變數。 在此範例中，閘道子網路使用 /27。 雖然您可以建立小至 /29 的閘道子網路，我們建議您選取至少 /28 或 /27，建立包含更多位址的較大子網路。 這將允許足夠的位址，以容納您未來可能需要的其他組態。

  ```powershell
  $fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
  $besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
  $gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
  ```
5. 建立 TestVNet1。

  ```powershell
  New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 `
  -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
  ```
6. 要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的閘道。 請注意，AllocationMethod 是動態的。 您無法指定想要使用的 IP 位址。 該 IP 位址會以動態方式配置給您的閘道。 

  ```powershell
  $gwpip1 = New-AzureRmPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
  ```
7. 建立閘道組態。 閘道器組態定義要使用的子網路和公用 IP 位址。 使用下列範例來建立閘道組態。

  ```powershell
  $vnet1 = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
  $subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```
8. 建立 TestVNet1 的閘道。 在此步驟中，您會建立 TestVNet1 的虛擬網路閘道。 VNet 對 VNet 組態需要 RouteBased VpnType。 建立閘道通常可能需要 45 分鐘或更久，視選取的閘道 SKU 而定。

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
  -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-3---create-and-configure-testvnet4"></a>步驟 3 - 建立及設定 TestVNet4

設定 TestVNet1 後，請建立 TestVNet4。 遵循下方步驟，視需要替換成您自己的值。 此步驟可以在相同的 PowerShell 工作階段中完成，因為其位於相同的訂用帳戶中。

1. 宣告變數。 請務必使用您想用於設定的值來取代該值。

  ```powershell
  $RG4 = "TestRG4"
  $Location4 = "West US"
  $VnetName4 = "TestVNet4"
  $FESubName4 = "FrontEnd"
  $BESubName4 = "Backend"
  $GWSubName4 = "GatewaySubnet"
  $VnetPrefix41 = "10.41.0.0/16"
  $VnetPrefix42 = "10.42.0.0/16"
  $FESubPrefix4 = "10.41.0.0/24"
  $BESubPrefix4 = "10.42.0.0/24"
  $GWSubPrefix4 = "10.42.255.0/27"
  $GWName4 = "VNet4GW"
  $GWIPName4 = "VNet4GWIP"
  $GWIPconfName4 = "gwipconf4"
  $Connection41 = "VNet4toVNet1"
  ```
2. 建立新的資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name $RG4 -Location $Location4
  ```
3. 建立 TestVNet4 的子網路設定。

  ```powershell
  $fesub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName4 -AddressPrefix $FESubPrefix4
  $besub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName4 -AddressPrefix $BESubPrefix4
  $gwsub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName4 -AddressPrefix $GWSubPrefix4
  ```
4. 建立 TestVNet4。

  ```powershell
  New-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4 `
  -Location $Location4 -AddressPrefix $VnetPrefix41,$VnetPrefix42 -Subnet $fesub4,$besub4,$gwsub4
  ```
5. 要求公用 IP 位址。

  ```powershell
  $gwpip4 = New-AzureRmPublicIpAddress -Name $GWIPName4 -ResourceGroupName $RG4 `
  -Location $Location4 -AllocationMethod Dynamic
  ```
6. 建立閘道組態。

  ```powershell
  $vnet4 = Get-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4
  $subnet4 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet4
  $gwipconf4 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName4 -Subnet $subnet4 -PublicIpAddress $gwpip4
  ```
7. 建立 TestVNet4 閘道。 建立閘道通常可能需要 45 分鐘或更久，視選取的閘道 SKU 而定。

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4 `
  -Location $Location4 -IpConfigurations $gwipconf4 -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-4---create-the-connections"></a>步驟 4 - 建立連線

1. 取得這兩個虛擬網路閘道。 如果這兩個閘道皆位於相同的訂用帳戶中 (如同其在此範例中)，您可以在相同的 PowerShell 工作階段中完成此步驟。

  ```powershell
  $vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
  $vnet4gw = Get-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4
  ```
2. 建立 TestVNet1 至 TestVNet4 的連線。 在此步驟中，您會從 TestVNet1 建立連線至 TestVNet4。 您會看到範例使用共用金鑰。 您可以使用自己的值，作為共用金鑰。 但請務必確認該共用金鑰必須適用於這兩個連線。 建立連線可能需要一段時間才能完成。

  ```powershell
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection14 -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet4gw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
3. 建立 TestVNet4 至 TestVNet1 的連線。 此步驟類似上面的步驟，只不過您是建立 TestVNet4 至 TestVNet1 的連線。 請確認共用的金鑰相符。 稍候幾分鐘就會建立連線。

  ```powershell
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection41 -ResourceGroupName $RG4 `
  -VirtualNetworkGateway1 $vnet4gw -VirtualNetworkGateway2 $vnet1gw -Location $Location4 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
4. 確認您的連線。 請參閱 [如何驗證您的連線](#verify)一節。

## <a name="difsub"></a>如何連接不同訂用帳戶中的 VNet

在此案例中，您會連接 TestVNet1 和 TestVNet5。 TestVNet1 和 TestVNet5 位於不同的訂用帳戶中。 訂用帳戶不需與相同的 Active Directory 租用戶相關聯。 這些步驟與前一組步驟的差別在於，第二個訂用帳戶的內容中有些設定步驟需在不同的 PowerShell 工作階段中執行。 尤其是當兩個訂用帳戶分屬不同的組織時。

### <a name="step-5---create-and-configure-testvnet1"></a>步驟 5 - 建立及設定 TestVNet1

您必須完成前一節的[步驟 1](#Step1) 和[步驟 2](#Step2)，以建立並設定 TestVNet1 和 TestVNet1 的 VPN 閘道。 在此設定中，您不需要建立前一節的 TestVNet4 ，雖然您若建立它，它就不與這些步驟發生衝突。 完成步驟 1 和步驟 2 後，繼續進行步驟 6 以建立 TestVNet5。 

### <a name="step-6---verify-the-ip-address-ranges"></a>步驟 6 - 驗證 IP 位址範圍

請務必確定新虛擬網路的 IP 位址空間 TestVNet5 不會與任何 VNet 範圍或區域網路閘道範圍重疊。 在此範例中，虛擬網路可能屬於不同的組織。 在這個練習中，您可以對 TestVNet5 使用下列的值：

**TestVNet5 的值︰**

* VNet 名稱︰TestVNet5
* 資源群組：TestRG5
* 位置：日本東部
* TestVNet5：10.51.0.0/16 和 10.52.0.0/16
* FrontEnd：10.51.0.0/24
* BackEnd：10.52.0.0/24
* GatewaySubnet：10.52.255.0.0/27
* GatewayName：VNet5GW
* 公用 IP：VNet5GWIP
* VPNType：RouteBased
* 連線︰VNet5toVNet1
* ConnectionType：VNet2VNet

### <a name="step-7---create-and-configure-testvnet5"></a>步驟 7 - 建立及設定 TestVNet5

在新訂用帳戶的內容中，必須完成這個步驟。 此部分可能會由不同組織中擁有訂用帳戶的系統管理員執行。

1. 宣告變數。 請務必使用您想用於設定的值來取代該值。

  ```powershell
  $Sub5 = "Replace_With_the_New_Subcription_Name"
  $RG5 = "TestRG5"
  $Location5 = "Japan East"
  $VnetName5 = "TestVNet5"
  $FESubName5 = "FrontEnd"
  $BESubName5 = "Backend"
  $GWSubName5 = "GatewaySubnet"
  $VnetPrefix51 = "10.51.0.0/16"
  $VnetPrefix52 = "10.52.0.0/16"
  $FESubPrefix5 = "10.51.0.0/24"
  $BESubPrefix5 = "10.52.0.0/24"
  $GWSubPrefix5 = "10.52.255.0/27"
  $GWName5 = "VNet5GW"
  $GWIPName5 = "VNet5GWIP"
  $GWIPconfName5 = "gwipconf5"
  $Connection51 = "VNet5toVNet1"
  ```
2. 連線到訂用帳戶 5。 開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

  ```powershell
  Connect-AzureRmAccount
  ```

  檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```

  指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName $Sub5
  ```
3. 建立新的資源群組。

  ```powershell
  New-AzureRmResourceGroup -Name $RG5 -Location $Location5
  ```
4. 建立 TestVNet5 的子網路設定。

  ```powershell
  $fesub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName5 -AddressPrefix $FESubPrefix5
  $besub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName5 -AddressPrefix $BESubPrefix5
  $gwsub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName5 -AddressPrefix $GWSubPrefix5
  ```
5. 建立 TestVNet5。

  ```powershell
  New-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5 -Location $Location5 `
  -AddressPrefix $VnetPrefix51,$VnetPrefix52 -Subnet $fesub5,$besub5,$gwsub5
  ```
6. 要求公用 IP 位址。

  ```powershell
  $gwpip5 = New-AzureRmPublicIpAddress -Name $GWIPName5 -ResourceGroupName $RG5 `
  -Location $Location5 -AllocationMethod Dynamic
  ```
7. 建立閘道組態。

  ```powershell
  $vnet5 = Get-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5
  $subnet5  = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet5
  $gwipconf5 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName5 -Subnet $subnet5 -PublicIpAddress $gwpip5
  ```
8. 建立 TestVNet5 閘道。

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5 -Location $Location5 `
  -IpConfigurations $gwipconf5 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-8---create-the-connections"></a>步驟 8 - 建立連線

在此範例中，因為閘道會在不同的訂用帳戶中，所以我們已將此步驟分作兩個 PowerShell 工作階段，其標示為 [訂用帳戶 1] 和 [訂用帳戶 5]。

1. **[訂用帳戶 1]** 取得訂用帳戶 1 的虛擬網路閘道。 先登入並連線至訂用帳戶 1，再執行下列範例︰

  ```powershell
  $vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
  ```

  複製下列項目的輸出，並透過電子郵件或其他方法將其傳送到訂用帳戶 5 的系統管理員。

  ```powershell
  $vnet1gw.Name
  $vnet1gw.Id
  ```

  這兩個元素的值會類似下列範例的輸出︰

  ```
  PS D:\> $vnet1gw.Name
  VNet1GW
  PS D:\> $vnet1gw.Id
  /subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroupsTestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
  ```
2. **[訂用帳戶 5]** 取得訂用帳戶 5 的虛擬網路閘道。 先登入並連線至訂用帳戶 5，再執行下列範例︰

  ```powershell
  $vnet5gw = Get-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5
  ```

  複製下列項目的輸出，並透過電子郵件或其他方法將其傳送到訂用帳戶 1 的系統管理員。

  ```powershell
  $vnet5gw.Name
  $vnet5gw.Id
  ```

  這兩個元素的值會類似下列範例的輸出︰

  ```
  PS C:\> $vnet5gw.Name
  VNet5GW
  PS C:\> $vnet5gw.Id
  /subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
  ```
3. **[訂用帳戶 1]** 建立 TestVNet1 至 TestVNet5 的連線。 在此步驟中，您會從 TestVNet1 建立連線至 TestVNet5。 此處的差別為直接取得 $vnet5gw，因為其位於不同的訂用帳戶中。 您必須使用上述步驟中從訂用帳戶 1 通訊的值來建立新的 PowerShell 物件。 請使用下方的範例。 以您自己的值來取代名稱、識別碼和共用金鑰。 但請務必確認該共用金鑰必須適用於這兩個連線。 建立連線可能需要一段時間才能完成。

  先連線至訂用帳戶 1，再執行下列範例︰

  ```powershell
  $vnet5gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
  $vnet5gw.Name = "VNet5GW"
  $vnet5gw.Id   = "/subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW"
  $Connection15 = "VNet1toVNet5"
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet5gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
4. **[訂用帳戶 5]** 建立 TestVNet5 至 TestVNet1 的連線。 此步驟類似上面的步驟，只不過您是建立 TestVNet5 至 TestVNet1 的連線。 針對基於從訂用帳戶 1 所取得的值來建立 PowerShell 物件，該程序也適用於此處。 在此步驟中，請確認共用金鑰相符。

  先連線至訂用帳戶 5，再執行下列範例︰

  ```powershell
  $vnet1gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
  $vnet1gw.Name = "VNet1GW"
  $vnet1gw.Id = "/subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW "
  $Connection51 = "VNet5toVNet1"
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection51 -ResourceGroupName $RG5 -VirtualNetworkGateway1 $vnet5gw -VirtualNetworkGateway2 $vnet1gw -Location $Location5 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```

## <a name="verify"></a>驗證連線

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="faq"></a>VNet 對 VNet 常見問題集

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>後續步驟

* 一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱 [虛擬機器文件](https://docs.microsoft.com/azure/#pivot=services&panel=Compute) 。
* 如需 BGP 的相關資訊，請參閱 [BGP 概觀](vpn-gateway-bgp-overview.md)和[如何設定 BGP](vpn-gateway-bgp-resource-manager-ps.md)。
