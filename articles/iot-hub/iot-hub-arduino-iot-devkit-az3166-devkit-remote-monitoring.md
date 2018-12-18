---
title: IoT DevKit 到雲端 -- 將 IoT MXChip DevKit 連線到 Azure IoT 中樞 | Microsoft Docs
description: 在本教學課程中，了解如何將 IoT DevKit AZ3166 上的感應器狀態傳送至 Azure IoT 遠端監視解決方案加速器。
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 02/02/2018
ms.author: liydu
ms.openlocfilehash: 79a44e3f5303aaf0d337333b482c2df670e0b3da
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42146192"
---
# <a name="connect-mxchip-iot-devkit-to-azure-iot-remote-monitoring-solution-accelerator"></a>將 MXChip IoT DevKit 連線到 Azure IoT 遠端監視解決方案加速器

在本教學課程中，您將會了解如何在 DevKit 上執行範例應用程式，以將感應器資料傳送至 Azure IoT 遠端監視解決方案加速器。

[MXChip IoT DevKit](https://aka.ms/iot-devkit) 是全方位 Arduino 相容板，具有豐富的週邊設備和感應器。 您可以使用[適用於 Arduino 的 Visual Studio Code 延伸模組](https://aka.ms/arduino)來對它進行開發。 它隨附不斷增長的[專案目錄](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/)，以引導您設定運用 Microsoft Azure 服務的物聯網 (IoT) 解決方案原型。

## <a name="what-you-need"></a>您需要什麼

完成[使用者入門指南](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)以便：

* 讓 DevKit 連線至 Wi-Fi
* 準備開發環境

有效的 Azure 訂用帳戶。 如果沒有，您可以透過下列兩種方法之一來註冊：

* 啟動 [30 天免費試用 Microsoft Azure 帳戶](https://azure.microsoft.com/free/)

* 如果您是 MSDN 或 Visual Studio 訂閱者，請認領您的 [Azure 點數](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)

## <a name="create-an-azure-iot-remote-monitoring-solution-accelerator"></a>建立 Azure IoT 遠端監視解決方案加速器

1. 移至 [Azure IoT 解決方案加速器網站](https://www.azureiotsolutions.com/)，然後按一下 [建立新解決方案]。

   ![選取 Azure IoT 解決方案加速器類型](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-solution-types.png)

   > [!WARNING]
   > 根據預設，此範例會在建立一個 IoT 遠端監視解決方案加速器之後，建立 S2 IoT 中樞。 如果此 IoT 中樞並未搭配大量裝置來使用，強烈建議您將其從 S2 降級為 S1，並刪除不再需要的 IoT 遠端監視解決方案加速器，以便能夠一併刪除相關的 IoT 中樞。 

2. 選取 [遠端監視]。

3. 輸入解決方案名稱，選取訂用帳戶和區域，然後按一下 [建立解決方案]。 此解決方案可能需要一些時間才能完成佈建。
  
   ![建立解決方案](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution.png)

4. 在佈建完成之後，按一下 [啟動]。 佈建程序進行期間會為解決方案建立一些模擬裝置。 按一下 **裝置** 將其簽出。

   ![儀表板](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution-created.png)
  
   ![主控台](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-console.png)

5. 按一下 [新增裝置]。

6. 在 [自訂裝置] 中按一下 [新增]。
  
   ![新增裝置](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-add-new-device.png)

7. 按一下 [讓我定義自己的裝置識別碼]，輸入 `AZ3166`，然後按一下 [建立]。
  
   ![使用識別碼建立裝置](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-device-configuration.png)

8. 記下 **IoT 中樞主機名稱**，然後按一下 [完成]。

## <a name="open-the-remotemonitoring-sample"></a>開啟 RemoteMonitoring 範例

1. 將已連接至您電腦的 DevKit 拆下。

2. 啟動 VS Code。

3. 將 DevKit 連接到您的電腦。 VS Code 會自動偵測您的 DevKit，並開啟下列頁面：

  * DevKit 簡介頁面。
  * Arduino 範例：用來開始使用 DevKit 的實際操作範例。

4. 展開左邊的 [ARDUINO 範例] 區段，瀏覽至 [MXCHIP AZ3166 的範例] > [AzureIoT]，然後選取 [RemoteMonitoring]。 這會開啟一個內含專案資料夾的新 VS Code 視窗。

   > [!NOTE]
   > 如果您不小心關閉該窗格，可予以重新開啟。 請使用 `Ctrl+Shift+P` (macOS：`Cmd+Shift+P`) 來開啟命令選擇區，輸入 **Arduino**，然後尋找並選取 [Arduino: Examples] \(Arduino: 範例\)。

## <a name="provision-required-azure-services"></a>佈建所需的 Azure 服務

在解決方案視窗中，於所提供的文字方塊中輸入 `task cloud-provision` 以透過 `Ctrl+P` (macOS：`Cmd+P`) 執行您的工作。

VS Code 終端機中會有互動式命令列引導您佈建所需的 Azure 服務。

![佈建 Azure 資源](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/provision.png)

## <a name="build-and-upload-the-device-code"></a>組建並上傳裝置程式碼

1. 使用 `Ctrl+P` (macOS：`Cmd + P`) 並輸入 **task config-device-connection**.

2. 終端機會詢問您是否要使用擷取自 `task cloud-provision` 步驟的連接字串。 您也可以按一下 [建立新的...] 來輸入自己的裝置連接字串

3. 終端機會提示您進入設定模式。 若要這樣做，請按住按鈕 A，然後按下並放開 [重設] 按鈕。 畫面會顯示 DevKit 識別碼和 [設定]。

   ![輸入連接字串](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/config-device-connection.png)

4. 在 `task config-device-connection` 完成之後，請按一下 `F1` 以載入 VS Code 命令並選取 `Arduino: Upload`。 VS Code 就會開始驗證並上傳 Arduino 草稿碼：
  
   ![驗證和上傳 Arduino 草圖](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/arduino-upload.png)

DevKit 會重新開機，然後開始執行程式碼。

## <a name="test-the-project"></a>測試專案

當範例應用程式執行時，DevKit 會透過 WiFi 將感應器資料傳送至 Azure IoT 遠端監視解決方案加速器。 若要查看結果，請遵循下列步驟：

1. 移至 Azure IoT 遠端監視解決方案加速器，然後按一下 [儀表板]。

2. 在遠端監視解決方案主控台中，您會看到 DevKit 感應器的狀態。

   ![Azure IoT 遠端監視解決方案加速器中的感應器資料](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/sensor-status.png)

## <a name="change-device-id"></a>變更裝置識別碼

若要將程式碼中的硬式編碼 **AZ3166** 變更為自訂裝置識別碼，請修改[遠端監視範例](https://github.com/Microsoft/devkit-sdk/blob/master/AZ3166/src/libraries/AzureIoT/examples/RemoteMonitoring/RemoteMonitoring.ino#L23)中顯示的程式碼。

## <a name="problems-and-feedback"></a>問題與意見反應

如果您遇到問題，請參閱 [IoT 開發人員套件常見問題集](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) \(英文\) 或使用下列管道與我們連絡：

* [Gitter.im](http://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stackoverflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>後續步驟

您已了解如何將 DevKit 裝置連線至 Azure IoT 遠端監視解決方案加速器並將感應器資料視覺化，以下是建議的後續步驟：

* [Azure IoT 解決方案加速器概觀](https://docs.microsoft.com/azure/iot-suite/)

* [將 MXChip IoT DevKit 裝置連線到您的 Azure IoT Central 應用程式](https://docs.microsoft.com/microsoft-iot-central/howto-connect-devkit)

* [IoT 開發人員套件](https://microsoft.github.io/azure-iot-developer-kit/) 
