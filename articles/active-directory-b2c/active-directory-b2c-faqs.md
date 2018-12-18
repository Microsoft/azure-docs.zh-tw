---
title: Azure Active Directory B2C 的常見問題集 | Microsoft Docs
description: Azure Active Directory B2C 常見問題集 (FAQ)。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6b5cc1eff5c6d080854754ec5b3b62068f1cbb9c
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39264710"
---
# <a name="azure-ad-b2c-frequently-asked-questions-faq"></a>Azure AD B2C：常見問題集 (FAQ) 
此頁面會回答有關 Azure Active Directory (Azure AD) B2C 的常見問題。 請隨時回來查看最新消息。

### <a name="why-cant-i-access-the-azure-ad-b2c-extension-in-the-azure-portal"></a>為什麼無法存取 Azure 入口網站中的 Azure AD B2C 擴充功能？
Azure AD 擴充功能無法運作有兩個常見原因。  Azure AD B2C 要求您在目錄中的使用者角色是全域管理員。  如果您認為自己具有存取權，請連絡您的管理員。  如果您有全域管理員權限，請確定您是在 Azure AD B2C 目錄中，而非 Azure Active Directory 目錄。  可至此查看[教學課程：建立 Azure Active Directory B2C 租用戶](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant)。

### <a name="can-i-use-azure-ad-b2c-features-in-my-existing-employee-based-azure-ad-tenant"></a>我可以在以員工為主的現有 Azure AD 租用戶中使用 Azure AD B2C 功能嗎？
Azure AD 和 Azure AD B2C 為個別的產品供應項目，無法共存於同一個租用戶。  一個 Azure AD 租用戶代表一個組織。  一個 Azure AD B2C 租用戶代表一組要用於信賴憑證者應用程式的身分識別。  透過自訂原則 (處於公開預覽狀態)，Azure AD B2C 可以與 Azure AD 結成同盟，所以能夠驗證組織中的員工。

### <a name="can-i-use-azure-ad-b2c-to-provide-social-login-facebook-and-google-into-office-365"></a>我可以使用 Azure AD B2C 來提供 Office 365 的社交登入 (Facebook 和 Google+) 嗎？
Azure AD B2C 無法用來驗證 Microsoft Office 365 的使用者。  Azure AD 是 Microsoft 為了管理員工存取 SaaS 應用程式而提出的解決方案，具有專為此用途而設計的功能，例如授權和條件式存取。  Azure AD B2C 提供身分識別和存取管理平台來建置 web 和行動應用程式。  當 Azure AD B2C 設定為與 Azure AD 租用戶結成同盟時，Azure AD 租用戶會管理員工如何存取依賴 Azure AD B2C 的應用程式。

### <a name="what-are-local-accounts-in-azure-ad-b2c-how-are-they-different-from-work-or-school-accounts-in-azure-ad"></a>Azure AD B2C 中的本機帳戶是什麼？ 與 Azure AD 中的工作或學校帳戶有何不同？
在 Azure AD 租用戶中，屬於租用戶的使用者是以 `<xyz>@<tenant domain>` 格式的電子郵件地址登入。  `<tenant domain>` 是租用戶中已驗證的其中一個網域，或初始的 `<...>.onmicrosoft.com` 網域。 這種類型的帳戶就是工作或學校帳戶。

在 Azure AD B2C 租用戶中，大部分應用程式都希望使用者以任意的電子郵件地址登入 (例如 joe@comcast.net、bob@gmail.com、sarah@contoso.com 或 jim@live.com)。 這種類型的帳戶就是本機帳戶。  我們也支援使用任意的使用者名稱作為本機帳戶 (例如，joe、bob、sarah 或 jim)。 在 Azure 入口網站中設定 Azure AD B2C 的識別提供者時，您可以從這兩個本機帳戶類型中選擇一個。 在 Azure AD B2C 租用戶中按一下 [識別提供者]，然後在 [本機帳戶] 下方選取 [使用者名稱]。 

應用程式的使用者帳戶一律透過註冊原則、註冊或登入原則，或使用 Azure AD 圖形 API 建立。 在 Azure 入口網站中建立的使用者帳戶只能用來管理租用戶。

### <a name="which-social-identity-providers-do-you-support-now-which-ones-do-you-plan-to-support-in-the-future"></a>你們現在支援哪些社交身分識別提供者？ 你們打算在未來支援哪些？
我們目前支援 Facebook、Google+、LinkedIn、Amazon、Twitter (預覽)、WeChat (預覽)、Weibo (預覽) 和 QQ (預覽)。 根據客戶需求，我們將會增加支援其他熱門的社交身分識別提供者。

Azure AD B2C 也新增了[自訂原則](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview-custom)的支援。  這些[自訂原則](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview-custom)可讓開發人員使用任何識別提供者建立自己的原則，可支援 [OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) 或 SAML。 

查看我們的[自訂原則入門套件](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack)，開始使用自訂原則。

