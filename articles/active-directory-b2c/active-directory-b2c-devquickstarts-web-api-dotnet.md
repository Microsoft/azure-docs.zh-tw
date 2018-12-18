---
title: 在 Azure Active Directory B2C 中呼叫受保護的 ASP.NET Web API | Microsoft Docs
description: 如何使用 Azure Active Directory B2C 和 OAuth 2.0 存取權杖，建置 .NET Web 應用程式並呼叫 Web API。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/17/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 0fd00672e53d0b0148b70b364df5959ced1e554a
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442452"
---
# <a name="azure-ad-b2c-call-a-net-web-api-from-a-net-web-app"></a>Azure AD B2C：從 .NET Web 應用程式呼叫 .NET Web API

使用 Azure AD B2C，您可以將強大的身分識別管理功能新增至 Web 應用程式和 Web API。 本文討論如何要求存取權杖，以及從 .NET「待辦事項清單」Web 應用程式呼叫 .NET Web API。

本文不涵蓋如何使用 Azure AD B2C 實作登入、註冊和管理設定檔。 而會著重在如何在使用者已通過驗證後呼叫 Web API。 如果您尚未執行此動作，您應該：

* 開始使用 [.NET Web 應用程式](active-directory-b2c-devquickstarts-web-dotnet-susi.md)
* 開始使用 [.NET Web API](active-directory-b2c-devquickstarts-api-dotnet.md)

## <a name="prerequisite"></a>必要條件

若要建置呼叫 Web API 的 Web 應用程式，您需要：

