---
title: 在 Azure Active Directory B2C 的自訂原則中定義 OAuth1 技術設定檔 | Microsoft Docs
description: 在 Azure Active Directory B2C 的自訂原則中定義 OAuth1 技術設定檔。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 10c90b060c184bb911ac149640e8a9570b59e2fb
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382384"
---
# <a name="define-a-oauth1-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>在 Azure Active Directory B2C 自訂原則中定義 OAuth1 技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory (Azure AD) B2C 可提供 [OAuth 1.0 通訊協定](http://tools.ietf.org/html/rfc5849)識別提供者的支援。 本文說明技術設定檔的詳細規格，可用來與支援此標準化通訊協定的宣告提供者互動。 使用 OAuth1 技術設定檔時，您可以與 OAuth1 式識別提供者 (例如 Twitter) 同盟，讓您的使用者夠使用其現有的社交或企業識別來登入。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `OAuth1`。 例如，**Twitter-OAUTH1** 技術設定檔的通訊協定是 `OAuth1`。

```XML
<TechnicalProfile Id="Twitter-OAUTH1">
  <DisplayName>Twitter</DisplayName>
  <Protocol Name="OAuth1" />
  ...    
```

## <a name="input-claims"></a>輸入宣告

**InputClaims** 和 **InputClaimsTransformations** 元素是空的或不存在。

## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素包含 OAuth1 識別提供者傳回的宣告清單。 您可能需要將原則中定義的宣告名稱對應至識別提供者中定義的名稱。 只要設定 **DefaultValue** 屬性，您也可以包含識別提供者未傳回的宣告。

**OutputClaimsTransformations** 元素可能包含 **OutputClaimsTransformation** 的集合，用來修改輸出宣告或產生新的輸出宣告。

下列範例顯示 Twitter 識別提供者傳回的宣告：

- 對應至 **socialIdpUserId** 宣告的 **user_id** 宣告。
- 對應至 **displayName** 宣告的 **screen_name** 宣告。
- **email** 宣告沒有名稱對應。

技術設定檔也會傳回識別提供者未傳回的宣告： 

- 包含識別提供者名稱的 **identityProvider** 宣告。
- 具有預設值 `socialIdpAuthentication` 的 **authenticationSource** 宣告。

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="user_id" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="screen_name" />
  <OutputClaim ClaimTypeReferenceId="email" />
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="twitter.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
</OutputClaims>
```

## <a name="metadata"></a>中繼資料

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| client_id | 是 | 識別提供者的應用程式識別碼。 |
| ProviderName | 否 | 識別提供者的名稱。 |
| request_token_endpoint | 是 | 依據 RFC 5849 的要求權杖端點 URL。 |
| authorization_endpoint | 是 | 依據 RFC 5849 的授權端點 URL。 |
| access_token_endpoint | 是 | 依據 RFC 5849 的權杖端點 URL。 |
| ClaimsEndpoint | 否 | 使用者資訊端點的 URL。 | 
| ClaimsResponseFormat | 否 | 宣告回應格式。|

## <a name="cryptographic-keys"></a>密碼編譯金鑰

**CryptographicKeys** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| client_secret | 是 | 識別提供者應用程式的用戶端密碼。   | 

## <a name="redirect-uri"></a>重新導向 URI

當您設定識別提供者的重新導向 URL 時，請輸入 `https://login.microsoftonline.com/te/tenant/policyId/oauth1/authresp`。 請務必將 **tenant** 取代為您的租用戶名稱 (例如 contosob2c.onmicrosoft.com)，並將 **policyId** 取代為原則的識別碼 (例如 b2c_1a_policy)。 重新導向 URL 必須全部是小寫。 您應該為所有使用識別提供者登入的原則新增重新導向 URL。 

如果您使用 **b2clogin.com** 網域，而非使用 **login.microsoftonline.com**，請務必使用 b2clogin.com，而非使用 login.microsoftonline.com。

範例：

- [使用自訂原則新增 Twitter 作為 OAuth1 識別提供者](active-directory-b2c-custom-setup-twitter-idp.md)













