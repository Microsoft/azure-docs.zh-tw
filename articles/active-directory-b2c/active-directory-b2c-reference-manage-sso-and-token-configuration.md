---
title: 在 Azure Active Directory B2C 中使用自訂原則來管理 SSO 和權杖自訂 | Microsoft Docs
description: 深入瞭解使用自訂原則來管理 SSO 和權杖自訂。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/02/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 811fb8b2de59c9d324ab4acb8b0f51b4cec80aee
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441792"
---
# <a name="azure-active-directory-b2c-manage-sso-and-token-customization-with-custom-policies"></a>Azure Active Directory B2C：使用自訂原則來管理 SSO 和權杖自訂
使用自訂原則所提供給您的控制項和透過內建原則所取到的控制項相同，均可讓您控制權杖、工作階段和單一登入 (SSO) 組態。  若要了解每個設定的功用，請參閱[這裡](#active-directory-b2c-token-session-sso)的文件。

## <a name="token-lifetimes-and-claims-configuration"></a>權杖存留期和宣告組態
若要變更權杖存留期的設定，您必須在想要影響之原則的信賴憑證者中，新增 `<ClaimsProviders>`元素。  `<ClaimsProviders>` 元素是 `<TrustFrameworkPolicy>` 的子系。  您必須在其中放入會影響權杖存留期的資訊。  XML 看起來像這個範例：

```XML
<ClaimsProviders>
   <ClaimsProvider>
      <DisplayName>Token Issuer</DisplayName>
      <TechnicalProfiles>
         <TechnicalProfile Id="JwtIssuer">
            <Metadata>
               <Item Key="token_lifetime_secs">3600</Item>
               <Item Key="id_token_lifetime_secs">3600</Item>
               <Item Key="refresh_token_lifetime_secs">1209600</Item>
               <Item Key="rolling_refresh_token_lifetime_secs">7776000</Item>
               <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
               <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
            </Metadata>
         </TechnicalProfile>
      </TechnicalProfiles>
   </ClaimsProvider>
</ClaimsProviders>
```

**存取權杖存留期**：使用 Key="token_lifetime_secs" (以秒為單位) 修改 `<Item>` 內的值，即可變更存取權杖存留期。  內建的預設值為 3600 秒 (60 分鐘)。

**識別碼權杖存留期**：使用 Key="id_token_lifetime_secs" (以秒為單位) 修改 `<Item>` 內的值，即可變更識別碼權杖存留期。  內建的預設值為 3600 秒 (60 分鐘)。

**重新整理權杖存留期**：使用 Key="refresh_token_lifetime_secs" (以秒為單位) 修改 `<Item>` 內的值，即可變更重新整理權杖存留期。  內建的預設值為 1209600 秒 (14 天)。

**重新整理權杖滑動視窗存留期**：如果您想要對重新整理權杖設定滑動視窗存留期，請使用 Key="rolling_refresh_token_lifetime_secs" (以秒為單位) 修改 `<Item>` 內的值。  內建的預設值為 7776000 (90 天)。  如果您不想強制執行滑動視窗存留期，請將此行改為：
```XML
<Item Key="allow_infinite_rolling_refresh_token">True</Item>
```

**簽發者 (iss) 宣告**：如果您想要變更簽發者 (iss) 宣告，請使用 Key="IssuanceClaimPattern" 修改 `<Item>` 內的值。  適用的值為 `AuthorityAndTenantGuid` 和 `AuthorityWithTfp`。

**設定代表原則識別碼的宣告**：用來設定此值的選項為 TFP (信任架構原則) 和 ACR (驗證內容參考)。  
我們的建議是將此值設定為 TFP，若要這樣做，請確定 Key="AuthenticationContextReferenceClaimPattern" 的 `<Item>` 存在，且值為 `None`。
在您的 `<OutputClaims>` 項目中，新增這個元素︰
```XML
<OutputClaim ClaimTypeReferenceId="trustFrameworkPolicy" Required="true" DefaultValue="{policy}" />
```
若設定為 ACR，則請移除 Key="AuthenticationContextReferenceClaimPattern" 的 `<Item>`。

**主體 (sub) 宣告**：此選項的預設值為 ObjectID，如果您想要將此值切換為 `Not Supported`，請執行下列動作：

換掉此行 
```XML
<OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
```
改用這行︰
```XML
<OutputClaim ClaimTypeReferenceId="sub" />
```

## <a name="session-behavior-and-sso"></a>工作階段行為和 SSO

若要變更工作階段行為和 SSO 組態，您必須在 `<RelyingParty>` 元素內新增 `<UserJourneyBehaviors>` 元素。  `<UserJourneyBehaviors>` 元素必須緊接在 `<DefaultUserJourney>` 之後。  `<UserJourneyBehavors>` 元素的內部看起來應該像這樣︰

```XML
<UserJourneyBehaviors>
   <SingleSignOn Scope="Application" />
   <SessionExpiryType>Absolute</SessionExpiryType>
   <SessionExpiryInSeconds>86400</SessionExpiryInSeconds>
</UserJourneyBehaviors>
```
**單一登入 (SSO) 設定**：若要變更單一登入設定，您必須修改 `<SingleSignOn>` 的值。  適用的值為 `Tenant`、`Application`、`Policy` 和 `Disabled`。 

**Web 應用程式工作階段存留期 (分鐘)**：若要變更 Web 應用程式工作階段存留期，您必須修改 `<SessionExpiryInSeconds>` 元素的值。  內建原則的預設值為 86400 秒 (1440 分鐘)。

**Web 應用程式工作階段逾時**：若要變更 Web 應用程式工作階段逾時，您必須修改 `<SessionExpiryType>` 的值。  適用的值為 `Absolute` 和 `Rolling`。
