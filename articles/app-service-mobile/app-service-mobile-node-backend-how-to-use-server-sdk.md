---
title: 如何使用適用於 Mobile Apps 的 Node.js 後端伺服器 SDK | Microsoft Docs
description: 了解如何使用適用於 Azure App Service Mobile Apps 的 Node.js 後端伺服器 SDK。
services: app-service\mobile
documentationcenter: ''
author: elamalani
manager: elamalani
editor: ''
ms.assetid: e7d97d3b-356e-4fb3-ba88-38ecbda5ea50
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: node
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: cbedb17bb7563620d0d9db81333d9a79301b4ee0
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42817503"
---
# <a name="how-to-use-the-mobile-apps-nodejs-sdk"></a>如何使用 Mobile Apps Node.js SDK

[!INCLUDE [app-service-mobile-selector-server-sdk](../../includes/app-service-mobile-selector-server-sdk.md)]

本文提供詳細的資訊及範例，說明如何在 Azure App Service 的 Mobile Apps 功能中使用 Node.js 後端。

## <a name="Introduction"></a>簡介

Mobile Apps 可讓您將行動最佳化資料存取 Web API 新增至 Web 應用程式。 Mobile Apps SDK 是針對 ASP.NET 和 Node.js Web 應用程式而提供的。 此 SDK 提供下列作業：

* 資料存取的資料表作業 (讀取、插入、更新、刪除)
* 自訂 API 作業

這兩項作業皆可用於 Azure App Service 所允許的所有身分識別提供者之間的驗證。 這些提供者包括 Facebook、Twitter、Google 和 Microsoft 等社交識別提供者，以及用於企業身分識別的 Azure Active Directory。

您可以在 [GitHub 上的範例目錄]中找到每個使用案例的範例。

## <a name="supported-platforms"></a>支援的平台

Mobile Apps Node.js SDK 支援 Node 目前的 LTS 版和更新版本。 目前最新的 LTS 版本是 Node v4.5.0。 其他版本的 Node 可能可以運作，但不受支援。

Mobile Apps Node.js SDK 支援兩種資料庫驅動程式： 

* node-mssql 驅動程式支援 Azure SQL Database 和本機 SQL Server 執行個體。  
* Sqlite3 驅動程式僅支援單一執行個體上的 SQLite 資料庫。

### <a name="howto-cmdline-basicapp"></a>使用命令列建立基本 Node.js 後端

每個 Mobile App Node.js 後端都會以 ExpressJS 應用程式的形式啟動。 在適用於 Node.js 的 Web 服務架構中，ExpressJS 最廣為使用。 您可以依照下列方式建立基本的 [Express] 應用程式：

1. 在命令或 PowerShell 視窗中，為您的專案建立目錄：

        mkdir basicapp

1. 執行 `npm init` 以初始化套件結構：

        cd basicapp
        npm init

   `npm init` 命令會詢問一組問題以初始化專案。 請參閱範例輸出：

   ![npm init 輸出][0]

1. 從 npm 存放庫安裝 `express` 和 `azure-mobile-apps` 資源庫：

        npm install --save express azure-mobile-apps

1. 建立 app.js 檔案以實作基本行動伺服器：

    ```javascript
    var express = require('express'),
        azureMobileApps = require('azure-mobile-apps');

    var app = express(),
        mobile = azureMobileApps();

    // Define a TodoItem table.
    mobile.tables.add('TodoItem');

    // Add the Mobile API so it is accessible as a Web API.
    app.use(mobile);

    // Start listening on HTTP.
    app.listen(process.env.PORT || 3000);
    ```

此應用程式會建立具有單一端點 (`/tables/TodoItem`) 的行動最佳化 WebAPI，讓使用者可使用動態結構描述存取基礎 SQL 資料存放區，不需經過驗證。 它適用於下列用戶端程式庫快速入門：

* [Android 用戶端快速入門]
* [Apache Cordova 用戶端快速入門]
* [iOS 用戶端快速入門]
* [Windows 市集用戶端快速入門]
* [Xamarin.iOS 用戶端快速入門]
* [Xamarin.Android 用戶端快速入門]
* [Xamarin.Forms 用戶端快速入門]

您可以在 [GitHub 上的 basicapp 範例]中找到此基本應用程式的程式碼。

### <a name="howto-vs2015-basicapp"></a>使用 Visual Studio 2015 建立 Node.js 後端

Visual Studio 2015 需要延伸模組才能在整合式開發環境 (IDE) 內開發 Node.js 應用程式。 首先，請安裝 [Node.js Tools 1.1 for Visual Studio]。 完成安裝後，請建立 Express 4.x 應用程式：

1. 開啟 [新增專案] 對話方塊 (從 [檔案] > [新增] > [專案...])。
1. 展開**範本** > **JavaScript** > **Node.js**。
1. 選取 [基本 Azure Node.js Express 4 應用程式]。
1. 填入專案名稱。 選取 [確定] 。

   ![Visual Studio 2015 新專案][1]
1. 以滑鼠右鍵按一下 **npm** 節點，然後選取 [安裝新的 npm 套件]。
1. 建立第一個 Node.js 應用程式之後，您可能必須重新整理 npm 目錄。 如有必要，請選取 [重新整理]。
1. 在搜尋方塊中輸入 **azure-mobile-apps** 。 選取 [azure-mobile-apps 2.0.0] 套件，然後選取 [安裝套件]。

   ![安裝新的 npm 套件][2]
1. 選取 [關閉] 。
1. 開啟 app.js 檔案，以新增 Mobile Apps SDK 的支援。 在程式庫 `require` 陳述式底部的第 6 行，加入下列程式碼：

    ```javascript
    var bodyParser = require('body-parser');
    var azureMobileApps = require('azure-mobile-apps');
    ```

    在其他 `app.use` 陳述式之後的大約第 27 行，加入下列程式碼：

    ```javascript
    app.use('/users', users);

    // Mobile Apps initialization
    var mobile = azureMobileApps();
    mobile.tables.add('TodoItem');
    app.use(mobile);
    ```

    儲存檔案。

