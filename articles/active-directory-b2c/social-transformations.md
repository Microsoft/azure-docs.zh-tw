---
title: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的社交帳戶宣告轉換範例 | Microsoft Docs
description: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的社交帳戶宣告轉換範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: d9b592e7f61b87860e4f6fa2aa4d46e253b6257e
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382681"
---
# <a name="social-accounts-claims-transformations"></a>社交帳戶宣告轉換

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

在 Azure Active Directory (Azure AD) B2C 中，社交帳戶識別儲存在 **alternativeSecurityIdCollection** 宣告類型的 `userIdentities` 屬性中。 **alternativeSecurityIdCollection** 中的每個項目會指定簽發者 (識別提供者名稱，例如 facebook.com)，以及 `issuerUserId`，這是簽發者的唯一使用者識別碼。 

```JSON
"userIdentities": [{
    "issuer": "google.com",
    "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw"
  },
  {
    "issuer": "facebook.com",
    "issuerUserId": "MTIzNDU="
  }]
```

本文會提供數個使用範例，介紹 Azure AD B2C 中，識別體驗架構結構描述的社交帳戶宣告轉換。 如需詳細資訊，請參閱 [ClaimsTransformations](claimstransformations.md)。

## <a name="createalternativesecurityid"></a>CreateAlternativeSecurityId

