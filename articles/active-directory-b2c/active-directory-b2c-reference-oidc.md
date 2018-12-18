---
title: 在 Azure Active Directory B2C 中利用 OpenID Connect 的 Web 登入 | Microsoft Docs
description: 使用 Azure Active Directory 的 OpenID Connect 驗證通訊協定實作來建置 Web 應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: f56c9f916e0bbbf380347af2ec3f17645063494d
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340345"
---
# <a name="azure-active-directory-b2c-web-sign-in-with-openid-connect"></a>Azure Active Directory B2C：利用 OpenID Connect 的 Web 登入
OpenID Connect 是建置在 OAuth 2.0 之上的驗證通訊協定，可用來將使用者安全地登入 Web 應用程式。 藉由使用 OpenID Connect 的 Azure Active Directory B2C (Azure AD B2C) 實作，您就能將 Web 應用程式中的註冊、登入及其他識別管理體驗外包給 Azure Active Directory (Azure AD)。 本指南會以與語言無關的方式示範如何執行此動作。 而是在說明如何傳送和接收 HTTP 訊息，但不使用我們的任何開放原始碼程式庫。

[OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) 擴充 OAuth 2.0 的*授權*通訊協定來做為*驗證*通訊協定。 讓您能利用 OAuth 來執行單一登入。 它引進「識別碼權杖」的概念，這是一種安全性權杖，可讓用戶端驗證使用者的身分識別，並取得有關使用者的基本設定檔資訊。

