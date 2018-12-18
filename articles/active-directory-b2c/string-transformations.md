---
title: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的字串宣告轉換範例 | Microsoft Docs
description: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的字串宣告轉換範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: ce2ad3e699b930f801ad47083d6cfcf6a7937a5c
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433441"
---
# <a name="string-claims-transformations"></a>字串宣告轉換

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

此文章提供在 Azure Active Directory (Azure AD) B2C 中，使用識別體驗架構結構描述字串宣告轉換的範例。 如需詳細資訊，請參閱 [ClaimsTransformations](claimstransformations.md)。

## <a name="assertstringclaimsareequal"></a>AssertStringClaimsAreEqual 

比較兩個宣告，如果根據指定的比較 inputClaim1、inputClaim2 和 stringComparison 它們並不相等，即會擲回例外狀況。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | inputClaim1 | 字串 | 要比較的第一個宣告類型。 |
| inputClaim | inputClaim2 | 字串 | 要比較的第二個宣告類型。 |
| InputParameter | stringComparison | 字串 | 字串比較，其中一個值：Ordinal、OrdinalIgnoreCase。 |

**AssertStringClaimsAreEqual** 宣告轉換一律會從[驗證技術設定檔](validation-technical-profile.md)執行，其會透過[自我判斷技術設定檔](self-asserted-technical-profile.md)來呼叫。 **UserMessageIfClaimsTransformationStringsAreNotEqual** 自我判斷技術設定檔中繼資料會控制要呈現給使用者的錯誤訊息。

![AssertStringClaimsAreEqual 執行](./media/string-transformations/assert-execution.png)

您可以使用此宣告轉換來確定兩個 ClaimTypes 具有相同的值。 如果沒有，則會擲回錯誤訊息。 下列範例會檢查 **strongAuthenticationEmailAddress** ClaimType 等於 **email** ClaimType。 否則會擲回錯誤訊息。 

```XML
<ClaimsTransformation Id="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" TransformationMethod="AssertStringClaimsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="strongAuthenticationEmailAddress" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
  </InputParameters>
</ClaimsTransformation>
```


**login-NonInteractive** 驗證技術設定檔會呼叫 **AssertEmailAndStrongAuthenticationEmailAddressAreEqual** 宣告轉換。
```XML
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

自我判斷技術設定檔會呼叫驗證 **login-NonInteractive** 技術設定檔。

```XML
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationStringsAreNotEqual">Custom error message the email addresses you provided are not the same.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **inputClaim1**：someone@contoso.com
    - **inputClaim2**：someone@outlook.com
 - 輸入參數：
    - **stringComparison**：ordinalIgnoreCase
- 結果：擲回錯誤

## <a name="changecase"></a>ChangeCase 

根據運算子，將所提供宣告的大小寫變更為小寫或大寫字母。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | 字串 | 要變更的 ClaimType。 |
| InputParameter | toCase | 字串 | 下列其中一個值：`LOWER` 或 `UPPER`。 |
| OutputClaim | outputClaim | 字串 | 叫用此宣告轉換之後所產生的 ClaimType。 |

使用此宣告轉換來將任何字串 ClaimType 變更為小寫或大寫字母。  

```XML
<ClaimsTransformation Id="ChangeToLower" TransformationMethod="ChangeCase">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="toCase" DataType="string" Value="LOWER" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **email**：SomeOne@contoso.com
- 輸入參數：
    - **toCase**：LOWER
- 輸出宣告：
    - **email**：someone@contoso.com

## <a name="createstringclaim"></a>CreateStringClaim 

從原則中提供的輸入參數建立字串宣告。

| Item | TransformationClaimType | 資料類型 | 注意 |
|----- | ----------------------- | --------- | ----- |
| InputParameter | value | 字串 | 要設定的字串 |
| OutputClaim | createdClaim | 字串 | 叫用此宣告轉換之後所產生的 ClaimType，並含有輸入參數中指定的值。 |

使用此宣告轉換來設定字串 ClaimType 值。

```XML
<ClaimsTransformation Id="CreateTermsOfService" TransformationMethod="CreateStringClaim">
  <InputParameters>
    <InputParameter Id="value" DataType="string" Value="Contoso terms of service..." />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="TOS" TransformationClaimType="createdClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入參數：
    - **value**：Contoso terms of service...
