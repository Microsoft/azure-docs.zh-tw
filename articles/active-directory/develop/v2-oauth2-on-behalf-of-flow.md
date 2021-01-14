---
title: Microsoft 身分識別平台和 OAuth 2.0 代理者流程 | Azure
titleSuffix: Microsoft identity platform
description: 本文說明如何使用 HTTP 訊息，以利用 OAuth2.0 代理者流程實作服務對服務驗證。
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/7/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 8c8167142876dfac0ae0aeff51e85b66c65c607b
ms.sourcegitcommit: f5b8410738bee1381407786fcb9d3d3ab838d813
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98208843"
---
# <a name="microsoft-identity-platform-and-oauth-20-on-behalf-of-flow"></a>Microsoft 身分識別平台和 OAuth 2.0 代理者流程


OAuth2.0 代理者流程 (OBO) 的使用案例，是應用程式叫用服務/Web API，而後者又需要呼叫另一個服務/Web API。 其概念是透過要求鏈傳播委派的使用者身分識別和權限。 若要讓中介層服務向下游服務提出已驗證的要求，其需要代表使用者保護來自 Microsoft 身分識別平台的存取權杖。

本文說明如何在您的應用程式中直接針對通訊協定進行程式設計。  可能的話，建議您改為使用支援的 Microsoft 驗證程式庫 (MSAL)，以[取得權杖並呼叫受保護的 Web API](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows)。  另請參閱[使用 MSAL 的範例應用程式](sample-v2-code.md)。


