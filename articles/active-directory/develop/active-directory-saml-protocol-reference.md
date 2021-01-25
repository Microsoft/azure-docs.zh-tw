---
title: Microsoft 身分識別平臺如何使用 SAML 通訊協定
description: 這篇文章提供 Azure Active Directory 中單一登入和單一登出 SAML 設定檔的概觀。
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/05/2018
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 4e9d63b8fe7fc281a87deb27ddadd794e14fa04d
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98755636"
---
# <a name="how-the-microsoft-identity-platform-uses-the-saml-protocol"></a>Microsoft 身分識別平臺如何使用 SAML 通訊協定

Microsoft 身分識別平臺使用 SAML 2.0 通訊協定，讓應用程式能夠為其使用者提供單一登入體驗。 Azure AD 的[單一登入](single-sign-on-saml-protocol.md)和[單一登出](single-sign-out-saml-protocol.md) SAML 設定檔會說明如何在識別提供者服務中使用 SAML 判斷提示、通訊協定和繫結。

SAML 通訊協定需要 (Microsoft 身分識別平臺) 的身分識別提供者，以及 (應用程式) 的服務提供者，以交換本身的相關資訊。

向 Azure AD 註冊應用程式時，應用程式開發人員會向 Azure AD 註冊同盟的相關資訊。 此資訊包括應用程式的 **重新導向 URI** 和 **中繼資料 URI**。

Microsoft 身分識別平臺使用雲端服務的 **中繼資料 uri** 來取得簽署金鑰和登出 URI。 客戶可以在 [Azure AD -> 應用程式註冊] 中開啟應用程式，然後在 [設定 -> 屬性] 中更新登出 URL。 如此一來，Microsoft 身分識別平臺就可以將回應傳送至正確的 URL。 

Azure Active Directory 會公開租用戶專屬和一般 (租用戶獨立) 單一登入和單一登出端點。 這些 URL 代表可定址的位置 (它們不只是識別碼)，因此您可以前往端點來讀取中繼資料。

* 租用戶專屬端點位於 `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`。 *\<TenantDomainName>* 預留位置代表 Azure AD 租使用者的已註冊功能變數名稱或 TENANTID GUID。 例如，contoso.com 租用戶的同盟中繼資料在：https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* 租用戶獨立端點位於 `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`。 在此端點位址中會出現 **common**，而不是租用戶網域名稱或識別碼。

如需 Azure AD 發行之同盟元資料檔案的相關資訊，請參閱 [同盟中繼資料](../azuread-dev/azure-ad-federation-metadata.md)。
