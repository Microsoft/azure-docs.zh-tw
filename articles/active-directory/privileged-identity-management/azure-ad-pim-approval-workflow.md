---
title: 在 PIM 中核准或拒絕 Azure AD 目錄角色的要求 | Microsoft Docs
description: 如何在 Azure AD Privileged Identity Management (PIM) 中核准或拒絕 Azure AD 目錄角色的要求。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: pim
ms.date: 08/29/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 9402824540f965cb89aa00791d093bd87712a89a
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665837"
---
# <a name="approve-or-deny-requests-for-azure-ad-directory-roles-in-pim"></a>在 PIM 中核准或拒絕 Azure AD 目錄角色的要求

透過 Azure AD Privileged Identity Management (PIM)，您可以將角色設定為需要核准才能啟用，並選擇一或多個使用者或群組作為委派核准者。 依照本文中的步驟核准或拒絕 Azure AD 目錄角色的要求。

## <a name="view-pending-requests"></a>檢視擱置的要求

身為委派核准者，當有 Azure AD 目錄角色要求正等待您的核准時，您會收到電子郵件通知。 您可以在 PIM 中檢視這些擱置的要求。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [核准要求]。

    ![PIM Azure AD 目錄角色 - 角色](./media/azure-ad-pim-approval-workflow/pim-directory-roles-approve-requests.png)

    您將看到一份等待您核准的要求清單。

## <a name="approve-requests"></a>核准要求

1. 選取您要核准的要求，然後按一下 [核准]，以開啟 [核准選取要求] 窗格。

    ![PIM 核准要求清單](./media/azure-ad-pim-approval-workflow/pim-approve-requests-list.png)

1. 在 [核准原因] 方塊中輸入原因。

    ![PIM 核准所選要求](./media/azure-ad-pim-approval-workflow/pim-approve-selected-requests.png)

1. 按一下 [核准]。

    狀態符號會隨著您的核准而更新。

    ![PIM 核准所選要求](./media/azure-ad-pim-approval-workflow/pim-approve-status.png)

## <a name="deny-requests"></a>拒絕要求

1. 選取您要拒絕的要求，然後按一下 [拒絕]，以開啟 [拒絕選取要求] 窗格。

    ![PIM 核准要求清單](./media/azure-ad-pim-approval-workflow/pim-deny-requests-list.png)

1. 在 [拒絕原因] 方塊中輸入原因。

    ![PIM 拒絕所選要求](./media/azure-ad-pim-approval-workflow/pim-deny-selected-requests.png)

1. 按一下 [拒絕]。

    狀態符號會隨著您的拒絕而更新。

## <a name="next-steps"></a>後續步驟

- [PIM 中的電子郵件通知](pim-email-notifications.md)
- [在 PIM 中核准或拒絕 Azure 資源角色的要求](pim-resource-roles-approval-workflow.md)