從 2018 年 5 月起，部分隱含流程衍生 `id_token` 無法用於 OBO 流程。 單頁應用程式 (SPA) 應該將 **存取** 權杖傳遞至中介層機密用戶端，才能改為執行 OBO 流程。 若要進一步了解哪些用戶端可執行 OBO 呼叫，請參閱[限制](#client-limitations)。

## <a name="protocol-diagram"></a>通訊協定圖表

假設使用者已使用 [OAuth 2.0 授權碼授與流程](v2-oauth2-auth-code-flow.md)或其他登入流程通過應用程式的驗證。 此時，應用程式具有一個 *API A* 的存取權杖 (權杖 A)，其中包含使用者的宣告與同意存取中介層 Web API (API A)。 現在，API A 需要向下游 Web API (API B) 提出已驗證的要求。

接下來的步驟由 OBO 流程構成，並搭配下圖協助說明。

![顯示 OAuth2.0 代理者流程](./media/v2-oauth2-on-behalf-of-flow/protocols-oauth-on-behalf-of-flow.png)

1. 用戶端應用程式使用權杖 A (含 API A 的 `aud` 宣告) 向 API A 提出要求。
1. API A 會向 Azure 身分識別平台權杖發行端點進行驗證，並要求存取 API B 的權杖。
1. Azure 身分識別平台權杖發行端點以權杖 A 驗證 API A 的認證，並對 API A 發出 API B 的存取權杖 (權杖 B)。
1. 權杖 B 是由 API A 設定於對 API B 的要求授權標頭中。
1. 來自受保護資源的資料會由 API B 傳給 API A，然後傳回給用戶端。

在此案例中，仲介層服務沒有使用者互動，無法讓使用者同意存取下游 API。 因此，在驗證期間必須先呈現授與存取下游 API 的選項，作為同意步驟的一部分。 若要深入了解如何為您的應用程式進行這個設定，請參閱[取得中介層應用程式的同意](#gaining-consent-for-the-middle-tier-application)。

## <a name="middle-tier-access-token-request"></a>仲介層存取權杖要求

若要要求存取權杖，請使用下列參數，對租用戶特定的 Microsoft 身分識別平台權杖端點提出 HTTP POST。

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token
```

有兩種情況，取決於用戶端應用程式是選擇透過共用密碼或憑證來保護。

### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一種情況︰使用共用密碼的存取權杖要求

使用共用密碼時，服務對服務存取權杖要求包含下列參數：

| 參數 | 類型 | 描述 |
| --- | --- | --- |
| `grant_type` | 必要 | 權杖要求的類型。 對於使用 JWT 的要求，值必須是 `urn:ietf:params:oauth:grant-type:jwt-bearer`。 |
| `client_id` | 必要 | [Azure 入口網站 - 應用程式註冊](https://go.microsoft.com/fwlink/?linkid=2083908)頁面指派給您應用程式的應用程式 (用戶端) 識別碼。 |
| `client_secret` | 必要 | 您在 Azure 入口網站 - 應用程式註冊頁面中為應用程式產生的用戶端秘密。 |
| `assertion` | 必要 | 傳送至中介層 API 的存取權杖。  此權杖必須有物件 (`aud` 應用程式) 宣告，讓此 OBO 要求 () 欄位所表示的應用程式 `client-id` 。 應用程式無法兌換不同應用程式的權杖 (因此，例如，如果用戶端傳送的 API 是用於 MS Graph 的權杖，則 API 無法使用 OBO 兌換該權杖。  它應該改為拒絕權杖) 。  |
| `scope` | 必要 | 權杖要求範圍的清單，各項目之間以空格分隔。 如需詳細資訊，請參閱[範圍](v2-permissions-and-consent.md)。 |
| `requested_token_use` | 必要 | 指定應該如何處理要求。 在 OBO 流程中，此值必須設定為 `on_behalf_of`。 |

#### <a name="example"></a>範例

下列 HTTP POST 會要求 https://graph.microsoft.com Web API 之 `user.read` 範圍的存取權杖與重新整理權杖。

```HTTP
//line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=2846f71b-a7a4-4987-bab3-760035b2f389
&client_secret=BYyVnAt56JpLwUcyo47XODd
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiIyO{a lot of characters here}
&scope=https://graph.microsoft.com/user.read+offline_access
&requested_token_use=on_behalf_of
```

### <a name="second-case-access-token-request-with-a-certificate"></a>第二種情況︰使用憑證的存取權杖要求

使用憑證的服務對服務存取權杖要求包含下列參數：

| 參數 | 類型 | 描述 |
| --- | --- | --- |
| `grant_type` | 必要 | 權杖要求的類型。 對於使用 JWT 的要求，值必須是 `urn:ietf:params:oauth:grant-type:jwt-bearer`。 |
| `client_id` | 必要 |  [Azure 入口網站 - 應用程式註冊](https://go.microsoft.com/fwlink/?linkid=2083908)頁面指派給您應用程式的應用程式 (用戶端) 識別碼。 |
| `client_assertion_type` | 必要 | 值必須是 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`。 |
| `client_assertion` | 必要 | 您必須建立判斷提示 (JSON Web 權杖)，並使用註冊的憑證來簽署，以作為應用程式的認證。 若要深入了解如何註冊您的憑證與判斷提示的格式，請參閱[憑證認證](active-directory-certificate-credentials.md)。 |
| `assertion` | 必要 |  傳送至中介層 API 的存取權杖。  此權杖必須有物件 (`aud` 應用程式) 宣告，讓此 OBO 要求 () 欄位所表示的應用程式 `client-id` 。 應用程式無法兌換不同應用程式的權杖 (因此，例如，如果用戶端傳送的 API 是用於 MS Graph 的權杖，則 API 無法使用 OBO 兌換該權杖。  它應該改為拒絕權杖) 。  |
| `requested_token_use` | 必要 | 指定應該如何處理要求。 在 OBO 流程中，此值必須設定為 `on_behalf_of`。 |
| `scope` | 必要 | 權杖要求範圍的清單，各項目之間以空格分隔。 如需詳細資訊，請參閱[範圍](v2-permissions-and-consent.md)。|

請注意，在透過共用密碼要求的情況中，參數幾乎相同，不同之處在於使用下列兩個參數來取代 `client_secret` 參數：`client_assertion_type` 和 `client_assertion`。

#### <a name="example"></a>範例

下列 HTTP POST 會使用憑證要求 https://graph.microsoft.com Web API 之 `user.read` 範圍的存取權杖。

```HTTP
// line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiO{a lot of characters here}
&requested_token_use=on_behalf_of
&scope=https://graph.microsoft.com/user.read+offline_access
```

## <a name="middle-tier-access-token-response"></a>仲介層存取權杖回應

成功的回應是 JSON OAuth 2.0 回應，包含下列參數。

| 參數 | 描述 |
| --- | --- |
| `token_type` | 表示權杖類型值。 Microsoft 身分識別平台唯一支援的類型是 `Bearer`。 如需有關持有人權杖的詳細資訊，請參閱 [OAuth 2.0 授權架構︰持有人權杖使用方式 (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) \(英文\)。 |
| `scope` | 在權杖中授與的存取範圍。 |
| `expires_in` | 存取權杖的有效時間長度 (以秒為單位)。 |
| `access_token` | 所要求的存取權杖。 呼叫端服務可以使用此權杖來向接收端服務進行驗證。 |
| `refresh_token` | 所要求之存取權杖的重新整理權杖。 呼叫端服務可以使用這個權杖，在目前的存取權杖過期之後，要求其他的存取權杖。 只有要求 `offline_access` 範圍時，才會提供重新整理權杖。 |

### <a name="success-response-example"></a>成功回應範例

下列範例顯示 https://graph.microsoft.com Web API 存取權杖要求的成功回應。

```json
{
  "token_type": "Bearer",
  "scope": "https://graph.microsoft.com/user.read",
  "expires_in": 3269,
  "ext_expires_in": 0,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkQ0NDYy0tY0hGa18wZE50MVEtc2loVzRMd2RwQVZISGpnTVdQZ0tQeVJIaGlDbUN2NkdyMEpmYmRfY1RmMUFxU21TcFJkVXVydVJqX3Nqd0JoN211eHlBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMzA1LCJuYmYiOjE0OTM5MzAzMDUsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQU9KYnFFWlRNTnEyZFcxYXpKN1RZMDlYeDdOT29EMkJEUlRWMXJ3b2ZRc1k9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJWR1ItdmtEZlBFQ2M1dWFDaENRSkFBIiwidmVyIjoiMS4wIn0.cubh1L2VtruiiwF8ut1m9uNBmnUJeYx4x0G30F7CqSpzHj1Sv5DCgNZXyUz3pEiz77G8IfOF0_U5A_02k-xzwdYvtJUYGH3bFISzdqymiEGmdfCIRKl9KMeoo2llGv0ScCniIhr2U1yxTIkIpp092xcdaDt-2_2q_ql1Ha_HtjvTV1f9XR3t7_Id9bR5BqwVX5zPO7JMYDVhUZRx08eqZcC-F3wi0xd_5ND_mavMuxe2wrpF-EZviO3yg0QVRr59tE3AoWl8lSGpVc97vvRCnp4WVRk26jJhYXFPsdk4yWqOKZqzr3IFGyD08WizD_vPSrXcCPbZP3XWaoTUKZSNJg",
  "refresh_token": "OAQABAAAAAABnfiG-mA6NTae7CdWW7QfdAALzDWjw6qSn4GUDfxWzJDZ6lk9qRw4An{a lot of characters here}"
}
```

上述存取權杖是 Microsoft Graph 的1.0 格式權杖。 這是因為權杖格式是根據所存取的 **資源** ，以及用來要求它的端點不相關的。 既然 Microsoft Graph 已設為接受 v1.0 權杖，所以當用戶端要求 Microsoft Graph 權杖時，Microsoft 身分識別平台即會產生 v1.0 存取權杖。 其他應用程式可能會指出他們想要的是 v2.0 格式權杖、v1.0 格式權杖，或甚至是專屬或加密的權杖格式。  V1.0 和 v2.0 端點都可以發出權杖的任何一種格式，如此一來，不論用戶端要求權杖的方式或位置為何，資源一律都可以取得正確的權杖格式。 

只有應用程式應該檢查存取權杖。 用戶端 **不得** 檢查存取權杖。 檢查程式碼中其他應用程式的存取權杖，會導致您的應用程式在應用程式變更權杖的格式或開始加密時，意外中斷。 

### <a name="error-response-example"></a>錯誤回應範例

嘗試取得下游 API 的存取權杖時，如果下游 API 有條件式存取原則，例如在其上設定[多重要素驗證](../authentication/concept-mfa-howitworks.md)時，權杖端點會傳回錯誤回應。 中介層服務應該向用戶端應用程式呈現此錯誤，以便用戶端應用程式可以提供使用者互動，以滿足條件式存取原則。

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>使用存取權杖來存取受保護資源

現在，中介層服務可以使用取得的權杖，在 `Authorization` 標頭中設定權杖，並向下游 Web API 提出已驗證的要求。

### <a name="example"></a>範例

```HTTP
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>搭配 OAuth2.0 OBO 流程取得的 SAML 判斷提示

某些 OAuth 型 Web 服務需要存取會在非互動流程中接受 SAML 判斷提示的其他 Web 服務 API。 Azure Active Directory 可以提供 SAML 判斷提示，以回應使用 SAML 型 SAML Web 服務作為目標資源的代理者流程。

這是針對 OAuth 2.0 代理者流程的非標準擴充，能允許 OAuth2 型應用程式存取會耗用 SAML 權杖的 Web 服務 API 端點。

> [!TIP]
> 當您從前端 Web 應用程式呼叫受 SAML 保護的 Web 服務時，只需呼叫 API 並搭配使用者現有的工作階段起始一般互動驗證流程。 當服務對服務呼叫需要 SAML 權杖以提供使用者內容時，您只需要使用 OBO 流程。

## <a name="gaining-consent-for-the-middle-tier-application"></a>取得中介層應用程式的同意

根據您應用程式的架構或使用方式，您可以考量不同的策略，以確保 OBO 流程成功。 在所有情況下，最終目標是要確保給予適當的同意，讓用戶端應用程式可以呼叫中介層應用程式，而中介層應用程式具有呼叫後端資源的權限。

> [!NOTE]
> 先前 Microsoft 帳戶系統 (個人帳戶) 不支援 [已知用戶端應用程式] 欄位，也不會顯示合併的同意。  已新增這個欄位，且 Microsoft 身分識別平台中的所有應用程式都可以使用已知用戶端應用程式方法來取得 OBO 呼叫的同意。

### <a name="default-and-combined-consent"></a>/.預設和合併的同意

中介層應用程式會將用戶端新增至已知用戶端應用程式清單的資訊清單中，然後用戶端可以針對自己本身與中介層應用程式觸發合併的同意流程。 在 Microsoft 身分識別平台端點上，此作業是使用 [`/.default` 範圍](v2-permissions-and-consent.md#the-default-scope)來完成。 當使用已知用戶端應用程式和 `/.default` 來觸發同意畫面時，同意畫面會顯示 **兩個** 用戶端對中介層 API 的權限，也會要求中介層 API 需要的任何權限。 使用者提供兩個應用程式的同意，然後 OBO 流程就能運作。

### <a name="pre-authorized-applications"></a>已預先授權應用程式

資源可以表示指定的應用程式一律有接收特定範圍的權限。 這主要是用來讓前端用戶端與後端資源之間的連線更順暢。 資源可以宣告多個已預先授權應用程式，任何此類應用程式可以在 OBO 流程中要求這些權限並接收它們，不需要使用者提供同意。

### <a name="admin-consent"></a>系統管理員同意

租用戶系統管理員可以藉由提供中介層應用程式的同意，保證應用程式有呼叫其所需 API 的權限。 若要這樣做，系統管理員可以在其租用戶中尋找中介層應用程式、開啟必要的權限頁面，並選擇提供應用程式的權限。 若要深入了解系統管理員同意，請參閱[同意和權限文件](v2-permissions-and-consent.md)。

### <a name="use-of-a-single-application"></a>使用單一應用程式

在某些案例中，您可能只有中介層與前端用戶端的單一配對。 在此案例中，您可能會發現使用單一應用程式更輕鬆，不管中介層應用程式的需求。 若要在前端與 Web API 之間進行驗證，您可以使用 Cookie、id_token 或針對應用程式本身要求的存取權杖。 然後，要求從這個單一應用程式到後端資源的同意。

## <a name="client-limitations"></a>用戶端限制

若用戶端使用隱含流程來取得 id_token，而且該用戶端的回覆 URL 中也有萬用字元，id_token 就無法用於 OBO 流程。  不過，機密用戶端仍可兌換透過隱含授與流程取得的存取權杖，即使起始用戶端已註冊萬用字元回覆 URL 亦然。

## <a name="next-steps"></a>後續步驟

進一步了解 OAuth 2.0 通訊協定，以及另一種使用用戶端認證來執行服務對服務驗證的方式。

* [Microsoft 身分識別平台中的 OAuth 2.0 用戶端認證授與](v2-oauth2-client-creds-grant-flow.md)
* [Microsoft 身分識別平台中的 OAuth 2.0 程式碼流程](v2-oauth2-auth-code-flow.md)
* [使用 `/.default` 範圍](v2-permissions-and-consent.md#the-default-scope)
