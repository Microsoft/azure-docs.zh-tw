---
title: 將內部部署網路連線至 Azure 虛擬網路：站對站 VPN：CLI | Microsoft Docs
description: 透過公用網際網路建立從內部部署網路至 Azure 虛擬網路之 IPsec 連線的步驟。 這些步驟可協助您使用 CLI 建立跨單位的站對站 VPN 閘道連線。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2018
ms.author: cherylmc
ms.openlocfilehash: a4400338baa77f82bafc5b74561695f8e9a70b5f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965801"
---
# <a name="create-a-virtual-network-with-a-site-to-site-vpn-connection-using-cli"></a>使用 CLI 建立具有站對站 VPN 連線的虛擬網路

本文說明如何使用 Azure CLI 來建立從內部部署網路到 VNet 的站對站 VPN 閘道連線。 本文中的步驟適用於 Resource Manager 部署模型。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：<br>

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-site-to-site-classic-portal.md)
> 
>


![站對站 VPN 閘道跨單位連線圖表](./media/vpn-gateway-howto-site-to-site-resource-manager-cli/site-to-site-diagram.png)

站對站 VPN 閘道連線可用來透過 IPsec/IKE (IKEv1 或 IKEv2) VPN 通道，將內部部署網路連線到 Azure 虛擬網路。 此類型的連線需要位於內部部署的 VPN 裝置，且您已對該裝置指派對外開放的公用 IP 位址。 如需 VPN 閘道的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。

## <a name="before-you-begin"></a>開始之前

在開始設定之前，請確認您已符合下列條件：

* 確定您有相容的 VPN 裝置以及能夠對其進行設定的人員。 如需相容 VPN 裝置和裝置組態的詳細資訊，請參閱[關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)。
* 確認您的 VPN 裝置有對外開放的公用 IPv4 位址。 此 IP 位址不能位於 NAT 後方。
* 如果您不熟悉位於內部部署網路組態的 IP 位址範圍，您需要與能夠提供那些詳細資料的人協調。 當您建立此組態時，您必須指定 IP 位址範圍的首碼，以供 Azure 路由傳送至您的內部部署位置。 內部部署網路的子網路皆不得與您所要連線的虛擬網路子網路重疊。
* 確認您已安裝最新版的 CLI 命令 (2.0 版或更新版本)。 如需關於安裝 CLI 命令的資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli) 和[開始使用 Azure CLI](/cli/azure/get-started-with-azure-cli)。

### <a name="example"></a>範例值

您可以使用下列值來建立測試環境，或參考這些值來進一步了解本文中的範例：

```
#Example values

VnetName                = TestVNet1 
ResourceGroup           = TestRG1 
Location                = eastus 
AddressSpace            = 10.11.0.0/16 
SubnetName              = Subnet1 
Subnet                  = 10.11.0.0/24 
GatewaySubnet           = 10.11.255.0/27 
LocalNetworkGatewayName = Site2 
LNG Public IP           = <VPN device IP address>
LocalAddrPrefix1        = 10.0.0.0/24
LocalAddrPrefix2        = 20.0.0.0/24   
GatewayName             = VNet1GW 
PublicIP                = VNet1GWIP 
VPNType                 = RouteBased 
GatewayType             = Vpn 
ConnectionName          = VNet1toSite2
```

## <a name="Login"></a>1.連線至您的訂用帳戶

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

## <a name="rg"></a>2.建立資源群組

下列範例會在「eastus」位置建立名為「TestRG1」的資源群組。 如果您在該區域中已有想要用來建立 VNet 的資源群組，您可以改為使用該資源群組。

```azurecli
az group create --name TestRG1 --location eastus
```

## <a name="VNet"></a>3.建立虛擬網路

如果您還沒有虛擬網路，請使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 命令建立一個。 在建立虛擬網路時，請確定您指定的位址空間沒有與您在內部部署網路上所擁有的任何位址空間重疊。

>[!NOTE]
>為了讓此 VNet 連線到內部部署位置，您需要與內部部署網路系統管理員協調，以切割出此虛擬網路專用的 IP 位址範圍。 如果 VPN 連線的兩端存在重複的位址範圍，流量就不會如預期的方式進行路由。 此外，如果您想要將此 VNet 連線到另一個 VNet，則位址空間不能與其他 VNet 重疊。 因此，請謹慎規劃您的網路組態。
>
>

下列範例會建立名為「TestVNet1」的虛擬網路和「Subnet1」子網路。

```azurecli
az network vnet create --name TestVNet1 --resource-group TestRG1 --address-prefix 10.11.0.0/16 --location eastus --subnet-name Subnet1 --subnet-prefix 10.11.0.0/24
```

## 4.<a name="gwsub"></a>建立閘道子網路


[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-include.md)]

使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) 命令建立子網路閘道。

