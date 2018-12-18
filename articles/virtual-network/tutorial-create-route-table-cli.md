---
title: 路由網路流量 - Azure CLI | Microsoft Docs
description: 在本文中，了解如何使用 Azure CLI 以路由表路由傳送網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 9b7e66e6789c1f24bbd784c9c3533ee2e2d678dc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981418"
---
# <a name="route-network-traffic-with-a-route-table-using-the-azure-cli"></a>使用 Azure CLI 以路由表路由網路流量

Azure 依預設會自動路由虛擬網路內所有子網路之間的流量。 您可以建立您自己的路由，以覆寫 Azure 的預設路由。 舉例來說，如果您想要通過網路虛擬設備 (NVA) 路由傳送子網路之間的流量，則建立自訂路由的能力很有幫助。 在本文中，您將了解：

* 建立路由表
* 建立路由
* 建立有多個子網路的虛擬網路
* 建立路由表與子網路的關聯
* 建立會路由傳送流量的 NVA
* 將虛擬機器 (VM) 部署到不同子網路
* 透過 NVA 從一個子網路將流量路由傳送到另一個子網路

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="create-a-route-table"></a>建立路由表

您必須先使用 [az group create](/cli/azure/group#az_group_create) 為本文中建立的所有資源建立資源群組，才能建立路由表。 

```azurecli-interactive
# Create a resource group.
az group create \
  --name myResourceGroup \
  --location eastus
``` 

使用 [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create) 建立路由表。 下列範例會建立名為 myRouteTablePublic 的路由表。 

```azurecli-interactive 
# Create a route table
az network route-table create \
  --resource-group myResourceGroup \
  --name myRouteTablePublic
```

## <a name="create-a-route"></a>建立路由

使用 [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create) 建立路由表中的路由。 

```azurecli-interactive
az network route-table route create \
  --name ToPrivateSubnet \
  --resource-group myResourceGroup \
  --route-table-name myRouteTablePublic \
  --address-prefix 10.0.1.0/24 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.2.4
``` 

## <a name="associate-a-route-table-to-a-subnet"></a>建立路由表與子網路的關聯

您必須先建立虛擬網路和子網路，才能讓路由表與子網路產生關聯。 使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立具有一個子網路的虛擬網路。

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) 建立另外兩個子網路。

```azurecli-interactive
# Create a private subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24

# Create a DMZ subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name DMZ \
  --address-prefix 10.0.2.0/24
```

使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update) 將 myRouteTablePublic 路由表關聯至公用子網路。

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Public \
  --resource-group myResourceGroup \
  --route-table myRouteTablePublic
```

## <a name="create-an-nva"></a>建立 NVA

NVA 是會執行網路功能的虛擬機器，例如路由、防火牆或 WAN 最佳化。

使用 [az vm create](/cli/azure/vm#az_vm_create)，在 DMZ 子網路中建立 NVA。 在建立虛擬機器時，Azure 依預設會建立公用 IP 位址，並將其指派給虛擬機器。 `--public-ip-address ""` 參數會指示 Azure 不要建立公用 IP 位址並將其指派給虛擬機器，因為並不需要從網際網路連線至虛擬機器。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。

```azure-cli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmNva \
  --image UbuntuLTS \
  --public-ip-address "" \
  --subnet DMZ \
  --vnet-name myVirtualNetwork \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 在 Azure 建立虛擬機器並傳回虛擬機器的相關輸出之前，請勿繼續執行後續步驟。 

