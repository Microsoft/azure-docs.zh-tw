---
title: 了解 Azure Active Directory B2C 中入門套件的自訂原則 | Microsoft Docs
description: Azure Active Directory B2C 自訂原則的主題。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/25/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: ebcd7a677acde12558b0f566bce9172a0d00233b
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442469"
---
# <a name="understanding-the-custom-policies-of-the-azure-ad-b2c-custom-policy-starter-pack"></a>了解 Azure AD B2C 自訂原則入門套件的自訂原則

本節會列出**入門套件**隨附之 B2C_1A_base 原則的所有核心元素，此原則也可用來透過繼承 B2C_1A_base_extensions 原則來撰寫您自己的原則。

因此，本節會特別著重在已定義的宣告類型、宣告轉換、內容定義、宣告提供者與其技術設定檔，以及核心的使用者旅程圖。

> [!IMPORTANT]
> Microsoft 對於以下所提供的資訊不做任何明示或暗示的保證。 任何時候 (GA 當下、之前或之後) 皆有可能導入變更。

您自己的原則和 B2C_1A_base_extensions 原則都可以覆寫這些定義，並視需要提供其他定義來擴充此父系原則。

B2C_1A_base 原則的核心元素是宣告類型、宣告轉換及內容定義。 您可以很容易地在您自己的原則和 B2C_1A_base_extensions 原則中參考這些元素。

## <a name="claims-schemas"></a>宣告結構描述

此宣告結構描述分為三個部分︰

1.  第一個部分會列出所需的最小宣告，以便使用者旅程能正常運作。
2.  第二個部分會列出所需的宣告，以便查詢字串參數和其他特殊參數能傳遞至其他宣告提供者，尤其是 login.microsoftonline.com 以便進行驗證。 **請勿修改這些宣告**。
3.  最後，第三個部分會列出其他任何選擇性宣告，這些宣告可從使用者收集而來、儲存在目錄中，以及在登入期間於權杖中傳送。 要從使用者收集而來和/或要在權杖中傳送的新宣告類型可以新增到這個部分。

> [!IMPORTANT]
> 對於某些宣告 (例如密碼和使用者名稱)，宣告結構描述會含有限制。 信任架構 (TF) 原則會將 Azure AD 視為其他宣告提供者，此原則的所有限制會在自訂原則中模型化。 原則經過修改後即可新增更多限制，或使用另一個宣告提供者來儲存認證，但該提供者會有它自己的限制。

以下列出可用的宣告類型。

### <a name="claims-that-are-required-for-the-user-journeys"></a>使用者旅程所需的宣告

需要有下列宣告才能讓使用者旅程正常運作︰

| 宣告類型 | 說明 |
|-------------|-------------|
| UserId | 使用者名稱 |
| signInName | 登入名稱 |
| tenantId | Azure AD B2C 中使用者物件的租用戶識別碼 (ID) |
| objectId | Azure AD B2C 中使用者物件的物件識別碼 (ID) |
| *password* | 密碼 |
| newPassword | |
| reenterPassword | |
| passwordPolicies | Azure AD B2C 用來決定密碼強度、到期日等規定的密碼原則。 |
| *sub* | |
| alternativeSecurityId | |
| identityProvider | |
| displayName | |
| strongAuthenticationPhoneNumber | 使用者的電話號碼 |
| Verified.strongAuthenticationPhoneNumber | |
| email | 可用來連絡使用者的電子郵件地址 |
| signInNamesInfo.emailAddress | 使用者可用來登入的電子郵件地址 |
| otherMails | 可用來連絡使用者的電子郵件地址 |
| userPrincipalName | Azure AD B2C 中所儲存的使用者名稱 |
| upnUserName | 用來建立使用者主體名稱的使用者名稱 |
| mailNickName | Azure AD B2C 中所儲存的使用者郵件別名 |
| newUser | |
| executed-SelfAsserted-Input | 此宣告會指出屬性是否收集自使用者 |
| executed-PhoneFactor-Input | 此宣告會指出新的電話號碼是否收集自使用者 |
| authenticationSource | 指出使用者是在社交識別提供者、login.microsoftonline.com 還是本機帳戶進行驗證的 |

### <a name="claims-required-for-query-string-parameters-and-other-special-parameters"></a>查詢字串參數和其他特殊參數所需的宣告

必須有下列宣告才能將特殊參數 (包括某些查詢字串參數) 傳遞給其他宣告提供者︰

| 宣告類型 | 說明 |
|-------------|-------------|
| nux | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| nca | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| prompt | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| mkt | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| lc | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| grant_type | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| scope | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| client_id | 傳遞給本機帳戶以向 login.microsoftonline.com 進行驗證的特殊參數 |
| objectIdFromSession | 預設工作階段管理提供者所提供的參數，用以指出物件識別碼是從 SSO 工作階段擷取來的 |
| isActiveMFASession | MFA 工作階段管理所提供的參數，用以指出使用者具有作用中的 MFA 工作階段 |

