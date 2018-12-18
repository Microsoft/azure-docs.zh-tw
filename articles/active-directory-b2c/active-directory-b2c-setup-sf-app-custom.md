---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 Salesforce SAML 提供者 | Microsoft Docs
description: 深入了解如何建立及管理 Azure Active Directory B2C 自訂原則。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/15/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5b7621bde0be02b4656c4678438b94499bb82b5b
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43345032"
---
# <a name="azure-active-directory-b2c-sign-in-by-using-salesforce-accounts-via-saml"></a>Azure Active Directory B2C︰使用 Salesforce 帳戶透過 SAML 來登入

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文將說明如何使用[自訂原則](active-directory-b2c-overview-custom.md)設定特定 Salesforce 組織的使用者登入。

## <a name="prerequisites"></a>必要條件

### <a name="azure-ad-b2c-setup"></a>Azure AD B2C 設定

請確定您已完成示範如何在 Azure Active Directory B2C (Azure AD B2C) 中[開始使用自訂原則](active-directory-b2c-get-started-custom.md)的所有步驟。

其中包含：

* 建立 Azure AD B2C 租用戶。
* 建立 Azure AD B2C 應用程式。
* 註冊兩個原則引擎應用程式。
* 設定金鑰。
* 設定入門套件。

### <a name="salesforce-setup"></a>Salesforce 設定

在本文中，我們假設您已經完成下列作業：

