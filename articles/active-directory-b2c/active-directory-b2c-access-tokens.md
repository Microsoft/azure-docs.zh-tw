---
title: 在 Azure Active Directory B2C 中要求存取權杖 | Microsoft Docs
description: 本文將說明如何設定用戶端應用程式，並取得存取權杖。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/09/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 9cd5789cd2ee6e167f3d3ed05c2fde077f7ec9a3
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344936"
---
# <a name="azure-ad-b2c-requesting-access-tokens"></a>Azure AD B2C︰要求存取權杖

存取權杖 (在 Azure AD B2C 的回應中表示為 **access\_token**) 是一種安全性權杖形式，用戶端可用來存取受[授權伺服器](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-protocols#the-basics)保護的資源，例如 web API。 存取權杖統稱為[JWT](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-tokens#types-of-tokens)，且包含關於預期資源伺服器和伺服器授與權限的資訊。 在呼叫資源伺服器時，存取權杖必須出現在 HTTP 要求中。

本文將討論如何設定用戶端應用程式與 Web API 以取得 **access\_token**。

> [!NOTE]
> **Azure AD B2C 不支援 Web API 鏈結 (代理者)。**
>
> 許多架構中都有一個 Web API 需要呼叫另一個下游 Web API，而兩者都受 Azure AD B2C 保護。 此情況常見於有 Web API 後端的原生用戶端，而後端會再呼叫 Azure AD 圖形 API 等 Microsoft 線上服務。
>
> 使用 OAuth 2.0 JWT 持有人認證授與可支援此鏈結的 Web API 案例，亦稱為「代理者流程」。 不過，Azure AD B2C 目前未實作代理者流程。

## <a name="register-a-web-api-and-publish-permissions"></a>註冊 Web API 與發佈權限

要求存取權杖之前，您必須先註冊 Web API 和發佈可以授與用戶端應用程式的權限 (範圍)。

### <a name="register-a-web-api"></a>註冊 Web API

1. 在 Azure 入口網站的 Azure AD B2C 功能功能表中，按一下 [應用程式]。
1. 按一下功能表頂端的 [+ 新增]。
1. 輸入應用程式的 **名稱** ，此名稱將會為取用者說明您的應用程式。 例如，您可以輸入「Contoso API」。
1. 將 [包括 web 應用程式 / web API] 切換為 [是]。
1. 在 [回覆 URL] 輸入任意值。 例如，輸入 `https://localhost:44316/`。 這個值無關緊要，因為 API 不應該直接從 Azure AD B2C 接收權杖。
1. 輸入 [應用程式識別碼 URI]。 這是您的 Web API 所使用的識別碼。 例如，在方塊中輸入「notes」。 [App ID URI] 即會顯示為 `https://{tenantName}.onmicrosoft.com/notes`。
1. 按一下 [建立]  以註冊您的應用程式。
1. 按一下您剛才建立的應用程式，並複製稍後要在程式碼中使用的全域唯一 **應用程式用戶端識別碼** 。

### <a name="publishing-permissions"></a>發佈權限

範圍類似權限，當應用程式在呼叫 API 時，範圍是不可或缺的。 舉例來說，「讀取」或「寫入」即是範圍。 假設您想要您的 Web 或原生應用程式從 API 「讀取」。 您的應用程式會呼叫 Azure AD B2C 並要求一個存取權杖，該權杖會將存取權授與「讀取」範圍。 為了讓 Azure AD B2C 發出這類存取權杖，必須將可從特定 API 「讀取」的權限授與該應用程式。 因此，您的 API 必須先發佈「讀取」範圍。

1. 在 Azure AD B2C [應用程式] 功能表中，開啟 Web API 應用程式 ("Contoso API")。
1. 按一下 [已發佈範圍]。 您會在此定義可授予其他應用程式的權限 (範圍)。
1. 視需要新增 [範圍值] \(例如「讀取」)。 根據預設，將會定義 "user_impersonation" 範圍。 想要的話，也可以忽略此步驟。 在 [範圍名稱] 資料行中輸入範圍的描述。
1. 按一下 [檔案] 。

> [!IMPORTANT]
> [範圍名稱] 是 [範圍值] 的描述。 當使用範圍時，請務必使用 [範圍值]。

## <a name="grant-a-native-or-web-app-permissions-to-a-web-api"></a>將原生或 Web 應用程式的權限授與 Web API

設定好 API 發佈範圍後，必須透過 Azure 入口網站將這些範圍授與用戶端應用程式。

1. 瀏覽至 Azure AD B2C 功能功能表中的 [應用程式] 功能表。
1. 如果您還沒有用戶端應用程式 ([Web 應用程式](active-directory-b2c-app-registration.md#register-a-web-app)或[原生用戶端](active-directory-b2c-app-registration.md#register-a-mobile-or-native-app))，請註冊一個。 如果您一開始便是跟隨本指南的說明進行，則需要註冊用戶端應用程式。
1. 按一下 [API 存取]。
1. 按一下 [新增]。
1. 選取您的 web API，以及您想要授與的範圍 (權限)。
1. 按一下 [確定]。

> [!NOTE]
> Azure AD B2C 不會要求您的用戶端應用程式使用者同意。 相反地，根據上述應用程式之間設定的權限，所有同意都係由系統管理員提供。 如果已撤銷應用程式的權限授與，所有先前能夠取得該權限的使用者將不再能夠執行這項操作。

## <a name="requesting-a-token"></a>要求權杖

在要求存取權杖時，用戶端應用程式必須在要求的 [scope] 參數中指定所需的權限。 例如，若要將 [App ID URI] 為 `https://contoso.onmicrosoft.com/notes` 的 API 的 [範圍值] 指定為 [read]，則範圍為 `https://contoso.onmicrosoft.com/notes/read`。 以下是對 `/authorize` 端點授權碼要求的範例。

> [!NOTE]
> 目前，自訂網域並未和存取權杖一起受到支援。 您必須在要求 URL 中使用您的 tenantName.onmicrosoft.com 網域。

```
https://<tenantName>.b2clogin.com/tfp/<tenantName>.onmicrosoft.com/<yourPolicyId>/oauth2/v2.0/authorize?client_id=<appID_of_your_client_application>&nonce=anyRandomValue&redirect_uri=<redirect_uri_of_your_client_application>&scope=https%3A%2F%2Fcontoso.onmicrosoft.com%2Fnotes%2Fread&response_type=code 
```

要在相同的要求中取得多個權限，您可以在單一**範圍**參數中新增多個項目，以空格分隔。 例如︰

解碼的 URL：

```
scope=https://contoso.onmicrosoft.com/notes/read openid offline_access
```

編碼的 URL：

```
scope=https%3A%2F%2Fcontoso.onmicrosoft.com%2Fnotes%2Fread%20openid%20offline_access
```

您可能會要求比授與用戶端應用程式更多的資源範圍 (權限)。 在此情況時，如果至少授與一個權限，則呼叫會成功。 所產生的**存取\_權杖**會僅使用已成功授與的權限填入其 “scp” 宣告。

> [!NOTE] 
> 我們在相同的要求中不支援兩個不同 web 資源要求的權限。 這種要求將會失敗。

### <a name="special-cases"></a>特殊案例

OpenID Connect 標準會指定數個特殊的「範圍」值。 下列特殊範圍代表「存取使用者設定檔」的權限︰

* **openid**︰這會要求識別碼權杖
* **offline\_access**︰這會要求重新整理權杖 (使用[授權碼流程](active-directory-b2c-reference-oauth-code.md))。

如果 `/authorize` 要求中的 `response_type` 參數包含 `token`，則 `scope` 參數必須包含至少一個會授與的資源範圍 (`openid` 與 `offline_access` 以外)。 否則，`/authorize` 要求將會結束並發生失敗。

## <a name="the-returned-token"></a>傳回的權杖

在成功產生**存取\_權杖** (從 `/authorize` 或 `/token` 端點)，會出現下列宣告︰

| 名稱 | 宣告 | 說明 |
| --- | --- | --- |
|對象 |`aud` |權杖授與存取權的單一資源之應用程式識別碼。 |
|影響範圍 |`scp` |授與給資源的權限。 多個授與權限將會以空格隔開。 |
|授權的合作對象 |`azp` |起始要求之用戶端應用程式的應用程式識別碼。 |

當您的 API 收到**存取\_權杖**時，它必須[驗證權杖](active-directory-b2c-reference-tokens.md)以證明權杖的真實性，並具有正確的宣告。

我們歡迎意見反應和建議！ 如果您對本主題有任何問題，請利用標記 ['azure-ad-b2c'](https://stackoverflow.com/questions/tagged/azure-ad-b2c) 並張貼至 Stack Overflow。 對於功能要求，請將它們新增到 [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160596-b2c)。

## <a name="next-steps"></a>後續步驟

* 使用 [.NET Core](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapi) 建置 Web API
* 使用 [Node.JS](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) 建置 Web API
