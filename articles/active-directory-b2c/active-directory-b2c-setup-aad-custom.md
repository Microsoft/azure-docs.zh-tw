---
title: 在 Azure Active Directory B2C 中使用自訂原則新增 Azure AD 提供者 | Microsoft Docs
description: 了解 Azure Active Directory B2C 自訂原則。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/15/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: af97a85b4d5d9c38f0e2bf8947482a0585fa6ee1
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338306"
---
# <a name="azure-active-directory-b2c-sign-in-by-using-azure-ad-accounts"></a>Azure Active Directory B2C︰使用 Azure AD 帳戶登入

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文將說明如何使用[自訂原則](active-directory-b2c-overview-custom.md)讓特定 Azure Active Directory (Azure AD) 組織的使用者登入。

## <a name="prerequisites"></a>必要條件

完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

這些步驟包括：

1. 建立 Azure Active Directory B2C (Azure AD B2C) 租用戶。
2. 建立 Azure AD B2C 應用程式。
3. 註冊兩個原則引擎應用程式。
4. 設定金鑰。
5. 設定入門套件。

## <a name="create-an-azure-ad-app"></a>建立 Azure AD 應用程式

若要讓特定 Azure AD 組織的使用者登入，您需要在組織 Azure AD 租用戶內註冊應用程式。

>[!NOTE]
> 我們在下列指示中使用 "contoso.com" 作為組織的 Azure AD 租用戶，以及使用 "fabrikamb2c.onmicrosoft.com" 作為 Azure AD B2C 租用戶。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頂端列上，選取您的帳戶。 從 [目錄] 清單中，選擇您要註冊應用程式的組織 Azure AD 租用戶 (contoso.com)。
3. 選取左側窗格中的 [更多服務]，然後搜尋「應用程式註冊」。
4. 選取 [新增應用程式註冊]。
5. 輸入應用程式的名稱 (例如，`Azure AD B2C App`)。
6. 選取 [Web 應用程式/API] 作為應用程式類型。
7. 針對 [登入 URL]，輸入下列 URL，其中 `yourtenant` 由 Azure AD B2C 租用戶的名稱 (`fabrikamb2c.onmicrosoft.com`) 取代：

    >[!NOTE]
    >在**登入 URL** 中，「yourtenant」的值必須全部小寫。

    ```
    https://yourtenant.b2clogin.com/te/yourtenant.onmicrosoft.com/oauth2/authresp
    ```

8. 儲存應用程式識別碼。
9. 選取新建立的應用程式。
10. 在 [設定] 刀鋒視窗下，選取 [金鑰]。
11. 輸入金鑰描述，選取持續時間，然後按一下 [儲存]。 隨即顯示金鑰值。 請複製金鑰值，下一節中的步驟將會使用它。

## <a name="add-the-azure-ad-key-to-azure-ad-b2c"></a>將 Azure AD 金鑰新增至 Azure AD B2C

您需要將 contoso.com 應用程式金鑰儲存在 Azure AD B2C 租用戶中。 作法：
1. 移至您的 Azure AD B2C 租用戶，並選取 [B2C 設定]  >  [身分識別體驗架構]  >  [原則金鑰]。
1. 選取 [+新增]。
1. 選取或輸入這些選項：
   * 選取 [手動]。
   * 針對 [名稱]，選擇與您的 Azure AD 租用戶名稱相符的名稱 (例如，`ContosoAppSecret`)。  金鑰名稱前面會自動新增前置詞 `B2C_1A_`。
   * 在 [祕密] 方塊中貼上您的應用程式金鑰。
   * 選取 [簽章]。
1. 選取 [建立] 。
1. 確認您已建立金鑰 `B2C_1A_ContosoAppSecret`。


## <a name="add-a-claims-provider-in-your-base-policy"></a>在基本原則中新增宣告提供者

如果想要讓使用者使用 Azure AD 登入，您需要將 Azure AD 定義為宣告提供者。 換句話說，您需要指定將與 Azure AD B2C 通訊的端點。 此端點將提供一組宣告，由 Azure AD B2C 用來確認特定使用者已驗證。 

您可以藉由將 Azure AD 新增至原則之擴充檔案中的 `<ClaimsProvider>` 節點，以將 Azure AD 定義為宣告提供者：

