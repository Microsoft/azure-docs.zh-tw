---
title: Azure Active Directory B2C 中的授權碼流程 | Microsoft Docs
description: 了解如何使用 Azure AD B2C 和 OpenID Connect 的驗證通訊協定來建置 web 應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: c6ab5ede0b8af6c601cc53e044a3e6902fbd2e11
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340805"
---
# <a name="azure-active-directory-b2c-oauth-20-authorization-code-flow"></a>Azure Active Directory B2C：OAuth 2.0 授權碼流程
在安裝於裝置上的應用程式中，您可以使用 OAuth 2.0 授權碼授與來存取受保護的資源，例如 Web API。 您可以使用 Azure Active Directory B2C (Azure AD B2C) 的 OAuth 2.0 實作，將註冊、登入及其他身分識別管理工作新增至行動及桌面應用程式。 這篇文章是與語言無關。 在本文中，我們將說明如何傳送及接收 HTTP 訊息，但不使用任何開放原始碼程式庫。

<!-- TODO: Need link to libraries -->

如需 OAuth 2.0 授權碼流程的說明，請參閱 [OAuth 2.0 規格的 4.1 節](http://tools.ietf.org/html/rfc6749)。 在大多數[應用程式類型](active-directory-b2c-apps.md) (包括 Web 應用程式和原生安裝的應用程式) 中，您都能利用它來執行驗證及授權作業。 您可以使用 OAuth 2.0 授權碼流程，安全地為您的應用程式取得存取權杖和重新整理權杖，而這些存取權杖可用來存取[授權伺服器](active-directory-b2c-reference-protocols.md)所保護的資源。  一旦存取權杖到期 (通常在一小時後)，重新整理權杖即可讓用戶端取得新的存取 (和重新整理) 權杖。

本文著重在**公開用戶端** OAuth 2.0 授權碼流程。 公開用戶端是指不可信任會安全地維護密碼完整性的任何用戶端應用程式。 這包括行動應用程式、桌面應用程式，以及基本上任何在裝置上執行且需要取得存取權杖的應用程式。 

> [!NOTE]
> 若要使用 Azure AD B2C 將身分識別管理新增至 Web 應用程式，請使用 [OpenID Connect](active-directory-b2c-reference-oidc.md)，而不是 OAuth 2.0。

Azure AD B2C 擴充標準的 OAuth 2.0 流程，功能更強大，而不僅止於簡單的驗證和授權。 它引進[原則參數](active-directory-b2c-reference-policies.md)。 透過內建原則，您可以利用 OAuth 2.0 來將使用者體驗新增至應用程式，例如註冊、登入和設定檔管理。 在本文中，我們將示範如何利用 OAuth 2.0 和原則，在您的原生應用程式中實作上述每一種體驗。 我們也會示範如何取得用來存取 Web API 的存取權杖。

在本文的範例 HTTP 要求中，我們會使用範例 Azure AD B2C 目錄 **fabrikamb2c.onmicrosoft.com**。 此外，也會使用我們的範例應用程式和原則。 您可以使用這些值來自行試驗要求，也可以將它們換成您自己的值。
了解如何[取得您自己的 Azure AD B2C 目錄、應用程式和原則](#use-your-own-azure-ad-b2c-directory)。

## <a name="1-get-an-authorization-code"></a>1.取得授權碼
授權碼流程始於用戶端將使用者導向 `/authorize` 端點。 這是使用者在流程中會採取動作的互動部分。 在此要求中，用戶端會在 `scope` 參數中指出必須向使用者索取的權限。 在 `p` 參數中，它會指出要執行的原則。 以下三個範例 (插入換行以提高可讀性) 各使用不同的原則。

### <a name="use-a-sign-in-policy"></a>使用登入原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code
&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob
&response_mode=query
&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&p=b2c_1_sign_in
```

### <a name="use-a-sign-up-policy"></a>使用註冊原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code
&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob
&response_mode=query
&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&p=b2c_1_sign_up
```

### <a name="use-an-edit-profile-policy"></a>使用編輯設定檔原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code
&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob
&response_mode=query
&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&p=b2c_1_edit_profile
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| client_id |必要 |在 [Azure 入口網站](https://portal.azure.com)中指派給應用程式的應用程式識別碼。 |
| response_type |必要 |回應類型，必須針對授權碼流程來加入 `code`。 |
| redirect_uri |必要 |應用程式的重新導向 URI，您的應用程式會在此處傳送及接收驗證回應。 除了必須是 URL 編碼，它必須與您在入口網站中註冊的其中一個重新導向 URI 完全相符。 |
| scope |必要 |範圍的空格分隔清單。 單一範圍值向 Azure Active Directory (Azure AD) 指出正在要求的兩個權限。 使用用戶端識別碼作為範圍時，表示您的應用程式需要可針對您自己的服務或 Web API 使用的存取權杖 (以相同的用戶端識別碼表示)。  `offline_access` 範圍表示您的應用程式需要重新整理權杖，才能長久存取資源。 您也可以使用 `openid` 範圍從 Azure AD B2C 要求識別碼權杖。 |
| response_mode |建議 |用來將產生的授權碼傳回至應用程式的方法。 可以是 `query`、`form_post` 或 `fragment`。 |
| state |建議 |會隨權杖回應傳回之要求中所包含的值。 它可以是您想要使用之任何內容的字串。 通常會使用隨機產生的唯一值，以防止跨網站偽造要求攻擊。 在驗證要求出現之前，也會使用此狀態將應用程式中使用者狀態的相關資訊編碼。 例如，使用者所在的頁面，或正在執行的原則。 |
| p |必要 |執行的原則。 這是在您的 Azure AD B2C 目錄中建立的原則名稱。 原則名稱值的開頭應該為**b2c\_1\_**。 若要深入了解原則，請參閱 [Azure AD B2C 內建原則](active-directory-b2c-reference-policies.md)。 |
| prompt |選用 |需要的使用者互動類型。 目前，唯一有效的值是 `login`，可強制使用者針對該要求輸入其認證。 單一登入將沒有作用。 |

此時會要求使用者完成原則的工作流程。 這可能會牽涉到讓使用者輸入自己的使用者名稱及密碼、以社交身分識別登入、註冊目錄，或是其他任何數目的步驟。 使用者動作取決於原則的定義方式。

當使用者完成原則之後，Azure AD 會透過您用於 `redirect_uri` 的值，將回應傳回給您的應用程式。 它會使用 `response_mode` 參數中指定的方法。 對於每個使用者動作情節來說，回應完全相同，與已執行的原則無關。

使用 `response_mode=query` 的成功回應如下所示：

```
GET urn:ietf:wg:oauth:2.0:oob?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...        // the authorization_code, truncated
&state=arbitrary_data_you_can_receive_in_the_response                // the value provided in the request
```

| 參數 | 說明 |
| --- | --- |
| code |應用程式所要求的授權碼。 應用程式可以使用授權碼來要求目標資源的存取權杖。 授權碼的存留期很短。 通常會在大約 10 分鐘後過期。 |
| state |如需完整說明，請參閱上一節的表格。 如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。 |

錯誤回應也能傳送到重新導向 URI，以便讓應用程式能夠適當地處理它們：

```
GET urn:ietf:wg:oauth:2.0:oob?
error=access_denied
&error_description=The+user+has+cancelled+entering+self-asserted+information
&state=arbitrary_data_you_can_receive_in_the_response
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用於將發生的錯誤類型分類的錯誤碼字串。 您也可以使用此字串對錯誤做出反應。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |
| state |如需完整說明，請參閱前一個表格。 如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。 |

## <a name="2-get-a-token"></a>2.取得權杖
既然已取得授權碼，您可以將 POST 要求傳送至 `/token` 端點，針對所需的資源來兌換權杖的 `code`。 在 Azure AD B2C 中，您唯一可以要求權杖的資源，就是應用程式本身的後端 Web API。 為您自己要求權杖時，依慣例會使用應用程式的用戶端識別碼作為範圍：

```
POST fabrikamb2c.onmicrosoft.com/oauth2/v2.0/token?p=b2c_1_sign_in HTTP/1.1
Host: https://fabrikamb2c.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob

```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| p |必要 |用來取得授權碼的原則。 您無法在此要求中使用不同的原則。 請注意，您要把這個參數新增到「查詢字串」 ，而不是 POST 主體中。 |
| client_id |必要 |在 [Azure 入口網站](https://portal.azure.com)中指派給應用程式的應用程式識別碼。 |
| grant_type |必要 |授與類型。 在授權碼流程中，授與類型必須的 `authorization_code`。 |
| scope |建議 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 使用用戶端識別碼作為範圍時，表示您的應用程式需要可針對您自己的服務或 Web API 使用的存取權杖 (以相同的用戶端識別碼表示)。  `offline_access` 範圍表示您的應用程式需要重新整理權杖，才能長久存取資源。  您也可以使用 `openid` 範圍從 Azure AD B2C 要求識別碼權杖。 |
| code |必要 |您在流程的第一個階段中取得的授權碼。 |
| redirect_uri |必要 |應用程式的重新導向 URI，您已在此處收到授權碼。 |

成功的權杖回應如下所示：

```
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```
| 參數 | 說明 |
| --- | --- |
| not_before |權杖生效的時間 (以新紀元 (Epoch) 時間表示)。 |
| token_type |權杖類型值。 Azure AD 唯一支援的類型是 Bearer。 |
| access_token |您所要求的已簽署 JSON Web 權杖 (JWT)。 |
| scope |權杖有效的範圍。 您也可以使用範圍來快取權杖，以供稍後使用。 |
| expires_in |權杖的有效時間長度 (以秒為單位)。 |
| refresh_token |OAuth 2.0 重新整理權杖。 應用程式在目前的權杖過期之後，可以使用這個權杖來取得其他權杖。 重新整理權杖是長期權杖。 您可以使用它們來長時間持續存取資源。 如需詳細資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 |

錯誤回應如下所示：

```
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用於將發生的錯誤類型分類的錯誤碼字串。 您也可以使用此字串對錯誤做出反應。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |

## <a name="3-use-the-token"></a>3.使用權杖
既然已成功取得存取權杖，在對後端 Web API 發出的要求中，您可以將權杖加入 `Authorization` 標頭中，即可使用權杖：

```
GET /tasks
Host: https://mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="4-refresh-the-token"></a>4.重新整理權杖
存取權杖和識別碼權杖的存留期短暫。 權杖過期之後，您必須重新整理權杖，才能繼續存取資源。 若要這樣做，請向 `/token` 端點提交另一個 POST 要求。 但這次要提供 `refresh_token`，而不是 `code`：

```
POST fabrikamb2c.onmicrosoft.com/oauth2/v2.0/token?p=b2c_1_sign_in HTTP/1.1
Host: https://fabrikamb2c.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&client_secret=JqQX2PNo9bpM0uEihUPzyrh&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| p |必要 |用來取得原始重新整理權杖的原則。 您無法在此要求中使用不同的原則。 請注意，您要把這個參數新增到「查詢字串」 ，而不是 POST 主體中。 |
| client_id |必要 |在 [Azure 入口網站](https://portal.azure.com)中指派給應用程式的應用程式識別碼。 |
| client_secret |必要 |在 [Azure 入口網站](https://portal.azure.com)中與 client_id 相關聯的 client_secret。 |
| grant_type |必要 |授與類型。 在授權碼流程的這個階段中，授與類型必須是 `refresh_token`。 |
| scope |建議 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 使用用戶端識別碼作為範圍時，表示您的應用程式需要可針對您自己的服務或 Web API 使用的存取權杖 (以相同的用戶端識別碼表示)。  `offline_access` 範圍表示您的應用程式需要重新整理權杖，才能長久存取資源。  您也可以使用 `openid` 範圍從 Azure AD B2C 要求識別碼權杖。 |
| redirect_uri |選用 |應用程式的重新導向 URI，您已在此處收到授權碼。 |
| refresh_token |必要 |您在流程的第二個階段中取得的原始重新整理權杖。 |

成功的權杖回應如下所示：

```
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```
| 參數 | 說明 |
| --- | --- |
| not_before |權杖生效的時間 (以新紀元 (Epoch) 時間表示)。 |
| token_type |權杖類型值。 Azure AD 唯一支援的類型是 Bearer。 |
| access_token |您所要求的已簽署 JWT。 |
| scope |權杖有效的範圍。 您也可以使用範圍來快取權杖，以供稍後使用。 |
| expires_in |權杖的有效時間長度 (以秒為單位)。 |
| refresh_token |OAuth 2.0 重新整理權杖。 應用程式在目前的權杖過期之後，可以使用這個權杖來取得其他權杖。 重新整理權杖的有效期很長，而且可以用來長期保留資源存取權。 如需詳細資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 |

錯誤回應如下所示：

```
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用於將發生的錯誤類型分類的錯誤碼字串。 您也可以使用此字串對錯誤做出反應。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |

## <a name="use-your-own-azure-ad-b2c-directory"></a>使用您自己的 Azure AD B2C 目錄
若要自行嘗試這些要求，請完成下列步驟。 使用您自己的值取代我們在本文中使用的範例值。

1. [建立 Azure AD B2C 目錄](active-directory-b2c-get-started.md)。 在要求中使用您的目錄名稱。
2. [建立應用程式](active-directory-b2c-app-registration.md)來取得應用程式識別碼和重新導向 URI。 在您的應用程式中包含原生用戶端。
3. [建立您的原則](active-directory-b2c-reference-policies.md) 來取得原則名稱。

