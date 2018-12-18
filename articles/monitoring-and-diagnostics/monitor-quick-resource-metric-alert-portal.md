---
title: 當計量值符合條件時收到通知
description: 協助使用者為邏輯應用程式建立計量的快速入門指南
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: quickstart
ms.date: 02/08/2018
ms.author: ancav
ms.custom: mvc
ms.component: alerts
ms.openlocfilehash: 8f3be2b1eb9c12aab212fb198283dce9ee3d3912
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982856"
---
# <a name="receive-a-notification-when-a-metric-value-meets-a-condition"></a>當計量值符合條件時收到通知

Azure 監視器提供適用於許多 Azure 資源的計量。 這些計量可表達這些資源的健康情況和效能。 在許多情況下，計量值可以指出資源發生問題。 您可以建立計量警示來監視異常行為，並於發生時收到通知。 此快速入門逐步引導您建立邏輯應用程式、建立工作，以及視覺化邏輯應用程式的計量。 接著會建立警示，並接收邏輯應用程式資源的計量通知。

如需計量和計量警示的詳細資訊，請參閱 [Azure 監視器計量概觀](./monitoring-overview-metrics.md)和 [Azure 監視器警示概觀](./monitoring-overview-alerts.md)。 

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-logic-app"></a>建立邏輯應用程式

1. 按一下 Azure 入口網站左上角的 [建立資源] 按鈕。

2. 搜尋並選取**邏輯應用程式**。 按一下 [ **建立** ] 按鈕。

3. 輸入名稱 myLogicApp 和資源群組 myResourceGroup。 使用您的訂用帳戶。  使用預設位置。 核取 [釘選到儀表板] 選項。  完成時，按一下 [建立]。 

    ![在入口網站中，輸入邏輯應用程式的基本資訊](./media/monitor-quick-resource-metric-alert-portal/create-logic-app-portal.png)  


4. 邏輯應用程式應該釘選到儀表板。 按一下以瀏覽至邏輯應用程式。

5. 在 [邏輯應用程式] 面板中，選取 [邏輯應用程式設計工具]

     ![在入口網站面板中，在邏輯應用程式設計工具中建立週期觸發程序](./media/monitor-quick-resource-metric-alert-portal/logic-app-designer.png)  

6. 設定您的值，如下圖所示。

    ![在入口網站面板中設定邏輯應用程式觸發程序](./media/monitor-quick-resource-metric-alert-portal/create-logic-app-triggers.png) 

7. 在設計工具中，選取 [週期] 觸發程序。

8. 設定間隔 20 和頻率 1 秒，以確保邏輯應用程式每隔 20 秒觸發一次。

9. 按一下 [新增步驟] 按鈕，然後選取 [新增動作]。

10. 選擇 [HTTP] 選項，然後選取 [HTTP-HTTP]。

11. 將 [方法] 設為 POST，將 [Uri] 設為您選擇的網址。

12. 按一下 [檔案] 。

13. 最多可能需要 5 分鐘才會發生邏輯應用程式執行動作。  

## <a name="view-metrics-for-your-logic-app"></a>檢視邏輯應用程式的計量

1. 在左側瀏覽窗格中，按一下 [監視器] 選項。

2. 選取 [計量] 索引標籤，填入邏輯應用程式的 [訂用帳戶]、[資源群組]、[資源類型] 和 [資源]資訊。

3. 從計量清單中，選擇 [執行失敗]。

4. 修改圖表的 [時間範圍] 以顯示過去一小時的資料。

5. 您現在應該會看到圖表繪出邏輯應用程式在過去一小時啟動執行的總次數。 如未看到任何內容，請確定從上一個步驟算起，您已等候至少 5 分鐘。 然後重新整理您的瀏覽器。 

    ![繪製邏輯應用程式資源的計量圖表](./media/monitor-quick-resource-metric-alert-portal/logic-app-metric-chart.png)

## <a name="create-a-metric-alert-for-your-logic-app"></a>建立邏輯應用程式的計量警示

1.  在計量面板的頂端部分，按一下 [新增計量警示] 按鈕。

2. 將計量警示命名為 'myLogicAppAlert'，並提供警示的簡短描述。

3. 將計量警示的 [條件] 設為「大於」，將 [閾值] 設為 '10'，將 [期間] 設為「過去 5 分鐘」。

4. 最後，在 [其他系統管理員電子郵件] 中輸入您的電子郵件地址。 萬一邏輯應用程式在 5 分鐘之內執行失敗超過 10 次，此警示可確保您會收到電子郵件。

    ![在入口網站面板中設定邏輯應用程式警示](./media/monitor-quick-resource-metric-alert-portal/logic-app-metrics-alert-portal.png)

## <a name="receive-metric-alert-notifications-for-your-logic-app"></a>接收邏輯應用程式的計量警示通知
1. 片刻之內，您應該會收到「Microsoft Azure 警示」的電子郵件，讓您知道警示「已啟動」。

2. 瀏覽回到邏輯應用程式，將週期觸發程序修改為間隔 1 和頻率 1 小時。

3. 幾分鐘之內，您應該會收到「Microsoft Azure 警示」的電子郵件，讓您知道警示「已解決」。

## <a name="clean-up-resources"></a>清除資源

此集合中的其他快速入門以本快速入門為基礎。 如果您打算繼續進行後續的快速入門或教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列步驟，在 Azure 入口網站中刪除本快速入門所建立的所有資源。

1. 從 Azure 入口網站的左側功能表，按一下 [監視器]。

2. 選取 [警示] 索引標籤，尋找您在本快速入門指南中建立的警示並按一下。

3. 在計量警示面板中，按一下 [刪除]。

4. 從 Azure 入口網站的左側功能表中，搜尋**邏輯應用程式**，然後按一下 [邏輯應用程式]。

5. 在面板上，在文字方塊中按一下您在本快速入門中建立的邏輯應用程式，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立資源的計量警示。 如需計量警示的詳細資訊，請瀏覽我們的警示概觀。

> [!div class="nextstepaction"]
> [Azure 監視器訂用帳戶動作警示](./monitor-quick-audit-notify-action-in-subscription.md )
