---
title: 進行警示的根本原因分析 - Azure | Microsoft Docs
description: 在此教學課程中，您會了解如何使用「Azure 時間序列深入解析」來進行警示的根本原因分析。
author: aditidugar
ms.author: adugar
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 09/11/2018
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: 8258c8f34b4b9a1b216d9d497dcdf7d3b8db1373
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46369340"
---
# <a name="tutorial-conduct-a-root-cause-analysis-on-an-alert"></a>教學課程：進行警示的根本原因分析

在此教學課程中，您會了解如何使用「遠端監視」解決方案加速器來診斷警示的根本原因。 您看到「遠端監視」解決方案儀表板中已觸發某個警示，接著便使用「Azure 時間序列深入解析」總管來調查根本原因。

此教學課程會使用兩個會傳送位置、海拔、速度及貨物溫度遙測資料的模擬送貨卡車裝置。 卡車是由名為 Contoso 的組織所管理，且已連線到遠端監視解決方案加速器。 身為 Contoso 操作員，您必須了解為何其中一輛貨車 (delivery-truck-02) 已記錄低溫警示。

在此教學課程中，您：

>[!div class="checklist"]
> * 篩選儀表板中的裝置
> * 檢視即時遙測
> * 在時間序列深入解析總管中探索資料
> * 進行根本原因分析
> * 根據您已獲得的知識建立新規則

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [iot-accelerators-tutorial-prereqs](../../includes/iot-accelerators-tutorial-prereqs.md)]

## <a name="choose-the-devices-to-display"></a>選擇要顯示的裝置

若要選取要在 [儀表板] 頁面上顯示哪些已連線的裝置，請使用篩選條件。 若只要顯示**卡車**裝置，請在篩選條件下拉式清單中選擇內建 [卡車] 篩選條件：

[![儀表板上的卡車篩選條件](./media/iot-accelerators-remote-monitoring-root-cause-analysis/filter-trucks-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/filter-trucks-expanded.png#lightbox)

當您套用篩選條件時，只有符合篩選條件的裝置會顯示在 [儀表板] 頁面的地圖上和遙測面板中。 您可以看到有兩輛卡車連線至解決方案加速器，其中包括 **truck-02**。

## <a name="view-real-time-telemetry"></a>檢視即時遙測

解決方案加速器會在 [儀表板] 頁面的圖表中繪製即時遙測。 此圖表預設會顯示海拔遙測資料，此資料會隨時間改變：

[![卡車海拔遙測繪圖](./media/iot-accelerators-remote-monitoring-root-cause-analysis/trucks-moving-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/trucks-moving-expanded.png#lightbox)

若要檢視卡車的溫度遙測資料，請按一下 [遙測面板] 中的 [溫度]。 您可以看到兩輛卡車在過去 15 分鐘的溫度變化。 您也可以看到在警示窗格中已針對 delivery-truck-02 觸發低溫警示。

[![含有低溫警示的 RM 儀表板](./media/iot-accelerators-remote-monitoring-root-cause-analysis/low-temp-alert-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/low-temp-alert-expanded.png#lightbox)

## <a name="explore-the-data"></a>探索資料

若要識別低溫警示的原因，請在「時間序列深入解析」總管中開啟送貨卡車遙測資料。 按一下儀表板上的任何一個 [在時間序列深入解析中探索] 連結：

[![已醒目提示 TSI 連結的 RM 儀表板](./media/iot-accelerators-remote-monitoring-root-cause-analysis/explore-tsi-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/explore-tsi-expanded.png#lightbox)

當總管啟動時，您會看到列出所有裝置：

[![TSI 總管初始檢視](./media/iot-accelerators-remote-monitoring-root-cause-analysis/initial-tsi-view-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/initial-tsi-view-expanded.png#lightbox)

請在篩選方塊中輸入 **delivery-truck**，然後在左側面板中選取 [溫度] 作為 [量值]：

[![TSI 總管卡車溫度](./media/iot-accelerators-remote-monitoring-root-cause-analysis/filter-tsi-temp-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/filter-tsi-temp-expanded.png#lightbox)

您會看到與「遠端監視」儀表板中所見相同的檢視，而現在可以放大觸發警示的時間範圍：

[![TSI 總管縮放](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-zoom-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-zoom-expanded.png#lightbox)

您也可以加入來自卡車的其他遙測資料流。 請按一下左上角的 [新增] 按鈕。 新增窗格隨即出現：

[![含有新窗格的 TSI 總管](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-pane-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-pane-expanded.png#lightbox)

在新窗格中，將新標籤的名稱變更為 [裝置]，使其與先前的名稱相符。 選取 [海拔] 作為 [量值]，並選取 [iothub-connection-device-id] 作為 [分割依據] 值，以將海拔遙測資料新增至您的檢視：

[![含有溫度和海拔的 TSI 總管](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-altitude-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-altitude-expanded.png#lightbox)

## <a name="diagnose-the-alert"></a>診斷警示

當您查看目前檢視中的資料流時，可以看到兩輛卡車的海拔設定檔非常不同。 此外，當 **delivery-truck-02** 到達高海拔時，該卡車便發生溫度下降的情況。 您對該項發現感到訝異，因為這些卡車都排定成行駛相同路線。

為確認您對該卡車採用不同旅程路徑的懷疑，請使用 [新增] 按鈕將另一個窗格加入側邊面板中。 在新窗格中，將新標籤的名稱變更為 [裝置]，使其與先前的名稱相符。 選取 [經度] 作為 [量值]，並選取 [iothub-connection-device-id] 作為 [分割依據] 值，以將經度遙測資料新增至您的檢視。 您可以藉由查看**經度**資料流之間的差異，看出貨車是否確實採用不同的旅程：

[![含有溫度、海拔及經度的 TSI 總管](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-longitude-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/tsi-add-longitude-expanded.png#lightbox)

## <a name="create-a-new-rule"></a>建立新的規則

雖然卡車路線通常會事先經過最佳化，但您了解交通模式、天氣及其他無法預測的事件都可能造成延誤，因此讓卡車司機根據其最佳判斷來決定最後的路線。 不過，由於您資產在車輛內的溫度相當重要，因此您應該在「遠端監視」解決方案中建立額外的規則，以確保當平均海拔在 1 分鐘的間隔內上升超過 350 英呎時會收到警告：

[![「遠端監視」之規則索引標籤的設定海拔規則](./media/iot-accelerators-remote-monitoring-root-cause-analysis/new-rule-altitude-inline.png)](./media/iot-accelerators-remote-monitoring-root-cause-analysis/new-rule-altitude-expanded.png#lightbox)

若要了解如何建立和編輯規則，請查看先前有關[偵測裝置問題](iot-accelerators-remote-monitoring-automate.md)的教學課程。

[!INCLUDE [iot-accelerators-tutorial-cleanup](../../includes/iot-accelerators-tutorial-cleanup.md)]

## <a name="next-steps"></a>後續步驟

此教學課程已說明如何搭配「遠端監視」解決方案加速器使用「時間序列深入解析」總管來診斷警示的根本原因。 若要了解如何使用解決方案加速器來找出並修正已連線裝置的問題，請繼續進行下一個教學課程。

> [!div class="nextstepaction"]
> [使用裝置警示來找出並修正與監視解決方案連線的裝置問題](iot-accelerators-remote-monitoring-maintain.md)
