---
title: 使用 RBAC 和 Azure PowerShell 來管理存取權 | Microsoft Docs
description: 了解如何使用角色型存取控制 (RBAC) 和 Azure PowerShell 來管理使用者、群組和應用程式的存取權。 這包括如何列出存取權、授與存取權以及移除存取權。
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
ms.openlocfilehash: dcc324f3b6c5153b294719ff8939aed8a3e82b1c
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37436825"
---
# <a name="manage-access-using-rbac-and-azure-powershell"></a>使用 RBAC 和 Azure PowerShell 來管理存取權

[角色型存取控制 (RBAC)](overview.md) 是您對 Azure 中的資源存取進行管理的機制。 本文將描述如何使用 RBAC 和 Azure PowerShell 來管理使用者、群組和應用程式的存取權。

## <a name="prerequisites"></a>先決條件

若要管理存取權，您需要下列其中一項：

* [Azure Cloud Shell 中的 PowerShell](/azure/cloud-shell/overview)
* [Azure PowerShell](/powershell/azure/install-azurerm-ps)

## <a name="list-roles"></a>列出角色

### <a name="list-all-available-roles"></a>列出所有可用的角色

若要列出可供指派的 RBAC 角色，以及檢查它們授與存取的作業，使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition)。

```azurepowershell
Get-AzureRmRoleDefinition | FT Name, Description
```

```Example
AcrImageSigner                                    acr image signer
AcrQuarantineReader                               acr quarantine data reader
AcrQuarantineWriter                               acr quarantine data writer
API Management Service Contributor                Can manage service and the APIs
API Management Service Operator Role              Can manage service but not the APIs
API Management Service Reader Role                Read-only access to service and APIs
Application Insights Component Contributor        Can manage Application Insights components
Application Insights Snapshot Debugger            Gives user permission to use Application Insights Snapshot Debugge...
Automation Job Operator                           Create and Manage Jobs using Automation Runbooks.
Automation Operator                               Automation Operators are able to start, stop, suspend, and resume ...
...
```

### <a name="list-a-specific-role"></a>列出特定角色

若要列出特定角色，使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition)。

```azurepowershell
Get-AzureRmRoleDefinition <role name>
```

```Example
PS C:\> Get-AzureRmRoleDefinition "Contributor"

Name             : Contributor
Id               : b24988ac-6180-42a0-ab88-20f7382dd24c
IsCustom         : False
Description      : Lets you manage everything except access to resources.
Actions          : {*}
NotActions       : {Microsoft.Authorization/*/Delete, Microsoft.Authorization/*/Write,
                   Microsoft.Authorization/elevateAccess/Action}
AssignableScopes : {/}
```

### <a name="list-a-specific-role-in-json-format"></a>使用 JSON 格式列出特定角色

若要使用 JSON 格式列出特定角色，使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition)。

```azurepowershell
Get-AzureRmRoleDefinition <role name> | ConvertTo-Json
```

```Example
PS C:\> Get-AzureRmRoleDefinition "Contributor" | ConvertTo-Json

{
    "Name":  "Contributor",
    "Id":  "b24988ac-6180-42a0-ab88-20f7382dd24c",
    "IsCustom":  false,
    "Description":  "Lets you manage everything except access to resources.",
    "Actions":  [
                    "*"
                ],
    "NotActions":  [
                       "Microsoft.Authorization/*/Delete",
                       "Microsoft.Authorization/*/Write",
                       "Microsoft.Authorization/elevateAccess/Action"
                   ],
    "AssignableScopes":  [
                             "/"
                         ]
}
```

### <a name="list-actions-of-a-role"></a>列出角色的動作

若要列出特定角色的動作，使用 [Get-AzureRmRoleDefinition](/powershell/module/azurerm.resources/get-azurermroledefinition)。

```azurepowershell
Get-AzureRmRoleDefinition <role name> | FL Actions, NotActions
```

