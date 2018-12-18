---
title: 將實際裝置新增至 Azure IoT Central 應用程式 | Microsoft Docs
description: 以操作員的身分，將實際裝置新增至 Azure IoT Central 應用程式。
author: sandeeppujar
ms.author: sandeepu
ms.date: 04/16/2018
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: peterpr
ms.openlocfilehash: 984457968de5ef5e43b15201dac213cd96b4b0e2
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49309751"
---
# <a name="tutorial-add-a-real-device-to-your-azure-iot-central-application"></a>教學課程：將實際裝置新增至 Azure IoT 中心應用程式

本教學課程將說明如何在 Microsoft Azure IoT Central 應用程式中新增和設定實際裝置。

本教學課程由兩個部分所組成：

1. 首先，身為操作員，您將了解如何在 Azure IoT Central 應用程式中新增和設定實際裝置。 在此部分結束時，您會擷取要在第二個單元中使用的連接字串。
2. 其次，身為裝置開發人員，您將了解實際裝置中的程式碼。 您會將第一個單元中的連接字串新增至範例程式碼。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 新增實際裝置
> * 設定實際裝置
> * 從應用程式取得適用於實際裝置的連接字串
> * 了解用戶端程式碼如何對應至應用程式
> * 設定實際裝置的用戶端程式碼

## <a name="prerequisites"></a>必要條件

在開始之前，建置者至少應完成第一個建置者教學課程，以建立 Azure IoT Central 應用程式：

* [定義新的裝置類型](tutorial-define-device-type.md) (必要)
* [為您的裝置設定規則和動作](tutorial-configure-rules.md) (選用)
* [自訂操作員的檢視](tutorial-customize-operator.md) (選用)

## <a name="add-a-real-device"></a>新增實際裝置

若要將實際裝置新增至應用程式，您應使用在[定義新的裝置類型](tutorial-define-device-type.md)教學課程中建立的**連線的空調**裝置範本。

1. 若要以操作員的身分新增裝置，請選擇左側導覽功能表中的 [裝置總管]：

   ![顯示連線空調的裝置總管頁面](media/tutorial-add-device/explorer.png)

   [裝置總管] 會顯示**連線的空調**裝置範本，以及建置者在建立裝置範本時自動建立的模擬裝置。

2. 若要開始連接實際的連線空調裝置，請選擇 [新增]，然後選擇 [實際]：

   ![開始新增，實際的連線空調裝置：](media/tutorial-add-device/newreal.png)

3. 輸入裝置識別碼 (**應該是小寫**)，或使用建議的裝置識別碼。您也可以輸入新裝置的名稱，然後選擇 [建立]。  

   ![將裝置重新命名](media/tutorial-add-device/rename.png)



## <a name="configure-a-real-device"></a>設定實際裝置

實際裝置可從**連線的空調**裝置範本建立： 您可以使用 [設定] 來設定裝置並設定屬性值，以記錄裝置的相關資訊。

1. 在 [設定] 頁面上應留意到，[設定溫度] 設定狀態為 [沒有更新]。 在實際裝置連線到應用程式並確認它已針對設定執行因應動作之前，都會處於此狀態。 

    ![設定顯示同步中](media/tutorial-add-device/settingssyncing.png)

2. 在您新的實際連線空調裝置的 [屬性] 頁面上，服務的位置和上次維修日期都是裝置的可編輯屬性。 在裝置連線到應用程式之前，序號和韌體版本欄位將是空的。 這些值是從裝置傳送的唯讀值，無法編輯。

    ![實際裝置的裝置屬性](media/tutorial-add-device/setproperties1.png)

3. 您可以檢視實際裝置的 [測量]、[規則] 和 [儀表板] 頁面。

## <a name="generate-connection-string-for-real-device-from-application"></a>從應用程式產生實際裝置的連接字串

裝置開發人員必須在執行於裝置上的程式碼中內嵌實際裝置的*連接字串*。 連接字串可讓裝置安全地連線至您的 Azure IoT Central 應用程式。 在後續步驟中準備以 Node.js 撰寫的用戶端程式碼時，您可以產生連接字串。 Node.js 應用程式代表實際連線的空調。 

## <a name="prepare-the-client-code"></a>準備用戶端程式碼

