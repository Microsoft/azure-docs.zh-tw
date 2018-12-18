---
title: 使用虛擬網路對等互連連線虛擬網路 - Azure CLI | Microsoft Docs
description: 在本文中，您將了解如何使用 Azure CLI，透過虛擬網路對等互連來連線虛擬網路。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 0f98f815c0417d1ee8ad5708589b5402a033f83a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989529"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-the-azure-cli"></a>使用 Azure CLI 以虛擬網路對等互連連線虛擬網路

您可以使用虛擬網路對等互連，讓虛擬網路彼此連線。 一旦虛擬網路對等互連，兩個虛擬網路中的資源就可以彼此通訊，且通訊時會有相同的延遲和頻寬，彷彿這些資源是位於相同的虛擬網路中。 在本文中，您將了解：

* 建立兩個虛擬網路
* 使用虛擬網路對等互連連線兩個虛擬網路
* 將虛擬機器 (VM) 部署到每個虛擬網路
* 虛擬機器之間的通訊

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="create-virtual-networks"></a>建立虛擬網路

建立虛擬網路之前，您必須為虛擬網路以及在本文中建立的所有其他資源，建立資源群組。 使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 下列範例會建立名為 myVirtualNetwork1 的虛擬網路，其位址首碼為 10.0.0.0/16。

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.0.0.0/24
```

建立名為 myVirtualNetwork2 的虛擬網路，其位址首碼為 10.1.0.0/16：

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --address-prefixes 10.1.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.1.0.0/24
```

## <a name="peer-virtual-networks"></a>對等互連虛擬網路

對等互連是建立於虛擬網路識別碼之間，所以您必須先使用 [az network vnet show](/cli/azure/network/vnet#az_network_vnet_show) 取得每個虛擬網路的識別碼，並且將識別碼儲存在變數中。

```azurecli-interactive
# Get the id for myVirtualNetwork1.
vNet1Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork1 \
  --query id --out tsv)

# Get the id for myVirtualNetwork2.
vNet2Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork2 \
  --query id \
  --out tsv)
```

使用 [az network vnet peering create](/cli/azure/network/vnet/peering#az_network_vnet_peering_create) 建立從 myVirtualNetwork1 到 myVirtualNetwork2 的對等互連。 如果未指定 `--allow-vnet-access` 參數，雖然會建立對等互連，但是沒有通訊可以通過它。

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --remote-vnet-id $vNet2Id \
  --allow-vnet-access
```

在執行上一個命令之後所傳回的輸出中，您會看到 **peeringState** 是「已啟動」。 對等互連會持續處於「已啟動」狀態，直到您建立從 myVirtualNetwork2 到 myVirtualNetwork1 的對等互連為止。 建立從 myVirtualNetwork2 到 myVirtualNetwork1 的對等互連。 

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork2-myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork2 \
  --remote-vnet-id $vNet1Id \
  --allow-vnet-access
```

在執行上一個命令之後所傳回的輸出中，您會看到 **peeringState** 是「已連線」。 Azure 也會將 myVirtualNetwork1-myVirtualNetwork2 對等互連的對等互連狀態變更為「已連線」。 使用 [az network vnet peering show](/cli/azure/network/vnet/peering#az_network_vnet_peering_show) 確認 myVirtualNetwork1-myVirtualNetwork2 對等互連的對等互連狀態變更為「已連線」。

```azurecli-interactive
az network vnet peering show \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --query peeringState
```

兩個虛擬網路之對等互連的 **PeeringState** 都是「已連線」之前，其中一個虛擬網路中的資源無法與另一個虛擬網路中的資源通訊。 

## <a name="create-virtual-machines"></a>建立虛擬機器

在每個虛擬網路中建立虛擬機器，以便您可以在稍後的步驟中於彼此之間通訊。

### <a name="create-the-first-vm"></a>建立第一個 VM

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會在 myVirtualNetwork1 虛擬網路中建立名為 myVm1 的虛擬機器。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。 `--no-wait` 選項會在背景建立虛擬機器，以便您繼續進行下一步。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork1 \
  --subnet Subnet1 \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>建立第二個 VM

在 myVirtualNetwork2 虛擬網路中建立虛擬機器。

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork2 \
  --subnet Subnet1 \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 建立虛擬機器之後，Azure CLI 會顯示類似下列範例的資訊： 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.1.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

請記下 **publicIpAddress**。 在稍後的步驟中，這個位址可用來從網際網路存取虛擬機器。

## <a name="communicate-between-vms"></a>虛擬機器之間的通訊

使用下列命令來對 myVm2 虛擬機器建立 SSH 工作階段。 以虛擬機器的公用 IP 位址取代 `<publicIpAddress>`。 在上述範例中，公用 IP 位址是 13.90.242.231。

```bash 
ssh <publicIpAddress>
```

針對 myVirtualNetwork1 中的虛擬機器執行 Ping 操作。

```bash 
ping 10.0.0.4 -c 4
```

您會收到四個回覆。 

關閉對 myVm2 虛擬機器的 SSH 工作階段。 

## <a name="clean-up-resources"></a>清除資源

請使用 [az group delete](/cli/azure/group#az_group_delete) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何使用虛擬網路對等互連來連線相同 Azure 區域中的兩個網路。 您也可以針對不同[支援區域](virtual-network-manage-peering.md#cross-region)和[不同 Azure 訂用帳戶](create-peering-different-subscriptions.md#cli)中的虛擬網路進行對等互連，以及使用對等互連來建立[中樞和輪輻網路設計](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#vnet-peering)。 若要深入了解虛擬網路對等互連，請參閱[虛擬網路對等互連概觀](virtual-network-peering-overview.md)和[管理虛擬網路對等互連](virtual-network-manage-peering.md)。

您可以透過 VPN [將自己的電腦連線到虛擬網路](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，並且與虛擬網路中或已對等互連虛擬網路中的資源進行互動。 如需可重複使用的指令碼來完成虛擬網路文章中涵蓋的許多工作，請參閱[指令碼範例](cli-samples.md)。