```Example
PS C:\> Get-AzureRmRoleDefinition "Contributor" | FL Actions, NotActions

Actions    : {*}
NotActions : {Microsoft.Authorization/*/Delete, Microsoft.Authorization/*/Write,
             Microsoft.Authorization/elevateAccess/Action}
```

```azurepowershell
(Get-AzureRmRoleDefinition <role name>).Actions
```

```Example
PS C:\> (Get-AzureRmRoleDefinition "Virtual Machine Contributor").Actions

Microsoft.Authorization/*/read
Microsoft.Compute/availabilitySets/*
Microsoft.Compute/locations/*
Microsoft.Compute/virtualMachines/*
Microsoft.Compute/virtualMachineScaleSets/*
Microsoft.DevTestLab/schedules/*
Microsoft.Insights/alertRules/*
Microsoft.Network/applicationGateways/backendAddressPools/join/action
Microsoft.Network/loadBalancers/backendAddressPools/join/action
...
```

## <a name="list-access"></a>列出存取權

在 RBAC 中，若要列出存取權，您可以列出角色指派。

### <a name="list-role-assignments-at-a-specific-scope"></a>列出特定範圍的角色指派

您可以查看指定訂用帳戶、資源群組或資源的所有角色指派。 例如，若要查看某個資源群組的所有使用中指派，使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment)。

```azurepowershell
Get-AzureRmRoleAssignment -ResourceGroupName <resource group name>
```

```Example
PS C:\> Get-AzureRmRoleAssignment -ResourceGroupName pharma-sales-projectforecast | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Alain Charon
RoleDefinitionName : Backup Operator
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast

DisplayName        : Alain Charon
RoleDefinitionName : Virtual Machine Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast
```

### <a name="list-role-assignments-for-a-user"></a>列出使用者的角色指派

若要列出指派給指定使用者的所有角色，使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment)。

```azurepowershell
Get-AzureRmRoleAssignment -SignInName <user email>
```

```Example
PS C:\> Get-AzureRmRoleAssignment -SignInName isabella@example.com | FL DisplayName, RoleDefinitionName, Scope

DisplayName        : Isabella Simonsen
RoleDefinitionName : BizTalk Contributor
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast
```

若要列出指派給指定使用者的所有角色，以及指派給該使用者所屬群組的角色，使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment)。

```azurepowershell
Get-AzureRmRoleAssignment -SignInName <user email> -ExpandPrincipalGroups
```

```Example
Get-AzureRmRoleAssignment -SignInName isabella@example.com -ExpandPrincipalGroups | FL DisplayName, RoleDefinitionName, Scope
```

### <a name="list-role-assignments-for-classic-service-administrator-and-co-administrators"></a>列出傳統服務管理員和共同管理員的角色指派

若要列出傳統訂用帳戶管理員和共同管理員的角色指派，請使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment)。

```azurepowershell
Get-AzureRmRoleAssignment -IncludeClassicAdministrators
```

## <a name="grant-access"></a>授與存取權

在 RBAC 中，若要授與存取權，您可以建立角色指派。

### <a name="search-for-object-ids"></a>搜尋物件識別碼

若要指派角色，您必須識別物件 (使用者、群組或應用程式) 和範圍。

如果您不知道訂用帳戶 ID，可以在 Azure 入口網站的 [訂用帳戶] 刀鋒視窗中找到，或者，可以使用 [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription)。

若要取得 Azure AD 群組的物件識別碼，使用 [Get-AzureRmADGroup](/powershell/module/azurerm.resources/get-azurermadgroup)：

```azurepowershell
Get-AzureRmADGroup -SearchString <group name in quotes>
```

若要取得 Azure AD 服務主體或應用程式的物件識別碼，請使用 [Get-AzureRmADServicePrincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal)。

```azurepowershell
Get-AzureRmADServicePrincipal -SearchString <service name in quotes>
```

### <a name="create-a-role-assignment-for-an-application-at-a-subscription-scope"></a>為訂用帳戶範圍的應用程式建立角色指派

