---
title: 如何透過「Azure 監視器」警示觸發複雜動作
description: 了解如何建立邏輯應用程動作來處理「Azure 監視器」警示。
author: dkamstra
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: dukek
ms.component: alerts
ms.openlocfilehash: 3c1bff98debf426fc02dbd4518c0bb798d7ba96d
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576012"
---
# <a name="how-to-trigger-complex-actions-with-azure-monitor-alerts"></a>如何透過「Azure 監視器」警示觸發複雜動作

本文示範如何設定及觸發邏輯應用程式，以在引發警示時於 Microsoft Teams 中建立交談。

## <a name="overview"></a>概觀
Azure 監視器警示會在觸發時呼叫[動作群組](monitoring-action-groups.md)。 動作群組可讓您觸發一個或多個動作，以通知他人發生警示並一併進行修復。

一般的程序如下：

-   建立個別警示類型的邏輯應用程式。

-   將個別警示類型的結構描述匯入至邏輯應用程式。

-   定義邏輯應用程式行為。

-   將邏輯應用程式的 HTTP 端點複製到 Azure 動作群組。

如果您想要讓邏輯應用程式執行不同動作，其程序也會類似。

## <a name="create-an-activity-log-alert-administrative"></a>建立活動記錄警示：系統管理

1.  在 Azure 入口網站中，選取左上角的 [建立資源]。

2.  搜尋並選取 [邏輯應用程式]，然後選取 [建立]。

3.  指定邏輯應用程式的 [名稱]、選擇 [資源群組] 等等。

    ![建立邏輯應用程式](media/monitoring-action-groups/create-logic-app-dialog.png "建立邏輯應用程式")

4.  選取 [建立] 以建立邏輯應用程式。 系統會顯示快顯訊息，指出已建立邏輯應用程式。 選取 [啟動資源]以開啟 [Logic Apps 設計工具]。

5.  選取觸發程序：[收到 HTTP 要求時]。

    ![邏輯應用程式觸發程序](media/monitoring-action-groups/logic-app-triggers.png "邏輯應用程式觸發程序")

6.  選取 [編輯] 以變更 HTTP 要求觸發程序。

    ![HTTP 要求觸發程序](media/monitoring-action-groups/http-request-trigger-shape.png "HTTP 要求觸發程序")

7.  選取 [使用範例承載來產生結構描述]。

    ![使用範例承載](media/monitoring-action-groups/use-sample-payload-button.png "使用範例承載")

8.  將下列範例結構描述複製並貼到對話方塊中：

    ```json
        {
            "schemaId": "Microsoft.Insights/activityLogs",
            "data": {
                "status": "Activated",
                "context": {
                "activityLog": {
                    "authorization": {
                    "action": "microsoft.insights/activityLogAlerts/write",
                    "scope": "/subscriptions/…"
                    },
                    "channels": "Operation",
                    "claims": "…",
                    "caller": "logicappdemo@contoso.com",
                    "correlationId": "91ad2bac-1afa-4932-a2ce-2f8efd6765a3",
                    "description": "",
                    "eventSource": "Administrative",
                    "eventTimestamp": "2018-04-03T22:33:11.762469+00:00",
                    "eventDataId": "ec74c4a2-d7ae-48c3-a4d0-2684a1611ca0",
                    "level": "Informational",
                    "operationName": "microsoft.insights/activityLogAlerts/write",
                    "operationId": "61f59fc8-1442-4c74-9f5f-937392a9723c",
                    "resourceId": "/subscriptions/…",
                    "resourceGroupName": "LOGICAPP-DEMO",
                    "resourceProviderName": "microsoft.insights",
                    "status": "Succeeded",
                    "subStatus": "",
                    "subscriptionId": "…",
                    "submissionTimestamp": "2018-04-03T22:33:36.1068742+00:00",
                    "resourceType": "microsoft.insights/activityLogAlerts"
                }
                },
                "properties": {}
            }
        }
    ```

9. [邏輯應用程式設計工具] 會顯示一個快顯示窗，以提醒您傳送給邏輯應用程式的要求必須將 **Content-Type** 標頭設定為 **application/json**。 請關閉快顯視窗。 「Azure 監視器」警示會設定此標頭。

    ![設定 Content-type 標頭](media/monitoring-action-groups/content-type-header.png "設定 Content-type 標頭")

10. 選取 [+ 新增步驟]，然後選擇 [新增動作]。

    ![新增動作](media/monitoring-action-groups/add-action.png "新增動作")

11. 搜尋並選取 Microsoft Teams 連接器。 選擇 [Microsoft Teams – 張貼訊息] 動作。

    ![動作](media/monitoring-action-groups/microsoft-teams-actions.png "動作")

