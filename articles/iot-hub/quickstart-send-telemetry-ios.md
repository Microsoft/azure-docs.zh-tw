---
title: 將遙測傳送至 Azure IoT 中樞快速入門 | Microsoft Docs
description: 本快速入門中，您可以執行範例 iOS 應用程式，將模擬的遙測傳送到 IoT 中樞以及從 IoT 中樞讀取遙測，以便在雲端中處理。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: quickstart
ms.custom: mvc
ms.date: 04/20/2018
ms.author: kgremban
ms.openlocfilehash: 96989f8c53508dd1520a38c0df408057ad673d53
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365368"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-ios"></a>快速入門：將遙測從裝置傳送到 IoT 中樞 (iOS)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

IoT 中樞是一項 Azure 服務，可讓您從 IoT 裝置將大量的遙測擷取到雲端進行儲存或處理。 在本文中，您會將遙測從模擬的裝置應用程式傳送到 IoT 中樞。 然後您可以從後端應用程式檢視資料。 

本文會使用預先撰寫的 Swift 應用程式傳送遙測，以及使用 CLI 公用程式從 IoT 中樞讀取遙測。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

- 從 [Azure 範例](https://github.com/Azure-Samples/azure-iot-samples-ios/archive/master.zip)下載程式碼範例 
- 最新版的 [XCode](https://developer.apple.com/xcode/)，其執行最新版的 iOS SDK。 本快速入門已使用 XCode 9.3 和 iOS 11.3 進行測試。
- 最新版的 [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)。

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

[!INCLUDE [iot-hub-quickstarts-create-hub](../../includes/iot-hub-quickstarts-create-hub.md)]

## <a name="register-a-device"></a>註冊裝置

裝置必須向的 IoT 中樞註冊，才能進行連線。 在本快速入門中，您會使用 Azure Cloud Shell 來註冊模擬的裝置。

1. 在 Azure Cloud Shell 中執行下列命令，以新增 IoT 中樞 CLI 擴充功能和建立裝置身分識別。 

   **YourIoTHubName**：以您為 IoT 中樞選擇的名稱取代此預留位置。

   **myiOSdevice**：這是為已註冊裝置指定的名稱。 使用所示的 myiOSdevice。 如果您為裝置選擇不同的名稱，則也必須在本文中使用該名稱，並先在範例應用程式中更新該裝置名稱，再執行應用程式。

   ```azurecli-interactive
   az extension add --name azure-cli-iot-ext
   az iot hub device-identity create --hub-name YourIoTHubName --device-id myiOSdevice
   ```

1. 執行下列命令，以針對您剛註冊的裝置取得_裝置連接字串_：

   ```azurecli-interactive
   az iot hub device-identity show-connection-string --hub-name YourIoTHubName --device-id myiOSdevice --output table
   ```

   記下裝置連接字串，它看起來如下：

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    您稍後會在快速入門中使用此值。

## <a name="send-simulated-telemetry"></a>傳送模擬的遙測

範例應用程式會在 iOS 裝置上執行，它會連線到 IoT 中樞上的裝置特定端點，並且傳送模擬的溫度和溼度遙測。 

### <a name="install-cocoapods"></a>安裝 CocoaPods

CocoaPods 可針對使用第三方程式庫的 iOS 專案管理相依性。

在本機終端機視窗中，巡覽至您在必要條件中下載的 Azure-IoT-Samples-iOS 資料夾。 然後，巡覽至範例專案：

```sh
cd quickstart/sample-device
```

確定已關閉 XCode，然後執行下列命令，以安裝在 **podfile** 檔案中宣告的 CocoaPods：

```sh
pod install
```

在安裝您的專案所需的 pod 時，安裝命令也建立了 XCode 工作區檔案，而該檔案已經設定為將 pod 使用於相依性。 

### <a name="run-the-sample-application"></a>執行範例應用程式 

1. 在 XCode 中開啟範例工作區。

   ```sh
   open "MQTT Client Sample.xcworkspace"
   ```

2. 展開 [MQTT 用戶端範例] 專案，然後展開同名的資料夾。  
3. 開啟 **ViewController.swift** 以便在 XCode 中編輯。 
4. 搜尋 **connectionString** 變數，並使用您先前記下的裝置連接字串來更新此值。
5. 儲存您的變更。 
6. 使用 [建置並執行] 按鈕或 **Command + r** 按鍵組合，在裝置模擬器中執行專案。 

   ![執行專案](media/quickstart-send-telemetry-ios/run-sample.png)

7. 當模擬器開啟時，在範例應用程式中選取 [啟動]。

下列螢幕擷取畫面顯示應用程式將模擬的遙測傳送到 IoT 中樞時的一些範例輸出：

   ![執行模擬的裝置](media/quickstart-send-telemetry-ios/view-d2c.png)

## <a name="read-the-telemetry-from-your-hub"></a>從您的中樞讀取遙測

您在 XCode 模擬器上執行的範例應用程式會顯示裝置所傳送訊息的相關資料。 您也可以透過 IoT 中樞檢視所收到的資料。 IoT 中樞 CLI 擴充功能可以連線到 IoT 中樞上的服務端**事件**端點。 擴充功能會接收模擬裝置所傳送的「裝置到雲端」訊息。 IoT 中樞後端應用程式通常在雲端中執行，以接收和處理「裝置到雲端」訊息。

在 Azure Cloud Shell 中執行下列命令，以您的 IoT 中樞名稱取代 `YourIoTHubName`：

```azurecli-interactive
az iot hub monitor-events --device-id myiOSdevice --hub-name YourIoTHubName
```

下列螢幕擷取畫面顯示由模擬裝置傳送遙測至中樞時，擴充功能接收遙測的輸出：

下列螢幕擷取畫面顯示您在本機終端機視窗中看到的遙測類型：

![檢視遙測](media/quickstart-send-telemetry-ios/view-telemetry.png)

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

在本文中，您已設定 IoT 中樞、註冊裝置，將模擬的遙測從 iOS 裝置傳送到中樞，並從中樞讀取遙測。 

若要了解如何從後端應用程式控制您的模擬裝置，請繼續下一個快速入門。

> [!div class="nextstepaction"]
> [快速入門：控制連線到 IoT 中樞的裝置](quickstart-control-device-node.md)

<!-- Links -->
[lnk-process-d2c-tutorial]: tutorial-routing.md
[lnk-device-management]: iot-hub-node-node-device-management-get-started.md
[lnk-iot-edge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-connect-device]: https://azure.microsoft.com/develop/iot/
