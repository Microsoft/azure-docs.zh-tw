---
title: Azure CLI 指令碼範例 - 安裝 IIS | Microsoft Docs
description: Azure CLI 指令碼範例 - 安裝 IIS
services: virtual-machines-Windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-machines-Windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-Windows
ms.workload: infrastructure
ms.date: 02/28/2017
ms.author: cynthn
ms.openlocfilehash: b6685aebc23cd50729fd39304b6330ff947a5ba8
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932818"
---
# <a name="quick-create-a-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 快速建立虛擬機器

此指令碼會建立執行 Windows Server 2016 的 Azure 虛擬機器，然後使用 Azure 虛擬機器自訂指令碼擴充功能來安裝 IIS。 執行指令碼之後，您可以在虛擬機器的公用 IP 位址上存取預設的 IIS 網站。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-windows-iis/create-vm-windows-iis.sh "Quick Create VM")]

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
| [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | 建立虛擬機器，並將它連線到網路卡、虛擬網路、子網路及網路安全性群組。 此命令也會指定要使用的虛擬機器映像和管理認證。  |
| [az vm open-port](https://docs.microsoft.com/cli/azure/network/nsg/rule#az_network_nsg_rule_create) | 建立網路安全性群組規則以允許輸入流量。 在此範例中，會開放連接埠 80 供 HTTP 流量使用。 |
| [azure vm extension set](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 對 VM 新增並執行虛擬機器擴充功能。 此範例使用自訂指令碼擴充功能來安裝 IIS。|
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Windows VM 文件](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
