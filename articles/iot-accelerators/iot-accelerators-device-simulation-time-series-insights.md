---
title: 使用 Azure 時間序列深入解析將裝置模擬遙測資料視覺化 | Microsoft Docs
description: 了解如何設定時間序列深入解析環境，來探索和分析裝置模擬解決方案加速器產生的的遙測資料。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.date: 08/20/2018
ms.topic: conceptual
ms.service: iot-accelerators
services: iot-accelerators
ms.openlocfilehash: 49cecca4e5ebef9f43fdda16cfa1fdc2ad7b6f94
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382994"
---
# <a name="use-time-series-insights-to-visualize-telemetry-sent-from-the-device-simulation-solution-accelerator"></a>使用時間序列深入分析將從裝置模擬解決方案加速器傳送的遙測資料視覺化

裝置模擬解決方案加速器可讓您從模擬裝置產生遙測資料，藉此測試您的 IoT 解決方案。 本操作指南會示範如何使用時間序列深入解析環境，來視覺化和分析模擬的遙測資料。

## <a name="prerequisites"></a>必要條件

若要遵循此操作指南中的步驟，您必須具備有效的 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

此操作指南中的步驟假設您已將裝置模擬解決方案加速器部署至您的 Azure 訂用帳戶。 如果您尚未部署此解決方案加速器，請先遵循[部署與執行雲端式裝置模擬解決方案](quickstart-device-simulation-deploy.md)快速入門中的步驟。

本文假設您解決方案加速器的名稱是 **contoso-simulation**。 完成下列步驟時，將 **contoso-simulation** 取代成為您解決方案加速器的名稱。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [iot-accelerators-create-tsi.md](../../includes/iot-accelerators-create-tsi.md)]

## <a name="start-a-simulation"></a>啟動模擬

使用「時間序列深入解析」總管之前，請先設定裝置模擬解決方案加速器來產生一些遙測資料。 下列螢幕擷取畫面顯示 10 個執行中的冷卻裝置模擬：

![正在執行裝置模擬](./media/iot-accelerators-device-simulation-time-series-insights/running-simulation.png)

## <a name="time-series-insights-explorer"></a>時間序列深入解析總管

「時間序列深入解析」總管是一個 Web 應用程式，可供您用來視覺化遙測資料。

1. 在 Azure 入口網站中，選取「時間序列深入解析」的 [概觀] 索引標籤。

1. 若要開啟「時間序列深入解析」總管 Web 應用程式，請按一下 [移至環境]：

    ![時間序列深入解析總管](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-environment.png)

1. 在時間選取範圍面板中，從快速時間功能表中選取 [過去 30 分鐘]，然後按一下 [搜尋]：

    ![時間序列深入解析總管搜尋](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-search-time.png)

1. 在左側的字詞面板中，選取 [溫度] 作為 [量值]，選取 [iothub-connection-device-id] 作為 [分割依據] 值：

    ![時間序列深入解析總管查詢](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query1.png)

1. 以滑鼠右鍵按一下圖表，然後選取 [探索事件]：

    ![時間序列深入解析總管事件](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explore-events.png)

1. 事件資料會顯示在方格中：

    ![時間序列深入解析總管表](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-table.png)

1. 按一下檢視方塊檢視按鈕：

    ![時間序列深入解析總管檢視方塊](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explorer-perspective.png)

1. 按一下 **+** 將新查詢新增至檢視方塊：

    ![時間序列深入解析總管的新增查詢](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-new-query.png)

1. 選取 [過去 30 分鐘] 作為時間範圍，選取 [濕度] 作為 [量值]，選取 [iothub-connection-device-id] 作為 [分割依據] 值：

    ![時間序列深入解析總管查詢](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query2.png)

1. 按一下檢視方塊檢視按鈕，以檢視裝置遙測儀表板：

    ![時間序列深入解析總管的儀表板](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-dashboard.png)

[!INCLUDE [iot-accelerators-cleanup-tsi.md](../../includes/iot-accelerators-cleanup-tsi.md)]

## <a name="next-steps"></a>後續步驟

若要深入了解如何在時間序列深入解析中探索和查詢資料，請參閱 [Azure 時間序列深入解析總管](../time-series-insights/time-series-insights-explorer.md)。
