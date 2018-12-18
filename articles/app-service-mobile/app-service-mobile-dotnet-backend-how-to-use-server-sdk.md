---
title: 如何使用適用於 Mobile Apps 的 .NET 後端伺服器 SDK | Microsoft Docs
description: 了解如何使用適用於 Azure App Service Mobile Apps 的 .NET 後端伺服器 SDK。
keywords: App Service, Azure App Service, 行動應用程式, 行動服務, 調整, 可調整, 應用程式部署, Azure 應用程式部署
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 0620554f-9590-40a8-9f47-61c48c21076b
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 8216dafdd846f10ca1c8fc33b710a093aca20c7b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38473708"
---
# <a name="work-with-the-net-backend-server-sdk-for-azure-mobile-apps"></a>使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK
[!INCLUDE [app-service-mobile-selector-server-sdk](../../includes/app-service-mobile-selector-server-sdk.md)]

本主題說明如何在主要的 Azure App Service Mobile Apps 案例中使用 .NET 後端伺服器 SDK Azure Mobile Apps SDK 可協助您從 ASP.NET 應用程式使用行動用戶端。

> [!TIP]
> [適用於 Azure Mobile Apps 的 .NET 伺服器 SDK][2]是 GitHub 上的開放原始碼。 儲存機制包含所有原始程式碼，其中包括整個伺服器 SDK 單元測試組件和一些範例專案。
>
>

## <a name="reference-documentation"></a>參考文件
伺服器 SDK 的參考文件位於此處：[Azure Mobile Apps .NET 參考資料][1]。

## <a name="create-app"></a>作法：建立 .NET 行動應用程式後端
如果您開始新的專案，您可以使用 [Azure 入口網站] 或 Visual Studio，建立 App Service 應用程式。 您可以在本機執行 App Service 應用程式，或將專案發佈至雲端架構 App Service 行動應用程式。

如果您將行動功能新增至現有的專案，請參閱 [下載並初始化 SDK](#install-sdk) 一節。

### <a name="create-a-net-backend-using-the-azure-portal"></a>使用 Azure 入口網站建立 .NET 後端
若要建立 App Service 行動後端，請遵循[快速入門教學課程][3]或遵循下列步驟︰

[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service-classic](../../includes/app-service-mobile-dotnet-backend-create-new-service-classic.md)]

