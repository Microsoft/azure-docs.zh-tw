---
title: Azure 活動記錄事件結構描述
description: 了解發出至活動記錄之資料的事件結構描述
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: reference
ms.date: 4/12/2018
ms.author: dukek
ms.component: activitylog
ms.openlocfilehash: d267ffd5085c27c60e9eb229e2d9026fa83ef848
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998218"
---
# <a name="azure-activity-log-event-schema"></a>Azure 活動記錄事件結構描述
透過「Azure 活動記錄」，您可深入了解 Azure 中發生的任何訂用帳戶層級事件。 本文說明每個資料類別的事件結構描述。 資料的結構描述取決於您是在入口網站、PowerShell、CLI，或直接透過 REST API 讀取資料，還是[使用記錄設定檔，將資料串流處理至儲存體或事件中樞](./monitoring-overview-activity-logs.md#export-the-activity-log-with-a-log-profile)。 下列範例顯示透過入口網站、PowerShell、CLI 和 REST API 提供的結構描述。 本文結尾會提供這些屬性與 [Azure 診斷記錄結構描述](./monitoring-diagnostic-logs-schema.md)的對應。

## <a name="administrative"></a>管理
透過 Resource Manager 執行的所有建立、更新、刪除和動作作業皆記錄在此類別中。 您可能會在此類別中看到的事件類型範例包括「建立虛擬機器」和「刪除網路安全性群組」。使用者或應用程式使用 Resource Manager 所執行的每個動作，都會成為特定資源類型上的作業模型。 如果作業類型為「寫入」、「刪除」或「動作」，則該作業的啟動及成功或失敗記錄皆會記錄在「系統管理」類別。 「系統管理」類別也包含訂用帳戶中角色型存取控制的所有變更。

### <a name="sample-event"></a>範例事件
```json
{
    "authorization": {
        "action": "Microsoft.Network/networkSecurityGroups/write",
        "scope": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG"
    },
    "caller": "rob@contoso.com",
    "channels": "Operation",
    "claims": {
        "aud": "https://management.core.windows.net/",
        "iss": "https://sts.windows.net/1114444b-7467-4144-a616-e3a5d63e147b/",
        "iat": "1234567890",
        "nbf": "1234567890",
        "exp": "1234567890",
        "_claim_names": "{\"groups\":\"src1\"}",
        "_claim_sources": "{\"src1\":{\"endpoint\":\"https://graph.windows.net/1114444b-7467-4144-a616-e3a5d63e147b/users/f409edeb-4d29-44b5-9763-ee9348ad91bb/getMemberObjects\"}}",
        "http://schemas.microsoft.com/claims/authnclassreference": "1",
        "aio": "A3GgTJdwK4vy7Fa7l6DgJC2mI0GX44tML385OpU1Q+z+jaPnFMwB",
        "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
        "appid": "355249ed-15d9-460d-8481-84026b065942",
        "appidacr": "2",
        "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "10845a4d-ffa4-4b61-a3b4-e57b9b31cdb5",
        "e_exp": "262800",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Robertson",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "Rob",
        "ipaddr": "111.111.1.111",
        "name": "Rob Robertson",
        "http://schemas.microsoft.com/identity/claims/objectidentifier": "f409edeb-4d29-44b5-9763-ee9348ad91bb",
        "onprem_sid": "S-1-5-21-4837261184-168309720-1886587427-18514304",
        "puid": "18247BBD84827C6D",
        "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "b-24Jf94A3FH2sHWVIFqO3-RSJEiv24Jnif3gj7s",
        "http://schemas.microsoft.com/identity/claims/tenantid": "1114444b-7467-4144-a616-e3a5d63e147b",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "rob@contoso.com",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "rob@contoso.com",
        "uti": "IdP3SUJGtkGlt7dDQVRPAA",
        "ver": "1.0"
    },
    "correlationId": "b5768deb-836b-41cc-803e-3f4de2f9e40b",
    "eventDataId": "d0d36f97-b29c-4cd9-9d3d-ea2b92af3e9d",
    "eventName": {
        "value": "EndRequest",
        "localizedValue": "End request"
    },
    "category": {
        "value": "Administrative",
        "localizedValue": "Administrative"
    },
    "eventTimestamp": "2018-01-29T20:42:31.3810679Z",
    "id": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG/events/d0d36f97-b29c-4cd9-9d3d-ea2b92af3e9d/ticks/636528553513810679",
    "level": "Informational",
    "operationId": "04e575f8-48d0-4c43-a8b3-78c4eb01d287",
    "operationName": {
        "value": "Microsoft.Network/networkSecurityGroups/write",
        "localizedValue": "Microsoft.Network/networkSecurityGroups/write"
    },
    "resourceGroupName": "myResourceGroup",
    "resourceProviderName": {
        "value": "Microsoft.Network",
        "localizedValue": "Microsoft.Network"
    },
    "resourceType": {
        "value": "Microsoft.Network/networkSecurityGroups",
        "localizedValue": "Microsoft.Network/networkSecurityGroups"
    },
    "resourceId": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-01-29T20:42:50.0724829Z",
    "subscriptionId": "<subscription ID>",
    "properties": {
        "statusCode": "Created",
        "serviceRequestId": "a4c11dbd-697e-47c5-9663-12362307157d",
        "responseBody": "",
        "requestbody": ""
    },
    "relatedEvents": []
}

```

### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| 授權 |事件的 RBAC 屬性的 blob。 通常包括 action、role 和 scope 屬性。 |
| 呼叫者 |已執行作業的使用者的電子郵件地址，根據可用性的 UPN 宣告或 SPN 宣告。 |
| 通道 |為下列其中一個值：Admin、Operation |
| claims |Active Directory 用來驗證使用者或應用程式，以便在 Resource Manager 中執行此作業的 JWT 權杖。 |
| correlationId |通常是字串格式的 GUID。 具有相同 correlationId、屬於同一 uber 動作的事件。 |
| 說明 |事件的靜態文字描述。 |
| eventDataId |事件的唯一識別碼。 |
| httpRequest |描述 HTTP 要求的 blob。 通常包括 “clientRequestId”、“clientIpAddress”和 “method” (HTTP 方法。 例如，PUT)。 |
| 層級 |事件的層級。 下列其中一個值：“Critical”、“Error”、“Warning” 和 “Informational” |
| resourceGroupName |受影響資源的資源群組的名稱。 |
| resourceProviderName |受影響資源的資源提供者的名稱。 |
| ResourceId |受影響資源的資源識別碼。 |
| operationId |對應至單一作業的事件共用的 GUID。 |
| operationName |作業名稱。 |
| properties |描述事件詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。 |
| status |字串，描述作業的狀態。 常見的值包括︰Started、In Progress、Succeeded、Failed、Active、Resolved。 |
| 子狀態 |通常包含對應 REST 呼叫的 HTTP 狀態碼，但也可以包含其他描述子狀態的字串，常見的值包括：確定 (HTTP 狀態碼︰200)，已建立 (HTTP 狀態碼︰201)、接受 (HTTP 狀態碼︰202)、沒有內容 (HTTP 狀態碼︰204)、不正確的要求 (HTTP 狀態碼︰400)、找不到 (HTTP 狀態碼︰404)，衝突 (HTTP 狀態碼︰409)、內部伺服器錯誤 (HTTP 狀態碼︰500)、服務無法使用 (HTTP 狀態碼︰503)、閘道逾時 (HTTP 狀態碼︰504)。 |
| eventTimestamp |處理與事件對應之要求的Azure 服務產生事件時的時間戳記。 |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |

## <a name="service-health"></a>服務健康情況
所有在 Azure 中發生的服務健康情況事件皆記錄在此類別中。 您可能會在此類別中看到的事件類型範例為「美國東部的 SQL Azure 發生停機事件」。 服務健康情況事件有五個種類：「需要採取動作」、「協助復原」、「事件」、「維護」、「資訊」或「安全性」，這些事件只會在訂用帳戶中有可能會受該事件影響的資源時顯示。

### <a name="sample-event"></a>範例事件
```json
{
  "channels": "Admin",
  "correlationId": "c550176b-8f52-4380-bdc5-36c1b59d3a44",
  "description": "Active: Network Infrastructure - UK South",
  "eventDataId": "c5bc4514-6642-2be3-453e-c6a67841b073",
  "eventName": {
      "value": null
  },
  "category": {
      "value": "ServiceHealth",
      "localizedValue": "Service Health"
  },
  "eventTimestamp": "2017-07-20T23:30:14.8022297Z",
  "id": "/subscriptions/<subscription ID>/events/c5bc4514-6642-2be3-453e-c6a67841b073/ticks/636361902148022297",
  "level": "Warning",
  "operationName": {
      "value": "Microsoft.ServiceHealth/incident/action",
      "localizedValue": "Microsoft.ServiceHealth/incident/action"
  },
  "resourceProviderName": {
      "value": null
  },
  "resourceType": {
      "value": null,
      "localizedValue": ""
  },
  "resourceId": "/subscriptions/<subscription ID>",
  "status": {
      "value": "Active",
      "localizedValue": "Active"
  },
  "subStatus": {
      "value": null
  },
  "submissionTimestamp": "2017-07-20T23:30:34.7431946Z",
  "subscriptionId": "<subscription ID>",
  "properties": {
    "title": "Network Infrastructure - UK South",
    "service": "Service Fabric",
    "region": "UK South",
    "communication": "Starting at approximately 21:41 UTC on 20 Jul 2017, a subset of customers in UK South may experience degraded performance, connectivity drops or timeouts when accessing their Azure resources hosted in this region. Engineers are investigating underlying Network Infrastructure issues in this region. Impacted services may include, but are not limited to App Services, Automation, Service Bus, Log Analytics, Key Vault, SQL Database, Service Fabric, Event Hubs, Stream Analytics, Azure Data Movement, API Management, and Azure Search. Multiple engineering teams are engaged in multiple workflows to mitigate the impact. The next update will be provided in 60 minutes, or as events warrant.",
    "incidentType": "Incident",
    "trackingId": "NA0F-BJG",
    "impactStartTime": "2017-07-20T21:41:00.0000000Z",
    "impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"UK South\"}],\"ServiceName\":\"Service Fabric\"}]",
    "defaultLanguageTitle": "Network Infrastructure - UK South",
    "defaultLanguageContent": "Starting at approximately 21:41 UTC on 20 Jul 2017, a subset of customers in UK South may experience degraded performance, connectivity drops or timeouts when accessing their Azure resources hosted in this region. Engineers are investigating underlying Network Infrastructure issues in this region. Impacted services may include, but are not limited to App Services, Automation, Service Bus, Log Analytics, Key Vault, SQL Database, Service Fabric, Event Hubs, Stream Analytics, Azure Data Movement, API Management, and Azure Search. Multiple engineering teams are engaged in multiple workflows to mitigate the impact. The next update will be provided in 60 minutes, or as events warrant.",
    "stage": "Active",
    "communicationId": "636361902146035247",
    "version": "0.1.1"
  }
}
```
請參閱[服務健康情況通知](./monitoring-service-notifications.md)一文，以取得關於屬性中之值的文件。

## <a name="resource-health"></a>資源健康情況
此類別包含 Azure 資源已發生的任何資源健康情況事件的記錄。 「虛擬機器健康情況已變更為無法使用」是您可能在此類別中看到的事件類型。 資源健康情況事件可以代表四個健康情況的其中一個：可用、無法使用、已降級、未知。 此外，資源健康情況事件可分類為「平台起始」或「使用者起始」。

### <a name="sample-event"></a>範例事件

```json
{
    "channels": "Admin, Operation",
    "correlationId": "28f1bfae-56d3-7urb-bff4-194d261248e9",
    "description": "",
    "eventDataId": "a80024e1-883d-37ur-8b01-7591a1befccb",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "ResourceHealth",
        "localizedValue": "Resource Health"
    },
    "eventTimestamp": "2018-09-04T15:33:43.65Z",
    "id": "/subscriptions/<subscription Id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>/events/a80024e1-883d-42a5-8b01-7591a1befccb/ticks/636716720236500000",
    "level": "Critical",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Resourcehealth/healthevent/Activated/action",
        "localizedValue": "Health Event Activated"
    },
    "resourceGroupName": "<resource group>",
    "resourceProviderName": {
        "value": "Microsoft.Resourcehealth/healthevent/action",
        "localizedValue": "Microsoft.Resourcehealth/healthevent/action"
    },
    "resourceType": {
        "value": "Microsoft.Compute/virtualMachines",
        "localizedValue": "Microsoft.Compute/virtualMachines"
    },
    "resourceId": "/subscriptions/<subscription Id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-09-04T15:36:24.2240867Z",
    "subscriptionId": "<subscription Id>",
    "properties": {
        "stage": "Active",
        "title": "Virtual Machine health status changed to unavailable",
        "details": "Virtual machine has experienced an unexpected event",
        "healthStatus": "Unavailable",
        "healthEventType": "Downtime",
        "healthEventCause": "PlatformInitiated",
        "healthEventCategory": "Unplanned"
    },
    "relatedEvents": []
}
```

### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| 通道 | 一律是 “Admin, Operation” |
| correlationId | 字串格式的 GUID。 |
| 說明 |警示事件的靜態文字描述。 |
| eventDataId |警示事件的唯一識別碼。 |
| category | 一律為 "ResourceHealth" |
| eventTimestamp |處理與事件對應之要求的Azure 服務產生事件時的時間戳記。 |
| 層級 |事件的層級。 下列其中一個值：「重大」、「錯誤」、「警告」、「資訊」與「詳細資訊」 |
| operationId |對應至單一作業的事件共用的 GUID。 |
| operationName |作業名稱。 |
| resourceGroupName |包含資源的資源群組名稱。 |
| resourceProviderName |一律為 "Microsoft.Resourcehealth/healthevent/action"。 |
| resourceType | 受資源健康情況事件影響的資源類型。 |
| ResourceId | 受影響資源之資源識別碼的名稱。 |
| status |描述健康情況事件狀態的字串。 值可以是：作用中、已解決、正在進行、已更新。 |
| subStatus | 針對警示通常為 null。 |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |
| properties |描述事件詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。|
| properties.title | 描述資源健康情況狀態的使用者易記字串。 |
| properties.details | 進一步描述事件詳細資料的使用者易記字串。 |
| properties.currentHealthStatus | 資源的目前健康情況狀態。 下列其中一個值：「可用」、「無法使用」、「已降級」與「未知」。 |
| properties.previousHealthStatus | 資源先前的健康情況狀態。 下列其中一個值：「可用」、「無法使用」、「已降級」與「未知」。 |
| properties.type | 描述資源健康情況事件的類型。 |
| properties.cause | 描述資源健康情況事件的原因。 "UserInitiated" 或 "PlatformInitiated"。 |


## <a name="alert"></a>警示
此類別包含所有 Azure 警示的啟用記錄。 您可能會在此類別中看到的事件類型範例為「myVM 上的 CPU 百分比在過去 5 分鐘內已超過 80」。 各種 Azure 系統都有警示概念，您可以定義某種類型的規則，並在條件符合該規則時接收通知。 每次支援的 Azure 警示類型「啟動」時，或產生通知的條件符合時，該啟用記錄會也會推送至此類別的活動記錄。

### <a name="sample-event"></a>範例事件

```json
{
  "caller": "Microsoft.Insights/alertRules",
  "channels": "Admin, Operation",
  "claims": {
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/alertRules"
  },
  "correlationId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert/incidents/L3N1YnNjcmlwdGlvbnMvZGY2MDJjOWMtN2FhMC00MDdkLWE2ZmItZWIyMGM4YmQxMTkyL3Jlc291cmNlR3JvdXBzL0NzbUV2ZW50RE9HRk9PRC1XZXN0VVMvcHJvdmlkZXJzL21pY3Jvc29mdC5pbnNpZ2h0cy9hbGVydHJ1bGVzL215YWxlcnQwNjM2MzYyMjU4NTM1MjIxOTIw",
  "description": "'Disk read LessThan 100000 ([Count]) in the last 5 minutes' has been resolved for CloudService: myResourceGroup/Production/Event.BackgroundJobsWorker.razzle (myResourceGroup)",
  "eventDataId": "149d4baf-53dc-4cf4-9e29-17de37405cd9",
  "eventName": {
    "value": "Alert",
    "localizedValue": "Alert"
  },
  "category": {
    "value": "Alert",
    "localizedValue": "Alert"
  },
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/Event.BackgroundJobsWorker.razzle/events/149d4baf-53dc-4cf4-9e29-17de37405cd9/ticks/636362258535221920",
  "level": "Informational",
  "resourceGroupName": "myResourceGroup",
  "resourceProviderName": {
    "value": "Microsoft.ClassicCompute",
    "localizedValue": "Microsoft.ClassicCompute"
  },
  "resourceId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/Event.BackgroundJobsWorker.razzle",
  "resourceType": {
    "value": "Microsoft.ClassicCompute/domainNames/slots/roles",
    "localizedValue": "Microsoft.ClassicCompute/domainNames/slots/roles"
  },
  "operationId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert/incidents/L3N1YnNjcmlwdGlvbnMvZGY2MDJjOWMtN2FhMC00MDdkLWE2ZmItZWIyMGM4YmQxMTkyL3Jlc291cmNlR3JvdXBzL0NzbUV2ZW50RE9HRk9PRC1XZXN0VVMvcHJvdmlkZXJzL21pY3Jvc29mdC5pbnNpZ2h0cy9hbGVydHJ1bGVzL215YWxlcnQwNjM2MzYyMjU4NTM1MjIxOTIw",
  "operationName": {
    "value": "Microsoft.Insights/AlertRules/Resolved/Action",
    "localizedValue": "Microsoft.Insights/AlertRules/Resolved/Action"
  },
  "properties": {
    "RuleUri": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert",
    "RuleName": "myalert",
    "RuleDescription": "",
    "Threshold": "100000",
    "WindowSizeInMinutes": "5",
    "Aggregation": "Average",
    "Operator": "LessThan",
    "MetricName": "Disk read",
    "MetricUnit": "Count"
  },
  "status": {
    "value": "Resolved",
    "localizedValue": "Resolved"
  },
  "subStatus": {
    "value": null
  },
  "eventTimestamp": "2017-07-21T09:24:13.522192Z",
  "submissionTimestamp": "2017-07-21T09:24:15.6578651Z",
  "subscriptionId": "<subscription ID>"
}
```

### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| caller | 一律是 Microsoft.Insights/alertRules |
| channels | 一律是 “Admin, Operation” |
| claims | 具有警示引擎的 SPN (服務主體名稱) 或資源類型的 JSON Blob。 |
| correlationId | 字串格式的 GUID。 |
| description |警示事件的靜態文字描述。 |
| eventDataId |警示事件的唯一識別碼。 |
| level |事件的層級。 下列其中一個值：“Critical”、“Error”、“Warning” 與 “Informational” |
| resourceGroupName |如果為計量警示，這是受影響資源的資源群組名稱。 針對其他警示類型，這是包含警示本身的資源群組名稱。 |
| resourceProviderName |如果為計量警示，這是受影響資源的資源提供者名稱。 針對其他警示類型，這是警示本身的資源提供者名稱。 |
| resourceId | 如果為計量警示，這是受影響資源的資源識別碼名稱。 針對其他警示類型，這是警示資源本身的資源識別碼。 |
| operationId |對應至單一作業的事件共用的 GUID。 |
| operationName |作業名稱。 |
| properties |描述事件詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。 |
| status |字串，描述作業的狀態。 常見的值包括︰Started、In Progress、Succeeded、Failed、Active、Resolved。 |
| 子狀態 | 針對警示通常為 null。 |
| eventTimestamp |處理與事件對應之要求的Azure 服務產生事件時的時間戳記。 |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |

### <a name="properties-field-per-alert-type"></a>每個警示類型的屬性欄位
屬性欄位將依據警示事件來源包含不同的值。 兩個常見的警示事件提供者為活動記錄警示和計量警示。

#### <a name="properties-for-activity-log-alerts"></a>活動記錄警示的屬性
| 元素名稱 | 說明 |
| --- | --- |
| properties.subscriptionId | 導致啟用此活動記錄警示規則之活動記錄事件的訂用帳戶識別碼。 |
| properties.eventDataId | 導致啟用此活動記錄警示規則之活動記錄事件的事件資料識別碼。 |
| properties.resourceGroup | 導致啟用此活動記錄警示規則之活動記錄事件的資源群組。 |
| properties.resourceId | 導致啟用此活動記錄警示規則之活動記錄事件的資源識別碼。 |
| properties.eventTimestamp | 導致啟用此活動記錄警示規則之活動記錄事件的事件時間戳記。 |
| properties.operationName | 導致啟用此活動記錄警示規則之活動記錄事件的作業名稱。 |
| properties.status | 導致啟用此活動記錄警示規則之活動記錄事件的狀態。|

#### <a name="properties-for-metric-alerts"></a>計量警示屬性
| 元素名稱 | 說明 |
| --- | --- |
| properties.RuleUri | 計量警示規則本身的資源識別碼。 |
| properties.RuleName | 計量警示規則的名稱。 |
| properties.RuleDescription | 計量警示規則的描述 (如警示規則中所定義)。 |
| properties.Threshold | 用於評估計量警示規則的臨界值。 |
| properties.WindowSizeInMinutes | 用於評估計量警示規則的視窗大小。 |
| properties.Aggregation | 定義於計量警示規則中的彙總類型。 |
| properties.Operator | 用於評估計量警示規則的條件運算子。 |
| properties.MetricName | 用於評估計量警示規則之計量的計量名稱。 |
| properties.MetricUnit | 用於評估計量警示規則之計量的計量單位。 |

## <a name="autoscale"></a>Autoscale
所有與自動調整引擎 (以訂用帳戶中定義的自動調整設定為基礎) 作業相關的所有事件皆記錄在此類別。 您可能會在此類別中看到的事件類型範例為「自動調整相應增加動作失敗」。 自動調整可讓您使用自動調整設定，依據每日時間和/或負載 (計量) 資料，自動相應放大或縮小受支援資源類型中的執行個體數目。 符合相應增加或相應減少的條件時，啟動及成功或失敗事件將會記錄在此類別中。

### <a name="sample-event"></a>範例事件
```json
{
  "caller": "Microsoft.Insights/autoscaleSettings",
  "channels": "Admin, Operation",
  "claims": {
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/autoscaleSettings"
  },
  "correlationId": "fc6a7ff5-ff68-4bb7-81b4-3629212d03d0",
  "description": "The autoscale engine attempting to scale resource '/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource' from 3 instances count to 2 instances count.",
  "eventDataId": "a5b92075-1de9-42f1-b52e-6f3e4945a7c7",
  "eventName": {
    "value": "AutoscaleAction",
    "localizedValue": "AutoscaleAction"
  },
  "category": {
    "value": "Autoscale",
    "localizedValue": "Autoscale"
  },
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/autoscalesettings/myResourceGroup-Production-myResource-myResourceGroup/events/a5b92075-1de9-42f1-b52e-6f3e4945a7c7/ticks/636361956518681572",
  "level": "Informational",
  "resourceGroupName": "myResourceGroup",
  "resourceProviderName": {
    "value": "microsoft.insights",
    "localizedValue": "microsoft.insights"
  },
  "resourceId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/autoscalesettings/myResourceGroup-Production-myResource-myResourceGroup",
  "resourceType": {
    "value": "microsoft.insights/autoscalesettings",
    "localizedValue": "microsoft.insights/autoscalesettings"
  },
  "operationId": "fc6a7ff5-ff68-4bb7-81b4-3629212d03d0",
  "operationName": {
    "value": "Microsoft.Insights/AutoscaleSettings/Scaledown/Action",
    "localizedValue": "Microsoft.Insights/AutoscaleSettings/Scaledown/Action"
  },
  "properties": {
    "Description": "The autoscale engine attempting to scale resource '/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource' from 3 instances count to 2 instances count.",
    "ResourceName": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource",
    "OldInstancesCount": "3",
    "NewInstancesCount": "2",
    "LastScaleActionTime": "Fri, 21 Jul 2017 01:00:51 GMT"
  },
  "status": {
    "value": "Succeeded",
    "localizedValue": "Succeeded"
  },
  "subStatus": {
    "value": null
  },
  "eventTimestamp": "2017-07-21T01:00:51.8681572Z",
  "submissionTimestamp": "2017-07-21T01:00:52.3008754Z",
  "subscriptionId": "<subscription ID>"
}

```

### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| 呼叫者 | 一律是 Microsoft.Insights/autoscaleSettings |
| 通道 | 一律是 “Admin, Operation” |
| claims | 具有自動調整引擎的 SPN (服務主體名稱) 或資源類型的 JSON Blob。 |
| correlationId | 字串格式的 GUID。 |
| 說明 |自動調整事件的靜態文字描述。 |
| eventDataId |自動調整事件的唯一識別碼。 |
| 層級 |事件的層級。 下列其中一個值：“Critical”、“Error”、“Warning” 和 “Informational” |
| resourceGroupName |自動調整設定的資源群組名稱。 |
| resourceProviderName |自動調整設定的資源提供者名稱。 |
| ResourceId |自動調整設定的資源識別碼。 |
| operationId |對應至單一作業的事件共用的 GUID。 |
| operationName |作業名稱。 |
| properties |描述事件詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。 |
| properties.Description | 自動調整引擎所執行之作業的詳細描述。 |
| properties.ResourceName | 受影響資源 (執行調整動作的資源) 的資源識別碼 |
| properties.OldInstancesCount | 自動調整動作生效之前的執行個體數目。 |
| properties.NewInstancesCount | 自動調整動作生效之後的執行個體數目。 |
| properties.LastScaleActionTime | 自動調整動作發生時的時間戳記。 |
| status |字串，描述作業的狀態。 常見的值包括︰Started、In Progress、Succeeded、Failed、Active、Resolved。 |
| 子狀態 | 針對自動調整通常為 null。 |
| eventTimestamp |處理與事件對應之要求的Azure 服務產生事件時的時間戳記。 |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |

## <a name="security"></a>安全性
此類別包含「Azure 資訊安全中心」所產生之任何警示的記錄。 其中一個您會在此類別中看到的事件類型範例為「執行疑似雙重附檔名的檔案」。

### <a name="sample-event"></a>範例事件
```json
{
    "channels": "Operation",
    "correlationId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "description": "Suspicious double extension file executed. Machine logs indicate an execution of a process with a suspicious double extension.\r\nThis extension may trick users into thinking files are safe to be opened and might indicate the presence of malware on the system.",
    "eventDataId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "eventName": {
        "value": "Suspicious double extension file executed",
        "localizedValue": "Suspicious double extension file executed"
    },
    "category": {
        "value": "Security",
        "localizedValue": "Security"
    },
    "eventTimestamp": "2017-10-18T06:02:18.6179339Z",
    "id": "/subscriptions/<subscription ID>/providers/Microsoft.Security/locations/centralus/alerts/965d6c6a-a790-4a7e-8e9a-41771b3fbc38/events/965d6c6a-a790-4a7e-8e9a-41771b3fbc38/ticks/636439033386179339",
    "level": "Informational",
    "operationId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "operationName": {
        "value": "Microsoft.Security/locations/alerts/activate/action",
        "localizedValue": "Microsoft.Security/locations/alerts/activate/action"
    },
    "resourceGroupName": "myResourceGroup",
    "resourceProviderName": {
        "value": "Microsoft.Security",
        "localizedValue": "Microsoft.Security"
    },
    "resourceType": {
        "value": "Microsoft.Security/locations/alerts",
        "localizedValue": "Microsoft.Security/locations/alerts"
    },
    "resourceId": "/subscriptions/<subscription ID>/providers/Microsoft.Security/locations/centralus/alerts/2518939942613820660_a48f8653-3fc6-4166-9f19-914f030a13d3",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": null
    },
    "submissionTimestamp": "2017-10-18T06:02:52.2176969Z",
    "subscriptionId": "<subscription ID>",
    "properties": {
        "accountLogonId": "0x2r4",
        "commandLine": "c:\\mydirectory\\doubleetension.pdf.exe",
        "domainName": "hpc",
        "parentProcess": "unknown",
        "parentProcess id": "0",
        "processId": "6988",
        "processName": "c:\\mydirectory\\doubleetension.pdf.exe",
        "userName": "myUser",
        "UserSID": "S-3-2-12",
        "ActionTaken": "Detected",
        "Severity": "High"
    },
    "relatedEvents": []
}

```

### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| 通道 | 一律為 “Operation” |
| correlationId | 字串格式的 GUID。 |
| 說明 |安全性事件的靜態文字描述。 |
| eventDataId |安全性事件的唯一識別碼。 |
| eventName |安全性事件的易記名稱。 |
| id |安全性事件的唯一資源識別碼。 |
| 層級 |事件的層級。 下列其中一個值：“Critical”、“Error”、“Warning” 和 “Informational” |
| resourceGroupName |資源的資源群組名稱。 |
| resourceProviderName |「Azure 資訊安全中心」的資源提供者名稱。 一律為 "Microsoft.Security"。 |
| resourceType |產生安全性事件的資源類型，例如˙ "Microsoft.Security/locations/alerts" |
| ResourceId |安全性警示的資源識別碼。 |
| operationId |對應至單一作業的事件共用的 GUID。 |
| operationName |作業名稱。 |
| properties |描述事件詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。 這些屬性會依安全性警示類型的不同而有所不同。 如需來自「資訊安全中心」之警示類型的描述，請參閱[這個頁面](../security-center/security-center-alerts-type.md)。 |
| properties.Severity |嚴重性層級。 可能的值為："High"、"Medium" 或 "Low"。 |
| status |字串，描述作業的狀態。 常見的值包括︰Started、In Progress、Succeeded、Failed、Active、Resolved。 |
| 子狀態 | 針對安全性事件通常為 null。 |
| eventTimestamp |處理與事件對應之要求的Azure 服務產生事件時的時間戳記。 |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |

## <a name="recommendation"></a>建議
這個類別包含為您的服務產生的任何新建議記錄。 建議的範例像是「使用可用性設定組改善容錯。」 可產生的 4 種建議事件：高可用性、效能、安全性和成本最佳化。 

### <a name="sample-event"></a>範例事件
```json
{
    "channels": "Operation",
    "correlationId": "92481dfd-c5bf-4752-b0d6-0ecddaa64776",
    "description": "The action was successful.",
    "eventDataId": "06cb0e44-111b-47c7-a4f2-aa3ee320c9c5",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "Recommendation",
        "localizedValue": "Recommendation"
    },
    "eventTimestamp": "2018-06-07T21:30:42.976919Z",
    "id": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MYVM/events/06cb0e44-111b-47c7-a4f2-aa3ee320c9c5/ticks/636640038429769190",
    "level": "Informational",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Advisor/generateRecommendations/action",
        "localizedValue": "Microsoft.Advisor/generateRecommendations/action"
    },
    "resourceGroupName": "MYRESOURCEGROUP",
    "resourceProviderName": {
        "value": "MICROSOFT.COMPUTE",
        "localizedValue": "MICROSOFT.COMPUTE"
    },
    "resourceType": {
        "value": "MICROSOFT.COMPUTE/virtualmachines",
        "localizedValue": "MICROSOFT.COMPUTE/virtualmachines"
    },
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MYVM",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-06-07T21:30:42.976919Z",
    "subscriptionId": "<Subscription ID>",
    "properties": {
        "recommendationSchemaVersion": "1.0",
        "recommendationCategory": "Security",
        "recommendationImpact": "High",
        "recommendationRisk": "None"
    },
    "relatedEvents": []
}

```
### <a name="property-descriptions"></a>屬性描述
| 元素名稱 | 說明 |
| --- | --- |
| 通道 | 一律為 “Operation” |
| correlationId | 字串格式的 GUID。 |
| 說明 |建議事件的靜態文字描述 |
| eventDataId | 建議事件的唯一識別碼。 |
| category | 一律為 "Recommendation" |
| id |建議事件的唯一資源識別碼。 |
| 層級 |事件的層級。 下列其中一個值：“Critical”、“Error”、“Warning” 和 “Informational” |
| operationName |作業名稱。  一律為 "Microsoft.Advisor/generateRecommendations/action"|
| resourceGroupName |資源的資源群組名稱。 |
| resourceProviderName |此建議適用之資源的資源提供者名稱，例如 "MICROSOFT.COMPUTE" |
| resourceType |此建議適用之資源的資源類型名稱，例如 "MICROSOFT.COMPUTE/virtualmachines" |
| ResourceId |建議適用之資源的資源識別碼 |
| status | 一律為 "Active" |
| submissionTimestamp |當事件變成可供查詢時的時間戳記。 |
| subscriptionId |Azure 訂用帳戶識別碼。 |
| properties |描述建議詳細資料的一組 `<Key, Value>` 配對 (也就是字典)。|
| properties.recommendationSchemaVersion| 活動記錄項目中發行的建議屬性結構描述版本 |
| properties.recommendationCategory | 建議的分類。 可能的值為 "High Availability"、"Performance"、"Security" 和 "Cost" |
| properties.recommendationImpact| 建議的影響。 可能的值為："High"、"Medium"、"Low" |
| properties.recommendationRisk| 建議的風險。 可能的值為 "Error"、"Warning"、"None" |

## <a name="mapping-to-diagnostic-logs-schema"></a>診斷記錄結構描述的對應

將 Azure 活動記錄串流處理至儲存體帳戶或事件中樞命名空間時，資料會遵循 [Azure 診斷記錄結構描述](./monitoring-diagnostic-logs-schema.md)。 以下是上述結構描述的屬性與診斷記錄結構描述的對應：

| 診斷記錄結構描述屬性 | 活動記錄 REST API 結構描述屬性 | 注意 |
| --- | --- | --- |
| 分析 | eventTimestamp |  |
| ResourceId | ResourceId | subscriptionId、resourceType、resourceGroupName 全都推斷自 resourceId。 |
| operationName | operationName.value |  |
| category | 作業名稱部分 | 作業類型分類："Write"/"Delete"/"Action" |
| resultType | status.value | |
| resultSignature | substatus.value | |
| resultDescription | 說明 |  |
| durationMs | N/A | 一律為 0 |
| callerIpAddress | httpRequest.clientIpAddress |  |
| correlationId | correlationId |  |
| 身分識別 | 宣告和授權屬性 |  |
| Level | Level |  |
| location | N/A | 處理事件所在的位置。 *這不是資源的位置，而是處理事件的位置。此屬性將會在之後的更新中移除。* |
| properties | properties.eventProperties |  |
| properties.eventCategory | category | 如果 properties.eventCategory 不存在，則類別為 "Administrative" |
| properties.eventName | eventName |  |
| properties.operationId | operationId |  |
| properties.eventProperties | properties |  |


## <a name="next-steps"></a>後續步驟
* [深入了解活動記錄檔 (之前的稽核記錄檔)](monitoring-overview-activity-logs.md)
* [將 Azure 活動記錄檔串流至事件中樞](monitoring-stream-activity-logs-event-hubs.md)
