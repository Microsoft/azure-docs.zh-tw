---
title: 檢查裝置對 Azure IoT 中樞的連線能力
description: 在開發期間使用 IoT 中樞工具，排解將裝置連線至您的 IoT 中樞時所發生的問題。
services: iot-hub
author: dominicbetts
manager: timlt
ms.author: dobett
ms.custom: mvc
ms.date: 05/29/2018
ms.topic: tutorial
ms.service: iot-hub
ms.openlocfilehash: 5a09211dc134cdb372a9712c1f45d7c44660a759
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966855"
---
# <a name="tutorial-use-a-simulated-device-to-test-connectivity-with-your-iot-hub"></a>教學課程：使用模擬裝置來測試與 IoT 中樞的連線能力

在本教學課程中，您會使用 Azure IoT 中樞入口網站工具和 Azure CLI 命令來測試裝置連線能力。 本教學課程也會使用您在桌面機器上執行的簡易裝置模擬器。

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 檢查您的裝置驗證
> * 檢查裝置到雲端的連線能力
> * 檢查雲端到裝置的連線能力
> * 檢查裝置對應項同步處理

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>必要條件

您在本教學課程中執行的 CLI 指令碼會使用 [Azure CLI 的 Microsoft Azure IoT 擴充功能](https://github.com/Azure/azure-iot-cli-extension/blob/master/README.md)。 若要安裝此擴充功能，請執行下列 CLI 命令：

```azurecli-interactive
az extension add --name azure-cli-iot-ext
```

您在本教學課程中執行的裝置模擬器應用程式，是使用 Node.js 撰寫的。 您的開發電腦上需要 Node.js 4.x.x 版或更高版本。

您可以從 [nodejs.org](https://nodejs.org) 下載適用於多種平台的 Node.js。

您可以使用下列命令，以確認開發電腦上目前的 Node.js 版本：

```cmd/sh
node --version
```

從 https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip 下載範例裝置模擬器 Node.js 專案，並將 ZIP 封存檔解壓縮。

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

如果您在上一個快速入門或教學課程中建立了免費或標準層 IoT 中樞，則可以略過此步驟。

[!INCLUDE [iot-hub-tutorials-create-free-hub](../../includes/iot-hub-tutorials-create-free-hub.md)]

## <a name="check-device-authentication"></a>檢查裝置驗證

裝置必須先對您的中樞進行驗證，才能與中樞交換資料。 您可以在入口網站的 [裝置管理] 區段中使用 [IoT 裝置] 工具來管理您的裝置，並檢查裝置所使用的驗證金鑰。 在教學課程的這一節中，您會新增新的測試裝置、擷取其金鑰，以及確認測試裝置可連線至中樞。 其後您會重設驗證金鑰，以觀察裝置嘗試使用過期的金鑰時會發生什麼情況。 教學課程的這一節會使用 Azure 入口網站來建立、管理及監視裝置，以及範例 Node.js 裝置模擬器。

登入入口網站，然後瀏覽至 IoT 中樞。 接著，瀏覽至 [IoT 裝置] 工具：

![IoT 裝置工具](media/tutorial-connectivity/iot-devices-tool.png)

若要註冊新裝置，請按一下 [+ 新增]，將 [裝置識別碼] 設定為 **MyTestDevice**，然後按一下 [儲存]：

![新增裝置](media/tutorial-connectivity/add-device.png)

若要擷取 **MyTestDevice** 的連接字串，請在裝置清單中按一下該裝置，然後複製 [連接字串 - 主要金鑰] 值。 連接字串會包含裝置的共用存取金鑰。

![擷取裝置連接字串](media/tutorial-connectivity/copy-connection-string.png)

若要模擬將遙測資料傳送至 IoT 中樞的 **MyTestDevice**，請執行您先前下載的 Node.js 模擬裝置應用程式。

在開發機器的終端機視窗中，瀏覽至您下載的範例 Node.js 專案的根資料夾。 然後，瀏覽至 **iot-hub\Tutorials\ConnectivityTests\simulated-device** 資料夾。

在終端機視窗中，執行下列命令以安裝模擬裝置應用程式所需的程式庫，並執行模擬裝置應用程式。 使用您在入口網站中新增裝置時所記下的裝置連接字串。

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

當裝置嘗試連線至您的中樞時，終端機視窗會顯示資訊：

![模擬裝置連線中](media/tutorial-connectivity/sim-1-connected.png)

您現在已使用 IoT 中樞所產生的裝置金鑰成功地從裝置進行驗證。

### <a name="reset-keys"></a>重設金鑰

在本節中，您會重設裝置金鑰，並觀察模擬裝置嘗試連線時所產生的錯誤。

若要重設 **MyTestDevice** 的主要裝置金鑰，請執行下列命令：

```azurecli-interactive
# Generate a new Base64 encoded key using the current date
read key < <(date +%s | sha256sum | base64 | head -c 32)

# Requires the IoT Extension for Azure CLI
# az extension add --name azure-cli-iot-ext

# Reset the primary device key for MyTestDevice
az iot hub device-identity update --device-id MyTestDevice --set authentication.symmetricKey.primaryKey=$key --hub-name {YourIoTHubName}
```

在開發機器的終端機視窗中，再次執行模擬裝置應用程式：

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

這次，您會在應用程式嘗試連線時看到驗證錯誤：

![連線錯誤](media/tutorial-connectivity/sim-1-fail.png)

### <a name="generate-shared-access-signature-sas-token"></a>產生共用存取簽章 (SAS) 權杖

如果您的裝置使用其中一個 IoT 中樞裝置 SDK，該 SDK 程式庫程式碼將會產生用來對中樞進行驗證的 SAS 權杖。 SAS 權杖會從您的中樞名稱、裝置名稱和裝置金鑰產生。

在某些情況下，例如雲端通訊協定閘道中或自訂驗證配置中，您可能需要自行產生 SAS 權杖。 若要對 SAS 產生碼的問題進行疑難排解，最好能夠產生已知良好的 SAS 權杖，以在測試期間使用。

> [!NOTE]
> SimulatedDevice-2.js 範例包含使用和不使用 SDK 來產生 SAS 權杖的範例。

若要使用 CLI 產生已知良好的 SAS 權杖，請執行下列命令：

```azurecli-interactive
az iot hub generate-sas-token --device-id MyTestDevice --hub-name {YourIoTHubName}
```

完整記下產生的 SAS 權杖文字。 SAS 權杖顯示如下：`'SharedAccessSignature sr=tutorials-iot-hub.azure-devices.net%2Fdevices%2FMyTestDevice&sig=....&se=1524155307'`

在開發機器的終端機視窗中，瀏覽至您下載的範例 Node.js 專案的根資料夾。 然後，瀏覽至 **iot-hub\Tutorials\ConnectivityTests\simulated-device** 資料夾。

在終端機視窗中，執行下列命令以安裝模擬的裝置應用程式所需的程式庫並執行模擬的裝置應用程式：

```cmd/sh
npm install
node SimulatedDevice-2.js "{Your SAS token}"
```

當裝置嘗試使用 SAS 權杖連線至您的中樞時，終端機視窗會顯示資訊：

![模擬裝置使用權杖連線中](media/tutorial-connectivity/sim-2-connected.png)

您現在已使用 CLI 命令所產生的測試 SAS 權杖成功地從裝置進行驗證。 **SimulatedDevice-2.js** 檔案包含範例程式碼，會說明如何在程式碼中產生 SAS 權杖。

### <a name="protocols"></a>通訊協定

裝置可以使用下列任何通訊協定連線至 IoT 中樞：

| 通訊協定 | 輸出連接埠 |
| --- | --- |
| MQTT |8883 |
| 透過 WebSocket 的 MQTT |443 |
| AMQP |5671 |
| 透過 WebSocket 的 AMQP |443 |
| HTTPS |443 |

如果輸出連接埠遭到防火牆封鎖，裝置就無法連線：

![已封鎖連接埠](media/tutorial-connectivity/port-blocked.png)

## <a name="check-device-to-cloud-connectivity"></a>檢查裝置到雲端的連線能力

裝置連線之後，通常會嘗試將遙測資料傳送至您的 IoT 中樞。 本節說明如何確認裝置所傳送的遙測資料是否送達您的中樞。

首先，請使用下列命令擷取模擬裝置目前的連接字串：

```azurecli-interactive
az iot hub device-identity show-connection-string --device-id MyTestDevice --output table --hub-name {YourIoTHubName}
```

若要執行會傳送訊息的模擬裝置，請在您下載的程式碼中瀏覽至 **iot-hub\Tutorials\ConnectivityTests\simulated-device** 資料夾。

在終端機視窗中，執行下列命令以安裝模擬的裝置應用程式所需的程式庫並執行模擬的裝置應用程式：

```cmd/sh
npm install
node SimulatedDevice-3.js "{your device connection string}"
```

當裝置將遙測資料傳送至您的中樞時，終端機視窗會顯示資訊：

![傳送訊息的模擬裝置](media/tutorial-connectivity/sim-3-sending.png)

您可以在入口網站中使用 [計量]，確認遙測訊息已送達您的 IoT 中樞：

![瀏覽至 IoT 中樞計量](media/tutorial-connectivity/metrics-portal.png)

在 [資源] 下拉式清單中選取您的 IoT 中樞，選取 [已傳送的遙測訊息] 作為計量，並將時間範圍設定為 [過去一小時]。 圖表會顯示模擬裝置所傳送訊息的彙總計數：

![顯示 IoT 中樞計量](media/tutorial-connectivity/metrics-active.png)

在您啟動模擬裝置後，可能要經過數分鐘才會產生計量。

## <a name="check-cloud-to-device-connectivity"></a>檢查雲端到裝置的連線能力

本節說明如何對裝置發出測試直接方法呼叫，以檢查雲端到裝置的連線能力。 您可以在開發機器上執行模擬裝置，以接聽來自中樞的直接方法呼叫。

在終端機視窗中，使用下列命令執行模擬裝置應用程式：

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

使用 CLI 命令對裝置呼叫直接方法：

```azurecli-interactive
az iot hub invoke-device-method --device-id MyTestDevice --method-name TestMethod --timeout 10 --method-payload '{"key":"value"}' --hub-name {YourIoTHubName}
```

模擬裝置在接收到直接方法呼叫時，會將訊息列印至主控台：

![模擬裝置接收直接方法呼叫](media/tutorial-connectivity/receive-method-call.png)

模擬裝置在成功接收到直接方法呼叫時，會將通知傳回至中樞：

![接收直接方法通知](media/tutorial-connectivity/method-acknowledgement.png)

## <a name="check-twin-synchronization"></a>檢查對應項同步處理

裝置會使用對應項來同步處理裝置與中樞之間的狀態。 在本節中，您會使用 CLI 命令將_所需的屬性_傳送至裝置，並讀取裝置所傳送的_報告屬性_。

您在本節中使用的模擬裝置會在每次啟動時將報告的屬性傳送至中樞，並且在每次接收到所需的屬性時將其列印至主控台。

在終端機視窗中，使用下列命令執行模擬裝置應用程式：

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

若要確認中樞已接收到來自裝置的報告屬性，請使用下列 CLI 命令：

```azurecli-interactive
az iot hub device-twin show --device-id MyTestDevice --hub-name {YourIoTHubName}
```

在命令的輸出中，您會在報告屬性區段中看到 **devicelaststarted** 屬性。 此屬性會顯示您上次啟動模擬裝置的時間和日期。

![檢視報告屬性](media/tutorial-connectivity/reported-properties.png)

若要確認中樞可將所需的屬性值傳送至裝置，請使用下列 CLI 命令：

```azurecli-interactive
az iot hub device-twin update --set properties.desired='{"mydesiredproperty":"propertyvalue"}' --device-id MyTestDevice --hub-name {YourIoTHubName}
```

模擬裝置在接收到來自中樞的所需屬性更新時，將會列印訊息：

![接收所需屬性](media/tutorial-connectivity/desired-properties.png)

除了在所需屬性變更產生時接收變更以外，模擬裝置也會在啟動時自動檢查是否有所需屬性。

## <a name="clean-up-resources"></a>清除資源

如果您不再需要 IoT 中樞，請在入口網站中刪除它和資源群組。 若要這麼做，請選取包含 IoT 中樞的 **tutorials-iot-hub-rg** 資源群組，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何檢查裝置金鑰、檢查裝置到雲端的連線能力、檢查雲端到裝置的連線能力，以及檢查裝置對應項同步處理。 若要深入了解如何監視您的 IoT 中樞，請瀏覽 IoT 中樞監視的操作說明文章。

> [!div class="nextstepaction"]
> [監視與診斷](iot-hub-monitor-resource-health.md)
