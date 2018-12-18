---
title: 從行動服務升級為 Azure App Service
description: 了解如何輕鬆地將您的行動服務應用程式升級為 App Service 行動 App
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 9c0ac353-afb6-462b-ab94-d91b8247322f
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 25eb5c732927dcfb18bfd92991391ff99d4e3629
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918253"
---
# <a name="upgrade-your-existing-net-azure-mobile-service-to-app-service"></a>將您現有的 .NET Azure 行動服務升級為 App Service
App Service Mobile 是一種使用 Microsoft Azure 建置行動應用程式的新方式。 若要深入了解，請參閱 [什麼是 Mobile Apps？]

本主題說明如何將現有的 .NET 後端應用程式從 Azure 行動服務升級為新的 App Service Mobile Apps。 執行此升級時，您現有的行動服務應用程式可以繼續運作。   如果您需要升級 Node.js 後端應用程式，請參閱[升級 Node.js 行動服務](app-service-mobile-node-backend-upgrading-from-mobile-services.md)。

當行動後端升級為 Azure App Service 時，就會具備所有 App Service 功能的存取權，而且會根據 [App Service 定價]而不是行動服務定價進行計費。

## <a name="migrate-vs-upgrade"></a>移轉與升級
[!INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

> [!TIP]
> 建議您在升級之前，先 [執行移轉](app-service-mobile-migrating-from-mobile-services.md) 。 如此一來，您就能夠在同一個 App Service 方案中放置兩個版本的應用程式，而不需支付額外成本。
>
>

### <a name="improvements-in-mobile-apps-net-server-sdk"></a>Mobile Apps .NET 伺服器 SDK 中的增強功能
升級為新的 [Mobile Apps SDK](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/) 提供下列優點：

* NuGet 相依性有更多彈性。 裝載環境不再提供它自己的 NuGet 封裝版本，因此您可以使用替代的相容版本。 不過，如果行動伺服器 SDK 或相依性有新的重要 Bug 修正或安全性更新，您必須手動更新您的服務。
* Mobile SDK 有更多彈性。 您可以明確地控制設定哪些功能和路由，例如驗證、資料表 API，以及推播註冊端點。 若要深入了解，請參閱[如何使用適用於 Azure Mobile Apps 的 .NET 伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。
* 支援其他 ASP.NET 專案類型和路由。 您現在可以在與行動後端專案相同的專案中裝載 MVC 和 Web API 控制器。
* 支援新的 App Service 驗證功能，可讓您跨 Web 和行動應用程式使用常見的驗證設定。

## <a name="overview"></a>基本升級概觀
在許多情況下，只需切換到新的 Mobile Apps 伺服器 SDK 並將程式碼重新發佈至新的行動 App 執行個體，即可完成升級。 但在某些情況下則需要一些額外的設定，例如進階驗證案例和使用排程工作。 後續各節將逐一加以討論。

> [!TIP]
> 建議您先閱讀並徹底了解本主題的其餘部分，再開始升級。 請記下您使用的下列任何功能。
>
>

行動服務用戶端 SDK 與新的 Mobile Apps 伺服器 SDK「不」  相容。 為了提供您應用程式的服務持續性，您不應該將變更發佈至目前正在服務已發佈之用戶端的網站。 而是應該建立新的行動應用程式做為重複項目。 您可以在同一個 App Service 方案中放置此應用程式，以避免產生額外的財務成本。

您之後會有兩個版本的應用程式：一個維持不變並為已發佈的現有應用程式提供服務，另一個則可升級且目標為新的用戶端版本。 您可依自己的步調移動並測試程式碼，但應該確定您所進行的任何錯誤修正都會套用到這兩個版本。 當您覺得已將現有用戶端應用程式的所需數量升級到最新版本，就可以視需要刪除原本移轉的應用程式。

完整的升級程序大致如下：

1. 建立新的行動 App
2. 更新專案以使用新的伺服器 SDK
3. 發行新版的用戶端應用程式
4. (選擇性) 刪除原始已移轉的執行個體

## <a name="mobile-app-version"></a>建立第二個應用程式執行個體
升級的第一個步驟是建立將主控新版應用程式的行動 App 資源。 如果您已經移轉現有的行動服務，則您會想要在同一個主控方案中建立這個版本。 請開啟 [Azure 入口網站] ，並瀏覽到您已移轉的應用程式。 請記下其正在其上執行的 App Service 方案。

接下來，依照 [.NET 後端建立指示](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#create-app)建立第二個應用程式執行個體。 當系統提示選取您的 App Service 方案或「主控方案」時，請選擇已移轉之應用程式的方案。

您可能會想要使用與行動服務中相同的資料庫和通知中心。 您可以複製這些值，方法是開啟 [Azure 入口網站]、瀏覽至原始的應用程式，然後依序按一下 [設定] > [應用程式設定]。 在 [連接字串] 下，複製 `MS_NotificationHubConnectionString` 和 `MS_TableConnectionString`。 瀏覽至新的升級網站並將它們貼上，覆寫任何現有的值。 針對您的應用程式需要的任何其他應用程式設定重複執行此程序。 如果不使用移轉的服務，您可以從 [Azure 傳統入口網站]上 [行動服務] 區段的 [設定] 索引標籤，讀取連接字串和 app 設定。

針對您的應用程式製作 ASP.NET 專案的複本，然後將它發佈到新的網站。 透過利用新 URL 更新的用戶端應用程式複本，驗證一切運作正常。

## <a name="updating-the-server-project"></a>更新伺服器專案
行動應用程式有新的 [行動應用程式伺服器 SDK] ，提供許多與行動服務執行階段相同的功能。 首先，您應該移除對行動服務封裝的所有參考。 在 NuGet 封裝管理員中，搜尋 `WindowsAzure.MobileServices.Backend`。 大部分的 app 將會在此處看見數個封裝，包括 `WindowsAzure.MobileServices.Backend.Tables` 和 `WindowsAzure.MobileServices.Backend.Entity`。 在這種情況下，請從相依性樹狀目錄中的最低封裝 (例如 `Entity`) 開始，然後將它移除。 出現提示時，請勿移除所有相依的封裝。 重複執行此程序，直到您移除了 `WindowsAzure.MobileServices.Backend` 本身為止。

此時，您的專案將不再參考行動服務 SDK。

接下來，將新增 Mobile Apps SDK 的參考。 在此升級中，大部分的開發人員都想要下載並安裝 `Microsoft.Azure.Mobile.Server.Quickstart` 封裝，因為這將會在整個必要集中提取。

屆時將會有不少因 SDK 之間的差異而產生的編譯器錯誤，但這些錯誤都很容易處理且將於本節的其餘部分加以說明。

### <a name="base-configuration"></a>基本組態
然後，在 WebApiConfig.cs 中，您可以取代：

        // Use this class to set configuration options for your mobile service
        ConfigOptions options = new ConfigOptions();

        // Use this class to set WebAPI configuration options
        HttpConfiguration config = ServiceConfig.Initialize(new ConfigBuilder(options));

取代為

        HttpConfiguration config = new HttpConfiguration();
        new MobileAppConfiguration()
            .UseDefaultConfiguration()
        .ApplyTo(config);

> [!NOTE]
> 如果您想要深入了解新的 .NET 伺服器 SDK 以及如何從您的 app 新增/移除功能，請參閱 [如何使用 .NET 伺服器 SDK] 主題。
>
>

如果您的應用程式會使用驗證功能，您也必須註冊 OWIN 中介軟體。 在此情況下，您應該將上述組態程式碼移入新的 OWIN 啟動類別。

1. 如果 NuGet 封裝 `Microsoft.Owin.Host.SystemWeb` 尚未包含於專案中，請新增它。
2. 在 Visual Studio 中，以滑鼠右鍵按一下專案，然後選取 [新增] -> [新項目]。 依序選取 [Web] -> [一般] -> [OWIN 啟動類別]。
3. 將 MobileAppConfiguration 的上述程式碼從 `WebApiConfig.Register()` 移至您新啟動類別的 `Configuration()` 方法。

確定 `Configuration()` 方法的結尾：

        app.UseWebApi(config)
        app.UseAppServiceAuthentication(config);

有一些其他與驗證相關的變更，這些內容涵蓋於下列的完整驗證章節中。

### <a name="working-with-data"></a>使用資料
在行動服務中，會提供行動 app 名稱做為 Entity Framework 安裝程式中的預設結構描述名稱。

若要確定您的結構描述與之前參考的一樣，請使用下列內容來設定 DbContext 中適用於您應用程式的結構描述：

        string schema = System.Configuration.ConfigurationManager.AppSettings.Get("MS_MobileServiceName");

如果您執行了上述設定，請確定您具有 MS_MobileServiceName。 如果您的應用程式先前已自訂這個結構描述，您也可以提供另一個結構描述名稱。

### <a name="system-properties"></a>系統屬性
#### <a name="naming"></a>命名
在 Azure 行動服務伺服器 SDK 中，系統屬性一律會包含適用於屬性的雙底線 (`__`) 前置詞：

* __createdAt
* __updatedAt
* __deleted
* __version

行動服務用戶端 SDK 具有特殊的邏輯，可用來剖析這種格式的系統屬性。

在 Azure Mobile Apps 中，系統屬性不再具有特殊格式並具有下列名稱：

* 建立時間
* 更新時間
* 已刪除
* version

Mobile Apps 用戶端 SDK 會使用新的系統屬性名稱，因此不需要對用戶端程式碼進行任何變更。 不過，如果您要直接對服務進行 REST 呼叫，則您應該據以變更您的查詢。

#### <a name="local-store"></a>本機存放區
變更系統屬性的名稱，表示適用於行動服務的離線同步處理本機資料庫與 Mobile Apps 不相容。 如果可能，在將暫止的變更傳送到伺服器之前，您應該避免將用戶端應用程式從行動服務升級為 Mobile Apps。 接著，升級的應用程式應該使用新的資料庫檔案名稱。

如果用戶端應用程式是從行動服務升級為 Mobile Apps，但同時在操作佇列中有暫止的離線變更，則系統資料庫必須更新，才能使用新的資料行名稱。 在 iOS 上，可以使用輕量型移轉來變更資料行名稱，以達成這一點。 在 Android 和 .NET 受控用戶端上，您應該撰寫自訂的 SQL，為資料物件資料表的資料行重新命名。

在 iOS 上，您應該變更資料實體的核心資料結構描述，來符合下列內容。 請注意，屬性 `createdAt`、`updatedAt` 和 `version` 不再需要 `ms_` 前置詞：

| 屬性 | 類型 | 附註 |
| --- | --- | --- |
| id |字串 (標示為必要) |遠端存放區中的主索引鍵 |
| 建立時間 |日期 |(選擇性) 對應至 createdAt 系統屬性 |
| 更新時間 |日期 |(選擇性) 對應至 updatedAt 系統屬性 |
| version |字串 |(選擇性) 用來偵測衝突，對應至版本 |

#### <a name="querying-system-properties"></a>查詢系統屬性
在 Azure 行動服務中，預設不會傳送系統屬性，只有在使用查詢字串 `__systemProperties`要求它們時才會傳送。 相反地，在 Azure Mobile Apps 中，會 **一律選取** 系統屬性，因為它們是伺服器 SDK 物件模型的一部分。

這項變更主要會影響網域管理員的自訂實作，例如 `MappedEntityDomainManager`的延伸模組。 在行動服務中，如果用戶端永遠不會要求任何系統屬性，就可以使用 `MappedEntityDomainManager` ，這實際上不會對應所有屬性。 不過，在 Azure Mobile Apps 中，這些未對應的屬性將導致 GET 查詢發生錯誤。

解決此問題的最簡單方式是修改您的 DTO，讓它們繼承自 `ITableData` 而非 `EntityData`。 接著，將 `[NotMapped]` 屬性新增到應省略的欄位。

例如，下列範例會定義 `TodoItem` 且不使用任何系統屬性：

    using System.ComponentModel.DataAnnotations.Schema;

    public class TodoItem : ITableData
    {
        public string Text { get; set; }

        public bool Complete { get; set; }

        public string Id { get; set; }

        [NotMapped]
        public DateTimeOffset? CreatedAt { get; set; }

        [NotMapped]
        public DateTimeOffset? UpdatedAt { get; set; }

        [NotMapped]
        public bool Deleted { get; set; }

        [NotMapped]
        public byte[] Version { get; set; }
    }

附註：如果您收到有關 `NotMapped` 的錯誤，請將參考新增至組件 `System.ComponentModel.DataAnnotations`。

### <a name="cors"></a>CORS
行動服務藉由包裝 ASP.NET CORS 解決方案，來包含一些適用於 CORS 的支援。 這個包裝層級已移除，以便讓開發人員擁有更多控制權，因此，您可以直接運用 [ASP.NET CORS 支援](http://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api)。

而使用 CORS 的主要考量，在於您必須允許使用 `eTag` 和 `Location` 標頭，用戶端 SDK 才能正常運作。

### <a name="push-notifications"></a>推播通知
對於推播，您可能會發現伺服器 SDK 中遺漏的主要項目是 PushRegistrationHandler 類別。 註冊在行動應用程式中處理方式稍有不同，依預設會啟用不具標籤的註冊。 管理標籤可使用自訂 API 來完成。 如需詳細資訊，請參閱 [註冊標籤](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags) 的指示。

### <a name="scheduled-jobs"></a>排程的工作
Mobile Apps 中並未內建排程的工作，因此您在 .NET 後端中的任何現有工作都必須個別升級。 其中一個選項是在行動應用程式程式碼網站上建立排程 [Web 工作] 。 您也可以設定用來保存工作程式碼的控制器，並設定依預期的排程在端點上執行的 [Azure 排程器] 。

### <a name="miscellaneous-changes"></a>其他變更
行動用戶端將取用的所有 ApiControllers 現在必須具備 `[MobileAppController]` 屬性。 預設不再包含此屬性，因此，其他 ApiControllers 不會受到行動格式器所影響。

`ApiServices` 物件不再是 SDK 的一部分。 若要存取行動應用程式設定，您可以使用下列：

    MobileAppSettingsDictionary settings = this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();

同樣地，現在已使用標準 ASP.NET 追蹤寫入完成記錄︰

    ITraceWriter traceWriter = this.Configuration.Services.GetTraceWriter();
    traceWriter.Info("Hello, World");  

## <a name="authentication"></a>驗證考量
行動服務的驗證元件現在已移到 App Service 驗證/授權功能中。 您可以閱讀 [將驗證新增至您行動應用程式](app-service-mobile-ios-get-started-users.md) 一文，以了解如何為網站啟用此功能。

對於某些提供者 (例如 AAD、Facebook 及 Google)，您應該能夠運用來自您複製應用程式的現有註冊。 您只需瀏覽至身分識別提供者的入口網站，並將新的重新導向 URL 新增至註冊即可。 接著，利用用戶端識別碼和密碼來設定 App Service 驗證/授權。

### <a name="controller-action-authorization"></a>控制器動作授權
`[AuthorizeLevel(AuthorizationLevel.User)]` 屬性的所有執行個體現在都必須變更，以使用標準的 ASP.NET `[Authorize]` 屬性。 此外，根據預設，控制器現在是匿名的，就如同在其他 ASP.NET 應用程式中。
如果您使用其中一個其他 AuthorizeLevel 選項 (例如系統管理員或應用程式)，請注意，這些項目都不見了。 您可以改為設定共用密碼的 AuthorizationFilters，或者設定 AAD 服務主體，安全地啟用服務對服務的呼叫。

### <a name="getting-additional-user-information"></a>取得其他使用者資訊
您可以取得其他使用者資訊，包括透過 `GetAppServiceIdentityAsync()` 方法存取權杖：

        FacebookCredentials creds = await this.User.GetAppServiceIdentityAsync<FacebookCredentials>();

此外，如果您的應用程式依存於使用者識別碼 (例如，將它們儲存於資料庫中)，請務必注意，行動服務和 App Service Mobile Apps 之間的使用者識別碼是不同的。 不過，您仍然可以取得行動服務使用者識別碼。 所有的 ProviderCredentials 子類別都具有 UserId 屬性。 因此，繼續之前的範例：

        string mobileServicesUserId = creds.Provider + ":" + creds.UserId;

如果您的應用程式依存於使用者識別碼，請務必盡可能使用相同的身分識別提供者註冊。 使用者識別碼的範圍通常限定在已使用的應用程式註冊，因此引入新的註冊，可能會讓使用者在比對其資料時發生問題。

### <a name="custom-authentication"></a>自訂驗證
如果您的應用程式使用自訂的驗證解決方案，您會想要確定已升級的網站具備系統的存取權。 請依照 [.NET 伺服器 SDK 概觀] 中適用於自訂驗證的新指示來整合您的解決方案。 請注意，自訂驗證元件仍處於預覽狀態。

## <a name="updating-clients"></a>更新用戶端
在您擁有可運作的行動 App 後端之後，就能在取用它的新版用戶端應用程式上運作。 Mobile Apps 也會包含新版的用戶端 SDK，而且與上述的伺服器升級類似，您必須先移除所有對行動服務 SDK 的參考，然後安裝 Mobile Apps 版本。

版本間的其中一個主要變更是建構函式不再需要應用程式金鑰。 您現在只需傳入行動 App 的 URL。 例如，在 .NET 用戶端上， `MobileServiceClient` 建構函式現在是：

        public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://contoso.azurewebsites.net", // URL of the Mobile App
        );

您可以透過下列連結，閱讀有關安裝新的 SDK 以及使用新結構的相關資訊：

* [iOS 3.0.0 版或更新版本](app-service-mobile-ios-how-to-use-client-library.md)
* [.NET (Windows/Xamarin) 2.0.0 版或更新版本](app-service-mobile-dotnet-how-to-use-client-library.md)

如果您的應用程式會使用推播通知，請記下每個平台特定的註冊指示，因為也已有一些變更。

當您準備好新的用戶端版本時，請嘗試對已升級的伺服器專案執行該版本。 驗證它的運作方式之後，您就能將新版的應用程式發行給客戶。 最後，在您的客戶有機會接收這些更新後，您就能刪除應用程式的行動服務版本。 現在，您已使用最新的 Mobile Apps 伺服器 SDK 完全升級為 App Service 行動應用程式。

<!-- URLs. -->

[Azure 入口網站]: https://portal.azure.com/
[Azure 傳統入口網站]: https://manage.windowsazure.com/
[什麼是 Mobile Apps？]: app-service-mobile-value-prop.md
[I already use web sites and mobile services – how does App Service help me?]: /en-us/documentation/articles/app-service-mobile-value-prop-migration-from-mobile-services
[行動應用程式伺服器 SDK]: http://www.nuget.org/packages/microsoft.azure.mobile.server
[Create a Mobile App]: app-service-mobile-xamarin-ios-get-started.md
[Add push notifications to your mobile app]: app-service-mobile-xamarin-ios-get-started-push.md
[Add authentication to your mobile app]: app-service-mobile-xamarin-ios-get-started-users.md
[Azure 排程器]: /azure/scheduler/
[Web 工作]: https://github.com/Azure/azure-webjobs-sdk/wiki
[如何使用 .NET 伺服器 SDK]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[Migrate from Mobile Services to an App Service Mobile App]: app-service-mobile-migrating-from-mobile-services.md
[Migrate your existing Mobile Service to App Service]: app-service-mobile-migrating-from-mobile-services.md
[App Service 定價]: https://azure.microsoft.com/pricing/details/app-service/
[.NET 伺服器 SDK 概觀]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
