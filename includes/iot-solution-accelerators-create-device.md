---
title: 包含檔案
description: 包含檔案
services: iot-accelerators
author: dominicbetts
ms.service: iot-accelerators
ms.topic: include
ms.date: 09/28/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 5eb3c08792b760bf66e443f79762d91210706c92
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435107"
---
在第一個情節中，您要將新的遙測類型新增至 Contoso 的現有 **Chiller** 裝置類型。

在第二個情節中，Contoso 需要測試新的智慧型燈泡裝置。 若要執行測試，您可以建立具有下列特性的新模擬裝置：

*屬性*

| 名稱                     | 值                      |
| ------------------------ | --------------------------- |
| 色彩                    | 白色、紅色、藍色            |
| 亮度               | 0 到 100                    |
| 估計的剩餘時間 | 10,000 個小時倒數計時 |

*遙測*

下表顯示燈泡 (lightbulb) 以資料流向雲端報告的資料：

| 名稱   | 值      |
| ------ | ----------- |
| 狀態 | 「開啟」、「關閉」 |
| 溫度 | 華氏度數 |
| 線上 | true、false |

> [!NOTE]
> **線上**遙測值會強制用於所有模擬類型。

*方法*

下表會顯示新裝置所支援的動作：

| 名稱        |
| ----------- |
| 開啟   |
| 關閉  |

*初始狀態*

下表會顯示裝置的初始狀態：

| 名稱                     | 值 |
| ------------------------ | -------|
| 初始色彩            | 白色  |
| 初始亮度       | 75     |
| 初始剩餘時間   | 10,000 |
| 初始遙測狀態 | 「開啟」   |
| 初始遙測溫度 | 200   |

若要完成此操作指南中的步驟，您必須具備有效的 Azure 訂用帳戶。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](cloud-shell-try-it.md)]

## <a name="prerequisites"></a>必要條件

若要遵循本操作說明指南，您需要：

