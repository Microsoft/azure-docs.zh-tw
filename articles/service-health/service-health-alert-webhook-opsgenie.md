---
title: 使用 OpsGenie 設定 Azure 服務健康情況警示 | Microsoft Docs
description: 取得關於 OpsGenie 執行個體服務健康情況事件的個人化通知。
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
ms.openlocfilehash: 6e9d44c4b64e41b04c40d0c511a638e539752da4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39435021"
---
# <a name="configure-service-health-alerts-with-opsgenie"></a>使用 OpsGenie 設定服務健康情況警示

本文會示範如何使用 Webhook 來設定 Azure 服務健康情況警示與 OpsGenie。 透過使用 [OpsGenie](https://www.opsgenie.com/) 的 Azure 服務健康情況整合，您可以將 Azure 服務健康情況警示轉送給 OpsGenie。 OpsGenie 可以依據值勤排程來決定要通知的合適人員，其方式是使用電子郵件、簡訊 (SMS)、電話、iOS 和 Android 推播通知及擴大警示，直到警示被接受或關閉。

## <a name="creating-a-service-health-integration-url-in-opsgenie"></a>在 OpsGenie 中建立服務健康情況的整合 URL
1.  請確定您已註冊並登入您的 [OpsGenie](https://www.opsgenie.com/) 帳戶。

1.  瀏覽至 OpsGenie 中的**整合**區段。

    ![OpsGenie 中的「整合」區段](./media/webhook-alerts/opsgenie-integrations-section.png)

1.  選取 [Azure 服務健康況] 整合按鈕。

    ![OpsGenie 中的 [Azure 服務健康情況] 按鈕](./media/webhook-alerts/opsgenie-azureservicehealth-button.png)

1.  為您的警示**命名**，並指定 [指派給小組] 欄位。

1.  填寫其他欄位，例如 [收件者]、[已啟用]和 [隱藏通知]。

1.  複製並儲存**整合 URL**，其應該已包含您的 `apiKey` (附加在結尾)。

    ![OpsGenie 中的「整合 URL」](./media/webhook-alerts/opsgenie-integration-url.png)

1.  選取 [儲存整合]

## <a name="create-an-alert-using-opsgenie-in-the-azure-portal"></a>在 Azure 入口網站使用 OpsGenie 建立警示
### <a name="for-a-new-action-group"></a>新的動作群組：
1. 遵循[使用 Azure 入口網站為新動作群組建立服務健康情況通知的警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)中的步驟 1 到步驟 8。

1. 在**動作**清單中定義：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 OpsGenie **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 以建立警示。

### <a name="for-an-existing-action-group"></a>現有的動作群組：
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取 [監視]。

1. 在 [設定] 區段上，選取 [動作群組]。

1. 尋找並選取您要編輯的動作群組。

1. 新增至**動作**清單：

    a. **動作類型：** *Webhook*

    b. **詳細資料：** 您先前儲存的 OpsGenie **整合 URL**。

    c. **名稱：** 的名稱、別名或識別項。

1. 完成後選取 [儲存] 來更新動作群組。

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>透過 HTTP POST 要求測試 Webhook 整合
1. 建立您想要傳送的服務健康情況承載。 您可以在 [Azure 活動記錄警示的 Webhook](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md) 上，找到服務服務健康情況 Webhook 承載範例。

1. 建立 HTTP POST 要求，如下所示：

    ```
    POST        https://api.opsgenie.com/v1/json/azureservicehealth?apiKey=<APIKEY>

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. 您應該會收到 `200 OK` 回應和狀態為「成功」的訊息。

1. 移至 [OpsGenie](https://www.opsgenie.com/)，以確認您的整合已設定成功。

## <a name="next-steps"></a>後續步驟
- 了解如何[設定現有問題管理系統的 Webhook 通知](service-health-alert-webhook-guide.md)。
- 檢閱[活動記錄警示 Webhook 結構描述](../monitoring-and-diagnostics/monitoring-activity-log-alerts-webhook.md)。 
- 深入了解[服務健康狀態通知](../monitoring-and-diagnostics/monitoring-service-notifications.md)。
- 深入了解[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)。