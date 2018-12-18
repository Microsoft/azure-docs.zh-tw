---
title: PowerShell 指令碼：將外部使用者新增至 Azure DevTest Labs 的實驗室中 | Microsoft Docs
description: 這個 PowerShell 指令碼會將外部使用者新增至 Azure DevTest Labs 的實驗室中。
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
ms.openlocfilehash: 0acff1eb4cee441187205b11a7e07cc718072cbf
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248184"
---
# <a name="use-powershell-to-add-an-external-user-to-a-lab-in-azure-devtest-labs"></a>使用 PowerShell 將外部使用者新增至 Azure DevTest Labs 的實驗室中

這個範例 PowerShell 指令碼會將外部使用者新增至 Azure DevTest Labs 的實驗室中。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>必要條件
* **實驗室**。 指令碼需要您擁有現有的實驗室。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/devtest-lab/add-external-user-to-lab/add-external-user-to-custom-lab.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [Get-AzureRmADUser](/powershell/module/azurerm.resources/get-azurermaduser) | 從 Azure Active Directory 擷取使用者物件。 |
| [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment) | 在指定的範圍中，將指定的角色指派給指定的主體。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure 實驗室服務 PowerShell 範例](../samples-powershell.md)中找到其他的 Azure 實驗室服務 PowerShell 指令碼範例。
