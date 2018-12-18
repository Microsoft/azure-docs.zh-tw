---
title: 在 Azure Active Directory B2C 中使用 REST API 宣告交換作為驗證 | Microsoft Docs
description: Azure Active Directory B2C 自訂原則的主題。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/24/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 0ac9b98a9dfe06492775481cd590bfb4d0db4b55
ms.sourcegitcommit: f983187566d165bc8540fdec5650edcc51a6350a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "45542577"
---
# <a name="walkthrough-integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journey-as-validation-on-user-input"></a>逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為使用者輸入的驗證

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

構成 Azure Active Directory B2C (Azure AD B2C) 基礎的識別體驗架構 (IEF)，可讓身分識別開發人員於使用者旅程圖中整合與 RESTful API 的互動。  

在本逐步解說結束時，您將能建立與 RESTful 服務互動的 Azure AD B2C 使用者旅程圖。

IEF 會在宣告中傳送資料，並在宣告中收到傳回的資料。 與 API 的互動：

- 可設計為 REST API 宣告交換，或作為在協調流程步驟內發生的驗證設定檔。
- 通常會驗證使用者的輸入。 如果系統拒絕使用者輸入的值，使用者可再次試著輸入有效值，但系統可能會傳回錯誤訊息。

您也可以將互動設計為協調流程步驟。 如需詳細資訊，請參閱[逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為協調流程步驟](active-directory-b2c-rest-api-step-custom.md)。

至於驗證設定檔範例，我們將使用入門套件檔案 ProfileEdit.xml 中的設定檔編輯使用者旅程圖。

我們可以驗證使用者在設定檔編輯中所提供的名稱不在排除清單中。

## <a name="prerequisites"></a>必要條件

- 如[開始使用](active-directory-b2c-get-started-custom.md)所述，設定為完成本機帳戶註冊/登入的 Azure AD B2C 租用戶。
- 要互動的 REST API 端點。 針對這個逐步解說，我們設定了名為 [WingTipGames](https://wingtipgamesb2c.azurewebsites.net/) 的示範網站，其中含有一個 REST API 服務。

## <a name="step-1-prepare-the-rest-api-function"></a>步驟 1：準備 REST API 函式

> [!NOTE]
> REST API 函式的設定不在本文討論範圍內。 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-reference) 提供了絕佳的工具組，供您在雲端建立 RESTful 服務。

我們建立了一個 Azure 函式，來接收它預期作為 `playerTag` 的宣告。 此函式會驗證此宣告是否存在。 您可以在 [GitHub](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/AzureFunctionsSamples) 中取得完整的 Azure 函式程式碼。

```csharp
if (requestContentAsJObject.playerTag == null)
{
  return request.CreateResponse(HttpStatusCode.BadRequest);
}

var playerTag = ((string) requestContentAsJObject.playerTag).ToLower();

if (playerTag == "mcvinny" || playerTag == "msgates123" || playerTag == "revcottonmarcus")
{
  return request.CreateResponse<ResponseContent>(
    HttpStatusCode.Conflict,
    new ResponseContent
    {
      version = "1.0.0",
      status = (int) HttpStatusCode.Conflict,
      userMessage = $"The player tag '{requestContentAsJObject.playerTag}' is already used."
    },
    new JsonMediaTypeFormatter(),
    "application/json");
}

return request.CreateResponse(HttpStatusCode.OK);
```

IEF 預期 Azure 函式會傳回 `userMessage` 宣告。 如果驗證失敗，即會以字串形式為使用者呈現此宣告，例如，當上述範例中傳回 409 衝突狀態時。

## <a name="step-2-configure-the-restful-api-claims-exchange-as-a-technical-profile-in-your-trustframeworkextensionsxml-file"></a>步驟 2：在 TrustFrameworkExtensions.xml 檔案中，將 RESTful API 宣告交換設為技術設定檔

