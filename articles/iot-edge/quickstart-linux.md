---
title: 快速入門 Azure IoT Edge + Linux | Microsoft Docs
description: 在本快速入門中，請了解如何將預先建置的程式碼從遠端部署至 IoT Edge 裝置。
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 08/14/2018
ms.topic: quickstart
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: 9683a33efba427e83b2ff27ec57d2d437c61d5ce
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166267"
---
# <a name="quickstart-deploy-your-first-iot-edge-module-to-a-linux-x64-device"></a>快速入門：將您的第一個 IoT Edge 模組部署至 Linux x64 裝置

Azure IoT Edge 會將雲端的強大功能移至您的物聯網裝置。 在本快速入門中，了解如何使用雲端介面將預先建置的程式碼從遠端部署到 IoT Edge 裝置。

在此快速入門中，您將了解如何：

1. 建立 IoT 中樞。
2. 向 IoT 中樞註冊 IoT Edge 裝置。
3. 在裝置上安裝並啟動 IoT Edge 執行階段。
4. 將模組從遠端部署到 IoT Edge 裝置。

![快速入門架構][2]

本快速入門會讓 Linux 電腦或虛擬機器變成 IoT Edge 裝置。 接著，您可以從 Azure 入口網站將模組部署至裝置。 您在本快速入門中部署的模組是一個模擬感應器，會產生溫度、溼度和壓力資料。 其他 Azure IoT Edge 教學課程會以您在此所做的工作為基礎，部署模組來分析模擬資料以產生商業見解。 

如果您沒有使用中的 Azure 訂用帳戶，請在開始前建立[免費帳戶][lnk-account]。


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

您可以使用 Azure CLI 來完成本快速入門中的許多步驟，而且 Azure IoT 有擴充功能可供啟用其他功能。 

將 Azure IoT 擴充功能新增至 Cloud Shell 執行個體。

   ```azurecli-interactive
   az extension add --name azure-cli-iot-ext
   ```
   
## <a name="prerequisites"></a>必要條件

雲端資源： 