* Visual Studio Code。 您可以[下載適用於 Mac、Linux 和 Windows 的 Visual Studio Code](https://code.visualstudio.com/download)。
* .NET Core。 您可以下載[適用於 Mac、Linux 和 Windows 的 .NET Code](https://www.microsoft.com/net/download)。
* [C# for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)
* Postman。 您可以下載[適用於 Mac、Windows 或 Linux 的 Postman](https://www.getpostman.com/apps)。
* [部署至 Azure 訂用帳戶的 IoT 中樞](../articles/iot-hub/iot-hub-create-through-portal.md)。 您需要 IoT 中樞的連接字串才能完成本指南中的步驟。 您可以從 Azure 入口網站取得連接字串。
* Cosmos DB 資料庫，其必須使用 SQL API，並已針對[強式一致性](../articles/cosmos-db/manage-account.md)進行設定。 您需要 Cosmos DB 資料庫的連接字串才能完成本指南中的步驟。 您可以從 Azure 入口網站取得連接字串。

## <a name="prepare-your-development-environment"></a>準備您的開發環境

請完成下列工作以準備開發環境：

* 下載裝置模擬微服務的來源。
* 下載儲存體配接器微服務的來源。
* 本機執行儲存體配接器微服務。

本文中的指示假設您使用的是 Windows。 如果您使用其他作業系統，則可能需要調整某些檔案路徑和命令以符合環境。

### <a name="download-the-microservices"></a>下載微服務

從 GitHub 將[遠端監視微服務](https://github.com/Azure/remote-monitoring-services-dotnet/archive/master.zip)下載並解壓縮到本機電腦上的適當位置。 本文假設這個資料夾的名稱是 **remote-monitoring-services-dotnet-master**。

從 GitHub 將[裝置模擬微服務](https://github.com/Azure/device-simulation-dotnet/archive/master.zip)下載並解壓縮到本機電腦上的適當位置。 本文假設這個資料夾的名稱是 **device-simulation-dotnet-master**。

### <a name="run-the-storage-adapter-microservice"></a>執行儲存體配接器微服務

在 Visual Studio Code 中開啟 **remote-monitoring-services-dotnet-master\storage-adapter** 資料夾。 按一下任何 [還原] 按鈕，以修正任何無法解決的相依性。

開啟 **.vscode/launch.json** 檔案，並將 Cosmos 連接字串指派到 **PCS_STORAGEADAPTER_DOCUMENTDB_CONNSTRING** 環境變數。

<!-- Open the **WebService/appsettings.ini** file and assign your Cosmos DB connection string to the **documentdb_connstring** configuration setting.-->

若要在本機執行微服務，請按一下 [偵錯] > [開始偵錯]。

Visual Studio Code 中的 [終端機] 視窗會顯示執行中微服務的輸出，包括 Web 服務健康情況檢查的 URL：[http://127.0.0.1:9022/v1/status](http://127.0.0.1:9022/v1/status)。 當您瀏覽至此位址時，狀態應該是「正確: 運作良好」。

在完成後續步驟時，請讓儲存體配接器微服務繼續在 Visual Studio Code 的這個執行個體中執行。

## <a name="modify-the-chiller"></a>修改 Chiller

在本節中，您會將新的**內部溫度**遙測類型新增至現有 **Chiller** 裝置類型：

1. 在本機電腦上建立新的資料夾 **C:\temp\devicemodels**。

1. 將下列檔案複製到所下載裝置模擬微服務複本中的新資料夾：

    | 來源 | 目的地 |
    | ------ | ----------- |
    | Services\data\devicemodels\chiller-01.json | C:\temp\devicemodels\chiller-01.json |
    | Services\data\devicemodels\scripts\chiller-01-state.js | C:\temp\devicemodels\scripts\chiller-01-state.js |
    | Services\data\devicemodels\scripts\Reboot-method.js | C:\temp\devicemodels\scripts\Reboot-method.js |
    | Services\data\devicemodels\scripts\FirmwareUpdate-method.js | C:\temp\devicemodels\scripts\FirmwareUpdate-method.js |
    | Services\data\devicemodels\scripts\EmergencyValveRelease-method.js | C:\temp\devicemodels\scripts\EmergencyValveRelease-method.js |
    | Services\data\devicemodels\scripts\IncreasePressure-method.js | C:\temp\devicemodels\scripts\IncreasePressure-method.js |

1. 開啟 **C:\temp\devicemodels\chiller-01.json** 檔案。

1. 在 **InitialState** 區段中，新增下列兩個定義：

    ```json
    "internal_temperature": 65.0,
    "internal_temperature_unit": "F",
    ```

1. 在**遙測** 陣列中，新增下列定義：

    ```json
    {
      "Interval": "00:00:05",
      "MessageTemplate": "{\"internal_temperature\":${internal_temperature},\"internal_temperature_unit\":\"${internal_temperature_unit}\"}",
      "MessageSchema": {
        "Name": "chiller-internal-temperature;v1",
        "Format": "JSON",
        "Fields": {
          "temperature": "double",
          "temperature_unit": "text"
        }
      }
    },
    ```

1. 儲存 **C:\temp\devicemodels\chiller-01.json** 檔案。

1. 開啟 **C:\temp\devicemodels\scripts\chiller-01-state.js** 檔案。

1. 將下列欄位新增至 **state** 變數：

    ```js
    internal_temperature: 65.0,
    internal_temperature_unit: "F",
    ```

1. 更新 **main** 函式，如下所示：

    ```js
    function main(context, previousState, previousProperties) {

        // Restore the global state before generating the new telemetry, so that
        // the telemetry can apply changes using the previous function state.
        restoreSimulation(previousState, previousProperties);

        // 75F +/- 5%,  Min 25F, Max 100F
        state.temperature = vary(75, 5, 25, 100);

        // 70% +/- 5%,  Min 2%, Max 99%
        state.humidity = vary(70, 5, 2, 99);

        // 65F +/- 2%,  Min 15F, Max 125F
        state.internal_temperature = vary(65, 2, 15, 125);

        log("Simulation state: " + state.simulation_state);
        if (state.simulation_state === "high_pressure") {
            // 250 psig +/- 25%,  Min 50 psig, Max 300 psig
            state.pressure = vary(250, 25, 50, 300);
        } else {
            // 150 psig +/- 10%,  Min 50 psig, Max 300 psig
            state.pressure = vary(150, 10, 50, 300);
        }

        updateState(state);
        return state;
    }
    ```

1. 儲存 **C:\temp\devicemodels\scripts\chiller-01-state.js** 檔案。

## <a name="create-the-lightbulb"></a>建立燈泡

在本節中，您會定義新的**燈泡**裝置類型：

1. 建立 **C:\temp\devicemodels\lightbulb-01.json** 檔案，並新增下列內容：

    ```json
    {
      "SchemaVersion": "1.0.0",
      "Id": "lightbulb-01",
      "Version": "0.0.1",
      "Name": "Lightbulb",
      "Description": "Smart lightbulb device.",
      "Protocol": "MQTT",
      "Simulation": {
        "InitialState": {
          "online": true,
          "temperature": 200.0,
          "temperature_unit": "F",
          "status": "on"
        },
        "Interval": "00:00:20",
        "Scripts": [
          {
            "Type": "javascript",
            "Path": "lightbulb-01-state.js"
          }
        ]
      },
      "Properties": {
        "Type": "Lightbulb",
        "Color": "White",
        "Brightness": 75,
        "EstimatedRemainingLife": 10000
      },
      "Tags": {
        "Location": "Building 2",
        "Floor": "2",
        "Campus": "Redmond"
      },
      "Telemetry": [
        {
          "Interval": "00:00:20",
          "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"status\":\"${status}\"}",
          "MessageSchema": {
            "Name": "lightbulb-status;v1",
            "Format": "JSON",
            "Fields": {
              "temperature": "double",
              "temperature_unit": "text",
              "status": "text"
            }
          }
        }
      ],
      "CloudToDeviceMethods": {
        "SwitchOn": {
          "Type": "javascript",
          "Path": "SwitchOn-method.js"
        },
        "SwitchOff": {
          "Type": "javascript",
          "Path": "SwitchOff-method.js"
        }
      }
    }
    ```

    將變更儲存到 **C:\temp\devicemodels\lightbulb-01.json**。

1. 建立 **C:\temp\devicemodels\scripts\lightbulb-01-state.js** 檔案，並新增下列內容：

    ```javascript
    "use strict";

    // Default state
    var state = {
      online: true,
      temperature: 200.0,
      temperature_unit: "F",
      status: "on"
    };

    // Default device properties
    var properties = {};

    /**
     * Restore the global state using data from the previous iteration.
     *
     * @param previousState device state from the previous iteration
     * @param previousProperties device properties from the previous iteration
     */
    function restoreSimulation(previousState, previousProperties) {
      // If the previous state is null, force a default state
      if (previousState) {
        state = previousState;
      } else {
        log("Using default state");
      }

      if (previousProperties) {
        properties = previousProperties;
      } else {
        log("Using default properties");
      }
    }

    /**
     * Simple formula generating a random value around the average
     * in between min and max
     *
     * @returns random value with given parameters
     */
    function vary(avg, percentage, min, max) {
      var value = avg * (1 + ((percentage / 100) * (2 * Math.random() - 1)));
      value = Math.max(value, min);
      value = Math.min(value, max);
      return value;
    }

    /**
     * Simple formula that sometimes flips the status of the lightbulb
     */
    function flip(value) {
      if (Math.random() < 0.2) {
        return (value == "on") ? "off" : "on"
      }
      return value;
    }

    /**
     * Entry point function called by the simulation engine.
     * Returns updated simulation state.
     * Device property updates must call updateProperties() to persist.
     *
     * @param context             The context contains current time, device model and id
     * @param previousState       The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState, previousProperties) {

      // Restore the global device properties and the global state before
      // generating the new telemetry, so that the telemetry can apply changes
      // using the previous function state.
      restoreSimulation(previousState, previousProperties);

      state.temperature = vary(200, 5, 150, 250);

      // Make this flip every so often
      state.status = flip(state.status);

      updateState(state);

      return state;
    }
    ```

    將變更儲存到 **C:\temp\devicemodels\scripts\lightbulb-01-state.js**。

1. 建立 **C:\temp\devicemodels\scripts\SwitchOn-method.js** 檔案，並新增下列內容：

    ```javascript
    "use strict";

    // Default state
    var state = {
      status: "on"
    };

    /**
     * Entry point function called by the method.
     *
     * @param context        The context contains current time, device model and id
     * @param previousState  The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState) {
      log("Executing lightbulb Switch On method.");
      state.status = "on";
      updateState(state);
    }
    ```

    將變更儲存到 **C:\temp\devicemodels\scripts\SwitchOn-method.js**。

1. 建立 **C:\temp\devicemodels\scripts\SwitchOff-method.js** 檔案，並新增下列內容：

    ```javascript
    "use strict";

    // Default state
    var state = {
      status: "on"
    };

    /**
     * Entry point function called by the method.
     *
     * @param context        The context contains current time, device model and id
     * @param previousState  The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState) {
      log("Executing lightbulb Switch Off method.");
      state.status = "off";
      updateState(state);
    }
    ```

    將變更儲存到 **C:\temp\devicemodels\scripts\SwitchOff-method.js**。

您現在已建立自訂的 **Chiller** 裝置類型版本，並已建立新的**燈泡**裝置類型。

## <a name="test-the-devices"></a>測試裝置

在本節中，您會測試在先前章節中於本機建立的裝置類型。

### <a name="run-the-device-simulation-microservice"></a>執行裝置模擬微服務

在 Visual Studio Code 的新執行個體中，開啟您從 GitHub 下載的 **device-simulation-dotnet-master** 資料夾。 按一下任何 [還原] 按鈕，以修正任何無法解決的相依性。

開啟 **.vscode/launch.json** 檔案，並將 IoT 中樞連接字串指派到 **PCS_IOTHUB_CONNSTRING** 環境變數。 在相同檔案中，新增 **PCS_STORAGEADAPTER_DOCUMENTDB_CONNSTRING** 環境變數，並且指派 Cosmos DB 資料庫的連接字串。

開啟 **WebService/Properties/launchSettings.json** 檔案，並將 IoT 中樞連接字串指派到 **PCS_IOTHUB_CONNSTRING** 環境變數。

開啟 **WebService/appsettings.ini** 檔案，並如下所示地修改設定：

```ini
device_models_folder = C:\temp\devicemodels\

device_models_scripts_folder = C:\temp\devicemodels\scripts\
```

若要在本機執行微服務，請按一下 [偵錯] > [開始偵錯]。

Visual Studio Code 中的 [終端機] 視窗會顯示執行中微服務的輸出。

在完成後續步驟時，請讓裝置模擬微服務繼續在 Visual Studio Code 的這個執行個體中執行。

### <a name="set-up-a-monitor-for-device-events"></a>設定裝置事件監視器

在本節中，您會使用 Azure CLI 來設定事件監視器，以檢視從連線至 IoT 中樞的裝置所傳來的遙測資料。

下列指令碼假設 IoT 中樞的名稱是 **device-simulation-test**。

```azurecli-interactive
# Install the IoT extension if it's not already installed
az extension add --name azure-cli-iot-ext

# Monitor telemetry sent to your hub
az iot hub monitor-events --hub-name device-simulation-test
```

在測試模擬裝置時，請讓事件監視器繼續執行。

### <a name="create-a-simulation-with-the-updated-chiller-device-type"></a>使用更新的 Chiller 裝置類型來建立模擬

在本節中，您會使用 Postman 工具來要求裝置模擬微服務，以使用更新的 Chiller 裝置類型來執行模擬。 Postman 這個工具可讓您傳送 REST 要求給 Web 服務。 您所需的 Postman 設定檔位於 **device-simulation-dotnet** 存放庫的本機複本。

若要設定 Postman：

1. 在本機電腦上開啟 Postman。

1. 按一下 [檔案] > [匯入]。 然後按一下 [選擇檔案]。

1. 瀏覽至 **device-simulation-dotnet-master/docs/postman** 資料夾。 選取 [Azure IoT Device Simulation solution accelerator.postman_collection] 和 [Azure IoT Device Simulation solution accelerator.postman_environment]，然後按一下 [開啟]。

1. 將 **Azure IoT Device Simulation solution accelerator** 展開至您可以傳送的要求。

1. 按一下 [無環境]，然後選取 [Azure IoT Device Simulation solution accelerator]。

現在，您已在 Postman 工作區中載入集合和環境，以供用來與裝置模擬微服務互動。

若要設定和執行模擬：

1. 在 Postman 集合中，選取 [建立已修改的 Chiller 模擬]，然後按一下 [傳送]。 此要求會針對模擬的 Chiller 裝置類型建立四個執行個體。

1. Azure CLI 視窗中的事件監視器輸出會顯示模擬裝置的遙測資料，包括新的 **internal_temperature** 值。

若要停止模擬，請在 Postman 中選取 [停止模擬] 要求，然後按一下 [傳送]。

### <a name="create-a-simulation-with-the-lightbulb-device-type"></a>使用燈泡裝置類型來建立模擬

在本節中，您會使用 Postman 工具來要求裝置模擬微服務，以使用燈泡裝置類型來執行模擬。 Postman 這個工具可讓您傳送 REST 要求給 Web 服務。

若要設定和執行模擬：

1. 在 Postman 集合中，選取 [建立燈泡模擬]，然後按一下 [傳送]。 此要求會針對模擬的燈泡裝置類型建立兩個執行個體。

1. Azure CLI 視窗中的事件監視器輸出會顯示模擬燈泡的遙測資料。

若要停止模擬，請在 Postman 中選取 [停止模擬] 要求，然後按一下 [傳送]。

## <a name="clean-up-resources"></a>清除資源

您可以停止兩個在其 Visual Studio Code 執行個體中本機執行的微服務 ([偵錯] > [停止偵錯])。

如果您不再需要 IoT 中樞和 Cosmos DB 執行個體，請從 Azure 訂用帳戶中加以刪除，以避免產生任何不必要的費用。