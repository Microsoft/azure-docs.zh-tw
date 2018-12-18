---
title: 使用 Azure Application Insights 監視 Node.js 服務 | Microsoft Docs
description: 使用 Application Insights 監視 Node.js 服務的效能和診斷問題。
services: application-insights
documentationcenter: nodejs
author: mrbullwinkle
manager: carmonm
ms.assetid: 2ec7f809-5e1a-41cf-9fcd-d0ed4bebd08c
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 05/01/2017
ms.author: mbullwin
ms.openlocfilehash: 28be3a1734639ac175e4d18d9e9f21b83b9a7e7c
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47093997"
---
# <a name="monitor-your-nodejs-services-and-apps-with-application-insights"></a>使用 Application Insights 監視 Node.js 服務和應用程式

[Azure Application Insights](app-insights-overview.md) 會在您部署後端服務和元件之後加以監視，協助您[探索並快速診斷效能和其他問題](app-insights-detect-triage-diagnose.md)。 您可以對在資料中心、在 Azure 虛擬機器和 Web 應用程式，和甚至其他公用雲端中託管的 Node.js 服務使用 Application Insights。

若要接收、儲存和探索您的監視資料，請在您的程式碼中包含 SDK，然後在 Azure 中設定對應的 Application Insights 資源。 SDK 會將資料傳送至該資源，進行進一步的分析和探索。

Node.js SDK 可以自動監視傳入和傳出 HTTP 要求、例外狀況、和一些系統計量。 從 0.20 版開始，SDK 也可以監視一些常見的第三方套件，像是 MongoDB、MySQL 和 Redis。 與傳入 HTTP 要求相關的所有事件都會相互關聯，以進行快速疑難排解。

您可以使用 TelemetryClient API 手動檢測和監視您的應用程式及系統的其他層面。 我們將在本文稍後更詳細說明 TelemetryClient API。

![範例效能監視圖表](./media/app-insights-nodejs/10-perf.png)

## <a name="get-started"></a>開始使用

完成下列工作來設定應用程式或服務的監視。

### <a name="prerequisites"></a>必要條件

開始之前，請確定您有 Azure 訂用帳戶或[免費取得一個新訂用帳戶][azure-free-offer]。 如果您的組織已經有 Azure 訂用帳戶，系統管理員可以依照 [這些指示][add-aad-user] 將您新增至該訂用帳戶。

[azure-free-offer]: https://azure.microsoft.com/free/
[add-aad-user]: https://docs.microsoft.com/azure/active-directory/active-directory-users-create-azure-portal


### <a name="resource"></a> 設定 Application Insights 資源


1. 登入 [Azure 入口網站][portal]。
2. 選取 [建立資源] > [開發人員工具] > [Application Insights]。 此資源包含用於接收遙測資料的端點、此資料的儲存體、已儲存的報告和儀表板、規則和警示組態等等。

  ![建立 Application Insights 資源](./media/app-insights-nodejs/03-new_appinsights_resource.png)

3. 在資源建立頁面上，於 [應用程式類型] 方塊中選取 [Node.js 應用程式]。 應用程式類型可決定建立的預設儀表板和報告。 (任何 Application Insights 資源都可以從任何語言和平台收集資料。)

  ![新增 Application Insights 資源表單](./media/app-insights-nodejs/04-create_appinsights_resource.png)

### <a name="sdk"></a> 設定 Node.js SDK

在您的應用程式中包含 SDK，以便蒐集資料。 

1. 從 Azure 入口網站複製資源的檢測金鑰 (也稱為 *ikey*)。 Application Insights 使用 ikey 來將資料對應至您的 Azure 資源。 在 SDK 可以使用您的 ikey 之前，您必須在環境變數或程式碼中指定 ikey。  

  ![複製檢測金鑰](./media/app-insights-nodejs/05-appinsights_ikey_portal.png)

2. 接著透過 package.json，將 Node.js SDK 程式庫新增至您應用程式的相依性。 從您應用程式的根資料夾，執行︰

  ```bash
  npm install applicationinsights --save
  ```

3. 在您的程式碼中明確地載入此程式庫。 因為 SDK 會將檢測插入其他許多程式庫中，請儘早載入程式庫，甚至插入在其他 `require` 陳述式之前。 

  在第一個 .js 檔案的頂端，加入下列程式碼。 `setup` 方法會針對所有追蹤的項目，設定預設要使用的金鑰 (以及 Azure 資源)。

  ```javascript
  const appInsights = require("applicationinsights");
  appInsights.setup("<instrumentation_key>");
  appInsights.start();
  ```
   
  您也可以透過環境變數 APPINSIGHTS\_INSTRUMENTATIONKEY 提供 ikey，而非以手動方式將它傳遞至 `setup()` 或 `new appInsights.TelemetryClient()`。 這種做法可讓您將 ikeys 保留在認可的原始程式碼之外，而您可以針對不同的環境指定不同的 ikey。

  如需其他組態選項，請參閱下列各節。

  藉由設定 `appInsights.defaultClient.config.disableAppInsights = true`，您可以嘗試 SDK，而不需要傳送遙測。

