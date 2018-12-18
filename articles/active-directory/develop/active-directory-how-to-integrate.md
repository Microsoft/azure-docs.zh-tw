---
title: 如何與 Azure Active Directory 整合 | Microsoft 文件
description: 與 Azure Active Directory 整合的優點和所需資源指南。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: d13bba54-96bd-4b81-bee9-c8025ffa1648
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/27/2017
ms.author: celested
ms.reviewer: bryanla
ms.custom: aaddev
ms.openlocfilehash: fdc33f4009ce9434b241611851fecec451783a1a
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495334"
---
# <a name="integrating-with-azure-active-directory"></a>與 Azure Active Directory 整合
[!INCLUDE [active-directory-devguide](../../../includes/active-directory-devguide.md)]

Azure Active Directory 為組織提供企業等級的雲端應用程式身分識別管理。 Azure AD 整合提供使用者簡化的登入程序，並可協助您的應用程式符合 IT 原則。

## <a name="how-to-integrate"></a>如何整合
整合應用程式與 Azure AD 的方式有幾種。 充分利用適合您應用程式使用的這些案例。

### <a name="support-azure-ad-as-a-way-to-sign-in-to-your-application"></a>支援 Azure AD 作為登入應用程式的方式
**減少登入的不便，並降低支援成本。** 透過使用 Azure AD 來登入您的應用程式，您的使用者會少一個要記住的名稱和密碼。 身為開發人員，您會少一個要儲存及保護的密碼。 單單無需處理忘記密碼重設就可以省下大量時間與金錢。 Azure AD 支援部分世界上最受歡迎雲端應用程式 (包括 Office 365 和 Microsoft Azure) 的登入。 有來自數百萬個組織的數百萬名使用者，可能是您的使用者已經登入 Azure AD。 深入了解[新增 Azure AD 登入支援](authentication-scenarios.md)。

**簡化應用程式的註冊程序。**  註冊應用程式的過程中，Azure AD 會寄出使用者的基本資訊，讓您可以預先填入註冊表單，或者完全不需要用到。 使用者可以透過類似社交媒體和行動應用程式所使用的常見同意程序，使用其 Azure AD 帳戶來註冊應用程式。 任何使用者都可以註冊並登入已與 Azure AD 整合的應用程式，而無需 IT 人員的介入。 深入了解[註冊應用程式以進行 Azure AD 帳戶登入](../../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md)。

### <a name="browse-for-users-manage-user-provisioning-and-control-access-to-your-application"></a>瀏覽以尋找使用者、管理使用者佈建，及控制存取應用程式
**瀏覽以尋找目錄中的使用者。**  邀請他人或授與存取權限時，與其要求使用者輸入電子郵件位址，您可以使用 Graph API 來協助使用者搜尋及瀏覽以尋找組織中的其他人。 使用者可以使用常見的通訊錄樣式介面進行瀏覽，包括檢視組織階層的詳細資料。 深入了解[圖形 API](active-directory-graph-api.md)。

**重複使用您的客戶已經在管理的 Active Directory 群組與通訊群組清單。**  Azure AD 包含您的客戶已經用於電子郵件通訊群組和管理存取的群組。 使用 Graph API，與其要求您的客戶在應用程式中建立和管理一組個別的群組，您可以重複使用這些群組。 群組資訊也可以透過登入權杖傳送至您的應用程式。 深入了解 [Graph API](active-directory-graph-api.md)。

**使用 Azure AD 來控制有權存取應用程式的人員。**  系統管理員和 Azure AD 中的應用程式擁有者可以將應用程式的存取權限指派給特定使用者和群組。 使用 Graph API，您可以讀取這份清單並用它來控制資源的佈建和取消佈建，以及應用程式內的存取。

