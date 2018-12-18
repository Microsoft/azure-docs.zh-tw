---
title: 使用 Azure PowerShell 建立自訂角色 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 來建立適用於角色型存取控制 (RBAC) 的自訂角色。 這包括如何列出、建立、更新及刪除自訂角色。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/20/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: c8e5f34bb6b38a3f187d86a1ebc0c7019c7f1046
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37437013"
---
# <a name="create-custom-roles-using-azure-powershell"></a>使用 Azure PowerShell 建立自訂角色

如果內建的角色無法滿足組織的特定需求，您可以建立自己的[自訂角色](built-in-roles.md)。 本文說明如何使用 Azure PowerShell 來建立和管理自訂角色。

## <a name="prerequisites"></a>先決條件

若要建立自訂角色，您需要：

- 建立自訂角色的權限，例如[擁有者](built-in-roles.md#owner)或[使用者存取管理員](built-in-roles.md#user-access-administrator)
- 已在本機安裝 [Azure PowerShell](/powershell/azure/install-azurerm-ps)

## <a name="list-custom-roles"></a>列出自訂角色

若要列出範圍中可供指派的角色，使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition) 命令。 下列範例會列出選取的訂用帳戶中可供指派的所有角色。

```azurepowershell
Get-AzureRmRoleDefinition | FT Name, IsCustom
```

```Example
Name                                              IsCustom
----                                              --------
Virtual Machine Operator                              True
AcrImageSigner                                       False
AcrQuarantineReader                                  False
AcrQuarantineWriter                                  False
API Management Service Contributor                   False
...
```

下列範例只會列出所選取訂用帳戶中可供指派的自訂角色。

```azurepowershell
Get-AzureRmRoleDefinition | ? {$_.IsCustom -eq $true} | FT Name, IsCustom
```

```Example
Name                     IsCustom
----                     --------
Virtual Machine Operator     True
```

如果選取的訂用帳戶不在角色的 `AssignableScopes` 中，將不會列出自訂角色。

## <a name="create-a-custom-role"></a>建立自訂角色

若要建立自訂角色，使用 [New-AzureRmRoleDefinition](/powershell/module/azurerm.resources/new-azurermroledefinition) 命令。 建構角色有兩種方法：使用 `PSRoleDefinition` 物件或 JSON 範本。 

### <a name="get-operations-for-a-resource-provider"></a>取得資源提供者的作業

當您建立自訂角色時，務必知道所有可能來自資源提供者的作業。
您可以檢視[資源提供者作業](resource-provider-operations.md)的清單，或者，可以使用 [Get-AzureRMProviderOperation](/powershell/module/azurerm.resources/get-azurermprovideroperation) 命令來取得此資訊。
例如，如果您想要檢查虛擬機器的所有可用作業，請使用此命令：

```azurepowershell
Get-AzureRMProviderOperation <operation> | FT OperationName, Operation, Description -AutoSize
```

```Example
PS C:\> Get-AzureRMProviderOperation "Microsoft.Compute/virtualMachines/*" | FT OperationName, Operation, Description -AutoSize

OperationName                                  Operation                                                      Description
-------------                                  ---------                                                      -----------
Get Virtual Machine                            Microsoft.Compute/virtualMachines/read                         Get the propertie...
Create or Update Virtual Machine               Microsoft.Compute/virtualMachines/write                        Creates a new vir...
Delete Virtual Machine                         Microsoft.Compute/virtualMachines/delete                       Deletes the virtu...
Start Virtual Machine                          Microsoft.Compute/virtualMachines/start/action                 Starts the virtua...
...
```

### <a name="create-a-custom-role-with-the-psroledefinition-object"></a>使用 PSRoleDefinition 物件建立自訂角色

使用 PowerShell 建立自訂角色時，您可以使用其中一個[內建角色](built-in-roles.md)當作起點，或者可以從頭開始。 本節的範例是以內建角色當作起點，然後使用較高權限來自訂它。 編輯屬性來新增您所需的 `Actions`、`NotActions` 或 `AssignableScopes`，然後將變更儲存為新角色。

