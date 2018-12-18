---
title: Azure CLI 範例 - 建立虛擬機器擴展集 | Microsoft Docs
description: Azure CLI 範例
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 60b0f418c1f4b372f0847edd15afed2497e90cf6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975474"
---
# <a name="create-a-virtual-machine-scale-set-with-the-azure-cli"></a>使用 Azure CLI 建立虛擬機器擴展集
此指令碼會建立具有 Ubuntu 作業系統和相關網路資源 (包括負載平衡器) 的 Azure 虛擬機器擴展集。 執行指令碼之後，您可以透過 SSH 存取 VM 執行個體。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼
[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine-scale-sets/simple-scale-set/simple-scale-set.sh "Create a simple virtual machine scale set")]

## <a name="clean-up-deployment"></a>清除部署
執行下列命令來移除資源群組、擴展集和所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明
此指令碼使用下列命令來建立資源群組、虛擬機器擴展集和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](/cli/azure/ad/group#az_ad_group_create) | 建立用來存放所有資源的資源群組。 |
| [az vmss create](/cli/azure/vmss#az_vmss_create) | 建立虛擬機器擴展集，並將它連線到虛擬網路、子網路及網路安全性群組。 若要將流量散發到多個虛擬機器執行個體，也會建立負載平衡器。 此命令也會指定要使用的 VM 映像和管理認證。  |
| [az group delete](/cli/azure/ad/group#delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure 虛擬機器擴展集文件](../cli-samples.md)中找到其他虛擬機器擴展集的 Azure CLI 指令碼範例。