---
title: 在 Azure Active Directory B2C 中保護 ASP.NET Web API | Microsoft Docs
description: 如何使用 Azure Active Directory B2C 建置 .NET Web API，並使用 OAuth 2.0 存取權杖進行驗證加以保護。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 01/14/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 2e93a8340699d1fcf68c53baa87990e799bc933d
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37447572"
---
# <a name="azure-active-directory-b2c-build-a-net-web-api"></a>Azure Active Directory B2C：建置 .NET Web API

透過 Azure Active Directory (Azure AD) B2C，您可以使用 OAuth 2.0 存取權杖來保護 Web API。 這些權杖可讓您的用戶端應用程式對 API 進行驗證。 本文將說明如何建立 .NET MVC「待辦事項清單」API，以便用戶端應用程式的使用者進行 CRUD 工作。 Web API 會使用 Azure AD B2C 進行保護，只允許已驗證的使用者管理他們的待辦事項清單。

## <a name="create-an-azure-ad-b2c-directory"></a>建立 Azure AD B2C 目錄

您必須先建立目錄或租用戶，才可使用 Azure AD B2C。 目錄是適用於所有使用者、app、群組等項目的容器。 如果您還沒有此資源，請先 [建立 B2C 目錄](active-directory-b2c-get-started.md) ，再繼續進行本指南。

> [!NOTE]
> 用戶端應用程式和 Web API 必須使用相同的 Azure AD B2C 目錄。
>

## <a name="create-a-web-api"></a>建立 Web API

接著，您必須在 B2C 目錄中建立 Web API 應用程式。 這會提供必要資訊給 Azure AD，讓它與應用程式安全地通訊。 若要建立應用程式，請遵循 [這些指示](active-directory-b2c-app-registration.md)。 請務必：

* 在應用程式中加入 **Web 應用程式**或 **Web API**。
* 使用 Web 應用程式的 [重新導向 URI] `https://localhost:44332/`。 這是適用於此程式碼範例的 Web 應用程式用戶端的預設位置。
* 複製指派給您的應用程式的 **應用程式識別碼** 。 稍後您將會用到此資訊。
* 在 [應用程式識別碼 URI] 中輸入應用程式識別碼。 複製完整的**應用程式識別碼 URI**。 稍後您將會用到此資訊。
* 透過 [發佈的範圍] 功能表新增權限。

## <a name="create-your-policies"></a>建立您的原則

在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 您必須建立原則，才能與 Azure AD B2C 通訊。 我們建議使用合併的註冊/登入原則 (如[原則參考文章](active-directory-b2c-reference-policies.md)所述)。 建立原則時，請務必：

* 選擇 [顯示名稱]  和原則中的其他註冊屬性。
* 針對每個原則選擇 [顯示名稱] 和 [物件識別碼] 宣告做為應用程式宣告。 您也可以選擇其他宣告。
* 在您建立每個原則之後，請複製原則的 [名稱]  。 您稍後需要用到此原則名稱。

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

當您成功建立原則之後，就可以開始建置您的應用程式。

## <a name="download-the-code"></a>下載程式碼

本教學課程的程式碼保留在 [GitHub](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) 上。 您可以藉由執行下列作業來複製範例︰

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

下載範例程式碼後，請開啟 Visual Studio .sln 檔案開始進行。 您的方案現在包含兩個專案：`TaskWebApp``TaskService`。 `TaskWebApp` 是使用者所互動的 MVC Web 應用程式。 `TaskService` 是應用程式的後端 Web API，其會儲存每位使用者的待辦事項清單。 本文只會討論 `TaskService` 應用程式。 若要了解如何使用 Azure AD B2C 建置 `TaskWebApp`，請參閱 [.NET Web 應用程式教學課程](active-directory-b2c-devquickstarts-web-dotnet-susi.md)。

### <a name="update-the-azure-ad-b2c-configuration"></a>更新 Azure AD B2C 組態

我們的範例已設定成使用示範租用戶的原則和用戶端識別碼。 如果您想要使用自己的租用戶，您必須執行下列作業︰

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
    * `api:ApiIdentifier`：使用您的「應用程式識別碼 URI」


## <a name="secure-the-api"></a>保護 API

當您有可呼叫 API 的用戶端時，您可以使用 OAuth 2.0 持有人權杖來保護 API (例如 `TaskService`)。 這可確保唯有要求具備持有人權杖時，API 的每個要求才有效。 您的 API 可以使用 Microsoft 的 Open Web Interface for .NET (OWIN) 程式庫來接受並驗證持有人權杖。

