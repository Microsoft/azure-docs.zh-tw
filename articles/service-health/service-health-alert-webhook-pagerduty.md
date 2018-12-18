---
title: 使用 PagerDuty 設定 Azure 服務健康情況警示 | Microsoft Docs
description: 取得關於 PagerDuty 執行個體服務健康情況事件的個人化通知。
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
ms.date: 11/14/2017
ms.author: shtabriz
ms.openlocfilehash: 5f6f3f61b5f7a06ac4056499edfb811780838cdc
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39441872"
---
# <a name="configure-service-health-alerts-with-pagerduty"></a>使用 PagerDuty 設定服務健康情況警示

本文會示範如何使用 Webhook 透過 PagerDuty 來設定 Azure 服務健康情況通知。 透過使用 [PagerDuty](https://www.pagerduty.com/) 的自訂 Microsoft Azure 整合類型，您可以毫不費力地將服務健康情況警示新增至新建或現有的 PagerDuty 服務。

## <a name="creating-a-service-health-integration-url-in-pagerduty"></a>在 PagerDuty 中建立服務健康情況的整合 URL
1.  請確定您已註冊並登入您的 [PagerDuty](https://www.pagerduty.com/) 帳戶。

1.  瀏覽至 PagerDuty 的 [服務] 區段。

    ![在 PagerDuty 中的 [服務] 區段](./media/webhook-alerts/pagerduty-services-section.png)

1.  選取 [新增服務] 或開啟已設定的現有服務。

1.  在 [整合設定] 中選取下列項目：

    a. **整合類型**：Microsoft Azure

    b. **整合名稱**：\<名稱\>

    ![PagerDuty 中的 [整合設定]](./media/webhook-alerts/pagerduty-integration-settings.png)

1.  填寫所有必要欄位，然後選取 [新增]。

1.  開啟這個新的整合，並複製及儲存**整合 URL**。

    ![PagerDuty 中的「整合 URL」](./media/webhook-alerts/pagerduty-integration-url.png)

## <a name="create-an-alert-using-pagerduty-in-the-azure-portal"></a>在 Azure 入口網站使用 PagerDuty 建立警示
### <a name="for-a-new-action-group"></a>新的動作群組：
1. 遵循[使用 Azure 入口網站為新動作群組建立服務健康情況通知的警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)中的步驟 1 到步驟 8。

1. 在**動作**清單中定義：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 PagerDuty **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 以建立警示。

### <a name="for-an-existing-action-group"></a>現有的動作群組：
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取 [監視]。

1. 在 [設定] 區段上，選取 [動作群組]。

1. 尋找並選取您要編輯的動作群組。

1. 新增至**動作**清單：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 PagerDuty **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 來更新動作群組。

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>透過 HTTP POST 要求測試 Webhook 整合
1. 建立您想要傳送的服務健康情況承載。 您可以在 [Azure 活動記錄警示的 Webhook](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md) 上，找到服務服務健康情況 Webhook 承載範例。

1. 建立 HTTP POST 要求，如下所示：

    ```
    POST        https://events.pagerduty.com/integration/<IntegrationKey>/enqueue

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. 您應該會收到 `202 Accepted` 與包含「事件識別碼」的訊息。

1. 移至 [PagerDuty](https://www.pagerduty.com/)，以確認您的整合已設定成功。

## <a name="next-steps"></a>後續步驟
- 了解如何[設定現有問題管理系統的 Webhook 通知](service-health-alert-webhook-guide.md)。
- 檢閱[活動記錄警示 Webhook 結構描述](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)。 
- 深入了解[服務健康狀態通知](../monitoring-and-diagnostics/monitoring-service-notifications.md)。
- 深入了解[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)。