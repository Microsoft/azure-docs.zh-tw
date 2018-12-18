---
title: 使用 PowerShell 建立和管理 Azure VPN 閘道 | Microsoft Docs
description: 教學課程 - 使用 Azure PowerShell 模組建立和管理 VPN 閘道
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure
ms.date: 05/14/2018
ms.author: yushwang
ms.custom: mvc
ms.openlocfilehash: b1435773f8d05f9cc730e5745c1a916d9b74321f
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340588"
---
# <a name="create-and-manage-vpn-gateway-with-the-azure-powershell-module"></a>使用 Azure PowerShell 模組建立和管理 VPN 閘道

Azure VPN 閘道提供客戶組織內部與 Azure 之間的跨單位連線。 本教學課程將說明基本的 Azure VPN 閘道部署項目，例如建立和管理 VPN 閘道。 您會了解如何：

> [!div class="checklist"]
> * 建立 VPN 閘道
> * 調整 VPN 閘道的大小
> * 重設 VPN 閘道

下圖顯示在本教學課程中建立的虛擬網路和 VPN 閘道。

![VNet 和 VPN 閘道](./media/vpn-gateway-tutorial-create-gateway-powershell/vnet1-gateway.png)

### <a name="azure-cloud-shell-and-azure-powershell"></a>Azure Cloud Shell 和 Azure PowerShell