- 輸出宣告：
    - **createdClaim**：TOS ClaimType 包含 "Contoso terms of service..." 值。

## <a name="compareclaims"></a>CompareClaims

判斷某個字串宣告是否等於另一個。 結果是新的布林值 ClaimType 且含有 `true` 或 `false` 的值。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | inputClaim1 | 字串 | 要比較的第一個宣告類型。 |
| inputClaim | inputClaim2 | 字串 | 要比較的第二個宣告類型。 |
| InputParameter | operator | 字串 | 可能的值：`EQUAL` 或 `NOT EQUAL`。 |
| InputParameter | ignoreCase | 布林值 | 指定這個比較是否應忽略要比較之字串的大小寫。 |
| OutputClaim | outputClaim | 布林值 | 叫用此宣告轉換之後所產生的 ClaimType。 |

使用此宣告轉換來檢查某個宣告是否等於另一個宣告。 例如，下列宣告轉換會檢查 **email** 宣告的值是否等於 **Verified.Email** 宣告。

```XML
<ClaimsTransformation Id="CheckEmail" TransformationMethod="CompareClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="Email" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="Verified.Email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="SameEmailAddress" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **inputClaim1**：someone@contoso.com
    - **inputClaim2**：someone@outlook.com
- 輸入參數：
    - **operator**：NOT EQUAL
    - **ignoreCase**：true
- 輸出宣告：
    - **outputClaim**：true

## <a name="compareclaimtovalue"></a>CompareClaimToValue

判斷宣告值是否等於輸入參數值。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | inputClaim1 | 字串 | 要比較的宣告類型。 |
| InputParameter | operator | 字串 | 可能的值：`EQUAL` 或 `NOT EQUAL`。 |
| InputParameter | compareTo | 字串 | 字串比較，其中一個值：Ordinal、OrdinalIgnoreCase。 |
| InputParameter | ignoreCase | 布林值 | 指定這個比較是否應忽略要比較之字串的大小寫。 |
| OutputClaim | outputClaim | 布林值 | 叫用此宣告轉換之後所產生的 ClaimType。 |

您可以使用此宣告轉換來檢查某個宣告是否等於您指定的值。 例如，下列宣告轉換會檢查 **termsOfUseConsentVersion** 宣告的值是否等於 `v1`。

```XML
<ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="compareTo" DataType="string" Value="V1" />
    <InputParameter Id="operator" DataType="string" Value="not equal" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例
- 輸入宣告：
    - **inputClaim1**：v1
- 輸入參數：
    - **compareTo**：V1
    - **operator**：EQUAL 
    - **ignoreCase**：true
- 輸出宣告：
    - **outputClaim**：true

## <a name="createrandomstring"></a>CreateRandomString

使用隨機號碼產生器來建立隨機字串。 如果隨機號碼產生器的類型為 `integer`，可能會選擇性地提供種子參數和最大數。 選擇性的字串格式參數允許使用它來將輸出格式化，而選擇性的 base64 參數會指定輸出是否為 base64 編碼的 randomGeneratorType [guid, integer] outputClaim (字串)。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputParameter | randomGeneratorType | 字串 | 指定要產生的隨機值：`GUID` (全域唯一識別碼) 或 `INTEGER` (數字)。 |
| InputParameter | stringFormat | 字串 | [選擇性] 將隨機值格式化。 |
| InputParameter | base64 | 布林值 | [選擇性] 將隨機值轉換為 base64。 如果套用字串格式，則會將字串格式之後的值編碼為 base64。 |
| InputParameter | maximumNumber | int | [選擇性] 僅適用於 `INTEGER` randomGeneratorType。 指定最大數。 |
| InputParameter | seed  | int | [選擇性] 僅適用於 `INTEGER` randomGeneratorType。 指定隨機值的種子。 注意：相同的種子會產生相同的隨機數字序列。 |
| OutputClaim | outputClaim | 字串 | 叫用此宣告轉換之後將產生的 ClaimType。 隨機值。 |

