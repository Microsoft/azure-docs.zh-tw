---
title: 建立具有靜態公用 IP 位址的 VM - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure 命令列介面 (CLI) 建立具有靜態公用 IP 位址的 VM。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 55bc21b0-2a45-4943-a5e7-8d785d0d015c
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: jdial
ms.openlocfilehash: 974c984cda2dd2198d09fb0dd004e640727b8c48
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982642"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-the-azure-cli"></a>使用 Azure CLI 建立具有靜態公用 IP 位址的虛擬機器

您可以建立具有靜態公用 IP 位址的虛擬機器。 公用 IP 位址可讓您從網際網路與虛擬機器通訊。 指派靜態公用 IP 位址 (而非動態位址)，以確保位址永遠不會變更。 深入了解[靜態公用 IP 位址](virtual-network-ip-addresses-overview-arm.md#allocation-method)。 若要將指派給現有虛擬機器的公用 IP 位址從動態變更為靜態，或要處理私人 IP 位址，請參閱[新增、變更或移除 IP 位址](virtual-network-network-interface-addresses.md)。 公用 IP 位址有[象徵性費用](https://azure.microsoft.com/pricing/details/ip-addresses)，而每個訂用帳戶可用的公用 IP 位址數目都有[限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)。

## <a name="create-a-virtual-machine"></a>建立虛擬機器

您可以從本機電腦或使用 Azure Cloud Shell 來完成下列步驟。 若要使用您的本機電腦，請確定已[安裝 Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json)。 若要使用 Azure Cloud Shell，請選取後續任何命令方塊右上角的 [試試看]。 Cloud Shell 可讓您登入 Azure。

1. 如果使用 Cloud Shell，請跳至步驟 2。 開啟命令工作階段，然後使用 `az login` 登入 Azure。
2. 使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 下列範例會在美國東部 Azure 區域中建立一個資源群組：

   ```azurecli-interactive
   az group create --name myResourceGroup --location eastus
   ```

3. 使用 [az vm create](/cli/azure/vm#az-vm-create) 命令建立虛擬機器。 `--public-ip-address-allocation=static` 選項可將靜態公用 IP 位址指派給虛擬機器。 下列範例會建立 Ubuntu 虛擬機器，其具有名為 *myPublicIpAddress* 的靜態、基本 SKU 公用 IP 位址：

   ```azurecli-interactive
   az vm create \
     --resource-group myResourceGroup \
     --name myVM \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys \
     --public-ip-address myPublicIpAddress \
     --public-ip-address-allocation static
   ```

   如果公用 IP 位址必須是標準 SKU，請將 `--public-ip-sku Standard` 新增至前一個命令。 深入了解[公用 IP 位址 SKU](virtual-network-ip-addresses-overview-arm.md#sku)。 如果虛擬機器將會新增至公用 Azure Load Balancer 的後端集區，則虛擬機器公用 IP 位址的 SKU 必須符合負載平衡器公用 IP 位址的 SKU。 如需詳細資訊，請參閱 [Azure Load Balancer](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#skus)。

4. 使用 [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) 檢視指派的公用 IP 位址，並確認它已建立為靜態、基本 SKU 位址：

   ```azurecli-interactive
   az network public-ip show \
     --resource-group myResourceGroup \
     --name myPublicIpAddress \
     --query [ipAddress,publicIpAllocationMethod,sku] \
     --output table
   ```

   Azure 已從您建立虛擬機器所在區域使用的位址中指派公用 IP 位址。 您可以針對 Azure [公開](https://www.microsoft.com/download/details.aspx?id=56519)、[美國政府](https://www.microsoft.com/download/details.aspx?id=57063)、[中國](https://www.microsoft.com/download/details.aspx?id=57062)及[德國](https://www.microsoft.com/download/details.aspx?id=57064)雲端，下載範圍 (前置詞) 清單。

> [!WARNING]
請勿修改虛擬機器的作業系統內的 IP 位址設定。 作業系統不會察覺 Azure 公用 IP 位址。 雖然您可以將私人 IP 位址設定新增至作業系統，但是建議不要這麼做 (除非有需要)，而且在閱讀[將私人 IP 位址新增至作業系統](virtual-network-network-interface-addresses.md#private)之後才能這麼做。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [az group delete](/cli/azure/group#az-group-delete) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

- 深入了解 Azure 中的[公用 IP 位址](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses)
- 深入了解所有[公用 IP 位址設定](virtual-network-public-ip-address.md#create-a-public-ip-address)
- 深入了解[私人 IP 位址](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses)，並將[靜態私人 IP 位址](virtual-network-network-interface-addresses.md#add-ip-addresses)指派給 Azure 虛擬機器
- 深入了解如何建立 [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 和 [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虛擬機器
