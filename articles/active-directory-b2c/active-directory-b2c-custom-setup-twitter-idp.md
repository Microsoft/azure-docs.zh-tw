---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 Twitter 作為 OAuth1 識別提供者 | Microsoft Docs
description: 透過 OAuth1 通訊協定使用 Twitter 作為識別提供者。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/23/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 28679ef07c2625908f7b08f808ff49c48ddb625b
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43339863"
---
# <a name="azure-active-directory-b2c-add-twitter-as-an-oauth1-identity-provider-by-using-custom-policies"></a>Azure Active Directory B2C：使用自訂原則新增 Twitter 作為 OAuth1 識別提供者
[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文將說明如何使用[自訂原則](active-directory-b2c-overview-custom.md)，讓 Twitter 帳戶的使用者登入。

## <a name="prerequisites"></a>必要條件
完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

## <a name="step-1-create-a-twitter-account-application"></a>步驟 1：建立 Twitter 帳戶應用程式
若要在 Azure Active Directory B2C (Azure AD B2C) 中使用 Twitter 作為識別提供者，您必須建立 Twitter 應用程式，並對其提供正確參數。 您可以前往 [Twitter 註冊頁面](https://twitter.com/signup)，註冊 Twitter 應用程式。

1. 前往 [Twitter Developers](https://apps.twitter.com/) 網站，使用您的 Twitter 帳戶認證登入，然後選取 [建立新的應用程式]。

    ![Twitter 帳戶 - 建立新的應用程式](media/active-directory-b2c-custom-setup-twitter-idp/adb2c-ief-setup-twitter-idp-new-app1.png)

2. 在 [建立應用程式] 視窗中，請執行下列動作：
 
    a. 針對新的應用程式輸入 [名稱]和 [說明]。 

    b. 在 [網站] 方塊中，貼上 **https://{tenant}.b2clogin.com**。 其中 **{tenant}** 是您的租用戶名稱 (例如 https://contosob2c.b2clogin.com))。

    c. 4. 在 [回呼 URL] 中輸入 `https://{tenant}.b2clogin.com/te/{tenant}.onmicrosoft.com/{policyId}/oauth1/authresp`。 務必使用租用戶名稱 (例如 contosob2c) 來取代 **{tenant}**，並使用原則識別碼 (例如 b2c_1_policy) 來取代 **{policyId}**。  **回呼 URL 必須是全部小寫。** 您應該為所有使用 Twitter 登入的原則新增回呼 URL。 如果要在應用程式中使用，請務必使用 `b2clogin.com` 而不是 ` login.microsoftonline.com`。

    d. 在頁面底部，請閱讀並接受條款，然後選取 [建立 Twitter 應用程式] 。

    ![Twitter 帳戶 - 新增應用程式](media/active-directory-b2c-custom-setup-twitter-idp/adb2c-ief-setup-twitter-idp-new-app2.png)

3. 在 [B2C 示範] 視窗中，選取 [設定]、選取 [允許此應用程式用來以 Twitter 登入] 核取方塊，然後選取 [更新設定]。

4. 選取 [金鑰和存取權杖]，並記下 [取用者密碼 (API 密碼)] 和 [取用者密碼 (API 密碼)] 值。

    ![Twitter 帳戶 - 設定應用程式屬性](media/active-directory-b2c-custom-setup-twitter-idp/adb2c-ief-setup-twitter-idp-new-app3.png)

    >[!NOTE]
    >消費者密碼是重要的安全性認證。 請勿將此密碼告訴任何人或隨應用程式一起散發。

## <a name="step-2-add-your-twitter-account-application-key-to-azure-ad-b2c"></a>步驟 2：將 Twitter 帳戶應用程式金鑰新增至 Azure AD B2C
具有 Twitter 帳戶的同盟需要 Twitter 帳戶的取用者密碼，才能代表應用程式信任 Azure AD B2C。 若要將 Twitter 應用程式的取用者密碼儲存在 Azure AD B2C 租用戶中，請執行下列動作： 

1. 在您的 Azure AD B2C 租用戶中，選取 [B2C 設定] > [識別體驗架構]。

2. 若要檢視租用戶中可用的金鑰，請選取 [原則金鑰]。

3. 選取 [新增] 。

4. 在 [選項] 方塊中，選取 [手動]。

5. 在 [名稱] 方塊中，選取 [TwitterSecret]。  
    可能會自動新增前置詞 *B2C_1A_*。

6. 在 [密碼] 方塊中，輸入來自[應用程式註冊入口網站](https://apps.dev.microsoft.com)的 Microsoft 應用程式密碼。

7. 針對 [金鑰使用方法] 使用 [加密]。

8. 選取 [建立] 。

9. 確認您已建立 `B2C_1A_TwitterSecret` 金鑰。

## <a name="step-3-add-a-claims-provider-in-your-extension-policy"></a>步驟 3：在擴充原則中新增宣告提供者

如果需要讓使用者使用 Twitter 帳戶登入，您必須將 Twitter 定義為宣告提供者。 換句話說，您必須指定要與 Azure AD B2C 通訊的端點。 此端點提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。

定義 Twitter 作為宣告提供者，方法是在擴充原則檔案中新增 `<ClaimsProvider>` 節點：

1. 在您的工作目錄中，開啟 *TrustFrameworkExtensions.xml* 擴充原則檔案。 

2. 搜尋 `<ClaimsProviders>` 區段。

3. 在 `<ClaimsProviders>` 節點中，新增下列 XML 程式碼片段：  

    ```xml
    <ClaimsProvider>
        <Domain>twitter.com</Domain>
        <DisplayName>Twitter</DisplayName>
        <TechnicalProfiles>
        <TechnicalProfile Id="Twitter-OAUTH1">
            <DisplayName>Twitter</DisplayName>
            <Protocol Name="OAuth1" />
            <Metadata>
            <Item Key="ProviderName">Twitter</Item>
            <Item Key="authorization_endpoint">https://api.twitter.com/oauth/authenticate</Item>
            <Item Key="access_token_endpoint">https://api.twitter.com/oauth/access_token</Item>
            <Item Key="request_token_endpoint">https://api.twitter.com/oauth/request_token</Item>
            <Item Key="ClaimsEndpoint">https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true</Item>
            <Item Key="ClaimsResponseFormat">json</Item>
            <Item Key="client_id">Your Twitter application consumer key</Item>
            </Metadata>
            <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_TwitterSecret" />
            </CryptographicKeys>
            <InputClaims />
            <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="user_id" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="screen_name" />
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="twitter.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
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

4. 使用您的 Twitter 帳戶應用程式取用者金鑰來取代 *client_id* 值。

5. 儲存檔案。

## <a name="step-4-register-the-twitter-account-claims-provider-to-your-sign-up-or-sign-in-user-journey"></a>步驟 4：向註冊或登入使用者旅程圖註冊 Twitter 帳戶宣告提供者
您已設定識別提供者。 不過，尚無法在任何註冊或登入視窗中使用。 您現在必須將 Twitter 帳戶識別提供者新增至您的使用者 `SignUpOrSignIn` 使用者旅程圖。

### <a name="step-41-make-a-copy-of-the-user-journey"></a>步驟 4.1：建立使用者旅程圖的副本
若要讓使用者旅程圖可供使用，您可以建立現有使用者旅程圖範本的複本，然後新增 Twitter 識別提供者：

>[!NOTE]
>如果您從原則的基底檔案將 `<UserJourneys>` 元素複製到 *TrustFrameworkExtensions.xml* 擴充檔案，就可以跳至下一節。

1. 開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。

2. 搜尋 `<UserJourneys>` 元素，選取 `<UserJourney>` 節點的整個內容，然後選取 [剪下]，將選取的文字移至剪貼簿。

3. 開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，然後搜尋 `<UserJourneys>` 元素。 如果此元素不存在，請新增。

4. 將 `<UserJourney>` 節點的整個內容 (您已在步驟 2 將這些內容移至剪貼簿) 貼入 `<UserJourneys>` 元素中。

### <a name="step-42-display-the-button"></a>步驟 4.2：顯示「按鈕」
`<ClaimsProviderSelections>` 元素會定義宣告提供者選擇的選項清單和它們的順序。 `<ClaimsProviderSelection>` 節點類似於註冊或登入頁面上的識別提供者按鈕。 如果您新增 Twitter 帳戶的 `<ClaimsProviderSelection>` 節點，當使用者登陸頁面時，會顯示新的按鈕。 若要新增此項目，請執行下列動作：

1. 在您複製的使用者旅程圖中，搜尋包含 `Id="SignUpOrSignIn"` 的 `<UserJourney>` 節點。

2. 找出包含 `Order="1"` 的 `<OrchestrationStep>` 節點。

3. 在 `<ClaimsProviderSelections>` 元素中，新增下列 XML 程式碼片段：

    ```xml
    <ClaimsProviderSelection TargetClaimsExchangeId="TwitterExchange" />
    ```

### <a name="step-43-link-the-button-to-an-action"></a>步驟 4.3：將按鈕連結至動作
現在已備妥按鈕，您必須將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Twitter 帳戶通訊以接收權杖。 透過連結 Twitter 帳戶宣告提供者的技術設定檔，將按鈕連結至動作：

1. 搜尋 `<OrchestrationStep>` 節點，其中包含 `<UserJourney>` 節點中的 `Order="2"`。
2. 在 `<ClaimsExchanges>` 元素中，新增下列 XML 程式碼片段：

    ```xml
    <ClaimsExchange Id="TwitterExchange" TechnicalProfileReferenceId="Twitter-OAUTH1" />
    ```

    >[!NOTE]
    >* 確定 `Id` 具有與上一節中之 `TargetClaimsExchangeId` 相同的值。
    >* 確定 `TechnicalProfileReferenceId` 識別碼設為您稍早建立的技術設定檔 (Twitter-OAUTH1)。

## <a name="step-5-upload-the-policy-to-your-tenant"></a>步驟 5：將原則上傳至您的租用戶
1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後選取 [Azure AD B2C]。

2. 選取 [識別體驗架構]。

3. 選取 [所有原則]。

4. 選取 [上傳原則]。

5. 選取 [覆寫已存在的原則] 核取方塊。

6. 上傳 *TrustFrameworkBase.xml* 和 *TrustFrameworkExtensions.xml*檔案，並確定它們通過驗證。

## <a name="step-6-test-the-custom-policy-by-using-run-now"></a>步驟 6：使用 [立即執行] 測試自訂原則

1. 選取 [Azure AD B2C 設定]，然後選取 [識別體驗架構]。

    >[!NOTE]
    >[立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

2. 開啟 **B2C_1A_signup_signin** (此為您上傳的信賴憑證者 (RP) 自訂原則)，然後選取 [立即執行]。  
    您現在應該能夠使用 Twitter 帳戶進行登入。

## <a name="step-7-optional-register-the-twitter-account-claims-provider-to-the-profile-edit-user-journey"></a>步驟 7：(選用) 向設定檔編輯使用者旅程圖註冊 Twitter 帳戶宣告提供者
您可能也想要將 Twitter 帳戶識別提供者新增至 `ProfileEdit` 使用者旅程圖。 若要讓使用者旅程圖可供使用，請重複「步驟 4」。 此時，選取包含 `Id="ProfileEdit"` 的 `<UserJourney>` 節點。 儲存、上傳，並測試您的原則。


## <a name="optional-download-the-complete-policy-files"></a>(選用) 下載完整的原則檔案
在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)逐步解說之後，建議您使用自己的自訂原則檔案來建置您的案例。 我們已提供[範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-setup-twitter-app)，供您參考。
