---
title: 開始使用 Azure IoT 中樞模組身分識別和模組對應項 (Python) | Microsoft Docs
description: 了解如何使用適用於 Python 的 IoT SDK，建立模組身分識別及更新模組對應項。
author: chrissie926
manager: ''
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: menchi
ms.openlocfilehash: 615dfc789db805e51ed3e7c11fed9da6d7079e96
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319093"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-python-back-end-and-python-device"></a>透過 Python 後端和 Python 裝置開始使用 IoT 中樞模組身分識別和模組對應項

> [!NOTE]
> [模組身分識別與模組對應項](iot-hub-devguide-module-twins.md)類似於 Azure IoT 中樞裝置身分識別與裝置對應項，但提供更精細的細微性。 雖然 Azure IoT 中樞裝置身分識別與裝置對應項可讓後端應用程式設定裝置並提供裝置狀況的可見性，但模組身分識別和模組對應項可提供裝置個別元件的上述功能。 在具備多個元件的可用裝置 (例如以作業系統為基礎的裝置或韌體裝置) 上，它允許每個元件有獨立的組態和狀況。
>

在本教學課程結尾，您將會有兩個 Python 應用程式：

* **CreateIdentities**，這會建立裝置身分識別、模組身分識別和相關聯的安全性金鑰，以連線到您的裝置和模組用戶端。
* **UpdateModuleTwinReportedProperties**，這會將已更新的模組對應項報告的屬性傳送到您的 IoT 中樞。

> [!NOTE]
> 如需可用來建置兩個應用程式，以在裝置與您的解決方案後端執行之 Azure IoT SDK 的相關資訊，請參閱 [Azure IoT SDK][lnk-hub-sdks]。
>

若要完成此教學課程，您需要下列項目：

* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)
* IoT 中樞。
* 安裝最新的 [Python SDK](https://github.com/Azure/azure-iot-sdk-python)。

您現在已經建立 IoT 中樞，因此您已具有完成本教學課程的其餘部分所需的主機名稱和 IoT 中樞連接字串。

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>在 IoT 中樞中建立裝置身分識別與模組身分識別

在本節中，您會建立 Python 應用程式，它會在 IoT 中樞的身分識別登錄中建立裝置身分識別和模組身分識別。 裝置或模組無法連線到 IoT 中樞，除非它在身分識別登錄中具有項目。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南][lnk-devguide-identity]的＜身分識別登錄＞一節。 當您執行此主控台應用程式時，它會針對裝置和模組產生唯一的識別碼和金鑰。 當裝置和模組將裝置到雲端的訊息傳送給 IoT 中樞時，裝置和模組會使用這些值來識別自己。 識別碼會區分大小寫。

將下列程式碼新增至 Python 檔案：

```python
import sys
import iothub_service_client
from iothub_service_client import IoTHubRegistryManager, IoTHubRegistryManagerAuthMethod, IoTHubError

CONNECTION_STRING = "YourConnString"
DEVICE_ID = "myFirstDevice"
MODULE_ID = "myFirstModule"

try:
    # RegistryManager
    iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

    # CreateDevice
    primary_key = ""
    secondary_key = ""
    auth_method = IoTHubRegistryManagerAuthMethod.SHARED_PRIVATE_KEY
    new_device = iothub_registry_manager.create_device(DEVICE_ID, primary_key, secondary_key, auth_method)
    print("new_device <" + DEVICE_ID + "> has primary key = " + new_device.primaryKey)

    # CreateModule
    new_module = iothub_registry_manager.create_module(DEVICE_ID, primary_key, secondary_key, MODULE_ID, auth_method)
    print("device/new_module <" + DEVICE_ID + "/" + MODULE_ID + "> has primary key = " + new_module.primaryKey)

except IoTHubError as iothub_error:
    print ( "Unexpected error {0}".format(iothub_error) )
except KeyboardInterrupt:
    print ( "IoTHubRegistryManager sample stopped" )
```

此應用程式會在 **myFirstDevice** 裝置下方建立識別碼為 **myFirstDevice** 的裝置身分識別，以及識別碼為 **myFirstModule** 的模組身分識別。 (如果該模組識別碼已經存在身分識別登錄中，程式碼就只會擷取現有的模組資訊)。接著，應用程式會顯示該身分識別的主要金鑰。 您會在模擬模組應用程式中使用此金鑰來連線到您的 IoT 中樞。

