---
title: 連線至 Azure AD 時對 MVC 專案所做的變更
description: 說明您使用 Visual Studio 已連線服務連線至 Azure AD 時，MVC 專案會有何狀況
services: active-directory
author: ghogen
manager: douge
ms.assetid: 8b24adde-547e-4ffe-824a-2029ba210216
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: aaddev, vs-azure
ms.openlocfilehash: f5ce147e61566ba75532103b4f17460e8029da12
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42144242"
---
# <a name="what-happened-to-my-mvc-project-visual-studio-azure-active-directory-connected-service"></a>我的 MVC 專案 (Visual Studio Azure Active Directory 已連線服務) 發生什麼事？

> [!div class="op_single_selector"]
> - [開始使用](vs-active-directory-dotnet-getting-started.md)
> - [發生什麼情形](vs-active-directory-dotnet-what-happened.md)

本文說明在[使用 Visual Studio 新增 Azure Active Directory 連線服務](vs-active-directory-add-connected-service.md)時，對 ASP.NET MVC 專案會進行哪些明確的變更。

如需使用連線服務的相關資訊，請參閱[開始使用](vs-active-directory-dotnet-getting-started.md)。

## <a name="added-references"></a>新增的參考

會影響專案檔 *.NET 參考和 `packages.config` (NuGet 參考)。

| 類型 | 參考 |
| --- | --- |
| .NET; NuGet | Microsoft.IdentityModel.Protocol.Extensions |
| .NET; NuGet | Microsoft.Owin |
| .NET; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET; NuGet | Microsoft.Owin.Security |
| .NET; NuGet | Microsoft.Owin.Security.Cookies |
| .NET; NuGet | Microsoft.Owin.Security.OpenIdConnect |
| .NET; NuGet | Owin |
| .NET        | System.IdentityModel |
| .NET; NuGet | System.IdentityModel.Tokens.Jwt |
| .NET        | System.Runtime.Serialization |

您選取了 [讀取目錄資料] 選項時的其他參考：

| 類型 | 參考 |
| --- | --- |
| .NET; NuGet | EntityFramework |
| .NET        | EntityFramework.SqlServer (僅限 Visual Studio 2015) |
| .NET; NuGet | Microsoft.Azure.ActiveDirectory.GraphClient |
| .NET; NuGet | Microsoft.Data.Edm |
| .NET; NuGet | Microsoft.Data.OData |
| .NET; NuGet | Microsoft.Data.Services.Client |
| .NET; NuGet | Microsoft.IdentityModel.Clients.ActiveDirectory |
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms (僅限 Visual Studio 2015) |
| .NET; NuGet | System.Spatial |

下列參考已移除 (僅限 ASP.NET 4 專案，如同在 Visual Studio 2015 中)：

| 類型 | 參考 |
| --- | --- |
| .NET; NuGet | Microsoft.AspNet.Identity.Core |
| .NET; NuGet | Microsoft.AspNet.Identity.EntityFramework |
| .NET; NuGet | Microsoft.AspNet.Identity.Owin |

## <a name="project-file-changes"></a>專案檔變更

- 將屬性 `IISExpressSSLPort` 設為相異數字。
- 如果您選取了 [讀取目錄資料] 選項，請將屬性 `WebProject_DirectoryAccessLevelKey` 設為 0。
- 將 `IISUrl` 設為 `https://localhost:<port>/`，其中，`<port>` 符合 `IISExpressSSLPort` 值。

## <a name="webconfig-or-appconfig-changes"></a>web.config 或 app.config 的變更

- 已新增下列組態項目：

    ```xml
    <appSettings>
        <add key="ida:ClientId" value="<ClientId from the new Azure AD app>" />
        <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
        <add key="ida:Domain" value="<your selected Azure domain>" />
        <add key="ida:TenantId" value="<the Id of your selected Azure AD tenant>" />
        <add key="ida:PostLogoutRedirectUri" value="<project start page, such as https://localhost:44335>" />
    </appSettings>
    ```

