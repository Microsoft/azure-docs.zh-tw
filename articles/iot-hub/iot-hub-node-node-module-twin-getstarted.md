---
title: 開始使用 Azure IoT 中樞模組身分識別和模組對應項 (Node.js) | Microsoft Docs
description: 了解如何使用適用於 Node.js 的 IoT SDK，建立模組身分識別及更新模組對應項。
author: chrissie926
manager: ''
ms.service: iot-hub
services: iot-hub
ms.devlang: node
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: menchi
ms.openlocfilehash: e1fcca7f37f928c488b1111d6d3299cb03c73cc1
ms.sourcegitcommit: 150a40d8ba2beaf9e22b6feff414f8298a8ef868
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37034614"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-nodejs-back-end-and-nodejs-device"></a>透過 Node.js 後端和 Node.js 裝置開始使用 IoT 中樞模組身分識別和模組對應項

> [!NOTE]
> [模組身分識別與模組對應項](iot-hub-devguide-module-twins.md)類似於 Azure IoT 中樞裝置身分識別與裝置對應項，但提供更精細的細微性。 雖然 Azure IoT 中樞裝置身分識別與裝置對應項可讓後端應用程式設定裝置並提供裝置狀況的可見性，但模組身分識別和模組對應項可提供裝置個別元件的上述功能。 在具備多個元件的可用裝置 (例如以作業系統為基礎的裝置或韌體裝置) 上，它允許每個元件有獨立的組態和狀況。

在本教學課程結尾處，您會有兩個 Node.js 應用程式：

* **CreateIdentities**，這會建立裝置身分識別、模組身分識別和相關聯的安全性金鑰，以連線到您的裝置和模組用戶端。
* **UpdateModuleTwinReportedProperties**，這會將已更新的模組對應項報告的屬性傳送到您的 IoT 中樞。

> [!NOTE]
> 如需可用來建置兩個應用程式，以在裝置與您的解決方案後端執行之 Azure IoT SDK 的相關資訊，請參閱 [Azure IoT SDK][lnk-hub-sdks]。

若要完成此教學課程，您需要下列項目：

* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)
* IoT 中樞。
* 安裝最新的 [Node.js SDK](https://github.com/Azure/azure-iot-sdk-node)。


您現在已經建立 IoT 中樞，因此您已具有完成本教學課程的其餘部分所需的主機名稱和 IoT 中樞連接字串。

<a id="DeviceIdentity_csharp"></a>
## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>在 IoT 中樞中建立裝置身分識別與模組身分識別

在本節中，您會建立 Node.js 應用程式，它會在 IoT 中樞的身分識別登錄中建立裝置身分識別和模組身分識別。 裝置或模組無法連線到 IoT 中樞，除非它在身分識別登錄中具有項目。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南][lnk-devguide-identity]的＜身分識別登錄＞一節。 當您執行此主控台應用程式時，它會針對裝置和模組產生唯一的識別碼和金鑰。 當裝置和模組將裝置到雲端的訊息傳送給 IoT 中樞時，裝置和模組會使用這些值來識別自己。 識別碼會區分大小寫。

1.  建立目錄來保存您的程式碼。
2. 在該目錄的內部，先執行 **npm init -y** 以使用預設值建立空白的 package.json。 這是程式碼的專案檔。
3. 執行 **npm install -S azure-iothub@modules-preview** 將服務 SDK 安裝在 **node_modules** 子目錄內。 

    > [!NOTE] 
    > 子目錄名稱 node_modules 使用字組模組來表示「節點程式庫」。 此詞彙在此處與 IoT 中樞模組無關。

4. 在您的目錄中建立以下 .js 檔案。 將它命名為 **add.js**。 複製並貼上中樞連接字串與中樞名稱。

    ```javascript
    var Registry = require('azure-iothub').Registry;
    var uuid = require('uuid');
    // Copy/paste your connection string and hub name here
    var serviceConnectionString = '<hub connection string from portal>';
    var hubName = '<hub name>.azure-devices.net';
    // Create an instance of the IoTHub registry
    var registry = Registry.fromConnectionString(serviceConnectionString);
    // Insert your device ID and moduleId here.
    var deviceId = 'myFirstDevice';
    var moduleId = 'myFirstModule';
    // Create your device as a SAS authentication device
    var primaryKey = new Buffer(uuid.v4()).toString('base64');
    var secondaryKey = new Buffer(uuid.v4()).toString('base64');
    var deviceDescription = {
      deviceId: deviceId,
      status: 'enabled',
      authentication: {
        type: 'sas',
        symmetricKey: {
          primaryKey: primaryKey,
          secondaryKey: secondaryKey
        }
      }
    };

    // First, create a device identity
    registry.create(deviceDescription, function(err) {
      if (err) {
        console.log('Error creating device identity: ' + err);
        process.exit(1);
      }
      console.log('device connection string = "HostName=' + hubName + ';DeviceId=' + deviceId + ';SharedAccessKey=' + primaryKey + '"');

    // Then add a module to that device
      registry.addModule({ deviceId: deviceId, moduleId: moduleId }, function(err) {
        if (err) {
          console.log('Error creating module identity: ' + err);
          process.exit(1);
        }

    // Finally, retrieve the module details from the hub so we can construct the connection string
        registry.getModule(deviceId, moduleId, function(err, foundModule) {
          if (err) {
            console.log('Error getting module back from hub: ' + err);
            process.exit(1);
          }
          console.log('module connection string = "HostName=' + hubName + ';DeviceId=' + foundModule.deviceId + ';ModuleId='+foundModule.moduleId+';SharedAccessKey=' + foundModule.authentication.symmetricKey.primaryKey + '"');
          process.exit(0);
        });
      });
    });

    ```

