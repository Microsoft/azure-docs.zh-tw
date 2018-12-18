---
title: iOS 上的 Azure Active Directory 憑證式驗證
description: 了解在有 iOS 裝置的解決方案中，設定憑證式驗證的支援案例和需求
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: article
ms.date: 01/15/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: annaba
ms.openlocfilehash: 655fa6b4bf0f04f2d88e9a3f11cb9d3917ea3dd3
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43346461"
---
# <a name="azure-active-directory-certificate-based-authentication-on-ios"></a>iOS 上的 Azure Active Directory 憑證式驗證

在連線至下列項目時，iOS 裝置可以使用憑證式驗證 (CBA) 向 Azure Active Directory 進行驗證 (透過其裝置上的用戶端憑證)：

* Office 行動應用程式，例如 Microsoft Outlook 與 Microsoft Word
* Exchange ActiveSync (EAS) 用戶端

設定這項功能之後，就不需要在行動裝置上的特定郵件和 Microsoft Office 應用程式中，輸入使用者名稱和密碼的組合。

本主題針對 Office 365 企業版、商務版、教育版、美國政府機關、中國和德國方案的租用戶使用者，提供在 iOS (Android) 裝置上設定 CBA 的需求和支援案例。

在 Office 365 US Government Defense 和 Federal 方案中，這項功能處於預覽版。

## <a name="microsoft-mobile-applications-support"></a>Microsoft 行動應用程式支援

| 應用程式 | 支援 |
| --- | --- |
| Azure 資訊保護應用程式 |![勾選][1] |
| Intune 公司入口網站 |![勾選][1] |
| Microsoft Teams |![勾選][1] |
| OneNote |![勾選][1] |
| OneDrive |![勾選][1] |
| Outlook |![勾選][1] |
| Power BI |![勾選][1] |
| 商務用 Skype |![勾選][1] |
| Word / Excel / PowerPoint |![勾選][1] |
| Yammer |![勾選][1] |

## <a name="requirements"></a>需求

裝置作業系統版本必須是 iOS 9 和更新版本

必須設定同盟伺服器。

iOS 上的 Office 應用程式都需要 Microsoft Authenticator。

ADFS 權杖必須要有下列宣告，Azure Active Directory 才能撤銷用戶端憑證︰

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>` (用戶端憑證的序號)
* `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>` (用戶端憑證的簽發者字串)

如果 ADFS 權杖 (或任何其他 SAML 權杖) 中有上述宣告，Azure Active Directory 就會將這些宣告新增至重新整理權杖。 當需要驗證重新整理權杖時，這項資訊會用於檢查撤銷。

最佳做法是應該以下列資訊來更新組織的 ADFS 錯誤頁面︰

* 在 iOS 上安裝 Microsoft Authenticator 的需求
* 如何取得使用者憑證的指示。

如需詳細資訊，請參閱[自訂 AD FS 登入頁面](https://technet.microsoft.com/library/dn280950.aspx)。

某些 Office 應用程式 (已啟用新式驗證) 會在其要求中將 ‘*prompt=login*’ 傳送至 Azure AD。 根據預設，Azure AD 會將對 ADFS 的要求中的 ‘*prompt=login*’ 轉譯成 ‘*wauth=usernamepassworduri*’ (請求 ADFS 進行 U/P 驗證) 和 ‘*wfresh=0*’ (請求 ADFS 忽略 SSO 狀態並進行全新驗證)。 如果您想要啟用這些應用程式的憑證型驗證，您必須修改預設的 Azure AD 行為。 只要將您的同盟網域設定中的 'PromptLoginBehavior' 設定為 ‘Disabled‘。
您可以使用 [MSOLDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings?view=azureadps-1.0) Cmdlet 來執行這項工作︰

`Set-MSOLDomainFederationSettings -domainname <domain> -PromptLoginBehavior Disabled`

## <a name="exchange-activesync-clients-support"></a>Exchange ActiveSync 用戶端支援

iOS 9 或更新版本支援原生 iOS 郵件用戶端。 針對其他所有 Exchange ActiveSync 應用程式，若要判斷這項功能是否受支援，請連絡您的應用程式開發人員。

## <a name="next-steps"></a>後續步驟

如果您想要在環境中設定憑證式驗證，相關指示請參閱[在 Android 上開始使用憑證式驗證](../authentication/active-directory-certificate-based-authentication-get-started.md)。

<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-ios/ic195031.png
