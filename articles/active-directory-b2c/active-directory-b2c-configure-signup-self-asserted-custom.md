---
title: 修改自訂原則中的註冊並設定自我判斷提示提供者 | Microsoft Docs
description: 逐步解說如何在註冊中新增宣告並設定使用者輸入
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/29/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 37492e22b5615ae0b266bc8b2bb6d8f039fdaabe
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43336850"
---
# <a name="azure-active-directory-b2c-modify-sign-up-to-add-new-claims-and-configure-user-input"></a>Azure Active Directory B2C︰修改註冊以新增宣告並設定使用者輸入。

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

在本文中，您會在您的註冊使用者旅程中新增使用者所提供的輸入 (宣告)。  您會將輸入設定為下拉式清單，並定義它是否為必要項目。

## <a name="prerequisites"></a>必要條件

* 完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。  測試註冊/登入使用者旅程以註冊新的本機帳戶，再繼續進行。


透過註冊/登入就能從使用者收集初始資料。  其他宣告則可在稍後透過設定檔編輯使用者旅程來收集。 每當 Azure AD B2C 以互動方式直接從使用者收集資訊時，身分識別體驗架構會使用其 `selfasserted provider`。 只要使用此提供者就適用下列步驟。


## <a name="define-the-claim-its-display-name-and-the-user-input-type"></a>定義宣告、其顯示名稱和使用者輸入類型
讓我們要求使用者輸入他們的所在城市。  在 TrustFrameworkBase 原則檔的 `<ClaimsSchema>` 元素中新增下列元素：

```xml
<ClaimType Id="city">
  <DisplayName>city</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your city</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```
您也可以在這裡進行其他選擇以自訂宣告。  如需完整的結構描述，請參閱**身分識別體驗架構技術參考指南**。  本指南很快就會發佈到參考區段中。

* `<DisplayName>` 是一個字串，會定義使用者端的「標籤」

* `<UserHelpText>` 可協助使用者了解所需項目

* `<UserInputType>` 有下面四個特別點出的選項︰
    * `TextBox`
```xml
<ClaimType Id="city">
  <DisplayName>city where you work</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your city</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

    * `RadioSingleSelectduration` - 強制讓您只能選取單一項目。
```xml
<ClaimType Id="city">
  <DisplayName>city where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>RadioSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="Kirkland" Value="kirkland" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

    * `DropdownSingleSelect` - 只允許選取有效值。

![下拉式清單選項的螢幕擷取畫面](./media/active-directory-b2c-configure-signup-self-asserted-custom/dropdown-menu-example.png)


```xml
<ClaimType Id="city">
  <DisplayName>city where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="Kirkland" Value="kirkland" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```


* `CheckboxMultiSelect` 允許選取一個或多個值。

![多重選取選項的螢幕擷取畫面](./media/active-directory-b2c-configure-signup-self-asserted-custom/multiselect-menu-example.png)


