---
title: 在 Azure Active Directory B2C 中使用自訂原則新增多租用戶 Azure AD 識別提供者 | Microsoft Docs
description: 使用自訂原則新增多租用戶 Azure AD 識別提供者 - Azure Active Directory B2C。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/14/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 68eab85c7f67ad3af18c6066c29e1250e1be3d23
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344401"
---
# <a name="azure-active-directory-b2c-allow-users-to-sign-in-to-a-multi-tenant-azure-ad-identity-provider-using-custom-policies"></a>Azure Active Directory B2C：使用自訂原則允許使用者登入多租用戶 Azure AD 識別提供者

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文會說明如何透過使用[自訂原則](active-directory-b2c-overview-custom.md)，讓使用 Azure Active Directory (Azure AD) 多租用戶端點的使用者能夠登入。 這可讓使用者從多個 Azure AD 租用戶登入 Azure AD B2C，而不需設定每個租用戶的技術提供者。 不過，所有這些租用戶中的來賓成員並**不能**登入。 因此，您必須[個別設定每個租用戶](active-directory-b2c-setup-aad-custom.md)。

>[!NOTE]
> 我們在下列指示中使用 "contoso.com" 作為組織的 Azure AD 租用戶，以及使用 "fabrikamb2c.onmicrosoft.com" 作為 Azure AD B2C 租用戶。

## <a name="prerequisites"></a>必要條件

完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

這些步驟包括：
     
1. 建立 Azure Active Directory B2C (Azure AD B2C) 租用戶。
1. 建立 Azure AD B2C 應用程式。    
1. 註冊兩個原則引擎應用程式。  
1. 設定金鑰。 
1. 設定入門套件。

## <a name="step-1-create-a-multi-tenant-azure-ad-app"></a>步驟 1. 建立多租用戶 Azure AD 應用程式

為了讓使用多租用戶 Azure AD 端點的使用者能夠登入，您必須在其中一個 Azure AD 租用戶中註冊多租用戶應用程式。 在本文中，我們會說明如何在 Azure AD B2C 租用戶中建立多租用戶 Azure AD 應用程式。 然後，透過使用該多租用戶 Azure AD 應用程式讓使用者能夠登入。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 在頂端列上，選取您的帳戶。 從 [目錄] 清單選擇要註冊 Azure AD 應用程式的 Azure AD B2C 租用戶 (fabrikamb2c.onmicrosoft.com)。
1. 選取左側窗格中的 [更多服務]，然後搜尋「應用程式註冊」。
1. 選取 [新增應用程式註冊]。
1. 輸入應用程式的名稱 (例如，`Azure AD B2C App`)。
1. 選取 [Web 應用程式/API] 作為應用程式類型。
1. 針對 [登入 URL]，輸入下列 URL，其中 `yourtenant` 由 Azure AD B2C 租用戶的名稱 (`fabrikamb2c.onmicrosoft.com`) 取代：

    >[!NOTE]
    >在**登入 URL** 中，「yourtenant」的值必須全部小寫。

    ```
    https://yourtenant.b2clogin.com/te/yourtenant.onmicrosoft.com/oauth2/authresp
    ```

1. 儲存應用程式識別碼。
1. 選取新建立的應用程式。
1. 在 [設定] 刀鋒視窗底下，選取 [屬性]。
1. 將 [多重租用戶] 設定為 [是]。
1. 在 [設定] 刀鋒視窗下，選取 [金鑰]。
1. 建立新的金鑰並且儲存。 您在下一節中的步驟將會用到它。

## <a name="step-2-add-the-azure-ad-key-to-azure-ad-b2c"></a>步驟 2. 將 Azure AD 金鑰新增至 Azure AD B2C

您需要在 Azure AD B2C 設定中註冊應用程式金鑰。 作法：

1. 移至 Azure AD B2C 的 [設定] 功能表
1. 按一下 [身分識別體驗架構] > [原則金鑰]。
1. 選取 [+新增]。
1. 選取或輸入這些選項：
   * 選取 [手動]。
   * 針對 [名稱]，選擇與您的 Azure AD 租用戶名稱相符的名稱 (例如，`AADAppSecret`)。  金鑰名稱前面會自動新增前置詞 `B2C_1A_`。
   * 在 [祕密] 方塊中貼上您的應用程式金鑰。
   * 選取 [簽章]。
1. 選取 [建立] 。
1. 確認您已建立金鑰 `B2C_1A_AADAppSecret`。

## <a name="step-3-add-a-claims-provider-in-your-base-policy"></a>步驟 3. 在基本原則中新增宣告提供者