技術設定檔是 RESTful 服務所需之交換的完整設定。 開啟 TrustFrameworkExtensions.xml 檔案，然後在 `<ClaimsProviders>` 元素內加入下列 XML 程式碼片段。

> [!NOTE]
> 在下列 XML 中，會將 RESTful 提供者 `Version=1.0.0.0` 描述為通訊協定。 請將它視為要與外部服務互動的函式。 <!-- TODO: A full definition of the schema can be found...link to RESTful Provider schema definition>-->

```xml
<ClaimsProvider>
    <DisplayName>REST APIs</DisplayName>
    <TechnicalProfiles>
        <TechnicalProfile Id="AzureFunctions-CheckPlayerTagWebHook">
            <DisplayName>Check Player Tag Web Hook Azure Function</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
                <Item Key="ServiceUrl">https://wingtipb2cfuncs.azurewebsites.net/api/CheckPlayerTagWebHook?code=L/05YRSpojU0nECzM4Tp3LjBiA2ZGh3kTwwp1OVV7m0SelnvlRVLCg==</Item>
                <Item Key="AuthenticationType">None</Item>
                <Item Key="SendClaimsIn">Body</Item>
            </Metadata>
            <InputClaims>
                <InputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="playerTag" />
            </InputClaims>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        <TechnicalProfile Id="SelfAsserted-ProfileUpdate">
            <ValidationTechnicalProfiles>
                <ValidationTechnicalProfile ReferenceId="AzureFunctions-CheckPlayerTagWebHook" />
            </ValidationTechnicalProfiles>
        </TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>
```

`InputClaims` 元素會定義將從 IEF 傳送至 REST 服務的宣告。 在此範例中，會將 `givenName` 宣告的內容傳送至 REST 服務以作為 `playerTag`。 在此範例中，IEF 不會預期傳回的宣告。 相反地，它會等待來自 REST 服務的回應，並根據它接收的狀態碼採取動作。

## <a name="step-3-include-the-restful-service-claims-exchange-in-self-asserted-technical-profile-where-you-want-to-validate-the-user-input"></a>步驟 3：在您想要用來驗證使用者輸入的自我判斷技術設定檔中加入 RESTful 服務宣告交換

驗證步驟最常用於與使用者互動。 使用者應該在其中提供輸入的所有互動就是「自我判斷的技術設定檔」。 在此範例中，我們會將此驗證新增到 Self-Asserted-ProfileUpdate 技術設定檔。 這是信賴憑證者 (RP) 原則檔案 `Profile Edit` 使用的技術設定檔。

在自我判斷的技術設定檔中新增宣告交換：

1. 開啟 TrustFrameworkBase.xml 檔案，並搜尋 `<TechnicalProfile Id="SelfAsserted-ProfileUpdate">`。
2. 檢閱此技術設定檔的設定。 觀察與使用者所進行的交換如何定義為向使用者要求的宣告 (輸入宣告)，以及如何定義為應該從自我判斷提供者傳回的宣告 (輸出宣告)。
3. 搜尋 `TechnicalProfileReferenceId="SelfAsserted-ProfileUpdate`，並注意系統會叫用此設定檔來作為 `<UserJourney Id="ProfileEdit">` 的協調流程步驟 5。

## <a name="step-4-upload-and-test-the-profile-edit-rp-policy-file"></a>步驟 4：上傳和測試設定檔編輯 RP 原則檔案

1. 上傳新版的 TrustFrameworkExtensions.xml 檔案。
2. 使用**立即執行**來測試設定檔編輯 RP 原則檔案。
3. 在 [名字] 欄位中提供一個現有名稱 (例如︰mcvinny) 來測試驗證。 如果一切設定皆正確，您應該會收到訊息，通知使用者，該玩家標記已在使用中。

## <a name="next-steps"></a>後續步驟

[修改設定檔編輯和使用者註冊以從使用者收集其他資訊](active-directory-b2c-create-custom-attributes-profile-edit-custom.md)

[逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為協調流程步驟](active-directory-b2c-rest-api-step-custom.md)
