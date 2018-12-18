---
title: 協助進行身分識別管理的 Azure 安全性功能 | Microsoft Docs
description: " 本文對協助進行身分識別管理的 Azure 安全性功能提供核心的概觀。 Microsoft 身分識別與存取管理解決方案會協助 IT 保護應用程式和跨公司資料中心和到雲端的存取，啟用其他層級的驗證，例如 Multi-Factor Authentication 和條件式存取原則。 "
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 5aa0a7ac-8f18-4ede-92a1-ae0dfe585e28
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: terrylan
ms.openlocfilehash: cbf6ac736db9935f5ec52fa4507dab1e56bcde43
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38611182"
---
# <a name="azure-identity-management-security-overview"></a>Azure 身分識別管理安全性概觀
Microsoft 身分識別和存取管理解決方案可協助 IT 人員保護從公司資料中心與雲端存取應用程式和資源的權限。 這種保護機制能提供其他層級的驗證方式，例如多重要素驗證和條件式存取原則。 透過進階的安全性報告、稽核和警示來監視可疑活動，有助於緩解潛在的安全性問題。 [Azure Active Directory Premium](../active-directory/active-directory-editions.md) 可以提供數千個雲端軟體即服務 (SaaS) 應用程式的單一登入 (SSO) 及存取您在內部部署執行的 Web 應用程式。

善用 Azure Active Directory (Azure AD) 的安全性優點，您便可以：

* 為您的混合式企業中的每位使用者建立和管理單一身分識別，讓使用者、群組和裝置保持同步。
* 提供您的應用程式 (包括數千個預先整合的 SaaS 應用程式) 的 SSO 存取。
* 藉由同時對內部部署和雲端應用程式強制以規則為基礎的 Multi-Factor Authentication，啟用應用程式存取安全性。
* 透過 Azure AD 應用程式 Proxy 佈建對內部部署 Web 應用程式的安全遠端存取。

本文的目的是對協助進行身分識別管理的 Azure 安全性功能提供核心的概觀。 我們也會提供文章的連結，更詳細說明每個功能，以讓您深入了解。  

本文著重於下列核心 Azure 身分識別管理功能︰

* 單一登入
* 反向 proxy
* Multi-Factor Authentication
* 安全性監視、警示以及機器學習服務型報告
* 消費者身分識別與存取管理
* 裝置註冊
* Privileged Identity Management
* 身分識別保護
* 混合式身分識別管理

## <a name="single-sign-on"></a>單一登入
SSO 表示只要使用單一使用者帳戶登入，就能夠存取所有執行業務所需的應用程式和資源。 登入之後，您可以存取所有需要的應用程式，而不需要再驗證一次 (例如，輸入密碼)。

許多組織依賴 SaaS 應用程式，例如 Office 365、Box 和 Salesforce 來提升使用者生產力。 在過去，IT 人員必須在每個 SaaS 應用程式中個別建立並更新使用者帳戶，使用者則必須記住每個 SaaS 應用程式的密碼。

Azure AD 將內部部署的 Active Directory 環境延伸到雲端，讓使用者不只能夠使用主要的組織帳戶登入加入網域的裝置和公司資源，也能登入作業所需的所有 Web 和 SaaS 應用程式。

不只使用者不需要管理多組使用者名稱和密碼，還可根據自身的組織群組，以及其員工的狀態，自動佈建或解除佈建其應用程式的存取權。 Azure AD 引進了安全性和存取管理控制措施，可讓您集中管理所有 SaaS 應用程式的使用者存取權。

深入了解：

* [單一登入概觀](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../active-directory/manage-apps/what-is-single-sign-on.md)
* [整合 Azure Active Directory 單一登入與 SaaS 應用程式](../active-directory/manage-apps/configure-single-sign-on-portal.md)

