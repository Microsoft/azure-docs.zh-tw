---
title: Azure PowerShell 指令碼範例 - 連結受控磁碟作為 OS 磁碟以建立 VM | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 連結受控磁碟作為 OS 磁碟以建立 VM
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
ms.openlocfilehash: d727b33c8b955667e78442387f27b9adecf66798
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386011"
---
# <a name="create-a-virtual-machine-using-an-existing-managed-os-disk-with-powershell"></a>使用現有受控 OS 磁碟搭配 PowerShell 以建立虛擬機器

此指令碼會連結現有受控磁碟作為 OS 磁碟，以建立虛擬機器。 在先前案例中使用此指令碼︰
* 從不同訂用帳戶中的受控磁碟複製而來的現有受控 OS 磁碟建立 VM
* 從特殊化 VHD 檔案建立的現有受控磁碟建立 VM 
* 從快照集建立的現有受控 OS 磁碟建立 VM 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-vm-from-snapshot/create-vm-from-snapshot.ps1 "Create VM from snapshot")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來取得受控磁碟屬性、將受控磁碟連結至新的 VM，以及建立 VM。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmDisk](/powershell/module/azurerm.compute/Get-AzureRmDisk) | 根據磁碟的名稱和資源群組取得磁碟物件。 傳回物件的 Id 屬性用來將磁碟連結至新的 VM |
| [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) | 建立 VM 組態。 此組態包括 VM 名稱、作業系統和系統管理認證等資訊。 建立 VM 時會使用此組態。 |
| [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk) | 使用磁碟的 Id 屬性將受控磁碟當作 OS 磁碟連結至新的虛擬機器 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址。 |
| [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) | 建立網路介面。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器。 |
|[Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

針對 Marketplace 映像，使用 [Set-AzureRmVMPlan](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmplan?view=azurermps-6.7.0) 設定方案資訊
```powershell
Set-AzureRmVMPlan -VM $VirtualMachine -Publisher $Publisher -Product $Product -Name $Bame
```

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure Windows VM 文件](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 PowerShell 指令碼範例。
