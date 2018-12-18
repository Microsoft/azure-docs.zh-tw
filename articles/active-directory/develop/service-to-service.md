---
title: Azure Active Directory 中的服務對服務應用程式
description: 描述服務對服務應用程式，以及此應用程式類型的通訊協定流程、註冊與權杖到期的基本概念。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ms.openlocfilehash: 8365f7e60dfb78aba16bea9708c99195a6b76b11
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972890"
---
# <a name="service-to-service-apps"></a>服務對服務應用程式

服務對服務應用程式可以是需要從 Web API 取得資源的精靈或伺服器應用程式。 有兩個子案例適用於本節：

- 需要呼叫 Web API 且以 OAuth 2.0 用戶端認證授與類型為基礎的精靈

    在此案例中，請務必了解幾件事。 首先，使用者無法與精靈應用程式互動，應用程式必須有自己的身分識別。 像是批次工作，或在背景執行的作業系統服務，就是精靈應用程式的例子。 這類應用程式會使用其應用程式識別，並向 Azure AD 出示其應用程式識別碼、認證 (密碼或憑證) 和應用程式識別碼 URI，以要求存取權杖。 成功驗證之後，精靈會從 Azure AD 收到存取權杖，然後用來呼叫 Web API。

- 伺服器應用程式 (例如 Web API)，需要呼叫 Web API 且以 OAuth 2.0 代理者草稿規格為基礎

    在此案例中，假設使用者已在原生應用程式上驗證，而此原生應用程式需要呼叫 Web API。 Azure AD 會發出 JWT 存取權杖來呼叫 Web API。 如果 Web API 需要呼叫另一個下游 Web API，它可以使用代表流程來委派使用者的身分識別，並向第二層 Web API 驗證。

## <a name="diagram"></a>圖表

![精靈或伺服器應用程式到 Web API 圖表](./media/authentication-scenarios/daemon_server_app_to_web_api.png)

## <a name="dprotocol-flow"></a>DProtocol 流程

### <a name="application-identity-with-oauth-20-client-credentials-grant"></a>採用 OAuth 2.0 用戶端認證授與的應用程式識別碼

1. 首先，伺服器應用程式本身必須向 Azure AD 驗證，沒有任何人為互動，例如互動式登入對話方塊。 它向 Azure AD 的權杖端點提出要求，並提供認證、應用程式識別碼和應用程式識別碼 URI。
1. Azure AD 驗證應用程式，並傳回用來呼叫 Web API 的 JWT 存取權杖。
1. Web 應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。 接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。

### <a name="delegated-user-identity-with-oauth-20-on-behalf-of-draft-specification"></a>採用 OAuth 2.0 On-Behalf-Of 草稿規格的委派使用者識別

下面討論的流程假設使用者已在另一個應用程式上驗證 (例如，原生應用程式)，且其使用者識別已用來取得第一層 Web API 的存取權杖。

1. 原生應用程式將存取權杖傳送到第一層 Web API。
1. 第一層 Web API 傳送要求至 Azure AD 的權杖端點，並提供其應用程式識別碼和認證，以及使用者的存取權杖。 此外，傳送的要求包含 on_behalf_of 參數，指出 Web API 正在代表原始使用者要求新權杖來呼叫下游 Web API。
1. Azure AD 確認第一層 Web API 有權存取第二層 Web API，並驗證要求，然後傳回 JWT 存取權杖和 JWT 重新整理權杖給第一層 Web API。
1. 接著，第一層 Web API 透過 HTTPS，在要求的 Authorization 標頭中附加權杖字串，以呼叫第二層 Web API。 只要存取權杖和重新整理權杖有效，第一層 Web API 就可以繼續呼叫第二層 Web API。

## <a name="code-samples"></a>程式碼範例

請參閱「精靈或伺服器應用程式到 Web API」案例的程式碼範例。 歡迎您經常回來看看，因為我們會隨時新增新的範例。 [伺服器或精靈應用程式到 Web API](sample-v1-code.md#daemon-applications-accessing-web-apis-with-the-applications-identity)

## <a name="app-registration"></a>應用程式註冊

* 單一租用戶 - 無論是應用程式識別或委派的使用者識別的情況，精靈或伺服器應用程式都必須註冊在 Azure AD 的相同目錄中。 Web API 可以設定為公開一組權限，用以限制精靈或伺服器應用程式對其資源的存取權。 如果使用委派的使用者識別類型，伺服器應用程式需要從 Azure 入口網站的 [其他應用程式的權限] 下拉式功能表中，選取所需的權限。 如果使用應用程式識別類型，則不需要此步驟。
* 多租用戶 - 首先，設定精靈或伺服器應用程式來指出它運作所需的權限。 當目的地目錄中的使用者或系統管理員同意應用程式時 (使得應用程式可供組織使用)，這份必要權限清單會顯示在對話方塊中。 有些應用程式只需要使用者層級權限，亦即組織中的任何使用者都可以同意應用程式。 其他應用程式需要系統管理員層級權限，亦即組織中的使用者無法同意應用程式。 只有目錄管理員才能對需要此權限層級的應用程式表示同意。 當使用者或系統管理員同意時，這兩個 Web API 都會註冊在他們的目錄中。

## <a name="token-expiration"></a>權杖到期

當第一個應用程式使用其授權碼來取得 JWT 存取權杖時，它也會收到 JWT 重新整理權杖。 當存取權杖到期時，重新整理權杖可用來重新驗證使用者，而不會提示輸入認證。 然後，此重新整理權杖用來驗證使用者，結果會產生新的存取權杖和重新整理權杖。

## <a name="next-steps"></a>後續步驟

- 深入了解其他[應用程式類型和案例](app-types.md)
- 深入了解 Azure AD [驗證基本概念](authentication-scenarios.md)
