---
title: 當地語系化 - Azure Active Directory B2C | Microsoft Docs
description: 指定 Azure Active Directory B2C 中自訂原則的 Localization 元素。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e9442302b8d15a3a6a4c9fe148b48845b3535204
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382449"
---
# <a name="localization"></a>當地語系化

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**Localization** 元素可讓您在使用者旅程圖的適用原則中支援多個地區設定或語言。 原則中的當地語系化支援能夠：

- 設定原則中可支援語言的明確清單，然後選擇預設語言。
- 提供特定語言的字串和集合。

```XML
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
  <LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedResources Id="api.localaccountsignup.es">
  ...
```

**Localization** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| 已啟用 | 否 | 可能的值：`true` 或 `false`。 |

**Localization** 元素包含下列 XML 元素

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| SupportedLanguages | 1:n | 支援語言清單。 | 
| LocalizedResources | 0:n | 當地語系化資源的清單。 |

## <a name="supportedlanguages"></a>SupportedLanguages

**SupportedLanguages** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| DefaultLanguage | 是 | 用來做為當地語系化資源的預設語言。 |
| MergeBehavior | 否 | 列舉值，這些值是與具有相同識別碼之父代原則中存在的 ClaimType 合併的值。 在覆寫基本原則中指定的宣告時，請使用這個屬性。 可能的值：`Append`、`Prepend` 或 `ReplaceAll`。 `Append` 值指定應該在父代原則中指定的集合結尾後，附加上存在的資料集合。 `Prepend` 值指定應該在父代原則中指定的集合之前，新增存在的資料集合。 `ReplaceAll` 值指定應該忽略父代原則中定義的資料集合，而改用目前原則中定義的資料。 |

### <a name="supportedlanguages"></a>SupportedLanguages

**SupportedLanguages** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| SupportedLanguage | 1:n | 根據 RFC 5646 顯示符合語言標記的內容 - 識別語言的標記。 | 

## <a name="localizedresources"></a>LocalizedResources

**LocalizedResources** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| id | 是 | 用來唯一識別當地語系化資源的識別碼。 |

**LocalizedResources** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| LocalizedCollections | 0:n | 定義不同文化中的整個集合。 對於不同的文化特性，集合可以有不同數量的項目和不同的字串。 集合範例會包含宣告類型中出現的列舉。 例如在下拉式清單中，會向使用者顯示國家/地區清單。 |
| LocalizedStrings | 0:n | 定義各種文化特性中的所有字串，出現在集合中的字串除外。 |

### <a name="localizedcollections"></a>LocalizedCollections

**LocalizedCollections** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| LocalizedCollection | 1:n | 支援語言清單。 |

#### <a name="localizedcollection"></a>LocalizedCollection

**LocalizedCollection** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ElementType | 是 | 參考原則檔中的 ClaimType 元素或使用者介面元素。 |
| ElementId | 是 | 此字串包含對 ClaimsSchema 區段中已定義的宣告類型，如果 **ElementType** 設為 ClaimType，則會使用該字串。 |
| TargetCollection | 是 | 目標集合。 |

**LocalizedCollection** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| Item | 0:n | 定義在使用者介面中可供使用者選取的某個宣告選項，例如下拉式清單中的值。 |

**Item** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| 文字 | 是 | 此選項應會在使用者介面中向使用者顯示的易記顯示字串。 |
| 值 | 是 | 與選取此選項相關聯的字串宣告值。 |

下列範例顯示 **LocalizedCollections** 元素的用法。 其包含兩個 **LocalizedCollection** 元素，一個適用於英文，另一個適用於西班牙文。 兩者皆會使用英文和西班牙文的項目清單，設定宣告 `Gender` 的**限制**集合。

```XML
<LocalizedResources Id="api.selfasserted.en">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Female" Value="F" />
      <Item Text="Male" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>

<LocalizedResources Id="api.selfasserted.es">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Femenino" Value="F" />
      <Item Text="Masculino" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>

```

### <a name="localizedstrings"></a>LocalizedStrings

**LocalizedStrings** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| LocalizedString | 1:n | 當地語系化的字串。 |

