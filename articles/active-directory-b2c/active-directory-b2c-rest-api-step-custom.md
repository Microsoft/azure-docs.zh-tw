---
title: 在 Azure Active Directory B2C 中使用 REST API 宣告交換作為協調流程步驟 | Microsoft Docs
description: 關於已與 API 整合之 Azure Active Directory B2C 自訂原則的主題。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/24/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: dddb42f53d4bb59113df937799bd4de10d31491c
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338774"
---
# <a name="walkthrough-integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journey-as-an-orchestration-step"></a>逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為協調流程步驟

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

構成 Azure Active Directory B2C (Azure AD B2C) 基礎的識別體驗架構 (IEF)，可讓身分識別開發人員於使用者旅程圖中整合與 RESTful API 的互動。  

在本逐步解說結束時，您將能建立與 RESTful 服務互動的 Azure AD B2C 使用者旅程圖。

IEF 會在宣告中傳送資料，並在宣告中收到傳回的資料。 REST API 宣告交換：

- 可以設計成協調流程步驟。
- 可以觸發外部動作。 例如，它可以在外部資料庫中記錄一個事件。
- 可用來擷取值，然後將它存放在使用者資料庫中。

您稍後可以使用接收的宣告來變更執行流程。

您也可以將互動設計成驗證設定檔。 如需詳細資訊，請參閱[逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為使用者輸入的驗證](active-directory-b2c-rest-api-validation-custom.md)。

有個案例是，當使用者執行設定檔編輯時，我們想要：

1. 查閱外部系統中的使用者。
2. 取得註冊該使用者的城市。
3. 以宣告形式將該屬性傳回應用程式。

## <a name="prerequisites"></a>必要條件

- 如[開始使用](active-directory-b2c-get-started-custom.md)所述，設定為完成本機帳戶註冊/登入的 Azure AD B2C 租用戶。
- 要互動的 REST API 端點。 本逐步解說使用簡單的 Azure 函式應用程式 Webhook 作為範例。
- *建議*︰完成 [REST API 宣告交換逐步解說以作為驗證步驟](active-directory-b2c-rest-api-validation-custom.md)。

## <a name="step-1-prepare-the-rest-api-function"></a>步驟 1：準備 REST API 函式

> [!NOTE]
> REST API 函式的設定不在本文討論範圍內。 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-reference) 提供了絕佳的工具組，供您在雲端建立 RESTful 服務。

我們設定了 Azure 函式來接收名為 `email` 的宣告，然後傳回指派值為 `Redmond` 的宣告 `city`。 範例 Azure 函式位於 [GitHub](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/AzureFunctionsSamples) 上。

在此內容中，Azure 函式所傳回的 `userMessage` 宣告是選擇性的，IEF 將予以略過。 您可以選擇使用它作為訊息，以便稍後傳遞給應用程式並向使用者呈現。

```csharp
if (requestContentAsJObject.email == null)
{
    return request.CreateResponse(HttpStatusCode.BadRequest);
}

var email = ((string) requestContentAsJObject.email).ToLower();

return request.CreateResponse<ResponseContent>(
    HttpStatusCode.OK,
    new ResponseContent
    {
        version = "1.0.0",
        status = (int) HttpStatusCode.OK,
        userMessage = "User Found",
        city = "Redmond"
    },
    new JsonMediaTypeFormatter(),
    "application/json");
```

Azure 函式應用程式可讓您輕鬆地取得函式 URL，此 URL 中包含特定函式的識別碼。 在此案例中，URL 是： https://wingtipb2cfuncs.azurewebsites.net/api/LookUpLoyaltyWebHook?code=MQuG7BIE3eXBaCZ/YCfY1SHabm55HEphpNLmh1OP3hdfHkvI2QwPrw==。 您可以使用它來測試。

## <a name="step-2-configure-the-restful-api-claims-exchange-as-a-technical-profile-in-your-trustframeworextensionsxml-file"></a>步驟 2：在 TrustFrameworExtensions.xml 檔案中，將 RESTful API 宣告交換設為技術設定檔

