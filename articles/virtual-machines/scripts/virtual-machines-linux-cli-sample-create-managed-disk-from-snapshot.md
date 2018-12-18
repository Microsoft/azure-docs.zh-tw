---
title: Azure CLI 指令碼範例 - 從快照集建立受控磁碟 | Microsoft Docs
description: Azure CLI 指令碼範例 - 從快照集建立受控磁碟
services: virtual-machines-linux
documentationcenter: storage
author: ramankumarlive
manager: kavithag
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: 3a6c741933d48c22ec38d98fee9591d45877ad84
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29849540"
---
# <a name="create-a-managed-disk-from-a-snapshot-with-cli"></a>使用 CLI 從快照集建立受控磁碟

此指令碼會從快照集建立受控磁碟。 使用它從 OS 和資料磁碟的快照集還原虛擬機器。 從個別的快照集建立 OS 和資料受控磁碟，再連結受控磁碟以建立新的虛擬機器。 您也可以連結從快照集建立的資料磁碟，以還原現有 VM 的資料磁碟。


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-managed-disks-from-snapshot/create-managed-disks-from-snapshot.sh "Create managed disk from snapshot")]


## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令從快照集建立受控磁碟。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az snapshot show](https://docs.microsoft.com/cli/azure/snapshot#az_snapshot_show) | 使用快照集的名稱和資源群組屬性，取得快照集的所有屬性。 使用 Id 屬性建立受控磁碟。  |
| [az disk create](https://docs.microsoft.com/cli/azure/disk#az_disk_create) | 使用受管理快照集的快照集識別碼建立受控磁碟 |

## <a name="next-steps"></a>後續步驟

[將受控磁碟連結為作業系統磁碟以建立虛擬機器](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../../app-service/app-service-cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器和受控磁碟 CLI 指令碼範例。
