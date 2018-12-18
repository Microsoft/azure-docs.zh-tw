---
title: IoT DevKit 到雲端-- 將 IoT DevKit AZ3166 連線到遠端監視 IoT 解決方案加速器 | Microsoft Docs
description: 在本教學課程中，了解如何將 IoT DevKit AZ3166 上的感應器狀態傳送至遠端監視 IoT 解決方案加速器來進行監視和呈現視覺效果。
author: isabelcabezasm
manager: ''
ms.service: iot-accelerators
services: iot-accelerators
ms.devlang: c
ms.topic: conceptual
ms.date: 12/03/2018
ms.author: isacabe
ms.openlocfilehash: 92ce85a3cc94702468a13348d3a41667498c68f5
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43187386"
---
# <a name="connect-mxchip-iot-devkit-az3166-to-the-iot-remote-monitoring-solution-accelerator"></a>將 MXChip IoT DevKit AZ3166 連線到 IoT 遠端監視解決方案加速器


[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

在本教學課程中，您將會了解如何在 DevKit 上執行範例應用程式，以將感應器資料傳送至解決方案加速器。

[MXChip IoT DevKit](https://aka.ms/iot-devkit) 是全方位 Arduino 相容板，具有豐富的週邊設備和感應器。 您可以使用[適用於 Arduino 的 Visual Studio Code 延伸模組](https://aka.ms/arduino)來對它進行開發。 它隨附不斷增長的[專案目錄](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/)，以引導您設定運用 Microsoft Azure 服務的物聯網 (IoT) 解決方案原型。

## <a name="what-you-need"></a>您需要什麼

完成[使用者入門指南](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)以便：

* 讓 DevKit 連線至 Wi-Fi
* 準備開發環境


## <a name="open-the-remotemonitoring-sample"></a>開啟 RemoteMonitoring 範例

1. 將已連接至您電腦的 DevKit 拆下。

2. 啟動 VS Code。

3. 將 DevKit 連接到您的電腦。 VS Code 會自動偵測您的 DevKit，並開啟下列頁面：
  * DevKit 簡介頁面。
  * Arduino 範例：用來開始使用 DevKit 的實際操作範例。

4. 展開左邊的 [ARDUINO 範例] 區段，瀏覽至 [MXCHIP AZ3166 的範例] > [AzureIoT]，然後選取 [RemoteMonitoringv2]。 這會開啟一個內含專案資料夾的新 VS Code 視窗。

  ![開啟遠端監視專案](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-arduino-examples.png)


  > [!NOTE]
  > 如果您不小心關閉該窗格，可予以重新開啟。 請使用 `Ctrl+Shift+P` (macOS：`Cmd+Shift+P`) 來開啟命令選擇區，輸入 **Arduino**，然後尋找並選取 [Arduino: Examples] \(Arduino: 範例\)。

## <a name="add-a-new-physical-device"></a>新增新實體裝置

在入口網站中，移至 [裝置] 區段，然後按一下 [+ 新裝置] 按鈕。 

![新增新裝置](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-add-device.png)

應填寫*新裝置表單*。
1. 按一下 [裝置類型] 區段中的 [實體]。
2. 定義您自己的裝置識別碼 (例如 *MXChip* 或 *AZ3166*)。
3. 選擇 [驗證金鑰] 區段中的 [自動產生金鑰]。
4. 按一下 [套用] 按鈕。

![新增新裝置表單](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-add-new-device-form.png)

等候入口網站完成新裝置的佈建。

![佈建新裝置 ](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-add-device-provisioning.png)


然後會顯示新裝置的設定。
複製產生的**連接字串**。

![裝置連接字串](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-new-device-connstring.png)


在下一節中會使用此連接字串。





## <a name="build-and-upload-the-device-code"></a>組建並上傳裝置程式碼

回到 Visual Studio Code： 

1. 使用 `Ctrl+P` (macOS：`Cmd + P`) 並輸入 **task config-device-connection**.

  ![選擇您的 Azure 訂閱和 IoT 中樞](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/iot-suite-task-config-device-conexion.png)

2. 終端機會詢問您是否要使用 IoT 裝置的連接字串。 選取 [建立新的]，現在將其貼上。

  ![貼上連接字串](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/iot-suite-task-config-device-conexion-choose-iot-hub-press-button-A.png)

3. 終端機有時會提示您進入設定模式。 若要這樣做，請按住按鈕 A，接著按下再放開重設按鈕，然後放開按鈕 A。畫面會顯示 DevKit 識別碼和「設定」。

  ![裝置 DevKit 畫面](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-devkit-screen.png)

  > [!NOTE]
  > 若您依循本教學課程的上一節，連接字串應儲存在剪貼簿中。 如果沒有，您應移至 Azure 入口網站並尋找遠端監視資源群組的 IoT 中樞。 您可以看到 IoT 中樞連接的裝置，並複製裝置連接字串。

  ![尋找連接字串](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-connection-string-of-a-device.png)


現在，您可以在 VS Code 區段「Azure IoT 中樞裝置」看到新實體裝置：

![注意新的 IoT 中樞裝置](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/iot-suite-new-iot-hub-device.png)

## <a name="test-the-project"></a>測試專案

範例應用程式執行時，DevKit 會透過 Wi-Fi 將感應器資料傳送至解決方案加速器。 若要查看結果，請遵循下列步驟：

1. 移至您的 IoT 解決方案加速器，並按一下 [儀表板]。

2. 在 IoT 解決方案加速器主控台中，您會看到 DevKit 感應器的狀態。 

![IoT 解決方案加速器中的感應器資料](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-dashboard.png)

如果您按一下感應器名稱 (AZ3166)，儀表板右側會開啟索引標籤，您可以在其中檢視即時 MX 晶片感應器圖表。


## <a name="send-a-c2d-message"></a>傳送 C2D 訊息
遠端監視 v2 可讓您在裝置上叫用遠端方法。
選取感應器時，可以在 [方法] 區段中看到 MX 晶片範例程式碼發佈的三種方法。

![方法 MX 晶片](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-methods.png)

您可以使用「LedColor」方法變更其中一個 MX 晶片 LED 的色彩。 若要這樣做，選取裝置的核取方塊，然後按一下 [排程] 按鈕。 

![方法 MX 晶片](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-schedule.png)

選擇下拉式清單的所有方法中名為 ChangeColor 的方法，寫入名稱並套用。

![下拉式清單 MX 晶片](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/iot-suite-change-color.png)

在幾秒鐘內，您的實體 MX 晶片應會變更 RGB LED 的色彩 (A 按鈕下方)

![LED MX 晶片](./media/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoringV2/azure-iot-suite-devkit-led.png)

## <a name="problems-and-feedback"></a>問題與意見反應

如果您遇到問題，請參閱[常見問題集](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/)或透過下列管道與我們連絡：

* [Gitter.im](http://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stackoverflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>後續步驟

您已了解如何將 DevKit 裝置連接至 IoT 解決方案加速器並將感應器資料視覺化，以下是建議的後續步驟：

* [IoT 解決方案加速器概觀](https://docs.microsoft.com/azure/iot-suite/)
* [將 MXChip IoT DevKit 裝置連線到您的 Microsoft IoT Central 應用程式](https://docs.microsoft.com/microsoft-iot-central/howto-connect-devkit)
* [IoT 開發人員套件](https://microsoft.github.io/azure-iot-developer-kit/)
