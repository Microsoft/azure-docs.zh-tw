---
title: 從行動服務升級為 Azure App Service - Node.js
description: 了解如何輕鬆地將您的行動服務應用程式升級為 App Service 行動 App
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: yochayk
editor: ''
ms.assetid: c58f6df0-5aad-40a3-bddc-319c378218e3
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile
ms.devlang: node
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 07ebf23270addc63793278d4e0510c187289b82c
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32154540"
---
# <a name="upgrade-your-existing-nodejs-azure-mobile-service-to-app-service"></a>將您現有的 Node.js Azure 行動服務升級為 App Service
App Service Mobile 是一種使用 Microsoft Azure 建置行動應用程式的新方式。 若要深入了解，請參閱 [何謂 Mobile Apps？]

本文說明如何將現有的 Node.js 後端應用程式從 Azure Mobile Service 升級為新的 App Service Mobile Apps。 執行此升級時，您現有的行動服務應用程式可以繼續運作。  如果您需要升級 Node.js 後端應用程式，請參閱[升級 .NET 行動服務](app-service-mobile-net-upgrading-from-mobile-services.md)。

當行動後端升級為 Azure App Service 時，就會具備所有 App Service 功能的存取權，而且會根據 [App Service 定價]而不是行動服務定價進行計費。

