---
title: 授與其他系統管理員存取權以管理 PIM - Azure | Microsoft Docs
description: 了解如何授與其他系統管理員存取權以管理 Azure AD Privileged Identity Management (PIM)。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.component: pim
ms.date: 08/29/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 9d5fce5a80ac1f281fdbe6afe7f9a97816807ccc
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287082"
---
# <a name="grant-access-to-other-administrators-to-manage-pim"></a>授與其他系統管理員存取權以管理 PIM

為組織啟用 Azure AD Privileged Identity Management (PIM) 的全域系統管理員，會自動取得角色指派及 PIM 的存取權。 然而依照預設，其他任何人皆不會取得寫入存取權，包括其他全域系統管理員在內。 其他全域系統管理員、安全性系統管理員及安全性讀取者具有 PIM 的唯讀存取權。 若要授與對 PIM 的存取權，第一位使用者可以將其他使用者指派給**特殊權限角色管理員**角色。

> [!NOTE]
> 管理 PIM 需要 Azure MFA。 因為 Microsoft 帳戶無法註冊 Azure MFA，所以使用 Microsoft 帳戶登入的使用者無法存取 PIM。

請確保特殊權限角色管理員角色中永遠至少有兩位使用者，以防一位使用者遭到鎖定或他們的帳戶遭刪除。

## <a name="grant-access-to-manage-pim"></a>授予管理 PIM 的存取權

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [角色]。

    ![PIM Azure AD 目錄角色 - 角色](./media/pim-how-to-give-access-to-pim/pim-directory-roles-roles.png)

1. 按一下 [特殊權限角色管理員] 角色開啟成員頁面。

    ![特殊權限角色管理員 - 成員](./media/pim-how-to-give-access-to-pim/pim-pra-members.png)

1. 按一下 [新增成員] 以開啟 [新增受控成員] 窗格。

1. 按一下 [選取成員] 以開啟 [選取成員] 窗格。

    ![特殊權限角色管理員 - 選取成員](./media/pim-how-to-give-access-to-pim/pim-pra-select-members.png)

1. 選取成員，然後按一下 [選取]。

1. 按一下 [確定]，使成員符合**特殊權限角色管理員**角色的資格。

    每當您指派新角色給 PIM 中的某位使用者時，系統都會自動將這些人設定成符合啟用該角色的**資格**。

1. 若要使成員永久符合資格，請在 [特殊權限角色管理員] 成員清單中按一下該使用者。

1. 按一下 [更多]，然後按一下 [設為永久] 進行永久指派。

    ![特殊權限角色管理員 - 永久符合資格](./media/pim-how-to-give-access-to-pim/pim-pra-make-permanent.png)

1. 將[開始使用 PIM](pim-getting-started.md) 的連結傳送給使用者。

## <a name="remove-access-to-manage-pim"></a>移除管理 PIM 的存取權

將某人自特殊權限角色管理員角色移除之前，請確保該角色至少仍有兩位指派的使用者。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [角色]。

1. 按一下 [特殊權限角色管理員] 角色開啟成員頁面。

1. 勾選您想要移除的使用者，然後按一下 [移除成員]。

    ![特殊權限角色管理員 - 移除成員](./media/pim-how-to-give-access-to-pim/pim-pra-remove-member.png)

1. 在詢問您是否要從角色移除成員的訊息中，按一下 [是]。

## <a name="next-steps"></a>後續步驟

- [開始使用 PIM](pim-getting-started.md)