---
title: 設定傳統計量警示使用 Webhook 通知非 Azure 系統
description: 了解如何重設 Azure 計量警示的路徑到其他非 Azure 系統。
author: snehithm
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/03/2017
ms.author: snmuvva
ms.component: alerts
ms.openlocfilehash: 08ba5e7cbdc041a41f1d006d69980bf6efc00101
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44380281"
---
# <a name="have-a-classic-metric-alert-notify-a-non-azure-system-using-a-webhook"></a>設定傳統計量警示使用 Webhook 通知非 Azure 系統
您可以使用 Webhook 將 Azure 警示通知路由到其他系統以進行後處理或自訂動作。 您可以針對警示使用 Webhook，以將警示路由到會傳送簡訊、記錄錯誤、透過聊天或傳訊服務通知小組，或進行其他許多動作的服務。 

本文說明如何針對 Azure 計量警示設定 Webhook。 文中也會說明 HTTP POST 至 Webhook 的承載資料樣貌。 如需有關 Azure 活動記錄警示 (事件警示) 設定和結構描述的詳細資訊，請參閱[針對 Azure 活動記錄警示呼叫 Webhook](insights-auditlog-to-webhook-email.md)。

Azure 警示會使用 HTTP POST 將警示內容以 JSON 格式傳送到您在建立警示時提供的 Webhook URI。 稍後在本文中會定義結構描述。 該 URI 必須是有效的 HTTP 或 HTTPS 端點。 當警示啟動時，Azure 會針對每個要求張貼一個項目。

## <a name="configure-webhooks-via-the-azure-portal"></a>透過 Azure 入口網站設定 Webhook
若要在 [Azure 入口網站](https://portal.azure.com/)中新增或更新 Webhook URI，請移至 [建立/更新警示]。

![新增警示規則窗格](./media/insights-webhooks-alerts/Alertwebhook.png)

您也可以使用 [Azure PowerShell Cmdlet](insights-powershell-samples.md#create-metric-alerts)、[跨平台 CLI](insights-cli-samples.md#work-with-alerts) 或 [Azure 監視器 REST API](https://msdn.microsoft.com/library/azure/dn933805.aspx) \(英文\) 設定警示以張貼至 Webhook URI。

## <a name="authenticate-the-webhook"></a>驗證 Webhook
Webhook 可透過使用權杖型授權來驗證。 儲存的 Webhook URI 含有權杖識別碼。 例如：`https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`

## <a name="payload-schema"></a>承載結構描述
POST 作業對於所有以計量為基礎的警示會包含下列 JSON 承載和結構描述：

```JSON
{
    "status": "Activated",
    "context": {
        "timestamp": "2015-08-14T22:26:41.9975398Z",
        "id": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.insights/alertrules/ruleName1",
        "name": "ruleName1",
        "description": "some description",
        "conditionType": "Metric",
        "condition": {
            "metricName": "Requests",
            "metricUnit": "Count",
            "metricValue": "10",
            "threshold": "10",
            "windowSize": "15",
            "timeAggregation": "Average",
            "operator": "GreaterThanOrEqual"
        },
        "subscriptionId": "s1",
        "resourceGroupName": "useast",
        "resourceName": "mysite1",
        "resourceType": "microsoft.foo/sites",
        "resourceId": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1",
        "resourceRegion": "centralus",
        "portalLink": "https://portal.azure.com/#resource/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1"
    },
    "properties": {
        "key1": "value1",
        "key2": "value2"
    }
}
```


| 欄位 | 強制 | 一組固定值 | 注意 |
|:--- |:--- |:--- |:--- |
| status |Y |Activated、Resolved |以您設定的條件為基礎的警示狀態。 |
| context |Y | |警示內容。 |
| timestamp |Y | |警示觸發的時間。 |
| id |Y | |每個警示規則都有唯一的識別碼。 |
| name |Y | |警示名稱。 |
| 說明 |Y | |警示的描述。 |
| conditionType |Y |Metric、Event |支援兩種類型的警示：計量和事件。 以計量條件為基礎的計量警示。 以活動記錄中的事件為基礎的事件警示。 使用此值來檢查警示是以計量或事件為基礎。 |
| condition |Y | |要以 **conditionType** 值為基礎來檢查的特定欄位。 |
| metricName |用於計量警示 | |定義規則所監視的計量名稱。 |
| metricUnit |用於計量警示 |Bytes、BytesPerSecond、Count、CountPerSecond、Percent、Seconds |計量允許的單位。 請參閱[允許的值](https://msdn.microsoft.com/library/microsoft.azure.insights.models.unit.aspx)。 |
| metricValue |用於計量警示 | |造成警示的計量實際值。 |
| threshold |用於計量警示 | |會啟動警示的臨界值。 |
| windowSize |用於計量警示 | |以 threshold 為基礎用來監視警示活動的時間長度。 值必須介於 5 分鐘到 1 天之間。 值必須為 ISO 8601 持續時間格式。 |
| timeAggregation |用於計量警示 |Average、Last、Maximum、Minimum、None、Total |收集的資料應如何隨著時間結合。 預設值為 Average。 請參閱[允許的值](https://msdn.microsoft.com/library/microsoft.azure.insights.models.aggregationtype.aspx)。 |
| operator |用於計量警示 | |用來比較目前計量資料與所設定臨界值的運算子。 |
| subscriptionId |Y | |Azure 訂用帳戶識別碼。 |
| resourceGroupName |Y | |受影響資源的資源群組名稱。 |
| resourceName |Y | |受影響資源的資源名稱。 |
| resourceType |Y | |受影響資源的資源類型。 |
| ResourceId |Y | |受影響資源的資源識別碼。 |
| resourceRegion |Y | |受影響資源的區域或位置。 |
| portalLink |Y | |入口網站資源摘要頁面的直接連結。 |
| properties |N |選用 |一組索引鍵/值組，具有事件的詳細資料。 例如： `Dictionary<String, String>`。 properties 欄位是選擇性的。 在自訂 UI 或邏輯應用程式的工作流程中，使用者可以輸入可透過承載傳遞的索引鍵/值。 另一種將自訂屬性傳回給 Webhook 的替代方式是透過 Webhook URI 本身 (做為查詢參數)。 |

> [!NOTE]
> 您只能使用 [Azure 監視器 REST API](https://msdn.microsoft.com/library/azure/dn933805.aspx) \(英文\) 來設定 **properties**欄位。
>
>

## <a name="next-steps"></a>後續步驟
* 請觀賞 [使用 PagerDuty 整合 Azure 警示](http://go.microsoft.com/fwlink/?LinkId=627080) \(英文\) 影片以深入了解 Azure 警示與 Webhook。
* 深入了解如何[對 Azure 警示執行 Azure 自動化指令碼 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081) \(英文\)。
* 深入了解如何[使用邏輯應用程式透過 Twilio 從 Azure 警示傳送簡訊](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app) \(英文\)。
* 深入了解如何[使用邏輯應用程式從 Azure 警示傳送 Slack 訊息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app) \(英文\)。
* 深入了解如何[使用邏輯應用程式從 Azure 警示將訊息傳送到 Azure 佇列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app) \(英文\)。
