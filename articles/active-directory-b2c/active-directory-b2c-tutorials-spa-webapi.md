---
title: 教學課程 - 使用 Azure Active Directory B2C 授與從單頁應用程式存取 ASP.NET Core Web API 的權限 | Microsoft Docs
description: 關於如何使用 Active Directory B2C 來保護 .NET Core Web API，以及如何從單頁應用程式呼叫它的教學課程。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.author: davidmu
ms.date: 3/02/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.component: B2C
ms.openlocfilehash: efe975fa4f89a262faef82df3cc79820d393b60e
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45605754"
---
# <a name="tutorial-grant-access-to-an-aspnet-core-web-api-from-a-single-page-app-using-azure-active-directory-b2c"></a>教學課程：使用 Azure Active Directory B2C 授與從單頁應用程式存取 ASP.NET Core Web API 的權限

此教學課程將說明如何從單頁應用程式呼叫 Azure Active Directory (Azure AD) B2C 所保護的 ASP.NET Core Web API 資源。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 在您的 Azure AD B2C 租用戶中註冊 Web API
> * 定義及設定 Web API 的範圍
> * 為應用程式授與 Web API 的權限
> * 更新範例程式碼以使用 Azure AD B2C 來保護 Web API

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先決條件

* 完成[在單頁應用程式中使用 Azure Active Directory B2C 進行使用者驗證](active-directory-b2c-tutorials-spa.md)教學課程。
* 安裝包含 **ASP.NET 和 Web 開發**工作負載的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。
* [.NET Core 2.0.0 SDK](https://www.microsoft.com/net/core) 或更新版本
* 安裝 [Node.js](https://nodejs.org/en/download/)

## <a name="register-web-api"></a>註冊 Web API

Web API 資源必須先在您的租用戶中註冊，才能接受及回應受到[用戶端應用程式](../active-directory/develop/developer-glossary.md#client-application)保護而提供來自 Azure Active Directory 之[存取權杖](../active-directory/develop/developer-glossary.md#access-token)的[資源要求](../active-directory/develop/developer-glossary.md#resource-server)。 註冊會在您的租用戶中建立[應用程式與服務主體物件](../active-directory/develop/developer-glossary.md#application-object)。 

以 Azure AD B2C 租用戶的全域管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。

[!INCLUDE [active-directory-b2c-switch-b2c-tenant](../../includes/active-directory-b2c-switch-b2c-tenant.md)]

1. 選擇 Azure 入口網站左上角的 [所有服務]，搜尋並選取 [Azure AD B2C]。 您現在應使用在前一個教學課程中建立的租用戶。

2. 選取 [應用程式]，然後選取 [新增]。

    若要在您的租用戶中註冊範例 Web API，請使用下列設定。
    
    ![新增 API](media/active-directory-b2c-tutorials-spa-webapi/web-api-registration.png)
    
    | 設定      | 建議的值  | 說明                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **名稱** | Hello Core API | 輸入向開發人員描述您的 Web API 的 [名稱]。 |
    | **包含 Web 應用程式 / Web API** | 是 | 針對 Web API 選取 [是]。 |
    | **允許隱含流程** | 是 | 請選取 [是]，因為 API 使用 [OpenID Connect 登入](active-directory-b2c-reference-oidc.md)。 |
    | **回覆 URL** | `http://localhost:5000` | 回覆 URL 是 Azure AD B2C 傳回您 API 要求之任何權杖的所在端點。 在此教學課程中，範例 Web API 會在本機 (localhost) 執行，並在連接埠 5000 上進行接聽 (在此教學課程中稍後這麼設定之後)。 |
    | **應用程式識別碼 URI** | HelloCoreAPI | URI 可唯一識別租用戶中的 API。 這可讓您為每個租用戶註冊多個 API。 [範圍](../active-directory/develop/developer-glossary.md#scopes)可控管對受保護 API 資源的存取，並就個別的應用程式識別碼 URI 進行定義。 |
    | **原生用戶端** | 否 | 這是 Web API，而不是原生用戶端，因此請選取 [否]。 |
    
3. 按一下 [建立]  以註冊您的 API。

已註冊的 API 會顯示在 Azure AD B2C 租用戶的應用程式清單中。 從清單中選取 Web API。 Web API 的 [屬性] 窗格隨即顯示。

![Web API 屬性](./media/active-directory-b2c-tutorials-spa-webapi/b2c-web-api-properties.png)

請記下 [應用程式用戶端識別碼]。 此識別碼可唯一識別 API，後續在此教學課程中設定 API 時將會用到。

將您的 Web API 註冊至 Azure AD B2C，可定義信任關係。 由於 API 已註冊至 B2C，因此 API 此時得以信任其接收自其他應用程式的 B2C 存取權杖。

## <a name="define-and-configure-scopes"></a>定義及設定範圍

[範圍](../active-directory/develop/developer-glossary.md#scopes)可用來控管對受保護資源的存取。 Web API 可使用範圍來實作以範圍為基礎的存取控制。 例如，有些使用者可能同時具有讀取和寫入權限，而有些則可能只有唯讀權限。 在此教學課程中，您會定義 Web API 的讀取權限。

### <a name="define-scopes-for-the-web-api"></a>定義 Web API 的範圍

已註冊的 API 會顯示在 Azure AD B2C 租用戶的應用程式清單中。 從清單中選取 Web API。 Web API 的 [屬性] 窗格隨即顯示。

按一下 [發佈的範圍 (預覽)]。

若要設定 API 的範圍，請新增下列項目。 

![Web API 中定義的範圍](media/active-directory-b2c-tutorials-spa-webapi/scopes-web-api.png)

| 設定      | 建議的值  | 說明                                        |
| ------------ | ------- | -------------------------------------------------- |
| **範圍** | demo.read | 示範 API 的讀取權限 |

按一下 [檔案] 。

發佈的範圍可以用來為用戶端應用程式授與對 Web API 的權限。

### <a name="grant-app-permissions-to-web-api"></a>為應用程式授與 Web API 的權限

若要從應用程式呼叫受保護的 Web API，您必須為應用程式授與對 API 的權限。 在此教學課程中，使用[在單頁應用程式 (JavaScript) 中使用 Azure Active Directory B2C 進行使用者驗證](active-directory-b2c-tutorials-spa.md)教學課程中建立的單頁應用程式。

1. 在 Azure 入口網站中，從服務清單中選取 [Azure AD B2C]，然後按一下 [應用程式] 以檢視已註冊的應用程式清單。

2. 從應用程式清單中選取 [我的範例單頁應用程式]、按一下 [API 存取 (預覽)]，然後按一下 [新增]。

3. 在 [選取 API] 下拉式清單中，選取您已註冊的 Web API **Hello Core API**。

4. 在 [選取範圍] 下拉式清單中，選取您在 Web API 註冊中定義的範圍。

    ![選取應用程式的範圍](media/active-directory-b2c-tutorials-spa-webapi/selecting-scopes-for-app.png)

5. 按一下 [確定]。

您的 [我的範例單頁應用程式] 會進行註冊以呼叫受保護的 [Hello Core API]。 使用者會向 Azure AD B2C 進行[驗證](../active-directory/develop/developer-glossary.md#authentication)以使用該單頁應用程式。 該單頁應用程式會從 Azure AD B2C 取得[授權授與](../active-directory/develop/developer-glossary.md#authorization-grant)，以存取受保護的 Web API。

## <a name="update-code"></a>更新程式碼

現在，Web API 已註冊，且您已定義範圍，接下來您必須設定 Web API 程式碼以使用您的 Azure AD B2C 租用戶。 在此教學課程中，您會設定可從 GitHub 下載的範例 .NET Core Web 應用程式。 

[下載 zip 檔案](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapi/archive/master.zip)，或從 GitHub 複製範例 Web 應用程式。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapi.git
```

### <a name="configure-the-web-api"></a>設定 Web API

1. 在 Visual Studio 中開啟 **B2C-WebAPI.sln** 解決方案。

2. 開啟 **appsettings.json** 檔案。 更新下列值，以將 Web API 設定為使用您的租用戶：

    ```javascript
    "AzureAdB2C": 
      {
        "Tenant": "<your tenant name>.onmicrosoft.com", 
        "ClientId": "<The Application ID for your web API obtained from the Azure portal>",
        "Policy": "<Your sign up sign in policy e.g. B2C_1_SiUpIn>",
        "ScopeRead": "demo.read"  
      },
    ```

#### <a name="enable-cors"></a>啟用 CORS

若要允許您的單頁應用程式呼叫 ASP.NET Core Web API，您必須啟用 [CORS](https://docs.microsoft.com/aspnet/core/security/cors)。

1. 在 **Startup.cs** 中，將 CORS 新增至 `ConfigureServices()` 方法。

    ```C#
    public void ConfigureServices(IServiceCollection services) {
      services.AddCors();
    ```

2. 在 **Startup.cs** 中，設定 `Configure()` 方法中的 CORS。

    ```C#
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory) {
      app.UseCors(builder =>
        builder.WithOrigins("http://localhost:6420").AllowAnyHeader().AllowAnyMethod());
    ```

3. 開啟 [屬性] 下的 [launchSettings.json] 檔案，找出 **iisSettings** *applicationURL* 設定，然後將連接埠號碼設定為已為 API 回覆 URL `http://localhost:5000` 註冊的連接埠號碼。

### <a name="configure-the-single-page-app"></a>設定單頁應用程式

單頁應用程式會使用 Azure AD B2C 進行使用者註冊、登入及呼叫受保護的 ASP.NET Core Web API。 您需要更新單頁應用程式以呼叫 .NET Core Web API。
若要變更應用程式設定：

1. 開啟 Node.js 單頁應用程式範例中的 `index.html` 檔案。
2. 使用 Azure AD B2C 租用戶註冊資訊設定此範例。 在下列程式碼中，將您的租用戶名稱新增至 **b2cScopes**，並將 **webApi** 值變更為您之前記錄的 *applicationURL* 值：

    ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    var applicationConfig = {
        clientID: '<Application ID for your SPA obtained from portal app registration>',
        authority: "https://<your-tenant-name>.b2clogin.com/tfp/<your-tenant-name>.onmicrosoft.com/B2C_1_SiUpIn",
        b2cScopes: ["https://<Your tenant name>.onmicrosoft.com/HelloCoreAPI/demo.read"],
        webApi: 'http://localhost:5000/api/values',
    };
    ```

## <a name="run-the-spa-app-and-web-api"></a>執行 SPA 應用程式和 Web API

您需要執行 Node.js 單頁應用程式和 .NET Core Web API。

### <a name="run-the-aspnet-core-web-api"></a>執行 ASP.NET Core Web API 

按 **F5** 以在 Visual Studio 中進行 **B2C-WebAPI.sln** 解決方案偵錯。

當專案啟動時，您的預設瀏覽器中會顯示一個網頁，宣告此 Web API 適用於要求。

### <a name="run-the-single-page-app"></a>執行單頁應用程式

1. 啟動 Node.js 命令提示字元。
2. 切換至包含 Node.js 範例的目錄。 例如 `cd c:\active-directory-b2c-javascript-msal-singlepageapp`
3. 執行下列命令：
    ```
    npm install && npm update
    node server.js
    ```

    主控台視窗會顯示裝載應用程式的連接埠號碼。
    
    ```
    Listening on port 6420...
    ```

4. 使用瀏覽器瀏覽至位址 `http://localhost:6420` 以檢視應用程式。
5. 使用[在單頁應用程式 (JavaScript) 中使用 Azure Active Directory B2C 進行使用者驗證](active-directory-b2c-tutorials-spa.md)中使用的電子郵件地址和密碼進行登入。
6. 按一下 [呼叫 API] 按鈕。

在您註冊使用者帳戶或使用使用者帳戶登入之後，此範例會呼叫受保護的 Web API 並傳回結果。

## <a name="clean-up-resources"></a>清除資源

如果您想要嘗試其他 Azure AD B2C 教學課程，您可以使用 Azure AD B2C 租用戶。 不再需要時，您可以[刪除您的 Azure AD B2C 租用戶](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant)。

## <a name="next-steps"></a>後續步驟

此文章逐步說明如何在 Azure AD B2C 中註冊和定義範圍，以保護 Web API。 瀏覽可用的 Azure AD B2C 程式碼範例，深入了解相關資訊。

> [!div class="nextstepaction"]
> [Azure AD B2C 程式碼範例](https://azure.microsoft.com/resources/samples/?service=active-directory-b2c&sort=0)