* 一個資源群組，用以管理本快速入門中使用的所有資源。 

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus
   ```

IoT Edge 裝置：

* 當作 IoT Edge 裝置的 Linux 裝置或虛擬機器。 如果您想要在 Azure 中建立虛擬機器，請使用下列命令快速展開作業：

   ```azurecli-interactive
   az vm create --resource-group IoTEdgeResources --name EdgeVM --image Canonical:UbuntuServer:16.04-LTS:latest --admin-username azureuser --generate-ssh-keys --size Standard_B1ms
   ```

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

使用 Azure CLI 建立 IoT 中樞，開始進行此快速入門。 

![建立 IoT 中樞][3]

此快速入門適用於 IoT 中樞的免費層級。 如果您在過去已使用過 IoT 中樞，並已建立可用的中樞，您可以使用該 IoT 中樞。 每個訂用帳戶只能有一個免費的 IoT 中樞。 

下列程式碼會在資源群組 **IoTEdgeResources** 中建立免費的 **F1** 中樞。 請以 IoT 中樞的唯一名稱取代 {hub_name}。

   ```azurecli-interactive
   az iot hub create --resource-group IoTEdgeResources --name {hub_name} --sku F1 
   ```

   如果因您的訂用帳戶中已有免費中樞而發生錯誤，請將 SKU 變更為 **S1**。 

## <a name="register-an-iot-edge-device"></a>註冊 IoT Edge 裝置

向新建立的 IoT 中樞註冊 IoT Edge 裝置。 
![註冊裝置][4]

建立模擬裝置的裝置身分識別，以便與 IoT 中樞通訊。 裝置身分識別存在於雲端，您可以使用唯一的裝置連接字串，讓實體裝置與裝置身分識別建立關聯。 

由於 IoT Edge 裝置的行為和管理方式不同於典型 IoT 裝置，您從開頭就要將此宣告為 IoT Edge 裝置。 

1. 在 Azure Cloud Shell 中輸入下列命令，以在中樞內建立名為 **myEdgeDevice** 的裝置。

   ```azurecli-interactive
   az iot hub device-identity create --hub-name {hub_name} --device-id myEdgeDevice --edge-enabled
   ```

1. 擷取裝置的連接字串，此字串將作為連結實體裝置與其在 IoT 中樞內的身分識別。 

   ```azurecli-interactive
   az iot hub device-identity show-connection-string --device-id myEdgeDevice --hub-name {hub_name}
   ```

1. 複製連接字串，並加以儲存。 您將在下一節中使用此值設定 IoT Edge 執行階段。 


## <a name="install-and-start-the-iot-edge-runtime"></a>安裝並啟動 IoT Edge 執行階段

在您的 IoT Edge 裝置上安裝並啟動 Azure IoT Edge 執行階段。 
![註冊裝置][5]

IoT Edge 執行階段會在所有 IoT Edge 裝置上部署。 它有三個元件。 **IoT Edge 安全性精靈**會在每次 Edge 裝置開機時啟動，並藉由啟動 IoT Edge 代理程式來啟動該裝置。 **IoT Edge 代理程式**有助於在 IoT Edge 裝置 (包括 IoT Edge 中樞) 上部署及監視模組。 **IoT Edge 中樞**會管理 IoT Edge 裝置上的模組通訊，以及裝置與 IoT 中樞之間的通訊。 

在執行階段設定期間，您必須提供裝置連接字串。 請使用您從 Azure CLI 擷取到的字串。 這個字串會讓實體裝置與 Azure 中的 IoT Edge 裝置身分識別建立關聯。 

在您已準備好作為 IoT Edge 裝置的 Linux 機器或 VM 中，完成下列步驟。 

### <a name="register-your-device-to-use-the-software-repository"></a>註冊裝置以使用軟體存放庫

要執行 IoT Edge 執行階段所需的套件可於軟體存放庫中進行管理。 請設定 IoT Edge 裝置以存取此存放庫。 

本節中的步驟適用於執行 **Ubuntu 16.04** 的x64 裝置。 若要存取其他 Linux 版本或裝置架構上的軟體存放庫，請參閱[在 Linux (x64) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux.md)或[在 Linux (ARM32v7/armhf) 上安裝 Azure IoT Edge 執行階段](how-to-install-iot-edge-linux-arm.md)。

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

1. 更新 **apt-get**。

   ```bash
   sudo apt-get update
   ```

2. 安裝 **Moby**，此為容器執行階段。

   ```bash
   sudo apt-get install moby-engine
   ```

3. 安裝 Moby 的 CLI 命令。 

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

3. 新增 IoT Edge 裝置連接字串。 找出 **device_connection_string** 變數，並使用您在註冊裝置時複製的字串來更新其值。 這個連接字串會讓實體裝置與您在 Azure 中建立的裝置身分識別產生關聯。

4. 儲存並關閉檔案。 

   `CTRL + X`、`Y`, `Enter`

5. 重新啟動 IoT Edge 安全性精靈，以套用您的變更。

   ```bash
   sudo systemctl restart iotedge
   ```

>[!TIP]
>您必須要有提高的權限才能執行 `iotedge` 命令。 當您在安裝 IoT Edge 執行階段之後登出機器，並第一次重新登入時，您的權限將會自動更新。 在那之前，請在這些命令前面使用 **sudo**。 

### <a name="view-the-iot-edge-runtime-status"></a>檢視 IoT Edge 執行階段狀態

確認已成功安裝並設定執行階段。

1. 確認 Edge 安全性精靈正以系統服務的形式執行。

   ```bash
   sudo systemctl status iotedge
   ```

   ![檢視以系統服務的形式執行的 Edge 精靈](./media/quickstart-linux/iotedged-running.png)

2. 如果您需要對服務進行疑難排解，請擷取服務記錄。 

   ```bash
   journalctl -u iotedge
   ```

3. 檢視在您的裝置上執行的模組。 

   ```bash
   sudo iotedge list
   ```

   ![檢視裝置上的一個模組](./media/quickstart-linux/iotedge-list-1.png)

IoT Edge 裝置現已設定完成。 並已準備好執行雲端部署的模組。 

## <a name="deploy-a-module"></a>部署模組

從雲端管理您的 Azure IoT Edge 裝置，以部署會將遙測資料傳送到 IoT 中樞的模組。
![註冊裝置][6]

[!INCLUDE [iot-edge-deploy-module](../../includes/iot-edge-deploy-module.md)]

## <a name="view-generated-data"></a>檢視產生的資料

在此快速入門中，您可以建立新的 IoT Edge 裝置，並在其中安裝 IoT Edge 執行階段。 然後，您會使用 Azure 入口網站來推送 IoT Edge 模組，讓其無須變更裝置本身就能在裝置上執行。 在此案例中，您推送的模組會建立可在教學課程中使用的環境資料。 

在您的 IoT Edge 裝置上再次開啟命令提示字元。 確認從雲端部署的模組正在 IoT Edge 裝置上執行：

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

您也可以在遙測資料送達您的 IoT 中樞時，使用[適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)加以檢視。 


## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 IoT Edge 教學課程，您可以使用在本快速入門中註冊和設定的裝置。 否則，您可以刪除您所建立的 Azure 資源，並從裝置中移除 IoT Edge 執行階段。 

### <a name="delete-azure-resources"></a>刪除 Azure 資源

如果您是在新的資源群組中建立虛擬機器和 IoT 中樞，您可以刪除該群組和所有相關聯的資源。 如果該資源群組中有您想要保留的項目，則只要刪除您要清除的個別資源即可。 

移除 **IoTEdgeResources** 群組。 

   ```azurecli-interactive
   az group delete --name IoTEdgeResources 
   ```

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
   sudo apt-get remove --purge moby-cli
   sudo apt-get remove --purge moby-engine
   ```

## <a name="next-steps"></a>後續步驟

本快速入門是所有 IoT Edge 教學課程的先決條件。 您可以繼續閱讀其他任何教學課程，以了解 Azure IoT Edge 可如何協助您將此資料轉換成具有優勢的商業洞見。

> [!div class="nextstepaction"]
> [使用 Azure 函式篩選感應器資料](tutorial-deploy-function.md)



<!-- Images -->
[0]: ./media/quickstart-linux/cloud-shell.png
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
[lnk-docker-ubuntu]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/ 
[lnk-account]: https://azure.microsoft.com/free
[lnk-portal]: https://portal.azure.com
[lnk-delete]: https://docs.microsoft.com/cli/azure/iot/hub?view=azure-cli-latest#az-iot-hub-delete

<!-- Anchor links -->
[anchor-register]: #register-an-iot-edge-device