由於它會擴充 OAuth 2.0，因此也能讓應用程式安全地取得「存取權杖」。 您可以使用 access_tokens，來存取受到[授權伺服器](active-directory-b2c-reference-protocols.md#the-basics)保護的資源。 如果您要建置的 Web 應用程式是裝載在伺服器上，且必須透過瀏覽器來存取，我們建議您使用 OpenID Connect。 如果您想要利用 Azure AD B2C 在行動或桌面應用程式中新增身分識別管理功能，您應該要使用 [OAuth 2.0](active-directory-b2c-reference-oauth-code.md) ，而不是 OpenID Connect。

Azure AD B2C 擴充標準的 OpenID Connect 通訊協定，功能更強大，而不僅止於簡單的驗證和授權。 它引進[原則參數](active-directory-b2c-reference-policies.md)，讓您能利用 OpenID Connect 來為應用程式新增使用者體驗，例如註冊、登入和設定檔管理。 以下我們將示範如何利用 OpenID Connect 和原則，在您的 Web 應用程式中實作上述每一種體驗。 我們也會示範如何取得用來存取 Web API 的存取權杖。

下一節的範例 HTTP 要求使用我們的範例 B2C 目錄 fabrikamb2c.onmicrosoft.com，以及我們的範例應用程式、 https://aadb2cplayground.azurewebsites.net 和原則。 您可以隨意使用這些值來自行試驗要求，也可以把它們換成您自己的值。
了解如何 [取得您自己的 B2C 租用戶、應用程式和原則](#use-your-own-b2c-directory)。

## <a name="send-authentication-requests"></a>傳送驗證要求
當 Web 應用程式需要驗證使用者和執行原則時，它可以將使用者導向至 `/authorize` 端點。 這是流程中的互動部分，可根據原則讓使用者能夠採取行動。

在這項要求中，用戶端會在 `scope` 參數中指出它需要向使用者要求的權限，以及在 `p` 參數中指出它要執行的原則。 後續小節中提供三個範例 (含有換行符號以提高可讀性)，各使用不同的原則。 為了瞭解每個要求的運作方式，請試著將要求貼到瀏覽器來執行。

#### <a name="use-a-sign-in-policy"></a>使用登入原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code+id_token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=form_post
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_sign_in
```

#### <a name="use-a-sign-up-policy"></a>使用註冊原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code+id_token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=form_post
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_sign_up
```

#### <a name="use-an-edit-profile-policy"></a>使用編輯設定檔原則
```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code+id_token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=form_post
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&p=b2c_1_edit_profile
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| client_id |必要 |[Azure 入口網站](https://portal.azure.com/) 指派給您應用程式的應用程式識別碼。 |
| response_type |必要 |回應類型，其中必須包含適用於 OpenID Connect 的識別碼權杖。 如果您的 Web 應用程式也需要權杖來呼叫 Web API，則可以使用 `code+id_token`，如同這裡的作法。 |
| redirect_uri |建議 |應用程式的 `redirect_uri` 參數，您的應用程式可在此傳送及接收驗證回應。 它必須與您在入口網站中註冊的其中一個 `redirect_uri` 參數完全相符，不過必須是 URL 編碼格式。 |
| scope |必要 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 `openid` 範圍指示使用識別碼權杖形式的權限，以登入使用者及取得使用者相關資料 (本文稍後將進一步說明)。 對於 Web 應用程式， `offline_access` 範圍是選擇性。 它表示您的應用程式將需要「重新整理權杖」，才能長久存取資源。 |
| response_mode |建議 |應該用來將產生的授權碼傳回應用程式的方法。 它可以是 `query`、`form_post` 或 `fragment`。  如需最佳安全性，建議使用 `form_post` 回應模式。 |
| state |建議 |要求中包含的值，也會隨權杖回應傳回。 它可以是您所想要內容中的字串。 隨機產生的唯一值通常用於防止跨站台要求偽造攻擊。 驗證要求出現前，也會先使用此狀態為使用者在應用程式中的狀態資訊編碼，例如他們先前所在的網頁。 |
| nonce |必要 |要求中包含的值 (由應用程式產生)，將包含於所產生的識別碼權杖中以作為宣告。 應用程式接著便可確認此值，以減少權杖重新執行攻擊。 此值通常是隨機的唯一字串，可用以識別要求的來源。 |
| p |必要 |系統將會執行的原則。 這是在您的 B2C 租用戶中所建立的原則名稱。 原則名稱值的開頭必須是 `b2c\_1\_`。 深入了解原則與[可延伸原則架構](active-directory-b2c-reference-policies.md)。 |
| prompt |選用 |需要的使用者互動類型。 此時唯一有效的值是 `login`，可強制使用者針對該要求輸入其認證。 單一登入將沒有作用。 |

此時會要求使用者完成原則的工作流程。 視原則的定義方式而定，這可能會牽涉到讓使用者輸入自己的使用者名稱及密碼、以社交身分識別登入、註冊目錄，或是其他任何數目的步驟。

當使用者完成原則之後，Azure AD 就會使用 `response_mode` 參數中指定的方法，以指示的 `redirect_uri` 參數將回應傳回您的應用程式。 上述各種情況的回應均相同，與已執行的原則無關。

使用 `response_mode=fragment` 的成功回應如下所示：

```
GET https://aadb2cplayground.azurewebsites.net/#
id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=arbitrary_data_you_can_receive_in_the_response
```

| 參數 | 說明 |
| --- | --- |
| id_token |應用程式所要求的識別碼權杖。 您可以使用識別碼權杖來確認使用者的身分識別，然後開始與使用者的工作階段。 如需識別碼權杖及其內容的詳細資料，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 |
| code |應用程式要求的授權碼 (如果您使用 `response_type=code+id_token`)。 應用程式可以使用授權碼來要求目標資源的存取權杖。 授權碼的存留期很短。 通常會在大約 10 分鐘後過期。 |
| state |如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。 |

錯誤回應也能夠傳送到 `redirect_uri` 參數，讓應用程式能適當地處理：

```
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用以分類所發生的錯誤類型，以及可用來回應錯誤的錯誤碼字串。 |
| error_description |協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |
| state |如需完整說明，請參閱本節的第一個表格。 如果要求中包含 `state` 參數，回應中就應該出現相同的值。 應用程式應該驗證要求和回應中的 `state` 值完全相同。 |

## <a name="validate-the-id-token"></a>驗證識別碼權杖
只收到識別碼權杖並不足以驗證使用者。 您必須驗證識別碼權杖的簽章，並依照應用程式的要求來確認權杖中的宣告。 Azure AD B2C 使用 [JSON Web Tokens (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 和公開金鑰加密編譯來簽署權杖及驗證其是否有效。

視您偏好的語言而定，有許多針對驗證 JWT 的開放原始碼程式庫可用。 我們建議您探索這些程式庫，而不是實作您自己的驗證邏輯。 當您在思考如何適當地運用那些程式庫時，這裡的資訊會很有用。

Azure AD B2C 具有 OpenID Connect 中繼資料端點，可讓應用程式在執行階段提取 Azure AD B2C 的相關資訊。 這項資訊包括端點、權杖內容和權杖簽署金鑰。 您的 B2C 租用戶中的每個原則都有一份 JSON 中繼資料文件。 例如，`fabrikamb2c.onmicrosoft.com` 中適用於 `b2c_1_sign_in` 原則的中繼資料文件位於：

`https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=b2c_1_sign_in`

此設定文件的屬性之一是 `jwks_uri`，就上述原則而言，它的值是：

`https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/discovery/v2.0/keys?p=b2c_1_sign_in`。

若要判斷使用了哪個原則來簽署識別碼權杖 (以及可從何處擷取中繼資料)，您有兩個選項。 首先，原則名稱包含於識別碼權杖的 `acr` 宣告中。 如需如何從識別碼權杖剖析宣告的相關資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 另一個選項是當您發出要求時在 `state` 參數的值中將原則編碼，然後將它解碼以判斷使用了哪個原則。 任一種方法都有效。

當您從 OpenID Connect 中繼資料端點取得中繼資料文件之後，就可以使用 RSA 256 公開金鑰 (位於此端點) 來驗證識別碼權杖的簽章。 此端點隨時都可能列出多個金鑰，每個金鑰都由 `kid` 宣告所識別。 識別碼權杖的標頭也會包含 `kid` 宣告，指出簽署該識別碼權杖所使用的金鑰。 如需詳細資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md) (特別是關於[驗證權杖](active-directory-b2c-reference-tokens.md#token-validation)的小節)。
<!--TODO: Improve the information on this-->

當您驗證識別碼權杖的簽章之後，必須驗證數個宣告。 例如：

* 您應該驗證 `nonce` 宣告以防止權杖重新執行攻擊。 其值應該是您在登入要求中所指定的內容。
* 您應該驗證 `aud` 宣告，以確認識別碼權杖是針對您的應用程式所核發。 這個值就是您應用程式的應用程式識別碼。
* 您應該驗證 `iat` 及 `exp` 宣告，以確保識別碼權杖尚未過期。

另外還有數個您應該執行的驗證。 詳細說明請參閱 [OpenID Connect 核心規格](http://openid.net/specs/openid-connect-core-1_0.html) \(英文\)。視您的案例而定，您可能也會想要驗證其他宣告。 一些常見的驗證包括：

* 確保使用者/組織已為應用程式註冊。
* 確保使用者有適當的授權/權限。
* 確保驗證方式有一定的強度，例如 Azure Multi-Factor Authentication。

如需識別碼權杖中宣告的詳細資訊，請參閱 [Azure AD B2C 權杖參考](active-directory-b2c-reference-tokens.md)。

當您驗證過識別碼權杖之後，便可開始關於該使用者的工作階段。 您可以使用識別碼權杖中的宣告，以便在應用程式中取得使用者的相關資訊。 這項資訊的用途包括顯示、記錄和授權。

## <a name="get-a-token"></a>取得權杖
如果您的 Web 應用程式只需執行原則，則可略過接下來的幾節。 這幾節僅適用於需要對 Web API 進行已驗證的呼叫，同時受 Azure AD B2C 保護的 Web 應用程式。

您可以藉由將 `POST` 要求傳送至 `/token` 端點，將取得 (使用 `response_type=code+id_token`) 的權杖授權碼兌換成所需的資源。 您目前唯一可要求權杖的資源，就是應用程式本身的後端 Web API。 用來為您自己要求權杖的慣例是使用應用程式的用戶端識別碼做為範圍：

```
POST fabrikamb2c.onmicrosoft.com/oauth2/v2.0/token?p=b2c_1_sign_in HTTP/1.1
Host: https://fabrikamb2c.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_secret=<your-application-secret>

```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| p |必要 |用來取得授權碼的原則。 您無法在此要求中使用不同的原則。 請注意，您要將這個參數新增到查詢字串，而不是 `POST` 主體。 |
| client_id |必要 |[Azure 入口網站](https://portal.azure.com/) 指派給您應用程式的應用程式識別碼。 |
| grant_type |必要 |授與的類型，針對授權碼流程來說，必須是 `authorization_code` 。 |
| scope |建議 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 `openid` 範圍指示使用 id_tokens 參數形式的權限，以登入使用者及取得使用者相關資料。 它可用來將權杖傳送到您應用程式本身的後端 Web API，其會由與用戶端相同的應用程式識別碼來表示。 `offline_access` 範圍表示您的應用程式需要重新整理權杖，才能長久存取資源。 |
| code |必要 |您在流程的第一個階段中取得的授權碼。 |
| redirect_uri |必要 |應用程式的 `redirect_uri` 參數，您會在此處收到授權碼。 |
| client_secret |必要 |您在 [Azure 入口網站](https://portal.azure.com/)中產生的應用程式祕密。 這個應用程式密碼是重要的安全性構件， 您應該要用安全的方法把它儲存在您的伺服器上。 您也應該定期輪換此用戶端祕密。 |

成功的權杖回應看起來如下：

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
| token_type |權杖類型值。 Azure AD 唯一支援的類型是 `Bearer`。 |
| access_token |您所要求已簽署的 JWT 權杖。 |
| scope |權杖有效的範圍。 這些可用來快取權杖，供以後使用。 |
| expires_in |存取權杖的有效時間長度 (以秒為單位)。 |
| refresh_token |OAuth 2.0 重新整理權杖。 應用程式在目前的權杖過期之後，可以使用這個權杖來取得其他權杖。 重新整理權杖的有效期很長，而且可用來長期保留資源存取權。 如需詳細資訊，請參閱 [B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 請注意，您必須同時在授權和權杖要求中使用範圍 `offline_access`，才能接收重新整理權杖。 |

錯誤回應看起來如下：

```
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用以分類所發生的錯誤類型，以及可用來回應錯誤的錯誤碼字串。 |
| error_description |協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |

## <a name="use-the-token"></a>使用權杖
現在您已成功取得存取權杖，因此可在對後端 Web API 發出的要求中使用該權杖，方法是在 `Authorization` 標頭中加入該權杖：

```
GET /tasks
Host: https://mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-token"></a>重新整理權杖
識別碼權杖只會短暫存在。 因此在該權杖過期之後，您必須重新整理權杖，才能繼續存取資源。 方法是向 `/token` 端點送出另一個 `POST` 要求， 此時，請提供 `refresh_token` 參數，而不是 `code` 參數：

```
POST fabrikamb2c.onmicrosoft.com/oauth2/v2.0/token?p=b2c_1_sign_in HTTP/1.1
Host: https://fabrikamb2c.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=openid offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_secret=<your-application-secret>
```

| 參數 | 必要 | 說明 |
| --- | --- | --- |
| p |必要 |用來取得原始重新整理權杖的原則。 您無法在此要求中使用不同的原則。 請注意，您要將這個參數新增到查詢字串，而不是 POST 主體。 |
| client_id |必要 |[Azure 入口網站](https://portal.azure.com/) 指派給您應用程式的應用程式識別碼。 |
| grant_type |必要 |授與的類型，對於授權碼流程的這個階段來說，必須是重新整理權杖。 |
| scope |建議 |範圍的空格分隔清單。 向 Azure AD 指出要求兩個權限的單一範圍值。 `openid` 範圍指示使用識別碼權杖形式的權限，以登入使用者及取得使用者相關資料。 它可用來將權杖傳送到您應用程式本身的後端 Web API，其會由與用戶端相同的應用程式識別碼來表示。 `offline_access` 範圍表示您的應用程式將需要重新整理權杖，才能長久存取資源。 |
| redirect_uri |建議 |應用程式的 `redirect_uri` 參數，您會在此處收到授權碼。 |
| refresh_token |必要 |您在流程的第二個階段中取得的原始重新整理權杖。 請注意，您必須同時在授權和權杖要求中使用範圍 `offline_access`，才能接收重新整理權杖。 |
| client_secret |必要 |您在 [Azure 入口網站](https://portal.azure.com/)中產生的應用程式祕密。 這個應用程式密碼是重要的安全性構件， 您應該要用安全的方法把它儲存在您的伺服器上。 您也應該定期輪換此用戶端祕密。 |

成功的權杖回應看起來如下：

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
| token_type |權杖類型值。 Azure AD 唯一支援的類型是 `Bearer`。 |
| access_token |您所要求已簽署的 JWT 權杖。 |
| scope |權杖有效的範圍，可用於快取權杖，供以後使用。 |
| expires_in |存取權杖的有效時間長度 (以秒為單位)。 |
| refresh_token |OAuth 2.0 重新整理權杖。 應用程式在目前的權杖過期之後，可以使用這個權杖來取得其他權杖。  重新整理權杖的有效期很長，而且可用來長期保留資源存取權。 如需詳細資訊，請參閱 [B2C 權杖參考](active-directory-b2c-reference-tokens.md)。 |

錯誤回應看起來如下：

```
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| 參數 | 說明 |
| --- | --- |
| 錯誤 |可用以分類所發生的錯誤類型，以及可用來回應錯誤的錯誤碼字串。 |
| error_description |協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |

## <a name="send-a-sign-out-request"></a>傳送登出要求
當您想要讓使用者登出應用程式時，只是清除應用程式的 Cookie，或是結束使用者的工作階段是不夠的。 您也必須把使用者重新導向至 Azure AD 來登出。如果您沒有這麼做，使用者可能不必重新輸入認證，就能夠向您的應用程式重新驗證自己的身分。 這是因為使用者將擁有有效的 Azure AD 單一登入工作階段。

您只要將使用者重新導向至 `end_session` 端點 (列於之前在＜驗證識別碼權杖＞一節中所述的 OpenID Connect 中繼資料文件中) 即可：

```
GET https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/oauth2/v2.0/logout?
p=b2c_1_sign_in
&post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| 參數 | 必要？ | 說明 |
| --- | --- | --- |
| p |必要 |您想要用來將使用者登出應用程式的原則。 |
| post_logout_redirect_uri |建議 |使用者在成功登出之後，應該要前往的 URL。若未包含此資料，Azure AD B2C 就會向使用者顯示一般訊息。 |

> [!NOTE]
> 儘管將使用者導向至 `end_session` 端點就能清除使用者於 Azure AD B2C 的部分單一登入狀態，但無法讓使用者登出其社交身分識別提供者 (IDP) 工作階段。 如果使用者之後在登入時選取相同的 IDP，該使用者不必輸入自己的認證，就能讓系統重新驗證自己的身分。 如果使用者想要登出您的 B2C 應用程式，不一定代表他們想要登出自己的 Facebook 帳戶。 不過，如果是使用本機帳戶，使用者的工作階段便會正確地結束。
> 
> 

## <a name="use-your-own-b2c-tenant"></a>使用您自己的 B2C 租用戶
如果您想要親自嘗試這些要求，必須先執行下列三個步驟，然後用您自己的值來取代上述的範例值：

1. [建立 B2C 租用戶](active-directory-b2c-get-started.md)，並在要求中使用您租用戶的名稱。
2. [建立應用程式](active-directory-b2c-app-registration.md)來取得應用程式識別碼。 在應用程式中加入 Web 應用程式/Web API。 也可以選擇建立應用程式祕密。
3. [建立您的原則](active-directory-b2c-reference-policies.md) 來取得原則名稱。

