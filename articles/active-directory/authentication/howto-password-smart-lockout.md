---
title: 使用 Azure AD 智慧鎖定防止暴力攻擊
description: Azure Active Directory 智慧鎖定有助於保護組織免於遭受嘗試猜測密碼的暴力攻擊
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: rogoya
ms.openlocfilehash: 9ea91f70a72b812803a20244bb4445b76b133b0c
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46296154"
---
# <a name="azure-active-directory-smart-lockout"></a>Azure Active Directory 智慧鎖定

智慧鎖定會使用雲端智慧，鎖定嘗試猜測使用者密碼或使用暴力方法登入的不良執行者。 該智慧可以辨識來自有效使用者的登入，並將其視為不同於攻擊者和其他不明來源所進行的登入。 智慧鎖定可在鎖定攻擊者的同時，讓您的使用者仍可繼續存取其帳戶並保有生產力。

根據預設，智慧鎖定會在 10 次失敗嘗試後鎖定帳戶，使其無法嘗試登入一分鐘。 在每次後續登入嘗試失敗之後，帳戶會再次鎖定，先鎖定一分鐘，後續嘗試鎖定時間會更長。

任何 Azure AD 客戶只要採用兼具適當安全性和可用性的預設設定，智慧鎖定就一律會啟用。 要使用組織的特定值自訂智慧鎖定設定，您的使用者必須具有 Azure AD Basic 或更高的授權。

智慧鎖定可與混合式部署整合，使用密碼雜湊同步或傳遞驗證來保護內部部署 Active Directory 帳戶不被攻擊者鎖定。 藉由適當地在 Azure AD 中設定智慧鎖定原則，攻擊將會在到達內部部署 Active Directory 之前遭到篩除。

使用[傳遞驗證](../hybrid/how-to-connect-pta.md)時，您必須確定：

   * Azure AD 的鎖定閾值「小於」Active Directory 帳戶的鎖定閾值。 請適當設定這些值，使 Active Directory 帳戶的鎖定閾值比 Azure AD 鎖定閾值至少長兩到三倍。 
   * Azure AD 的鎖定持續期間 **(以秒表示)** 比 Active Directory 的「下列時間過後重設帳戶鎖定計數器」持續期間 **(分鐘)**「還長」。

> [!IMPORTANT]
> 目前，如果使用者的雲端帳戶已被智慧鎖定功能鎖定，則系統管理員無法將其解除鎖定。 系統管理員必須等待鎖定持續期間結束。

## <a name="verify-on-premises-account-lockout-policy"></a>驗證內部部署帳戶鎖定原則

請使用下列指示來驗證您的內部部署 Active Directory 帳戶鎖定原則：

1. 開啟群組原則管理工具。
2. 編輯包含組織帳戶鎖定原則的群組原則，例如**預設網域原則**。
3. 瀏覽到 **[電腦設定]** > **[原則]** > **[Windows 設定]** > **[安全性設定]** > **[帳戶原則]** > **[帳戶鎖定原則]**。
4. 確認 [帳戶鎖定閾值] 和 [下列時間過後重設帳戶鎖定計數器] 的值。

![使用群組原則物件修改內部部署 Active Directory 帳戶鎖定原則](./media/howto-password-smart-lockout/active-directory-on-premises-account-lockout-policy.png)

## <a name="manage-azure-ad-smart-lockout-values"></a>管理 Azure AD 智慧鎖定值

根據您的組織需求，智慧鎖定值可能需要進行自訂。 要使用組織的特定值自訂智慧鎖定設定，您的使用者必須具有 Azure AD Basic 或更高的授權。

若要檢查或修改組織的智慧鎖定值，請使用下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com) 並按一下 [Azure Active Directory]，然後按一下 [驗證方法]。
1. 根據帳戶遭到鎖定之前所允許的失敗登入次數，設定 [鎖定閾值]。 預設值為 10。
1. 將 [鎖定持續時間 (秒)] 設為每次鎖定的秒數。 預設值為 60 秒 (一分鐘)。

> [!NOTE]
> 如果鎖定之後的第一個登入也失敗，帳戶將會再次鎖定。 如果帳戶重複鎖定，鎖定持續時間將會增加。

![在 Azure 入口網站中自訂 Azure AD 智慧鎖定原則](./media/howto-password-smart-lockout/azure-active-directory-custom-smart-lockout-policy.png)
## <a name="next-steps"></a>後續步驟

[了解如何使用 Azure AD 在組織中禁用不當密碼。](howto-password-ban-bad.md)

[設定自助式密碼重設以允許使用者解除鎖定其本身的帳戶。](quickstart-sspr.md)