**LocalizedString** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ElementType | 是 | 參考原則中的宣告類型元素或使用者介面元素。 可能的值：`ClaimType`、`UxElement`、`ErrorMessage`、`Predicate` 或  。 `ClaimType` 值用於將其中一個宣告屬性當地語系化，如 StringId 中所指定。 `UxElement` 值用於將其中一個使用者介面元素當地語系化，如 StringId 中所指定。 `ErrorMessage` 值用於將其中一個系統錯誤訊息當地語系化，如 StringId 中所指定。 `Predicate` 值用於將其中一個 [Predicate](predicates.md) 錯誤訊息當地語系化，如 StringId 中所指定。 `InputValidation` 值用於將其中一個 [PredicateValidation](predicates.md) 群組錯誤訊息當地語系化，如 StringId 中所指定。 |
| ElementId | 是 | 如果 **ElementType** 設為 `ClaimType``Predicate` 或 `InputValidation`，則此元素會包含對 ClaimsSchema 區段中已定義之宣告類型的參考。 | 
| StringId | 是 | 如果 **ElementType** 設為 `ClaimType`，則此元素會包含對宣告類型之屬性的參考。 可能的值：`DisplayName`、`AdminHelpText` 或 `PatternHelpText`。 `DisplayName` 值用於設定宣告顯示名稱。 `AdminHelpText` 值用於設定宣告使用者的說明文字名稱。 `PatternHelpText` 值用於設定宣告模式說明文字。 如果 **ElementType** 設為 `UxElement`，則此元素會包含使用者介面元素之屬性的參考。 如果 **ElementType** 設為 `ErrorMessage`，則此元素會指定錯誤訊息的識別碼。 如需 `UxElement` 識別碼的完整清單，請參閱[當地語系化字串識別碼](localization-string-ids.md)。|


下列範例列出了當地語系化的註冊頁面。 前三個 **LocalizedString** 值設定的是宣告屬性。 第三個會變更繼續按鈕的值。 最後一個會變更錯誤訊息。

```XML
<LocalizedResources Id="api.selfasserted.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

下列範例顯示 **Predicate** 之當地語系化的 **UserHelpText**，其識別碼為 `IsLengthBetween8And64`。 以及 **PredicateGroup** 之當地語系化的 **UserHelpText**，其識別碼為 **PredicateValidation** 的 `CharacterClasses`，其識別碼為 `StrongPassword`。

```XML
<PredicateValidation Id="StrongPassword">
  <PredicateGroups>
    ...
    <PredicateGroup Id="CharacterClasses">
    ...
    </PredicateGroup>
  </PredicateGroups>
</PredicateValidation>

...

<Predicate Id="IsLengthBetween8And64" Method="IsLengthRange">
  ...
</Predicate>
...


<LocalizedString ElementType="InputValidation" ElementId="StrongPassword" StringId="CharacterClasses">The password must have at least 3 of the following:</LocalizedString>

<LocalizedString ElementType="Predicate" ElementId="IsLengthBetween8And64" StringId="HelpText">The password must be between 8 and 64 characters.</LocalizedString>              
```

## <a name="set-up-localization"></a>設定當地語系化

本文說明如何在使用者旅程圖的適用原則中，支援多個地區設定或語言。 當地語系化需要三個步驟：設定支援語言的明確清單、提供特定語言的字串與集合，以及編輯頁面的 ContentDefinition。

### <a name="set-up-the-explicit-list-of-supported-languages"></a>設定支援語言的明確清單

請在 **BuildingBlocks** 元素下，透過支援語言清單新增 **Localization** 元素。 下列範例示範如何定義英文 (預設) 和西班牙文的當地語系化支援：

```XML
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
</Localization>
```

### <a name="provide-language-specific-strings-and-collections"></a>提供特定語言的字串和集合 

請在關閉 **SupportedLanguages** 元素之後，在 **Localization** 元素內新增 **LocalizedResources** 元素。 替每個頁面 (內容定義) 與您想要支援的語言新增 **LocalizedResources** 元素。 若要自訂英文、西班牙文和法文的統一註冊或登入頁面、註冊頁面和多重要素驗證 (MFA) 頁面，請新增下列 **LocalizedResources** 元素。  
- 統一註冊或登入頁面 (英文) `<LocalizedResources Id="api.signuporsignin.en">`
- 統一註冊或登入頁面 (西班牙文) `<LocalizedResources Id="api.signuporsignin.es">`
- 統一註冊或登入頁面 (法文) `<LocalizedResources Id="api.signuporsignin.fr">` 
- 註冊 (英文) `<LocalizedResources Id="api.localaccountsignup.en">`
- 註冊 (西班牙文) `<LocalizedResources Id="api.localaccountsignup.es">`
- 註冊 (法文) `<LocalizedResources Id="api.localaccountsignup.fr">`
- 多重要素驗證 (英文) `<LocalizedResources Id="api.phonefactor.en">`
- 多重要素驗證 (西班牙文) `<LocalizedResources Id="api.phonefactor.es">`
- 多重要素驗證 (法文) `<LocalizedResources Id="api.phonefactor.fr">`

每個 **LocalizedResources** 元素包含所有必要的 **LocalizedStrings** 元素，其中有多個 **LocalizedString** 元素和 **LocalizedCollections** 元素，以及多個 **LocalizedCollection** 元素。  下列範例會新增註冊頁面英文當地語系化： 

注意：此範例會參考 `Gender` 和 `City` 宣告類型。 若要使用此範例，請務必定義這些宣告。 如需詳細資訊，請參閱 [ClaimsSchema](claimsschema.md)。

```XML
<LocalizedResources Id="api.localaccountsignup.en">

 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Female" Value="F" />
      <Item Text="Male" Value="M" />
    </LocalizedCollection>
   <LocalizedCollection ElementType="ClaimType" ElementId="City" TargetCollection="Restriction">
      <Item Text="New York" Value="NY" />
      <Item Text="Paris" Value="Paris" />
      <Item Text="London" Value="London" />
    </LocalizedCollection>
  </LocalizedCollections>

  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

