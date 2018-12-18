---
title: 了解活動記錄警示中使用的 Webhook 結構描述
description: 深入了解活動記錄警示啟動時，張貼至 Webhook URL 的 JSON 結構描述。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 03/31/2017
ms.author: johnkem
ms.component: alerts
ms.openlocfilehash: e989406c852b7c87123681dd875f9cd8229524c1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971921"
---
# <a name="webhooks-for-azure-activity-log-alerts"></a>Azure 活動記錄警示的 Webhook
在定義動作群組的過程中，您可以設定 Webhook 端點以接收活動記錄警示通知。 您可以使用 Webhook 將這些通知路由到其他系統，以進行後置處理或自訂動作。 本文會說明 HTTP POST 至 Webhook 的承載資料樣貌。

如需有關活動記錄警示的詳細資訊，請參閱如何[建立 Azure 活動記錄警示](monitoring-activity-log-alerts.md)。

如需有關動作群組的資訊，請參閱如何[建立動作群組](monitoring-action-groups.md)。

## <a name="authenticate-the-webhook"></a>驗證 Webhook
Webhook 可以選擇使用以權杖作為基礎的授權來進行驗證。 Webhook URI 是以權杖識別碼儲存，例如，`https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`。

## <a name="payload-schema"></a>承載結構描述
POST 作業中所包含的 JSON 承載，會根據承載的 data.context.activityLog.eventSource 欄位而有所不同。

### <a name="common"></a>一般
```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "channels": "Operation",
                "correlationId": "6ac88262-43be-4adf-a11c-bd2179852898",
                "eventSource": "Administrative",
                "eventTimestamp": "2017-03-29T15:43:08.0019532+00:00",
                "eventDataId": "8195a56a-85de-4663-943e-1a2bf401ad94",
                "level": "Informational",
                "operationName": "Microsoft.Insights/actionGroups/write",
                "operationId": "6ac88262-43be-4adf-a11c-bd2179852898",
                "status": "Started",
                "subStatus": "",
                "subscriptionId": "52c65f65-0518-4d37-9719-7dbbfc68c57a",
                "submissionTimestamp": "2017-03-29T15:43:20.3863637+00:00",
                ...
            }
        },
        "properties": {}
    }
}
```
### <a name="administrative"></a>管理
```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "authorization": {
                    "action": "Microsoft.Insights/actionGroups/write",
                    "scope": "/subscriptions/52c65f65-0518-4d37-9719-7dbbfc68c57b/resourceGroups/CONTOSO-TEST/providers/Microsoft.Insights/actionGroups/IncidentActions"
                },
                "claims": "{...}",
                "caller": "me@contoso.com",
                "description": "",
                "httpRequest": "{...}",
                "resourceId": "/subscriptions/52c65f65-0518-4d37-9719-7dbbfc68c57b/resourceGroups/CONTOSO-TEST/providers/Microsoft.Insights/actionGroups/IncidentActions",
                "resourceGroupName": "CONTOSO-TEST",
                "resourceProviderName": "Microsoft.Insights",
                "resourceType": "Microsoft.Insights/actionGroups"
            }
        },
        "properties": {}
    }
}

```
### <a name="servicehealth"></a>服務健康狀況
```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
    "status": "Activated",
    "context": {
        "activityLog": {
        "channels": "Admin",
        "correlationId": "bbac944f-ddc0-4b4c-aa85-cc7dc5d5c1a6",
        "description": "Active: Virtual Machines - Australia East",
        "eventSource": "ServiceHealth",
        "eventTimestamp": "2017-10-18T23:49:25.3736084+00:00",
        "eventDataId": "6fa98c0f-334a-b066-1934-1a4b3d929856",
        "level": "Informational",
        "operationName": "Microsoft.ServiceHealth/incident/action",
        "operationId": "bbac944f-ddc0-4b4c-aa85-cc7dc5d5c1a6",
        "properties": {
            "title": "Virtual Machines - Australia East",
            "service": "Virtual Machines",
            "region": "Australia East",
            "communication": "Starting at 02:48 UTC on 18 Oct 2017 you have been identified as a customer using Virtual Machines in Australia East who may receive errors starting Dv2 Promo and DSv2 Promo Virtual Machines which are in a stopped &quot;deallocated&quot; or suspended state. Customers can still provision Dv1 and Dv2 series Virtual Machines or try deploying Virtual Machines in other regions, as a possible workaround. Engineers have identified a possible fix for the underlying cause, and are exploring implementation options. The next update will be provided as events warrant.",
            "incidentType": "Incident",
            "trackingId": "0NIH-U2O",
            "impactStartTime": "2017-10-18T02:48:00.0000000Z",
            "impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"Australia East\"}],\"ServiceName\":\"Virtual Machines\"}]",
            "defaultLanguageTitle": "Virtual Machines - Australia East",
            "defaultLanguageContent": "Starting at 02:48 UTC on 18 Oct 2017 you have been identified as a customer using Virtual Machines in Australia East who may receive errors starting Dv2 Promo and DSv2 Promo Virtual Machines which are in a stopped &quot;deallocated&quot; or suspended state. Customers can still provision Dv1 and Dv2 series Virtual Machines or try deploying Virtual Machines in other regions, as a possible workaround. Engineers have identified a possible fix for the underlying cause, and are exploring implementation options. The next update will be provided as events warrant.",
            "stage": "Active",
            "communicationId": "636439673646212912",
            "version": "0.1.1"
        },
        "status": "Active",
        "subscriptionId": "45529734-0ed9-4895-a0df-44b59a5a07f9",
        "submissionTimestamp": "2017-10-18T23:49:28.7864349+00:00"
        }
    },
    "properties": {}
    }
}
```

