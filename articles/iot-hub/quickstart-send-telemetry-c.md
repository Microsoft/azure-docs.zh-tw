---
title: 將遙測傳送至 Azure IoT 中樞快速入門 (C) | Microsoft Docs
description: 在此快速入門中，您會執行兩個範例 C 應用程式，以將模擬的遙測資料傳送至 IoT 中樞以及從 IoT 中樞讀取遙測資料，以便在雲端中處理。
author: wesmc7777
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: mvc
ms.date: 08/27/2018
ms.author: wesmc
ms.openlocfilehash: 3fa4c536313375ed88f6f0223218a663d4be3eb3
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49364762"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-the-telemetry-from-the-hub-with-a-back-end-application-c"></a>快速入門：將遙測從裝置傳送至 IoT 中樞，並使用後端應用程式從中樞讀取遙測 (C)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

IoT 中樞是一項 Azure 服務，可讓您從 IoT 裝置將大量的遙測擷取到雲端進行儲存或處理。 在此快速入門中，您透過 IoT 中樞，將遙測從模擬裝置應用程式傳送到後端應用程式以進行處理。

此快速入門使用來自[適用於 C 的 Azure IoT 裝置 SDK](iot-hub-device-sdk-c-intro.md) 的 C 範例應用程式來將遙測資料傳送到 IoT 中樞。 Azure IoT 裝置 SDK 是針對可攜性與廣泛的平台相容性以 [ANSI C (C99)](https://wikipedia.org/wiki/C99) 所撰寫。 執行範例應用程式之前，您將先建立 IoT 中樞並向該中樞註冊模擬裝置。

此文章是針對 Windows 所撰寫，但您也可以在 Linux 上完成此快速入門。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

* 安裝 [Visual Studio 2017](https://www.visualstudio.com/vs/) 並啟用[使用 C++ 的桌面開發](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/)工作負載。
* 安裝最新版的 [Git](https://git-scm.com/download/)。

## <a name="prepare-the-development-environment"></a>準備開發環境

針對此快速入門，您將使用[適用於 C 的 Azure IoT 裝置 SDK](iot-hub-device-sdk-c-intro.md)。 

您可以透過針對下列環境安裝套件與程式庫，來使用該 SDK：

* **Linux**：apt-get 套件可從 Ubuntu 16.04 與 18.04 取得，CPU 結構如下：amd64、arm64、armhf 與 i386。 如需詳細資訊，請參閱[在Ubuntu 上使用 apt-get 來建立 C 裝置用戶端專案](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/ubuntu_apt-get_sample_setup.md) \(英文\)。

* **mbed**：針對在 mbed 上建立裝置應用程式的開發人員，我們已發佈可讓您快速了解如何使用 Azure IoT 中樞的程式庫與範例。 如需詳細資訊，請參閱[使用 mbed 程式庫](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#mbed) \(英文\)。

* **Arduino**：若您在 Arduino 上進行開發，您可以利用 Arduino IDE 程式庫管理員中可用的 Azure IoT 程式庫。 如需詳細資訊，請參閱 [適用於 Arduino 的 Azure IoT 中樞](https://github.com/azure/azure-iot-arduino)。

* **iOS**：IoT 中樞裝置 SDK 能以 CocoaPod 形式取得並用於 Mac 與 iOS 裝置開發。 如需詳細資訊，請參閱 [Microsoft Azure IoT 的 iOS 範例](https://cocoapods.org/pods/AzureIoTHubClient) \(英文\)。

不過，在此快速入門中，您將會準備用來從 GitHub 複製並建置 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) 的開發環境。 GitHub 上的 SDK 包括此快速入門中使用的範例程式碼。 


1. 下載 [CMake 建置系統](https://cmake.org/download/)3.11.4 版。 請確認下載的二進位檔使用對應的密碼編譯雜湊值。 下列範例使用 Windows PowerShell 來驗證 x64 MSI 發行版本 3.11.4 的密碼編譯雜湊：

    ```PowerShell
    PS C:\Downloads> $hash = get-filehash .\cmake-3.11.4-win64-x64.msi
    PS C:\Downloads> $hash.Hash -eq "56e3605b8e49cd446f3487da88fcc38cb9c3e9e99a20f5d4bd63e54b7a35f869"
    True
    ```
    
    在撰寫此文章時，CMake 網站上列出了3.11.4 版的下列雜湊值：

    ```
    6dab016a6b82082b8bcd0f4d1e53418d6372015dd983d29367b9153f1a376435  cmake-3.11.4-Linux-x86_64.tar.gz
    72b3b82b6d2c2f3a375c0d2799c01819df8669dc55694c8b8daaf6232e873725  cmake-3.11.4-win32-x86.msi
    56e3605b8e49cd446f3487da88fcc38cb9c3e9e99a20f5d4bd63e54b7a35f869  cmake-3.11.4-win64-x64.msi
    ```

    在開始安裝 `CMake` **之前**，請務必將 Visual Studio 先決條件 (Visual Studio 和「使用 C++ 進行桌面開發」工作負載) 安裝在您的機器上。 在符合先決條件，並且驗證過下載項目之後，請安裝 CMake 建置系統。

2. 開啟命令提示字元或 Git Bash 殼層。 執行下列命令以複製 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) GitHub 存放庫：
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```
    此存放庫的大小目前約為 220 MB。 預期此作業需要幾分鐘的時間才能完成。


3. 在 git 存放庫的根目錄中建立 `cmake` 子目錄，並瀏覽至該資料夾。 

    ```cmd/sh
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

4. 執行下列命令 (該命令會建置您開發用戶端平台特有的 SDK 版本)。 `cmake` 目錄中會產生模擬裝置的 Visual Studio 解決方案。 

    ```cmd
    cmake ..
    ```
    
    如果 `cmake` 找不到 C++ 編譯，您在執行上述命令時，可能會收到建置錯誤。 如果發生這種情況，請嘗試在 [Visual Studio 命令提示字元](https://docs.microsoft.com/dotnet/framework/tools/developer-command-prompt-for-vs)中執行此命令。 

    建置成功後，最後幾行輸出會類似於下列輸出：

    ```cmd/sh
    $ cmake ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```


## <a name="create-an-iot-hub"></a>建立 IoT 中樞

[!INCLUDE [iot-hub-quickstarts-create-hub](../../includes/iot-hub-quickstarts-create-hub.md)]

## <a name="register-a-device"></a>註冊裝置

裝置必須向的 IoT 中樞註冊，才能進行連線。 在此節中，您將會使用 Azure Cloud Shell 搭配 [IoT 擴充功能](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot?view=azure-cli-latest)來註冊模擬裝置。

1. 在 Azure Cloud Shell 中執行下列命令，以新增 IoT 中樞 CLI 擴充功能和建立裝置身分識別。 

   **YourIoTHubName**：以您為 IoT 中樞選擇的名稱取代此預留位置。

   **MyCDevice**：這是為已註冊裝置指定的名稱。 使用所示的 MyCDevice。 如果您為裝置選擇不同的名稱，則也必須在本文中使用該名稱，並先在範例應用程式中更新該裝置名稱，再執行應用程式。

    ```azurecli-interactive
    az extension add --name azure-cli-iot-ext
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyCDevice
    ```

2. 在 Azure Cloud Shell 中執行下列命令，以針對您剛註冊的裝置取得_裝置連接字串_：

   **YourIoTHubName**：以您為 IoT 中樞選擇的名稱取代此預留位置。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name YourIoTHubName --device-id MyCDevice --output table
    ```

    記下裝置連接字串，它看起來如下：

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    您稍後會在快速入門中使用此值。

## <a name="send-simulated-telemetry"></a>傳送模擬的遙測

模擬裝置應用程式會連線到您 IoT 中樞上的裝置特定端點，並且以模擬遙測形式傳送字串。

1. 使用文字編輯器開啟 iothub_convenience_sample.c 原始程式檔並檢閱用於傳送遙測的範例程式碼。 該檔案位於下列位置：

    ```
    \azure-iot-sdk-c\iothub_client\samples\iothub_convenience_sample\iothub_convenience_sample.c
    ```

2. 尋找 `connectionString` 常數的宣告：

    ```C
    /* Paste in your device connection string  */
    static const char* connectionString = "[device connection string]";
    ```
    使用先前所記錄的裝置連接字串來取代 `connectionString` 常數的值。 接著，將您的變更儲存到 **iothub_convenience_sample.c**。

3. 在本機終端機視窗中，瀏覽到您在 Azure IoT SDK 中建立之 CMake 目錄中的 *iothub_convenience_sample* 專案目錄。

    ```
    cd /azure-iot-sdk-c/cmake/iothub_client/samples/iothub_convenience_sample
    ```

4. 在本機終端機視窗中執行 CMake，以使用已更新的 `connectionString` 值建置範例：

    ```cmd/sh
    cmake --build . --target iothub_convenience_sample --config Debug
    ```

5. 在本機終端機視窗中，執行下列命令以執行模擬裝置應用程式：

    ```cmd/sh
    Debug\iothub_convenience_sample.exe
    ```

    下列螢幕擷取畫面顯示模擬裝置應用程式將遙測傳送到 IoT 中樞時的輸出：

    ![執行模擬的裝置](media/quickstart-send-telemetry-c/simulated-device-app.png)

## <a name="read-the-telemetry-from-your-hub"></a>從您的中樞讀取遙測


在此節中，您將使用 Azure Cloud Shell 搭配 [IoT 擴充功能](https://docs.microsoft.com/cli/azure/ext/azure-cli-iot-ext/iot?view=azure-cli-latest)來監視由模擬裝置傳送的裝置訊息。

1. 使用 Azure Cloud Shell，執行下列命令以連線到您的 IoT 中樞並讀取訊息：

   **YourIoTHubName**：以您為 IoT 中樞選擇的名稱取代此預留位置。

    ```azurecli-interactive
    az iot hub monitor-events --hub-name YourIoTHubName --output table
    ```

    ![使用 Azure CLI 來閱讀裝置訊息](media/quickstart-send-telemetry-c/read-device-to-cloud-messages-app.png)

    

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已經設定 IoT 中樞、註冊裝置、使用 C 應用程式將模擬的遙測傳送到中樞，並使用 Azure Cloud Shell 從中樞讀取遙測。

若要深入了解如何搭配 Azure IoT Hub C SDK 進行開發，請繼續閱讀下列操作指南：

> [!div class="nextstepaction"]
> [使用 Azure IoT 中樞 C SDK 進行開發](iot-hub-devguide-develop-for-constrained-devices.md)