---
title: 將 REST API 宣告交換整合到 Azure Active Directory B2C 使用者旅程圖中 | Microsoft Docs
description: 將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為使用者輸入的驗證。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/30/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e3d938c4464fc5141b97f85220bf096920e17d00
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43339588"
---
# <a name="integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journey-as-validation-of-user-input"></a>將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中以作為使用者輸入的驗證

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

您可以使用構成 Azure Active Directory B2C (Azure AD B2C) 基礎的識別體驗架構，與使用者旅程圖中的 RESTful API 整合。 在本逐步解說中，您將了解 Azure AD B2C 與.NET Framework RESTful 服務 (Web API) 之間的互動方式。

## <a name="introduction"></a>簡介
您可以使用 Azure AD B2C，呼叫自己的 RESTful 服務，將自己的商務邏輯新增至使用者旅程圖。 識別體驗架構會在*輸入宣告*集合中將資料傳送至 RESTful 服務，並在*輸出宣告*集合中從 RESTful 接收資料。 使用 RESTful 服務整合，您可以：

* **驗證使用者輸入資料**：此動作可防止將格式不正確的資料持續保存至 Azure AD。 如果來自使用者的值無效，您的 RESTful 服務會傳回錯誤訊息，指示使用者提供輸入。 例如，您可能會驗證使用者提供的電子郵件地址存在於客戶資料庫中。
* **覆寫輸入宣告**：例如，如果使用者輸入全部小寫或全部大寫的名字，您可以將名稱格式化為只有第一個字母大寫。
* **與公司的企業營運應用程式進一步整合來擴充使用者資料**：您的 RESTful 服務可以接收使用者的電子郵件地址、查詢客戶的資料庫，以及將使用者的忠誠度號碼傳回給 Azure AD B2C。 傳回宣告可能會儲存在使用者的 Azure AD 帳戶中，於下一個*協調流程步驟*中評估，或包含在存取權杖中。
* **執行自訂商務邏輯**：您可以傳送推播通知、更新公司資料庫、執行使用者移轉程序、管理權限、稽核資料庫，以及執行其他動作。

您可以使用下列方式來設計與 RESTful 服務的整合：

* **驗證技術設定檔**：對 RESTful 服務的呼叫發生在所指定技術設定檔的驗證技術設定檔內部。 在使用者旅程圖前進之前，驗證技術設定檔會驗證使用者提供的資料。 您可以使用驗證技術設定檔：
   * 傳送輸入宣告。
   * 驗證輸入宣告並擲回自訂錯誤訊息。
   * 送回輸出宣告。

* **宣告交換**：此設計類似於驗證技術設定檔，但發生在協調流程步驟內部。 這個定義僅限於：
   * 傳送輸入宣告。
   * 送回輸出宣告。

## <a name="restful-walkthrough"></a>RESTful 逐步解說
在本逐步解說中，您將開發會驗證使用者輸入並提供使用者忠誠度號碼的 .NET Framework Web API。 例如，您的應用程式可以根據忠誠度號碼來授與「白金權益」的存取權。

概觀：
* 開發 RESTful 服務 (.NET Framework Web API)。
* 在使用者旅程圖中使用 RESTful 服務。
* 傳送輸入宣告，並在您的程式碼中讀取。
* 驗證使用者的名字。
* 傳回忠誠度號碼。 
* 將忠誠度號碼新增至 JSON Web 權杖 (JWT)。

## <a name="prerequisites"></a>必要條件
完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟。

## <a name="step-1-create-an-aspnet-web-api"></a>步驟 1：建立 ASP.NET Web API

1. 在 Visual Studio 中，選取 [檔案] > [新增] > [專案] 以建立專案。

