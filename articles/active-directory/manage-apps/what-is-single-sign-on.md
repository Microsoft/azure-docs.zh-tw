---
title: 什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？ | Microsoft Docs
description: 您可以使用 Azure Active Directory 來單一登入到您進行工作所需的所有 SaaS 和 Web 應用程式。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: users-groups-roles
ms.workload: identity
ms.topic: conceptual
ms.date: 07/16/2018
ms.author: barbkess
ms.reviewer: asmalser
ms.custom: it-pro
ms.openlocfilehash: 1a7ecbe42857e522785d5919d46b783feae4caeb
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494120"
---
# <a name="what-is-application-access-and-single-sign-on-with-azure-active-directory"></a>什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？
單一登入表示只要使用單一使用者帳戶登入，就能夠存取所有進行工作所需的應用程式和資源。 登入之後，您可以存取所有需要的應用程式，而不需要再驗證一次 (例如，輸入密碼)。

許多組織依賴軟體即服務 (SaaS) 應用程式，例如 Office 365、Box 和 Salesforce 來提升使用者生產力。 在過去，IT 人員必須在每個 SaaS 應用程式中個別建立並更新使用者帳戶，使用者則必須記住每個 SaaS 應用程式的密碼。

Azure Active Directory 將內部部署的 Active Directory 延伸到雲端，讓使用者不只能夠使用主要的組織帳戶登入加入網域的裝置和公司資源，也能登入工作所需的所有 Web 和 SaaS 應用程式。

所以不只使用者不需要管理多組使用者名稱和密碼，還可根據其組織的群組成員，以及其身為員工的狀態，自動佈建或解除佈建其應用程式的存取權。 Azure Active Directory 引進了安全性和存取管理控制，可讓您集中管理所有 SaaS 應用程式的使用者存取權。

Azure AD 能輕鬆整合到許多現今熱門的 SaaS 應用程式。它提供身分識別與存取管理，並可讓使用者直接單一登入應用程式，或從入口網站 (例如 Office 365 或 Azure AD 存取面板) 探索並啟動應用程式。

整合的架構包含下列四個主要建置區塊：

* 單一登入可讓使用者根據其在 Azure AD 中的組織帳戶存取其 SaaS 應用程式。 單一登入可讓使用者使用單一組織帳戶向應用程式進行驗證。
* 使用者佈建可讓使用者根據在 Windows Server Active Directory 和/或 Azure AD 中所做的變更，佈建和解除佈建到目標 SaaS。 佈建的帳戶透過單一登入驗證之後，可授權使用者使用應用程式。
* 在 Azure 入口網站中的集中式應用程式存取管理，可以單點存取 SaaS 應用程式和進行管理，並且可以委派應用程式存取權的決策和核准給組織中的任何人。
* 統一報告和監視 Azure AD 中的使用者活動

## <a name="how-does-single-sign-on-with-azure-active-directory-work"></a>單一登入如何搭配 Azure Active Directory 運作？
當使用者登入應用程式時，會經歷一個驗證程序，需要證明他們的身分。 如果沒有單一登入，此驗證程序通常是藉由輸入儲存在應用程式中的密碼進行，而使用者必須知道此密碼。

Azure AD 支援三種不同的方式登入應用程式：

* **同盟單一登入**可讓應用程式重新導向至 Azure AD 進行使用者驗證，而不是提示使用者自己的密碼。 支援如 SAML 2.0、WS-同盟或 OpenID Connect 等通訊協定的應用程式都支援同盟單一登入，而且這是最豐富的單一登入模式。
* **密碼單一登入**可以使用網頁瀏覽器延伸或行動應用程式，安全儲存應用程式的密碼以及重新執行。 密碼型單一登入會使用應用程式提供的現有程序，但是讓系統管理員能夠管理密碼，而且使用者不需要知道密碼。
* **現有單一登入**可讓 Azure AD 運用應用程式已設定的任何現有單一登入，但可讓這些應用程式連結至 Office 365 或 Azure AD 存取面板入口網站，當其中有應用程式啟動時，Azure AD 中也會有額外的報告。

