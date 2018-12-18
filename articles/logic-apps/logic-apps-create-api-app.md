---
title: 為 Azure Logic Apps 建立 Web API 和 REST API | Microsoft Docs
description: 建立 Web API 和 REST API 來呼叫您的 API、服務或系統以在 Azure Logic Apps 中進行系統整合
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, jehollan, LADocs
ms.topic: article
ms.assetid: bd229179-7199-4aab-bae0-1baf072c7659
ms.date: 05/26/2017
ms.openlocfilehash: a761e384a356a0cbf5531eee7340ddbbd1526909
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43122241"
---
# <a name="create-custom-apis-you-can-call-from-azure-logic-apps"></a>建立您可以從 Azure Logic Apps 呼叫的自訂 API

雖然 Azure Logic Apps 提供了 [100 多個內建連接器](../connectors/apis-list.md)，可供您在邏輯應用程式工作流程中使用，但建議您呼叫無法用來作為連接器的 API、系統與服務。 您可以建立自己的 API 來提供要在邏輯應用程式中使用的動作和觸發程序。 以下是您可能會想要建立自己的且可從邏輯應用程式工作流程呼叫之 API 的其他原因：

* 延伸您目前的系統整合及資料整合工作流程。
* 協助客戶使用您的服務來管理專業或個人工作。
* 展開您服務的觸達、搜尋功能以及用途。

