---
title: 什麼是 Azure Active Directory (Azure AD)？ | Microsoft Docs
description: 了解如何使用 Azure Active Directory 將現有的內部部署身分識別延伸至雲端，或開發 Azure AD 整合式應用程式。
services: active-directory
author: eross-msft
manager: mtillman
ms.author: lizross
ms.assetid: 498820c4-9ebe-42be-bda2-ecf38cc514ca
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: overview
ms.date: 09/13/2018
ms.custom: it-pro
ms.openlocfilehash: 08d32a99e2f1a77bcc7a5900b47e6771c33ec356
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393316"
---
# <a name="what-is-azure-active-directory"></a>什麼是 Azure Active Directory？
Azure Active Directory (Azure AD) 是 Microsoft 的多重租用戶雲端式目錄和身分識別管理服務。 Azure AD 能將核心目錄服務、應用程式存取管理，以及身分識別保護結合為單一解決方案，並提供以標準為基礎的平台，以協助開發人員根據集中式原則及規則為其應用程式傳遞存取控制。

![Azure AD Connect 堆疊](./media/active-directory-whatis/Azure_Active_Directory.png)

## <a name="benefits-of-azure-ad"></a>Azure 的優點
Azure AD 可協助您：

-   為全體企業的每個使用者建立和管理單一身分識別，讓使用者、群組和裝置與 [Azure AD Connect](../connect/active-directory-aadconnect.md) 保持同步。

-   提供應用程式 (包括數千個預先整合的 SaaS 應用程式) 的單一登入存取，以及使用 [Azure AD 應用程式 Proxy](../manage-apps/application-proxy.md) 為內部部署 SaaS 應用程式提供更安全的遠端存取。

-   藉由同時對內部部署和雲端應用程式強制以規則為基礎的[多重要素驗證](../authentication/concept-mfa-howitworks.md)原則，以允許應用程式存取安全性。

-   利用[自助式密碼重設](../user-help/user-help-reset-password.md)來提升使用者生產力，以及使用 [MyApps 入口網站](../user-help/active-directory-saas-access-panel-introduction.md)來改善群組和應用程式存取要求。

