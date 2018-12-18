---
title: ESP8266 連線到雲端 - 將 Feather HUZZAH ESP8266 連線到 Azure IoT 中樞 | Microsoft Docs
description: 了解如何設定及連線 Adafruit Feather HUZZAH ESP8266 和 Azure IoT 中樞，在本教學課程中將資料傳送到 Azure 雲端平台。
author: rangv
manager: nasing
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/11/2018
ms.author: rangv
ms.openlocfilehash: ea7754c9bf755a5fc00823629df17317be0f8901
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343673"
---
# <a name="connect-adafruit-feather-huzzah-esp8266-to-azure-iot-hub-in-the-cloud"></a>將 Adafruit Feather HUZZAH ESP8266 連線到位於雲端的 Azure IoT 中樞

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

![DHT22、Feather HUZZAH ESP8266 與 IoT 中樞之間的連線](media/iot-hub-arduino-huzzah-esp8266-get-started/1_connection-hdt22-feather-huzzah-iot-hub.png)

## <a name="what-you-do"></a>您要做什麼

將 Adafruit Feather HUZZAH ESP8266 連線到您將建立的 IoT 中樞。 然後，在 ESP8266 上執行範例應用程式，以收集 DHT22 感應器中的溫度和溼度資料。 最後，將感應器資料傳送至 IoT 中樞。

