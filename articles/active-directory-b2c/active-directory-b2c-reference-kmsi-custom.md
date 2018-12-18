---
title: 讓我在 Azure Active Directory B2C 中保持登入 | Microsoft Docs
description: 了解如何在 Azure Active Directory B2C 中啟用「讓我保持登入 (KMSI)」。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6d58a62ef70cb5bacb44a3a9832516a30fc91ffa
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248054"
---
# <a name="enable-keep-me-signed-in-kmsi-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中啟用「讓我保持登入 (KMSI)」

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

您可以為 Azure Active Directory (Azure AD) B2C 中的 Web 和原生應用程式啟用「讓我保持登入 (KMSI)」功能。 此功能會授與讓使用者回到應用程式，而不需要重新輸入使用者名稱和密碼提示的存取權。 當使用者登出時，即會撤銷此存取權。 

使用者應該不要在公用電腦上啟用此選項。 

![啟用讓我保持登入](./media/active-directory-b2c-reference-kmsi-custom/kmsi.PNG)

## <a name="prerequisites"></a>必要條件

已設定為允許本機帳戶註冊和登入的 Azure AD B2C 租用戶。 如果您還沒有租用戶，您可以使用[教學課程：建立 Azure Active Directory B2C 租用戶](tutorial-create-tenant.md)中的步驟建來立一個。

## <a name="add-a-content-definition-element"></a>新增內容定義元素 

在擴充檔案的 **BuildingBlocks** 元素底下，新增 **ContentDefinitions** 元素。 

1. 在 **ContentDefinitions** 元素底下，新增識別碼為 `api.signuporsigninwithkmsi` 的 **ContentDefinition** 元素。
2. 在 **ContentDefinition** 元素底下，新增 **LoadUri**、**RecoveryUri** 和 **DataUri** 元素。 **DataUri** 元素的 `urn:com:microsoft:aad:b2c:elements:unifiedssp:1.1.0` 值是機器可識別的識別碼，會在登入頁面中帶出 KMSI 核取方塊。 請勿變更此值。 

    ```XML
    <BuildingBlocks>
      <ContentDefinitions>
        <ContentDefinition Id="api.signuporsigninwithkmsi">
          <LoadUri>~/tenant/default/unified.cshtml</LoadUri>
          <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
          <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.1.0</DataUri>
          <Metadata>
            <Item Key="DisplayName">Signin and Signup</Item>
          </Metadata>
        </ContentDefinition>
      </ContentDefinitions>
    </BuildingBlocks>                       
    ```

## <a name="add-a-sign-in-claims-provider-for-a-local-account"></a>新增本機帳戶的登入宣告提供者  

您可以使用原則擴充檔案中的 **ClaimsProvider** 元素，將本機帳戶登入定義為宣告提供者：

