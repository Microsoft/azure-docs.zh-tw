---
title: Azure CLI 指令碼範例 - 使用 DSC 建立具有 IIS 的 Windows Server 2016 VM | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用 DSC 建立具有 IIS 的 Windows Server 2016 VM
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: jeconnoc
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-machines-Windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.custom: mvc
ms.openlocfilehash: bb3f10c44ddf8ded35f4d80b599cda7d44226652
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34654652"
---
# <a name="create-a-vm-with-iis-using-dsc"></a>使用 DSC 建立具有 IIS 的 VM

此指令碼會建立虛擬機器，並使用 Azure 虛擬機器 DSC 自訂指令碼擴充功能來安裝及設定 IIS。 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-windows-iis-using-dsc/create-windows-iis-using-dsc.sh "Quick Create VM")]

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
| [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | 建立虛擬機器，並將它連線到網路卡、虛擬網路、子網路及 NSG。 此命令也會指定要使用的虛擬機器映像和管理認證。  |
| [az vm extension set](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | 將自訂指令碼擴充功能新增至會叫用指令碼來安裝 IIS 的虛擬機器。 |
| [az vm open-port](https://docs.microsoft.com/cli/azure/vm#az_vm_open_port) | 建立網路安全性群組規則以允許輸入流量。 在此範例中，會開放連接埠 80 供 HTTP 流量使用。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Windows VM 文件](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