### <a name="resourcehealth"></a>ResourceHealth
```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "channels": "Admin, Operation",
                "correlationId": "a1be61fd-37ur-ba05-b827-cb874708babf",
                "eventSource": "ResourceHealth",
                "eventTimestamp": "2018-09-04T23:09:03.343+00:00",
                "eventDataId": "2b37e2d0-7bda-4de7-ur8c6-1447d02265b2",
                "level": "Informational",
                "operationName": "Microsoft.Resourcehealth/healthevent/Activated/action",
                "operationId": "2b37e2d0-7bda-489f-81c6-1447d02265b2",
                "properties": {
                    "title": "Virtual Machine health status changed to unavailable",
                    "details": "Virtual machine has experienced an unexpected event",
                    "currentHealthStatus": "Unavailable",
                    "previousHealthStatus": "Available",
                    "type": "Downtime",
                    "cause": "PlatformInitiated",
                },
                "resourceId": "/subscriptions/<subscription Id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>",
                "resourceGroupName": "<resource group>",
                "resourceProviderName": "Microsoft.Resourcehealth/healthevent/action",
                "status": "Active",
                "subscriptionId": "<subscription Id",
                "submissionTimestamp": "2018-09-04T23:11:06.1607287+00:00",
                "resourceType": "Microsoft.Compute/virtualMachines"
            }
        }
    }
}
```

如需服務健康狀態通知活動記錄警示的特定結構描述詳細資料，請參閱[服務健康狀態通知](monitoring-service-notifications.md)。 此外，了解如何[使用現有問題管理解決方案來設定服務健康情況的 Webhook 通知](../service-health/service-health-alert-webhook-guide.md)。

如需了解所有其他活動記錄警示的特定結構詳細資料，請參閱 [Azure 活動記錄的概觀](monitoring-overview-activity-logs.md)。

| 元素名稱 | 說明 |
| --- | --- |
| status |用於度量警示。 針對活動記錄警示，一律設為「啟動」。 |
| context |事件的內容。 |
| resourceProviderName |受影響資源的資源提供者。 |
| conditionType |一律為「事件」。 |
| name |警示規則的名稱。 |
| id |警示的資源識別碼。 |
| 說明 |建立警示時，會設定警示描述。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |
| timestamp |處理要求的 Azure 服務產生事件的時間。 |
| ResourceId |受影響資源的資源識別碼。 |
| resourceGroupName |受影響資源的資源群組的名稱。 |
| properties |一組包含事件相關詳細資料的`<Key, Value>`配對 (也就是 `Dictionary<String, String>`)。 |
| 事件 |包含事件相關中繼資料的元素。 |
| 授權 |事件的角色型存取控制屬性。 這些屬性通常包括動作、角色和範圍。 |
| category |事件的類別。 支援值包括「管理」、「警示」、「安全性」、「ServiceHealth」和「建議」。 |
| 呼叫者 |已執行作業的使用者的電子郵件地址，根據可用性的 UPN 宣告或 SPN 宣告。 特定系統呼叫可為 Null。 |
| correlationId |通常是字串格式的 GUID。 具有屬於同一個較大動作的 correlationId 的事件，且通常會共用 correlationId。 |
| eventDescription |事件的靜態文字描述。 |
| eventDataId |事件的唯一識別碼。 |
| eventSource |產生事件的 Azure 服務或基礎結構的名稱。 |
| httpRequest |要求通常包括 “clientRequestId”、“clientIpAddress” 和 HTTP “method” (例如 PUT)。 |
| 層級 |下列其中一個值：Critical、Error、Warning 和 Informational。 |
| operationId |通常是對應至單一作業的事件之間共用的 GUID。 |
| operationName |作業名稱。 |
| properties |事件的屬性。 |
| status |字串。 作業的狀態。 常見的值包括︰「已啟動」、「進行中」、「成功」、「失敗」、「使用中」和「已解決」。 |
| 子狀態 |通常包含對應 REST 呼叫的 HTTP 狀態碼。 它也可以包含其他描述子狀態的字串。 常見子狀態的值包括：確定 (HTTP 狀態碼︰200)，已建立 (HTTP 狀態碼︰201)、接受 (HTTP 狀態碼︰202)、沒有內容 (HTTP 狀態碼︰204)、不正確的要求 (HTTP 狀態碼︰400)、找不到 (HTTP 狀態碼︰404)，衝突 (HTTP 狀態碼︰409)、內部伺服器錯誤 (HTTP 狀態碼︰500)、服務無法使用 (HTTP 狀態碼︰503) 和閘道逾時 (HTTP 狀態碼︰504)。 |

## <a name="next-steps"></a>後續步驟
* [深入了解活動記錄](monitoring-overview-activity-logs.md)。
* [對 Azure 警示執行 Azure 自動化指令碼 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)。
* [使用邏輯應用程式透過 Twilio 從 Azure 警示傳送 SMS](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)。 此範例適用於計量警示，但經過修改後即可用於活動記錄警示。
* [使用邏輯應用程式從 Azure 警示傳送 Slack 訊息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)。 此範例適用於計量警示，但經過修改後即可用於活動記錄警示。
* [使用邏輯應用程式從 Azure 警示將訊息傳送到 Azure 佇列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)。 此範例適用於計量警示，但經過修改後即可用於活動記錄警示。
