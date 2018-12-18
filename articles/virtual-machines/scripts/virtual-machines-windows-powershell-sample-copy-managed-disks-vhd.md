---
title: Azure PowerShell 指令碼範例 - 將受控磁碟的 VHD 匯出/複製到不同區域的儲存體帳戶 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 將受控磁碟的 VHD 匯出/複製到相同或不同區域的儲存體帳戶
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
ms.date: 09/17/2018
ms.author: ramankum
ms.openlocfilehash: 978ac07037e1b7e29d83cc3258df01c6f902cd36
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48045234"
---
# <a name="exportcopy-the-vhd-of-a-managed-disk-to-a-storage-account-in-different-region-with-powershell"></a>使用 PowerShell 將受控磁碟的 VHD 匯出/複製到不同區域的儲存體帳戶

此指令碼會將受控磁碟的 VHD 匯出到不同區域的儲存體帳戶。 它會先產生受控磁碟的 SAS URI，然後用它來將基礎 VHD 複製到不同區域的儲存體帳戶。 使用此指令碼將受控磁碟複製到其他區域以進行區域擴充。  

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-managed-disks-vhd-to-storage-account/copy-managed-disks-vhd-to-storage-account.ps1 "Copy the VHD of a managed disk")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來產生受控磁碟的 SAS URI，並使用 SAS URI 將基礎 VHD 複製到儲存體帳戶。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Grant-AzureRmDiskAccess](/powershell/module/azurerm.compute/grant-azurermdiskaccess) | 產生受控磁碟的 SAS URI，用來將基礎 VHD 複製到儲存體帳戶。 |
| [New-AzureStorageContext](/powershell/module/azure.storage/New-AzureStorageContext) | 使用帳戶名稱與金鑰建立儲存體帳戶內容。 此內容可用來對儲存體帳戶執行讀取/寫入作業。 |
| [Start-AzureStorageBlobCopy](/powershell/module/azure.storage/Start-AzureStorageBlobCopy) | 將快照集的底層 VHD 複製到儲存體帳戶 |

## <a name="next-steps"></a>後續步驟

[從 VHD 建立受控磁碟](virtual-machines-windows-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fpowershell%2fmodule%2ftoc.json)

[從受控磁碟建立虛擬機器](./virtual-machines-windows-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure Windows VM 文件](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 PowerShell 指令碼範例。