* 註冊 Salesforce 帳戶。 您可以註冊[免費的開發人員版本帳戶](https://developer.salesforce.com/signup)。
* 為您的 Salesforce 組織[設定網域](https://help.salesforce.com/articleView?id=domain_name_setup.htm&language=en_US&type=0)。

## <a name="set-up-salesforce-so-users-can-federate"></a>設定 Salesforce，讓使用者可以建立同盟

若要協助 Azure AD B2C 與 Salesforce 通訊，您必須取得 Salesforce 中繼資料 URL。

### <a name="set-up-salesforce-as-an-identity-provider"></a>將 Salesforce 設定為識別提供者

> [!NOTE]
> 在本文中，我們假設您使用 [Salesforce Lightning 經驗](https://developer.salesforce.com/page/Lightning_Experience_FAQ)。

1. [登入 Salesforce](https://login.salesforce.com/)。 
2. 在左側功能表的 [設定] 下，展開 [身分識別]，然後按一下 [識別提供者]。
3. 按一下 [啟用識別提供者]。
4. 在 [選取憑證] 下，選取您想要讓 Salesforce 在與 Azure AD B2C 通訊時使用的憑證。 (您可以使用預設憑證。)按一下 [檔案] 。 

### <a name="create-a-connected-app-in-salesforce"></a>在 Salesforce 中建立連線應用程式

1. 在 [識別提供者] 分頁中，移至 [服務提供者]。
2. 按一下 [現在透過連線應用程式建立服務提供者]**。** 按一下這裡。
3. 在 [基本資訊] 下，為連線應用程式輸入必要值。
4. 在 [Web 應用程式設定] 下，選取 [啟用 SAML] 核取方塊。
5. 在 [實體識別碼] 欄位中，輸入下列 URL。 請確定您取代 `tenantName` 的值。
      ```
      https://tenantName.b2clogin.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase
      ```
6. 在 [ACS URL] 欄位中，輸入下列 URL。 請確定您取代 `tenantName` 的值。
      ```
      https://tenantName.b2clogin.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
      ```
7. 將所有其他設定保留預設值。
8. 捲動到清單底部，然後按一下 [儲存]。

### <a name="get-the-metadata-url"></a>取得中繼資料 URL

1. 在連線應用程式的概觀分頁上，按一下 [管理]。
2. 複製**中繼資料探索端點**的值，並將其儲存。 您將在本文稍後使用它。

### <a name="set-up-salesforce-users-to-federate"></a>設定 Salesforce 使用者以建立同盟

1. 在連線應用程式的 [管理] 分頁上，移至 [設定檔]。
2. 按一下 [管理設定檔]。
3. 選取您想要與 Azure AD B2C 建立同盟的設定檔 (或使用者群組)。 身為系統管理員，選取 [系統管理員] 核取方塊，以便使用您的 Salesforce 帳戶建立同盟。

## <a name="generate-a-signing-certificate-for-azure-ad-b2c"></a>產生 Azure AD B2C 的簽署憑證

傳送至 Salesforce 的要求需要 Azure AD B2C 簽署。 若要產生簽署憑證，請開啟 Azure PowerShell，然後執行下列命令。

> [!NOTE]
> 請確定您更新前兩行的租用戶名稱和密碼。

```PowerShell
$tenantName = "<YOUR TENANT NAME>.onmicrosoft.com"
$pwdText = "<YOUR PASSWORD HERE>"

$Cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -DnsName "SamlIdp.$tenantName" -Subject "B2C SAML Signing Cert" -HashAlgorithm SHA256 -KeySpec Signature -KeyLength 2048

$pwd = ConvertTo-SecureString -String $pwdText -Force -AsPlainText

Export-PfxCertificate -Cert $Cert -FilePath .\B2CSigningCert.pfx -Password $pwd
```

## <a name="add-the-saml-signing-certificate-to-azure-ad-b2c"></a>將 SAML 簽署憑證新增至 Azure AD B2C

將簽署憑證上傳至 Azure AD B2C 租用戶： 

1. 移至您的 Azure AD B2C 租用戶。 按一下 [設定]  >  [身分識別體驗架構]  >  [原則金鑰]。
2. 按一下 [+新增]，然後：
    1. 按一下 [選項]  >  [上傳]。
    2. 輸入**名稱** (例如，SAMLSigningCert)。 金鑰名稱前面會自動新增前置詞 B2C_1A_。
    3. 若要選取您的憑證，請選取 [上傳檔案控制項]。 
    4. 輸入您在 PowerShell 指令碼中設定的憑證密碼。
3. 按一下頁面底部的 [新增] 。
4. 請確認您已建立金鑰 (例如，B2C_1A_SAMLSigningCert)。 記下完整名稱 (包括 B2C_1A_)。 您將在原則稍後參考這個金鑰。

## <a name="create-the-salesforce-saml-claims-provider-in-your-base-policy"></a>在基底原則中建立 Salesforce SAML 宣告提供者

您需要將 Salesforce 定義為宣告提供者，使用者才能使用 Salesforce 登入。 換句話說，您需要指定將與 Azure AD B2C 通訊的端點。 此端點將提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。 若要完成這個操作，您可以在原則的擴充檔案中為 Salesforce 新增 `<ClaimsProvider>`：

1. 在您的工作目錄中開啟擴充檔案 (TrustFrameworkExtensions.xml)。
2. 找出區段 `<ClaimsProviders>`。 如果不存在，請在根節點下建立。
3. 新增 `<ClaimsProvider>`：

    ```XML
    <ClaimsProvider>
      <Domain>salesforce</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="salesforce">
          <DisplayName>Salesforce</DisplayName>
          <Description>Login with your Salesforce account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="RequestsSigned">false</Item>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="WantsSignedAssertions">false</Item>
            <Item Key="PartnerEntity">https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="userId"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="username"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="SAMLIdp" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

在 `<ClaimsProvider>` 節點下：

1. 將 `<ClaimsProvider>` 的值變更為可以讓 `<Domain>` 與其他識別提供者有所區別的唯一值。
2. 將 `<DisplayName>` 的值更新為宣告提供者的顯示名稱。 目前不使用此值。

### <a name="update-the-technical-profile"></a>更新技術設定檔

若要從 Salesforce 取得 SAML 權杖，請定義 Azure AD B2C 用來與 Azure Active Directory (Azure AD) 進行通訊的通訊協定。 在 `<ClaimsProvider>` 的 `<TechnicalProfile>` 元素中，執行這個動作：

1. 更新 `<TechnicalProfile>` 節點的識別碼。 此識別碼可讓原則的其他部分參考此技術設定檔。
2. 更新 `<DisplayName>` 的值。 這個值會顯示在登入分頁的登入按鈕上。
3. 更新 `<Description>` 的值。
4. Salesforce 使用 SAML 2.0 通訊協定。 請確認 `<Protocol>` 的值是 **SAML2**。

更新上述 XML 的 `<Metadata>` 區段，以反映特定 Salesforce 帳戶的設定。 在 XML 中，更新中繼資料值︰

1. 使用您稍早複製的 Salesforce 中繼資料 URL 更新 `<Item Key="PartnerEntity">` 的值。 其具備下列格式： 

    `https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp/connectedapp.xml`

2. 在 `<CryptographicKeys>` 區段中，將 `StorageReferenceId` 的執行個體值更新為簽署憑證的金鑰名稱 (例如，B2C_1A_SAMLSigningCert)。

### <a name="upload-the-extension-file-for-verification"></a>上傳擴充檔案準備驗證

您的原則現在已設定，因此 Azure AD B2C 知道與 Salesforce 的通訊方式。 嘗試上傳原則的擴充檔案，以確認到目前為止沒有任何問題。 若要上傳原則的擴充檔案：

1. 在 Azure AD B2C 租用戶中，移至 [所有原則] 刀鋒視窗。
2. 選取 [覆寫已存在的原則] 核取方塊。
3. 上傳擴充檔案 (TrustFrameworkExtensions.xml)。 請確定它不會驗證失敗。

## <a name="register-the-salesforce-saml-claims-provider-to-a-user-journey"></a>向使用者旅程圖註冊 Salesforce SAML 宣告提供者

接下來，將 Salesforce SAML 識別提供者新增至其中一個使用者旅程圖。 目前，識別提供者已設定，但還未出現在任何使用者註冊或登入分頁中。 若要將識別提供者新增至登入分頁上，首先，建立現有範本使用者旅程圖的複本。 然後修改範本，讓它也有 Azure AD 識別提供者。

1. 開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。
2. 尋找 `<UserJourneys>` 元素，然後複製含有 Id="SignUpOrSignIn" 的整個 `<UserJourney>`。
3. 開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)。 尋找 `<UserJourneys>` 元素。 如果元素不存在，請建立。
4. 貼上複製的整個 `<UserJourney>` 作為 `<UserJourneys>` 元素的子元素。
5. 重新命名新 `<UserJourney>` 的識別碼 (例如，SignUpOrSignUsingContoso)。

### <a name="display-the-identity-provider-button"></a>顯示識別提供者按鈕

`<ClaimsProviderSelection>` 元素類似於註冊或登入分頁上的識別提供者按鈕。 新增 Salesforce 的 `<ClaimsProviderSelection>` 元素後，當使用者移至此分頁時，將會出現新的按鈕。 若要顯示識別提供者按鈕：

1. 在您剛才建立的 `<UserJourney>` 中，尋找含有 `Order="1"` 的 `<OrchestrationStep>`。
2. 新增下列 XML：

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

3. 將 `TargetClaimsExchangeId` 設定為邏輯值。 建議您與其他人一樣，遵循相同的慣例 (例如，\[ClaimProviderName\]Exchange)。

### <a name="link-the-identity-provider-button-to-an-action"></a>將識別提供者按鈕連結至動作

現在已備妥識別提供者按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Salesforce 通訊以接收 SAML 權杖。 作法是連結 Salesforce SAML 宣告提供者的技術設定檔：

1. 在 `<UserJourney>` 節點中，尋找含有 `Order="2"` 的 `<OrchestrationStep>`。
2. 新增下列 XML：

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="ContosoProfile" />
    ```

3. 將 `Id` 更新為您稍早用於 `TargetClaimsExchangeId` 的相同值。
4. 將 `TechnicalProfileReferenceId` 更新為您稍早建立之技術設定檔的 `Id` (例如，ContosoProfile)。

### <a name="upload-the-updated-extension-file"></a>上傳更新的擴充檔案

您已修改擴充檔案。 儲存並上傳這個檔案。 請確定所有驗證都成功。

### <a name="update-the-relying-party-file"></a>更新信賴憑證者檔案

接下來，更新信賴憑證者 (RP) 檔案，此檔案將起始您剛才建立的使用者旅程圖：

1. 在您的工作目錄中建立一份 SignUpOrSignIn.xml 複本。 然後，將它重新命名 (例如，SignUpOrSignInWithAAD.xml)。
2. 開啟新檔案，將 `<TrustFrameworkPolicy>` 的 `PolicyId` 屬性更新成唯一值。 這會成為您的原則名稱 (例如，SignUpOrSignInWithAAD)。
3. 修改 `<DefaultUserJourney>` 中的 `ReferenceId` 屬性，以符合您建立之新使用者旅程圖的 `Id` (例如，SignUpOrSignUsingContoso)。
4. 儲存變更然後上傳檔案。

## <a name="test-and-troubleshoot"></a>測試及疑難排解

若要測試您剛才上傳的自訂原則，在 Azure 入口網站中，移至原則刀鋒視窗，然後按一下 [立即執行]。 如果失敗，請參閱[針對自訂原則進行疑難排解](active-directory-b2c-troubleshoot-custom.md)。

## <a name="next-steps"></a>後續步驟

請將意見反應寄到 [AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com)。
