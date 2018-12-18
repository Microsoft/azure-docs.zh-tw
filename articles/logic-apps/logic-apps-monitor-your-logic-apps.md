---
title: 檢查狀態、設定記錄並取得警示 - Azure Logic Apps | Microsoft Docs
description: 監視 Azure Logic Apps 的狀態、記錄診斷資料，以及設定其警示
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 5c1b1e15-3b6c-49dc-98a6-bdbe7cb75339
ms.date: 07/21/2017
ms.openlocfilehash: a08cd6289fc85b79ccec731126a33a9549d60546
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123528"
---
# <a name="monitor-status-set-up-diagnostics-logging-and-turn-on-alerts-for-azure-logic-apps"></a>監視狀態、設定診斷記錄，以及開啟 Azure Logic Apps 的警示

在您[建立和執行邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)之後，即可檢查其執行歷程記錄、觸發程式歷程記錄、狀態和效能。 針對即時事件監視和更豐富的偵錯，設定邏輯應用程式的[診斷記錄](#azure-diagnostics)。 因此，您可以[尋找並檢視事件](#find-events)，例如觸發程序事件、執行事件和動作事件。 您也可以[搭配使用此診斷資料與其他服務](#extend-diagnostic-data)，例如 Azure 儲存體和 Azure 事件中樞。 

若要取得失敗或其他可能問題的通知，請設定[警示](#add-azure-alerts)。 例如，您可以建立警示來偵測「一小時有五個以上的執行失敗時」。 您也可以使用 [Azure 診斷事件設定和內容](#diagnostic-event-properties)，透過程式設計方式設定監視、追蹤和記錄。

## <a name="view-runs-and-trigger-history-for-your-logic-app"></a>檢視邏輯應用程式的執行和觸發程序歷程記錄

1. 若要在 [Azure 入口網站](https://portal.azure.com)中尋找邏輯應用程式，請在主要 Azure 功能表上選擇 [所有服務]。 在搜尋方塊中，輸入「邏輯應用程式」，然後選擇 [邏輯應用程式]。

   ![尋找邏輯應用程式](./media/logic-apps-monitor-your-logic-apps/find-your-logic-app.png)

   Azure 入口網站會顯示與 Azure 訂用帳戶相關聯的所有邏輯應用程式。 

2. 選取邏輯應用程式，然後選擇 [概觀]。

   Azure 入口網站會顯示邏輯應用程式的執行歷程記錄和觸發程序歷程記錄。 例如︰

   ![邏輯應用程式執行歷程記錄和觸發程序歷程記錄](media/logic-apps-monitor-your-logic-apps/overview.png)

   * **執行歷程記錄**會顯示邏輯應用程式的所有執行。 
   * **觸發程序歷程記錄**會顯示邏輯應用程式的所有觸發程序活動。

   如需狀態描述，請參閱[針對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

   > [!TIP]
   > 如果找不到您預期的資料，請在工具列上選擇 [重新整理]。

3. 若要檢視特定執行的步驟，請在 [執行歷程記錄] 下選取該執行。 

   監視檢視會顯示該執行中的每個步驟。 例如︰

   ![特定執行的動作](media/logic-apps-monitor-your-logic-apps/monitor-view-updated.png)

4. 若要取得執行的其他詳細資訊，請選擇 [執行詳細資料]。 這項資訊摘要說明執行的步驟、狀態、輸入和輸出。 

   ![選擇 [執行詳細資料]](media/logic-apps-monitor-your-logic-apps/run-details.png)

   例如，您可以取得執行的**相互關聯識別碼**，在您使用 [REST API for Logic Apps](https://docs.microsoft.com/rest/api/logic) 時可能需要此識別碼。

5. 若要取得特定步驟的詳細資料，請選擇該步驟。 您現在可以檢閱詳細資料，例如輸入、輸出以及該步驟所發生的任何錯誤。 例如︰

   ![步驟詳細資料](media/logic-apps-monitor-your-logic-apps/monitor-view-details.png)
   
   > [!NOTE]
   > 在 Logic Apps 服務內，會加密所有執行階段詳細資料和事件。 只有在使用者要求檢視該資料時，才對其進行解密。 您也可以使用 [Azure 角色型存取控制 (RBAC)](../role-based-access-control/overview.md) 來控制對這些事件的存取權。

6. 若要取得特定觸發程序事件的詳細資料，請回到 [概觀] 窗格。 在 [觸發程序歷程記錄] 下，選取觸發程序事件。 您現在可以檢閱輸入和輸出這類詳細資料，例如：

   ![觸發程序事件輸出詳細資料](media/logic-apps-monitor-your-logic-apps/trigger-details.png)

<a name="azure-diagnostics"></a>

## <a name="turn-on-diagnostics-logging-for-your-logic-app"></a>開啟邏輯應用程式的診斷記錄

如需使用執行階段詳細資料和事件進行更豐富的偵錯，您可以使用 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 設定診斷記錄。 Log Analytics 是 Azure 中的一項服務，可監視雲端和內部部署環境，協助您維護其可用性和效能。 

開始之前，您需要有 Log Analytics 工作區。 了解[如何建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取邏輯應用程式。 

2. 在邏輯應用程式刀鋒視窗功能表上，於 [監視] 下選擇 [診斷] > [診斷設定]。

   ![移至 [監視]、[診斷]、[診斷設定]](media/logic-apps-monitor-your-logic-apps/logic-app-diagnostics.png)

3. 在 [診斷設定] 下，選擇 [開啟]。

   ![開啟診斷記錄](media/logic-apps-monitor-your-logic-apps/turn-on-diagnostics-logic-app.png)

4. 現在選取用於記錄的 Log Analytics 工作區和事件類別目錄，如下所示：

   1. 選取 [傳送至 Log Analytics]。 
   2. 在 [Log Analytics] 下，選擇 [設定]。 
   3. 在 [OMS 工作區] 下，選取要用於記錄的 Log Analytics 工作區。
   4. 在 [記錄] 下，選取 [WorkflowRuntime] 分類。
   5. 選擇計量間隔。
   6. 完成之後，請選擇 [儲存]。

   ![選取用於記錄的 Log Analytics 工作區和資料](media/logic-apps-monitor-your-logic-apps/send-diagnostics-data-log-analytics-workspace.png)

現在，您可以尋找事件，以及觸發程序事件、執行事件和動作事件的其他資料。

<a name="find-events"></a>

## <a name="find-events-and-data-for-your-logic-app"></a>尋找邏輯應用程式的事件和資料

若要在邏輯應用程式中尋找並檢視觸發程序事件、執行事件和動作事件這類事件，請遵循下列步驟。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選擇 [所有服務]。 搜尋 "log analytics"，然後選擇 [Log Analytics]，如下所示：

   ![選擇 [Log Analytics]](media/logic-apps-monitor-your-logic-apps/browseloganalytics.png)

2. 在 [Log Analytics] 下，尋找並選取 Log Analytics 工作區。 

   ![選取 Log Analytics 工作區](media/logic-apps-monitor-your-logic-apps/selectla.png)

3. 在 [管理] 下，選擇 [記錄搜尋]。

   ![選擇 [記錄搜尋]](media/logic-apps-monitor-your-logic-apps/log-search.png)

4. 在搜尋方塊中，指定您想要尋找的欄位，然後按 **Enter**。 當您開始輸入時，會看到您可以使用的可能相符項目和作業。 

   例如，若要尋找所發生的前 10 個事件，請輸入並選取此搜尋查詢：**search Category == "WorkflowRuntime" | limit 10**

   ![輸入搜尋字串](media/logic-apps-monitor-your-logic-apps/oms-start-query.png)

   深入了解[如何在 Log Analytics 中尋找資料](../log-analytics/log-analytics-log-searches.md)。

5. 在結果頁面上，於左列中選擇您想要檢視的時間範圍。
若要新增篩選來調整您的查詢，請選擇 [+新增]。

   ![選擇查詢結果的時間範圍](media/logic-apps-monitor-your-logic-apps/query-results.png)

6. 在 [新增篩選] 下，輸入篩選名稱，以找到您想要的篩選。 選取篩選，然後選擇 [+新增]。

   此範例使用 "status" 這個字，在 **AzureDiagnostics** 下尋找失敗事件。
   在這裡，已選取 **status_s** 的篩選。

   ![選取篩選](media/logic-apps-monitor-your-logic-apps/log-search-add-filter.png)

7. 在左列中，選取您想要使用的篩選值，然後選擇 [套用]。

   ![選取篩選值，然後選擇 [套用]](media/logic-apps-monitor-your-logic-apps/log-search-apply-filter.png)

8. 現在，回到您要建置的查詢。 使用您所選取的篩選和值來更新您的查詢。 現在也會篩選您的先前結果。

   ![回到具有已篩選結果的查詢](media/logic-apps-monitor-your-logic-apps/log-search-query-filtered-results.png)

9. 若要儲存查詢供日後使用，請選擇 [儲存]。 了解[如何儲存查詢](../logic-apps/logic-apps-track-b2b-messages-omsportal-query-filter-control-number.md#save-oms-query)。

<a name="extend-diagnostic-data"></a>

## <a name="extend-how-and-where-you-use-diagnostic-data-with-other-services"></a>延伸搭配使用診斷資料與其他服務的方式和位置

使用 Azure Log Analytics，即可延伸如何搭配使用邏輯應用程式的診斷資料與其他 Azure services，例如： 

* [在 Azure 儲存體中封存 Azure 診斷記錄](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md)
* [將 Azure 診斷記錄串流至 Azure 事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md) 

您可以接著使用其他服務的遙測和分析來取得即時監視，例如 [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) 和 [Power BI](../log-analytics/log-analytics-powerbi.md)。 例如︰

* [將資料從事件中樞串流至串流分析](../stream-analytics/stream-analytics-define-inputs.md)
* [使用串流分析分析串流資料並在 Power BI 中建立即時分析儀表板](../stream-analytics/stream-analytics-power-bi-dashboard.md)

根據您要設定的選項，確定您先[建立 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)或[建立 Azure 事件中樞](../event-hubs/event-hubs-create.md)。 然後選取您要傳送診斷資料的選項：

![將資料傳送至 Azure 儲存體帳戶或事件中樞](./media/logic-apps-monitor-your-logic-apps/storage-account-event-hubs.png)

> [!NOTE]
> 只有在您選擇使用儲存體帳戶時，才會套用保留期間。

<a name="add-azure-alerts"></a>

## <a name="set-up-alerts-for-your-logic-app"></a>設定邏輯應用程式的警示

若要監視邏輯應用程式的特定計量或已超過閾值，請設定 [Azure 中的警示](../monitoring-and-diagnostics/monitoring-overview-alerts.md)。 了解 [Azure 中的計量](../monitoring-and-diagnostics/monitoring-overview-metrics.md)。 

若要在不使用 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 的情況下設定警示，請遵循下列步驟。 如需更進階的警示準則和動作，也請[設定 Log Analytics](#azure-diagnostics)。

1. 在邏輯應用程式刀鋒視窗功能表上，於 [監視] 下選擇 [診斷] > [警示規則] > [新增警示]，如下所示：

   ![新增邏輯應用程式的警示](media/logic-apps-monitor-your-logic-apps/set-up-alerts.png)

2. 在 [新增警示規則] 刀鋒視窗上，建立您的警示，如下所示：

   1. 在 [資源] 下，選取尚未選取的邏輯應用程式。 
   2. 提供警示的名稱和描述。
   3. 選取您想要追蹤的 [計量] 或事件。
   4. 選取 [條件]，並指定計量的 [閾值]，然後選取用於監視此計量的 [期間]。
   5. 選取是否傳送警示的郵件。 
   6. 指定用於傳送警示的任何其他電子郵件地址。 
   您也可以指定要傳送警示的 Webhook URL。

   例如，一小時有五個以上的執行失敗時，此規則會傳送警示：

   ![建立計量警示規則](media/logic-apps-monitor-your-logic-apps/create-alert-rule.png)

> [!TIP]
> 若要從警示執行邏輯應用程式，您可以在工作流程中包含[要求觸發程序](../connectors/connectors-native-reqres.md)，以讓您執行工作，例如下列範例：
> 
> * [張貼到 Slack](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)
> * [傳送文字](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)
> * [將訊息新增至佇列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

<a name="diagnostic-event-properties"></a>

## <a name="azure-diagnostics-event-settings-and-details"></a>Azure 診斷事件設定和詳細資料

每個診斷事件都會有邏輯應用程式和該事件的詳細資料，例如，狀態、開始時間、結束時間等等。 若要以程式設計方式設定監視、追蹤和記錄，您可以搭配使用這些詳細資料與 [REST API for Azure Logic Apps](https://docs.microsoft.com/rest/api/logic) 和 [REST API for Azure Diagnostics](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftlogicworkflows)。

例如，`ActionCompleted` 事件具有可用於追蹤和監視的 `clientTrackingId` 和 `trackedProperties` 屬性：

``` json
{
    "time": "2016-07-09T17:09:54.4773148Z",
    "workflowId": "/SUBSCRIPTIONS/<subscription-ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/MYLOGICAPP",
    "resourceId": "/SUBSCRIPTIONS/<subscription-ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/MYLOGICAPP/RUNS/08587361146922712057/ACTIONS/HTTP",
    "category": "WorkflowRuntime",
    "level": "Information",
    "operationName": "Microsoft.Logic/workflows/workflowActionCompleted",
    "properties": {
        "$schema": "2016-06-01",
        "startTime": "2016-07-09T17:09:53.4336305Z",
        "endTime": "2016-07-09T17:09:53.5430281Z",
        "status": "Succeeded",
        "code": "OK",
        "resource": {
            "subscriptionId": "<subscription-ID>",
            "resourceGroupName": "MyResourceGroup",
            "workflowId": "cff00d5458f944d5a766f2f9ad142553",
            "workflowName": "MyLogicApp",
            "runId": "08587361146922712057",
            "location": "westus",
            "actionName": "Http"
        },
        "correlation": {
            "actionTrackingId": "e1931543-906d-4d1d-baed-dee72ddf1047",
            "clientTrackingId": "<my-custom-tracking-ID>"
        },
        "trackedProperties": {
            "myTrackedProperty": "<value>"
        }
    }
}
```

* `clientTrackingId`： 如果未提供，Azure 會自動產生這個識別碼，並相互關聯邏輯應用程式執行之間的事件，包括從邏輯應用程式呼叫的任何巢狀工作流程。 您可以使用觸發程序要求中的自訂識別碼值來傳遞 `x-ms-client-tracking-id` 標頭，以從觸發程序手動指定此識別碼。 您可以使用要求觸發程序、HTTP 觸發程序或 Webhook 觸發程序。

* `trackedProperties`：若要追蹤診斷資料中的輸入或輸出，您可以將追蹤的屬性新增至邏輯應用程式 JSON 定義中的動作。 追蹤的屬性只能追蹤單一動作的輸入和輸出，但您可以使用事件的 `correlation` 屬性來跨執行中的動作進行相互關聯。

  若要追蹤一或多個屬性，請將您想要的 `trackedProperties` 區段和屬性新增至動作定義。 例如，假設您想要追蹤遙測中「訂單識別碼」這類資料：

  ``` json
  "myAction": {
    "type": "http",
    "inputs": {
        "uri": "http://uri",
        "headers": {
            "Content-Type": "application/json"
        },
        "body": "@triggerBody()"
    },
    "trackedProperties": {
        "myActionHTTPStatusCode": "@action()['outputs']['statusCode']",
        "myActionHTTPValue": "@action()['outputs']['body']['<content>']",
        "transactionId": "@action()['inputs']['body']['<content>']"
    }
  }
  ```

## <a name="next-steps"></a>後續步驟

* [建立範本以進行邏輯應用程式部署和發行管理](../logic-apps/logic-apps-create-deploy-template.md)
* [B2B 案例搭配企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [監視 B2B 訊息](../logic-apps/logic-apps-monitor-b2b-message.md)
