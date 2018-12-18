---
title: 在 Linux 上模擬 Azure IoT Edge | Microsoft Docs
description: 在本快速入門中，請了解如何將預先建置的程式碼從遠端部署至 IoT Edge 裝置。
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 07/02/2018
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: dfbe931bbe5887e9c0545558c4d2b2565718dd0a
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578485"
---
# <a name="quickstart-deploy-your-first-iot-edge-module-to-a-linux-x64-device"></a>快速入門：將您的第一個 IoT Edge 模組部署至 Linux x64 裝置

Azure IoT Edge 可讓您在裝置上執行分析和資料處理，而不必將所有的資料推送至雲端。 IoT Edge 教學課程說明如何部署不同類型的模組，但首先您需要能夠測試的裝置。 

在此快速入門中，您將了解如何：

1. 建立 IoT 中樞。
2. 向 IoT 中樞註冊 IoT Edge 裝置。
3. 啟動 IoT Edge 執行階段。
4. 將模組從遠端部署至 IoT Edge 裝置。

![教學課程架構][2]

本快速入門會讓 Linux 電腦或虛擬機器成為 IoT Edge 裝置。 接著，您可以從 Azure 入口網站將模組部署至裝置。 您在本快速入門中部署的模組是一個模擬感應器，會產生溫度、溼度和壓力資料。 其他 Azure IoT Edge 教學課程會以您在此所做的工作為基礎，部署模組來分析模擬資料以產生商業見解。 

如果您沒有使用中的 Azure 訂用帳戶，請在開始前建立[免費帳戶][lnk-account]。

## <a name="prerequisites"></a>必要條件

本快速入門會以 Linux 機器作為 IoT Edge 裝置。 如果您還沒有可用於測試的裝置，請依照[在 Azure 入口網站中建立 Linux 虛擬機器](../virtual-machines/linux/quick-create-portal.md)中的指示操作。 
* 您不需要依照這些步驟來安裝和執行 Web 伺服器。 在連線至虛擬機器後，您即可停止作業。  
* 在新的資源群組中建立虛擬機器，以供您在建立本快速入門的其餘 Azure 資源時使用。 為其指定可辨識的名稱，例如 *IoTEdgeResources*。 
* 測試 IoT Edge 並不需要非常大型的虛擬機器。 **B1ms** 的大小即已足夠。 

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

在 Azure 入口網站中建立 IoT 中樞，以開始進行此快速入門。
![建立 IoT 中樞][3]

[!INCLUDE [iot-hub-create-hub](../../includes/iot-hub-create-hub.md)]

## <a name="register-an-iot-edge-device"></a>註冊 IoT Edge 裝置

向新建立的 IoT 中樞註冊 IoT Edge 裝置。
![註冊裝置][4]

[!INCLUDE [iot-edge-register-device](../../includes/iot-edge-register-device.md)]


## <a name="install-and-start-the-iot-edge-runtime"></a>安裝並啟動 IoT Edge 執行階段

安裝並啟動您裝置上的 Azure IoT Edge 執行階段。 
![註冊裝置][5]

IoT Edge 執行階段會在所有 IoT Edge 裝置上部署。 它有三個元件。 **IoT Edge 安全性精靈**會在每次 Edge 裝置開機時啟動，並藉由啟動 IoT Edge 代理程式來啟動該裝置。 **IoT Edge 代理程式**有助於在 IoT Edge 裝置 (包括 IoT Edge 中樞) 上部署及監視模組。 **IoT Edge 中樞**會管理 IoT Edge 裝置上的模組通訊，以及裝置與 IoT 中樞之間的通訊。 

在您準備用於本快速入門的 Linux 機器或 VM 中，完成下列步驟中。 

### <a name="register-your-device-to-use-the-software-repository"></a>註冊裝置以使用軟體存放庫

要執行 IoT Edge 執行階段所需的套件可於軟體存放庫中進行管理。 請設定 IoT Edge 裝置以存取此存放庫。 

本節中的步驟適用於執行 **Ubuntu 16.04** 的裝置。 若要存取其他 Linux 版本上的軟體存放庫，請參閱[在 Linux (x64) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux.md)或[在 Linux (ARM32v7/armhf) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux-arm.md)。

1. 在要作為 IoT Edge 裝置的機器上，安裝存放庫組態。

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > ./microsoft-prod.list
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

2. 安裝用以存取存放庫的公開金鑰。

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

### <a name="install-a-container-runtime"></a>安裝容器執行階段

IoT Edge 執行階段是一組容器，而部署至 IoT Edge 裝置的邏輯會封裝為容器。 安裝容器執行階段，讓您的裝置準備好使用這些元件。

更新 **apt-get**。

   ```bash
   sudo apt-get update
   ```

安裝 **Moby**，此為容器執行階段。

   ```bash
   sudo apt-get install moby-engine
   ```

安裝 Moby 的 CLI 命令。 

   ```bash
   sudo apt-get install moby-cli
   ```

### <a name="install-and-configure-the-iot-edge-security-daemon"></a>安裝並設定 IoT Edge 安全性精靈

安全性精靈會安裝為系統服務，以便在每次裝置開機時啟動 IoT Edge 執行階段。 此安裝也包含某個版本的 **hsmlib**，它可讓安全性精靈與裝置的硬體安全性互動。 

1. 下載並安裝 IoT Edge 安全性精靈。 

   ```bash
   sudo apt-get update
   sudo apt-get install iotedge
   ```

2. 開啟 IoT Edge 組態檔。 此檔案受到保護，因此您可能必須使用提高的權限才能加以存取。
   
   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

3. 新增 IoT Edge 裝置連接字串。 找出 **device_connection_string** 變數，並使用您在註冊裝置時複製的字串來更新其值。

