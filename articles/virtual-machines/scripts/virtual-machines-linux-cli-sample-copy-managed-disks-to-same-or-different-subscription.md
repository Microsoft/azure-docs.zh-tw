---
title: Azure CLI 指令碼範例 - 將受控磁碟複製 (移動) 到相同或不同的訂用帳戶 | Microsoft Docs
description: Azure CLI 指令碼範例 - 將受控磁碟複製 (移動) 到相同或不同的訂用帳戶
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
ms.openlocfilehash: dfdbc0563810447a1a214356b5153afe38d9cf2f
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29846786"
---
# <a name="copy-managed-disks-to-same-or-different-subscription-with-cli"></a>使用 CLI將受控磁碟複製到相同或不同的訂用帳戶

此指令碼會將受控磁碟複製到相同的訂用帳戶，或同區域的不同訂用帳戶。 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/virtual-machine/copy-managed-disks-to-same-or-different-subscription/copy-managed-disks-to-same-or-different-subscription.sh "Copy managed disk")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令，在使用來源受控磁碟識別碼的目標訂用帳戶中，建立新的受控磁碟。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az disk show](https://docs.microsoft.com/cli/azure/disk#az_disk_show) | 使用受控磁碟的名稱和資源群組屬性，取得受控磁碟的所有屬性。 使用 Id 屬性將受控磁碟複製到不同的訂用帳戶。  |
| [az disk create](https://docs.microsoft.com/cli/azure/disk#az_disk_create) | 使用父受控磁碟的識別碼和名稱，在不同的訂閱中建立一個新的受控磁碟管，來複製受控磁碟。  |

## <a name="next-steps"></a>後續步驟

[從受控磁碟建立虛擬機器](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure Linux VM 文件](../../app-service/app-service-cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器和受控磁碟 CLI 指令碼範例。
