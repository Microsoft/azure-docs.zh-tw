---
title: 如何在 Azure Active Directory 中新增或刪除使用者 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 新增使用者或刪除現有的使用者。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: conceptual
ms.date: 09/04/2018
ms.author: lizross
ms.reviewer: jeffsta
ms.custom: it-pro
ms.openlocfilehash: 782363144a6b1dd87aff515c38588b6ce70b61bd
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295099"
---
# <a name="how-to-add-or-delete-users-using-azure-active-directory"></a>如何：使用 Azure Active Directory 新增或刪除使用者
使用 Azure AD 從您的 Azure Active Directory (Azure AD) 租用戶新增使用者或刪除現有的使用者。

## <a name="add-a-new-user"></a>新增使用者
您可以使用 Azure Active Directory 建立新使用者。

### <a name="to-add-a-new-user"></a>新增使用者
1. 以目錄的全域系統管理員或使用者系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [Azure Active Directory] 並選取 [使用者]，然後選取 [新增使用者]。

    ![使用者 - 已反白顯示 [新增使用者] 的 [所有使用者] 頁面](media/add-users-azure-active-directory/new-user-all-users-blade.png)

3. 在 [使用者] 頁面上，填妥必要資訊。

    ![新增使用者，包含使用者資訊的 [使用者] 頁面](media/add-users-azure-active-directory/new-user-user-blade.png)

    - **名稱 (必要)。** 新使用者的姓氏與名字。 例如，Mary Parker。

    - **使用者名稱 (必要)。** 新使用者的使用者名稱。 例如：mary@contoso.com。 
    
        使用者名稱的網域部分必須使用初始預設網域名稱 (<_yourdomainname_>.onmicrosoft.com)，或自訂網域名稱，例如 contoso.com。 如需如何建立自訂網域名稱的詳細資訊，請參閱[如何將自訂網域名稱新增到 Azure Active Directory](add-custom-domain.md)。

    - **設定檔。** (選擇性) 您可以新增更多有關使用者的資訊。 您也可以稍後新增使用者資訊。 如需新增使用者資訊的詳細資訊，請參閱[如何新增或變更使用者設定檔資訊](active-directory-users-profile-azure-portal.md)。

    - **群組。** (選擇性) 您可以將使用者新增到一或多個現有的群組。 您也可以稍後再將使用者新增到群組。 如需有關將使用者新增到群組的詳細資訊，請參閱[如何建立基本的群組及新增成員](active-directory-groups-create-azure-portal.md)。

    - **目錄角色。** (選擇性) 您可以將使用者新增到目錄角色。 您可以將使用者指派為 Azure AD 中的全域系統管理員，或指派給一或多個其他系統管理員角色。 如需有關指派角色的詳細資訊，請參閱[如何將角色指派給使用者](active-directory-users-assign-role-azure-portal.md)。

4. 複製在 [密碼] 方塊中提供的自動產生密碼。 您必須將此密碼提供給使用者，讓其進行初始登入程序。

5. 選取 [建立]。

    使用者已建立並新增至您的 Azure AD 租用戶。

## <a name="add-a-new-user-within-a-hybrid-environment"></a>在混合式環境內新增使用者
若您的環境同時具有 Azure Active Directory (雲端) 與 Windows Server Active Directory (內部部署)，您可以透過同步現有的使用者帳戶資料來新增使用者。 如需有關混合式環境與使用者的詳細資訊，請參閱[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)。

## <a name="delete-a-user"></a>刪除使用者
您可以使用 Azure Active Directory 刪除現有的使用者。

### <a name="to-delete-a-user"></a>刪除使用者
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [Azure Active Directory] 並選取 [使用者]，然後搜尋並選取您要從您的 Azure AD 租用戶刪除的使用者。 例如，_Mary Parker_。

3. 選取 [刪除使用者]。

    ![使用者 - 已反白顯示 [刪除使用者] 的 [所有使用者] 頁面](media/add-users-azure-active-directory/delete-user-all-users-blade.png)

    使用者會被刪除，而且再也不會出現在 [使用者 - 所有使用者] 頁面上。 在接下來的 30 天，您可以在 [刪除的使用者] 頁面上看到該使用者，而且在此期間內可將該使用者還原。 如需有關還原使用者的詳細資訊，請參閱[如何還原或永久移除最近刪除的使用者](active-directory-users-restore.md)。

    >[!Note]
    >您必須使用 Windows Server Active Directory 來更新授權來源是 Windows Server Active Directory 之使用者的身分識別、連絡資訊或作業資訊。 完成更新之後，您必須等候下一次同步循環完成才能看到您所做的變更。

## <a name="next-steps"></a>後續步驟
新增使用者之後，您可以執行下列基本程序：

- [新增或變更設定檔資訊](active-directory-users-profile-azure-portal.md)

- [將角色指派給使用者](active-directory-users-assign-role-azure-portal.md)

- [建立基本群組並新增成員](active-directory-groups-create-azure-portal.md)

- [使用動態群組與使用者](../users-groups-roles/groups-create-rule.md)

或者，您可以執行其他使用者管理工作，例如[從另一個目錄新增來賓使用者](../b2b/what-is-b2b.md)或[還原已刪除的使用者](active-directory-users-restore.md)。 如需有關其他可用動作的詳細資訊，請參閱 [Azure Active Directory 使用者管理文件](../users-groups-roles/index.yml)。