技術設定檔是 RESTful 服務所需之交換的完整設定。 開啟 TrustFrameworkExtensions.xml 檔案，然後在 `<ClaimsProvider>` 元素內加入下列 XML 程式碼片段。

> [!NOTE]
> 在下列 XML 中，會將 RESTful 提供者 `Version=1.0.0.0` 描述為通訊協定。 請將它視為要與外部服務互動的函式。 <!-- TODO: A full definition of the schema can be found...link to RESTful Provider schema definition>-->

```XML
<ClaimsProvider>
    <DisplayName>REST APIs</DisplayName>
    <TechnicalProfiles>
        <TechnicalProfile Id="AzureFunctions-LookUpLoyaltyWebHook">
            <DisplayName>Check LookUpLoyalty Web Hook Azure Function</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
                <Item Key="ServiceUrl">https://wingtipb2cfuncs.azurewebsites.net/api/LookUpLoyaltyWebHook?code=MQuG7BIE3eXBaCZ/YCfY1SHabm55HEphpNLmh1OP3hdfHkvI2QwPrw==</Item>
                <Item Key="AuthenticationType">None</Item>
                <Item Key="SendClaimsIn">Body</Item>
            </Metadata>
            <InputClaims>
                <InputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="email" />
            </InputClaims>
            <OutputClaims>
                <OutputClaim ClaimTypeReferenceId="city" PartnerClaimType="city" />
            </OutputClaims>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>
```

`<InputClaims>` 元素會定義將從 IEF 傳送至 REST 服務的宣告。 在此範例中，會將 `givenName` 宣告的內容傳送至 REST 服務以作為宣告 `email`。  

`<OutputClaims>` 元素會定義 IEF 預期要從 REST 服務收到的宣告。 不論收到多少宣告，IEF 只會使用這裡所識別的宣告。 在此範例中，以 `city` 形式收到的宣告會對應到名為 `city` 的 IEF 宣告。

## <a name="step-3-add-the-new-claim-city-to-the-schema-of-your-trustframeworkextensionsxml-file"></a>步驟 3：將新的宣告 `city` 新增至 TrustFrameworkExtensions.xml 檔案的結構描述

宣告 `city` 尚未在結構描述的其他地方加以定義。 因此，請在元素 `<BuildingBlocks>` 內部新增定義。 您可以在 TrustFrameworkExtensions.xml 檔案開頭處找到此元素。

```XML
<BuildingBlocks>
    <!--The claimtype city must be added to the TrustFrameworkPolicy-->
    <!-- You can add new claims in the BASE file Section III, or in the extensions file-->
    <ClaimsSchema>
        <ClaimType Id="city">
            <DisplayName>City</DisplayName>
            <DataType>string</DataType>
            <UserHelpText>Your city</UserHelpText>
            <UserInputType>TextBox</UserInputType>
        </ClaimType>
    </ClaimsSchema>
</BuildingBlocks>
```

## <a name="step-4-include-the-rest-service-claims-exchange-as-an-orchestration-step-in-your-profile-edit-user-journey-in-trustframeworkextensionsxml"></a>步驟 4：在 TrustFrameworkExtensions.xml 的設定檔編輯使用者旅程圖中，將 REST 服務宣告交換納入為協調流程步驟

在使用者通過驗證之後新增設定檔編輯使用者旅程圖的步驟 (下列 XML 中的協調流程步驟 1-4)，而且使用者已提供更新的設定檔資訊 (步驟 5)。

> [!NOTE]
> 有許多使用案例都可將 REST API 呼叫用來作為協調流程步驟。 作為協調流程步驟，它可在使用者成功完成工作 (例如首次註冊) 後作為外部系統的更新，或作為設定檔更新以讓資訊保持同步。 在此情況下，它會用來加強設定檔編輯之後提供給應用程式的資訊。

將設定檔編輯使用者旅程圖 XML 程式碼從 TrustFrameworkBase.xml 檔案複製到 `<UserJourneys>` 元素內的 TrustFrameworkExtensions.xml 檔案。 然後在步驟 6 中進行修改。

