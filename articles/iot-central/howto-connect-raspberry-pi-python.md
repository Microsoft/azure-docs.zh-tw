---
title: 將 Raspberry Pi 連線到 Azure IoT Central 應用程式 (Python) | Microsoft Docs
description: 如何以裝置開發人員身分，將 Raspberry Pi 連線到使用 Python 的 Azure IoT Central 應用程式。
author: dominicbetts
ms.author: dobett
ms.date: 01/23/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: timlt
ms.openlocfilehash: b5632db57e902eef76860f85de6e76f85861090a
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45728958"
---
# <a name="connect-a-raspberry-pi-to-your-azure-iot-central-application-python"></a>將 Raspberry Pi 連線到 Azure IoT Central 應用程式 (Python)

[!INCLUDE [howto-raspberrypi-selector](../../includes/iot-central-howto-raspberrypi-selector.md)]

本文說明如何以裝置開發人員身分，將 Raspberry Pi 連線到使用 Python 程式設計語言的 Microsoft Azure IoT Central 應用程式。

## <a name="before-you-begin"></a>開始之前

若要完成這篇文章中的步驟，您需要下列項目︰

* Azure IoT Central 應用程式是從**範例 Devkits** 應用程式範本建立而來。 如需詳細資訊，請參閱[建立 Azure IoT Central 應用程式](howto-create-application.md)。
* 執行 Raspbian 作業系統的 Raspberry Pi 裝置。 您需要在 Raspberry Pi 接上監視器、鍵盤和滑鼠以便存取 GUI 環境。 Raspberry Pi 必須能夠[連線到網際網路](https://www.raspberrypi.org/learning/software-guide/wifi/)。
* (選擇性) 適用於 Raspberry Pi 的 [Sense Hat](https://www.raspberrypi.org/products/sense-hat/) 附加元件面板。 此面板會收集各種感應器的遙測資料，以傳送至 Azure IoT Central 應用程式。 如果您沒有 **Sense Hat** 面板，可以改用模擬器 (可作為 Raspberry Pi 映像的一部份使用)。

## <a name="sample-devkits-application"></a>**範例 Devkits** 應用程式

從**範例 Devkits** 應用程式範本建立的應用程式包含具有下列特性的 **Raspberry Pi**  裝置範本： 

- 包含裝置**溼度**、**溫度**、**壓力**、**磁強計** (沿著 X、Y、Z 軸測量)、**加速計** (沿著 X、Y、Z 軸測量) 和**陀螺儀** (沿著 X、Y、Z 軸測量) 等測量值的遙測。
- 顯示**電壓**、**電流**、**風扇速度**和 **IR** 切換的設定。
- 包含裝置屬性**骰子數字**和**位置**雲端屬性的屬性。


如需裝置範本的詳細設定資訊，請參閱 [Raspberry PI 裝置範本詳細資料](howto-connect-raspberry-pi-python.md#raspberry-pi-device-template-details)
    

## <a name="add-a-real-device"></a>新增真實裝置

在 Azure IoT Central 應用程式中，從 **Raspberry Pi** 裝置範本新增真實裝置，並記下裝置連線詳細資料 (**範圍識別碼、裝置識別碼、主要金鑰**)。 如需詳細資訊，請參閱[將真實裝置新增至 Azure IoT Central 應用程式](tutorial-add-device.md)。


### <a name="configure-the-raspberry-pi"></a>設定 Raspberry Pi

下列步驟說明如何從 GitHub 下載 Python 應用程式範例並進行設定。 此應用程式範例會：

* 將遙測和屬性值傳送至 Azure IoT Central。
* 回應在 Azure IoT Central 中所做的設定變更。

若要設定裝置，請[遵循 GitHub 上的逐步指示。](http://aka.ms/iotcentral-docs-Raspi-releases)


> [!NOTE]
> 如需 Raspberry Pi Python 範例的詳細資訊，請參閱 GitHub 上的[讀我檔案](http://aka.ms/iotcentral-docs-Raspi-releases)。


1. 當裝置完成設定後，您的裝置應該會立即開始將資料傳送至 Azure IoT Central。
1. 在 Azure IoT Central 應用程式中，您會看到在 Raspberry Pi 上執行的程式碼如何與應用程式互動：

    * 在真實裝置的 [量測] 頁面上，您會看到從 Raspberry Pi 傳送過來的遙測。 如果您使用的是 **Sense HAT 模擬器**，則可以在 Raspberry Pi 的 GUI 中修改遙測值。
    * 在 [屬性] 頁面上，您可以看到所報告 [模具編號] 屬性的值。
    * 在 [設定] 頁面上，您可以變更 Raspberry Pi 的各種設定，例如電壓和風扇。 當 Raspberry Pi 確認變更時，Azure IoT Central 中的設定便會顯示為 [已同步]。


## <a name="raspberry-pi-device-template-details"></a>Raspberry PI 裝置範本詳細資料

從**範例 Devkits** 應用程式範本建立的應用程式包含具有下列特性的 **Raspberry Pi**  裝置範本：

### <a name="telemetry-measurements"></a>遙測量測

| 欄位名稱     | Units  | 最小值 | 最大值 | 小數位數 |
| -------------- | ------ | ------- | ------- | -------------- |
| 溼度       | %      | 0       | 100     | 0              |
| temp           | °C     | -40     | 120     | 0              |
| pressure       | hPa    | 260     | 1260    | 0              |
| magnetometerX  | mgauss | -1000   | 1000    | 0              |
| magnetometerY  | mgauss | -1000   | 1000    | 0              |
| magnetometerZ  | mgauss | -1000   | 1000    | 0              |
| accelerometerX | mg     | -2000   | 2000    | 0              |
| accelerometerY | mg     | -2000   | 2000    | 0              |
| accelerometerZ | mg     | -2000   | 2000    | 0              |
| gyroscopeX     | mdps   | -2000   | 2000    | 0              |
| gyroscopeY     | mdps   | -2000   | 2000    | 0              |
| gyroscopeZ     | mdps   | -2000   | 2000    | 0              |

### <a name="settings"></a>設定

數值設定

| 顯示名稱 | 欄位名稱 | Units | 小數位數 | 最小值 | 最大值 | Initial |
| ------------ | ---------- | ----- | -------------- | ------- | ------- | ------- |
| 電壓      | setVoltage | 伏特 | 0              | 0       | 240     | 0       |
| Current      | setCurrent | 安培  | 0              | 0       | 100     | 0       |
| 風扇速度    | fanSpeed   | RPM   | 0              | 0       | 1000    | 0       |

切換設定

| 顯示名稱 | 欄位名稱 | 開啟文字 | 關閉文字 | Initial |
| ------------ | ---------- | ------- | -------- | ------- |
| IR           | activateIR | 開啟      | 關      | 關閉     |

### <a name="properties"></a>properties

| 類型            | 顯示名稱 | 欄位名稱 | 資料類型 |
| --------------- | ------------ | ---------- | --------- |
| 裝置屬性 | 模具編號   | dieNumber  | number    |
| 文字            | 位置     | location   | N/A       |

## <a name="next-steps"></a>後續步驟

您現在已了解如何將 Raspberry Pi 連線至 Azure IoT Central 應用程式，以下是建議的後續步驟：

* [將一般 Node.js 用戶端應用程式連線至 Azure IoT Central](howto-connect-nodejs.md)
