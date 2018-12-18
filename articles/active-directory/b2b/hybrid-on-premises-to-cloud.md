---
title: 以 Azure AD B2B 使用者的身分對本機管理的夥伴帳戶授與雲端資源的存取權 | Microsoft Docs
description: 使用和 Azure AD B2B 共同作業相同的認證，對本機管理的外部夥伴提供本機和雲端資源的存取權。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 04/24/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 722ab51d0f591b8f16924d1d5661385267b7f0e6
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295150"
---
# <a name="grant-locally-managed-partner-accounts-access-to-cloud-resources-using-azure-ad-b2b-collaboration"></a>使用 Azure AD B2B 共同作業對本機管理的夥伴帳戶授與雲端資源的存取權

在 Azure Active Directory (Azure AD) 之前，具有內部部署身分識別系統的組織傳統上是在其內部部署目錄中管理夥伴帳戶的。 在這類組織中，當您開始將應用程式移至 Azure AD 時，建議您確保夥伴可以存取其所需的資源。 資源是在內部部署還是雲端中並不重要。 此外，建議您要讓夥伴使用者能夠對內部部署和 Azure AD 資源使用相同的登入認證。 

如果您在內部部署目錄中為您的外部夥伴建立帳戶 (例如，您在 partners.contoso.com 網域中為名為 Wendy Moran 的外部使用者建立登入名稱為 "wmoran" 的帳戶)，您現在可以將這些帳戶同步至雲端。 具體來說，您可以使用 Azure AD Connect 將夥伴帳戶同步至雲端來作為 Azure AD B2B 使用者 (亦即，UserType = Guest 的使用者)。 這可讓您的夥伴使用者使用和其本機帳戶相同的認證存取雲端資源，而不需對其提供超過其所需的存取權。 

## <a name="identify-unique-attributes-for-usertype"></a>識別 UserType 的獨特屬性

在啟用 UserType 屬性的同步處理之前，您必須先決定要如何從內部部署 Active Directory 衍生 UserType 屬性。 換句話說，內部部署環境中的哪些參數是外部共同作業者所獨有的？ 請決定可供區分這些外部共同作業者與貴組織成員的參數。

針對此一目的，常見的方法有二：

- 指定未使用的內部部署 Active Directory 屬性 (例如 extensionAttribute1) 來作為來源屬性。 
- 或者，從其他屬性衍生 UserType 屬性的值。 例如，如果使用者的內部部署 Active Directory UserPrincipalName 屬性是以 *@partners.contoso.com* 網域作為結尾，您就需要以 Guest 身分來同步處理所有使用者。
 
如需詳細的屬性需求，請參閱[啟用 UserType 的同步處理](../hybrid/how-to-connect-sync-change-the-configuration.md#enable-synchronization-of-usertype)。 

## <a name="configure-azure-ad-connect-to-sync-users-to-the-cloud"></a>設定 Azure AD Connect 以將使用者同步至雲端

在識別獨有屬性後，您可以設定 Azure AD Connect 來將這些使用者同步處理至雲端以作為 Azure AD B2B 使用者 (亦即，UserType = Guest 的使用者)。 從授權的觀點來看，這些使用者與透過 Azure AD B2B 共同作業邀請程序所建立的 B2B 使用者並無不同。

如需實作指示，請參閱[啟用 UserType 的同步處理](../hybrid/how-to-connect-sync-change-the-configuration.md#enable-synchronization-of-usertype)。

## <a name="next-steps"></a>後續步驟

- [混合式組織的 Azure Active Directory B2B 共同作業](hybrid-organizations.md)
- [對 Azure AD 中的 B2B 使用者授與內部部署應用程式的存取權](hybrid-cloud-to-on-premises.md)
- 如需 Azure AD Connect 的概觀，請參閱[整合您的內部部署目錄與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)。