1. 從您的工作目錄中開啟擴充檔案 (TrustFrameworkExtensions.xml)。
1. 找出區段 `<ClaimsProviders>`。 如果不存在，請在根節點下新增。
1. 新增 `<ClaimsProvider>` 節點，如下所示︰

    ```XML
    <ClaimsProvider>
        <Domain>Contoso</Domain>
        <DisplayName>Login using Contoso</DisplayName>
        <TechnicalProfiles>
            <TechnicalProfile Id="ContosoProfile">
                <DisplayName>Contoso Employee</DisplayName>
                <Description>Login with your Contoso account</Description>
                <Protocol Name="OpenIdConnect"/>
                <OutputTokenFormat>JWT</OutputTokenFormat>
                <Metadata>
                    <Item Key="METADATA">https://login.windows.net/contoso.com/.well-known/openid-configuration</Item>
                    <Item Key="ProviderName">https://sts.windows.net/00000000-0000-0000-0000-000000000000/</Item>
                    <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
                    <Item Key="IdTokenAudience">00000000-0000-0000-0000-000000000000</Item>
                    <Item Key="response_types">id_token</Item>
                    <Item Key="UsePolicyInRedirectUri">false</Item>
                </Metadata>
                <CryptographicKeys>
                    <Key Id="client_secret" StorageReferenceId="B2C_1A_ContosoAppSecret"/>
                </CryptographicKeys>
                <OutputClaims>
                    <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="oid"/>
                    <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid"/>
                    <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
                    <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
                    <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
                    <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
                    <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="AzureADContoso" />
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

1. 在 `<ClaimsProvider>` 節點下，將 `<Domain>` 的值更新為可以與其他識別提供者有所區別的唯一值。
1. 在 `<ClaimsProvider>` 節點下，將 `<DisplayName>` 的值更新為宣告提供者的易記名稱。 目前不使用此值。

### <a name="update-the-technical-profile"></a>更新技術設定檔

若要從 Azure AD 端點取得權杖，您需要定義 Azure AD B2C 應該用來與 Azure AD 進行通訊的通訊協定。 這是在 `<ClaimsProvider>` 的 `<TechnicalProfile>` 元素內完成。
1. 更新 `<TechnicalProfile>` 節點的識別碼。 此識別碼可讓原則的其他部分參考此技術設定檔。
1. 更新 `<DisplayName>` 的值。 這個值會顯示在登入畫面的登入按鈕上。
1. 更新 `<Description>` 的值。
1. Azure AD 使用 OpenID Connect 通訊協定，因此請確定 `<Protocol>` 是 `"OpenIdConnect"`。

您必須更新稍早參考之 XML 檔案中的 `<Metadata>` 區段，以反映特定 Azure AD 租用戶的組態設定。 在 XML 檔案中，更新中繼資料值，如下所示︰

1. 將 `<Item Key="METADATA">` 設定為 `https://login.windows.net/yourAzureADtenant/.well-known/openid-configuration`，其中 `yourAzureADtenant` 是您的 Azure AD 租用戶名稱 (contoso.com)。
1. 開啟瀏覽器並移至剛更新的 `METADATA` URL。
1. 在瀏覽器中，尋找 'issuer' 物件並複製其值。 如下所示：`https://sts.windows.net/{tenantId}/`。
1. 將 `<Item Key="ProviderName">` 的值貼到 XML 檔案中。
1. 將 `<Item Key="client_id">` 設定為應用程式註冊時的應用程式識別碼。
1. 將 `<Item Key="IdTokenAudience">` 設定為應用程式註冊時的應用程式識別碼。
1. 確定 `<Item Key="response_types">` 設定為 `id_token`。
1. 確定 `<Item Key="UsePolicyInRedirectUri">` 設定為 `false`。

您也需要將您在 Azure AD B2C 租用戶中註冊的 Azure AD 祕密連結至 Azure AD `<ClaimsProvider>` 資訊：

* 在上述 XML 檔案的 `<CryptographicKeys>` 區段中，將 `StorageReferenceId` 的值更新為您定義之祕密的參考識別碼 (例如，`ContosoAppSecret`)。

### <a name="upload-the-extension-file-for-verification"></a>上傳擴充檔案準備驗證