### <a name="can-i-configure-scopes-to-gather-more-information-about-consumers-from-various-social-identity-providers"></a>我可以設定範圍，以便從各種社交身分識別提供者收集取用者的詳細資訊嗎？
否，但這項功能已在我們的規劃中。 我們支援的一組社交身分識別提供者所使用的預設範圍如下：

* Facebook: email
* Google+: email
* Microsoft 帳戶︰openid 電子郵件設定檔
* Amazon: profile
* LinkedIn: r_emailaddress, r_basicprofile

### <a name="does-my-application-have-to-be-run-on-azure-for-it-work-with-azure-ad-b2c"></a>我的應用程式必須在 Azure 上執行，才能使用 Azure AD B2C 嗎？
否，您可以將應用程式裝載於任何地方 (雲端或內部部署)。 只要能夠在公開存取的端點上傳送和接收 HTTP 要求，就可以與 Azure AD B2C 互動。

### <a name="i-have-multiple-azure-ad-b2c-tenants-how-can-i-manage-them-on-the-azure-portal"></a>我有多個 Azure AD B2C 租用戶。 如何在 Azure 入口網站上管理它們？
在 Azure 入口網站的左側功能表中開啟 'Azure AD B2C' 之前，您必須切換到需要管理的目錄。  在 Azure 入口網站右上角按一下您的身分識別來切換目錄，然後選擇出現在下拉式清單中的目錄。  如需逐步映像，請參閱[瀏覽至 Azure AD B2C 設定](active-directory-b2c-app-registration.md#navigate-to-b2c-settings)。

### <a name="how-do-i-customize-verification-emails-the-content-and-the-from-field-sent-by-azure-ad-b2c"></a>我如何自訂 Azure AD B2C 傳送的驗證電子郵件 (內容和 [寄件者:] 欄位)？
您可以使用 [公司商標功能](../active-directory/fundamentals/customize-branding.md) 自訂驗證電子郵件的內容。 明確地說，您可以自訂電子郵件的下列兩個元素：

* **橫幅標幟**：顯示在右下方。
* **背景色彩**：顯示在頂端。

    ![自訂驗證電子郵件的螢幕擷取畫面](./media/active-directory-b2c-faqs/company-branded-verification-email.png)

電子郵件簽章包含您第一次建立 Azure AD B2C 租用戶時提供的 Azure AD B2C 租用戶名稱。 您可以使用這些指示變更名稱：

1. 以全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
1. 開啟 [Azure Active Directory] 刀鋒視窗。
1. 按一下 [屬性] 索引標籤。
1. 變更 [名稱] 欄位。
1. 按一下頁面頂端的 [儲存]。

目前無法變更電子郵件中的 [從:] 欄位。 請在 [feedback.azure.com](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/15334335-fully-customizable-verification-emails) 上投票，表示您有興趣自訂驗證電子郵件的本文。

### <a name="how-can-i-migrate-my-existing-user-names-passwords-and-profiles-from-my-database-to-azure-ad-b2c"></a>我如何將現有的使用者名稱、密碼和設定檔從資料庫移轉至 Azure AD B2C？
您可以使用 Azure AD 圖形 API 來撰寫您的移轉工具。 如需詳細資訊，請參閱[使用者移轉指南](active-directory-b2c-user-migration.md)。

### <a name="what-password-policy-is-used-for-local-accounts-in-azure-ad-b2c"></a>Azure AD B2C 中用於本機帳戶的密碼原則為何？
Azure AD B2C 的本機帳戶密碼原則是以 Azure AD 的原則為基礎。 Azure AD B2C 的註冊、註冊或登入和密碼重設原則會使用「強式」密碼強度，而且不會讓任何密碼到期。 如需更詳細的資料，請閱讀 [Azure AD 中的密碼原則](https://msdn.microsoft.com/library/azure/jj943764.aspx) 。

### <a name="can-i-use-azure-ad-connect-to-migrate-consumer-identities-that-are-stored-on-my-on-premises-active-directory-to-azure-ad-b2c"></a>我可以使用 Azure AD Connect，將儲存於內部部署 Active Directory 的取用者身分識別移轉至 Azure AD B2C 嗎？
否，Azure AD Connect 不是設計來搭配 Azure AD B2C 一起使用。 請考慮使用[圖形 API](active-directory-b2c-devquickstarts-graph-dotnet.md) 來移轉使用者。  如需詳細資訊，請參閱[使用者移轉指南](active-directory-b2c-user-migration.md)。

### <a name="can-my-app-open-up-azure-ad-b2c-pages-within-an-iframe"></a>我的應用程式是否能在 iFrame 內開啟 Azure AD B2C 頁面？
否，基於安全性考量，無法在 iFrame 內開啟 Azure AD B2C 頁面。  我們的服務會與瀏覽器通訊以禁止 iFrame。  安全性社群整體和 OAUTH2 規格的建議是不要使用 iFrame 來提供身分識別體驗，因為會有點擊劫持風險。

### <a name="does-azure-ad-b2c-work-with-crm-systems-such-as-microsoft-dynamics"></a>Azure AD B2C 可以搭配 Microsoft Dynamics 之類的 CRM 系統一起使用嗎？
與 Microsoft Dynamics 365 入口網站的整合已可供使用。  請參閱[設定 Dynamics 365 入口網站，以使用 Azure AD B2C 進行驗證](https://docs.microsoft.com/dynamics365/customer-engagement/portals/azure-ad-b2c)。

### <a name="does-azure-ad-b2c-work-with-sharepoint-on-premises-2016-or-earlier"></a>Azure AD B2C 可以搭配 SharePoint 內部部署的 2016 或更舊版本一起使用嗎？
Azure AD B2C 不適用於 SharePoint 外部夥伴共用的情節。請改以參閱 [Azure AD B2B](https://cloudblogs.microsoft.com/enterprisemobility/2015/09/15/learn-all-about-the-azure-ad-b2b-collaboration-preview/)。

### <a name="should-i-use-azure-ad-b2c-or-b2b-to-manage-external-identities"></a>我應該使用 Azure AD B2C 或 B2B 來管理外部身分識別？
請閱讀這篇關於 [外部身分識別](../active-directory/active-directory-b2b-compare-external-identities.md) 的文章，以深入了解將適當的功能套用至外部身分識別的案例。

### <a name="what-reporting-and-auditing-features-does-azure-ad-b2c-provide-are-they-the-same-as-in-azure-ad-premium"></a>Azure AD B2C 提供哪些報告和稽核功能？ 它們與 Azure AD Premium 的功能相同嗎？
否，Azure AD B2C 不支援與 Azure AD Premium 相同的一組報告。 不過有許多共同點：

* **登入報告**會提供每次登入的記錄，並縮減詳細資料。
* **稽核報告**包含管理活動以及應用程式活動。 
* **使用報告**包含使用者數目、登入數目，以及 MFA 的磁碟區。 

### <a name="can-i-localize-the-ui-of-pages-served-by-azure-ad-b2c-what-languages-are-supported"></a>我可以將 Azure AD B2C 所提供的頁面 UI 當地語系化嗎？ 支援哪些語言？
可以！  請參閱[語言自訂](active-directory-b2c-reference-language-customization.md) (處於公開預覽狀態)。  我們提供 36 種語言的翻譯，您可以覆寫任何字串以符合您的需求。

### <a name="can-i-use-my-own-urls-on-my-sign-up-and-sign-in-pages-that-are-served-by-azure-ad-b2c-for-instance-can-i-change-the-url-from-loginmicrosoftonlinecom-to-logincontosocom"></a>我可以在 Azure AD B2C 提供的註冊與登入頁面上使用自己的 URL 嗎？ 例如，我可以將 URL 從 login.microsoftonline.com 變更為 login.contoso.com 嗎？
目前不支援。 但這項功能已在我們的規劃中。 在 Azure 入口網站的 [網域] 索引標籤中驗證您的網域，並無法達成此目標。

### <a name="how-do-i-delete-my-azure-ad-b2c-tenant"></a>如何刪除 Azure AD B2C 租用戶？
請遵循下列步驟來刪除 Azure AD B2C 租用戶︰

1. 遵循下列步驟，以在 Azure 入口網站上[瀏覽至 Azure AD B2C 設定](active-directory-b2c-app-registration.md#navigate-to-b2c-settings)。
1. 瀏覽至 [應用程式]、[識別提供者] 和 [所有原則]，並刪除其中所有的輸入。
1. 現在以訂用帳戶管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。 (使用您註冊 Azure 時所用的相同公司或學校帳戶，或相同的 Microsoft 帳戶。)
1. 切換至您想要刪除的 Azure AD B2C 租用戶。
2. 巡覽至左側的 [Active Directory] 功能表。
3. 選取 [使用者和群組]。
4. 逐一選取每個使用者 (不包括您目前登入的「訂用帳戶管理員」使用者身分)。 按一下頁面底部的 [刪除]，然後在出現提示時按一下 [是]。
5. 按一下 [應用程式註冊]。
6. 選取稱為 **b2c-extensions-app** 的應用程式。 按一下 [刪除]，然後在系統提示時按一下 [是]。
7. 選取 [概觀]。
8. 按一下 [刪除目錄]。 若要完成程序，請依照畫面上的指示。

### <a name="can-i-get-azure-ad-b2c-as-part-of-enterprise-mobility-suite"></a>我可以從 Enterprise Mobility Suite 中取得 Azure AD B2C 嗎？
否，Azure AD B2C 是隨用隨付的 Azure 服務，並不是 Enterprise Mobility Suite 的一部分。

### <a name="how-do-i-report-issues-with-azure-ad-b2c"></a>如何報告有關 Azure AD B2C 的問題？
請參閱 [提出 Azure Active Directory B2C 的支援要求](active-directory-b2c-support.md)。
