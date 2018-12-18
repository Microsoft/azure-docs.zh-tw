---
title: Azure CLI 指令碼範例 - 使用 CLI 將受控磁碟快照集複製 (移動) 到相同或不同的訂用帳戶 | Microsoft Docs
description: Azure CLI 指令碼範例 - 使用 CLI 將受控磁碟快照集複製 (移動) 到相同或不同的訂用帳戶
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
ms.openlocfilehash: 904f564d2f3c0584851309780eb2ccca379534fa
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29850200"
---
# <a name="copy-snapshot-of-a-managed-disk-to-same-or-different-subscription-with-cli"></a>使用 CLI將受控磁碟快照集複製到相同或不同的訂用帳戶

此指令碼會將受控磁碟的快照集複製到相同或不同的訂用帳戶。 使用這個指令碼可將快照集移至與父快照集同區域但不同的訂用帳戶。


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/virtual-machine/copy-snapshot-to-same-or-different-subscription/copy-snapshot-to-same-or-different-subscription.sh "Copy snapshot")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令，使用來源快照集的識別碼在目標訂用帳戶中建立快照集。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az snapshot show](https://docs.microsoft.com/cli/azure/snapshot#az_snapshot_show) | 使用快照集的名稱和資源群組屬性，取得快照集的所有屬性。 使用 Id 屬性將快照集複製到不同的訂用帳戶。  |
| [az snapshot create](https://docs.microsoft.com/cli/azure/snapshot#az_snapshot_create) | 在使用父快照集識別碼和名稱的不同訂用帳戶中建立快照集，以複製快照集。  |

## <a name="next-steps"></a>後續步驟

[從快照集建立虛擬機器](./virtual-machines-linux-cli-sample-create-vm-from-snapshot.md?toc=%2fpowershell%2fmodule%2ftoc.json)

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../../app-service/app-service-cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器和受控磁碟 CLI 指令碼範例。
