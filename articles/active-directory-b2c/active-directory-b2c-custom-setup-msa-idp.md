---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 Microsoft 帳戶 (MSA) 作為識別提供者 | Microsoft Docs
description: 透過 OpenID 連線 (OIDC) 通訊協定使用 Microsoft 作為識別提供者的範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6a981f112c97ee35b476c92f6f698e68a12a1363
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43336816"
---
# <a name="azure-active-directory-b2c-add-microsoft-account-msa-as-an-identity-provider-using-custom-policies"></a>Azure Active Directory B2C︰使用自訂原則新增 Microsoft 帳戶 (MSA) 作為識別提供者

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文將說明如何使用[自訂原則](active-directory-b2c-overview-custom.md)，讓 Microsoft 帳戶 (MSA) 帳戶的使用者登入。

## <a name="prerequisites"></a>必要條件
完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

這些步驟包括：

1.  建立 Microsoft 帳戶應用程式。
2.  將 Microsoft 帳戶應用程式金鑰新增至 Azure AD B2C
3.  將宣告提供者新增至原則
4.  向使用者旅程圖註冊 Microsoft 帳戶宣告提供者
5.  將原則上傳至 Azure AD B2C 租用戶並加以測試

## <a name="create-a-microsoft-account-application"></a>建立 Microsoft 帳戶應用程式
若要在 Azure Active Directory (Azure AD) B2C 中使用 Microsoft 帳戶做為識別提供者，您必須建立 Microsoft 帳戶應用程式，然後對其提供正確參數。 您需要 Microsoft 帳戶。 如果您沒有帳戶，請造訪 [https://www.live.com/](https://www.live.com/)。

1.  移至 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)，並使用您的 Microsoft 帳戶認證登入。
2.  按一下 [新增應用程式] 。

    ![Microsoft 帳戶 - 新增應用程式](media/active-directory-b2c-custom-setup-ms-account-idp/msa-add-new-app.png)

3.  提供應用程式的 [名稱]、[連絡人電子郵件]、取消核取 [讓我們幫助您開始]，然後按一下 [建立]。

    ![Microsoft 帳戶 - 註冊您的應用程式](media/active-directory-b2c-custom-setup-ms-account-idp/msa-app-name.png)

4.  複製 [應用程式識別碼] 的值。您必須使用此值，在租用戶中將 Microsoft 帳戶設定為識別提供者。

    ![Microsoft 帳戶 - 複製 [應用程式識別碼] 的值](media/active-directory-b2c-custom-setup-ms-account-idp/msa-app-id.png)

5.  按一下 [新增平台]

    ![Microsoft 帳戶 - 新增平台](media/active-directory-b2c-custom-setup-ms-account-idp/msa-add-platform.png)

6.  從 [平台] 清單中，選擇 [Web]。

    ![Microsoft 帳戶 - 從平台清單中選擇 Web](media/active-directory-b2c-custom-setup-ms-account-idp/msa-web.png)

7.  在 [重新導向 URI] 欄位中輸入 `https://{tenant}.b2clogin.com/te/{tenant}.onmicrosoft.com/oauth2/authresp`。 使用您的租用戶名稱 (例如 contosob2c) 來取代 **{tenant}**。

    ![Microsoft 帳戶 - 設定重新導向 URL](media/active-directory-b2c-custom-setup-ms-account-idp/msa-redirect-url.png)

8.  按一下 [應用程式密碼] 區段底下的 [產生新密碼]。 複製螢幕上顯示的新密碼。 您必須使用此值，在租用戶中將 Microsoft 帳戶設定為識別提供者。 此密碼是重要的安全性認證。

    ![Microsoft 帳戶 - 產生新密碼](media/active-directory-b2c-custom-setup-ms-account-idp/msa-generate-new-password.png)

    ![Microsoft 帳戶 - 複製新密碼](media/active-directory-b2c-custom-setup-ms-account-idp/msa-new-password.png)

9.  勾選 [進階選項] 區段底下顯示 [Live SDK 支援] 的方塊。 按一下 [檔案] 。

    ![Microsoft 帳戶 - Live SDK 支援](media/active-directory-b2c-custom-setup-ms-account-idp/msa-live-sdk-support.png)

