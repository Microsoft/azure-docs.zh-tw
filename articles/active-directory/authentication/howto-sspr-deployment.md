---
title: 自助式密碼重設部署指南 - Azure Active Directory
description: 成功推出 Azure AD 自助式密碼重設的祕訣
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/17/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: f786a20f61c8851c1d50a89edd392b9b974db076
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39622315"
---
# <a name="how-to-successfully-roll-out-self-service-password-reset"></a>如何成功推出自助式密碼重設

為確保 Azure Active Directory (Azure AD) 自助式密碼重設 (SSPR) 功能順利推出，大部分的客戶會完成下列步驟：

> [!VIDEO https://www.youtube.com/embed/OZn5btP6ZXw]

1. 透過您組織的一小部分，完成試驗推出。
   * 如需試驗方式的相關資訊，請參閱[教學課程：完成 Azure AD 自助式密碼重設試驗推出](tutorial-sspr-pilot.md)。
1. 教育您的技術服務人員。
   * 他們要如何協助您的使用者？
   * 您會強制使用者使用 SSPR，而不允許技術服務人員協助使用者嗎？
   * 您是否為他們提供可供註冊和重設的 URL？
      * 註冊： https://aka.ms/ssprsetup
      * 重設： https://aka.ms/sspr
1. 教育您的使用者。
   * 本文件的下列幾節介紹範例通訊、密碼入口網站、如何強制執行註冊，以及填入驗證資料。
   * Azure Active Directory 產品群組已建立[逐步部署方案](https://aka.ms/SSPRDeploymentPlan)，組織能平行使用此方案，並搭配在此網站中找到的文件，來製作業務案例及規劃自助式重設密碼的部署。
1. 為您的整個組織啟用自助式密碼重設。
   * 當您準備好時，就可以為所有使用者啟用密碼重設，將 [已啟用自助式密碼重設] 切換為 [全部]。

## <a name="sample-communication"></a>範例通訊

許多客戶發現要讓使用者使用 SSPR 的最簡單方法，是使用包含易於使用指示的電子郵件行銷活動。 [我們建立了可作為範本的簡單電子郵件和其他宣傳品，協助您推出](https://www.microsoft.com/download/details.aspx?id=56768)：

* **即將推出**：於推出前幾週或前幾天使用電子郵件範本，可讓使用者知道他們需要執行一些動作。
* **立即可用**：於程式推出當天使用的電子郵件範本，可驅使使用者註冊並確認其驗證資料。 如果使用者立即註冊，就可在需要時使用 SSPR。
* **註冊提醒**：電子郵件範本，可在部署後幾天至幾週提醒使用者進行註冊並確認其驗證資料。
* **SSPR 海報**：您可以自訂的海報，以在您首次推出前後數天和數週在您的組織附近展示。
* **SSPR 表格三角立牌**：您可以放在午餐食堂、會議室或桌子上的表格三角立牌，以鼓勵您的使用者完成註冊。
* **SSPR 貼紙**：您可以自訂及列印的貼紙範本，貼在筆記型電腦、監視器、鍵盤或行動電話上以提醒如何存取 SSPR。

![SSPR 電子郵件範例][Email]

## <a name="create-your-own-password-portal"></a>建立您自己的密碼入口網站

很多客戶都選擇裝載網頁並建立 DNS 根項目，例如 https://passwords.contoso.com。 它們會使用下列資訊的連結填入此頁面：

* [Azure AD 密碼重設入口網站 - https://aka.ms/sspr](https://aka.ms/sspr)
* [Azure AD 密碼重設註冊入口網站 - https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)
* [Azure AD 密碼變更入口網站 - https://account.activedirectory.windowsazure.com/ChangePassword.aspx](https://account.activedirectory.windowsazure.com/ChangePassword.aspx)
* 其他組織特定資訊

您送出的任何電子郵件或傳單都可以包含一個品牌化、令人印象深刻的 URL，使用者可以在需要使用服務時前往該 URL。 為方便您使用，我們建立了[範例密碼重設頁面](https://github.com/ajamess/password-reset-page)，您可以使用並根據組織需求加以自訂。

## <a name="use-enforced-registration"></a>使用強制註冊

如果您希望使用者註冊密碼重設，可以要求他們在透過 Azure AD 登入時進行註冊。 您可以從目錄的 [密碼重設] 窗格啟用此選項，方法是啟用 [註冊] 索引標籤上的 [要求使用者在登入時註冊] 選項。

系統管理員可以要求使用者在一段特定的時間之後重新註冊。 他們可以將 [要求使用者重新確認其驗證資訊的等候天數] 選項設定為 0 到 730 天。

啟用此選項之後，使用者會在登入時會看到一則訊息，告知系統管理員已要求他們確認其驗證資訊。

## <a name="populate-authentication-data"></a>填入驗證資料

您應該考慮[為您的使用者預先填入一些驗證資料](howto-sspr-authenticationdata.md)。 如此一來，使用者不必註冊重設密碼就能夠使用 SSPR。 只要使用者提供的驗證資料符合您定義的密碼重設原則，他們就能夠重設其密碼。

## <a name="disable-self-service-password-reset"></a>啟用自助式密碼重設

如果您的組織決定停用自助式密碼重設，這是很簡單的程序。 開啟您的 Azure AD 租用戶並移至 [密碼重設] > [屬性]，然後選取 [已啟用自助式密碼重設] 之下的 [無]。 使用者仍將維持其已註冊的驗證方法，供日後使用。

## <a name="next-steps"></a>後續步驟

* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)
* [啟用 Azure Multi-Factor Authentication 的聚合式註冊和 Azure AD 自助密碼重設](concept-registration-mfa-sspr-converged.md)
* [您有授權問題嗎？](concept-sspr-licensing.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [使用 SSPR 的原則選項有哪些？](concept-sspr-policy.md)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [如何回報 SSPR 中的活動？](howto-sspr-reporting.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [我認為有中斷。如何針對 SSPR 進行疑難排解？](active-directory-passwords-troubleshoot.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)

[Email]: ./media/howto-sspr-deployment/sspr-emailtemplates.png "自訂這些電子郵件範本，以符合您組織的需求"
