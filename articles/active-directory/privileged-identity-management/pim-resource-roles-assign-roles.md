---
title: 在 PIM 中指派 Azure 資源角色 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中指派 Azure 資源角色。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: pim
ms.date: 08/30/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: daf83baf5f5c3a2b7bbfe39e043e2b8411d6dd06
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465770"
---
# <a name="assign-azure-resource-roles-in-pim"></a>在 PIM 中指派 Azure 資源角色

Azure AD PIM 可以管理內建 Azure 資源角色及自訂角色，包括 (但不限於)：

- 擁有者
- 使用者存取系統管理員
- 參與者
- 安全性系統管理員
- 安全性管理員以及其他

>[!NOTE]
指派至擁有者或使用者存取管理員角色的使用者或群組成員，以及在 Azure AD 中啟用訂閱管理的全域管理員，為資源管理員。 這些系統管理員可指派角色、設定角色設定，並使用適用於 Azure 資源的 PIM 進行檢閱存取。 請檢視[Azure 資源的內建角色](../../role-based-access-control/built-in-roles.md)清單。

## <a name="assign-a-role"></a>指派角色

請遵循下列步驟來讓使用者有資格獲派 Azure 資源角色。

1. 以屬於[特殊權限角色管理員](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator)角色一員的使用者身分登入 [Azure 入口網站](https://portal.azure.com/)。

    如需如何授與其他系統管理員存取權以管理 PIM 的資訊，請參閱[授與其他系統管理員存取權以管理 PIM](pim-how-to-give-access-to-pim.md)。

1. 開啟 **Azure AD Privileged Identity Management**。

    如果您尚未在 Azure 入口網站中起始 PIM，請移至[開始使用 PIM](pim-getting-started.md)。

1. 按一下 [Azure 資源]。

1. 使用 [資源篩選] 來篩選管理的資源清單。

    ![要管理的 Azure 資源清單](./media/pim-resource-roles-assign-roles/resources-list.png)

1. 按一下您想要管理的資源，例如訂用帳戶或管理群組。

1. 按一下 [管理] 下方的 [角色]，以查看 Azure 資源角色的清單。

    ![Azure 資源角色](./media/pim-resource-roles-assign-roles/resources-roles.png)

1. 按一下 [新增成員] 以開啟 [新增指派] 窗格。

1. 按一下 [選取角色] 以開啟 [選取角色] 窗格。

    ![新增指派窗格](./media/pim-resource-roles-assign-roles/resources-select-role.png)

1. 按一下您想要指派的角色，然後按一下 [選取]。

    [選取成員或群組] 窗格會隨即開啟。

1. 按一下您想要指派給角色的成員或群組，然後按一下 [選取]。

    ![選取成員或群組窗格](./media/pim-resource-roles-assign-roles/resources-select-member-or-group.png)

    [成員資格設定] 窗格會即開啟。

1. 在 [指派類型] 清單中選取 [合格] 或 [有效]。

    ![成員資格設定窗格](./media/pim-resource-roles-assign-roles/resources-membership-settings-type.png)

    適用於 Azure 資源的 PIM 提供兩種不同的指派類型：

    - **合格**的指派會要求角色成員先執行某個動作才能使用此角色。 動作可能包含執行多重要素驗證 (MFA) 檢查、提供業務理由，或是向指定的核准者要求核准。

    - **有效**的指派不要求成員先執行某個動作才能使用此角色。 指派為有效的成員隨時具有指派給角色的權限。

1. 如果指派應為永久性 (永久合格或永久指派)，請選取 [永久] 核取方塊。

    根據不同角色設定，核取方塊可能不會出現或可能設為不可修改。

1. 若要指定特定指派的持續時間，請清除核取方塊，並修改開始和/或結束日期和時間方塊。

    ![成員資格設定 - 日期和時間](./media/pim-resource-roles-assign-roles/resources-membership-settings-date.png)

1. 完成後，請按一下 [完成]。

    ![新增指派 - 新增](./media/pim-resource-roles-assign-roles/resources-new-assignment-add.png)

1. 若要建立新的角色指派，請按一下 [新增]。 狀態通知會隨即顯示。

    ![新增指派 - 通知](./media/pim-resource-roles-assign-roles/resources-new-assignment-notification.png)

## <a name="update-or-remove-an-existing-role-assignment"></a>更新或移除現有角色指派

請遵循下列步驟來更新或移除現有角色指派。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure 資源]。

1. 按一下您想要管理的資源，例如訂用帳戶或管理群組。

1. 按一下 [管理] 下方的 [角色]，以查看 Azure 資源角色的清單。

    ![Azure 資源角色 - 選取角色](./media/pim-resource-roles-assign-roles/resources-update-select-role.png)

1. 按一下您想要更新或移除的角色。

1. 在 [合格角色] 或 [有效角色] 索引標籤上尋找角色指派。

    ![更新或移除角色指派](./media/pim-resource-roles-assign-roles/resources-update-remove.png)

1. 按一下 [更新] 或 [移除] 來更新或移除角色指派。

    如需有關延長角色指派的相關資訊，請參閱[在 PIM 中延長或更新 Azure 資源角色](pim-resource-roles-renew-extend.md)。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中延長或更新 Azure 資源角色](pim-resource-roles-renew-extend.md)
- [在 PIM 中設定 Azure 資源角色設定](pim-resource-roles-configure-role-settings.md)
- [在 PIM 中指派 Azure AD 目錄角色](pim-how-to-add-role-to-user.md)