2. 在 [新增專案] 視窗中，選取 [Visual C#] > [Web] > [ASP.NET Web 應用程式 (.NET Framework)]。

3. 在 [名稱] 方塊中，輸入應用程式的名稱 (例如 *Contoso.AADB2C.API*)，然後選取 [確定]。

    ![建立新的 Visual Studio 專案](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-create-project.png)

4. 在 [新的 ASP.NET Web 應用程式] 視窗中，選取 [Web API] 或 [Azure API 應用程式] 範本。

    ![選取 Web API 範本](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-select-web-api.png)

5. 確定驗證已設為 [沒有驗證]。

6. 選取 [確定] 以建立專案。

## <a name="step-2-prepare-the-rest-api-endpoint"></a>步驟 2：準備 REST API 端點

### <a name="step-21-add-data-models"></a>步驟 2.1：新增資料模型
模型會呈現 RESTful 服務中的輸入宣告以及輸出宣告資料。 您的程式碼會利用將來自 JSON 字串的輸入宣告模型還原序列化為 C# 物件 (您的模型)，藉此讀取輸入資料。 ASP.NET Web API 會將輸出宣告模型自動還原序列化回 JSON，然後將序列化資料寫入至 HTTP 回應訊息的本文。 

執行下列步驟建立一個代表輸入宣告的模型：

1. 如果尚未開啟 [方案總管]，請選取 [檢視] > [方案總管]。 
2. 在 [方案總管] 中，以滑鼠右鍵按一下 **Models** 資料夾，選取 [新增]，然後選取 [類別]。

    ![新增模型](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-add-model.png)

3. 將類別命名為 `InputClaimsModel`，然後將下列屬性新增至 `InputClaimsModel` 類別：

    ```csharp
    namespace Contoso.AADB2C.API.Models
    {
        public class InputClaimsModel
        {
            public string email { get; set; }
            public string firstName { get; set; }
            public string lastName { get; set; }
        }
    }
    ```

4. 建立新的模型 (`OutputClaimsModel`)，然後將下列屬性新增至 `OutputClaimsModel` 類別：

    ```csharp
    namespace Contoso.AADB2C.API.Models
    {
        public class OutputClaimsModel
        {
            public string loyaltyNumber { get; set; }
        }
    }
    ```

5. 再建立一個模型 (`B2CResponseContent`)，用來擲回輸入驗證錯誤訊息。 將下列屬性新增至 `B2CResponseContent` 類別、提供遺漏的參考，然後儲存檔案：

    ```csharp
    namespace Contoso.AADB2C.API.Models
    {
        public class B2CResponseContent
        {
            public string version { get; set; }
            public int status { get; set; }
            public string userMessage { get; set; }

            public B2CResponseContent(string message, HttpStatusCode status)
            {
                this.userMessage = message;
                this.status = (int)status;
                this.version = Assembly.GetExecutingAssembly().GetName().Version.ToString();
            }    
        }
    }
    ```

### <a name="step-22-add-a-controller"></a>步驟 2.2：新增控制器
在 Web API 中，_控制器_是處理 HTTP 要求的物件。 控制器會傳回輸出宣告，或如果名字無效，則會擲回「衝突」HTTP 錯誤訊息。

1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Controllers** 資料夾，選取 [新增]，然後選取 [控制器]。

    ![新增控制器](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-add-controller-1.png)

2. 在 [新增 Scaffold] 視窗中，選取 [Web API 控制器 - 空白]，然後選取 [新增]。

    ![選取 Web API 2 控制器 - 空的](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-add-controller-2.png)

3. 在 [新增控制器]視窗中，將控制器命名為 **IdentityController**，然後選取 [新增]。

    ![輸入控制器名稱](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-add-controller-3.png)

    Scaffolding 會在 [控制器] 資料夾中建立名為 *IdentityController.cs* 的檔案。

4. 如果 *IdentityController.cs* 檔案尚未開啟，請按兩下該檔案，然後將檔案中的程式碼取代為下列程式碼：

    ```csharp
    using Contoso.AADB2C.API.Models;
    using Newtonsoft.Json;
    using System;
    using System.NET;
    using System.Web.Http;

    namespace Contoso.AADB2C.API.Controllers
    {
        public class IdentityController: ApiController
        {
            [HttpPost]
            public IHttpActionResult SignUp()
            {
                // If no data came in, then return
                if (this.Request.Content == null) throw new Exception();

                // Read the input claims from the request body
                string input = Request.Content.ReadAsStringAsync().Result;

                // Check the input content value
                if (string.IsNullOrEmpty(input))
                {
                    return Content(HttpStatusCode.Conflict, new B2CResponseContent("Request content is empty", HttpStatusCode.Conflict));
                }

                // Convert the input string into an InputClaimsModel object
                InputClaimsModel inputClaims = JsonConvert.DeserializeObject(input, typeof(InputClaimsModel)) as InputClaimsModel;

                if (inputClaims == null)
                {
                    return Content(HttpStatusCode.Conflict, new B2CResponseContent("Can not deserialize input claims", HttpStatusCode.Conflict));
                }

                // Run an input validation
                if (inputClaims.firstName.ToLower() == "test")
                {
                    return Content(HttpStatusCode.Conflict, new B2CResponseContent("Test name is not valid, please provide a valid name", HttpStatusCode.Conflict));
                }

                // Create an output claims object and set the loyalty number with a random value
                OutputClaimsModel outputClaims = new OutputClaimsModel();
                outputClaims.loyaltyNumber = new Random().Next(100, 1000).ToString();

                // Return the output claim(s)
                return Ok(outputClaims);
            }
        }
    }
    ```

## <a name="step-3-publish-the-project-to-azure"></a>步驟 3：將專案發佈到 Azure
1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Contoso.AADB2C.API** 專案，然後選取 [發佈]。

    ![發佈至 Microsoft Azure App Service](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-publish-to-azure-1.png)

2. 在 [發佈] 視窗中，選取 [Microsoft Azure App Service]，然後選取 [發佈]。

    ![建立新的 Microsoft Azure App Service](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-publish-to-azure-2.png)

    [建立 App Service] 視窗隨即開啟。 在其中，您可以建立要在 Azure 中執行 ASP.NET Web 應用程式所需的所有 Azure 資源。

    > [!NOTE]
    >如需如何發佈的詳細資訊，請參閱[在 Azure 中建立 ASP.NET Web 應用程式](https://docs.microsoft.com/azure/app-service-web/app-service-web-get-started-dotnet#publish-to-azure)。

3. 在 [Web 應用程式名稱] 方塊中，輸入唯一的應用程式名稱 (有效字元為 a-z、0-9 及連字號 (-))。 Web 應用程式的 URL 是 http://<app_name>.azurewebsites.NET，其中 *app_name* 是 Web 應用程式的名稱。 您可以接受自動產生的名稱 (這是唯一的)。

    ![提供 App Service 屬性](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-publish-to-azure-3.png)

4. 若要開始建立 Azure 資源，請選取 [建立]。  
    在建立了 ASP.NET Web 應用程式之後，精靈會將其發佈至 Azure，然後在預設瀏覽器中啟動應用程式。

6. 複製 Web 應用程式的 URL。

## <a name="step-4-add-the-new-loyaltynumber-claim-to-the-schema-of-your-trustframeworkextensionsxml-file"></a>步驟 4：將新的 `loyaltyNumber` 宣告新增至 TrustFrameworkExtensions.xml 檔案的結構描述
`loyaltyNumber` 宣告尚未定義在結構描述中。 在 `<BuildingBlocks>` 元素內新增一個定義，您可以在 *TrustFrameworkExtensions.xml* 檔案開頭處找到此元素。

```xml
<BuildingBlocks>
    <ClaimsSchema>
        <ClaimType Id="loyaltyNumber">
            <DisplayName>loyaltyNumber</DisplayName>
            <DataType>string</DataType>
            <UserHelpText>Customer loyalty number</UserHelpText>
        </ClaimType>
    </ClaimsSchema>
</BuildingBlocks>
```

## <a name="step-5-add-a-claims-provider"></a>步驟 5：新增宣告提供者 
每個宣告提供者必須有一或多個技術設定檔，以決定與該宣告提供者進行通訊所需的端點以及通訊協定。 

基於各種原因，宣告提供者可以有多個技術設定檔。 例如，可能定義了多個技術設定檔，因為宣告提供者支援多個通訊協定，端點可以具有不同的功能，或是版本可以包含具有各種保證層級的宣告。 它可能會在一個使用者旅程圖中接受釋出機密宣告，但在另一個則不接受。 

下列 XML 程式碼片段包含一個宣告提供者節點，其中具有兩個技術設定檔：

* **TechnicalProfile Id="REST-API-SignUp"**：定義您的 RESTful 服務。 
   * `Proprietary` 會描述為以 RESTful 為基礎之提供者的通訊協定。 
   * `InputClaims` 會定義將從 Azure AD B2C 傳送至 REST 服務的宣告。 

   在此範例中，`givenName` 宣告的內容會傳送至 REST 服務做為 `firstName`，宣告 `surname` 的內容會傳送至 REST 服務做為 `lastName`，而 `email` 會依原狀傳送。 `OutputClaims` 元素定義的宣告會從 RESTful 服務擷取回 Azure AD B2C。

* **TechnicalProfile Id="LocalAccountSignUpWithLogonEmail"**：將驗證技術設定檔新增至現有的技術設定檔 (定義在基本原則中)。 在註冊旅程期間，驗證技術設定檔會叫用上述技術設定檔。 如果 RESTful 服務傳回 HTTP 錯誤 409 (衝突錯誤)，錯誤訊息會顯示給使用者。 

找出 `<ClaimsProviders>` 節點，然後在 `<ClaimsProviders>` 節點下方新增下列 XML 程式碼片段：

```xml
<ClaimsProvider>
    <DisplayName>REST APIs</DisplayName>
    <TechnicalProfiles>
    
    <!-- Custom Restful service -->
    <TechnicalProfile Id="REST-API-SignUp">
        <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
        <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
        <Item Key="AuthenticationType">None</Item>
        <Item Key="SendClaimsIn">Body</Item>
        </Metadata>
        <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName" />
        <InputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName" />
        </InputClaims>
        <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="loyaltyNumber" />
        </OutputClaims>
        <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>

<!-- Change LocalAccountSignUpWithLogonEmail technical profile to support your validation technical profile -->
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
        <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="loyaltyNumber" />
        </OutputClaims>
        <ValidationTechnicalProfiles>
        <ValidationTechnicalProfile ReferenceId="REST-API-SignUp" />
        </ValidationTechnicalProfiles>
    </TechnicalProfile>

    </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="step-6-add-the-loyaltynumber-claim-to-your-relying-party-policy-file-so-the-claim-is-sent-to-your-application"></a>步驟 6：將宣告 `loyaltyNumber` 新增至您的信賴憑證者原則檔案，以便將宣告傳送至您的應用程式
編輯您的 *SignUpOrSignIn.xml* 信賴憑證者 (RP) 檔案，並修改 TechnicalProfile Id="PolicyProfile" 元素以新增下列內容：`<OutputClaim ClaimTypeReferenceId="loyaltyNumber" />`。

新增宣告之後，信賴憑證者程式碼看起來像這樣：

```xml
<RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
        <DisplayName>PolicyProfile</DisplayName>
        <Protocol Name="OpenIdConnect" />
        <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" DefaultValue="" />
        </OutputClaims>
        <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
    </RelyingParty>
</TrustFrameworkPolicy>
```

## <a name="step-7-upload-the-policy-to-your-tenant"></a>步驟 7：將原則上傳至您的租用戶

1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C]。

2. 選取 [識別體驗架構]。

3. 開啟 [所有原則]。 

4. 選取 [上傳原則]。

5. 選取 [覆寫已存在的原則] 核取方塊。

6. 上傳 TrustFrameworkExtensions.xml 檔案，確定它通過驗證。

7. 使用 SignUpOrSignIn.xml 檔重複上述步驟。

## <a name="step-8-test-the-custom-policy-by-using-run-now"></a>步驟 8：使用 [立即執行] 測試自訂原則
1. 選取 [Azure AD B2C 設定]，然後移至 [識別體驗架構]。

    > [!NOTE]
    > [立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

2. 開啟 **B2C_1A_signup_signin** (此為您上傳的信賴憑證者 (RP) 自訂原則)，然後選取 [立即執行]。

    ![B2C_1A_signup_signin 視窗](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-run.png)

3. 在 [名字] 方塊中，輸入 **Test** 來測試流程。  
    Azure AD B2C 會在視窗頂端顯示錯誤訊息。

    ![測試您的原則](media/aadb2c-ief-rest-api-netfw/aadb2c-ief-rest-api-netfw-test.png)

4.  在 [名字] 方塊中輸入名字 ("Test" 除外)。  
    Azure AD B2C 會註冊使用者，然後將 loyaltyNumber傳送至您的應用程式。 記下此 JWT 中的數字。

```
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1507125903,
  "nbf": 1507122303,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1507122303,
  "auth_time": 1507122303,
  "loyaltyNumber": "290",
  "given_name": "Emily",
  "emails": ["B2cdemo@outlook.com"]
}
```

## <a name="optional-download-the-complete-policy-files-and-code"></a>(選用) 下載完整的原則檔案和程式碼
* 在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)逐步解說之後，建議您使用自己的自訂原則檔案來建置您的情節。 我們已提供[範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw)，供您參考。
* 您可以從[供參考的範例 Visual Studio 解決方案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw/)中下載完整程式碼。
    
## <a name="next-steps"></a>後續步驟
* [使用基本驗證 (使用者名稱和密碼) 保護您的 RESTful API](active-directory-b2c-custom-rest-api-netfw-secure-basic.md)
* [使用用戶端憑證保護您的 RESTful API](active-directory-b2c-custom-rest-api-netfw-secure-cert.md)
