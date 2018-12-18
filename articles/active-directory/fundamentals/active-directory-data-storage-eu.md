---
title: Azure AD 為歐洲客戶儲存身分識別資料的位置 |Microsoft Docs
description: 了解 Microsoft Azure Active Directory 將其歐洲客戶的身分識別相關資料儲存於何處。
services: active-directory
author: eross-msft
manager: mtillman
ms.author: lizross
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 05/17/2018
ms.custom: it-pro
ms.openlocfilehash: d0015f40714b639d15245827ae0da3ca0f132df4
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45733392"
---
# <a name="where-does-microsoft-azure-active-directory-azure-ad-store-identity-data-for-european-customers"></a>Microsoft Azure Active Directory (Azure AD) 為歐洲客戶儲存身分識別資料的位置
Azure AD 可協助您管理使用者身分識別，以及建立以資訊為依據的存取原則，以利確保組織資源的安全。 身分識別資料的儲存位置，會以組織在您訂閱服務時所提供的位址為基礎。 例如，當您訂閱 Office 365 或 Azure 時。 若想了解有關於您的身分識別資料儲存於何處的特定資訊，您可以使用 Microsoft 信任中心的 [您的資料所在位置](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) 區段。

雖然大多數與 Azure AD 有關的歐洲身分識別資料都儲存在歐洲資料中心，但有五項使用者相關屬性通常會儲存在美國資料中心。 這些屬性是 GivenName、Surname、userPrincipalName、Domain 和 PasswordHash。 但 PasswordHash 屬性有時屬例外；如果您使用內部部署的同盟驗證方法，而使 PasswordHash 值無法與 Azure AD 同步處理，該屬性就不會儲存在美國。 此外，有一些服務特有的作業資料是一般 Azure AD 作業不可或缺的，此類資料會儲存在美國，但不會包含任何個人資料。

## <a name="data-stored-outside-of-european-datacenters-for-european-customers"></a>儲存在歐洲資料中心以外的歐洲客戶資料

就具有歐洲區位址的組織而言，與 Azure AD 有關的歐洲身分識別資料，大多都會保存在歐洲資料中心。 不會儲存在歐洲資料中心的 Azure AD 資料包括：

- **身分識別相關屬性**

    下列身分識別相關屬性將會複寫至美國：

    - GivenName
    - Surname
    - userPrincipalName
    - 網域
    - PasswordHash
    - SourceAnchor
    - AccountEnabled
    - PasswordPolicies
    - StrongAuthenticationRequirement
    - ApplicationPassword
    - PUID

- **Microsoft Azure 多因素驗證 (MFA) 和 Azure AD 自助式密碼重設 (SSPR)**
    
    MFA 會將所有待用的使用者資料儲存在歐洲資料中心。 不過，某些 MFA 服務特有的資料會儲存在美國，包括：
    
    - 如果您使用 MFA 或 SSPR，則雙因素驗證及其相關個人資料可能會儲存在美國。
        - 所有使用通話或 SMS 的雙因素驗證都可能須由美國電信業者完成。
        - 使用 Microsoft Authenticator 應用程式的推播通知需要來自製造商通知服務 (Apple 或 Google) 的通知，而這可能不在歐洲境內。
        - OATH 代碼一律會在美國進行驗證。 
    - 某些 MFA 和 SSPR 記錄會儲存在美國 30 天，無論驗證類型為何。

- **Microsoft Azure Active Directory B2C (Azure AD B2C)**

    Azure AD B2C 會將所有待用的使用者資料儲存在歐洲資料中心。 不過，作業資料 (不含個人資料) 則會存留在使用者存取服務之處。 例如，如果 B2C 使用者在美國存取服務，作業記錄就會存留在美國。此外，所有不含個人資料的原則組態資料都只會儲存在美國。如需原則組態的詳細資訊，請參閱 [Azure Active Directory B2C：內建原則](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-policies)一文。

- **Microsoft Azure Active Directory B2B (Azure AD B2B)** 
    
    Azure AD B2B 會將所有待用的使用者資料儲存在歐洲資料中心。 不過，B2B 會將其非個人中繼資料儲存在美國資料中心的資料表中。 此資料表會包含 redeemUrl、invitationTicket、資源租用戶識別碼、InviteRedirectUrl 和 InviterAppId 等欄位。

- **Microsoft Azure Active Directory Domain Services (Azure AD DS)**

    Azure AD DS 會將使用者資料儲存在與客戶選取的 Azure 虛擬網路相同的位置。 因此，如果網路不在歐洲境內，則資料會複寫並儲存在歐洲以外。

- **與 Azure AD 整合的服務和應用程式**

    任何與 Azure AD 整合的服務和應用程式，都可以存取身分識別資料。 請評估每項服務和應用程式，以確認該服務和應用程式處理身分識別資料的方式，以及它們是否符合公司的資料儲存需求。

    若想進一步了解 Microsoft 服務的資料存放處，請參閱 Microsoft 信任中心的 [您的資料所在位置](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) 區段。

## <a name="next-steps"></a>後續步驟
如需前述任何特性和功能的詳細資訊，請參閱下列文章。
- [何謂 Multi-Factor Authentication？](https://docs.microsoft.com/azure/active-directory/authentication/multi-factor-authentication)
- [Azure AD 自助式密碼重設](https://docs.microsoft.com/azure/active-directory/authentication/active-directory-passwords-overview)
- [什麼是 Azure Active Directory B2C？](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-overview)
- [何謂 Azure AD B2B 共同作業？](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)
- [Azure Active Directory (AD) Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-overview)
