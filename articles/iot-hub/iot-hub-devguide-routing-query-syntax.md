---
title: 對 Azure IoT 中樞訊息路由進行查詢 | Microsoft Docs
description: 開發人員指南 - Azure IoT 中樞上之訊息路由的查詢語法。
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: asrastog
ms.openlocfilehash: 8e9321e72727c1a3149ff2e78b8cb1248734cb88
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978500"
---
# <a name="iot-hub-message-routing-query-syntax"></a>IoT 中樞訊息路由查詢語法

訊息路由可讓使用者將不同的資料類型，也就是裝置遙測訊息、裝置生命週期事件，以及裝置對應項變更事件路由至各個端點。 您也可以在路由此資料之前先套用豐富查詢，以僅接收對您最重要的資料。 本文會說明 IoT 中樞訊息路由查詢語言，並提供一些常見的查詢模式 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

訊息路由可讓您對訊息屬性和訊息本文，以及裝置對應項標記和裝置對應項屬性進行查詢。 如果訊息本文不是 JSON，訊息路由仍可以路由該訊息，但將無法把查詢套用至訊息本文。  查詢會被描述為布林運算式，其中布林值為 True 會使查詢成功，並路由所有傳入資料，而布林值為 False 則會使查詢失敗，且不會路由任何資料。 如果運算式評估為 Null 或未定義，系統會將它視為 False，並在發生失敗的情況下於診斷記錄中產生錯誤。 查詢語法必須正確以儲存並評估路由。  

## <a name="message-routing-query-based-on-message-properties"></a>根據訊息屬性的訊息路由查詢 

IoT 中樞會針對所有裝置到雲端訊息定義[常見格式](../iot-hub/iot-hub-devguide-messages-construct.md)，以在通訊協定之間提供互通性。 IoT 中樞訊息會針對訊息假設下列 JSON 表示法。 系統會針對訊息的所有使用者和身分識別內容新增系統屬性。 使用者可以選擇性地將應用程式屬性新增至訊息。 我們建議使用唯一屬性名稱，因為 IoT 中樞的裝置到雲端訊息並不會區分大小寫。 例如，如果您有多個具相同名稱的屬性，IoT 中樞將只會傳送其中一個屬性。  

```json
{ 
  "message": { 
    "systemProperties": { 
      "contentType": "application/json", 
      "contentEncoding": "UTF-8", 
      "iothub-message-source": "deviceMessages", 
      "iothub-enqueuedtime": "2017-05-08T18:55:31.8514657Z" 
    }, 
    "appProperties": { 
      "processingPath": "{cold | warm | hot}", 
      "verbose": "{true, false}", 
      "severity": 1-5, 
      "testDevice": "{true | false}" 
    }, 
    "body": "{\"Weather\":{\"Temperature\":50}}" 
  } 
} 
```
### <a name="system-properties"></a>系統屬性

系統屬性可協助識別訊息的內容和來源。 

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| contentType | 字串 | 使用者會指定訊息的內容類型。 若要允許對訊息本文進行查詢，此值應該設為 application/JSON。 |
| contentEncoding | 字串 | 使用者會指定訊息的編碼類型。 允許的值為 UTF-8、UTF-16、UTF-32 (若 contentType 是設為 application/JSON)。 |
| connectionDeviceId | 字串 | 此值是由 IoT 中樞設定，並能識別訊息的來源。 這可以是裝置遙測訊息、裝置對應項變更通知，或裝置生命週期事件。 此項目無法被查詢。 |
| iothub-enqueuedtime | 字串 | 此值是由 IoT 中樞設定，並代表將訊息加入佇列的實際時間 (以 UTC 表示)。 若要查詢，請使用 `'enqueuedTime'`。 |

如 [IoT 中樞訊息](iot-hub-devguide-messages-construct.md)中所述，訊息中還有其他系統屬性。 除了 **contentType**、**contentEncoding** 和 **enqueuedTime** 之外，可查詢的屬性還包括 **connectionDeviceId** 和 **connectionModuleId**。

### <a name="application-properties"></a>應用程式屬性

應用程式屬性為可新增至訊息的使用者定義字串。 這些欄位為選擇性。  

### <a name="query-expressions"></a>查詢運算式

針對訊息系統屬性的查詢，需要其前面加上 `'$'` 符號。 針對應用程式屬性的查詢是透過屬性名稱來存取，因此不應該其前面加上 `'$'` 符號。 如果應用程式屬性名稱是以 `'$'` 作為開頭，則 IoT 中樞將會在系統屬性中搜尋它，並在找不到它的情況下於應用程式屬性中尋找它。 例如︰ 

