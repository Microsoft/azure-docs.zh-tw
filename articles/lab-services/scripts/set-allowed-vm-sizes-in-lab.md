---
title: PowerShell 指令碼：在 Azure 實驗室服務中設定允許的 VM 大小 | Microsoft Docs
description: 這個 PowerShell 指令碼會在 Azure 實驗室服務中設定允許的 VM 大小。
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
ms.openlocfilehash: 159f175e7bb27b2d89001e1eba737c67adb89e50
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34638138"
---
# <a name="use-powershell-to-set-allowed-vm-sizes-in-azure-lab-services"></a>使用 PowerShell 在 Azure 實驗室服務中設定允許的 VM 大小

此範例 PowerShell 指令碼會在 Azure 實驗室服務中設定允許的虛擬機器 (VM) 大小。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>先決條件
* **實驗室**。 指令碼需要您擁有現有的實驗室。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/devtest-lab/set-allowed-vm-sizes-in-lab/set-allowed-vm-sizes-in-lab.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [Find-AzureRmResource](/module/azurerm.resources/find-azurermresource) | 根據指定的參數搜尋資源。 |
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | 取得資源。 |
| [Set-AzureRmResource](/powershell/module/azurerm.resources/set-azurermresource) | 修改資源。 |
| [New-AzureRmResource](/powershell/module/azurerm.resources/new-azurermresource) | 建立資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure 實驗室服務 PowerShell 範例](../samples-powershell.md)中找到其他的 Azure 實驗室服務 PowerShell 指令碼範例。