12. 設定 Microsoft Teams 動作。 [Logic Apps 設計工具] 會要求您向 Office 365 帳戶進行驗證。 選擇要對其傳送訊息的 [小組識別碼] 和 [通道識別碼]。

13. 使用靜態文字與對動態內容中 \<欄位\> 之參考的組合來設定訊息。 將下列文字複製並貼到 [訊息] 欄位中：

    ```text
      Activity Log Alert: <eventSource>
      operationName: <operationName>
      status: <status>
      resourceId: <resourceId>
    ```

    然後搜尋 \<欄位\>，並以相同名稱的動態內容標記取代這些欄位。

    > [!NOTE]
    > 有兩個名為 **status** 的動態欄位。 請將這兩個欄位都新增到訊息中。 使用 **activityLog** 屬性包中的欄位，並刪除另一個欄位。 將滑鼠停留在 **status** 欄位上，即可看到完整的欄位參考，如以下螢幕擷取畫面所示：

    ![Microsoft Teams 動作：張貼訊息](media/monitoring-action-groups/teams-action-post-message.png "Microsoft Teams 動作：張貼訊息")

14. 在 [Logic Apps 設計工具] 頂端，選取 [儲存] 以儲存邏輯應用程式。

15. 開啟現有的動作群組，然後新增要參考邏輯應用程式的動作。 如果您沒有現有的動作群組，請參閱[在 Azure 入口網站中建立和管理動作群組](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups)來建立一個。 請記得儲存變更。

    ![更新動作群組](media/monitoring-action-groups/update-action-group.png "更新動作群組")

下一次警示呼叫您的動作群組時，系統就會呼叫您的邏輯應用程式。

## <a name="create-a-service-health-alert"></a>建立服務健康狀態警示

