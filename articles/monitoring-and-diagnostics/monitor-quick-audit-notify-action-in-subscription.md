---
title: 稽核和接收 Azure 訂用帳戶中關於重要動作的通知
description: 了解活動記錄中之資源管理、服務健康狀況及其他訂用帳戶活動的歷程記錄，然後使用活動記錄警示，在您的訂用帳戶中執行高權限作業時接收電子郵件通知。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: quickstart
ms.date: 09/25/2017
ms.author: johnkem
ms.custom: mvc
ms.component: alerts
ms.openlocfilehash: 5a6f4d7ab978543a6871eb0ac3926fa25fb65ad2
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263325"
---
# <a name="audit-and-receive-notifications-about-important-actions-in-your-azure-subscription"></a>稽核和接收 Azure 訂用帳戶中關於重要動作的通知

**Azure 活動記錄**提供 Azure 中的訂用帳戶層級事件歷程記錄。 它提供的資訊是關於「何人」於「何時」建立、更新或刪除了「什麼」資源。 您可以建立**活動記錄警示**，當有活動符合您的警示條件時，就能收到電子郵件、簡訊或 Webhook 通知。 此快速入門會逐步引導您建立簡單的網路安全性群組、瀏覽活動記錄以了解發生的事件，然後撰寫活動記錄警示，以在之後建立任何網路安全性群組時收到通知。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-network-security-group"></a>建立網路安全性群組

1. 按一下 Azure 入口網站左上角的 [建立資源] 按鈕。

2. 選擇 [網路]，然後選取 [網路安全性群組]。

3. 輸入 myNetworkSG 作為 [名稱]，並建立名為 **myResourceGroup** 的新資源群組。 按一下 [ **建立** ] 按鈕。

    ![在入口網站中建立網路安全性群組](./media/monitor-quick-audit-notify-action-in-subscription/create-network-security-group.png)

## <a name="browse-the-activity-log-in-the-portal"></a>在入口網站中瀏覽活動記錄

活動記錄中已新增一個描述建立網路安全性群組的事件。 請依照下列指示來識別該事件。

1. 按一下左側瀏覽清單中的 [監視器] 按鈕。 它會開啟 [活動記錄] 區段。 此區段包含使用者對您訂用帳戶中的資源執行之所有動作的歷程記錄，可以數個屬性 (如 [資源群組]、[時間範圍] 和 [類別目錄]) 篩選。

2. 在 [活動記錄] 區段中，按一下 [資源群組] 下拉式清單，並選取 **myResourceGroup**。 將 [時間範圍] 下拉式清單變更為 [過去 1 小時]。 按一下 [套用]。

    ![篩選活動記錄](./media/monitor-quick-audit-notify-action-in-subscription/browse-activity-log.png)

3. 按一下所顯示的事件資料表中的 **Write NetworkSecurityGroups** 事件。

## <a name="browse-an-event-in-the-activity-log"></a>瀏覽活動記錄中的事件

顯示的區段包含已執行作業的基本詳細資料，包括名稱、時間戳記以及執行作業的使用者或應用程式。

![檢視活動記錄中的事件摘要](./media/monitor-quick-audit-notify-action-in-subscription/activity-log-summary.png)

按一下 [JSON] 索引標籤以檢視完整的事件詳細資料。 這包括以何種方式授權使用者或應用程式執行作業的詳細資料、事件類別目錄和層級，以及作業的狀態。

![檢視活動記錄中的事件詳細資料](./media/monitor-quick-audit-notify-action-in-subscription/activity-log-json.png)

## <a name="create-an-activity-log-alert"></a>建立活動記錄警示

1. 按一下 [摘要]索引標籤以返回事件摘要。

2. 在顯示的 [摘要] 區段中，按一下 [新增活動記錄警示]。

    ![在入口網站中建立網路安全性群組](./media/monitor-quick-audit-notify-action-in-subscription/activity-log-summary.png)

3. 在顯示的區段中，為活動記錄警示指定名稱和描述。

4. 在 [準則] 下方，確認 [事件類別目錄] 設為**管理**，[資源類型] 設為**網路安全性群組**，[作業名稱] 設為**建立或更新網路安全性群組**，[狀態] 設為**成功**，而其他所有的準則欄位則為空白或設為**所有**。 準則定義的規則，會決定活動記錄中出現新事件時是否應啟用警示。

    ![在入口網站中建立網路安全性群組](./media/monitor-quick-audit-notify-action-in-subscription/activity-log-alert-criteria.png)

5. 在 [透過下列方式發出警示] 下方，選取 [新增] 動作群組，並提供動作群組的 [名稱] 和 [簡短名稱]。 動作群組定義啟用警示時 (準則符合新事件時) 所採取的一組動作。

6. 在 [動作] 下方，藉由提供動作的 [名稱]、[動作類型] (例如，電子郵件、簡訊或 Webhook) 及該特定動作類型 (例如，Webhook URL、電子郵件地址或簡訊號碼) 的 [詳細資料]，以新增一或多個動作。

    ![在入口網站中建立網路安全性群組](./media/monitor-quick-audit-notify-action-in-subscription/activity-log-alert-actions.png)

7. 按一下 [確定] 以儲存活動記錄警示。

## <a name="test-the-activity-log-alert"></a>測試活動記錄警示

> [!NOTE]
> 活動記錄警示大約需要 5 分鐘才會變成完全啟用。 在活動記錄警示完全啟用前所發生的事件，將不會產生通知。
>
>

若要測試警示，請重複前面小節以**建立網路安全性群組**，但為此網路安全性群組指定不同的名稱，並重複使用現有的資源群組。 您將在幾分鐘內收到已建立網路安全性群組的通知。

## <a name="clean-up-resources"></a>清除資源

當資源群組與網路安全性群組不再需要時，請予以刪除。 若要這樣做，在入口網站最上方的 [搜尋] 方塊中輸入您所建立之資源群組的名稱，然後按一下該資源群組的名稱。 在顯示的區段中，按一下 [刪除資源群組] 按鈕，輸入資源群組的名稱，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在這個快速入門中，您執行作業以產生活動記錄事件，接著建立活動記錄警示，以在此作業日後再度發生時收到通知。 接著，您藉由再次執行該作業以測試警示。 Azure 可提供過去 90 天的活動記錄事件。 如果您要保留超過 90 天的事件，請嘗試連同其他監視資料一併封存活動記錄資料。

> [!div class="nextstepaction"]
> [封存監視資料](./monitor-tutorial-archive-monitoring-data.md)
