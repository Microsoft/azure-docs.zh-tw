---
title: 將一般 Node.js 用戶端應用程式連線至 Azure IoT Central | Microsoft Docs
description: 如何以裝置開發人員身分將一般 Node.js 裝置連線到 Azure IoT 中心應用程式。
author: tbhagwat3
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 55ce85702804d99d806220d7f0a4ea0820975f4f
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39206032"
---
# <a name="connect-a-generic-client-application-to-your-azure-iot-central-application-nodejs"></a>將一般用戶端應用程式連線到 Azure IoT 中心應用程式 (Node.js)

本文說明如何以裝置開發人員身分，將代表實體裝置的一般 Node.js 應用程式連線到 Microsoft Azure IoT 中心應用程式。

## <a name="before-you-begin"></a>開始之前

若要完成這篇文章中的步驟，您需要下列項目︰

1. Azure IoT Central 應用程式。 如需詳細資訊，請參閱[建立 Azure IoT 中心應用程式](howto-create-application.md)。
1. 已安裝 [Node.js](https://nodejs.org/) 4.0.0 版或更新版本的開發電腦。 您可以在命令列執行 `node --version` 來檢查版本。 Node.js 適用於多種作業系統。

## <a name="create-a-device-template"></a>建立裝置範本

在 Azure IoT 中心應用程式中，您必須具有已定義下列量值和裝置屬性的裝置範本：

### <a name="telemetry-measurements"></a>遙測量值

在 [量值] 頁面中新增下列遙測：

| 顯示名稱 | 欄位名稱  | Units | Min | max | 小數位數 |
| ------------ | ----------- | ----- | --- | --- | -------------- |
| 溫度  | 溫度 | F     | 60  | 110 | 0              |
| 溼度     | 溼度    | %     | 0   | 100 | 0              |
| 壓力     | pressure    | kPa   | 80  | 110 | 0              |

> [!NOTE]
  遙測量值的資料類型為 double。

在裝置範本中輸入完全如上表所示的欄位名稱。 如果欄位名稱不相符，則無法在應用程式中顯示遙測。

### <a name="state-measurements"></a>狀態量值

在 [量值] 頁面中新增下列狀態：

| 顯示名稱 | 欄位名稱  | 值 1 | 顯示名稱 | 值 2 | 顯示名稱 |
| ------------ | ----------- | --------| ------------ | ------- | ------------ | 
| 風扇模式     | fanmode     | 1       | 執行中      | 0       | 已停止      |

> [!NOTE]
  狀態量值的資料類型為字串。

在裝置範本中輸入完全如上表所示的欄位名稱。 如果欄位名稱不相符，則無法在應用程式中顯示狀態。

### <a name="event-measurements"></a>事件量值

在 [量值] 頁面中新增下列事件：

| 顯示名稱 | 欄位名稱  | 嚴重性 |
| ------------ | ----------- | -------- |
| 過熱  | overheat    | Error    |

> [!NOTE]
  事件量值的資料類型為字串。

### <a name="device-properties"></a>裝置屬性

在**屬性頁面**中新增下列裝置屬性：

| 顯示名稱        | 欄位名稱        | 資料類型 |
| ------------------- | ----------------- | --------- |
| 序號       | serialNumber      | text      |
| 裝置製造商 | manufacturer      | text      |

在裝置範本中輸入完全如上表所示的欄位名稱。 如果欄位名稱不相符，應用程式就無法顯示屬性值。

### <a name="settings"></a>設定

在**設定頁面**中新增下列**數目**設定：

| 顯示名稱    | 欄位名稱     | Units | 小數位數 | Min | max  | Initial |
| --------------- | -------------- | ----- | -------- | --- | ---- | ------- |
| 風扇速度       | fanSpeed       | rpm   | 0        | 0   | 3000 | 0       |
| 設定溫度 | setTemperature | F     | 0        | 20  | 200  | 80      |

在裝置範本中輸入完全如上表所示的欄位名稱。 如果欄位名稱不相符，則裝置無法接收設定值。

## <a name="add-a-real-device"></a>新增真實裝置

在 Azure IoT 中心應用程式中，從您建立的裝置範本新增真實裝置，並記下裝置連接字串。 如需詳細資訊，請參閱[將真實裝置新增至 Azure IoT 中心應用程式](tutorial-add-device.md)。

### <a name="create-a-nodejs-application"></a>建立 Node.js 應用程式

下列步驟示範如何建立用戶端應用程式，以實作您新增至應用程式的真實裝置。

1. 在電腦上建立名為 `connected-air-conditioner-adv` 的資料夾。 在命令列環境中瀏覽至該資料夾。

1. 若要初始化 Node.js 專案，請執行下列命令：

    ```cmd/sh
    npm init
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

1. 在 `connected-air-conditioner-adv` 資料夾中建立一個名為 **connectedAirConditionerAdv.js** 的檔案。

1. 在 **connectedAirConditionerAdv.js** 檔案的開頭，新增下列 `require` 陳述式：

    ```javascript
    "use strict";

    // Use the Azure IoT device SDK for devices that connect to Azure IoT Central.
    var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    var ConnectionString = require('azure-iot-device').ConnectionString;
    ```

1. 將下列變數宣告新增至檔案：

    ```javascript
    var connectionString = '{your device connection string}';
    var targetTemperature = 0;
    var client = clientFromConnectionString(connectionString);
    ```

    以您的裝置連接字串更新預留位置 `{your device connection string}`。 當您新增真實裝置時，您已從連線詳細資料頁面複製此值。 此範例中，我們將 `targetTemperature` 初始化為零，您可以選擇性地採用裝置的目前讀數或裝置對應項的值。 

1. 若要將遙測、狀態和事件量值傳送至 Azure IoT 中心應用程式，請將下列函式新增至檔案：

    ```javascript
    // Send device measurements.
    function sendTelemetry() {
      var temperature = targetTemperature + (Math.random() * 15);
      var humidity = 70 + (Math.random() * 10);
      var pressure = 90 + (Math.random() * 5);
      var fanmode = 0;
      var data = JSON.stringify({ 
        temperature: temperature, 
        humidity: humidity, 
        pressure: pressure,
        fanmode: (temperature > 25) ? "1" : "0",
        overheat: (temperature > 35) ? "ER123" : undefined });
      var message = new Message(data);
      client.sendEvent(message, (err, res) => console.log(`Sent message: ${message.getData()}` +
        (err ? `; error: ${err.toString()}` : '') +
        (res ? `; status: ${res.constructor.name}` : '')));
    }
    ```

    1. 若要將裝置屬性傳送至 Azure IoT 中心應用程式，請將下列函式新增至檔案：

    ```javascript
    // Send device properties.
    function sendDeviceProperties(twin) {
      var properties = {
        serialNumber: '123-ABC',
        manufacturer: 'Contoso'
      };
      twin.properties.reported.update(properties, (err) => console.log(`Sent device properties; ` +
        (err ? `error: ${err.toString()}` : `status: success`)));
    }
    ```

1. 若要定義您的裝置所回應的設定，請新增下列定義：

    ```javascript
    // Add any settings your device supports,
    // mapped to a function that is called when the setting is changed.
    var settings = {
      'fanSpeed': (newValue, callback) => {
          // Simulate it taking 1 second to set the fan speed.
          setTimeout(() => {
            callback(newValue, 'completed');
          }, 1000);
      },
      'setTemperature': (newValue, callback) => {
        // Simulate the temperature setting taking two steps.
        setTimeout(() => {
          targetTemperature = targetTemperature + (newValue - targetTemperature) / 2;
          callback(targetTemperature, 'pending');
          setTimeout(() => {
            targetTemperature = newValue;
            callback(targetTemperature, 'completed');
          }, 5000);
        }, 5000);
      }
    };
    ```

1. 若要從 Azure IoT 中心應用程式處理更新後的設定，請新增下列檔案：

    ```javascript
    // Handle settings changes that come from Azure IoT Central via the device twin.
    function handleSettings(twin) {
      twin.on('properties.desired', function (desiredChange) {
        for (let setting in desiredChange) {
          if (settings[setting]) {
            console.log(`Received setting: ${setting}: ${desiredChange[setting].value}`);
            settings[setting](desiredChange[setting].value, (newValue, status, message) => {
              var patch = {
                [setting]: {
                  value: newValue,
                  status: status,
                  desiredVersion: desiredChange.$version,
                  message: message
                }
              }
              twin.properties.reported.update(patch, (err) => console.log(`Sent setting update for ${setting}; ` +
                (err ? `error: ${err.toString()}` : `status: success`)));
            });
          }
        }
      });
    }
    ```

1. 新增下列內容來完成 Azure IoT 中心連線，並且在用戶端程式碼中勾取函式：

    ```javascript
    // Handle device connection to Azure IoT Central.
    var connectCallback = (err) => {
      if (err) {
        console.log(`Device could not connect to Azure IoT Central: ${err.toString()}`);
      } else {
        console.log('Device successfully connected to Azure IoT Central');

        // Send telemetry measurements to Azure IoT Central every 1 second.
        setInterval(sendTelemetry, 1000);

        // Get device twin from Azure IoT Central.
        client.getTwin((err, twin) => {
          if (err) {
            console.log(`Error getting device twin: ${err.toString()}`);
          } else {
            // Send device properties once on device start up.
            sendDeviceProperties(twin);
            // Apply device settings and handle changes to device settings.
            handleSettings(twin);
          }
        });
      }
    };

    // Start the device (connect it to Azure IoT Central).
    client.open(connectCallback);
    ```

## <a name="run-your-nodejs-application"></a>執行 Node.js 應用程式

在命令列環境中執行下列命令：

```cmd/sh
node connectedAirConditionerAdv.js
```

身為 Azure IoT 中心應用程式的操作者，您可以針對真實裝置：

* 在 [量值] 頁面上檢視遙測：

    ![檢視遙測](media/howto-connect-nodejs/viewtelemetry.png)

* 在 [屬性] 頁面上檢視從裝置傳送的裝置屬性值。

    ![檢視裝置屬性](media/howto-connect-nodejs/viewproperties.png)

* 從 [設定] 頁面設定風扇速度和目標溫度。

    ![設定風扇速度](media/howto-connect-nodejs/setfanspeed.png)

## <a name="next-steps"></a>後續步驟

您現在已了解如何將一般 Node.js 用戶端連線至 Azure IoT 中心應用程式，以下是建議的後續步驟：
* [準備及連線 Raspberry Pi](howto-connect-raspberry-pi-python.md)
<!-- Next how-tos in the sequence -->
