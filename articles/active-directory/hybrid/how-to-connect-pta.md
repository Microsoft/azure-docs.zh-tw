---
title: Azure AD Connect：傳遞驗證 | Microsoft Docs
description: 這篇文章說明 Azure Active Directory (Azure AD) 傳遞驗證，以及它如何向內部部署 Active Directory 驗證使用者的密碼以允許 Azure AD 登入。
services: active-directory
keywords: 什麼是 Azure AD Connect 傳遞驗證、安裝 Active Directory，以及 Azure AD、SSO、單一登入的必要元件
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 4ae84ff481b25f1e91a29c067dea74ba25f924aa
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320317"
---
# <a name="user-sign-in-with-azure-active-directory-pass-through-authentication"></a>使用 Azure Active Directory 傳遞驗證來進行使用者登入

## <a name="what-is-azure-active-directory-pass-through-authentication"></a>什麼是 Azure Active Directory 傳遞驗證？

Azure Active Directory (Azure AD) 傳遞驗證可讓您的使用者以相同密碼登入內部部署和雲端式應用程式。 這項功能可讓您的使用者獲得更好的體驗，不僅少了一個要記住的密碼，還會因為使用者不太可能忘了如何登入而降低 IT 技術服務人員成本。 當使用者使用 Azure AD 登入時，此功能會向您的內部部署 Active Directory 直接驗證使用者的密碼。

>[!VIDEO https://www.youtube.com/embed/PyeAC85Gm7w]

這是 [Azure AD 密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)的替代功能，可為組織提供相同的雲端驗證好處。 不過，某些想要強制執行其內部部署 Active Directory 安全性和密碼原則的組織，可以選擇改用傳遞驗證。 如需各種 Azure AD 登入方法的比較，以及如何為組織選擇正確的登入方法，請檢閱[這份指南](https://docs.microsoft.com/azure/security/azure-ad-choose-authn)。

![Azure AD 傳遞驗證](./media/how-to-connect-pta/pta1.png)

您可以將傳遞驗證與[無縫單一登入](how-to-connect-sso.md)功能結合在一起。 如此一來，當使用者在公司網路內的公司電腦上存取應用程式時，就不需要輸入密碼來進行登入。

## <a name="key-benefits-of-using-azure-ad-pass-through-authentication"></a>使用 Azure AD 傳遞驗證的主要好處

- 良好的使用者體驗
  - 使用者使用相同的密碼來登入內部部署和雲端型的應用程式。
  - 使用者可以減少尋求 IT 技術服務人員解決密碼相關問題所耗費的對話時間。
  - 使用者可以在雲端中完成[自助式密碼管理](../authentication/active-directory-passwords-overview.md)工作。
- 容易部署和管理
  - 不必再進行複雜的內部部署或網路設定。
  - 只需要在內部部署環境安裝輕量型代理程式。
  - 沒有任何額外的管理負荷。 代理程式會自動收到改進和錯誤的修正。
- *安全*
  - 內部部署密碼絕對不會以任何形式儲存在雲端。
  - 代理程式只會從您的網路內進行輸出連線。 因此，不需要將代理程式安裝在周邊網路 (又稱做 DMZ) 中。
  - 與 [Azure AD 條件式存取原則](../active-directory-conditional-access-azure-portal.md) (包括 Multi-Factor Authentication (MFA)) 緊密配合、[封鎖舊版驗證](../conditional-access/conditions.md)，並[篩除暴力密碼破解攻擊](../authentication/howto-password-smart-lockout.md)，藉此保護您的使用者帳戶。
- *高可用性*
  - 可以在多部內部部署伺服器上安裝其他代理程式，以提供高可用性的登入要求。

## <a name="feature-highlights"></a>功能要點

- 可讓使用者登入到使用[新式驗證](https://aka.ms/modernauthga)的所有 Web 瀏覽器型應用程式和 Microsoft Office 用戶端應用程式。
- 登入使用者名稱可以是內部部署的預設使用者名稱 (`userPrincipalName`)，或在 Azure AD Connect 中設定的另一個屬性 (又稱為 `Alternate ID`)。
- 此功能可與[條件式存取](../active-directory-conditional-access-azure-portal.md)功能 (例如，Multi-Factor Authentication (MFA)) 緊密配合，以協助保護您的使用者。
- 與雲端式[自助式密碼管理](../authentication/active-directory-passwords-overview.md)整合，包括可將密碼回寫至內部部署 Active Directory，以及藉由禁用常見密碼來提供密碼保護。
- 如果 AD 樹系之間有樹系信任且名稱尾碼路由已正確設定，就支援多樹系環境。
- 這是免費功能，您不需要任何付費的 Azure AD 版本即可使用。
- 您可以透過 [Azure AD Connect](whatis-hybrid-identity.md) 啟用它。
- 它會使用輕量型內部部署代理程式來接聽並回應密碼驗證要求。
- 安裝多個代理程式就能提供高可用性的登入要求。
- 它能[保護](../authentication/howto-password-smart-lockout.md)您的內部部署帳戶，不讓其遭到雲端暴力密碼破解攻擊的威脅。

## <a name="next-steps"></a>後續步驟

- [快速入門](how-to-connect-pta-quick-start.md)：開始執行 Azure AD 傳遞驗證。
- [從 AD FS 遷移到傳遞驗證](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true) \(英文\)：從 AD FS (或其他同盟技術) 遷移到傳遞驗證的詳細指南。
- [智慧鎖定](../authentication/howto-password-smart-lockout.md)：在租用戶中設定智慧鎖定功能來保護使用者帳戶。
- [目前的限制](how-to-connect-pta-current-limitations.md)：了解支援與不支援的案例。
- [技術性深入探討](how-to-connect-pta-how-it-works.md) - 了解這項功能的運作方式。
- [常見問題集](how-to-connect-pta-faq.md) - 常見問題集的答案。
- [疑難排解](tshoot-connect-pass-through-authentication.md) - 了解如何解決此功能的常見問題。
- [安全性深入探討](how-to-connect-pta-security-deep-dive.md)：關於此功能的其他深入技術資訊。
- [Azure AD 無縫 SSO](how-to-connect-sso.md)：深入了解此互補功能。
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) - 用於提出新的功能要求。
