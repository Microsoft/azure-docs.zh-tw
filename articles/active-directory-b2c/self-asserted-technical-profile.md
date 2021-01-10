---
title: 在自訂原則中定義自我判斷技術設定檔
titleSuffix: Azure AD B2C
description: 在 Azure Active Directory B2C 的自訂原則中定義自我判斷技術設定檔。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/26/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 08b08e3e799ff7b579889a62ecec70677a3cbce9
ms.sourcegitcommit: 31cfd3782a448068c0ff1105abe06035ee7b672a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/10/2021
ms.locfileid: "98059053"
---
# <a name="define-a-self-asserted-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>在 Azure Active Directory B2C 自訂原則中定義自我判斷技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) 中的所有互動都必須提供自動判斷的技術設定檔，才能讓使用者提供輸入。 例如，註冊頁面、登入頁面或密碼重設頁面。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `Proprietary`。 **handler** 屬性必須包含 Azure AD B2C 所使用之通訊協定處理常式組件的完整名稱，以進行自我判斷：`Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`

下列範例顯示適用於電子郵件註冊的自我判斷技術設定檔：

```xml
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>Email signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
```

## <a name="input-claims"></a>輸入宣告

在自我判斷技術設定檔中，您可以使用 **InputClaims** 和 **InputClaimsTransformations** 元素，預先填入出現在自我判斷提示頁面上的宣告值， (顯示宣告) 。 例如，在編輯設定檔原則中，使用者旅程圖會先從 Azure AD B2C 目錄服務中讀取使用者設定檔，接著，自我判斷技術設定檔會使用儲存於使用者設定檔的使用者資料來設定輸入宣告。 這些宣告均收集自使用者設定檔，然後呈現給使用者，讓使用者接著可以編輯現有的資料。

```xml
<TechnicalProfile Id="SelfAsserted-ProfileUpdate">
...
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <InputClaim ClaimTypeReferenceId="userPrincipalName" />
    <InputClaim ClaimTypeReferenceId="givenName" />
    <InputClaim ClaimTypeReferenceId="surname" />
  </InputClaims>
```

## <a name="display-claims"></a>顯示宣告

顯示宣告功能目前為 **預覽** 狀態。

**DisplayClaims** 元素包含要在螢幕上呈現以收集使用者資料的宣告清單。 若要預先填入顯示宣告的值，請使用先前所述的輸入宣告。 元素可能也包含預設值。

**DisplayClaims** 中的宣告順序會指定 Azure AD B2C 在螢幕上呈現宣告的順序。 若要強制使用者提供特定宣告的值，請將 **DisplayClaim** 元素的 **必要** 屬性設定為 `true` 。

**DisplayClaims** 集合中的 **ClaimType** 元素必須將 **>userinputtype** 元素設定為 Azure AD B2C 所支援的任何使用者輸入類型。 例如，`TextBox` 或 `DropdownSingleSelect`。

### <a name="add-a-reference-to-a-displaycontrol"></a>新增控制項的參考

在 [顯示宣告] 集合中，您可以包含已建立之 [控制項](display-controls.md) 的參考。 顯示控制項是具有特殊功能並與 Azure AD B2C 後端服務互動的使用者介面元素。 它可讓使用者在頁面上執行動作，該頁面會在後端叫用驗證技術設定檔。 例如，確認電子郵件地址、電話號碼或客戶忠誠度號碼。

下列範例 `TechnicalProfile` 說明如何搭配顯示控制項使用顯示宣告。

* 第一個顯示宣告會參考 `emailVerificationControl` 顯示控制項，以收集並驗證電子郵件地址。
* 第五個顯示宣告會參考 `phoneVerificationControl` 顯示控制項，以收集並驗證電話號碼。
* 系統會 Claimtype 其他顯示宣告，以便從使用者收集。

```xml
<TechnicalProfile Id="Id">
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
    <DisplayClaim DisplayControlReferenceId="phoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
  </DisplayClaims>
</TechnicalProfile>
```

如前所述，具有顯示控制項參考的顯示宣告可能會執行自己的驗證，例如驗證電子郵件地址。 此外，自我判斷頁面支援使用驗證技術設定檔來驗證整個頁面，包括任何使用者輸入 (宣告類型或顯示控制項) ，然後再繼續進行下一個協調流程步驟。

### <a name="combine-usage-of-display-claims-and-output-claims-carefully"></a>仔細合併顯示宣告和輸出宣告的使用方式

如果您在自我判斷技術設定檔中指定一或多個 **DisplayClaim** 專案，您必須針對想要在螢幕上顯示並從使用者收集的 *每個* 宣告使用 DisplayClaim。 自動判斷提示的技術設定檔（至少包含一個顯示宣告）不會顯示任何輸出宣告。

