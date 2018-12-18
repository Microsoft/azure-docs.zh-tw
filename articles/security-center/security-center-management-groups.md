---
title: 取得 Azure 資訊安全中心的全租用戶可見性 | Microsoft Docs
description: 了解如何在 Azure 資訊安全中心取得全租用戶可見性。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: b85c0e93-9982-48ad-b23f-53b367f22b10
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/22/2018
ms.author: terrylan
ms.openlocfilehash: 91d1be062dbf05f4c7c9c5c4a1eb3dfcfdb001af
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42441689"
---
# <a name="gain-tenant-wide-visibility-for-azure-security-center"></a>取得 Azure 資訊安全中心的全租用戶可見性
本文可協助您先執行數個可充分發揮 Azure Security Center 效用的動作，以著手使用。 執行這些動作可讓您檢視所有連結至 Azure Active Directory 租用戶的 Azure 訂用帳戶，並以彙總的方式跨多個訂用帳戶套用安全性原則，以有效而大規模地管理組織的安全性狀態。

## <a name="management-groups"></a>管理群組
Azure 管理群組可讓您有效管理訂用帳戶群組的存取、原則和報告，並藉由對根管理群組執行動作來有效管理整個 Azure 資產。 每個 Azure AD 租用戶都會有一個最上層管理群組，名為根管理群組。 這個根管理群組會建置於階層內，讓所有的管理群組和訂用帳戶摺疊於其中。 此群組可讓全域原則和 RBAC 指派套用於目錄層級。 

當您執行下列任何動作時，即會自動建立根管理群組： 
1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)中的 [管理群組]，以選擇使用 Azure 管理群組。
2. 透過 API 呼叫建立管理群組。
3. 透過 PowerShell 建立管理群組。

如需管理群組的詳細概觀，請參閱[使用 Azure 管理群組來組織資源](../azure-resource-manager/management-groups-overview.md)一文。

## <a name="create-a-management-group-in-the-azure-portal"></a>在 Azure 入口網站中建立管理群組
您可以將訂用帳戶組織到管理群組中，並將控管原則套用至管理群組。 管理群組內的所有訂用帳戶都會自動繼承套用到管理群組的原則。 當管理群組不需要加入資訊安全中心時，強烈建議您至少建立一個管理群組，以建立根管理群組。 在群組建立後，Azure AD 租用戶下的所有訂用帳戶都會連結至該群組。 如需 PowerShell 的指示和詳細資訊，請參閱[建立資源和組織管理的管理群組](../azure-resource-manager/management-groups-create.md)。

 
1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 選取 [所有服務] > [管理群組]。
3. 在主頁面上，選取 [新增管理群組]。 

    ![主要群組](./media/security-center-management-groups/main.png) 
4.  填寫管理群組識別碼欄位。 
    - [管理群組識別碼] 是用來在此管理群組上提交命令的目錄唯一識別碼。 此識別碼在建立後即無法編輯，因為整個 Azure 系統會使用它來識別此群組。 
    - [顯示名稱] 欄位是顯示在 Azure 入口網站內的名稱。 在建立管理群組時，不同的顯示名稱是選擇性欄位，並且可以隨時進行變更。  

      ![建立](./media/security-center-management-groups/create_context_menu.png)  
5.  選取 [儲存]。

### <a name="view-management-groups-in-the-azure-portal"></a>檢視 Azure 入口網站中的管理群組
1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 若要檢視管理群組，請在 Azure 主功能表下選取 [所有服務]。
3. 在 [一般] 下，選取 [管理群組]。

    ![建立管理群組](./media/security-center-management-groups/all-services.png)

## <a name="grant-tenant-level-visibility-and-the-ability-to-assign-policies"></a>授與租用戶層級的可見性和指派原則的能力

若要能夠檢視所有在 Azure AD 租用戶中註冊的訂用帳戶安全性狀態，必須在根管理群組上指派具有足夠讀權限的 RBAC 角色。

