---
title: 在 PIM 中設定 Azure 資源角色設定 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中設定 Azure 資源角色設定。
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
ms.openlocfilehash: a4aecd276df8e5453f0c35d6290bbe8a8d156ffa
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669358"
---
# <a name="configure-azure-resource-role-settings-in-pim"></a>在 PIM 中設定 Azure 資源角色設定

當您設定 Azure 資源角色設定時，會定義預設設定以套用至 Azure AD Privileged Identity Management (PIM) 中的 Azure 資源角色指派。 請使用下列程序來設定核准工作流程，並指定可以核准或拒絕要求的人員。

## <a name="open-role-settings"></a>開啟角色設定

請遵循下列步驟來開啟 Azure 資源角色的設定。

1. 以屬於[特殊權限角色管理員](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator)角色一員的使用者身分登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure 資源]。

1. 按一下您想要管理的資源，例如訂用帳戶或管理群組。

    ![要管理的 Azure 資源清單](./media/pim-resource-roles-configure-role-settings/resources-list.png)

1. 按一下 [角色設定]。

    ![角色設定](./media/pim-resource-roles-configure-role-settings/resources-role-settings.png)

1. 按一下您想要設定其設定的角色。

    ![角色設定詳細資料](./media/pim-resource-roles-configure-role-settings/resources-role-setting-details.png)

1. 按一下 [編輯] 以開啟 [角色設定] 窗格。

    ![編輯角色設定](./media/pim-resource-roles-configure-role-settings/resources-role-settings-edit.png)

    在每個角色的 [角色設定] 窗格上，有許多您可以設定的設定。

## <a name="assignment-duration"></a>指派持續時間

當您進行角色的設定時，每個指派類型 (合格和有效) 都有兩個指派持續時間選項可選擇。 將成員指派給 PIM 中的角色後，下列選項才會變成預設最長持續時間。

您可以從下列**合格**指派持續時間選項中選擇一個：

| | |
| --- | --- |
| **允許永久合格的指派** | 資源管理員可以指派永久的合格成員資格。 |
| **合格指派的有效期限** | 資源管理員可以要求所有合格指派有指定的開始和結束日期。 |

此外，您可以從下列**有效**指派持續時間選項中選擇一個：

| | |
| --- | --- |
| **允許永久有效的指派** | 資源管理員可以指派永久的有效成員資格。 |
| **有效指派的有效期限** | 資源管理員可以要求所有有效指派有指定的開始和結束日期。 |

> [!NOTE] 
> 資源管理員可以更新所有具有指定結束日期的指派。 此外，成員可以將自助服務要求初始化，以[延長或更新角色指派](pim-resource-roles-renew-extend.md)。

## <a name="require-multi-factor-authentication"></a>需要多重要素驗證

PIM 針對兩個不同的案例提供選擇性強制 Azure Multi-Factor Authentication (MFA)。

### <a name="require-multi-factor-authentication-on-active-assignment"></a>要求為有效指派進行 Multi-Factor Authentication

在某些情況下，您可能想要將成員短期 (例如一天) 指派給某個角色。 在此情況下，他們不需要指派的成員來要求啟用。 在此情況下，PIM 無法在成員使用其角色指派時強制執行 MFA，因為從指派他們的那一刻起，他們就已經具備有效的角色。

若要確保執行指派的資源管理員身分屬實，您可以藉由核取 [進行有效指派時需要 Multi-Factor Authentication] 方塊，來對有效指派強制執行 MFA。

### <a name="require-multi-factor-authentication-on-activation"></a>啟用時需要 Multi-Factor Authentication

您可以要求角色的合格成員先執行 MFA 才可啟用。 此程序確保要求啟用的使用者是表明具有合理確定性的人員。 在使用者帳戶可能受到危害的情況下，強制執行這個選項可保護重要資源。

若要要求合格成員在啟用之前執行 MFA，請核取 [啟用時需要 Multi-Factor Authentication] 方塊。

如需詳細資訊，請參閱[多重要素驗證 (MFA) 和 PIM](pim-how-to-require-mfa.md)。

## <a name="activation-maximum-duration"></a>啟用持續時間上限

使用 [啟用持續時間上限] 滑桿，可設定角色在到期前維持作用中狀態的最長時間 (以小時為單位)。 此值可介於 1 到 24 小時。

## <a name="require-justification"></a>需要理由

您可以要求成員在進行有效指派時或啟用時提供理由。 若要要求提供理由，請核取 [進行有效指派時需要提供理由] 方塊，或核取 [啟用時需要提供理由] 方塊。

## <a name="require-approval-to-activate"></a>需要核准才可啟用

如果您想要在啟用角色前先經過核准，請遵循下列步驟。

1. 請核取 [需要核准才可啟用] 核取方塊。

1. 按一下 [選取核准者] 以開啟 [選取成員或群組] 窗格。

    ![選取成員或群組](./media/pim-resource-roles-configure-role-settings/resources-role-settings-select-approvers.png)

1. 選取至少一個成員或群組後，按一下 [選取]。 您可以新增任何成員和群組的組合。 您必須至少選取一個核准者。 沒有任何預設核准者。

    您的選項將出現在所選取的核准者清單中。

1. 指定所有角色設定後，按一下 [更新] 以儲存變更。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中指派 Azure 資源角色](pim-resource-roles-assign-roles.md)
- [在 PIM 中設定 Azure 資源角色的安全性警示](pim-resource-roles-configure-alerts.md)
