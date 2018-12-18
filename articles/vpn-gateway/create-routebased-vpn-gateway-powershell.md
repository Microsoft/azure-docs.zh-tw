---
title: 建立路由型 Azure VPN 閘道：PowerShell | Microsoft Docs
description: 使用 PowerShell 來快速建立路由型 VPN 閘道
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/04/2018
ms.author: cherylmc
ms.openlocfilehash: efa07a68cda60ea2d8256a8d068639305f7f4c86
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38295450"
---
# <a name="create-a-route-based-vpn-gateway-using-powershell"></a>使用 PowerShell 來建立路由型 VPN 閘道

本文將協助您使用 PowerShell 來快速建立路由型 Azure VPN 閘道。 建立與內部部署網路的 VPN 連線時，會使用 VPN 閘道。 您也可以使用 VPN 閘道來連線至 VNet。

本文中的步驟將會建立 VNet、子網路、閘道子網路，以及路由型 VPN 閘道 (虛擬網路閘道)。 完成閘道建立之後，您接著就可建立連線。 這些步驟需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，您必須使用 Azure PowerShell 模組 5.3.0 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

```azurepowershell-interactive
New-AzureRmResourceGroup -Name TestRG1 -Location EastUS
```

## <a name="vnet"></a>建立虛擬網路

使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 下列範例會在 **EastUS** 位置中建立名為 **VNet1** 的虛擬網路：

```azurepowershell-interactive
$virtualNetwork = New-AzureRmVirtualNetwork `
  -ResourceGroupName TestRG1 `
  -Location EastUS `
  -Name VNet1 `
  -AddressPrefix 10.1.0.0/16
```

使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) Cmdlet 來建立子網路設定。

```azurepowershell-interactive
$subnetConfig = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name Frontend `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork) Cmdlet 來設定虛擬網路的子網路設定。


```azurepowershell-interactive
$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="gwsubnet"></a>新增閘道子網路

閘道子網路包含虛擬網路閘道服務所使用的保留 IP 位址。 請使用下列範例來新增閘道子網路：

設定 VNet 的變數。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG1 -Name VNet1
```

使用 [Add-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/Add-AzureRmVirtualNetworkSubnetConfig) Cmdlet 來建立閘道子網路。

```azurepowershell-interactive
Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $vnet
```

使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork) Cmdlet 來設定虛擬網路的子網路設定。

```azurepowershell-interactive
$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="PublicIP"></a>要求公用 IP 位址

VPN 閘道必須具有動態配置的公用 IP 位址。 當您建立與 VPN 閘道的連線時，這就是您指定的 IP 位址。 請使用下列範例來要求公用 IP 位址：

```azurepowershell-interactive
$gwpip= New-AzureRmPublicIpAddress -Name VNet1GWIP -ResourceGroupName TestRG1 -Location 'East US' -AllocationMethod Dynamic
```

## <a name="GatewayIPConfig"></a>建立閘道 IP 位址設定

閘道器組態定義要使用的子網路和公用 IP 位址。 使用下列範例來建立閘道組態：

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork -Name VNet1 -ResourceGroupName TestRG1
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
```
## <a name="CreateGateway"></a>建立 VPN 閘道

VPN 閘道建立作業可能需要花費 45 分鐘以上的時間。 閘道建立完成之後，您便可以在您的虛擬網路與另一個 VNet 之間建立連線。 或是在您的虛擬網路與內部部署位置之間建立連線。 請使用 [New-AzureRmVirtualNetworkGateway](/powershell/module/azurerm.network/New-AzureRmVirtualNetworkGateway) Cmdlet 來建立 VPN 閘道。

```azurepowershell-interactive
New-AzureRmVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'East US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased -GatewaySku VpnGw1
```

## <a name="viewgw"></a>檢視 VPN 閘道

您可以使用 [Get-AzureRmVirtualNetworkGateway](/powershell/module/azurerm.network/Get-AzureRmVirtualNetworkGateway) Cmdlet 來檢視 VPN 閘道。

```azurepowershell-interactive
Get-AzureRmVirtualNetworkGateway -Name Vnet1GW -ResourceGroup TestRG1
```

輸出會看似以下範例：

```
Name                   : VNet1GW
ResourceGroupName      : TestRG1
Location               : eastus
Id                     : /subscriptions/<subscription ID>/resourceGroups/TestRG1/provide
                         rs/Microsoft.Network/virtualNetworkGateways/VNet1GW
Etag                   : W/"0952d-9da8-4d7d-a8ed-28c8ca0413"
ResourceGuid           : dc6ce1de-2c4494-9d0b-20b03ac595
ProvisioningState      : Succeeded
Tags                   :
IpConfigurations       : [
                           {
                             "PrivateIpAllocationMethod": "Dynamic",
                             "Subnet": {
                               "Id": "/subscriptions/<subscription ID>/resourceGroups/Te
                         stRG1/providers/Microsoft.Network/virtualNetworks/VNet1/subnets/GatewaySubnet"
                             },
                             "PublicIpAddress": {
                               "Id": "/subscriptions/<subscription ID>/resourceGroups/Te
                         stRG1/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP"
                             },
                             "Name": "default",
                             "Etag": "W/\"0952d-9da8-4d7d-a8ed-28c8ca0413\"",
                             "Id": "/subscriptions/<subscription ID>/resourceGroups/Test
                         RG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/de
                         fault"
                           }
                         ]
GatewayType            : Vpn
VpnType                : RouteBased
EnableBgp              : False
ActiveActive           : False
GatewayDefaultSite     : null
Sku                    : {
                           "Capacity": 2,
                           "Name": "VpnGw1",
                           "Tier": "VpnGw1"
                         }
VpnClientConfiguration : null
BgpSettings            : {
     
```

## <a name="viewgwpip"></a>檢視公用 IP 位址

若要檢視 VPN 閘道的公用 IP 位址，請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/Get-AzureRmPublicIpAddress) Cmdlet。

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -Name VNet1GWIP -ResourceGroupName TestRG1
```

在範例回應中，IpAddress 值是公用 IP 位址。

```
Name                     : VNet1GWIP
ResourceGroupName        : TestRG1
Location                 : eastus
Id                       : /subscriptions/<subscription ID>/resourceGroups/TestRG1/provi
                           ders/Microsoft.Network/publicIPAddresses/VNet1GWIP
Etag                     : W/"5001666a-bc2a-484b-bcf5-ad488dabd8ca"
ResourceGuid             : 3c7c481e-9828-4dae-abdc-f95b383
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Dynamic
IpAddress                : 13.90.153.3
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                             "Id": "/subscriptions/<subscription ID>/resourceGroups/Test
                           RG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/
                           default"
                           }
DnsSettings              : null
Zones                    : {}
Sku                      : {
                             "Name": "Basic"
                           }
IpTags                   : {}
```

## <a name="clean-up-resources"></a>清除資源

當您不再需要所建立的資源時，請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來刪除資源群組。 這會刪除資源群組及其包含的所有資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name TestRG1
```

## <a name="next-steps"></a>後續步驟

閘道建立完成之後，您便可在您的虛擬網路與另一個 VNet 之間建立連線。 或是在您的虛擬網路與內部部署位置之間建立連線。

> [!div class="nextstepaction"]
> [建立站對站連線](vpn-gateway-create-site-to-site-rm-powershell.md)<br><br>
> [建立點對站連線](vpn-gateway-howto-point-to-site-rm-ps.md)<br><br>
> [建立與另一個 VNet 的連線](vpn-gateway-vnet-vnet-rm-ps.md)