**使用 Azure AD 進行角色型存取控制。**  系統管理員和應用程式擁有者可以將使用者和群組指派至您在 Azure AD 中註冊應用程式時所定義的角色。 角色資訊會透過登入權杖傳送至您的應用程式，並可使用 Graph API 進行讀取。 深入了解 [使用 Azure AD 進行授權](https://cloudblogs.microsoft.com/enterprisemobility/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles/)。

### <a name="get-access-to-users-profile-calendar-email-contacts-files-and-more"></a>取得使用者設定檔、行事曆、電子郵件、連絡人、檔案及其他等存取權限
**Azure AD 是 Office 365 和其他 Microsoft 商務服務的授權伺服器。**  如果您的應用程式支援 Azure AD 登入，或支援使用 OAuth 2.0 將目前使用者帳戶連結至 Azure AD 使用者帳戶，則您可以要求使用者設定檔、行事曆、電子郵件、連絡人、檔案及其他資訊的讀取和寫入存取權限。 您可以順利地將事件寫入使用者的行事曆，並讀取檔案或將檔案寫入使用者的 OneDrive。 深入了解 [存取 Office 365 API](https://msdn.microsoft.com/office/office365/howto/platform-development-overview)。

### <a name="promote-your-application-in-the-azure-and-office-365-marketplaces"></a>在 Azure 和 Office 365 Marketplace 中推廣您的應用程式
**將您的應用程式擴廣到數百萬個已使用 Azure AD 的組織。**  搜尋和瀏覽 Marketplace 的使用者已經使用一或多項雲端服務，讓他們成為合格的雲端服務客戶。 深入了解如何在 [Azure Marketplace](https://azure.microsoft.com/marketplace/partner-program/)中推廣您的應用程式。

**當使用者註冊您的應用程式時，它便會出現在其 Azure AD 存取面板和 Office 365 應用程式啟動程式中。**  使用者將能夠迅速且輕鬆地返回您的應用程式，以增加使用者參與度。 深入了解 [Azure AD 存取面板](../user-help/active-directory-saas-access-panel-introduction.md)。

### <a name="secure-device-to-service-and-service-to-service-communication"></a>保護裝置對服務和服務對服務通訊的安全
**使用 Azure AD 進行服務和裝置的身分識別管理，可減少您需要撰寫的程式碼，並可讓 IT 管理存取權限。**  服務與裝置可以使用 OAuth 取得 Azure AD 的權杖，並使用這些權杖來存取 Web API。 使用 Azure AD，您可以避免撰寫複雜的驗證程式碼。 因為服務和裝置的身分識別會儲存在 Azure AD 中，IT 可以在同一個地方管理金鑰和撤銷，而無需在您的應用程式中分別執行此動作。

## <a name="benefits-of-integration"></a>整合的優點
與 Azure AD 整合帶來您無需撰寫額外程式碼的優點。

### <a name="integration-with-enterprise-identity-management"></a>與企業身分識別管理整合
**協助您的應用程式符合 IT 原則。**  組織會將其企業身分識別管理系統與 Azure AD 整合，所以當某人離開組織時，它們會自動喪失對應用程式的存取權限，而且 IT 無需採取額外步驟。 IT 可以管理誰可以存取您的應用程式，並判斷需要哪些存取原則 (例如，多因素驗證)，以降低撰寫程式碼以符合複雜公司原則的需求。 Azure AD 提供系統管理員詳盡的稽核記錄，內容包括登入應用程式的使用者，以便 IT 可以追蹤使用情況。

**Azure AD 將 Active Directory 延伸到雲端，讓您的應用程式可以與 AD 整合。**  世界各地許多組織使用 Active Directory 作為其主要登入和身分識別管理系統，並要求他們的應用程式使用 AD。 與 Azure AD 整合即可將 Active Directory 和您的應用程式整合。

### <a name="advanced-security-features"></a>進階的安全性功能
**多因素驗證。**  Azure AD 提供原生多因素驗證。 IT 系統管理員可以要求多因素驗證才能存取應用程式，因此您無需自行撰寫這項支援的程式碼。 深入了解 [多因素驗證](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)。

**登入偵測異常。**  Azure AD 每天處理超過 10 億個登入，同時使用機器學習演算法來偵測可疑活動，並向 IT 系統管理員通知可能發生的問題。 藉由支援 Azure AD 登入，您的應用程式便會獲得這項保護的好處。 深入了解[檢視 Azure Active Directory 存取報告](../active-directory-view-access-usage-reports.md)。

**條件式存取。**  除了多因素驗證，系統管理員還可以要求使用者在登入您的應用程式之前必須符合特定條件。 可以設定的條件包括用戶端裝置的 IP 位址範圍、指定群組的成員資格，以及用於存取的裝置狀態。 深入了解 [Azure Active Directory 條件式存取](../active-directory-conditional-access-azure-portal.md)。

### <a name="easy-development"></a>容易開發
**業界標準通訊協定。**  Microsoft 致力於支援業界標準。 Azure AD 支援 SAML 2.0、OpenID Connect 1.0、OAuth 2.0 和 WS 同盟 1.2 驗證通訊協定。 Graph API 屬於 OData 4.0 標準。 如果您的應用程式已經支援 SAML 2.0 或 OpenID Connect 1.0 通訊協定的同盟登入，那新增 Azure AD 支援便相當直接。 深入了解 [Azure AD 支援的驗證通訊協定](active-directory-authentication-protocols.md)。

**開放原始碼程式庫。**  Microsoft 針對常用的語言和平台提供完整支援的開放原始碼程式庫，進而加快開發速度。 原始碼採用 Apache 2.0 授權，您可以任何分岔以及貢獻回饋給專案。 深入了解 [Azure AD 驗證程式庫](active-directory-authentication-libraries.md)。

### <a name="worldwide-presence-and-high-availability"></a>全球的目前狀態和高可用性
**Azure AD 會在世界各地的資料中心內進行部署，並受到全天候的管理和監控。**  Azure AD 是適用於 Microsoft Azure 和 Office 365 的身分識別管理系統，並在世界各地的 28 個資料中心內進行部署。 保證將目錄資料複寫到至少三個資料中心。 全域負載平衡器會確保使用者可存取包含其資料的最接近 Azure AD 複本，並在偵測到問題時，自動將要求重新路由到其他資料中心。

## <a name="next-steps"></a>後續步驟
[開始撰寫程式碼](azure-ad-developers-guide.md#get-started)

[使用 Azure AD 登入使用者](authentication-scenarios.md)

