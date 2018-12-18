---
title: 建置與 Azure AD 整合來進行驗證和授權的 .NET MVC Web API | Microsoft Docs
description: 如何建置與 Azure AD 整合來進行驗證和授權的 .NET MVC Web API。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 67e74774-1748-43ea-8130-55275a18320f
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jmprieur, andret
ms.custom: aaddev
ms.openlocfilehash: 239c0d0adbe89dd3d1d7bc7244a52ab079a36ad4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952540"
---
# <a name="quickstart-build-a-net-web-api-that-integrates-with-azure-ad-for-authentication-and-authorization"></a>快速入門：建置與 Azure AD 整合來進行驗證和授權的 .NET Web API

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

如果您要建置可存取受保護資源的應用程式，您必須知道如何防止這些資源遭受不當存取。 Azure Active Directory (Azure AD) 只需幾行程式碼，即可使用 OAuth 2.0 持有人權杖，以既簡單又直接的方式保護 Web API。

在 ASP.NET Web 應用程式中，您可以透過使用 Microsoft 的社群導向 OWIN 中介軟體 (包含在 .NET Framework 4.5 中) 實作，來完成這項規劃。 這裡會使用 OWIN 建置「待辦事項清單」Web API：

* 指定要保護哪些 API。
* 驗證 Web API 呼叫是否包含有效的存取權杖。

在本快速入門中，您將建置待辦事項清單 API，並了解如何：

1. 向 Azure AD 註冊應用程式。
2. 設定應用程式以使用 OWIN 驗證管線。
3. 設定用戶端應用程式以呼叫 Web API。

## <a name="prerequisites"></a>必要條件

若要開始，請完成以下必要條件：

* [下載應用程式基本架構](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/skeleton.zip)或[下載完整的範例](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/complete.zip)。 每一個都是 Visual Studio 2013 解決方案。
* 一個可供註冊應用程式的 Azure AD 租用戶。 如果您還沒有租用戶， [了解如何取得租用戶](quickstart-create-new-tenant.md)。

## <a name="step-1-register-an-application-with-azure-ad"></a>步驟 1︰向 Azure AD 註冊應用程式

若要協助保護您的應用程式，您必須先在您的租用戶中建立應用程式，然後提供 Azure AD 幾個重要的資訊。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頁面的右上角選取您的帳戶，選取 [切換目錄] 瀏覽，然後選取適當的租用戶，以選擇您的 Azure AD 租用戶。
    * 如果您的帳戶底下只有一個 Azure AD 租用戶，或如果您已選取適當的 Azure AD 租用戶，請略過此步驟。

3. 選取左側導覽窗格中的 [Azure Active Directory]。
4. 選取 [應用程式註冊]，然後選取 [新增]。
5. 依照提示並建立新的「Web 應用程式和/或 Web API」。
    * [名稱] 可向使用者描述您的應用程式。 輸入**待辦事項清單服務**。
    * 「重新導向 URI」是配置與字串的組合，Azure AD 會用它來傳回應用程式所要求的任何權杖。 請為此值輸入 `https://localhost:44321/` 。

6. 從應用程式的 [設定] > [屬性] 頁面，更新應用程式識別碼 URI。 輸入租用戶特定識別碼。 例如，輸入 `https://contoso.onmicrosoft.com/TodoListService`。
7. 儲存組態。 讓入口網站保持開啟，因為您很快就必須一併註冊用戶端應用程式。

## <a name="step-2-set-up-the-app-to-use-the-owin-authentication-pipeline"></a>步驟 2：設定應用程式以使用 OWIN 驗證管線

若要驗證連入要求和權杖，您必須設定讓應用程式與 Azure AD 進行通訊。

1. 若要開始進行，請開啟方案，然後使用「封裝管理器主控台」將 OWIN 中介軟體 NuGet 套件新增到 TodoListService 專案中。

    ```
    PM> Install-Package Microsoft.Owin.Security.ActiveDirectory -ProjectName TodoListService
    PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoListService
    ```

2. 將 OWIN 啟動類別加入至名為 `Startup.cs`的 TodoListService 專案。  在專案上按一下滑鼠右鍵，選取 [新增] > [新增項目]，然後搜尋 **OWIN**。 OWIN 中介軟體將會在應用程式啟動時叫用 `Configuration(…)` 方法。

3. 將類別宣告變更為 `public partial class Startup`。 我們已在另一個檔案中為您實作此類別的一部分。 請在 `Configuration(…)` 方法中，呼叫 `ConfgureAuth(…)` 來為您的 Web 應用程式設定驗證。

    ```csharp
    public partial class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            ConfigureAuth(app);
        }
    }
    ```

