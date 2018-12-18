---
title: Azure CLI 指令碼範例 - 建立兩個分別具有內部和外部 NSG 的 VM | Microsoft Docs
description: Azure CLI 指令碼範例 - 建立兩個分別具有內部和外部 NSG 的 VM
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: jeconnoc
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.custom: mvc
ms.openlocfilehash: 7a43bd6a7f64278553c3e1d83a7ef9d57831f998
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34654312"
---
# <a name="secure-network-traffic-between-virtual-machines"></a>保護虛擬機器之間的網路流量

此指令碼會建立兩部虛擬機器，並保護兩者的傳入流量。 一部虛擬機器可在網際網路上存取，並已將網路安全性群組 (NSG) 設定為允許連接埠 3389 和連接埠 80 上的流量。 第二部虛擬機器則無法在網際網路上存取，並已將 NSG 設定為只允許來自第一部虛擬機器的流量。 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-nsg/create-windows-vm-nsg.sh "Create VM with NSG")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az_network_vnet_create) | 建立 Azure 虛擬網路和子網路。 |
| [az network vnet subnet create](https://docs.microsoft.com/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) | 建立子網路。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | 建立虛擬機器，並將它連線到網路卡、虛擬網路、子網路及 NSG。 此命令也會指定要使用的虛擬機器映像和管理認證。  |
| [az network nsg rule update](https://docs.microsoft.com/cli/azure/network/nsg/rule#az_network_nsg_rule_update) | 更新 NSG 規則。 此範例會將後端規則更新為只傳遞來自前端子網路的流量。 |
| [az network nsg rule list](https://docs.microsoft.com/cli/azure/network/nsg/rule#az_network_nsg_rule_list) | 傳回網路安全性群組規則的相關資訊。 在此範例中，規則名稱會儲存在變數中，以供稍後在指令碼中使用。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Windows VM 文件](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
