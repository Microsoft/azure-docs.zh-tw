---
title: Azure AD 同盟相容性清單
description: 此頁面包含可用來實作單一登入的非 Microsoft 識別提供者。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: curtand
ms.assetid: 22c8693e-8915-446d-b383-27e9587988ec
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/23/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 4a31e7ed12e75bd7dee3d53a8d650179d8e118d8
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304497"
---
# <a name="azure-ad-federation-compatibility-list"></a>Azure AD 同盟相容性清單
Azure Active Directory 在不需要任何第三方解決方案的情況下，針對 Office 365 和混合式與僅限雲端實作的其他 Microsoft 線上服務，提供單一登入和增強的應用程式存取安全性。 Office 365，就像大部分 Microsoft 線上服務一樣，已經與 Azure Active Directory 整合以提供目錄服務、驗證及授權。 Azure Active Directory 也為數千個 SaaS 應用程式和內部部署 Web 應用程式提供單一登入。 請參閱 Azure Active Directory [應用程式庫](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps)以了解支援的 SaaS 應用程式有哪些。 

## <a name="idp-validation"></a>IDP 驗證
如果貴組織使用第三方同盟解決方案，您可以使用 Microsoft Online Services (例如，Office 365) 為內部部署 Active Directory 使用者設定單一登入，但前提是第三方同盟解決方案可與 Azure Active Directory 相容。  如有關於相容性的問題，請連絡識別提供者。  如果您想要查看先前已通過 Microsoft 所進行的 Azure AD 相容性測試的識別提供者清單，請按一下[這裡](https://www.microsoft.com/download/details.aspx?id=56843)。 

>[!NOTE]
>Microsoft 不會再對獨立識別提供者提供 Azure Active Directory 相容性驗證測試。 如果您想要測試您的產品是否具有互通性，請參閱這些[指導方針](https://www.microsoft.com/download/details.aspx?id=56843)。 

## <a name="next-steps"></a>後續步驟

- [整合您的內部部署目錄與 Azure Active Directory](whatis-hybrid-identity.md)
- [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)