如果想要讓使用者使用 Azure AD 登入，您需要將 Azure AD 定義為宣告提供者。 換句話說，您需要指定將與 Azure AD B2C 通訊的端點。 此端點將提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。 

您可以藉由將 Azure AD 新增至原則之擴充檔案中的 `<ClaimsProvider>` 節點，以將 Azure AD 定義為宣告提供者：

1. 從您的工作目錄中開啟擴充檔案 (TrustFrameworkExtensions.xml)。
1. 找出區段 `<ClaimsProviders>`。 如果不存在，請在根節點下新增。
1. 新增 `<ClaimsProvider>` 節點，如下所示︰

```XML
<ClaimsProvider>
  <Domain>commonaad</Domain>
  <DisplayName>Common AAD</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Common-AAD">
      <DisplayName>Multi-Tenant AAD</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <!-- Update the Client ID below to the Application ID -->
        <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
        <Item Key="UsePolicyInRedirectUri">0</Item>
        <Item Key="METADATA">https://login.microsoftonline.com/common/.well-known/openid-configuration</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid</Item>
        <Item Key="response_mode">form_post</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="DiscoverMetadataByTokenIssuer">true</Item>
        
        <!-- The key below allows you to specify each of the Azure AD tenants that can be used to sign in. Update the GUIDs below for each tenant. -->
        <Item Key="ValidTokenIssuerPrefixes">https://sts.windows.net/00000000-0000-0000-0000-000000000000,https://sts.windows.net/11111111-1111-1111-1111-111111111111</Item>

        <!-- The commented key below specifies that users from any tenant can sign-in. Uncomment if you would like anyone with an Azure AD account to be able to sign in. -->
        <!-- <Item Key="ValidTokenIssuerPrefixes">https://sts.windows.net/</Item> -->

      </Metadata>
      <CryptographicKeys>
      <!-- Make sure to update the reference ID of the client secret below you just created (B2C_1A_AADAppSecret) -->
        <Key Id="client_secret" StorageReferenceId="B2C_1A_AADAppSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
        <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
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

1. 在 `<ClaimsProvider>` 節點下，將 `<Domain>` 的值更新為可以與其他識別提供者有所區別的唯一值。
1. 在 `<TechnicalProfile>` 節點下，更新 `<DisplayName>` 的值。 這個值會顯示在登入畫面的登入按鈕上。
1. 更新 `<Description>` 的值。
1. 將 `<Item Key="client_id">` 設定為註冊 Azure AD 多租用戶應用程式時所得到的應用程式識別碼。

### <a name="step-31-restrict-access-to-a-specific-list-of-azure-ad-tenants"></a>步驟 3.1 限制只能存取特定的 Azure AD 租用戶清單

> [!NOTE]
> 若使用 `https://sts.windows.net` 作為 **ValidTokenIssuerPrefixes** 的值，將可允許所有 Azure AD 使用者登入您的應用程式。

您必須更新有效權杖簽發者清單，並限制只能存取由使用者可登入的 Azure AD 租用戶所構成的特定清單。 若要取得這些值，您必須查看您想要讓使用者從中登入的每個特定 Azure AD 租用戶中繼資料。 資料的格式看起來如下：`https://login.windows.net/yourAzureADtenant/.well-known/openid-configuration`，其中 `yourAzureADtenant` 是 Azure AD 租用戶名稱 (contoso.com 或任何其他 Azure AD 租用戶)。
1. 開啟瀏覽器，然後移至中繼資料 URL。
1. 在瀏覽器中，尋找 'issuer' 物件並複製其值。 如下所示：`https://sts.windows.net/{tenantId}/`。
1. 貼上 `ValidTokenIssuerPrefixes` 索引鍵的值。 您可以使用逗號分隔來新增多個值。 上面的 XML 範例中有將這一點的範例標記為註解。

## <a name="step-4-register-the-azure-ad-account-claims-provider"></a>步驟 4. 註冊 Azure AD 帳戶宣告提供者

### <a name="step-41-make-a-copy-of-the-user-journey"></a>步驟 4.1 建立使用者旅程圖的副本

您現在需要將 Azure AD 識別提供者新增至其中一個使用者旅程圖。 目前，識別提供者已設定，但還未出現在任何註冊/登入畫面中。

為了這樣做，我們將建立現有範本使用者旅程圖的複本，然後將它修改成也有 Azure AD 識別提供者：