建立使用者 alternativeSecurityId 屬性的 JSON 表示法，該屬性可用於對 Azure Active Directory 進行呼叫。 如需詳細資訊，請參閱 [AlternativeSecurityId's schema](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#AlternativeSecurityIdType)。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | 索引鍵 | 字串 | 此 ClaimType 可指定社交識別提供者所使用的唯一使用者識別碼。 |
| InputClaim | identityProvider | 字串 | 此 ClaimType 可指定社交帳戶識別提供者名稱，例如 facebook.com。 |
| OutputClaim | alternativeSecurityId | 字串 | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 包含社交帳戶使用者的身分識別相關資訊。 **issuer** 是 `identityProvider` 宣告的值。 **issuerUserId** 是 `key` base64 格式的值。 |

使用此宣告轉換以產生 `alternativeSecurityId` ClaimType。 該宣告類型由所有社交識別提供者技術設定檔使用，例如 `Facebook-OAUTH`。 下列宣告轉換會收到使用者社交帳戶識別碼與識別提供者名稱。 此技術設定檔的輸出是可用於 Azure Active Directory 服務的 JSON 字串格式。  

```XML
<ClaimsTransformation Id="CreateAlternativeSecurityId" TransformationMethod="CreateAlternativeSecurityId">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="socialIdpUserId" TransformationClaimType="key" />
    <InputClaim ClaimTypeReferenceId="identityProvider" TransformationClaimType="identityProvider" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="alternativeSecurityId" TransformationClaimType="alternativeSecurityId" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **key**: 12334
    - **identityProvider**: Facebook.com
- 輸出宣告：
    - **alternativeSecurityId**: { "issuer": "facebook.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw"}

## <a name="additemtoalternativesecurityidcollection"></a>AddItemToAlternativeSecurityIdCollection

將 `AlternativeSecurityId` 加入 `alternativeSecurityIdCollection` 宣告。 

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | item | 字串 | 要加入輸出宣告的 ClaimType。 |
| InputClaim | collection | alternativeSecurityIdCollection | 宣告轉換使用的 ClaimTypes (如果在原則中可用)。 如果已提供，則宣告轉換會在集合結尾加入 `item`。 |
| OutputClaim | collection | alternativeSecurityIdCollection | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 新集合包含來自輸入 `collection` 和 `item` 的項目。 |

下列範例會連結新的社交身分識別與現有帳戶。 若要連結新的社交身分識別： 
1. 在 **AAD-UserReadUsingAlternativeSecurityId** 和 **AAD-UserReadUsingObjectId** 技術設定檔中，輸出使用者的 **alternativeSecurityIds** 宣告。
1. 要求使用者使用與該使用者無關之其中一個識別提供者進行登入。 
1. 使用 **CreateAlternativeSecurityId** 宣告轉換，建立名稱為 `AlternativeSecurityId2` 的新 **alternativeSecurityId** 宣告類型 
1. 呼叫 **AddItemToAlternativeSecurityIdCollection** 宣告轉換，以將 **AlternativeSecurityId2** 宣告加入現有的 **AlternativeSecurityIds** 宣告。 
1. 將 **alternativeSecurityIds** 宣告保存至使用者帳戶

```XML
<ClaimsTransformation Id="AddAnotherAlternativeSecurityId" TransformationMethod="AddItemToAlternativeSecurityIdCollection">
  <InputClaims>
      <InputClaim ClaimTypeReferenceId="AlternativeSecurityId2" TransformationClaimType="item" />
      <InputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
  </InputClaims>
  <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **item**: { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" }
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" } ]
- 輸出宣告：
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]

## <a name="getidentityprovidersfromalternativesecurityidcollectiontransformation"></a>GetIdentityProvidersFromAlternativeSecurityIdCollectionTransformation

將 **alternativeSecurityIdCollection** 宣告的簽發者清單傳回至新的 **stringCollection** 宣告。

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | alternativeSecurityIdCollection | alternativeSecurityIdCollection | 用於取得識別提供者 (簽發者) 清單的 ClaimType。 |
| OutputClaim | identityProvidersCollection | stringCollection | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 與 alternativeSecurityIdCollection 輸入宣告相關聯的識別提供者清單 |

下列宣告轉換會讀取使用者 **alternativeSecurityIds** 宣告，並擷取與該帳戶相關聯的識別提供者名稱清單。 使用輸出 **identityProvidersCollection**，向使用者顯示與該帳戶相關聯的識別提供者清單。 或者到識別提供者選取頁面中，根據輸出 **identityProvidersCollection** 宣告篩選識別提供者清單。 因此，使用者可以選擇連結尚未與該帳戶建立關聯的新社交身分識別。 

```XML
<ClaimsTransformation Id="ExtractIdentityProviders" TransformationMethod="GetIdentityProvidersFromAlternativeSecurityIdCollectionTransformation">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="alternativeSecurityIds" TransformationClaimType="alternativeSecurityIdCollection" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="identityProviders" TransformationClaimType="identityProvidersCollection" />
  </OutputClaims>
</ClaimsTransformation>
```

- 輸入宣告：
    - **alternativeSecurityIdCollection**: [ { "issuer": "google.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]
- 輸出宣告：
    - **identityProvidersCollection**: [ "facebook.com", "google.com" ]

## <a name="removealternativesecurityidbyidentityprovider"></a>RemoveAlternativeSecurityIdByIdentityProvider

將 **AlternativeSecurityId** 從 **alternativeSecurityIdCollection** 宣告中移除。 

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | identityProvider | 字串 | 包含要從集合中移除之識別提供者名稱的 ClaimType。 |
| InputClaim | collection | alternativeSecurityIdCollection | 宣告轉換所使用的 ClaimType。 宣告轉換會將 identityProvider 從集合中移除。 |
| OutputClaim | collection | alternativeSecurityIdCollection | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 從集合中移除 identityProvider 之後的新集合。 |

下列範例會將其中一個社交身分識別與現有帳戶取消連結。 取消連結社交身分識別： 
1. 在 **AAD-UserReadUsingAlternativeSecurityId** 和 **AAD-UserReadUsingObjectId** 技術設定檔中，輸出使用者的 **alternativeSecurityIds** 宣告。
2. 要求使用者選取要從與該使用者相關聯之識別提供者清單中移除的社交帳戶。 
3. 以識別提供者名稱呼叫宣告轉換技術設定檔，該設定檔會呼叫 **RemoveAlternativeSecurityIdByIdentityProvider** 宣告轉換，而該轉換會移除選取的社交身分識別。
4. 將 **alternativeSecurityIds** 宣告保存至使用者帳戶。

```XML
<ClaimsTransformation Id="RemoveAlternativeSecurityIdByIdentityProvider" TransformationMethod="RemoveAlternativeSecurityIdByIdentityProvider">
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="secondIdentityProvider" TransformationClaimType="identityProvider" />
        <InputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
    </InputClaims>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
    </OutputClaims>
</ClaimsTransformation>               
</ClaimsTransformations>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **identityProvider**: facebook.com
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]
- 輸出宣告：
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" } ]