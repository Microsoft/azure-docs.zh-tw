---
title: 使用 OAuth2.0 代理者進行 Azure AD 服務對服務驗證草稿規格 | Microsoft Docs
description: 本文說明如何使用 HTTP 訊息，以利用 OAuth2.0 代理者流程實作服務對服務驗證。
services: active-directory
documentationcenter: .net
author: navyasric
manager: mtillman
editor: ''
ms.assetid: 09f6f318-e88b-4024-9ee1-e7f09fb19a82
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/06/2017
ms.author: celested
ms.reviewer: hirsin, nacanuma
ms.custom: aaddev
ms.openlocfilehash: ce29c6a9df49721ca23f84da3f1c97bcc83ab4a7
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39580125"
---
# <a name="service-to-service-calls-using-delegated-user-identity-in-the-on-behalf-of-flow"></a>服務對服務呼叫使用在代理者流程中委派的使用者身分識別
OAuth 2.0 代理者流程 (OBO) 的使用案例是應用程式叫用服務/Web API，而後者又需要呼叫另一個服務/Web API。 其概念是透過要求鏈傳播委派的使用者身分識別和權限。 中介層服務若要向下游服務提出已驗證的要求，需要代表使用者保護來自 Azure Active Directory (Azure AD) 存取權杖。

