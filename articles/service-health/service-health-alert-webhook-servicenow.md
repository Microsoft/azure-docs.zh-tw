---
title: 使用 ServiceNow 設定 Azure 服務健康情況警示 | Microsoft Docs
description: 取得關於 ServiceNow 執行個體服務健康情況事件的個人化通知。
author: shawntabrizi
services: service-health
documentationcenter: service-health
ms.assetid: ''
ms.service: service-health
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/14/2017
ms.author: shtabriz
ms.openlocfilehash: 1f5984f8f28832c33d3a5a844fde72e7286ad251
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39433784"
---
# <a name="configure-service-health-alerts-with-servicenow"></a>使用 ServiceNow 設定服務健康情況警示

本文會示範如何使用 Webhook 來整合 Azure 服務健康情況警示與 ServiceNow。 設定好 Webhook 與 ServiceNow 執行個體的整合之後，當 Azure 服務的問題影響到您時，您就會收到警示 (透過現有的通知基礎結構)。 每次引發 Azure 服務健康情況警示時，它就會透過 ServiceNow 的指令碼式 REST API 呼叫 Webhook。

## <a name="creating-a-scripted-rest-api-in-servicenow"></a>在 ServiceNow 中建立指令碼式的 REST API
1.  請確定您已註冊並登入您的 [ServiceNow](https://www.servicenow.com/) 帳戶。

1.  瀏覽至 ServiceNow 中的**系統 Web 服務**區段，並選取**指令碼式 REST API**。

    ![ServiceNow 中的 [指令碼式 Web 服務] 區段](./media/webhook-alerts/servicenow-sws-section.png)

1.  選取 [新增] 來建立新的指令碼式 REST 服務。
 
    ![ServiceNow 中的 [新的指令碼式 REST API] 按鈕](./media/webhook-alerts/servicenow-new-button.png)

1.  為 REST API 新增**名稱**，並將 **API 識別碼**設定為 `azureservicehealth`。

1.  選取 [提交]。

    ![ServiceNow 中的「REST API 設定」](./media/webhook-alerts/servicenow-restapi-settings.png)

1.  選取您所建立的 REST API，並在 [資源] 索引標籤下選取 [新增]。

    ![ServiceNow 中的 [資源] 索引標籤](./media/webhook-alerts/servicenow-resources-tab.png)

1.  為您的新資源**命名**`event`，並將 **HTTP 方法**變更為 `POST`。

1.  在 [指令碼] 區段中，加入下列 JavaScript 程式碼：

    >[!NOTE]
    >您需要在下列指令碼中更新 `<secret>`、`<group>`和 `<email>` 的值。
    >* `<secret>` 應是隨機字串，就像 GUID
    >* `<group>` 應是您想要對其指派事件的 ServiceNow 群組
    >* `<email>` 應是您想要對其指派事件的人員 (選用)
    >

    ```javascript
    (function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
        var apiKey = request.queryParams['apiKey'];
        var secret = '<secret>';
        if (apiKey == secret) {
            var event = request.body.data;
            var responseBody = {};
            if (event.data.context.activityLog.operationName == 'Microsoft.ServiceHealth/incident/action') {
                var inc = new GlideRecord('incident');
                var incidentExists = false;
                inc.addQuery('number', event.data.context.activityLog.properties.trackingId);
                inc.query();
                if (inc.hasNext()) {
                    incidentExists = true;
                    inc.next();
                } else {
                    inc.initialize();
                }
                var short_description = "Azure Service Health";
                if (event.data.context.activityLog.properties.incidentType == "Incident") {
                    short_description += " - Service Issue - ";
                } else if (event.data.context.activityLog.properties.incidentType == "Maintenance") {
                    short_description += " - Planned Maintenance - ";
                } else if (event.data.context.activityLog.properties.incidentType == "Information" || event.data.context.activityLog.properties.incidentType == "ActionRequired") {
                    short_description += " - Health Advisory - ";
                }
                short_description += event.data.context.activityLog.properties.title;
                inc.short_description = short_description;
                inc.description = event.data.context.activityLog.properties.communication;
                inc.work_notes = "Impacted subscription: " + event.data.context.activityLog.subscriptionId;
                if (incidentExists) {
                    if (event.data.context.activityLog.properties.stage == 'Active') {
                        inc.state = 2;
                    } else if (event.data.context.activityLog.properties.stage == 'Resolved') {
                        inc.state = 6;
                    } else if (event.data.context.activityLog.properties.stage == 'Closed') {
                        inc.state = 7;
                    }
                    inc.update();
                    responseBody.message = "Incident updated.";
                } else {
                    inc.number = event.data.context.activityLog.properties.trackingId;
                    inc.state = 1;
                    inc.impact = 2;
                    inc.urgency = 2;
                    inc.priority = 2;
                    inc.assigned_to = '<email>';
                    inc.assignment_group.setDisplayValue('<group>');
                    var subscriptionId = event.data.context.activityLog.subscriptionId;
                    var comments = "Azure portal Link: https://app.azure.com/h";
                    comments += "/" + event.data.context.activityLog.properties.trackingId;
                    comments += "/" + subscriptionId.substring(0, 3) + subscriptionId.slice(-3);
                    var impactedServices = JSON.parse(event.data.context.activityLog.properties.impactedServices);
                    var impactedServicesFormatted = "";
                    for (var i = 0; i < impactedServices.length; i++) {
                        impactedServicesFormatted += impactedServices[i].ServiceName + ": ";
                        for (var j = 0; j < impactedServices[i].ImpactedRegions.length; j++) {
                            if (j != 0) {
                                impactedServicesFormatted += ", ";
                            }
                            impactedServicesFormatted += impactedServices[i].ImpactedRegions[j].RegionName;
                        }

                        impactedServicesFormatted += "\n";

                    }
                    comments += "\n\nImpacted Services:\n" + impactedServicesFormatted;
                    inc.comments = comments;
                    inc.insert();
                    responseBody.message = "Incident created.";
                }
            } else {
                responseBody.message = "Hello from the other side!";
            }
            response.setBody(responseBody);
        } else {
            var unauthorized = new sn_ws_err.ServiceError();
            unauthorized.setStatus(401);
            unauthorized.setMessage('Invalid apiKey');
            response.setError(unauthorized);
        }
    })(request, response);
    ```

1.  在 [安全性] 索引標籤中，取消核取 [需要驗證]，並選取 [提交]。 您設定的 `<secret>` 則會保護此 API。

    ![ServiceNow 中的 [需要驗證] 核取方塊](./media/webhook-alerts/servicenow-resource-settings.png)

1.  回到 [指令碼式 REST API] 區段，您應該尋找**基本 API 路徑**作為新的 REST API:

     ![ServiceNow 中的「基本 API 路徑」](./media/webhook-alerts/servicenow-base-api-path.png)

1.  完整的整合 URL 看起來會像：
        
         https://<yourInstanceName>.service-now.com/<baseApiPath>?apiKey=<secret>


## <a name="create-an-alert-using-servicenow-in-the-azure-portal"></a>在 Azure 入口網站使用 ServiceNow 建立警示
### <a name="for-a-new-action-group"></a>新的動作群組：
1. 請遵循[本文](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)中的步驟 1 到 8，來建立具有新動作群組的警示。

1. 在**動作**清單中定義：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 ServiceNow **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 以建立警示。

### <a name="for-an-existing-action-group"></a>現有的動作群組：
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取 [監視]。

1. 在 [設定] 區段上，選取 [動作群組]。

1. 尋找並選取您要編輯的動作群組。

1. 新增至**動作**清單：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 ServiceNow **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 來更新動作群組。

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>透過 HTTP POST 要求測試 Webhook 整合
1. 建立您想要傳送的服務健康情況承載。 您可以在 [Azure 活動記錄警示的 Webhook](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md) 上，找到服務服務健康情況 Webhook 承載範例。

1. 建立 HTTP POST 要求，如下所示：

    ```
    POST        https://<yourInstanceName>.service-now.com/<baseApiPath>?apiKey=<secret>

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. 您應該會收到 `200 OK` 回應和「事件已建立」的訊息。

1. 移至 [ServiceNow](https://www.servicenow.com/)，以確認您的整合已設定成功。

## <a name="next-steps"></a>後續步驟
- 了解如何[設定現有問題管理系統的 Webhook 通知](service-health-alert-webhook-guide.md)。
- 檢閱[活動記錄警示 Webhook 結構描述](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)。 
- 深入了解[服務健康狀態通知](../monitoring-and-diagnostics/monitoring-service-notifications.md)。
- 深入了解[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)。