4. 儲存並關閉檔案。 

   `CTRL + X`、`Y`, `Enter`

4. 重新啟動 IoT Edge 安全性精靈。

   ```bash
   sudo systemctl restart iotedge
   ```

5. 確認 Edge 安全性精靈正以系統服務的形式執行。

   ```bash
   sudo systemctl status iotedge
   ```

   ![檢視以系統服務的形式執行的 Edge 精靈](./media/quickstart-linux/iotedged-running.png)

   您也可以執行下列命令，以查看 Edge 安全性精靈中的記錄：

   ```bash
   journalctl -u iotedge
   ```

6. 檢視在您的裝置上執行的模組。 

   >[!TIP]
   >受先您必須使用 *sudo* 來執行 `iotedge` 命令。 登出機器並重新登入以更新權限，其後您即可執行 `iotedge` 命令而不需要提升的權限。 

   ```bash
   sudo iotedge list
   ```

   ![檢視裝置上的一個模組](./media/quickstart-linux/iotedge-list-1.png)

## <a name="deploy-a-module"></a>部署模組

從雲端管理您的 Azure IoT Edge 裝置，以部署會將遙測資料傳送到 IoT 中樞的模組。
![註冊裝置][6]

[!INCLUDE [iot-edge-deploy-module](../../includes/iot-edge-deploy-module.md)]


## <a name="view-generated-data"></a>檢視產生的資料

在此快速入門中，您可以建立新的 IoT Edge 裝置，並在其中安裝 IoT Edge 執行階段。 然後，您會使用 Azure 入口網站來推送 IoT Edge 模組，讓其無須變更裝置本身就能在裝置上執行。 在此案例中，您推送的模組會建立可在教學課程中使用的環境資料。 

在執行模擬裝置的電腦上再次開啟命令提示字元。 確認從雲端部署的模組正在 IoT Edge 裝置上執行：

   ```bash
   sudo iotedge list
   ```

   ![在您的裝置上檢視三個模組](./media/quickstart-linux/iotedge-list-2.png)

檢視從 tempSensor 模組送出的訊息：

  ```bash
   sudo iotedge logs tempSensor -f 
   ```

登出再登入後，上述命令就不需要 *sudo*。

![從您的模組中檢視資料](./media/quickstart-linux/iotedge-logs.png)

如果您在記錄中看到的最後一行是 `Using transport Mqtt_Tcp_Only`，則表示溫度感應器模組可能正在等候連線至 Edge 中樞。 請嘗試終止此模組，並讓 Edge 代理程式重新啟動模組。 您可以使用 `sudo docker stop tempSensor` 命令來終止模組。

您也可以使用[適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)，來檢視裝置所傳送的遙測資料。 

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 IoT Edge 教學課程，您可以使用在本快速入門中註冊和設定的裝置。 否則，您可以刪除您所建立的 Azure 資源，並從裝置中移除 IoT Edge 執行階段。 

### <a name="delete-azure-resources"></a>刪除 Azure 資源

如果您是在新的資源群組中建立虛擬機器和 IoT 中樞，您可以刪除該群組和所有相關聯的資源。 如果該資源群組中有您想要保留的項目，則只要刪除您要清除的個別資源即可。 

若要移除資源群組，請依照下列步驟操作： 

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [資源群組]。
2. 在 [依名稱篩選...] 文字方塊中，輸入包含 IoT 中樞的資源群組名稱。 
3. 在結果清單中的資源群組右側，按一下 **...**，然後按一下 [刪除資源群組]。
4. 系統將會要求您確認是否刪除資源。 再次輸入您的資源群組名稱進行確認，然後按一下 [刪除]。 片刻過後，系統便會刪除該資源群組及其所有內含的資源。

### <a name="remove-the-iot-edge-runtime"></a>移除 IoT Edge 執行階段

如果您想要從裝置移除這些安裝，請使用下列命令。  

移除 IoT Edge 執行階段。

   ```bash
   sudo apt-get remove --purge iotedge
   ```

IoT Edge 執行階段移除後，它所建立的容器隨即停止，但仍會存在於您的裝置上。 檢視所有容器。

   ```bash
   sudo docker ps -a
   ```

刪除 IoT Edge 執行階段在您的裝置上建立的容器。 如果您將 tempSensor 容器命名為其他名稱，請變更其名稱。 

   ```bash
   sudo docker rm -f tempSensor
   sudo docker rm -f edgeHub
   sudo docker rm -f edgeAgent
   ```

移除容器執行階段。

   ```bash
   sudo apt-get remove --purge moby
   ```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立新的 IoT Edge 裝置，並使用 Azure IoT Edge 雲端介面將程式碼部署至裝置上。 現在，您已有模擬裝置，可產生其環境的相關原始資料。 

本快速入門是所有 IoT Edge 教學課程的先決條件。 您可以繼續閱讀任何教學課程，以了解 Azure IoT Edge 可如何協助您將此資料轉換成 Edge 上的商業見解。

> [!div class="nextstepaction"]
> [使用 Azure 函式篩選感應器資料](tutorial-deploy-function.md)


<!-- Images -->
[1]: ./media/quickstart-linux/view-module.png
[2]: ./media/quickstart-linux/install-edge-full.png
[3]: ./media/quickstart-linux/create-iot-hub.png
[4]: ./media/quickstart-linux/register-device.png
[5]: ./media/quickstart-linux/start-runtime.png
[6]: ./media/quickstart-linux/deploy-module.png
[7]: ./media/quickstart-linux/iotedged-running.png
[8]: ./media/tutorial-simulate-device-linux/running-modules.png
[9]: ./media/tutorial-simulate-device-linux/sensor-data.png

<!-- Links -->
[lnk-account]: https://azure.microsoft.com/free
[lnk-docker-ubuntu]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/ 
