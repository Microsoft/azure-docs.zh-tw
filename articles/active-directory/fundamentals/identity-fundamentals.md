---
title: Azure 身分識別管理的基本概念 | Microsoft Docs
description: 雲端型身分識別現在是持續控制與查看使用者以何種方式及在何時存取公司應用程式和資料的最佳方式。
keywords: ''
author: eross-msft
manager: mtillman
ms.reviewer: jsnow
ms.author: lizross
ms.date: 08/07/2018
ms.topic: overview
ms.prod: ''
ms.service: active-directory
ms.component: fundamentals
ms.technology: ''
ms.assetid: ''
ms.custom: it-pro
ms.openlocfilehash: 327cecd129befb56c33d7fcf2d59ee5b58a18549
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42023567"
---
# <a name="fundamentals-of-azure-identity-management"></a>Azure 身分識別管理的基本概念

隨著存放在公司網路外部 (雲端及裝置) 的公司數位資源日益增多，絕佳的雲端身分識別和存取管理解決方案更加不可或缺。 雲端型身分識別現在是持續控制與查看使用者以何種方式及在何時存取公司應用程式和資料的最佳方式。

十多年來，Microsoft 一直保護雲端架構身分識別的安全，而現在透過 [Azure Active Directory (AD)](active-directory-whatis.md) 持續為您提供相同的保護系統。 使用 Azure AD，企業系統管理員可透過前所未有的更佳安全性和監管，輕易地確保使用者和系統管理員負有責任。

