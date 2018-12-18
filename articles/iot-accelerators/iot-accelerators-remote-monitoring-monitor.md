---
title: 從 Azure 解決方案監視 IoT 裝置的教學課程 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用遠端監視解決方案加速器來監視 IoT 裝置。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 07/19/2018
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: 1f9e5885e79e184b621ba2be7e2a8f329e31a6b1
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39284484"
---
# <a name="tutorial-monitor-your-iot-devices"></a>教學課程：監視 IoT 裝置

在本教學課程中，您會使用遠端監視解決方案加速器來監視已連線的 IoT 裝置。 您可使用解決方案儀表板來檢視遙測、裝置資訊、警示和 KPI。

本教學課程會使用兩個傳送位置、速度和貨物溫度遙測資料的模擬卡車裝置。 卡車是由名為 Contoso 的組織所管理，且已連線到遠端監視解決方案加速器。 作為 Contoso 操作員，您必須在現場監視您其中一輛卡車 (truck-02) 的位置和行為。

在本教學課程中，您：

>[!div class="checklist"]
> * 篩選儀表板中的裝置
> * 檢視即時遙測
> * 檢視裝置詳細資料
> * 從您的裝置檢視警示
> * 檢視系統 KPI

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [iot-accelerators-tutorial-prereqs](../../includes/iot-accelerators-tutorial-prereqs.md)]

## <a name="choose-the-devices-to-display"></a>選擇要顯示的裝置

若要選取要在 [儀表板] 頁面上顯示哪些已連線的裝置，請使用篩選條件。 若只要顯示**卡車**裝置，請在篩選條件下拉式清單中選擇內建 [卡車] 篩選條件：

[![儀表板上的卡車篩選條件](./media/iot-accelerators-remote-monitoring-monitor/dashboardtruckfilter-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardtruckfilter-expanded.png#lightbox)

當您套用篩選條件時，只有符合篩選條件的裝置會顯示在 [儀表板] 頁面的地圖上和遙測面板中。 您可以看到，有兩輛卡車連線至解決方案加速器，其中包括 truck-02：

[![只有卡車會顯示在地圖上](./media/iot-accelerators-remote-monitoring-monitor/dashboardtruckmap-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardtruckmap-expanded.png#lightbox)

若要建立、編輯和刪除篩選條件，請按一下 [管理裝置群組]。

## <a name="view-real-time-telemetry"></a>檢視即時遙測

解決方案加速器會在 [儀表板] 頁面的圖表中繪製即時遙測。 遙測圖表的頂端會依目前的篩選條件顯示所選裝置的可用遙測類型，包括 truck-02。 根據預設，圖表會顯示卡車的緯度，而 truck-02 似乎是靜止的：

[![卡車遙測類型](./media/iot-accelerators-remote-monitoring-monitor/dashboardtelemetryview-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardtelemetryview-expanded.png#lightbox)

若要檢視卡車的溫度遙測資料，請按一下 [溫度]： 您可以看到 truck-02 的溫度在過去一小時有所變化：

[![卡車溫度遙測繪圖](./media/iot-accelerators-remote-monitoring-monitor/dashboardselecttelemetry-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardselecttelemetry-expanded.png#lightbox)

## <a name="view-the-map"></a>檢視地圖

地圖會顯示目前篩選條件所選取之模擬卡車的相關資訊。 您可以縮放及移動瀏覽地圖，以顯示更多或更少詳細資料的位置。 地圖上的裝置圖示顏色會指出裝置是否有任何有效的**警示** (深藍色) 或**警告** (紅色)。 **警示**和**警告**的數目摘要會顯示在地圖左邊。

若要檢視 truck-02 的詳細資料，請移動瀏覽及縮放地圖以找出卡車，然後在地圖上選取卡車。 接著按一下裝置標籤以開啟 [裝置詳細資料] 面板。 裝置詳細資料包括：

* 最近的遙測值
* 裝置支援的方法
* 裝置屬性

[![在儀表板上檢視裝置詳細資料](./media/iot-accelerators-remote-monitoring-monitor/dashboarddevicedetail-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboarddevicedetail-expanded.png#lightbox)

## <a name="view-alerts"></a>檢視警示

[警示] 面板會顯示與您裝置中的最新警示有關的詳細資訊。 來自 truck-02 的警示指出高於正常貨物溫度：

[![在儀表板上檢視裝置警示](./media/iot-accelerators-remote-monitoring-monitor/dashboardsystemalarms-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardsystemalarms-expanded.png#lightbox)

您可以使用篩選條件來調整最近警示的時間範圍。 根據預設，面板會顯示過去一小時內的警示。

## <a name="view-the-system-kpis"></a>檢視系統 KPI

[儀表板] 頁面會在 [Analytics] 面板中顯示解決方案加速器所計算的系統 KPI：

[![儀表板 KPI](./media/iot-accelerators-remote-monitoring-monitor/dashboardkpis-inline.png)](./media/iot-accelerators-remote-monitoring-monitor/dashboardkpis-expanded.png#lightbox)

此儀表板會針對目前裝置和時間範圍篩選條件所選取的警示顯示三個 KPI：

* 針對已觸發最多警示的規則顯示作用中警示數目。
* 依裝置類型顯示警示的比例。
* 重大警示的警示百分比。

針對 truck-02，所有警示都是高於正常貨物溫度的警告。

為警示設定時間範圍以及控制會顯示哪些裝置的相同篩選條件會決定 KPI 的彙總方式。 依預設，面板會顯示過去一小時內彙總的 KPI。

[!INCLUDE [iot-accelerators-tutorial-cleanup](../../includes/iot-accelerators-tutorial-cleanup.md)]

## <a name="next-steps"></a>後續步驟

本教學課程示範了如何使用遠端監視解決方案加速器中的 [儀表板] 頁面，來篩選和監視模擬的卡車。 若要了解如何使用解決方案加速器來偵測已連線裝置的問題，請繼續進行下一個教學課程。

> [!div class="nextstepaction"]
> [偵測與監視解決方案連線的裝置問題](iot-accelerators-remote-monitoring-automate.md)