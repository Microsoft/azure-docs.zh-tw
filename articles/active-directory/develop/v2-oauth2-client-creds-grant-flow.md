---
title: 使用 Azure AD v2.0 存取安全資源且無需使用者互動 | Microsoft Docs
description: 使用 Azure AD 實作 OAuth 2.0 驗證通訊協定以建置 Web 應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 9b7cfbd7-f89f-4e33-aff2-414edd584b07
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/07/2017
ms.author: celested
ms.reviewer: hirsin, dastrock
ms.custom: aaddev
ms.openlocfilehash: 02d5bf9ef293731ce707596a90cd7e9aa0b96450
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39580230"
---
# <a name="azure-active-directory-v20-and-the-oauth-20-client-credentials-flow"></a>Azure Active Directory v2.0 和 OAuth 2.0 用戶端認證流程
您可以使用 RFC 6749 中指定的 [OAuth 2.0 用戶端認證授與](http://tools.ietf.org/html/rfc6749#section-4.4) (有時稱為「雙方 OAuth」，透過使用應用程式識別碼來存取 Web 主控資源。 這類型的授與通常用於必須在背景中執行 (不需與使用者直接互動) 的伺服器對伺服器互動。 這些類型的應用程式通常稱為「精靈」或「服務帳戶」。

> [!NOTE]
> v2.0 端點並未支援所有的 Azure Active Directory 案例和功能。 若要判斷您是否應該使用 v2.0 端點，請參閱 [v2.0 限制](active-directory-v2-limitations.md)。
>
>

在較為典型的「三腳 OAuth」中，用戶端應用程式會獲得代表特定使用者存取資源的權限。 此權限是由使用者委派給應用程式 (通常是在[同意](v2-permissions-and-consent.md)過程中)。 不過，在用戶端認證流程中，權限會直接授與應用程式本身。 當應用程式向資源出示權杖時，資源會強制要求應用程式本身具備執行動作的授權，而不是使用者具備授權。

## <a name="protocol-diagram"></a>通訊協定圖表
整個用戶端認證流程看起來類似下圖。 我們會在本文稍後說明每個步驟。

![用戶端認證流程](./media/v2-oauth2-client-creds-grant-flow/convergence_scenarios_client_creds.png)

## <a name="get-direct-authorization"></a>取得直接授權
應用程式通常會使用下列兩種方法其中之一來接收可存取資源的直接授權︰透過資源上的存取控制清單 (ACL)，或透過 Azure Active Directory (Azure AD) 中的應用程式權限指派。 這兩種方法是 Azure AD 中最常見的方法，並建議用於執行用戶端認證流程的用戶端和資源。 不過，資源也可以選擇以其他方法授權其用戶端。 每個資源伺服器都可以選擇最適合其應用程式的方法。

### <a name="access-control-lists"></a>存取控制清單
資源提供者可能會根據它所知的「應用程式識別碼」清單，強制執行授權檢查並授與特定層級的存取權。 當資源從 v2.0 端點收到權杖時，它可以將此權杖解碼，並從 `appid` 和 `iss` 宣告擷取用戶端的「應用程式識別碼」。 然後，它會將應用程式與它所保有的 ACL 進行比較。 各資源之間的 ACL 細微性及方法可能有極大的差異。

常見的使用案例是使用 ACL 來針對 Web 應用程式或 Web API 執行測試。 Web API 可能只會將完整權限中的一部分授與特定用戶端。 為了在 API 上執行端對端測試，請建立測試用戶端，以從 v2.0 端點取得權杖，然後將權杖傳送至 API。 接著，API 會檢查 ACL 是否有測試用戶端的「應用程式識別碼」，以便提供 API 完整功能的完整存取權。 如果您使用這類 ACL，請務必不要只驗證呼叫者的 `appid` 值。 請一併驗證權杖的 `iss` 值是否受信任。

對於需要存取具有個人 Microsoft 帳戶之取用者使用者所擁有資料的精靈和服務帳戶來說，這種授權相當常見。 對於組織所擁有的資料，建議您透過應用程式權限取得必要的授權。

### <a name="application-permissions"></a>應用程式權限
您可以不使用 ACL，而是使用 API 來公開一組應用程式權限。 應用程式權限是由組織的系統管理員授與應用程式，並且只能用來存取該組織及其員工所擁有的資料。 例如，Microsoft Graph 會公開數個可執行下列操作的應用程式權限︰

* 讀取所有信箱中的郵件
* 讀取和寫入所有信箱中的郵件
* 以任何使用者身分傳送郵件
* 讀取目錄資料

如需有關應用程式權限的詳細資訊，請參閱 [Microsoft Graph](https://graph.microsoft.io)。

若要在您的應用程式中使用應用程式權限，請執行我們在後續小節中討論的步驟。

#### <a name="request-the-permissions-in-the-app-registration-portal"></a>在應用程式註冊入口網站中要求權限
1. 在[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)中移至您的應用程式，或[建立應用程式](quickstart-v2-register-an-app.md) (如果您尚未這麼做)。 建立應用程式時，您將必須至少使用一個「應用程式密碼」。
2. 找出 [Microsoft Graph 權限] 區段，然後新增您應用程式所需的**應用程式權限**。
3. [儲存] 應用程式註冊。

#### <a name="recommended-sign-the-user-in-to-your-app"></a>建議︰將使用者登入您的應用程式
通常，當您建置使用應用程式權限的應用程式時，應用程式會需要一個可供系統管理員核准應用程式權限的頁面或檢視。 此頁面可以是應用程式登入流程的一部分、應用程式設定的一部分，或是專用的「連接」流程。 在許多情況下，應用程式只在使用者利用工作或學校 Microsoft 帳戶登入之後顯示此「連接」檢視是很合理的。

將使用者登入應用程式時，您可以先識別使用者所屬的組織，然後再要求使用者核准應用程式權限。 雖然這並非絕對必要，但這麼做可協助您為使用者建立更直覺式的體驗。 若要讓使用者登入，請遵循我們的 [v2.0 通訊協定教學課程](active-directory-v2-protocols.md)。

#### <a name="request-the-permissions-from-a-directory-admin"></a>向目錄管理員要求權限
當您已準備好向組織的系統管理員要求權限時，您可以將使用者重新導向到 v2.0「系統管理員同意端點」。

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the following request in a browser!
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| 參數 | 條件 | 說明 |
| --- | --- | --- |
| tenant |必要 |您想要要求權限的目錄租用戶。 這可以採用 GUID 或易記名稱格式。 如果您不知道使用者屬於哪個租用戶，而想要讓他們以任何租用戶登入，請使用 `common`。 |
| client_id |必要 |[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)指派給您應用程式的「應用程式識別碼」。 |
| redirect_uri |必要 |您想要傳送回應以供應用程式處理的重新導向 URI。 它必須與您在入口網站中註冊的其中一個重新導向 URI 完全相符，只是它必須是採用 URL 編碼，並且可以有額外的路徑區段。 |
| state |建議 |一個包含在要求中而將一併在權杖回應中傳回的值。 它可以是您所想要內容中的字串。 此狀態用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |

此時，Azure AD 會強制只有租用戶系統管理員可以登入來完成要求。 系統會請系統管理員核准您在應用程式註冊入口網站中，為您應用程式要求的所有直接應用程式權限。

##### <a name="successful-response"></a>成功回應
如果系統管理員為您的應用程式核准權限，則成功的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| 參數 | 說明 |
| --- | --- | --- |
| tenant |將應用程式所要求的權限授與應用程式的目錄租用戶 (採用 GUID 格式)。 |
| state |一個包含在要求中而將一併在權杖回應中傳回的值。 它可以是您所想要內容中的字串。 此狀態用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| admin_consent |設定為 **True**。 |

##### <a name="error-response"></a>錯誤回應
如果系統管理員沒有為您的應用程式核准權限，則失敗的回應看起來會像這樣︰

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| 參數 | 說明 |
| --- | --- | --- |
| 錯誤 |您可用來分類錯誤類型並對錯誤做出反應的錯誤碼字串。 |
| error_description |可協助您識別錯誤根本原因的特定錯誤訊息。 |

從應用程式佈建端點收到成功回應後，您的應用程式便已取得它所要求的直接應用程式權限。 現在，您可以針對您想要的資源要求權杖。

## <a name="get-a-token"></a>取得權杖
取得應用程式的必要授權後，請繼續取得 API 的存取權杖。 若要使用用戶端認證授與來取得權杖，請將 POST 要求傳送至 `/token` v2.0 端點︰

### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一種情況︰使用共用密碼的存取權杖要求

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
```

| 參數 | 條件 | 說明 |
| --- | --- | --- |
| tenant |必要 | 應用程式預期要對其執行作業的目錄租用戶 (以 GUID 或網域名稱格式)。 |
| client_id |必要 |[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)指派給您應用程式的「應用程式識別碼」。 |
| scope |必要 |在這個要求中針對 `scope` 參數傳遞的值應該是您所要資源的資源識別碼 (應用程式識別碼 URI)，並附加 `.default` 尾碼。 就 Microsoft Graph 範例而言，值為 `https://graph.microsoft.com/.default`。 這個值會通知 v2.0 端點有關您已為應用程式設定的所有直接應用程式權限，它應該會針對與您所要使用資源關聯的權限發出權杖。 |
| client_secret |必要 |您在應用程式註冊入口網站中為應用程式產生的「應用程式密碼」。 用戶端密碼必須在傳送之前先進行 URL 編碼。|
| grant_type |必要 |必須是 `client_credentials`。 |

### <a name="second-case-access-token-request-with-a-certificate"></a>第二種情況︰使用憑證的存取權杖要求

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

| 參數 | 條件 | 說明 |
| --- | --- | --- |
| tenant |必要 | 應用程式預期要對其執行作業的目錄租用戶 (以 GUID 或網域名稱格式)。 |
| client_id |必要 |[應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)指派給您應用程式的「應用程式識別碼」。 |
| scope |必要 |在這個要求中針對 `scope` 參數傳遞的值應該是您所要資源的資源識別碼 (應用程式識別碼 URI)，並附加 `.default` 尾碼。 就 Microsoft Graph 範例而言，值為 `https://graph.microsoft.com/.default`。 這個值會通知 v2.0 端點有關您已為應用程式設定的所有直接應用程式權限，它應該會針對與您所要使用資源關聯的權限發出權杖。 |
| client_assertion_type |必要 |值必須是 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| client_assertion |必要 | 您必須建立判斷提示 (JSON Web 權杖)，並使用註冊的憑證來簽署，以作為應用程式的認證。 請參閱[憑證認證](active-directory-certificate-credentials.md)，以了解如何註冊您的憑證與判斷提示的格式。|
| grant_type |必要 |必須是 `client_credentials`。 |

請注意，在透過共用密碼要求的情況中，參數幾乎相同，不同之處在於使用下列兩個參數來取代 client_secret 參數：client_assertion_type 和 client_assertion。

### <a name="successful-response"></a>成功回應
成功的回應看起來會像這樣：

```
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
}
```

| 參數 | 說明 |
| --- | --- |
| access_token |所要求的存取權杖。 應用程式可以使用這個權杖向受保護的資源 (例如 Web API) 進行驗證。 |
| token_type |表示權杖類型值。 Azure AD 唯一支援的類型是 `bearer`。 |
| expires_in |存取權杖的有效期 (以秒為單位)。 |

### <a name="error-response"></a>錯誤回應
錯誤回應看起來像這樣：

```
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |您可用來分類發生的錯誤類型並對錯誤做出反應的錯誤碼字串。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |
| error_codes |可協助進行診斷的 STS 特定錯誤碼清單。 |
| timestamp |發生錯誤的時間。 |
| trace_id |可協助進行診斷的要求唯一識別碼。 |
| correlation_id |可協助跨元件進行診斷的要求唯一識別碼。 |

## <a name="use-a-token"></a>使用權杖
既然您已取得權杖，請使用該權杖來對資源提出要求。 當權杖到期時，只要重複執行對 `/token` 端點的要求，即可取得全新的存取權杖。

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

```
// Pro tip: Try the following command! (Replace the token with your own.)
```

```
curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q" 'https://graph.microsoft.com/v1.0/me/messages'
```

## <a name="code-sample"></a>程式碼範例
若要查看使用系統管理員同意端點來實作用戶端認證授與的應用程式範例，請參閱我們的 [v2.0 精靈程式碼範例](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)。
