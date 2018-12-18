---
title: 在 Azure Active Directory B2C 中使用 HTTP 基本驗證保護 RESTful 服務 | Microsoft Docs
description: 使用 HTTP 基本驗證在您的 Azure AD B2C 中保護您的自訂 REST API 宣告交換。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/25/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: e6cff4e2daf86b63bc0db0d4f2d537322d2841df
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409298"
---
# <a name="secure-your-restful-services-by-using-http-basic-authentication"></a>使用 HTTP 基本驗證保護 RESTful 服務

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

在 [Azure AD B2C 相關文章](active-directory-b2c-custom-rest-api-netfw.md)中，您在未經驗證的情況下，即建立了與 Azure Active Directory B2C (Azure AD B2C) 使用者旅程圖整合的 RESTful 服務 (Web API)。 

在此文章中，您將 HTTP 基本驗證新增至 RESTful 服務，使得只有經過驗證的使用者 (包括 B2C) 可存取您的 API。 利用 HTTP 基本驗證，您可以在自訂原則中設定使用者認證 (應用程式識別碼和應用程式密碼)。 

如需詳細資訊，請參閱[ASP.NET Web API 中的基本驗證](https://docs.microsoft.com/aspnet/web-api/overview/security/basic-authentication)。

## <a name="prerequisites"></a>必要條件
完成[將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程圖中](active-directory-b2c-custom-rest-api-netfw.md)文章的步驟。

## <a name="step-1-add-authentication-support"></a>步驟 1：新增驗證支援

### <a name="step-11-add-application-settings-to-your-projects-webconfig-file"></a>步驟 1.1：將應用程式設定新增至專案的 web.config 檔案
1. 開啟您稍早建立的 Visual Studio 專案。 

2. 將下列應用程式設定新增至 web.config 檔案的 `appSettings` 元素下：

    ```XML
    <add key="WebApp:ClientId" value="B2CServiceUserAccount" />
    <add key="WebApp:ClientSecret" value="your secret" />
    ```

3. 建立密碼並設定 `WebApp:ClientSecret` 值。

    若要產生複雜的密碼，請執行下列 PowerShell 程式碼。 您可以使用任意值。

    ```PowerShell
    $bytes = New-Object Byte[] 32
    $rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
    $rand.GetBytes($bytes)
    $rand.Dispose()
    [System.Convert]::ToBase64String($bytes)
    ```

### <a name="step-12-install-owin-libraries"></a>步驟 1.2：安裝 OWIN 程式庫
首先，請使用 Visual Studio Package Manager Console，將 OWIN 中介軟體 NuGet 封裝新增至專案：

```
PM> Install-Package Microsoft.Owin
PM> Install-Package Owin
PM> Install-Package Microsoft.Owin.Host.SystemWeb
```

### <a name="step-13-add-an-authentication-middleware-class"></a>步驟 1.3：新增驗證中介軟體類別
在 App_Start 資料夾下新增 `ClientAuthMiddleware.cs` 類別。 若要這樣做：

1. 以滑鼠右鍵按一下 App_Start 資料夾，選取 [新增]，然後選取 [類別]。

   ![在 App_Start 資料夾中新增 ClientAuthMiddleware.cs 類別](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-secure-basic-OWIN-startup-auth1.png)

2. 在 [名稱] 方塊中，輸入 **ClientAuthMiddleware.cs**。

   ![建立新的 C# 類別](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-secure-basic-OWIN-startup-auth2.png)

3. 開啟 App_Start\ClientAuthMiddleware.cs 檔案，並以下列程式碼取代檔案內容：

    ```csharp
    
    using Microsoft.Owin;
    using System;
    using System.Collections.Generic;
    using System.Configuration;
    using System.Linq;
    using System.Security.Principal;
    using System.Text;
    using System.Threading.Tasks;
    using System.Web;
    
    namespace Contoso.AADB2C.API
    {
        /// <summary>
        /// Class to create a custom owin middleware to check for client authentication
        /// </summary>
        public class ClientAuthMiddleware
        {
            private static readonly string ClientID = ConfigurationManager.AppSettings["WebApp:ClientId"];
            private static readonly string ClientSecret = ConfigurationManager.AppSettings["WebApp:ClientSecret"];
    
            /// <summary>
            /// Gets or sets the next owin middleware
            /// </summary>
            private Func<IDictionary<string, object>, Task> Next { get; set; }
    
            /// <summary>
            /// Initializes a new instance of the <see cref="ClientAuthMiddleware"/> class.
            /// </summary>
            /// <param name="next"></param>
            public ClientAuthMiddleware(Func<IDictionary<string, object>, Task> next)
            {
                this.Next = next;
            }
    
            /// <summary>
            /// Invoke client authentication middleware during each request.
            /// </summary>
            /// <param name="environment">Owin environment</param>
            /// <returns></returns>
            public Task Invoke(IDictionary<string, object> environment)
            {
                // Get wrapper class for the environment
                var context = new OwinContext(environment);
    
                // Check whether the authorization header is available. This contains the credentials.
                var authzValue = context.Request.Headers.Get("Authorization");
                if (string.IsNullOrEmpty(authzValue) || !authzValue.StartsWith("Basic ", StringComparison.OrdinalIgnoreCase))
                {
                    // Process next middleware
                    return Next(environment);
                }
    
                // Get credentials
                var creds = authzValue.Substring("Basic ".Length).Trim();
                string clientId;
                string clientSecret;
    
                if (RetrieveCreds(creds, out clientId, out clientSecret))
                {
                    // Set transaction authenticated as client
                    context.Request.User = new GenericPrincipal(new GenericIdentity(clientId, "client"), new string[] { "client" });
                }
    
                return Next(environment);
            }
    
            /// <summary>
            /// Retrieve credentials from header
            /// </summary>
            /// <param name="credentials">Authorization header</param>
            /// <param name="clientId">Client identifier</param>
            /// <param name="clientSecret">Client secret</param>
            /// <returns>True if valid credentials were presented</returns>
            private bool RetrieveCreds(string credentials, out string clientId, out string clientSecret)
            {
                string pair;
                clientId = clientSecret = string.Empty;
    
                try
                {
                    pair = Encoding.UTF8.GetString(Convert.FromBase64String(credentials));
                }
                catch (FormatException)
                {
                    return false;
                }
                catch (ArgumentException)
                {
                    return false;
                }
    
                var ix = pair.IndexOf(':');
                if (ix == -1)
                {
                    return false;
                }
    
                clientId = pair.Substring(0, ix);
                clientSecret = pair.Substring(ix + 1);
    
                // Return whether credentials are valid
                return (string.Compare(clientId, ClientAuthMiddleware.ClientID) == 0 &&
                    string.Compare(clientSecret, ClientAuthMiddleware.ClientSecret) == 0);
            }
        }
    }
    ```

### <a name="step-14-add-an-owin-startup-class"></a>步驟 1.4：新增 OWIN 啟動類別
將名為 `Startup.cs` 的 OWIN 啟動類別新增至 API。 若要這樣做：
1. 在專案上按一下滑鼠右鍵，選取 [新增] > [新增項目]，然後搜尋 **OWIN**。

   ![新增 OWIN 啟動類別](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-secure-basic-OWIN-startup.png)

2. 開啟 Startup.cs 檔案，並以下列程式碼取代檔案內容：

    ```csharp
    using Microsoft.Owin;
    using Owin;
    
    [assembly: OwinStartup(typeof(Contoso.AADB2C.API.Startup))]
    namespace Contoso.AADB2C.API
    {
        public class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                    app.Use<ClientAuthMiddleware>();
            }
        }
    }
    ```

### <a name="step-15-protect-the-identity-api-class"></a>步驟 1.5：保護身分識別 API 類別
開啟 Controllers\IdentityController.cs，並將 `[Authorize]` 標記新增至控制器類別。 此標記會限制符合授權需求之使用者對控制器的存取。

![將授權標記新增至控制器](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-secure-basic-authorize.png)

## <a name="step-2-publish-to-azure"></a>步驟 2：發佈至 Azure
若要發佈您的專案，請在 [方案總管] 中，以滑鼠右鍵按一下 **Contoso.AADB2C.API**專案，然後選取 [發佈]。

## <a name="step-3-add-the-restful-services-app-id-and-app-secret-to-azure-ad-b2c"></a>步驟 3：將 RESTful 服務應用程式識別碼及應用程式密碼新增至 Azure AD B2C
利用用戶端識別碼 (使用者名稱) 和密碼保護 RESTful 服務之後，您必須將認證儲存在您的 Azure AD B2C 租用戶中。 您的自訂原則在叫用 RESTful 服務時會提供認證。 

### <a name="step-31-add-a-restful-services-client-id"></a>步驟 3.1：新增 RESTful 服務用戶端識別碼
1. 在您的 Azure AD B2C 租用戶中，選取 [B2C 設定] > [識別體驗架構]。


2. 若要檢視租用戶中可用的金鑰，請選取 [原則金鑰]。

3. 選取 [新增] 。

4. 針對 [選項]，選取 [手動]。

5. 針對 [名稱]，輸入 **B2cRestClientId**。  
    可能會自動新增前置詞 B2C_1A_。

6. 在 [密碼] 方塊中，輸入您先前定義的應用程式識別碼。

7. 針對 [金鑰使用方法] 選取 [簽章]。

8. 選取 [建立]。

9. 確認您已建立 `B2C_1A_B2cRestClientId` 金鑰。

### <a name="step-32-add-a-restful-services-client-secret"></a>步驟 3.2：新增 RESTful 服務用戶端密碼
1. 在您的 Azure AD B2C 租用戶中，選取 [B2C 設定] > [識別體驗架構]。

2. 選取 [原則金鑰] 以檢視您的租用戶中可用的金鑰。

3. 選取 [新增]。

4. 針對 [選項]，選取 [手動]。

5. 針對 [名稱]，輸入 **B2cRestClientSecret**。  
    可能會自動新增前置詞 B2C_1A_。

6. 在 [密碼] 方塊中，輸入您先前定義的應用程式密碼。

7. 針對 [金鑰使用方法] 選取 [簽章]。

8. 選取 [建立]。

9. 確認您已建立 `B2C_1A_B2cRestClientSecret` 金鑰。

## <a name="step-4-change-the-technical-profile-to-support-basic-authentication-in-your-extension-policy"></a>步驟 4：在擴充原則中變更技術設定檔以支援基本驗證
1. 在您的工作目錄中，開啟擴充原則檔案 (TrustFrameworkExtensions.xml)。

2. 搜尋包含 `Id="REST-API-SignUp"` 的 `<TechnicalProfile>` 節點。

3. 找出 `<Metadata>` 元素。

4. 將 *AuthenticationType* 變更為 *Basic*，如下所示：
    ```xml
    <Item Key="AuthenticationType">Basic</Item>
    ```

5. 在關閉 `<Metadata>` 元素之後，立即新增下列 XML 程式碼片段： 

    ```xml
    <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_B2cRestClientId" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_B2cRestClientSecret" />
    </CryptographicKeys>
    ```
    在新增程式碼片段之後，您的技術設定檔看起來應該類似下列 XML 程式碼：
    
    ![新增基本驗證 XML 元素](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-secure-basic-add-1.png)

## <a name="step-5-upload-the-policy-to-your-tenant"></a>步驟 5：將原則上傳至您的租用戶

1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後開啟 [Azure AD B2C]。

2. 選取 [識別體驗架構]。

3. 開啟 [所有原則]。

4. 選取 [上傳原則]。

5. 選取 [覆寫已存在的原則] 核取方塊。

6. 上傳 TrustFrameworkExtensions.xml 檔案，然後確定它通過驗證。

## <a name="step-6-test-the-custom-policy-by-using-run-now"></a>步驟 6：使用 [立即執行] 測試自訂原則
1. 開啟 [Azure AD B2C 設定]，然後選取 [識別體驗架構]。

    >[!NOTE]
    >[立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

2. 開啟 **B2C_1A_signup_signin** (此為您上傳的信賴憑證者 (RP) 自訂原則)，然後選取 [立即執行]。

3. 在 [名字] 方塊中，輸入 **Test** 來測試流程。  
    Azure AD B2C 會在視窗頂端顯示錯誤訊息。

    ![測試您的身分識別 API](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-test.png)

4. 在 [名字] 方塊中輸入名字 ("Test" 除外)。  
    Azure AD B2C 會註冊使用者，然後將會員號碼傳送至您的應用程式。 記下此範例中的數字：

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
* 在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)逐步解說之後，建議您使用自己的自訂原則檔案來建置您的情節。 我們已提供[範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw-secure-basic)，供您參考。
* 您可以從[供參考的範例 Visual Studio 解決方案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw/Contoso.AADB2C.API)中下載完整程式碼。

## <a name="next-steps"></a>後續步驟
* [使用用戶端憑證來保護您的 RESTful API](active-directory-b2c-custom-rest-api-netfw-secure-cert.md)