> [!NOTE]
> 如果您使用其他 ESP8266 電路版，依然可以遵循下列步驟將它連線到 IoT 中樞。 根據您使用的 ESP8266 電路板，您可能需要重新設定 `LED_PIN`。 例如，如果您使用 AI-Thinker 的 ESP8266，可以將它從 `0` 變更為 `2`。 還沒有套件嗎？ 從 [Azure 網站](http://azure.com/iotstarterkits)取得它。

## <a name="what-you-learn"></a>您學到什麼

* 如何建立 IoT 中樞並為 Feather HUZZAH ESP8266 註冊裝置
* 如何連接 Feather HUZZAH ESP8266 與感應器和電腦
* 如何在 Feather HUZZAH ESP8266 上執行範例應用程式來收集感應器資料
* 如何將感應器資料傳送至 IoT 中樞

## <a name="what-you-need"></a>您需要什麼

![本教學課程所需的零件](media/iot-hub-arduino-huzzah-esp8266-get-started/2_parts-needed-for-the-tutorial.png)

若要完成此作業，您需要 Feather HUZZAH ESP8266 入門套件中的下列零件：

* Feather HUZZAH ESP8266 電路板
* Micro USB 轉 Type A 的 USB 纜線

您的開發環境還需要下列事項：

* 有效的 Azure 訂用帳戶。 如果您沒有 Azure 帳戶，請花幾分鐘的時間[建立免費的 Azure 試用帳戶](https://azure.microsoft.com/free/)。
* 執行 Windows 或 Ubuntu 的 Mac 或 PC。
* 供 Feather HUZZAH ESP8266 連線到的無線網路。
* 用來下載設定工具的網際網路連線。
* [適用於 Arduino 的 Visual Studio Code 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-arduino) \(英文\)。

> [!Note]
> 適用於 Arduino 的 Visual Studio Code 擴充功能所使用的 Arduino IDE 版本必須是 1.6.8 版或更新版本。 舊版無法與 AzureIoT 程式庫搭配運作。

如果您沒有感應器，下列項目可供選用。 您也可以選擇使用模擬的感應器資料。

* Adafruit DHT22 溫度和溼度感應器
* 麵包板
* M/M 跳線

[!INCLUDE [iot-hub-get-started-create-hub-and-device](../../includes/iot-hub-get-started-create-hub-and-device.md)]

## <a name="connect-feather-huzzah-esp8266-with-the-sensor-and-your-computer"></a>連接 Feather HUZZAH ESP8266 與感應器和電腦

在本節中，您可以將感應器連接至面板。 然後您可以將裝置插入電腦以進一步使用。

### <a name="connect-a-dht22-temperature-and-humidity-sensor-to-feather-huzzah-esp8266"></a>將 DHT22 溫度和溼度感應器連接至 Feather HUZZAH ESP8266

使用麵包板和跳線來進行連接，如下所示。 如果您沒有感應器，請略過本節，因為您可以改為使用模擬的感應器資料。

![連接參考](media/iot-hub-arduino-huzzah-esp8266-get-started/17_connections_on_breadboard.png)

針對感應器針腳，請使用下列接線方式：

| 開始 (感應器)           | 結束 (電路版)            | 纜線顏色   |
| -----------------------  | ---------------------- | ------------  |
| VDD (針腳 31F)            | 3V (針腳 58H)           | 紅色纜線     |
| DATA (針腳 32F)           | GPIO 2 (針腳 46A)       | 藍色纜線    |
| GND (針腳 34F)            | GND (針腳 56I)          | 黑色纜線   |

如需詳細資訊，請參閱 [Adafruit DHT22 感應器安裝](https://learn.adafruit.com/dht/connecting-to-a-dhtxx-sensor)和 [Adafruit Feather HUZZAH Esp8266 接腳圖](https://learn.adafruit.com/adafruit-feather-huzzah-esp8266/using-arduino-ide?view=all#pinouts)。

現在，Feather Huzzah ESP8266 應該已經和作用中的感應器連接。

![連接 DHT22 與 Feather Huzzah](media/iot-hub-arduino-huzzah-esp8266-get-started/8_connect-dht22-feather-huzzah.png)

### <a name="connect-feather-huzzah-esp8266-to-your-computer"></a>將 Feather HUZZAH ESP8266 連接到電腦

如下所示，請使用「Micro USB 轉 Type A 的 USB 纜線」將 Feather HUZZAH ESP8266 連接到電腦。

![將 Feather Huzzah 連接到電腦](media/iot-hub-arduino-huzzah-esp8266-get-started/9_connect-feather-huzzah-computer.png)

### <a name="add-serial-port-permissions-ubuntu-only"></a>新增序列埠權限 (僅限 Ubuntu)

如果您使用 Ubuntu，請確定您具有操作 Feather HUZZAH ESP8266 之 USB 連接埠的權限。 若要新增序列埠權限，請遵循下列步驟：

1. 在終端機執行下列命令：

   ```bash
   ls -l /dev/ttyUSB*
   ls -l /dev/ttyACM*
   ```

   您會得到下列其中一個輸出︰

   * crw-rw---- 1 root uucp xxxxxxxx
   * crw-rw---- 1 root dialout xxxxxxxx

   在輸出中，請注意 `uucp` 或 `dialout` 是 USB 連接埠的群組擁有者名稱。

2. 執行下列命令，將使用者新增至群組︰

   ```bash
   sudo usermod -a -G <group-owner-name> <username>
   ```

   `<group-owner-name>` 是您在上一個步驟中取得的群組擁有者名稱。 `<username>` 是 Ubuntu 使用者名稱。

3. 登出 Ubuntu，然後再次登入，以顯示變更。

## <a name="collect-sensor-data-and-send-it-to-your-iot-hub"></a>收集感應器資料，並將它傳送至 IoT 中樞

在本節中，您可以在 Feather HUZZAH ESP8266 上部署和執行範例應用程式。 範例應用程式會在 Feather HUZZAH ESP8266 上閃爍 LED，並將從 DHT22 感應器收集的溫度和溼度資料傳送至 IoT 中樞。

### <a name="get-the-sample-application-from-github"></a>從 GitHub 取得範例應用程式

範例應用程式會裝載在 GitHub 上。 請從 GitHub 複製包含範例應用程式的範例存放庫。 若要複製範例存放庫，請遵循下列步驟︰

1. 開啟命令提示字元或終端機視窗。

2. 移至想要儲存範例應用程式的資料夾。

3. 執行以下命令：

   ```bash
   git clone https://github.com/Azure-Samples/iot-hub-feather-huzzah-client-app.git
   ```

   接著，在 Visual Studio Code 中安裝適用於 Feather HUZZAH ESP8266 的套件。

4. 開啟範例應用程式儲存所在的資料夾。

5. 在 Visual Studio Code 中，開啟應用程式資料夾中的 app.ino 檔案。

   ![在 Visual Studio Code 中開啟範例應用程式](media/iot-hub-arduino-huzzah-esp8266-get-started/10_vscode-open-sample-app.png)

6. 在 Visual Studio Code 中，輸入 `F1`。

7. 輸入 **Arduino**，然後選取 [Arduino: 面板管理員]。

8. 在 [Arduino 面板管理員] 索引標籤中，按一下 [其他 URL]。

   ![VS Code Arduino 面板管理員](media/iot-hub-arduino-huzzah-esp8266-get-started/11_vscode-arduino-board-manager.png)

9. 在 [使用者設定] 視窗中，複製下列內容並貼至檔案結尾

   ```
   "arduino.additionalUrls": "http://arduino.esp8266.com/stable/package_esp8266com_index.json"
   ```
   
   ![在 VS Code 中設定 Arduino 套件 URL](media/iot-hub-arduino-huzzah-esp8266-get-started/12_vscode-package-url.png)

10. 儲存檔案並關閉 [使用者設定] 索引標籤。

11. 按一下 [重新整理套件索引]。 重新整理完成之後，搜尋 **esp8266**。

12. 針對 esp8266 按一下 [安裝] 按鈕。

   [電路板管理員] 指出已安裝的 ESP8266 版本為 2.2.0 版或更新版本。

   ![已安裝 esp8266 套件](media/iot-hub-arduino-huzzah-esp8266-get-started/13_vscode-esp8266-installed.png)

13. 輸入 `F1`，然後輸入 **Arduino** 並選取 [Arduino: 面板設定]。

14. 按一下 [選取的面板:] 的方塊並輸入 **esp8266**，然後選取 [Adafruit HUZZAH ESP8266 (esp8266)]。

   ![選取 esp8266 面板](media/iot-hub-arduino-huzzah-esp8266-get-started/14_vscode-select-esp8266.png)

### <a name="install-necessary-libraries"></a>安裝必要的程式庫

1. 在 Visual Studio Code 中，輸入 `F1`，然後輸入 **Arduino** 並選取 [Arduino: 程式庫管理員]。

2. 逐一搜尋下列程式庫名稱。 針對您找到的每個程式庫，按一下 [安裝]。
   * `AzureIoTHub`
   * `AzureIoTUtility`
   * `AzureIoTProtocol_MQTT`
   * `ArduinoJson`
   * `DHT sensor library`
   * `Adafruit Unified Sensor`

### <a name="dont-have-a-real-dht22-sensor"></a>沒有真正的 DHT22 感應器嗎？

如果您沒有真正的 DHT22 感應器，範例應用程式可以模擬溫度和溼度資料。 若要設定讓範例應用程式使用模擬資料，請遵循下列步驟：

1. 開啟 `app` 資料夾中的 `config.h` 檔案。

2. 找出下面這行程式碼，並將值從 `false` 變更為 `true`：

   ```c
   define SIMULATED_DATA true
   ```
   
   ![設定範例應用程式以使用模擬資料](media/iot-hub-arduino-huzzah-esp8266-get-started/15_vscode-configure-app-use-simulated-data.png)

3. 儲存檔案。

### <a name="deploy-the-sample-application-to-feather-huzzah-esp8266"></a>將範例應用程式部署至 Feather HUZZAH ESP8266

1. 在 Visual Studio Code 中，按一下狀態列上的 [<Select Serial Port>]，然後按一下 Feather HUZZAH ESP8266 的序列埠。

2. 輸入 `F1`，然後輸入 **Arduino** 並選取 [Arduino: 上傳]，以建置範例應用程式並將其部署至 Feather HUZZAH ESP8266。

### <a name="enter-your-credentials"></a>輸入認證

上傳程序成功完成後，請遵循這些步驟來輸入認證：

1. 開啟 Arduino IDE，按一下 [Tools] \(工具\) > [Serial Monitor] \(序列監視器\)。

2. 在 [序列監視器] 視窗中，請注意右下角的兩個下拉式清單。

3. 在左邊的下拉式清單中選取 [無行尾結束符號]。

4. 在右邊的下拉式清單中選取 [115200 傳輸速率]。

5. 在位於 [序列監視器] 視窗頂端的輸入方塊中，輸入系統要求您提供的下列資訊，然後按一下 [傳送]。

   * Wi-Fi SSID
   * Wi-Fi 密碼
   * 裝置連接字串

> [!Note]
> 認證資訊會儲存在 Feather HUZZAH ESP8266 的 EEPROM 中。 如果您按一下 Feather HUZZAH ESP8266 電路板上的重設按鈕，範例應用程式會詢問您是否要清除資訊。 輸入 `Y` 將資訊清除。 系統會要求您再次提供資訊。

### <a name="verify-the-sample-application-is-running-successfully"></a>確認範例應用程式已成功執行

如果您在 [序列監視器] 視窗看到下列輸出，並在 Feather HUZZAH ESP8266 上看到閃爍的 LED，則表示範例應用程式已成功執行。

![Arduino IDE 中的最終輸出](media/iot-hub-arduino-huzzah-esp8266-get-started/16_arduino-ide-final-output.png)

## <a name="next-steps"></a>後續步驟

您已成功將 Feather HUZZAH ESP8266 連線到 IoT 中樞，並將擷取到的感應器資料傳送至 IoT 中樞。 

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]