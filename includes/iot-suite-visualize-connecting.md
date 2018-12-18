---
title: 包含檔案
description: 包含檔案
services: iot-suite
author: dominicbetts
ms.service: iot-suite
ms.topic: include
ms.date: 04/24/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 5702c6e9c9d75c6cccb82f1c57684ef7b9898c34
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34665995"
---
## <a name="view-device-telemetry"></a>檢視裝置遙測資料

您可以在解決方案中，檢視從您 [裝置] 頁面上之裝置所傳送的遙測。

1. 在 [裝置] 頁面上選取裝置清單中佈建的裝置。 面板會顯示您裝置的相關資訊，包括裝置遙測繪圖：

    ![請參閱裝置詳細資料](media/iot-suite-visualize-connecting/devicesdetail.png)

1. 選擇 [壓力] 以變更遙測顯示器：

    ![檢視壓力遙測](media/iot-suite-visualize-connecting/devicespressure.png)

1. 若要檢視有關您裝置的診斷資訊，請向下捲動至 [診斷]：

    ![檢視裝置診斷](media/iot-suite-visualize-connecting/devicesdiagnostics.png)

## <a name="act-on-your-device"></a>在裝置上採取行動

若要在您的裝置上叫用方法，請使用遠端監視解決方案中的**裝置**頁面。 例如，在遠端監視解決方案中，**Chiller** 裝置會實作 **FirmwareUpdate** 方法。

1. 選擇 [裝置] 以瀏覽至解決方案中的 [裝置] 頁面。

1. 在 [裝置] 頁面上選取裝置清單中佈建的裝置：

    ![選取您的實體裝置](media/iot-suite-visualize-connecting/devicesselect.png)

1. 若要顯示您可以在裝置上呼叫的方法清單，請依序選擇 [作業] 和 [執行方法]。 若要排程可在多個裝置上執行的作業，您可以在清單中選取多個裝置。 [作業] 面板會顯示所有您選取之裝置的通用方法。

1. 選擇 [FirmwareUpdate]，將作業名稱設定為 [UpdatePhysicalChiller]。 將 [韌體版本] 設定為 [2.0.0]，將 [韌體 URI] 設定為 **http://contoso.com/updates/firmware.bin**，然後選擇 [套用]：

    ![排程韌體更新](media/iot-suite-visualize-connecting/deviceschedule.png)

1. 當模擬裝置處理該方法時，會在執行裝置程式碼的主控台中顯示一系列訊息。

1. 更新完成時，新的韌體版本會顯示在 [裝置] 頁面上：

    ![更新完成](media/iot-suite-visualize-connecting/complete.png)

> [!NOTE]
> 若要在解決方案中追蹤作業的狀態，請選擇 [檢視]。

## <a name="next-steps"></a>後續步驟

[自訂遠端監視解決方案加速器](../articles/iot-accelerators/iot-accelerators-remote-monitoring-customize.md)一文中，說明了一些自訂解決方案加速器的方法。