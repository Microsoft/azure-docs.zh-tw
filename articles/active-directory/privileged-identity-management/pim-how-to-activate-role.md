---
title: 在 PIM 中啟用我的 Azure AD 目錄角色 | Microsoft Docs
description: 如何在 Azure AD Privileged Identity Management (PIM) 中啟用 Azure AD 目錄角色。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.component: pim
ms.date: 08/27/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 59dab4651366c3ad6579e0da660baee0c653d1a3
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665997"
---
# <a name="activate-my-azure-ad-directory-roles-in-pim"></a>在 PIM 中啟用我的 Azure AD 目錄角色

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) 簡化了企業管理以特殊權限身分存取 Azure AD 中的資源和其他 Microsoft 線上服務 (如 Office 365 或 Microsoft Intune) 的方式。  

如果您已被設為符合系統管理角色資格，即表示您可以在需要執行特殊權限動作時，啟用該角色。 例如，如果您偶爾會管理 Office 365 功能，則貴組織的特殊權限角色管理員可能不會讓您成為永久全域管理員，因為該角色也會影響其他服務。 他們反而會讓您符合 Azure AD 角色 (例如「Exchange Online 管理員」) 的資格。 您可以在需要權限時，要求啟用該角色，然後您將會在預定的時段內擁有系統管理員控制權。

本文適用於需要在 PIM 中啟用 Azure AD 目錄角色的系統管理員。

## <a name="activate-a-role"></a>啟用角色

當您需要擔任某個 Azure AD 目錄角色時，您可以在 PIM 中使用 [我的角色] 導覽選項來要求啟用。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。 如需如何將 [PIM] 圖格新增至儀表板的資訊，請參閱[開始使用 PIM](pim-getting-started.md)。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [我的角色] 可查看符合資格的 Azure AD 目錄角色清單。

    ![Azure AD 目錄角色 - 我的角色](./media/pim-how-to-activate-role/directory-roles-my-roles.png)

1. 尋找您想要啟用的角色。

    ![Azure AD 目錄角色 - 我的角色清單](./media/pim-how-to-activate-role/directory-roles-my-roles-activate.png)

1. 按一下 [啟用]以開啟角色啟用詳細資料窗格。

1. 如果您的角色需要多重要素驗證 (MFA)，請按一下 [先驗證您的身分識別後再繼續]。 您只需在每個工作階段驗證一次。

    ![在啟用角色之前先以 MFA 驗證](./media/pim-how-to-activate-role/directory-roles-my-roles-mfa.png)

1. 按一下 [驗證我的身分識別]，並遵循指示來提供其他安全性驗證。

    ![其他安全性驗證](./media/pim-how-to-activate-role/additional-security-verification.png)

1. 按一下 [啟用] 以開啟啟用窗格。

    ![啟用窗格](./media/pim-how-to-activate-role/directory-roles-activate.png)

1. 如有必要，請指定自訂啟用開始時間。

1. 指定啟用持續時間。

1. 在 [啟用原因]方塊中輸入此啟用要求的原因。 有些角色會要求您提供問題票證號碼。

    ![已完成的啟用窗格](./media/pim-how-to-activate-role/directory-roles-activation-pane.png)

1. 按一下 [啟用]。

    如果角色不需要核准，它現在就會啟用，角色會出現在使用中角色的清單中。 如果[角色需要核准](./azure-ad-pim-approval-workflow.md)才能啟用，通知會出現在瀏覽器右上角，通知您要求正在等待核准。

    ![要求擱置通知](./media/pim-how-to-activate-role/directory-roles-activate-notification.png)

## <a name="view-the-status-of-your-requests"></a>檢視要求狀態

您可以檢視要啟用的擱置要求狀態。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [我的要求] 以查看要求清單。

    ![Azure AD 目錄角色 - 我的要求](./media/pim-how-to-activate-role/directory-roles-my-requests.png)

## <a name="use-a-role-immediately-after-activation"></a>啟用後可立即使用角色

因為快取的關係，啟用不會立即在 Azure 入口網站中生效，必須重新整理畫面。 如果您需要降低啟用角色之後的延遲機率，您可以使用入口網站中的**應用程式存取**頁面。 從此頁面存取的應用程式會立即檢查新的角色指派。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [應用程式存取] 頁面。

    ![PIM 應用程式存取](./media/pim-how-to-activate-role/pim-application-access.png)

1. 按一下 [Azure Active Directory]，即可重新開啟入口網站的 [所有使用者] 頁面。

    當您按一下此連結後，即可強制重新整理，並檢查新的 Azure AD 角色指派。

## <a name="deactivate-a-role"></a>停用角色

角色一經啟用，就會在達到時間限制 (合格的持續時間) 時自動停用。

如果您提早完成系統管理員工作，也可以在 Azure AD Privileged Identity Management 中手動停用角色。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [我的角色]。

1. 按一下 [使用中的角色]，以查看使用中的角色清單。

1. 尋找您使用完畢的角色，然後按一下 [停用]。

## <a name="cancel-a-pending-request"></a>取消擱置要求

如果您不需要啟用需要核准的角色，您可以隨時取消擱置要求。

1. 開啟 Azure AD Privileged Identity Management。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [我的要求]。

1. 針對您想要取消的角色，按一下 [取消] 按鈕。

    當您按一下 [取消] 時，要求將會取消。 若要再次啟用角色，您必須提交新的啟用要求。

   ![取消擱置要求](./media/pim-how-to-activate-role/directory-role-cancel.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中啟用我的 Azure 資源角色](pim-resource-roles-activate-your-roles.md)
