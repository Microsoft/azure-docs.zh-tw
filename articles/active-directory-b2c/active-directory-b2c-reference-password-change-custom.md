---
title: Azure Active Directory B2C 中的自助式密碼變更 | Microsoft Docs
description: 此主題示範如何在 Azure Active Directory B2C 中為您的取用者設定自助式密碼變更。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/05/2016
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 491b3988a6581387c71b4214907e689119fcb979
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344984"
---
# <a name="azure-active-directory-b2c-configure-password-change-in-custom-policies"></a>Azure Active Directory B2C：在自訂原則中設定密碼變更  
[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

利用密碼變更功能，登入的取用者 (使用本機帳戶) 可以變更其密碼而不需要透過電子郵件驗證來證明其真實性，如[自助式密碼重設流程](active-directory-b2c-reference-sspr.md)中所述。 如果取用者進入密碼變更流程時工作階段到期，系統會提示使用者重新登入。 

## <a name="prerequisites"></a>必要條件

如[開始使用](active-directory-b2c-get-started-custom.md)所述，設定為完成本機帳戶註冊/登入的 Azure AD B2C 租用戶。

## <a name="how-to-configure-password-change-in-custom-policy"></a>如何在自訂原則中設定密碼變更

若要設定自訂原則中的密碼變更，請在您信任的可構擴充原則中進行下列變更。 

## <a name="define-a-claimtype-oldpassword"></a>定義 ClaimType 'oldPassword'

您的自訂原則的整體結構必須包含 `ClaimsSchema` 並定義新的 `ClaimType` 'oldPassword'，如下所示。 

```XML
  <BuildingBlocks>
    <ClaimsSchema>
      <ClaimType Id="oldPassword">
        <DisplayName>Old Password</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>Enter password</UserHelpText>
        <UserInputType>Password</UserInputType>
      </ClaimType>
    </ClaimsSchema>
  </BuildingBlocks>
```

這些元素的用途如下所示：

- `ClaimsSchema` 會定義要驗證的宣告。  在此情況下，將會驗證「舊密碼」。 

## <a name="add-a-password-change-claims-provider-with-its-supporting-elements"></a>新增密碼變更宣告提供者與及其支援元素

密碼變更宣告提供者將

1. 驗證使用者的舊密碼
2. 如果「新密碼」符合「確認新密碼」，則這個值會儲存在 B2C 資料存放區，因此變更密碼已成功。 

![影像](images/passwordchange.jpg)

將下列宣告提供者新增至您的擴充原則。 

```XML
<ClaimsProviders>
    <ClaimsProvider>
      <DisplayName>Local Account SignIn</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="login-NonInteractive">
          <Metadata>
           <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
           <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
            <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
          </InputClaims>
        </TechnicalProfile>
        <TechnicalProfile Id="login-NonInteractive-PasswordChange">
          <DisplayName>Local Account SignIn</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">We can't seem to find your account</Item>
            <Item Key="UserMessageIfInvalidPassword">Your password is incorrect</Item>
            <Item Key="UserMessageIfOldPasswordUsed">Looks like you used an old password</Item>
            <Item Key="ProviderName">https://sts.windows.net/</Item>
            <Item Key="METADATA">https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/.well-known/openid-configuration</Item>
            <Item Key="authorization_endpoint">https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/oauth2/token</Item>
            <Item Key="response_types">id_token</Item>
            <Item Key="response_mode">query</Item>
            <Item Key="scope">email openid</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
            <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
          </Metadata>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="signInName" PartnerClaimType="username" Required="true" />
            <InputClaim ClaimTypeReferenceId="oldPassword" PartnerClaimType="password" Required="true" />
            <InputClaim ClaimTypeReferenceId="grant_type" DefaultValue="password" />
            <InputClaim ClaimTypeReferenceId="scope" DefaultValue="openid" />
            <InputClaim ClaimTypeReferenceId="nca" PartnerClaimType="nca" DefaultValue="1" />
            <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
            <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="oid" />
            <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="userPrincipalName" PartnerClaimType="upn" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
          </OutputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
      <DisplayName>Local Account Password Change</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="LocalAccountWritePasswordChangeUsingObjectId">
          <DisplayName>Change password (username)</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
          </CryptographicKeys>
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="objectId" />
          </InputClaims>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="oldPassword" Required="true" />
            <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
            <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="login-NonInteractive-PasswordChange" />
            <ValidationTechnicalProfile ReferenceId="AAD-UserWritePasswordUsingObjectId" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
  </ClaimsProviders>
```



### <a name="add-the-application-ids-to-your-custom-policy"></a>將應用程式識別碼新增至您的自訂原則

在擴充檔案 (`TrustFrameworkExtensions.xml`) 中新增應用程式識別碼：

1. 在擴充檔案 (TrustFrameworkExtensions.xml) 中，尋找 `<TechnicalProfile Id="login-NonInteractive">` 元素和 `<TechnicalProfile Id="login-NonInteractive-PasswordChange">`

2. 如[入門](active-directory-b2c-get-started-custom.md)所述，將 `IdentityExperienceFrameworkAppId` 的所有執行個體以身分識別體驗架構應用程式的應用程式識別碼取代。 下列是一個範例：

   ```
   <Item Key="client_id">8322dedc-cbf4-43bc-8bb6-141d16f0f489</Item>
   ```

3. 如[快速入門](active-directory-b2c-get-started-custom.md)中所述，以 Proxy 識別體驗架構應用程式的應用程式識別碼，取代 `ProxyIdentityExperienceFrameworkAppId` 的所有執行個體。

4. 儲存擴充檔案。



## <a name="create-a-password-change-user-journey"></a>建立密碼變更使用者旅程圖

```XML
 <UserJourneys>
    <UserJourney Id="PasswordChange">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
          <ClaimsProviderSelections>
            <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSigninEmailExchange" />
          </ClaimsProviderSelections>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="NewCredentials" TechnicalProfileReferenceId="LocalAccountWritePasswordChangeUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
  </UserJourneys>
```

您已修改擴充檔案。 儲存並上傳這個檔案。 請確定所有驗證都成功。



## <a name="create-a-relying-party-rp-file"></a>建立信賴憑證者 (RP) 檔案

接下來，更新信賴憑證者 (RP) 檔案，此檔案將起始您剛才建立的使用者旅程圖：

1. 在您的工作目錄中建立 ProfileEdit.xml 的副本。 然後，將它重新命名 (例如，PasswordChange.xml)。
2. 開啟新檔案，將 `<TrustFrameworkPolicy>` 的 `PolicyId` 屬性更新成唯一值。 這是原則的名稱 (例如，PasswordChange)。
3. 修改 `<DefaultUserJourney>` 中的 `ReferenceId` 屬性，以符合您建立之新使用者旅程圖的 `Id` (例如，PasswordChange)。
4. 儲存變更然後上傳檔案。
5. 若要測試您上傳的自訂原則，在 Azure 入口網站中，移至原則刀鋒視窗，然後按一下 [立即執行]。




## <a name="link-to-password-change-sample-policy"></a>連結至密碼變更範例原則

您可以在[這裡](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/password-change)找到範例原則。 