> [!NOTE]
> IoT 中樞身分識別登錄只會儲存裝置和模組身分識別，以啟用對 IoT 中樞的安全存取。 身分識別登錄會儲存裝置識別碼和金鑰，以作為安全性認證使用。 身分識別登錄也會儲存每個裝置的已啟用/已停用旗標，以便您用來停用該裝置的存取權。 如果您的應用程式需要儲存其他裝置特定的中繼資料，它應該使用應用程式專用的存放區。 模組身分識別沒有啟用/停用旗標。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南][lnk-devguide-identity]。
>

## <a name="update-the-module-twin-using-python-device-sdk"></a>使用 Python 裝置 SDK 更新模組對應項

在本節中，您會在模擬裝置上建立 Python 應用程式，以便更新模組對應項報告的屬性。

1. **取得您的模組連接字串** - 如果您登入 [Azure 入口網站][lnk-portal]，請立即取得。 瀏覽至您的 IoT 中樞並按一下 IoT 裝置。 尋找 myFirstDevice 並加以開啟，您會看到已成功建立 myFirstModule。 複製模組連接字串。 在下一個步驟中需要用到它。

  ![Azure 入口網站模組詳細資料][15]

1. **建立 UpdateModuleTwinReportedProperties 應用程式** 在 **Program.cs** 檔的頂端新增下列 `using` 陳述式：

    ```python
    import sys
    import iothub_service_client
    from iothub_service_client import IoTHubRegistryManager, IoTHubRegistryManagerAuthMethod, IoTHubDeviceTwin, IoTHubError

    CONNECTION_STRING = "FILL IN CONNECTION STRING"
    DEVICE_ID = "MyFirstDevice"
    MODULE_ID = "MyFirstModule"

    UPDATE_JSON = "{\"properties\":{\"desired\":{\"telemetryInterval\":122}}}"

    try:
        iothub_twin = IoTHubDeviceTwin(CONNECTION_STRING)

        twin_info = iothub_twin.get_twin(DEVICE_ID, MODULE_ID)
        print ( "" )
        print ( "Twin before update    :" )
        print ( "{0}".format(twin_info) )

        twin_info_updated = iothub_twin.update_twin(DEVICE_ID, MODULE_ID, UPDATE_JSON)
        print ( "" )
        print ( "Twin after update     :" )
        print ( "{0}".format(twin_info_updated) )

    except IoTHubError as iothub_error:
        print ( "Unexpected error {0}".format(iothub_error) )
    except KeyboardInterrupt:
        print ( "IoTHubRegistryManager sample stopped" )
    ```

此程式碼範例顯示如何擷取模組對應項，並使用 AMQP 通訊協定更新報告的屬性。 

## <a name="get-updates-on-the-device-side"></a>取得裝置端的更新

除了上述程式碼以外，您也可以新增下列程式碼區塊，以在裝置上取得對應項更新訊息。

```python
import random
import time
import sys
import iothub_client
from iothub_client import IoTHubModuleClient, IoTHubClientError, IoTHubTransportProvider, IoTHubClientResult

PROTOCOL = IoTHubTransportProvider.AMQP
CONNECTION_STRING = ""

def module_twin_callback(update_state, payload, user_context):
    print ("")
    print ("Twin callback called with:")
    print ("updateStatus: %s" % update_state )
    print ("context: %s" % user_context )
    print ("payload: %s" % payload )

try:
    module_client = IoTHubModuleClient(CONNECTION_STRING, PROTOCOL)
    module_client.set_module_twin_callback(module_twin_callback, 1234)

    print ("Waiting for incoming twin messages.  Hit Control-C to exit.")
    while True:

        time.sleep(1000000)

except IoTHubError as iothub_error:
    print ( "Unexpected error {0}".format(iothub_error) )
except KeyboardInterrupt:
    print ( "module client sample stopped" )
```


## <a name="next-steps"></a>後續步驟

若要繼續開始使用 IoT 中樞並瀏覽其他 IoT 案例，請參閱︰

* [開始使用裝置管理][lnk-device-management]
* [開始使用 IoT Edge][lnk-iot-edge]


<!-- Images. -->
[15]:./media\iot-hub-csharp-csharp-module-twin-getstarted/module-detail.JPG
<!-- Links -->
[lnk-hub-sdks]: iot-hub-devguide-sdks.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal]: https://portal.azure.com/

[lnk-device-management]: iot-hub-node-node-device-management-get-started.md
[lnk-iot-edge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-devguide-identity]: iot-hub-devguide-identity-registry.md
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
