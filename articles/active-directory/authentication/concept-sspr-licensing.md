---
title: 授權 Azure Active Directory 自助式密碼
description: Azure AD 自助式密碼重設授權需求
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/17/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 83054c505689768c14d168841764a4557c3e1f8b
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39158993"
---
# <a name="licensing-requirements-for-azure-ad-self-service-password-reset"></a>Azure AD 自助式密碼重設的授權需求

Azure Active Directory (Azure AD) 共有四種版本：Free、Basic、Premium P1 及 Premium P2。 有幾個構成自助式密碼重設的不同功能 (包括變更、重設、解除鎖定和回寫)，可在不同的 Azure AD 版本中提供。 本文嘗試說明差異。 如需每個 Azure AD 版本中包含的功能詳細資料，請參閱 [Azure Active Directory 定價頁面](https://azure.microsoft.com/pricing/details/active-directory/)。

## <a name="compare-editions-and-features"></a>比較版本和功能

Azure AD 自助式密碼重設會依使用者授權來維持合規性，而組織必須將適當的授權指派給其使用者。

* 雲端使用者的自助式密碼變更
   * 我是**僅限雲端的使用者**而且知道我的密碼。
      * 我想要將我的密碼**變更**為新密碼。
   * 這項功能包含在所有的 Azure AD 版本中。

* 雲端使用者的自助式密碼重設
   * 我是**僅限雲端的使用者**而且忘了我的密碼。
      * 我想要將我的密碼**重設**為我所知道的密碼。
   * 這項功能包含在 Azure AD Basic、Premium P1 或 Premium P2 版本中。

* 自助式密碼重設/變更/**使用內部部署回寫**來解鎖
   * 我是**混合式使用者**，我的內部部署 Active Directory 使用者帳戶已使用 Azure AD Connect 與我的 Azure AD 帳戶同步處理。 我想要變更我的密碼、忘了我的密碼，或已鎖定。
      * 我想要變更我的密碼或將它重設為我知道的，或將我的帳戶解除鎖定，**以及**將該變更同步處理回到內部部署 Active Directory。
   * 這項功能包含在 Azure AD Premium P1 或 Premium P2 版本中。

> [!WARNING]
> 獨立的 Office 365 授權方案**不支援密碼回寫**，而且需要 Azure AD Premium P1, or Premium P2 選項，這項功能才能運作。
>

在下列分頁可以找到額外的授權資訊 (包括成本)：

* [Azure Active Directory 價格網站](https://azure.microsoft.com/pricing/details/active-directory/)
* [Azure Active Directory 功能和功用](https://www.microsoft.com/cloud-platform/azure-active-directory-features)
* [Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security)
* [Microsoft 365 企業版](https://www.microsoft.com/microsoft-365/enterprise)

## <a name="enable-group-or-user-based-licensing"></a>啟用以群組或使用者為基礎的授權

Azure AD 現在支援以群組為基礎的授權。 系統管理員可以將大量授權指派給一群使用者，而不是一次指派一個授權。 如需詳細資訊，請參閱[指派、驗證授權及解決其問題](../users-groups-roles/licensing-groups-assign.md#step-1-assign-the-required-licenses)。

並非所有位置都可使用某些 Microsoft 服務。 系統管理員必須先指定使用者的 [使用位置] 屬性，才能將授權指派給使用者。 可以在 Azure 入口網站中的 [使用者]  >  [設定檔]  >  [設定] 區段之下進行授權指派。 使用群組授權指派時，未指定使用位置的任何使用者在指派期間會繼承目錄的位置。

## <a name="next-steps"></a>後續步驟

* [如何完成 SSPR 成功首度發行？](howto-sspr-deployment.md)
* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [哪些驗證方法可供使用者使用？](concept-sspr-howitworks.md#authentication-methods)
* [使用 SSPR 的原則選項有哪些？](concept-sspr-policy.md)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [如何回報 SSPR 中的活動？](howto-sspr-reporting.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [我認為有中斷。如何針對 SSPR 進行疑難排解？](active-directory-passwords-troubleshoot.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)