1. 在本機執行應用程式 (已在 http://localhost:3000) 上提供 API)，或發佈至 Azure。

### <a name="create-node-backend-portal"></a>使用 Azure 入口網站建立 Node.js 後端

您可以在 [Azure 入口網站]中直接建立 Mobile Apps 後端。 您可以完成下列步驟，或者依照[建立行動應用程式](app-service-mobile-ios-get-started.md)教學課程，同時建立用戶端和伺服器。 本教學課程包含下列指示的簡化版本，最適用於概念驗證專案。

[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service-classic](../../includes/app-service-mobile-dotnet-backend-create-new-service-classic.md)]

回到 [開始使用] 窗格，在 [建立資料表 API] 下選擇 [Node.js]，作為您的後端語言。
選取 [我了解此將覆寫所有網站內容] 的方塊，然後選取 [建立 TodoItem 資料表]。

### <a name="download-quickstart"></a>使用 Git 下載 Node.js 後端快速入門程式碼專案

當您使用入口網站的 [快速入門] 窗格建立 Node.js Mobile Apps 後端時，系統會為您建立 Node.js 專案，並將其部署至您的網站。 您可以在入口網站中新增資料表和 API，並編輯 Node.js 後端的程式碼檔案。 您也可以使用各種部署工具來下載後端專案，以便新增或修改資料表和 API，然後重新發佈專案。 如需詳細資訊，請參閱 [Azure App Service 部署指南]。

下列程序會使用 Git 存放庫來下載快速入門專案程式碼：