下列範例會產生全域唯一識別碼。 此宣告轉換可用來建立隨機的 UPN (使用者主體名稱)。

```XML
<ClaimsTransformation Id="CreateRandomUPNUserName" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="GUID" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>範例

- 輸入參數：
    - **randomGeneratorType**：GUID
- 輸出宣告： 
    - **outputClaim**：bc8bedd2-aaa3-411e-bdee-2f1810b73dfc

下列範例會產生介於 0 到 1000 之間的整數隨機值。 此值會格式化為 OTP_{隨機值}。

```XML
<ClaimsTransformation Id="SetRandomNumber" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="INTEGER" />
    <InputParameter Id="maximumNumber" DataType="int" Value="1000" />
    <InputParameter Id="stringFormat" DataType="string" Value="OTP_{0}" />
    <InputParameter Id="base64" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="randomNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入參數：
    - **randomGeneratorType**：INTEGER
    - **maximumNumber**：1000
    - **stringFormat**：OTP_{0}
    - **base64**：false
- 輸出宣告： 
    - **outputClaim**：OTP_853


## <a name="formatstringclaim"></a>FormatStringClaim

根據提供的格式字串來將宣告格式化。 此轉換會使用 C# `String.Format` 方法。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |字串 |做為字串格式 {0} 參數的 ClaimType。 |
| InputParameter | stringFormat | 字串 | 字串格式，包括 {0} 參數。 |
| OutputClaim | outputClaim | 字串 | 叫用此宣告轉換之後所產生的 ClaimType。 |

使用此宣告轉換，利用一個參數 {0} 來將任何字串格式化。 下列範例會建立 **userPrincipalName**。 所有社交識別提供者技術設定檔 (例如 `Facebook-OAUTH`) 會呼叫 **CreateUserPrincipalName** 來產生 **userPrincipalName**。   

```XML
<ClaimsTransformation Id="CreateUserPrincipalName" TransformationMethod="FormatStringClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="cpim_{0}@{RelyingPartyTenantId}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **inputClaim**：5164db16-3eee-4629-bfda-dcc3326790e9
- 輸入參數：
    - **stringFormat**：cpim_{0}@{RelyingPartyTenantId}
- 輸出宣告：
    - **outputClaim**：cpim_5164db16-3eee-4629-bfda-dcc3326790e9@b2cdemo.onmicrosoft.com

## <a name="formatstringmultipleclaims"></a>FormatStringMultipleClaims

根據提供的格式字串來將兩個宣告格式化。 此轉換會使用 C# **String.Format** 方法。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |字串 | 做為字串格式 {0} 參數的 ClaimType。 |
| InputClaim | inputClaim | 字串 | 做為字串格式 {1} 參數的 ClaimType。 |
| InputParameter | stringFormat | 字串 | 字串格式，包括 {0} 和 {1} 參數。 |
| OutputClaim | outputClaim | 字串 | 叫用此宣告轉換之後所產生的 ClaimType。 |

使用此宣告轉換，利用兩個參數 ({0} 和 {1}) 來將任何字串格式化。 下列範例會使用指定的格式來建立 **displayName**：

```XML
<ClaimsTransformation Id="CreateDisplayNameFromFirstNameAndLastName" TransformationMethod="FormatStringMultipleClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="surName" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="{0} {1}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="displayName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **inputClaim1**：Joe
    - **inputClaim2**：Fernando
- 輸入參數：
    - **stringFormat**：{0} {1}
- 輸出宣告：
    - **outputClaim**：Joe Fernando

## <a name="getmappedvaluefromlocalizedcollection"></a>GetMappedValueFromLocalizedCollection

從宣告 **Restriction** 集合查詢項目。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | mapFromClaim | 字串 | 包含要使用 **Restriction** 集合在 **restrictionValueClaim** 宣告中查詢之文字的宣告。  |
| OutputClaim | restrictionValueClaim | 字串 | 包含 **Restriction** 集合的宣告。 叫用宣告轉換之後，此宣告的值會包含所選取項目的值。 |

