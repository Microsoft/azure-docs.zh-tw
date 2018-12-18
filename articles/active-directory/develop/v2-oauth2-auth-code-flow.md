---
title: Azure AD v2.0 OAuth 授權碼流程 | Microsoft Docs
description: 使用 Azure AD 實作 OAuth 2.0 驗證通訊協定建置 Web 應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: ae1d7d86-7098-468c-aa32-20df0a10ee3d
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: celested
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: d94aaa93596a18cf92b745267a6be9966454e36f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971534"
---
# <a name="v20-protocols---oauth-20-authorization-code-flow"></a>v2.0 通訊協定 - OAuth 2.0 授權碼流程

OAuth 2.0 授權碼授與可用於裝置上所安裝的應用程式中，以存取受保護的資源，例如 Web API。 透過應用程式模型的 v2.0 實作 OAuth 2.0，您可以將登入及 API 存取新增至您的行動應用程式和桌面應用程式。 本指南不限於特定語言，其中說明如何在不使用任何 [Azure 開放原始碼驗證程式庫](active-directory-authentication-libraries.md)的情況下，傳送和接收 HTTP 訊息。

> [!NOTE]
> v2.0 端點並非支援每個 Azure Active Directory 案例和功能。 若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](active-directory-v2-limitations.md)。