### <a name="elevate-access-for-a-global-administrator-in-azure-active-directory"></a>提高 Azure Active Directory 中全域管理員的存取權
Azure Active Directory 租用戶管理員並沒有 Azure 訂用帳戶的直接存取權。 但身為目錄管理員，他們有權將自身提高為具有存取權的角色。 Azure AD 租用戶管理員必須將自身提高為根管理群組層級的使用者存取管理員，才能指派 RBAC 角色。 如需 PowerShell 指示和相關資訊，請參閱[提高 Azure Active Directory 中全域管理員的存取權](../role-based-access-control/elevate-access-global-admin.md)。 


1. 登入 [Azure 入口網站](https://portal.azure.com)或 [Azure Active Directory 系統管理中心](https://aad.portal.azure.com)。

2. 在瀏覽清單中，按一下 [Azure Active Directory]，然後按一下 [屬性]。

   ![Azure AD 屬性 - 螢幕擷取畫面](./media/security-center-management-groups/aad-properties.png)

3. 在 [全域管理員可以管理 Azure 訂用帳戶與管理群組] 下，將開關設定為 [是]。

   ![全域管理員可以管理 Azure 訂用帳戶與管理群組 - 螢幕擷取畫面](./media/security-center-management-groups/aad-properties-global-admin-setting.png)

   - 當您將開關設定為 [是] 時，您的全域管理員帳戶 (目前登入的使用者) 會新增至根目錄範圍 (`/`) 之 Azure RBAC 中的使用者存取系統管理員角色，這會授權您存取與您的 Azure AD 租用戶建立關聯之所有 Azure 訂用帳戶中的檢視和報表。

   - 當您將開關設定為 [否] 時，您的全域管理員帳戶 (目前登入的使用者) 會從 Azure RBAC 的使用者存取系統管理員角色中移除。 您無法看到與 Azure AD 租用戶建立關聯的所有 Azure 訂用帳戶，而且您只能檢視及管理已獲得存取權的 Azure 訂用帳戶。

4. 按一下 [儲存] 儲存您的設定。

    - 此設定不是全域屬性，而且只會套用至目前登入的使用者。

5. 執行您需要以更高存取權完成的工作。 當您完成時，請將開關設回 [否]。


### <a name="assign-rbac-roles-to-users"></a>將 RBAC 角色指派給使用者
若要取得所有訂用帳戶的可見性，租用戶系統管理員必須在根管理群組層級，將適當的 RBAC 角色指派給他們想要授與整個租用戶可見性的任何使用者 (包括本身)。 建議指派的角色為 [安全性管理員] 或 [安全性讀取者]。 一般而言，需具備安全性管理員角色，才能在根層級套用原則，而安全性讀取者就足以提供租用戶層級的可見性。 如需有關這些角色所授與權限的詳細資訊，請參閱[安全性管理員內建角色描述](../role-based-access-control/built-in-roles.md#security-admin)或[安全性讀取者內建角色描述](../role-based-access-control/built-in-roles.md#security-reader)。


#### <a name="assign-rbac-roles-to-users-through-the-azure-portal"></a>透過 Azure 入口網站將 RBAC 角色指派給使用者： 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 若要檢視管理群組，請在 Azure 主功能表下選取 [所有服務]，然後選取 [管理群組]。
3.  選取管理群組，然後按一下 [詳細資料]。

    ![管理群組詳細資料螢幕擷取畫面](./media/security-center-management-groups/management-group-details.PNG)
 
4. 按一下 [存取控制 (IAM)]，然後按一下 [新增]。
5. 選取要指派的角色和使用者，然後按一下 [儲存]。  
   
   ![新增安全性讀取者角色螢幕擷取畫面](./media/security-center-management-groups/asc-security-reader.png)


#### <a name="assign-rbac-roles-to-users-with-powershell"></a>使用 PowerShell 將 RBAC 角色指派給使用者： 
1. 安裝 [Azure PowerShell](/powershell/azure/install-azurerm-ps)。
2. 執行下列命令： 

    ```azurepowershell
    # Install Management Groups Powershell module
    Install-Module AzureRM.Resources
    
    # Login to Azure as a Global Administrator user
    Login-AzureRmAccount
    ```

3. 出現提示時，請使用全域管理員認證登入。 

    ![登入提示螢幕擷取畫面](./media/security-center-management-groups/azurerm-sign-in.PNG)

4. 執行下列命令以授與讀者角色權限：

    ```azurepowershell
    # Add Reader role to the required user on the Root Management Group
    # Replace "user@domian.com” with the user to grant access to
    New-AzureRmRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/"
    ```
5. 若要移除角色，請使用下列命令： 

    ```azurepowershell
    Remove-AzureRmRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/" 
    ```

### <a name="open-or-refresh-security-center"></a>開啟或重新整理資訊安全中心
提高存取權限後，請開啟或重新整理 Azure 資訊安全中心，以確認您可以檢視 Azure AD 租用戶下的所有訂用帳戶。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 請務必在訂用帳戶選取器中，選取想要在資訊安全中心檢視的所有訂用帳戶。
    ![訂用帳戶選取器螢幕擷取畫面](./media/security-center-management-groups/subscription-selector.png)
1. 在 Azure 主功能表下選取 [所有服務]，然後選取 [資訊安全中心]。
2. 在 [概觀] 中，會有一個訂用帳戶涵蓋範圍表。 
    ![訂用帳戶涵蓋範圍表螢幕擷取畫面](./media/security-center-management-groups/security-center-subscription-coverage.png)
3. 按一下 [涵蓋範圍] 以查看涵蓋的訂用帳戶清單。 
    ![訂用帳戶涵蓋範圍清單螢幕擷取畫面](./media/security-center-management-groups/security-center-coverage.png)

### <a name="remove-elevated-access"></a>移除已提高的存取權 
在 RBAC 角色指派給使用者後，租用戶管理員應將本身從使用者存取管理員角色中移除。

1. 登入 [Azure 入口網站](https://portal.azure.com)或 [Azure Active Directory 系統管理中心](https://aad.portal.azure.com)。

2. 在瀏覽清單中，按一下 [Azure Active Directory]，然後按一下 [屬性]。

3. 在 [全域管理員可以管理 Azure 訂用帳戶與管理群組] 下，將開關設定為 [否]。

4. 按一下 [儲存] 儲存您的設定。



## <a name="adding-subscriptions-to-a-management-groups"></a>將訂用帳戶新增至管理群組
您可以將訂用帳戶新增至已建立的管理群組。 下列步驟並非取得全租用戶可見性以及全域原則和存取管理的必要步驟。

1. 在 [管理群組] 下，選取要新增訂用帳戶的管理群組。

    ![選取要新增訂用帳戶的管理群組](./media/security-center-management-groups/management-group-subscriptions.png)

2. 選取 [新增現有項目]。

    ![新增現有項目](./media/security-center-management-groups/add-existing.png)

3. 在 [新增現有資源] 下輸入訂用帳戶，然後按一下 [儲存]。

4. 重複步驟 1 到 3，直到您已新增範圍內的所有訂用帳戶為止。

 > [!NOTE]
 > 管理群組可同時包含訂用帳戶和子管理群組。 當您將使用者的 RBAC 角色指派給父管理群組時，子管理群組的訂用帳戶即會繼承存取權。 子系也會繼承在父管理群組上設定的原則。 

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何取得 Azure 資訊安全中心的全租用戶可見性。 如要深入了解資訊安全中心，請參閱下列文章：

> [!div class="nextstepaction"]
> [Azure 資訊安全中心的安全性健康情況監視](security-center-monitoring.md)

> [!div class="nextstepaction"]
> [在 Azure 資訊安全中心管理和回應安全性警示](security-center-managing-and-responding-alerts.md)