請考慮下列範例，其中的宣告 `age` 會在基底原則中定義為 **輸出** 宣告。 將任何顯示宣告新增到自我判斷技術設定檔之前，宣告 `age` 會顯示在螢幕上，以便從使用者收集資料：

```xml
<TechnicalProfile Id="id">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="age" />
  </OutputClaims>
</TechnicalProfile>
```

如果繼承該基底的分葉原則接著將指定 `officeNumber` 為 **顯示** 宣告：

```xml
<TechnicalProfile Id="id">
  <DisplayClaims>
    <DisplayClaim ClaimTypeReferenceId="officeNumber" />
  </DisplayClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="officeNumber" />
  </OutputClaims>
</TechnicalProfile>
```

`age`基底原則中的宣告不會再顯示在使用者的畫面上，它實際上是「隱藏」。 若要顯示宣告 `age` 並從使用者收集存留期值，您必須新增 `age` **DisplayClaim**。

## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素包含要傳回至下一個協調流程步驟的宣告清單。 只有從未設定過宣告時， **DefaultValue** 屬性才會生效。 如果設定在先前的協調流程步驟中，即使使用者將值保留空白，預設值也不會生效。 若要強制使用預設值，請將 **AlwaysUseDefaultValue** 屬性設定為 `true`。

基於安全性理由， (設定為) 的密碼宣告值 `UserInputType` `Password` 僅適用于自我判斷技術設定檔的驗證技術設定檔。 您無法在下一個協調流程步驟中使用密碼宣告。 

> [!NOTE]
> 在舊版 Identity Experience Framework (IEF) 中，會使用輸出宣告來收集使用者的資料。 若要從使用者收集資料，請改用 **DisplayClaims** 集合。

**OutputClaimsTransformations** 元素可能含有 **OutputClaimsTransformation** 的集合，用於修改輸出宣告或產生新的輸出宣告。

### <a name="when-you-should-use-output-claims"></a>當您應使用輸出宣告時

在自我判斷技術設定檔中，輸出宣告集合會將宣告傳回到下一個協調流程步驟。

使用輸出宣告的時機：

- **宣告是由輸出宣告轉換輸出**。
- **在輸出宣告中設定預設值** ，而不需要從使用者收集資料，或從驗證技術設定檔傳回資料。 `LocalAccountSignUpWithLogonEmail` 自我判斷提示技術設定檔會將 **executed-SelfAsserted-Input** 宣告設定為 `true`。
- **驗證技術設定檔會傳回輸出宣告**：您的技術設定檔可能會呼叫驗證技術設定檔來傳回某些宣告。 您可能想要提升宣告，並將其傳回到使用者旅程圖中的下一個協調流程步驟。 例如，使用本機帳戶登入時，名為 `SelfAsserted-LocalAccountSignin-Email` 的自我判斷技術設定檔會呼叫名為 `login-NonInteractive` 的驗證技術設定檔。 此技術設定檔會驗證使用者認證，也會傳回使用者設定檔。 例如，'userPrincipalName'、'displayName'、'givenName' 與 'surName'。
- **顯示控制項會傳回輸出宣告** ，您的技術設定檔可能會有 [顯示控制項](display-controls.md)的參考。 顯示控制項會傳回一些宣告，例如已驗證的電子郵件地址。 您可能想要提升宣告，並將其傳回到使用者旅程圖中的下一個協調流程步驟。 顯示控制項功能目前為 **預覽** 狀態。

下列範例示範如何使用可同時使用顯示宣告和輸出宣告的自我判斷技術設定檔。

```xml
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>Email signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="IpAddressClaimReferenceId">IpAddress</Item>
    <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
    <Item Key="language.button_continue">Create</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" />
  </InputClaims>
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim DisplayControlReferenceId="SecondaryEmailVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
  </DisplayClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" Required="true" />
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" DefaultValue="true" />
    <OutputClaim ClaimTypeReferenceId="authenticationSource" />
    <OutputClaim ClaimTypeReferenceId="newUser" />
  </OutputClaims>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail" />
  </ValidationTechnicalProfiles>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>
```

### <a name="output-claims-sign-up-or-sign-in-page"></a>輸出宣告註冊或登入頁面

