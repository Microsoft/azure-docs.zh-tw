---
title: Azure PowerShell 指令碼範例 - 將受控磁碟快照集複製 (移動) 到相同或不同的訂用帳戶 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 將受控磁碟快照集複製 (移動) 到相同或不同的訂用帳戶
services: virtual-machines-windows
documentationcenter: storage
author: ramankumarlive
manager: kavithag
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 06/06/2017
ms.author: ramankum
ms.openlocfilehash: 064b5355da10fe683563fa078cfafc65080f7ea2
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23116496"
---
# <a name="copy-snapshot-of-a-managed-disk-in-same-subscription-or-different-subscription-with-powershell"></a>使用 PowerShell 複製相同訂用帳戶或不同訂用帳戶中的受控磁碟快照集

此指令碼會在相同訂用帳戶或不同訂用帳戶中建立快照集的複本。 使用此指令碼將快照集移至不同的訂用帳戶以供進行資料保留。 將快照集儲存到不同的訂用帳戶中可保護您不因在主要訂用帳戶中意外刪除快照集而產生損失。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-snapshot-to-same-or-different-subscription/copy-snapshot-to-same-or-different-subscription.ps1 "Copy snapshot")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令，使用來源快照集的識別碼在目標訂用帳戶中建立快照集。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmSnapshotConfig](/powershell/module/azurerm.compute/New-AzureRmSnapshotConfig) | 建立用於建立快照集的快照集組態。 其中包含父代快照集的資源識別碼以及與父代快照集位置相同的位置。  |
| [New-AzureRmSnapshot](/powershell/module/azurerm.compute/New-AzureRmDisk) | 使用當作參數傳遞的快照集組態、快照集名稱和資源群組名稱來建立快照集。 |


## <a name="next-steps"></a>後續步驟

[從快照集建立虛擬機器](./virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fpowershell%2fmodule%2ftoc.json)

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure Windows VM 文件](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 PowerShell 指令碼範例。