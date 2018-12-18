---
title: 建立路由型 Azure VPN 閘道：CLI | Microsoft Docs
description: 快速了解如何使用 CLI 來建立 VPN 閘道
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
ms.openlocfilehash: 870fa0f369a7296534ccad54a31fec9da8cb5ed2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977582"
---
# <a name="create-a-route-based-vpn-gateway-using-cli"></a>使用 CLI 來建立路由型 VPN 閘道

本文將協助您使用 Azure CLI 來快速建立路由型 Azure VPN 閘道。 建立與內部部署網路的 VPN 連線時，會使用 VPN 閘道。 您也可以使用 VPN 閘道來連線至 VNet。

本文中的步驟將會建立 VNet、子網路、閘道子網路，以及路由型 VPN 閘道 (虛擬網路閘道)。 虛擬網路閘道的建立作業可能需要花費 45 分鐘以上的時間。 完成閘道建立之後，您接著就可建立連線。 這些步驟需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.4 版或更新版本。 若要尋找已安裝的版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 


```azurecli-interactive 
az group create --name TestRG1 --location eastus
```

## <a name="vnet"></a>建立虛擬網路

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 命令來建立虛擬網路。 下列範例會在 **EastUS** 位置中建立名為 **VNet1** 的虛擬網路：

```azurecli-interactive 
az network vnet create \
  -n VNet1 \
  -g TestRG1 \
  -l eastus \
  --address-prefix 10.1.0.0/16 \
  --subnet-name Frontend \
  --subnet-prefix 10.1.0.0/24
```

## <a name="gwsubnet"></a>新增閘道子網路

閘道子網路包含虛擬網路閘道服務所使用的保留 IP 位址。 請使用下列範例來新增閘道子網路：

```azurepowershell-interactive
az network vnet subnet create \
  --vnet-name VNet1 \
  -n GatewaySubnet \
  -g TestRG1 \
  --address-prefix 10.1.255.0/27 
```

## <a name="PublicIP"></a>要求公用 IP 位址

VPN 閘道必須具有動態配置的公用 IP 位址。 公用 IP 位址將會配置給您為虛擬網路建立的 VPN 閘道。 請使用下列範例來要求公用 IP 位址：

```azurecli-interactive
az network public-ip create \
  -n VNet1GWIP \
  -g TestRG1 \
  --allocation-method Dynamic 
```

## <a name="CreateGateway"></a>建立 VPN 閘道

使用 [az network vnet-gateway create](/cli/azure/group#az_network_vnet_gateway_create) 命令建立 VPN 閘道。

如果您使用 `--no-wait` 參數來執行此命令，就不會看到任何意見反應或輸出。 `--no-wait` 參數允許在背景中建立閘道。 但是，並不表示 VPN 閘道會立即建立。

```azurecli-interactive
az network vnet-gateway create \
  -n VNet1GW \
  -l eastus \
  --public-ip-address VNet1GWIP \
  -g TestRG1 \
  --vnet VNet1 \
  --gateway-type Vpn \
  --sku VpnGw1 \
  --vpn-type RouteBased \
  --no-wait
```

VPN 閘道建立作業可能需要花費 45 分鐘以上的時間。

## <a name="viewgw"></a>檢視 VPN 閘道

```azurecli-interactive
az network vnet-gateway show \
  -n VNet1GW \
  -g TestRG1
```

回應看起來與以下類似：

```
{
  "activeActive": false,
  "bgpSettings": null,
  "enableBgp": false,
  "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
  "gatewayDefaultSite": null,
  "gatewayType": "Vpn",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
  "ipConfigurations": [
    {
      "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
      "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
      "name": "vnetGatewayConfig0",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Updating",
      "publicIpAddress": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
        "resourceGroup": "TestRG1"
      },
      "resourceGroup": "TestRG1",
      "subnet": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworks/VNet1/subnets/GatewaySubnet",
        "resourceGroup": "TestRG1"
      }
    }
  ],
  "location": "eastus",
  "name": "VNet1GW",
  "provisioningState": "Updating",
  "resourceGroup": "TestRG1",
  "resourceGuid": "69c269e3-622c-4123-9231",
  "sku": {
    "capacity": 2,
    "name": "VpnGw1",
    "tier": "VpnGw1"
  },
  "tags": null,
  "type": "Microsoft.Network/virtualNetworkGateways",
  "vpnClientConfiguration": null,
  "vpnType": "RouteBased"
}
```

### <a name="view-the-public-ip-address"></a>檢視公用 IP 位址

若要檢視指派給閘道的公用 IP 位址，請使用下列範例：

```azurecli-interactive
az network public-ip show \
  --name VNet1GWIP \
  --resource-group TestRG11
```

與 **ipAddress** 欄位關聯的值即是您 VPN 閘道的公用 IP。

範例回應：

```
{
  "dnsSettings": null,
  "etag": "W/\"a12d4d03-b27a-46cc-b222-8d9364b8166a\"",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "13.90.195.184",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
```
## <a name="clean-up-resources"></a>清除資源

當您不再需要所建立的資源時，請使用 [az group delete](/cli/azure/group#az_group_delete) 來刪除資源群組。 這會刪除資源群組及其包含的所有資源。

```azurecli-interactive 
az group delete --name TestRG1 --yes
```

## <a name="next-steps"></a>後續步驟

閘道建立完成之後，您便可在您的虛擬網路與另一個 VNet 之間建立連線。 或是在您的虛擬網路與內部部署位置之間建立連線。

> [!div class="nextstepaction"]
> [建立站對站連線](vpn-gateway-create-site-to-site-rm-powershell.md)<br><br>
> [建立點對站連線](vpn-gateway-howto-point-to-site-rm-ps.md)<br><br>
> [建立與另一個 VNet 的連線](vpn-gateway-vnet-vnet-rm-ps.md)