在合併的註冊和登入頁面上，使用內容定義 [DataUri](contentdefinitions.md#datauri) 專案時，請注意下列 `unifiedssp` `unifiedssd` 事項：指定或頁面類型：

- 只會轉譯使用者名稱和密碼宣告。
- 前兩個輸出宣告必須是使用者名稱，而密碼 (順序) 。 
- 任何其他宣告都不會呈現;針對這些宣告，您必須設定 `defaultValue` 或叫用宣告表單驗證技術設定檔。 

## <a name="persist-claims"></a>保存宣告

未使用 PersistedClaims 元素。 自我判斷技術設定檔不會保存要 Azure AD B2C 的資料。 而是改為呼叫負責保存資料的驗證技術設定檔。 例如，註冊原則會使用 `LocalAccountSignUpWithLogonEmail` 自我判斷技術設定檔來收集新的使用者設定檔。 `LocalAccountSignUpWithLogonEmail` 技術設定檔會呼叫驗證技術設定檔來建立 Azure AD B2C 中的帳戶。

## <a name="validation-technical-profiles"></a>驗證技術設定檔

驗證技術設定檔可用於驗證參考技術設定檔的部分或所有輸出宣告。 驗證技術設定檔的輸入宣告必須出現在自我判斷技術設定檔的輸出宣告中。 驗證技術設定檔會驗證使用者輸入，並且可將錯誤傳回給使用者。

驗證技術設定檔可以是原則中的任何技術設定檔，例如 [Azure Active Directory](active-directory-technical-profile.md) 或 [REST API](restful-technical-profile.md) 技術設定檔。 在上述範例中，`LocalAccountSignUpWithLogonEmail` 技術設定檔會驗證 signinName 不存在目錄中。 如果沒有，驗證技術設定檔就會建立本機帳戶，並傳回 objectId、authenticationSource、newUser。 `SelfAsserted-LocalAccountSignin-Email` 技術設定檔會呼叫 `login-NonInteractive` 驗證技術設定檔來驗證使用者認證。

您也可以使用您的商務邏輯來呼叫 REST API 技術設定檔、覆寫輸入宣告，或透過進一步整合企業營運應用程式來擴充使用者資料。 如需詳細資訊，請參閱[驗證技術設定檔](validation-technical-profile.md)

## <a name="metadata"></a>中繼資料

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| 設定. operatingMode <sup>1</sup>| 否 | 對於登入頁面，此屬性會控制使用者名稱欄位的行為，例如輸入驗證和錯誤訊息。 預期的值：`Username` 或 `Email`。  |
| AllowGenerationOfClaimsWithNullValues| 否| 允許產生具有 null 值的宣告。 例如，在案例中，使用者不會選取核取方塊。|
| ContentDefinitionReferenceId | 是 | 與此技術設定檔相關聯的[內容定義](contentdefinitions.md)識別碼。 |
| EnforceEmailVerification | 否 | 針對註冊或設定檔編輯，強制執行電子郵件驗證。 可能的值：`true` (預設) 或 `false`。 |
| setting.retryLimit | 否 | 控制使用者可以嘗試提供針對驗證技術設定檔所檢查之資料的次數。 例如，使用者嘗試使用已經存在的帳戶進行註冊，並持續嘗試，直到達到限制為止。
| SignUpTarget <sup>1</sup>| 否 | 註冊目標交換識別碼。 當使用者按一下註冊按鈕時，Azure AD B2C 就會執行指定的交換識別碼。 |
| setting.showCancelButton | 否 | 顯示取消按鈕。 可能的值：`true` (預設) 或 `false` |
| setting.showContinueButton | 否 | 顯示繼續按鈕。 可能的值：`true` (預設) 或 `false` |
| 設定. showSignupLink <sup>2</sup>| 否 | 顯示註冊按鈕。 可能的值：`true` (預設) 或 `false` |
| 設定. forgotPasswordLinkLocation <sup>2</sup>| 否| 顯示忘記的密碼連結。 可能的值： `AfterLabel` (預設值) 在標籤之後或在沒有標籤的情況下，于 [密碼輸入] 欄位之後顯示連結、在 [  `AfterInput` 密碼輸入] 欄位之後顯示連結、在 `AfterButtons` 按鈕之後顯示表單底部的連結，或 `None` 移除 [忘記密碼] 連結。|
| 設定. enableRememberMe <sup>2</sup>| 否| 顯示 [ [讓我保持登入](session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi) ] 核取方塊。 可能的值： `true` 或 `false` (預設) 。 |
| 設定. inputVerificationDelayTimeInMilliseconds <sup>3</sup>| 否| 藉由等待使用者停止輸入，然後驗證值，來改善使用者體驗。 預設值為2000毫秒。 |
| IncludeClaimResolvingInClaimsHandling  | 否 | 針對輸入和輸出宣告，指定技術設定檔中是否包含 [宣告解析](claim-resolver-overview.md) 。 可能的值為：`true` 或 `false` (預設)。 如果您想要在技術設定檔中使用宣告解析程式，請將此設定為 `true` 。 |

注意：
1. 適用于內容定義 [DataUri](contentdefinitions.md#datauri) 類型的 `unifiedssp` 或 `unifiedssd` 。
1. 適用于內容定義 [DataUri](contentdefinitions.md#datauri) 類型的 `unifiedssp` 或 `unifiedssd` 。 1.1.0 和更新[版本的頁面配置](page-layout.md)。
1. 適用于1.2.0 和更新 [版本的頁面配置](page-layout.md) 。

## <a name="cryptographic-keys"></a>密碼編譯金鑰

不使用 **CryptographicKeys** 元素。