### <a name="install-owin"></a>安裝 OWIN

首先，使用 Visual Studio Package Manager Console 來安裝 OWIN OAuth 驗證管線。

```Console
PM> Install-Package Microsoft.Owin.Security.OAuth -ProjectName TaskService
PM> Install-Package Microsoft.Owin.Security.Jwt -ProjectName TaskService
PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TaskService
```

這會安裝 OWIN 中介軟體，以便接受並驗證持有人權杖。

### <a name="add-an-owin-startup-class"></a>新增 OWIN 啟動類別

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

### <a name="configure-oauth-20-authentication"></a>設定 OAuth 2.0 驗證

開啟 `App_Start\Startup.Auth.cs` 檔案，然後實作 `ConfigureAuth(...)` 方法。 例如，它可能如下所示︰

```CSharp
// App_Start\Startup.Auth.cs

 public partial class Startup
    {
        // These values are pulled from web.config
        public static string AadInstance = ConfigurationManager.AppSettings["ida:AadInstance"];
        public static string Tenant = ConfigurationManager.AppSettings["ida:Tenant"];
        public static string ClientId = ConfigurationManager.AppSettings["ida:ClientId"];
        public static string SignUpSignInPolicy = ConfigurationManager.AppSettings["ida:SignUpSignInPolicyId"];
        public static string DefaultPolicy = SignUpSignInPolicy;

        /*
         * Configure the authorization OWIN middleware.
         */
        public void ConfigureAuth(IAppBuilder app)
        {
            TokenValidationParameters tvps = new TokenValidationParameters
            {
                // Accept only those tokens where the audience of the token is equal to the client ID of this app
                ValidAudience = ClientId,
                AuthenticationType = Startup.DefaultPolicy
            };

            app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
            {
                // This SecurityTokenProvider fetches the Azure AD B2C metadata & signing keys from the OpenIDConnect metadata endpoint
                AccessTokenFormat = new JwtFormat(tvps, new OpenIdConnectCachingSecurityTokenProvider(String.Format(AadInstance, Tenant, DefaultPolicy)))
            });
        }
    }
```

### <a name="secure-the-task-controller"></a>保護工作控制器

將應用程式設定為使用 OAuth 2.0 驗證之後，只需將 `[Authorize]` 標記加入工作控制器，就能保護您的 Web API。 所有的待辦事項清單操作都會在此控制器中進行，因此您應該在類別層級上保護整個控制器。 您也可以將 `[Authorize]` 標記加入個別的動作，以進行更細微的控制。

```CSharp
// Controllers\TasksController.cs

[Authorize]
public class TasksController : ApiController
{
    ...
}
```

### <a name="get-user-information-from-the-token"></a>從權杖取得使用者資訊

`TasksController` 會將工作儲存在資料庫中，而每個工作都有一個「擁有」此工作的相關聯使用者。 擁有者是藉由使用者的**物件識別碼**來識別。 (這就是為什麼您需要在所有原則中新增物件識別碼以做為應用程式宣告的原因)。

```CSharp
// Controllers\TasksController.cs

public IEnumerable<Models.Task> Get()
{
    string owner = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
    IEnumerable<Models.Task> userTasks = db.Tasks.Where(t => t.owner == owner);
    return userTasks;
}
```

### <a name="validate-the-permissions-in-the-token"></a>驗證權杖中的權限

Web API 的一個常見需求是驗證權杖中是否有「範圍」存在。 這可確保使用者已對存取待辦事項清單服務所需的權限表示同意。

```CSharp
public IEnumerable<Models.Task> Get()
{
    if (ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/scope").Value != "read")
    {
        throw new HttpResponseException(new HttpResponseMessage {
            StatusCode = HttpStatusCode.Unauthorized,
            ReasonPhrase = "The Scope claim does not contain 'read' or scope claim not found"
        });
    }
    ...
}
```

## <a name="run-the-sample-app"></a>執行範例應用程式

最後，請建置並執行 `TaskWebApp` 和 `TaskService`。 在使用者的待辦事項清單中建立一些工作，觀察即使停止並重新啟動用戶端之後，這些工作仍持續存在 API 中。

## <a name="edit-your-policies"></a>編輯您的原則

在您使用 Azure AD B2C 來保護 API 之後，就可以試驗登入/註冊原則，並檢視對 API 產生的效果 (或沒有效果)。 您可以操作原則中的應用程式宣告，然後變更 Web API 中可用的使用者資訊。 如本文先前所述，您加入的任何宣告都可在 `ClaimsPrincipal` 物件中供您的 .NET MVC Web API 使用。
