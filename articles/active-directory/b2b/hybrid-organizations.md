---
title: 混合式組織的 B2B 共同作業 - Azure Active Directory | Microsoft Docs
description: 透過 Azure AD B2B 共同作業對夥伴提供內部部署和雲端資源的存取權。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 773ae5aefab59e79822c0320773753b47cfefdeb
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45983864"
---
# <a name="azure-active-directory-b2b-collaboration-for-hybrid-organizations"></a>混合式組織的 Azure Active Directory B2B 共同作業

Azure Active Directory (Azure AD) B2B 共同作業可讓您輕鬆地對外部夥伴提供組織中應用程式和資源的存取權。 即使是在同時擁有內部部署和雲端式資源的混合式組態中，也是如此。 如果您目前是在內部部署身分識別系統本機管理外部夥伴帳戶，或如果您是以 Azure AD B2B 使用者身分在雲端中管理外部帳戶，都沒有關係。 您現在可以使用和這兩種環境相同的登入認證，對這些使用者授與任一位置資源的存取權。

## <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-apps"></a>對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權

如果您的組織使用 Azure AD B2B 共同作業功能邀請夥伴組織的來賓使用者進入您 Azure AD，您現在可以對這些 B2B 使用者提供內部部署應用程式的存取權。

對於使用 SAML 型驗證的應用程式，您可以透過 Azure 入口網站將這些應用程式提供給 B2B 使用者，並使用 AD 應用程式 Proxy 進行驗證。

對於搭配使用整合式 Windows 驗證 (IWA) 和 Kerberos 限制委派 (KCD) 的應用程式，您也可以使用 Azure AD Proxy 來進行驗證。 不過，若要讓驗證運作，內部部署 Windows Server Active Directory 中必須要有使用者物件。 有兩種方法可用來建立代表 B2B 來賓使用者的本機使用者物件。

- 您可以使用 Microsoft Identity Manager (MIM) 2016 SP1 和 Microsoft Graph 的 MIM 管理代理程式。
- 您可以使用 PowerShell 指令碼。 (此解決方案不需要 MIM)。

如需如何實作這些解決方案的詳細資訊，請參閱[對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權](hybrid-cloud-to-on-premises.md)。

## <a name="grant-locally-managed-partner-accounts-access-to-cloud-resources"></a>對本機管理的夥伴帳戶授與雲端資源的存取權

在 Azure AD 之前，具有內部部署身分識別系統的組織傳統上是在其內部部署目錄中管理夥伴帳戶的。 如果您是這類組織，請確定您在將應用程式和其他資源移至雲端後，您的夥伴仍繼續擁有存取權。 在理想情況下，建議您讓這些使用者使用同一組認證來存取雲端和內部部署資源。 

我們現在提供方法讓您使用 Azure AD Connect 以「來賓使用者」身分將這些本機帳戶同步至雲端，這些帳戶的行為就和 Azure AD B2B 使用者一樣。

若要協助保護公司資料，您可以控制存取權只能存取正確的資源，並設定授權原則以不同於您員工的方式來對待這些來賓使用者。

如需實作詳細資料，請參閱[使用 Azure AD B2B 共同作業對本機管理的夥伴帳戶授與雲端資源的存取權](hybrid-on-premises-to-cloud.md)。
 
## <a name="next-steps"></a>後續步驟

- [對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權](hybrid-cloud-to-on-premises.md)
- [使用 Azure AD B2B 共同作業對本機管理的夥伴帳戶授與雲端資源的存取權](hybrid-on-premises-to-cloud.md)


