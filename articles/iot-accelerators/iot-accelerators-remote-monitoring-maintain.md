---
title: 在遠端監視解決方案中使用警示並修正裝置問題的教學課程 - Azure | Microsoft Docs
description: 本教學課程示範如何使用警示來找出和修正以下裝置問題：連線到遠端監視解決方案加速器的裝置。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 07/19/2018
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: 3138f0ebb6316e69c873a37d479ddc0279a361ef
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39285068"
---
# <a name="tutorial-troubleshoot-and-fix-device-issues"></a>教學課程：對裝置問題進行疑難排解並加以修正

在本教學課程中，您可使用遠端監視解決方案加速器，來找出並修正已連線的 IoT 裝置問題。 您可在解決方案加速器儀表板中使用警示來找出問題，然後執行遠端作業來修正這些問題。

Contoso 會在現場測試新的**原型**裝置。 身為 Contoso 操作員，您會注意到，在測試期間**原型**裝置會意外觸發儀表板上的溫度警示。 您必須立即調查此錯誤**原型**裝置的行為，並且解決問題。

在本教學課程中，您：

>[!div class="checklist"]
> * 調查來自裝置的警示
> * 解決裝置問題

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [iot-accelerators-tutorial-prereqs](../../includes/iot-accelerators-tutorial-prereqs.md)]

## <a name="investigate-an-alert"></a>調查警示

在 [儀表板] 頁面上，您會注意到有非預期的溫度警示來自與**原型**裝置相關聯的規則：

[![儀表板上顯示的警示](./media/iot-accelerators-remote-monitoring-maintain/dashboardalarm-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/dashboardalarm-expanded.png#lightbox)

如需進一步調查問題，請選擇警示旁的 [探索警示] 選項：

[![從儀表板探索警示](./media/iot-accelerators-remote-monitoring-maintain/dashboardexplorealarm-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/dashboardexplorealarm-expanded.png#lightbox)

警示的詳細資料檢視會顯示：

* 觸發警示的時機
* 與警示相關聯裝置的狀態資訊
* 來自與警示相關聯裝置的遙測

[![警示詳細資料](./media/iot-accelerators-remote-monitoring-maintain/maintenancealarmdetail-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenancealarmdetail-expanded.png#lightbox)

若要認可警示，選取所有 [警示發生項目]，然後選擇 [認可]。 這個動作可讓其他操作員得知您已看到警示且正在進行處理：

[![認可警示](./media/iot-accelerators-remote-monitoring-maintain/maintenanceacknowledge-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenanceacknowledge-expanded.png#lightbox)

當您認可警示之後，發生項目的狀態就會變更為 [已認可]。

在出現警示的裝置清單中，您可以看到負責引發溫度警示的**原型**裝置：

[![列出造成警示的裝置](./media/iot-accelerators-remote-monitoring-maintain/maintenanceresponsibledevice-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenanceresponsibledevice-expanded.png#lightbox)

## <a name="resolve-the-issue"></a>解決問題

若要使用**原型**裝置解決問題，您必須在裝置上呼叫 **DecreaseTemperature** 方法。

若要對裝置採取行動，請在出現警示的裝置清單中選取該裝置，然後選擇 [作業]。 **原型**裝置型號支援六種方法：

[![檢視裝置支援的方法](./media/iot-accelerators-remote-monitoring-maintain/maintenancemethods-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenancemethods-expanded.png#lightbox)

選擇 **DecreaseTemperature**，並將作業名稱設定為 **DecreaseTemperature**。 然後，按一下 [套用]：

[![建立作業以降低溫度](./media/iot-accelerators-remote-monitoring-maintain/maintenancecreatejob-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenancecreatejob-expanded.png#lightbox)

若要追蹤作業狀態，請按一下 [檢視作業狀態]。 使用 [作業] 檢視可追蹤解決方案中的所有作業和呼叫方法：

[![監視作業以降低溫度](./media/iot-accelerators-remote-monitoring-maintain/maintenancerunningjob-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/maintenancerunningjob-expanded.png#lightbox)

您可以在 [儀表板] 頁面上檢視遙測資料，確認裝置的溫度是否下降：

[![檢視溫度下降](./media/iot-accelerators-remote-monitoring-maintain/jobresult-inline.png)](./media/iot-accelerators-remote-monitoring-maintain/jobresult-expanded.png#lightbox)

[!INCLUDE [iot-accelerators-tutorial-cleanup](../../includes/iot-accelerators-tutorial-cleanup.md)]

## <a name="next-steps"></a>後續步驟

本教學課程示範如何使用警示來識別您的裝置問題，以及如何在這些裝置上採取行動來解決問題。 若要了解如何將實體裝置連線到您的解決方案加速器，請繼續閱讀操作說明文章。

您現在已了解如何管理裝置問題，建議進行的下一個步驟是了解如何[將裝置連線到遠端監視解決方案加速器](iot-accelerators-connecting-devices.md)。