### <a name="monitor"></a> 監視您的應用程式

SDK 會自動蒐集有關 Node.js 執行階段和一些常見第三方模組的遙測。 使用您的應用程式來產生一些資料。

然後，在 [Azure 入口網站][portal]中，前往您稍早建立的 Application Insights 資源。 在**概觀時間表**中，尋找您的前幾個資料點。 若要查看更詳細的資料，請在圖表中選取不同的元件。

![第一個資料點](./media/app-insights-nodejs/12-first-perf.png)

若要檢視針對您的應用程式找到的拓撲，請選取 [應用程式對應] 按鈕。 在對應中選取元件來查看更多詳細資料。

![簡單的應用程式對應](./media/app-insights-nodejs/06-appinsights_appmap.png)

若要深入了解您的應用程式，以及疑難排解問題，請在 [調查] 區段中，選取可用的其他檢視。

![調查區段](./media/app-insights-nodejs/07-appinsights_investigate_blades.png)

#### <a name="no-data"></a>沒有資料？

因為 SDK 會分批提交資料，所以項目可能會延遲顯示在入口網站中。 如果未在您的資源中看到資料，請嘗試以下一些修正方式：

* 繼續使用應用程式。 採取更多動作以產生更多遙測。
* 按一下入口網站資源檢視中的 [重新整理]。 圖表會自行定期重新整理，但手動重新整理會強制圖表立即重新整理。
* 確認[所需的連出連接埠](app-insights-ip-addresses.md)已開啟。
* 使用 [搜尋](app-insights-diagnostic-search.md) 來尋找特定的事件。
* 查看[常見問題集][FAQ]。


## <a name="sdk-configuration"></a>SDK 組態

下列程式碼範例列出 SDK 的組態方法和預設值。

若要使服務中的事件完全相互關聯，請務必設定 `.setAutoDependencyCorrelation(true)`。 利用設定這個選項，可讓 SDK 追蹤 Node.js 中所有非同步回呼的內容。

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>")
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .start();
```

## <a name="telemetryclient-api"></a>TelemetryClient API

如需 TelemetryClient API 的完整說明，請參閱[自訂事件和度量的 Application Insights API](app-insights-api-custom-events-metrics.md)。

您可以使用 Application Insights Node.js SDK 來追蹤任何要求、事件、計量或例外狀況。 下列程式碼範例示範您可以使用的一些 API：

```javascript
let appInsights = require("applicationinsights");
appInsights.setup().start(); // assuming ikey is in env var
let client = appInsights.defaultClient;

client.trackEvent({name: "my custom event", properties: {customProperty: "custom property value"}});
client.trackException({exception: new Error("handled exceptions can be logged with this method")});
client.trackMetric({name: "custom metric", value: 3});
client.trackTrace({message: "trace message"});
client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL"});
client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});

let http = require("http");
http.createServer( (req, res) => {
  client.trackNodeHttpRequest({request: req, response: res}); // Place at the beginning of your request handler
});
```

### <a name="track-your-dependencies"></a>追蹤相依項目

您可以使用下列程式碼來追蹤相依項目：

```javascript
let appInsights = require("applicationinsights");
let client = appInsights.defaultClient;

var success = false;
let startTime = Date.now();
// Execute dependency call here...
let duration = Date.now() - startTime;
success = true;

client.trackDependency({dependencyTypeName: "dependency name", name: "command name", duration: duration, success: success});
```

### <a name="add-a-custom-property-to-all-events"></a>將自訂屬性新增至所有事件

您可以使用下列程式碼來將自訂屬性新增至所有事件：

```javascript
appInsights.defaultClient.commonProperties = {
    environment: process.env.SOME_ENV_VARIABLE
};
```

### <a name="track-http-get-requests"></a>追蹤 HTTP GET 要求

您可以使用下列程式碼來追蹤 HTTP GET 要求：

```javascript
var server = http.createServer((req, res) => {
    if ( req.method === "GET" ) {
            appInsights.defaultClient.trackNodeHttpRequest({request: req, response: res});
    }
    // Other work here...
    res.end();
});
```

### <a name="track-server-startup-time"></a>追蹤伺服器啟動時間

您可以使用下列程式碼來追蹤伺服器的啟動時間：

```javascript
let start = Date.now();
server.on("listening", () => {
    let duration = Date.now() - start;
    appInsights.defaultClient.trackMetric({name: "server startup time", value: duration});
});
```

## <a name="next-steps"></a>後續步驟

* [在入口網站中監視遙測](app-insights-dashboards.md)
* [寫您的遙測的分析查詢](app-insights-analytics-tour.md)

<!--references-->

[portal]: https://portal.azure.com/
[FAQ]: app-insights-troubleshoot-faq.md