```xml
<ClaimType Id="city">
  <DisplayName>Receive updates from which cities?</DisplayName>
  <DataType>string</DataType>
  <UserInputType>CheckboxMultiSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="Kirkland" Value="kirkland" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

## <a name="add-the-claim-to-the-sign-upsign-in-user-journey"></a>將宣告新增至在使用者旅程的註冊/登入

1. 以 `<OutputClaim ClaimTypeReferenceId="city"/>` 的形式將宣告新增至 TechnicalProfile `LocalAccountSignUpWithLogonEmail` (可在 TrustFrameworkBase 原則檔中找到)。  請注意，此 TechnicalProfile 使用 SelfAssertedAttributeProvider。

  ```xml
  <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
    <DisplayName>Email signup</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    <Metadata>
      <Item Key="IpAddressClaimReferenceId">IpAddress</Item>
      <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
      <Item Key="language.button_continue">Create</Item>
    </Metadata>
    <CryptographicKeys>
      <Key Id="issuer_secret" StorageReferenceId="TokenSigningKeyContainer" />
    </CryptographicKeys>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="objectId" />
      <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true" />
      <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
      <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
      <OutputClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" DefaultValue="true" />
      <OutputClaim ClaimTypeReferenceId="authenticationSource" />
      <OutputClaim ClaimTypeReferenceId="newUser" />
      <!-- Optional claims, to be collected from the user -->
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surName" />
      <OutputClaim ClaimTypeReferenceId="city"/>
    </OutputClaims>
    <ValidationTechnicalProfiles>
      <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail" />
    </ValidationTechnicalProfiles>
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
  </TechnicalProfile>
  ```

2. 以 `<PersistedClaim ClaimTypeReferenceId="city" />` 的形式將宣告新增至 AAD-UserWriteUsingLogonEmail，以在收集使用者的宣告後將宣告寫入到 AAD 目錄。 如果您不想在目錄中保留宣告以供日後使用，則可略過此步驟。

  ```xml
  <!-- Technical profiles for local accounts -->
  <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
    <Metadata>
      <Item Key="Operation">Write</Item>
      <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    </Metadata>
    <IncludeInSso>false</IncludeInSso>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" Required="true" />
    </InputClaims>
    <PersistedClaims>
      <!-- Required claims -->
      <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" />
      <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password" />
      <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
      <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration" />
      <!-- Optional claims. -->
      <PersistedClaim ClaimTypeReferenceId="givenName" />
      <PersistedClaim ClaimTypeReferenceId="surname" />
      <PersistedClaim ClaimTypeReferenceId="city" />
    </PersistedClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="objectId" />
      <OutputClaim ClaimTypeReferenceId="newUser" PartnerClaimType="newClaimsPrincipalCreated" />
      <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
      <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
      <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
    </OutputClaims>
    <IncludeTechnicalProfile ReferenceId="AAD-Common" />
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
  </TechnicalProfile>
  ```

3. 以 `<OutputClaim ClaimTypeReferenceId="city" />` 的形式將宣告新增至 TechnicalProfile，以在使用者登入時從目錄中讀取此宣告

  ```xml
  <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
    <Metadata>
      <Item Key="Operation">Read</Item>
      <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
      <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">An account could not be found for the provided user ID.</Item>
    </Metadata>
    <IncludeInSso>false</IncludeInSso>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames" Required="true" />
    </InputClaims>
    <OutputClaims>
      <!-- Required claims -->
      <OutputClaim ClaimTypeReferenceId="objectId" />
      <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
      <!-- Optional claims -->
      <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="otherMails" />
      <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
      <OutputClaim ClaimTypeReferenceId="city" />
    </OutputClaims>
    <IncludeTechnicalProfile ReferenceId="AAD-Common" />
  </TechnicalProfile>
  ```

4. 將 `<OutputClaim ClaimTypeReferenceId="city" />` 新增至 RP 原則檔案 SignUporSignIn.xml，讓系統在使用者旅程成功後將這個宣告傳送至權杖中的應用程式。

  ```xml
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
  ```

## <a name="test-the-custom-policy-using-run-now"></a>使用 [立即執行] 測試自訂原則

1. 開啟 [Azure AD B2C] 刀鋒視窗，然後巡覽至 [識別體驗架構] > [自訂原則]。
2. 選取您上傳的自訂原則，按一下 [立即執行] 按鈕。
3. 您應該可以使用電子郵件地址註冊。

測試模式中的註冊畫面看起來應該像這樣：

![修改過的註冊選項螢幕擷取畫面](./media/active-directory-b2c-configure-signup-self-asserted-custom/signup-with-city-claim-dropdown-example.png)

  傳回到應用程式的權杖現在會包含 `city` 宣告，如下所示
```json
{
  "exp": 1493596822,
  "nbf": 1493593222,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "sub": "9c2a3a9e-ac65-4e46-a12d-9557b63033a9",
  "aud": "4e87c1dd-e5f5-4ac8-8368-bc6a98751b8b",
  "acr": "b2c_1a_trustf_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1493593222,
  "auth_time": 1493593222,
  "email": "joe@outlook.com",
  "given_name": "Joe",
  "family_name": "Ras",
  "city": "Bellevue",
  "name": "unknown"
}
```

## <a name="optional-remove-email-verification-from-signup-journey"></a>選擇性：移除註冊旅程中的電子郵件驗證

若要略過電子郵件驗證，原則的作者可以選擇移除 `PartnerClaimType="Verified.Email"`。 除非您移除 “Required” = true，否則電子郵件地址雖為必要項目，但不會進行驗證。  請仔細想清楚此選項是否適用於您的使用案例！

入門套件中 TrustFrameworkBase 原則檔案的 `<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">` 預設會啟用驗證的電子郵件︰
```xml
<OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true" />
```

## <a name="next-steps"></a>後續步驟

藉由變更下列 TechnicalProfiles，將新的宣告新增至社交帳戶登入的的流程。 社交/同盟帳戶登入會使用這些項目，以 alternativeSecurityId 作為定位器來寫入和讀取使用者資料。
```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
```
