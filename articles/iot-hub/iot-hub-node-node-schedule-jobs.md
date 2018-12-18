---
title: 使用 Azure IoT 中樞 (Node) 排定作業 | Microsoft Docs
description: 如何排定 Azure IoT 中樞作業在多個裝置上叫用直接方法。 您可以使用適用於 Node.js 的 Azure IoT SDK，實作模擬裝置應用程式和服務應用程式來執行作業。
author: juanjperez
manager: cberlin
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 10/06/2017
ms.author: juanpere
ms.openlocfilehash: 69469d6b302f951301c92c0ee7984df95911624e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221293"
---
# <a name="schedule-and-broadcast-jobs-node"></a>排定及廣播作業 (Node)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Azure IoT 中樞是一項完全受控的服務，可讓後端應用程式建立作業來排定和更新數百萬個裝置，並追蹤作業。  作業可用於下列動作：

* 更新所需屬性
* 更新標籤
* 叫用直接方法

就概念而言，作業會包裝上述其中一個動作，然後針對由裝置對應項 (twin) 查詢所定義的一組裝置，追蹤執行進度。  例如，後端應用程式可以在 10,000 個裝置上使用作業叫用重新啟動方法，此方法由裝置對應項查詢指定並排定在未來執行。  接著，該應用程式可以追蹤每個這些裝置接收和執行重新啟動方法的進度。

從下列文章深入了解這當中的每一項功能：

* 裝置對應項和屬性：[開始使用裝置對應項][lnk-get-started-twin]和[教學課程：如何使用裝置對應項屬性][lnk-twin-props]
* 直接方法：[IoT 中樞開發人員指南 - 直接方法][lnk-dev-methods]和[教學課程：直接方法][lnk-c2d-methods]

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

本教學課程說明如何：

* 建立具有直接方法的 Node.js 模擬裝置應用程式，此直接方法會啟用 **lockDoor** 供解決方案後端呼叫。
* 建立 Node.js 主控台應用程式，此應用程式會使用作業在模擬裝置應用程式中呼叫 **lockDoor** 直接方法，並使用裝置作業來更新所需屬性。

在本教學課程結尾處，您會有兩個 Node.js 應用程式：

**simDevice.js** 會使用裝置身分識別連接到您的 IoT 中樞，並接收 **lockDoor** 直接方法。

**scheduleJobService.js** 會使用作業在模擬裝置應用程式中呼叫直接方法，並更新裝置對應項 (twin) 的所需屬性。

若要完成此教學課程，您需要下列項目：

* Node.js 4.0.x 版或更新版本， <br/>  [準備您的開發環境][lnk-dev-setup]說明如何在 Windows 或 Linux 上安裝本教學課程的 Node.js。
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>建立模擬裝置應用程式
在本節中，您建立 Node.js 主控台應用程式，回應雲端所呼叫的直接方法，可觸發模擬的 **lockDoor** 方法。

1. 建立名為 **simDevice** 的新空白資料夾。  在命令提示字元中，使用下列命令在 **simDevice** 資料夾中建立 package.json 檔案。  接受所有預設值：
   
    ```
    npm init
    ```
2. 在命令提示字元中，於 **simDevice** 資料夾中執行下列命令來安裝 **azure-iot-device** 裝置 SDK 套件和 **azure-iot-device-mqtt** 套件：
   
    ```
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```
3. 使用文字編輯器，在 [simDevice] 資料夾中建立新的 **simDevice.js** 檔案。
4. 在 **simDevice.js** 檔案的開頭新增下列 'require' 陳述式：
   
    ```
    'use strict';
   
    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```
5. 新增 **connectionString** 變數，並用它來建立**用戶端**執行個體。  
   
    ```
    var connectionString = 'HostName={youriothostname};DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```
6. 新增下列函式以處理 **lockDoor** 方法。
   
    ```
    var onLockDoor = function(request, response) {
   
        // Respond the cloud app for the direct method
        response.send(200, function(err) {
            if (!err) {
                console.error('An error occured when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });
   
        console.log('Locking Door!');
    };
    ```
7. 新增下列程式碼以註冊 **lockDoor** 方法的處理常式。
   
    ```
    client.open(function(err) {
        if (err) {
            console.error('Could not connect to IotHub client.');
        }  else {
            console.log('Client connected to IoT Hub. Register handler for lockDoor direct method.');
            client.onDeviceMethod('lockDoor', onLockDoor);
        }
    });
    ```
8. 儲存並關閉 **simDevice.js** 檔案。

