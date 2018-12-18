---
title: TechnicalProfiles | Microsoft Docs
description: 指定 Azure Active Directory B2C 中自訂原則的 TechnicalProfiles 元素。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: f1242b299c6d2278bd75b576f225987854a2d8a5
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382700"
---
# <a name="technicalprofiles"></a>TechnicalProfiles

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**TechnicalProfiles** 元素包含受到宣告提供者支援的一組技術設定檔。 每個宣告提供者必須有一或多個技術設定檔，以決定與該宣告提供者進行通訊所需的端點以及通訊協定。 宣告提供者可以有多個技術設定檔。

```XML
<ClaimsProvider>
  <DisplayName>Display name</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Technical profile identifier">
      <DisplayName>Display name of technical profile</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">URL of service</Item>
        <Item Key="AuthenticationType">None</Item>
        <Item Key="SendClaimsIn">Body</Item>
      </Metadata>
      <InputTokenFormat>JWT</InputTokenFormat>
      <OutputTokenFormat>JWT</OutputTokenFormat>
      <CryptographicKeys>
        <Key ID="Key identifier" StorageReferenceId="Storage key container identifier"/>
        ...
      </CryptographicKeys>
      <InputClaimsTransformations>
        <InputClaimsTransformation ReferenceId="Claims transformation identifier" />
        ...
      <InputClaimsTransformations>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="givenName" DefaultValue="givenName" PartnerClaimType="firstName" />
        ...
      </InputClaims>
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="givenName" DefaultValue="givenName" PartnerClaimType="firstName" />
        ...
      </PersistedClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" DefaultValue="loyaltyNumber" PartnerClaimType="loyaltyNumber" />
      </OutputClaims>
      <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="Claims transformation identifier" />
        ...
      <OutputClaimsTransformations>
      <ValidationTechnicalProfiles>
        <ValidationTechnicalProfile ReferenceId="Technical profile identifier" />
        ...
      </ValidationTechnicalProfiles>
      <SubjectNamingInfo ClaimType="Claim type identifier" />
      <IncludeTechnicalProfile ReferenceId="Technical profile identifier" />
      <UseTechnicalProfileForSessionManagement ReferenceId="Technical profile identifier" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

**TechnicalProfile** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
|---------|---------|---------|
| id | 是 | 技術設定檔的唯一識別碼。 可使用此識別碼，從原則設定檔的其他元素參考此技術設定檔。 例如，**OrchestrationSteps** 和 **ValidationTechnicalProfile**。 |

**TechnicalProfile** 包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| 網域 | 0:1 | 技術設定檔的網域名稱。 例如，如果您的技術設定檔指定 Facebook 識別提供者，則網域名稱為 Facebook.com。 |
| DisplayName | 0:1 | 可向使用者顯示的技術設定檔名稱。 |
| 說明 | 0:1 | 可向使用者顯示的技術設定檔描述。 |
| 通訊協定 | 0:1 | 用於與其他合作對象進行通訊的通訊協定。 |
| 中繼資料 | 0:1 | 通訊協定使用的機碼值組，用於在交易過程中與端點通訊。 |
| InputTokenFormat | 0:1 | 輸入權杖的格式。 可能的值：`JSON`、`JWT`、`SAML11` 或 `SAML2`。 `JWT` 值代表以 IETF 規格為基礎的 JSON Web 權杖。 `SAML11` 值代表以 OASIS 規格為基礎的 SAML 1.1 安全性權杖。  `SAML2` 值代表以 OASIS 規格為基礎的 SAML 2.0 安全性權杖。 |
| OutputTokenFormat | 0:1 | 輸出權杖的格式。 可能的值：`JSON`、`JWT`、`SAML11` 或 `SAML2`。 |
| CryptographicKeys | 0:1 | 技術設定檔中所使用的密碼編譯金鑰清單。 |
| InputClaimsTransformations | 0:1 | 先前定義的宣告轉換參考清單，應在將任何宣告傳送至宣告提供者或信賴憑證者之前執行這些轉換。 |
| InputClaims | 0:1 | 先前定義的宣告類型參考清單，這類宣告類型會在技術設定檔中做為輸入來源。 |
| PersistedClaims | 0:1 | 先前定義的宣告類型參考清單，與技術設定檔相關的宣告提供者會保留這類宣告類型。 |
| OutputClaims | 0:1 | 先前定義的宣告類型參考清單，會在技術設定檔中輸出這類宣告類型。 |
| OutputClaimsTransformations | 0:1 | 先前定義的宣告轉換參考清單，應在來自宣告提供者的宣告受到接收後再執行。 |
| ValidationTechnicalProfiles | 0:n | 其他技術設定檔的參考清單，技術設定檔會用來進行用途的驗證。 如需詳細資訊，請參閱[驗證技術設定檔](validation-technical-profile.md)|
| SubjectNamingInfo | 0:1 | 控制權杖中主體名稱的產生，其中主體名稱與宣告分開指定。 例如，OAuth 或 SAML。  |
| IncludeClaimsFromTechnicalProfile | 0:1 | 技術設定檔的識別碼，您打算將所有輸入和輸出宣告加入此技術設定檔。 必須在相同原則檔中定義參考的技術設定檔。 | 
| IncludeTechnicalProfile |0:1 | 技術設定檔的識別碼，您打算將所有資料加入此技術設定檔。 參考的技術設定檔必須存在於相同的原則檔中。 |
| UseTechnicalProfileForSessionManagement | 0:1 | 用於工作階段管理的不同技術設定檔。 |
|EnabledForUserJourneys| 0:1 |控制是否可在使用者旅程圖中執行技術設定檔。  |

### <a name="protocol"></a>通訊協定

**Protocol** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| 名稱 | 是 | Azure AD B2C 所支援的有效通訊協定名稱，可用來作為技術設定檔的一部分。 可能的值：`OAuth1`、`OAuth2`、`SAML2`、`OpenIdConnect`、`WsFed`、`WsTrust`、`Proprietary`、`session management`、`self-asserted` 或 `None`。 |
| 處理常式 | 否 | 當通訊協定名稱設為 `Proprietary` 時，請指定 Azure AD B2C 用於判斷通訊協定處理常式之組件的完整名稱。 |

### <a name="metadata"></a>中繼資料

**Metadata** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| Item | 0:n | 與技術設定檔相關的中繼資料。 每一種技術設定檔有一組不同的中繼資料項目。 如需詳細資訊，請參閱技術設定檔類型章節。 |

#### <a name="item"></a>Item

**Metadata** 元素的 **Item** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| Key | 是 | 中繼資料索引鍵。 如需中繼資料項目的清單，請參閱每個技術設定檔類型。 |

### <a name="cryptographickeys"></a>CryptographicKeys

**CryptographicKeys** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| Key | 1:n | 此技術設定檔中使用的密碼編譯金鑰。 |

#### <a name="key"></a>Key

**Key** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| id | 否 | 原則檔中的其他元素所參考之特定金鑰組的唯一識別碼。 |
| StorageReferenceId | 是 | 原則檔中的其他元素所參考之儲存體金鑰容器的識別碼。 |

### <a name="inputclaimstransformations"></a>InputClaimsTransformations

**InputClaimsTransformations** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| InputClaimsTransformation | 1:n | 宣告轉換的識別碼，應在將任何宣告傳送至宣告提供者或信賴憑證者之前執行此轉換。 宣告轉換可用於修改現有的 ClaimsSchema 宣告或產生新的宣告。 |

#### <a name="inputclaimstransformation"></a>InputClaimsTransformation

**InputClaimsTransformation** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ReferenceId | 是 | 已在原則檔或父原則檔中定義之宣告轉換的識別碼。 |

### <a name="inputclaims"></a>InputClaims

**InputClaims** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | 預期的輸入宣告類型。 |

#### <a name="inputclaim"></a>InputClaim 

**InputClaim** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | 是 | 已在原則檔或父代原則檔之 ClaimsSchema 區段中定義的宣告類型識別碼。 |
| DefaultValue | 否 | 當 ClaimTypeReferenceId 指示的宣告不存在時，此預設值可用於建立宣告，以便產生的宣告可由技術設定檔用來作為 InputClaim。 |
| PartnerClaimType | 否 | 指定的原則宣告類型對應之外部夥伴的宣告類型識別碼。 如果未指定 PartnerClaimType 屬性，則指定的原則宣告類型會對應至相同名稱的夥伴宣告類型。 當您的宣告類型名稱不同於其他合作對象時，請使用這個屬性。 例如，第一個宣告名稱是 'givenName'，而夥伴會使用名稱為 'first_name' 的宣告。 |

### <a name="persistedclaims"></a>PersistedClaims

**PersistedClaims** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| PersistedClaim | 1:n | 要保存的宣告類型。 |

#### <a name="persistedclaim"></a>PersistedClaim 

**PersistedClaim** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | 是 | 已在原則檔或父代原則檔之 ClaimsSchema 區段中定義的宣告類型識別碼。 |
| DefaultValue | 否 | 當 ClaimTypeReferenceId 指示的宣告不存在時，此預設值可用於建立宣告，以便產生的宣告可由技術設定檔用來作為 InputClaim。 |
| PartnerClaimType | 否 | 指定的原則宣告類型對應之外部夥伴的宣告類型識別碼。 如果未指定 PartnerClaimType 屬性，則指定的原則宣告類型會對應至相同名稱的夥伴宣告類型。 當您的宣告類型名稱不同於其他合作對象時，請使用這個屬性。 例如，第一個宣告名稱是 'givenName'，而夥伴會使用名稱為 'first_name' 的宣告。 |

### <a name="outputclaims"></a>OutputClaims

**OutputClaims** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| OutputClaim | 1:n | 預期的輸出宣告類型。 |

#### <a name="outputclaim"></a>OutputClaim 

**OutputClaim** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | 是 | 已在原則檔或父代原則檔之 ClaimsSchema 區段中定義的宣告類型識別碼。 |
| DefaultValue | 否 | 當 ClaimTypeReferenceId 指示的宣告不存在時，此預設值可用於建立宣告，以便產生的宣告可由技術設定檔用來作為 InputClaim。 |
|AlwaysUseDefaultValue |否 |強制使用預設值。  |
| PartnerClaimType | 否 | 指定的原則宣告類型對應之外部夥伴的宣告類型識別碼。 如果未指定 PartnerClaimType 屬性，則指定的原則宣告類型會對應至相同名稱的夥伴宣告類型。 當您的宣告類型名稱不同於其他合作對象時，請使用這個屬性。 例如，第一個宣告名稱是 'givenName'，而夥伴會使用名稱為 'first_name' 的宣告。 |

### <a name="outputclaimstransformations"></a>OutputClaimsTransformations

**OutputClaimsTransformations** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| OutputClaimsTransformation | 1:n | 宣告轉換的識別碼，應在將任何宣告傳送至宣告提供者或信賴憑證者之前執行這些轉換。 宣告轉換可用於修改現有的 ClaimsSchema 宣告或產生新的宣告。 |

#### <a name="outputclaimstransformation"></a>OutputClaimsTransformation

**OutputClaimsTransformation** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ReferenceId | 是 | 已在原則檔或父原則檔中定義之宣告轉換的識別碼。 |

### <a name="validationtechnicalprofiles"></a>ValidationTechnicalProfiles

**ValidationTechnicalProfiles** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | 使用的技術設定檔識別碼可驗證參考技術設定檔的部分或所有輸出宣告。 參考技術設定檔的所有輸入宣告必須出現在參考技術設定檔的輸出宣告中。 |

#### <a name="validationtechnicalprofile"></a>ValidationTechnicalProfile

**ValidationTechnicalProfile** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ReferenceId | 是 | 已在原則檔或父原則檔中定義之技術設定檔的識別碼。 |

###  <a name="subjectnaminginfo"></a>SubjectNamingInfo

**SubjectNamingInfo** 包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ClaimType | 是 | 已在原則檔之 ClaimsSchema 區段中定義的宣告類型識別碼。 |

### <a name="includetechnicalprofile"></a>IncludeTechnicalProfile

**IncludeTechnicalProfile** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ReferenceId | 是 | 已在原則檔或父原則檔中定義之技術設定檔的識別碼。 |

### <a name="usetechnicalprofileforsessionmanagement"></a>UseTechnicalProfileForSessionManagement

**UseTechnicalProfileForSessionManagement** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
| --------- | -------- | ----------- |
| ReferenceId | 是 | 已在原則檔或父原則檔中定義之技術設定檔的識別碼。 |

### <a name="enabledforuserjourneys"></a>EnabledForUserJourneys
使用者旅程圖中的 **ClaimsProviderSelections** 會定義宣告提供者選項清單及其順序。 使用您篩選的 **EnabledForUserJourneys** 元素，其宣告提供者可供使用者使用。 **EnabledForUserJourneys** 元素包含下列其中一個值：

- **Always**，執行技術設定檔。
- **Never**，跳過技術設定檔。 
- **OnClaimsExistence**，僅於存在技術設定檔中指定的特定宣告時執行。 
- **OnItemExistenceInStringCollectionClaim**，僅在項目存在於字串集合宣告中時執行。 
- **OnItemAbsenceInStringCollectionClaim**，僅在項目存在於字串集合宣告中時執行。

使用 **OnClaimsExistence****OnItemExistenceInStringCollectionClaim** 或 **OnItemAbsenceInStringCollectionClaim**，要求您提供下列中繼資料：**ClaimTypeOnWhichToEnable** 可指定要評估的宣告類型，**ClaimValueOnWhichToEnable** 可指定要比較的值。

只有在 **identityProviders** 字串集合包含 `facebook.com` 的值時，才會執行下列技術設定檔：

```XML
<TechnicalProfile Id="UnLink-Facebook-OAUTH">
  <DisplayName>Unlink Facebook</DisplayName>
...
    <Metadata>
        <Item Key="ClaimTypeOnWhichToEnable">identityProviders</Item>
        <Item Key="ClaimValueOnWhichToEnable">facebook.com</Item>
    </Metadata>        
...
  <EnabledForUserJourneys>OnItemExistenceInStringCollectionClaim</EnabledForUserJourneys>
</TechnicalProfile>  
```












