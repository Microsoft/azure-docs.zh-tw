---
title: Azure PowerShell 指令碼範例 - 備份與還原服務 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 備份與還原服務
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.devlang: na
ms.topic: sample
ms.date: 11/16/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: 8776530f3a0aa7065948d0f7fd2fa6f1b852f0cb
ms.sourcegitcommit: f58fc4748053a50c34a56314cf99ec56f33fd616
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/04/2018
ms.locfileid: "48267532"
---
# <a name="backup-and-restore-service"></a>備份與還原服務

本文所示的此範例會示範如何備份與還原 API 管理服務執行個體。 

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/api-management/backup-restore-apim-service/backup_restore_apim_service.ps1 "Backup and restore the APIM service instance")]

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/azure/overview)。

您可以在 [PowerShell 範例](../powershell-samples.md)中找到適用於「Azure API 管理」的其他 Azure PowerShell 範例。