基本上，連接器是使用 REST 作為隨插即用介面、[Swagger 中繼資料格式](http://swagger.io/specification/)作為文件，以及 JSON 作為其資料交換格式的 web API。 因為連接器是透過 HTTP 端點進行通訊的 REST API，您可以使用諸如 .NET、Java 或 Node.js 等任何語言來建置連接器。 您也可以在 [Azure App Service](../app-service/app-service-web-overview.md) 上裝載您的 API，Azure App Service 是平台即服務 (PaaS) 供應項目，能為 API 裝載提供最佳、最簡單且擴充性最高的方法。 

若要讓自訂 API 與 Logic Apps 搭配使用，您的 API 可以提供[*動作*](./logic-apps-overview.md#logic-app-concepts)，能在邏輯應用程式工作流程中執行特定的工作。 您的 API 也可作為[*觸發程序*](./logic-apps-overview.md#logic-app-concepts)，當新資料或事件符合指定的條件時，能啟動邏輯應用程式工作流程。 本主題描述常見的模式，以您想要 API 提供的行為作為基礎，加以遵循即可在您的 API 中建置動作和觸發程序。

您可以將 API 裝載在 [Azure App Service](../app-service/app-service-web-overview.md) 上，這是一個平台即服務 (PaaS) 供應項目，提供擴充性高且簡便的 API 裝載服務。

> [!TIP] 
> 雖然您可以將 API 部署成 Web 應用程式，但請考慮將您的 API 部署成 API 應用程式，如此一來，當您在雲端和內部部署環境中建置、裝載及取用 API 時，將可讓您的作業更輕鬆。 您不需要在 API 中變更任何程式碼 -- 只需將您的程式碼部署至 API 應用程式。 例如，了解如何使用下列語言來建置 API 應用程式： 
> 
> * [ASP.NET](../app-service/app-service-web-get-started-dotnet.md). 
> * [Java](../app-service/app-service-web-get-started-java.md)
> * [Node.js](../app-service/app-service-web-get-started-nodejs.md)
> * [PHP](../app-service/app-service-web-get-started-php.md)
> * [Python](../app-service/containers/quickstart-python.md)
> * [Ruby](../app-service/containers/quickstart-ruby.md)
>
> 如需針對 Logic Apps 建置的 API 應用程式範例，請瀏覽 [Azure Logic Apps GitHub 存放庫](http://github.com/logicappsio)或[部落格](http://aka.ms/logicappsblog)。

## <a name="how-do-custom-apis-differ-from-custom-connectors"></a>自訂 API 與自訂連接器有何不同？

自訂 API 和[自訂連接器](../logic-apps/custom-connector-overview.md)是針對可插式介面使用 REST、針對文件使用 [Swagger 中繼資料格式](http://swagger.io/specification/)以及使用 JSON 作為其資料交換格式的 Web API。 而由於這些 API 和連接器是透過 HTTP 端點進行通訊的 REST API，因此您可以使用任何語言 (例如 .NET、Java 或 Node.js) 來建置自訂 API 和連接器。

自訂 API 可讓您呼叫不是連接器的 API，並提供您可以使用 HTTP + Swagger、「Azure API 管理」或「應用程式服務」來呼叫的端點。 自訂連接器的運作方式與自訂 API 類似，但還具有下列屬性：

* 在 Azure 中已註冊為「Logic Apps 連接器」資源。
* 在「Logic Apps 設計工具」中會顯示在 Microsoft 受控連接器旁邊且帶有圖示。
* 只有連接器的作者，以及在部署邏輯應用程式的區域中具有相同 Azure Active Directory 租用戶和 Azure 訂用帳戶的使用者，才能使用。

您也可以提名已註冊的連接器來進行 Microsoft 認證。 此程序會確認已註冊的連接器符合公開使用準則，然後將這些連接器提供給 Microsoft Flow 和 Microsoft PowerApps 中的使用者使用。

如需有關自訂連接器的詳細資訊，請參閱 

* [自訂連接器概觀](../logic-apps/custom-connector-overview.md)
* [從 Web API 建立自訂連接器](../logic-apps/custom-connector-build-web-api-app-tutorial.md)
* [在 Azure Logic Apps 中註冊自訂連接器](../logic-apps/logic-apps-custom-connector-register.md)

## <a name="helpful-tools"></a>實用工具

當 API 也具有描述 API 作業和參數的 [Swagger 文件](http://swagger.io/specification/)時，自訂 API 最適合與 Logic Apps 搭配使用。
許多程式庫 (例如 [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle)) 可為您自動產生 Swagger 檔案。 若要標註 Swagger 檔案的顯示名稱、屬性類型等等，您也可以使用 [TRex](https://github.com/nihaue/TRex) ，讓您的 Swagger 檔案能適用於 Logic Apps。

<a name="actions"></a>

## <a name="action-patterns"></a>動作模式

針對執行工作的 Logic Apps，您的自訂 API 應提供[*動作*](./logic-apps-overview.md#logic-app-concepts)。 您 API 中的每項作業都會對應至動作。 基本的動作是可接受 HTTP 要求並傳回 HTTP 回應的控制器。 比方說，邏輯應用程式會將 HTTP 要求傳送至 web 應用程式或 API 應用程式。 接著，您的應用程式會傳回 HTTP 回應，以及邏輯應用程式可以處理的內容。

針對標準動作，您可以在 API 中撰寫 HTTP 要求方法，並在 Swagger 檔案中描述該方法。 接著，您可以使用 [HTTP 動作](../connectors/connectors-native-http.md)或 [HTTP + Swagger](../connectors/connectors-native-http-swagger.md) 動作直接呼叫您的 API。 根據預設，必須在[要求逾時限制](./logic-apps-limits-and-config.md)內將回應傳回。 

![標準動作模式](./media/logic-apps-create-api-app/standard-action.png)

<a name="pattern-overview"></a> 若要讓您的 API 在完成執行時間較長的工作時等候邏輯應用程式，您的 API 可以遵循本主題中所述的[非同步輪詢模式](#async-pattern)或[非同步 webhook 模式](#webhook-actions)。 如需比喻來協助您以視覺化方式檢視這些模式的不同行為，您可想像從麵包店訂購客製化蛋糕的程序。 輪詢模式所鏡像的行為，就如同您每隔 20 分鐘打電話給這家麵包店檢查蛋糕是否就緒。 Webhook 模式所鏡像的行為，就如同這家麵包店向您要求電話號碼，讓它們可以在蛋糕就緒時打電話給您。

如需範例，請瀏覽 [Logic Apps GitHub 存放庫](https://github.com/logicappsio)。 此外，深入了解[動作的使用量計量](logic-apps-pricing.md)。

<a name="async-pattern"></a>

### <a name="perform-long-running-tasks-with-the-polling-action-pattern"></a>以輪詢動作模式來執行長時間執行的工作

若要讓您 API 執行的工作可執行超過[要求逾時限制](./logic-apps-limits-and-config.md)的時間，您可以使用非同步輪詢模式。 此模式可讓您的 API 在個別的執行緒中執行工作，但保留作用中連線給 Logic Apps 引擎。 這樣一來，邏輯應用程式就不會逾時，或是在您的 API 完成工作之前，繼續工作流程中的下一個步驟。

以下是一般模式：

1. 請確定引擎知道您的 API 已接受要求並啟動工作。
2. 當引擎針對作業狀態提出後續要求時，請讓引擎知道您的 API 何時會完成工作。
3. 將相關資料傳回引擎，如此邏輯應用程式工作流程才能繼續。

<a name="bakery-polling-action"></a> 現在，將先前的麵包店比喻套用至輪詢模式，再想像您打電話給這家麵包店並訂購客製化蛋糕要求外送。 製作蛋糕的程序需要時間，而您不想要在麵包店製作蛋糕時等候電話。 這家麵包店確認您的訂單，並要求您每隔 20 分鐘打電話詢問蛋糕的狀態。 過了 20 分鐘之後，您打電話給麵包店，但它們告訴您蛋糕還沒完成，您要等 20 分鐘後再打電話過來。 這個往來的程序會持續直到您打電話，且這家麵包店告訴您訂單備妥且您的蛋糕送出為止。 

現在讓我們來對應回此輪詢模式。 這家麵包店就代表您的自訂 API，而身為蛋糕客戶的您則代表 Logic Apps 引擎。 當引擎呼叫您的 API 並提出要求時，您的 API 會確認要求，並在時間間隔內回應，讓引擎可以檢查作業狀態。 引擎會繼續檢查作業狀態，直到您的 API 回應作業已完成，並將資料傳回您的邏輯應用程式，然後會繼續進行工作流程。 

![輪詢動作模式](./media/logic-apps-create-api-app/custom-api-async-action-pattern.png)

以下可供您 API 遵循的特定步驟，是從 API 的觀點加以描述：

1. 當您的 API 開始進行 HTTP 要求時，立即傳回 HTTP `202 ACCEPTED` 回應，並包含 `location` 此步驟中稍後所述的標頭。 這個回應可讓 Logic Apps 引擎知道您的 API 已取得要求、接受了要求承載 (資料輸入)，並正在處理中。 
   
   `202 ACCEPTED` 回應應包含這些標頭：
   
   * 必要：`location` 標頭所指定的 URL 絕對路徑，可讓 Logic Apps 引擎在其中檢查您的 API 作業狀態

   * 選擇性：`retry-after` 標頭所指定的秒數，是引擎在檢查 `location` URL 以了解作業狀態時所需等候的時間。 

     根據預設，引擎每隔 20 秒會進行檢查。 若要指定不同的間隔，請包括 `retry-after` 標頭，以及下一次輪詢之前的秒數。

2. 過了指定的時間之後，Logic Apps 引擎會輪詢 `location` URL 來檢查作業狀態。 您的 API 應執行這些檢查，並傳回這些回應：
   
   * 如果作業已完成，就會傳回 HTTP `200 OK` 回應，以及回應承載 (下一個步驟的輸入)。

   * 如果作業仍在處理︰傳回另一個 HTTP `202 ACCEPTED` 回應，並以相同的標頭作為原始回應。

當您的 API 遵循此模式時，您不需要在邏輯應用程式工作流程定義中採取任何動作，即可繼續檢查作業狀態。 當引擎取得 HTTP `202 ACCEPTED` 回應及有效 `location` 標頭時，引擎會遵循非同步模式，並檢查 `location` 標頭，直到您的 API 傳回非 202 回應為止。

> [!TIP]
> 如需非同步模式範例，請檢閱此 [GitHub 中的非同步控制器回應範例](https://github.com/logicappsio/LogicAppsAsyncResponseSample)。

<a name="webhook-actions"></a>

### <a name="perform-long-running-tasks-with-the-webhook-action-pattern"></a>以 Webhook 動作模式來執行長時間執行的工作

或者，您可以使用 webhook 模式進行長時間執行的工作和非同步處理。 此模式具有邏輯應用程式暫停，並等候來自您 API 的「回呼」完成處理後，才會繼續工作流程。 此回呼是 HTTP POST，在事件發生時，會將訊息傳送到 URL。 

<a name="bakery-webhook-action"></a> 現在，將先前的麵包店比喻套用至 Webhook 模式，再想像您打電話給這家麵包店並訂購客製化蛋糕要求外送。 製作蛋糕的程序需要時間，而您不想要在麵包店製作蛋糕時等候電話。 這家麵包店確認了您的訂單，但這次，您向它們提供您的電話號碼，讓它們在蛋糕完成時可以打電話給您。 此時，這家麵包店告訴您訂單已就緒，並送出您的蛋糕。

當我們對應回此 webhook 模式時，這家麵包店就代表您的自訂 API，而身為蛋糕客戶的您則代表 Logic Apps 引擎。 引擎會使用要求呼叫您的 API，並包含「回呼」URL。
當作業完成時，您的 API 會使用此 URL 來通知引擎，並將資料傳回給您的邏輯應用程式，然後繼續進行工作流程。 

對於此模式，設定您控制器上的兩個端點：`subscribe` 和 `unsubscribe`

*  `subscribe` 端點：當執行在工作流程中觸達您 API 的動作時，Logic Apps 引擎會呼叫 `subscribe` 端點。 這個步驟會導致邏輯應用程式建立您的 API 所儲存之回呼 URL，然後在工作完成時，等候您 API 所發出的回呼。 接著，您的 API 會使用 HTTP POST 回呼 URL，並傳遞任何傳回的內容和標頭來作為邏輯應用程式的輸入。

* `unsubscribe` 端點：如果取消邏輯應用程式執行，Logic Apps 引擎就會呼叫 `unsubscribe` 端點。 接著，您的 API 就可以取消註冊回呼 URL，並視需要停止任何處理程序。

![Webhook 動作模式](./media/logic-apps-create-api-app/custom-api-webhook-action-pattern.png)

> [!NOTE]
> 目前，Logic App Designer 並不支援透過 Swagger 探索 webhook 端點。 因此對於此模式，您必須新增 [**Webhook** 動作](../connectors/connectors-native-webhook.md)並指定 URL、標頭，以及您的要求主體。 另請參閱[工作流程動作與觸發程序](logic-apps-workflow-actions-triggers.md#apiconnection-webhook-action)。 若要傳入回呼 URL，您可以視需要在任何一個先前的欄位中使用 `@listCallbackUrl()` 工作流程函式。

> [!TIP]
> 如需範例 webhook 模式，請檢閱此 [GitHub 中的 webhook 觸發程序範例](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs)。

<a name="triggers"></a>

## <a name="trigger-patterns"></a>觸發程序模式

您的自訂 API 可作為[*觸發程序*](./logic-apps-overview.md#logic-app-concepts)，當新資料或事件符合指定的條件時，能啟動邏輯應用程式。 這個觸發程序可以定期檢查，或是等候並接聽您服務端點上的新資料或事件。 如果新的資料或事件符合指定的條件，觸發程序就會引發，並啟動邏輯應用程式，用來接聽該觸發程序。 若要用這種方式啟動 Logic Apps，您的 API 可以遵循[*輪詢觸發程序*](#polling-triggers)或 [ *webhook 觸發程序*](#webhook-triggers)模式。 這些模式會與其[輪詢動作](#async-pattern)和 [webhook 動作](#webhook-actions)的對應類似。 此外，深入了解[觸發程序的使用量計量](logic-apps-pricing.md)。

<a name="polling-triggers"></a>

### <a name="check-for-new-data-or-events-regularly-with-the-polling-trigger-pattern"></a>使用輪詢觸發程序模式定期檢查新的資料或事件

輪詢觸發程序作用很像本主題先前所述的[輪詢動作](#async-pattern)。 Logic Apps 引擎會定期呼叫並檢查新的資料或事件，以查看觸發程序端點。 如果引擎發現符合您指定條件的新資料或事件，就會引發觸發程序。 然後，引擎會建立處理資料的邏輯應用程式執行個體作為輸入。 

![輪詢觸發程序](./media/logic-apps-create-api-app/custom-api-polling-trigger-pattern.png)

> [!NOTE]
> 每個輪詢要求都會計算為動作執行，即使未建立邏輯應用程式執行個體時亦然。 若要避免多次處理相同的資料，您的觸發程序應該將已讀取並傳遞至邏輯應用程式的資料清除。

以下適用於輪詢觸發程序的特定步驟，是從 API 的觀點加以描述：

| 找到新資料或事件了嗎？  | API 回應 | 
| ------------------------- | ------------ |
| 已找到 | 傳回包含回應承載的 HTTP `200 OK` 狀態 (下一個步驟的輸入)。 <br/>此回應會建立邏輯應用程式執行個體，並啟動工作流程。 | 
| 找不到 | 傳回包含 `location` 標頭和 `retry-after` 標頭的 HTTP`202 ACCEPTED` 狀態。 <br/>針對觸發程序，`location` 標頭也應該包含 `triggerState` 查詢參數，通常是「時間戳記」。 您的 API 可以利用這個識別碼，來追蹤邏輯應用程式所觸發的最後時間。 | 
||| 

例如，定期檢查您的服務是否有新檔案，您可能會建立具有這些行為的輪詢觸發程序：

| 要求是否包含 `triggerState`？ | API 回應 | 
| -------------------------------- | -------------| 
| 否 | 傳回 HTTP `202 ACCEPTED` 狀態與 `location` 標頭，並將 `triggerState` 設為目前的時間，且 `retry-after` 間隔設為 15 秒。 | 
| 是 | 請檢查您的服務是否有 `triggerState` 的 `DateTime` 之後新增的檔案。 | 
||| 

| 找到的檔案數 | API 回應 | 
| --------------------- | -------------| 
| 單一檔案 | 傳回 HTTP `200 OK` 狀態和內容承載、將傳回檔案的 `triggerState` 更新為 `DateTime`，並將 `retry-after` 間隔設定為 15 秒。 | 
| 多個檔案 | 一次傳回一個檔案與 HTTP `200 OK` 狀態、更新 `triggerState`，並將 `retry-after` 間隔設定為 0 秒。 </br>這些步驟會讓引擎知道有更多資料可用，以及引擎應立即從 `location` 標頭中的 URL 要求資料。 | 
| 沒有任何檔案 | 傳回 HTTP `202 ACCEPTED` 狀態、不要變更 `triggerState`，並將 `retry-after` 間隔設定為 15 秒。 | 
||| 

> [!TIP]
> 如需範例輪詢觸發程序模式，請檢閱此 [GitHub 中的輪詢觸發程序控制器範例](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/PollTriggerController.cs)。

<a name="webhook-triggers"></a>

### <a name="wait-and-listen-for-new-data-or-events-with-the-webhook-trigger-pattern"></a>等候並接聽使用 webhook 觸發程序模式的新資料或事件

Webhook 觸發程序是推送觸發程序，會等候並接聽您服務端點中的新資料或事件。 如果新的資料或事件符合指定的條件，觸發程序就會引發，並建立邏輯應用程式執行個體，然後處理資料作為輸入。
Webhook 觸發程序作用很像本主題之前所述的 [webhook 動作](#webhook-actions)，並使用 `subscribe` 和 `unsubscribe` 端點加以設定。 

* `subscribe` 端點：當您在邏輯應用程式中新增及儲存 webhook 觸發程序時，Logic Apps 引擎會呼叫 `subscribe` 端點。 這個步驟會導致邏輯應用程式建立您 API 所儲存的回呼 URL。 當沒有新的資料或事件符合指定的條件時，您的 API 會使用 HTTP POST 回呼 URL。 內容承載和標頭會作為輸入傳遞至邏輯應用程式。

* `unsubscribe` 端點：如果將 webhook 觸發程序或整個邏輯應用程式刪除時，Logic Apps 引擎會呼叫 `unsubscribe` 端點。 接著，您的 API 就可以取消註冊回呼 URL，並視需要停止任何處理程序。

![Webhook 觸發程序](./media/logic-apps-create-api-app/custom-api-webhook-trigger-pattern.png)

> [!NOTE]
> 目前，Logic App Designer 並不支援透過 Swagger 探索 webhook 端點。 因此對於此模式，您必須新增 [**Webhook** 觸發程序](../connectors/connectors-native-webhook.md)並指定 URL、標頭，以及您的要求主體。 另請參閱 [HTTPWebhook 觸發程序](logic-apps-workflow-actions-triggers.md#httpwebhook-trigger)。 若要傳入回呼 URL，您可以視需要在任何一個先前的欄位中使用 `@listCallbackUrl()` 工作流程函式。
>
> 若要避免多次處理相同的資料，您的觸發程序應該將已讀取並傳遞至邏輯應用程式的資料清除。

> [!TIP]
> 如需範例 webhook 模式，請檢閱此 [GitHub 中的 webhook 觸發程序控制器範例](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs)。

## <a name="secure-calls-to-your-apis-from-logic-apps"></a>保護從邏輯應用程式對 API 發出的呼叫

建立您的自訂 API 之後，請為 API 設定驗證，以便可以從邏輯應用程式安全地呼叫它們。 了解[如何保護從邏輯應用程式對自訂 API 發出的呼叫](../logic-apps/logic-apps-custom-api-authentication.md)。

## <a name="deploy-and-call-your-apis"></a>部署和呼叫您的 API

設定驗證之後，請為您的 API 設定部署。 了解[如何從邏輯應用程式部署和呼叫自訂 API](../logic-apps/logic-apps-custom-api-host-deploy-call.md)。

## <a name="publish-custom-apis-to-azure"></a>將自訂 API 發佈至 Azure

若要將您的自訂 API 提供給 Azure 中的其他 Logic Apps 使用者使用，您必須新增安全性並將它們註冊為 Logic Apps 連接器。 如需詳細資訊，請參閱[自訂連接器概觀](../logic-apps/custom-connector-overview.md)。 

若要將您的自訂 API 提供給 Logic Apps、Microsoft Flow 及 Microsoft PowerApps 中的所有使用者使用，您必須新增安全性、將 API 註冊為 Logic Apps 連接器，並提名您的連接器來進行 [Microsoft Azure 認證計劃](https://azure.microsoft.com/marketplace/programs/certified/logic-apps/)。 

## <a name="get-support"></a>取得支援

* 如需自訂 API 的特定說明，請連絡 [customapishelp@microsoft.com](mailto:customapishelp@microsoft.com)。

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。

* 若要協助改善 Logic Apps，請在 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)上票選或提交想法。 

## <a name="next-steps"></a>後續步驟

* [處理錯誤和例外狀況](../logic-apps/logic-apps-exception-handling.md)
* [透過 HTTP 端點呼叫、觸發或巢狀處理 Logic Apps](../logic-apps/logic-apps-http-endpoint.md)
* [動作和觸發程序的使用量計量](../logic-apps/logic-apps-pricing.md)
