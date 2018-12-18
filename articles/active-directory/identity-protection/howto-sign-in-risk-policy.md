---
title: 如何在 Azure Active Directory Identity Protection 中設定登入風險原則 | Microsoft Docs
description: 了解如何設定 Azure AD Identity Protection 登入風險原則。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: e7434eeb-4e98-4b6b-a895-b5598a6cccf1
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/13/2018
ms.author: markvi
ms.reviewer: raluthra
ms.openlocfilehash: bae71c55bdea838d0ef5e0ae3acbac3e98a92fa0
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055034"
---
# <a name="how-to-configure-the-sign-in-risk-policy"></a>如何：設定登入風險原則

Azure Active Directory 會以即時和離線方式偵測[風險事件類型](../reports-monitoring/concept-risk-events.md#risk-event-types)。 使用者登入時偵測到的每個風險事件，構成了名為有風險的登入的邏輯概念。 有風險的登入表示可能不是由使用者帳戶合法擁有者執行的嘗試登入。


## <a name="what-is-the-sign-in-risk-policy"></a>什麼是登入風險原則？

Azure AD 會分析使用者的每次登入。 分析的目的是要偵測伴隨登入出現的可疑動作。 例如，使用匿名 IP 位址完成登入，或是從不熟悉的位置起始登入？ 在 Azure AD 中，系統可以偵測到的可疑動作也稱為風險事件。 根據在登入期間偵測到的風險事件，Azure AD 會計算一個值。 此值代表不是由合法使用者執行登入的可能性 (低、中、高)。 此可能性稱為**登入風險層級**。

登入風險原則是您可以針對特定登入風險層級設定的自動化回應。 在您的回應中，您可以封鎖對資源的存取，或需要傳遞多重要素驗證 (MFA) 查問以取得存取。

   
## <a name="how-do-i-access-the-sign-in-risk-policy"></a>如何存取登入風險原則？
   
登入風險原則位於 [Azure AD Identity Protection] 頁面的 [設定](https://portal.azure.com/#blade/Microsoft_AAD_ProtectionCenter/IdentitySecurityDashboardMenuBlade/SignInPolicy) 區段中。
   
![登入風險原則](./media/howto-sign-in-risk-policy/1014.png "登入風險原則")


## <a name="policy-settings"></a>原則設定

當您設定登入風險原則時，您必須設定：

- 要套用原則的使用者和群組：

    ![使用者和群組](./media/howto-sign-in-risk-policy/11.png)

- 可觸發原則的登入風險層級：

    ![登入風險層級](./media/howto-sign-in-risk-policy/12.png)

- 您想要在符合登入風險層級時強制執行的存取類型：  

    ![Access](./media/howto-sign-in-risk-policy/13.png)

- 原則的狀態：

    ![強制執行原則](./media/howto-sign-in-risk-policy/14.png)


原則組態對話方塊會提供一個選項，以供您評估重新設定的影響。

![預估的影響](./media/howto-sign-in-risk-policy/15.png)

## <a name="what-you-should-know"></a>您應該知道的事情

您可以設定登入風險安全性原則以要求 MFA︰

![需要 MFA](./media/howto-sign-in-risk-policy/16.png)

不過，基於安全性理由，此設定只適用於已註冊 MFA 的使用者。 如果具有 MFA 需求的使用者尚未註冊 MFA，則 Identity Protection 會封鎖他們。

如果您想要要求有風險的登入進行 MFA，您應該：

1. 為受影響的使用者啟用 [Multi-Factor Authentication 註冊原則](#multi-factor-authentication-registration-policy)。

2. 要求受影響的使用者登入沒有危險的工作階段以執行 MFA 註冊。

完成這些步驟可確保有風險的登入一定需要 Multi-Factor Authentication。

登入風險原則：

- 會套用至所有使用新式驗證的瀏覽器流量和登入。

- 不會套用至使用較舊安全性通訊協定的應用程式，其做法是停用位於同盟 IDP (例如 ADFS) 的 WS-Trust 端點。


如需相關的使用者經驗概觀，請參閱︰

* [有風險的登入復原](flows.md#risky-sign-in-recovery)
* [已封鎖有風險的登入](flows.md#risky-sign-in-blocked)  
* [使用 Azure AD Identity Protection 時的登入體驗](flows.md)  

## <a name="best-practices"></a>最佳作法

選擇 [高]  臨界值可減少觸發原則的次數，並將對使用者的影響降至最低。  

不過，它會從原則中排除標示 [低] 和 [中] 度風險的登入，而無法阻止攻擊者利用遭到入侵的身分識別。

設定原則時，

- 排除不會 / 不能有 Multi-Factor Authentication 的使用者

- 在啟用原則並不實用 (例如無法存取技術服務人員) 的地區設定中排除使用者

- 排除可能會產生大量誤判的使用者 (開發人員、安全性分析人員)

- 在原則推出初期，或如果您必須盡量減少使用者所看到的挑戰，請使用 [高] 閾值。

- 如果您的組織需要更高的安全性，請使用 [低]  臨界值。 選取 [低]  臨界值會帶來額外的使用者登入挑戰，並提高安全性。

大部分組織的建議預設值是設定 [中]  臨界值的規則，以取得可用性與安全性之間的平衡。






## <a name="next-steps"></a>後續步驟

若要取得 Azure AD Identity Protection 的概觀，請參閱 [Azure AD Identity Protection 概觀](overview.md)。