下列範例會根據錯誤索引鍵查詢錯誤訊息說明。 **ResponseMsg** 宣告會包含要呈現給使用者或傳送到信賴憑證者的錯誤訊息集合。

```XML
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cant sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
宣告轉換會查詢項目的文字並傳回其值。 如果限制會使用 `<LocalizedCollection>` 進行當地語系化，則宣告轉換會傳回當地語系化的值。

```XML
<ClaimsTransformation Id="GetResponseMsgMappedToResponseCode" TransformationMethod="GetMappedValueFromLocalizedCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="responseCode" TransformationClaimType="mapFromClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="restrictionValueClaim" />         
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **mapFromClaim**：B2C_V1_90001
- 輸出宣告：
    - **restrictionValueClaim**：You cant sign in because you are a minor。

## <a name="lookupvalue"></a>LookupValue

根據另一個宣告的值，從值清單中查詢某個宣告值。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputParameterId | 字串 | 包含查閱值的宣告 |
| InputParameter | |字串 | InputParameters 的集合。 |
| InputParameter | errorOnFailedLookup | 布林值 | 控制沒有相符的查閱時是否要傳回錯誤。 |
| OutputClaim | inputParameterId | 字串 | 叫用此宣告轉換之後將產生的 ClaimType。 比對識別碼的值。 |

下列範例會查詢其中一個 inpuParameters 集合中的網域名稱。 宣告轉換會查詢識別項中的網域名稱，並傳回其值 (應用程式識別碼)。

```XML
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation> 
```

### <a name="example"></a>範例

- 輸入宣告：
    - **inputParameterId**：test.com
- 輸入參數：
    - **contoso.com**：13c15f79-8fb1-4e29-a6c9-be0d36ff19f1
    - **microsoft.com**：0213308f-17cb-4398-b97e-01da7bd4804e
    - **test.com**：c7026f88-4299-4cdb-965d-3f166464b8a9
    - **errorOnFailedLookup**：false
- 輸出宣告：
    - **outputClaim**：c7026f88-4299-4cdb-965d-3f166464b8a9

## <a name="nullclaim"></a>NullClaim

清除指定宣告的值。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | claim_to_null | 字串 | 值為 NULL 的宣告。 |

使用此宣告轉換，從宣告屬性包中移除不必要的資料。 因此，工作階段 Cookie 將變得比較小。 下列範例會移除 `TermsOfService` 宣告類型的值。

```XML
<ClaimsTransformation Id="SetTOSToNull" TransformationMethod="NullClaim">
  <OutputClaims>
  <OutputClaim ClaimTypeReferenceId="TermsOfService" TransformationClaimType="claim_to_null" />
  </OutputClaims>
</ClaimsTransformation>
```

- 輸入宣告：
    - **outputClaim**：歡迎使用 Contoso 應用程式。 如果您繼續瀏覽並使用此網站，表示您同意遵循並遵守下列條款及條件...
- 輸出宣告：
    - **outputClaim**：NULL

## <a name="parsedomain"></a>ParseDomain

取得電子郵件地址的網域部分。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | emailAddress | 字串 | 包含電子郵件地址的 ClaimType。 |
| OutputClaim | 網域 | 字串 | 叫用此宣告轉換之後所產生的 ClaimType - 網域。 |

使用此宣告轉換來剖析使用者 @ 符號之後的網域名稱。 這有助於從稽核資料中移除個人識別資訊 (PII)。 下列宣告轉換示範如何從 **email** 宣告中剖析網域名稱。