1. [建立 Azure AD B2C 租用戶](active-directory-b2c-get-started.md)。
2. [註冊 Web API](active-directory-b2c-app-registration.md#register-a-web-api)。
3. [註冊 Web 應用程式](active-directory-b2c-app-registration.md#register-a-web-app)。
4. [設定原則](active-directory-b2c-reference-policies.md)。
5. [為 Web 應用程式授與權限以使用 Web API](active-directory-b2c-access-tokens.md#publishing-permissions)。

> [!IMPORTANT]
> 用戶端應用程式和 Web API 必須使用相同的 Azure AD B2C 目錄。
>

## <a name="download-the-code"></a>下載程式碼

本教學課程的程式碼保留在 [GitHub](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) 上。 您可以藉由執行下列作業來複製範例︰

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

下載範例程式碼後，請開啟 Visual Studio .sln 檔案開始進行。 您的方案現在包含兩個專案：`TaskWebApp``TaskService`。 `TaskWebApp` 是使用者所互動的 MVC Web 應用程式。 `TaskService` 是應用程式的後端 Web API，其會儲存每位使用者的待辦事項清單。 本文未涵蓋建置 `TaskWebApp` Web 應用程式或 `TaskService` Web API。 若要了解如何使用 Azure AD B2C 建置 .NET Web 應用程式，請參閱 [.NET Web 應用程式教學課程](active-directory-b2c-devquickstarts-web-dotnet-susi.md)。 若要了解如何使用 Azure AD B2C 建置 .NET Web API，請參閱 [.NET Web API 教學課程](active-directory-b2c-devquickstarts-api-dotnet.md)。

### <a name="update-the-azure-ad-b2c-configuration"></a>更新 Azure AD B2C 組態

我們的範例已設定成使用示範租用戶的原則和用戶端識別碼。 如果您想要使用自己的租用戶：

1. 在 `TaskService` 專案中開啟 `web.config`，並取代下列各項的值：

    * `ida:Tenant`：使用您的租用戶名稱
    * `ida:ClientId`：使用您的 Web API 應用程式識別碼
    * `ida:SignUpSignInPolicyId`：使用您的「註冊或登入」原則名稱

2. 在 `TaskWebApp` 專案中開啟 `web.config`，並取代下列各項的值：

    * `ida:Tenant`：使用您的租用戶名稱
    * `ida:ClientId`：使用您的 Web 應用程式識別碼
    * `ida:ClientSecret`：使用您的 Web 應用程式祕密金鑰
    * `ida:SignUpSignInPolicyId`：使用您的「註冊或登入」原則名稱
    * `ida:EditProfilePolicyId`：使用您的「編輯設定檔」原則名稱
    * `ida:ResetPasswordPolicyId`：使用您的「重設密碼」原則名稱



## <a name="requesting-and-saving-an-access-token"></a>要求及儲存存取權杖

### <a name="specify-the-permissions"></a>指定權限

若要呼叫 Web API，您必須驗證使用者 (使用您的註冊/登入原則) 並從 Azure AD B2C [接收存取權杖](active-directory-b2c-access-tokens.md)。 若要接收存取權杖，您必須先指定您想要授與存取權杖的權限。 當您對 `/authorize` 端點提出要求時，權限是指定於 `scope` 參數中。 例如，若要取得具備資源應用程式 (其應用程式識別碼 URI 為 `https://contoso.onmicrosoft.com/tasks`) 之「讀取」權限的存取權杖，則範圍會是 `https://contoso.onmicrosoft.com/tasks/read`。

若要在我們的範例中指定範圍，請開啟檔案 `App_Start\Startup.Auth.cs`，並在 OpenIdConnectAuthenticationOptions 中定義 `Scope` 變數。

```CSharp
// App_Start\Startup.Auth.cs

    app.UseOpenIdConnectAuthentication(
        new OpenIdConnectAuthenticationOptions
        {
            ...

            // Specify the scope by appending all of the scopes requested into one string (seperated by a blank space)
            Scope = $"{OpenIdConnectScopes.OpenId} {ReadTasksScope} {WriteTasksScope}"
        }
    );
}
```

### <a name="exchange-the-authorization-code-for-an-access-token"></a>交換存取權杖的授權碼

使用者完成註冊或登入體驗之後，您的應用程式將會收到來自 Azure AD B2C 的授權碼。 OWIN OpenID Connect 中介軟體將儲存此授權碼，但不會針對存取權杖進行交換。 您可以使用 [MSAL 程式庫 (英文)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) 進行交換。 在我們的範例中，我們設定了每次收到授權碼時對 OpenID Connect 中介軟體的回呼通知。 在回呼中，我們使用 MSAL 來交換權杖的授權碼，並將權杖儲存到快取。

```CSharp
/*
* Callback function when an authorization code is received
*/
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
{
    // Extract the code from the response notification
    var code = notification.Code;

    var userObjectId = notification.AuthenticationTicket.Identity.FindFirst(ObjectIdElement).Value;
    var authority = String.Format(AadInstance, Tenant, DefaultPolicy);
    var httpContext = notification.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase;

    // Exchange the code for a token. Make sure to specify the necessary scopes
    ClientCredential cred = new ClientCredential(ClientSecret);
    ConfidentialClientApplication app = new ConfidentialClientApplication(authority, Startup.ClientId,
                                            RedirectUri, cred, new NaiveSessionCache(userObjectId, httpContext));
    var authResult = await app.AcquireTokenByAuthorizationCodeAsync(new string[] { ReadTasksScope, WriteTasksScope }, code, DefaultPolicy);
}
```

## <a name="calling-the-web-api"></a>呼叫 Web API

本節將討論如何使用註冊/登入期間所收到的權杖與 Azure AD B2C，以便存取 Web API。

### <a name="retrieve-the-saved-token-in-the-controllers"></a>擷取控制器中已儲存的權杖

`TasksController` 負責與 Web API 通訊，並傳送 HTTP 要求給 API 來讀取、建立和刪除工作。 因為 API 受到 Azure AD B2C 的保護，您必須先擷取您在上一個步驟中儲存的權杖。

```CSharp
// Controllers\TasksController.cs

/*
* Uses MSAL to retrieve the token from the cache
*/
private async void acquireToken(String[] scope)
{
    string userObjectID = ClaimsPrincipal.Current.FindFirst(Startup.ObjectIdElement).Value;
    string authority = String.Format(Startup.AadInstance, Startup.Tenant, Startup.DefaultPolicy);

    ClientCredential credential = new ClientCredential(Startup.ClientSecret);

    // Retrieve the token using the provided scopes
    ConfidentialClientApplication app = new ConfidentialClientApplication(authority, Startup.ClientId,
                                        Startup.RedirectUri, credential,
                                        new NaiveSessionCache(userObjectID, this.HttpContext));
    AuthenticationResult result = await app.AcquireTokenSilentAsync(scope);

    accessToken = result.Token;
}
```

### <a name="read-tasks-from-the-web-api"></a>從 Web API 讀取工作

當您擁有權杖之後，就可以把權杖附加 HTTP `GET` 要求的 `Authorization` 標頭中，以安全地呼叫 `TaskService`：

```CSharp
// Controllers\TasksController.cs

public async Task<ActionResult> Index()
{
    try {

        // Retrieve the token with the specified scopes
        acquireToken(new string[] { Startup.ReadTasksScope });

        HttpClient client = new HttpClient();
        HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, apiEndpoint);

        // Add token to the Authorization header and make the request
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
        HttpResponseMessage response = await client.SendAsync(request);

        // Handle response ...
}

```

### <a name="create-and-delete-tasks-on-the-web-api"></a>在 Web API 上建立及刪除工作

請遵循與您傳送 `POST` 和 `DELETE` 要求給 Web API 時的相同模式，使用 MSAL 從快取擷取存取權杖。

## <a name="run-the-sample-app"></a>執行範例應用程式

最後，建置並執行這兩個應用程式。 註冊和登入，並為登入的使用者建立工作。 請登出應用程式，再以不同的使用者身分登入， 然後為該使用者建立工作。 您會發現，這些工作在 API 上是依照使用者來儲存的 ，因為 API 會從自己收到的權杖中擷取使用者的身分識別。 也可以嘗試使用範圍。 移除「寫入」的權限，然後嘗試新增一項工作。 只需確定每次您變更範圍時會登出。

