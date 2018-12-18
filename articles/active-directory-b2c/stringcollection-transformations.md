---
title: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的 StringCollection 宣告轉換範例 | Microsoft Docs
description: 適用於 Azure Active Directory B2C 識別體驗架構結構描述的 StringCollection 宣告轉換範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 924da8c92bad1c5345d5d1833723a0c4e8ad7d29
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432654"
---
# <a name="stringcollection-claims-transformations"></a>StringCollection 宣告轉換

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

此文章提供在 Azure Active Directory (Azure AD) B2C 中，使用識別體驗架構結構描述 StringCollection 宣告轉換的範例。 如需詳細資訊，請參閱 [ClaimsTransformations](claimstransformations.md)。

## <a name="additemtostringcollection"></a>AddItemToStringCollection

將字串宣告新增至新的 stringCollection 宣告。 

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | item | 字串 | 要新增至輸出宣告的 ClaimType。 |
| InputClaim | collection | stringCollection | [選擇性] 如果指定，宣告轉換就會複製此集合中的項目，並將項目新增至輸出集合宣告的結尾。 |
| OutputClaim | collection | stringCollection | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 |

使用此宣告轉換來將字串新增至新的或現有的 stringCollection。 它通常用於 **AAD-UserWriteUsingAlternativeSecurityId** 技術設定檔。 建立新的社交帳戶之前，**CreateOtherMailsFromEmail** 宣告轉換會讀取 ClaimType，並將值新增至 **otherMails** ClaimType。 

下列宣告轉換會將 **email** ClaimType 新增至 **otherMails** ClaimType。

```XML
<ClaimsTransformation Id="CreateOtherMailsFromEmail" TransformationMethod="AddItemToStringCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="item" />
    <InputClaim ClaimTypeReferenceId="otherMails" TransformationClaimType="collection" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="otherMails" TransformationClaimType="collection" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **collection**：["someone@outlook.com"]
    - **item**："admin@contoso.com"
- 輸出宣告： 
    - **collection**：["someone@outlook.com", "admin@contoso.com"]

## <a name="addparametertostringcollection"></a>AddParameterToStringCollection

將字串參數新增至新的 stringCollection 宣告。 

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | collection | stringCollection | [選擇性] 如果指定，宣告轉換就會複製此集合中的項目，並將項目新增至輸出集合宣告的結尾。 |
| InputParameter | item | 字串 | 要新增至輸出宣告的值。 |
| OutputClaim | collection | stringCollection | 叫用此 ClaimsTransformation 之後將產生的 ClaimType。 |

使用此宣告轉換來將字串值新增至新的或現有的 stringCollection。 下列範例會將常數的電子郵件地址 (admin@contoso.com) 新增至 **otherMails** 宣告。 

```XML
<ClaimsTransformation Id="SetCompanyEmail" TransformationMethod="AddParameterToStringCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="otherMails" TransformationClaimType="collection" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="item" DataType="string" Value="admin@contoso.com" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="otherMails" TransformationClaimType="collection" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **collection**：["someone@outlook.com"]
- 輸入參數 
    - **item**："admin@contoso.com"
- 輸出宣告：
    - **collection**：["someone@outlook.com", "admin@contoso.com"]

## <a name="getsingleitemfromstringcollection"></a>GetSingleItemFromStringCollection

從提供的字串集合中取得第一個項目。 

| Item | TransformationClaimType | 資料類型 | 注意 |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | collection | stringCollection | 宣告轉換用來取得項目的 ClaimType。 |
| OutputClaim | extractedItem | 字串 | 叫用此 ClaimsTransformation 之後所產生的 ClaimType。 集合中的第一個項目。 |

下列範例會讀取 **otherMails** 宣告，並將第一個項目傳回到 **email** 宣告。 

```XML
<ClaimsTransformation Id="CreateEmailFromOtherMails" TransformationMethod="GetSingleItemFromStringCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="otherMails" TransformationClaimType="collection" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="extractedItem" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>範例

- 輸入宣告：
    - **collection**["someone@outlook.com", "someone@contoso.com"]
- 輸出宣告： 
    - **extractedItem**："someone@outlook.com"