此應用程式會在 **myFirstDevice** 裝置下方建立識別碼為 **myFirstDevice** 的裝置身分識別，以及識別碼為 **myFirstModule** 的模組身分識別。 (如果該模組識別碼已經存在身分識別登錄中，程式碼就只會擷取現有的模組資訊)。接著，應用程式會顯示該身分識別的主要金鑰。 您會在模擬模組應用程式中使用此金鑰來連線到您的 IoT 中樞。

5. 使用 add.js 節點執行。 它會為裝置身分識別提供一個連接字串，並為模組身分識別提供另一個連接字串。

    > [!NOTE]
    > IoT 中樞身分識別登錄只會儲存裝置和模組身分識別，以啟用對 IoT 中樞的安全存取。 身分識別登錄會儲存裝置識別碼和金鑰，以作為安全性認證使用。 身分識別登錄也會儲存每個裝置的已啟用/已停用旗標，以便您用來停用該裝置的存取權。 如果您的應用程式需要儲存其他裝置特定的中繼資料，它應該使用應用程式專用的存放區。 模組身分識別沒有啟用/停用旗標。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南][lnk-devguide-identity]。


<a id="D2C_csharp"></a>
## <a name="update-the-module-twin-using-nodejs-device-sdk"></a>使用 Node.js 裝置 SDK 更新模組對應項

在這一節中，您會在模擬裝置上建立 Node.js 應用程式，以便更新模組對應項報告的屬性。

1. **取得您的模組連接字串** - 如果您登入 [Azure 入口網站][lnk-portal]，請立即取得。 瀏覽至您的 IoT 中樞並按一下 IoT 裝置。 尋找 myFirstDevice 並加以開啟，您會看到已成功建立 myFirstModule。 複製模組連接字串。 在下一個步驟中需要用到它。

    ![Azure 入口網站模組詳細資料][15]

2. 類似前述的步驟，為您的裝置程式碼建立目錄，然後使用 NPM 加以初始化，再安裝裝置 SDK (**npm install -S azure-iot-device-amqp@modules-preview**)。

    > [!NOTE]
    > 您可能會感覺 npm 安裝命令執行速度慢。 請耐心等候，它正從封裝存放庫中提取大量程式碼。

    > [!NOTE] 
    > 如果您看見錯誤訊息表示 npm ERR! registry error parsing json，可放心忽略此訊息。 如果您看見錯誤訊息表示 npm ERR! registry error parsing json，可放心忽略此訊息。

3. 建立名為 twin.js 的檔案。 將模組身分識別字串複製貼上。

    ```javascript
    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-amqp').Amqp;
    // Copy/paste your module connection string here.
    var connectionString = '<insert module connection string here>';
    // Create a client using the Amqp protocol.
    var client = Client.fromConnectionString(connectionString, Protocol);
    client.on('error', function (err) {
      console.error(err.message);
    });
    // connect to the hub
    client.open(function(err) {
      if (err) {
        console.error('error connecting to hub: ' + err);
        process.exit(1);
      }
      console.log('client opened');
    // Create device Twin
      client.getTwin(function(err, twin) {
        if (err) {
          console.error('error getting twin: ' + err);
          process.exit(1);
        }
    // Output the current properties
        console.log('twin contents:');
        console.log(twin.properties);
    // Add a handler for desired property changes
        twin.on('properties.desired', function(delta) {
            console.log('new desired properties received:');
            console.log(JSON.stringify(delta));
        });
    // create a patch to send to the hub
        var patch = {
          updateTime: new Date().toString(),
          firmwareVersion:'1.2.1',
          weather:{
            temperature: 72,
            humidity: 17
          }
        };
    // send the patch
        twin.properties.reported.update(patch, function(err) {
          if (err) throw err;
          console.log('twin state reported');
        });
      });
    });
    ```

2. 使用 **node twin.js** 命令執行此動作。

    ```
    F:\temp\module_twin>node twin.js
    client opened
    twin contents:
    { reported: { update: [Function: update], '$version': 1 },
      desired: { '$version': 1 } }
    new desired properties received:
    {"$version":1}
    twin state reported
    ```

## <a name="next-steps"></a>後續步驟

若要繼續開始使用 IoT 中樞並瀏覽其他 IoT 案例，請參閱︰

* [開始使用裝置管理][lnk-device-management]
* [開始使用 IoT Edge][lnk-iot-edge]


<!-- Images. -->
[15]: ./media\iot-hub-csharp-csharp-module-twin-getstarted/module-detail.JPG
<!-- Links -->
[lnk-hub-sdks]: iot-hub-devguide-sdks.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal]: https://portal.azure.com/

[lnk-device-management]: iot-hub-node-node-device-management-get-started.md
[lnk-iot-edge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-devguide-identity]: iot-hub-devguide-identity-registry.md
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/