---
title: 使用 Webhook 設定現有問題管理系統的健康情況通知 | Microsoft Docs
description: 針對現有問題管理系統的相關服務健康情況事件取得個人化通知。
author: shawntabrizi
manager: scotthit
editor: ''
services: service-health
documentationcenter: service-health
ms.assetid: ''
ms.service: service-health
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/27/2018
ms.author: shtabriz
ms.openlocfilehash: 8535caf482b10912e6f7bc6df445756094d7603f
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/29/2018
ms.locfileid: "30261407"
---
# <a name="configure-health-notifications-for-existing-problem-management-systems-using-a-webhook"></a>使用 Webhook 設定現有問題管理系統的健康情況通知

本文會示範如何設定您的服務健康狀態警示，以便透過 Webhook 將資料傳送至您現有的通知系統。

現在，您可以設定服務健康狀態警示，以便在 Azure 服務事件影響您時取得通知，您可以透過簡訊或電子郵件收到通知。
不過，您可能已有想要使用的現有外部通知系統。
本文件會示範 Webhook 承載的最重要部分，以及如何建立自訂警示，以在服務問題影響您時收到通知。

如果您想要使用預先設定的整合，請參閱以下作法：
* [使用 ServiceNow 設定警示](service-health-alert-webhook-servicenow.md)
* [使用 PagerDuty 設定警示](service-health-alert-webhook-pagerduty.md)
* [使用 OpsGenie 設定警示](service-health-alert-webhook-opsgenie.md)

## <a name="configuring-a-custom-notification-using-the-service-health-webhook-payload"></a>使用服務健康情況 Webhook 承載來設定自訂通知
如果您想要設定自己的自訂 Webhook 整合，您必須剖析在服務健康狀態通知時傳送的 JSON 承載。

[請參閱此處的範例](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)，了解 `ServiceHealth` Webhook 承載看起來是什麼樣子。

您可以藉由在 `context.eventSource == "ServiceHealth"` 上查看，以發現到這是服務健康狀態警示。 在該處最相關的內嵌屬性是：
 * `data.context.activityLog.status`
 * `data.context.activityLog.level`
 * `data.context.activityLog.subscriptionId`
 * `data.context.activityLog.properties.title`
 * `data.context.activityLog.properties.impactStartTime`
 * `data.context.activityLog.properties.communication`
 * `data.context.activityLog.properties.impactedServices`
 * `data.context.activityLog.properties.trackingId`

## <a name="creating-a-direct-link-to-the-service-health-dashboard-for-an-incident"></a>針對事件建立 Azure 服務健康狀態儀表板的直接連結
您可以藉由產生特殊的 URL，在桌面或行動裝置上，建立 Azure 服務健康狀態儀表板的直接連結。 使用 `trackingId`，以及您 `subscriptionId` 的前三個和最後三個數字組成：
```
https://app.azure.com/h/<trackingId>/<first and last three digits of subscriptionId>
```

例如，如果您的 `subscriptionId` 是 `bba14129-e895-429b-8809-278e836ecdb3`，而您的 `trackingId` 是 `0DET-URB`，那麼您的 Azure 服務健康狀態 URL 就是：

```
https://app.azure.com/h/0DET-URB/bbadb3
```

## <a name="using-the-level-to-detect-the-severity-of-the-issue"></a>使用偵測問題嚴重性的層級
從嚴重性最低到最高，承載中的 `level` 屬性可以是 `Informational`、`Warning`、`Error`和 `Critical` 其中一個。

## <a name="parsing-the-impacted-services-to-understand-the-full-scope-of-the-incident"></a>剖析受影響的服務，以了解事件的完整範圍
Azure 服務健康狀態警示會通知您多個區域和服務的相關問題。 若要取得完整詳細資料，您必須剖析 `impactedServices` 的值。
其中的內容是 [JSON 逸出](http://json.org/) 字串，在未逸出時，則包含另一個可以定期剖析的 JSON 物件。

```json
{"data.context.activityLog.properties.impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"Australia East\"},{\"RegionName\":\"Australia Southeast\"}],\"ServiceName\":\"Alerts & Metrics\"},{\"ImpactedRegions\":[{\"RegionName\":\"Australia Southeast\"}],\"ServiceName\":\"App Service\"}]"}
```

變成：

```json
[
   {
      "ImpactedRegions":[
         {
            "RegionName":"Australia East"
         },
         {
            "RegionName":"Australia Southeast"
         }
      ],
      "ServiceName":"Alerts & Metrics"
   },
   {
      "ImpactedRegions":[
         {
            "RegionName":"Australia Southeast"
         }
      ],
      "ServiceName":"App Service"
   }
]
```

這表示在澳大利亞東部和東南部有「警示及度量」的問題，以及在澳大利亞東南部有 "App Service" 的問題。


## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>透過 HTTP POST 要求測試 Webhook 整合
1. 建立您想要傳送的服務健康狀態承載。 您可以在 [Azure 活動記錄警示的 Webhook](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md) 上，找到服務服務健康狀態 Webhook 承載範例。

2. 建立 HTTP POST 要求，如下所示：

    ```
    POST        https://your.webhook.endpoint

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
3. 您應該會收到 `2XX - Successful` 回應。

4. 移至 [PagerDuty](https://www.pagerduty.com/)，以確認您的整合已設定成功。

## <a name="next-steps"></a>後續步驟
- 檢閱[活動記錄警示 Webhook 結構描述](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)。 
- 深入了解[服務健康狀態通知](../monitoring-and-diagnostics/monitoring-service-notifications.md)。
- 深入了解[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)。