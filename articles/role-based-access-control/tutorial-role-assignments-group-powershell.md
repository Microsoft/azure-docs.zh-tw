---
title: 教學課程 - 使用 RBAC 與 Azure PowerShell 為群組授與存取權 | Microsoft Docs
description: 透過 Azure PowerShell 使用角色型存取控制 (RBAC) 為群組授與在訂用帳戶中檢視任何項目以及管理資源群組中所有項目的存取權。
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 06/11/2018
ms.author: rolyon
ms.openlocfilehash: 8bb06493683dabb92dfe75f371f96db14a7951b3
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43300998"
---
# <a name="tutorial-grant-access-for-a-group-using-rbac-and-azure-powershell"></a>教學課程 - 使用 RBAC 與 Azure PowerShell 為群組授與存取權

[角色型存取控制 (RBAC)](overview.md) 是您對 Azure 中的資源存取進行管理的機制。 在本教學課程中，您會使用 Azure PowerShell 為群組授與在訂用帳戶中檢視任何項目以及管理資源群組中所有項目的存取權。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在不同範圍授與群組存取權
> * 列出存取權
> * 移除存取

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

- 在 Azure Active Directory 中建立群組的權限 (或使用現有的群組)
- [Azure Cloud Shell](/azure/cloud-shell/quickstart-powershell)

## <a name="role-assignments"></a>角色指派

在 RBAC 中，若要授與存取權，您可以建立角色指派。 角色指派由三項元素所組成：安全性主體、角色定義和範圍。 以下是您將在本教學課程中執行的兩個角色指派：

