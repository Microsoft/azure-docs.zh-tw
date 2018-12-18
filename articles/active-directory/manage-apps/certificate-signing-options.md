---
title: 適用於 Azure Active Directory 中預先整合之應用程式的 SAML 權杖的進階憑證簽署選項 | Microsoft Docs
description: 瞭解如何在 Azure Active Directory 中，針對預先整合之應用程式使用 SAML 權杖中的進階憑證簽署選項
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: barbkess
ms.reviewer: jeedes
ms.custom: aaddev
ms.openlocfilehash: e7488abb3e82f90f63fa338b84a6516202e504ec
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44714525"
---
# <a name="advanced-certificate-signing-options-in-the-saml-token-for-gallery-apps-in-azure-active-directory"></a>在 Azure Active Directory 中，針對資源庫應用程式使用 SAML 權杖中的進階憑證簽署選項
現今 Azure Active Directory (Azure AD) 在 Azure Active Directory 應用程式庫中支援數千種預先整合的應用程式。 其中包括超過 500 個使用 SAML 2.0 通訊協定支援單一登入的應用程式。 當使用者使用 SAML 透過 Azure AD 驗證應用程式時，Azure AD 會將權杖傳送給應用程式 (透過 HTTP POST)。 然後，應用程式會驗證並使用權杖將使用者登入，而不會提示輸入使用者名稱和密碼。 這些 SAML 權杖會使用 Azure AD 中產生的唯一憑證以及特定標準演算法產生的唯一憑證加以簽署。

Azure AD 為資源庫應用程式使用某些預設設定。 預設值是設定根據應用程式需求設定的。

Azure AD 支援進階憑證簽署設定。 若要選取這些選項，請先選取 [顯示進階憑證簽署設定] 核取方塊：

![顯示進階憑證簽署設定](./media/certificate-signing-options/saml-advance-certificate.png)

選取此核取方塊之後，您可以設定憑證簽署選項和憑證簽署演算法。

## <a name="certificate-signing-options"></a>憑證簽署選項

Azure AD 支援三個憑證簽署選項：

* **簽署 SAML 判斷提示**。 此預設選項是針對大部分的資源庫應用程式而設定的。 如果選取此選項，則作為 IdP 的 Azure AD 會使用應用程式的 X509 憑證簽署 SAML 判斷提示及憑證。 它也會使用可從 [簽署演算法] 下列下拉式清單中選取的簽署演算法。

* **簽署 SAML 回應**。 如果選取此選項，則作為 IdP 的 Azure AD 會使用應用程式的 X509 憑證簽署 SAML 回應。 它也會使用可從 [簽署演算法] 下列下拉式清單中選取的簽署演算法。

* **簽署 SAML 回應及判斷提示**。 如果選取此選項，則作為 IdP 的 Azure AD 會使用應用程式的 X509 憑證簽署整個 SAML 權杖。 它也會使用可從 [簽署演算法] 下列下拉式清單中選取的簽署演算法。

    ![憑證簽署選項](./media/certificate-signing-options/saml-signing-options.png)

## <a name="certificate-signing-algorithms"></a>憑證簽署演算法

Azure AD 支援兩種簽署 SAML 回應的簽署演算法：

* **SHA-256**。 Azure AD 使用這個預設演算法簽署 SAML 回應。 這是最新的演算法，而且被視為比 SHA-1 更安全。 大部分的應用程式都支援 SHA-256 演算法。 如果應用程式僅支援 SHA-1 作為簽署演算法，您可以加以變更。 否則，我們建議使用 SHA-256 演算法簽署 SAML 回應。

    ![SHA-256 憑證簽署演算法](./media/certificate-signing-options/saml-signing-algo-sha256.png)

* **SHA-1**。 這是較舊的演算法，而且被視為比 SH-256 更不安全。 如果應用程式僅支援此簽署演算法，您可以在 [簽署演算法] 下拉式清單中選取此選項。 接著，Azure AD 會使用 SHA-1 演算法簽署 SAML 回應。

    ![SHA-1 憑證簽署演算法](./media/certificate-signing-options/saml-signing-algo-sha1.png)

## <a name="next-steps"></a>後續步驟
* [設定對不在 Azure Active Directory 應用程式庫中的應用程式的單一登入](configure-federated-single-sign-on-non-gallery-applications.md)
* [針對 SAML 型單一登入進行疑難排解](../develop/howto-v1-debug-saml-sso-issues.md)