如需 OAuth 2.0 授權碼流程的說明，請參閱 [OAuth 2.0 規格的 4.1 節](http://tools.ietf.org/html/rfc6749)。 在大部分的應用程式類型中，其用於執行驗證與授權，包括 [Web Apps](v2-app-types.md#web-apps) 和[原始安裝的應用程式](v2-app-types.md#mobile-and-native-apps)。 此流程可讓應用程式安全地取得可用來存取受 v2.0 端點保護之資源的 access_token。 

## <a name="protocol-diagram"></a>通訊協定圖表

概括而言，原生/行動應用程式的整個驗證流程看起來像是這樣：

![OAuth 授權碼流程](./media/v2-oauth2-auth-code-flow/convergence_scenarios_native.png)

## <a name="request-an-authorization-code"></a>要求授權碼

授權碼流程始於用戶端將使用者導向 `/authorize` 端點。 在這項要求中，用戶端會指出必須向使用者索取的權限：

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&state=12345
```

> [!TIP]
> 按一下下面的連結以執行此要求！ 登入之後，您的瀏覽器應重新導向至在位址列中有 `code` 的 `https://localhost/myapp/`。
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&state=12345" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| 參數             |             | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-----------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| tenant                | 必要    | 要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。 允許的值為 `common`、`organizations`、`consumers` 及租用戶識別碼。 如需更多詳細資訊，請參閱 [通訊協定基本概念](active-directory-v2-protocols.md#endpoints)。                                                                                                                                                                                                                                                                                                                    |
| client_id             | 必要    | 註冊入口網站 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)) 指派給應用程式的應用程式識別碼。  |
| response_type         | 必要    | 授權碼流程必須包含 `code`。       |
| redirect_uri          | 建議使用 | 應用程式的 redirect_uri，您的應用程式可在此傳送及接收驗證回應。 其必須完全符合您在入口網站中註冊的其中一個 redirect_uris，不然就必須得是編碼的 url。 對於原生和行動應用程式，請使用 `https://login.microsoftonline.com/common/oauth2/nativeclient` 的預設值。   |
| scope                 | 必要    | 您要使用者同意的 [範圍](v2-permissions-and-consent.md) 空格分隔清單。           |
| response_mode         | 建議使用 | 指定將產生的權杖送回到應用程式所應該使用的方法。 可以是 `query`、`fragment` 或 `form_post`。 `query` 會提供程式碼，以作為重新導向 URI 的查詢字串參數。 如果您要求可使用隱含流程的識別碼權杖，就無法使用 [OpenID 規格](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations)中指定的 `query`。如果您只要求程式碼，您可以使用 `query`、`fragment` 或 `form_post`。 `form_post` 會執行 POST，其中包含您重新導向 URI 的程式碼。 如需詳細資訊，請參閱 [OpenID Connect 通訊協定](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-openid-connect-code)。  |
| state                 | 建議使用 | 同樣會隨權杖回應傳回之要求中所包含的值。 其可以是您想要之任何內容的字串。 隨機產生的唯一值通常用於 [防止跨站台要求偽造攻擊](http://tools.ietf.org/html/rfc6749#section-10.12)。 此值也可以將驗證要求發生前使用者在應用程式中的狀態相關資訊 (例如他們所在的網頁或檢視) 編碼。 |
| prompt                | 選用    | 表示需要的使用者互動類型。 此時的有效值為「登入」、「無」和「同意」。 `prompt=login` 會強制使用者在該要求上輸入認證，否定單一登入。 `prompt=none` 則相反 - 它會確保不會對使用者顯示任何互動式提示。 如果要求無法透過單一登入以無訊息方式完成，v2.0 端點會傳回 `interaction_required` 錯誤。 `prompt=consent` 會在使用者登入之後觸發 OAuth 同意對話方塊，詢問使用者是否要授與權限給應用程式。 |
| login_hint            | 選用    | 如果您事先知道其使用者名稱，可用來預先填入使用者登入頁面的使用者名稱/電子郵件地址欄位。 通常應用程式會在重新驗證期間使用此參數，已經使用 `preferred_username` 宣告從上一個登入擷取使用者名稱。                                                                                                                                                                                                                                                                                                    |
| domain_hint           | 選用    | 可以是 `consumers` 或 `organizations` 其中一個。 如果包含，它會略過使用者在 v2.0 登入頁面上所經歷以電子郵件為基礎的探索程序，進而提供略為更有效率的使用者體驗。 通常應用程式會在重新驗證 (擷取上一次登入的 `tid` ) 期間使用此參數。 如果 `tid` 宣告值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，您應該使用 `domain_hint=consumers`。 否則，使用 `domain_hint=organizations`。                                                                                                              |
| code_challenge_method | 選用    | 用來為 `code_challenge` 參數編碼 `code_verifier` 的方法。 可以是 `plain` 或 `S256` 其中一個。 如果排除，則當包含 `code_challenge` 時，會假設 `code_challenge` 是純文字。 Azure AAD v2.0 同時支援 `plain` 和 `S256`。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |
| code_challenge        | 選用    | 用來透過來自原生用戶端的「代碼交換的證明金鑰」(PKCE) 保護授權碼授與。 如果包含 `code_challenge_method`，則為必要參數。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。                                                                                                                                                                                                                                                                                                                                                                    |

此時，會要求使用者輸入其認證並完成驗證。 v2.0 端點也會確保使用者已經同意 `scope` 查詢參數所示的權限。 如果使用者未曾同意這些權限的任何一項，就會要求使用者同意要求的權限。 [這裡提供權限、同意與多租用戶應用程式](v2-permissions-and-consent.md)的詳細資料。

一旦使用者驗證並同意，v2.0 端點就會使用 `response_mode` 參數中指定的方法，將回應傳回至位於指定所在 `redirect_uri` 的應用程式。

#### <a name="successful-response"></a>成功回應

使用 `response_mode=query` 的成功回應如下所示：

```
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=12345
```

| 參數 | 說明                                                                                                                                                                                                                        |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code      | 應用程式要求的 authorization_code。 應用程式可以使用授權碼要求目標資源的存取權杖。 Authorization_code 的有效期很短，通常約 10 分鐘後即到期。 |
| state     | 如果要求中包含狀態參數，回應中就應該出現相同的值。 應用程式應該確認要求和回應中的狀態值完全相同。                                            |

#### <a name="error-response"></a>錯誤回應

錯誤回應可能也會傳送至 `redirect_uri` ，讓應用程式可以適當地處理：

```
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| 參數         | 說明                                                                                                       |
|-------------------|-------------------------------------------------------------------------------------------------------------------|
| 錯誤             | 用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error_description | 協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。          |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>授權端點錯誤的錯誤碼

下表說明各種可能在錯誤回應的 `error` 參數中傳回的錯誤碼。

| 錯誤碼                | 說明                                                                                                           | 用戶端動作                                                                                                                                                                                                                               |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| invalid_request           | 通訊協定錯誤，例如遺漏必要的參數。                                                               | 修正並重新提交要求。 這是通常會在初始測試期間擷取到的開發錯誤。                                                                                                                                     |
| unauthorized_client       | 不允許用戶端應用程式要求授權碼。                                           | 此錯誤通常會在用戶端應用程式未在 Azure AD 中註冊，或未新增至使用者的 Azure AD 租用戶時發生。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |
| access_denied             | 資源擁有者拒絕同意                                                                                         | 用戶端應用程式可以通知使用者，除非使用者同意，否則無法繼續進行。                                                                                                                                               |
| unsupported_response_type | 授權伺服器不支援要求中的回應類型。                                         | 修正並重新提交要求。 這是通常會在初始測試期間擷取到的開發錯誤。                                                                                                                                     |
| server_error              | 伺服器發生非預期的錯誤。                                                                         | 重試要求。 這些錯誤可能是由暫時性狀況所引起。 用戶端應用程式可能會向使用者解釋其回應因暫時性錯誤而延遲。                                                                |
| temporarily_unavailable   | 伺服器暫時過於忙碌而無法處理要求。                                                           | 重試要求。 用戶端應用程式可能會向使用者解釋，說明其回應因暫時性狀況而延遲。                                                                                                               |
| invalid_resource          | 目標資源無效，因為它不存在、Azure AD 無法找到它，或是它並未正確設定。 | 此錯誤表示尚未在租用戶中設定資源 (如果存在)。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。                                          |
|login_required             | 找到太多使用者或找不到使用者 | 用戶端已要求無訊息驗證 (`prompt=none`)，但找不到單一使用者。 這可能表示工作階段中有多個作用中使用者，或沒有任何使用者。 這會將選擇的租用戶列入考慮 (例如，如果有 2 個使用中的 AAD 帳戶和一個 MSA，且選擇了 `consumers`，無訊息驗證即可運作)。 |
|interaction_required       | 要求需要使用者互動。 | 必須進行其他驗證步驟或同意。 請在不使用 `prompt=none` 的情況下重試要求。 |

## <a name="request-an-access-token"></a>要求存取權杖

取得 authorization_code 並獲得使用者授權之後，現在即可兌換所需資源之 `access_token` 的 `code`。 做法是將 `POST` 要求傳送給 `/token` 端點：

```
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=authorization_code
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps
```

> [!TIP]
> 嘗試在 Postman 中執行這項要求！ (別忘了取代 `code`) [![在 Postman 中執行](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)

| 參數     |                       | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| tenant        | 必要              | 要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。 允許的值為 `common`、`organizations`、`consumers` 及租用戶識別碼。 如需更多詳細資訊，請參閱 [通訊協定基本概念](active-directory-v2-protocols.md#endpoints)。                                                                                                                                                   |
| client_id     | 必要              | 註冊入口網站 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)) 指派給應用程式的應用程式識別碼。                                                                                                                                                                                                                             |
| grant_type    | 必要              | 必須是授權碼流程的 `authorization_code` 。                                                                                                                                                                                                                                                                                                                                                                            |
| scope         | 必要              | 範圍的空格分隔清單。 在此階段中要求的範圍必須相當於或為第一個階段中所要求的範圍子集。 如果這個要求中指定的範圍遍及多個資源伺服器，v2.0 端點就會傳回第一個範圍中所指定資源的權杖。 如需範圍的詳盡說明，請參閱 [權限、同意和範圍](v2-permissions-and-consent.md)。 |
| code          | 必要              | 您在流程的第一個階段中取得的 authorization_code。                                                                                                                                                                                                                                                                                                                                                                   |
| redirect_uri  | 必要              | 用來取得 authorization_code 的相同 redirect_uri 值。                                                                                                                                                                                                                                                                                                                                                             |
| client_secret | Web Apps 所需 | 您在應用程式註冊入口網站中為應用程式建立的應用程式密碼。 其不應用於原生應用程式，因為裝置無法穩當地儲存 client_secret。 Web Apps 和 Web API 都需要應用程式密碼，其能夠將 client_secret 安全地儲存在伺服器端。  用戶端密碼必須在傳送之前先進行 URL 編碼。                                                                                                                    |
| code_verifier | 選用              | 用來取得 authorization_code 的相同 code_verifier。 如果在授權碼授與要求中已使用 PKCE，則為必要參數。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636) \(英文\)                                                                                                                                                                                                                                                                                             |

#### <a name="successful-response"></a>成功回應

成功的權杖回應如下：

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```
| 參數     | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| access_token  | 所要求的存取權杖。 應用程式可以使用這個權杖驗證受保護的資源，例如 Web API。                                                                                                                                                                                                                                                                                                                                    |
| token_type    | 表示權杖類型值。 Azure AD 唯一支援的類型是 Bearer。                                                                                                                                                                                                                                                                                                                                                                           |
| expires_in    | 存取權杖的有效期 (以秒為單位)。                                                                                                                                                                                                                                                                                                                                                                                                       |
| scope         | access_token 有效的範圍。                                                                                                                                                                                                                                                                                                                                                                                                         |
| refresh_token | OAuth 2.0 重新整理權杖。 應用程式可以使用這個權杖，在目前的存取權杖過期之後，取得其他的存取權杖。 Refresh_token 的有效期很長，而且可以用來延長保留資源存取權的時間。 如需有關重新整理存取權杖的詳細資訊，請參閱[下一節](#refresh-the-access-token)。 <br> **注意：** 只在要求 `offline_access` 範圍時提供。                                               |
| id_token      | 不帶正負號的 JSON Web Token (JWT)。 應用程式可以將這個權杖的區段解碼，來要求已登入使用者的相關資訊。 應用程式可以快取並顯示值，但不應依賴這些值來取得任何授權或安全性界限。 如需有關 id_token 的詳細資訊，請參閱 [`id_token reference`](id-tokens.md)。 <br> **注意：** 只在要求 `openid` 範圍時提供。 |
#### <a name="error-response"></a>錯誤回應
錯誤回應格式如下：

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| 參數         | 說明                                                                                                       |
|-------------------|-------------------------------------------------------------------------------------------------------------------|
| 錯誤             | 用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error_description | 協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。          |
| error_codes       | 有助於診斷的 STS 特定錯誤碼清單。                                                |
| timestamp         | 發生錯誤的時間。                                                                           |
| trace_id          | 有助於診斷的要求唯一識別碼。                                               |
| correlation_id    | 有助於跨元件診斷的要求唯一識別碼。                             |

#### <a name="error-codes-for-token-endpoint-errors"></a>權杖端點錯誤的錯誤碼

| 錯誤碼              | 說明                                                                                                           | 用戶端動作                                                                                                                                                                                                                               |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| invalid_request         | 通訊協定錯誤，例如遺漏必要的參數。                                                               | 修正並重新提交要求                                                                                                                                                                                                                |
| invalid_grant           | 授權碼或 PKCE 代碼驗證器無效或已過期。                                             | 嘗試向 `/authorize` 端點提出新的要求，並確認 code_verifier 參數正確。                                                                                                                                   |
| unauthorized_client     | 未授權驗證用戶端使用此授權授與類型。                                     | 這通常會在用戶端應用程式未在 Azure AD 中註冊，或未加入至使用者的 Azure AD 租用戶時發生。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |
| invalid_client          | 用戶端驗證失敗。                                                                                        | 用戶端認證無效。 若要修正，應用程式系統管理員必須更新認證。                                                                                                                                       |
| unsupported_grant_type  | 授權伺服器不支援授權授與類型。                                              | 變更要求中的授與類型。 這種類型的錯誤應該只會在開發期間發生，並且會在初始測試期間偵測出來。                                                                                                      |
| invalid_resource        | 目標資源無效，因為它不存在、Azure AD 無法找到它，或是它並未正確設定。 | 這表示尚未在租用戶中設定資源 (如果存在)。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。                                           |
| interaction_required    | 要求需要使用者互動。 例如，必須進行其他驗證步驟。                   | 以相同資源重試要求。                                                                                                                                                                                                  |
| temporarily_unavailable | 伺服器暫時過於忙碌而無法處理要求。                                                            | 重試要求。 用戶端應用程式可能會向使用者解釋，說明其回應因暫時性狀況而延遲。                                                                                                                |

## <a name="use-the-access-token"></a>使用存取權杖

既然您已經成功取得 `access_token`，您就可以透過在 `Authorization` 標頭中包含權杖，在 Web API 的要求中使用權杖：

> [!TIP]
> 在 Postman 中執行這項要求！ (先取代 `Authorization` 標頭) [![在 Postman 中執行](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)
> 
> 

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-access-token"></a>重新整理存取權杖

Access_token 有效期很短，且您必須在其到期後重新整理，才能繼續存取資源。 方法是：向 `/token` 端點送出另一個 `POST` 要求，這次提供 `refresh_token`，而不提供 `code`：

```
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=refresh_token
&client_secret=JqQX2PNo9bpM0uEihUPzyrh      // NOTE: Only required for web apps
```

> [!TIP]
> 嘗試在 Postman 中執行這項要求！ (別忘了取代 `refresh_token`) [![在 Postman 中執行](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)
> 
> 

| 參數     |                       | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|---------------|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| tenant        | 必要              | 要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。 允許的值為 `common`、`organizations`、`consumers` 及租用戶識別碼。 如需更多詳細資訊，請參閱 [通訊協定基本概念](active-directory-v2-protocols.md#endpoints)。                                                                                                                                                                                  |
| client_id     | 必要              | 註冊入口網站 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)) 指派給應用程式的應用程式識別碼。                                                                                                                                                                                                                                                            |
| grant_type    | 必要              | 必須是授權碼流程此階段的 `refresh_token` 。                                                                                                                                                                                                                                                                                                                                                                                                    |
| scope         | 必要              | 範圍的空格分隔清單。 在此階段中要求的範圍必須相當於或為原始 authorization_code 要求階段中所要求的範圍子集。 如果這個要求中指定的範圍遍及多個資源伺服器，v2.0 端點就會傳回第一個範圍中所指定資源的權杖。 如需範圍的詳盡說明，請參閱 [權限、同意和範圍](v2-permissions-and-consent.md)。 |
| refresh_token | 必要              | 您在流程的第二個階段中取得的 refresh_token。                                                                                                                                                                                                                                                                                                                                                                                                      |
| redirect_uri  | 必要              | 用來取得 authorization_code 的相同 redirect_uri 值。                                                                                                                                                                                                                                                                                                                                                                                            |
| client_secret | Web Apps 所需 | 您在應用程式註冊入口網站中為應用程式建立的應用程式密碼。 其不應用於原生應用程式，因為裝置無法穩當地儲存 client_secret。 Web Apps 和 Web API 都需要應用程式密碼，其能夠將 client_secret 安全地儲存在伺服器端。                                                                                                                                                    |

#### <a name="successful-response"></a>成功回應

成功的權杖回應如下：

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```
| 參數     | 說明                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| access_token  | 所要求的存取權杖。 應用程式可以使用這個權杖驗證受保護的資源，例如 Web API。                                                                                                                                                                                                                                                                                                                                     |
| token_type    | 表示權杖類型值。 Azure AD 唯一支援的類型是 Bearer。                                                                                                                                                                                                                                                                                                                                                                           |
| expires_in    | 存取權杖的有效期 (以秒為單位)。                                                                                                                                                                                                                                                                                                                                                                                                        |
| scope         | access_token 有效的範圍。                                                                                                                                                                                                                                                                                                                                                                                                          |
| refresh_token | 新的 OAuth 2.0 重新整理權杖。 您應該用新取得的重新整理權杖取代舊的重新整理權杖，以確定盡可能保持重新整理權杖有效的時間。 <br> **注意：** 只在要求 `offline_access` 範圍時提供。                                                                                                                                                                                                |
| id_token      | 不帶正負號的 JSON Web Token (JWT)。 應用程式可以將這個權杖的區段解碼，來要求已登入使用者的相關資訊。 應用程式可以快取並顯示值，但不應依賴這些值來取得任何授權或安全性界限。 如需有關 id_token 的詳細資訊，請參閱 [`id_token reference`](id-tokens.md)。 <br> **注意：** 只在要求 `openid` 範圍時提供。 |

#### <a name="error-response"></a>錯誤回應

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| 參數         | 說明                                                                                                       |
|-------------------|-------------------------------------------------------------------------------------------------------------------|
| 錯誤             | 用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error_description | 協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。           |
| error_codes |有助於診斷的 STS 特定錯誤碼清單。 |
| timestamp |發生錯誤的時間。 |
| trace_id |有助於診斷的要求唯一識別碼。 |
| correlation_id |有助於跨元件診斷的要求唯一識別碼。 |

如需錯誤碼及建議的用戶端動作的說明，請參閱[權杖端點錯誤的錯誤碼](#error-codes-for-token-endpoint-errors)。
