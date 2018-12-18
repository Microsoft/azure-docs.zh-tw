---
title: 使用 Azure MQTT 用戶端程式庫將訊息傳送至 MQTT 伺服器 | Microsoft Docs
description: 使用 DevKit 作為用戶端，將訊息傳送至 MQTT 伺服器
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/02/2018
ms.author: liydu
ms.openlocfilehash: fc74613e00adc459f7a7b0a16c6f773fe4bf601d
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39593650"
---
# <a name="send-messages-to-an-mqtt-server"></a>將訊息傳送至 MQTT 伺服器

物聯網 (IoT) 系統經常會遇到間歇性、品質不佳或緩慢的網際網路連線。 MQTT 是電腦對電腦 (M2M) 連線通訊協定，開發人員在開發時謹記這類挑戰。 

這裡使用的 MQTT 用戶端程式庫是 [Eclipse Paho](http://www.eclipse.org/paho/) 專案的一部分，它提供 API 以便透過多種傳輸方式使用 MQTT。

## <a name="what-you-learn"></a>您學到什麼

在此專案中，您將了解：
- 如何使用 MQTT 用戶端程式庫，將訊息傳送至 MQTT 訊息代理程式。
- 如何設定您的 MXChip Iot DevKit 作為 MQTT 用戶端。

## <a name="what-you-need"></a>您需要什麼

完成[使用者入門指南](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)以便：

* 讓 DevKit 連線至 Wi-Fi
* 準備開發環境

## <a name="open-the-project-folder"></a>開啟專案資料夾

1. 如果 DevKit 已經與您的電腦連線，請中斷其連線。

2. 啟動 VS Code。

3. 將 DevKit 連接到您的電腦。

## <a name="open-the-mqttclient-sample"></a>開啟 MQTTClient 範例

展開左邊的 [ARDUINO 範例] 區段，瀏覽至 [MXCHIP AZ3166 的範例] > [MQTT]，然後選取 [MQTTClient]。 隨即開啟一個內含專案資料夾的新 VS Code 視窗。

> [!NOTE]
> 您也可以從命令選擇區開啟範例。 請使用 `Ctrl+Shift+P` (macOS：`Cmd+Shift+P`) 來開啟命令選擇區，輸入 **Arduino**，然後尋找並選取 [Arduino: Examples] \(Arduino: 範例\)。

## <a name="build-and-upload-the-arduino-sketch-to-the-devkit"></a>建置 Arduino 草圖並將其上傳至 DevKit

輸入 `Ctrl+P` (macOS：`Cmd+P`) 以執行 `task device-upload`。 上傳完成後，DevKit 就會重新啟動並執行草圖。

![device-upload](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/device-upload.jpg)

> [!NOTE]
> 您可能會收到「錯誤：AZ3166：未知的套件」錯誤訊息。 未正確重新整理面板套件索引時，就會發生此錯誤。 若要解決此錯誤，請參閱 [IoT DevKit 常見問題集的開發一節](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development)。

## <a name="test-the-project"></a>測試專案

在 VS Code 中，依照此程序來開啟和設定「序列監視器」：

1. 按一下狀態列上的 `COM[X]` 文字，以設定含有 `STMicroelectronics` 的正確 COM 連接埠：![set-com-port](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-com-port.jpg)

2. 按一下狀態列上的電源插頭圖示，以開啟「序列監視器」：![serial-monitor](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-monitor.jpg)
  
3. 在狀態列上，按一下代表「傳輸速率」的數字，然後設定為 `115200`![set-baud-rate](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-baud-rate.jpg)

「序列監視器」會顯示範例草圖傳送的所有訊息。 此草圖會將 DevKit 連線至 Wi-Fi。 一旦成功連線 Wi-Fi，草圖就會將訊息傳送至 MQTT 訊息代理程式。 然後，範例會分別使用 QoS 0 和 QoS 1，重複傳送兩則 "iot.eclipse.org" 訊息。

![serial-output](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-output.jpg)

## <a name="problems-and-feedback"></a>問題與意見反應

如果發生問題，請參閱 [IoT DevKit 常見問題集](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/)，或使用下列管道與我們連絡：

* [Gitter.im](http://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stackoverflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="see-also"></a>另請參閱

* [將 IoT DevKit AZ3166 連線到雲端中的 Azure IoT 中樞](iot-hub-arduino-iot-devkit-az3166-get-started.md)
* [搖一搖以獲取推文](iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message.md)

## <a name="next-steps"></a>後續步驟

現在您已經了解如何設定 MXChip Iot DevKit 作為 MQTT 用戶端，以及使用 MQTT 用戶端程式庫將訊息傳送至 MQTT 訊息代理程式，以下是後續建議步驟：

* [Azure IoT 遠端監視解決方案加速器概觀](https://docs.microsoft.com/azure/iot-suite/)
* [將 MXChip IoT DevKit 裝置連線到您的 Azure IoT Central 應用程式](https://docs.microsoft.com/microsoft-iot-central/howto-connect-devkit)