## <a name="add-the-microsoft-account-application-key-to-azure-ad-b2c"></a>將 Microsoft 帳戶應用程式金鑰新增至 Azure AD B2C
具有 Microsoft 帳戶的同盟需要 Microsoft 帳戶的用戶端祕密，才能代表應用程式信任 Azure AD B2C。 您需要在 Azure AD B2C 租用戶中儲存您的 Microsoft 帳戶應用程式祕密：   

1.  移至您的 Azure AD B2C 租用戶，並選取 [B2C 設定] > [身分識別體驗架構]
2.  選取**原則金鑰**以檢視您的租用戶中可用的金鑰。
3.  按一下 [+新增]。
4.  針對 [選項] 使用 [手動]。
5.  針對 [名稱] 使用 `MSASecret`。  
    可能會自動加入前置詞 `B2C_1A_`。
6.  在 [祕密] 方塊中，從 https://apps.dev.microsoft.com 輸入您的 Microsoft 應用程式祕密
7.  針對 [金鑰使用方法] 使用 [簽章]。
8.  按一下 [建立] 
9.  確認您已建立金鑰 `B2C_1A_MSASecret`。

## <a name="add-a-claims-provider-in-your-extension-policy"></a>在擴充原則中新增宣告提供者
如果需要讓使用者使用 Microsoft 帳戶登入，您必須將 Microsoft 帳戶定義為宣告提供者。 換句話說，您必須指定要與 Azure AD B2C 通訊的端點。 此端點會提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。

定義 Microsoft 帳戶作為宣告提供者，方法是在擴充原則檔案中新增 `<ClaimsProvider>` 節點：

1.  從您的工作目錄中開啟擴充原則檔案 (TrustFrameworkExtensions.xml)。 如果您需要 XML 編輯器，請[試用 Visual Studio 程式碼](https://code.visualstudio.com/download)，這是一個輕巧的跨平台編輯器。
2.  尋找 `<ClaimsProviders>` 區段
3.  在 `ClaimsProviders` 元素下，新增下列 XML 程式碼片段：

    ```xml
<ClaimsProvider>
    <Domain>live.com</Domain>
    <DisplayName>Microsoft Account</DisplayName>
    <TechnicalProfiles>
    <TechnicalProfile Id="MSA-OIDC">
        <DisplayName>Microsoft Account</DisplayName>
        <Protocol Name="OpenIdConnect" />
        <Metadata>
        <Item Key="ProviderName">https://login.live.com</Item>
        <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
        <Item Key="response_types">code</Item>
        <Item Key="response_mode">form_post</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">0</Item>
        <Item Key="client_id">Your Microsoft application client id</Item>
        </Metadata>
    <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_MSASecret" />
    </CryptographicKeys>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" />
        <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
        <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
        <OutputClaim ClaimTypeReferenceId="email" />
        </OutputClaims>
        <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
        <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
        <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
        <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
        </OutputClaimsTransformations>
        <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
    </TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>
```

4.  使用您的 Microsoft 帳戶應用程式用戶端識別碼來取代 `client_id` 值

5.  儲存檔案。

## <a name="register-the-microsoft-account-claims-provider-to-sign-up-or-sign-in-user-journey"></a>向註冊或登入使用者旅程圖註冊 Microsoft 帳戶宣告提供者

目前，識別提供者已設定，但還未出現在任何註冊/登入畫面中。 您現在必須將 Microsoft 帳戶識別提供者新增至您的使用者 `SignUpOrSignIn` 使用者旅程圖。 若要使用，我們建立了重複的現有範本使用者旅程圖。  然後我們會新增 Microsoft 帳戶識別提供者：

> [!NOTE]
>
>如果您先前已從原則的基底檔案將 `<UserJourneys>` 元素複製到擴充檔案 `TrustFrameworkExtensions.xml`，就可以跳過本節。

1.  開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。
2.  尋找 `<UserJourneys>` 元素，並複製 `<UserJourneys>` 節點的完整內容。
3.  開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，並尋找 `<UserJourneys>` 元素。 如果此元素不存在，請新增。
4.  貼上您複製作為 `<UserJourneys>` 元素之子元素的 `<UserJourneys>` 節點的完整內容。

### <a name="display-the-button"></a>顯示按鈕
`<ClaimsProviderSelections>` 元素會定義宣告提供者選擇的選項清單和它們的順序。  `<ClaimsProviderSelection>` 元素類似於註冊/登入頁面上的識別提供者按鈕。 如果您新增 Microsoft 帳戶的 `<ClaimsProviderSelection>` 元素，當使用者登陸頁面時，會出現新的按鈕。 若要新增此元素：

1.  在您複製的使用者旅程圖中，尋找包含 `Id="SignUpOrSignIn"` 的 `<UserJourney>` 節點。
2.  找出包含 `Order="1"` 的 `<OrchestrationStep>` 節點
3.  在 `<ClaimsProviderSelections>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsProviderSelection TargetClaimsExchangeId="MicrosoftAccountExchange" />
```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作
現在已備妥按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Microsoft 帳戶通訊以接收權杖。 透過連結 Microsoft 帳戶宣告提供者的技術設定檔，將按鈕連結至動作：

1.  尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
2.  在 `<ClaimsExchanges>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsExchange Id="MicrosoftAccountExchange" TechnicalProfileReferenceId="MSA-OIDC" />
```

> [!NOTE]
>
>   * 請確定 `Id` 具有與上一節中之 `TargetClaimsExchangeId` 相同的值
>   * 請確定 `TechnicalProfileReferenceId` 識別碼設定為您稍早建立之技術設定檔的識別碼 (MSA-OIDC)。

## <a name="upload-the-policy-to-your-tenant"></a>將原則上傳至您的租用戶
1.  在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C] 刀鋒視窗。
2.  選取 [識別體驗架構]。
3.  開啟 [所有原則] 刀鋒視窗。
4.  選取 [上傳原則]。
5.  勾選 [覆寫已存在的原則] 方塊。
6.  **上傳** TrustFrameworkExtensions.xml，並確定它通過驗證