下列範例會從[虛擬機器參與者](built-in-roles.md#virtual-machine-contributor)內建角色開始，來建立稱為「虛擬機器操作員」(Virtual Machine Operator) 的自訂角色。 新角色會授與對 *Microsoft.Compute*、*Microsoft.Storage* 和 *Microsoft.Network* 資源提供者之所有讀取作業的存取權，以及授與對啟動、重新啟動和監視虛擬機器的存取權。 自訂角色可用於兩個訂用帳戶中。

```azurepowershell
$role = Get-AzureRmRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Virtual Machine Operator"
$role.Description = "Can monitor and restart virtual machines."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/*/read")
$role.Actions.Add("Microsoft.Network/*/read")
$role.Actions.Add("Microsoft.Compute/*/read")
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Authorization/*/read")
$role.Actions.Add("Microsoft.Resources/subscriptions/resourceGroups/read")
$role.Actions.Add("Microsoft.Insights/alertRules/*")
$role.Actions.Add("Microsoft.Support/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/00000000-0000-0000-0000-000000000000")
$role.AssignableScopes.Add("/subscriptions/11111111-1111-1111-1111-111111111111")
New-AzureRmRoleDefinition -Role $role
```

下列範例示範建立「虛擬機器操作員」自訂角色的另一種方法。 它一開始會建立新的 `PSRoleDefinition` 物件。 動作作業會指定於 `perms` 變數中，並設定為 `Actions` 屬性。 從[虛擬機器參與者](built-in-roles.md#virtual-machine-contributor)內建角色讀取 `NotActions`，藉以設定 `NotActions` 屬性。 由於[虛擬機器參與者](built-in-roles.md#virtual-machine-contributor)沒有任何 `NotActions`，因此不需要這一行，但它會顯示如何從另一個角色擷取資訊。

```azurepowershell
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Name = 'Virtual Machine Operator 2'
$role.Description = 'Can monitor and restart virtual machines.'
$role.IsCustom = $true
$perms = 'Microsoft.Storage/*/read','Microsoft.Network/*/read','Microsoft.Compute/*/read'
$perms += 'Microsoft.Compute/virtualMachines/start/action','Microsoft.Compute/virtualMachines/restart/action'
$perms += 'Microsoft.Authorization/*/read','Microsoft.Resources/subscriptions/resourceGroups/read'
$perms += 'Microsoft.Insights/alertRules/*','Microsoft.Support/*'
$role.Actions = $perms
$role.NotActions = (Get-AzureRmRoleDefinition -Name 'Virtual Machine Contributor').NotActions
$subs = '/subscriptions/00000000-0000-0000-0000-000000000000','/subscriptions/11111111-1111-1111-1111-111111111111'
$role.AssignableScopes = $subs
New-AzureRmRoleDefinition -Role $role
```

### <a name="create-a-custom-role-with-json-template"></a>使用 JSON 範本建立自訂角色

JSON 範本可作為自訂角色的來源定義。 下列範例會建立自訂角色來允許讀取儲存體和計算資源及存取支援，然後將該角色新增至兩個訂用帳戶。 建立含有下列範例的新檔案 `C:\CustomRoles\customrole1.json`。 最初建立角色時，應該將 Id 設為 `null`，因為會自動產生新的識別碼。 

```json
{
  "Name": "Custom Role 1",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

若要將角色新增至訂用帳戶，請執行下列 PowerShell 命令︰

```azurepowershell
New-AzureRmRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="update-a-custom-role"></a>更新自訂角色

類似於建立自訂角色，您可以使用 `PSRoleDefinition` 物件或 JSON 範本來修改現有的自訂角色。

### <a name="update-a-custom-role-with-the-psroledefinition-object"></a>使用 PSRoleDefinition 物件更新自訂角色

若要修改自訂角色，首先使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition) 命令來擷取角色定義。 接著，對角色定義進行想要的變更。 最後，使用 [Set-AzureRmRoleDefinition](/powershell/module/azurerm.resources/set-azurermroledefinition) 命令來儲存已修改的角色定義。

下列範例會將 `Microsoft.Insights/diagnosticSettings/*` 作業加入到 *Virtual Machine Operator* 自訂角色。

```azurepowershell
$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
Set-AzureRmRoleDefinition -Role $role
```

```Example
PS C:\> $role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
PS C:\> $role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
PS C:\> Set-AzureRmRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}
```

下列範例會將 Azure 訂用帳戶新增到 *Virtual Machine Operator* 自訂角色的可指派範圍。

```azurepowershell
Get-AzureRmSubscription -SubscriptionName Production3

$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
Set-AzureRmRoleDefinition -Role $role
```

```Example
PS C:\> Get-AzureRmSubscription -SubscriptionName Production3

Name     : Production3
Id       : 22222222-2222-2222-2222-222222222222
TenantId : 99999999-9999-9999-9999-999999999999
State    : Enabled

PS C:\> $role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
PS C:\> $role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
PS C:\> Set-AzureRmRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111,
                   /subscriptions/22222222-2222-2222-2222-222222222222}
```

### <a name="update-a-custom-role-with-a-json-template"></a>使用 JSON 範本更新自訂角色

您可以利用先前的 JSON 範本，輕鬆修改現有的自訂角色來新增或移除 Actions。 更新 JSON 範本，並新增網路的讀取動作，如下範例所示。 範本中所列的定義不會累積套用至現有的定義，這表示角色會完全依照您在範本中指定的樣子出現。 您也需要以角色的識別碼來更新 Id 欄位。 如果不確定此值，您可以使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition) Cmdlet 來取得這項資訊。

```json
{
  "Name": "Custom Role 1",
  "Id": "acce7ded-2559-449d-bcd5-e9604e50bad1",
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

若要更新現有的角色，請執行下列 PowerShell 命令︰

```azurepowershell
Set-AzureRmRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="delete-a-custom-role"></a>刪除自訂角色

若要刪除自訂角色，使用 [Remove-AzureRmRoleDefinition](/powershell/module/azurerm.resources/remove-azurermroledefinition) 命令。

下列範例會移除 *Virtual Machine Operator* 自訂角色

```azurepowershell
Get-AzureRmRoleDefinition "Virtual Machine Operator"
Get-AzureRmRoleDefinition "Virtual Machine Operator" | Remove-AzureRmRoleDefinition
```

```Example
PS C:\> Get-AzureRmRoleDefinition "Virtual Machine Operator"

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}

PS C:\> Get-AzureRmRoleDefinition "Virtual Machine Operator" | Remove-AzureRmRoleDefinition

Confirm
Are you sure you want to remove role definition with name 'Virtual Machine Operator'.
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```

## <a name="next-steps"></a>後續步驟

- [教學課程：使用 Azure PowerShell 建立自訂角色](tutorial-custom-role-powershell.md)
- [Azure 中的自訂角色](custom-roles.md)
- [Azure Resource Manager 資源提供者作業](resource-provider-operations.md)