```XML
<ClaimsTransformation Id="SetDomainName" TransformationMethod="ParseDomain">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="emailAddress" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="domain" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **emailAddress**：joe@outlook.com
- 輸出宣告：
    - **domain**：outlook.com

## <a name="setclaimsifstringsareequal"></a>SetClaimsIfStringsAreEqual

檢查宣告字串和 `matchTo` 輸入參數相等，並使用 `stringMatchMsg` 和 `stringMatchMsgCode` 輸入參數中存在的值來設定輸出宣告，以及比較結果輸出宣告，其會根據比較的結果設定為 `true` 或 `false`。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | inputClaim | 字串 | 要比較的宣告類型。 |
| InputParameter | matchTo | 字串 | 要與 `inputClaim` 比較的字串。 |
| InputParameter | stringComparison | 字串 | 可能的值：`Ordinal` 或 `OrdinalIgnoreCase`。 |
| InputParameter | stringMatchMsg | 字串 | 字串相等時要設定的第一個值。 |
| InputParameter | stringMatchMsgCode | 字串 | 字串相等時要設定的第二個值。 |
| OutputClaim | outputClaim1 | 字串 | 如果字串相等，此輸出宣告會包含 `stringMatchMsg` 輸入參數的值。 |
| OutputClaim | outputClaim2 | 字串 | 如果字串相等，此輸出宣告會包含 `stringMatchMsgCode` 輸入參數的值。 |
| OutputClaim | stringCompareResultClaim | 布林值 | 比較結果輸出宣告類型，其會根據比較的結果設定為 `true` 或 `false`。 |

您可以使用此宣告轉換來檢查某個宣告是否等於您指定的值。 例如，下列宣告轉換會檢查 **termsOfUseConsentVersion** 宣告的值是否等於 `v1`。 如果是，將值變更為 `v2`。 

```XML
<ClaimsTransformation Id="CheckTheTOS" TransformationMethod="SetClaimsIfStringsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="v1" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="stringMatchMsg" DataType="string" Value="B2C_V1_90005" />
    <InputParameter Id="stringMatchMsgCode" DataType="string" Value="The TOS is upgraded to v2" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="outputClaim1" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeCode" TransformationClaimType="outputClaim2" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeResult" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>範例

- 輸入宣告：
    - **inputClaim**：v1
- 輸入參數：
    - **matchTo**：V1
    - **stringComparison**：ordinalIgnoreCase 
    - **stringMatchMsg**：B2C_V1_90005
    - **stringMatchMsgCode**：TOS 會升級到 v2
- 輸出宣告：
    - **outputClaim1**：B2C_V1_90005
    - **outputClaim2**：TOS 會升級到 v2
    - **stringCompareResultClaim**：true

## <a name="setclaimsifstringsmatch"></a>SetClaimsIfStringsMatch

檢查宣告字串和 `matchTo` 輸入參數相等，並使用 `outputClaimIfMatched` 輸入參數中存在的值來設定輸出宣告，以及比較結果輸出宣告，其會根據比較的結果設定為 `true` 或 `false`。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | claimToMatch | 字串 | 要比較的宣告類型。 |
| InputParameter | matchTo | 字串 | 要與 inputClaim 比較的字串。 |
| InputParameter | stringComparison | 字串 | 可能的值：`Ordinal` 或 `OrdinalIgnoreCase`。 |
| InputParameter | outputClaimIfMatched | 字串 | 字串相等時要設定的值。 |
| OutputClaim | outputClaim | 字串 | 如果字串相等，此輸出宣告會包含 `outputClaimIfMatched` 輸入參數的值。 或者，如果字串不符，則為 Null。 |
| OutputClaim | stringCompareResultClaim | 布林值 | 比較結果輸出宣告類型，其會根據比較的結果設定為 `true` 或 `false`。 |

例如，下列宣告轉換會檢查 **ageGroup** 宣告的值是否等於 `Minor`。 如果是，會將值傳回到 `B2C_V1_90001`。 

```XML
<ClaimsTransformation Id="SetIsMinor" TransformationMethod="SetClaimsIfStringsMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="ageGroup" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="Minor" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="B2C_V1_90001" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isMinor" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isMinorResponseCode" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **claimToMatch**：Minor
- 輸入參數：
    - **matchTo**：Minor
    - **stringComparison**：ordinalIgnoreCase 
    - **outputClaimIfMatched**：B2C_V1_90001
- 輸出宣告：
    - **isMinorResponseCode**：B2C_V1_90001
    - **isMinor**：true