若要對訂用帳戶範圍的應用程式授與存取權，請使用 [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment)。

```azurepowershell
New-AzureRmRoleAssignment -ObjectId <application id> -RoleDefinitionName <role name> -Scope <subscription id>
```

```Example
PS C:\> New-AzureRmRoleAssignment -ObjectId 77777777-7777-7777-7777-777777777777 -RoleDefinitionName "Reader" -Scope /subscriptions/00000000-0000-0000-0000-000000000000

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/66666666-6666-6666-6666-666666666666
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
DisplayName        : MyApp1
SignInName         :
RoleDefinitionName : Reader
RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
ObjectId           : 77777777-7777-7777-7777-777777777777
ObjectType         : ServicePrincipal
CanDelegate        : False
```

### <a name="create-a-role-assignment-for-a-user-at-a-resource-group-scope"></a>為資源群組範圍的使用者建立角色指派

若要對資源群組範圍的使用者授與存取權，請使用 [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment)。

```azurepowershell
New-AzureRmRoleAssignment -SignInName <email of user> -RoleDefinitionName <role name in quotes> -ResourceGroupName <resource group name>
```

```Example
PS C:\> New-AzureRmRoleAssignment -SignInName alain@example.com -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName pharma-sales-projectforecast


RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast/pr
                     oviders/Microsoft.Authorization/roleAssignments/55555555-5555-5555-5555-555555555555
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales-projectforecast
DisplayName        : Alain Charon
SignInName         : alain@example.com
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : 44444444-4444-4444-4444-444444444444
ObjectType         : User
CanDelegate        : False
```

### <a name="create-a-role-assignment-for-a-group-at-a-resource-scope"></a>為資源範圍的群組建立角色指派

若要對資源群組範圍的群組授與存取權，請使用 [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment)。

```azurepowershell
New-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name in quotes> -ResourceName <resource name> -ResourceType <resource type> -ParentResource <parent resource> -ResourceGroupName <resource group name>
```

```Example
PS C:\> Get-AzureRmADGroup -SearchString "Pharma"

SecurityEnabled DisplayName         Id                                   Type
--------------- -----------         --                                   ----
           True Pharma Sales Admins aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa Group

PS C:\> New-AzureRmRoleAssignment -ObjectId aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa -RoleDefinitionName "Virtual Machine Contributor" -ResourceName RobertVirtualNetwork -ResourceType Microsoft.Network/virtualNetworks -ResourceGroupName RobertVirtualNetworkResourceGroup

RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/RobertVirtualNetwork/providers/Microsoft.Authorizat
                     ion/roleAssignments/bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup
                     /providers/Microsoft.Network/virtualNetworks/RobertVirtualNetwork
DisplayName        : Pharma Sales Admins
SignInName         :
RoleDefinitionName : Virtual Machine Contributor
RoleDefinitionId   : 9980e02c-c2be-4d73-94e8-173b1dc7cf3c
ObjectId           : aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
ObjectType         : Group
CanDelegate        : False
```

## <a name="remove-access"></a>移除存取

在 RBAC 中，若要移除存取權，您需使用 [Remove-AzureRmRoleAssignment](/powershell/module/azurerm.resources/remove-azurermroleassignment) 來移除角色指派。

```azurepowershell
Remove-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name> -Scope <scope such as subscription id>
```

```Example
PS C:\> Remove-AzureRmRoleAssignment -SignInName alain@example.com -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName pharma-sales-projectforecast
```

## <a name="next-steps"></a>後續步驟

- [教學課程：使用 RBAC 與 Azure PowerShell 為群組授與存取權](tutorial-role-assignments-group-powershell.md)
- [教學課程：使用 Azure PowerShell 建立自訂角色](tutorial-custom-role-powershell.md)
- [使用 Azure PowerShell 管理資源](../azure-resource-manager/powershell-azure-resource-manager.md)