## <a name="test-the-custom-policy-by-using-run-now"></a>使用 [立即執行] 測試自訂原則

1.  開啟 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。
> [!NOTE]
>
>[立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。
2.  開啟 **B2C_1A_signup_signin**，此為您上傳的信賴憑證者 (RP) 自訂原則。 選取 [立即執行]。
3.  您應該能夠使用 Microsoft 帳戶進行登入。

## <a name="optional-register-the-microsoft-account-claims-provider-to-profile-edit-user-journey"></a>[選用] 向設定檔編輯使用者旅程圖註冊 Microsoft 帳戶宣告提供者
您也需要將 Microsoft 帳戶識別提供者新增至您的使用者 `ProfileEdit` 使用者旅程圖。 若要使用，我們會重複最後兩個步驟：

### <a name="display-the-button"></a>顯示按鈕
1.  開啟原則的擴充檔案 (例如，TrustFrameworkExtensions.xml)。
2.  在您複製的使用者旅程圖中，尋找包含 `Id="ProfileEdit"` 的 `<UserJourney>` 節點。
3.  找出包含 `Order="1"` 的 `<OrchestrationStep>` 節點
4.  在 `<ClaimsProviderSelections>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsProviderSelection TargetClaimsExchangeId="MSAExchange" />
```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作
1.  尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
2.  在 `<ClaimsExchanges>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsExchange Id="MSAExchange" TechnicalProfileReferenceId="MSA-OIDC" />
```

### <a name="test-the-custom-profile-edit-policy-by-using-run-now"></a>使用 [立即執行] 來測試自訂設定檔編輯原則
1.  開啟 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。
2.  開啟 **B2C_1A_ProfileEdit**，此為您上傳的信賴憑證者 (RP) 自訂原則。 選取 [立即執行]。
3.  您應該能夠使用 Microsoft 帳戶進行登入。

## <a name="download-the-complete-policy-files"></a>下載完整的原則檔案
選擇性：建議您在使用自訂原則逐步解說完成使用者入門後，使用自己的自訂原則檔案來建置您的情節，而非使用這些範例檔案。  [參考的範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-setup-msa-app)
