---
title: 了解 Azure AD 中的 OAuth 2.0 授權碼流程
description: 本文說明如何使用 HTTP 訊息來使用 Azure Active Directory 和 OAuth 2.0 授權存取租用戶中的 Web 應用程式和 Web API。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
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
ms.openlocfilehash: bd9d3a677d9fea54331200258d4b9b8e07a54312
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956892"
---
# <a name="authorize-access-to-azure-active-directory-web-applications-using-the-oauth-20-code-grant-flow"></a>使用 OAuth 2.0 授權碼授與流程，授權存取 Azure Active Directory Web 應用程式

Azure Active Directory (Azure AD) 使用 OAuth 2.0 讓您授權存取 Azure AD 租用戶中的 Web 應用程式和 Web API。 本指南不限於特定語言，其中說明如何在不使用任何[開放原始碼程式庫](active-directory-authentication-libraries.md)的情況下，傳送和接收 HTTP 訊息。

如需 OAuth 2.0 授權碼流程的說明，請參閱 [OAuth 2.0 規格的 4.1 節](https://tools.ietf.org/html/rfc6749#section-4.1)。 在大多數的應用程式類型中，其用於執行驗證與授權，包括 Web Apps 和原始安裝的應用程式。

[!INCLUDE [active-directory-protocols-getting-started](../../../includes/active-directory-protocols-getting-started.md)]

## <a name="oauth-20-authorization-flow"></a>OAuth 2.0 授權流程

概括而言，應用程式的整個授權流程看起來有點像這樣：

![OAuth 授權碼流程](./media/v1-protocols-oauth-code/active-directory-oauth-code-flow-native-app.png)

## <a name="request-an-authorization-code"></a>要求授權碼

授權碼流程始於用戶端將使用者導向 `/authorize` 端點。 在這項要求中，用戶端會指出必須向使用者索取的權限。 您可以選取 Azure 入口網站中的 [應用程式註冊] > [端點]，以取得租用戶的 OAuth 2.0 授權端點。

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%3A12345
&response_mode=query
&resource=https%3A%2F%2Fservice.contoso.com%2F
&state=12345
```

| 參數 |  | 說明 |
| --- | --- | --- |
| tenant |必要 |要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。 租用戶獨立權杖允許的值為租用戶識別碼，例如 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.onmicrosoft.com` 或 `common` |
| client_id |必要 |向 Azure AD 註冊應用程式時，指派給您應用程式的應用程式識別碼。 您可以在 Azure 入口網站中找到這個值。 按一下服務側邊欄的 [Azure Active Directory]，再按一下 [應用程式註冊]，然後選擇應用程式。 |
| response_type |必要 |授權碼流程必須包含 `code`。 |
| redirect_uri |建議使用 |應用程式的 redirect_uri，您的應用程式可在此傳送及接收驗證回應。 其必須完全符合您在入口網站中註冊的其中一個 redirect_uris，不然就必須得是編碼的 url。 對於原生和行動應用程式，請使用 `urn:ietf:wg:oauth:2.0:oob` 的預設值。 |
| response_mode |選用 |指定將產生的權杖送回到應用程式所應該使用的方法。 可以是 `query`、`fragment` 或 `form_post`。 `query` 會提供程式碼，以作為重新導向 URI 的查詢字串參數。 如果您要求可使用隱含流程的識別碼權杖，就無法使用 [OpenID 規格](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations)中指定的 `query`。如果您只要求程式碼，您可以使用 `query`、`fragment` 或 `form_post`。 `form_post` 會執行 POST，其中包含您重新導向 URI 的程式碼。 預設值是程式碼流程的 `query`。  |
| state |建議使用 |要求中包含的值，也會隨權杖回應傳回。 隨機產生的唯一值通常用於 [防止跨站台要求偽造攻擊](http://tools.ietf.org/html/rfc6749#section-10.12)。 此狀態也用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| resource | 建議使用 |目標 Web API (受保護的資源) 應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 入口網站中，按一下 [Azure Active Directory]，再按一下 [應用程式註冊]，開啟應用程式的 [設定] 頁面，再按一下 [屬性]。 其也可能是外部的資源，例如 `https://graph.microsoft.com`。 授權或權杖要求會需要此 URI。 為盡量減少授權提示次數，請將之放置於授權要求內，以確保收到使用者的同意。 |
| scope | **已忽略** | 若為 v1 Azure AD 應用程式，請務必前往 Azure 入口網站，至應用程式 [設定] 下方的 [所需權限]，以統計方式設定範圍。 |
| prompt |選用 |表示需要的使用者互動類型。<p> 有效值為： <p> *login*：應提示使用者重新驗證。 <p> select_account：使用者會收到選取帳戶的提示，中斷單一登入。 使用者可以選取現有登入帳戶、為已記住的帳戶輸入認證，或者選擇使用完全不同的帳戶。 <p> *consent*：已授與使用者同意，但需要更新。 應提示使用者同意。 <p> *admin_consent*：應提示管理員代表其組織內的所有使用者同意 |
| login_hint |選用 |如果您事先知道其使用者名稱，可用來預先填入使用者登入頁面的使用者名稱/電子郵件地址欄位。 通常應用程式會在重新驗證期間使用此參數，並已經使用 `preferred_username` 宣告從上一個登入擷取使用者名稱。 |
| domain_hint |選用 |提供有關使用者應該用來登入之租用戶或網域的提示。 domain_hint 的值是租用戶的註冊網域。 如果租用戶與內部部署目錄結成同盟，AAD 會重新導向至指定的租用戶同盟伺服器。 |
| code_challenge_method | 建議使用    | 用來為 `code_challenge` 參數編碼 `code_verifier` 的方法。 可以是 `plain` 或 `S256` 其中一個。 如果排除，則當包含 `code_challenge` 時，會假設 `code_challenge` 是純文字。 Azure AAD v1.0 同時支援 `plain` 和 `S256`。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |
| code_challenge        | 建議使用    | 用於透過來自原生用戶端或公開用戶端之代碼交換的證明金鑰 (PKCE) 保護授權碼授與。 如果包含 `code_challenge_method`，則為必要參數。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |

> [!NOTE]
> 如果使用者隸屬於組織，組織的系統管理員可以代表使用者同意或拒絕，或允許使用者自行同意。 只有當系統管理員允許時，使用者才會獲得同意的選項。
>
>

此時，會要求使用者輸入其認證，並同意 Azure 入口網站中應用程式所要求的權限。 一旦使用者驗證並予以同意，Azure AD 便會在要求的 `redirect_uri` 位址中傳送回應給應用程式，並附上該代碼。

### <a name="successful-response"></a>成功回應
成功的回應看起來可能像這樣︰

```
GET  HTTP/1.1 302 Found
Location: http://localhost:12345/?code= AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA&session_state=7B29111D-C220-4263-99AB-6F6E135D75EF&state=D79E5777-702E-4260-9A62-37F75FF22CCE
```

| 參數 | 說明 |
| --- | --- |
| admin_consent |如果系統管理員已對同意要求提示表示同意，則值為 True。 |
| code |應用程式要求的授權碼。 應用程式可以使用授權碼要求目標資源的存取權杖。 |
| session_state |識別目前使用者工作階段的唯一值。 這個值是 GUID，但應視為不檢查即傳遞的不透明值。 |
| state |如果要求中包含狀態參數，回應中就應該出現相同的值。 應用程式最好在使用回應之前確認要求和回應中的狀態值完全相同。 這有助於偵測對用戶端發動的 [跨網站偽造要求 (CSRF) 攻擊](https://tools.ietf.org/html/rfc6749#section-10.12) 。 |

### <a name="error-response"></a>錯誤回應
錯誤回應也可以傳送到 `redirect_uri` ，讓應用程式能適當地處理。

```
GET http://localhost:12345/?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |[OAuth 2.0 授權架構](http://tools.ietf.org/html/rfc6749)的 5.2 節中所定義的錯誤碼值。 下一份資料表會描述 Azure AD 傳回的錯誤碼。 |
| error_description |更詳細的錯誤描述。 此訊息的目的並非要方便使用者了解。 |
| state |狀態值是隨機產生的非重複使用值，會在要求中傳送並在回應中傳回以防止跨網站偽造要求 (CSRF) 攻擊。 |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>授權端點錯誤的錯誤碼
下表說明各種可能在錯誤回應的 `error` 參數中傳回的錯誤碼。

| 錯誤碼 | 說明 | 用戶端動作 |
| --- | --- | --- |
| invalid_request |通訊協定錯誤，例如遺漏必要的參數。 |修正並重新提交要求。 這是通常會在初始測試期間擷取到的開發錯誤。 |
| unauthorized_client |不允許用戶端應用程式要求授權碼。 |這通常會在用戶端應用程式未在 Azure AD 中註冊，或未加入至使用者的 Azure AD 租用戶時發生。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |
| access_denied |資源擁有者拒絕同意 |用戶端應用程式可以通知使用者，除非使用者同意，否則無法繼續進行。 |
| unsupported_response_type |授權伺服器不支援要求中的回應類型。 |修正並重新提交要求。 這是通常會在初始測試期間擷取到的開發錯誤。 |
| server_error |伺服器發生非預期的錯誤。 |重試要求。 這些錯誤可能是由暫時性狀況所引起。 用戶端應用程式可能會向使用者解釋，說明其回應因暫時性錯誤而延遲。 |
| temporarily_unavailable |伺服器暫時過於忙碌而無法處理要求。 |重試要求。 用戶端應用程式可能會向使用者解釋，說明其回應因暫時性狀況而延遲。 |
| invalid_resource |目標資源無效，因為它不存在、Azure AD 無法找到它，或是它並未正確設定。 |這表示尚未在租用戶中設定資源 (如果存在)。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |

## <a name="use-the-authorization-code-to-request-an-access-token"></a>使用授權碼來要求存取權杖
既然已經取得授權碼並獲得使用者授權，您就可以將 POST 要求傳送至 `/token` 端點，兌換授權碼以取得所需資源的存取權杖：

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code
&client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA
&redirect_uri=https%3A%2F%2Flocalhost%3A12345
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=p@ssw0rd

//NOTE: client_secret only required for web apps
```

| 參數 |  | 說明 |
| --- | --- | --- |
| tenant |必要 |要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。 租用戶獨立權杖允許的值為租用戶識別碼，例如 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.onmicrosoft.com` 或 `common` |
| client_id |必要 |向 Azure AD 註冊應用程式時，指派給您的應用程式的識別碼。 您可以在 Azure 入口網站中找到這個值。 應用程式識別碼會顯示在應用程式註冊的設定中。 |
| grant_type |必要 |必須是授權碼流程的 `authorization_code` 。 |
| code |必要 |您在上一節中取得的 `authorization_code` |
| redirect_uri |必要 |用來取得 `authorization_code` 的相同 `redirect_uri` 值。 |
| client_secret |Web 應用程式必備，公用用戶端不允許 |您在 Azure 入口網站的 [金鑰] 中，為應用程式建立的應用程式秘密。 無法在原生應用程式 (公用用戶端) 中使用，因為 client_secret 無法妥善地儲存在裝置中。 Web 應用程式和 Web API (所有機密用戶端) 都需要應用程式秘密，其能夠將 `client_secret` 安全地儲存在伺服器端。 client_secret 應該在傳送之前先進行 URL 編碼。 |
| resource | 建議使用 |目標 Web API (受保護的資源) 應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 入口網站中，按一下 [Azure Active Directory]，再按一下 [應用程式註冊]，開啟應用程式的 [設定] 頁面，再按一下 [屬性]。 其也可能是外部的資源，例如 `https://graph.microsoft.com`。 授權或權杖要求會需要此 URI。 為盡量減少授權提示次數，請將之放置於授權要求內，以確保收到使用者的同意。 若授權要求和權杖要求均有，則資源的參數一定要相符。 | 
| code_verifier | 選用 | 用來取得 authorization_code 的相同 code_verifier。 如果在授權碼授與要求中已使用 PKCE，則為必要參數。 如需詳細資訊，請參閱 [PKCE RFC](https://tools.ietf.org/html/rfc7636) \(英文\)   |

若要尋找應用程式識別碼 URI，請在 Azure 入口網站中，按一下 [Azure Active Directory]，再按一下 [應用程式註冊]，開啟應用程式的 [設定] 頁面，再按一下 [屬性]。

### <a name="successful-response"></a>成功回應
Azure AD 在成功回應時會傳回[存取權杖](access-tokens.md)。 為了減少來自用戶端應用程式和與其相關延遲的網路呼叫，用戶端應用程式應該快取存取權杖達 OAuth 2.0 回應中所指定的權杖存留期。 若要判斷權杖存留期，請使用 `expires_in` 或 `expires_on` 參數值。

如果 Web API 資源傳回 `invalid_token` 錯誤碼，這可能表示資源判定權杖已過期。 如果用戶端和資源的時鐘時間不同 (稱為「時間偏差」)，在從用戶端快取中清除權杖之前，資源可能會將權杖視為已過期。 如果發生這種情況，請從快取中清除權杖，即使它仍在其計算的存留期內，也是如此。

成功的回應看起來可能像這樣︰

```
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1388444763",
  "resource": "https://service.contoso.com/",
  "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA",
  "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83ZmU4MTQ0Ny1kYTU3LTQzODUtYmVjYi02ZGU1N2YyMTQ3N2UvIiwiaWF0IjoxMzg4NDQwODYzLCJuYmYiOjEzODg0NDA4NjMsImV4cCI6MTM4ODQ0NDc2MywidmVyIjoiMS4wIiwidGlkIjoiN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlIiwib2lkIjoiNjgzODlhZTItNjJmYS00YjE4LTkxZmUtNTNkZDEwOWQ3NGY1IiwidXBuIjoiZnJhbmttQGNvbnRvc28uY29tIiwidW5pcXVlX25hbWUiOiJmcmFua21AY29udG9zby5jb20iLCJzdWIiOiJKV3ZZZENXUGhobHBTMVpzZjd5WVV4U2hVd3RVbTV5elBtd18talgzZkhZIiwiZmFtaWx5X25hbWUiOiJNaWxsZXIiLCJnaXZlbl9uYW1lIjoiRnJhbmsifQ."
}

```

| 參數 | 說明 |
| --- | --- |
| access_token |要求的[存取權杖](access-tokens.md)，其形式為帶正負號的 JSON Web 權杖 (JWT)。 應用程式可以使用這個權杖驗證受保護的資源，例如 Web API。 |
| token_type |表示權杖類型值。 Azure AD 唯一支援的類型是 Bearer。 如需持有人權杖的詳細資訊，請參閱 [OAuth2.0 授權架構︰持有人權杖用法 (RFC 6750)](http://www.rfc-editor.org/rfc/rfc6750.txt) |
| expires_in |存取權杖的有效期 (以秒為單位)。 |
| expires_on |存取權杖的到期時間。 日期會表示為從 1970-01-01T0:0:0Z UTC 至到期時間的秒數。 這個值用來判斷快取權杖的存留期。 |
| resource |Web API (受保護的資源) 應用程式識別碼 URI。 |
| scope |授與用戶端應用程式的模擬權限。 預設權限為 `user_impersonation`。 受保護資源的擁有者可以在 Azure AD 中註冊其他的值。 |
| refresh_token |OAuth 2.0 重新整理權杖。 應用程式可以使用這個權杖，在目前的存取權杖過期之後，取得其他的存取權杖。 重新整理權杖的有效期很長，而且可以用來長期保留資源存取權。 |
| id_token |不帶正負號的 JSON Web 權杖 (JWT)，代表[識別碼權杖](id-tokens.md)。 應用程式可以 base64Url 解碼這個權杖的區段，要求已登入使用者的相關資訊。 應用程式可以快取並顯示值，但不應依賴這些值來取得任何授權或安全性界限。 |

如需 JSON Web 權杖的詳細資訊，請參閱 [JWT IETF 草稿規格](http://go.microsoft.com/fwlink/?LinkId=392344)。   若要深入了解 `id_tokens`，請參閱 [v1.0 OpenID Connect 流程](v1-protocols-openid-connect-code.md)。

### <a name="error-response"></a>錯誤回應
權杖發行端點錯誤是 HTTP 錯誤碼，因為用戶端會直接呼叫權杖發行端點。 除了 HTTP 狀態碼，Azure AD 權杖發行端點也會傳回 JSON 文件與描述錯誤的物件。

範例錯誤回應看起來可能像這樣︰

```
{
  "error": "invalid_grant",
  "error_description": "AADSTS70002: Error validating credentials. AADSTS70008: The provided authorization code or refresh token is expired. Send a new interactive authorization request for this user and resource.\r\nTrace ID: 3939d04c-d7ba-42bf-9cb7-1e5854cdce9e\r\nCorrelation ID: a8125194-2dc8-4078-90ba-7b6592a7f231\r\nTimestamp: 2016-04-11 18:00:12Z",
  "error_codes": [
    70002,
    70008
  ],
  "timestamp": "2016-04-11 18:00:12Z",
  "trace_id": "3939d04c-d7ba-42bf-9cb7-1e5854cdce9e",
  "correlation_id": "a8125194-2dc8-4078-90ba-7b6592a7f231"
}
```
| 參數 | 說明 |
| --- | --- |
| 錯誤 |用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error_description |協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |
| error_codes |有助於診斷的 STS 特定錯誤碼清單。 |
| timestamp |發生錯誤的時間。 |
| trace_id |有助於診斷的要求唯一識別碼。 |
| correlation_id |有助於跨元件診斷的要求唯一識別碼。 |

#### <a name="http-status-codes"></a>HTTP 狀態碼
下表列出權杖發行端點傳回的 HTTP 狀態碼。 在某些情況下，錯誤碼就足以描述回應，但若發生錯誤，您就必須剖析隨附的 JSON 文件並檢查其錯誤碼。

| HTTP 代碼 | 說明 |
| --- | --- |
| 400 |預設的 HTTP 代碼。 用於大部分情況，通常是因為要求的格式不正確。 修正並重新提交要求。 |
| 401 |驗證失敗。 例如，要求遺漏 client_secret 參數。 |
| 403 |授權失敗。 例如，使用者沒有存取資源的權限。 |
| 500 |服務發生內部錯誤。 重試要求。 |

#### <a name="error-codes-for-token-endpoint-errors"></a>權杖端點錯誤的錯誤碼
| 錯誤碼 | 說明 | 用戶端動作 |
| --- | --- | --- |
| invalid_request |通訊協定錯誤，例如遺漏必要的參數。 |修正並重新提交要求 |
| invalid_grant |授權碼無效或已過期。 |嘗試向 `/authorize` 端點提出新的要求 |
| unauthorized_client |未授權驗證用戶端使用此授權授與類型。 |這通常會在用戶端應用程式未在 Azure AD 中註冊，或未加入至使用者的 Azure AD 租用戶時發生。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |
| invalid_client |用戶端驗證失敗。 |用戶端認證無效。 若要修正，應用程式系統管理員必須更新認證。 |
| unsupported_grant_type |授權伺服器不支援授權授與類型。 |變更要求中的授與類型。 這種類型的錯誤應該只會在開發期間發生，並且會在初始測試期間偵測出來。 |
| invalid_resource |目標資源無效，因為它不存在、Azure AD 無法找到它，或是它並未正確設定。 |這表示尚未在租用戶中設定資源 (如果存在)。 應用程式可以對使用者提示關於安裝應用程式，並將它加入至 Azure AD 的指示。 |
| interaction_required |要求需要使用者互動。 例如，必須進行其他驗證步驟。 | 並非為非互動式要求，請以相同資源的互動式授權要求重試。 |
| temporarily_unavailable |伺服器暫時過於忙碌而無法處理要求。 |重試要求。 用戶端應用程式可能會向使用者解釋，說明其回應因暫時性狀況而延遲。 |

## <a name="use-the-access-token-to-access-the-resource"></a>使用存取權杖來存取資源
既然您已經成功取得 `access_token`，您就可以透過在 `Authorization` 標頭中包含權杖，在 Web API 的要求中使用權杖。 [RFC 6750](http://www.rfc-editor.org/rfc/rfc6750.txt) 規格會說明如何在 HTTP 要求中使用持有人權杖來存取受保護的資源。

### <a name="sample-request"></a>範例要求
```
GET /data HTTP/1.1
Host: service.contoso.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ
```

### <a name="error-response"></a>錯誤回應
實作 RFC 6750 的受保護資源會發出 HTTP 狀態碼。 如果要求不包含驗證認證或是遺漏權杖，回應中會包含 `WWW-Authenticate` 標頭。 當要求失敗時，資源伺服器會回應 HTTP 狀態碼和錯誤碼。

以下是用戶端要求未包含持有人權杖時的不成功回應範例︰

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer authorization_uri="https://login.microsoftonline.com/contoso.com/oauth2/authorize",  error="invalid_token",  error_description="The access token is missing.",
```

#### <a name="error-parameters"></a>錯誤參數
| 參數 | 說明 |
| --- | --- |
| authorization_uri |授權伺服器的 URI (實體端點)。 此值也可做為查閱索引鍵，以從探索端點取得伺服器的詳細資訊。 <p><p> 用戶端必須確認授權伺服器受到信任。 當資源受到 Azure AD 保護時，便足以確認 URL 開頭為 https://login.microsoftonline.com 或 Azure AD 支援的其他主機名稱。 租用戶特定資源應該一律會傳回租用戶特定授權 URI。 |
| 錯誤 |[OAuth 2.0 授權架構](http://tools.ietf.org/html/rfc6749)的 5.2 節中所定義的錯誤碼值。 |
| error_description |更詳細的錯誤描述。 此訊息的目的並非要方便使用者了解。 |
| resource_id |傳回資源的唯一識別碼。 用戶端應用程式可在要求資源的權杖時，使用此識別碼做為 `resource` 參數的值。 <p><p> 用戶端應用程式務必要確認此值，否則惡意服務或許可以引發**提升權限**攻擊 <p><p> 防止攻擊的建議策略是確認 `resource_id` 符合要存取的 Web API URL 的基礎。 例如，如果正在存取 https://service.contoso.com/data，`resource_id` 可以是 htttps://service.contoso.com/。 除非有可確認識別碼的可靠替代方法，否則用戶端應用程式必須拒絕開頭不是基礎 URL 的 `resource_id` 。 |

#### <a name="bearer-scheme-error-codes"></a>持有人配置錯誤碼
RFC 6750 規格會針對在回應中使用 WWW 驗證標頭和持有人配置的資源，定義下列錯誤。

| HTTP 狀態碼 | 錯誤碼 | 說明 | 用戶端動作 |
| --- | --- | --- | --- |
| 400 |invalid_request |要求的格式不正確。 例如，它可能遺漏參數或使用相同參數兩次。 |修正錯誤，然後重試要求。 這種類型的錯誤應該只會在開發期間發生，並且會在初始測試中偵測出來。 |
| 401 |invalid_token |存取權杖遺漏、無效或已撤銷。 error_description 參數的值會提供其他詳細資料。 |從授權伺服器要求新的權杖。 如果新的權杖失敗，則表示已發生未預期的錯誤。 傳送錯誤訊息給使用者，並在隨機的延遲之後重試。 |
| 403 |insufficient_scope |存取權杖不包含存取資源所需的模擬權限。 |將新的授權要求傳送至授權端點。 如果回應包含範圍參數，則在資源的要求中使用範圍值。 |
| 403 |insufficient_access |權杖的主體沒有存取資源所需的權限。 |提示使用者使用不同的帳戶，或要求所指定資源的權限。 |

## <a name="refreshing-the-access-tokens"></a>重新整理存取權杖

存取權杖有效期很短，到期後必須重新整理，才能繼續存取資源。 您可以重新整理 `access_token`，方法是向 `/token` 端點送出另一個 `POST` 要求，但這次提供 `refresh_token`，而不提供 `code`。

重新整理權杖並沒有指定的存留期。 一般而言，重新整理權杖的存留期相當長。 不過，在某些情況下，重新整理權杖會過期、遭到撤銷或對要執行的動作缺乏足夠的權限。 應用程式必須預期並正確處理權杖發行端點所傳回的錯誤。

當您收到具有重新整理權杖錯誤的回應時，請捨棄目前的重新整理權杖並要求新的授權碼或存取權杖。 特別是，在授權碼授與流程中使用重新整理權杖時，如果您收到的回應含有 `interaction_required` 或 `invalid_grant` 錯誤代碼，請捨棄重新整理權杖並要求新的授權碼。

使用重新整理權杖來取得新存取權杖之**租用戶專屬**端點 (您也可以使用**一般**端點) 的要求範例看起來像這樣：

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps
```

### <a name="successful-response"></a>成功回應
成功的權杖回應如下：

```
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1460404526",
  "resource": "https://service.contoso.com/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "refresh_token": "AwABAAAAv YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl PM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfmVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA"
}
```
| 參數 | 說明 |
| --- | --- |
| token_type |權杖類型。 唯一支援的值為 **bearer**。 |
| expires_in |權杖的剩餘存留期 (秒)。 一般值為 3600 (1 小時)。 |
| expires_on |權杖的到期日期和時間。 日期會表示為從 1970-01-01T0:0:0Z UTC 至到期時間的秒數。 |
| resource |識別存取權杖可用來存取的受保護的資源。 |
| scope |授與原生用戶端應用程式的模擬權限。 預設權限為 **user_impersonation**。 目標資源的擁有者可以在 Azure AD 中註冊替代值。 |
| access_token |所要求的新存取權杖。 |
| refresh_token |此回應中的存取權杖到期時，可用來要求新存取權杖的新 OAuth 2.0 refresh_token。 |

### <a name="error-response"></a>錯誤回應
範例錯誤回應看起來可能像這樣︰

```
{
  "error": "invalid_resource",
  "error_description": "AADSTS50001: The application named https://foo.microsoft.com/mail.read was not found in the tenant named 295e01fc-0c56-4ac3-ac57-5d0ed568f872. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You might have sent your authentication request to the wrong tenant.\r\nTrace ID: ef1f89f6-a14f-49de-9868-61bd4072f0a9\r\nCorrelation ID: b6908274-2c58-4e91-aea9-1f6b9c99347c\r\nTimestamp: 2016-04-11 18:59:01Z",
  "error_codes": [
    50001
  ],
  "timestamp": "2016-04-11 18:59:01Z",
  "trace_id": "ef1f89f6-a14f-49de-9868-61bd4072f0a9",
  "correlation_id": "b6908274-2c58-4e91-aea9-1f6b9c99347c"
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error_description |協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |
| error_codes |有助於診斷的 STS 特定錯誤碼清單。 |
| timestamp |發生錯誤的時間。 |
| trace_id |有助於診斷的要求唯一識別碼。 |
| correlation_id |有助於跨元件診斷的要求唯一識別碼。 |

如需錯誤碼及建議的用戶端動作的說明，請參閱[權杖端點錯誤的錯誤碼](#error-codes-for-token-endpoint-errors)。