- 已在 `System.IdentityModel.Tokens.Jwt` 和 `Microsoft.IdentityModel.Protocol.Extensions` 的 `<runtime><assemblyBinding>` 節點下新增 `<dependentAssembly>` 元素。

您選取了 [讀取目錄資料] 選項時的其他變更：

- 已在 `<appSettings>` 下新增下列組態項目：

    ```xml
    <add key="ida:ClientSecret" value="<Azure AD app's new client secret>" />
    ```

- 已在 `<configuration>` 下新增下列元素；project-mdf-file 和 project-catalog-id 會有不同的值：

    ```xml
    <configSections>
      <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
      <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    </configSections>

    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\<project-mdf-file>.mdf;Initial Catalog=<project-catalog-id>;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>

    <entityFramework>
      <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
        <parameters>
          <parameter value="mssqllocaldb" />
        </parameters>
      </defaultConnectionFactory>
      <providers>
        <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      </providers>
    </entityFramework>
    ```

- 已在 `Microsoft.Data.Services.Client`、`Microsoft.Data.Edm` 和 `Microsoft.Data.OData` 的 `<runtime><assemblyBinding>` 節點下新增 `<dependentAssembly>` 元素。

## <a name="code-changes-and-additions"></a>程式碼變更和新增

- 已將 `[Authorize]` 屬性新增至 `Controllers/HomeController.cs` 和任何其他現有的控制器。

- 已新增驗證啟動類別 `App_Start/Startup.Auth.cs` (內含 Azure AD 驗證的啟動邏輯)。 如果您選取了 [讀取目錄資料] 選項，此檔案也會包含用來接收 OAuth 驗證碼和交換存取權杖的程式碼。

- 已新增控制器類別 `Controllers/AccountController.cs` (內含 `SignIn` 和 `SignOut` 方法)。

- 已新增部分檢視 `Views/Shared/_LoginPartial.cshtml` (內含 `SignIn` 和 `SignOut` 的動作連結)。

- 已新增部分檢視 `Views/Account/SignoutCallback.cshtml` (內含登出 UI 的 HTML)。

- 已更新 `Startup.Configuration` 方法，而在類別已存在時納入對 `ConfigureAuth(app)` 的呼叫；否則會新增包含呼叫方法的 `Startup` 類別。

- 已新增 `Connected Services/AzureAD/ConnectedService.json` (Visual Studio 2017) 或 `Service References/Azure AD/ConnectedService.json` (Visual Studio 2015)，內含 Visual Studio 用來追蹤連線服務新增情形的資訊。

- 如果您選取了 [讀取目錄資料] 選項，則會新增 `Models/ADALTokenCache.cs` 和 `Models/ApplicationDbContext.cs` 以支援權杖快取。 此外也已新增其他控制器和檢視，以說明使用 Azure 圖形 API 存取使用者設定檔資訊：`Controllers/UserProfileController.cs`、`Views/UserProfile/Index.cshtml` 和 `Views/UserProfile/Relogin.cshtml`

### <a name="file-backup-visual-studio-2015"></a>檔案備份 (Visual Studio 2015)

在新增連線服務時，Visual Studio 2015 會備份已變更和移除的檔案。 所有受影響的檔案都會儲存在資料夾 `Backup/AzureAD` 中。 Visual Studio 2017 並不會建立備份。

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ManageViewModels.cs`
- `Views\Shared\_LoginPartial.cshtml`

## <a name="changes-on-azure"></a>Azure 上的變更

- 已在您於新增連線服務時選取的網域中建立 Azure AD 應用程式。
- 已更新應用程式，以在選取了 [讀取目錄資料] 選項時包含「讀取目錄資料」權限。

[深入了解 Azure Active Directory](https://azure.microsoft.com/services/active-directory/)。

## <a name="next-steps"></a>後續步驟

- [Azure Active Directory 的驗證案例](authentication-scenarios.md)
- [將「使用 Microsoft 登入」新增至 ASP.NET Web 應用程式](quickstart-v1-aspnet-webapp.md)
