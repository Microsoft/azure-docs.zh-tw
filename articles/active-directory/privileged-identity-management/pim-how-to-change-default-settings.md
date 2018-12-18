---
title: 在 PIM 中設定 Azure AD 目錄角色設定 | Microsoft Docs
description: 如何在 Azure AD Privileged Identity Management (PIM) 中設定 Azure AD 目錄角色設定。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.component: pim
ms.date: 09/21/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 77a57afda3859de4a246c15d3b5c0b4ba8e2a8c8
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038987"
---
# <a name="configure-azure-ad-directory-role-settings-in-pim"></a>在 PIM 中設定 Azure AD 目錄角色設定

特殊權限角色管理員可以自訂其組織中的 Azure AD Privileged Identity Management (PIM)，包括變更啟用合格角色指派之使用者的體驗。

## <a name="open-role-settings"></a>開啟角色設定

請遵循下列步驟來開啟 Azure AD 目錄角色的設定。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 按一下 [Azure AD 目錄角色]。

1. 按一下 [設定] 。

    ![Azure AD 目錄角色 - 設定](./media/pim-how-to-change-default-settings/pim-directory-roles-settings.png)

1. 按一下 [角色]。

1. 按一下您想要設定其設定的角色。

    ![Azure AD 目錄角色 - 設定角色](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-role.png)

    在每個角色的設定頁面上，有許多您可以設定的設定。 這些設定只會影響身為**合格**指派 (而不是**永久**指派) 的使用者。

## <a name="activations"></a>啟用

使用 [啟用] 滑桿，設定角色在到期前維持作用中狀態的最長時間 (以小時為單位)。 此值可介於 1 到 72 小時。

## <a name="notifications"></a>通知

使用 [通知] 的切換，指定是否要讓系統傳送電子郵件給系統管理員來確認他們已啟用角色。 這對偵測未經授權或不合法的啟用而言相當有用。

## <a name="incidentrequest-ticket"></a>事件/要求票證

使用 [事件/要求票證] 的切換，指定是否要要求合格系統管理員在啟用其角色時包含票證號碼。 當您執行角色存取稽核時，這會相當有用。

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

使用 [Multi-Factor Authentication] 的切換，指定是否要要求使用者在啟用其角色之前，先以 MFA 驗證其身分識別。 他們只需在每一工作階段進行一次驗證，而不需在每次啟用角色時都進行驗證。 啟用 MFA 時，需要記住兩個秘訣：

* 使用 Microsoft 帳戶作為電子郵件地址 (通常是 @outlook.com，但不一定) 的使用者無法註冊 Azure MFA。 如果您想要將角色指派給使用 Microsoft 帳戶的使用者，您應該將他們設為永久系統管理員，或是停用該角色的 MFA。
* 您無法將 Azure AD 和 Office365 中高特殊權限角色的 MFA 停用。 這是一項安全功能，因為這些角色應該嚴密地受到保護：  
  
  * 計費管理員
  * 雲端應用程式系統管理員
  * 規範管理員
  * 條件式存取系統管理員
  * CRM 服務管理員
  * 客戶 LockBox 存取核准者
  * 目錄撰寫者
  * Exchange 系統管理員
  * 全域管理員
  * 資訊保護管理員
  * Intune 服務管理員
  * Power BI 服務管理員
  * 特殊權限角色管理員
  * 安全性系統管理員
  * SharePoint 服務管理員
  * 商務用 Skype 管理員
  * 使用者管理員

如需詳細資訊，請參閱[多重要素驗證 (MFA) 和 PIM](pim-how-to-require-mfa.md)。

## <a name="require-approval"></a>需要核准

如果您想要在啟用角色前先經過核准，請遵循下列步驟。

1. 將 [需要核准] 切換為 [啟用]。 展開的窗格會有選取核准者的選項。

    ![Azure AD 目錄角色 - 設定 - 需要核准](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-require-approval.png)

    如果您**尚未**指定任何核准者，特殊權限角色管理員就會變成預設核准者。 需要有特殊權限角色管理員，才能核准此角色的**所有**啟用要求。

1. 若要指定核准者，請按一下 [選取核准者]。

    ![Azure AD 目錄角色 - 設定 - 需要核准](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-require-approval-select-approvers.png)

1. 選取一個或多個核准者，然後按一下 [選取]。 您可以選取使用者或群組。 建議至少選取 2 個核准者。 不允許自我核准。

    您的選項將出現在所選取的核准者清單中。

1. 指定所有角色設定後，按一下 [儲存] 以儲存變更。


<!--PLACEHOLDER: Need an explanation of what the temporary Global Administrator setting is for.-->

## <a name="next-steps"></a>後續步驟

- [在 PIM 中指派 Azure AD 目錄角色](pim-how-to-add-role-to-user.md)
- [在 PIM 中設定 Azure AD 目錄角色的安全性警示](pim-how-to-configure-security-alerts.md)
