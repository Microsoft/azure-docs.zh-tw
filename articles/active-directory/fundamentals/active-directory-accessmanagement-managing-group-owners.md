---
title: 如何新增或移除 Azure Active Directory 群組擁有者 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 新增或移除群組擁有者。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: lizross
ms.custom: it-pro
ms.openlocfilehash: f546ea5b5f9288849334d27cd1721f0c22fb8806
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297770"
---
# <a name="how-to-add-or-remove-group-owners-in-azure-active-directory"></a>如何：在 Azure Active Directory 中新增或移除群組擁有者
Azure Active Directory (Azure AD) 群組是由群組擁有者所擁有及管理。 群組擁有者是由資源擁有者 (系統管理員) 指派來管理群組與其成員。 群組擁有者不需要是該群組的成員。 指派群組擁有者之後，只有資源擁有者可以新增或移除擁有者。

在某些案例中，身為系統管理員的您可能決定不指派群組擁有者。 在此案例中，您會成為群組擁有者。 此外，擁有者可以指派其他擁有者到其群組，除非您已在群組設定中限制這樣做。

## <a name="add-an-owner-to-a-group"></a>將擁有者新增至群組
使用 Azure AD 將額外的群組擁有者新增到群組。

### <a name="to-add-a-group-owner"></a>新增群組擁有者
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。

2. 選取 [Azure Active Directory] 並選取 [群組]，然後選取您要新增擁有者的群組 (針對此案例，是 _MDM policy - West_)。

3. 在 [MDM 原則- 西部概觀] 頁面上，選取 [擁有者]。

    ![已反白顯示 [擁有者] 選項的 [MDM 原則 - 西部概觀] 頁面](media/active-directory-accessmanagement-managing-group-owners/add-owners-option-overview-blade.png)

4. 在 [MDM 原則 - 西部 - 擁有者] 頁面上，選取 [新增擁有者]，然後搜尋並選取將成為新群組擁有者的使用者，接著選擇 [選取]。

    ![已反白顯示 [新增擁有者] 選項的 [MDM 原則 - 西部 - 擁有者] 頁面](media/active-directory-accessmanagement-managing-group-owners/add-owners-owners-blade.png)

    選取新擁有者之後，重新整理 [擁有者] 頁面即可看到該名稱已新增到擁有者清單。

## <a name="remove-an-owner-from-a-group"></a>移除群組中的擁有者
使用 Azure AD 從群組移除擁有者。

### <a name="to-remove-an-owner"></a>移除擁有者
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。

2. 選取 [Azure Active Directory] 並選取 [群組]，然後選取您要新增擁有者的群組 (針對此案例，是 _MDM policy - West_)。

3. 在 [MDM 原則- 西部概觀] 頁面上，選取 [擁有者]。

    ![已反白顯示 [擁有者] 選項的 [MDM 原則 - 西部概觀] 頁面](media/active-directory-accessmanagement-managing-group-owners/remove-owners-option-overview-blade.png)

4. 在 [MDM 原則 - 西部 - 擁有者] 頁面上，選取您要移除其群組擁有者身分的使用者，從使用者的資訊頁面選取 [移除]，然後選取 [是] 以確認您的決定。

    ![已反白顯示 [移除] 選項的使用者資訊頁面](media/active-directory-accessmanagement-managing-group-owners/remove-owner-info-blade.png)

    移除擁有者之後，返回 [擁有者] 頁面即可看到該名稱已從擁有者清單移除。

## <a name="next-steps"></a>後續步驟
- [使用 Azure Active Directory 群組來管理資源的存取權](active-directory-manage-groups.md)

- [設定群組設定的 Azure Active Directory Cmdlet](../users-groups-roles/groups-settings-cmdlets.md)

- [使用群組來指派對整合 SaaS 應用程式的存取權](../users-groups-roles/groups-saasapps.md)

- [整合內部部署身分識別與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)

- [設定群組設定的 Azure Active Directory Cmdlet](../users-groups-roles/groups-settings-v2-cmdlets.md)
