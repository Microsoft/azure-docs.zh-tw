---
title: 自訂 Azure AD 自助式密碼重設
description: Azure AD 自助式密碼重設的自訂選項
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: 69f6ed7814feacbd5adf60325aae123d388ffb61
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39222783"
---
# <a name="customize-the-azure-ad-functionality-for-self-service-password-reset"></a>自訂 Azure AD 的自助式密碼重設功能

想要在 Azure Active Directory (Azure AD) 中部署自助式密碼重設 (SSPR) 的 IT 專業人員可以自訂體驗以符合其使用者需求。

## <a name="customize-the-contact-your-administrator-link"></a>自訂 [請連絡您的系統管理員] 連結

即使未啟用 SSPR，使用者在密碼重設入口網站上仍然會有 [請連絡您的系統管理員] 連結。 如果使用者選取此連結，它將會執行下列其中一項操作：

   * 傳送電子郵件給系統管理員，請他們協助關變更使用者密碼。
   * 將使用者傳送到您所指定的 URL 以取得協助。

建議您將此連絡人設定為使用者已針對支援問題使用的電子郵件地址或網站等項目。

![連絡人][Contact]

這封連絡人電子郵件會依下列順序傳送給下列收件者︰

1. 如果已指派**密碼管理員**角色，就會通知具備此角色的系統管理員。
2. 如果未指派任何密碼管理員，就會通知具備**使用者管理員**角色的系統管理員。
3. 如果上述角色都未指派，則會通知**全域管理員**。

在所有情況下，最多 100 位收件者會收到通知。

若要深入了解不同的系統管理員角色，以及如何指派這些角色，請參閱[在 Azure Active Directory 中指派系統管理員角色](../users-groups-roles/directory-assign-admin-roles.md)。

### <a name="disable-contact-your-administrator-emails"></a>停用 [請連絡您的系統管理員] 電子郵件

如果您的組織不想傳送密碼重設要求通知給系統管理員，則您可以啟用下列設定：

* 對所有使用者啟用自助式密碼重設。 此選項位於 [密碼重設] > [屬性] 底下。 如果您不想讓使用者重設其自己的密碼，可以將存取範圍設定為空群組。 不建議使用此選項。
* 自訂技術服務人員連結，以提供可讓使用者取得協助的 Web URL 或 mailto︰位址。 此選項位於 [密碼重設] > [自訂] > [自訂的技術服務人員電子郵件或 URL] 底下。

## <a name="customize-the-ad-fs-sign-in-page-for-sspr"></a>自訂 SSPR 的 AD FS 登入頁面

「Active Directory 同盟服務」(AD FS) 系統管理員可以使用[新增登入頁面描述](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/add-sign-in-page-description) \(機器翻譯\) 一文中所提供的指引，新增其登入頁面的連結。

若要新增 AD FS 登入頁面的連結，請在您的 AD FS 伺服器上使用下列命令。 使用者可以使用此頁面來進入 SSPR 工作流程。

``` Set-ADFSGlobalWebContent -SigninPageDescriptionText "<p><A href='https://passwordreset.microsoftonline.com' target='_blank'>Can’t access your account?</A></p>" ```

## <a name="customize-the-sign-in-page-and-access-panel-look-and-feel"></a>自訂登入頁面和存取面板的外觀與風格

您可以自訂登入頁面。 您可以新增與符合您公司商標的影像一起顯示的標誌。

您選擇的圖形會在下列情況下顯示：

* 在使用者輸入其使用者名稱之後
* 如果使用者存取自訂的 URL，則會透過下列方式：
    * 藉由傳遞 *whr* 參數傳送到密碼重設頁面，例如 "https://login.microsoftonline.com/?whr=contoso.com"
    * 藉由傳遞 *username* 參數到密碼重設頁面，例如 "https://login.microsoftonline.com/?username=admin@contoso.com"

如需有關如何設定公司商標的詳細資料，請參閱[將公司商標新增至 Azure AD 中的登入頁面](../fundamentals/customize-branding.md)一文。

### <a name="directory-name"></a>目錄名稱

您可以在 [Azure Active Directory] > [屬性] 底下變更目錄名稱屬性。 您可以顯示一個會在入口網站及自動化通訊中顯示的易記組織名稱。 這個選項最常以下列形式出現在自動化電子郵件中：

* 電子郵件中的易記名稱，例如「Microsoft 代表 CONTOSO 示範」
* 電子郵件中的主旨列，例如「CONTOSO 示範帳戶電子郵件驗證碼」

## <a name="next-steps"></a>後續步驟

* [如何完成 SSPR 成功首度發行？](howto-sspr-deployment.md)
* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)
* [您有授權問題嗎？](concept-sspr-licensing.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [哪些驗證方法可供使用者使用？](concept-sspr-howitworks.md#authentication-methods)
* [使用 SSPR 的原則選項有哪些？](concept-sspr-policy.md)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [如何回報 SSPR 中的活動？](howto-sspr-reporting.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [我認為有中斷。如何針對 SSPR 進行疑難排解？](active-directory-passwords-troubleshoot.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)

[Contact]: ./media/concept-sspr-customization/sspr-contact-admin.png "連絡您的系統管理員以協助重設您的密碼電子郵件範例"