```XML
<OrchestrationStep Order="6" Type="ClaimsExchange">
    <ClaimsExchanges>
        <ClaimsExchange Id="GetLoyaltyData" TechnicalProfileReferenceId="AzureFunctions-LookUpLoyaltyWebHook" />
    </ClaimsExchanges>
</OrchestrationStep>
```

> [!IMPORTANT]
> 如果操作順序不符合您的版本，請確定和該步驟一樣在 `ClaimsExchange` 類型 `SendClaims` 前插入程式碼。

使用者旅程圖的最終 XML 看起來應該像這樣：

```XML
<UserJourney Id="ProfileEdit">
    <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
            <ClaimsProviderSelections>
                <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange" />
                <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSigninEmailExchange" />
            </ClaimsProviderSelections>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
                <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="3" Type="ClaimsExchange">
            <Preconditions>
                <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
                    <Value>authenticationSource</Value>
                    <Value>localAccountAuthentication</Value>
                    <Action>SkipThisOrchestrationStep</Action>
                </Precondition>
            </Preconditions>
            <ClaimsExchanges>
                <ClaimsExchange Id="AADUserRead" TechnicalProfileReferenceId="AAD-UserReadUsingAlternativeSecurityId" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="4" Type="ClaimsExchange">
            <Preconditions>
                <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
                    <Value>authenticationSource</Value>
                    <Value>socialIdpAuthentication</Value>
                    <Action>SkipThisOrchestrationStep</Action>
                </Precondition>
            </Preconditions>
            <ClaimsExchanges>
                <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="5" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="B2CUserProfileUpdateExchange" TechnicalProfileReferenceId="SelfAsserted-ProfileUpdate" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <!-- Add a step 6 to the user journey before the JWT token is created-->
        <OrchestrationStep Order="6" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="GetLoyaltyData" TechnicalProfileReferenceId="AzureFunctions-LookUpLoyaltyWebHook" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    </OrchestrationSteps>
    <ClientDefinition ReferenceId="DefaultWeb" />
</UserJourney>
```

## <a name="step-5-add-the-claim-city-to-your-relying-party-policy-file-so-the-claim-is-sent-to-your-application"></a>步驟 5：將宣告 `city` 新增至您的信賴憑證者原則檔案，以便將宣告傳送至您的應用程式

請編輯您的 ProfileEdit.xml 信賴憑證者 (RP) 檔案，並修改 `<TechnicalProfile Id="PolicyProfile">` 元素以新增下列內容︰`<OutputClaim ClaimTypeReferenceId="city" />`。

新增宣告之後，技術設定檔看起來像這樣：

```XML
<DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="city" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
</TechnicalProfile>
```

## <a name="step-6-upload-your-changes-and-test"></a>步驟 6：上傳您的變更和測試

覆寫現有的原則版本。

1.  (選擇性：) 繼續之前，請先儲存擴充檔案的現有版本 (透過下載)。 建議您不要上傳多個擴充檔案版本，以免一開始的複雜性太高。
2.  (選擇性：) 藉由變更 `PolicyId="B2C_1A_TrustFrameworkProfileEdit"`，來為原則編輯檔案重新命名原則識別碼的新版本。
3.  上傳擴充檔案。
4.  上傳原則編輯 RP 檔案。
5.  使用 [立即執行] 來測試原則。 檢閱 IEF 傳回給應用程式的權杖。

如果一切設定皆正確，權杖會包含新的宣告 `city`，且值為 `Redmond`。

```JSON
{
  "exp": 1493053292,
  "nbf": 1493049692,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "sub": "a58e7c6c-7535-4074-93da-b0023fbaf3ac",
  "aud": "4e87c1dd-e5f5-4ac8-8368-bc6a98751b8b",
  "acr": "b2c_1a_trustframeworkprofileedit",
  "nonce": "defaultNonce",
  "iat": 1493049692,
  "auth_time": 1493049692,
  "city": "Redmond"
}
```

## <a name="next-steps"></a>後續步驟

[使用 REST API 來作為驗證步驟](active-directory-b2c-rest-api-validation-custom.md)

[修改設定檔編輯以從使用者收集其他資訊](active-directory-b2c-create-custom-attributes-profile-edit-custom.md)