| 安全性主體 | 角色定義 | 影響範圍 |
| --- | --- | --- |
| 群組<br>(RBAC 教學課程群組) | [讀取者](built-in-roles.md#reader) | 訂用帳戶 |
| 群組<br>(RBAC 教學課程群組)| [參與者](built-in-roles.md#contributor) | 資源群組<br>(rbac-tutorial-resource-group) |

   ![群組的角色指派](./media/tutorial-role-assignments-group-powershell/rbac-role-assignments.png)

## <a name="create-a-group"></a>建立群組

若要指派角色，您必須要有使用者、群組或服務主體。 如果您還沒有群組，您可以建立一個。

- 在 Azure Cloud Shell 中，使用 [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup) 命令建立新的群組。

   ```azurepowershell
   New-AzureADGroup -DisplayName "RBAC Tutorial Group" `
     -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
   ```

   ```Example
   ObjectId                             DisplayName         Description
   --------                             -----------         -----------
   11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
   ```

如果您沒有建立群組的權限，您可以嘗試改用[教學課程：使用 RBAC 與 Azure PowerShell 為使用者授與存取權](tutorial-role-assignments-user-powershell.md)。

## <a name="create-a-resource-group"></a>建立資源群組

您可以使用資源群組顯示如何指派資源群組範圍的角色。

1. 使用 [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation) 命令取得區域位置清單。

   ```azurepowershell
   Get-AzureRmLocation | select Location
   ```

1. 選取您附近的位置，並將其指派給變數。

   ```azurepowershell
   $location = "westus"
   ```

1. 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立新的資源群組。

   ```azurepowershell
   New-AzureRmResourceGroup -Name "rbac-tutorial-resource-group" -Location $location
   ```

   ```Example
   ResourceGroupName : rbac-tutorial-resource-group
   Location          : westus
   ProvisioningState : Succeeded
   Tags              :
   ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
   ```

## <a name="grant-access"></a>授與存取權

若要為群組授與存取權，您可以使用 [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment) 命令來指派角色。 您必須指定安全性主體、角色定義和範圍。

1. 使用 [Get-AzureADGroup](/powershell/module/azuread/new-azureadgroup) 命令取得群組的物件識別碼。

    ```azurepowershell
    Get-AzureADGroup -SearchString "RBAC Tutorial Group"
    ```

    ```Example
    ObjectId                             DisplayName         Description
    --------                             -----------         -----------
    11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
    ```

1. 在變數中儲存群組物件識別碼。

    ```azurepowershell
    $groupId = "11111111-1111-1111-1111-111111111111"
    ```

1. 使用 [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription) 命令取得訂用帳戶的識別碼。

    ```azurepowershell
    Get-AzureRmSubscription
    ```

    ```Example
    Name     : Pay-As-You-Go
    Id       : 00000000-0000-0000-0000-000000000000
    TenantId : 22222222-2222-2222-2222-222222222222
    State    : Enabled
    ```

1. 將訂用帳戶範圍儲存在變數中。

    ```azurepowershell
    $subScope = "/subscriptions/00000000-0000-0000-0000-000000000000"
    ```

1. 在訂用帳戶範圍中，將[讀者](built-in-roles.md#reader)角色指派給群組。

    ```azurepowershell
    New-AzureRmRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/44444444-4444-4444-4444-444444444444
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

1. 在資源群組範圍中，將[參與者](built-in-roles.md#contributor)角色指派給群組。

    ```azurepowershell
    New-AzureRmRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

## <a name="list-access"></a>列出存取權

1. 若要確認訂用帳戶的存取權，請使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment) 命令列出角色指派。

    ```azurepowershell
    Get-AzureRmRoleAssignment -ObjectId $groupId -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    在輸出中，您可以看到「讀者」角色已在訂用帳戶範圍中指派給 RBAC 教學課程群組。

1. 若要確認資源群組的存取權，請使用 [Get-AzureRmRoleAssignment](/powershell/module/azurerm.resources/get-azurermroleassignment) 命令列出角色指派。

    ```azurepowershell
    Get-AzureRmRoleAssignment -ObjectId $groupId -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    在輸出中，您可以看到「參與者」和「讀者」角色都已指派給 RBAC 教學課程群組。 「參與者」角色會位於 rbac-tutorial-resource-group 範圍中，此外會在訂用帳戶範圍中繼承「讀者」角色。

## <a name="optional-list-access-using-the-azure-portal"></a>(選擇性) 使用 Azure 入口網站列出存取權

1. 若要查看角色指派在 Azure 入口網站中的顯示情形，請檢視訂用帳戶的 [存取控制 (IAM)] 刀鋒視窗。

    ![訂用帳戶範圍的群組角色指派](./media/tutorial-role-assignments-group-powershell/role-assignments-subscription.png)

1. 檢視資源群組的 [存取控制 (IAM)] 刀鋒視窗。

    ![資源群組範圍的群組角色指派](./media/tutorial-role-assignments-group-powershell/role-assignments-resource-group.png)

## <a name="remove-access"></a>移除存取

若要移除使用者、群組和應用程式的存取權，請使用 [Remove-AzureRmRoleAssignment](/powershell/module/azurerm.resources/remove-azurermroleassignment) 移除角色指派。

1. 使用下列命令，移除群組在資源群組範圍的「參與者」角色指派。

    ```azurepowershell
    Remove-AzureRmRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

1. 使用下列命令，移除群組在訂用帳戶範圍的「讀者」角色指派。

    ```azurepowershell
    Remove-AzureRmRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

## <a name="clean-up-resources"></a>清除資源

若要清除本教學課程所建立的資源，請刪除資源群組和群組。

1. 使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令刪除資源群組。

    ```azurepowershell
    Remove-AzureRmResourceGroup -Name "rbac-tutorial-resource-group"
    ```

    ```Example
    Confirm
    Are you sure you want to remove resource group 'rbac-tutorial-resource-group'
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    ```
    
1. 當系統要求您確認時，請輸入 **Y**。可能需要幾秒鐘才會刪除。

1. 使用 [Remove-AzureADGroup](/powershell/module/azuread/remove-azureadgroup) 命令刪除群組。

    ```azurepowershell
    Remove-AzureADGroup -ObjectId $groupId
    ```
    
    如果您在嘗試刪除群組時收到錯誤，您也可以在入口網站中刪除群組。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 RBAC 和 PowerShell 來管理存取權](role-assignments-powershell.md)