Azure AD Premium 是雲端架構身分識別和存取管理解決方案，其具備可對所有應用程式啟用一個安全身分識別的進階保護功能、身分識別保護 (由 [Microsoft 智慧安全性圖表](https://www.microsoft.com/security/intelligence)增強)，以及 Privileged Identity Management。 Azure AD Premium 不只是另一個監視或報告工具，還可即時保護使用者的身分識別，並可讓您建立以風險為基礎的調適性存取原則來保護您組織的資料。

觀看以下短片，快速概覽 Azure AD 身分識別管理和保護功能︰
>[!VIDEO https://www.youtube.com/embed/9LGIJ2-FKIM]

Microsoft 不只提供可讓您隨處通行的身分識別，而且也提供一組工具來自動執行、協助保護及管理您組織內的 IT。 即使在雲端運算問世以後，仍有管理和控制 IT 工作的需求，例如重設使用者密碼的技術支援請求、使用者群組管理，以及應用程式要求。 更複雜的是，員工現在可使其個人裝置運作和使用立即可用的 SaaS 應用程式，這會讓跨公司資料中心和公用雲端平台控管其應用程式成為一大挑戰。

[!INCLUDE [identity](../../../includes/azure-ad-licenses.md)]

## <a name="connect-on-premises-active-directory-with-azure-ad-and-office-365"></a>連接內部部署 Active Directory 與 Azure AD 和 Office 365
大量投資內部部署 Active Directory 的組織可以將其內部部署 Azure AD 目錄整合到[混合式身分識別管理](https://aka.ms/aadframework)，進而將這些投資延伸至雲端。 這麼做可提供通用身分識別來存取各地資源，而讓您的使用者更具生產力。 然後，使用者和組織可以使用單一登入 (SSO) 來存取內部部署資源和雲端服務，例如 Office 365。

完成整合，您只需要 [Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。 Azure AD Connect 會提供功能來支援身分識別同步需求，並取代舊版的身分識別整合工具，如 DirSync 和 Azure AD Sync。使用 Azure AD Connect，身分識別管理和內部部署與 Azure AD 之間的同步處理是透過下列項目啟用︰

- 同步處理 - 此元件負責建立使用者、群組和其他物件。 它也負責確保您的內部部署使用者和群組的身分識別資訊符合雲端。 也可以啟用密碼回寫，當使用者在 Azure AD 中更新其密碼時，讓內部部署目錄保持同步。
- 驗證 - 以 Azure AD 作為新的控制平面時，驗證將是雲端存取的基礎。 選擇正確的驗證方法，是設定 Azure AD 混合式身分識別解決方案的重要決策。 請檢閱[本指南](https://aka.ms/auth-options)，從雲端驗證 (密碼雜湊同步處理/傳遞驗證) 和同盟驗證 (AD FS) 中擇一用於您的組織。
- 狀況監控 - [Azure AD Connect Health](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health) 可以提供健全監控，並在 Azure 入口網站中提供檢視此活動的中央位置。

## <a name="increase-productivity-and-reduce-helpdesk-costs-with-self-service-and-single-sign-on-experiences"></a>經由自助式和單一登入體驗提高生產力並降低技術支援成本

當員工只需要記住單一使用者名稱密碼並擁有每個裝置的一致經驗時，更具有生產力。 當他們可以執行自助式工作，例如[重設遺忘的密碼](https://docs.microsoft.com/azure/active-directory/active-directory-passwords)或要求應用程式的存取，而不需等待技術支援人員的協助時，也可節省時間。

Azure AD [將內部部署 Active Directory](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 延伸到雲端，讓使用者能夠使用其主要組織帳戶來登入已加入網域的裝置和公司資源，也能登入用於完成其作業所需的所有 Web 和 SaaS 應用程式。 除了不需要記住多組使用者名稱和密碼，使用者還可根據其組織群組成員資格及其身為員工的狀態，自動佈建 (或解除佈建) 其應用程式存取權。 而且，您可以透過 [Azure AD 應用程式 Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-get-started) 控制資源庫應用程式或您已開發並發佈之自有內部部署應用程式的存取權。

## <a name="manage-and-control-access-to-corporate-resources"></a>管理和控制公司資源的存取權
Microsoft 身分識別與存取管理解決方案會協助 IT 保護應用程式和跨公司資料中心和到雲端的存取，啟用其他層級的驗證，例如 [Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-whats-next) 和[條件式存取原則](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)。 透過進階的安全性報告、稽核和警示來監視可疑活動，有助於減緩潛在的安全性問題。

Azure AD Premium 中的條件式存取原則讓身為企業系統管理員的您能夠針對任何 Azure AD 連線的應用程式 (SaaS 應用程式、在雲端執行的自訂應用程式或內部部署 Web 應用程式中) 建立以原則為基礎的存取規則。 Azure AD 會即時評估這些原則，並且在使用者嘗試存取應用程式時強制執行這些原則。 Azure 身分識別保護原則可在偵測到可疑的活動時，讓您自動採取動作。 這些動作包括封鎖高風險的使用者存取、強制執行 Multi-Factor Authentication，以及重設使用者密碼，如果認證疑似遭入侵。


## <a name="azure-active-directory-privileged-identity-management"></a>Azure Active Directory Privileged Identity Management

Azure Active Directory Premium P2 供應項目隨附的 [Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started)，可讓您探索、限制及監視系統管理帳戶與其對 Azure Active Directory 資源和其他 Microsoft 線上服務的存取權。 它也會協助您管理所需確切期間內的隨選系統管理存取權。

Privileged Identity Management 可以強制隨選系統管理員權限，讓系統管理員可以在其帳戶回到正常使用者狀態之前，要求進行 Multi-Factor Authentication，針對預先設定的期間暫時提高其權限。

## <a name="benefits-of-azure-identity"></a>Azure 身分識別的優點

透過 Azure 身分識別管理，您可以：

-   為全體企業的每位使用者建立和管理單一身分識別，讓使用者、群組和裝置與 [Azure Active Directory Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect) 保持同步。

-   提供應用程式 (包括數千個預先整合的 SaaS 應用程式) 的單一登入存取，或使用 [Azure AD 應用程式 Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-get-started) 提供內部部署 SaaS 應用程式的遠端安全存取。

-   藉由同時對內部部署和雲端應用程式強制以規則為基礎的 [Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-whats-next)，啟用應用程式存取安全性。

-   利用[自助式密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-passwords)來提升使用者生產力，以及使用 [MyApps 入口網站](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-user-help)來改善群組和應用程式存取要求。

-   利用全球性、企業層級、雲端架構身分識別和存取管理解決方案的[高可用性和可靠性](https://docs.microsoft.com/azure/architecture/resiliency/high-availability-azure-applications)。

## <a name="next-steps"></a>後續步驟
[深入了解 Azure 身分識別解決方案](https://docs.microsoft.com/azure/active-directory/understand-azure-identity-solutions)
