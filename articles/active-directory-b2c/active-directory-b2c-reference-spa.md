---
title: 在 Azure Active Directory B2C 中使用隱含流程的單一頁面應用程式 | Microsoft Docs
description: 了解如何使用 OAuth 2.0 隱含流程搭配 Azure Active Directory B2C，直接建置單一頁面應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/06/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: b00eb1b2d25187dc50be53425ebae347edde33b4
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344806"
---
# <a name="azure-ad-b2c-single-page-app-sign-in-by-using-oauth-20-implicit-flow"></a>Azure AD B2C：使用 OAuth 2.0 隱含流程的單一頁面應用程式登入

許多新式應用程式都有一個單頁應用程式前端，主要是以 JavaScript 撰寫。 通常會使用 AngularJS、Ember.js、Durandal 等架構來撰寫應用程式。 主要在瀏覽器上執行的單一頁面應用程式和其他 JavaScript 應用程式，在驗證時會面臨一些額外的挑戰：

* 這些應用程式的安全性特性與傳統伺服器型 Web 應用程式大不相同。
* 許多授權伺服器與識別提供者不支援跨原始來源資源共用 (CORS) 要求。
* 重新導向離開應用程式的完整網頁瀏覽器，對使用者體驗有明顯的侵入性。

為了支援這些應用程式，Azure Active Directory B2C (Azure AD B2C) 使用 OAuth 2.0 隱含流程。 如需 OAuth 2.0 授權隱含授權流程的說明，請參閱 [OAuth 2.0 規格的 4.2 節](http://tools.ietf.org/html/rfc6749) 。 在隱含流程中，應用程式會直接從 Azure Active Directory (Azure AD) 授權端點接收權杖，而不需執行任何伺服器對伺服器交換。 所有驗證邏輯和工作階段處理完全都在 JavaScript 用戶端中進行，而不需執行其他的頁面重新導向。

Azure AD B2C 會擴充標準的 OAuth 2.0 隱含流程，功能更強大，而不僅止於簡單的驗證與授權。 Azure AD B2C 導入了[原則參數](active-directory-b2c-reference-policies.md)。 利用原則參數，您可以使用 OAuth 2.0 來為應用程式新增更多使用者體驗，例如註冊、登入和設定檔管理。 在本文中，我們會說明如何使用隱含流程與 Azure AD，在您的單一頁面應用程式中實作這些體驗。 為了協助您快速入門，請查看我們的 [Node.js](https://github.com/Azure-Samples/active-directory-b2c-javascript-singlepageapp-nodejs-webapi) \(英文\) 或 [Microsoft .NET](https://github.com/Azure-Samples/active-directory-b2c-javascript-singlepageapp-dotnet-webapi) \(英文\) 範例。

在本文的範例 HTTP 要求中，我們會使用範例 Azure AD B2C 目錄 **fabrikamb2c.onmicrosoft.com**。 此外，也會使用我們的範例應用程式和原則。 您可以使用這些值來自行試驗要求，也可以將它們換成您自己的值。
了解如何[取得您自己的 Azure AD B2C 目錄、應用程式和原則](#use-your-own-b2c-tenant)。


## <a name="protocol-diagram"></a>通訊協定圖表

隱含登入流程看起來像下圖。 本文稍後將詳細說明每個步驟。

![OpenID Connect 區隔線](../media/active-directory-v2-flows/convergence_scenarios_implicit.png)

## <a name="send-authentication-requests"></a>傳送驗證要求
當 Web 應用程式需要驗證使用者並執行原則時，它會將使用者導向至 `/authorize` 端點。 這是流程中的互動部分，可根據原則讓使用者能夠採取行動。 使用者會從 Azure AD 端點取得識別碼權杖。

在此要求中，用戶端會在 `scope` 參數中指出必須向使用者索取的權限。 在 `p` 參數中，它會指出要執行的原則。 以下提供三個範例 (內含換行符號以提高可讀性)，各使用不同的原則。 為了瞭解每個要求的運作方式，請試著將要求貼到瀏覽器來執行。

### <a name="use-a-sign-in-policy"></a>使用登入原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_sign_in
```

### <a name="use-a-sign-up-policy"></a>使用註冊原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_sign_up
```

### <a name="use-an-edit-profile-policy"></a>使用編輯設定檔原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_edit_profile
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| client_id |必要 |在 [Azure 入口網站](https://portal.azure.com)中指派給應用程式的應用程式識別碼。 |
| response_type |必要 |必須包含 OpenID Connect 登入的 `id_token` 。 它也可能包含回應類型 `token`。 如果您使用 `token`，您的應用程式就能立即從授權端點接收存取權杖，而不需向授權端點進行第二次要求。  如果您使用 `token` 回應類型，`scope` 參數就必須包含範圍，以指出要對哪個資源發出權杖。 |
| redirect_uri |建議 |應用程式的重新導向 URI，您的應用程式可在此傳送及接收驗證回應。 它必須與您在入口網站中註冊的其中一個重新導向 URI 完全相符，不過必須是 URL 編碼格式。 |
| response_mode |建議 |指定用來將產生的權杖送回應用程式的方法。  針對隱含流程，請使用 `fragment`。 |
| scope |必要 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 `openid` 範圍指示使用識別碼權杖形式的權限，以登入使用者及取得使用者相關資料。 (我們將在本文稍後深入討論這部分)。對於 Web 應用程式， `offline_access` 範圍是選擇性。 它指出您的應用程式需要重新整理權杖，才能長久存取資源。 |
| state |建議 |包含於也會隨權杖回應傳回之要求中的值。 它可以是您想要使用之任何內容的字串。 通常會使用隨機產生的唯一值來防止跨網站偽造要求攻擊。 驗證要求出現前，也會先使用此狀態來為使用者在應用程式中的狀態相關資訊編碼，例如他們先前所在的網頁。 |
| nonce |必要 |要求中所含的值 (由應用程式產生)，它會以宣告形式包含於產生的識別碼權杖中。 應用程式接著便可確認此值，以減少權杖重新執行攻擊。 此值通常是隨機的唯一字串，可用以識別要求的來源。 |
| p |必要 |要執行的原則。 這是在您的 Azure AD B2C 租用戶中建立的原則名稱。 原則名稱值的開頭必須是 **b2c\_1\_**。 如需詳細資訊，請參閱 [Azure AD B2C 內建原則](active-directory-b2c-reference-policies.md)。 |
| prompt |選用 |需要的使用者互動類型。 目前唯一支援的值為 `login`。 這會強制使用者在該要求上輸入認證。 單一登入將沒有作用。 |

此時會要求使用者完成原則的工作流程。 這可能會牽涉到讓使用者輸入自己的使用者名稱及密碼、以社交身分識別登入、註冊目錄，或是其他任何數目的步驟。 使用者動作取決於原則的定義方式。

當使用者完成原則之後，Azure AD 會透過您用於 `redirect_uri` 的值，將回應傳回給您的應用程式。 它會使用 `response_mode` 參數中指定的方法。 對於每個使用者動作案例來說，回應完全相同，與已執行的原則無關。

### <a name="successful-response"></a>成功回應
使用 `response_mode=fragment` 和 `response_type=id_token+token` 的成功回應如下所示 (內含換行符號以利閱讀)：

```
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&token_type=Bearer
&expires_in=3599
&scope="90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
```

| 參數 | 說明 |
| --- | --- |
| access_token |應用程式要求的存取權杖。  存取權杖不得進行解碼或檢查。 其可被視為不透明的字串。 |
| token_type |權杖類型值。 Azure AD 唯一支援的類型是 Bearer。 |
| expires_in |存取權杖的有效時間長度 (以秒為單位)。 |
| scope |權杖有效的範圍。 您也可以使用範圍來快取權杖，以供稍後使用。 |
| id_token |應用程式所要求的識別碼權杖。 您可以使用識別碼權杖來確認使用者的身分識別，然後開始與使用者的工作階段。 如需識別碼權杖及其內容的詳細資料，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 |
| state |如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。 |

### <a name="error-response"></a>錯誤回應
錯誤回應也能傳送到重新導向 URI，以便讓應用程式能夠適當地處理它們：

```
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |用於將發生之錯誤分類的錯誤碼字串。 您也可以使用錯誤碼進行錯誤處理。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |
| state |如需完整說明，請參閱前一個表格。 如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。|

## <a name="validate-the-id-token"></a>驗證識別碼權杖
收到識別碼權杖並不足以驗證使用者。 您必須驗證識別碼權杖的簽章，並依照應用程式的要求來確認權杖中的宣告。 Azure AD B2C 使用 [JSON Web Tokens (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 和公開金鑰加密編譯來簽署權杖及驗證其是否有效。

視您偏好使用的語言而定，有許多開放原始碼程式庫可用來驗證 JWT。 建議您探索可用的開放原始碼程式庫，而不是實作您自己的驗證邏輯。 您可以使用本文中的資訊，以協助了解如何適當使用這些程式庫。

Azure AD B2C 具有 OpenID Connect 中繼資料端點。 應用程式可以使用此端點，在執行階段擷取 Azure AD B2C 的相關資訊。 這項資訊包括端點、權杖內容和權杖簽署金鑰。 您的 Azure AD B2C 租用戶中的每個原則都有一份 JSON 中繼資料文件。 例如，fabrikamb2c.onmicrosoft.com 租用戶中 b2c_1_sign_in 原則的中繼資料文件位於：

`https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=b2c_1_sign_in`

此設定文件的屬性之一是 `jwks_uri`。 相同原則的值會是：

`https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/discovery/v2.0/keys?p=b2c_1_sign_in`

若要判斷使用了哪個原則來簽署識別碼權杖 (以及可從何處擷取中繼資料)，您有兩個選項。 首先，原則名稱包含在 `id_token` 的 `acr` 宣告中。 如需如何剖析識別碼權杖中的宣告相關資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 另一個選項是當您發出要求時，在 `state` 參數的值中將原則編碼。 然後將 `state` 參數解碼，以判斷使用了哪個原則。 任一種方法都有效。

當您從 OpenID Connect 中繼資料端點取得中繼資料文件之後，就可以使用 RSA-256 公開金鑰 (位於此端點) 來驗證識別碼權杖的簽章。 此端點可能隨時會列出多個金鑰，每個都由 `kid` 所識別。 `id_token` 的標頭也包含 `kid` 宣告。 它指出使用了這其中哪個金鑰來簽署識別碼權杖。 如需詳細資訊 (包括了解如何[驗證權杖](active-directory-b2c-reference-tokens.md))，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md#token-validation)。
<!--TODO: Improve the information on this-->

驗證識別碼權杖的簽章之後，也需要驗證數個宣告。 例如︰

* 驗證 `nonce` 宣告，以防止權杖重新執行攻擊。 其值應該是您在登入要求中所指定的內容。
* 驗證 `aud` 宣告，以確保已針對您的應用程式簽發識別碼權杖。 這個值就是您應用程式的應用程式識別碼。
* 驗證 `iat` 與 `exp` 宣告，以確保識別碼權杖沒有過期。

另外還有幾個您應該執行的驗證，在 [OpenID Connect 核心規格](http://openid.net/specs/openid-connect-core-1_0.html) \(英文\) 中會有其詳細說明。視您的案例而定，您可能也會想要驗證其他宣告。 一些常見的驗證包括：

* 確保使用者或組織已為應用程式註冊。
* 確保使用者有適當的授權與權限。
* 確保驗證方式有一定的強度，例如使用 Azure Multi-Factor Authentication。

如需識別碼權杖中宣告的詳細資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。

在您已徹底驗證識別碼權杖之後，便可開始與使用者的工作階段。 在應用程式中，使用識別碼權杖中的宣告來取得使用者的相關資訊。 這項資訊可以用於顯示、記錄、授權等等。

## <a name="get-access-tokens"></a>取得存取權杖
如果您的 Web 應用程式只需要執行原則，則可以略過接下來的幾節。 下列幾節中的資訊僅適用於需要對 Web API 進行已驗證的呼叫，同時受 Azure AD B2C 保護的 Web 應用程式。

現在，您已將使用者註冊到單一頁面應用程式，便可以取得存取權杖，以呼叫受 Azure AD 保護的 Web API。 即使您已經使用 `token` 回應類型收到權杖，仍可使用此方法來取得其他資源的權杖，而不需重新導向使用者進行再次登入。

在一般的 Web 應用程式流程中，您會透過對 `/token` 端點進行要求來完成這個操作。  不過，該端點不支援 CORS 要求，因此無法選擇進行 AJAX 呼叫以取得及重新整理權杖。 相反地，您可以在隱藏的 HTML iframe 元素中使用隱含流程，為其他 Web API 取得新權杖。 以下是範例 (內含換行符號以利閱讀)：

```

https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
&response_mode=fragment
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&prompt=none
&domain_hint=organizations
&login_hint=myuser@mycompany.com
&p=b2c_1_sign_in
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| client_id |必要 |在 [Azure 入口網站](https://portal.azure.com)中指派給應用程式的應用程式識別碼。 |
| response_type |必要 |必須包含 OpenID Connect 登入的 `id_token` 。  它也可能包含回應類型 `token`。 如果您在這裡使用 `token`，應用程式就能立即從授權端點接收存取權杖，而不需向授權端點進行第二次要求。 如果您使用 `token` 回應類型，`scope` 參數就必須包含範圍，以指出要對哪個資源發出權杖。 |
| redirect_uri |建議 |應用程式的重新導向 URI，您的應用程式可在此傳送及接收驗證回應。 它必須與您在入口網站中註冊的其中一個重新導向 URI 完全相符，不過必須是 URL 編碼格式。 |
| scope |必要 |範圍的空格分隔清單。  若要取得權杖，請包含您針對所需資源要求的所有範圍。 |
| response_mode |建議 |指定將產生之權杖送回到應用程式要使用的方法。  可以是 `query`、`form_post` 或 `fragment`。 |
| state |建議 |會隨權杖回應傳回之要求中所包含的值。  它可以是您想要使用之任何內容的字串。  通常會使用隨機產生的唯一值來防止跨網站偽造要求攻擊。  此狀態也用來在驗證要求出現之前，於應用程式中將使用者狀態的相關資訊編碼。 例如，使用者所在的網頁或檢視。 |
| nonce |必要 |要求中所含的值 (由應用程式產生)，它會以宣告形式包含於產生的識別碼權杖中。  應用程式接著便可確認此值，以減少權杖重新執行攻擊。 此值通常是隨機的唯一字串，可用以識別要求的來源。 |
| prompt |必要 |若要重新整理並取得隱藏 iframe 中的權杖，請使用 `prompt=none` 以確保 iframe 不會停滯在登入頁面上，並立即返回。 |
| login_hint |必要 |若要重新整理並取得隱藏 iframe 中的權杖，請在此提示中包含使用者的使用者名稱，以區分使用者在特定時間點可能具有的多個工作階段。 您可以使用 `preferred_username` 宣告來擷取先前登入的使用者名稱。 |
| domain_hint |必要 |可以是 `consumers` 或 `organizations`。  若要重新整理並取得隱藏 iframe 中的權杖，您必須在要求中包含 `domain_hint` 值。  從先前登入的識別碼權杖擷取 `tid` 宣告，以判斷要使用哪一個值。  如果 `tid` 宣告值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，請使用 `domain_hint=consumers`。  否則，使用 `domain_hint=organizations`。 |

一旦設定 `prompt=none` 參數，此要求就會立即成功或失敗，並返回您的應用程式。  藉由使用 `response_mode` 參數中指定的方法，以指定的重新導向 URI 將成功的回應傳送至您的應用程式。

### <a name="successful-response"></a>成功回應
使用 `response_mode=fragment` 的成功回應看起來像這樣：

```
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
```

| 參數 | 說明 |
| --- | --- |
| access_token |應用程式要求的權杖。 |
| token_type |權杖類型一律為持有人。 |
| state |如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應確認要求和回應中的 `state` 值完全相同。 |
| expires_in |存取權杖的有效期 (以秒為單位)。 |
| scope |存取權杖有效的範圍。 |

### <a name="error-response"></a>錯誤回應
錯誤回應也能傳送到重新導向 URI，以便讓應用程式能夠適當地處理它們。  針對 `prompt=none`，預期的錯誤看起來像這樣：

```
GET https://aadb2cplayground.azurewebsites.net/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用於將發生的錯誤分類的錯誤碼字串。 您也可以使用字串來回應錯誤。 |
| error_description |可協助您識別驗證錯誤根本原因的特定錯誤訊息。 |

如果您在 iframe 要求中收到此錯誤，使用者必須再次以互動方式登入以擷取新的權杖。 您可以利用對應用程式有意義的方式來處理此錯誤。

## <a name="refresh-tokens"></a>重新整理權杖
識別碼權杖和存取權杖都會在短期內過期。 您的應用程式必須準備好定期重新整理這些權杖。  若要重新整理任一種類型的權杖，可使用 `prompt=none` 參數來控制 Azure AD 步驟，藉以執行我們在上述範例中所使用的相同隱藏 iframe 要求。  若要接收新的 `id_token` 值，請務必使用 `response_type=id_token` 和 `scope=openid`，以及 `nonce` 參數。

## <a name="send-a-sign-out-request"></a>傳送登出要求
當您想要將使用者登出應用程式時，請將使用者重新導向到 Azure AD 進行登出。如果沒有這麼做，使用者可能不必重新輸入認證，就能夠向您的應用程式重新驗證自己的身分。 這是因為使用者將擁有有效的 Azure AD 單一登入工作階段。

您只要將使用者重新導向至 `end_session_endpoint` (列於[驗證識別碼權杖](#validate-the-id-token)中所述的相同 OpenID Connect 中繼資料文件中) 即可。 例如︰

```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/logout?
p=b2c_1_sign_in
&post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| p |必要 |用來將使用者登出應用程式的原則。 |
| post_logout_redirect_uri |建議 |使用者在成功登出之後，應該要前往的 URL。若未包含此項，Azure AD B2C 就會向使用者顯示一般訊息。 |

> [!NOTE]
> 將使用者導向至 `end_session_endpoint`，會利用 Azure AD B2C 清除使用者的部分單一登入狀態。 不過，它不會將使用者登出使用者的社交身分識別提供者工作階段。 如果使用者之後在登入時選取相同的識別提供者，該使用者不必輸入自己的認證，就能讓系統重新驗證自己的身分。 舉例來說，如果使用者想要登出您的 Azure AD B2C 應用程式，不一定代表他們想要完全登出自己的 Facebook 帳戶。 不過，如果使用本機帳戶，使用者的工作階段將會正確地結束。
> 
> 

## <a name="use-your-own-azure-ad-b2c-tenant"></a>使用您自己的 Azure AD B2C 租用戶
若要自行嘗試這些要求，請完成下列三個步驟。 使用您自己的值取代我們在本文中使用的範例值：

1. [建立 Azure AD B2C 租用戶](active-directory-b2c-get-started.md)。 在要求中使用您租用戶的名稱。
2. [建立應用程式](active-directory-b2c-app-registration.md)來取得應用程式識別碼和 `redirect_uri` 值。 在應用程式中加入 Web 應用程式或 Web API。 (選擇性) 您可以建立應用程式祕密。
3. [建立您的原則](active-directory-b2c-reference-policies.md) 來取得原則名稱。

## <a name="samples"></a>範例

* [使用 Node.js 建立單一頁面應用程式](https://github.com/Azure-Samples/active-directory-b2c-javascript-singlepageapp-nodejs-webapi) \(英文\)
* [使用 .NET 建立單一頁面應用程式](https://github.com/Azure-Samples/active-directory-b2c-javascript-singlepageapp-dotnet-webapi) \(英文\)