西班牙文的註冊頁面當地語系化。

```XML
<LocalizedResources Id="api.localaccountsignup.es">

 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Femenino" Value="F" />
      <Item Text="Masculino" Value="M" />
    </LocalizedCollection>
   <LocalizedCollection ElementType="ClaimType" ElementId="City" TargetCollection="Restriction">
      <Item Text="Nueva York" Value="NY" />
      <Item Text="París" Value="Paris" />
      <Item Text="Londres" Value="London" />
    </LocalizedCollection>
  </LocalizedCollections>

  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Dirección de correo electrónico</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Dirección de correo electrónico que puede usarse para ponerse en contacto con usted.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Introduzca una dirección de correo electrónico.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Crear</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">Ya existe un usuario con el id. especificado. Elija otro diferente.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

### <a name="edit-the-contentdefinition-for-the-page"></a>編輯頁面的 ContentDefinition 

對於欲進行當地語系化的每個頁面，請指定要在 **ContentDefinition** 中尋找的語言代碼。

在下列範例中，英文 (en) 和西班牙文 (es) 自訂字串已新增至註冊頁面。 每個 **LocalizedResourcesReference** 的 **LocalizedResourcesReferenceId** 與其地區設定相同，但可使用任何字串做為識別碼。 至於每個語言和頁面組合，請指向先前建立的對應 **LocalizedResources**。

```XML
<ContentDefinition Id="api.localaccountsignup">
...
  <LocalizedResourcesReferences MergeBehavior="Prepend">
    <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
    <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
  </LocalizedResourcesReferences>
</ContentDefinition>
```

下列範例顯示的是最終的 XML：

```XML
<BuildingBlocks>
  <ContentDefinitions>
    <ContentDefinition Id="api.localaccountsignup">
      <!-- Other content definitions elements... -->
      <LocalizedResourcesReferences MergeBehavior="Prepend">
         <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
         <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
      </LocalizedResourcesReferences>
    </ContentDefinition>
    <!-- More content definitions... -->
  </ContentDefinitions>

  <Localization Enabled="true">

    <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
      <SupportedLanguage>en</SupportedLanguage>
      <SupportedLanguage>es</SupportedLanguage>
      <!-- More supported language... -->
    </SupportedLanguages>

    <LocalizedResources Id="api.localaccountsignup.en">
      <LocalizedCollections>
        <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
          <Item Text="Female" Value="F" />
          <Item Text="Male" Value="M" />
          <!-- More items... -->
        </LocalizedCollection>
        <LocalizedCollection ElementType="ClaimType" ElementId="City" TargetCollection="Restriction">
          <Item Text="New York" Value="NY" />
          <Item Text="Paris" Value="Paris" />
          <Item Text="London" Value="London" />
        </LocalizedCollection>
        <!-- More localized collections... -->
      </LocalizedCollections>
      <LocalizedStrings>
        <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
        <LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
       <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
        <!-- More localized strings... -->
      </LocalizedStrings>
    </LocalizedResources>

    <LocalizedResources Id="api.localaccountsignup.es">
      <LocalizedCollections>
       <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
          <Item Text="Femenino" Value="F" />
          <Item Text="Masculino" Value="M" />
        </LocalizedCollection>
        <LocalizedCollection ElementType="ClaimType" ElementId="City" TargetCollection="Restriction">
          <Item Text="Nueva York" Value="NY" />
          <Item Text="París" Value="Paris" />
          <Item Text="Londres" Value="London" />
        </LocalizedCollection>
      </LocalizedCollections>
      <LocalizedStrings>
        <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Dirección de correo electrónico</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Dirección de correo electrónico que puede usarse para ponerse en contacto con usted.</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Introduzca una dirección de correo electrónico.</LocalizedString>
        <LocalizedString ElementType="UxElement" StringId="button_continue">Crear</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">Ya existe un usuario con el id. especificado. Elija otro diferente.</LocalizedString>
      </LocalizedStrings>
    </LocalizedResources>
    <!-- More localized resources... -->
  </Localization>
</BuildingBlocks>
```