若要讓網路介面能夠將傳送給它的網路流量轉送 (目的地不是它自己的 IP 位址)，則必須對網路介面啟用 IP 轉送。 使用 [az network nic update](/cli/azure/network/nic#az_network_nic_update) 啟用網路介面的 IP 轉送。

```azurecli-interactive
az network nic update \
  --name myVmNvaVMNic \
  --resource-group myResourceGroup \
  --ip-forwarding true
```

在虛擬機器內，作業系統或在虛擬機器內執行的應用程式也必須能夠轉送網路流量。 使用 [az vm extension set](/cli/azure/vm/extension#az_vm_extension_set) 在虛擬機器作業系統內啟用 IP 轉送：

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVmNva \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"sudo sysctl -w net.ipv4.ip_forward=1"}'
```
執行此命令可能需要一分鐘。

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路中建立兩部虛擬機器，您就可以在後續步驟中驗證來自「公用」子網路的流量是否透過 NVA 路由傳送至「私人」子網路。 

使用 [az vm create](/cli/azure/vm#az_vm_create) 在「公用」子網路中建立虛擬機器。 `--no-wait` 參數可以讓 Azure 在背景中執行命令，因此您可以繼續執行下一個命令。 為了簡化本文，我們會使用密碼。 金鑰通常用於生產環境部署中。 如果您使用金鑰，您還必須設定 SSH 代理程式轉送。 如需詳細資訊，請參閱 SSH 用戶端的文件。 在下列命令中，將 `<replace-with-your-password>` 取代為您選擇的密碼。

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --admin-username azureuser \
  --admin-password $adminPassword \
  --no-wait
```

在「私人」子網路中建立虛擬機器。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --admin-username azureuser \
  --admin-password $adminPassword
```

建立 VM 需要幾分鐘的時間。 建立虛擬機器之後，Azure CLI 會顯示類似下列範例的資訊： 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPrivate",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```
請記下 **publicIpAddress**。 在稍後的步驟中，這個位址可用來從網際網路存取虛擬機器。

## <a name="route-traffic-through-an-nva"></a>透過 NVA 路由傳送流量

使用下列命令來對 myVmPrivate 虛擬機器建立 SSH 工作階段。 以虛擬機器的公用 IP 位址取代 <publicIpAddress>。 在上述範例中，IP 位址是 *13.90.242.231*。

```bash 
ssh azureuser@<publicIpAddress>
```

出現密碼的提示時，請輸入您在[建立虛擬機器](#create-virtual-machines)中選取的密碼。

使用下列命令安裝 myVmPrivate 虛擬機器的追蹤路由：

```bash 
sudo apt-get install traceroute
```

使用下列命令以測試從 myVmPrivate 虛擬機器到 myVmPublic 虛擬機器之網路流量的路由。

```bash
traceroute myVmPublic
```

回應如下列範例所示：

```bash
traceroute to myVmPublic (10.0.0.4), 30 hops max, 60 byte packets
1  10.0.0.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

您可以看到流量是直接從 myVmPrivate 虛擬機器直接路由傳送到 myVmPublic 虛擬機器。 Azure 的預設路由會直接路由傳送子網路之間的流量。 

使用下列命令以從 myVmPrivate 虛擬機器透過 SSH 連線到 myVmPublic 虛擬機器：

```bash 
ssh azureuser@myVmPublic
```

使用下列命令安裝 myVmPublic 虛擬機器的追蹤路由：

```bash 
sudo apt-get install traceroute
```

使用下列命令以測試從 myVmPublic 虛擬機器到 myVmPrivate 虛擬機器之網路流量的路由。

```bash
traceroute myVmPrivate
```

回應如下列範例所示：

```bash
traceroute to myVmPrivate (10.0.1.4), 30 hops max, 60 byte packets
1  10.0.2.4 (10.0.2.4)  0.781 ms  0.780 ms  0.775 ms
2  10.0.1.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```
您可以看到第一個躍點是 10.0.2.4，也就是 NVA 的私人 IP 位址。 第二躍點是 10.0.1.4，也就是 myVmPrivate 虛擬機器的私人 IP 位址。 新增至 myRouteTablePublic 路由表且與「公用」子網路產生關聯的路由，會導致 Azure 透過 NVA 路由傳送流量，而不是直接路由傳送到「私人」子網路。

同時關閉與 myVmPublic 和 myVmPrivate 虛擬機器的 SSH 工作階段。

## <a name="clean-up-resources"></a>清除資源

請使用 [az group delete](/cli/azure/group#az_group_delete) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文中，您已建立路由表並將其與子網路產生關聯。 您已建立簡單的 NVA，它會將來自公用子網路的流量路由傳送至私人子網路。 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) 部署各種預先設定的 NVA，這些設備會執行例如防火牆和 WAN 最佳化的網路功能。 若要深入了解路由，請參閱[路由概觀](virtual-networks-udr-overview.md)和[管理路由表](manage-route-table.md)。

雖然您可以在虛擬網路內部署許多 Azure 資源，但是某些 Azure PaaS 服務的資源無法部署到虛擬網路中。 您仍可將某些 Azure PaaS 服務的資源存取，限制為僅來自虛擬網路子網路的流量。 若要深入了解，請參閱[限制對 PaaS 資源的網路存取](tutorial-restrict-network-access-to-resources-cli.md)。