-   利用全球性、企業層級、雲端架構身分識別和存取管理解決方案的[高可用性和可靠性](https://docs.microsoft.com/azure/architecture/checklist/availability)。

## <a name="who-uses-azure-ad"></a>Azure AD 的使用者
Azure AD 適用於 IT 系統管理員、應用程式開發人員，以及 Office 365、Azure 或 Dynamics CRM Online 的使用者。

- **IT 系統管理員。** Azure AD 為組織提供更安全的解決方案，使用更強的身分識別管理和單一登入 (SSO) 存取來存取數千個[雲端式 SaaS 應用程式](../saas-apps/tutorial-list.md)和內部部署應用程式。 透過這些應用程式，您也能為使用者提供雲端式的應用程式安全性、順暢的存取、增強的共同作業，以及自動化的身分識別生命週期，以協助提高安全性與合規性。

    此外，透過 [Azure AD Connect](../connect/active-directory-aadconnect-get-started-express.md)，您可以將 Azure AD 與現有的 Windows Server Active Directory 整合，讓組織能夠運用現有的內部部署身分識別資產，來管理雲端式 SaaS 應用程式的存取權。

- **針對應用程式開發人員。** Azure AD 可整合由全球各地數百萬個組織所使用的身分識別管理解決方案，來協助您專注於應用程式的建置之上。

- **針對 Office 365、Azure 或 Dynamics CRM Online 的客戶。** 您已經在使用 Azure AD。 每個 Office 365、Azure、Dynamics CRM Online 租用戶實際上就是 Azure AD 租用戶，這可讓您立即開始管理使用者對整合後雲端應用程式的存取。

## <a name="how-reliable-is-azure-ad"></a>Azure AD 有多可靠？
Azure AD 多租用戶、地理分布周全，且高可用性的設計，表示您可以依賴 Azure AD 來應付最重要的商務需求。 Azure AD 執行於遍佈世界各地的 28 個資料中心，並具有自動容錯移轉的能力。 這代表就算其中一個資料中心關機，您目錄資料的複本仍會存在於至少兩個位於其他區域的資料中心內，並可供立即存取。

如需服務等級協定的詳細資訊，請參閱[服務等級協定](https://azure.microsoft.com/support/legal/sla/)。

## <a name="choose-an-edition"></a>選擇版本
所有 Microsoft Online 商務服務都依賴 Azure AD 來進行登入和其他身分識別需求。 如果您訂閱任何 Microsoft Online 商務服務 (例如 Office 365、Microsoft Azure)，您就會自動取得 Azure AD 並可存取所有免費的功能。 使用 Azure Active Directory Free 版本，您可以管理使用者和群組、與內部部署目錄同步處理，以及取得 Azure、Office 365 和數千個熱門 SaaS 應用程式 (像是 Salesforce、Workday、Concur、DocuSign、Google Apps、Box、ServiceNow、Dropbox) 之間的單一登入，以及更多其他功能。 

若要增強您的 AD 實作，您也可以升級至 Azure Active Directory Basic、Premium P1 或 Premium P2 版本來新增付費功能。 付費版本的 Azure AD 會建立在您現有的免費目錄上，提供適用於行動員工的自助服務、增強監視、安全性報告、多重要素驗證 (MFA) 及安全存取的企業級功能。

> [!NOTE]
> 這兩種版本的詳細價格請參閱 [Azure Active Directory 價格](https://azure.microsoft.com/pricing/details/active-directory/)。 目前在中國不支援 Premium P1、Premium P2 及 Azure Active Directory Basic。 如需有關 Azure AD 定價的詳細資訊，請透過 Azure Active Directory 論壇與我們連絡。

- **Azure Active Directory Basic。** 此版本適用於有雲端優先需求的工作者，提供以雲端為中心的應用程式存取及自助式身分識別管理解決方案。 透過 Basic 版本，您可獲得生產力增強和成本節約功能，例如群組式存取管理、雲端應用程式的自助式密碼重設，以及 Azure Active Directory 應用程式 Proxy (以使用 Azure AD 發佈內部部署 Web 應用程式)，且全都由可獲得 99.9% 運作時間的企業 SLA 支援。

- **Azure Active Directory Premium P1。** Azure Active Directory Premium 版本旨在讓身分識別及存取管理需求較高的組織獲得能力，新增了功能豐富的企業級身分識別管理功能，並可讓混合式使用者無縫存取內部部署及雲端功能。 此版本包含資訊背景工作角色和混合環境中身分識別管理員對於應用程式存取、自助身分識別和存取管理 (IAM)、雲端中的身分識別保護和安全性所需的一切。 它支援進階管理和委派資源，例如：動態群組和自助群組管理。 它包含 Microsoft Identity Manager (一項內部部署及身分識別和存取管理套件)，並提供可為您的內部部署使用者啟用自助密碼重設之類解決方案的雲端回寫功能。

- **Azure Active Directory Premium P2。** 這個新供應項目旨在為使用者和系統管理員提供進階的保護，其不僅包括 Azure AD Premium P1 中的所有功能，也包括 Identity Protection 與 Privileged Identity Management。 Azure Active Directory Identity Protection 利用數十億個訊號，針對應用程式及重要公司資料提供以風險為基礎的條件式存取。 此外，我們還使用 Azure Active Directory Privileged Identity Management 來協助您管理及保護授權帳戶，讓您能夠探索、限制和監視系統管理員及他們對資源的存取，並在需要時，提供及時的存取權。  

> [!NOTE]
> 有數個 Azure Active Directory 功能也可以透過「隨用隨付」版本取得：<ul><li>**Azure Active Directory B2C。** 適用於消費者面向應用程式的身分識別和存取管理解決方案。 如需詳細資訊，請參閱 [Azure Active Directory B2C](https://azure.microsoft.com/documentation/services/active-directory-b2c/)。</li><li>**Azure Multi-Factor Authentication。** 用於個別使用者或個別驗證提供者。 如需詳細資訊，請參閱[什麼是 Azure Multi-Factor Authentication？](../authentication/multi-factor-authentication.md)。

## <a name="as-an-admin-how-do-i-get-started"></a>身為系統管理員，我該如何開始使用？
註冊免費的 30 天試用並部署您的第一個雲端解決方案，請參閱 [Azure Active Directory Premium 試用](https://azure.microsoft.com/trial/get-started-active-directory/)。

## <a name="as-a-developer-how-do-i-get-started"></a>身為開發人員，我該如何開始使用？
註冊免費的 30 天試用並開始與 Azure AD 整合您的應用程式，請參閱 [Azure Active Directory Premium 試用](https://azure.microsoft.com/trial/get-started-active-directory/)。 如需詳細資訊，您也可以參閱適用於 Azure Active Directory 的[開發人員指南](../develop/azure-ad-developers-guide.md)。

## <a name="next-steps"></a>後續步驟
- [深入了解 Azure 身分識別和存取管理的基本概念](identity-fundamentals.md)。

- [與 Windows Server Active Directory 整合 Azure AD](../hybrid/how-to-connect-install-express.md)。
