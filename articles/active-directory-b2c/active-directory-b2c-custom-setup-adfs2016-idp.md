---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 ADFS 作為 SAML 識別提供者 | Microsoft Docs
description: 在 Azure Active Directory B2C 中使用 SAML 通訊協定和自訂原則來設定 ADFS 2016
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/31/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 2c2e6861fda42a9e8c1aabcba303bfede47ac3c1
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669221"
---
# <a name="add-adfs-as-a-saml-identity-provider-using-custom-policies-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中使用自訂原則將 ADFS 新增為 SAML 識別提供者

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文將示範如何在 Azure Active Directory (Azure AD) B2C 中使用[自訂原則](active-directory-b2c-overview-custom.md)來允許以 ADFS 使用者帳戶登入。

## <a name="prerequisites"></a>必要條件

完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

## <a name="add-the-adfs-account-application-key-to-azure-ad-b2c"></a>將 ADFS 帳戶應用程式金鑰新增至 Azure AD B2C

具有 ADFS 帳戶的同盟需要帳戶的用戶端祕密，才能代表應用程式信任 Azure AD B2C。 您需要將 ADFS 憑證儲存在 Azure AD B2C 租用戶中。 

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 Azure 入口網站的右上角切換到您的 Azure AD B2C 租用戶，確定您使用的目錄包含該租用戶。 選取 [切換目錄]，然後選擇包含所建立租用戶的目錄。 本教學課程會使用 contoso 目錄，其中包含名為 contoso0522Tenant.onmicrosoft.com 的租用戶。

    ![切換目錄](./media/active-directory-b2c-custom-setup-adfs2016-idp/switch-directories.png)

3. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。 您現在應使用您的租用戶。
4. 在 [概觀] 頁面上，選取 [識別體驗架構]。
5. 選取 [原則金鑰] 以檢視您租用戶中可用的金鑰，然後按一下 [新增]。
6. 選擇 [上傳] 選項。
7. 輸入 `ADFSSamlCert` 作為名稱。 可能會自動加入前置詞 `B2C_1A_`。
8. 瀏覽至包含私密金鑰的憑證 .pfx 檔案，並選取該檔案。 此憑證 (含私密金鑰) 應該與發給及用於 ADFS 信賴憑證者的是同一個。
9. 按一下 [建立]，並確認您已建立 `B2C_1A_ADFSSamlCert` 金鑰。

## <a name="add-a-claims-provider-in-your-extension-policy"></a>在擴充原則中新增宣告提供者

如果需要讓使用者使用 ADFS 帳戶登入，您必須將該帳戶定義為宣告提供者。 若要這麼做，您可以指定要與 Azure AD B2C 通訊的端點。 此端點會提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。

定義 ADFS 作為宣告提供者，方法是在擴充原則檔案中新增 **ClaimsProvider** 元素。