查詢系統屬性 contentEncoding 

```sql
$contentEncoding = 'UTF-8'
```

查詢應用程式屬性 processingPath
```sql
processingPath = 'hot'
```

若要結合這些查詢，您可以使用布林運算式和函式 
```sql
$contentEncoding = 'UTF-8' AND processingPath = 'hot'
```

如需支援的運算子和函式的完整清單，請參閱[運算式和條件](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-query-language#expressions-and-conditions
)

## <a name="message-routing-query-based-on-message-body"></a>根據訊息本文的訊息路由查詢 

若要啟用針對訊息本文的查詢，訊息必須是 UTF-8、UTF-16 或 UTF-32 編碼的 JSON 格式檔案。 `contentType` 必須設定為 `application/JSON`，而 `contentEncoding` 則必須設定為系統屬性中其中一個支援的 UTF 編碼。 如果未指定這些屬性，IoT 中樞將不會針對訊息本文評估查詢運算式。 

下列範例說明如何以經過正確格式化和編碼的 JSON 主體建立訊息： 

```javascript
var messageBody = JSON.stringify(Object.assign({}, {
    "Weather": {
        "Temperature": 50,
        "Time": "2017-03-09T00:00:00.000Z",
        "PrevTemperatures": [
            20,
            30,
            40
        ],
        "IsEnabled": true,
        "Location": {
            "Street": "One Microsoft Way",
            "City": "Redmond",
            "State": "WA"
        },
        "HistoricalData": [
            {
                "Month": "Feb",
                "Temperature": 40
            },
            {
                "Month": "Jan",
                "Temperature": 30
            }
        ]
    }
}));

// Encode message body using UTF-8  
var messageBytes = Buffer.from(messageBody, "utf8");

var message = new Message(messageBytes);

// Set message body type and content encoding 
message.contentEncoding = "utf-8";
message.contentType = "application/json";

// Add other custom application properties   
message.properties.add("Status", "Active");

deviceClient.sendEvent(message, (err, res) => {
    if (err) console.log('error: ' + err.toString());
    if (res) console.log('status: ' + res.constructor.name);
});
```

### <a name="query-expressions"></a>查詢運算式

針對訊息本文的查詢，需要在其前面加上 `$body`。 您可以在查詢運算式中使用本文參考、本文陣列參考或多個本文參考。 您的查詢運算式也可以將本文參考和訊息系統屬性，以及訊息應用程式屬性參考進行結合。 例如，以下是所有有效的查詢運算式： 

```sql
$body.Weather.HistoricalData[0].Month = 'Feb' 
```
```sql
$body.Weather.Temperature = 50 AND $body.Weather.IsEnabled 
```
```sql
length($body.Weather.Location.State) = 2 
```
```sql
$body.Weather.Temperature = 50 AND processingPath = 'hot'
```

## <a name="message-routing-query-based-on-device-twin"></a>根據裝置對應項的訊息路由查詢 

訊息路由可讓您針對[裝置對應項](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-device-twins)標記和屬性 (其為 JSON 物件) 進行查詢。 請注意，不支援針對模組對應項進行查詢。 範例的裝置對應項標記和屬性如下所示。

```JSON
{
    "tags": { 
        "deploymentLocation": { 
            "building": "43", 
            "floor": "1" 
        } 
    }, 
    "properties": { 
        "desired": { 
            "telemetryConfig": { 
                "sendFrequency": "5m" 
            }, 
            "$metadata" : {...}, 
            "$version": 1 
        }, 
        "reported": { 
            "telemetryConfig": { 
                "sendFrequency": "5m", 
                "status": "success" 
            },
            "batteryLevel": 55, 
            "$metadata" : {...}, 
            "$version": 4 
        } 
    } 
} 
```

### <a name="query-expressions"></a>查詢運算式

針對訊息本文的查詢，需要在其前面加上 `$twin`。 您的查詢運算式也可以將對應項標記或屬性參考和本文參考、訊息系統屬性，以及訊息應用程式屬性參考進行結合。 我們建議在標記和屬性中使用唯一名稱，因為查詢不會區分大小寫。 此外，也請避免使用 `twin`、`$twin`、`body` 或 `$body` 作為屬性名稱。 例如，以下是所有有效的查詢運算式： 

```sql
$twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```
```sql
$body.Weather.Temperature = 50 AND $twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```
```sql
$twin.tags.deploymentLocation.floor = 1 
```

## <a name="next-steps"></a>後續步驟

* 了解[訊息路由](iot-hub-devguide-messages-d2c.md)
* 嘗試[訊息路由教學課程](tutorial-routing.md)