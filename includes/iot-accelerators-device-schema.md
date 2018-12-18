---
title: 包含檔案
description: 包含檔案
services: iot-accelerators
author: dominicbetts
ms.service: iot-accelerators
ms.topic: include
ms.date: 07/26/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: dc87079083b8f07ad18f5f871bff64de8d492ebd
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39285641"
---
## <a name="the-parts-of-the-device-model-schema"></a>裝置機型結構描述的部分

每個裝置機型 (例如 chiller 或 truck) 都會定義模擬服務可模擬的裝置類型。 每個裝置機型是以下列最上層結構描述儲存在 JSON 檔案中：

```json
{
  "SchemaVersion": "1.0.0",
  "Id": "elevator-01",
  "Version": "0.0.1",
  "Name": "Elevator",
  "Description": "Elevator with floor, vibration and temperature sensors.",
  "Protocol": "AMQP",
  "Simulation": {
    // Specify the simulation behavior
  },
  "Properties": {
    // Define properties
  },
  "Telemetry": [
    // Specify telemetry
  ],
  "CloudToDeviceMethods": {
    // Specify methods
  }
}
```

您可以在 GitHub 上的 [devicemodels 資料夾](https://github.com/Azure/device-simulation-dotnet/tree/master/Services/data/devicemodels)中檢視預設模擬裝置的結構描述檔案。

下表描述最上層結構描述項目：

| 結構描述項目 | 說明 |
| -- | --- |
| `SchemaVersion` | 結構描述版本一律為 `1.0.0` 且為此檔案格式專屬。 |
| `Id` | 此裝置機型的唯一 ID。 |
| `Version` | 識別裝置機型的版本。 |
| `Name` | 裝置機型的易記名稱。 |
| `Description` | 裝置機型的說明。 |
| `Protocol` | 裝置使用的連線通訊協定。 可以是 `AMQP`、`MQTT` 和 `HTTP` 其中一個。 |

下列章節說明 JSON 結構描述中的其他區段：

## <a name="simulation"></a>模擬

在 `Simulation` 區段中，您定義模擬裝置的內部狀態。 裝置傳送的任何遙測值都必須是此裝置狀態的一部分。

裝置狀態的定義具有兩個元素：

* `InitialState` 定義裝置狀態物件之所有屬性的初始值。
* `Script` 識別排程執行以更新裝置狀態的 JavaScript 檔案。 您可以使用此指令碼檔案讓裝置傳送的遙測值隨機。

若要了解有關更新裝置狀態物件之 JavaScript 檔案的詳細資訊，請參閱[了解裝置機型行為](../articles/iot-accelerators/iot-accelerators-device-simulation-device-behavior.md)。

下列範例會顯示模擬 chiller 裝置之裝置狀態物件的定義：

```json
"Simulation": {
  "InitialState": {
    "online": true,
    "temperature": 75.0,
    "temperature_unit": "F",
    "humidity": 70.0,
    "humidity_unit": "%",
    "pressure": 150.0,
    "pressure_unit": "psig",
    "simulation_state": "normal_pressure"
  },
  "Interval": "00:00:10",
  "Scripts": {
    "Type": "javascript",
    "Path": "chiller-01-state.js"
  }
}
```

模擬服務每 5 秒執行 **chiller-01-state.js** 檔案以更新裝置狀態。 您可以在 GitHub 上的 [scripts 資料夾](https://github.com/Azure/device-simulation-dotnet/tree/master/Services/data/devicemodels/scripts)中查看預設模擬裝置的 JavaScript 檔案。 依照慣例，這些 JavaScript 檔案具有尾碼 **-state**，以便與實作方法行為的檔案有所區別。

## <a name="properties"></a>properties

結構描述的 `Properties` 區段會定義裝置回報給解決方案的屬性值。 例如︰

```json
"Properties": {
  "Type": "Elevator",
  "Location": "Building 2",
  "Latitude": 47.640792,
  "Longitude": -122.126258
}
```

解決方案啟動時，它會查詢所有模擬裝置，以建置一份在 UI 中使用的 `Type` 值清單。 解決方案會使用 `Latitiude` 和 `Longitude` 屬性，以將裝置的位置新增至儀表板上的地圖。

## <a name="telemetry"></a>遙測

`Telemetry` 陣列會列出模擬裝置傳送至解決方案的所有遙測類型。

下列範例會每 10 秒傳送具有電梯感應器之 `floor`、`vibration` 和 `temperature` 資料的 JSON 遙測訊息：

```json
"Telemetry": [
  {
    "Interval": "00:00:10",
    "MessageTemplate": "{\"floor\":${floor},\"vibration\":${vibration},\"vibration_unit\":\"${vibration_unit}\",\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\"}",
    "MessageSchema": {
      "Name": "elevator-sensors;v1",
      "Format": "JSON",
      "Fields": {
        "floor": "integer",
        "vibration": "double",
        "vibration_unit": "text",
        "temperature": "double",
        "temperature_unit": "text"
      }
    }
  }
]
```

`MessageTemplate` 會定義模擬裝置所傳送之 JSON 訊息的結構。 `MessageTemplate` 中的預留位置會使用語法 `${NAME}`，其中 `NAME` 是來自[裝置狀態物件](#simulation)的索引鍵。 字串應該加上引號，數字則不應該。

`MessageSchema` 會定義模擬裝置所傳送之訊息的結構描述。 訊息結構描述也會發行到 IoT 中樞，讓後端應用程式重複使用資訊以解譯內送遙測。

目前，您只能使用 JSON 訊息結構描述。 結構描述中所列的欄位可以是下列類型：

* 物件 - 使用 JSON 序列化
* 二進位 - 使用 base64 序列化
* 文字
* BOOLEAN
* 整數 
* 兩倍
* Datetime

若要在不同的間隔傳送遙測訊息，請將多個遙測類型新增至 `Telemetry` 陣列。 下列範例會每 10 秒傳送溫度和溼度資料，每一分鐘傳送光線狀態：

```json
"Telemetry": [
  {
    "Interval": "00:00:10",
    "MessageTemplate":
      "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"humidity\":\"${humidity}\"}",
    "MessageSchema": {
      "Name": "RoomComfort;v1",
      "Format": "JSON",
      "Fields": {
        "temperature": "double",
        "temperature_unit": "text",
        "humidity": "integer"
      }
    }
  },
  {
    "Interval": "00:01:00",
    "MessageTemplate": "{\"lights\":${lights_on}}",
    "MessageSchema": {
      "Name": "RoomLights;v1",
      "Format": "JSON",
      "Fields": {
        "lights": "boolean"
      }
    }
  }
],
```

## <a name="cloudtodevicemethods"></a>CloudToDeviceMethods

模擬裝置可以回應從 IoT 中樞呼叫的雲端到裝置方法。 裝置機型結構描述檔案中的 `CloudToDeviceMethods` 區段：

* 定義模擬裝置可以回應的方法。
* 識別 JavaScript 檔案，其中包含要執行的邏輯。

模擬裝置會將其支援的方法清單傳送給所連線的 IoT 中樞。

若要了解有關實作裝置行為之 JavaScript 檔案的詳細資訊，請參閱[了解裝置機型行為](../articles/iot-accelerators/iot-accelerators-device-simulation-device-behavior.md)。

下列範例會指定三個支援的方法和實作這些方法的 JavaScript 檔案：

```json
"CloudToDeviceMethods": {
  "Reboot": {
    "Type": "javascript",
    "Path": "Reboot-method.js"
  },
  "EmergencyValveRelease": {
    "Type": "javascript",
    "Path": "EmergencyValveRelease-method.js"
  },
  "IncreasePressure": {
    "Type": "javascript",
    "Path": "IncreasePressure-method.js"
  }
}
```

您可以在 GitHub 上的 [scripts 資料夾](https://github.com/Azure/device-simulation-dotnet/tree/master/Services/data/devicemodels/scripts)中查看預設模擬裝置的 JavaScript 檔案。 依照慣例，這些 JavaScript 檔案具有尾碼 **-method**，以便與實作狀態行為的檔案有所區別。