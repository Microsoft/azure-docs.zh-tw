---
title: 在 Azure Active Directory B2C 自訂原則中的密碼複雜度 | Microsoft Docs
description: 如何在自訂原則中設定密碼複雜度需求。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: ed0001d8d88a2604e3128a4d5f7a365aeb7b00b1
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440786"
---
# <a name="configure-password-complexity-in-custom-policies"></a>在自訂原則中設定密碼複雜度

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文是密碼複雜度運作方式的進階描述，並使用 Azure AD B2C 自訂原則來啟用。

## <a name="azure-ad-b2c-configure-complexity-requirements-for-passwords"></a>Azure AD B2C：設定密碼複雜度需求

Azure Active Directory B2C (Azure AD B2C) 支援的變更密碼複雜度需求，是在建立帳戶時由使用者所提供的。  根據預設，Azure AD B2C 是使用**強式**密碼。  Azure AD B2C 也支援組態選項，可控制客戶可以使用的密碼複雜度。  本文示範如何在自訂原則中設定密碼複雜度。  它也可使用[在內建原則中設定密碼複雜度](active-directory-b2c-reference-password-complexity.md)。

## <a name="prerequisites"></a>先決條件

如[開始使用](active-directory-b2c-get-started-custom.md)所述，設定為完成本機帳戶註冊/登入的 Azure AD B2C 租用戶。

## <a name="how-to-configure-password-complexity-in-custom-policy"></a>如何在自訂原則中設定密碼複雜度

若要設定自訂原則中的密碼複雜度，自訂原則的整體結構必須包含 `BuildingBlocks` 內的 `ClaimsSchema`、`Predicates`，和 `InputValidations` 元素。

```XML
  <BuildingBlocks>
    <ClaimsSchema>...</ClaimsSchema>
    <Predicates>...</Predicates>
    <InputValidations>...</InputValidations>
  </BuildingBlocks>
```

這些元素的用途如下所示：

- 每個 `Predicate` 元素都會定義傳回 true 或 false 的基本字串驗證檢查。
- `InputValidations` 元素具有一或多個 `InputValidation` 元素。  每個 `InputValidation` 都是使用一系列的 `Predicate` 元素來建構。 這個元素可讓您執行布林值的彙總 (類似於 `and` 和 `or`)。
- `ClaimsSchema` 會定義要驗證的宣告。  然後它會定義要用來驗證該宣告的 `InputValidation` 規則。

### <a name="defining-a-predicate-element"></a>定義述詞元素

述詞有兩個方法的類型：IsLengthRange 或 MatchesRegex。 讓我們檢閱每一個的範例。  首先，我們要看 MatchesRegex，是用來比對規則運算式的範例。  在此範例中，它必須符合包含數字的字串。

```XML
      <Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be a pin.">
        <Parameters>
          <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
        </Parameters>
      </Predicate>
```

接著，我們要檢閱 IsLengthRange 的範例。  這個方法會採用最小和最大的字串長度。

```XML
      <Predicate Id="Length" Method="IsLengthRange" HelpText="The password must be between 8 and 16 characters.">
        <Parameters>
          <Parameter Id="Minimum">8</Parameter>
          <Parameter Id="Maximum">16</Parameter>
        </Parameters>
      </Predicate>
```

如果檢查失敗，可使用 `HelpText` 屬性來提供錯誤訊息給使用者。  可以使用[語言自訂功能](active-directory-b2c-reference-language-customization.md)將這個字串當地語系化。

### <a name="defining-an-inputvalidation-element"></a>定義 InputValidation 元素

`InputValidation` 是 `PredicateReferences` 的彙總。 每個 `PredicateReferences` 都必須為 true，`InputValidation` 才會成功。  不過，`PredicateReferences` 元素內部會使用名為 `MatchAtLeast`的屬性來指定有幾個 `PredicateReference` 檢查必須傳回 true。  選擇性地定義 `HelpText` 屬性，可覆寫其所參考之 `Predicate` 元素中的錯誤訊息。

```XML
      <InputValidation Id="PasswordValidation">
        <PredicateReferences Id="LengthGroup" MatchAtLeast="1">
          <PredicateReference Id="Length" />
        </PredicateReferences>
        <PredicateReferences Id="3of4" MatchAtLeast="3" HelpText="You must have at least 3 of the following character classes:">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </InputValidation>
```

