---
title: Azure Active Directory B2C 中的驗證通訊協定 | Microsoft Docs
description: 如何直接使用 Azure Active Directory B2C 支援的通訊協定來建置應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 01/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6cfa3e0cb23f13b50c0fb7cd4f07a8d53c0f3186
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340761"
---
# <a name="azure-ad-b2c-authentication-protocols"></a>Azure AD B2C：驗證通訊協定
Azure Active Directory B2C (Azure AD B2C) 支援 OpenID Connect 與 OAuth 2.0 兩種業界標準通訊協定，為您的 app 提供身分識別即服務。 這是符合標準的服務，但是這些通訊協定在任兩個實作之間仍會有些微差異。 

若您藉由直接傳送和處理 HTTP 要求來撰寫程式碼，而非使用開放原始碼程式庫，則本指南中的資訊很有用。 建議您在深入探討每個特定通訊協定的詳細資料之前，先閱讀此頁面。 但若您已經熟悉 Azure AD B2C，您可以直接閱讀[通訊協定參考指南](#protocols)。

<!-- TODO: Need link to libraries above -->

## <a name="the-basics"></a>基本概念
使用 Azure AD B2C 的每個 app 都必須在 [Azure 入口網站](https://portal.azure.com)內您的 B2C 目錄中註冊。 App 註冊處理序會收集與指派一些值給您的 app：

* 可唯一識別應用程式的 **應用程式識別碼** 。
* 可將回應導回至應用程式的**重新導向 URI**或**套件識別碼**。
* 其他幾個狀況特定的值。 如需詳細資訊，請了解[如何註冊您的應用程式](active-directory-b2c-app-registration.md)。

註冊您的應用程式之後，它會傳送要求給端點，藉此與 Azure Active Directory (Azure AD) 通訊：

```
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/oauth2/v2.0/authorize
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/oauth2/v2.0/token
```

幾乎在所有的 OAuth 和 OpenID Connect 流程中，都包含四個參與交換的合作對象：

![OAuth 2.0 角色](./media/active-directory-b2c-reference-protocols/protocols_roles.png)

* **授權伺服器**是 Azure AD 端點。 它會安全地處理與使用者資訊和存取相關的任何項目。 它也會處理流程中合作對象之間的信任關係。 其負責驗證使用者的身分識別、授與及撤銷資源存取權，以及核發權杖。 它亦稱為身分識別提供者。

* **資源擁有者** 通常是使用者。 其是擁有資料的一方，而且有權允許第三方存取該資料或資源。

* **OAuth 用戶端** 是您的應用程式。 它是透過其應用程式識別碼來識別。 它通常是使用者互動的合作對象。 它也會向授權伺服器要求權杖。 資源擁有者必須授與用戶端者授權，才能存取資源。

* **資源伺服器** 是資源或資料所在位置。 它會信任授權伺服器，以便安全地驗證和授權 OAuth 用戶端。 它也會使用持有人存取權杖，以確保可授與資源的存取權。

## <a name="policies"></a>原則
Azure AD B2C 原則可視為服務的最重要功能。 Azure AD B2C 藉由引進原則來延伸標準的 OAuth 2.0 和 OpenID Connect 通訊協定。 這些原則讓 Azure AD B2C 能夠執行簡單驗證與授權以外的更多操作。 

原則可完整描述取用者身分識別體驗，包括註冊、登入及設定檔編輯。 原則可定義於系統管理 UI 中。 您可以在 HTTP 驗證要求中使用特定的查詢參數來執行原則。 

原則並非 OAuth 2.0 和 OpenID Connect 的標準功能，因此您應該花點時間瞭解它們。 如需詳細資訊，請參閱 [Azure AD B2C 原則參考指南](active-directory-b2c-reference-policies.md)。

## <a name="tokens"></a>權杖
OAuth 2.0 和 OpenID Connect 的 Azure AD B2C 實作廣泛運用持有人權杖，包括以 JSON Web 權杖 (JWT) 表示的持有人權杖。 持有人權杖是輕巧型安全性權杖，授權「持有人」存取受保護的資源。

持有人是可出示權杖的任何一方。 Azure AD 必須先驗證合作對象，才能接收持有人權杖。 但若傳輸和儲存時未採取必要的步驟來保護權杖，它可能會被非預期的一方攔截和使用。

某些安全性權杖有內建的機制，可防止未授權的合作對象使用它們，但持有人權杖沒有這項機制。 它們必須在安全的通道 (例如傳輸層安全性 (HTTPS)) 中傳輸。 

如果持有人權杖是在安全通道外部進行傳輸，則惡意人士就能使用攔截式攻擊來取得權杖，未經授權地使用該權杖來存取受保護的資源。 儲存或快取持有人權杖供以後使用時，也適用相同的安全性原則。 務必確定您的應用程式以安全的方式傳輸和儲存持有人權杖。

如需持有人權杖的其他安全性考量，請參閱 [RFC 6750 第 5 節](http://tools.ietf.org/html/rfc6750)。

如需 Azure AD B2C 中所用各種不同權杖類型的詳細資訊，請參閱 [Azure AD 權杖參考](active-directory-b2c-reference-tokens.md)。

## <a name="protocols"></a>通訊協定
當您準備好要檢閱一些範例要求時，您可以開從下列其中一個教學課程開始。 每個教學課程皆對應至特定的驗證情節。 若您在判斷適用的流程時需要協助，請參閱[您可以使用 Azure AD B2C 建置的應用程式類型](active-directory-b2c-apps.md)。

* [使用 OAuth 2.0 建置行動與原生應用程式](active-directory-b2c-reference-oauth-code.md)
* [使用 OpenID Connect 建置 Web 應用程式](active-directory-b2c-reference-oidc.md)
* [使用 OAuth 2.0 隱含流程建置單一頁面應用程式](active-directory-b2c-reference-spa.md)

