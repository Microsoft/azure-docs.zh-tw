---
title: Azure PowerShell 指令碼範例 - 加密 Windows VM | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 加密 Windows VM
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/12/2017
ms.author: cynthn
ms.openlocfilehash: af12630839d31bd5e3baa4c67cdd881f936e9d19
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37927524"
---
# <a name="encrypt-a-windows-virtual-machine-with-azure-powershell"></a>使用 Azure PowerShell 加密 Windows 虛擬機器

此指令碼會建立一個安全的 Azure Key Vault、加密金鑰、Azure Active Directory 服務主體，以及 Window 虛擬機器 (VM)。 然後會使用 Key Vault 中的加密金鑰和服務主體認證將 VM 加密。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/virtual-machine/encrypt-vm/encrypt-windows-vm.ps1 "Encrypt VM disks")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/new-azurermkeyvault) | 建立 Azure Key Vault 來儲存安全資料，例如加密金鑰。 |
| [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) | 在 Key Vault 中建立加密金鑰。 |
| [New-AzureRmADServicePrincipal](/powershell/module/azurerm.resources/new-azurermadserviceprincipal) | 建立 Azure Active Directory 服務主體，以安全地進行驗證及控制對加密金鑰的存取。 |
| [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) | 在 Key Vault 上設定權限，以授與服務主體對加密金鑰的存取權。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器，並將它連線到網路卡、虛擬網路、子網路及網路安全性群組。 此命令也會開啟連接埠 80 並設定系統管理認證。 |
| [Get-AzureRmKeyVault](/powershell/module/azurerm.keyvault/get-azurermkeyvault) | 取得 Key Vault 的必要資訊 |
| [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) | 使用服務主體認證和加密金鑰在 VM 上啟用加密。 |
| [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) | 顯示 VM 加密程序的狀態。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure Windows VM 文件](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中找到其他的虛擬機器 PowerShell 指令碼範例。
