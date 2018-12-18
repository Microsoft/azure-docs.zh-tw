---
title: 在 Azure Active Directory 中從企業應用程式移除使用者或群組指派 | Microsoft Docs
description: 如何在 Azure Active Directory 中從企業應用程式移除使用者或群組的存取權指派
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
ms.date: 02/14/2018
ms.author: barbkess
ms.reviewer: asteen
ms.custom: it-pro
ms.openlocfilehash: fde6d5fa2488d86af542f409df7c5b76d2510f08
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368646"
---
# <a name="remove-a-user-or-group-assignment-from-an-enterprise-app-in-azure-active-directory"></a>在 Azure Active Directory 中從企業應用程式移除使用者或群組指派
在 Azure Active Directory (Azure AD) 中，針對已獲指派其中一個企業應用程式存取權的使用者或群組，您可以輕鬆移除已指派的存取權。 您必須具備適當的權限，才能管理企業應用程式，而且必須是目錄的全域管理員。

> [!NOTE]
> 如果是 Microsoft 應用程式 (例如 Office 365 應用程式)，請使用 PowerShell 移除企業應用程式的使用者。

## <a name="how-do-i-remove-a-user-or-group-assignment-to-an-enterprise-app-in-the-azure-portal"></a>如何在 Azure 入口網站中移除對企業應用程式的使用者或群組指派？
1. 使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2. 選取 [更多服務]，在文字方塊中輸入 **Azure Active Directory**，然後選取 **Enter**。
3. 在 [Azure Active Directory - directoryname] 頁面 (也就是您所管理之目錄的 Azure AD 頁面) 上，選取 [企業應用程式]。

    ![開啟企業應用程式](./media/remove-user-or-group-access-portal/open-enterprise-apps.png)
4. 在 [企業應用程式] 頁面上，選取 [所有應用程式]。 您將會看到一份您可以管理的應用程式清單。
5. 在 [企業應用程式 - 所有應用程式]  頁面上，選取一個應用程式。
6. 在 [appname] 頁面 (亦即標題中含有所選應用程式名稱的頁面) 上，選取 [使用者和群組]。

    ![選取使用者或群組](./media/remove-user-or-group-access-portal/remove-app-users.png)
7. 在 [appname - 使用者和群組指派] 頁面上，選取一或多個使用者或群組，然後選取 [移除] 命令。 出現提示時，請確認您的決定。

    ![選取 [移除] 命令](./media/remove-user-or-group-access-portal/remove-users.png)

## <a name="how-do-i-remove-a-user-or-group-assignment-to-an-enterprise-app-using-powershell"></a>如何使用 PowerShell 移除對企業應用程式的使用者或群組指派？
1. 開啟已提高權限的 Windows PowerShell 命令提示字元。

    >[!NOTE] 
    > 您必須安裝 AzureAD 模組 (使用命令 `Install-Module -Name AzureAD`)。 如果系統提示您安裝 NuGet 模組或新的 Azure Active Directory V2 PowerShell 模組，請輸入 Y 然後按 ENTER。

2. 執行 `Connect-AzureAD` 並以全域管理員的使用者帳戶登入。
3. 您可以使用下列指令碼，將使用者和角色指派給應用程式：

    ```powershell
    # Store the proper parameters
    $user = get-azureaduser -ObjectId <objectId>
    $spo = Get-AzureADServicePrincipal -ObjectId <objectId>

    #Get the ID of role assignment 
    $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId | Where {$_.PrincipalDisplayName -eq $user.DisplayName}

    #if you run the following, it will show you what is assigned what
    $assignments | Select *

    #To remove the App role assignment run the following command.
    Remove-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId -AppRoleAssignmentId $assignments[assignment #].ObjectId
    ``` 
## <a name="next-steps"></a>後續步驟

- [查看我的所有群組](../fundamentals/active-directory-groups-view-azure-portal.md)
- [將使用者或群組指派給企業應用程式](assign-user-or-group-access-portal.md)
- [停用企業應用程式的使用者登入](disable-user-sign-in-portal.md)
- [變更企業應用程式的名稱或標誌](change-name-or-logo-portal.md)
