---
title: 在 Azure Active Directory 中將使用者或群組指派給企業應用程式 | Microsoft Docs
description: 如何在 Azure Active Directory 中選取企業應用程式以將使用者或群組指派給它
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/06/2018
ms.author: barbkess
ms.reviewer: luleon
ms.openlocfilehash: f23c9976dacc1ca696772d6bf02b5d59e3e0b4d5
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368686"
---
# <a name="assign-a-user-or-group-to-an-enterprise-app-in-azure-active-directory"></a>在 Azure Active Directory 中將使用者或群組指派給企業應用程式
若要將使用者或群組指派給企業應用程式，您必須具備適當的權限，才能管理企業應用程式，而且必須是目錄的全域管理員。

> [!NOTE]
> 本文中討論的這些功能都需要 Azure Active Directory Premium P1 或 Premium P2 授權。 如需詳細資訊，請參閱 [Azure Active Directory 定價頁面](https://azure.microsoft.com/pricing/details/active-directory)。

> [!NOTE]
> 如果是 Microsoft 應用程式 (例如 Office 365 應用程式)，請使用 PowerShell 將使用者指派給企業應用程式。


## <a name="how-do-i-assign-user-access-to-an-enterprise-app-in-the-azure-portal"></a>如何在 Azure 入口網站中將企業應用程式的存取權指派給使用者？
1. 使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2. 選取 [所有服務]、在文字方塊中輸入 Azure Active Directory，然後選取 **Enter** 鍵。
3. 在 [Azure Active Directory - directoryname] 刀鋒視窗 (也就是您所管理目錄的 Azure AD 刀鋒視窗) 上，選取 [企業應用程式]。

    ![開啟企業應用程式](./media/assign-user-or-group-access-portal/open-enterprise-apps.png)
4. 在 [企業應用程式] 刀鋒視窗上，選取 [所有應用程式]。 此動作會列出您可以管理的應用程式。
5. 在 [企業應用程式 - 所有應用程式]  刀鋒視窗上，選取一個應用程式。
6. 在 [***appname***] 刀鋒視窗 (亦即標題中含有所選應用程式名稱的刀鋒視窗) 上，選取 [使用者和群組]。

    ![選取所有應用程式命令](./media/assign-user-or-group-access-portal/select-app-users.png)
7. 在 [***appname*** - 使用者和群組指派] 刀鋒視窗上，選取 [新增] 命令。
8. 在 [新增指派] 刀鋒視窗上，選取 [使用者和群組]。

    ![將使用者或群組指派給應用程式](./media/assign-user-or-group-access-portal/assign-users.png)
9. 在 [使用者和群組] 刀鋒視窗上，從清單中選取一或多個使用者或群組，然後選取刀鋒視窗底部的 [選取] 按鈕。
10. 在 [新增指派] 刀鋒視窗上，選取 [角色]。 然後，在 [選取角色] 刀鋒視窗上，選取要套用到所選使用者或群組的角色，然後選取刀鋒視窗底部的 [確定] 按鈕。
11. 在 [新增指派] 刀鋒視窗上，選取刀鋒視窗底部的 [指派] 按鈕。 受指派的使用者或群組將會具備此企業應用程式的所選角色所定義的權限。

## <a name="how-do-i-assign-a-user-to-an-enterprise-app-using-powershell"></a>如何使用 PowerShell 將使用者指派給企業應用程式？

1. 開啟已提高權限的 Windows PowerShell 命令提示字元。

    >[!NOTE] 
    > 您必須安裝 AzureAD 模組 (使用命令 `Install-Module -Name AzureAD`)。 如果系統提示您安裝 NuGet 模組或新的 Azure Active Directory V2 PowerShell 模組，請輸入 Y 然後按 ENTER。

2. 執行 `Connect-AzureAD` 並以全域管理員的使用者帳戶登入。
3. 您可以使用下列指令碼，將使用者和角色指派給應用程式：

    ```powershell
    # Assign the values to the variables
    $username = "<You user's UPN>"
    $app_name = "<Your App's display name>"
    $app_role_name = "<App role display name>"
    
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```     

如需有關如何將使用者指派給應用程式角色的詳細資訊，請瀏覽 [New-AzureADUserAppRoleAssignment](https://docs.microsoft.com/powershell/module/azuread/new-azureaduserapproleassignment?view=azureadps-2.0) 的文件

若要將群組指派給企業應用程式，您必須將 `Get-AzureADUser` 取代為 `Get-AzureADGroup`。

### <a name="example"></a>範例

此範例會使用 PowerShell 將使用者 Britta Simon 指派至 [Microsoft 工作場所分析](https://products.office.com/business/workplace-analytics)應用程式。

1. 在 PowerShell 中，將對應值指派給變數 $username、$app_name 和 $app_role_name。 

    ```powershell
    # Assign the values to the variables
    $username = "britta.simon@contoso.com"
    $app_name = "Workplace Analytics"
    ```

2. 在此範例中，我們不清楚要指派給 Britta Simon 的應用程式角色是什麼名稱。 執行下列命令，以利用使用者 UPN 和服務主體顯示名稱來取得使用者 ($user) 和服務主體 ($sp)。

    ```powershell
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    ```
        
3. 執行命令 `$sp.AppRoles`，顯示工作場所分析應用程式可用的角色。 在此範例中，我們要指派「分析師」(存取權有限) 角色給 Britta Simon。
    
    ![工作場所分析角色](./media/assign-user-or-group-access-portal/workplace-analytics-role.png)

4. 指派角色名稱給 `$app_role_name` 變數。
        
    ```powershell
    # Assign the values to the variables
    $app_role_name = "Analyst (Limited access)"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    ```

5. 執行下列命令，將使用者指派給應用程式角色：

    ```powershell
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```

## <a name="next-steps"></a>後續步驟
* [查看我的所有群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [從企業應用程式中移除使用者或群組指派](remove-user-or-group-access-portal.md)
* [停用企業應用程式的使用者登入](disable-user-sign-in-portal.md)
* [變更企業應用程式的名稱或標誌](change-name-or-logo-portal.md)
