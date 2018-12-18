---
title: TrustFrameworkPolicy - Azure Active Directory B2C | Microsoft Docs
description: 指定 Azure Active Directory B2C 中自訂原則的 TrustFrameworkPolicy 元素。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 16e98811b65e215d8688e030ea8dcbb1f9446a5b
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44382670"
---
# <a name="trustframeworkpolicy"></a>TrustFrameworkPolicy

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

自訂原則以一或多個 XML 格式的檔案表示，各檔案在階層鏈中彼此參考。 XML 元素會定義原則的元素，例如宣告結構描述、宣告轉換、內容定義、宣告提供者、技術設定檔、使用者旅程圖，和協調流程步驟。 每個原則檔定義在原則檔之 **TrustFrameworkPolicy** 元素的最上層。 

```XML
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
  ...
```


**TrustFrameworkPolicy** 元素包含下列屬性：

| 屬性 | 必要 | 說明 |
|---------- | -------- | ----------- |
| PolicySchemaVersion | 是 | 用來執行此原則的結構描述版本。 值必須是 `0.3.0.0` |
| TenantObjectId | 否 | Azure Active Directory (Azure AD) B2C 租用戶的唯一物件識別碼。 |
| TenantId | 是 | 此原則所屬之租用戶的唯一識別碼。 |
| PolicyId | 是 | 原則的唯一識別碼。 此識別碼必須加上前置詞 *B2C_1A_* |
| PublicPolicyUri | 是 | 原則的 URI，也就是租用戶識別碼和原則識別碼的組合。 |
| DeploymentMode | 否 | 可能的值：`Production`、`Debugging` 或 `Development`。 `Production` 為預設值。 使用此屬性以偵錯您的原則。 如需詳細資訊，請參閱[收集記錄](active-directory-b2c-troubleshoot-custom.md)。 |
| UserJourneyRecorderEndpoint | 否 | 當 **DeploymentMode** 設為 `Development` 時，可使用端點。 值必須是 `urn:journeyrecorder:applicationinsights`。 如需詳細資訊，請參閱[收集記錄](active-directory-b2c-troubleshoot-custom.md)。 |


下列範例顯示如何指定 **TrustFrameworkPolicy** 元素：

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="http://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkBase"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

## <a name="inheritance-model"></a>繼承模型

這些類型的原則檔通常用於使用者旅程圖中：

- **基底**檔案，包含大部分的定義。 為了有利於原則的疑難排解和長期維護，建議您盡可能不要變更這個檔案。
- **擴充**檔案，其中保存租用戶的唯一組態變更。 此原則檔衍生自基底檔案。 使用此檔案以新增新的功能，或覆寫現有功能。 例如，使用此檔案與新的識別提供者形成同盟。
- **信賴憑證者 (RP)** 檔案，這是以單一工作為主的檔案，直接由信賴憑證者叫用，例如 Web、行動或桌面應用程式。 各項唯一且不重複的工作，例如註冊或登入、密碼重設，或設定檔編輯，均需要專屬的 RP 原則檔。 此原則檔衍生自擴充檔案。 

信賴憑證者應用程式會呼叫 RP 原則檔來執行特定工作。 起始登入流程即是一例。 Azure AD B2C 中的識別體驗架構會先從基底原則檔開始，再到擴充原則檔和 RP 原則檔，逐步新增所有元素，以組合目前生效的原則。 RP 檔案會覆寫擴充原則檔中相同類型和名稱的元素，而擴充原則檔會覆寫基底原則檔。 下圖顯示原則檔和信賴憑證者應用程式之間的關聯性。

![繼承模型](./media/trustframeworkpolicy/custom-policy-Inheritance-model.png)

繼承模型如下所示：

- 父原則和子原則皆屬於相同的結構描述。
- 任何層級的子原則可以繼承自父原則，並藉由新增新元素以進行擴充。
- 層級數沒有限制。

如需詳細資訊，請參閱[開始使用自訂原則](active-directory-b2c-get-started-custom.md)。

## <a name="base-policy"></a>基底原則

若要從另一個原則繼承原則，必須在原則檔的 **TrustFrameworkPolicy** 元素下，宣告 **BasePolicy** 元素。 **BasePolicy** 元素是從中衍生此原則之基底原則的參考。  

**BasePolicy** 元素包含下列元素：

| 元素 | 發生次數 | 說明 |
| ------- | ----------- | --------|
| TenantId | 1:1 | Azure AD B2C 租用戶的識別碼。 |
| PolicyId | 1:1 | 父代原則的識別碼。 |


下列範例顯示如何指定基底原則。 此 **B2C_1A_TrustFrameworkExtensions** 元則衍生自 **B2C_1A_TrustFrameworkBase** 原則。 

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="http://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkExtensions"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions">

  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase</PolicyId>
  </BasePolicy>
  ...
</TrustFrameworkPolicy>
```

## <a name="policy-execution"></a>原則執行

信賴憑證者應用程式 (例如 Web、行動或桌面應用程式) 會呼叫[信賴憑證者 (RP) 原則](relyingparty.md)。 RP 原則檔會執行特定工作，例如登入、重設密碼或編輯設定檔。 RP 原則會設定信賴憑證者應用程式收到的宣告清單，做為已簽發之權杖的一部分。 多個應用程式可使用相同的原則。 所有應用程式都會透過宣告接收相同的權杖，而使用者會經歷相同的使用者旅程圖。 單一應用程式可使用多個原則。

在 RP 原則檔中，指定 **DefaultUserJourney** 元素，該元素指向 [UserJourney](userjourneys.md)。 通常是在基底或擴充原則中定義使用者旅程圖。

B2C_1A_signup_signin 原則：

```XML
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn">
  ...
```

B2C_1A_TrustFrameWorkBase 或 B2C_1A_TrustFrameworkExtensionPolicy：

```XML
<UserJourneys>
  <UserJourney Id="SignOrSignIn">
  ...
```

使用者旅程圖定義使用者經歷的商務邏輯。 每個使用者旅程圖都是一組協調流程步驟，依照驗證和資訊集合的順序，執行一系列動作。 

[starter pack](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-get-started-custom#download-starter-pack-and-modify-policies)中的 **SocialAndLocalAccounts** 原則檔包含 SignUpOrSignIn、ProfileEdit、PasswordReset 使用者旅程圖。 您可以針對其他情況新增更多使用者旅程圖，例如變更電子郵件地址、連結和取消連結社交帳戶，或重設密碼。 

協調流程步驟可呼叫[技術設定檔](technicalprofiles.md)。 技術設定檔提供一個架構，其中包含與不同類型的對象進行通訊的內建機制。 例如，技術設定檔可以執行以下動作：

- 轉譯使用者體驗。
- 讓使用者使用社交或企業帳戶登入，例如 Facebook、Microsoft 帳戶、Google、Salesforce 或任何其他識別提供者。
- 設定多重要素驗證的電話驗證。
- 從 Azure AD B2C 身分識別存放區讀取和寫入資料。
- 呼叫自訂的 Restful API 服務。

![原則執行](./media/trustframeworkpolicy/custom-policy-execution.png)

 **TrustFrameworkPolicy** 元素包含下列元素：

- 如上所述的 BasePolicy
- [BuildingBlocks](buildingblocks.md)
- [ClaimsProviders](claimsproviders.md)
- [UserJourneys](userjourneys.md)
- [RelyingParty](relyingparty.md)

