---
title: 連線至 Azure AD 時對 WebAPI 專案所做的變更
description: 說明當您使用 Visual Studio 連線至 Azure AD 時，您的 WebAPI 專案會發生什麼狀況
services: active-directory
author: ghogen
manager: douge
ms.assetid: 57630aee-26a2-4326-9dbb-ea2a66daa8b0
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: aaddev
ms.openlocfilehash: 60b51f28f5836fcc0f1f2da2d17971ea4f24eaca
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39495184"
---
# <a name="what-happened-to-my-webapi-project-visual-studio-azure-active-directory-connected-service"></a>我的 WebAPI 專案 (Visual Studio Azure Active Directory 連線服務) 發生什麼狀況

> [!div class="op_single_selector"]
> - [開始使用](vs-active-directory-webapi-getting-started.md)
> - [發生什麼情形](vs-active-directory-webapi-what-happened.md)

本文說明在[使用 Visual Studio 新增 Azure Active Directory 連線服務](vs-active-directory-add-connected-service.md)時，對 ASP.NET WebAPI、ASP.NET 單頁應用程式和 ASP.NET Azure API 專案會進行哪些明確的變更。 本文也適用於 Visual Studio 2015 中的 ASP.NET Azure 行動服務專案。

如需使用連線服務的相關資訊，請參閱[開始使用](vs-active-directory-webapi-getting-started.md)。

## <a name="added-references"></a>新增的參考

會影響專案檔 *.NET 參考和 `packages.config` (NuGet 參考)。

| 類型 | 參考 |
| --- | --- |
| .NET; NuGet | Microsoft.Owin |
| .NET; NuGet | Microsoft.Owin.Host.SystemWeb |
| .NET; NuGet | Microsoft.Owin.Security |
| .NET; NuGet | Microsoft.Owin.Security.ActiveDirectory |
| .NET; NuGet | Microsoft.Owin.Security.Jwt |
| .NET; NuGet | Microsoft.Owin.Security.OAuth |
| .NET; NuGet | Owin |
| .NET; NuGet | System.IdentityModel.Tokens.Jwt |

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
| .NET        | Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms<br>(僅限 Visual Studio 2015) |
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
        <add key="ida:Tenant" value="<your selected Azure domain>" />
        <add key="ida:Audience" value="<your selected domain + / + project name>" />
    </appSettings>
    ```

- 僅限 Visual Studio 2017：也在 `<appSettings>`" 下新增了下列項目

    ```xml
    <add key="ida:MetadataAddress" value="<domain URL + /federationmetadata/2007-06/federationmetadata.xml>" />
    ```

- 已在 `System.IdentityModel.Tokens.Jwt` 的 `<runtime><assemblyBinding>` 節點下新增 `<dependentAssembly>` 元素。

- 如果您選取了 [讀取目錄資料] 選項，則會在 `<appSettings>` 下新增下列組態項目：

    ```xml
    <add key="ida:Password" value="<Your Azure AD app's new password>" />
    ```

## <a name="code-changes-and-additions"></a>程式碼變更和新增

- 已將 `[Authorize]` 屬性新增至 `Controllers/ValueController.cs` 和任何其他現有的控制器。

- 已新增驗證啟動類別 `App_Start/Startup.Auth.cs` (內含 Azure AD 驗證的啟動邏輯)，或對其進行相應的修改。 如果您選取了 [讀取目錄資料] 選項，此檔案也會包含用來接收 OAuth 驗證碼和交換存取權杖的程式碼。

- (僅限具有 ASP.NET 4 應用程式的 Visual Studio 2015) 已移除 `App_Start/IdentityConfig.cs`，並新增 `Controllers/AccountController.cs`、`Models/IdentityModel.cs` 和 `Providers/ApplicationAuthProvider.cs`。

- 已新增 `Connected Services/AzureAD/ConnectedService.json` (Visual Studio 2017) 或 `Service References/Azure AD/ConnectedService.json` (Visual Studio 2015)，內含 Visual Studio 用來追蹤連線服務新增情形的資訊。

### <a name="file-backup-visual-studio-2015"></a>檔案備份 (Visual Studio 2015)

在新增連線服務時，Visual Studio 2015 會備份已變更和移除的檔案。 所有受影響的檔案都會儲存在資料夾 `Backup/AzureAD` 中。 Visual Studio 2017 並不會建立備份。

- `Startup.cs`
- `App_Start\IdentityConfig.cs`
- `App_Start\Startup.Auth.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Models\ApplicationOAuthProvider.cs`

## <a name="changes-on-azure"></a>Azure 上的變更

- 已在您於新增連線服務時選取的網域中建立 Azure AD 應用程式。
- 已更新應用程式，以在選取了 [讀取目錄資料] 選項時包含「讀取目錄資料」權限。

[深入了解 Azure Active Directory](https://azure.microsoft.com/services/active-directory/)。

## <a name="next-steps"></a>後續步驟

- [Azure Active Directory 的驗證案例](authentication-scenarios.md)
- [將「使用 Microsoft 登入」新增至 ASP.NET Web 應用程式](quickstart-v1-aspnet-webapp.md)