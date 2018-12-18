---
title: 在 Azure Active Directory B2C 的自訂原則中定義自我判斷技術設定檔 | Microsoft Docs
description: 在 Azure Active Directory B2C 的自訂原則中定義自我判斷技術設定檔。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 262c4e622d7ef915e59f29c8f4272e488acaeccf
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166906"
---
# <a name="define-a-self-asserted-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>在 Azure Active Directory B2C 自訂原則中定義自我判斷技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory (Azure AD) B2C 中預期使用者要在其中提供輸入的所有互動均為自我判斷技術設定檔。 例如，註冊頁面、登入頁面或密碼重設頁面。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `Proprietary`。 **handler** 屬性必須包含 Azure AD B2C 所使用之通訊協定處理常式組件的完整名稱，以進行自我判斷：`Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`

下列範例顯示適用於電子郵件註冊的自我判斷技術設定檔：

```XML
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>Email signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
```
 
## <a name="input-claims"></a>輸入宣告

在自我判斷技術設定檔中，您可以使用 **InputClaims** 與 **InputClaimsTransformations** 元素，預先填入要出現在自我判斷頁面上的宣告值 (輸出宣告)。 例如，在編輯設定檔原則中，使用者旅程圖會先從 Azure AD B2C 目錄服務中讀取使用者設定檔，接著，自我判斷技術設定檔會使用儲存於使用者設定檔的使用者資料來設定輸入宣告。 這些宣告均收集自使用者設定檔，然後呈現給使用者，讓使用者接著可以編輯現有的資料。

```XML
<TechnicalProfile Id="SelfAsserted-ProfileUpdate">
...
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <InputClaim ClaimTypeReferenceId="userPrincipalName" />
    <InputClaim ClaimTypeReferenceId="givenName" />
    <InputClaim ClaimTypeReferenceId="surname" />
  </InputClaims>
```


## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素包含一份要顯示來向使用者收集資料的宣告清單。 若要使用一些值預先填入輸出宣告，請使用先前所述的輸入宣告。 元素可能也包含預設值。 **OutputClaims** 中宣告的順序會控制 Azure AD B2C 在螢幕上呈現宣告的順序。 **DefaultValue** 屬性只有在先前未曾設定宣告時才會生效。 但是，如果之前已在先前的協調流程步驟中設定它，即使使用者將該值保留為空白，預設值還是不會生效。 若要強制使用預設值，請將 **AlwaysUseDefaultValue** 屬性設定為 `true`。 若要強制使用者提供特定輸出宣告的值，請將 **OutputClaims** 元素的 **Required** 屬性設定為 `true`。

**OutputClaims** 集合中的 **ClaimType** 元素必須將 **UserInputType** 元素設定為 Azure AD B2C 所支援的任何使用者輸入類型，例如 `TextBox` 或 `DropdownSingleSelect`。 或者，**OutputClaim** 元素必須設定 **DefaultValue**。  

**OutputClaimsTransformations** 元素可能包含 **OutputClaimsTransformation** 的集合，以用來修改輸出宣告或產生新的輸出宣告。

下列輸出宣告一律會設定為 `live.com`：

```XML
<OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" AlwaysUseDefaultValue="true" />
```

### <a name="use-case"></a>使用案例

有四個適用於輸出宣告的案例：

- **向使用者收集輸出宣告**：當您需要向使用者收集資訊 (例如出生日期) 時，應該將宣告新增至 **OutputClaims** 集合。 呈現給使用者的宣告必須指定 **UserInputType**，例如 `TextBox` 或 `DropdownSingleSelect`。 如果自我判斷技術設定檔包含會輸出相同宣告的驗證技術設定檔，Azure AD B2C 就不會向使用者呈現宣告。 如果沒有任何要呈現給使用者的輸出宣告，Azure AD B2C 就會略過技術設定檔。
- **設定輸出宣告中的預設值**：不會向使用者收集資料或從驗證技術設定檔傳回資料。 `LocalAccountSignUpWithLogonEmail` 自我判斷提示技術設定檔會將 **executed-SelfAsserted-Input** 宣告設定為 `true`。
- **驗證技術設定檔會傳回輸出宣告**：您的技術設定檔可能會呼叫驗證技術設定檔來傳回某些宣告。 您可能想要提升宣告，並將其傳回到使用者旅程圖中的下一個協調流程步驟。 例如，使用本機帳戶登入時，名為 `SelfAsserted-LocalAccountSignin-Email` 的自我判斷技術設定檔會呼叫名為 `login-NonInteractive` 的驗證技術設定檔。 此技術設定檔會驗證使用者認證，也會傳回使用者設定檔。 例如，'userPrincipalName'、'displayName'、'givenName' 與 'surName'。
- **透過輸出宣告轉換來輸出宣告**

