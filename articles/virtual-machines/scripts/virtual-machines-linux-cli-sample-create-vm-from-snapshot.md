---
title: Azure CLI 指令碼範例 - 從快照集建立 VM | Microsoft Docs
description: Azure CLI 指令碼範例 - 從快照集建立 VM
services: virtual-machines-linux
documentationcenter: virtual-machines
author: ramankum
manager: kavithag
editor: ramankum
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/10/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: f3e236cab559ffc1779ab301b153232d63ca2336
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29847493"
---
# <a name="create-a-virtual-machine-from-a-snapshot-with-cli"></a>使用 CLI 從快照集建立虛擬機器

此指令碼會從 OS 磁碟的快照集建立虛擬機器。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-from-snapshot/create-vm-from-snapshot.sh "Create VM from snapshot")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立受控磁碟、虛擬機器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az snapshot show](https://docs.microsoft.com/cli/azure/snapshot#az_snapshot_show) | 使用快照集名稱和資源群組名稱取得快照集。 傳回物件的 Id 屬性用來建立受控磁碟。  |
| [az disk create](https://docs.microsoft.com/cli/azure/disk#az_disk_create) | 使用快照集識別碼、磁碟名稱、儲存體類型和大小，從快照集建立受控磁碟  |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | 建立使用受控 OS 磁碟的 VM |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器 CLI 指令碼範例。
