---
title: Azure PowerShell 指令碼範例 - 從快照集建立 VM | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 從快照集建立 VM
services: virtual-machines-windows
documentationcenter: virtual-machines
author: ramankum
manager: kavithag
editor: ramankum
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 05/10/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: 63d108bbfd0f58f8a40bf1c7c8649e3a1f7ed288
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23116716"
---
# <a name="create-a-virtual-machine-from-a-snapshot-with-powershell"></a>使用 PowerShell 從快照集建立虛擬機器

此指令碼會從 OS 磁碟的快照集建立虛擬機器。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-vm-from-snapshot/create-vm-from-snapshot.ps1 "Create VM from managed os disk")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來取得快照集屬性、從快照集建立受控磁碟，以及建立 VM。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmSnapshot](/powershell/module/azurerm.compute/get-azurermsnapshot) | 使用快照集名稱來取得快照集。 |
| [New-AzureRmDiskConfig](/powershell/module/azurerm.compute/new-azurermdiskconfig) | 建立磁碟組態。 此組態可使用於磁碟建立程序。 |
| [New-AzureRmDisk](/powershell/module/azurerm.compute/new-azurermdisk) | 建立受控磁碟。 |
| [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) | 建立 VM 組態。 此組態包括 VM 名稱、作業系統和系統管理認證等資訊。 建立 VM 時會使用此組態。 |
| [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk) | 將受控磁碟當作 OS 磁碟連結至虛擬機器 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址。 |
| [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) | 建立網路介面。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器。 |
|[Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure Windows VM 文件](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 PowerShell 指令碼範例。