本文中的範例程式碼是以 [Node.js](https://nodejs.org/) 撰寫的，並顯示剛好足以執行下列作業的程式碼：

* 以裝置連線至 Azure IoT Central 應用程式。
* 以連線的空調裝置傳送溫度遙測資料。
* 將裝置屬性傳送至 Azure IoT Central 應用程式。
* 回應使用 [設定溫度] 設定的操作員。
* 處理來自 Azure IoT Central 應用程式的回應命令。


[後續步驟](#next-steps)一節中參考的「作法」文章將提供更多完整範例，並說明如何使用其他程式設計語言。 如需關於裝置如何連線至 Azure IoT Central 的詳細資訊，請參閱[裝置連線能力](concepts-connectivity.md)一文。

下列步驟說明如何準備 [Node.js](https://nodejs.org/) 範例：

1. 在電腦上安裝 [Node.js](https://nodejs.org/) 4.0.x 版或更新版本。 Node.js 適用於多種作業系統。

1. 在電腦上建立名為 `connectedairconditioner` 的資料夾。

1. 在命令列環境中，瀏覽至您所建立的 `connectedairconditioner` 資料夾。

1. 使用下列命令安裝 DPS 金鑰產生器：
    
    ```cmd/sh
    npm i -g dps-keygen
    ```

   深入了解[命令列工具](https://www.npmjs.com/package/dps-keygen)。

1. 從 [GitHub](https://github.com/Azure/dps-keygen/tree/master/bin) 下載並解壓縮 (Windows) dps_cstr 工具

    確定您的選擇與平台相符。 例如，在 Windows 上，資料夾中此時應該會有 dps_cstr.exe 和 dll 檔案可供使用。 

1. 應用程式中的裝置執行個體所適用的連接字串會從 IoT Central 所提供的裝置資訊產生。

   返回 IoT Central 入口網站。 在實際「連線的空調」的裝置畫面上，選擇 [連線]。

   ![顯示檢視連線資訊連結的裝置頁面](media/tutorial-add-device/connectionlink.png)


1. 在 [裝置連線] 頁面上，將 [範圍識別碼]、[裝置識別碼] 和 [主要金鑰] 複製並貼到文字編輯器上，然後儲存。 您在下一個步驟將用到這些值。

   ![連線詳細資料](media/tutorial-add-device/device-connect.PNG)

1. 返回命令列環境，並執行下列命令以產生連接字串：

   ```cmd/sh
   dps_cstr <scope_id> <device_id> <Primary Key>
   ```
   
   複製輸出，並將其儲存在新的檔案中 (例如 connection.txt)。

1. 若要初始化 Node.js 專案，請以所有預設值執行下列命令：

   ```cmd/sh
    npm init
   ```

1. 若要安裝必要的套件，請執行下列命令︰

   ```cmd/sh
   npm install azure-iot-device azure-iot-device-mqtt --save
   ```

1. 使用文字編輯器，在 `connectedairconditioner` 資料夾中建立名為 **ConnectedAirConditioner.js** 的檔案。

1. 在 **ConnectedAirConditioner.js** 檔案的開頭處新增下列 `require` 陳述式：

   ```javascript
   'use strict';

   var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
   var Message = require('azure-iot-device').Message;
   var ConnectionString = require('azure-iot-device').ConnectionString;
   ```

1. 將下列變數宣告新增至該檔案：

   ```javascript
   var connectionString = '{your device connection string}';
   var targetTemperature = 0;
   var client = clientFromConnectionString(connectionString);
   ```
   > [!NOTE]
   > 您會在稍後的步驟中更新預留位置 `{your device connection string}`。 

1. 儲存您到目前為止所做的變更，但將檔案保持為開啟。

## <a name="understand-how-client-code-maps-to-the-application"></a>了解用戶端程式碼如何對應至應用程式

在上一節中，您為連線至 Azure IoT Central 應用程式的應用程式建立了基本架構 Node.js 專案。 在本節中，您會新增程式碼，以：

* 連線至 Azure IoT Central 應用程式。
* 將遙測資料傳送至 Azure IoT Central 應用程式。
* 將裝置屬性傳送至 Azure IoT Central 應用程式。
* 從 Azure IoT Central 應用程式接收設定。
* 處理來自 Azure IoT Central 應用程式的回應命令。


1. 若要將溫度遙測資料傳送至 Azure IoT Central 應用程式，請將下列程式碼新增至 **ConnectedAirConditioner.js** 檔案：

   ```javascript
   // Send device telemetry.
   function sendTelemetry() {
     var temperature = targetTemperature + (Math.random() * 15);
     var data = JSON.stringify({ temperature: temperature });
     var message = new Message(data);
     client.sendEvent(message, (err, res) => console.log(`Sent message: ${message.getData()}` +
       (err ? `; error: ${err.toString()}` : '') +
       (res ? `; status: ${res.constructor.name}` : '')));
   }
   ```

   您在 JSON 中傳送的欄位名稱，必須符合您為裝置範本中的溫度遙測資料指定的欄位名稱。 在此範例中，欄位的名稱為 **temperature**。


1. 若要傳送裝置屬性 (例如 **firmwareVersion** 和 **serialNumber**)，請新增下列定義：

   ```javascript
   // Send device properties
   function sendDeviceProperties(twin) {
     var properties = {
       firmwareVersion: "9.75",
       serialNumber: "10001"
     };
     twin.properties.reported.update(properties, (errorMessage) => 
       console.log(` * Sent device properties ` + (errorMessage ? `Error: ${errorMessage.toString()}` : `(success)`)));
   }
   ```

1. 若要定義您的裝置所支援的設定 (例如 **setTemperature**)，請新增下列定義：

   ```javascript
   // Add any settings your device supports
   // mapped to a function that is called when the setting is changed.
   var settings = {
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

1. 若要處理從 Azure IoT Central 傳送的設定，請新增下列函式，以找出並執行適當的裝置程式碼：

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

    此函式會：

    * 監看傳送所需屬性的 Azure IoT Central。
    * 找出要呼叫以處理設定變更的適當函式。
    * 將通知傳回至 Azure IoT Central 應用程式。

1. 若要從 Azure IoT Central 應用程式回應命令 (例如 **echo**)，請新增下列檔案：

   ```javascript
   // Respond to the echo command
   function onCommandEcho(request, response) {
     // Display console info
     console.log(' * Echo command received');
     // Respond
     response.send(10, 'Success', function (errorMessage) {});
   }
   ```

1. 新增下列程式碼以完成對 Azure IoT Central 的連線，並連結用戶端程式碼中的函式：

   ```javascript
   // Handle device connection to Azure IoT Central.
   var connectCallback = (err) => {
     if (err) {
       console.log(`Device could not connect to Azure IoT Central: ${err.toString()}`);
     } else {
       console.log('Device successfully connected to Azure IoT Central');
       // Send telemetry measurements to Azure IoT Central every 1 second.
       setInterval(sendTelemetry, 1000);
       // Setup device command callbacks
       client.onDeviceMethod('echo', onCommandEcho);
       // Get device twin from Azure IoT Central.
       client.getTwin((err, twin) => {
         if (err) {
           console.log(`Error getting device twin: ${err.toString()}`);
         } else {
           // Send device properties once on device start up
           sendDeviceProperties(twin);
           // Apply device settings and handle changes to device settings.
           handleSettings(twin);
         }
       });
     }
   };

   client.open(connectCallback);
   ```

1. 儲存您到目前為止所做的變更，但將檔案保持為開啟。

## <a name="configure-client-code-for-the-real-device"></a>設定實際裝置的用戶端程式碼

<!-- Add the connection string to the sample code, build, and run --> 若要設定用戶端程式碼以連線至 Azure IoT Central 應用程式，您必須新增稍早在此教學課程中記下的實際裝置連接字串。

1. 在 **ConnectedAirConditioner.js** 檔案中，尋找以下這一行程式碼：

   ```javascript
   var connectionString = '{your device connection string}';
   ```

1. 將 `{your device connection string}` 取代為實際裝置的連接字串。 您先前已將連接字串儲存在文字編輯器中。

1. 將變更儲存到 **ConnectedAirConditioner.js** 檔案。

1. 若要執行範例，請在命令列環境中輸入下列命令：

   ```cmd/sh
   node ConnectedAirConditioner.js
   ```

   > [!NOTE]
   > 請確定您在執行此命令時位於 `connectedairconditioner` 資料夾中。

1. 應用程式會將輸出列印至主控台：

   ![用戶端應用程式輸出](media/tutorial-add-device/output.png)

1. 約 30 秒後，您就會在裝置的 [測量] 頁面上看到遙測資料：

   ![實際的遙測資料](media/tutorial-add-device/realtelemetry.png)

1. 在 [設定] 頁面上，您可以看到設定此時處於已同步狀態。 裝置第一次連線時，將會接收設定值並確認變更：

   ![設定已同步](media/tutorial-add-device/settingsynced.png)

1. 在 [設定] 頁面上，將裝置溫度設定為 **95**，並選擇 [更新裝置]。 範例應用程式會接收並處理這項變更：

   ![接收並處理設定](media/tutorial-add-device/receivesetting.png)

   > [!NOTE]
   > 「設定更新」訊息有兩種。 一種會在傳送 `pending` 狀態時顯示，另一種則在傳送 `completed` 狀態時顯示。

1. 在 [測量] 頁面上，您可以看到裝置正在傳送較高的溫度值：

    ![目前的溫度遙測資料較高](media/tutorial-add-device/highertemperature.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="nextstepaction"]
> * 新增實際裝置
> * 設定新裝置
> * 從應用程式取得適用於實際裝置的連接字串
> * 了解用戶端程式碼如何對應至應用程式
> * 設定實際裝置的用戶端程式碼

現在，您已將實際裝置連線至 Azure IoT Central 應用程式，以下是建議的後續步驟：

身為操作員，您可以了解如何：

* [管理您的裝置](howto-manage-devices.md)
* [使用裝置集合](howto-use-device-sets.md)
* [建立自訂分析](howto-create-analytics.md)

身為裝置開發人員，您可以了解如何：

* [準備及連線 DevKit](howto-connect-devkit.md)
* [準備及連線 Raspberry Pi](howto-connect-raspberry-pi-python.md)
* [將一般 Node.js 用戶端連線至 Azure IoT Central 應用程式](howto-connect-nodejs.md)
* [自訂您的程式碼][lnk-nodejs-device-ref]


[lnk-nodejs-device-ref]: /javascript/api/azure-iot-device/?view=azure-iot-typescript-latest