[!INCLUDE [working with cloudshell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.3 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。 

## <a name="common-network-parameter-values"></a>一般網路參數值

請根據您的環境和網路設定變更下列值。

```azurepowershell-interactive
$RG1         = "TestRG1"
$VNet1       = "VNet1"
$Location1   = "East US"
$FESubnet1   = "FrontEnd"
$BESubnet1   = "Backend"
$GwSubnet1   = "GatewaySubnet"
$VNet1Prefix = "10.1.0.0/16"
$FEPrefix1   = "10.1.0.0/24"
$BEPrefix1   = "10.1.1.0/24"
$GwPrefix1   = "10.1.255.0/27"
$VNet1ASN    = 65010
$DNS1        = "8.8.8.8"
$Gw1         = "VNet1GW"
$GwIP1       = "VNet1GWIP"
$GwIPConf1   = "gwipconf1"
```

## <a name="create-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 必須先建立資源群組。 在下列範例中，會在 [美國東部] 區域中建立名為 *TestRG1* 的資源群組：

```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName $RG1 -Location $Location1
```

## <a name="create-a-virtual-network"></a>建立虛擬網路

Azure VPN 閘道可為您的虛擬網路提供跨單位連線和 P2S VPN 伺服器功能。 請將 VPN 閘道新增至現有的虛擬網路，或建立新的虛擬網路和閘道。 此範例會使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 和 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立具有三個子網路 (前端、後端和 GatewaySubnet) 的新虛擬網路：

```azurepowershell-interactive
$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubnet1 -AddressPrefix $FEPrefix1
$besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubnet1 -AddressPrefix $BEPrefix1
$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubnet1 -AddressPrefix $GwPrefix1
$vnet   = New-AzureRmVirtualNetwork `
            -Name $VNet1 `
            -ResourceGroupName $RG1 `
            -Location $Location1 `
            -AddressPrefix $VNet1Prefix `
            -Subnet $fesub1,$besub1,$gwsub1
```

## <a name="request-a-public-ip-address-for-the-vpn-gateway"></a>要求 VPN 閘道的公用 IP 位址

Azure VPN 閘道會透過網際網路與您的內部部署 VPN 裝置通訊，以執行 IKE (網際網路金鑰交換) 交涉並建立 IPsec 通道。 請依照下列範例的說明，使用 [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) 和 [New-AzureRmVirtualNetworkGatewayIpConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworkgatewayipconfig) 建立公用 IP 位址，並將其指派給您的 VPN 閘道：

> [!IMPORTANT]
> 您目前只能將動態公用 IP 位址用於該閘道。 Azure VPN 閘道不支援靜態 IP 位址。

```azurepowershell-interactive
$gwpip    = New-AzureRmPublicIpAddress -Name $GwIP1 -ResourceGroupName $RG1 `
              -Location $Location1 -AllocationMethod Dynamic
$subnet   = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' `
              -VirtualNetwork $vnet
$gwipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GwIPConf1 `
              -Subnet $subnet -PublicIpAddress $gwpip
```

## <a name="create-vpn-gateway"></a>建立 VPN 閘道

VPN 閘道建立作業可能需要花費 45 分鐘以上的時間。 閘道建立完成之後，您便可以在您的虛擬網路與另一個 VNet 之間建立連線。 或者，您可以在虛擬網路與內部部署位置之間建立連線。 請使用 [New-AzureRmVirtualNetworkGateway](/powershell/module/azurerm.network/New-AzureRmVirtualNetworkGateway) Cmdlet 來建立 VPN 閘道。

```azurepowershell-interactive
New-AzureRmVirtualNetworkGateway -Name $Gw1 -ResourceGroupName $RG1 `
  -Location $Location1 -IpConfigurations $gwipconf -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
```

金鑰參數值：
* GatewayType：使用 **Vpn** 進行站對站和 VNet 對 VNet 連線
* VpnType：使用 **RouteBased** 與較大範圍的 VPN 裝置和較多路由功能互動
* GatewaySku：**VpnGw1** 是預設值；如果您需要更高的輸送量或更多連線，請將其變更為 VpnGw2 或 VpnGw3。 如需詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

閘道建立完成之後，您便可以在您的虛擬網路與另一個 VNet 之間建立連線，或是在您的虛擬網路與內部部署位置之間建立連線。 您也可以設定從用戶端電腦到 VNet 的 P2S 連線。

## <a name="resize-vpn-gateway"></a>調整 VPN 閘道的大小

您可以在閘道建立後變更 VPN 閘道 SKU。 不同的閘道 SKU 支援不同的規格，例如輸送量、連線數目等。下列範例會使用 [Resize-AzureRmVirtualNetworkGateway](/powershell/module/azurerm.network/Resize-AzureRmVirtualNetworkGateway) 將閘道的大小從 VpnGw2 調整為 VpnGw1。 如需詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

```azurepowershell-interactive
$gw = Get-AzureRmVirtualNetworkGateway -Name $Gw1 -ResourceGroup $RG1
Resize-AzureRmVirtualNetworkGateway -GatewaySku VpnGw2 -VirtualNetworkGateway $gateway
```

雖然為 VPN 閘道調整大小的作業並**不會**干擾或移除現有的連線和組態，但也需要約 30 到 45 分鐘的時間。

## <a name="reset-vpn-gateway"></a>重設 VPN 閘道

在執行疑難排解步驟時，您可以重設 Azure VPN 閘道，以強制 VPN 閘道重新啟動 IPsec/IKE 通道設定。 請使用 [Reset-AzureRmVirtualNetworkGateway](/powershell/module/azurerm.network/Reset-AzureRmVirtualNetworkGateway) 重設您的閘道。

```azurepowershell-interactive
$gw = Get-AzureRmVirtualNetworkGateway -Name $Gw1 -ResourceGroup $RG1
Reset-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gateway
```

如需詳細資訊，請參閱[重設 VPN 閘道](vpn-gateway-resetgw-classic.md)。

## <a name="get-the-gateway-public-ip-address"></a>取得閘道公用 IP 位址

如果您知道公用 IP 位址的名稱，請使用 [Get-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermpublicipaddress?view=azurermps-6.8.1) 來顯示指派給該閘道的公用 IP 位址。

```azurepowershell-interactive
$myGwIp = Get-AzureRmPublicIpAddress -Name $GwIP1 -ResourceGroup $RG1
$myGwIp.IpAddress
```

## <a name="delete-vpn-gateway"></a>刪除 VPN 閘道

要進行完整的跨單位和 VNet 對 VNet 連線設定，除了 VPN 閘道以外，還需要多種資源類型。 刪除閘道本身之前，請先刪除與 VPN 閘道相關聯的連線。 刪除閘道後，您就可以刪除閘道的公用 IP 位址。 請參閱[刪除 VPN 閘道](vpn-gateway-delete-vnet-gateway-powershell.md)以取得詳細步驟。

如果閘道屬於原型或概念證明部署的一部分，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組、VPN 閘道和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name $RG1
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解基本的 VPN 閘道建立和管理，像是如何：

> [!div class="checklist"]
> * 建立 VPN 閘道
> * 調整 VPN 閘道的大小
> * 重設 VPN 閘道

請繼續進行下列教學課程，以了解 S2S、VNet 對 VNet 和 P2S 連線。

> [!div class="nextstepaction"]
> * [建立 S2S 連線](vpn-gateway-tutorial-vpnconnection-powershell.md)
> * [建立 VNet 對 VNet 連線](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [建立 P2S 連線](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
