---
title: 篩選網路流量 - Azure CLI | Microsoft Docs
description: 在本文中，您將了解如何使用 Azure CLI 透過網路安全性群組篩選傳至子網路的網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 27102f3523749802dc16a28e28f8859d35814990
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952744"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-the-azure-cli"></a>使用 Azure CLI 透過網路安全性群組篩選網路流量

您可以透過網路安全性群組篩選輸入虛擬網路子網路和從中輸出的網路流量。 網路安全性群組包含可依 IP 位址、連接埠和通訊協定篩選網路流量的安全性規則。 安全性規則會套用至子網路中部署的資源。 在本文中，您將了解：

* 建立網路安全性群組和安全性規則
* 建立虛擬網路，並將網路安全性群組與子網路產生關聯
* 將虛擬機器 (VM) 部署至子網路中
* 測試流量篩選

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 


## <a name="create-a-network-security-group"></a>建立網路安全性群組

網路安全性群組包含安全性規則。 安全性規則可指定來源和目的地。 來源和目的地可以是應用程式安全性群組。

### <a name="create-application-security-groups"></a>建立應用程式安全性群組

首先，請使用 [az group create](/cli/azure/group#az_group_create) 為本文中建立的所有資源建立資源群組。 下列範例會在 *eastus* 位置建立資源群組： 

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

使用 [az network nsg create](/cli/azure/network/asg#az_network_asg_create) 建立應用程式安全性群組。 應用程式安全性群組可讓您將具有類似連接埠篩選需求的伺服器分組在一起。 下列範例會建立兩個應用程式安全性群組。

```azurecli-interactive
az network asg create \
  --resource-group myResourceGroup \
  --name myAsgWebServers \
  --location eastus

az network asg create \
  --resource-group myResourceGroup \
  --name myAsgMgmtServers \
  --location eastus
```

### <a name="create-a-network-security-group"></a>建立網路安全性群組

使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例會建立名為 myNsg 的網路安全性群組： 

```azurecli-interactive 
# Create a network security group
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsg
```

### <a name="create-security-rules"></a>建立安全性規則

使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 建立安全性規則。 下列範例會建立允許透過連接埠 80 和 443 從網際網路將流量輸入 *myWebServers* 應用程式安全性群組的規則：

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-Web-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgWebServers" \
  --destination-port-range 80 443
```

下列範例會建立允許透過連接埠 22 從網際網路將流量輸入 *myMgmtServers* 應用程式安全性群組的規則：

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgMgmtServers" \
  --destination-port-range 22
```

在本文中，會針對 *myAsgMgmtServers* VM 將 SSH (連接埠 22) 公開至網際網路。 在生產環境中則不應將連接埠 22 公開至網際網路，而是建議您使用 [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或[私人](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)網路連線連接到您想要管理的 Azure 資源。

## <a name="create-a-virtual-network"></a>建立虛擬網路

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 下列範例會建立名為 myVirtualNetwork 的虛擬網路：

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16
```

使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) 將子網路新增至虛擬網路。 下列範例會將名為 mySubnet 的子網路新增至虛擬網路，並將其與 myNsg 網路安全性群組產生關聯：

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name mySubnet \
  --address-prefix 10.0.0.0/24 \
  --network-security-group myNsg
```

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路中建立兩個 VM，以便在後續步驟中驗證流量篩選。 

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會建立將作為 Web 伺服器的 VM。 `--asgs myAsgWebServers` 選項會使 Azure 將它為 VM 建立的網路介面設為 *myAsgWebServers* 應用程式安全性群組的成員。

指定 `--nsg ""` 選項時，Azure 將無法為它在建立 VM 時所建立的網路介面建立預設網路安全性群組。 為了簡化本文，我們會使用密碼。 金鑰通常用於生產環境部署中。 如果您使用金鑰，您還必須設定 SSH 代理程式轉送才能完成其餘步驟。 如需詳細資訊，請參閱 SSH 用戶端的文件。 在下列命令中，將 `<replace-with-your-password>` 取代為您選擇的密碼。

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmWeb \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgWebServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

建立 VM 需要幾分鐘的時間。 建立 VM 後，將會傳回與下列範例類似的輸出： 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmWeb",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

請記下 **publicIpAddress**。 在稍後的步驟中，這個位址可用來從網際網路存取虛擬機器。  建立作為管理伺服器的 VM：

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmMgmt \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgMgmtServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

建立 VM 需要幾分鐘的時間。 VM 建立後，請記下傳回之輸出中的 **publicIpAddress**。 後續步驟會使用此位址來存取 VM。 在 VM 建立完成前，請勿繼續進行下一個步驟。

## <a name="test-traffic-filters"></a>測試流量篩選

使用下列命令建立 myVmMgmt VM 的 SSH 工作階段。 以虛擬機器的公用 IP 位址取代 <publicIpAddress>。 在上述範例中，IP 位址是 *13.90.242.231*。

```bash 
ssh azureuser@<publicIpAddress>
```

出現輸入密碼的提示時，請輸入您在[建立 VM](#create-virtual-machines) 中輸入的密碼。

由於允許連接埠 22 從網際網路將流量輸入連結至 myVmMgmt VM 的網路介面所在的 myAsgMgmtServers 應用程式安全性群組，因此連線會成功。

使用下列命令，從 myVmMgmt VM 透過 SSH 連線至 myVmWeb VM：

```bash 
ssh azureuser@myVmWeb
```

由於每個網路安全性群組中的預設安全性規則允許透過所有連接埠在虛擬網路內的所有 IP 位址之間傳輸流量，因此連線會成功。 您無法透過 SSH 網際網路連線至 myVmWeb VM，因為 myAsgWebServers 的安全性規則不允許從網際網路的連接埠 22 輸入。

使用下列命令在 myVmWeb VM 上安裝 nginx Web 伺服器：

```bash 
# Update package source
sudo apt-get -y update

# Install NGINX
sudo apt-get -y install nginx
```

由於預設安全性規則允許所有對網際網路的輸出流量，因此允許 MyVmWeb VM 輸出至網際網路以擷取 nginx。 結束 myVmWeb SSH 工作階段，這樣會讓您停留在 myVmMgmt VM 的 `username@myVmMgmt:~$` 提示上。 若要從 myVmWeb VM 擷取 nginx 歡迎畫面，請輸入下列命令：

```bash
curl myVmWeb
```

登出 myVmMgmt VM。 若要確認您可以從 Azure 外部存取 myVmWeb Web 伺服器，請從您自己的電腦輸入 `curl <publicIpAddress>`。 由於允許連接埠 80 從網際網路將流量輸入連結至 myVmWeb VM 的網路介面所在的 myAsgWebServers 應用程式安全性群組，因此連線會成功。

## <a name="clean-up-resources"></a>清除資源

請使用 [az group delete](/cli/azure/group#az_group_delete) 來移除不再需要的資源群組以及其所包含的所有資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文中，您已建立網路安全性群組，並將其與虛擬網路子網路產生關聯。 若要深入了解網路安全性群組，請參閱[網路安全性群組概觀](security-overview.md)和[管理網路安全性群組](manage-network-security-group.md)。

Azure 依預設會路由傳送子網路之間的流量。 您可以改採其他方式，例如，透過作為防火牆的 VM 路由傳送子網路之間的流量。 若要了解操作方式，請參閱[建立路由表](tutorial-create-route-table-cli.md)。