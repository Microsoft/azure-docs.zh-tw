---
title: Azure PowerShell 指令碼範例 - 輪替儲存體帳戶存取金鑰 | Microsoft Docs
description: 建立 Azure 儲存體帳戶，然後擷取並輪替其中一個帳戶存取金鑰。
services: storage
documentationcenter: na
author: tamram
manager: timlt
editor: tysonn
ms.assetid: ''
ms.custom: mvc
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: sample
ms.date: 06/13/2017
ms.author: tamram
ms.openlocfilehash: a7aaacf316799540a6a72b699ba8ea8bb389c8a8
ms.sourcegitcommit: 29bac59f1d62f38740b60274cb4912816ee775ea
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/29/2017
ms.locfileid: "25984156"
---
# <a name="create-a-storage-account-and-rotate-its-account-access-keys"></a>建立儲存體帳戶，並輪替其帳戶存取金鑰

此指令碼會建立 Azure 儲存體帳戶，顯示新儲存體帳戶的主要存取金鑰，然後更新 (輪替) 該金鑰。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/storage/rotate-storage-account-keys/rotate-storage-account-keys.ps1 "Rotate storage account keys")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、儲存體帳戶和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name rotatekeystestrg
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立儲存體帳戶，並擷取及輪替其中一個存取金鑰。 下表中的每個項目都會連結至特定命令的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation) | 取得所有位置和每個位置支援的資源提供者。 |
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立 Azure 資源群組。 |
| [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) | 建立儲存體帳戶。 |
| [Get-AzureRmStorageAccountKey](/powershell/module/azurerm.storage/get-azurermstorageaccountkey) | 取得 Azure 儲存體帳戶的存取金鑰。 |
| [New-AzureRmStorageAccountKey](/powershell/module/azurerm.storage/new-azurermstorageaccountkey) | 重新產生 Azure 儲存體帳戶的存取金鑰。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

在 [Azure Blob 儲存體的 PowerShell 範例](../blobs/storage-samples-blobs-powershell.md)中，提供了其他儲存體 PowerShell 指令碼範例。
