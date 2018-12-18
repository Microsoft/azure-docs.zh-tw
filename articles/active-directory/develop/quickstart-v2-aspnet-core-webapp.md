---
title: Azure AD v2.0 ASP.NET Core Web 應用程式快速入門 | Microsoft Docs
description: 深入了解如何使用 OpenID Connect，在 ASP.NET Core Web 應用程式上實作 Microsoft 登入
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/25/2018
ms.author: andret
ms.custom: aaddev
ms.openlocfilehash: 4ab3d0b74e8305d67af862020197c69b15221086
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48830220"
---
# <a name="add-sign-in-with-microsoft-to-an-aspnet-core-web-app"></a>將「使用 Microsoft 登入」新增至 ASP.NET Core Web 應用程式

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

本快速入門包含程式碼範例，該範例會示範 ASP.NET Core Web 應用程式如何以個人 (hotmail.com、live.com 及其他) 與公司和學校帳戶從任何 Azure Active Directory 執行個體登入。

![此快速入門所產生範例應用程式的運作方式](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.png)


> [!div renderon="docs"]
> ## <a name="register-your-application-and-download-your-quickstart-app"></a>註冊您的應用程式，並下載您的快速入門應用程式
>
> ### <a name="register-and-configure-your-application-and-code-sample"></a>註冊並設定您的應用程式和程式碼範例
> #### <a name="step-1-register-your-application"></a>步驟 1：註冊您的應用程式
> 
> 1. 移至 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/portal/register-app)。
> 1. 輸入應用程式的名稱，確定未選取 [引導式設定] 選項，然後按一下 [建立]。
> 1. 按一下 `Add Platform`，然後選取 `Web`。
> 1. 請確定已*選取* [允許隱含流程]。
> 1. 在 [重新導向 URL] 中，輸入 `http://localhost:3110/`。
> 1. 向下捲動至頁面底部，然後按一下 [儲存]。

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-azure-portal"></a>步驟 1：在 Azure 入口網站中設定您的應用程式
> 若要讓本快速入門中的程式碼範例能正常運作，您需要新增 `http://localhost:3110/` 作為回覆 URL。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [為我進行此變更]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已設定](media/quickstart-v2-aspnet-core-webapp/green-check.png) 您的應用程式已設定了這個屬性

#### <a name="step-2-download-your-aspnet-core-project"></a>步驟 2：下載您的 ASP.NET Core 專案

- [下載 ASP.NET Core 專案](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/master.zip)

#### <a name="step-3-configure-your-project"></a>步驟 3：設定您的專案

1. 將 ZIP 檔案解壓縮至根資料夾附近的本機資料夾 - 例如 **C:\Azure-Samples**
1. 如果您使用 Visual Studio 2017，請在 Visual Studio 中開啟專案 (選用)
1. 編輯 **appsettings.json**，並以您剛剛所註冊應用程式的識別碼取代 `ClientId` 值：

    ```json
    "ClientId": "Enter_the_Application_Id_here"
    "TenantId": "common"
    ```
1. 如果您的應用程式是「單一租用戶應用程式」(目標僅限於目前目錄中的帳戶)，請在您的 **appsettings.json** 檔案中尋找 `TenantId` 的值，並且以您的**租用戶識別碼**或**租用戶名稱** (例如，contoso.microsoft.com) 取代 `common`。 您可以在 [概觀] 頁面中取得租用戶名稱。

## <a name="more-information"></a>詳細資訊

本節會概述登入使用者所需的程式碼。 這有助於了解程式碼的運作方式、主要引數，以及如何將登入新增至現有的 ASP.NET Core 應用程式。

### <a name="startup-class"></a>啟始類別

Microsoft.AspNetCore.Authentication 中介軟體會使用啟始類別，這會在初始化主機處理序時執行：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(sharedOptions =>
    {
        sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddAzureAd(options => Configuration.Bind("AzureAd", options))
    .AddCookie();

    services.AddMvc();
}
```

`AddAuthentication` 方法會將服務設定為可新增 Cookie 型驗證 (用於瀏覽器案例)，以及可對 OpenIdConnect 設定挑戰。 

包含 `.AddAzureAd` 的程式碼行會將 Azure Active Directory 驗證新增至您的應用程式。

除此之外，**AzureAdAuthenticationBuilderExtensions.cs** 檔案會將擴充方法新增至 AzureAd 驗證管線。 這個擴充方法會設定 Azure AD 驗證所需的屬性。 `IConfigureNamedOptions` 介面中的 `Configure` 方法包含下列項目：

```csharp
public void Configure(string name, OpenIdConnectOptions options)
{
    options.ClientId = _azureOptions.ClientId;
    options.Authority = $"{_azureOptions.Instance}common/v2.0";   // V2 specific
    options.UseTokenLifetime = true;
    options.RequireHttpsMetadata = false;
    options.TokenValidationParameters.ValidateIssuer = false;     // accept several tenants (here simplified)
}
```
> |Where  |  |
> |---------|---------|
> |ClientId     |來自註冊於 Azure 入口網站中應用程式的應用程式識別碼|
> |授權單位 | 使用者進行驗證的 STS 端點。 通常針對公用雲端使用 https://login.microsoftonline.com/{tenant}/v2.0，其中 {tenant} 是您租用戶的名稱、您的租用戶識別碼，或者針對參考至通用端點 (用於多租用戶應用程式) 使用 common|
> |UseTokenLifetime |指出驗證 cookie 應該與驗證權杖的 cookie 相符|
> |RequireHttpsMetadata     |中繼資料位址或授權單位需要 HTTPS。 建議將此值變更為 `True`|
> |TokenValidationParameters     | 權杖驗證參數清單。 在此案例中，`ValidateIssuer` 設為 `false`，表示它可以接受來自任何個人或公司或學校帳戶的登入|

### <a name="initiate-an-authentication-challenge"></a>起始驗證挑戰

您可以藉由在您的控制站要求驗證挑戰，就像在 **AccountController.cs** 中一樣，強制使用者登入：

```csharp
public IActionResult SignIn()
{
    var redirectUrl = Url.Action(nameof(HomeController.Index), "Home");
    return Challenge(
        new AuthenticationProperties { RedirectUri = redirectUrl },
        OpenIdConnectDefaults.AuthenticationScheme);
}
```

> [!TIP]
> 使用上述方法要求驗證挑戰是選擇性的，通常會在您想要讓已驗證和未經驗證使用者均可進行檢視時使用。 您可以使用下一節所述的方法，來保護控制站。

### <a name="protect-a-controller-or-a-controllers-method"></a>保護控制站或控制站方法

您可以使用 `[Authorize]` 屬性來保護控制站或控制站方法。 這個屬性會將控制站或方法的存取限制為只允許已驗證的使用者 - 這表示如果使用者未經過驗證，則可啟動驗證挑戰來存取控制站。

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱此 ASP.NET Core 快速入門的 GitHub 存放庫 - 包括如何將驗證新增至全新 ASP.NET Core Web 應用程式的指示：

> [!div class="nextstepaction"]
> [ASP.NET Core Web 應用程式的程式碼範例](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]