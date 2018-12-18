---
title: PowerShell 指令碼：從 Azure 實驗室服務中的 VHD 檔案建立自訂映像 | Microsoft Docs
description: 此 PowerShell 指令碼會從 Azure 實驗室服務中的 VHD 檔案建立自訂映像。
services: lab-services
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/11/2018
ms.author: spelluru
ms.openlocfilehash: 19b7c3c6018ec56b056761c336bc56c8b63b47a2
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34636397"
---
# <a name="use-powershell-to-create-a-custom-image-from-a-vhd-file-in-azure-lab-services"></a>使用 PowerShell 從 Azure 實驗室服務中的 VHD 檔案建立自訂映像

此範例 PowerShell 指令碼會從 Azure 實驗室服務中的 VHD 檔案建立自訂映像

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>先決條件
* **實驗室**。 指令碼需要您擁有現有的實驗室。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/devtest-lab/create-custom-image-from-vhd/create-custom-image-from-vhd.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | 取得資源。 |
| [Get-AzureRmStorageAccountKey](/powershell/module/azurerm.storage/get-azurermstorageaccountkey) | 取得 Azure 儲存體帳戶的存取金鑰。 |
| [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) | 將 Azure 部署加入資源群組。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure 實驗室服務 PowerShell 範例](../samples-powershell.md)中找到其他的 Azure 實驗室服務 PowerShell 指令碼範例。