一旦使用者通過應用程式的驗證，他們在應用程式中也必須具有佈建的帳戶記錄，以便告訴應用程式在應用程式內部具有權限和存取層級的位置。 佈建此帳戶記錄可以自動執行，或是在提供使用者單一登入存取之前由系統管理員手動執行。

 以下是有關這些單一登入模式和佈建的詳細資訊。

### <a name="federated-single-sign-on"></a>同盟單一登入
同盟單一登入可讓 Azure AD 使用 Azure AD 的使用者帳戶資訊，將組織中的使用者自動登入協力廠商 SaaS 應用程式。

在此情節中，如果您已經登入 Azure AD，而且您想要存取由協力廠商 SaaS 應用程式控制的資源，同盟就可以讓使用者不需要重新驗證。

Azure AD 可以對支援 SAML 2.0、WS-同盟或 OpenID Connect 通訊協定的應用程式支援同盟單一登入。

亦請參閱： [管理同盟單一登入的憑證](manage-certificates-for-federated-single-sign-on.md)

### <a name="password-based-single-sign-on"></a>密碼單一登入
設定密碼單一登入可讓您組織中的使用者，使用協力廠商 SaaS 應用程式的使用者帳戶資訊，由 Azure AD 自動登入協力廠商的 SaaS 應用程式。 當您啟用此功能時，Azure AD 會收集並安全地儲存使用者帳戶資訊和相關的密碼。

凡是任何具有 HTML 登入頁面的雲端應用程式，Azure AD 都可以支援密碼單一登入。 AAD 會使用自訂的瀏覽器外掛程式，從目錄安全地擷取應用程式認證 (例如使用者名稱和密碼)，將登入程序自動化，並代替使用者將這些認證輸入應用程式的登入頁面。 有兩個使用案例：

1. **系統管理員管理認證** – 系統管理員可以建立和管理應用程式認證，並將這些認證指派給需要存取應用程式的使用者或群組。 在這些情況下，使用者不需要知道認證，但只要在其存取面板中或透過提供的連結按一下認證，仍然可以單一登入存取應用程式。 此程序既可讓系統管理員對認證進行週期管理，對終端使用者也很方便，因為終端使用者不需要記住或管理應用程式專用的密碼。 使用者在自動登入程序期間的認證會經過模糊處理，不過使用者使用 Web 偵錯工具在技術上還是可以找到這些認證，因此使用者和系統管理員應該遵循與使用者直接提出認證時相同的安全性原則。 當提供的帳戶存取權是由許多使用者共用時，例如社交媒體或文件共用應用程式，系統管理員提供的認證會很有用。
2. **使用者管理認證** – 系統管理員可以指派應用程式給使用者或群組，並允許使用者在存取面板中第一次存取應用程式時直接輸入自己的認證。 這樣可以方便使用者，每次存取應用程式時，不需要一再輸入應用程式專用的密碼。 使用者可以繼續管理他們的密碼，視需要更新或刪除它們。 這個使用案例也可以用來做為系統性管理認證的跳板，系統管理員可以在日後設定應用程式的新認證，而不需要變更使用者的應用程式存取體驗。

在上述兩種情況下，認證都會以加密的狀態儲存在目錄中，而且在自動登入程序期間只會透過 HTTPS 傳遞。 使用密碼單一登入，Azure AD 就能對無法支援同盟通訊協定的應用程式提供方便的身分識別存取管理解決方案。

密碼 SSO 依賴瀏覽器延伸模組從 Azure AD 安全地擷取應用程式和使用者特定資訊，並將它套用至服務。 Azure AD 支援的大多數協力廠商 SaaS 應用程式都支援這項功能。

