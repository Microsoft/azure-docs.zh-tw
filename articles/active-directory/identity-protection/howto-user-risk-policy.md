---
title: 如何在 Azure Active Directory Identity Protection 中設定使用者風險原則 | Microsoft Docs
description: 了解如何設定 Azure AD Identity Protection 使用者風險原則。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: e7434eeb-4e98-4b6b-a895-b5598a6cccf1
ms.service: active-directory
ms.component: identity-protection
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/08/2017
ms.author: markvi
ms.reviewer: raluthra
ms.openlocfilehash: a54403b7794d26d87c810f5cd20050db35c078f1
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47054309"
---
# <a name="how-to-configure-the-user-risk-policy"></a>如何：設定使用者風險原則

透過使用者風險，Azure AD 可偵測使用者帳戶遭到入侵的可能性。 系統管理員可以設定使用者風險條件式存取原則，來自動回應特定登入風險層級。
 
本文提供設定使用者風險原則所需的資訊。


## <a name="what-is-a-user-risk-policy"></a>什麼是使用者風險原則？

Azure AD 會分析使用者的每次登入。 分析的目的是要偵測伴隨登入出現的可疑動作。 在 Azure AD 中，系統可以偵測到的可疑動作也稱為風險事件。 雖然可以即時偵測到有些風險事件，但也有宜些風險事件需要更多時間。 例如，若要偵測不可能到達非典型位置的旅行，系統需要 14 天的初始學習期間，才能了解使用者的一般行為。 有數個選項可以解決偵測到的風險事件。 例如，您可以手動解決個別的風險事件，或者使用登入風險或使用者風險條件式存取原則來解決這些事件。

針對使用者偵測到但未獲得解決的所有風險事件也稱為作用中風險事件。 與使用者相關聯的作用中風險事件也稱為使用者風險。 Azure AD 會根據使用者風險，計算使用者遭到入侵的可能性 (低、中、高)。 此可能性稱為使用者風險層級。

![使用者風險](./media/howto-user-risk-policy/1031.png)

使用者風險原則是您可以針對特定使用者風險層級設定的自動化回應。 利用使用者風險原則，您可以封鎖對資源的存取，或需要變更密碼，讓使用者帳戶回到初始狀態。


## <a name="how-do-i-access-the-sign-in-risk-policy"></a>如何存取登入風險原則？
   
登入風險原則位於 [Azure AD Identity Protection] 頁面的 [設定](https://portal.azure.com/#blade/Microsoft_AAD_ProtectionCenter/IdentitySecurityDashboardMenuBlade/SignInPolicy) 區段中。
   
![使用者風險原則](./media/howto-user-risk-policy/1014.png)



## <a name="policy-settings"></a>原則設定

當您設定登入風險原則時，您必須設定：

- 要套用原則的使用者和群組：

    ![使用者和群組](./media/howto-user-risk-policy/11.png)

- 可觸發原則的登入風險層級：

    ![使用者風險層級](./media/howto-user-risk-policy/12.png)

- 您想要在符合登入風險層級時強制執行的存取類型：  

    ![Access](./media/howto-user-risk-policy/13.png)

- 原則的狀態：

    ![強制執行原則](./media/howto-user-risk-policy/14.png)

原則組態對話方塊會提供一個選項，以供您評估您的組態影響。

![預估的影響](./media/howto-user-risk-policy/15.png)

## <a name="what-you-should-know"></a>您應該知道的事情

您可以設定使用者風險安全性原則，以根據風險層級防止使用者登入。

![封鎖](./media/howto-user-risk-policy/16.png)


封鎖登入：

* 避免對受影響的使用者產生新的使用者風險事件
* 可讓系統管理員手動地補救會影響使用者身分識別的風險事件，並將它還原到安全的狀態

## <a name="best-practices"></a>最佳作法

選擇 [高]  臨界值可減少觸發原則的次數，並將對使用者的影響降至最低。
不過，這會從原則中排除標示 [低] 和 [中] 度風險的使用者，而無法保護先前疑似或已知遭到入侵的身分識別或裝置。

設定原則時，

* 排除可能會產生大量誤判的使用者 (開發人員、安全性分析人員)
* 在啟用原則並不實用 (例如無法存取技術服務人員) 的地區設定中排除使用者
* 在原則推出初期，或如果您必須盡量減少使用者所看到的挑戰，請使用 [高]  臨界值。
* 如果您的組織需要更高的安全性，請使用 [低]  臨界值。 選取 [低]  臨界值會帶來額外的使用者登入挑戰，並提高安全性。

大部分組織的建議預設值是設定 [中]  臨界值的規則，以取得可用性與安全性之間的平衡。

如需相關的使用者經驗概觀，請參閱︰

* [遭到入侵的帳戶復原流程](flows.md#compromised-account-recovery)。  
* [遭到入侵的帳戶封鎖流程](flows.md#compromised-account-blocked)。  

**開啟相關的組態對話方塊**：

- 在 [Azure AD Identity Protection] 刀鋒視窗的 [設定] 區段中，按一下 [使用者風險原則]。

    ![使用者風險原則](./media/howto-user-risk-policy/1009.png "使用者風險原則")




## <a name="next-steps"></a>後續步驟

若要取得 Azure AD Identity Protection 的概觀，請參閱 [Azure AD Identity Protection 概觀](overview.md)。
