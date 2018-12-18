---
title: 搭配 Azure Logic Apps 進行 IoT 遠端監視和通知 | Microsoft Docs
description: 使用 Azure Logic Apps 進行 IoT 中樞的 IoT 溫度監視，並對於偵測到的任何異常自動將電子郵件通知寄送到您的信箱。
author: rangv
manager: ''
keywords: iot 監視、iot 通知、iot 溫度監視
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/11/2018
ms.author: rangv
ms.openlocfilehash: e59577e99114f1b2061a2f9075976da3f0b1811f
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34634449"
---
# <a name="iot-remote-monitoring-and-notifications-with-azure-logic-apps-connecting-your-iot-hub-and-mailbox"></a>搭配連接 IoT 中樞和信箱的 Azure Logic Apps 進行 IoT 遠端監視和通知

![端對端圖表](media/iot-hub-get-started-e2e-diagram/7.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

Azure Logic Apps 可用來以一連串的步驟使程序自動進行。 邏輯應用程式可以連接多種服務和通訊協定。 一開始會有觸發程序，例如「新增帳戶時」，後面接著一連串動作，例如「傳送推播通知」。 此功能可讓 Logic Apps 成為 IoT 監視的絕佳 IoT 解決方案，例如注意異常行為，以及其他使用情況。

## <a name="what-you-learn"></a>您學到什麼

您學會如何建立連接 IoT 中樞和信箱的邏輯應用程式用於溫度監視和通知。 溫度超過 30°C 時，用戶端應用程式會在傳送到 IoT 中樞的訊息中標示 `temperatureAlert = "true"`。 該訊息會觸發邏輯應用程式傳送電子郵件通知給您。

## <a name="what-you-do"></a>您要做什麼

* 建立服務匯流排命名空間，並將佇列加入到服務匯流排命名空間。
* 將端點和路由規則加入到 IoT 中樞。
* 建立、設定和測試邏輯應用程式。

## <a name="what-you-need"></a>您需要什麼

* 完成涵蓋下列需求的[設定裝置](iot-hub-raspberry-pi-kit-node-get-started.md)教學課程︰
  * 有效的 Azure 訂用帳戶。
  * 位於您訂用帳戶中的 Azure IoT 中樞。
  * 將訊息傳送到您 Azure IoT 中樞的用戶端應用程式。

## <a name="create-service-bus-namespace-and-add-a-queue-to-it"></a>建立服務匯流排命名空間，並將佇列加入到服務匯流排命名空間

### <a name="create-a-service-bus-namespace"></a>建立服務匯流排命名空間

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [建立資源] > [企業整合] > [服務匯流排]。
1. 請提供下列資訊：

   **名稱**︰服務匯流排的名稱。

   **定價層**︰ 按一下 [基本] > [選取]。 「基本」層足供本教學課程之用。

   **資源群組**︰使用 IoT 中樞所用的相同資源群組。

   **位置**：使用 IoT 中樞使用的同一個位置。
1. 按一下頁面底部的 [新增] 。

   ![在 Azure 入口網站中建立服務匯流排命名空間](media/iot-hub-monitoring-notifications-with-azure-logic-apps/1_create-service-bus-namespace-azure-portal.png)

### <a name="add-a-service-bus-queue"></a>新增服務匯流排佇列

1. 開啟服務匯流排命名空間，然後按一下 [+ 佇列]。
1. 輸入佇列的名稱，然後按一下 [建立]。
1. 開啟服務匯流排佇列，然後按一下 [共用存取原則] > [+ 新增]。
1. 輸入原則名稱，並勾選 [管理]，然後按一下 [建立]。

   ![在 Azure 入口網站中新增服務匯流排佇列](media/iot-hub-monitoring-notifications-with-azure-logic-apps/2_add-service-bus-queue-azure-portal.png)

## <a name="add-an-endpoint-and-a-routing-rule-to-your-iot-hub"></a>將端點和路由規則新增到 IoT 中樞

### <a name="add-an-endpoint"></a>新增端點。

1. 開啟 IoT 中樞，並按一下 [端點] > [+ 新增]。
1. 輸入以下資訊：

   **名稱**：端點的名稱。

   **端點類型**：選取**服務匯流排佇列**。

   **服務匯流排命名空間**：選取您建立的命名空間。

   **服務匯流排佇列**：選取您建立的佇列。
1. 按一下 [確定]。

   ![將端點新增至 Azure 入口網站的 IoT 中樞](media/iot-hub-monitoring-notifications-with-azure-logic-apps/3_add-iot-hub-endpoint-azure-portal.png)

### <a name="add-a-routing-rule"></a>新增路由規則

1. 在 IoT 中樞，按一下 [路由] > [+ 新增]。
1. 輸入以下資訊：

   **名稱**：路由規則的名稱。

   **資料來源**：選取 [裝置訊息]。

   **端點**：選取您建立的端點。

   **查詢字串**：輸入 `temperatureAlert = "true"`。
1. 按一下 [檔案] 。

   ![在 Azure 入口網站中新增路由規格](media/iot-hub-monitoring-notifications-with-azure-logic-apps/4_add-routing-rule-azure-portal.png)

## <a name="create-and-configure-a-logic-app"></a>建立並設定邏輯應用程式

### <a name="create-a-logic-app"></a>建立邏輯應用程式

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [建立資源] > [企業整合] > [邏輯應用程式]。
1. 輸入以下資訊：

   **名稱**︰ 邏輯應用程式的名稱。

   **資源群組**︰使用 IoT 中樞所用的相同資源群組。

   **位置**：使用 IoT 中樞使用的同一個位置。
1. 按一下頁面底部的 [新增] 。

### <a name="configure-the-logic-app"></a>設定邏輯應用程式

1. 開啟在邏輯應用程式設計工具中開啟的邏輯應用程式。
1. 在邏輯應用程式設計工具中，按一下 [空白邏輯應用程式]。

   ![在 Azure 入口網站中，從空白的邏輯應用程式開始。](media/iot-hub-monitoring-notifications-with-azure-logic-apps/5_start-with-blank-logic-app-azure-portal.png)

1. 按一下 [服務匯流排]。

   ![選取服務匯流排，開始在 Azure 入口網站中建立邏輯應用程式](media/iot-hub-monitoring-notifications-with-azure-logic-apps/6_select-service-bus-when-creating-blank-logic-app-azure-portal.png)

1. 按一下 [服務匯流排 – 一個或多個訊息到達佇列時 (自動完成)]。
1. 建立服務匯流排連接。
   1. 輸入連線名稱。
   1. 按一下服務匯流排命名空間 > 服務匯流排原則 > [建立]。

      ![在 Azure 入口網站中建立邏輯應用程式的服務匯流排連接](media/iot-hub-monitoring-notifications-with-azure-logic-apps/7_create-service-bus-connection-in-logic-app-azure-portal.png)

   1. 建立服務匯流排連接之後，按一下 [繼續]。
   1. 選取您建立的佇列，並對於 [最大訊息計數] 輸入 `175`

      ![對於邏輯應用程式中的服務匯流排連接指定最大訊息計數](media/iot-hub-monitoring-notifications-with-azure-logic-apps/8_specify-maximum-message-count-for-service-bus-connection-logic-app-azure-portal.png)
   1. 按一下 [儲存] 按鈕可儲存變更。

1. 建立 SMTP 服務連接。
   1. 按一下 [新增步驟] > [新增動作]。
   1. 輸入 `SMTP`，並按一下搜尋結果中的 **SMTP** 服務，然後按一下 [SMTP - 傳送電子郵件]。

      ![在 Azure 入口網站的邏輯應用程式中建立 SMTP 連接](media/iot-hub-monitoring-notifications-with-azure-logic-apps/9_create-smtp-connection-logic-app-azure-portal.png)

   1. 輸入您信箱的 SMTP 資訊，然後按一下 [建立]。

      ![在 Azure 入口網站的邏輯應用程式中輸入 SMTP 連接資訊](media/iot-hub-monitoring-notifications-with-azure-logic-apps/10_enter-smtp-connection-info-logic-app-azure-portal.png)

      取得 [Hotmail/Outlook.com](https://support.office.com/en-us/article/Add-your-Outlook-com-account-to-another-mail-app-73f3b178-0009-41ae-aab1-87b80fa94970)、[Gmail](https://support.google.com/a/answer/176600?hl=en) 和 [Yahoo Mail](https://help.yahoo.com/kb/SLN4075.html) 的 SMTP 資訊。
   1. 輸入**寄件者**和**收件者**的電子郵件地址，並對於**主旨**和**內文**輸入 `High temperature detected`。
   1. 按一下 [檔案] 。

邏輯應用程式隨即在您儲存時開始運作。

## <a name="test-the-logic-app"></a>測試邏輯應用程式

1. 在 [將 ESP8266 連接到 Azure IoT 中樞](iot-hub-arduino-huzzah-esp8266-get-started.md) 中，啟動您部署至裝置的用戶端應用程式。
1. 將 SensorTag 周圍的環境溫度增加到 30°C 以上。例如，在 SensorTag 周圍點蠟燭。
1. 您應該會收到邏輯應用程式傳送的電子郵件通知。

   > [!NOTE]
   > 您的電子郵件服務提供者可能需要驗證寄件者身分，確定是您傳送電子郵件。

## <a name="next-steps"></a>後續步驟

您已成功建立連接 IoT 中樞的邏輯應用程式，以及溫度監視和通知的信箱。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