若是密碼 SSO，則使用者的瀏覽器可以是：
* Internet Explorer 11 - 在 Windows 7 或更新版本上
* Windows 10 Anniversary Edition 或更新版本上的 Edge 
* Chrome - 在 Windows 7 或更新版本，和在 MacOS X 或更新版本上
* Firefox 26.0 或更新版本 - 在 Windows XP SP2 或更新版本，和在 Mac OS X 10.6 或更新版本上

### <a name="existing-single-sign-on"></a>現有單一登入
設定應用程式的單一登入時，Azure 入口網站提供「現有單一登入」的第三個選項。 這個選項只允許系統管理員建立應用程式的連結，並將連結放在選定使用者的存取面板上。

例如，如果有一個應用程式設定為使用 Active Directory Federation Services 2.0 來驗證使用者，系統管理員可以使用 [現有單一登入] 選項在存取面板上建立應用程式的連結。 當使用者存取此連結時，會使用 Active Directory Federation Services 2.0，或由應用程式提供的任何現有單一登入解決方案來驗證使用者。

### <a name="user-provisioning"></a>使用者佈建
對於選取的應用程式，Azure AD 使用您的 Windows Server Active Directory 或 Azure AD 身分識別資訊，可從 Azure 入口網站內在協力廠商 SaaS 應用程式中進行帳戶的自動化使用者佈建和解除佈建。 當使用者在 Azure AD 中獲得其中一個應用程式的權限時，就會在目標 SaaS 應用程式中自動建立 (佈建) 帳戶。

當使用者被刪除，或其資訊在 Azure AD 中有所變更時，這些變更也會反映在 SaaS 應用程式中。 這表示，設定自動化的身分識別週期管理，可讓系統管理員控制和提供 SaaS 應用程式的自動化佈建和解除佈建。 在 Azure AD 中，使用者佈建已啟用此自動化身分識別週期管理。

若要深入了解，請參閱 [SaaS 應用程式的自動化使用者佈建和解除佈建](../active-directory-saas-app-provisioning.md)

## <a name="get-started-with-the-azure-ad-application-gallery"></a>開始使用 Azure AD 應用程式庫
準備好開始了嗎？ 若要在 Azure AD 和您的組織所使用的 SaaS 應用程式之間部署單一登入，請遵循這些指導方針。