1. 如果尚未安裝 Git，請先安裝。 安裝 Git 所需的步驟會因作業系統而有所不同。 如需作業系統特定的散發套件和安裝指引，請參閱[安裝 Git](http://git-scm.com/book/en/Getting-Started-Installing-Git) 。
1. 請參閱[準備您的存放庫](../app-service/app-service-deploy-local-git.md#prepare-your-repository)，啟用後端網站的 Git 存放庫。 請記下部署使用者名稱和密碼。
1. 在您 Mobile Apps 後端的視窗中，記下 [Git 複製 URL]  設定。
1. 使用 Git 複製 URL執行 `git clone` 命令。 在需要時輸入您的密碼，如下列範例所示：

        $ git clone https://username@todolist.scm.azurewebsites.net:443/todolist.git

1. 瀏覽至本機目錄 (在上述範例中為 `/todolist`)，並留意專案檔案已下載。 在 `/tables` 目錄中找出 todoitem.json 檔案。 該檔案定義了資料表上的權限。 同時，在相同的目錄中找出 todoitem.js 檔案。 它會定義資料表的 CRUD 作業指令碼。
1. 在您變更專案檔案之後，請執行下列命令，以新增、認可變更，然後將變更上傳至網站：

        $ git commit -m "updated the table script"
        $ git push origin master

   當您將新檔案新增至專案時，您必須先執行 `git add .` 命令。

每次有一組新的認可推送至網站時，就會重新發佈網站。

### <a name="howto-publish-to-azure"></a>將您的 Node.js 後端發佈至 Azure

Microsoft Azure 提供許多將 Mobile Apps Node.js 後端發佈至 Azure 服務的機制。 這些機制包括整合至 Visual Studio 中的部署工具、命令列工具，和以原始檔控制為基礎的連續部署選項。 如需詳細資訊，請參閱 [Azure App Service 部署指南]。

Azure App Service 提供 Node.js 應用程式方面的具體建議，您應該在發佈後端之前先加以檢閱：

* 如何[指定 Node 版本]
* 如何 [使用 Node 模組]

### <a name="howto-enable-homepage"></a>啟用應用程式的首頁

有許多應用程式都是 Web 和行動應用程式的組合。 您可以使用 ExpressJS 架構來結合這兩個 Facet。 但有些時候，您可能只想要實作行動介面。 提供首頁以確保 App Service 能夠啟動並執行，會很有幫助。 您可以提供您自己的首頁，或啟用暫時的首頁。 若要啟用暫時的首頁，請使用以下程式碼將 Mobile Apps 具現化︰

```javascript
var mobile = azureMobileApps({ homePage: true });
```

如果您想要讓此選項僅在本機開發時可供使用，您可以將此設定新增至 azureMobile.js 檔案。

## <a name="TableOperations"></a>資料表作業

azure-mobile-apps Node.js Server SDK 提供將儲存在 Azure SQL Database 中的資料表公開為 Web API 的機制。 提供的作業有五種：

| 作業 | 說明 |
| --- | --- |
| GET /tables/*tablename* |取得資料表中的所有記錄。 |
| GET /tables/*tablename*/:id |取得資料表中的特定記錄。 |
| POST /tables/*tablename* |在資料表中建立記錄。 |
| PATCH /tables/*tablename*/:id |更新資料表中的記錄。 |
| DELETE /tables/*tablename*/:id |刪除資料表中的記錄。 |

此 Web API 支援 [OData]，並可擴充資料表結構描述以支援[離線資料同步]。

### <a name="howto-dynamicschema"></a>使用動態結構描述定義資料表

您必須先定義資料表，才能加以使用。 您可以使用靜態結構描述來定義資料表 (您在結構描述中定義資料行)，或以動態方式定義 (SDK 會根據傳入的要求控制結構描述)。 此外，您可將 Javascript 程式碼新增至定義，以控制 Web API 的特定層面。

根據最佳做法，您應在 `tables.import()` 目錄中的 JavaScript 檔案內定義每個資料表，然後使用 `tables` 方法匯入資料表。 擴充基本應用程式範例時，您會調整 app.js 檔案：

```javascript
var express = require('express'),
    azureMobileApps = require('azure-mobile-apps');

var app = express(),
    mobile = azureMobileApps();

// Define the database schema that is exposed.
mobile.tables.import('./tables');

// Provide initialization of any tables that are statically defined.
mobile.tables.initialize().then(function () {
    // Add the Mobile API so it is accessible as a Web API.
    app.use(mobile);

    // Start listening on HTTP.
    app.listen(process.env.PORT || 3000);
});
```

在 ./tables/TodoItem.js 中定義資料表：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Additional configuration for the table goes here.

module.exports = table;
```

資料表依預設會使用動態結構描述。 若要全域關閉動態結構描述，請在 Azure 入口網站中將應用程式設定 `MS_DynamicSchema` 設為 false。

您可以在 [GitHub 上的待辦事項範例]中找到完整範例。

### <a name="howto-staticschema"></a>使用靜態結構描述定義資料表

您可以將資料行明確定義為要透過 Web API 公開。 azure-mobile-apps Node.js SDK 會自動將離線資料同步所需的任何額外的資料行新增至您所提供的清單。 例如，快速入門用戶端應用程式需要具有兩個資料行的資料表：`text` (字串) 和 `complete` (布林值)。  
此資料表可以定義在資料表定義 JavaScript 檔案中 (位於 `tables` 目錄中)，如下所示：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Define the columns within the table.
table.columns = {
    "text": "string",
    "complete": "boolean"
};

// Turn off the dynamic schema.
table.dynamicSchema = false;

module.exports = table;
```

如果您以靜態方式定義資料表，則必須同時呼叫 `tables.initialize()` 方法，以在啟動時建立資料庫結構描述。 `tables.initialize()` 方法會傳回 [Promise]，避免 Web 服務在資料庫初始化之前處理要求。

### <a name="howto-sqlexpress-setup"></a>以 SQL Server Express 作為本機電腦上的開發資料存放區

Mobile Apps Node.js SDK 提供三種現成可用的資料提供選項：

* 使用**記憶體**驅動程式，提供非持續性的範例存放區。
* 使用 **mssql** 驅動程式，提供可供開發使用的 SQL Server Express 資料存放區。
* 使用 **mssql** 驅動程式，提供可供生產使用的 Azure SQL Database 資料存放區。

Mobile Apps Node.js SDK 會使用 [mssql Node.js 套件] 來建立及使用 SQL Server Express 和 SQL Database 的連線。 要使用此套件，您必須在 SQL Server Express 執行個體上啟用 TCP 連線。

> [!TIP]
> 記憶體驅動程式未提供完整的測試工具集。 如果您想要在本機上測試後端，建議您使用 SQL Server Express 資料存放區和 mssql 驅動程式。

1. 下載並安裝 [Microsoft SQL Server 2014 Express]。 請確實安裝 SQL Server 2014 Express with Tools 版。 除非您明確需要 64 位元支援，32 位元版本在執行時會耗用較少的記憶體。
1. 執行 SQL Server 2014 組態管理員：

   a. 在樹狀結構功能表中，展開 [SQL Server 網路組態] 節點。

   b. 選取 [SQLEXPRESS 的通訊協定]。

   c. 以滑鼠右鍵按一下 [TCP/IP]，然後選取 [啟用]。 在快顯對話方塊中選取 [確定]。

   d. 以滑鼠右鍵按一下 [TCP/IP]，然後選取 [屬性]。

   e. 選取 [IP 位址] 索引標籤。

   f. 尋找 **IPAll** 節點。 在 [TCP 連接埠] 欄位中，輸入 **1433**。

      ![進行 SQL Server Express 的 TCP/IP 設定][3]

   g. 選取 [確定] 。 在快顯對話方塊中選取 [確定]。

   h. 在樹狀結構功能表中，選取 [SQL Server 服務]。

   i. 以滑鼠右鍵按一下 [SQL Server (SQLEXPRESS)]，然後選取 [重新啟動]。

   j. 關閉 SQL Server 2014 組態管理員。

1. 執行 SQL Server 2014 Management Studio，並連線至您的本機 SQL Server Express 執行個體：

   1. 以滑鼠右鍵按一下您在物件總管中的執行個體，然後選取 [屬性]。
   1. 選取 [安全性] 頁面  。
   1. 確定已選取 [SQL Server 及 Windows 驗證模式]。
   1. 選取 [確定] 。

      ![設定 SQL Server Express 驗證][4]
   1. 在物件總管中展開 [安全性] > [登入]。
   1. 以滑鼠右鍵按一下 [登入]，然後選取 [新增登入]。
   1. 輸入登入名稱。 選取 [SQL Server 驗證] 。 輸入密碼，然後在 [確認密碼] 中輸入相同的密碼。 密碼必須符合 Windows 複雜性需求。
   1. 選取 [確定] 。

      ![將新的使用者新增至 SQL Server Express][5]
   1. 以滑鼠右鍵按一下新的登入，然後選取 [屬性]。
   1. 選取 [伺服器角色] 頁面。
   1. 選取 **dbcreator** 伺服器角色的核取方塊。
   1. 選取 [確定] 。
   1. 關閉 SQL Server 2015 Management Studio。

請確實記下您選取的使用者名稱和密碼。 您可能需要根據您特定的資料庫需求，指派其他伺服器角色或權限。

Node.js 應用程式會讀取 `SQLCONNSTR_MS_TableConnectionString` 環境變數，以取得此資料庫的連接字串。 您可以將此變數設定在環境中。 例如，您可以使用 PowerShell 來設定此環境變數：

    $env:SQLCONNSTR_MS_TableConnectionString = "Server=127.0.0.1; Database=mytestdatabase; User Id=azuremobile; Password=T3stPa55word;"

透過 TCP/IP 連線存取資料庫。 提供連線的使用者名稱和密碼。

### <a name="howto-config-localdev"></a>設定專案以在本機上進行開發

Mobile Apps 會從本機檔案系統讀取名為 *azureMobile.js* 的 JavaScript 檔案。 請勿使用此檔案在生產環境中設定 Mobile Apps SDK。 請改用 [Azure 入口網站]中的 [應用程式設定]。

azureMobile.js 檔案應會匯出組態物件。 最常見的設定如下：

* 資料庫設定
* 診斷記錄設定
* 替代 CORS 設定

以下範例 **azureMobile.js** 範例會實作上述資料庫設定：

```javascript
module.exports = {
    cors: {
        origins: [ 'localhost' ]
    },
    data: {
        provider: 'mssql',
        server: '127.0.0.1',
        database: 'mytestdatabase',
        user: 'azuremobile',
        password: 'T3stPa55word'
    },
    logging: {
        level: 'verbose'
    }
};
```

建議您將 **azureMobile.js** 新增至您的 **.gitignore** 檔案 (或其他原始程式碼控制忽略檔案)，以防止密碼儲存在雲端中。 請一律在 [Azure 入口網站]內的 [應用程式設定] 中設定生產設定。

### <a name="howto-appsettings"></a>設定行動應用程式的應用程式設定

azureMobile.js 檔案中的大部分設定在 [Azure 入口網站]中都有對等的 [應用程式設定]。 請使用下列清單，在 [應用程式設定] 中設定您的應用程式：

| 應用程式設定 | azureMobile.js 設定 | 說明 | 有效值 |
|:--- |:--- |:--- |:--- |
| **MS_MobileAppName** |name |應用程式的名稱 |字串 |
| **MS_MobileLoggingLevel** |logging.level |要記錄的訊息的最小記錄層級 |error、warning、info、verbose、debug、silly |
| **MS_DebugMode** |debug |啟用或停用偵錯模式 |true、false |
| **MS_TableSchema** |data.schema |SQL 資料表的預設結構描述名稱 |字串 (預設值：dbo) |
| **MS_DynamicSchema** |data.dynamicSchema |啟用或停用偵錯模式 |true、false |
| **MS_DisableVersionHeader** |version (設定為未定義) |停用 X-ZUMO-Server-Version 標頭 |true、false |
| **MS_SkipVersionCheck** |skipversioncheck |停用用戶端 API 版本檢查 |true、false |

若要設定應用程式設定：

1. 登入 [Azure 入口網站]。
1. 選取 [所有資源] 或 [應用程式服務]，然後選取行動應用程式的名稱。
1. 根據預設，[設定] 窗格會隨即開啟。 如果沒有，請選取 [設定]。
1. 在 [一般] 功能表上，選取 [應用程式設定]。
1. 捲動至 [應用程式設定] 區段。
1. 如果您的應用程式設定已存在，請選取應用程式設定的值以編輯該值。
   如果您的應用程式設定不存在，請在 [機碼] 方塊中輸入應用程式設定，並在 [值] 方塊中輸入值。
1. 選取 [ **儲存**]。

變更大部分的應用程式設定都需要重新啟動服務。

### <a name="howto-use-sqlazure"></a>使用 SQL Database 作為您的生產資料存放區

<!--- ALTERNATE INCLUDE - we can't use ../includes/app-service-mobile-dotnet-backend-create-new-service.md - slightly different semantics -->

無論是何種 Azure App Service 應用程式類型，以 SQL Database 作為資料存放區的程序都是相同的。 如果您尚未執行，請依照下列步驟建立 Mobile Apps 後端：

1. 登入 [Azure 入口網站]。
1. 在視窗左上方，依序選取 [+新增] 按鈕 > [Web + 行動] > [行動應用程式]，然後為您的 Mobile Apps 後端命名。
1. 在 [資源群組]  方塊中，輸入與您應用程式相同的名稱。
1. 系統將會選取預設 App Service 方案。 如果您要變更 App Service 方案：

   a. 選取 [App Service 方案] > [+新建]。

   b. 為新的應用程式服務方案命名，並選取適當位置。

   c. 為服務選取適當的定價層。 選取 [檢視全部] 以檢視其他價格選項，例如 [免費] 和 [共用]。

   d. 按一下 [選取] 按鈕。

   e. 回到 [App Service 方案] 窗格，然後選取 [確定]。
1. 選取 [建立] 。

佈建 Mobile Apps 後端可能需要幾分鐘的時間。 Mobile Apps 後端佈建完成後，入口網站會開啟 Mobile Apps 後端的 [設定] 窗格。

您可以選擇將現有的 SQL Database 連線至 Mobile Apps 後端，或建立新的 SQL 資料庫。 在這一節中，您將建立 SQL Database。

> [!NOTE]
> 如果您已有與 Mobile Apps 後端位於相同的位置的資料庫，您可以改為選取 [使用現有的資料庫]，然後選取該資料庫。 我們不建議您使用位在不同位置的資料庫，因為這會產生更高的延遲。

1. 在新的 Mobile Apps 後端中，依序選取 [設定] > [行動應用程式] > [資料] > [+新增]。
1. 在 [新增資料連線] 窗格中，依序選取 [SQL Database - 設定必要設定] > [建立新的資料庫]。 在 [名稱] 方塊中輸入新資料庫的名稱。
1. 選取 [伺服器]。 在 [新增伺服器] 窗格中，於 [伺服器名稱] 方塊中輸入唯一的伺服器名稱，然後提供合適的伺服器管理登入和密碼。 確定已選取 [允許 Azure 服務存取伺服器]。 選取 [確定] 。

   ![建立 Azure SQL 資料庫][6]
1. 在 [新增資料庫] 窗格中，選取 [確定]。
1. 返回 [新增資料連線] 窗格中，選取 [連接字串]，然後輸入您建立資料庫時提供的登入與密碼。 如果您使用現有的資料庫，請提供該資料庫的登入認證。 選取 [確定] 。
1. 再次返回 [新增資料連接] 窗格中，選取 [確定] 以建立資料庫。

<!--- END OF ALTERNATE INCLUDE -->

建立資料庫可能需要幾分鐘的時間。 使用 [通知]  區域來監視部署的進度。 在資料庫成功部署之前，請勿繼續進行。 資料庫部署完成後，將會在您 Mobile Apps 的應用程式設定中建立 SQL Database 執行個體的連接字串。 您可以在 [設定] > [應用程式設定] > [連接字串]中檢視此應用程式設定。

### <a name="howto-tables-auth"></a>需經過驗證才能存取資料表

如果您想要在 `tables` 端點使用 App Service 驗證，就必須先在 [Azure 入口網站]中設定 App Service 驗證。 如需詳細資訊，請參閱您要使用之身分識別提供者的設定指南：

* [設定 Azure Active Directory 驗證]
* [設定 Facebook 驗證]
* [設定 Google 驗證]
* [設定 Microsoft 驗證]
* [設定 Twitter 驗證]

每個資料表都有一個存取屬性可用來控制對資料表的存取。 下列範例說明以靜態方式定義且需要驗證的資料表。

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Define the columns within the table.
table.columns = {
    "text": "string",
    "complete": "boolean"
};

// Turn off the dynamic schema.
table.dynamicSchema = false;

// Require authentication to access the table.
table.access = 'authenticated';

module.exports = table;
```

存取屬性可採用三種值：

* *anonymous* 表示允許用戶端應用程式未經驗證就可讀取資料。
* *authenticated* 表示用戶端應用程式必須隨要求傳送有效的驗證權杖。
* *disabled* 表示此資料表目前已停用。

如果未定義存取屬性，則會允許未經驗證的存取。

### <a name="howto-tables-getidentity"></a>透過資料表使用驗證宣告
您可以設定驗證設定時所要求的各種宣告。 這些宣告通常無法透過 `context.user` 物件取得。 不過，您可以使用 `context.user.getIdentity()` 方法加以擷取。 `getIdentity()` 方法會傳回可解析為物件的 Promise。 此物件會以驗證方法建立索引 (`facebook`、`google`、`twitter`、`microsoftaccount` 或 `aad`)。

例如，如果您設定 Microsoft 帳戶驗證並要求電子郵件地址宣告，您可以利用下列資料表控制器將電子郵件地址新增至記錄：

```javascript
var azureMobileApps = require('azure-mobile-apps');

// Create a new table definition.
var table = azureMobileApps.table();

table.columns = {
    "emailAddress": "string",
    "text": "string",
    "complete": "boolean"
};
table.dynamicSchema = false;
table.access = 'authenticated';

/**
* Limit the context query to those records with the authenticated user email address
* @param {Context} context the operation context
* @returns {Promise} context execution Promise
*/
function queryContextForEmail(context) {
    return context.user.getIdentity().then((data) => {
        context.query.where({ emailAddress: data.microsoftaccount.claims.emailaddress });
        return context.execute();
    });
}

/**
* Adds the email address from the claims to the context item - used for
* insert operations
* @param {Context} context the operation context
* @returns {Promise} context execution Promise
*/
function addEmailToContext(context) {
    return context.user.getIdentity().then((data) => {
        context.item.emailAddress = data.microsoftaccount.claims.emailaddress;
        return context.execute();
    });
}

// Configure specific code when the client does a request.
// READ: only return records that belong to the authenticated user.
table.read(queryContextForEmail);

// CREATE: add or overwrite the userId based on the authenticated user.
table.insert(addEmailToContext);

// UPDATE: only allow updating of records that belong to the authenticated user.
table.update(queryContextForEmail);

// DELETE: only allow deletion of records that belong to the authenticated user.
table.delete(queryContextForEmail);

module.exports = table;
```

若要查看哪些宣告可用，請使用網頁瀏覽器來檢視網站的 `/.auth/me` 端點。

### <a name="howto-tables-disabled"></a>停用對特定資料表作業的存取

除了會出現在資料表上以外，存取屬性也可用來控制個別作業。 共有四種作業：

* `read` 是對資料表執行的 RESTful GET 作業。
* `insert` 是對資料表執行的 RESTful POST 作業。
* `update` 是對資料表執行的 RESTful PATCH 作業。
* `delete` 是對資料表執行的 RESTful DELETE 作業。

比方說，您可能會想要提供未經驗證的唯讀資料表：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Read-only table. Only allow READ operations.
table.read.access = 'anonymous';
table.insert.access = 'disabled';
table.update.access = 'disabled';
table.delete.access = 'disabled';

module.exports = table;
```

### <a name="howto-tables-query"></a>調整與資料表作業搭配使用的查詢

資料表作業的常見需求是提供受限制的資料檢視。 例如，您可能會提供標有已驗證之使用者 ID 的資料表，因此您只能讀取或更新自己的記錄。 下列資料表定義提供這項功能：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Define a static schema for the table.
table.columns = {
    "userId": "string",
    "text": "string",
    "complete": "boolean"
};
table.dynamicSchema = false;

// Require authentication for this table.
table.access = 'authenticated';

// Ensure that only records for the authenticated user are retrieved.
table.read(function (context) {
    context.query.where({ userId: context.user.id });
    return context.execute();
});

// When adding records, add or overwrite the userId with the authenticated user.
table.insert(function (context) {
    context.item.userId = context.user.id;
    return context.execute();
});

module.exports = table;
```

正常執行的查詢作業，擁有供您使用 `where` 子句來調整的查詢屬性。 查詢屬性是一種 [QueryJS] 物件，可用來將 OData 查詢轉換成資料後端可以處理的項目。 在簡單的等號比較案例中 (如同上例)，您可以使用對應。 您也可以新增特定的 SQL 子句︰

```javascript
context.query.where('myfield eq ?', 'value');
```

### <a name="howto-tables-softdelete"></a>設定資料表上的虛刪除

虛刪除並不會實際刪除記錄。 它會將已刪除的資料行設定為 true，而將記錄標示為已在資料庫中刪除。 Mobile Apps SDK 會自動從結果中移除已虛刪除的記錄，除非 Mobile Client SDK 使用 `IncludeDeleted()`。 若要為資料表設定虛刪除，請在資料表定義檔中設定 `softDelete` 屬性：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Define the columns within the table.
table.columns = {
    "text": "string",
    "complete": "boolean"
};

// Turn off the dynamic schema.
table.dynamicSchema = false;

// Turn on soft delete.
table.softDelete = true;

// Require authentication to access the table.
table.access = 'authenticated';

module.exports = table;
```

您應建立刪除記錄的機制：用戶端應用程式、WebJob、Azure Function 或自訂 API。

### <a name="howto-tables-seeding"></a>在您的資料庫中植入資料

在建立新的應用程式時，您可能會想要在資料表中植入資料。 您可以在資料表定義 JavaScript 檔案中完成此作業，如下所示：

```javascript
var azureMobileApps = require('azure-mobile-apps');

var table = azureMobileApps.table();

// Define the columns within the table.
table.columns = {
    "text": "string",
    "complete": "boolean"
};
table.seed = [
    { text: 'Example 1', complete: false },
    { text: 'Example 2', complete: true }
];

// Turn off the dynamic schema.
table.dynamicSchema = false;

// Require authentication to access the table.
table.access = 'authenticated';

module.exports = table;
```

只有在您已使用 Mobile Apps SDK 建立資料表時，才會發生資料植入。 如果資料表已存在於資料庫中，則不會在資料表中插入任何資料。 如果開啟動態結構描述，則會從植入的資料推斷結構描述。

建議您明確呼叫 `tables.initialize()` 方法，以在服務開始執行時建立資料表。

### <a name="Swagger"></a>啟用 Swagger 支援
Mobile Apps 隨附內建的 [Swagger] 支援。 若要啟用 Swagger 支援，請先安裝 swagger-ui 作為相依性：

    npm install --save swagger-ui

接著，您可以在 Mobile Apps 建構函式中啟用 Swagger 支援：

```javascript
var mobile = azureMobileApps({ swagger: true });
```

您可能只想要在開發版本中啟用 Swagger 支援。 您可以使用 `NODE_ENV` 應用程式設定來執行這項工作：

```javascript
var mobile = azureMobileApps({ swagger: process.env.NODE_ENV !== 'production' });
```

`swagger` 端點位於 http://*yoursite*.azurewebsites.net/swagger。 您可以透過 `/swagger/ui` 端點存取 Swagger UI。 如果您選擇需要在整個應用程式進行驗證，Swagger 會產生錯誤。 為了獲得最佳結果，請在「Azure App Service 驗證/授權」設定中選擇允許未經驗證的要求通過，然後使用 `table.access` 屬性來控制驗證。

如果您想要只在進行本機開發時才使用 Swagger 支援，您也可以將 Swagger 選項新增至您的 azureMobile.js 檔案。

## <a name="a-namepushpush-notifications"></a><a name="push"/>推播通知

Mobile Apps 會與 Azure 通知中樞整合，讓您能跨所有主要平台，將目標推播通知傳送給數百萬個裝置。 藉由使用通知中樞，您可以將推播通知傳送至 iOS、Android 和 Windows 裝置。 若要深入了解您可以使用通知中樞執行的所有功能，請參閱 [通知中樞概觀](../notification-hubs/notification-hubs-push-notification-overview.md)。

### <a name="send-push"></a>傳送推播通知

下列程式碼示範如何使用 `push` 物件來傳送廣播推播通知給已註冊的 iOS 裝置︰

```javascript
// Create an APNS payload.
var payload = '{"aps": {"alert": "This is an APNS payload."}}';

// Only do the push if configured.
if (context.push) {
    // Send a push notification by using APNS.
    context.push.apns.send(null, payload, function (error) {
        if (error) {
            // Do something or log the error.
        }
    });
}
```

藉由從用戶端建立範本的推播註冊，您可以改為傳送範本推播訊息至所有支援平台上的裝置。 下列程式碼示範如何傳送範本通知︰

```javascript
// Define the template payload.
var payload = '{"messageParam": "This is a template payload."}';

// Only do the push if configured.
if (context.push) {
    // Send a template notification.
    context.push.send(null, payload, function (error) {
        if (error) {
            // Do something or log the error.
        }
    });
}
```

### <a name="push-user"></a>使用標記將推播通知傳送給已驗證的使用者
當驗證的使用者註冊推播通知之後，使用者識別碼便會自動加入到註冊中。 藉由使用這個標記，您可以傳送推播通知給特定使用者已註冊的所有裝置。 下列程式碼會取得提出要求之使用者的 SID，並將範本推播通知傳送至該使用者的每個裝置註冊︰

```javascript
// Only do the push if configured.
if (context.push) {
    // Send a notification to the current user.
    context.push.send(context.user.id, payload, function (error) {
        if (error) {
            // Do something or log the error.
        }
    });
}
```

在註冊來自已驗證用戶端的推播通知時，請確定驗證已完成，然後再嘗試註冊。

## <a name="CustomAPI"></a> 自訂 API

### <a name="howto-customapi-basic"></a>定義自訂 API

除了透過 `/tables` 端點的資料存取 API 以外，Mobile Apps 也可提供自訂 API 涵蓋範圍。 自訂 API 會以類似於資料表定義的方法定義，並且可存取所有相同功能，包括驗證。

如果您想要在自訂 API 使用 App Service 驗證，就必須先在 [Azure 入口網站]中設定 App Service 驗證。 如需詳細資訊，請參閱您要使用之身分識別提供者的設定指南：

* [設定 Azure Active Directory 驗證]
* [設定 Facebook 驗證]
* [設定 Google 驗證]
* [設定 Microsoft 驗證]
* [設定 Twitter 驗證]

定義自訂 API 的方法與資料表 API 很類似：

1. 建立 `api` 目錄。
1. 在 `api` 目錄中建立 API 定義 JavaScript 檔案。
1. 使用匯入方法將 `api` 目錄匯入。

以下是根據我們先前使用的基本應用程式範例所做的原型 API 定義：

```javascript
var express = require('express'),
    azureMobileApps = require('azure-mobile-apps');

var app = express(),
    mobile = azureMobileApps();

// Import the custom API.
mobile.api.import('./api');

// Add the Mobile API so it is accessible as a Web API.
app.use(mobile);

// Start listening on HTTP
app.listen(process.env.PORT || 3000);
```

我們將使用一個會透過 `Date.now()` 方法傳回伺服器日期的範例 API。 以下是 api/date.js 檔案：

```javascript
var api = {
    get: function (req, res, next) {
        var date = { currentTime: Date.now() };
        res.status(200).type('application/json').send(date);
    });
};

module.exports = api;
```

每個參數都是標準 RESTful 動詞之一：GET、POST、PATCH 或 DELETE。 此方法是會傳送必要輸出的標準 [ExpressJS 中介軟體] 函式。

### <a name="howto-customapi-auth"></a>需經過驗證才能存取自訂 API

Mobile Apps SDK 對於 `tables` 端點和自訂 API 會使用相同的方式實作驗證。 若要在前一節開發的 API 中新增驗證，請新增 `access` 屬性：

```javascript
var api = {
    get: function (req, res, next) {
        var date = { currentTime: Date.now() };
        res.status(200).type('application/json').send(date);
    });
};
// All methods must be authenticated.
api.access = 'authenticated';

module.exports = api;
```

您也可以指定特定作業的驗證：

```javascript
var api = {
    get: function (req, res, next) {
        var date = { currentTime: Date.now() };
        res.status(200).type('application/json').send(date);
    }
};
// The GET methods must be authenticated.
api.get.access = 'authenticated';

module.exports = api;
```

對於需要驗證的自訂 API，必須使用 `tables` 端點所使用的相同權杖。

### <a name="howto-customapi-auth"></a>處理大型檔案上傳

Mobile Apps SDK 會使用[本文剖析器中介軟體](https://github.com/expressjs/body-parser)來接受您提交的本文內容並加以解碼。 您可以預先設定讓本文剖析器接受較大的檔案上傳：

```javascript
var express = require('express'),
    bodyParser = require('body-parser'),
    azureMobileApps = require('azure-mobile-apps');

var app = express(),
    mobile = azureMobileApps();

// Set up large body content handling.
app.use(bodyParser.json({ limit: '50mb' }));
app.use(bodyParser.urlencoded({ limit: '50mb', extended: true }));

// Import the custom API.
mobile.api.import('./api');

// Add the Mobile API so it is accessible as a Web API.
app.use(mobile);

// Start listening on HTTP.
app.listen(process.env.PORT || 3000);
```

檔案會在傳輸之前經過 base-64 編碼。 此編碼會增加實際上傳內容的大小 (也是您必須考量的大小)。

### <a name="howto-customapi-sql"></a>執行自訂 SQL 陳述式

Mobile Apps SDK 可讓您透過要求物件存取完整內容。 您可以輕鬆地對已定義的資料提供者執行參數化 SQL 陳述式：

```javascript
var api = {
    get: function (request, response, next) {
        // Check for parameters. If not there, pass on to a later API call.
        if (typeof request.params.completed === 'undefined')
            return next();

        // Define the query. Anything that the mssql
        // driver can handle is allowed.
        var query = {
            sql: 'UPDATE TodoItem SET complete=@completed',
            parameters: [{
                completed: request.params.completed
            }]
        };

        // Execute the query. The context for Mobile Apps is available through
        // request.azureMobile. The data object contains the configured data provider.
        request.azureMobile.data.execute(query)
        .then(function (results) {
            response.json(results);
        });
    }
};

api.get.access = 'authenticated';
module.exports = api;
```

## <a name="Debugging"></a>偵錯、簡單資料表及簡單的 API

### <a name="howto-diagnostic-logs"></a>為 Mobile Apps 偵錯、診斷和進行疑難排解

Azure App Service 提供數個適用於 Node.js 應用程式的偵錯和疑難排解技術。
若要開始對您的 Node.js Mobile Apps 後端進行疑難排解，請參閱下列文章︰

* [監視 Azure App Service]
* [在 Azure App Service 中啟用診斷記錄]
* [在 Visual Studio 中進行 Azure App Service 的疑難排解]

Node.js 應用程式可存取多種不同的診斷記錄工具。 在內部，Mobile Apps Node.js SDK 會使用 [Winston] 進行診斷記錄。 當您啟用偵錯模式，或是在 [Azure 入口網站]中將 `MS_DebugMode` 應用程式設定設為 true 時，即會自動啟用記錄功能。 產生的記錄會顯示在 [Azure 入口網站]的診斷記錄中。

### <a name="in-portal-editing"></a><a name="work-easy-tables"></a>在 Azure 入口網站中使用簡單資料表

您可以直接在入口網站中使用簡單資料表來建立及使用資料表。 您可以將資料集以 CSV 格式上傳至簡單資料表。 請注意，您無法使用與 Mobile Apps 後端的系統屬性名稱衝突的屬性名稱 (在您的 CSV 資料集內)。 系統屬性名稱為：
* 建立時間
* 更新時間
* 已刪除
* version

您甚至可以使用 App Service 編輯器來編輯資料表作業。 當您選取後端網站設定中的 [簡單資料表] 時，您可以新增、修改或刪除資料表。 您也可以查看資料表中的資料。

![使用簡單資料表](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-easy-tables.png)

下列命令可用於資料表的命令列：

* **變更權限**：修改在資料表上讀取、插入、更新和刪除作業的權限。
 選項包括允許匿名存取、要求驗證，或停用作業的所有存取。
* **編輯指令碼**：資料表的指令碼檔案會在 App Service 編輯器中開啟。
* **管理結構描述**：新增或刪除資料行，或變更資料表索引。
* **清除資料表**：截斷現有的資料表可能會刪除所有資料列，但結構描述會維持不變。
* **刪除資料列**：刪除個別的資料列。
* **檢視串流記錄**：將您連線至您的網站的串流記錄服務。

### <a name="work-easy-apis"></a>在 Azure 入口網站中使用簡單 API

您可以直接在入口網站中使用簡單 API 來建立及使用自訂 API。 您可以使用 App Service 編輯器來編輯 API 指令碼。

當您選取後端網站設定中的 [簡單資料表] 時，您可以新增、修改或刪除自訂 API 端點。

![使用簡單 API](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-easy-apis.png)

在入口網站中，您可以變更 HTTP 動作的存取權限、在 App Service 編輯器中編輯 API 指令碼檔案，或檢視串流記錄。

### <a name="online-editor"></a>在 App Service 編輯器中編輯程式碼

使用 Azure 入口網站，可讓您在 App Service 編輯器中編輯 Node.js 後端指令碼檔案，而不需將專案下載到本機電腦。 若要在線上編輯器中編輯指令碼檔案：

1. 在您 Mobile Apps 後端的窗格中，選取 [所有設定] > [簡單資料表] 或 [簡單 API]。 選取資料表或 API，然後選取 [編輯指令碼]。 指令碼檔案會在 App Service 編輯器中開啟。

   ![App Service 編輯器](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-visual-studio-editor.png)
1. 在線上編輯器中變更程式碼檔案。 當您輸入資料時，會自動儲存變更。

<!-- Images -->
[0]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/npm-init.png
[1]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/vs2015-new-project.png
[2]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/vs2015-install-npm.png
[3]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-config.png
[4]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-authconfig.png
[5]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-newuser-1.png
[6]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/dotnet-backend-create-db.png

<!-- URLs -->
[Android 用戶端快速入門]: app-service-mobile-android-get-started.md
[Apache Cordova 用戶端快速入門]: app-service-mobile-cordova-get-started.md
[iOS 用戶端快速入門]: app-service-mobile-ios-get-started.md
[Xamarin.iOS 用戶端快速入門]: app-service-mobile-xamarin-ios-get-started.md
[Xamarin.Android 用戶端快速入門]: app-service-mobile-xamarin-android-get-started.md
[Xamarin.Forms 用戶端快速入門]: app-service-mobile-xamarin-forms-get-started.md
[Windows 市集用戶端快速入門]: app-service-mobile-windows-store-dotnet-get-started.md
[離線資料同步]: app-service-mobile-offline-data-sync.md
[設定 Azure Active Directory 驗證]: ../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md
[設定 Facebook 驗證]: ../app-service/app-service-mobile-how-to-configure-facebook-authentication.md
[設定 Google 驗證]: ../app-service/app-service-mobile-how-to-configure-google-authentication.md
[設定 Microsoft 驗證]: ../app-service/app-service-mobile-how-to-configure-microsoft-authentication.md
[設定 Twitter 驗證]: ../app-service/app-service-mobile-how-to-configure-twitter-authentication.md
[Azure App Service 部署指南]: ../app-service/app-service-deploy-local-git.md
[監視 Azure App Service]: ../app-service/web-sites-monitor.md
[在 Azure App Service 中啟用診斷記錄]: ../app-service/web-sites-enable-diagnostic-log.md
[在 Visual Studio 中進行 Azure App Service 的疑難排解]: ../app-service/web-sites-dotnet-troubleshoot-visual-studio.md
[指定 Node 版本]: ../nodejs-specify-node-version-azure-apps.md
[使用 Node 模組]: ../nodejs-use-node-modules-azure-apps.md
[Create a new Azure App Service]: ../app-service/
[azure-mobile-apps]: https://www.npmjs.com/package/azure-mobile-apps
[Express]: http://expressjs.com/
[Swagger]: http://swagger.io/

[Azure 入口網站]: https://portal.azure.com/
[OData]: http://www.odata.org
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[GitHub 上的 basicapp 範例]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/basic-app
[GitHub 上的待辦事項範例]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/todo
[GitHub 上的範例目錄]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples
[static-schema sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/static-schema
[QueryJS]: https://github.com/Azure/queryjs
[Node.js Tools 1.1 for Visual Studio]: https://github.com/Microsoft/nodejstools/releases/tag/v1.1-RC.2.1
[mssql Node.js 套件]: https://www.npmjs.com/package/mssql
[Microsoft SQL Server 2014 Express]: http://www.microsoft.com/en-us/server-cloud/Products/sql-server-editions/sql-server-express.aspx
[ExpressJS 中介軟體]: http://expressjs.com/guide/using-middleware.html
[Winston]: https://github.com/winstonjs/winston
