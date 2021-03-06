---
title: Azure CLI 範例 - 連結及使用資料磁碟
description: 此指令碼會使用 Azure CLI 建立 Azure 虛擬機器擴展集，以及連結和準備資料磁碟。
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 3fed38814d3ed00c97288af41a2aa57d6a696d4e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "87499733"
---
# <a name="attach-and-use-data-disks-with-a-virtual-machine-scale-set-with-the-azure-cli"></a>使用 Azure CLI 連結及使用虛擬機器擴展集所適用的資料磁碟
此指令碼會建立虛擬機器擴展集，以及連結和準備資料磁碟。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼
[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine-scale-sets/use-data-disks/use-data-disks.sh "Create a virtual machine scale set with data disks")]

## <a name="clean-up-deployment"></a>清除部署
執行下列命令來移除資源群組、擴展集和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明
此指令碼使用下列命令來建立資源群組、虛擬機器擴展集和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [az group create](/cli/azure/ad/group) | 建立用來存放所有資源的資源群組。 |
| [az vmss create](/cli/azure/vmss) | 建立虛擬機器擴展集，並將它連線到虛擬網路、子網路及網路安全性群組。 若要將流量散發到多個虛擬機器執行個體，也會建立負載平衡器。 此命令也會指定要使用的 VM 映像和管理認證。  |
| [az vmss disk attach](/cli/azure/vmss/disk) | 建立資料磁碟並將其連結到虛擬機器擴展集。 |
| [az vmss extension set](/cli/azure/vmss/extension) | 安裝 Azure 自訂指令碼擴充功能，以執行指令碼來準備每個 VM 執行個體上的資料磁碟。 |
| [az group delete](/cli/azure/ad/group) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure/overview)。
