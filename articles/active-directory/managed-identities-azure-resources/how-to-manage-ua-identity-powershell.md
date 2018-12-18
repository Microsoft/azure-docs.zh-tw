---
title: 如何使用 Azure PowerShell 來建立、列出及刪除使用者指派的受控識別
description: 如何使用 Azure PowerShell 建立、列出和刪除使用者指派的受控識別有關的逐步說明。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/16/2018
ms.author: daveba
ms.openlocfilehash: c7191f60b8780e8ccee9b330aa21d8174f0f0148
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106304"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-azure-powershell"></a>使用 Azure PowerShell 來建立、列出及刪除使用者指派的受控識別

[!INCLUDE [preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

適用於 Azure 資源的受控識別會在 Azure Active Directory 中為 Azure 服務提供受控識別。 您可以使用此身分識別來向支援 Azure AD 驗證的服務進行驗證，而不需要您程式碼中的認證。 

在本文中，您會了解如何使用 Azure PowerShell 建立、列出和刪除使用者指派的受控識別。

## <a name="prerequisites"></a>必要條件

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 如果您尚未安裝[最新版的 Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM)，請先安裝。
- 如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，您必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 
- 如果您在本機執行 PowerShell，您也需要： 
    - 執行 `Login-AzureRmAccount` 來建立與 Azure 的連線。
    - 安裝[最新版的 PowerShellGet](/powershell/gallery/installing-psget#for-systems-with-powershell-50-or-newer-you-can-install-the-latest-powershellget)。
    - 執行 `Install-Module -Name PowerShellGet -AllowPrerelease` 以取得 `PowerShellGet` 模組的搶鮮版 (執行此命令以安裝 `AzureRM.ManagedServiceIdentity` 模組後，您可能需要以 `Exit` 退出目前的 PowerShell 工作階段)。
    - 執行 `Install-Module -Name AzureRM.ManagedServiceIdentity -AllowPrerelease` 安裝 `AzureRM.ManagedServiceIdentity` 模組的搶鮮版，以執行本文中由使用者指派的受控識別作業。
- 若要執行本文中的管理作業，您的帳戶需要下列角色指派：
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立、讀取 (列出)、更新和刪除使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可讀取 (列出) 使用者指派受控識別的屬性。

## <a name="create-a-user-assigned-managed-identity"></a>建立使用者指派的受控識別

若要建立使用者指派的受控識別，請使用 [New-AzureRmUserAssignedIdentity](/powershell/module/azurerm.managedserviceidentity/new-azurermuserassignedidentity) 命令。 `ResourceGroupName` 參數會指定要建立使用者指派之受控識別的資源群組，而 `-Name` 參數則指定其名稱。 將 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 參數取代為您自己的值：

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

 ```azurepowershell-interactive
New-AzureRmUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>列出使用者指派的受控識別

若要列出使用者指派的受控識別，請使用 [Get-AzureRmUserAssigned](/powershell/module/azurerm.managedserviceidentity/get-azurermuserassignedidentity) 命令。  `-ResourceGroupName` 參數會指定建立使用者指派的受控識別所在的資源群組。 將 `<RESOURCE GROUP>` 取代為您自己的值：

```azurepowershell-interactive
Get-AzureRmUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP>
```
在回應中，使用者指派的受控識別具備`"Microsoft.ManagedIdentity/userAssignedIdentities"`為索引鍵傳回的值`Type`。

`Type :Microsoft.ManagedIdentity/userAssignedIdentities`

## <a name="delete-a-user-assigned-managed-identity"></a>刪除使用者指派的受控識別

若要刪除使用者指派的受控識別，請使用 [Remove-AzureRmUserAssignedIdentity](/powershell/module/azurerm.managedserviceidentity/remove-azurermuserassignedidentity) 命令。  `-ResourceGroupName` 參數會指定建立使用者指派之身分識別所在的資源群組，而 `-Name` 參數會指定其名稱。 將 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 參數取代為您自己的值：

 ```azurepowershell-interactive
Remove-AzurRmUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP> -Name <USER ASSIGNED IDENTITY NAME>
```
> [!NOTE]
> 將使用者指派的受控識別從受指派的任何資源中刪除，並不會移除參考。 必須個別移除身分識別指派。

## <a name="next-steps"></a>後續步驟

如需 Azure PowerShell Azure 資源受控識別命令的完整清單和更多詳細資料，請參閱 [AzureRM.ManagedServiceIdentity](/powershell/module/azurerm.managedserviceidentity#managed_service_identity)。
