---
title: 定義 Azure Active Directory B2C 自訂原則中的 OpenId Connect 技術設定檔 | Microsoft Docs
description: 定義 Azure Active Directory B2C 自訂原則中的 OpenId Connect 技術設定檔。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 40922080857563b86d538586b90513381edb5d89
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382664"
---
# <a name="define-a-openid-connect-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>定義 Azure Active Directory B2C 自訂原則中的 OpenId Connect 技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory (Azure AD) B2C 可支援 [OpenId Connect](http://openid.net/2015/04/17/openid-connect-certification-program/) 通訊協定識別提供者。 OpenID Connect 1.0 會定義 OAuth 2.0 上的身分識別層，是先進新式驗證通訊協定的代表。  透過 OpenId Connect 技術設定檔，您可以與 OpenId Connect 式識別提供者 (例如 Azure AD) 形成同盟，讓使用者能夠使用其現有的社交或企業身分識別登入。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `OpenIdConnect`。 例如，**MSA-OIDC** 技術設定檔的通訊協定是 `OpenIdConnect`：

```XML
<TechnicalProfile Id="MSA-OIDC">
  <DisplayName>Microsoft Account</DisplayName>
  <Protocol Name="OpenIdConnect" />
  ...    
```

## <a name="input-claims"></a>輸入宣告

**InputClaims** 和 **InputClaimsTransformations** 元素不是必要項目。 但您可能會想要將其他參數傳送至您的識別提供者。 下列範例會將 **domain_hint** 查詢字串參數 (包含 `contoso.com` 的值) 新增至授權要求。

```XML
<InputClaims>
  <InputClaim ClaimTypeReferenceId="domain_hint" DefaultValue="contoso.com" />
</InputClaims>
```

## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素包含 OpenId Connect 識別提供者傳回的宣告清單。 您可能需要將原則中定義的宣告名稱對應至識別提供者中定義的名稱。 只要設定了 `DefaultValue` 屬性，也可以加入識別提供者未傳回的宣告。

**OutputClaimsTransformations** 元素可能含有 **OutputClaimsTransformation** 的集合，用於修改輸出宣告或產生新的輸出宣告。

下列範例顯示 Microsoft 帳戶識別提供者傳回的宣告：

- 對應至 **socialIdpUserId** 宣告的 **sub** 宣告。
- 對應至 **displayName** 宣告的 **name** 宣告。
- **email** 無名稱對應。

技術設定檔也會傳回識別提供者未傳回的宣告：

- **identityProvider** 宣告，其包含識別提供者的名稱。
- **authenticationSource** 宣告，包含 **socialIdpAuthentication** 的預設值。

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
  <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="sub" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
</OutputClaims>
```

## <a name="metadata"></a>中繼資料

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| client_id | 是 | 識別提供者的應用程式識別碼。 |
| IdTokenAudience | 否 | id_token 的對象。 如果已指定，Azure AD B2C 會檢查權杖是否在識別提供者傳回的宣告中，以及是否等於指定的宣告。 |
| METADATA | 是 | 根據 OpenID Connect Discovery 規格 (也就是已知的 openid 設定端點)，指向 JSON 設定文件的 URL。 |
| ProviderName | 否 | 識別提供者的名稱。 |
| response_types | 否 | 根據 OpenID Connect Core 1.0 規格的回應類型。 可能的值：`id_token`、`code` 或 `token`。 |
| response_mode | 否 | 識別提供者用於將結果傳送回 Azure AD B2C 的方法。 可能的值：`query`、`form_post` (預設) 或 `fragment`。 |
| scope | 否 | 根據 OpenID Connect Core 1.0 規格定義之存取要求的範圍。 例如，`openid`, `profile` 和 `email`。 |
| HttpBinding | 否 | 繫結至存取權杖和宣告權杖端點的預期 HTTP。 可能的值：`GET` 或 `POST`。  |
| ValidTokenIssuerPrefixes | 否 | 此金鑰可在使用多租戶識別提供者 (例如 Azure Active Directory) 時，用於登入每個租用戶。 |
| UsePolicyInRedirectUri | 否 | 表明在建構重新導向 URI 時，是否使用原則。 在設定識別提供者中的應用程式時，需要指定重新導向 URI。 重新導向 URI 會指向 Azure AD B2C，`https://login.microsoftonline.com/te/{tenant}/oauth2/authresp` (login.microsoftonline.com 可能會隨著 your-tenant-name.b2clogin.com 而變更)。  如果指定 `false`，則需要為每個使用的原則新增重新導向 URI。 例如： `https://login.microsoftonline.com/te/{tenant}/{policy}/oauth2/authresp` 。 |
| MarkAsFailureOnStatusCode5xx | 否 | 表明如果 Http 狀態碼在 5xx 範圍內，是否應該將外部服務的要求標記為失敗。 預設值為 `false`。 |
| DiscoverMetadataByTokenIssuer | 否 | 表明是否應該使用 JWT 權杖中的簽發者探索 OIDC 中繼資料。 |

## <a name="cryptographic-keys"></a>密碼編譯金鑰

**CryptographicKeys** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| client_secret | 是 | 識別提供者應用程式的用戶端密碼。 只有在 **response_types** 中繼資料設為 `code` 時，才需要密碼編譯金鑰。 在此情況下，Azure AD B2C 會進行另一次呼叫，以交換存取權杖的授權碼。 如果中繼資料設為 `id_token`，您可以省略密碼編譯金鑰。  |  

## <a name="redirect-uri"></a>重新導向 Uri
 
在設定識別提供者的重新導向 URI 時，請輸入 `https://login.microsoftonline.com/te/tenant/oauth2/authresp`。 請務必使用您的租用戶名稱 (例如 contosob2c.onmicrosoft.com) 或租用戶 ID 來取代 **tenant**。重新導向 URI 必須全部小寫。

如果使用的是 **b2clogin.com** 網域，而非使用 **login.microsoftonline.com**，請務必使用 b2clogin.com，而非使用 login.microsoftonline.com。

範例：

- [使用自訂原則新增 Microsoft 帳戶 (MSA) 作為識別提供者](active-directory-b2c-custom-setup-msa-idp.md)
- [使用 Azure AD 帳戶來登入](active-directory-b2c-setup-aad-custom.md)
- [使用自訂原則允許使用者登入多租用戶 Azure AD 識別提供者](active-directory-b2c-setup-commonaad-custom.md)

 














