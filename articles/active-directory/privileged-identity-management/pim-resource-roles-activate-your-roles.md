---
title: 在 PIM 中啟用我的 Azure 資源角色 | Microsoft Docs
description: 了解如何在 Azure AD Privileged Identity Management (PIM) 中啟用 Azure 資源角色。
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
ms.date: 08/31/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 59bce2c61db5838bb21a29757d4e354311ecffd5
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666242"
---
# <a name="activate-my-azure-resource-roles-in-pim"></a>在 PIM 中啟用我的 Azure 資源角色

Azure 資源的合格角色成員可使用 Azure AD Privileged Identity Management (PIM)，排程未來啟用的日期與時間。 他們也可以在最大範圍內選取特定的啟用期間 (由管理員設定)。

本文適用於需要在 PIM 中啟用其 Azure 資源角色的成員。

## <a name="activate-a-role"></a>啟用角色

當您需要擔任某個 Azure 資源角色時，您可以在 PIM 中使用 [我的角色] 導覽選項來要求啟用。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。 如需如何將 [PIM] 圖格新增至儀表板的資訊，請參閱[開始使用 PIM](pim-getting-started.md)。

1. 按一下 [我的角色] 可查看符合資格的 Azure AD 目錄角色與 AD 資源角色清單。

    ![Azure AD 目錄角色與 Azure 資源角色 - 我的角色](./media/pim-resource-roles-activate-your-roles/resources-my-roles.png)

1. 在 [Azure 資源角色] 清單中，尋找您要啟用的角色。

    ![Azure 資源角色 - 我的角色清單](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate.png)

1. 按一下 [啟用] 以開啟 [啟用] 窗格。

1. 如果您的角色需要多重要素驗證 (MFA)，請按一下 [先驗證您的身分識別後再繼續]。 您只需在每個工作階段驗證一次。

    ![在啟用角色之前先以 MFA 驗證](./media/pim-resource-roles-activate-your-roles/resources-my-roles-mfa.png)

1. 按一下 [驗證我的身分識別]，並遵循指示來提供其他安全性驗證。

    ![其他安全性驗證](./media/pim-resource-roles-activate-your-roles/resources-mfa-enter-code.png)

1. 如果您想要指定縮小的範圍，請按一下 [範圍] 開啟 [資源] 篩選窗格。

    最佳做法是僅要求存取您需要的資源。 在 [資源] 篩選窗格中，您可以指定資源群組或您需要存取的群組。

    ![啟用 - 資源篩選](./media/pim-resource-roles-activate-your-roles/resources-my-roles-resource-filter.png)

1. 如有必要，請指定自訂啟用開始時間。 成員會在選取的時間後啟用。

1. 在 [原因] 方塊中輸入此啟用要求的原因。

    ![已完成的啟用窗格](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-done.png)

1. 按一下 [啟用]。

    如果角色不需要核准，它現在就會啟用，角色會出現在使用中角色的清單中。 如果[角色需要核准](pim-resource-roles-approval-workflow.md)才能啟用，通知會出現在瀏覽器右上角，通知您要求正在等待核准。

    ![要求擱置通知](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-notification.png)

## <a name="view-the-status-of-your-requests"></a>檢視要求狀態

您可以檢視要啟用的擱置要求狀態。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [我的要求] 可查看您的 Azure AD 目錄角色與 AD 資源角色要求清單。

    ![Azure AD 目錄角色與 Azure 資源角色 - 我的要求](./media/pim-resource-roles-activate-your-roles/resources-my-requests.png)

1. 捲動至右側可檢視 [要求狀態] 欄。

## <a name="use-a-role-immediately-after-activation"></a>啟用後可立即使用角色

因為快取的關係，啟用不會立即在 Azure 入口網站中生效，必須重新整理畫面。 如果您需要降低啟用角色之後的延遲機率，您可以使用入口網站中的**應用程式存取**頁面。 從此頁面存取的應用程式會立即檢查新的角色指派。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [應用程式存取] 頁面。

    ![PIM 應用程式存取 - 螢幕擷取畫面](./media/pim-resource-roles-activate-your-roles/pim-application-access.png)

1. 按一下 [Azure 資源]，即可重新開啟入口網站的 [所有資源] 頁面。

    當您按一下此連結後，即可強制重新整理，並檢查新的 Azure 資源角色指派。

## <a name="cancel-a-pending-request"></a>取消擱置要求

如果您不需要啟用需要核准的角色，您可以隨時取消擱置要求。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [我的要求]。

1. 針對您想要取消的角色，按一下 [取消] 連結。

    當您按一下 [取消] 時，要求將會取消。 若要再次啟用角色，您必須提交新的啟用要求。

   ![取消擱置要求](./media/pim-resource-roles-activate-your-roles/resources-my-requests-cancel.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中延長或更新 Azure 資源角色](pim-resource-roles-renew-extend.md)
- [在 PIM 中啟用我的 Azure AD 目錄角色](pim-how-to-activate-role.md)