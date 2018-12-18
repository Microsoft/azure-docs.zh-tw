---
title: 將裝置中的檔案上傳至使用節點的 Azure IoT 中樞 | Microsoft Docs
description: 如何使用適用於 Node.js 的 Azure IoT 裝置 SDK 將檔案從裝置上傳至雲端。 上傳的檔案會儲存在 Azure 儲存體 blob 容器中。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 06/28/2017
ms.author: dobett
ms.openlocfilehash: 936063e1419d5e2261033ea74d75687eade928e8
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39187312"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub"></a>使用 IoT 中樞將檔案從裝置上傳至雲端

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

本教學課程是以[使用 IoT 中樞傳送雲端到裝置訊息](iot-hub-node-node-c2d.md)教學課程中的程式碼來建置，示範如何使用 [IoT 中樞的檔案上傳功能](iot-hub-devguide-file-upload.md)將檔案上傳至 [Azure Blob 儲存體](../storage/index.yml)。 本教學課程說明如何：

- 安全地將 Azure Blob URI 提供給裝置，以便上傳檔案。
- 您可以使用 IoT 中樞檔案上傳通知來觸發在您的應用程式後端中處理此檔案。

[開始使用 IoT 中樞](quickstart-send-telemetry-node.md)教學課程示範「IoT 中樞」的基本裝置到雲端傳訊功能。 不過，在某些情況下，您無法輕易地將裝置傳送的資料對應到 IoT 中樞接受且相對較小的裝置到雲端訊息。 例如︰

* 包含映像的大型檔案
* 影片
* 取樣高頻率的震動資料
* 某種經前置處理過的資料。

這些檔案通常會使用工具 (例如 [Azure Data Factory](../data-factory/introduction.md) 或 [Hadoop](../hdinsight/index.yml) 堆疊) 在雲端中進行批次處理。 當您需要從裝置上傳檔案時，您仍然可以使用安全可靠的 IoT 中樞。

在本教學課程結尾處，您會執行兩個 Node.js 主控台應用程式：

* **SimulatedDevice.js**，它會使用 IoT 中樞提供的 SAS URI，將檔案上傳到儲存體。
* **ReadFileUploadNotification.js**，它會接收來自 IoT 中樞的檔案上傳通知。

> [!NOTE]
> IoT 中樞透過 Azure IoT 裝置 SDK 來支援許多裝置平台和語言 (包括 C、.NET、Javascript、Python 和 Java)。 如需如何將您的裝置連接到 Azure IoT 中樞的逐步指示，請參閱 [Azure IoT 開發人員中心]。

若要完成此教學課程，您需要下列項目：

* Node.js 4.0.x 版或更新版本。
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>從裝置應用程式上傳檔案

在本節中，您可以建立裝置應用程式來將檔案上傳到 IoT 中樞。

1. 建立稱為 ```simulateddevice``` 的空資料夾。  在 ```simulateddevice``` 資料夾中，於命令提示字元使用下列命令建立 package.json 檔案。  接受所有預設值：

    ```cmd/sh
    npm init
    ```

1. 在 ```simulateddevice``` 資料夾中，於命令提示字元執行下列命令來安裝 **azure-iot-device** 裝置 SDK 套件和 **azure-iot-device-mqtt** 套件：

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

1. 使用文字編輯器，在 ```simulateddevice``` 資料夾中建立 **SimulatedDevice.js** 檔案。

1. 在 **SimulatedDevice.js** 檔案開頭新增下列 ```require``` 陳述式：

    ```nodejs
    'use strict';
    
    var fs = require('fs');
    var mqtt = require('azure-iot-device-mqtt').Mqtt;
    var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
    ```

1. 新增 ```deviceconnectionstring``` 變數，並用它來建立**用戶端**執行個體。  以您在＜建立 IoT 中樞＞ 一節中建立的裝置名稱取代 ```{deviceconnectionstring}```：

    ```nodejs
    var connectionString = '{deviceconnectionstring}';
    var filename = 'myimage.png';
    ```

    > [!NOTE]
    > 為了方便起見，連接字串已包含在程式碼中：這不是建議作法，而且根據您的使用案例和架構，您可能要考慮用更安全的方式來儲存此密碼。

1. 新增下列程式碼以連線至用戶端：

    ```nodejs
    var client = clientFromConnectionString(connectionString);
    console.log('Client connected');
    ```