> [!IMPORTANT]
> 使用 [OAuth 2.0 隱含授與](v1-oauth2-implicit-grant-flow.md)的公用用戶端不能使用 OBO 流程。 這些用戶端必須將其存取權杖傳遞至中介層機密用戶端，才能執行 OBO 流程。 若要進一步了解哪些用戶端可執行 OBO 呼叫，請參閱[用戶端限制](#client-limitations)。

## <a name="on-behalf-of-flow-diagram"></a>代理者流程圖
假設使用者已使用 [OAuth 2.0 授權碼授與流程](v1-protocols-oauth-code.md)通過應用程式的驗證。 此時，應用程式有一個包含使用者宣告的存取權杖 (權杖 A)，且同意存取中介層 Web API (API A)。 現在，API A 需要向下游 Web API (API B) 提出已驗證的要求。

接下來的步驟由代理者流程構成，並搭配下圖協助說明。

![OAuth2.0 代理者流程](./media/v1-oauth2-on-behalf-of-flow/active-directory-protocols-oauth-on-behalf-of-flow.png)


1. 用戶端應用程式使用權杖 A 向 API A 提出要求。
2. API A會向 Azure AD 權杖發行端點進行驗證，並要求存取 API B 的權杖。
3. Azure AD 權杖發行端點以權杖 A 驗證 API A 的認證，並發出 API B 的存取權杖 (權杖 B)。
4. 在對 API B 的要求的授權標頭中設定權杖 B。
5. API B 傳回來自受保護資源的資料。

## <a name="register-the-application-and-service-in-azure-ad"></a>在 Azure AD 中註冊應用程式和服務
在 Azure AD 中註冊用戶端應用程式和中介層服務。
### <a name="register-the-middle-tier-service"></a>註冊中介層服務
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頂端列上按一下您的帳戶，然後在 [目錄] 清單底下，選擇您要註冊應用程式的 Active Directory 租用戶。
3. 按一下左側瀏覽區中的 [更多服務]，然後選擇 [Azure Active Directory]。
4. 按一下 [應用程式註冊]，然後選擇 [新應用程式註冊]。
5. 為應用程式輸入易記名稱，然後選取應用程式類型。 根據應用程式的類型，設定登入 URL 或重新導向 URL (導向基底 URL)。 按一下 [建立] 來建立應用程式。
6. 仍然是在 Azure 入口網站中，選擇您的應用程式，按一下 [設定]。 從 [設定] 功能表中，選擇 [金鑰] 並新增金鑰 - 金鑰的持續時間請選取 1 年或 2 年。 當您儲存此頁面時，會顯示金鑰值，複製此值並儲存在安全的位置 - 稍後在您的實作中您將需要此金鑰進行應用程式設定 - 此金鑰值不會再次顯示，也無法以任何其他方法擷取，因此，請在 Azure 入口網站中顯示金鑰值時盡快記下它。

### <a name="register-the-client-application"></a>註冊用戶端應用程式
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頂端列上按一下您的帳戶，然後在 [目錄] 清單底下，選擇您要註冊應用程式的 Active Directory 租用戶。
3. 按一下左側瀏覽區中的 [更多服務]，然後選擇 [Azure Active Directory]。
4. 按一下 [應用程式註冊]，然後選擇 [新應用程式註冊]。
5. 為應用程式輸入易記名稱，然後選取應用程式類型。 根據應用程式的類型，設定登入 URL 或重新導向 URL (導向基底 URL)。 按一下 [建立] 來建立應用程式。
6. 設定應用程式的權限 - 在 [設定] 功能表中，選擇 [必要權限] 區段，依序按一下 [新增] 和 [選取 API]，然後在文字方塊中輸入中介層服務的名稱。 接著，按一下 [選取權限] 並選取 [存取*服務名稱*]。

### <a name="configure-known-client-applications"></a>設定已知的用戶端應用程式
在此案例中，中介層服務不會利用使用者互動來取得使用者的下游 API 存取同意。 因此，在驗證期間必須先呈現授與存取下游 API 的選項，作為同意步驟的一部分。
若要這麼做，請遵循下列步驟，明確繫結 Azure AD 中的用戶端應用程式註冊與中介層服務註冊，這會將用戶端和中介層皆需要的同意合併在單一對話方塊中。
1. 瀏覽至中介層服務註冊，按一下 [資訊清單] 開啟資訊清單編輯器。
2. 在資訊清單中，找到 `knownClientApplications` 陣列屬性，並新增用戶端應用程式的用戶端識別碼作為元素。
3. 按一下儲存按鈕以儲存資訊清單。

## <a name="service-to-service-access-token-request"></a>服務對服務的存取權杖要求
若要要求存取權杖，請對租用戶特定的 Azure AD 端點提出 HTTP POST 並搭配下列參數。

```
https://login.microsoftonline.com/<tenant>/oauth2/token
```
有兩種情況，取決於用戶端應用程式是選擇透過共用密碼或憑證來保護。

### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一種情況︰使用共用密碼的存取權杖要求
使用共用密碼時，服務對服務存取權杖要求包含下列參數：

| 參數 |  | 說明 |
| --- | --- | --- |
| grant_type |必要 | 權杖要求的類型。 若是使用 JWT 的要求，此值必須是 **urn:ietf:params:oauth:grant-type:jwt-bearer**。 |
| assertion |必要 | 要求中使用的權杖值。 |
| client_id |必要 | 在向 Azure AD 註冊期間指派的用來呼叫服務的應用程式識別碼。 若要在 Azure 管理入口網站中尋找應用程式識別碼，依序按一下 [Active Directory]、目錄，然後按一下應用程式名稱。 |
| client_secret |必要 | 在 Azure AD 中註冊之呼叫端服務的金鑰。 此值應該在註冊期間記下來。 |
| resource |必要 | 接收端服務 (受保護的資源) 的應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 管理入口網站中，依序按一下 [Active Directory]、目錄、應用程式名稱、[所有設定] 及 [屬性]。 |
| requested_token_use |必要 | 指定應該如何處理要求。 在代理者流程中，此值必須是 **on_behalf_of**。 |
| scope |必要 | 權杖要求範圍的清單，各項目之間以空格分隔。 若為 OpenID connect，則必須指定 **openid** 範圍。|

#### <a name="example"></a>範例
下列 HTTP POST 會要求提供 https://graph.windows.net Web API 的存取權杖。 `client_id` 會識別要求存取權杖的服務。

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_secret=0Y1W%2BY3yYb3d9N8vSjvm8WrGzVZaAaHbHHcGbcgG%2BoI%3D
&resource=https%3A%2F%2Fgraph.windows.net
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

### <a name="second-case-access-token-request-with-a-certificate"></a>第二種情況︰使用憑證的存取權杖要求
使用憑證的服務對服務存取權杖要求包含下列參數：

| 參數 |  | 說明 |
| --- | --- | --- |
| grant_type |必要 | 權杖要求的類型。 若是使用 JWT 的要求，此值必須是 **urn:ietf:params:oauth:grant-type:jwt-bearer**。 |
| assertion |必要 | 要求中使用的權杖值。 |
| client_id |必要 | 在向 Azure AD 註冊期間指派的用來呼叫服務的應用程式識別碼。 若要在 Azure 管理入口網站中尋找應用程式識別碼，依序按一下 [Active Directory]、目錄，然後按一下應用程式名稱。 |
| client_assertion_type |必要 |值必須是 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| client_assertion |必要 | 您必須建立判斷提示 (JSON Web 權杖)，並使用註冊的憑證來簽署，以作為應用程式的認證。 請參閱[憑證認證](active-directory-certificate-credentials.md)，以了解如何註冊您的憑證與判斷提示的格式。|
| 資源 |必要 | 接收端服務 (受保護的資源) 的應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 管理入口網站中，依序按一下 [Active Directory]、目錄、應用程式名稱、[所有設定] 及 [屬性]。 |
| requested_token_use |必要 | 指定應該如何處理要求。 在代理者流程中，此值必須是 **on_behalf_of**。 |
| scope |必要 | 權杖要求範圍的清單，各項目之間以空格分隔。 若為 OpenID connect，則必須指定 **openid** 範圍。|

請注意，在透過共用祕密要求的情況中，參數幾乎相同，不同之處在於使用下列兩個參數來取代 client_secret 參數：client_assertion_type 和 client_assertion。

#### <a name="example"></a>範例
下列 HTTP POST 會使用憑證要求提供 https://graph.windows.net Web API 的存取權杖。 `client_id` 會識別要求存取權杖的服務。

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&resource=https%3A%2F%2Fgraph.windows.net
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

## <a name="service-to-service-access-token-response"></a>服務對服務的存取權杖回應
成功的回應是 JSON OAuth 2.0 回應，包含下列參數。

| 參數 | 說明 |
| --- | --- |
| token_type |表示權杖類型值。 Azure AD 唯一支援的類型是 [持有人] 。 如需持有人權杖的詳細資訊，請參閱 [OAuth 2.0 授權架構︰持有人權杖用法 (RFC 6750)](http://www.rfc-editor.org/rfc/rfc6750.txt)。 |
| scope |在權杖中授與的存取範圍。 |
| expires_in |存取權杖的有效時間長度 (以秒為單位)。 |
| expires_on |存取權杖的到期時間。 日期會表示為從 1970-01-01T0:0:0Z UTC 至到期時間的秒數。 這個值用來判斷快取權杖的存留期。 |
| resource |接收端服務 (受保護的資源) 的應用程式識別碼 URI。 |
| access_token |所要求的存取權杖。 呼叫端服務可以使用此權杖來向接收端服務進行驗證。 |
| id_token |所要求的識別碼權杖。 呼叫端服務可以使用此識別碼權杖來確認使用者的身分識別，然後開始與使用者的工作階段。 |
| refresh_token |所要求之存取權杖的重新整理權杖。 呼叫端服務可以使用這個權杖，在目前的存取權杖過期之後，要求其他的存取權杖。 |

### <a name="success-response-example"></a>成功回應範例
下列範例顯示 https://graph.windows.net Web API 存取權杖要求的成功回應。

```
{
    "token_type":"Bearer",
    "scope":"User.Read",
    "expires_in":"43482",
    "ext_expires_in":"302683",
    "expires_on":"1493466951",
    "not_before":"1493423168",
    "resource":"https://graph.windows.net",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ",
    "refresh_token":"AQABAAAAAABnfiG-mA6NTae7CdWW7QfdjKGu9-t1scy_TDEmLi4eLQMjJGt_nAoVu6A4oSu1KsRiz8XyQIPKQxSGfbf2FoSK-hm2K8TYzbJuswYusQpJaHUQnSqEvdaCeFuqXHBv84wjFhuanzF9dQZB_Ng5za9xKlUENrNtlq9XuLNVKzxEyeUM7JyxzdY7JiEphWImwgOYf6II316d0Z6-H3oYsFezf4Xsjz-MOBYEov0P64UaB5nJMvDyApV-NWpgklLASfNoSPGb67Bc02aFRZrm4kLk-xTl6eKE6hSo0XU2z2t70stFJDxvNQobnvNHrAmBaHWPAcC3FGwFnBOojpZB2tzG1gLEbmdROVDp8kHEYAwnRK947Py12fJNKExUdN0njmXrKxNZ_fEM33LHW1Tf4kMX_GvNmbWHtBnIyG0w5emb-b54ef5AwV5_tGUeivTCCysgucEc-S7G8Cz0xNJ_BOiM_4bAv9iFmrm9STkltpz0-Tftg8WKmaJiC0xXj6uTf4ZkX79mJJIuuM7XP4ARIcLpkktyg2Iym9jcZqymRkGH2Rm9sxBwC4eeZXM7M5a7TJ-5CqOdfuE3sBPq40RdEWMFLcrAzFvP0VDR8NKHIrPR1AcUruat9DETmTNJukdlJN3O41nWdZOVoJM-uKN3uz2wQ2Ld1z0Mb9_6YfMox9KTJNzRzcL52r4V_y3kB6ekaOZ9wQ3HxGBQ4zFt-2U0mSszIAA",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8yNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IvIiwiaWF0IjoxNDkzNDIzMTY4LCJuYmYiOjE0OTM0MjMxNjgsImV4cCI6MTQ5MzQ2Njk1MSwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXRpIjoieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsInZlciI6IjEuMCJ9."
}
```

### <a name="error-response-example"></a>錯誤回應範例
嘗試取得下游 API 的存取權杖時，如果下游 API 有條件式存取原則，例如在其上設定多重要素驗證時，Azure AD 權杖端點會傳回錯誤回應。 中介層服務應該向用戶端應用程式呈現此錯誤，以便用戶端應用程式可以提供使用者互動，以滿足條件式存取原則。

```
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
```
GET /me?api-version=2013-11-08 HTTP/1.1
Host: graph.windows.net
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ
```
## <a name="client-limitations"></a>用戶端限制
含有萬用字元回覆 URL 的公用用戶端無法針對 OBO 流程使用 `id_token`。 不過，機密用戶端仍可兌換透過隱含授與流程取得的存取權杖，即使公用用戶端已註冊萬用字元重新導向 URL 亦然。

## <a name="next-steps"></a>後續步驟
進一步了解 OAuth 2.0 通訊協定，以及另一種使用用戶端認證來執行服務對服務驗證的方式。
* [使用 Azure AD 中授與的 OAuth 2.0 用戶端認證來進行服務對服務授權](v1-oauth2-client-creds-grant-flow.md)
* [Azure AD 中的 OAuth 2.0](v1-protocols-oauth-code.md)