### <a name="using-the-azure-ad-application-gallery"></a>使用 Azure AD 應用程式庫
[Azure Active Directory 應用程式資源庫](https://azure.microsoft.com/marketplace/active-directory/all/) 提供一份已知能支援單一登入搭配 Azure Active Directory 的應用程式清單。

![Azure 線上應用程式庫](media/what-is-single-sign-on/onlineappgallery.png)

以下是尋找應用程式支援哪些功能的一些提示：

* Azure AD 支援自動佈建和解除佈建 [Azure Active Directory 應用程式資源庫](https://azure.microsoft.com/marketplace/active-directory/all/)中所有「精選的」應用程式。
* 在 [這裡](http://social.technet.microsoft.com/wiki/contents/articles/20235.azure-active-directory-application-gallery-federated-saas-apps.aspx)則可以找到特別支援使用如 SAML，WS-同盟或 OpenID Connect 通訊協定之同盟單一登入的同盟應用程式清單。

一旦您找到您的應用程式，就可以遵循應用程式資源庫和 Azure 入口網站中的逐步指示，啟用單一登入。

### <a name="application-not-in-the-gallery"></a>不在資源庫中的應用程式呢？
如果 Azure AD 應用程式庫中找不到您的應用程式，您可以選擇：

* **新增您正在使用但未列出的應用程式** - 使用 Azure 入口網站內應用程式資源庫中的自訂類別目錄，連線您的組織正在使用但未列出的應用程式。 您可以加入支援 SAML 2.0 的任何應用程式做為同盟應用程式，或者加入具有 HTML 登入頁面的任何應用程式做為密碼 SSO 應用程式。 如需詳細資訊，請參閱 [加入自己的應用程式](../application-config-sso-how-to-configure-federated-sso-non-gallery.md)一文。
* **新增您正在開發的專屬應用程式** - 如果您自己開發了應用程式，請依照 Azure AD 開發人員文件中的指導方針來實作同盟單一登入或使用 Azure AD 圖形 API 的佈建。 如需詳細資訊，請參閱這些資源：
  
  * [Azure AD 的驗證案例](../develop/authentication-scenarios.md)
  * [https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet)
  * [https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet)
  * [https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore](https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore)
* **要求應用程式整合** - 使用 [Azure AD 意見反應論壇](https://feedback.azure.com/forums/169401-azure-active-directory/)要求所需應用程式的支援。

### <a name="using-the-azure-portal"></a>使用 Azure 入口網站
您可以使用 Azure 入口網站中的 Active Directory 延伸模組來設定應用程式單一登入。 首先，您必須從入口網站的 [Active Directory] 區段中選取目錄：

![](./media/what-is-single-sign-on/azuremgmtportal.png)

若要管理您的協力廠商 SaaS 應用程式，您可以切換至所選目錄的 [應用程式] 索引標籤。 這個檢視可讓系統管理員：

* 新增 Azure AD 資源庫的應用程式，以及您正在開發的應用程式
* 刪除整合的應用程式
* 管理已經整合的應用程式

協力廠商 SaaS 應用程式的一般管理工作有：

* 向 Azure AD 啟用單一登入、使用密碼 SSO，或做為目標 SaaS、同盟 SSO (如果適用)
* (選擇性) 啟用使用者佈建來佈建和解除佈建使用者 (身分識別週期管理)
* 對於啟用 
* 使用者佈建的應用程式，選取哪些使用者可以存取該應用程式

對於支援同盟單一登入的資源庫應用程式，通常需要您提供其他組態設定，例如用來建立協力廠商應用程式與 Azure AD 之間同盟信任的憑證和中繼資料。 組態精靈會引導您完成細節，並且提供特定資料和指示讓您輕鬆存取 SaaS 應用程式。

對於支援自動使用者佈建的資源庫應用程式，您需要賦予 Azure AD 權限來管理您在 SaaS 應用程式中的帳戶。 至少，您需要提供 Azure AD 在向目標應用程式進行驗證時應該使用的認證。 是否需要提供額外的組態設定，取決於應用程式的要求。

## <a name="deploying-azure-ad-integrated-applications-to-users"></a>對使用者部署 Azure AD 整合應用程式
Azure AD 提供幾種可自訂的方式，來對您組織中的使用者部署應用程式：

* Azure AD 存取面板
* Office 365 應用程式啟動程式
* 直接登入同盟應用程式
* 同盟、密碼或現有應用程式的深層連結

您選擇要在組織中部署哪一種方法由您自行決定。

### <a name="azure-ad-access-panel"></a>Azure AD 存取面板
位於 https://myapps.microsoft.com 的「存取面板」是一種 Web 入口網站，可讓 Azure Active Directory 中擁有組織帳戶的使用者檢視和啟動 Azure AD 系統管理員已授與他們存取權的雲端應用程式。 如果您是 [Azure Active Directory Premium](https://azure.microsoft.com/pricing/details/active-directory/) 的使用者，也可以透過 [存取面板] 使用自助式群組管理功能。

![Azure AD 存取面板](media/what-is-single-sign-on/azure-ad-access-panel.png)

存取面板與 Azure 入口網站不同，使用者不需要具備 Azure 訂用帳戶或 Office 365 訂用帳戶。

如需有關 Azure AD 存取面板的詳細資訊，請參閱 [存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

### <a name="office-365-application-launcher"></a>Office 365 應用程式啟動程式
針對已部署 Office 365 的組織，透過 Azure AD 指派給使用者的應用程式也會出現在位於 https://portal.office.com/myapps 的 Office 365 入口網站中。 這對於組織中的使用者來說，能夠啟動應用程式又不需要使用第二個入口網站，非常簡單而且方便，建議使用 Office 365 的組織採取這個應用程式啟動解決方案。

![](./media/what-is-single-sign-on/officeapphub.png)

如需有關 Office 365 應用程式啟動程式的詳細資訊，請參閱 [讓您的應用程式出現在 Office 365 應用程式啟動程式中](https://msdn.microsoft.com/office/office365/howto/connect-your-app-to-o365-app-launcher)。

### <a name="direct-sign-on-to-federated-apps"></a>直接登入同盟應用程式
大部分支援 SAML 2.0、WS-同盟或 OpenID Connect 的同盟應用程式也支援使用者在應用程式啟動，然後再透過 Azure AD 的自動重新導向或按一下連結登入。 這稱為服務提供者起始的登入，Azure AD 應用程式資源庫中大部分的同盟應用程式都支援這個方式 (請參閱 Azure 入口網站中應用程式的單一登入設定精靈連結的文件來了解詳細資訊)。

![](./media/what-is-single-sign-on/workdaymobile.png)

### <a name="direct-sign-on-links-for-federated-password-based-or-existing-apps"></a>同盟、密碼或現有應用程式的直接登入連結
Azure AD 也支援對支援密碼單一登入、現有單一登入，以及任何形式的同盟單一登入的個別應用程式使用直接單一登入連結。

這些連結是特別撰寫的 URL，會透過 Azure AD 登入程序針對特定應用程式傳送給使用者，使用者不需要從 Azure AD 存取面板或 Office 365 啟動。 您可以在 Azure 入口網站 [Active Directory] 區段中任何預先整合之應用程式的 [儀表板] 索引標籤下，找到這些單一登入 URL，如以下螢幕擷取畫面所示。

![](./media/what-is-single-sign-on/deeplink.png)

您可以複製這些連結，然後貼到任何您想要提供選取應用程式登入連結的位置。 可以是在電子郵件中，或是任何您已設定使用者應用程式存取權的自訂網頁型入口網站中。 以下是 Azure AD 直接單一登入 Twitter 的 URL 範例：

`https://myapps.microsoft.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced`

類似於存取面板的組織特定 URL，您可以在 myapps.microsoft.com 網域之後加入其中一個作用中或經過驗證的網域做為您的目錄，進一步自訂這個 URL。 這樣可以確保使用者不需要先輸入其使用者識別碼，登入頁面就可以立即載入任何組織品牌：

`https://myapps.microsoft.com/contosobuild.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced`

當授權的使用者按一下其中一個應用程式特定連結時，他們會先看到其組織登入頁面 (假設他們尚未登入)，登入之後會重新導向至該應用程式而不會先停在存取面板。 如果使用者遺漏存取應用程式所需的先決條件，例如密碼單一登入瀏覽器延伸，則連結將會提示使用者安裝遺漏的延伸。 如果應用程式的單一登入組態有變更，連結 URL 也會維持不變。

這些連結使用與存取面板和 Office 365 相同的存取控制機制，只有在 Azure 入口網站中已指派給應用程式的使用者或群組能夠順利通過驗證。 不過，任何未經授權的使用者都會看到一個訊息，說明他們未獲得存取權，且會獲得一個載入存取面板的連結，以檢視可存取的應用程式。

## <a name="related-articles"></a>相關文章
* [Article Index for Application Management in Azure Active Directory (Azure Active Directory 中應用程式管理的文件索引)](../active-directory-apps-index.md)
* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](../saas-apps/tutorial-list.md)
* [設定 Cloud Discovery](/cloud-app-security/set-up-cloud-discovery)
* [管理應用程式存取簡介](what-is-access-management.md)
* [比較 Azure AD 中管理外部身分識別的功能](../active-directory-b2b-compare-b2c.md)


