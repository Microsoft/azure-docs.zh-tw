---
title: 移動 Azure 中的 Linux VM | Microsoft Docs
description: 在 Resource Manager 部署模型中將 Linux VM 移至另一個 Azure 訂用帳戶或資源群組。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: d635f0a5-4458-4b95-a5f8-eed4f41eb4d4
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: article
ms.date: 12/14/2017
ms.author: cynthn
ms.openlocfilehash: a4a7dd5541fe298675232ffa803f749e71f6a03f
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30907486"
---
# <a name="move-a-linux-vm-to-another-subscription-or-resource-group"></a>將 Linux VM 移至另一個訂用帳戶或資源群組
本文將逐步引導您了解如何在資源群組或訂用帳戶之間移動 Linux VM。 如果您在個人訂用帳戶中建立了 VM，而現在想要將它移至您的公司訂用帳戶以繼續工作，在訂用帳戶之間移動 VM 會很方便。

> [!IMPORTANT]
>此時，您無法移動受控磁碟。 
>
>移動過程中會建立新的資源識別碼。 移動 VM 之後，您必須更新工具和指令碼以使用新的資源識別碼。 
> 
> 

## <a name="use-the-azure-cli-to-move-a-vm"></a>使用 Azure CLI 移動 VM


在使用 CLI 移動 VM 之前，您需要確定來源和目的地訂用帳戶存在於相同的租用戶。 若要檢查這兩個訂用帳戶是否有相同的租用戶識別碼，請使用 [az account show](/cli/azure/account#az_account_show)。

```azurecli-interactive
az account show --subscription mySourceSubscription --query tenantId
az account show --subscription myDestinationSubscription --query tenantId
```
如果來源和目的地訂用帳戶的租用戶識別碼不相同，您必須連絡[支援人員](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)以將資源移至新的租用戶。

若要成功移動 VM，您需要移動 VM 及其所有支援的資源。 使用 [az resource list](/cli/azure/resource#az_resource_list) 命令來列出資源群組中的所有資源及其識別碼。 它有助於將此命令的輸出透過管線送至檔案，以便您將識別碼複製並貼到稍後的命令中。

```azurecli-interactive
az resource list --resource-group "mySourceResourceGroup" --query "[].{Id:id}" --output table
```

若要將 VM 與其資源移到另一個資源群組，請使用 [az resource move](/cli/azure/resource#az_resource_move)。 下列範例說明如何移動 VM 與其所需的大多數常見資源。 使用 **-ids** 參數，並針對要移動的資源傳入以逗號分隔的識別碼清單 (不含空格)。

```azurecli-interactive
vm=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
nic=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkInterfaces/myNIC
nsg=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG
pip=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIPAddress
vnet=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/virtualNetworks/myVNet
diag=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mydiagnosticstorageaccount
storage=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageacountname    

az resource move \
    --ids $vm,$nic,$nsg,$pip,$vnet,$storage,$diag \
    --destination-group "myDestinationResourceGroup"
```

如果您想要將 VM 及其資源移至不同的訂用帳戶，請加入 **--destination-subscriptionId** 參數以指定目的地訂用帳戶。

如果系統要求您確認是否要移動指定的資源， 輸入 **Y** 確認您要移除資源。

[!INCLUDE [virtual-machines-common-move-vm](../../../includes/virtual-machines-common-move-vm.md)]

## <a name="next-steps"></a>後續步驟
您可以在資源群組和訂用帳戶之間移動許多不同類型的資源。 如需詳細資訊，請參閱 [將資源移動到新的資源群組或訂用帳戶](../../resource-group-move-resources.md)。    

