---
title: PowerShell 指令碼：在 Azure DevTest Labs 的實驗室中建立自訂角色 | Microsoft Docs
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
ms.openlocfilehash: 295f742342fba7d77b556724c8005f3ac4816482
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34636710"
---
# <a name="use-powershell-to-create-a-custom-role-in-a-lab-in-azure-devtest-labs"></a>使用 PowerShell 在 Azure DevTest Labs 的實驗室中建立自訂角色

此範例 PowerShell 指令碼會建立自訂角色，以在 Azure DevTest Labs 的實驗室中使用。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>先決條件
* **實驗室**。 指令碼需要您擁有現有的實驗室。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/devtest-lab/create-custom-role-in-lab/create-custom-role-in-lab.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [Get-AzureRmProviderOperation](/powershell/module/azurerm.resources/get-azurermprovideroperation) | 取得使用 Azure RBAC 保護安全的 Azure 資源提供者作業。 |
| [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition) | 列出可用來指派的所有 Azure RBAC 角色。 |
| [New-AzureRmRoleDefinition](/powershell/module/azurerm.resources/new-azurermroledefinition) | 建立自訂角色。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure 實驗室服務 PowerShell 範例](../samples-powershell.md)中找到其他的 Azure 實驗室服務 PowerShell 指令碼範例。