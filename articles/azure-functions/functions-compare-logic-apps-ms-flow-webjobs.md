---
title: 比較 Flow、Logic Apps、Functions 和 WebJob - Azure
description: 比較已針對整合工作最佳化的 Microsoft 雲端服務：Flow、Logic Apps、Functions 和 WebJob。
services: functions, logic-apps
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: microsoft flow, 流程, logic apps, azure functions, 函數, azure webjobs, webjobs, 事件處理, 動態計算, 無伺服器架構
ms.service: azure-functions
ms.devlang: multiple
ms.topic: overview
ms.date: 04/09/2018
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: e449703dd8d012037af9540397ecd3b7bc0c7e7f
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114630"
---
# <a name="compare-flow-logic-apps-functions-and-webjobs"></a>比較 Flow、Logic Apps、Functions 和 WebJob

本文會比較下列 Microsoft 雲端服務：

* [Microsoft Flow](https://flow.microsoft.com/)
* [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)
* [Azure Functions](https://azure.microsoft.com/services/functions/)
* [Azure App Service WebJobs](../app-service/web-sites-create-web-jobs.md)

所有這些服務都可以解決整合問題，以及自動化商業流程。 它們全都可以定義輸入、動作、條件和輸出。 您可以在排程或觸發程序上執行上述各項服務。 不過，各服務都有獨特的優點，本文將說明這些差異。

## <a name="compare-microsoft-flow-and-azure-logic-apps"></a>比較 Microsoft Flow 和 Azure Logic Apps

Flow 和 Logic Apps 都是可建立工作流程的「Designer First (設計工具優先)」整合服務。 這兩個服務皆可與各種 SaaS 和企業應用程式整合。 

Flow 是以 Logic Apps 為基礎所建置。 它們共用相同的工作流程設計工具和相同的[連接器](../connectors/apis-list.md)。 

Flow 可讓任何辦公室工作人員有能力執行簡單的整合 (例如，SharePoint 文件庫中的核准程序)，而不必透過開發人員或 IT。 另一方面，Logic Apps 則可以實現需要企業級 DevOps 和安全性做法的進階整合 (例如 B2B 處理程序)。 一般來說，商務工作流程會隨著時間而趨於複雜。 因此，一開始您可以先從流程著手，然後再視需要將它轉換為邏輯應用程式。

下表可協助您判斷最適合所給定整合的是 Flow 還是 Logic Apps。

|  | Flow | Logic Apps |
| --- | --- | --- |
| 使用者 |辦公室員工、商務使用者、SharePoint 系統管理員 |專業的整合人員和開發人員，IT 專業人員 |
| 案例 |自助服務 |進階整合 |
| 設計工具 |瀏覽器內及行動裝置應用程式，僅限 UI |有瀏覽器內和 [Visual Studio](../logic-apps/logic-apps-deploy-from-vs.md)、[程式碼檢視](../logic-apps/logic-apps-author-definitions.md)可用 |
| 應用程式生命週期管理 (ALM) |在非生產環境中設計及測試，在就緒時升級到生產環境。 |DevOps：在 [Azure 資源管理](../logic-apps/logic-apps-create-deploy-azure-resource-manager-templates.md)中的原始檔控制、測試支援、自動化及管理性 |
| 管理員體驗 |管理流量環境和資料外洩防護 (DLP) 原則，追蹤授權 [https://admin.flow.microsoft.com](https://admin.flow.microsoft.com) |管理資源群組、連線、存取管理和記錄 [https://portal.azure.com](https://portal.azure.com) |
| 安全性 |Office 365 安全性與相容性稽核記錄、資料外洩防護 (DLP)、敏感性資料[靜止時加密](https://wikipedia.org/wiki/Data_at_rest#Encryption)等等。 |Azure 的安全性保證︰[Azure 安全性](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity)、[資訊安全中心](https://azure.microsoft.com/services/security-center/)、[稽核記錄檔](https://azure.microsoft.com/blog/azure-audit-logs-ux-refresh/)等。 |

## <a name="compare-azure-functions-and-azure-logic-apps"></a>比較 Azure Functions 和 Azure Logic Apps

Functions 和 Logic Apps 都是可啟用無伺服器工作負載的 Azure 服務。 Azure Functions 是無伺服器計算服務，而 Azure Logic Apps 可提供無伺服器工作流程。 兩者皆可建立複雜的「協調流程」。 協調流程是函式或步驟的集合，在 Logic Apps 中稱為「動作」，您可執行其來完成複雜工作。 例如處理命令批次，您可以平行執行許多函式執行個體、等待執行個體完成，然後執行函式來計算彙總結果。

對於 Azure Functions，您可以透過撰寫程式碼和使用 [Durable Functions 擴充功能](durable-functions-overview.md)來開發協調流程。 對於 Logic apps，您可以使用 GUI 或編輯組態檔來建立協調流程。

您可以在建置協調流程時混合搭配服務，從邏輯應用程式呼叫函式和從函式呼叫邏輯應用程式。 選擇如何建置每個協調流程，取決於服務的功能或您的個人喜好。 下表列出這些服務之間的一些主要差異：
 
|  | 長期函式 | Logic Apps |
| --- | --- | --- |
| 開發 | Code First (命令式) | Designer First (宣告式) |
| 連線能力 | [約有十幾個內建繫結類型](functions-triggers-bindings.md#supported-bindings)、撰寫自訂繫結的程式碼 | [連接器的大型集合](../connectors/apis-list.md)、[適用於 B2B 的企業整合套件案例](../logic-apps/logic-apps-enterprise-integration-overview.md)、[建置自訂連接器](../logic-apps/custom-connector-overview.md) |
| 動作 | 每個活動都是 Azure 函式；撰寫活動函式的程式碼 |[現成動作的大型集合](../logic-apps/logic-apps-workflow-actions-triggers.md)|
| 監視 | [Azure Application Insights](../application-insights/app-insights-overview.md) | [Azure 入口網站](../logic-apps/quickstart-create-first-logic-app-workflow.md)，[Log Analytics](../logic-apps/logic-apps-monitor-your-logic-apps.md)|
| 管理性 | [REST API](durable-functions-http-api.md)、[Visual Studio](https://docs.microsoft.com/azure/vs-azure-tools-resources-managing-with-cloud-explorer) | [Azure 入口網站](../logic-apps/quickstart-create-first-logic-app-workflow.md)、[REST API](https://docs.microsoft.com/rest/api/logic/)[PowerShell](https://docs.microsoft.com/powershell/module/azurerm.logicapp/?view=azurermps-5.6.0)[Visual Studio](https://docs.microsoft.com/azure/logic-apps/manage-logic-apps-with-visual-studio) |
| 執行內容 | 可以在[本機](functions-runtime-overview.md)或雲端中執行。 | 只可在雲端中執行。|

<a name="function"></a>

## <a name="compare-functions-and-webjobs"></a>比較 Functions 和 Webjob

如同 Azure Functions，搭配 WebJobs SDK 的 Azure App Service WebJobs 是針對開發人員所設計的 Code First 整合服務。 兩者皆以 [Azure App Service](../app-service/app-service-web-overview.md) 為基礎，並支援[原始檔控制整合](../app-service/app-service-continuous-deployment.md)、[驗證](../app-service/app-service-authentication-overview.md)和[使用 Application Insights 整合進行監視](functions-monitoring.md)等功能。

### <a name="webjobs-and-the-webjobs-sdk"></a>WebJob 和 WebJob SDK

App Service 的 WebJobs 功能可讓您在 App Service Web 應用程式的環境中執行指令碼或程式碼。 WebJobs SDK 是為 WebJobs 設計的架構，可簡化您在回應 Azure 服務中的事件時所撰寫的程式碼。 例如，您可藉由建立縮圖影像來回應 Azure 儲存體中的映像 Blob 建立。 WebJobs SDK 以 .NET 主控台應用程式的形式執行，您可以將其部署至 WebJob。 

搭配使用 WebJobs 和 WebJobs SDK 的效果最好，但您也可以在沒有 WebJobs SDK 情況下使用 WebJobs，反之亦然。 WebJob 可執行 App Service 沙箱中執行的任何程式或指令碼。 WebJobs SDK 主控台應用程式可以在任何可執行主控台應用程式的地方執行，例如內部部署伺服器。

### <a name="comparison-table"></a>比較表

Azure Functions 是以 WebJobs SDK 為基礎，因此其共用許多相同的事件觸發程序及與其他 Azure 服務的連線。 以下是在 Azure Functions 和搭配 WebJobs SDK 的 WebJobs 之間做選擇時的一些考量因素：

|  | Functions | 搭配 WebJobs SDK 的 WebJobs |
| --- | --- | --- |
|[無伺服器應用程式模型](https://azure.microsoft.com/solutions/serverless/)與[自動調整](functions-scale.md#how-the-consumption-plan-works)|✔||
|[在瀏覽器中開發與測試](functions-create-first-azure-function.md) |✔||
|[按使用次數付費的定價方式](functions-scale.md#consumption-plan)|✔||
|[與 Logic Apps 整合](functions-twitter-email.md)|✔||
| 觸發程序事件 |[計時器](functions-bindings-timer.md)<br>[Azure 儲存體佇列和 Blob](functions-bindings-storage-blob.md)<br>[Azure 服務匯流排佇列和主題](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Azure 事件中樞](functions-bindings-event-hubs.md)<br>[HTTP/WebHook (GitHub、Slack)](functions-bindings-http-webhook.md)<br>[事件格線](functions-bindings-event-grid.md)|[計時器](functions-bindings-timer.md)<br>[Azure 儲存體佇列和 Blob](functions-bindings-storage-blob.md)<br>[Azure 服務匯流排佇列和主題](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Azure 事件中樞](functions-bindings-event-hubs.md)<br>[檔案系統](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Files/FileTriggerAttribute.cs)|
| 支援的語言  |C#<br>F#<br>JavaScript<br>Java (預覽) |C#<sup>1</sup>|
|套件管理員|NPM 和 NuGet|NuGet<sup>2</sup>|

<sup>1</sup> WebJobs (沒有 WebJobs SDK) 支援 C#、JavaScript、Bash、.cmd、.bat、PowerShell、PHP、TypeScript、Python 等其他項目。 沒有完整清單；WebJob 可執行 App Service 沙箱中執行的任何程式或指令碼。

<sup>2</sup> WebJobs (沒有 WebJobs SDK) 支援 NPM 和 NuGet。

### <a name="summary"></a>總結

Azure Functions 提供更高的開發人員生產力、更多的程式設計語言選項、更多的開發環境選項、更多的 Azure 服務整合選項，以及更多定價選項。 大部分的情況下，這是最佳選擇。

但在以下兩個案例中，WebJobs 可能是最好的選擇：

* 您需要更充分地掌控接聽事件的程式碼 (`JobHost` 物件)。 Functions 提供有限的方式來自訂 [host.json](functions-host-json.md) 檔案中的 `JobHost` 行為。 有時候，您需要執行的一些作業，無法由 JSON 檔案中的字串來指定。 例如，只有 WebJobs SDK 可讓您為 Azure 儲存體設定自訂的重試原則。
* 您有要為其執行程式碼片段的 App Service 應用程式，而且想要在相同的 DevOps 環境中一起管理。

針對其他需執行程式碼片段來整合 Azure 或第三方服務的情況，請選擇 Azure Functions，而不是搭配 WebJobs SDK 的 WebJobs。

<a name="together"></a>

## <a name="flow-logic-apps-functions-and-webjobs-together"></a>在 Flow、Logic Apps、Functions 和 WebJobs 之間做選擇

您不必只選擇其中一項服務；這些服務可彼此整合，如同與外部服務搭配使用時一樣。

流程可呼叫邏輯應用程式。 邏輯應用程式可呼叫函式，而函式可呼叫邏輯應用程式。 如需範例，可參閱[建立與 Azure Logic Apps 整合的函式](functions-twitter-email.md)。

Flow、Logic Apps 和 Functions 之間的整合會隨時間持續改進。 您可以在某項服務中建置某物並用於其他服務。

## <a name="next-steps"></a>後續步驟

開始建立您的第一個資料流程、邏輯應用程式或函式應用程式。 按一下下列任何連結︰

* [開始使用 Microsoft Flow](https://flow.microsoft.com/en-us/documentation/getting-started/)
* [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [建立您的第一個 Azure 函式](functions-create-first-azure-function.md)

或者，透過下列連結取得有關這些整合服務的詳細資訊︰

* [利用 Azure Functions 和 Azure App Service 來進行整合案例 - 主講人：Christopher Anderson](http://www.biztalk360.com/integrate-2016-resources/leveraging-azure-functions-azure-app-service-integration-scenarios/)
* [整合變得簡單，主講人：Charles Lamanna](http://www.biztalk360.com/integrate-2016-resources/integrations-made-simple/)
* [Logic Apps 即時網路廣播](http://aka.ms/logicappslive)
* [Microsoft Flow 常見問題集](https://flow.microsoft.com/documentation/frequently-asked-questions/)