### <a name="additional-optional-claims-that-can-be-collected"></a>其他可以收集的 (選擇性) 宣告

下列宣告是可以從使用者收集而來、儲存在目錄中，以及在權杖中傳送的其他宣告。 如先前所述，您可以將其他宣告新增至此清單。

| 宣告類型 | 說明 |
|-------------|-------------|
| givenName | 使用者的名字 (也就是第一個名字) |
| surname | 使用者的姓氏 |
| Extension_picture | 使用者的社交網站相片 |

## <a name="claim-transformations"></a>宣告轉換

以下列出可用的宣告轉換。

| 宣告轉換 | 說明 |
|----------------------|-------------|
| CreateOtherMailsFromEmail | |
| CreateRandomUPNUserName | |
| CreateUserPrincipalName | |
| CreateSubjectClaimFromObjectID | |
| CreateSubjectClaimFromAlternativeSecurityId | |
| CreateAlternativeSecurityId | |

## <a name="content-definitions"></a>內容定義

本節說明 B2C_1A_base 原則中已經宣告的內容定義。 您可以視需要輕易地在您自己的原則和 B2C_1A_base_extensions 原則中，參考、覆寫和/或擴充這些內容定義。

| 宣告提供者 | 說明 |
|-----------------|-------------|
| *Facebook* | |
| 本機帳戶登入 | |
| PhoneFactor | |
| *Azure Active Directory* | |
| 自我判斷提示 | |
| 本機帳戶 | |
| *工作階段管理* | |
| Trustframework 原則引擎 | |
| TechnicalProfiles | |
| 權杖簽發者 | |

## <a name="technical-profiles"></a>技術設定檔

本節說明 B2C_1A_base 原則中每個宣告提供者已經宣告的技術設定檔。 您可以視需要輕易地在您自己的原則和 B2C_1A_base_extensions 原則中，進一步參考、覆寫和/或擴充這些技術設定檔。

### <a name="technical-profiles-for-facebook"></a>Facebook 的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| Facebook-OAUTH | |

### <a name="technical-profiles-for-local-account-signin"></a>本機帳戶登入的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| Login-NonInteractive | |

### <a name="technical-profiles-for-phone-factor"></a>PhoneFactor 的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| PhoneFactor-Input | |
| PhoneFactor-InputOrVerify | |
| PhoneFactor-Verify | |

### <a name="technical-profiles-for-azure-active-directory"></a>Azure Active Directory 的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| AAD-Common | 其他 AAD-xxx 技術設定檔所包含的技術設定檔 |
| AAD-UserWriteUsingAlternativeSecurityId | 社交登入的技術設定檔 |
| AAD-UserReadUsingAlternativeSecurityId | 社交登入的技術設定檔 |
| AAD-UserReadUsingAlternativeSecurityId-NoError | 社交登入的技術設定檔 |
| AAD-UserWritePasswordUsingLogonEmail | 本機帳戶的技術設定檔 |
| AAD-UserReadUsingEmailAddress | 本機帳戶的技術設定檔 |
| AAD-UserWriteProfileUsingObjectId | 可供使用 objectId 來更新使用者記錄的技術設定檔 |
| AAD-UserWritePhoneNumberUsingObjectId | 可供使用 objectId 來更新使用者記錄的技術設定檔 |
| AAD-UserWritePasswordUsingObjectId | 可供使用 objectId 來更新使用者記錄的技術設定檔 |
| AAD-UserReadUsingObjectId | 此技術設定檔可用來在驗證使用者後讀取資料 |

### <a name="technical-profiles-for-self-asserted"></a>自我判斷提示的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| SelfAsserted-Social | |
| SelfAsserted-ProfileUpdate | |

### <a name="technical-profiles-for-local-account"></a>本機帳戶的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| LocalAccountSignUpWithLogonEmail | |

### <a name="technical-profiles-for-session-management"></a>工作階段管理的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| SM-Noop | |
| SM-AAD | |
| SM-SocialSignup | 設定檔名稱將會用來區分註冊與登入的 AAD 工作階段 |
| SM-SocialLogin | |
| SM-MFA | |

### <a name="technical-profiles-for-the-trust-framework-policy-engine"></a>信任架構原則引擎的技術設定檔

**Trustframework 原則引擎 TechnicalProfiles** 宣告提供者目前沒有已定義的技術設定檔。

### <a name="technical-profiles-for-token-issuer"></a>權杖簽發者的技術設定檔

| 技術設定檔 | 說明 |
|-------------------|-------------|
| JwtIssuer | |

## <a name="user-journeys"></a>使用者旅程

本節說明 B2C_1A_base 原則中已經宣告的使用者旅程。 您可以視需要輕易地在您自己的原則和 B2C_1A_base_extensions 原則中，進一步參考、覆寫和/或擴充這些使用者旅程。

| 使用者旅程 | 說明 |
|--------------|-------------|
| SignUp | |
| SignIn | |
| SignUpOrSignIn | |
| EditProfile | |
| PasswordReset | |
