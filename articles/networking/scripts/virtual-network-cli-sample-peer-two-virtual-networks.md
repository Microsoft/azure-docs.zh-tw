---
title: Azure CLI 指令碼範例 - 讓兩個虛擬網路對等互連 | Microsoft Docs
description: Azure CLI 指令碼範例 - 讓兩個虛擬網路對等互連
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: timlt
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: kumud
ms.openlocfilehash: 8ad1e7de85f851b5db6764175bc1136dd19d418d
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29852365"
---
# <a name="peer-two-virtual-networks"></a>讓兩個虛擬網路對等互連

此指令碼會在同一個區域中建立兩個虛擬網路，並透過 Azure 網路連接這兩個虛擬網路。 執行此指令碼之後，您將會在兩個虛擬網路之間建立對等互連。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/peer-two-virtual-networks/peer-two-virtual-networks.sh "Peer two networks")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az_network_vnet_create) | 建立 Azure 虛擬網路和子網路。 |
| [az network vnet peering create](https://docs.microsoft.com/cli/azure/network/vnet/peering#az_network_vnet_peering_create) | 在兩個虛擬網路之間建立對等互連。  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure 網路概觀文件](../cli-samples.md)中找到其他網路 CLI 指令碼範例。