在下列範例中，`LocalAccountSignUpWithLogonEmail` 自我判斷技術設定檔示範輸出宣告的使用方式，並將 **executed-SelfAsserted-Input** 設定為 `true`。 `objectId`、`authenticationSource`、`newUser` 宣告是 `AAD-UserWriteUsingLogonEmail` 驗證技術設定檔的輸出，不會向使用者顯示。

```XML
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>Email signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="IpAddressClaimReferenceId">IpAddress</Item>
    <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
    <Item Key="language.button_continue">Create</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
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
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surName" />
  </OutputClaims>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail" />
  </ValidationTechnicalProfiles>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>

```

## <a name="persist-claims"></a>保存宣告

如果 **PersistedClaims** 元素不存在，自我判斷技術設定檔就不會將資料保存到 Azure AD B2C， 而是改為呼叫負責保存資料的驗證技術設定檔。 例如，註冊原則會使用 `LocalAccountSignUpWithLogonEmail` 自我判斷技術設定檔來收集新的使用者設定檔。 `LocalAccountSignUpWithLogonEmail` 技術設定檔會呼叫驗證技術設定檔來建立 Azure AD B2C 中的帳戶。

## <a name="validation-technical-profiles"></a>驗證技術設定檔

驗證技術設定檔可用於驗證參考技術設定檔的部分或所有輸出宣告。 驗證技術設定檔的輸入宣告必須出現在自我判斷技術設定檔的輸出宣告中。 驗證技術設定檔會驗證使用者輸入，並且可將錯誤傳回給使用者。 

驗證技術設定檔可以是原則中的任何技術設定檔，例如 [Azure Active Directory](active-directory-technical-profile.md) 或 [REST API](restful-technical-profile.md) 技術設定檔。 在上述範例中，`LocalAccountSignUpWithLogonEmail` 技術設定檔會驗證 signinName 不存在目錄中。 如果沒有，驗證技術設定檔就會建立本機帳戶，並傳回 objectId、authenticationSource、newUser。 `SelfAsserted-LocalAccountSignin-Email` 技術設定檔會呼叫 `login-NonInteractive` 驗證技術設定檔來驗證使用者認證。

您也可以使用您的商務邏輯來呼叫 REST API 技術設定檔、覆寫輸入宣告，或透過進一步整合企業營運應用程式來擴充使用者資料。 如需詳細資訊，請參閱[驗證技術設定檔](validation-technical-profile.md)

## <a name="metadata"></a>中繼資料

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| setting.showContinueButton | 否 | 顯示繼續按鈕。 可能的值：`true` (預設) 或 `false` |
| setting.showCancelButton | 否 | 顯示取消按鈕。 可能的值：`true` (預設) 或 `false` |
| setting.operatingMode | 否 | 對於登入頁面，此屬性會控制使用者名稱欄位的行為，例如輸入驗證和錯誤訊息。 預期的值：`Username` 或 `Email`。 |
| ContentDefinitionReferenceId | 是 | 與此技術設定檔相關聯的[內容定義](contentdefinitions.md)識別碼。 |
| EnforceEmailVerification | 否 | 針對註冊或設定檔編輯，強制執行電子郵件驗證。 可能的值：`true` (預設) 或 `false`。 | 
| setting.showSignupLink | 否 | 顯示註冊按鈕。 可能的值：`true` (預設) 或 `false` |
| setting.retryLimit | 否 | 控制使用者可嘗試提供資料以針對驗證技術設定檔進行檢查的次數。 例如，使用者嘗試使用已經存在的帳戶進行註冊，並持續嘗試，直到達到限制為止。
| SignUpTarget | 否 | 註冊目標交換識別碼。 當使用者按一下註冊按鈕時，Azure AD B2C 就會執行指定的交換識別碼。 |

## <a name="cryptographic-keys"></a>密碼編譯金鑰

不使用 **CryptographicKeys** 元素。