1. 建立回呼並使用 **uploadToBlob** 函式來上傳檔案。

    ```nodejs
    fs.stat(filename, function (err, stats) {
        const rr = fs.createReadStream(filename);
    
        client.uploadToBlob(filename, rr, stats.size, function (err) {
            if (err) {
                console.error('Error uploading file: ' + err.toString());
            } else {
                console.log('File uploaded');
            }
        });
    });
    ```

1. 儲存並關閉 **SimulatedDevice.js** 檔案。

1. 將映像檔複製到 `simulateddevice` 資料夾，並重新命名為 `myimage.png`。

## <a name="receive-a-file-upload-notification"></a>接收檔案上傳通知

在本節中，您要建立一個 Node.js 主控台應用程式，接收來自 IoT 中樞的檔案上傳通知訊息。

您可以從 IoT 中樞使用 **iothubowner** 連接字串來完成此區段。 您會在 [共用存取原則] 刀鋒視窗的 [Azure 入口網站](https://portal.azure.com/)中找到連接字串。

1. 建立稱為 ```fileuploadnotification``` 的空資料夾。  在 ```fileuploadnotification``` 資料夾中，於命令提示字元使用下列命令建立 package.json 檔案。  接受所有預設值：

    ```cmd/sh
    npm init
    ```

1. 在 ```fileuploadnotification``` 資料夾中，於命令提示字元執行下列命令以安裝 **azure-iothub** SDK 套件：

    ```cmd/sh
    npm install azure-iothub --save
    ```

1. 使用文字編輯器在 ```fileuploadnotification``` 資料夾中建立 **FileUploadNotification.js** 檔案。

1. 在 **FileUploadNotification.js** 檔案開頭新增下列 ```require``` 陳述式：

    ```nodejs
    'use strict';
    
    var Client = require('azure-iothub').Client;
    ```

1. 新增 ```iothubconnectionstring``` 變數，並用它來建立**用戶端**執行個體。  將 ```{iothubconnectionstring}``` 替換為您在＜建立 IoT 中樞＞一節中為 IoT 中樞所建立的連接字串：

    ```nodejs
    var connectionString = '{iothubconnectionstring}';
    ```

    > [!NOTE]
    > 為了方便起見，連接字串已包含在程式碼中：這不是建議作法，而且根據您的使用案例和架構，您可能要考慮用更安全的方式來儲存此密碼。

1. 新增下列程式碼以連線至用戶端：

    ```nodejs
    var serviceClient = Client.fromConnectionString(connectionString);
    ```

1. 開啟用戶端，並使用 **getFileNotificationReceiver** 函式接收狀態更新。

    ```nodejs
    serviceClient.open(function (err) {
      if (err) {
        console.error('Could not connect: ' + err.message);
      } else {
        console.log('Service client connected');
        serviceClient.getFileNotificationReceiver(function receiveFileUploadNotification(err, receiver){
          if (err) {
            console.error('error getting the file notification receiver: ' + err.toString());
          } else {
            receiver.on('message', function (msg) {
              console.log('File upload from device:')
              console.log(msg.getData().toString('utf-8'));
            });
          }
        });
      }
    });
    ```

1. 儲存並關閉 **FileUploadNotification.js** 檔案。

## <a name="run-the-applications"></a>執行應用程式

現在您已經準備好執行應用程式。

在 `fileuploadnotification` 資料夾的命令提示字元中，執行下列命令：

```cmd/sh
node FileUploadNotification.js
```

在 `simulateddevice` 資料夾的命令提示字元中，執行下列命令：

```cmd/sh
node SimulatedDevice.js
```

下列螢幕擷取畫面顯示 **SimulatedDevice** 應用程式的輸出：

![simulated-device 應用程式的輸出](./media/iot-hub-node-node-file-upload/simulated-device.png)

下列螢幕擷取畫面顯示 **FileUploadNotification** 應用程式的輸出：

![read-file-upload-notification 應用程式的輸出](./media/iot-hub-node-node-file-upload/read-file-upload-notification.png)

您可以使用入口網站檢視您所設定之儲存體容器中的上傳檔案：

![上傳的檔案](./media/iot-hub-node-node-file-upload/uploaded-file.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已學到如何使用 IoT 中樞的檔案上傳功能來簡化從裝置上傳檔案。 您可以利用下列文章繼續探索 IoT 中樞功能和案例：

* [以程式設計方式建立 IoT 中樞][lnk-create-hub]
* [C SDK 簡介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

<!-- Links -->
[Azure IoT 開發人員中心]: http://azure.microsoft.com/develop/iot

[lnk-create-hub]: iot-hub-rm-template-powershell.md
[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md
