---
title: Azure Active Directory B2C 中的驗證、註冊、密碼重設 | Microsoft Docs
description: 如何使用 Azure Active Directory B2C 建置支援註冊/登入、設定檔編輯及密碼重設的 Web 應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/17/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 838717aa8f426ea54f4736453aac8c6fcf9a0099
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344492"
---
# <a name="create-an-aspnet-web-app-with-azure-active-directory-b2c-sign-up-sign-in-profile-edit-and-password-reset"></a>建立支援 Azure Active Directory B2C 註冊、登入、設定檔編輯及密碼重設的 ASP.NET Web 應用程式

本教學課程說明如何：

> [!div class="checklist"]
> * 將 Azure AD B2C 身分識別功能新增至您的 Web 應用程式
> * 在您的 Azure AD B2C 目錄中註冊 Web 應用程式
> * 為您的 Web 應用程式建立使用者註冊/登入、編輯設定檔和密碼重設原則

## <a name="prerequisites"></a>必要條件

- 您必須將 B2C 租用戶與 Azure 帳戶連接。 您可以在[這裡](https://azure.microsoft.com/)建立免費的 Azure 帳戶。
- 您需要 [Microsoft Visual Studio](https://www.visualstudio.com/) 或類似的程式來檢視與修改範例程式碼。

## <a name="create-an-azure-ad-b2c-tenant"></a>建立 Azure AD B2C 租用戶

您必須先建立租用戶，才可使用 Azure AD B2C。 租用戶就是您所有使用者、應用程式、群組等項目的容器。 如果您還沒有此資源，請先建立 B2C 租用戶，再繼續進行本指南。

[!INCLUDE [active-directory-b2c-create-tenant](../../includes/active-directory-b2c-create-tenant.md)]

> [!NOTE]
> 
> 您必須將 Azure AD B2C 租用戶與您的 Azure 訂用帳戶連接。 選取 [建立] 後，請選取 [連結現有 Azure AD B2C 租用戶至我的 Azure 訂用帳戶] 選項，然後在 [Azure AD B2C 租用戶] 下拉式清單中選取想要連接的租用戶。

## <a name="create-and-register-an-application"></a>建立並註冊應用程式

接下來需在 Azure AD B2C 租用戶中建立並註冊應用程式。 這會提供必要資訊給 Azure AD B2C，讓它與應用程式安全地通訊。 

選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。 您現在應使用先前建立的租用戶。

[!INCLUDE [active-directory-b2c-register-web-api](../../includes/active-directory-b2c-register-web-api.md)]

完成後，您的應用程式設定中會有 API 與原生應用程式。

## <a name="create-policies-on-your-b2c-tenant"></a>在 B2C 租用戶上建立原則

在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 此程式碼範例包含三種身分識別體驗：**註冊和登入**、**設定檔編輯**和**密碼重設**。  您必須為每個類型建立一個原則，如 [原則參考文章](active-directory-b2c-reference-policies.md)所述。 針對每個原則，請務必選取 [顯示名稱] 屬性或宣告，並複製原則名稱以供稍後使用。

### <a name="add-your-identity-providers"></a>新增識別提供者

從您的設定中，選取 [識別提供者]，然後選擇 [使用者名稱註冊] 或 [電子郵件註冊]。

### <a name="create-a-sign-up-and-sign-in-policy"></a>建立註冊與登入原則

[!INCLUDE [active-directory-b2c-create-sign-in-sign-up-policy](../../includes/active-directory-b2c-create-sign-in-sign-up-policy.md)]

### <a name="create-a-profile-editing-policy"></a>建立設定檔編輯原則

[!INCLUDE [active-directory-b2c-create-profile-editing-policy](../../includes/active-directory-b2c-create-profile-editing-policy.md)]

### <a name="create-a-password-reset-policy"></a>建立密碼重設原則

[!INCLUDE [active-directory-b2c-create-password-reset-policy](../../includes/active-directory-b2c-create-password-reset-policy.md)]

建立您的原則後，就可以開始建置您的應用程式。

## <a name="download-the-sample-code"></a>下載範例程式碼

本教學課程的程式碼保留在 [GitHub](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) 上。 您可以藉由執行下列作業來複製範例︰

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

下載範例程式碼後，請開啟 Visual Studio .sln 檔案開始進行。 您的方案現在包含兩個專案：`TaskWebApp``TaskService`。 `TaskWebApp` 是使用者所互動的 MVC Web 應用程式。 `TaskService` 是應用程式的後端 Web API，其會儲存每位使用者的待辦事項清單。 本文只會討論 `TaskWebApp` 應用程式。 若要了解如何使用 Azure AD B2C 建置 `TaskService`，請參閱 [.NET Web API 教學課程](active-directory-b2c-devquickstarts-api-dotnet.md)。

## <a name="update-code-to-use-your-tenant-and-policies"></a>更新程式碼以使用您的租用戶與原則

我們的範例已設定成使用示範租用戶的原則和用戶端識別碼。 若要將它與您自己的租用戶連接，您必須在 `TaskWebApp` 專案中開啟 `web.config`，然後取代以下的值：

* `ida:Tenant`：使用您的租用戶名稱
* `ida:ClientId`：使用您的 Web 應用程式識別碼
* `ida:ClientSecret`：使用您的 Web 應用程式祕密金鑰
* `ida:SignUpSignInPolicyId`：使用您的「註冊或登入」原則名稱
* `ida:EditProfilePolicyId`：使用您的「編輯設定檔」原則名稱
* `ida:ResetPasswordPolicyId`：使用您的「重設密碼」原則名稱

## <a name="launch-the-app"></a>啟動應用程式
在 Visual Studio 內啟動應用程式。 瀏覽至 [待辦事項清單] 索引標籤，並記下 URl 為： https://*YourTenantName*.b2clogin.com/*YourTenantName*/oauth2/v2.0/authorize?p=*YourSignUpPolicyName*&client_id=*YourclientID*.....

使用電子郵件地址或使用者名稱來註冊應用程式。 登出後再次登入，然後編輯設定檔或重設密碼。 請登出應用程式，再以不同的使用者身分登入， 

## <a name="add-social-idps"></a>新增社交 IDP

目前，應用程式僅支援使用**本機帳戶**的使用者註冊與登入；本機帳戶是指儲存在您 B2C 目錄中使用使用者名稱與密碼的帳戶。 藉由使用 Azure AD B2C，您可以新增對其他 **身分識別提供者** (IDP) 的支援，但不需變更任何程式碼。

若要將社交 IDP 加入至應用程式，請依照下列文章中的詳細指示開始。 針對您想要支援的每個 IDP，您必須在該系統中註冊應用程式並取得用戶端識別碼。

* [將 Facebook 設定為 IDP](active-directory-b2c-setup-fb-app.md)
* [將 Google 設定為 IDP](active-directory-b2c-setup-goog-app.md)
* [將 Amazon 設定為 IDP](active-directory-b2c-setup-amzn-app.md)
* [將 LinkedIn 設定為 IDP](active-directory-b2c-setup-li-app.md)

當您將識別提供者新增到 B2C 目錄之後，請一一編輯您的三個原則以包含新的 IDP，如[原則參考文章](active-directory-b2c-reference-policies.md)中所述。 儲存您的原則之後，重新執行應用程式。  在每一次的身分識別體驗中，您應該會看到新的 IDP 已加入成為登入和註冊選項。

您可以在範例應用程式上試驗原則並觀察其效果。 新增或移除 IDP、管理應用程式宣告，或變更註冊屬性。 試驗到您能夠了解原則、驗證要求和 OWIN 如何結合在一起為止。

## <a name="sample-code-walkthrough"></a>範例程式碼逐步解說
以下各節說明如何設定範例應用程式程式碼。 您日後開發應用程式時，可以將此程式碼當作指引。

### <a name="add-authentication-support"></a>新增驗證支援

您現在可以設定您的應用程式以使用 Azure AD B2C。 您的應用程式會藉由傳送 OpenID Connect 驗證要求，來與 Azure AD B2C 通訊。 要求會藉由指定原則來決定您的應用程式想要執行的使用者經驗。 您可以使用 Microsoft 的 OWIN 程式庫來傳送這些要求、執行原則、管理使用者工作階段等等。

#### <a name="install-owin"></a>安裝 OWIN

首先，請使用 Visual Studio Package Manager Console，將 OWIN 中介軟體 NuGet 封裝加入至專案。

```Console
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
PM> Install-Package Microsoft.Owin.Security.Cookies
PM> Install-Package Microsoft.Owin.Host.SystemWeb
```

#### <a name="add-an-owin-startup-class"></a>新增 OWIN 啟動類別

將 OWIN 啟動類別新增至名為 `Startup.cs` 的 API。  以滑鼠右鍵按一下專案，依序選取 [新增] 和 [新增項目]，然後搜尋 OWIN。 OWIN 中介軟體將會在應用程式啟動時叫用 `Configuration(…)` 方法。

在我們的範例中，我們將類別宣告變更為 `public partial class Startup` 並且在 `App_Start\Startup.Auth.cs` 中實作類別的其他部分。 在 `Configuration` 方法中，我們將呼叫新增至 `ConfigureAuth`(其定義於 `Startup.Auth.cs`)。 修改之後，`Startup.cs` 如下所示︰

```CSharp
// Startup.cs

public partial class Startup
{
    // The OWIN middleware will invoke this method when the app starts
    public void Configuration(IAppBuilder app)
    {
        // ConfigureAuth defined in other part of the class
        ConfigureAuth(app);
    }
}
```

#### <a name="configure-the-authentication-middleware"></a>設定驗證中介軟體

開啟檔案 `App_Start\Startup.Auth.cs` 並實作 `ConfigureAuth(...)` 方法。 您在 `OpenIdConnectAuthenticationOptions` 中所提供的參數會做為座標，讓您的應用程式與 Azure AD B2C 進行通訊。 如果未指定特定參數，它會使用預設值。 例如，我們不在範例中指定 `ResponseType`，因此預設值 `code id_token` 將用於每個對 Azure AD B2C 的外送要求。

您還必須設定 Cookie 驗證。 OpenID Connect 中介軟體會使用 Cookie 維護使用者工作階段，此外還有其他用途。

```CSharp
// App_Start\Startup.Auth.cs

public partial class Startup
{
    // Initialize variables ...

    // Configure the OWIN middleware
    public void ConfigureAuth(IAppBuilder app)
    {
        app.UseCookieAuthentication(new CookieAuthenticationOptions());
        app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

        app.UseOpenIdConnectAuthentication(
            new OpenIdConnectAuthenticationOptions
            {
                // Generate the metadata address using the tenant and policy information
                MetadataAddress = String.Format(AadInstance, Tenant, DefaultPolicy),

                // These are standard OpenID Connect parameters, with values pulled from web.config
                ClientId = ClientId,
                RedirectUri = RedirectUri,
                PostLogoutRedirectUri = RedirectUri,

                // Specify the callbacks for each type of notifications
                Notifications = new OpenIdConnectAuthenticationNotifications
                {
                    RedirectToIdentityProvider = OnRedirectToIdentityProvider,
                    AuthorizationCodeReceived = OnAuthorizationCodeReceived,
                    AuthenticationFailed = OnAuthenticationFailed,
                },

                // Specify the claims to validate
                TokenValidationParameters = new TokenValidationParameters
                {
                    NameClaimType = "name"
                },

                // Specify the scope by appending all of the scopes requested into one string (seperated by a blank space)
                Scope = $"openid profile offline_access {ReadTasksScope} {WriteTasksScope}"
            }
        );
    }

    // Implement the "Notification" methods...
}
```

在上述的 `OpenIdConnectAuthenticationOptions` 中，我們會定義一組 OpenID Connect 中介軟體所收到的特定通知之回呼函式。 這些行為會使用 `OpenIdConnectAuthenticationNotifications` 物件來定義，並儲存至 `Notifications` 變數。 在我們的範例中，我們會根據事件定義三個不同的回呼。

### <a name="using-different-policies"></a>使用不同的原則

`RedirectToIdentityProvider` 在對 Azure AD B2C 要求時就會觸發通知。 在回呼函式 `OnRedirectToIdentityProvider` 中，如果我們想要使用不同的原則，我們會簽入撥出呼叫。 若要執行密碼重設或編輯設定檔，您必須使用對應的原則，例如密碼重設原則，而不是預設的「註冊或登入」原則。

在我們的範例中，當使用者想要重設密碼或編輯設定檔時，我們會新增我們想要在 OWIN 內容中使用的原則。 可以透過下列方式完成︰

```CSharp
    // Let the middleware know you are trying to use the edit profile policy
    HttpContext.GetOwinContext().Set("Policy", EditProfilePolicyId);
```

且您可以藉由執行下列方法實作回呼函式 `OnRedirectToIdentityProvider`︰

```CSharp
/*
*  On each call to Azure AD B2C, check if a policy (e.g. the profile edit or password reset policy) has been specified in the OWIN context.
*  If so, use that policy when making the call. Also, don't request a code (since it won't be needed).
*/
private Task OnRedirectToIdentityProvider(RedirectToIdentityProviderNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> notification)
{
    var policy = notification.OwinContext.Get<string>("Policy");

    if (!string.IsNullOrEmpty(policy) && !policy.Equals(DefaultPolicy))
    {
        notification.ProtocolMessage.Scope = OpenIdConnectScopes.OpenId;
        notification.ProtocolMessage.ResponseType = OpenIdConnectResponseTypes.IdToken;
        notification.ProtocolMessage.IssuerAddress = notification.ProtocolMessage.IssuerAddress.Replace(DefaultPolicy, policy);
    }

    return Task.FromResult(0);
}
```

### <a name="handling-authorization-codes"></a>處理授權碼

收到授權碼時會觸發 `AuthorizationCodeReceived` 通知。 OpenID Connect 中介軟體不支援存取權杖的交換程式碼。 您可以手動交換回呼函式中的權杖程式碼。 如需詳細資訊，請參閱說明做法的[文件](active-directory-b2c-devquickstarts-web-api-dotnet.md)。

### <a name="handling-errors"></a>處理錯誤

驗證失敗時，會觸發 `AuthenticationFailed` 通知。 在其回呼方法中，您可以如您所願處理錯誤。 不過，您應該新增錯誤碼 `AADB2C90118` 的檢查。 在執行「註冊或登入」原則期間，使用者有機會選取 [忘記密碼] 連結。 在此情況下，Azure AD B2C 會將該錯誤代碼傳送給您的應用程式，其指出您的應用程式須改為使用密碼重設原則來進行要求。

```CSharp
/*
* Catch any failures received by the authentication middleware and handle appropriately
*/
private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> notification)
{
    notification.HandleResponse();

    // Handle the error code that Azure AD B2C throws when trying to reset a password from the login page
    // because password reset is not supported by a "sign-up or sign-in policy"
    if (notification.ProtocolMessage.ErrorDescription != null && notification.ProtocolMessage.ErrorDescription.Contains("AADB2C90118"))
    {
        // If the user clicked the reset password link, redirect to the reset password route
        notification.Response.Redirect("/Account/ResetPassword");
    }
    else if (notification.Exception.Message == "access_denied")
    {
        notification.Response.Redirect("/");
    }
    else
    {
        notification.Response.Redirect("/Home/Error?message=" + notification.Exception.Message);
    }

    return Task.FromResult(0);
}
```

### <a name="send-authentication-requests-to-azure-ad"></a>將驗證要求傳送至 Azure AD

您的應用程式現在已正確設定，將使用 OpenID Connect 驗證通訊協定與 Azure AD B2C 進行通訊。 OWIN 會管理有關製作驗證訊息、驗證 Azure AD B2C 的權杖，以及維護使用者工作階段的細節。 剩下的就是啟動每個使用者的流程。

當使用者在 Web 應用程式中選取 [註冊/登入]、[編輯設定檔] 或 [重設密碼] 時，`Controllers\AccountController.cs`中會叫用相關聯的動作：

```CSharp
// Controllers\AccountController.cs

/*
*  Called when requesting to sign up or sign in
*/
public void SignUpSignIn()
{
    // Use the default policy to process the sign up / sign in flow
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge();
        return;
    }

    Response.Redirect("/");
}

/*
*  Called when requesting to edit a profile
*/
public void EditProfile()
{
    if (Request.IsAuthenticated)
    {
        // Let the middleware know you are trying to use the edit profile policy (see OnRedirectToIdentityProvider in Startup.Auth.cs)
        HttpContext.GetOwinContext().Set("Policy", Startup.EditProfilePolicyId);

        // Set the page to redirect to after editing the profile
        var authenticationProperties = new AuthenticationProperties { RedirectUri = "/" };
        HttpContext.GetOwinContext().Authentication.Challenge(authenticationProperties);

        return;
    }

    Response.Redirect("/");

}

/*
*  Called when requesting to reset a password
*/
public void ResetPassword()
{
    // Let the middleware know you are trying to use the reset password policy (see OnRedirectToIdentityProvider in Startup.Auth.cs)
    HttpContext.GetOwinContext().Set("Policy", Startup.ResetPasswordPolicyId);

    // Set the page to redirect to after changing passwords
    var authenticationProperties = new AuthenticationProperties { RedirectUri = "/" };
    HttpContext.GetOwinContext().Authentication.Challenge(authenticationProperties);

    return;
}
```

您也可以使用 OWIN 將使用者登出應用程式。 在 `Controllers\AccountController.cs` 中，我們有︰

```CSharp
// Controllers\AccountController.cs

/*
*  Called when requesting to sign out
*/
public void SignOut()
{
    // To sign out the user, you should issue an OpenIDConnect sign out request.
    if (Request.IsAuthenticated)
    {
        IEnumerable<AuthenticationDescription> authTypes = HttpContext.GetOwinContext().Authentication.GetAuthenticationTypes();
        HttpContext.GetOwinContext().Authentication.SignOut(authTypes.Select(t => t.AuthenticationType).ToArray());
        Request.GetOwinContext().Authentication.GetAuthenticationTypes();
    }
}
```

除了明確叫用原則外，您也可以在控制器中使用會在使用者未登入時執行原則的 `[Authorize]` 標記。 開啟 `Controllers\HomeController.cs`，將 `[Authorize]` 標籤新增至宣告控制器。  在觸及 `[Authorize]` 標籤時，OWIN 會選取所設定的最後一個原則。

```CSharp
// Controllers\HomeController.cs

// You can use the Authorize decorator to execute a certain policy if the user is not already signed into the app.
[Authorize]
public ActionResult Claims()
{
  ...
```

### <a name="display-user-information"></a>顯示使用者資訊

當您使用 OpenID Connect 驗證使用者時，Azure AD B2C 會將包含 **宣告**的 ID 權杖傳回給應用程式。 這些是有關使用者的判斷提示。 您可以使用宣告來個人化應用程式。

開啟 `Controllers\HomeController.cs` 檔案。 您可以透過 `ClaimsPrincipal.Current` 安全性主體物件來存取控制器中的使用者宣告。

```CSharp
// Controllers\HomeController.cs

[Authorize]
public ActionResult Claims()
{
    Claim displayName = ClaimsPrincipal.Current.FindFirst(ClaimsPrincipal.Current.Identities.First().NameClaimType);
    ViewBag.DisplayName = displayName != null ? displayName.Value : string.Empty;
    return View();
}
```

您可以用相同方式存取您的應用程式所接收的任何宣告。  您可以在 [宣告]  頁面看到應用程式收到的所有宣告的清單。