> [!NOTE]
> 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如[暫時性錯誤處理](/azure/architecture/best-practices/transient-faults)一文中所建議，實作重試原則 (例如指數型輪詢)。
> 
> 

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>排定用於呼叫直接方法及更新裝置對應項 (twin) 屬性的作業
在本節中，您會建立一個 Node.js 主控台 App，此 App 會使用直接方法在裝置上初始化遠端 **lockDoor**，並更新裝置對應項 (twin) 的屬性。

1. 建立名為 **scheduleJobService** 的新空白資料夾。  在命令提示字元中，使用下列命令在 **scheduleJobService** 資料夾中建立 package.json 檔案。  接受所有預設值：
   
    ```
    npm init
    ```
2. 在命令提示字元中，於 **scheduleJobService** 資料夾中執行下列命令來安裝 **azure-iothub** 裝置 SDK 套件和 **azure-iot-device-mqtt** 套件：
   
    ```
    npm install azure-iothub uuid --save
    ```
3. 使用文字編輯器，在 [scheduleJobService] 資料夾中建立新的 **scheduleJobService.js** 檔案。
4. 在 **dmpatterns_gscheduleJobServiceetstarted_service.js** 檔案的開頭新增下列 'require' 陳述式：
   
    ```
    'use strict';
   
    var uuid = require('uuid');
    var JobClient = require('azure-iothub').JobClient;
    ```
5. 新增下列變數宣告，並取代預留位置值︰
   
    ```
    var connectionString = '{iothubconnectionstring}';
    var queryCondition = "deviceId IN ['myDeviceId']";
    var startTime = new Date();
    var maxExecutionTimeInSeconds =  300;
    var jobClient = JobClient.fromConnectionString(connectionString);
    ```
6. 新增下列將用來監視作業執行的函式：
   
    ```
    function monitorJob (jobId, callback) {
        var jobMonitorInterval = setInterval(function() {
            jobClient.getJob(jobId, function(err, result) {
            if (err) {
                console.error('Could not get job status: ' + err.message);
            } else {
                console.log('Job: ' + jobId + ' - status: ' + result.status);
                if (result.status === 'completed' || result.status === 'failed' || result.status === 'cancelled') {
                clearInterval(jobMonitorInterval);
                callback(null, result);
                }
            }
            });
        }, 5000);
    }
    ```
7. 新增下列程式碼來排定呼叫裝置方法的作業：
   
    ```
    var methodParams = {
        methodName: 'lockDoor',
        payload: null,
        responseTimeoutInSeconds: 15 // Timeout after 15 seconds if device is unable to process method
    };
   
    var methodJobId = uuid.v4();
    console.log('scheduling Device Method job with id: ' + methodJobId);
    jobClient.scheduleDeviceMethod(methodJobId,
                                queryCondition,
                                methodParams,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule device method job: ' + err.message);
        } else {
            monitorJob(methodJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor device method job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```
8. 新增下列程式碼來排定更新裝置對應項 (twin) 的作業：
   
    ```
    var twinPatch = {
       etag: '*', 
       properties: {
           desired: {
               building: '43', 
               floor: 3
           }
       }
    };
   
    var twinJobId = uuid.v4();
   
    console.log('scheduling Twin Update job with id: ' + twinJobId);
    jobClient.scheduleTwinUpdate(twinJobId,
                                queryCondition,
                                twinPatch,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule twin update job: ' + err.message);
        } else {
            monitorJob(twinJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor twin update job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```
9. 儲存並關閉 **scheduleJobService.js** 檔案。

## <a name="run-the-applications"></a>執行應用程式
現在您已經準備好執行應用程式。

1. 在命令提示字元中，於 **simDevice** 資料夾中執行下列命令來開始接聽重新啟動直接方法。
   
    ```
    node simDevice.js
    ```
2. 在 **scheduleJobService** 資料夾的命令提示字元中，執行下列命令以觸發鎖門作業並更新對應項
   
    ```
    node scheduleJobService.js
    ```
3. 您會在主控台中看到直接方法的裝置回應。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已使用作業來排定裝置的直接方法，以及更新裝置對應項 (twin) 的屬性。

若要繼續開始使用「IoT 中樞」和裝置管理模式 (例如遠端無線韌體更新)，請參閱︰

[教學課程：如何進行韌體更新][lnk-fwupdate]

若要繼續開始使用 IoT 中樞，請參閱[開始使用 Azure IoT Edge][lnk-iot-edge]。

[lnk-get-started-twin]: iot-hub-node-node-twin-getstarted.md
[lnk-twin-props]: tutorial-device-twins.md
[lnk-c2d-methods]: quickstart-control-device-node.md
[lnk-dev-methods]: iot-hub-devguide-direct-methods.md
[lnk-fwupdate]: tutorial-firmware-update.md
[lnk-iot-edge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