## <a name="reverse-proxy"></a>反向 proxy
Azure AD 應用程式 Proxy 可讓您在私人網路內發佈內部部署應用程式 (例如 [SharePoint](https://support.office.com/article/What-is-SharePoint-97b915e6-651b-43b2-827d-fb25777f446f?ui=en-US&rs=en-US&ad=US) 網站、[Outlook Web 應用程式](https://technet.microsoft.com/library/jj657718.aspx)和 [IIS](http://www.iis.net/) 型應用程式)，並提供網路外部使用者的安全存取。 應用程式 Proxy 不僅針對多種內部部署 Web 應用程式提供遠端存取與 SSO，同時還提供 Azure AD 支援的數千個 SaaS 應用程式。 員工可以從家裡使用自己的裝置登入您的應用程式，還可透過這個雲端 Proxy 進行驗證。

深入了解：

* [啟用 Azure AD 應用程式 Proxy](../active-directory/manage-apps/application-proxy-enable.md)
* [使用 Azure AD 應用程式 Proxy 發佈應用程式](../active-directory/active-directory-application-proxy-publish.md)
* [使用應用程式 Proxy 進行單一登入](../active-directory/manage-apps/application-proxy-configure-single-sign-on-with-kcd.md)
* [使用條件式存取](../active-directory/manage-apps/application-proxy-integrate-with-sharepoint-server.md)

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication
Azure Multi-Factor Authentication 是需要使用多種驗證方法，並在使用者登入和交易中新增重要的第二層安全性的驗證方法。 Multi-Factor Authentication 有助於保護對資料與應用程式的存取，同時滿足使用者對簡單登入程序的需求。 它可以透過一些驗證選項：例如電話、文字訊息，或行動應用程式通知或驗證代碼，以及第三方 OAuth 權杖，來提供強大的驗證功能。

深入了解：

* [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)
* [什麼是 Azure Multi-Factor Authentication？](../active-directory/authentication/multi-factor-authentication.md)
* [Azure Multi-Factor Authentication 的作用](../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>安全性監視、警示以及機器學習服務型報告
安全性監視、警示以及機器學習式報告，會識別不一致的存取模式，可以協助您保護企業安全。 您可以使用 Azure AD 的存取和使用情況報告來了解貴組織的目錄完整性和安全性。 利用此資訊，目錄管理員更能夠判斷可能發生安全性風險的位置，以便適當地規劃來減輕這些風險。

在 Azure 入口網站中，報告會按照下列類別加以區分：

* **異常報告**：包含我們發現異常的登入事件。 我們的目標在於使您注意這類活動，並讓您決定事件是否可疑。
* **整合式應用程式報告**：可供深入了解雲端應用程式在組織中的使用方式。 Azure AD 提供與數千個雲端應用程式的整合。
* **錯誤報告**：指出將帳戶佈建至外部應用程式時可能發生的錯誤。
* **使用者特定報告**：顯示特定使用者的裝置登入活動資料。
* **活動記錄檔**：包含過去 24 小時、過去 7 天或過去 30 天內所有稽核事件的記錄，以及群組活動變更、密碼重設和登錄活動。

深入了解：

* [檢視存取和使用情況報告](../active-directory/active-directory-view-access-usage-reports.md)
* [開始使用 Azure Active Directory 報告](../active-directory/active-directory-reporting-getting-started.md)
* [Azure Active Directory 報告指南](../active-directory/active-directory-reporting-guide.md)

## <a name="consumer-identity-and-access-management"></a>消費者身分識別與存取管理
Azure AD B2C 是高可用性的全域身分識別管理服務，可用於處理數億個身分識別的消費者端應用程式。 此服務可跨行動及 Web 平台進行整合。 可自訂的使用經驗讓您的消費者可以使用其現有的社交帳戶，或是建立新的認證來登入您所有的應用程式。

在過去，應用程式開發人員若想要註冊並讓客戶登入其應用程式，他們會編寫自己的程式碼。 而且他們會使用內部部署資料庫或系統來儲存使用者名稱和密碼。 Azure AD B2C 為您的組織提供更理想的作法，透過安全且以標準為基礎的平台以及更大的可延伸原則組合，協助將取用者身分識別管理整合至應用程式。

當您使用 Azure AD B2C 時，您的取用者可以使用現有的社交帳戶 (Facebook、Google、Amazon、LinkedIn) 註冊應用程式，或是建立新的認證 (電子郵件地址與密碼，或使用者名稱與密碼)。

深入了解：

* [什麼是 Azure Active Directory B2C？](https://azure.microsoft.com/services/active-directory-b2c/)
* [Azure Active Directory B2C 預覽：在您的應用程式中註冊與登入取用者](../active-directory-b2c/active-directory-b2c-overview.md)
* [Azure Active Directory B2C 預覽：應用程式類型](../active-directory-b2c/active-directory-b2c-apps.md)

## <a name="device-registration"></a>裝置註冊
Azure AD 裝置註冊是裝置型[條件式存取](../active-directory/active-directory-conditional-access-device-registration-overview.md)案例的基礎。 當裝置已註冊時，Azure AD 裝置註冊會在使用者登入時，對裝置提供用來驗證裝置的身分識別。 然後已驗證的裝置和裝置的屬性即可用來對裝載於雲端和內部部署的應用程式，強制執行條件式存取原則。

與 Intune 這類的行動裝置管理解決方案結合時，將會以裝置的其他相關資訊更新 Azure AD 中的裝置屬性。 接著您就可以建立條件式存取規則，強制讓裝置的存取符合您的安全性和相容性標準。

深入了解：

* [開始使用 Azure AD 裝置註冊](../active-directory/active-directory-conditional-access-device-registration-overview.md)
* [自動向 Azure AD 註冊加入網域的 Windows 裝置](../active-directory/active-directory-conditional-access-automatic-device-registration.md)
* [設定讓已加入網域的 Windows 裝置自動向 Azure AD 註冊](../active-directory/active-directory-conditional-access-automatic-device-registration-setup.md)

## <a name="privileged-identity-management"></a>Privileged Identity Management
有了 Azure AD Privileged Identity Management ，您便可管理、控制和監視特殊權限身分識別，以及存取 Azure AD 和 Office 365 與 Microsoft Intune 等其他 Microsoft Online 服務的資源。

使用者有時候需要在 Azure 或 Office 365 資源或其他 SaaS 應用程式中執行特殊權限的作業。 這類需求通常表示組織必須授與使用者永久的 Azure AD 特殊權限存取權。 這種存取權會提高雲端資源的安全性風險，因為組織無法滴水不漏地監視這些使用者利用其管理員權限的所作所為。 此外，如果具特殊存取權限的使用者帳戶遭到入侵，這個缺口就會影響組織的整體雲端安全性。 Azure AD Privileged Identity Management 有助於降低此風險。

有了 Azure AD Privileged Identity Management，您便可以：

* 查看哪些使用者是 Azure AD 管理員。
* 視需要啟用 Office 365 和 Intune 等 Microsoft 服務的 just-in-time (JIT) 系統管理存取權限。
* 取得有關系統管理員存取記錄與系統管理員指派變更的報告。
* 取得有關特殊權限角色存取的警示。

深入了解：

* [Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md)
* [Azure AD Privileged Identity Management 中的角色](../active-directory/privileged-identity-management/pim-roles.md)
* [Azure AD Privileged Identity Management：如何新增或移除使用者角色](../active-directory/privileged-identity-management/pim-how-to-add-role-to-user.md)

## <a name="identity-protection"></a>身分識別保護
Azure AD Identity Protection 是一項安全性服務，可供整合檢視會影響組織身分識別的風險事件和潛在弱點。 Identity Protection 會利用現有的 Azure AD 異常偵測功能，這些功能可透過 Azure AD 異常活動報告取得。 Identity Protection 也引進新的風險事件類型，可以即時偵測異常行為。

深入了解：

* [Azure AD Identity Protection](../active-directory/active-directory-identityprotection.md)
* [第 9 頻道：Azure AD 和身分識別展示：Identity Protection 預覽](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

## <a name="hybrid-identity-management"></a>混合式身分識別管理
Microsoft 的身分識別方法可跨越內部部署和雲端架構功能，建立單一使用者身分識別以用於所有資源的驗證和授權，不論位於何處。

深入了解：

* [混合式身分識別白皮書](http://download.microsoft.com/download/D/B/A/DBA9E313-B833-48EE-998A-240AA799A8AB/Hybrid_Identity_White_Paper.pdf)
* [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)
* [Azure AD 小組部落格](https://blogs.technet.microsoft.com/ad/)