現在，您應該已設定原則，所以 Azure AD B2C 知道如何與 Azure AD 目錄進行通訊。 嘗試上傳原則的擴充檔案，這只是為了確認它到目前為止沒有任何問題。 若要這樣做：

1. 開啟 Azure AD B2C 租用戶中的 [所有原則] 刀鋒視窗。
1. 勾選 [覆寫已存在的原則] 方塊。
1. 上傳擴充檔案 (TrustFrameworkExtensions.xml)，並確定它通過驗證。

## <a name="register-the-azure-ad-claims-provider-to-a-user-journey"></a>向使用者旅程圖註冊 Azure AD 宣告提供者

您現在需要將 Azure AD 識別提供者新增至其中一個使用者旅程圖。 目前，識別提供者已設定，但還未出現在任何註冊/登入畫面中。 為了這樣做，我們將建立現有範本使用者旅程圖的複本，然後將它修改成也有 Azure AD 識別提供者：

1. 開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。
1. 尋找 `<UserJourneys>` 元素，並複製含有 `Id="SignUpOrSignIn"` 的整個 `<UserJourney>`。
1. 開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，並尋找 `<UserJourneys>` 元素。 如果此元素不存在，請新增。
1. 貼上您複製的整個 `<UserJourney>` 節點作為 `<UserJourneys>` 元素的子元素。
1. 重新命名新使用者旅程圖的識別碼 (例如，重新命名為 `SignUpOrSignUsingContoso`)。

### <a name="display-the-button"></a>顯示「按鈕」

`<ClaimsProviderSelection>` 元素類似於註冊/登入畫面上的識別提供者按鈕。 如果您新增 Azure AD 的 `<ClaimsProviderSelection>` 元素，當使用者登陸頁面時，會出現新的按鈕。 若要新增此元素：

1. 在您剛剛建立的使用者旅程圖中，尋找包含 `Order="1"` 的 `<OrchestrationStep>` 節點。
1. 新增下列內容：

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

1. 將 `TargetClaimsExchangeId` 設定為適當的值。 建議您與其他人一樣，遵循相同的慣例：\[ClaimProviderName\]Exchange。

### <a name="link-the-button-to-an-action"></a>將按鈕連結至動作

現在已備妥按鈕，您需要將它連結至動作。 在此案例中，動作是讓 Azure AD B2C 與 Azure AD 通訊以接收權杖。 藉由連結 Azure AD 宣告提供者的技術設定檔，將按鈕連結至動作：

1. 尋找 `<OrchestrationStep>`，它在 `<UserJourney>` 節點中包含 `Order="2"`。
1. 新增下列內容：

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="ContosoProfile" />
    ```

1. 將 `Id` 更新為與上一節中之 `TargetClaimsExchangeId` 相同的值。
1. 將 `TechnicalProfileReferenceId` 更新為您稍早建立之技術設定檔的識別碼 (ContosoProfile)。

### <a name="upload-the-updated-extension-file"></a>上傳更新的擴充檔案

您已修改擴充檔案。 將其儲存。 然後上傳檔案，並確保所有驗證都成功。

### <a name="update-the-rp-file"></a>更新 RP 檔案

您現在需要更新信賴憑證者 (RP) 檔案，此檔案將起始您剛才建立的使用者旅程圖：

1. 將 SignUpOrSignIn.xml 複製到您的工作目錄，並重新命名 (例如，將它重新命名為 SignUpOrSignInWithAAD.xml)。
1. 開啟新檔案，將 `<TrustFrameworkPolicy>` 的 `PolicyId` 屬性更新成唯一值 (例如，SignUpOrSignInWithAAD)。 <br> 這會成為您的原則名稱 (例如，B2C\_1A\_SignUpOrSignInWithAAD)。
1. 修改 `<DefaultUserJourney>` 中的 `ReferenceId` 屬性，以符合您建立之新使用者旅程圖的識別碼 (例如，SignUpOrSignUsingContoso)。
1. 儲存變更並上傳檔案。

## <a name="troubleshooting"></a>疑難排解

測試您剛才上傳的自訂原則，方法是開啟其刀鋒視窗，然後按一下 [立即執行]。 若要診斷問題，請參閱[疑難排解](active-directory-b2c-troubleshoot-custom.md)。

## <a name="next-steps"></a>後續步驟

請將意見反應寄到 [AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com)。
