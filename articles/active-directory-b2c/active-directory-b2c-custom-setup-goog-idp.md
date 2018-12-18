---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 Google+ 作為 OAuth2 識別提供者 | Microsoft Docs
description: 透過 OAuth2 通訊協定使用 Google+ 作為識別提供者的範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: f076a906ba38e6c8e8c9530baba1607553b41ea6
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338323"
---
# <a name="azure-active-directory-b2c-add-google-as-an-oauth2-identity-provider-using-custom-policies"></a>Azure Active Directory B2C︰使用自訂原則新增 Google+ 作為 OAuth2 識別提供者

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本指南將說明如何使用[自訂原則](active-directory-b2c-overview-custom.md)，讓 Google+ 帳戶的使用者登入。

## <a name="prerequisites"></a>必要條件

完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

這些步驟包括：

1.  建立 Google+ 帳戶應用程式。
2.  將 Google+ 帳戶應用程式金鑰新增至 Azure AD B2C
3.  將宣告提供者新增至原則
4.  向使用者旅程圖註冊 Google+ 帳戶宣告提供者
5.  將原則上傳至 Azure AD B2C 租用戶並加以測試

## <a name="create-a-google-account-application"></a>建立 Google+ 帳戶應用程式
若要在 Azure Active Directory (Azure AD) B2C 中使用 Google+ 做為身分識別提供者，您必須建立 Google+ 應用程式，並對其提供正確參數。 您可以在此處註冊 Google+ 應用程式：[https://accounts.google.com/SignUp](https://accounts.google.com/SignUp)

1.  前往 [Google 開發人員主控台](https://console.developers.google.com/) ，並以您的 Google + 帳戶認證登入。
2.  按一下 [建立專案]，輸入 [專案名稱]，接著按一下 [建立]。

3.  按一下 [專案功能表]。

    ![Google+ 帳戶 - 選取專案](media/active-directory-b2c-custom-setup-goog-idp/goog-add-new-app1.png)

4.  按一下 [+] 按鈕。

    ![Google+ 帳戶 - 建立新的專案](media/active-directory-b2c-custom-setup-goog-idp//goog-add-new-app2.png)

5.  輸入 [專案名稱]，然後按一下 [建立]。

    ![Google+ 帳戶 - 新增專案](media/active-directory-b2c-custom-setup-goog-idp//goog-app-name.png)

6.  等候專案就緒，然後按一下 [專案功能表]。

    ![Google+ 帳戶 - 等候新的專案就緒可供使用](media/active-directory-b2c-custom-setup-goog-idp//goog-select-app1.png)

7.  按一下專案名稱。

    ![Google+ 帳戶 - 選取新的專案](media/active-directory-b2c-custom-setup-goog-idp//goog-select-app2.png)

8.  在左側導覽中，按一下 [API 管理員]，然後按一下 [認證]。
9.  按一下位於頂端的 [OAuth 同意畫面]  索引標籤。

    ![Google+ 帳戶 - 設定 OAuth 同意畫面](media/active-directory-b2c-custom-setup-goog-idp/goog-add-cred.png)

10.  選取或指定有效的**電子郵件地址**、提供**產品名稱**，然後按一下 [儲存]。

    ![Google+ - 應用程式認證](media/active-directory-b2c-custom-setup-goog-idp/goog-consent-screen.png)

11.  按一下 [新增認證]，然後選擇 [OAuth 用戶端識別碼]。

    ![Google+ - 建立新的應用程式認證](media/active-directory-b2c-custom-setup-goog-idp/goog-add-oauth2-client-id.png)

12.  在 [應用程式類型] 下方，選取 [Web 應用程式]。

    ![Google+ - 選取應用程式類型](media/active-directory-b2c-custom-setup-goog-idp/goog-web-app.png)

13.  提供應用程式的**名稱**，在 [授權 JavaScript 來源] 欄位中輸入 `https://{tenant}.b2clogin.com`，接著在 [授權重新導向 URI] 欄位中輸入 `https://{tenant}.b2clogin.com/te/{tenant}.onmicrosoft.com/oauth2/authresp`。 使用您的租用戶名稱 (例如 contosob2c) 來取代 **{tenant}**。 **{tenant}** 值會區分大小寫。 按一下頁面底部的 [新增] 。

    ![Google+ - 提供授權的 JavaScript 來源及重新導向 URI](media/active-directory-b2c-custom-setup-goog-idp/goog-create-client-id.png)

14.  複製 [用戶端識別碼] 和 [用戶端祕密] 的值。 您必須使用這兩個值，將 Google+ 設為租用戶中的識別提供者。 **用戶端密碼** 是重要的安全性認證。

    ![Google+ - 複製用戶端識別碼和用戶端祕密](media/active-directory-b2c-custom-setup-goog-idp/goog-client-secret.png)

## <a name="add-the-google-account-application-key-to-azure-ad-b2c"></a>將 Google+ 帳戶應用程式金鑰新增至 Azure AD B2C
具有 Google+ 帳戶的同盟需要 Google+ 帳戶的用戶端祕密，才能代表應用程式信任 Azure AD B2C。 您需要在 Azure AD B2C 租用戶中儲存您的 Google+ 應用程式祕密：  

1.  移至您的 Azure AD B2C 租用戶，並選取 [B2C 設定] > [身分識別體驗架構]
2.  選取**原則金鑰**以檢視您的租用戶中可用的金鑰。
3.  按一下 [+新增]。
4.  針對 [選項] 使用 [手動]。
5.  針對 [名稱] 使用 `GoogleSecret`。  
    可能會自動加入前置詞 `B2C_1A_`。
6.  在 [祕密] 方塊中，從您在上方複製的 [Google 開發人員主控台](https://console.developers.google.com/)輸入 Google 應用程式祕密。
7.  針對 [金鑰使用方法] 使用 [簽章]。
8.  按一下 [建立] 
9.  確認您已建立金鑰 `B2C_1A_GoogleSecret`。

## <a name="add-a-claims-provider-in-your-extension-policy"></a>在擴充原則中新增宣告提供者

如果需要讓使用者使用 Google+ 帳戶登入，您必須將 Google+ 帳戶定義為宣告提供者。 換句話說，您必須指定要與 Azure AD B2C 通訊的端點。 此端點會提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。

定義 Google+ 帳戶作為宣告提供者，方法是在擴充原則檔案中新增 `<ClaimsProvider>` 節點：

1.  從您的工作目錄中開啟擴充原則檔案 (TrustFrameworkExtensions.xml)。 如果您需要 XML 編輯器，請[試用 Visual Studio 程式碼](https://code.visualstudio.com/download)，這是一個輕巧的跨平台編輯器。
2.  尋找 `<ClaimsProviders>` 區段
3.  在 `ClaimsProviders`元素下新增下列 XML 程式碼片段，並在儲存檔案之前將 `client_id` 的值取代為 Google+ 帳戶應用程式用戶端識別碼。  

```xml
<ClaimsProvider>
    <Domain>google.com</Domain>
    <DisplayName>Google</DisplayName>
    <TechnicalProfiles>
    <TechnicalProfile Id="Google-OAUTH">
        <DisplayName>Google</DisplayName>
        <Protocol Name="OAuth2" />
        <Metadata>
        <Item Key="ProviderName">google</Item>
        <Item Key="authorization_endpoint">https://accounts.google.com/o/oauth2/auth</Item>
        <Item Key="AccessTokenEndpoint">https://accounts.google.com/o/oauth2/token</Item>
        <Item Key="ClaimsEndpoint">https://www.googleapis.com/oauth2/v1/userinfo</Item>
        <Item Key="scope">email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">0</Item>
        <Item Key="client_id">Your Google+ application ID</Item>
        </Metadata>
        <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_GoogleSecret" />
        </CryptographicKeys>
        <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="id" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="google.com" />
        <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
        </OutputClaims>
        <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
        <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
        <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
        <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
        </OutputClaimsTransformations>
        <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        <ErrorHandlers>
        <ErrorHandler>
            <ErrorResponseFormat>json</ErrorResponseFormat>
            <ResponseMatch>$[?(@@.error == 'invalid_grant')]</ResponseMatch>
            <Action>Reauthenticate</Action>
            <!--In case of authorization code used error, we don't want the user to select his account again.-->
            <!--AdditionalRequestParameters Key="prompt">select_account</AdditionalRequestParameters-->
        </ErrorHandler>
        </ErrorHandlers>
    </TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="register-the-google-account-claims-provider-to-sign-up-or-sign-in-user-journey"></a>向註冊或登入使用者旅程圖註冊 Google+ 帳戶宣告提供者

已設定識別提供者。  不過，無法在任何註冊/登入畫面中使用。 將 Google+ 帳戶識別提供者新增至您的使用者 `SignUpOrSignIn` 使用者旅程圖。 若要使用，我們建立了重複的現有範本使用者旅程圖。  然後我們會新增 Google+ 帳戶識別提供者：

>[!NOTE]
>
>如果您從原則的基底檔案將 `<UserJourneys>` 元素複製到擴充檔案 (TrustFrameworkExtensions.xml)，就可以跳過本節。

1.  開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。
2.  尋找 `<UserJourneys>` 元素，並複製 `<UserJourneys>` 節點的完整內容。
3.  開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，並尋找 `<UserJourneys>` 元素。 如果此元素不存在，請新增。
4.  貼上您複製作為 `<UserJourneys>` 元素之子元素的 `<UserJourney>` 節點的完整內容。

### <a name="display-the-button"></a>顯示按鈕
`<ClaimsProviderSelections>` 元素會定義宣告提供者選擇的選項清單和它們的順序。  `<ClaimsProviderSelection>` 元素類似於註冊/登入頁面上的識別提供者按鈕。 如果您新增 Google+ 帳戶的 `<ClaimsProviderSelection>` 元素，當使用者登陸頁面時，會出現新的按鈕。 若要新增此元素：

1.  在您複製的使用者旅程圖中，尋找包含 `Id="SignUpOrSignIn"` 的 `<UserJourney>` 節點。
2.  找出包含 `Order="1"` 的 `<OrchestrationStep>` 節點
3.  在 `<ClaimsProviderSelections>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsProviderSelection TargetClaimsExchangeId="GoogleExchange" />
```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作
現在已備妥按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Google+ 帳戶通訊以接收權杖。

1.  尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
2.  在 `<ClaimsExchanges>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
```

>[!NOTE]
>
> * 請確定 `Id` 具有與上一節中之 `TargetClaimsExchangeId` 相同的值
> * 請確定 `TechnicalProfileReferenceId` 識別碼設定為您稍早建立之技術設定檔的識別碼 (Google-OAUTH)。

## <a name="upload-the-policy-to-your-tenant"></a>將原則上傳至您的租用戶
1.  在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C] 刀鋒視窗。
2.  選取 [識別體驗架構]。
3.  開啟 [所有原則] 刀鋒視窗。
4.  選取 [上傳原則]。
5.  勾選 [覆寫已存在的原則] 方塊。
6.  **上傳** TrustFrameworkExtensions.xml，並確定它通過驗證

## <a name="test-the-custom-policy-by-using-run-now"></a>使用 [立即執行] 測試自訂原則
1.  開啟 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。

    >[!NOTE]
    >
    >    [立即執行] 需要在租用戶上至少預先註冊一個應用程式。 
    >    若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。


2.  開啟 **B2C_1A_signup_signin**，此為您上傳的信賴憑證者 (RP) 自訂原則。 選取 [立即執行]。
3.  您應該能夠使用 Google+ 帳戶進行登入。

## <a name="optional-register-the-google-account-claims-provider-to-profile-edit-user-journey"></a>[選用] 向設定檔編輯使用者旅程圖註冊 Google+ 帳戶宣告提供者
您也需要將 Google+ 帳戶識別提供者新增至您的使用者 `ProfileEdit` 使用者旅程圖。 若要使用，我們會重複最後兩個步驟：

### <a name="display-the-button"></a>顯示按鈕
1.  開啟原則的擴充檔案 (例如，TrustFrameworkExtensions.xml)。
2.  在您複製的使用者旅程圖中，尋找包含 `Id="ProfileEdit"` 的 `<UserJourney>` 節點。
3.  找出包含 `Order="1"` 的 `<OrchestrationStep>` 節點
4.  在 `<ClaimsProviderSelections>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsProviderSelection TargetClaimsExchangeId="GoogleExchange" />
```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作
1.  尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
2.  在 `<ClaimsExchanges>` 節點下，新增下列 XML 程式碼片段：

```xml
<ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
```

### <a name="upload-the-policy-to-your-tenant"></a>將原則上傳至您的租用戶
1.  在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C] 刀鋒視窗。
2.  選取 [識別體驗架構]。
3.  開啟 [所有原則] 刀鋒視窗。
4.  選取 [上傳原則]。
5.  勾選 [覆寫已存在的原則] 方塊。
6.  **上傳** TrustFrameworkExtensions.xml，並確定它通過驗證。

### <a name="test-the-custom-profile-edit-policy-by-using-run-now"></a>使用 [立即執行] 來測試自訂設定檔編輯原則

1.  開啟 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。
2.  開啟 **B2C_1A_ProfileEdit**，此為您上傳的信賴憑證者 (RP) 自訂原則。 選取 [立即執行]。
3.  您應該能夠使用 Google+ 帳戶進行登入。

## <a name="download-the-complete-policy-files"></a>下載完整的原則檔案
選擇性：建議您在使用自訂原則逐步解說完成使用者入門後，使用自己的自訂原則檔案來建置您的情節，而非使用這些範例檔案。  [參考的範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-setup-goog-app)
