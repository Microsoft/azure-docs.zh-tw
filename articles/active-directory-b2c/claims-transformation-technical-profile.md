---
title: 定義 Azure Active Directory B2C 自訂原則中的宣告轉換技術設定檔 | Microsoft Docs
description: 定義 Azure Active Directory B2C 自訂原則中的宣告轉換技術設定檔。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: fd1e2aa5162ce9263d521edf3ae11e0508353b46
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382444"
---
# <a name="define-a-claims-transformation-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>定義 Azure Active Directory B2C 自訂原則中的宣告轉換技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

 宣告轉換技術設定檔可讓您呼叫輸出宣告轉換，以操作宣告值、驗證宣告，或設定一組輸出宣告的預設值。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `Proprietary`。 **handler** 屬性必須包含 Azure AD B2C 所使用之通訊協定處理常式組件的完整名稱：`Web.TPEngine.Providers.ClaimsTransformationProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`。

下列範例顯示的是宣告轉換技術設定檔：

```XML
<TechnicalProfile Id="Facebook-OAUTH-UnLink">
    <DisplayName>Unlink Facebook</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ClaimsTransformationProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...    
```

## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素是必要項目。 請至少提供一個技術設定檔傳回的輸出宣告。 下列範例顯示了如何設定輸出宣告中的預設值：

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="ageGroup" DefaultValue="Undefined" />
  <OutputClaim ClaimTypeReferenceId="ageGroupValueChanged" DefaultValue="false" />
</OutputClaims>
```

## <a name="output-claims-transformations"></a>輸出宣告轉換

**OutputClaimsTransformations** 元素可能含有 **OutputClaimsTransformation** 的集合，用於修改宣告或產生新的宣告。 下列技術設定檔會呼叫 **RemoveAlternativeSecurityIdByIdentityProvider** 宣告轉換。 此宣告轉換會從 **AlternativeSecurityIds** 的集合中移除社交身分識別。 此技術設定檔的輸出宣告是 **identityProvider2** (設為 `facebook.com`)，以及 **AlternativeSecurityIds**，該宣告含有移除 facebook.com 身分識別後，與該使用者有所關聯的社交身分識別清單。

```XML
<ClaimsTransformations>
  <ClaimsTransformation Id="RemoveAlternativeSecurityIdByIdentityProvider" 
TransformationMethod="RemoveAlternativeSecurityIdByIdentityProvider">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="IdentityProvider2"
TransformationClaimType="identityProvider" />
      <InputClaim ClaimTypeReferenceId="AlternativeSecurityIds" 
TransformationClaimType="collection" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="AlternativeSecurityIds" 
TransformationClaimType="collection" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations>
...
<TechnicalProfile Id="Facebook-OAUTH-UnLink">
    <DisplayName>Unlink Facebook</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ClaimsTransformationProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="identityProvider2" DefaultValue="facebook.com" AlwaysUseDefaultValue="true" />
    </OutputClaims>
    <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="RemoveAlternativeSecurityIdByIdentityProvider" />
    </OutputClaimsTransformations>
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

宣告轉換技術設定檔可讓您從任何使用者旅程圖協調流程步驟執行宣告轉換。 在下列範例中，協調流程步驟會呼叫其中一個取消連結的技術設定檔，例如 **UnLink-Facebook-OAUTH**。 此技術設定檔會呼叫宣告轉換技術設定檔 **RemoveAlternativeSecurityIdByIdentityProvider**，其會產生含有使用者社交身分識別清單的全新 **AlternativeSecurityIds2** 宣告，同時從集合中移除 Facebook 身分識別。

```XML
<UserJourney Id="AccountUnLink">
  <OrchestrationSteps>    
    ...
    <OrchestrationStep Order="8" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="UnLinkFacebookExchange" TechnicalProfileReferenceId="UnLink-Facebook-OAUTH" />
        <ClaimsExchange Id="UnLinkMicrosoftExchange" TechnicalProfileReferenceId="UnLink-Microsoft-OAUTH" />
        <ClaimsExchange Id="UnLinkGitHubExchange" TechnicalProfileReferenceId="UnLink-GitHub-OAUTH" />
      </ClaimsExchanges>
    </OrchestrationStep>
    ...
  </OrchestrationSteps>
</UserJourney>
```


## <a name="use-a-validation-technical-profile"></a>使用驗證技術設定檔

宣告轉換技術設定檔可用來驗證資訊。 在下列範例中，名稱為 **LocalAccountSignUpWithLogonEmail** 的[自我判斷技術設定檔](self-asserted-technical-profile.md)會要求使用者輸入兩次電子郵件，然後呼叫名稱為 **Validate-Email** 的[驗證技術設定檔](validation-technical-profile.md)，以驗證電子郵件。 **Validate-Email** 技術設定檔會呼叫宣告轉換 **AssertEmailAreEqual**，以比較兩個宣告 **email** 和 **emailRepeat**，如果根據指定的比較不相等，則會擲回例外狀況。

```XML
<ClaimsTransformations>
  <ClaimsTransformation Id="AssertEmailAreEqual" TransformationMethod="AssertStringClaimsAreEqual">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
      <InputClaim ClaimTypeReferenceId="emailRepeat" TransformationClaimType="inputClaim2" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    </InputParameters>
  </ClaimsTransformation>
</ClaimsTransformations>
```

宣告轉換技術設定檔會呼叫 **AssertEmailAreEqual** 宣告轉換，其會判斷使用者提供的電子郵件是否相同。

```XML
<TechnicalProfile Id="Validate-Email">
    <DisplayName>Unlink Facebook</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ClaimsTransformationProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="emailRepeat" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" />
  </OutputClaims>          
  <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="AssertEmailAreEqual" />
    </OutputClaimsTransformations>
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile> 
```

自我判斷技術設定檔可以呼叫驗證技術設定檔，並顯示 **UserMessageIfClaimsTransformationStringsAreNotEqual** 中繼資料中指定的錯誤訊息。

```XML
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>User ID signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    ...
    <Item Key="UserMessageIfClaimsTransformationStringsAreNotEqual">The email addresses you provided are not the same</Item>
  </Metadata>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" />
    <OutputClaim ClaimTypeReferenceId="emailRepeat" />
    ...
  </OutputClaims>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="Validate-Email" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>  
```