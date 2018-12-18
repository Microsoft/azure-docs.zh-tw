---
title: Azure AD SAML 通訊協定參考 | Microsoft Docs
description: 這篇文章提供 Azure Active Directory 中單一登入和單一登出 SAML 設定檔的概觀。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 88125cfc-45c1-448b-9903-a629d8f31b01
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/21/2017
ms.author: celested
ms.custom: aaddev
ms.reviewer: hirsin, dastrock
ms.openlocfilehash: 067924294838459c866a0603ab092d139f1e6331
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39579226"
---
# <a name="how-azure-active-directory-uses-the-saml-protocol"></a>Azure Active Directory 使用 SAML 通訊協定的方式
Azure Active Directory (Azure AD) 會使用 SAML 2.0 通訊協定，讓應用程式能為其使用者提供單一登入體驗。 Azure AD 的[單一登入](single-sign-on-saml-protocol.md)和[單一登出](single-sign-out-saml-protocol.md) SAML 設定檔會說明如何在識別提供者服務中使用 SAML 判斷提示、通訊協定和繫結。

SAML 通訊協定需要識別提供者 (Azure AD) 和服務提供者 (應用程式)，才能交換本身的相關資訊。

向 Azure AD 註冊應用程式時，應用程式開發人員會向 Azure AD 註冊同盟的相關資訊。 此資訊包括應用程式的**重新導向 URI** 和**中繼資料 URI**。

Azure AD 會使用雲端服務的**中繼資料 URI** 來擷取簽署金鑰和登出 URI。 如果應用程式不支援中繼資料 URI，開發人員必須連絡 Microsoft 支援服務以提供登出 URI 和簽署金鑰。

Azure Active Directory 會公開租用戶專屬和一般 (租用戶獨立) 單一登入和單一登出端點。 這些 URL 代表可定址的位置 (它們不只是識別碼)，因此您可以前往端點來讀取中繼資料。

* 租用戶專屬端點位於 `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`。 *<TenantDomainName>* 預留位置代表 Azure AD 租用戶的已註冊網域名稱或 TenantID GUID。 例如，contoso.com 租用戶的同盟中繼資料在：https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* 租用戶獨立端點位於 `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`。 在此端點位址中會出現 **common**，而不是租用戶網域名稱或識別碼。

如需 Azure AD 所發佈同盟中繼資料文件的相關資訊，請參閱[同盟中繼資料](azure-ad-federation-metadata.md)。