1. 在您的工作目錄中開啟 TrustFrameworkExtensions.xml 原則檔案。 如果您需要 XML 編輯器，請[試用 Visual Studio Code](https://code.visualstudio.com/download)，這是一個輕巧的跨平台編輯器。
2. 在 **ClaimsProviders** 元素底下新增下列 XML，然後將 **your-ADFS-domain** 取代為您的 ADFS 網域名稱，以及將 **identityProvider** 輸出宣告的值取代為您的 DNS (代表您網域的任意值)，然後儲存該檔案。 

    ```xml
    <ClaimsProvider>
      <Domain>contoso.com</Domain>
      <DisplayName>Contoso ADFS</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso-SAML2">
          <DisplayName>Contoso ADFS</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="RequestsSigned">false</Item>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="PartnerEntity">https://your-ADFS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_ADFSSamlCert"/>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_ADFSSamlCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="userPrincipalName" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
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

## <a name="register-the-claims-provider-for-sign-up-and-sign-in"></a>註冊宣告提供者來進行註冊和登入

若要讓 ADFS 帳戶識別提供者可在註冊和登入頁面中使用，您需要將其新增至您的 **SignUpOrSignIn** 使用者旅程圖。 

建立現有使用者旅程圖範本的複本，然後加以修改，使其包含 ADFS 識別提供者：

>[!NOTE]
>如果您先前已從原則的基底檔案將 **UserJourneys** 元素複製到擴充檔案 (TrustFrameworkExtensions.xml)，就可以跳過本節。

1. 開啟原則的基底檔案。 例如，TrustFrameworkBase.xml。
2. 複製整個 **UserJourneys** 元素的內容。
3. 開啟擴充檔案 (TrustFrameworkExtensions.xml) 並貼上整個 **UserJourneys** 元素的內容 (您從擴充檔案中複製的內容)。

### <a name="display-the-button"></a>顯示按鈕

**ClaimsProviderSelections** 元素會定義宣告提供者選項清單和其順序。  **ClaimsProviderSelection** 元素類似於註冊和登入頁面上的識別提供者按鈕。 如果您新增 ADFS 帳戶的 **ClaimsProviderSelection** 元素，則使用者會在查看頁面時看到新增的按鈕。 若要新增此元素：

1. 在您複製的使用者旅程圖中，從識別碼為 `SignUpOrSignIn` 的 **UserJourney** 元素中，找出 `Order="1"` 的 **OrchestrationStep**元素。
2. 在 **ClaimsProviderSelections** 元素下新增下列 **ClaimsProviderSelection** 元素：

    ```xml
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作

現在已備妥按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 ADFS 帳戶通訊以接收權杖。 透過連結 ADFS 帳戶宣告提供者的技術設定檔，將按鈕連結至動作：

1. 在 **UserJourney** 元素下尋找 `Order="2"` 的 **OrchestrationStep**。
2. 在 **ClaimsExchanges** 元素下方新增下列 **ClaimsExchange** 元素：

    ```xml
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
    ```

> [!NOTE]
> * 請確定 `Id` 的值與上一節中的 `TargetClaimsExchangeId` 相同。
> * 請確定 `TechnicalProfileReferenceId` 設定為您稍早建立之技術設定檔的識別碼 (Contoso-SAML2)。


## <a name="optional-register-the-claims-provider-for-profile-edit"></a>[選擇性] 註冊設定檔編輯的宣告提供者

您也可以將 ADFS 帳戶識別提供者新增至設定檔編輯使用者旅程圖。

### <a name="display-the-button"></a>顯示按鈕

1. 開啟原則的擴充檔案。 例如 TrustFrameworkExtensions.xml。
2. 在您複製的使用者旅程圖中，從識別碼為 `ProfileEdit` 的 **UserJourney** 元素中，找出 `Order="1"` 的 **OrchestrationStep** 元素。
3. 在 **ClaimsProviderSelections** 元素下新增下列 **ClaimsProviderSelection** 元素：

    ```xml
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作

1. 在 **UserJourney** 元素下尋找 `Order="2"` 的 **OrchestrationStep**。
2. 在 **ClaimsExchanges** 元素下方新增下列 **ClaimsExchange** 元素：

    ```xml
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
    ```

## <a name="upload-the-policy-to-your-tenant"></a>將原則上傳至您的租用戶

1. 在 Azure 入口網站中，選取 [所有原則]。
2. 選取 [上傳原則]。
3. 啟用 [覆寫現有的原則]。
4. 瀏覽並選取您的 TrustFrameworkExtensions.xml 原則檔案，然後選取 [上傳]。 請確定驗證已成功。


## <a name="configure-an-adfs-relying-party-trust"></a>設定 ADFS 信賴憑證者信任

若要在 Azure AD B2C 中使用 ADFS 作為識別提供者，您必須使用 Azure AD B2C SAML 中繼資料建立 ADFS 信賴憑證者信任。 下列範例顯示 Azure AD B2C 技術設定檔 SAML 中繼資料的 URL 位址：

```
https://login.microsoftonline.com/te/your-tenant/your-policy/samlp/metadata?idptp=your-technical-profile
```

取代下列值：

- 將 **your-tenant** 取代為您的租用戶名稱，例如 your-tenant.onmicrosoft.com。
- 將 **your-policy** 取代為您的原則名稱。 使用您用來設定 SAML 提供者技術設定檔的原則，或繼承自該原則的原則。
- 將 **your-technical-profile** 取代為 SAML 識別提供者技術設定檔的名稱。
 
開啟瀏覽器並瀏覽至此 URL。 請確定您輸入正確的 URL，而且您可以存取 XML 中繼資料檔案。

若要使用 ADFS 管理嵌入式管理單元來新增新的信賴憑證者信任並手動進行設定，請在同盟伺服器上執行下列程序。 必須至少具有本機電腦上的**管理員**或同等成員資格，才能完成此程序。 請在[本機與網域預設群組](http://go.microsoft.com/fwlink/?LinkId=83477)檢閱有關使用適當帳戶和群組成員資格的詳細資料。

1. 在 [伺服器管理員] 中選取 [工具]，然後選取 [ADFS 管理]。
2. 選取 [新增信賴憑證者信任] 。
3. 在 [歡迎] 頁面上選擇 [宣告感知]，然後按一下 [開始]。
4. 在 [選取資料來源] 頁面上，選取 [匯入線上或區域網路上發行的信賴憑證者相關資料]，提供您的 Azure AD B2C 中繼資料 URL，然後按 [下一步]。
5. 在 [指定顯示名稱] 頁面上輸入**顯示名稱**、在 [備忘稿] 下輸入此信賴憑證者信任的描述，然後按 [下一步]。
6. 在 [選擇存取控制原則] 頁面上選擇原則，然後按 [下一步]。
7. 在 [準備新增信任] 頁面上檢閱設定，然後按 [下一步] 以儲存您的信賴憑證者信任資訊。
8. 在 [完成] 頁面上按一下 [關閉]，此動作就會自動顯示 [編輯宣告規則] 對話方塊。
9. 選取 [新增規則]。  
10. 在 [宣告規則範本] 中，選取 [傳送 LDAP 屬性作為宣告]。
11. 提供**宣告規則名稱**。 針對 [屬性存放區]，選取 [選取 Active Directory] 來新增下列宣告，然後按一下 [完成] 和 [確定]。

    ![設定規則屬性](./media/active-directory-b2c-custom-setup-adfs2016-idp/aadb2c-ief-setup-adfs2016-idp-claims-3.png)

12.  根據不同憑證類型，您可能需要設定雜湊演算法。 在信賴憑證者信任 (B2C 示範) 屬性視窗上，選取 [進階] 索引標籤，然後將 [安全雜湊演算法] 變更為 `SHA-1` 或 `SHA-256`，並按一下 [確定]。  

### <a name="update-the-relying-party-metadata"></a>更新信賴憑證者中繼資料

若要變更 SAML 技術設定檔，您必須透過更新的中繼資料版本來更新 ADFS。 您不需要在建立信賴憑證者應用程式時更新中繼資料，但當您進行變更時，必須更新 ADFS 中的中繼資料。

1. 在 [伺服器管理員] 中選取 [工具]，然後選取 [ADFS 管理]。
2. 選取您建立的信賴憑證者信任，並選取 [從同盟中繼資料中更新]，然後按一下 [更新]。 

### <a name="test-the-policy-by-using-run-now"></a>使用 [立即執行] 測試原則

1.  開啟 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。
2.  開啟 **B2C_1A_ProfileEdit**，此為您上傳的信賴憑證者 (RP) 自訂原則。 選取 [立即執行]。 您應該能夠使用 ADFS 帳戶進行登入。

## <a name="download-the-complete-policy-files"></a>下載完整的原則檔案

選擇性：您可以在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)中的步驟之後，使用自己的自訂原則檔案來建置案例。 如需範例檔案，請參閱[僅供參考的原則範例檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-setup-adfs2016-app)。