1. 開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。
1. 尋找 `<UserJourneys>` 元素，並複製含有 `Id="SignUpOrSignIn"` 的整個 `<UserJourney>`。
1. 開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，並尋找 `<UserJourneys>` 元素。 如果此元素不存在，請新增。
1. 貼上您複製的整個 `<UserJourney>` 節點作為 `<UserJourneys>` 元素的子元素。
1. 重新命名新使用者旅程圖的識別碼 (例如，重新命名為 `SignUpOrSignUsingAzureAD`)。 

### <a name="step-42-display-the-button"></a>步驟 4.2 顯示「按鈕」

`<ClaimsProviderSelection>` 元素類似於註冊/登入畫面上的識別提供者按鈕。 如果您新增 Azure AD 的 `<ClaimsProviderSelection>` 元素，當使用者登陸頁面時，會出現新的按鈕。 若要新增此元素：

1. 在您建立的使用者旅程圖中，尋找包含 `Order="1"` 的 `<OrchestrationStep>` 節點。
1. 新增下列內容：

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="AzureADExchange" />
    ```

1. 將 `TargetClaimsExchangeId` 設定為適當的值。 建議您與其他人一樣，遵循相同的慣例：\[ClaimProviderName\]Exchange。

### <a name="step-43-link-the-button-to-an-action"></a>步驟 4.3 將按鈕連結至動作

現在已備妥按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Azure AD 通訊以接收權杖。 藉由連結 Azure AD 宣告提供者的技術設定檔，將按鈕連結至動作：

1. 尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
1. 新增下列內容：

    ```XML
    <ClaimsExchange Id="AzureADExchange" TechnicalProfileReferenceId="Common-AAD" />
    ```

1. 將 `Id` 更新為與上一節中之 `TargetClaimsExchangeId` 相同的值。
1. 將 `TechnicalProfileReferenceId` 更新為您稍早建立的技術設定檔識別碼 (Common-AAD)。

## <a name="step-5-create-a-new-rp-policy"></a>步驟 5：建立新的 RP 原則

您現在需要更新信賴憑證者 (RP) 檔案，此檔案將起始您剛才建立的使用者旅程圖：
 
1. 將 SignUpOrSignIn.xml 複製到您的工作目錄，並重新命名 (例如，將它重新命名為 SignUpOrSignInWithAAD.xml)。  
1. 開啟新檔案，將 `<TrustFrameworkPolicy>` 的 `PolicyId` 屬性更新成唯一值 (例如，SignUpOrSignInWithAAD)。 這會成為您的原則名稱 (例如，B2C\_1A\_SignUpOrSignInWithAAD)。 
1. 修改 `<DefaultUserJourney>` 中的 `ReferenceId` 屬性，以符合您建立之新使用者旅程圖的識別碼 (SignUpOrSignUsingAzureAD)。 
1. 儲存變更並上傳檔案。 

## <a name="step-6-upload-the-policy-to-your-tenant"></a>步驟 6：將原則上傳至您的租用戶

1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後選取 [Azure AD B2C]。
1. 選取 [識別體驗架構]。
1. 選取 [所有原則]。
1. 選取 [上傳原則]。
1. 選取 [覆寫已存在的原則] 核取方塊。
1. 上傳 `TrustFrameworkExtensions.xml` 檔案和 RP 檔案 (例如 `SignUpOrSignInWithAAD.xml`)，並確保這兩個檔案都通過驗證。

## <a name="step-7-test-the-custom-policy-by-using-run-now"></a>步驟 7：使用 [立即執行] 測試自訂原則

1. 選取 [Azure AD B2C 設定]，然後選取 [識別體驗架構]。
    > [!NOTE]
    > [立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

1. 開啟您上傳的信賴憑證者 (RP) 自訂原則 (*B2C\_1A\_SignUpOrSignInWithAAD*)，然後選取 [立即執行]。
1. 您現在應該能夠使用 Azure AD 帳戶進行登入。

## <a name="optional-step-8-register-the-azure-ad-account-claims-provider-to-the-profile-edit-user-journey"></a>(選擇性) 步驟 8：向設定檔編輯使用者旅程圖註冊 Azure AD 帳戶宣告提供者

您也可以將 Azure AD 帳戶識別提供者新增至 `ProfileEdit` 使用者旅程圖。 若要讓使用者旅程圖可供使用，請重複步驟 4 到 6。 此時，選取包含 `Id="ProfileEdit"` 的 `<UserJourney>` 節點。 儲存、上傳，並測試您的原則。

## <a name="troubleshooting"></a>疑難排解

若要診斷問題，請參閱[疑難排解](active-directory-b2c-troubleshoot-custom.md)。

## <a name="next-steps"></a>後續步驟

請將意見反應寄到 [AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com)。
