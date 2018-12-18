---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: 99eabd8f9c9b3ab86c348350e8924cea0eb668ba
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843057"
---
## <a name="set-up-your-project"></a>設定專案

本節說明使用 OpenID Connect 在 ASP.NET 專案中，透過 OWIN 中介軟體安裝及設定驗證管線的步驟。 

> 想要改為下載此範例的 Visual Studio 專案嗎？ [下載專案](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip)並跳至[設定步驟](#register-your-application)，以在執行之前先設定程式碼範例。

### <a name="create-your-aspnet-project"></a>建立 ASP.NET 專案

1. 在 Visual Studio 中：`File` > `New` > `Project`
2. 在 *Visual C#\Web* 底下，選取 `ASP.NET Web Application (.NET Framework)`。
3. 為您的應用程式命名並按一下 [確定]
4. 選取 `Empty` 並選取核取方塊以新增 `MVC` 參考

## <a name="add-authentication-components"></a>新增驗證元件

1. 在 Visual Studio 中：`Tools` > `Nuget Package Manager` > `Package Manager Console`
2. 在 [套件管理器主控台] 視窗中輸入下列內容以新增「OWIN 中介軟體 NuGet 套件」：

    ```powershell
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Owin.Host.SystemWeb
    ```

<!--start-collapse-->
> ### <a name="about-these-libraries"></a>關於這些程式庫
>上面的程式庫可透過 Cookie 型驗證，使用 OpenID Connect 啟用單一登入 (SSO)。 完成驗證並將代表使用者的權杖傳送至您的應用程式之後，OWIN 中介軟體就會建立工作階段 Cookie。 接著瀏覽器會在後續要求中使用此 Cookie，因此使用者不需要重新輸入密碼，也不需要進行其他驗證。
<!--end-collapse-->

## <a name="configure-the-authentication-pipeline"></a>設定驗證管線
下面的步驟是用來建立 OWIN 中介軟體啟動類別，以設定 OpenID Connect 驗證。 此類別將會在您的 IIS 處理序啟動時自動執行。

> [!TIP]
> 如果您專案的根資料夾中沒有 `Startup.cs` 檔案：
> 1. 在專案的根資料夾上按一下滑鼠右鍵： > `Add` > `New Item...` > `OWIN Startup class`<br/>
> 2. 將它命名為 `Startup.cs`
>
>> 確定選取的類別是 OWIN 啟動類別，而非標準 C# 類別。 如果命名空間上方顯示 `[assembly: OwinStartup(typeof({NameSpace}.Startup))]`，請勾選以確認此項目。

1. 將 *OWIN* 和 *Microsoft.IdentityModel* 參考新增至 `Startup.cs`：

    ```csharp
    using Microsoft.Owin;
    using Owin;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.IdentityModel.Tokens;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Microsoft.Owin.Security.Notifications;
    ```

2. 用下面的程式碼取代啟動類別：

    ```csharp
    public class Startup
    {
        // The Client ID is used by the application to uniquely identify itself to Azure AD.
        string clientId = System.Configuration.ConfigurationManager.AppSettings["ClientId"];

        // RedirectUri is the URL where the user will be redirected to after they sign in.
        string redirectUri = System.Configuration.ConfigurationManager.AppSettings["RedirectUri"];

        // Tenant is the tenant ID (e.g. contoso.onmicrosoft.com, or 'common' for multi-tenant)
        static string tenant = System.Configuration.ConfigurationManager.AppSettings["Tenant"];

        // Authority is the URL for authority, composed by Azure Active Directory v2 endpoint and the tenant name (e.g. https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0)
        string authority = String.Format(System.Globalization.CultureInfo.InvariantCulture, System.Configuration.ConfigurationManager.AppSettings["Authority"], tenant);

        /// <summary>
        /// Configure OWIN to use OpenIdConnect 
        /// </summary>
        /// <param name="app"></param>
        public void Configuration(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());
            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {
                    // Sets the ClientId, authority, RedirectUri as obtained from web.config
                    ClientId = clientId,
                    Authority = authority,
                    RedirectUri = redirectUri,
                    // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it is using the home page
                    PostLogoutRedirectUri = redirectUri,
                    Scope = OpenIdConnectScope.OpenIdProfile,
                    // ResponseType is set to request the id_token - which contains basic information about the signed-in user
                    ResponseType = OpenIdConnectResponseType.IdToken,
                    // ValidateIssuer set to false to allow personal and work accounts from any organization to sign in to your application
                    // To only allow users from a single organizations, set ValidateIssuer to true and 'tenant' setting in web.config to the tenant name
                    // To allow users from only a list of specific organizations, set ValidateIssuer to true and use ValidIssuers parameter 
                    TokenValidationParameters = new TokenValidationParameters()
                    {
                        ValidateIssuer = false
                    },
                    // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to OnAuthenticationFailed method
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthenticationFailed = OnAuthenticationFailed
                    }
                }
            );
        }

        /// <summary>
        /// Handle failed authentication requests by redirecting the user to the home page with an error in the query string
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> context)
        {
            context.HandleResponse();
            context.Response.Redirect("/?errormessage=" + context.Exception.Message);
            return Task.FromResult(0);
        }
    }
    ```

<!--start-collapse-->
> ### <a name="more-information"></a>相關資訊
> 您在 *OpenIDConnectAuthenticationOptions* 中提供的參數會作為供應用程式與 Azure AD 進行通訊的座標。 因為 OpenID Connect 中介軟體會在背景中使用 Cookie，所以您也必須設定在上述程式碼顯示時進行 Cookie 驗證。 *ValidateIssuer* 值會告知 OpenIdConnect 不要針對某一特定組織限制存取。
<!--end-collapse-->

