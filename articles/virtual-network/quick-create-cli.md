---
title: 建立虛擬網路 -快速入門 - Azure CLI | Microsoft Docs
description: 在此快速入門中，您將了解如何使用 Azure 入口網站建立虛擬網路。 虛擬網路可讓 Azure 資源 (例如虛擬機器) 彼此及與網際網路進行私密通訊。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/09/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 2290d2975326c35ec4ec18a77a181a4ad0a205b2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962788"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-cli"></a>快速入門：使用 Azure CLI 建立虛擬網路

虛擬網路可讓 Azure 資源 (例如虛擬機器 (VM)) 彼此以及與網際網路進行私密通訊。 在此快速入門中，您將了解如何建立虛擬網路。 建立虛擬網路之後，您需將兩個 VM 部署至虛擬網路中。 接著，您需從網際網路連線至其中一個 VM，然後與另一個 VM 進行私密通訊。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找已安裝的版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 


## <a name="create-a-virtual-network"></a>建立虛擬網路

建立虛擬網路之前，您必須先建立資源群組來包含虛擬網路。 使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 以下範例會建立一個名為 *myVirtualNetwork* 的預設虛擬網路，其中含有一個名為 *default* 的子網路：

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --subnet-name default
```

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路內建立兩個 VM：

### <a name="create-the-first-vm"></a>建立第一個 VM

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。 `--no-wait` 選項會在背景建立 VM，以便您繼續進行下一步。 下列範例會建立一個名為 *myVm1* 的 VM：

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>建立第二個 VM

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 建立 VM 之後，Azure CLI 會傳回類似以下範例的輸出： 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm1",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

請記下 **publicIpAddress**。 在下一個步驟中，會使用此位址從網際網路連線至 VM。

## <a name="connect-to-a-vm-from-the-internet"></a>從網際網路連線至 VM

使用您的 *myVm2* VM 來取代下列命令中的 `<publicIpAddress>`，然後輸入下列命令：

```bash 
ssh <publicIpAddress>
```

## <a name="communicate-between-vms"></a>虛擬機器之間的通訊

若要確認 *myVm2* 與 *myVm1* VM 之間的私密通訊，請輸入下列命令：

```bash
ping myVm1 -c 4
```

您會從 *10.0.0.4* 收到四個回覆。

結束與 *myVm2* VM 的 SSH 工作階段。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [az group delete](/cli/azure/group#az_group_delete) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您建立了一個預設的虛擬網路和兩個 VM。 您從網際網路連線至其中一個 VM，然後在該 VM 與另一個 VM 進行私密通訊。 若要深入了解虛擬網路設定，請參閱[管理虛擬網路](manage-virtual-network.md)。 

Azure 預設會允許虛擬機器之間進行不受限制的私密通訊，但只允許從網際網路到 Windows VM 的輸入遠端桌面連線。 若要了解如何允許或限制進出 VM 的不同網路通訊類型，請參閱[篩選網路流量](tutorial-filter-network-traffic.md)。
