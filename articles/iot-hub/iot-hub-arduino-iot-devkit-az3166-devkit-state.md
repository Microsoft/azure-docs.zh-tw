---
title: 使用 Azure 裝置對應項來控制 MXChip IoT DevKit 使用者 LED | Microsoft Docs
description: 在本教學課程中，了解如何使用 Azure IoT 中樞裝置對應項來監視 DevKit 狀態和控制使用者 LED。
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/04/2018
ms.author: liydu
ms.openlocfilehash: 6bc1255c5bbb9cf74c97b88600f34e7fcd90ae4f
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343142"
---
# <a name="mxchip-iot-devkit"></a>MXChip IoT DevKit

您可以使用這個範例來監視 MXChip IoT DevKit WiFi 資訊和感應器狀態，以及使用 Azure IoT 中樞裝置對應項來控制使用者 LED 的色彩。

## <a name="what-you-learn"></a>您學到什麼

- 如何監視 MXChip IoT DevKit 感應器狀態。

- 如何使用 Azure 裝置對應項來控制 DevKit RGB LED 的色彩。

## <a name="what-you-need"></a>您需要什麼

- 依照[快速入門指南](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)來設定開發環境。

- 在 GitBash 終端機視窗 (或其他 Git 命令列介面) 中，輸入下列命令：

   ```bash
   git clone https://github.com/DevKitExamples/DevKitState.git
   cd DevKitState
   code .
   ```

## <a name="provision-azure-services"></a>佈建 Azure 服務

1. 按一下 Visual Studio Code 中的 [工作] 下拉式功能表，然後選取 [執行工作...]  - [cloud-provision]。

2. 您的進度會顯示在 [歡迎使用] 面板的 [終端機] 索引標籤之下。

3. 當系統提示「您要選擇哪個訂用帳戶」訊息時，請選取一個訂用帳戶。

4. 選取或選擇資源群組。 
 
   > [!NOTE]
   > 如果您已經有免費的 IoT 中樞，您可以略過此步驟。

5. 當系統提示「您要選擇哪個 IoT 中樞」訊息時，請選取或建立一個 IoT 中樞。

6. 隨即顯示類似 function app: function app name: xxx 的項目。 請記下函式應用程式名稱，後續步驟將會使用該名稱。

7. 等候 Azure Resource Manager 範本部署完成，也就是「Resource Manager 範本部署：完成」訊息顯示時。

## <a name="deploy-function-app"></a>部署函式應用程式

1. 按一下 Visual Studio Code 中的 [工作] 下拉式功能表，然後選取 [執行工作...]  - [cloud-deploy]。

2. 等候函式應用程式程式碼上傳程序完成；「函式應用程式部署：完成」訊息隨即顯示。

## <a name="configure-iot-hub-device-connection-string-in-devkit"></a>在 DevKit 中設定 IoT 中樞裝置連接字串

1. 將 MXChip IoT DevKit 連接到您的電腦。

2. 按一下 Visual Studio Code 中的 [工作] 下拉式功能表，然後選取 [執行工作...]  - [config-device-connection]。

3. 在 MXChip IoT DevKit，按住按鈕 **A**、按 [重設] 按鈕，然後放開按鈕 **A**，讓 DekKit 進入設定模式。

4. 等候連接字串設定程序完成。

## <a name="upload-arduino-code-to-devkit"></a>將 Arduino 程式碼上傳至 DevKit

將 MXChip IoT DevKit 連接到您的電腦後：

1. 按一下 Visual Studio Code 中的 [工作] 下拉式功能表，然後選取 [執行組建工作...]Arduino 草稿碼會經過編譯並上傳至 DevKit。

2. 當草稿碼上傳成功後，「建立並上傳草稿碼：成功」訊息隨即顯示。

## <a name="monitor-devkit-state-in-browser"></a>在瀏覽器中監視 DevKit 狀態

1. 在網頁瀏覽器中，開啟 `DevKitState\web\index.html` 檔案--該檔案建立於[您需要什麼](#whatyouneed)步驟期間。

2. 以下網頁隨即出現：![指定函式應用程式名稱。](media/iot-hub-arduino-iot-devkit-az3166-devkit-state/devkit-state-function-app-name.png)

3. 輸入您稍早記下的函式應用程式名稱。

4. 按一下 [連線] 按鈕。

5. 在幾秒內，頁面就會重新整理並顯示 DevKit 的 WiFi 連線狀態和每個內建感應器的狀態。

## <a name="control-the-devkits-user-led"></a>控制 DevKit 的使用者 LED

1. 按一下網頁圖例上的使用者 LED 圖片。

2. 在幾秒內，畫面就會重新整理並顯示使用者 LED 的目前色彩狀態。

3. 嘗試按一下 RGB 滑桿控制項上的不同位置，以變更 RGB LED 的色彩值。

## <a name="example-operation"></a>範例作業

![範例測試程序](media/iot-hub-arduino-iot-devkit-az3166-devkit-state/devkit-state.gif)

> [!NOTE]
> 您可以在 Azure 入口網站中查看裝置對應項的未經處理資料：IoT 中樞 -\> IoT 裝置\> *\<您的裝置\>* -\> 裝置對應項。

## <a name="next-steps"></a>後續步驟

您已了解如何︰
- 將 MXChip IoT DevKit 裝置連線到 Azure IoT 遠端監視解決方案加速器。
- 使用 Azure IoT 裝置對應項函式來感測和控制 DevKit RGB LED 的色彩。

以下是建議的後續步驟：

* [Azure IoT 遠端監視解決方案加速器概觀](https://docs.microsoft.com/azure/iot-suite/)
* [將 MXChip IoT DevKit 裝置連線到您的 Azure IoT Central 應用程式](https://docs.microsoft.com/microsoft-iot-central/howto-connect-devkit)
