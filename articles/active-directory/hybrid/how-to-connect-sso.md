---
title: Azure AD Connect：無縫單一登入 | Microsoft Docs
description: 本主題描述 Azure Active Directory (Azure AD) 的無縫單一登入，以及它如何讓您為公司網路內部的公司桌面使用者，提供真正的單一登入。
services: active-directory
keywords: 何謂 Azure AD Connect、安裝 Active Directory、Azure AD、SSO、單一登入的必要元件
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/24/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 4e9292b5932c36d55069ff300bf7c0a99b280fc5
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47054179"
---
# <a name="azure-active-directory-seamless-single-sign-on"></a>Azure Active Directory 無縫單一登入

## <a name="what-is-azure-active-directory-seamless-single-sign-on"></a>何謂 Azure Active Directory 無縫單一登入？

使用者位於連線到公司網路的公司裝置時，Azure Active Directory 無縫單一登入 (Azure AD 無縫 SSO) 就會自動將他們登入。 當此功能啟用時，使用者不需要輸入密碼就能登入 Azure AD，而且在大部分情況下，甚至也不用輸入其使用者名稱。 這項功能可讓使用者輕鬆存取雲端式應用程式，而不需要任何額外的內部部署元件。

>[!VIDEO https://www.youtube.com/embed/PyeAC85Gm7w]

無縫 SSO 可以與[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)或[傳遞驗證](how-to-connect-pta.md)登入方法合併使用。 無縫 SSO 不適用於 Active Directory 同盟服務 (ADFS)。

![無縫單一登入](./media/how-to-connect-sso/sso1.png)

>[!IMPORTANT]
>無縫 SSO 需要將使用者的裝置**加入網域**，但不需要將裝置[加入 Azure AD](../active-directory-azureadjoin-overview.md)。

## <a name="key-benefits"></a>主要權益

- 良好的使用者體驗
  - 使用者會自動登入內部部署和雲端式應用程式。
  - 使用者不需要重複輸入其密碼。
- 容易部署和管理
  - 在內部部署上不需要任何其他元件，即可進行這項工作。
  - 與任何雲端驗證方法搭配運作：[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)或[傳遞驗證](how-to-connect-pta.md)。
  - 可以推出給使用群組原則的一部分使用者或所有使用者。
  - 可透過 Azure AD 註冊非 Windows 10 裝置，無需任何的 AD FS 基礎結構。 此功能需要使用 2.1 版或更新版本的[加入工作場所用戶端](https://www.microsoft.com/download/details.aspx?id=53554)。

## <a name="feature-highlights"></a>功能要點

- 登入使用者名稱可以是內部部署的預設使用者名稱 (`userPrincipalName`)，或在 Azure AD Connect 中設定的另一個屬性 (`Alternate ID`)。 兩種使用案例均可行，因為無縫 SSO 在 Kerberos 票證中使用 `securityIdentifier` 宣告在 Azure AD 中查詢對應的使用者物件。
- 無縫 SSO 是一種靈活變換的功能。 如果因任何原因而失敗，使用者登入體驗會改回其一般行為；亦即，使用者必須在登入頁面上輸入密碼。
- 如果應用程式 (例如 https://myapps.microsoft.com/contoso.com)) 在其 Azure AD 登入要求中轉送 `domain_hint` (OpenID Connect) 或 `whr` (SAML) 參數來識別您的租用戶，或是轉送 `login_hint` 參數來識別使用者，則會自動將使用者登入，而不需要輸入使用者名稱或密碼。
- 如果應用程式 (例如， https://contoso.sharepoint.com)) 將登入要求傳送至 Azure AD 的租用戶端點 (也就是 https://login.microsoftonline.com/contoso.com/<..> 或 https://login.microsoftonline.com/<tenant_ID>/<..>) 而不是 Azure AD 的一般端點 (也就是 https://login.microsoftonline.com/common/<...>)，使用者也可獲得無訊息登入體驗。
- 支援登出。 這可讓使用者選擇使用另一個 Azure AD 帳戶來進行登入，而不自動使用「無縫 SSO」來自動登入。
- 使用非互動式流程，來支援 Office 365 Win32 用戶端 (Outlook、Word、Excel 和其他產品) 16.0.8730.xxxx 版和更新版本。 針對 OneDrive，您必須啟用 [OneDrive 無訊息設定功能](https://techcommunity.microsoft.com/t5/Microsoft-OneDrive-Blog/Previews-for-Silent-Sync-Account-Configuration-and-Bandwidth/ba-p/120894) \(英文\) 以獲得無訊息登入體驗。
- 您可以透過 Azure AD Connect 啟用它。
- 這是免費功能，您不需要任何付費的 Azure AD 版本即可使用。
- 它是在能夠使用 Kerberos 驗證的平台和瀏覽器上，支援[新式驗證](https://aka.ms/modernauthga)的網頁瀏覽器型用戶端和 Office 用戶端來支援：

| 作業系統\瀏覽器 |Internet Explorer|Edge|Google Chrome|Mozilla Firefox|Safari|
| --- | --- |--- | --- | --- | -- 
|Windows 10|是\*|否|是|是\*\*\*|N/A
|Windows 8.1|是\*|N/A|是|是\*\*\*|N/A
|Windows 8|是\*|N/A|是|是\*\*\*|N/A
|Windows 7|是\*|N/A|是|是\*\*\*|N/A
|Windows Server 2012 R2 或更新版本|是\*\*|N/A|是|是\*\*\*|N/A
|Mac OS X|N/A|N/A|是\*\*\*|是\*\*\*|是\*\*\*


\*需要 Internet Explorer 第 10 版或更新版本

\*\*需要 Internet Explorer 第 10 版或更新版本。 停用增強保護模式

\*\*\*需要[其他設定](how-to-connect-sso-quick-start.md#browser-considerations)

>[!NOTE]
>對於 Windows 10，建議使用 [Azure AD Join](../active-directory-azureadjoin-overview.md) 以獲得 Azure AD 最佳單一登入體驗。

## <a name="next-steps"></a>後續步驟

- [**快速入門**](how-to-connect-sso-quick-start.md) - 開始使用 Azure AD 無縫 SSO。
- [**部署方案**](https://aka.ms/AuthenticationDeploymentPlan) - 逐步部署方案。
- [**技術性深入探討**](how-to-connect-sso-how-it-works.md) - 了解這項功能的運作方式。
- [**常見問題集**](how-to-connect-sso-faq.md) - 常見問題集的答案。
- [**疑難排解**](tshoot-connect-sso.md) - 了解如何解決此功能的常見問題。
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) - 用於提出新的功能要求。