## <a name="migrate-vs-upgrade"></a>移轉與升級
[!INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

> [!TIP]
> 建議您在升級之前，先 [執行移轉](app-service-mobile-migrating-from-mobile-services.md) 。 如此一來，您就能夠在同一個 App Service 方案中放置兩個版本的應用程式，而不需支付額外成本。
>
>

### <a name="improvements-in-mobile-apps-nodejs-server-sdk"></a>Mobile Apps Node.js 伺服器 SDK 中的改進功能
升級至新的 [Mobile Apps SDK](https://www.npmjs.com/package/azure-mobile-apps) 提供了許多改進功能，包括：

* 根據 [Express 架構](http://expressjs.com/en/index.html)，新的節點 SDK 是輕量型，而其設計目的是在它們發佈時可以跟上新的節點版本。您可以利用 Express 中介軟體來自訂應用程式行為。
* 相較於行動服務 SDK，有顯著的效能改進。
* 您現在可以將網站和您的行動後端裝載在一起。同樣地，很容易就能將 Azure Mobile SDK 新增至任何現有的 express.v4 應用程式。
* Mobile Apps SDK 是建置來進行跨平台和本機開發，可在 Windows、Linux 和 OSX 平台上進行本機開發與執行。 現在常見的節點開發技術非常容易使用，像是在部署之前執行 [Mocha](https://mochajs.org/) 測試。

## <a name="overview"></a>基本升級概觀
為了協助升級 Node.js 後端，Azure App Service 提供了相容性套件。  在升級之後，您將會擁有可部署到新 App Service 網站的全新網站。

行動服務用戶端 SDK 與新的 Mobile Apps 伺服器 SDK「不」  相容。 為了提供您應用程式的服務持續性，您不應該將變更發佈至目前正在服務已發佈之用戶端的網站。 而是應該建立新的行動應用程式做為重複項目。 您可以在同一個 App Service 方案中放置此應用程式，以避免產生額外的財務成本。

您之後會有兩個版本的應用程式：一個維持不變並為已發佈的現有應用程式提供服務，另一個則可升級且目標為新的用戶端版本。 您可依自己的步調移動並測試程式碼，但應該確定您所進行的任何錯誤修正都會套用到這兩個版本。 當您覺得已將現有用戶端 app 的所需數量升級到最新版本，就可以視需要刪除原本移轉的 app。 如果裝載於與您行動 app 相同的 App Service 方案中，就不會產生任何額外的金錢成本。

完整的升級程序大致如下：

1. 下載您現有的 (已移轉) Azure 行動服務。
2. 使用相容性套件將專案轉換至 Azure 行動應用程式。
3. 更正任何差異 (例如驗證設定)。
4. 將轉換後的 Azure 行動應用程式專案部署到新的 App Service。
5. 發行使用新行動應用程式的新版用戶端應用程式。
6. (可省略) 刪除您已移轉的原始行動服務應用程式。

當已移轉的原始行動服務沒有任何流量時，就能加以刪除。

## <a name="install-npm-package"></a> 安裝必要元件
您應該在本機電腦上安裝 [節點]。  您也應該安裝相容性套件。  安裝了節點之後，您可以從新的 cmd 或 PowerShell 命令提示字元執行下列命令︰

```npm i -g azure-mobile-apps-compatibility```

## <a name="obtain-ams-scripts"></a> 取得 Azure 行動服務指令碼
* 登入 [Azure 入口網站]。
* 使用 [所有資源] 或 [應用程式服務]，尋找您的行動服務網站。
* 在網站內按一下 [工具] -> [Kudu] -> [執行] 以開啟 Kudu 網站。
* 按一下 [偵錯主控台] -> [PowerShell] 以開啟偵錯主控台。
* 輪流按一下每個目錄以瀏覽至 `site/wwwroot/App_Data/config`
* 按一下 `scripts` 目錄旁的 [下載] 圖示。

這會下載 ZIP 格式的指令碼。  在本機電腦上建立新的目錄，並在該目錄內解壓縮 `scripts.ZIP` 檔案。  這會建立 `scripts` 目錄。

## <a name="scaffold-app"></a> 建立新的 Azure Mobile Apps 後端的結構
從包含指令碼目錄的目錄執行下列命令︰

```scaffold-mobile-app scripts out```

這會在 `out` 目錄中建立已建立結構的 Azure Mobile Apps 後端。  雖然並非必要，但最好將 `out` 目錄放置到您選擇的原始程式碼儲存機制。

## <a name="deploy-ama-app"></a> 部署 Azure Mobile Apps 後端
在部署期間，您必須執行下列動作︰

1. 在 [Azure 入口網站]中建立新的行動應用程式。
2. 在連線的資料庫上執行 `createViews.sql` 指令碼。
3. 將連結至行動服務的資料庫連結至新的 App Service。
4. 將任何其他資源 (例如通知中樞) 連結到新的 App Service。
5. 將產生的程式碼部署到新網站。

### <a name="create-a-new-mobile-app"></a>建立新的行動 App
1. 登入 [Azure 入口網站]。
2. 按一下 [+ 新增]  >  [Web + 行動]  >  [行動應用程式]，然後為您的行動應用程式後端提供名稱。
3. 針對 [資源群組] ，選取現有的資源群組或建立新的群組 (使用與應用程式相同的名稱)。

    您可以選取另一個 App Service 方案或建立新方案。 如需有關應用程式服務方案以及如何在不同的定價層和您所要的位置建立新方案的詳細資訊，請參閱 [Azure App Service 方案深入概觀](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)。
4. 若為 **App Service 方案**，則會選取預設方案 (在 [標準層](https://azure.microsoft.com/pricing/details/app-service/))。 您也可以選取不同的方案，或[建立新的方案](../app-service/app-service-plan-manage.md#create-an-app-service-plan)。 App Service 方案的設定會決定與您應用程式相關聯的[位置、功能、成本和計算資源](https://azure.microsoft.com/pricing/details/app-service/)。

    在決定方案之後，按一下 [建立] 。 這會建立行動應用程式後端。

### <a name="run-createviewssql"></a>執行 CreateViews.SQL
已建立結構的應用程式包含稱為 `createViews.sql`的檔案。  您必須對目標資料庫執行這個指令碼。  您可自 [連接字串] 下方的 [設定] 頁面，從已移轉的行動服務取得目標資料庫的連接字串。  其名稱為 `MS_TableConnectionString`。

您可以從 SQL Server Management Studio 或 Visual Studio 內執行這個指令碼。

### <a name="link-the-database-to-your-app-service"></a>將資料庫連結到 App Service
將現有資料庫連結到 App Service：

* 在 [Azure 入口網站]中，開啟您的 App Service。
* 選取 [所有設定] -> [資料連接]。
* 按一下 [+ 新增] 。
* 在下拉式清單中，選取 [SQL Database] 
* 在 [SQL Database] 底下選取現有資料庫，然後按一下 [選取]。
* 在 [連接字串] 底下，輸入資料庫的使用者名稱和密碼，然後按一下 [確定]。
* 在 [新增資料連接] 頁面中，按一下 [確定]。

檢視已移轉的行動服務中目標資料庫的連接字串，就能找到使用者名稱和密碼。

### <a name="set-up-authentication"></a>設定驗證
Azure Mobile Apps 可讓您在服務內設定 Azure Active Directory、Facebook、Google、Microsoft 和 Twitter 驗證。  自訂驗證則必須另外開發。  如需詳細資訊，請參閱[驗證概念]文件和[驗證快速入門]文件。  

## <a name="updating-clients"></a>更新行動用戶端
在您擁有可運作的行動 App 後端之後，就能在取用它的新版用戶端應用程式上運作。 Mobile Apps 也會包含新版的用戶端 SDK，而且與上述的伺服器升級類似，您必須先移除所有對行動服務 SDK 的參考，然後安裝 Mobile Apps 版本。

版本間的其中一個主要變更是建構函式不再需要應用程式金鑰。
您現在只需傳入行動 App 的 URL。 例如，在 .NET 用戶端上， `MobileServiceClient` 建構函式現在是：

        public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://contoso.azurewebsites.net" // URL of the Mobile App
        );

您可以透過下列連結，閱讀有關安裝新的 SDK 以及使用新結構的相關資訊：

* [Android 2.2 版或更新版本](app-service-mobile-android-how-to-use-client-library.md)
* [iOS 3.0.0 版或更新版本](app-service-mobile-ios-how-to-use-client-library.md)
* [.NET (Windows/Xamarin) 2.0.0 版或更新版本](app-service-mobile-dotnet-how-to-use-client-library.md)
* [Apache Cordova 2.0 版或更新版本](app-service-mobile-cordova-how-to-use-client-library.md)

如果您的應用程式會使用推播通知，請記下每個平台特定的註冊指示，因為也已有一些變更。

當您準備好新的用戶端版本時，請嘗試對已升級的伺服器專案執行該版本。 驗證它的運作方式之後，您就能將新版的應用程式發行給客戶。 最後，在您的客戶有機會接收這些更新後，您就能刪除應用程式的行動服務版本。 現在，您已使用最新的 Mobile Apps 伺服器 SDK 完全升級為 App Service 行動應用程式。

<!-- URLs. -->

[Azure 入口網站]: https://portal.azure.com/
[Azure classic portal]: https://manage.windowsazure.com/
[何謂 Mobile Apps？]: app-service-mobile-value-prop.md
[I already use web sites and mobile services – how does App Service help me?]: /en-us/documentation/articles/app-service-mobile-value-prop-migration-from-mobile-services
[Mobile App Server SDK]: https://www.npmjs.com/package/azure-mobile-apps
[Create a Mobile App]: app-service-mobile-xamarin-ios-get-started.md
[Add push notifications to your mobile app]: app-service-mobile-xamarin-ios-get-started-push.md
[Add authentication to your mobile app]: app-service-mobile-xamarin-ios-get-started-users.md
[Azure Scheduler]: /en-us/documentation/services/scheduler/
[Web Job]: https://github.com/Azure/azure-webjobs-sdk/wiki
[How to use the .NET server SDK]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[Migrate from Mobile Services to an App Service Mobile App]: app-service-mobile-migrating-from-mobile-services.md
[Migrate your existing Mobile Service to App Service]: app-service-mobile-migrating-from-mobile-services.md
[App Service 定價]: https://azure.microsoft.com/pricing/details/app-service/
[.NET server SDK overview]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[驗證概念]: ../app-service/app-service-authentication-overview.md
[驗證快速入門]: app-service-mobile-auth.md

[Azure 入口網站]: https://portal.azure.com/
[OData]: http://www.odata.org
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[basicapp sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/basic-app
[todo sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/todo
[samples directory on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples
[static-schema sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/static-schema
[QueryJS]: https://github.com/Azure/queryjs
[Node.js Tools 1.1 for Visual Studio]: https://github.com/Microsoft/nodejstools/releases/tag/v1.1-RC.2.1
[mssql Node.js package]: https://www.npmjs.com/package/mssql
[Microsoft SQL Server 2014 Express]: http://www.microsoft.com/en-us/server-cloud/Products/sql-server-editions/sql-server-express.aspx
[ExpressJS Middleware]: http://expressjs.com/guide/using-middleware.html
[Winston]: https://github.com/winstonjs/winston