「Azure 服務健康狀態」項目是活動記錄的一部分。 建立警示的程序與[建立活動記錄警示](#create-an-activity-log-alert-administrative)類似，但有一些變更：

- 步驟 1 至 7 都相同。
- 針對步驟 8，請將下列範例結構描述用於 HTTP 觸發程序：

    ```json
    {
        "schemaId": "Microsoft.Insights/activityLogs",
        "data": {
            "status": "Activated",
            "context": {
                "activityLog": {
                    "channels": "Admin",
                    "correlationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "description": "…",
                    "eventSource": "ServiceHealth",
                    "eventTimestamp": "2018-04-03T22:44:43.7467716+00:00",
                    "eventDataId": "9ce152f5-d435-ee31-2dce-104228486a6d",
                    "level": "Informational",
                    "operationName": "Microsoft.ServiceHealth/incident/action",
                    "operationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "properties": {
                        "title": "…",
                        "service": "…",
                        "region": "Global",
                        "communication": "…",
                        "incidentType": "Incident",
                        "trackingId": "…",
                        "impactStartTime": "2018-03-22T21:40:00.0000000Z",
                        "impactMitigationTime": "2018-03-22T21:41:00.0000000Z",
                        "impactedServices": "[{"ImpactedRegions"}]",
                        "defaultLanguageTitle": "…",
                        "defaultLanguageContent": "…",
                        "stage": "Active",
                        "communicationId": "11000001466525",
                        "version": "0.1.1"
                    },
                    "status": "Active",
                    "subscriptionId": "…",
                    "submissionTimestamp": "2018-04-03T22:44:50.8013523+00:00"
                }
            },
            "properties": {}
        }
    }
    ```

-  步驟 9 和 10 相同。
-  針對步驟 11 至 14，請使用下列程序：

   1. 選取 [+ 新增步驟]，然後選擇 [新增條件]。 設定下列條件，只有當輸入資料符合下列值的時候，邏輯應用程式才會執行。  將版本值輸入到文字方塊時，在值的周圍加上引號 ("0.1.1")，以確保系統會將值評估為字串，而不是數值類型。  如果您返回頁面，但是基礎程式碼仍然維持字串類型，則系統不會顯示引號。   
       - `schemaId == Microsoft.Insights/activityLogs`
       - `eventSource == ServiceHealth`
       - `version == "0.1.1"`

      ![「服務健康狀態承載條件」](media/monitoring-action-groups/service-health-payload-condition.png "服務健康狀態承載條件")

   1. 在 [若為 true] 條件中，依照[建立活動記錄警示](#create-an-activity-log-alert-administrative)中步驟 11 至 13 的指示來新增 Microsoft Teams 動作。

   1. 使用 HTML 與動態內容的組合來定義訊息。 將下列內容複製並貼到 [訊息] 欄位中。 針對 `[incidentType]`、`[trackingID]`、`[title]` 及 `[communication]` 欄位，以相同名稱的動態內容標記取代這些欄位：

       ```html
       <p>
       <b>Alert Type:</b>&nbsp;<strong>[incidentType]</strong>&nbsp;
       <strong>Tracking ID:</strong>&nbsp;[trackingId]&nbsp;
       <strong>Title:</strong>&nbsp;[title]</p>
       <p>
       <a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.</a>
       </p>
       <p>[communication]</p>
       ```

       ![「服務健康狀態的 True 條件張貼動作」](media/monitoring-action-groups/service-health-true-condition-post-action.png "服務健康狀態的 True 條件張貼動作")

   1. 針對 [若為 false] 條件，提供有用的訊息：

       ```html
       <p><strong>Service Health Alert</strong></p>
       <p><b>Unrecognized alert schema</b></p>
       <p><a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.\</a></p>
       ```

       ![「服務健康情況的 False 條件張貼動作」](media/monitoring-action-groups/service-health-false-condition-post-action.png "服務健康情況的 False 條件張貼動作")

- 步驟 15 相同。 請依照指示來儲存邏輯應用程式並更新動作群組。

## <a name="create-a-metric-alert"></a>建立計量警示

建立計量警示的程序與[建立活動記錄警示](#create-an-activity-log-alert-administrative)類似，但有一些變更：

- 步驟 1 至 7 都相同。
- 針對步驟 8，請將下列範例結構描述用於 HTTP 觸發程序：

    ```json
    {
    "schemaId": "AzureMonitorMetricAlert",
    "data": {
        "version": "2.0",
        "status": "Activated",
        "context": {
        "timestamp": "2018-04-09T19:00:07.7461615Z",
        "id": "…",
        "name": "TEST-VM CPU Utilization",
        "description": "",
        "conditionType": "SingleResourceMultipleMetricCriteria",
        "condition": {
            "windowSize": "PT15M",
            "allOf": [
                {
                    "metricName": "Percentage CPU",
                    "dimensions": [
                    {
                        "name": "ResourceId",
                        "value": "d92fc5cb-06cf-4309-8c9a-538eea6a17a6"
                    }
                ],
                "operator": "GreaterThan",
                "threshold": "5",
                "timeAggregation": "PT15M",
                "metricValue": 1.0
            }
            ]
        },
        "subscriptionId": "…",
        "resourceGroupName": "TEST",
        "resourceName": "test-vm",
        "resourceType": "Microsoft.Compute/virtualMachines",
        "resourceId": "…",
        "portalLink": "…"
        },
        "properties": {}
    }
    }
    ```

- 步驟 9 和 10 相同。
- 針對步驟 11 至 14，請使用下列程序：

   1. 選取 [+ 新增步驟]，然後選擇 [新增條件]。 設定下列條件，只有當輸入資料符合下列值的時候，邏輯應用程式才會執行。 將版本值輸入到文字方塊時，在值的周圍加上引號 ("2.0")，以確保系統會將值評估為字串，而不是數值類型。  如果您返回頁面，但是基礎程式碼仍然維持字串類型，則系統不會顯示引號。 
       - `schemaId == AzureMonitorMetricAlert`
       - `version == "2.0"`
       
       ![「計量警示承載條件」](media/monitoring-action-groups/metric-alert-payload-condition.png "計量警示承載條件")

   1. 在 [若為 true] 條件中，新增 **For each** 迴圈和 Microsoft Teams 動作。 使用 HTML 與動態內容的組合來定義訊息。

       ![「計量警示的 True 條件張貼動作」](media/monitoring-action-groups/metric-alert-true-condition-post-action.png "計量警示的 True 條件張貼動作")

   1. 在 [若為 false] 條件中，定義 Microsoft Teams 動作，以傳達計量警示不符合邏輯應用程式預期的訊息。 請包含自訂 JSON 承載。 請注意如何在 `json()` 運算式中參考 `triggerBody` 動態內容。

       ![「計量警示的 False 條件張貼動作」](media/monitoring-action-groups/metric-alert-false-condition-post-action.png "計量警示的 False 條件張貼動作")

- 步驟 15 相同。 請依照指示來儲存邏輯應用程式並更新動作群組。

## <a name="calling-other-applications-besides-microsoft-teams"></a>呼叫 Microsoft Teams 以外的其他應用程式
Logic Apps 有許多不同的連接器，可讓您在各種應用程式和資料庫中觸發動作。 Slack、SQL Server、Oracle、Salesforce 都只是一部分的範例。 如需有關連接器的詳細資訊，請參閱 [Logic App 連接器](../connectors/apis-list.md)。  

## <a name="next-steps"></a>後續步驟
* 取得 [Azure 活動記錄警示的概觀](monitoring-overview-alerts.md)，並了解如何接收警示。  
* 了解如何[設定 Azure 服務健康狀態通知公佈時的警示](monitoring-activity-log-alerts-on-service-notifications.md)。
* 深入了解[動作群組](monitoring-action-groups.md)。
