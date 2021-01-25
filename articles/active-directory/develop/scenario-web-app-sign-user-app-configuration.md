---
title: 設定用來登入使用者的 web 應用程式 |蔚藍
titleSuffix: Microsoft identity platform
description: '瞭解如何建立 web 應用程式，以 (程式碼設定登入使用者) '
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 45f3a066283a921f60909a4aa3cfdc76f3faad06
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98753262"
---
# <a name="web-app-that-signs-in-users-code-configuration"></a>登入使用者的 Web 應用程式：程式碼設定

瞭解如何設定 web 應用程式的程式碼以登入使用者。

## <a name="libraries-for-protecting-web-apps"></a>用來保護 web 應用程式的程式庫

<!-- This section can be in an include for web app and web APIs -->
用來保護 web 應用程式 (和 web API) 的程式庫為：

| 平台 | 程式庫 | 描述 |
|----------|---------|-------------|
| ![.NET](media/sample-v2-code/logo_NET.png) | [適用于 .NET 的身分識別模型延伸模組](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) | ASP.NET 和 ASP.NET Core 直接使用，適用于 .NET 的 Microsoft 身分識別模型延伸模組會建議一組在 .NET Framework 和 .NET Core 上執行的 Dll。 從 ASP.NET 或 ASP.NET Core web 應用程式，您可以使用 **TokenValidationParameters** (類別來控制權杖驗證，特別是在某些) 的合作夥伴案例中。 在實務上，複雜度會封裝在 [web.config](https://aka.ms/ms-identity-web) 中。 |
| ![Java](media/sample-v2-code/small_logo_java.png) | [MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki) | JAVA web 應用程式的支援 |
| ![Python](media/sample-v2-code/small_logo_python.png) | [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | Python web 應用程式的支援 |

選取對應至您感興趣之平臺的索引標籤：

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

本文中的程式碼片段和下列程式碼會從 [ASP.NET Core 的 web 應用程式增量教學課程（第1章）](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-1-MyOrg)進行解壓縮。

您可以參考此教學課程，以取得完整的執行詳細資料。

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

本文中的程式碼片段和下列程式碼會從 [ASP.NET web 應用程式範例](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect)中解壓縮。

您可能會想要參考此範例以取得完整的執行詳細資料。

# <a name="java"></a>[Java](#tab/java)

本文中的程式碼片段和下列程式碼會從 [java web 應用程式](https://github.com/Azure-Samples/ms-identity-java-webapp) （在 MSAL java 中呼叫 Microsoft graph 範例）進行解壓縮。

您可能會想要參考此範例以取得完整的執行詳細資料。

# <a name="python"></a>[Python](#tab/python)

本文中的程式碼片段和下列程式碼會從在 MSAL Python 中 [呼叫 Microsoft graph 範例的 python web 應用程式](https://github.com/Azure-Samples/ms-identity-python-webapp) 中解壓縮。

您可能會想要參考此範例以取得完整的執行詳細資料。

---

## <a name="configuration-files"></a>組態檔

使用 Microsoft 身分識別平臺來登入使用者的 Web 應用程式，是透過設定檔進行設定。 您需要填寫的設定如下：

- `Instance`如果您想要讓應用程式在國家雲端中執行（例如，），雲端實例 () 
- 租使用者識別碼中的物件 (`TenantId`) 
- 從 Azure 入口網站複製的應用程式的用戶端識別碼 (`ClientId`) 

有時候，應用程式可以由和的串連來參數化 `Authority` `Instance` `TenantId` 。

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

在 ASP.NET Core 中，這些設定位於 [檔案中的 [appsettings.js](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/appsettings.json#L2-L8) ] 區段的 [AzureAd] 區段中。

```Json
{
  "AzureAd": {
    // Azure cloud instance among:
    // - "https://login.microsoftonline.com/" for Azure public cloud
    // - "https://login.microsoftonline.us/" for Azure US government
    // - "https://login.microsoftonline.de/" for Azure AD Germany
    // - "https://login.chinacloudapi.cn/" for Azure AD China operated by 21Vianet
    "Instance": "https://login.microsoftonline.com/",

    // Azure AD audience among:
    // - "TenantId" as a GUID obtained from the Azure portal to sign in users in your organization
    // - "organizations" to sign in users in any work or school account
    // - "common" to sign in users with any work or school account or Microsoft personal account
    // - "consumers" to sign in users with a Microsoft personal account only
    "TenantId": "[Enter the tenantId here]",

    // Client ID (application ID) obtained from the Azure portal
    "ClientId": "[Enter the Client Id]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath": "/signout-oidc"
  }
}
```

在 ASP.NET Core 中，) 上的另一個 ([properties\launchSettings.js](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/Properties/launchSettings.json#L6-L7) 的 `applicationUrl` 檔案包含您的 `sslPort` 應用程式和各種設定檔的 URL () 和 TLS/SSL 埠 () 。

```Json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3110/",
      "sslPort": 44321
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "webApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:3110/"
    }
  }
}
```

在 Azure 入口網站中，您必須在應用程式的 [ **驗證** ] 頁面上註冊的回復 uri 必須符合這些 url。 針對上述兩個設定檔，其為 `https://localhost:44321/signin-oidc` 。 原因是 `applicationUrl` `http://localhost:3110` ，但 `sslPort` (44321) 指定。 `CallbackPath` 是 `/signin-oidc` ，如中所定義 `appsettings.json` 。

以同樣的方式，登出 URI 會設定為 `https://localhost:44321/signout-oidc` 。

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

在 ASP.NET 中，應用程式是透過 [Web.config](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Web.config#L12-L15) 檔案（行12到15）來設定。

```XML
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:ClientId" value="[Enter your client ID, as obtained from the app registration portal]" />
    <add key="ida:ClientSecret" value="[Enter your client secret, as obtained from the app registration portal]" />
    <add key="ida:AADInstance" value="https://login.microsoftonline.com/{0}{1}" />
    <add key="ida:RedirectUri" value="https://localhost:44326/" />
    <add key="vs:EnableBrowserLink" value="false" />
  </appSettings>
```

在 Azure 入口網站中，您必須在應用程式的 [ **驗證** ] 頁面上註冊的回復 uri 必須符合這些 url。 也就是說，它們應該是 `https://localhost:44326/` 。

# <a name="java"></a>[Java](#tab/java)

在 JAVA 中，設定位於位於底下的 [應用程式。屬性](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/resources/application.properties) 檔 `src/main/resources` 。

```Java
aad.clientId=Enter_the_Application_Id_here
aad.authority=https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/
aad.secretKey=Enter_the_Client_Secret_Here
aad.redirectUriSignin=http://localhost:8080/msal4jsample/secure/aad
aad.redirectUriGraph=http://localhost:8080/msal4jsample/graph/me
```

在 Azure 入口網站中，您必須在應用程式的 [ **驗證** ] 頁面上註冊的回復 uri 必須符合 `redirectUri` 應用程式所定義的實例。 也就是說，它們應該是 `http://localhost:8080/msal4jsample/secure/aad` 和 `http://localhost:8080/msal4jsample/graph/me` 。

# <a name="python"></a>[Python](#tab/python)

以下是 [app_config .py](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.1.0/app_config.py)中的 Python 設定檔：

```Python
CLIENT_SECRET = "Enter_the_Client_Secret_Here"
AUTHORITY = "https://login.microsoftonline.com/common""
CLIENT_ID = "Enter_the_Application_Id_here"
ENDPOINT = 'https://graph.microsoft.com/v1.0/users'
SCOPE = ["User.ReadBasic.All"]
SESSION_TYPE = "filesystem"  # So the token cache will be stored in a server-side session
```

> [!NOTE]
> 為了簡單起見，本快速入門建議您將用戶端密碼儲存在設定檔中。 在您的生產環境應用程式中，您會想要使用其他方式來儲存您的密碼，例如金鑰保存庫或環境變數，如 [Flask 的檔](https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables)所述。
>
> ```python
> CLIENT_SECRET = os.getenv("CLIENT_SECRET")
> if not CLIENT_SECRET:
>     raise ValueError("Need to define CLIENT_SECRET environment variable")
> ```

---

## <a name="initialization-code"></a>初始化程式碼

初始化程式碼會根據平臺而有所不同。 針對 ASP.NET Core 和 ASP.NET，登入使用者會委派給 OpenID Connect 中介軟體。 ASP.NET 或 ASP.NET Core 範本會產生 Azure Active Directory (Azure AD) v1.0 端點的 web 應用程式。 需要進行一些設定，才能將其調整為 Microsoft 身分識別平臺。 在 JAVA 的案例中，它會與應用程式的合作進行春季處理。

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

在 ASP.NET Core web apps (和 web Api) 中，應用程式會受到保護，因為您在 `[Authorize]` 控制器或控制器動作上有屬性。 此屬性會檢查使用者是否經過驗證。 正在初始化應用程式的程式碼位於 *Startup.cs* 檔案中。

若要使用 Microsoft 身分識別平臺新增驗證 (先前 Azure AD v2.0) ，您將需要新增下列程式碼。 程式碼中的批註應該一目了然。

> [!NOTE]
> 如果您想要直接開始使用 Microsoft 身分識別平臺的新 ASP.NET Core 範本，以利用 Web.config，您可以下載包含 .NET Core 3.1 和 .NET 5.0 專案範本的預覽 NuGet 套件。 然後，安裝之後，您就可以直接將 ASP.NET Core 的 web 應用程式具現化 (MVC 或 Blazor) 。 如需詳細資訊，請參閱 [Microsoft web 應用程式專案範本](https://aka.ms/ms-id-web/webapp-project-templates) 。 這是最簡單的方法，因為它會為您執行下列所有步驟。
>
> 如果您想要使用 Visual Studio 中的目前預設 ASP.NET Core Web 專案來啟動專案，或使用 `dotnet new mvc --auth SingleAuth` 或 `dotnet new webapp --auth SingleAuth` ，您將會看到如下所示的程式碼：
>
>```c#
>  services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
>          .AddAzureAD(options => Configuration.Bind("AzureAd", options));
> ```
>
> 此程式碼會使用舊版的 **AspNetCore AzureAD** ，這是用來建立 Azure AD v1.0 應用程式的 NuGet 套件。 本文說明如何建立 Microsoft 身分識別平臺 (Azure AD v2.0) 應用程式，以取代該程式碼。
>

1. 將 [ [web](https://www.nuget.org/packages/Microsoft.Identity.Web) ] 和 [[web.config] NuGet 套件](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) 新增至您的專案。 移除 AspNetCore AzureAD. UI NuGet 套件（如果有的話）。

2. 更新中的 `ConfigureServices` 程式碼，使其使用 `AddMicrosoftIdentityWebAppAuthentication` 和 `AddMicrosoftIdentityUI` 方法。

   ```c#
   public class Startup
   {
    ...
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
     services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAd");

     services.AddRazorPages().AddMvcOptions(options =>
     {
      var policy = new AuthorizationPolicyBuilder()
                    .RequireAuthenticatedUser()
                    .Build();
      options.Filters.Add(new AuthorizeFilter(policy));
     }).AddMicrosoftIdentityUI();
    ```

3. 在 `Configure` *Startup.cs* 的方法中，使用呼叫來啟用驗證 `app.UseAuthentication();`

   ```c#
   // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
    // more code here
    app.UseAuthentication();
    app.UseAuthorization();
    // more code here
   }
   ```

在上述程式碼中：
- `AddMicrosoftIdentityWebAppAuthentication`擴充方法是在 **web.config** 中定義。 強烈
  - 新增驗證服務。
  - 設定選項以從 "AzureAD" 區段讀取設定檔 () 
  - 設定 OpenID Connect 選項，讓授權單位是 Microsoft 身分識別平臺。
  - 驗證權杖的簽發者。
  - 確保對應至名稱的宣告會從識別碼權杖中的宣告進行對應 `preferred_username` 。

- 除了設定物件之外，您還可以在呼叫時指定設定區段的名稱 `AddMicrosoftIdentityWebAppAuthentication` 。 依預設，它是 `AzureAd` 。

- `AddMicrosoftIdentityWebAppAuthentication` 具有 advanced 案例的其他參數。 例如，追蹤 OpenID Connect 中介軟體事件可協助您疑難排解 web 應用程式（如果驗證無法運作）。 將選擇性參數設定 `subscribeToOpenIdConnectMiddlewareDiagnosticsEvents` 為， `true` 會顯示 ASP.NET Core 中介軟體從 HTTP 回應到中的使用者身分識別時，如何處理資訊 `HttpContext.User` 。

- `AddMicrosoftIdentityUI`擴充方法是在 **web.config** 中定義。 它提供預設控制器來處理登入和登出。

您可以找到更多有關如何讓您在中建立 web 應用程式的詳細資料。 <https://aka.ms/ms-id-web/webapp>

> [!WARNING]
> 目前，當您使用 Azure AD 作為和外部登入提供者時， (將使用者帳戶儲存在應用程式) 中，則目前不支援 **個別使用者帳戶** 的案例。 如需詳細資訊，請參閱： [AzureAD/microsoft-identity-web # 133](https://github.com/AzureAD/microsoft-identity-web/issues/133)

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

ASP.NET web 應用程式和 web Api 中的驗證相關程式碼位於 [App_Start/startup.auth.cs](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/App_Start/Startup.Auth.cs#L17-L61) 檔中。

```csharp
 public void ConfigureAuth(IAppBuilder app)
 {
  app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

  app.UseCookieAuthentication(new CookieAuthenticationOptions());

  app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
     // Authority` represents the identity platform endpoint - https://login.microsoftonline.com/common/v2.0.
     // `Scope` describes the initial permissions that your app will need.
     //  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/.
     ClientId = clientId,
     Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0"),
     RedirectUri = redirectUri,
     Scope = "openid profile",
     PostLogoutRedirectUri = redirectUri,
    });
 }
```

# <a name="java"></a>[Java](#tab/java)

JAVA 範例會使用春季架構。 應用程式會受到保護，因為您會執行可攔截每個 HTTP 回應的篩選準則。 在 JAVA web 應用程式的快速入門中，此篩選器 `AuthFilter` 在中 `src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java` 。

篩選器會處理 OAuth 2.0 授權碼流程，並檢查使用者是否已)  (`isAuthenticated()` 方法進行驗證。 如果使用者未經過驗證，則會計算 Azure AD 授權端點的 URL，並將瀏覽器重新導向至這個 URI。

當回應抵達（包含授權碼）時，它會使用 MSAL JAVA 取得權杖。 當它最後收到權杖端點的權杖時 (重新導向 URI) 上的權杖，使用者就會登入。

如需詳細資訊，請參閱 `doFilter()` [AuthFilter](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/master/msal-java-webapp-sample/src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java)中的方法。

> [!NOTE]
> 的 `doFilter()` 程式碼是以稍有不同的順序撰寫，但流程則是所述。

如需此方法觸發之授權碼流程的詳細資訊，請參閱 [Microsoft 身分識別平臺和 OAuth 2.0 授權碼流程](v2-oauth2-auth-code-flow.md)。

# <a name="python"></a>[Python](#tab/python)

Python 範例會使用 Flask。 Flask 和 MSAL Python 的初始化是在 [.py # L1-L28](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L1-L28)中完成。

```Python
import uuid
import requests
from flask import Flask, render_template, session, request, redirect, url_for
from flask_session import Session  # https://pythonhosted.org/Flask-Session
import msal
import app_config


app = Flask(__name__)
app.config.from_object(app_config)
Session(app)
```

---

## <a name="next-steps"></a>後續步驟

在下一篇文章中，您將瞭解如何觸發登入和登出。

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

請移至本案例的下一篇文章，登 [入並登出](./scenario-web-app-sign-user-sign-in.md?tabs=aspnetcore)。

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

請移至本案例的下一篇文章，登 [入並登出](./scenario-web-app-sign-user-sign-in.md?tabs=aspnet)。

# <a name="java"></a>[Java](#tab/java)

請移至本案例的下一篇文章，登 [入並登出](./scenario-web-app-sign-user-sign-in.md?tabs=java)。

# <a name="python"></a>[Python](#tab/python)

請移至本案例的下一篇文章，登 [入並登出](./scenario-web-app-sign-user-sign-in.md?tabs=python)。

---