### <a name="defining-a-claimsschema-element"></a>定義 ClaimsSchema 元素

宣告類型 `newPassword` 和 `reenterPassword` 會視為特殊的，因此不要變更這些名稱。  UI 會以這些 `ClaimType` 元素作為基礎，驗證使用者是否在帳戶建立期間正確地重新輸入其密碼。  若要尋找相同的 `ClaimType` 元素，請在入門套件中查看 TrustFrameworkBase.xml。  此範例中的最新消息是我們要覆寫這些元素來定義 `InputValidationReference`。 這個新元素的 `ID` 屬性指向我們所定義的 `InputValidation` 元素。

```XML
    <ClaimsSchema>
      <ClaimType Id="newPassword">
        <InputValidationReference Id="PasswordValidation" />
      </ClaimType>
      <ClaimType Id="reenterPassword">
        <InputValidationReference Id="PasswordValidation" />
      </ClaimType>
    </ClaimsSchema>
```

### <a name="putting-it-all-together"></a>總整理

這個範例會說明如何將所有部分拼湊在一起來形成工作原則。  若要使用此範例：

1. 遵循[使用者入門](active-directory-b2c-get-started-custom.md)中必要條件的指示，下載、設定及上傳 TrustFrameworkBase.xml 和 TrustFrameworkExtensions.xml
1. 使用本節中的範例內容來建立 SignUporSignIn.xml 檔案。
1. 將 `yourtenant` 取代為您的 Azure AD B2C 租用戶名稱以更新 SignUporSignIn.xml。
1. 最後上傳 SignUporSignIn.xml 原則檔案。

這個範例包含驗證 PIN 密碼，另一個則是用於強式密碼：

- 尋找 `PINpassword`。 這個 `InputValidation` 元素會驗證任何長度的 PIN。  目前不會使用它，因為 `ClaimType` 內的 `InputValidationReference` 元素不會參考它。 
- 尋找 `PasswordValidation`。 這項 `InputValidation` 元素驗證的密碼為 8 到 16 個字元，並包含 3 到 4 個數字、大寫、小寫或符號。  會在 `ClaimType` 參考它。  因此，此原則中會強制執行這個規則。

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="yourtenant.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin"
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_signup_signin">
 <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <BuildingBlocks>
    <ClaimsSchema>
      <ClaimType Id="newPassword">
        <InputValidationReference Id="PasswordValidation" />
      </ClaimType>
      <ClaimType Id="reenterPassword">
        <InputValidationReference Id="PasswordValidation" />
      </ClaimType>
    </ClaimsSchema>
    <Predicates>
      <Predicate Id="Lowercase" Method="MatchesRegex" HelpText="a lowercase">
        <Parameters>
          <Parameter Id="RegularExpression">[a-z]+</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Uppercase" Method="MatchesRegex" HelpText="an uppercase">
        <Parameters>
          <Parameter Id="RegularExpression">[A-Z]+</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Number" Method="MatchesRegex" HelpText="a number">
        <Parameters>
          <Parameter Id="RegularExpression">[0-9]+</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Symbol" Method="MatchesRegex" HelpText="a symbol">
        <Parameters>
          <Parameter Id="RegularExpression">[!@#$%^*()]+</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Length" Method="IsLengthRange" HelpText="The password must be between 8 and 16 characters.">
        <Parameters>
          <Parameter Id="Minimum">8</Parameter>
          <Parameter Id="Maximum">16</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be a pin.">
        <Parameters>
          <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
        </Parameters>
      </Predicate>
    </Predicates>
    <InputValidations>
      <InputValidation Id="PasswordValidation">
        <PredicateReferences Id="LengthGroup" MatchAtLeast="1">
          <PredicateReference Id="Length" />
        </PredicateReferences>
        <PredicateReferences Id="3of4" MatchAtLeast="3" HelpText="You must have at least 3 of the following character classes:">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </InputValidation>
      <InputValidation Id="PINpassword">
        <PredicateReferences Id="PINGroup">
          <PredicateReference Id="PIN" />
        </PredicateReferences>
      </InputValidation>
    </InputValidations>
  </BuildingBlocks>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration, DisableStrongPassword" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```