4. 開啟檔案 `App_Start\Startup.Auth.cs` 並實作 `ConfigureAuth(…)` 方法。 您在 `WindowsAzureActiveDirectoryBearerAuthenticationOptions` 中提供的參數會作為供應用程式與 Azure AD 進行通訊的座標。

    ```csharp
    public void ConfigureAuth(IAppBuilder app)
    {
        app.UseWindowsAzureActiveDirectoryBearerAuthentication(
            new WindowsAzureActiveDirectoryBearerAuthenticationOptions
            {
                Audience = ConfigurationManager.AppSettings["ida:Audience"],
                Tenant = ConfigurationManager.AppSettings["ida:Tenant"]
            });
    }
    ```

5. 使用 `[Authorize]` 屬性以「JSON Web 權杖」(JWT) 持有者驗證協助保護您的控制器和動作。 使用授權標記裝飾 `Controllers\TodoListController.cs` 類別，這會強制使用者存取該頁面前必須先登入。

    ```csharp
    [Authorize]
    public class TodoListController : ApiController
    {
    ```

    當授權的呼叫者成功叫用其中一個 `TodoListController` API 時，此動作可能需要存取呼叫者的相關資訊。 OWIN 可讓您透過 `ClaimsPrincpal` 物件存取持有人權杖內部的宣告。  

6. Web API 的一個常見需求是驗證權杖中是否有「範圍」存在，這可確保使用者已經取得存取「待辦事項清單服務」所需權限的同意。

    ```csharp
    public IEnumerable<TodoItem> Get()
    {
        // user_impersonation is the default permission exposed by applications in Azure AD
        if (ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/scope").Value != "user_impersonation")
        {
            throw new HttpResponseException(new HttpResponseMessage {
              StatusCode = HttpStatusCode.Unauthorized,
              ReasonPhrase = "The Scope claim does not contain 'user_impersonation' or scope claim not found"
            });
        }
        ...
    }
    ```

7. 開啟 TodoListService 專案根目錄中的 `web.config` 檔案，然後在 `<appSettings>` 區段中輸入您的組態值。
    * `ida:Tenant` 是您 Azure AD 租用戶的名稱 (例如 contoso.onmicrosoft.com)。
    * `ida:Audience` 是您在 Azure 入口網站中輸入之應用程式的「應用程式識別碼 URI」。

## <a name="step-3-configure-a-client-application-and-run-the-service"></a>步驟 3：設定用戶端應用程式並執行服務

在看到「待辦事項清單服務」運作之前，您必須設定「待辦事項清單」用戶端，以便讓它能夠從 Azure AD 取得權杖，並對服務進行呼叫。

1. 回到 [Azure 入口網站](https://portal.azure.com)。
1. 在 Azure AD 租用戶中建立新的應用程式，然後在產生的提示中選取 [ **原生用戶端應用程式** ]。
    * **名稱**向使用者描述您的應用程式。
    * 請輸入 `http://TodoListClient/` 作為 [重新導向 URI] 的值。

1. 完成註冊之後，Azure AD 會為應用程式指派一個唯一的應用程式識別碼。 您會在後續小節中用到這個值，所以請從應用程式頁面中複製此值。
1. 從 [設定] 頁面中，選取 [必要權限]，然後選取 [新增]。 找出並選取 [待辦事項清單服務]，在 [委派的權限] 底下新增 [存取 TodoListService] 權限，然後選取 [完成]。
1. 在 Visual Studio 中，開啟 TodoListClient 專案中的 `App.config`，然後在 `<appSettings>` 區段中輸入您的組態值。

    * `ida:Tenant` 是您 Azure AD 租用戶的名稱，例如 contoso.onmicrosoft.com。
    * `ida:ClientId` 是您從 Azure 入口網站中複製的應用程式識別碼。
    * `todo:TodoListResourceId` 是您在 Azure 入口網站中輸入之「待辦事項清單服務」應用程式的「應用程式識別碼 URI」。

1. 清除、建置及執行每個專案。
1. 如果您還沒有這麼做，請使用 *.onmicrosoft.com 網域在租用戶中建立新的使用者。
1. 使用該使用者來登入「待辦事項清單」用戶端，然後在使用者的待辦清單中新增一些工作。

## <a name="next-steps"></a>後續步驟

* 如需參考資料，請從 [GitHub](https://github.com/AzureADQuickStarts/WebAPI-Bearer-DotNet/archive/complete.zip) 下載完整的範例 (不含您的設定值)。 您現在可以繼續前進到其他身分識別案例。