```azurecli
az network vnet subnet create --address-prefix 10.11.255.0/27 --name GatewaySubnet --resource-group TestRG1 --vnet-name TestVNet1
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="localnet"></a>5.建立區域網路閘道

區域網路閘道通常是指您的內部部署位置。 請賦予網站可供 Azure 參考的名稱，然後指定您想要與其建立連線之內部部署 VPN 裝置的 IP 位址。 也請指定 IP 位址首碼，以供系統透過 VPN 閘道路由至 VPN 裝置。 您指定的位址首碼是位於內部部署網路上的首碼。 如果您的內部部署網路有所變更，您可以輕鬆地更新首碼。

輸入下列值：

* --gateway-ip-address 是內部部署 VPN 裝置的 IP 位址。 VPN 裝置不能位於 NAT 後方。
* --local-address-prefixes 是內部部署位址空間。

使用 [az network local-gateway create](/cli/azure/network/local-gateway#az_network_local_gateway_create) 命令新增具有多個位址首碼的區域網路閘道：

```azurecli
az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --resource-group TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24
```

## <a name="PublicIP"></a>6.要求公用 IP 位址

VPN 閘道必須具有公用 IP 位址。 您會先要求 IP 位址資源，然後在建立虛擬網路閘道時參考它。 建立 VPN 閘道時，系統會將 IP 位址動態指派給此資源。 VPN 閘道目前僅支援*動態*公用 IP 位址配置。 您無法要求靜態公用 IP 位址指派。 不過，這不表示 IP 位址變更之後已被指派至您的 VPN 閘道。 公用 IP 位址只會在刪除或重新建立閘道時變更。 它不會因為重新調整、重設或 VPN 閘道的其他內部維護/升級而變更。

使用 [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) 命令來要求動態公用 IP 位址。

```azurecli
az network public-ip create --name VNet1GWIP --resource-group TestRG1 --allocation-method Dynamic
```

## <a name="CreateGateway"></a>7.建立 VPN 閘道

建立虛擬網路 VPN 閘道。 建立 VPN 閘道可能需要 45 分鐘以上的時間才能完成。

輸入下列值：

* 網站間組態的 --gateway-type 是 Vpn。 閘道器類型永遠是您實作的組態的特定類型。 如需詳細資訊，請參閱[閘道類型](vpn-gateway-about-vpn-gateway-settings.md#gwtype)。
* --vpn-type 可以是 RouteBased (在某些文件中稱為動態閘道器)，或 PolicyBased (在某些文件中稱為靜態閘道器)。 這是您所連線裝置之需求的特有設定。 如需 VPN 閘道類型的詳細資訊，請參閱[關於 VPN 閘道組態設定](vpn-gateway-about-vpn-gateway-settings.md#vpntype)。
* 選取您想要使用的閘道 SKU。 某些 SKU 有組態限制。 如需詳細資訊，請參閱[閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。

使用 [az network vnet-gateway create](/cli/azure/network/vnet-gateway#az_network_vnet_gateway_create) 命令建立 VPN 閘道。 如果您使用 '--no-wait' 參數執行此命令，您不會看到任何意見反應或輸出。 此參數允許在背景中建立閘道。 大約需要 45 分鐘的時間來建立閘道。

```azurecli
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWIP --resource-group TestRG1 --vnet TestVNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait 
```

## <a name="VPNDevice"></a>8.設定 VPN 裝置

內部部署網路的站對站連線需要 VPN 裝置。 在此步驟中，設定 VPN 裝置。 在設定 VPN 裝置時，您需要下列項目：

- 共用金鑰。 這個共同金鑰與您建立站對站 VPN 連線時指定的共用金鑰相同。 在我們的範例中，我們會使用基本的共用金鑰。 我們建議您產生更複雜的金鑰以供使用。
- 虛擬網路閘道的公用 IP 位址。 您可以使用 Azure 入口網站、PowerShell 或 CLI 來檢視公用 IP 位址。 若要尋找虛擬網路閘道的公用 IP 位址，請使用 [az network public-ip list](/cli/azure/network/public-ip#az_network_public_ip_list) 命令。 為方便閱讀，系統會將輸出格式化為以資料表格式顯示公用 IP 的清單。

  ```azurecli
  az network public-ip list --resource-group TestRG1 --output table
  ```


[!INCLUDE [Configure VPN device](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]


## <a name="CreateConnection"></a>9.建立 VPN 連線

您會在虛擬網路閘道與內部部署 VPN 裝置之間建立站對站 VPN 連線。 請特別注意共用金鑰值，此值必須符合您的 VPN 裝置所設定的共用金鑰值。

使用 [az network vpn-connection create](/cli/azure/network/vpn-connection#az_network_vpn_connection_create) 命令建立連線。

```azurecli
az network vpn-connection create --name VNet1toSite2 -resource-group TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key abc123 --local-gateway2 Site2
```

過一會兒，連接將會建立。

## <a name="toverify"></a>10.驗證 VPN 連線

[!INCLUDE [verify connection](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

如果您想要使用其他方法來確認您的連線，請參閱[確認 VPN 閘道連線](vpn-gateway-verify-connection-resource-manager.md)。

## <a name="connectVM"></a>連線至虛擬機器

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-s2s-include.md)]

## <a name="tasks"></a>常見工作

本節包含使用站對站組態時很實用的常見命令。 如需 CLI 網路命令的完整清單，請參閱 [Azure CLI - 網路](/cli/azure/network)。

[!INCLUDE [local network gateway common tasks](../../includes/vpn-gateway-common-tasks-cli-include.md)]

## <a name="next-steps"></a>後續步驟

* 一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。
* 如需 BGP 的相關資訊，請參閱 [BGP 概觀](vpn-gateway-bgp-overview.md)和[如何設定 BGP](vpn-gateway-bgp-resource-manager-ps.md)。
* 如需強制通道的相關資訊，請參閱[關於強制通道](vpn-gateway-forced-tunneling-rm.md)。
* 如需高可用性主動-主動連線的相關資訊，請參閱[高可用性跨單位和 VNet 對 VNet 連線能力](vpn-gateway-highlyavailable.md)。
* 如需 Azure CLI 網路命令的清單，請參閱 [Azure CLI](https://docs.microsoft.com/cli/azure/network)。
* 如需使用 Azure Resource Manager 範本建立站對站 VPN 連線的相關資訊，請參閱[建立站對站 VPN 連線](https://azure.microsoft.com/resources/templates/101-site-to-site-vpn-create/)。
* 如需使用 Azure Resource Manager 範本建立 vnet 對 vnet VPN 連線的相關資訊，請參閱[部署 HB 異地複寫](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-replication-geo/)。
