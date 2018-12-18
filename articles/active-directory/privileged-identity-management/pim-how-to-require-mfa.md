---
title: 多重要素驗證 (MFA) 和 PIM - Azure | Microsoft Docs
description: 了解 Azure AD Privileged Identity Management (PIM) 如何驗證多重要素驗證 (MFA)。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.component: pim
ms.date: 08/31/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: cfa092e8aebe92192ecca8aec2721282e603cc5b
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669269"
---
# <a name="multi-factor-authentication-mfa-and-pim"></a>多重要素驗證 (MFA) 和 PIM

建議您針對所有系統管理員都要求進行多重要素驗證 (MFA)。 這可降低密碼遭入侵所導致的攻擊風險。

您可以要求使用者在登入時完成 MFA 挑戰。 您也可以要求使用者在啟用 Azure AD Privileged Identity Management (PIM) 中的角色時，完成 MFA 挑戰。 如此一來，如果使用者沒有在登入時完成 MFA 挑戰，PIM 就會提示他們執行這項作業。

> [!IMPORTANT]
> 目前 Azure MFA 只能與工作帳戶或學校帳戶搭配運作，而無法與 Microsoft 帳戶 (通常是用來登入 Skype、Xbox、Outlook.com 等 Microsoft 服務的個人帳戶) 搭配運作。 因此，任何使用者只要是使用 Microsoft 帳戶就無法成為合格的系統管理員，因為他們無法使用 MFA 來啟用其角色。 如果這些使用者需要繼續使用 Microsoft 帳戶來管理工作負載，請立即將他們提升為永久系統管理員。

## <a name="how-pim-validates-mfa"></a>PIM 如何驗證 MFA

在使用者啟用角色時有兩個選項可用來驗證 MFA。

最簡單的選項是讓啟用特殊權限角色的使用者依賴 Azure MFA。 若要這樣做，請先確認這些使用者已獲授權；必要時，針對 Azure MFA 進行註冊。 如需如何部署 Azure MFA 的詳細資訊，請參閱[部署雲端式 Azure Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)。 建議 (但非必要) 設定 Azure AD，在這些使用者登入時，強制為他們執行 MFA。 這是因為 MFA 檢查將是由 PIM 本身所進行。

或者，如果使用者驗證內部部署，您可以讓您的身分識別提供者負責進行 MFA。 例如，如果您已在存取 Azure AD 之前將 Active Directory 同盟服務設定為需要以智慧卡為基礎的驗證， [使用 Azure Multi-Factor Authentication 與 AD FS 保護雲端資源](../authentication/howto-mfa-adfs.md) 包含用來設定 AD FS 來將宣告傳送至 Azure AD 的相關指示。 當使用者嘗試啟動角色時，PIM 將會在其收到適當宣告時，接受系統已針對使用者驗證過 MFA。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中設定 Azure AD 目錄角色設定](pim-how-to-change-default-settings.md)
- [在 PIM 中設定 Azure 資源角色設定](pim-resource-roles-configure-role-settings.md)