1. 從您的工作目錄中開啟 TrustFrameworkExtensions.xml 檔案。 
2. 尋找 **ClaimsProviders** 元素。 如果該元素不存在，請在根項目下新增。 [入門套件](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip)包含本機帳戶登入宣告提供者。 
3. 以 **DisplayName** 和 **TechnicalProfile** 來新增 **ClaimsProvider** 元素，如下列範例所示：

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
        </TechnicalProfiles>
      </ClaimsProvider>
    </ClaimsProviders>
    ```

### <a name="add-the-application-identifiers-to-your-custom-policy"></a>將應用程式識別碼新增至您的自訂原則

將至應用程式識別碼新增至 TrustFrameworkExtensions.xml 檔案。

1. 在 TrustFrameworkExtensions.xml 檔案中，找到識別碼為 `login-NonInteractive` 的 **TechnicalProfile** 元素，以及識別碼為 `login-NonInteractive-PasswordChange` 的 **TechnicalProfile** 元素，並將 `IdentityExperienceFrameworkAppId` 的所有值取代為[使用者入門](active-directory-b2c-get-started-custom.md)中所述識別體驗架構應用程式的應用程式識別碼。

    ```
    <Item Key="client_id">8322dedc-cbf4-43bc-8bb6-141d16f0f489</Item>
    ```

2. 如[快速入門](active-directory-b2c-get-started-custom.md)中所述，以 Proxy 識別體驗架構應用程式的應用程式識別碼取代 `ProxyIdentityExperienceFrameworkAppId` 的所有值。
3. 儲存擴充檔案。

## <a name="create-a-kmsi-enabled-user-journey"></a>建立已啟用 KMSI 的使用者旅程圖

將本機帳戶的登入宣告提供者新增至使用者旅程圖。 

1. 開啟原則的基底檔案。 例如，TrustFrameworkBase.xml。
2. 尋找 **UserJourneys** 元素，並將使用 `SignUpOrSignIn` 識別碼的 **UserJourney** 元素內容整個複製。
3. 開啟擴充檔案。 例如，TrustFrameworkExtensions.xml，並尋找 **UserJourneys** 元素。 如果此元素不存在，請新增。
4. 貼上您複製的整個 **UserJourney** 元素，以作為 **UserJourneys** 元素的子項目。
5. 變更新使用者旅程圖的識別碼值。 例如： `SignUpOrSignInWithKmsi`。
6. 最後，在第一個協調流程步驟中，將 **ContentDefinitionReferenceId** 的值變更為 `api.signuporsigninwithkmsi`。 設定此值會啟用使用者旅程圖中的核取方塊。 
7. 儲存並上傳擴充檔案，並確保所有驗證都成功。

    ```XML
    <UserJourneys>
      <UserJourney Id="SignUpOrSignInWithKmsi">
        <OrchestrationSteps>
          <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsigninwithkmsi">
            <ClaimsProviderSelections>
              <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmailExchange" />
            </ClaimsProviderSelections>
            <ClaimsExchanges>
              <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="2" Type="ClaimsExchange">
            <Preconditions>
              <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
                <Value>objectId</Value>
                <Action>SkipThisOrchestrationStep</Action>
              </Precondition>
            </Preconditions>
            <ClaimsExchanges>
              <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <!-- This step reads any user attributes that we may not have received when in the token. -->
          <OrchestrationStep Order="3" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
        </OrchestrationSteps>
        <ClientDefinition ReferenceId="DefaultWeb" />
      </UserJourney>
    </UserJourneys>
    ```

## <a name="create-a-relying-party-file"></a>建立信賴憑證者檔案

更新信賴憑證者 (RP) 檔案，此檔案將起始您剛才建立的使用者旅程圖。

1. 在您的工作目錄中建立一份 SignUpOrSignIn.xml 檔案複本，然後重新命名。 例如，SignUpOrSignInWithKmsi.xml。
2. 開啟新檔案，並以唯一值更新 **TrustFrameworkPolicy** 的 **PolicyId** 屬性。 這是您原則的名稱。 例如： `SignUpOrSignInWithKmsi`。
3. 變更 **DefaultUserJourney** 元素的 **ReferenceId** 屬性，以符合您建立的新使用者旅程圖識別碼。 例如： `SignUpOrSignInWithKmsi`。

    KMSI 會使用 **UserJourneyBehaviors** 元素來設定。 **KeepAliveInDays** 屬性會控制使用者保持登入的時間長度。 在下列範例中，KMSI 工作階段會在 `7` 天後自動到期，而不論使用者執行無訊息驗證的頻率為何。 將 **KeepAliveInDays** 值設定為 `0` 會關閉 KMSI 功能。 根據預設，此值為 `0`。 如果 **SessionExpiryType** 的值是 `Rolling`，那麼，每當使用者執行無訊息驗證，KMSI 工作階段即會延長 `7` 天。  如果選取 `Rolling`，您應該將天數保留為最小值。 

    **SessionExpiryInSeconds** 的值代表 SSO 工作階段的到期時間。 這會由 Azure AD B2C 在內部使用，以檢查 KMSI 的工作階段是否已過期。 **KeepAliveInDays** 值會決定 Web 瀏覽器中 SSO cookie 的「到期時間」/「最大存留期」值。 不同於 **SessionExpiryInSeconds**，**KeepAliveInDays** 會用來防止在瀏覽器關閉時清除 cookie。 使用者只能在 SSO 工作階段的 cookie 存在時 (這會由 **KeepAliveInDays** 控制)，以及尚未過期時 (這會由 **SessionExpiryInSeconds** 控制)，才能進行無訊息登入。 我們建議您將 **SessionExpiryInSeconds** 的值設定為與 **KeepAliveInDays** 相同的時間 (以秒為單位)，如下列範例所示。

    ```XML
    <RelyingParty>
      <DefaultUserJourney ReferenceId="SignUpOrSignInWithKmsi" />
      <UserJourneyBehaviors>
        <SingleSignOn Scope="Tenant" KeepAliveInDays="7" />
        <SessionExpiryType>Absolute</SessionExpiryType>
        <SessionExpiryInSeconds>604800</SessionExpiryInSeconds>
      </UserJourneyBehaviors>
      <TechnicalProfile Id="PolicyProfile">
        <DisplayName>PolicyProfile</DisplayName>
        <Protocol Name="OpenIdConnect" />
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="displayName" />
          <OutputClaim ClaimTypeReferenceId="givenName" />
          <OutputClaim ClaimTypeReferenceId="surname" />
          <OutputClaim ClaimTypeReferenceId="email" />
          <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        </OutputClaims>
        <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```

4. 儲存變更然後上傳檔案。
5. 若要測試您上傳的自訂原則，在 Azure 入口網站中，移至原則視窗，然後選取 [立即執行]。

您可以在[這裡](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/keep%20me%20signed%20in)找到範例原則。








