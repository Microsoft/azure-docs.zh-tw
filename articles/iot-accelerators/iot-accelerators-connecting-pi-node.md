---
title: 以 Node.js 將 Raspberry Pi 佈建到遠端監視 - Azure | Microsoft Docs
description: 描述如何使用以 Node.js 編寫的應用程式，將 Raspberry Pi 裝置連線到遠端監視解決方案加速器。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 01/24/2018
ms.author: dobett
ms.openlocfilehash: 696bd6ec80f39e8a9f3418426a754ffc038171e2
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39325077"
---
# <a name="connect-your-raspberry-pi-device-to-the-remote-monitoring-solution-accelerator-nodejs"></a>將 Raspberry Pi 裝置連線到遠端監視解決方案加速器 (Node.js)

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

本教學課程示範如何將實體裝置連線到遠端監視解決方案加速器。 在本教學課程中，您使用的 Node.js 是適用於環境資源限制最小的絕佳選項。

### <a name="required-hardware"></a>必要的硬體

一部桌上型電腦，可讓您從遠端連線到 Raspberry Pi 上的命令列。

[適用於 Raspberry Pi 3 的 Microsoft IoT 入門套件](https://azure.microsoft.com/develop/iot/starter-kits/)或對等的元件。 本教學課程會使用套件中的下列項目：

- Raspberry Pi 3
- MicroSD 卡 (具有 NOOBS)
- USB Mini 連接線
- 乙太網路連接線

### <a name="required-desktop-software"></a>必要的桌面軟體

您需要透過桌上型電腦上的 SSH 用戶端，才能從遠端存取 Raspberry Pi 上的命令列。

- Windows 不包含 SSH 用戶端。 我們建議使用 [PuTTY](http://www.putty.org/)。
- 大部分的 Linux 散發套件和 Mac OS 都包含命令列 SSH 公用程式。 如需詳細資訊，請參閱[使用 Linux 或 Mac OS 的 SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)。

### <a name="required-raspberry-pi-software"></a>必要的 Raspberry Pi 軟體

如果您尚未這麼做，請在 Raspberry Pi 上安裝 Node.js 4.0.0 版或更新版本。 下列步驟向您示範如何在 Raspberry Pi 上安裝 Node.js v6：

1. 使用 `ssh` 連線至您的 Raspberry Pi。 如需詳細資訊，請參閱 [Raspberry Pi 網站](https://www.raspberrypi.org/)上的 [SSH (安全殼層)](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md)。

1. 若要更新 Raspberry Pi，請使用下列命令︰

    ```sh
    sudo apt-get update
    ```

1. 使用下列命令，從 Raspberry Pi 移除任何現有的 Node.js 安裝：

    ```sh
    sudo apt-get remove nodered -y
    sudo apt-get remove nodejs nodejs-legacy -y
    sudo apt-get remove npm  -y
    ```

1. 使用下列命令，下載 Node.js v6 並將其安裝在 Raspberry Pi 上：

    ```sh
    curl -sL https://deb.nodesource.com/setup_6.x | sudo bash -
    sudo apt-get install nodejs npm
    ```

1. 若要確認您已成功安裝 Node.js 6.11.4 版，請使用下列命令︰

    ```sh
    node --version
    ```

## <a name="create-a-nodejs-solution"></a>建立 Node.js 解決方案

使用 `ssh` 與 Raspberry Pi 的連線完成下列步驟：

1. 在 Raspberry Pi 的主資料夾中，建立名為 `remotemonitoring` 的資料夾。 在命令列中瀏覽至此資料夾：

    ```sh
    cd ~
    mkdir remotemonitoring
    cd remotemonitoring
    ```

1. 若要下載並安裝完成範例應用程式所需的套件，請執行下列命令︰

    ```sh
    npm install async azure-iot-device azure-iot-device-mqtt
    ```

1. 在 `remotemonitoring` 資料夾中，建立名為 **remote_monitoring.js** 的檔案。 在文字編輯器中開啟這個檔案。 在 Raspberry Pi 上，您可以使用 `nano` 或 `vi` 文字編輯器。

1. 在 **remote_monitoring.js** 檔案中，新增下列 `require` 陳述式︰

    ```nodejs
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var ConnectionString = require('azure-iot-device').ConnectionString;
    var Message = require('azure-iot-device').Message;
    var async = require('async');
    ```

1. 在 `require` 陳述式之後新增下列變數宣告。 針對您在遠端監視解決方案中所佈建的裝置，使用您記下的值來取代 `{device connection string}` 預留位置值：

    ```nodejs
    var connectionString = '{device connection string}';
    var deviceId = ConnectionString.parse(connectionString).DeviceId;
    ```

1. 如需定義一些基本遙測資料，請新增下列變數︰

    ```nodejs
    var temperature = 50;
    var temperatureUnit = 'F';
    var humidity = 50;
    var humidityUnit = '%';
    var pressure = 55;
    var pressureUnit = 'psig';
    ```

1. 如需定義一些屬性值，請新增下列變數︰

    ```nodejs
    var temperatureSchema = 'chiller-temperature;v1';
    var humiditySchema = 'chiller-humidity;v1';
    var pressureSchema = 'chiller-pressure;v1';
    var interval = "00:00:05";
    var deviceType = "Chiller";
    var deviceFirmware = "1.0.0";
    var deviceFirmwareUpdateStatus = "";
    var deviceLocation = "Building 44";
    var deviceLatitude = 47.638928;
    var deviceLongitude = -122.13476;
    var deviceOnline = true;
    ```

1. 新增下列變數來定義要傳送至解決方案的報告屬性。 這些屬性包括可說明裝置所使用的方法和遙測之中繼資料：

    ```nodejs
    var reportedProperties = {
      "Protocol": "MQTT",
      "SupportedMethods": "Reboot,FirmwareUpdate,EmergencyValveRelease,IncreasePressure",
      "Telemetry": {
        "TemperatureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\"}",
          "MessageSchema": {
            "Name": temperatureSchema,
            "Format": "JSON",
            "Fields": {
              "temperature": "Double",
              "temperature_unit": "Text"
            }
          }
        },
        "HumiditySchema": {
          "Interval": interval,
          "MessageTemplate": "{\"humidity\":${humidity},\"humidity_unit\":\"${humidity_unit}\"}",
          "MessageSchema": {
            "Name": humiditySchema,
            "Format": "JSON",
            "Fields": {
              "humidity": "Double",
              "humidity_unit": "Text"
            }
          }
        },
        "PressureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"pressure\":${pressure},\"pressure_unit\":\"${pressure_unit}\"}",
          "MessageSchema": {
            "Name": pressureSchema,
            "Format": "JSON",
            "Fields": {
              "pressure": "Double",
              "pressure_unit": "Text"
            }
          }
        }
      },
      "Type": deviceType,
      "Firmware": deviceFirmware,
      "FirmwareUpdateStatus": deviceFirmwareUpdateStatus,
      "Location": deviceLocation,
      "Latitude": deviceLatitude,
      "Longitude": deviceLongitude,
      "Online": deviceOnline
    }
    ```

1. 若要列印作業結果，請新增下列協助程式函式︰

    ```nodejs
    function printErrorFor(op) {
        return function printError(err) {
            if (err) console.log(op + ' error: ' + err.toString());
        };
    }
    ```

1. 新增下列協助程式函式，用來將排遙測值隨機化︰

    ```nodejs
    function generateRandomIncrement() {
        return ((Math.random() * 2) - 1);
    }
    ```

1. 新增下列泛型函式以處理解決方案的直接方法呼叫。 此函式會顯示已叫用之直接方法的相關資訊，但在此範例中不會以任何方式修改裝置。 解決方案會使用直接方法在裝置上執行：

    ```nodejs
    function onDirectMethod(request, response) {
      // Implement logic asynchronously here.
      console.log('Simulated ' + request.methodName);

      // Complete the response
      response.send(200, request.methodName + ' was called on the device', function (err) {
        if (err) console.error('Error sending method response :\n' + err.toString());
        else console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');
      });
    }
    ```

1. 新增下列函式以處理解決方案的 **FirmwareUpdate** 直接方法呼叫。 此函式會驗證傳入直接方法酬載中的參數，然後以非同步方式執行韌體更新模擬：

    ```node.js
    function onFirmwareUpdate(request, response) {
      // Get the requested firmware version from the JSON request body
      var firmwareVersion = request.payload.Firmware;
      var firmwareUri = request.payload.FirmwareUri;
      
      // Ensure we got a firmware values
      if (!firmwareVersion || !firmwareUri) {
        response.send(400, 'Missing firmware value', function(err) {
          if (err) console.error('Error sending method response :\n' + err.toString());
          else console.log('400 Response to method \'' + request.methodName + '\' sent successfully.');
        });
      } else {
        // Respond the cloud app for the device method
        response.send(200, 'Firmware update started.', function(err) {
          if (err) console.error('Error sending method response :\n' + err.toString());
          else {
            console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');

            // Run the simulated firmware update flow
            runFirmwareUpdateFlow(firmwareVersion, firmwareUri);
          }
        });
      }
    }
    ```

1. 新增下列函式以模擬長時間執行的韌體更新流程，此流程會將進度回報至解決方案：

    ```node.js
    // Simulated firmwareUpdate flow
    function runFirmwareUpdateFlow(firmwareVersion, firmwareUri) {
      console.log('Simulating firmware update flow...');
      console.log('> Firmware version passed: ' + firmwareVersion);
      console.log('> Firmware URI passed: ' + firmwareUri);
      async.waterfall([
        function (callback) {
          console.log("Image downloading from " + firmwareUri);
          var patch = {
            FirmwareUpdateStatus: 'Downloading image..'
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(10000, callback);
        },
        function (callback) {
          console.log("Downloaded, applying firmware " + firmwareVersion);
          deviceOnline = false;
          var patch = {
            FirmwareUpdateStatus: 'Applying firmware..',
            Online: false
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(8000, callback);
        },
        function (callback) {
          console.log("Rebooting");
          var patch = {
            FirmwareUpdateStatus: 'Rebooting..'
          };
          reportUpdateThroughTwin(patch, callback);
          sleep(10000, callback);
        },
        function (callback) {
          console.log("Firmware updated to " + firmwareVersion);
          deviceOnline = true;
          var patch = {
            FirmwareUpdateStatus: 'Firmware updated',
            Online: true,
            Firmware: firmwareVersion
          };
          reportUpdateThroughTwin(patch, callback);
          callback(null);
        }
      ], function(err) {
        if (err) {
          console.error('Error in simulated firmware update flow: ' + err.message);
        } else {
          console.log("Completed simulated firmware update flow");
        }
      });

      // Helper function to update the twin reported properties.
      function reportUpdateThroughTwin(patch, callback) {
        console.log("Sending...");
        console.log(JSON.stringify(patch, null, 2));
        client.getTwin(function(err, twin) {
          if (!err) {
            twin.properties.reported.update(patch, function(err) {
              if (err) callback(err);
            });      
          } else {
            if (err) callback(err);
          }
        });
      }

      function sleep(milliseconds, callback) {
        console.log("Simulate a delay (milleseconds): " + milliseconds);
        setTimeout(function () {
          callback(null);
        }, milliseconds);
      }
    }
    ```

1. 新增下列程式碼將遙測資料傳送至解決方案。 用戶端應用程式會將屬性新增至訊息，以找出訊息結構描述：

    ```node.js
    function sendTelemetry(data, schema) {
      if (deviceOnline) {
        var d = new Date();
        var payload = JSON.stringify(data);
        var message = new Message(payload);
        message.properties.add('$$CreationTimeUtc', d.toISOString());
        message.properties.add('$$MessageSchema', schema);
        message.properties.add('$$ContentType', 'JSON');

        console.log('Sending device message data:\n' + payload);
        client.sendEvent(message, printErrorFor('send event'));
      } else {
        console.log('Offline, not sending telemetry');
      }
    }
    ```

1. 新增下列程式碼，以建立用戶端執行個體︰

    ```nodejs
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

1. 新增下列程式碼，以便：

    * 開啟連線。
    * 設定所需屬性的處理常式。
    * 傳送報告屬性。
    * 登錄直接方法的處理常式。 此範例會針對韌體更新直接方法，使用個別的處理常式。
    * 開始傳送遙測。

    ```nodejs
    client.open(function (err) {
      if (err) {
        printErrorFor('open')(err);
      } else {
        // Create device Twin
        client.getTwin(function (err, twin) {
          if (err) {
            console.error('Could not get device twin');
          } else {
            console.log('Device twin created');

            twin.on('properties.desired', function (delta) {
              // Handle desired properties set by solution
              console.log('Received new desired properties:');
              console.log(JSON.stringify(delta));
            });

            // Send reported properties
            twin.properties.reported.update(reportedProperties, function (err) {
              if (err) throw err;
              console.log('Twin state reported');
            });

            // Register handlers for all the method names we are interested in.
            // Consider separate handlers for each method.
            client.onDeviceMethod('Reboot', onDirectMethod);
            client.onDeviceMethod('FirmwareUpdate', onFirmwareUpdate);
            client.onDeviceMethod('EmergencyValveRelease', onDirectMethod);
            client.onDeviceMethod('IncreasePressure', onDirectMethod);
          }
        });

        // Start sending telemetry
        var sendTemperatureInterval = setInterval(function () {
          temperature += generateRandomIncrement();
          var data = {
            'temperature': temperature,
            'temperature_unit': temperatureUnit
          };
          sendTelemetry(data, temperatureSchema)
        }, 5000);

        var sendHumidityInterval = setInterval(function () {
          humidity += generateRandomIncrement();
          var data = {
            'humidity': humidity,
            'humidity_unit': humidityUnit
          };
          sendTelemetry(data, humiditySchema)
        }, 5000);

        var sendPressureInterval = setInterval(function () {
          pressure += generateRandomIncrement();
          var data = {
            'pressure': pressure,
            'pressure_unit': pressureUnit
          };
          sendTelemetry(data, pressureSchema)
        }, 5000);

        client.on('error', function (err) {
          printErrorFor('client')(err);
          if (sendTemperatureInterval) clearInterval(sendTemperatureInterval);
          if (sendHumidityInterval) clearInterval(sendHumidityInterval);
          if (sendPressureInterval) clearInterval(sendPressureInterval);
          client.close(printErrorFor('client.close'));
        });
      }
    });
    ```

1. 將變更儲存至 **remote_monitoring.js** 檔案。

1. 若要啟動範例應用程式，請在 Raspberry Pi 上的命令提示字元中執行下列命令：

    ```sh
    node remote_monitoring.js
    ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]