回到 [開始使用] 刀鋒視窗，在 [建立資料表 API] 底下，選擇 [C#] 作為您的 [後端語言]。 按一下 [下載] ，將壓縮的專案檔案解壓縮至您的本機電腦，並在 Visual Studio 中開啟方案。

### <a name="create-a-net-backend-using-visual-studio-2017"></a>使用 Visual Studio 2017 建立 .NET 後端

透過 Visual Studio 安裝程式安裝 Azure 工作負載，以從 Visual Studio 發佈至 Azure 行動應用程式專案。 當您安裝 SDK 之後，使用下列步驟建立 ASP.NET 應用程式：

1. 開啟 [新增專案] 對話方塊 (從 [檔案]  >  [新增]  >  [專案...])。
2. 展開 [Visual C#]，然後選取 [Web]。
3. 選取 [ASP.NET Web 應用程式 (.NET Framework)]。
4. 填入專案名稱。 然後按一下 [確定] 。
5. 從範本清單選取 [Azure 行動應用程式]。
6. 按一下 [確定] 以建立方案。
7. 以滑鼠右鍵按一下 [方案總管] 中的專案，選擇 [發佈]，然後選擇 [應用程式服務] 作為發佈目標。
8. 遵循提示進行驗證，並選擇新的或現有 Azure 應用程式服務來發佈。

### <a name="create-a-net-backend-using-visual-studio-2015"></a>使用 Visual Studio 2015 建立 .NET 後端

安裝 [Azure SDK for .NET][4] (2.9.0 版或更新版本)，以在 Visual Studio 中建立 Azure Mobile Apps 專案。 當您安裝 SDK 之後，使用下列步驟建立 ASP.NET 應用程式：

1. 開啟 [新增專案] 對話方塊 (從 [檔案]  >  [新增]  >  [專案...])。
2. 展開 [範本] > [Visual C#]，然後選取 [Web]。
3. 選取 [ASP.NET Web 應用程式] 。
4. 填入專案名稱。 然後按一下 [確定] 。
5. 在 [ASP.NET 4.5.2 範本] 底下，選取 [Azure 行動應用程式]。 核取 [雲端中的主機]  以在雲端 (您可以在其中發佈這個專案) 建立行動後端。
6. 按一下 [確定]。

## <a name="install-sdk"></a>如何：下載並初始化 SDK
SDK 可於 [NuGet.org]取得。此封裝包含開始使用 SDK 所需的基本功能。 若要初始化 SDK，您需要在 **HttpConfiguration** 物件上執行動作。

### <a name="install-the-sdk"></a>安裝 SDK
若要安裝 SDK，請以滑鼠右鍵按一下 Visual Studio 中的伺服器專案，選取 [管理 NuGet 套件]，搜尋 [Microsoft.Azure.Mobile.Server] 套件，然後按一下 [安裝]。

### <a name="server-project-setup"></a> 初始化伺服器專案
初始化 .NET 後端伺服器專案的方式類似其他 ASP.NET 專案，可藉由包含 OWIN 啟動類別來完成。 請確定您已參考 NuGet 封裝 `Microsoft.Owin.Host.SystemWeb`。 若要在 Visual Studio 中新增這個類別，請在伺服器專案上按一下滑鼠右鍵，選取 **[新增]** >
 **[新項目]**，然後依序選取 **[Web]** > **[一般]** > **[OWIN 啟動類別]**。  隨即產生具有下列屬性的類別：

    [assembly: OwinStartup(typeof(YourServiceName.YourStartupClassName))]

在 OWIN 啟動類別的 `Configuration()` 方法中，使用 **HttpConfiguration** 物件來設定 Azure Mobile Apps 環境。
下列範例會初始化伺服器專案，不新增任何功能：

    // in OWIN startup class
    public void Configuration(IAppBuilder app)
    {
        HttpConfiguration config = new HttpConfiguration();

        new MobileAppConfiguration()
            // no added features
            .ApplyTo(config);

        app.UseWebApi(config);
    }

若要啟用個別功能，您必須在呼叫 **ApplyTo** 之前，於 **MobileAppConfiguration** 物件上呼叫擴充方法。 例如，下列程式碼會在初始化期間，將預設路由加入具有屬性 `[MobileAppController]` 的所有 API 控制器中：

    new MobileAppConfiguration()
        .MapApiControllers()
        .ApplyTo(config);

Azure 入口網站的伺服器快速入門會呼叫 **UseDefaultConfiguration()**。 此命令相當於下列設定：

        new MobileAppConfiguration()
            .AddMobileAppHomeController()             // from the Home package
            .MapApiControllers()
            .AddTables(                               // from the Tables package
                new MobileAppTableConfiguration()
                    .MapTableControllers()
                    .AddEntityFramework()             // from the Entity package
                )
            .AddPushNotifications()                   // from the Notifications package
            .MapLegacyCrossDomainController()         // from the CrossDomain package
            .ApplyTo(config);

使用的擴充方法如下︰

* `AddMobileAppHomeController()` 提供預設 Azure Mobile Apps 首頁。
* `MapApiControllers()` 會為以 `[MobileAppController]` 屬性裝飾的 WebAPI 控制器提供自訂 API 功能。
* `AddTables()` 提供 `/tables` 端點與資料表控制器的對應。
* `AddTablesWithEntityFramework()` 是使用以 Entity Framework 為基礎的控制器對應 `/tables` 端點的速記法。
* `AddPushNotifications()` 提供向通知中樞註冊裝置的簡單方法。
* `MapLegacyCrossDomainController()` 提供用於本機開發的標準 CORS 標頭。

### <a name="sdk-extensions"></a>SDK 延伸模組
下列 NuGet 型擴充套件提供了許多您應用程式可以使用的行動功能。 您可以使用 **MobileAppConfiguration** 物件，在初始化期間啟用擴充功能。

* [Microsoft.Azure.Mobile.Server.Quickstart] 支援基本的 Mobile Apps 設定。 在初始化期間，透過呼叫 **UseDefaultConfiguration** 擴充方法來新增到組態。 此擴充包含下列擴充功能：通知、驗證、實體、資料表、跨網域和首頁封裝。 此封裝由 Azure 入口網站上可取得的 Mobile Apps 快速入門使用。
* [Microsoft.Azure.Mobile.Server.Home](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Home/) 實作網站根目錄的預設 [此行動應用程式已啟動並執行中] 頁面。 透過呼叫 **AddMobileAppHomeController** 擴充方法來新增到組態。
* [Microsoft.Azure.Mobile.Server.Tables](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Tables/) 包含適用於處理資料與設定資料管線的類別。 透過呼叫 **AddTables** 擴充方法來加入設定中。
* [Microsoft.Azure.Mobile.Server.Entity](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Entity/) 讓 Entity Framework 能存取 SQL Database 中的資料。 透過呼叫 **AddTablesWithEntityFramework** 擴充方法來加入設定中。
* [Microsoft.Azure.Mobile.Server.Authentication] 啟用驗證，並設定用來驗證權杖的 OWIN 中介軟體。 透過呼叫 **AddAppServiceAuthentication** 與 **IAppBuilder**.**UseAppServiceAuthentication** 擴充方法來新增到組態。
* [Microsoft.Azure.Mobile.Server.Notifications] 啟用推播通知，並定義推播註冊端點。 透過呼叫 **AddPushNotifications** 擴充方法來加入設定中。
* [Microsoft.Azure.Mobile.Server.CrossDomain](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.CrossDomain/) 建立從行動應用程式提供資料給舊版網頁瀏覽器的控制器。 透過呼叫 **MapLegacyCrossDomainController** 擴充方法來加入設定中。
* [Microsoft.Azure.Mobile.Server.Login] 會提供 AppServiceLoginHandler.CreateToken() 方法，這是在自訂驗證案例期間使用的靜態方法。

## <a name="publish-server-project"></a>做法：發佈伺服器專案
本節說明如何從 Visual Studio 發佈 .NET 後端專案。 您也可以使用 [Git](../app-service/app-service-deploy-local-git.md) 或那裡提供的任何其他方法來部署後端專案。

1. 在 Visual Studio 中，重新建置專案以還原 NuGet 封裝。
2. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，然後按一下 [發佈] 。 第一次發佈時，您必須定義發行設定檔。 在定義設定檔後，您可以選取該設定檔，然後按一下 [發佈]。
3. 如果系統要求您選取發佈目標，請按一下 [Microsoft Azure App Service] > [下一步]，然後視需要使用您的 Azure 認證來登入。
   Visual Studio 會直接從 Azure 下載並安全地儲存您的發佈設定。

    ![](./media/app-service-mobile-dotnet-backend-how-to-use-server-sdk/publish-wizard-1.png)
4. 選擇您的 [訂用帳戶]，從 [檢視] 中選取 [資源類型]，展開 [行動應用程式]，按一下您的行動應用程式後端，然後按一下 [確定]。

    ![](./media/app-service-mobile-dotnet-backend-how-to-use-server-sdk/publish-wizard-2.png)
5. 驗證發佈設定檔資訊，然後按一下 [發佈] 。

    ![](./media/app-service-mobile-dotnet-backend-how-to-use-server-sdk/publish-wizard-3.png)

    當您的行動應用程式後端已成功發佈後，您會看到表示成功的登陸頁面。

    ![](./media/app-service-mobile-dotnet-backend-how-to-use-server-sdk/publish-success.png)

## <a name="define-table-controller"></a> 做法：定義資料表控制器
定義資料表控制器，以對行動用戶端公開 SQL 資料表。  設定資料表控制器需要三個步驟︰

1. 建立資料傳輸物件 (DTO) 類別。
2. 在行動 DbContext 類別中設定資料表參考。
3. 建立資料表控制器。

資料傳輸物件 (DTO) 是繼承自 `EntityData`純 C# 物件。  例如︰

    public class TodoItem : EntityData
    {
        public string Text { get; set; }
        public bool Complete {get; set;}
    }

DTO 用來定義 SQL Database 內的資料表。  若要建立資料庫項目，將 `DbSet<>` 屬性新增至您正在使用的 DbContext。  在 Azure Mobile Apps 的預設專案範本中，DbContext 稱為 `Models\MobileServiceContext.cs`：

    public class MobileServiceContext : DbContext
    {
        private const string connectionStringName = "Name=MS_TableConnectionString";

        public MobileServiceContext() : base(connectionStringName)
        {

        }

        public DbSet<TodoItem> TodoItems { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Conventions.Add(
                new AttributeToColumnAnnotationConvention<TableColumnAttribute, string>(
                    "ServiceColumnTable", (property, attributes) => attributes.Single().ColumnType.ToString()));
        }
    }

如果已安裝 Azure SDK，您現在可以建立範本資料表控制器，如下所示︰

1. 以滑鼠右鍵按一下 [控制器] 資料夾，然後選取 [新增]  > 。
2. 選取 [Azure Mobile Apps 資料表控制器] 選項，然後按一下 [新增]。
3. 在 [新增控制器]  對話方塊中：
   * 在 [模型類別]  下拉式清單中，選取您新的 DTO。
   * 在 [DbContext]  下拉式清單中，選取 [行動服務 DbContext] 類別。
   * 為您建立控制器名稱。
4. 按一下 [新增] 。

快速入門伺服器專案包含簡單的 **TodoItemController**範例。

### <a name="adjust-pagesize"></a>作法：調整資料表分頁大小
根據預設，Azure Mobile Apps 針對每個要求會傳回 50 個記錄。  分頁可確保用戶端不會佔用其 UI 執行緒或伺服器太久，以確保有良好的使用者體驗。 若要變更資料表分頁大小，請增加伺服器端「允許的查詢大小」和用戶端頁面大小。使用 `EnableQuery` 屬性可調整伺服器端「允許的查詢大小」︰

    [EnableQuery(PageSize = 500)]

確定 PageSize 等於或大於用戶端所要求的大小。  如需變更用戶端頁面大小的詳細資料，請參閱特定用戶端的做法文件。

## <a name="how-to-define-a-custom-api-controller"></a>做法：定義自訂 API 控制器
自訂 API 控制器透過公開端點，提供最基本的功能給您的行動應用程式後端。 您可以使用屬性 [MobileAppController] 來註冊行動裝置特定 API 控制器。 `MobileAppController` 屬性會註冊路由、設定 Mobile Apps JSON 序列化程式，以及開啟 [用戶端版本檢查](app-service-mobile-client-and-server-versioning.md)。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 [控制器] 資料夾，然後按一下 [新增] > [控制器]，選取 [Web API 2 控制器&mdash;空白]，然後按一下 [新增]。
2. 提供 [控制器名稱] \(例如 `CustomController`)，然後按一下 [新增]。
3. 在新的控制器類別檔案中，新增下列 Using 陳述式：

        using Microsoft.Azure.Mobile.Server.Config;
4. 將 **[MobileAppController]** 屬性套用到 API 控制器類別定義，如下列範例所示：

        [MobileAppController]
        public class CustomController : ApiController
        {
              //...
        }
5. 在 App_Start/Startup.MobileApp.cs 檔案中，新增對 **MapApiControllers** 擴充方法的呼叫，如下列範例所示：

        new MobileAppConfiguration()
            .MapApiControllers()
            .ApplyTo(config);

您也可以使用 `UseDefaultConfiguration()` 擴充方法，而不是 `MapApiControllers()`。 用戶端仍然可以存取任何沒有套用 **MobileAppControllerAttribute** 的控制器，但是使用任何行動應用程式用戶端 SDK 的用戶端可能會無法正確取用。

## <a name="how-to-work-with-authentication"></a>做法：使用驗證
Azure Mobile Apps 會使用 App Service 驗證 / 授權來保護您的行動後端。  本節說明如何在 .NET 後端伺服器專案中執行下列驗證相關工作：

* [做法：將驗證新增至伺服器專案](#add-auth)
* [做法：針對應用程式使用自訂驗證](#custom-auth)
* [做法：取出已驗證的使用者資訊](#user-info)
* [做法︰限制授權使用者的資料存取](#authorize)

### <a name="add-auth"></a>做法：將驗證新增至伺服器專案
您可以透過擴充 **MobileAppConfiguration** 物件並設定 OWIN 中介軟體，將驗證加入您的伺服器專案中。 安裝 [Microsoft.Azure.Mobile.Server.Quickstart] 封裝並呼叫 **UseDefaultConfiguration** 擴充方法時，您可以跳到步驟 3。

1. 在 Visual Studio 中，安裝 [Microsoft.Azure.Mobile.Server.Authentication] 封裝。
2. 在 Startup.cs 專案檔案中，在 **Configuration** 方法的開頭加入下列程式碼行：

        app.UseAppServiceAuthentication(config);

    此 OWIN 中介軟體元件會驗證相關聯 App Service 閘道所發出的權杖。
3. 將 `[Authorize]` 屬性加入任何需要驗證的控制器或方法中。

若要了解如何在您的 Mobile Apps 後端驗證用戶端，請參閱 [將驗證新增至您的應用程式](app-service-mobile-ios-get-started-users.md)。

### <a name="custom-auth"></a>做法：針對應用程式使用自訂驗證
> [!IMPORTANT]
> 若要啟用自訂驗證，您必須先啟用 App Service 驗證，但不需在 Azure 入口網站中選取 App Service 的提供者。 這會在裝載時啟用 WEBSITE_AUTH_SIGNING_KEY 環境變數。
> 
> 
如果您不想要使用其中一個 App Service 驗證/授權提供者，您可以實作自己的登入系統。 安裝 [Microsoft.Azure.Mobile.Server.Login] 封裝，協助產生驗證權杖。  提供您自己的程式碼來驗證使用者認證。 例如，您可以針對資料庫中的 salted 和雜湊密碼進行檢查。 在下列範例中， `isValidAssertion()` 方法 (定義於其他地方) 會負責這些檢查。

自訂驗證的公開方式為建立 ApiController 及公開 `register` 和 `login` 動作。 用戶端應該使用的自訂 UI，向使用者收集資訊。  此資訊會接著透過標準 HTTP POST 呼叫提交至 API。 伺服器驗證這項判斷提示之後，便使用 `AppServiceLoginHandler.CreateToken()` 方法來發行權杖。  ApiController **不得**使用 `[MobileAppController]`屬性。

範例 `login` 動作︰

        public IHttpActionResult Post([FromBody] JObject assertion)
        {
            if (isValidAssertion(assertion)) // user-defined function, checks against a database
            {
                JwtSecurityToken token = AppServiceLoginHandler.CreateToken(new Claim[] { new Claim(JwtRegisteredClaimNames.Sub, assertion["username"]) },
                    mySigningKey,
                    myAppURL,
                    myAppURL,
                    TimeSpan.FromHours(24) );
                return Ok(new LoginResult()
                {
                    AuthenticationToken = token.RawData,
                    User = new LoginResultUser() { UserId = userName.ToString() }
                });
            }
            else // user assertion was not valid
            {
                return this.Request.CreateUnauthorizedResponse();
            }
        }

在上述範例中，LoginResult 和 LoginResultUser 是公開必要屬性的可序列化物件。 用戶端預期有以下格式的 JSON 物件傳回登入回應：

        {
            "authenticationToken": "<token>",
            "user": {
                "userId": "<userId>"
            }
        }

`AppServiceLoginHandler.CreateToken()` 方法包含 audience 和 issuer 參數。 這兩個參數會使用 HTTPS 配置設定為應用程式根目錄的 URL。 同樣地，您應該將 secretKey 設定為您應用程式的簽署金鑰值。 請勿散發用戶端中的簽署金鑰，因為它可用來仿造金鑰和模擬使用者。 您可以藉由參考 WEBSITE\_AUTH\_SIGNING\_KEY 環境變數，在裝載於 App Service 時取得此簽署金鑰。 如果在本機偵錯內容中有需要，請依照 [使用驗證進行本機偵錯](#local-debug) 一節中的指示以取出金鑰，並將它儲存為應用程式設定。

發行的權杖可能也包含其他宣告和到期日。  發行的權杖至少必須包含一個主體 (**sub**) 宣告。

您可以藉由多載驗證路由來支援標準用戶端 `loginAsync()` 方法。  如果用戶端呼叫 `client.loginAsync('custom');` 進行登入，您的路由必須是 `/.auth/login/custom`。  您可以使用 `MapHttpRoute()`設定自訂驗證控制器的路由：

    config.Routes.MapHttpRoute("custom", ".auth/login/custom", new { controller = "CustomAuth" });

> [!TIP]
> 使用 `loginAsync()` 方法以確保驗證權杖會附加至後續對服務的呼叫。
>
>

### <a name="user-info"></a>做法：取出已驗證的使用者資訊
當 App Service 驗證使用者時，您可以存取指派的使用者識別碼及其他 .NET 後端程式碼中的資訊。 此使用者資訊可用於在後端進行授權決策。 下列程式碼會取得與要求相關聯的使用者識別碼︰

    // Get the SID of the current user.
    var claimsPrincipal = this.User as ClaimsPrincipal;
    string sid = claimsPrincipal.FindFirst(ClaimTypes.NameIdentifier).Value;

SID 衍生自提供者特定的使用者識別碼，且對指定的使用者和登入提供者而言都是靜態的。  SID 對無效的驗證權杖無效。

App Service 也可讓您向登入提供者要求特定宣告。 每個識別提供者均可使用識別提供者 SDK 來提供更多資訊。  例如，您可以將 Facebook 圖形 API 用於朋友的資訊。  您可以指定在 Azure 入口網站的提供者刀鋒視窗中要求的宣告。 某些宣告需要搭配識別提供者的額外設定。

下列程式碼會呼叫 **GetAppServiceIdentityAsync** 擴充方法以取得登入認證，其中包含對 Facebook 圖形 API 提出要求所需的存取權杖：

    // Get the credentials for the logged-in user.
    var credentials =
        await this.User
        .GetAppServiceIdentityAsync<FacebookCredentials>(this.Request);

    if (credentials.Provider == "Facebook")
    {
        // Create a query string with the Facebook access token.
        var fbRequestUrl = "https://graph.facebook.com/me/feed?access_token="
            + credentials.AccessToken;

        // Create an HttpClient request.
        var client = new System.Net.Http.HttpClient();

        // Request the current user info from Facebook.
        var resp = await client.GetAsync(fbRequestUrl);
        resp.EnsureSuccessStatusCode();

        // Do something here with the Facebook user information.
        var fbInfo = await resp.Content.ReadAsStringAsync();
    }

新增 `System.Security.Principal` 的 using 陳述式，以提供 **GetAppServiceIdentityAsync** 擴充方法。

### <a name="authorize"></a>做法︰限制授權使用者的資料存取
在上一節中，我們已說明如何取出已驗證使用者的使用者識別碼。 您可以根據此值限制存取資料和其他資源。 例如，將 userId 資料行新增到資料表，以及依使用者識別碼篩選查詢結果，是一種將傳回的資料限制為只有授權使用者的簡單方式。 下列程式碼只有在 SID 符合 TodoItem 資料表上 UserId 資料行中的值時才會傳回資料：

    // Get the SID of the current user.
    var claimsPrincipal = this.User as ClaimsPrincipal;
    string sid = claimsPrincipal.FindFirst(ClaimTypes.NameIdentifier).Value;

    // Only return data rows that belong to the current user.
    return Query().Where(t => t.UserId == sid);

`Query()` 方法會傳回可由 LINQ 操作的 `IQueryable`來處理篩選。

## <a name="how-to-add-push-notifications-to-a-server-project"></a>做法：將推播通知新增至伺服器專案
透過擴充 **MobileAppConfiguration** 物件並建立通知中樞用戶端，將推播通知加入您的伺服器專案中。

1. 在 Visual Studio 中，以滑鼠右鍵按一下伺服器專案並按一下 [管理 NuGet 封裝]，搜尋 `Microsoft.Azure.Mobile.Server.Notifications`，然後按一下 [安裝]。
2. 重複這個步驟以安裝 `Microsoft.Azure.NotificationHubs` 封裝，其中包含通知中樞用戶端程式庫。
3. 在 App_Start/Startup.MobileApp.cs 中，於初始化期間新增對 **AddPushNotifications** 擴充方法的呼叫：

        new MobileAppConfiguration()
            // other features...
            .AddPushNotifications()
            .ApplyTo(config);
4. 新增下列建立通知中樞用戶端的程式碼：

        // Get the settings for the server project.
        HttpConfiguration config = this.Configuration;
        MobileAppSettingsDictionary settings =
            config.GetMobileAppSettingsProvider().GetMobileAppSettings();

        // Get the Notification Hubs credentials for the Mobile App.
        string notificationHubName = settings.NotificationHubName;
        string notificationHubConnection = settings
            .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

        // Create a new Notification Hub client.
        NotificationHubClient hub = NotificationHubClient
            .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

您現在可以使用通知中樞用戶端，將推播通知傳送到已註冊的裝置。 如需詳細資訊，請參閱 [將推播通知新增至您的應用程式](app-service-mobile-ios-get-started-push.md)。 若要進一步了解通知中樞，請參閱 [通知中樞概觀](../notification-hubs/notification-hubs-push-notification-overview.md)。

## <a name="tags"></a>作法：使用標籤啟用有目標的推播
通知中樞可讓您使用標記將目標通知傳送至特定註冊。 自動建立數個標籤︰

* 安裝識別碼會識別特定裝置。
* 以驗證的 SID 為基礎的使用者識別碼會識別特定使用者。

您可以從 **MobileServiceClient** 上的 **installationId** 屬性存取安裝識別碼。  以下範例示範如何在通知中樞內使用安裝識別碼將標記加入特定安裝︰

    hub.PatchInstallation("my-installation-id", new[]
    {
        new PartialUpdateOperation
        {
            Operation = UpdateOperationType.Add,
            Path = "/tags",
            Value = "{my-tag}"
        }
    });

建立安裝時，後端會忽略用戶端在推播通知註冊期間提供的任何標籤。 若要讓用戶端將標籤加入安裝，您必須建立自訂 API，以便加入使用上述模式的標籤。

如需相關範例，請參閱 App Service Mobile Apps 完整快速入門範例中的[用戶端新增的推播通知標籤][5]。

## <a name="push-user"></a>做法：將推播通知傳送給已驗證的使用者
當驗證的使用者註冊推播通知之後，使用者識別碼便會自動加入到註冊中。 藉由使用這個標籤，您可以傳送推播通知給該人員已註冊的所有裝置。 下列程式碼會取得提出要求之使用者的 SID，並將範本推播通知傳送至該人員的每個裝置註冊︰

    // Get the current user SID and create a tag for the current user.
    var claimsPrincipal = this.User as ClaimsPrincipal;
    string sid = claimsPrincipal.FindFirst(ClaimTypes.NameIdentifier).Value;
    string userTag = "_UserId:" + sid;

    // Build a dictionary for the template with the item message text.
    var notification = new Dictionary<string, string> { { "message", item.Text } };

    // Send a template notification to the user ID.
    await hub.SendTemplateNotificationAsync(notification, userTag);

在註冊來自已驗證用戶端的推播通知時，請確定驗證已完成，然後再嘗試註冊。 如需詳細資訊，請參閱適用於 .NET 後端之 App Service Mobile Apps 完整快速入門範例中的[推播給使用者][6]。

## <a name="how-to-debug-and-troubleshoot-the-net-server-sdk"></a>做法：針對 .NET 伺服器 SDK 進行偵錯和疑難排解
Azure App Service 提供了數個適用於 ASP.NET 應用程式的偵錯和疑難排解技術：

* [監視 Azure App Service](../app-service/web-sites-monitor.md)
* [在 Azure App Service 中啟用診斷記錄](../app-service/web-sites-enable-diagnostic-log.md)
* [在 Visual Studio 中疑難排解 Azure App Service](../app-service/web-sites-dotnet-troubleshoot-visual-studio.md)

### <a name="logging"></a>記錄
您可以使用標準的 ASP.NET 追蹤寫入來寫入 App Service 診斷記錄： 您必須在行動應用程式後端中啟用診斷，才能寫入至記錄檔。

若要啟用診斷並寫入至記錄檔：

1. 依照 [如何啟用診斷](../app-service/web-sites-enable-diagnostic-log.md#enablediag)中的步驟執行。
2. 在您的程式碼檔案中新增下列 using 陳述式：

        using System.Web.Http.Tracing;
3. 建立從 .NET 後端寫入至診斷記錄的追蹤寫入器，如下所示：

        ITraceWriter traceWriter = this.Configuration.Services.GetTraceWriter();
        traceWriter.Info("Hello, World");
4. 重新發佈您的伺服器專案，並存取行動應用程式後端，以執行記錄的程式碼路徑。
5. 下載記錄並進行評估，如 [作法：下載記錄](../app-service/web-sites-enable-diagnostic-log.md#download)中所述。

### <a name="local-debug"></a>使用驗證進行本機偵錯
您可以在將變更發佈至雲端之前，在本機執行您的應用程式以測試變更。 對於大部分的 Azure Mobile Apps 後端，請在Visual Studio 中時按 F5  。 不過，使用驗證時有一些其他考量。

您必須擁有雲端式行動應用程式並且已設定 App Service 驗證/授權，而且您的用戶端必須有指定的雲端端點做為替代登入主機。 請參閱您用戶端平台的文件，以取得所需的特定步驟。

確定您的行動後端已安裝 [Microsoft.Azure.Mobile.Server.Authentication] 。 然後，在 `MobileAppConfiguration` 已套用至您的 `HttpConfiguration` 之後，在您的應用程式的 OWIN 啟動類別中加入下列項目：

        app.UseAppServiceAuthentication(new AppServiceAuthenticationOptions()
        {
            SigningKey = ConfigurationManager.AppSettings["authSigningKey"],
            ValidAudiences = new[] { ConfigurationManager.AppSettings["authAudience"] },
            ValidIssuers = new[] { ConfigurationManager.AppSettings["authIssuer"] },
            TokenHandler = config.GetAppServiceTokenHandler()
        });

在上述範例中，您應該使用 HTTPS 配置，將 Web.config 檔案中的 authAudience 和 authIssuer 應用程式設定，都設定為您應用程式根目錄的 URL。 同樣地，您應該將 authSigningKey 設定為您應用程式的簽署金鑰值。
若要取得簽署金鑰：

1. 在 [Azure 入口網站]
2. 按一下 [工具]、[Kudu]、[執行]。
3. 在 Kudu 管理網站中，按一下 [環境] 。
4. 尋找 *WEBSITE\_AUTH\_SIGNING\_KEY* 的值。

在本機應用程式組態中使用 authSigningKey  參數的簽署金鑰。您的行動後端現已裝備，可在本機執行時驗證用戶端從雲端式端點取得的權杖。

[1]: https://msdn.microsoft.com/library/azure/dn961176.aspx
[2]: https://github.com/Azure/azure-mobile-apps-net-server
[3]: app-service-mobile-ios-get-started.md
[4]: https://azure.microsoft.com/downloads/
[5]: https://github.com/Azure-Samples/app-service-mobile-dotnet-backend-quickstart/blob/master/README.md#client-added-push-notification-tags
[6]: https://github.com/Azure-Samples/app-service-mobile-dotnet-backend-quickstart/blob/master/README.md#push-to-users
[Azure 入口網站]: https://portal.azure.com
[NuGet.org]: http://www.nuget.org/
[Microsoft.Azure.Mobile.Server]: http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/
[Microsoft.Azure.Mobile.Server.Quickstart]: http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Quickstart/
[Microsoft.Azure.Mobile.Server.Authentication]: http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Authentication/
[Microsoft.Azure.Mobile.Server.Login]: http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Login/
[Microsoft.Azure.Mobile.Server.Notifications]: http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Notifications/
[MapHttpAttributeRoutes]: https://msdn.microsoft.com/library/dn479134(v=vs.118).aspx
